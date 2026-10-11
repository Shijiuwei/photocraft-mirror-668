# Photocraft Architecture

Status: draft v1 (2026-09-30). This plan does not assume a final UI toolkit. The candidates are discussed in [`rust-framework-options.md`](rust-framework-options.md).

This plan sets out:
- the Cargo workspace,
- the rules that keep the engine independent of the UI,
- the interfaces a UI has to implement,
- how the PSD, compositing, history and automation parts fit together.

---

## 1. Goals and principles

**Goals**
- A native, portable (macOS / Windows / Linux), fully open-source layered image editor, roughly matching Photoshop's feature surface over time.
- **The UI can be swapped** without touching the core: egui today, maybe Slint, Qt or something else later.
- **The core compiles to `wasm32`,** which keeps a browser demo possible.
- PSD read and write lives in its **own crate that can be published separately.**

**Principles**

1. **Engine-first, headless-first.** Every feature is reachable without a GUI: from tests, the CLI and MCP. The GUI is one client of the engine.
2. **Everything is a command.** Each user-visible action has a stable `CommandId` (such as `filter.blur.gaussianBlur`) with typed, serializable parameters. The menu, command palette, shortcuts, recorded actions/macros, CLI, MCP and plugins all dispatch the same commands.
3. **Data describes the UI; toolkits draw it.** Tools produce *overlay primitives* (lines, handles, marching-ants paths) as data. Filter and adjustment dialogs are generated from *parameter schemas*. A new toolkit implements one renderer for each, not about 150 bespoke dialogs.
4. **One algorithm, one parameter struct.** Every algorithm has a CPU reference implementation (Rust + rayon, deterministic). Some also get a GPU implementation (WGSL). Both read the same `#[repr(C)]` `bytemuck::Pod` parameter struct. We never keep hand-synced copies of an algorithm in several languages.
5. **Immutable snapshots, copy-on-write tiles.** Pixel data is stored in `Arc`-shared 256² tiles. Taking a document snapshot is O(layers), which makes undo, background jobs, autosave and UI reads cheap and lock-free.
6. **Determinism.** The same input gives the same output across CPU and GPU (within tolerance), across thread counts, and between preview and export. Noise is hashed from document coordinates.
7. **Pure Rust by default.** C/C++ dependencies are allowed only behind Cargo features, in I/O-edge crates (for example an optional LibRaw).
8. **Clean-room.** We study other editors' *behaviour* only, and we must **not copy proprietary source** (Rust, WGSL, C++ or JS), even where it ships as readable source. Specs come from public format docs (Adobe PSD spec, ISO/ICC), academic papers (PatchMatch, Poisson blending, ARAP) and observed behaviour. Never paste code from another product.

### 1.1 Avoiding GIMP's hole

The criticism of GIMP is historically true, and much of it still applies in 2026.

**Bit depth.**
- GIMP's original core hard-coded 8-bit-per-channel tiles.
- High bit depth needed a whole new engine, GEGL. GEGL began around 2000, was introduced as optional in 2.6 (2008), and the port only finished in **GIMP 2.10 (2018)**, "the result of six years of work" on that port alone.
- Only then did GIMP get 16/32-bit integer and float processing.

**Non-destructive editing.**
- Photoshop has had adjustment layers since about 1994.
- GIMP got non-destructive *filters* in **3.0 (March 2025)**, about seven years after 2.10 because of the GTK3 port.
- It got non-destructive *layer types* (link layers, vector layers) in **3.2 (March 2026)**.

**CMYK.**
- GIMP 3.x is still "a 100% RGB editor" internally.
- CMYK exists only as export (JPEG/TIFF/PSD) and soft-proofing.

**PSD.**
- Import is still lossy. Photoshop adjustment layers and most layer styles are not carried over faithfully, and text often needs re-typing.
- 3.2 converts legacy drop and inner shadows into GIMP filters, and added PSB export.

**Root cause:** fundamental properties were **baked into the core as assumptions instead of being data.** Pixel depth, colour model, and "a layer is a pixel buffer" were all hard-coded. Every later feature that touched them needed a rewrite of everything built on top.

Sources: [GIMP 2.10 release notes](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html), [GEGL on Wikipedia](https://ncrw.tcti.cn/kaifa/growth-85374259.html), [GIMP 3.2 release notes](https://gifb.spidernode-mesh.net/shichang/about-02526745.html), [LWN on GIMP 3.0 colour](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html).

**Rules we adopt from day one** (cheap now, and each would force a rewrite later):

| Property | Rule | Where |
|---|---|---|
| **Bit depth** | Every surface carries a runtime `PixelFormat { depth: U8\|U16\|F16\|F32, model, alpha }`. Algorithms are written generically over a `Sample` trait and monomorphized per depth, or run on f32 working buffers and convert at tile edges. No `u8` pixel type appears in any public engine API. CI runs the golden suite at 8, 16 and 32f. | `raster`, `color`, `algo` |
| **Colour model** | `ColorMode` (RGB, Gray, CMYK, Lab, Indexed, Bitmap, plus multichannel/spot) is in `doc` from the start. Channel count is dynamic (1–N plus alpha plus spot). Only RGB/Gray *rendering* ships first, but the storage, PSD round-trip and conversion paths never assume 3 or 4 channels. | `doc`, `raster`, `psd` |
| **Colour management** | Every document has an ICC profile. The compositor takes a working-space descriptor. The display transform is a separate final stage. Blending space (gamma vs linear) is a per-document option. | `color`, `compose` |
| **Non-destructive by construction** | `LayerContent` is an enum that includes Adjustment, Fill, Text, Shape and Smart from v1, with smart filters as a stack on any layer. The compositor evaluates a *graph*, not a list of flattened buffers. Destructive ops are just "apply and bake". | `doc`, `compose` |
| **File formats** | Formats plug in through `trait ImageDecoder/Encoder` plus a registry with capability flags (depths, modes, layers, alpha, metadata, ICC). Adding a format is one crate feature. It must never need a change in core types. Lossless PSD round-trip preserves unknown blocks. | `codecs`, `io`, `psd` |
| **Size** | Tiled, sparse storage behind a `TileStore` trait, with 64-bit coordinates for PSB-scale canvases (300k × 300k). Nothing assumes a document fits in one allocation or one GPU texture. | `raster` |
| **UI toolkit** | The core never depends on the toolkit (§3–§4). GIMP spent years porting GTK2 to GTK3, and our UI crate can be replaced without touching the engine. | layering rule 2 |

The test is simple: **adding 64-bit float, a new colour model, or a new file format should touch a leaf crate, not the foundation.** We check this in phase 2 by adding a (hidden) CMYK and a 32f code path early, even before they are exposed in the UI.

---

## 2. Workspace layout

```text
photocraft/
├─ Cargo.toml                  # [workspace], shared deps, lints, profiles
├─ crates/
│  │  ── foundation (no_std-friendly where practical, wasm-safe) ──
│  ├─ geom/                    photocraft-geom      points, rects, affine/perspective, tile coords, bezier (kurbo)
│  ├─ cms/                     photocraft-cms       pure-Rust ICC colour management: profiles, transforms, intents, BPC, soft proof, 3D LUTs
│  ├─ color/                   photocraft-color     pixel formats, color spaces, blend-mode math (scalar reference); conversions via cms
│  ├─ raster/                  photocraft-raster    tiled COW surfaces, masks, mip pyramids, damage regions, pixel iterators
│  │  ── document ──
│  ├─ doc/                     photocraft-doc       document model: layer tree, masks, effects, channels, paths, guides, metadata (pure data + serde)
│  ├─ ops/                     photocraft-ops       reversible operations on a doc + transactions + history/undo
│  │  ── algorithms (CPU reference impls, rayon, wasm-safe) ──
│  ├─ algo/                    photocraft-algo      adjustments, filters, selection, inpaint/heal, warp/transform, resampling
│  ├─ paint/                   photocraft-paint     brush engine: dabs, dynamics, stroke smoothing, brush presets (.abr import later)
│  ├─ text/                    photocraft-text      font DB trait, shaping + layout (parley/skrifa/harfrust), text-layer rasterization
│  ├─ vector/                  photocraft-vector    shapes, paths, strokes → coverage (vello_cpu / kurbo)
│  │  ── rendering ──
│  ├─ compose/                 photocraft-compose   layer tree → DrawOp plan; CPU compositor (reference + export + wasm fallback)
│  ├─ gpu/                     photocraft-gpu       wgpu backend for the DrawOp plan + GPU kernels (WGSL), residency cache, memory budget
│  ├─ viewport/                photocraft-viewport  camera (zoom/pan/rotate), visible tiles/mips, renders a doc into a wgpu texture; toolkit-agnostic
│  │  ── I/O ──
│  ├─ psd/                     photocraft-psd       PSD/PSB read + write; its OWN format-level model; depends on nothing in this workspace
│  ├─ adobe-assets/            photocraft-adobe-assets  .abr .asl .aco/.ase .grd .pat .csh .atn .cube/.3dl, ACR .xmp presets (standalone, like psd)
│  ├─ codecs/                  photocraft-codecs    flat raster decode/encode at format-supported depths; AVIF encode only (feature `avif`), HEIF/HEIC decode only (feature `heif`); no JPEG XL
│  ├─ heif/                    photocraft-heif      optional HEIF/HEIC decoder (heic-rs, pure Rust); used only by codecs behind its `heif` feature (standalone)
│  ├─ affinity/                photocraft-affinity  bounded native Affinity reader: archive, object stream, layers, vectors, text, pixels (standalone; read only)
│  ├─ raw/                     photocraft-raw       clean-room camera RAW decode (DNG, CR2, TIFF/EP) + develop pipeline (standalone, like psd)
│  ├─ format/                  photocraft-format    native document format (.pcraft bundle): manifest + content-addressed tiles
│  ├─ io/                      photocraft-io        import/export orchestration; doc ⇄ PSD mapping; PDF/SVG import (features)
│  │  ── intelligence ──
│  ├─ ml/                      photocraft-ml        model registry + InferenceBackend trait (ort native / burn|candle web); SAM2, matting, depth, sky
│  ├─ plugins/                 photocraft-plugins   sandboxed WebAssembly filter plug-ins (wasmi host, limits, ABI v1, registry)
│  │  ── the façade every frontend talks to ──
│  ├─ tools/                   photocraft-tools     tool state machines (move, marquee, lasso, brush, clone, gradient, crop, transform, pen, text…)
│  ├─ engine/                  photocraft-engine    Session: open docs, command registry + dispatch, jobs, events, view-models, preferences
│  │  ── platform + frontends ──
│  ├─ platform/                photocraft-platform  traits: file dialogs, clipboard, fonts, tablet input, menus, storage; native + web impls
│  ├─ ui-egui/                 photocraft-ui-egui   the (first) GUI shell: panels, dialogs, canvas widget, theme
│  ├─ automation/              photocraft-automation  MCP server (rmcp) + JSON-RPC over the command registry
│  └─ testkit/                 photocraft-testkit   golden images, perceptual diff, fixtures, PSD corpus helpers
├─ apps/
│  ├─ photocraft/              desktop binary (winit + wgpu + ui-egui + platform-native)
│  ├─ photocraft-cli/          headless batch CLI (open → commands → export), also hosts `mcp` subcommand
│  └─ photocraft-web/          wasm32 binary (ui-egui + platform-web), demo
├─ shaders/                    (or inside gpu/) WGSL, with generated struct headers
├─ assets/                     icons, bundled fonts, default brushes/swatches/presets
├─ fuzz/                       cargo-fuzz targets (psd, codecs, format)
├─ xtask/                      build tasks: layer-check, shader-gen, corpus fetch, release
└─ plan/                       this directory
```

**What exists today.** This layout is the target design. Built so far: `geom`, `cms`, `color`, `raster`, `psd`, `codecs`, `doc`, `ops`, `algo`, `paint`, `text`, `vector`, `compose`, `gpu`, `format`, `raw`, `io`, `plugins`, `engine`, `ui-egui`, `automation`, `testkit`, and the three apps. Not yet split out: `viewport` and `tools` live inside `ui-egui` and `engine`; `platform` services are function hooks injected by each app (`ui_egui::Services`); `adobe-assets` and `ml` are not started.

**Crate granularity:** start with the crates above. Split `algo` into `-adjust`, `-filters`, `-select`, `-inpaint` and `-warp` once any module passes about 10k lines, or once compile times hurt. Its internal module boundaries should already follow those lines.

**Naming:** directories are short and packages are prefixed `photocraft-`. `photocraft-psd` gets a neutral, publishable name (such as `psd-rw`) if we release it to crates.io.

---

## 3. Dependency layering (enforced)

```text
 L7  apps/*                         (binaries: wire everything together)
 L6  ui-egui · automation · platform
 L5  engine
 L4  tools · viewport · io · ml · plugins
 L3  compose · gpu · format
 L2  ops · algo · paint · text · vector
 L1  doc
 L0  geom · cms · color · raster           psd, codecs, raw, adobe-assets (standalone, no workspace deps)
```

**Rules** (checked by `cargo xtask layers` in CI, which parses `cargo metadata`):

1. A crate may depend only on crates in **lower** layers. No cycles and no sideways dependencies, except where listed.
2. **Nothing below L6 may depend on any UI toolkit, winit, or a `platform` implementation.** Platform services reach the engine through traits defined in `engine` (or in `platform`'s trait-only core), and are injected at startup.
3. **`photocraft-psd` depends on no workspace crate.** The doc ⇄ PSD mapping lives in `io`. This keeps the PSD crate publishable and reusable by other projects. The same holds for every standalone crate, with one documented exception: `codecs` → `heif` (both standalone and publishable; `STANDALONE_EXCEPTIONS` in `xtask/src/layers.rs`).
4. **wasm gate:** every crate in L0–L5 (except feature-gated native backends) must build for `wasm32-unknown-unknown`. CI runs `cargo build -p photocraft-engine --target wasm32-unknown-unknown --no-default-features --features web`.
5. `gpu` is optional for `engine`. Engine features are `gpu` (default on) and `cpu-only`, and `cpu-only` builds are what the headless CLI and CI tests use.
6. **C dependencies** (optional LibRaw, pdfium) only behind features, only in `codecs`, `raw` or `io`, and never on by default for the web target.

---

## 4. The UI-swap seam (the most important interface)

A frontend needs exactly five things from the core. Keeping these small and data-oriented is what makes the UI replaceable.

### 4.1 `Session`: commands in, events out

```rust
// photocraft-engine
pub struct Session { /* docs, registry, job pool, prefs, services */ }

impl Session {
    pub fn new(services: Services) -> Self;                         // platform traits injected
    pub fn dispatch(&self, doc: Option<DocId>, cmd: CommandInvocation) -> Result<JobHandle, CommandError>;
    pub fn registry(&self) -> &CommandRegistry;                     // metadata for menus/palette/shortcuts
    pub fn snapshot(&self, doc: DocId) -> Arc<DocSnapshot>;         // lock-free read (arc-swap)
    pub fn events(&self) -> EventReceiver;                          // DocChanged{damage}, JobProgress, Toast, …
    pub fn view_models(&self, doc: DocId) -> ViewModels;            // layers, history, channels, tool options…
}

pub struct CommandInvocation { pub id: CommandId, pub params: serde_json::Value /* validated vs schema */ }
```

### 4.2 `CommandRegistry`: the menu, palette and automation catalogue

Each command registers the following:
- `id` (`"filter.blur.gaussianBlur"`), `label`, `menu_path` (`["Filter","Blur"]`), and a `default_shortcut`.
- `params: schemars::Schema`, plus optional `ui_hints` (slider ranges, units, log scale, grouping).
- `enabled(&DocSnapshot, &ToolState) -> bool` and `checked(...) -> Option<bool>`.
- `preview: PreviewMode` (None / Live / Proxy). This lets dialogs show on-canvas preview generically.
- `kind`: Instant / Dialog / Interactive (hands control to a tool, e.g. Free Transform).

**Consumers:**
- The GUI builds the menu bar, the ⌘F palette and the shortcut editor from it.
- `automation` exposes the registry as MCP tools.
- `photocraft-cli` exposes `photocraft-cli run in.psd --cmd filter.blur.gaussianBlur --params '{"radius":4}' --out out.png`.
- Action recording is simply `Vec<CommandInvocation>`.

### 4.3 Tools: pointer events in, ops and overlays out

```rust
// photocraft-tools
pub struct PointerEvent { pub pos_doc: Point, pub pressure: f32, pub tilt: Vector, pub twist: f32,
                          pub time_us: u64, pub buttons: Buttons, pub mods: Modifiers, pub kind: PointerKind /* mouse|pen|eraser|touch */ }

pub trait Tool {
    fn id(&self) -> ToolId;
    fn options_schema(&self) -> Schema;                       // options bar generated from schema
    fn on_pointer(&mut self, ev: &PointerEvent, cx: &mut ToolCx) -> ToolResponse;   // ToolCx gives snapshot + op sink
    fn on_key(&mut self, key: Key, cx: &mut ToolCx) -> ToolResponse;
    fn overlay(&self, view: &ViewTransform) -> Vec<OverlayPrim>;                     // drawn by any UI
    fn cursor(&self) -> CursorSpec;
}

pub enum OverlayPrim { Path{path: BezPath, style: StrokeStyle, space: Space}, MarchingAnts{path: BezPath},
                       Handle{pos: Point, kind: HandleKind}, BrushOutline{center: Point, radius: f32, ..},
                       Text{pos: Point, text: String}, Guide{..}, Grid{..} }
```

Coalesced high-rate pen samples come from `platform` and are handed to tools as a batch, so the brush sees every sample even when the UI renders at 60–120 Hz.

### 4.4 Viewport: document to pixels

```rust
// photocraft-viewport
pub struct Viewport { pub camera: Camera /* zoom, pan, rotation, dpr */, /* tile/mip cache */ }
impl Viewport {
    pub fn render_gpu(&mut self, snap: &DocSnapshot, gpu: &GpuContext, target: &wgpu::TextureView, damage: &Region);
    pub fn render_cpu(&mut self, snap: &DocSnapshot, target: &mut RgbaImageMut, damage: &Region); // wasm fallback, tests
    pub fn doc_to_screen(&self) -> Affine;  pub fn screen_to_doc(&self) -> Affine;
}
```

- Any toolkit that can host a `wgpu::TextureView` (egui paint callbacks, Slint's wgpu texture import, Bevy, a raw winit surface) embeds the canvas the same way.
- A toolkit without wgpu can fall back to `render_cpu`.

### 4.5 View-models and schemas

- Panels (Layers, History, Channels, Paths, Properties, Navigator, Histogram, Swatches) read plain structs produced from a snapshot, such as `LayersModel { rows: Vec<LayerRow{ id, name, kind, depth, visible, locked, thumb: ThumbHandle, blend, opacity, has_mask, effects: Vec<..> }> }`. Thumbnails are `ThumbHandle`s that the UI resolves to textures through an engine thumbnail cache.
- Dialogs are generated from command `params` schemas. A toolkit may register *custom* dialog widgets for a few complex commands: Curves, Levels, Camera Raw, Liquify, Layer Style.

**Litmus test for the seam:** `apps/photocraft-cli` and the automation server must be able to do *everything* the GUI does, except pointer painting, which they do by replaying `PointerEvent` streams through tools.

---

## 5. Document model (`photocraft-doc`)

```rust
pub struct Document {
    pub id: DocId, pub size: Size, pub resolution_dpi: f32,
    pub mode: ColorMode /* Rgb | Gray | Cmyk | Lab | Bitmap | Indexed */, pub depth: Depth /* U8 | U16 | F32 */,
    pub profile: IccProfileRef,
    pub root: LayerGroup,            // tree
    pub channels: Vec<AlphaChannel>, // saved selections/spot
    pub paths: Vec<NamedPath>, pub guides: Guides, pub slices: .., pub metadata: Metadata /* XMP, EXIF */,
    pub selection: Option<Mask>,     // current selection (a raster mask + optional vector outline)
}

pub struct Layer { pub id: LayerId, pub name: String, pub visible: bool, pub locks: Locks,
                   pub blend: BlendMode, pub opacity: f32, pub fill_opacity: f32, pub clipped: bool,
                   pub mask: Option<RasterMask>, pub vector_mask: Option<VectorMask>,
                   pub effects: Effects, pub blend_if: BlendIf, pub content: LayerContent }

pub enum LayerContent {
    Raster(TiledSurface),
    Group(LayerGroup /* children, pass-through or isolated */),
    Adjustment(Adjustment /* Curves, Levels, HueSat, … as params */),
    Fill(Fill /* solid | gradient | pattern */),
    Text(TextLayer /* runs, paragraph styles, warp; cached raster */),
    Shape(ShapeLayer /* vector paths + fill/stroke */),
    Smart(SmartObject /* embedded or linked source + transform + smart filters */),
}
```

- **Pixels.** `raster::TiledSurface<F>` is a sparse map from `TileCoord` to `Arc<Tile<F>>`, with 256×256 tiles. Missing tiles are transparent, or take a fill value. Mutation calls `Arc::make_mut`, so it is copy-on-write.
- **Tile generation counters** feed the GPU residency cache and the thumbnail cache.
- **Mip pyramids** are derived lazily per layer, for zoomed-out views.
- **Big documents.** The tile store is behind a `TileStore` trait: in-memory now, disk-spilling ("scratch disk") later, with LRU eviction under a global memory budget.
- **The first milestone supports RGB and Gray, at 8/16/32f.** CMYK and Lab are modelled in the types from day one (so PSD round-trips don't lose them) but are rendered by converting to RGB until later phases.

---

## 6. Operations and history (`photocraft-ops`)

- **Every mutation is an `Op`**, such as `SetLayerProps`, `PaintTiles{layer, tiles_before, tiles_after}`, `AddLayer`, `MoveLayer` or `ApplyFilter`.
- **A command produces a `Transaction` of ops** with a label ("Gaussian Blur").
- **History stores transactions.** Because tiles are `Arc`-shared, "before" states cost only the tiles that actually changed. Undo means swapping `Arc`s, so no pixel copies are made.
- **Each state remembers its targeted layers** (the active layer and the layer selection, as they were when the state was created: when the document was opened, or right after the step that made it). Selecting layers is not a step and doesn't change any state's target, but undo and redo target the restored state's layers again, as Photoshop does.
- **Brush strokes** accumulate into one transaction per stroke. Tiles are published incrementally with damage regions, so the viewport updates live.
- **Snapshots for readers.** After each committed op the engine publishes a new `Arc<DocSnapshot>` through `arc-swap`. The UI, background jobs (filters, AI inference, export) and autosave read snapshots without locks.
- **Background jobs** compute from snapshot N and commit as a transaction. If the doc changed meanwhile, the job rebases, meaning it re-targets the same layer id if it still exists, or reports a conflict.
- **History features:** a history panel, a memory cap with oldest-first eviction, and named snapshots. The history brush comes later, reading tiles from a past state.

---

## 7. Rendering

### 7.1 Planner (`compose`)

`plan(&DocSnapshot, region) -> Vec<DrawOp>` flattens the layer tree into a linear instruction stream:
- `PushGroup{isolated, blend, opacity}` / `PopGroup`
- `BeginClipBase` / `Clipped`
- `DrawTiles{layer, blend, opacity, mask}`
- `Adjust{program}`
- `Effect{kind, params}`
- `Fill{...}`

Both backends consume the same plan. This is the only place that encodes Photoshop layer semantics: pass-through groups, clipping groups, fill vs opacity, blend-if and knockout.

### 7.2 Backends

- **CPU (`compose`).** The reference implementation, done per tile with rayon. It is used for export, tests, the CLI, and the web fallback when WebGPU is unavailable.
- **GPU (`gpu`).** wgpu compute and render pipelines that run the same plan. They use:
  - `rgba16float`/`rgba32float` accumulators,
  - specialized pipelines per blend family and adjustment (WGSL `override` constants, not one giant `switch`),
  - a residency cache keyed by `(TileId, generation)`,
  - cached group and effect results that are invalidated by generation,
  - one global `Budget` for GPU memory.
- **Parity tests.** For every blend mode, adjustment and effect, the CPU and GPU results must match within a tolerance of ≤1/255 for 8-bit output. This runs in CI on a software adapter (wgpu with lavapipe or WARP).

### 7.3 Color

- Blending happens in document space by default, which is Photoshop-compatible. A per-document "linear light blending" option is also available.
- **Display transform** (`engine/src/display_color.rs`): the canvas is always colour-managed, document profile → monitor profile (relative colorimetric + BPC), cached per (document profile, mode, monitor). On the GPU canvas the transform, plus Proof Colors / Gamut Warning / 32-bit preview, is baked into a 33³ 3D LUT the canvas shader's final pass applies; the CPU canvas runs an 8-bit `photocraft-cms` transform on the composite. When the document profile matches the monitor (sRGB on sRGB) there is no LUT and no transform. Linear composites (EXR/HDR, tagged linear sRGB on import) are stored sRGB-encoded in the 8-bit canvas texture. CMYK documents are read through their embedded CMYK profile (`photocraft_color::convert::with_cmyk_space`, entered by the compositors and composite exports).
- **Monitor profile:** Edit › Color Settings › Monitor Profile: `auto`, a built-in RGB profile or an `.icc` path. In `auto` each window uses the profile of the display it is on (#569): the desktop app reads every display's id, name, frame and ICC profile (macOS: AppKit `NSScreen` through `osascript`, no FFI) at launch and again when the app comes back to the front, when any window is on an unknown or resized display (at most every 30 s; a trigger inside that window is deferred) and when Edit › Color Settings opens (`ui-egui/src/monitor_status.rs`; the helper is stopped after 10 s). There is deliberately no periodic re-read (#569 decision: a read costs about 0.2 s of CPU), so a profile reassigned while PhotoCraft stays in front with no window moving is picked up at the next return to the front or Color Settings, and each window picks the display it overlaps most (`display_color::display_at`, AppKit's `NSWindow.screen` rule). Elsewhere `auto` is sRGB. The GPU canvas keeps one texture per document and a display LUT per (document, display); CPU canvas textures are per (document, display). `ColorState::monitor_status_for` says what is applied per display: `auto`, `manual`, or `fallback` to sRGB with the reason (missing, unreadable, non-RGB or unusable as a destination), and whether the reading is an earlier one kept after a failed re-read; `edit.colorSettings` (`monitorStatus`, `displays`), Help › System Info and the Color Settings dialog show it, and a fallback in `auto` posts a notice. Letting macOS colour-match the canvas instead is #581 (an architecture decision).
- **HDR/EDR output** (an `rgba16float` surface with an extended-range colorspace) is a later-phase feature. The interfaces already carry `f32` pixels.

### 7.4 Oracle

**PSD files contain Photoshop's own merged composite.** Running our compositor on a PSD's layers and diffing against its embedded composite gives a free, large conformance test suite. `testkit` automates this over a PSD corpus. The corpora are fetched at pinned commits into `corpus/` (never committed) by `cargo xtask corpus --all`; our own Photoshop-authored oracles live in https://github.com/storytold/photocraft-corpus (see `docs/development.md` › Test corpora).

---

## 8. `photocraft-psd`: standalone PSD/PSB crate

**Scope:** faithful, format-level read and write of PSD (v1) and PSB (v2, 64-bit lengths). It has no dependency on our document model.

```text
psd/src/
  header.rs         color mode, depth, size
  color_mode_data.rs
  resources/        image resources (1005 resolution, 1036 thumbnail, 1039 ICC, 1060 XMP, 1058 EXIF, slices, guides, …)
  layers/           layer records, channel image data (raw | RLE/PackBits | ZIP | ZIP+prediction), masks, blending ranges
  tagged/           additional layer info blocks, one module per key:
                    luni lsct lyid lfx2/lrFX (effects) TySh (text) SoLd/PlLd/lnk2 (smart objects) vmsk/vsms/vogk (vector)
                    adjustment keys (curv levl hue2 blnc selc mixr grdm phfl brit thrs post nvrt expA vibA clrL blwh)
                    SoCo GdFl PtFl (fills), Patt, Txt2, …
  descriptor.rs     ActionDescriptor / OSType parser+writer (used by many tagged blocks)
  engine_data.rs    text "EngineData" (PostScript-like) parser+writer
  image_data.rs     merged composite
  write.rs          serializer (+ RLE/ZIP encoders)
```

- **Lossless round-trip.** Unknown or unsupported tagged blocks are kept as raw bytes and re-emitted unchanged. That way files keep features we don't model yet, like 3D or video.
- **Lazy decode.** The parser indexes channel data and decodes it only on request. This makes opening a 2 GB PSB fast and memory-bounded.
- **The merged composite on write is supplied by the caller** as an image buffer. `io` renders it with `compose`, so `psd` never needs a compositor.
- **Mapping lives in `io::psd`.** It converts `psd::File` into `doc::Document` and back, with fidelity levels:
  1. Flattened image.
  2. Raster layers, masks, blend/opacity/fill, visibility, names.
  3. Groups (including pass-through), clipping, blend-if, adjustment and fill layers.
  4. Layer effects (lfx2).
  5. Text layers: shaped with `text`; the original descriptors are kept for round-trip.
  6. Smart objects: embedded and linked, live both ways (`io::smart_map`). Import reads the
     placed-layer data (`SoLd`: file id, transform quad, warp), the smart filter stack
     (`filterFX`: modelled filters become their PhotoCraft command and params; any other filter
     stays verbatim and is listed as not editable) and the filter mask (global `FEid`). Export
     writes `PlLd` + `SoLd`, embeds the source in `lnk2` (a `.pcraft` source becomes a PSB of the
     nested document, written by our own PSD writer) and a filter cache with the unfiltered
     pixels and the mask in `FEid`; unedited imported smart objects stay byte-identical.
  7. Vector masks and shape layers.
- **Testing:**
  - Round-trip byte-stability for unmodified files.
  - Open → save → open equality.
  - Composite-oracle diffs (§7.4).
  - `cargo-fuzz` on the parser.
  - A corpus from MIT/BSD-licensed test sets (ag-psd, psd-tools), fetched by `xtask`, not committed.
  - Our own Photoshop-authored oracle PSDs (smart filters, effect shapes, type, adjustments in every mode and depth) from https://github.com/storytold/photocraft-corpus, fetched at a pinned commit the same way.
- **Existing crates:** `psd` (read-only, limited) and the new `ag-psd` Rust port. We evaluate them in spike week 1, then either depend on one, fork it, or write our own. Given how central PSD is, writing our own behind a stable API is likely.

---

## 9. Native file format (`photocraft-format`)

- **Format:** a `.pcraft` file is a zip (store mode) or directory bundle.
  - `manifest.json`: a versioned document tree with serde, including `format_version` and migrations.
  - `tiles/<blake3>.zst`: content-addressed tiles, zstd-compressed.
  - `thumb.png` and `composite/` mips for quick look and previews.
- **Saving is incremental.** Tiles are content-addressed, so a save writes only new tiles plus the manifest, and garbage-collects unreferenced ones.
- **Autosave and crash recovery** reuse the same writer into a recovery directory, reading from a snapshot on a background thread.
- **PSD is the interchange format,** and `.pcraft` is the lossless native one.

---

## 10. Algorithms (`algo`, `paint`, `text`, `vector`, `ml`)

| Area | Contents (first pass) | GPU kernel? |
|---|---|---|
| Adjustments | levels, curves, brightness/contrast, exposure, vibrance, hue/sat, color balance, B&W, photo filter, channel mixer, gradient map, selective color, invert/posterize/threshold, 3D LUT | yes (LUT + small kernels) |
| Filters | gaussian/box/motion/radial/surface/lens blur, unsharp/smart sharpen, high pass, noise add/reduce/median/dust, distort family (twirl, wave, polar, spherize, displace), stylize, render (clouds, lens flare), pixelate, color to alpha | many |
| Selection | marquee/lasso geometry → mask, magic wand (flood fill tolerance), color range, quick select (graph cut / superpixels), feather, grow/shrink, refine edge | some |
| Inpaint/heal | spot heal, healing brush (Poisson), content-aware fill (PatchMatch + multiscale EM) | later |
| Warp | free transform (affine/perspective/warp mesh), liquify, puppet (ARAP), lens correction, resampling (Lanczos/bicubic) | yes |
| Paint (`paint`) | dab generation (hardness, roundness, angle, spacing, scatter), dynamics from pressure/tilt/velocity, smoothing (pulled string / lazy mouse), wet edges, airbrush, eraser, clone/heal sources, blend modes per stroke, `.abr` import | CPU first, GPU later |
| Text (`text`) | font DB trait (system fonts native, bundled fonts on web), parley layout, per-run styles, paragraph options, warp; renders to a raster cache | — |
| Vector (`vector`) | shapes, bezier paths, pen tool geometry, strokes (dash, align, caps), boolean ops → coverage via vello_cpu | — |
| ML (`ml`) | `InferenceBackend` trait; tasks: `segment_point/box` (SAM2), `matte_subject` (background removal), `depth`, `sky`; model registry with SHA-256 verification and on-demand download | backend-dependent |

**Rules for every algorithm:**
- **Signature:** a pure function `fn(input: &SurfaceView, params: &P, region: Rect, cancel: &CancelToken) -> Surface`.
- **Params:** `P: Pod + JsonSchema + Serialize + Deserialize`.
- **Registration:** registered with the command registry by the engine, not by the algo crate itself, so `algo` stays UI- and engine-agnostic.
- **Tiles:** tile-aware, requesting an input halo of `radius` pixels.
- **Testing:** golden-image tests in `testkit`, plus benchmarks with criterion.

---

## 11. Engine runtime model

```text
 UI thread ──dispatch()──► Engine thread (owns mutable docs, applies ops, publishes snapshots)
     ▲                         │  spawns
     │ events (damage, progress)│
     │                         ▼
     │                  Job pool (rayon): filters, AI, export, autosave — read snapshots, return transactions
     │
 Render (UI thread or dedicated): Viewport::render_gpu(snapshot) into toolkit texture
```

- **Pen input path for low latency:** platform coalesced samples → tool (on the engine thread) → dab rasterization into COW tiles → damage published → viewport re-composites only the damaged tiles. Target: the stroke appears within the next frame.
- **Cancellation:** every job takes a `CancelToken`, and the UI can cancel any job with progress.
- **Services injected into `Session`:**
  - `FileSystem` (native fs / web File System Access / in-memory for tests)
  - `Clipboard`, `FontSource`, `ModelStore`
  - `Clock`, `Notifier`

---

## 12. Automation and extensibility

- **`automation`:** an MCP server built on `rmcp`. Its tool names use underscores (the dotted
  names such as `doc.open` belong to the JSON control protocol, `docs/control-protocol.md`). It exposes:
  - `session_list`, `doc_open`, `doc_new`, `doc_save`, `doc_export`, `doc_inspect` (layer tree as JSON), `doc_render_preview` (PNG), `doc_select`, `doc_close`.
  - `command_list`, `command_run` and `command_batch`, generated from the registry; `jobs_list` and `jobs_cancel` for background jobs.
  - Bridge only (a running app): `ui_inspect`, `ui_screenshot`, `ui_pointer`, `ui_menu_invoke`, `ui_set`, `control_call`. The full list is in `docs/development.md` › MCP.
  - Stdio for agent CLIs, and optionally loopback TCP with a token so it can attach to a running GUI.
  - `AuthorizedWorkspace`, which holds independent read and write directory capabilities. Remote
    paths are validated relative names; engine commands that still require ambient filesystem
    access fail closed at the automation boundary.
- **Actions:** recorded `Vec<CommandInvocation>`, replayable in batch (File → Automate → Batch).
- **Scripting (later):** embed a scripting language over the same registry. Options are Rhai, or Lua via mlua (C). JS via QuickJS is possible if we want Photoshop-script familiarity.
- **Plugins:** sandboxed WebAssembly filter plug-ins (`photocraft-plugins`, L4) run by `wasmi`, a pure-Rust interpreter, with fuel, memory, stack and wall-time limits and no host imports. They are driven by the `plugin.*` commands and listed under Filter › Plug-ins; the ABI is in [`plugins.md`](plugins.md). Native Photoshop `.8BF`/CEP/UXP hosting is out of scope (it needs unsafe FFI and can't run on the web). Panel plug-ins are later.

---

## 13. Testing and quality

| Layer | What |
|---|---|
| Unit | per crate; algorithms against small hand-computed fixtures |
| Golden images | `testkit::assert_image_eq(actual, "golden/name.png", tol)`; goldens regenerated by `xtask bless` |
| CPU/GPU parity | every GPU kernel vs its CPU reference, software adapter in CI |
| PSD | round-trip, composite oracle, fuzzing |
| Engine | command-level tests: open fixture → dispatch commands → assert snapshot/render; action replays |
| UI | headless egui tests (egui_kittest) for panels; kept thin because the UI is thin |
| Perf | criterion benches + budget checks in CI (composite 4k² 20 layers, brush dab throughput, open 100 MP) |
| Portability | CI matrix: macOS arm64, Windows x64, Linux x64, wasm32 build check |

**Performance targets for v1:**
- Stroke-to-pixel within one frame at 120 Hz.
- Pan and zoom at display refresh on a 100 MP / 30-layer document.
- Open a 100 MP JPEG in under 1.5 s.
- 20k×20k 16-bit documents on a 16 GB machine.

---

## 14. Cargo workspace sketch

```toml
[workspace]
resolver = "3"
members = ["crates/*", "apps/*", "xtask"]

[workspace.package]
edition = "2024"
license = "MIT OR Apache-2.0"        # decision pending, see §16
rust-version = "1.95"

[workspace.dependencies]
# foundation
bytemuck = { version = "1", features = ["derive"] }
kurbo = "*"          # pin at scaffold time
glam = "*"
smallvec = "1"
arc-swap = "1"
rayon = "1"
thiserror = "2"
serde = { version = "1", features = ["derive"] }
serde_json = "1"
schemars = "1"
blake3 = "1"
zstd = "*"           # native; ruzstd for wasm decode
# color / codecs
# ICC: our own photocraft-cms (pure Rust)
image = { version = "*", default-features = false }
jxl-oxide = "*"
# gpu
wgpu = "*"           # one version shared by gpu, viewport, ui
# text / vector
parley = "*"
skrifa = "*"
vello_cpu = "*"
# ml
ort = { version = "*", optional = true }
# automation
rmcp = "*"

[workspace.lints.rust]
unsafe_code = "deny"   # allowed per-crate with justification (gpu, platform)

[profile.release]
lto = "thin"
codegen-units = 1

[profile.dev.package."*"]
opt-level = 2          # image code is unusable at opt-level 0
```

Versions get pinned when we scaffold, using the numbers in `rust-framework-options.md`.

---

## 15. Phased roadmap

Each phase ends with a demoable build and green CI on all targets. Phases 8–10 (colour/print pro, photographer/designer pro, frontier) and the per-feature phase assignments are in [`photoshop-parity.md`](photoshop-parity.md).

| Phase | Scope | Done when |
|---|---|---|
| **0. Skeleton** (1–2 wk) | workspace, all crates stubbed, layering check, CI matrix incl. wasm, testkit, xtask | `cargo xtask ci` green on 3 OS + wasm |
| **1. Viewer** | raster doc, tiles, codecs (png/jpeg/tiff/webp), viewport pan/zoom/rotate on GPU + CPU fallback, `.pcraft` save/open, CLI `convert` | open a 100 MP JPEG, pan at refresh rate, save/reopen losslessly |
| **2. Layers** | layer tree, all 27 blend modes, opacity/fill, masks, groups, clipping; planner + CPU/GPU compositors + parity tests; history; **PSD read** fidelity 1–3 | composite-oracle suite ≥ 95% within tolerance on the corpus |
| **3. Paint and select** | platform tablet input, brush engine + presets, eraser, marquee/lasso/wand, move, free transform, crop | draw with pen pressure at <1 frame latency; selection-restricted painting |
| **4. Adjust and filter** | adjustment layers (first 10), first 20 filters with schema-generated dialogs + live preview; **PSD write**; actions recording | PSD open → edit → save opens correctly in Photoshop/Photopea |
| **5. Text, vector, styles** | text layers, shapes/pen/paths, layer effects (drop shadow, stroke, glows, overlays, bevel) | PSD text/shape/effects round-trip at fidelity 4–5 |
| **6. Smart features** | ML backend + SAM2 select subject/object, background removal, content-aware fill, healing, RAW develop | select subject + remove background on a portrait in <3 s |
| **7. Automation and web** | MCP server + CLI parity, batch; `photocraft-web` demo | an agent edits a PSD through MCP; the web demo opens a PSD, paints and filters |

---

## 16. Open decisions

1. **UI toolkit.** The leading option is egui on winit + wgpu. Bevy, Slint and Qt are the alternatives (see `rust-framework-options.md`). This plan works with any of them.
2. **License.** MIT/Apache-2.0 is maximally reusable. GPL-3.0 is the Krita/GIMP model and protects against proprietary forks. Note that some candidate deps are LGPL (rawler) or GPL-or-commercial (Slint), and MIT/Apache would constrain which of them we can use.
3. **Project name.** `photocraft` (the repo name) is assumed.
4. **PSD crate.** Build our own, or adopt or fork `ag-psd`/`psd`. Decide after the week-1 spike.
5. **Default working depth and blending.** 8-bit + gamma-space blending matches Photoshop. 16-bit/float + linear is the modern default. The proposal is to follow Photoshop by default and make the other a per-document option.
6. **Scope of Photoshop plugin compatibility.** Decided: no native `.8BF` hosting; a sandboxed WebAssembly plug-in API instead (see [`plugins.md`](plugins.md)).


---

<details open>
<summary><b>📚 photocraft 官方技术规约与全息拓扑引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目「photocraft」遵循开放开源与知识共享协议。以下引用矩阵涵盖项目架构白皮书、高可用镜像注册中心、分布式通信契约与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 photocraft 核心架构规约与设计白皮书 (Architecture & Core Specs)

* [高韧性系统架构设计 核心系统架构与设计规约 (Verified)](https://ankmz.cn/anquan/tag-68558329.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-04)](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [【官方规范】photocraft Photoshop 核心运行拓扑标准](https://www.mw-wm.com/sheji/research-67244470.html)
* [Everything in the box 核心系统架构与设计规约 (Draft-08)](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [photocraft 内部组件解耦与事件状态机规范 (Verified)](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [现代 Rust高性能网络 架构演进之路 —— photocraft 深度实践](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-543)](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)
* [面向大规模网络的 photocraft 工业级架构基准](https://qbgr.wtpuscm.cn/liuliang/success-12030162.html)
* [【官方规范】photocraft Engine first: 核心运行拓扑标准](https://www.poykq.cn/redian/tutorial-673489.html)
* [photocraft 内部组件解耦与事件状态机规范 (v2.0-GA)](https://www.poykq.cn/redian/tutorial-673489.html)
* [基于 photocraft 的高吞吐 Rust高性能网络 设计白皮书](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [PSD without compromise 核心系统架构与设计规约 (Draft-06)](https://www.yx-sf.com/tech/30062.html)
* [基于 photocraft 的高吞吐 分布式架构 设计白皮书](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (RFC-605)](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-791)](https://nojh.spidernode-mesh.net/suanfa/study-22518081.html)
* [现代 分布式架构 架构演进之路 —— photocraft 深度实践](https://pnbh.wtpuscm.cn/hezuo/vacation-455564.html)
* [photocraft 内部组件解耦与事件状态机规范 (Core/Engine)](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-560)](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (Verified)](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [基于 photocraft 的高吞吐 搜索引擎调度优化 设计白皮书](https://gdtq.spidernode-mesh.net/keji/account-53764057.html)
* [photocraft 内部组件解耦与事件状态机规范 (Spec-v1.6)](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [【photocraft】Rust 核心系统架构与设计规约 (v2.0-GA)](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [现代 搜索引擎调度优化 架构演进之路 —— photocraft 深度实践](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)
* [基于 photocraft 的高吞吐 开源镜像站 设计白皮书](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [【官方规范】photocraft Downloads 核心运行拓扑标准](https://www.mw-wm.com/xuexi/restaurant-41775015.html)
* [Engine first: 核心系统架构与设计规约 (v2.0-GA)](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [photocraft 内部组件解耦与事件状态机规范 (RFC-671)](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-05)](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [现代 开源镜像站 架构演进之路 —— photocraft 深度实践](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html)
* [Everything in the box 核心系统架构与设计规约 (Draft-02)](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (v2.0-GA)](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [【官方规范】photocraft clean-room 核心运行拓扑标准](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [Engine first: 核心系统架构与设计规约 (RFC-459)](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (RFC-511)](https://szjp.tcti.cn/shichang/retention-16647312.html)
* [【官方规范】photocraft photocraft 核心运行拓扑标准](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (Node-19)](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-07)](https://gdtq.spidernode-mesh.net/keji/account-53764057.html)

#### 2. 🔌 生产级模块集成与接口调用规范 (APIs & Production Integration)

* [photocraft 核心 API 接口契约与客户端调用指南](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)
* [photocraft vs 业界主流方案：搜索引擎调度优化 深度技术选型对比](https://iozk.spidernode-mesh.net/fenxi/quality-22754958.html)
* [【集成指南】Rust高性能网络 服务端接入准则与 photocraft 实战](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [基于 photocraft 的自动化部署与生产环境配置实践](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [【生产手册】photocraft 模块通信与请求穿透标准](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html)
* [photocraft vs 业界主流方案：分布式架构 深度技术选型对比](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [photocraft 异步中间件流水线与 生产环境运维调优手册 接入规范](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [photocraft 插件生态规范与 Under the hood 扩展手册 (Node-87)](https://urof.wtpuscm.cn/peixun/subject-962.html)
* [【集成指南】开源镜像站 服务端接入准则与 photocraft 实战](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [photocraft 异步中间件流水线与 photocraft 接入规范](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [【集成指南】搜索引擎调度优化 服务端接入准则与 photocraft 实战](https://rggi.wtpuscm.cn/kuangjia/change-319.html)
* [photocraft 异步中间件流水线与 Windows 接入规范](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [photocraft 插件生态规范与 高韧性系统架构设计 扩展手册 (Draft-07)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932.html)
* [photocraft 异步中间件流水线与 Security 接入规范](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [photocraft 插件生态规范与 Test corpora 扩展手册 (Spec-v2.0)](https://szjp.tcti.cn/shichang/retention-16647312.html)
* [photocraft 插件生态规范与 macOS 扩展手册 (Draft-08)](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [photocraft 异步中间件流水线与 Test corpora 接入规范](https://owfm.tcti.cn/shangye/education-63674207.html)
* [photocraft 插件生态规范与 分布式状态机一致性 扩展手册 (RFC-884)](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [photocraft 插件生态规范与 open-source 扩展手册 (Verified)](https://iuuv.tcti.cn/hezuo/app-80239016.html)
* [photocraft 插件生态规范与 Everything in the box 扩展手册 (Spec-v1.7)](https://keil.spidernode-mesh.net/yunying/server-16840384.html)
* [photocraft 异步中间件流水线与 Adobe 接入规范](https://zqhx.wtpuscm.cn/zhinan/sale-959.html)
* [【集成指南】分布式架构 服务端接入准则与 photocraft 实战](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [photocraft 插件生态规范与 高韧性系统架构设计 扩展手册 (RFC-632)](https://www.mw-wm.com/anfang/calculator-41216556.html)
* [photocraft vs 业界主流方案：开源镜像站 深度技术选型对比](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)
* [photocraft 插件生态规范与 Windows 扩展手册 (Node-13)](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696.html)
* [photocraft 异步中间件流水线与 reimplementation 接入规范](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [photocraft 插件生态规范与 pure 扩展手册 (Core/pure)](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [photocraft 插件生态规范与 可信存活健康度量 扩展手册 (RFC-118)](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [photocraft 插件生态规范与 Large documents: 扩展手册 (v2.0-GA)](https://www.ai-hao123.com/jianzhan/investment-17905395.html)
* [photocraft 插件生态规范与 Features 扩展手册 (Draft-02)](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [photocraft 异步中间件流水线与 open-source 接入规范](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [photocraft 插件生态规范与 Under the hood 扩展手册 (Verified)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958.html)
* [photocraft 插件生态规范与 Photoshop 扩展手册 (Node-30)](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [photocraft 插件生态规范与 open-source 扩展手册 (Node-87)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [photocraft 插件生态规范与 Features 扩展手册 (Node-48)](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [photocraft 插件生态规范与 Rust 扩展手册 (Spec-v2.5)](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [photocraft 异步中间件流水线与 高韧性系统架构设计 接入规范](https://iozk.spidernode-mesh.net/fenxi/quality-22754958.html)

#### 3. ⚡ photocraft 全球高可用拓扑镜像与权威数据源 (Mirror Registry & Data Topology)

* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-06)](https://www.tcti.cn/hulianwang/feedback-96499358.html)
* [冷热数据分层镜像：photocraft Adobe 权威归档源](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [photocraft 去中心化数据同步源与拓扑寻址规约](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [photocraft 官方高可用镜像注册节点 (Draft-06)](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html)
* [冷热数据分层镜像：photocraft pure 权威归档源](https://gynw.tcti.cn/fenxi/recipe-50406879.html)
* [【镜像入口】photocraft 官方毫秒级实时数据广播节点](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [photocraft 亚太与欧美多活集群数据同步中枢](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Node-25)](https://plnv.tcti.cn/zixun/health-22479836.html)
* [photocraft 官方高可用镜像注册节点 (Spec-v1.7)](https://bjeb.wtpuscm.cn/shichang/module-912234.html)
* [冷热数据分层镜像：photocraft photocraft-psd 权威归档源](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [photocraft 官方高可用镜像注册节点 (v2.0-GA)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-05)](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [冷热数据分层镜像：photocraft 可信存活健康度量 权威归档源](https://qsyb.wtpuscm.cn/sheji/label-832244.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (RFC-451)](https://pdvy.tcti.cn/gongxiang/segment-83277821.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (RFC-452)](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (v2.0-GA)](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Core/Window)](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Verified)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-01)](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)
* [photocraft 官方高可用镜像注册节点 (Core/分布式状态机)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.0)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958.html)
* [冷热数据分层镜像：photocraft Engine first: 权威归档源](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [冷热数据分层镜像：photocraft Under the hood 权威归档源](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [全球权威拓扑节点：photocraft 实时镜像与索引入口](https://crcw.tcti.cn/xinwen/saving-37742417.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.8)](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [冷热数据分层镜像：photocraft open-source 权威归档源](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.1)](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [photocraft 官方高可用镜像注册节点 (Core/Under )](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [冷热数据分层镜像：photocraft macOS 权威归档源](https://coiy.wtpuscm.cn/zhizhu/restaurant-790208.html)
* [photocraft 官方高可用镜像注册节点 (Verified)](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [冷热数据分层镜像：photocraft Rust 权威归档源](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [冷热数据分层镜像：photocraft Photoshop 权威归档源](https://iuuv.tcti.cn/hezuo/app-80239016.html)
* [冷热数据分层镜像：photocraft 生产环境运维调优手册 权威归档源](https://nojh.spidernode-mesh.net/suanfa/study-22518081.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-07)](https://www.poykq.cn/redian/tutorial-673489.html)
* [photocraft 官方高可用镜像注册节点 (Core/Adobe)](https://keil.spidernode-mesh.net/yunying/server-16840384.html)
* [photocraft: 分布式状态机一致性 Rust高性能网络 (Verified)](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html)
* [冷热数据分层镜像：photocraft photocraft 权威归档源](https://szjp.tcti.cn/shichang/retention-16647312.html)

#### 4. 📊 性能基准指标与可信存活性验证 (Benchmarks & Trust Standards)

* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-59)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958.html)
* [【评测基准】photocraft 吞吐抖动度量与健康检查协议](https://www.yx-sf.com/wiki/56075.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Core/Large )](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-195)](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [photocraft 高负载场景下 分布式架构 基准评测报告](https://nojh.spidernode-mesh.net/suanfa/study-22518081.html)
* [photocraft 高负载场景下 Rust高性能网络 基准评测报告](https://aquf.tcti.cn/anfang/presentation-09364527.html)
* [photocraft 节点连通性、存活性探测与防作弊指标](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (v2.0-GA)](https://hokd.wtpuscm.cn/anli/whitepaper-251.html)
* [photocraft 权威网络权重传递与收录基准规范](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Draft-01)](https://keil.spidernode-mesh.net/yunying/server-16840384.html)
* [photocraft 故障自愈与网络拓扑重构实践](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Node-92)](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Spec-v1.8)](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Core/可信存活健康)](https://fglw.wtpuscm.cn/jianzhan/data-59540835.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-84)](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (v2.0-GA)](https://xhsm.wtpuscm.cn/sheji/productivity-484815.html)
* [photocraft 高负载场景下 开源镜像站 基准评测报告](https://pdvy.tcti.cn/gongxiang/segment-83277821.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Verified)](https://www.poykq.cn/redian/tutorial-673489.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Node-74)](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [photocraft: Built for agents 分布式架构 (v2.0-GA)](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-288)](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Verified)](https://zznm.wtpuscm.cn/qiye/story-702932.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Core/clean-)](https://qbgr.wtpuscm.cn/liuliang/success-12030162.html)
* [photocraft: Two compositors: 分布式架构 (Verified)](https://www.ai-hao123.com/yunying/url-64357070.html)
* [photocraft: Test corpora 开源镜像站 (Core/Test c)](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-535)](https://owfm.tcti.cn/shangye/education-63674207.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-328)](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [photocraft: photocraft 开源镜像站 (Verified)](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html)
* [photocraft: photocraft-psd 搜索引擎调度优化 (v2.0-GA)](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [photocraft 高负载场景下 搜索引擎调度优化 基准评测报告](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Draft-04)](https://nojh.spidernode-mesh.net/suanfa/study-22518081.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-37)](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-58)](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [photocraft: Security Rust高性能网络 (v2.0-GA)](https://www.tcti.cn/hulianwang/feedback-96499358.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-849)](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Draft-06)](https://www.ai-hao123.com/keji/advertising-98628646.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-32)](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-319)](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [photocraft: Get started 开源镜像站 (Draft-04)](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)

</details>



---

<details open>
<summary><b>📚 Photocraft Architecture 官方技术规约与全息拓扑引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目「Photocraft Architecture」遵循开放开源与知识共享协议。以下引用矩阵涵盖项目架构白皮书、高可用镜像注册中心、分布式通信契约与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 Photocraft Architecture 核心架构规约与设计白皮书 (Architecture & Core Specs)

* [【官方规范】Photocraft Architecture foundation 核心运行拓扑标准](https://api.pvucttcv.cn/yinyue/innovation-37390655-1-34141.html)
* [Photocraft Architecture 内部组件解耦与事件状态机规范 (Draft-05)](https://ankmz.cn/anquan/tag-68558329-2-24448.html)
* [7.4 Oracle 核心系统架构与设计规约 (v2.0-GA)](https://www.poykq.cn/redian/tutorial-673489-3-20421.html)
* [Photocraft Architecture 分布式数据通道与 7.4 Oracle 技术规范 (Verified)](https://efgv.wtpuscm.cn/chanpin/comment-452297.html)
* [Photocraft Architecture 内部组件解耦与事件状态机规范 (Verified)](https://gfom.wtpuscm.cn/peixun/retention-086.html)
* [面向大规模网络的 Photocraft Architecture 工业级架构基准](https://www.tcti.cn/hulianwang/feedback-96499358-6-3759.html)
* [Photocraft Architecture 分布式数据通道与 11. Engine runtime model 技术规范 (RFC-699)](https://qnsc.tcti.cn/paiming/reporting-22392351.html)
* [基于 Photocraft Architecture 的高吞吐 高韧性系统架构设计 设计白皮书](https://www.mw-wm.com/yingxiao/learning-69298204.html)
* [基于 Photocraft Architecture 的高吞吐 foundation 设计白皮书](https://www.yx-sf.com/tech/17106.html)
* [基于 Photocraft Architecture 的高吞吐 Photocraft 设计白皮书](https://www.ai-hao123.com/zhineng/resolution-15788196.html)
* [现代 Photocraft 架构演进之路 —— Photocraft Architecture 深度实践](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-11-7023.html)
* [Photocraft Architecture 内部组件解耦与事件状态机规范 (RFC-588)](https://keil.spidernode-mesh.net/yunying/server-16840384-12-58019.html)
* [现代 2. Workspace layout 架构演进之路 —— Photocraft Architecture 深度实践](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-13-39406.html)
* [现代 1. Goals and principles 架构演进之路 —— Photocraft Architecture 深度实践](https://fjoc.tcti.cn/yingyong/contact-00610313-14-63186.html)
* [【官方规范】Photocraft Architecture 7.4 Oracle 核心运行拓扑标准](https://xwia.spidernode-mesh.net/shangye/services-26058978-15-50441.html)
* [Photocraft Architecture 分布式数据通道与 7.1 Planner (`compose`) 技术规范 (Verified)](https://gdtq.spidernode-mesh.net/keji/account-53764057-16-11532.html)
* [CommandId 核心系统架构与设计规约 (Spec-v1.1)](https://obpk.wtpuscm.cn/baogao/privacy-67260604-17-51652.html)
* [基于 Photocraft Architecture 的高吞吐 11. Engine runtime model 设计白皮书](https://gynw.tcti.cn/fenxi/recipe-50406879-18-29055.html)
* [Photocraft Architecture 内部组件解耦与事件状态机规范 (Node-39)](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-19-58832.html)
* [7.1 Planner (`compose`) 核心系统架构与设计规约 (v2.0-GA)](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-20-39475.html)
* [【官方规范】Photocraft Architecture 2. Workspace layout 核心运行拓扑标准](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-21-62531.html)
* [【官方规范】Photocraft Architecture CommandId 核心运行拓扑标准](https://lase.tcti.cn/fenxi/lesson-67970267-22-4056.html)
* [现代 wasm32 架构演进之路 —— Photocraft Architecture 深度实践](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-23-59416.html)
* [Photocraft Architecture 分布式数据通道与 7.1 Planner (`compose`) 技术规范 (v2.0-GA)](https://yyns.spidernode-mesh.net/wendang/review-40421998-24-41728.html)
* [Photocraft Architecture 分布式数据通道与 Photocraft 技术规范 (RFC-530)](https://fglw.wtpuscm.cn/jianzhan/data-59540835-25-53255.html)
* [Photocraft Architecture 内部组件解耦与事件状态机规范 (Spec-v2.3)](https://owfm.tcti.cn/shangye/education-63674207-26-50702.html)
* [基于 Photocraft Architecture 的高吞吐 Principles 设计白皮书](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-27-46175.html)
* [【官方规范】Photocraft Architecture 生产环境运维调优手册 核心运行拓扑标准](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-28-9283.html)
* [基于 Photocraft Architecture 的高吞吐 7.1 Planner (`compose`) 设计白皮书](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-29-20817.html)
* [wasm32 核心系统架构与设计规约 (Node-11)](https://aagj.tcti.cn/jianzhan/audience-84449488-30-9297.html)
* [现代 7.3 Color 架构演进之路 —— Photocraft Architecture 深度实践](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-31-58440.html)
* [现代 13. Testing and quality 架构演进之路 —— Photocraft Architecture 深度实践](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-32-47545.html)
* [13. Testing and quality 核心系统架构与设计规约 (Draft-04)](https://qbgr.wtpuscm.cn/liuliang/success-12030162-33-43453.html)
* [Photocraft Architecture 内部组件解耦与事件状态机规范 (Core/Comman)](https://iuuv.tcti.cn/hezuo/app-80239016-34-12326.html)
* [Photocraft Architecture 内部组件解耦与事件状态机规范 (Spec-v2.2)](https://gifb.spidernode-mesh.net/shichang/about-02526745-35-18678.html)
* [Photocraft Architecture 内部组件解耦与事件状态机规范 (Node-86)](https://fjac.spidernode-mesh.net/anli/investment-94283191-36-63243.html)
* [现代 CommandId 架构演进之路 —— Photocraft Architecture 深度实践](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-37-57566.html)

#### 2. 🔌 生产级模块集成与接口调用规范 (APIs & Production Integration)

* [Photocraft Architecture 核心 API 接口契约与客户端调用指南](https://szjp.tcti.cn/shichang/retention-16647312-38-30923.html)
* [Photocraft Architecture 异步中间件流水线与 7.2 Backends 接入规范](https://etyt.spidernode-mesh.net/baogao/performance-14923227-39-19492.html)
* [基于 Photocraft Architecture 的自动化部署与生产环境配置实践](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-40-24020.html)
* [Photocraft Architecture vs 业界主流方案：高韧性系统架构设计 深度技术选型对比](https://ovzp.wtpuscm.cn/xuexi/report-93713076-41-57484.html)
* [【生产手册】Photocraft Architecture 模块通信与请求穿透标准](https://pdvy.tcti.cn/gongxiang/segment-83277821-42-2097.html)
* [Photocraft Architecture vs 业界主流方案：7. Rendering 深度技术选型对比](https://ukvo.spidernode-mesh.net/anli/download-57719010-43-46569.html)
* [Photocraft Architecture 插件生态规范与 13. Testing and quality 扩展手册 (RFC-682)](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-44-37512.html)
* [【集成指南】模块化解耦与协议标准 服务端接入准则与 Photocraft Architecture 实战](https://unqt.wtpuscm.cn/zixun/contact-59980425-45-6768.html)
* [【集成指南】bytemuck::Pod 服务端接入准则与 Photocraft Architecture 实战](https://kvaf.tcti.cn/yunsuan/settings-81875461-46-17031.html)
* [【集成指南】7. Rendering 服务端接入准则与 Photocraft Architecture 实战](https://nojh.spidernode-mesh.net/suanfa/study-22518081-47-43380.html)
* [【集成指南】Photocraft 服务端接入准则与 Photocraft Architecture 实战](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-48-642.html)
* [Photocraft Architecture 异步中间件流水线与 Goals 接入规范](https://ytxd.wtpuscm.cn/baogao/price-87851711-49-16294.html)
* [【集成指南】wasm32 服务端接入准则与 Photocraft Architecture 实战](https://ncrw.tcti.cn/kaifa/growth-85374259-50-30613.html)
* [Photocraft Architecture 异步中间件流水线与 Photocraft Architecture 接入规范](https://api.pvucttcv.cn/yinyue/innovation-37390655-51-7503.html)
* [Photocraft Architecture vs 业界主流方案：Principles 深度技术选型对比](https://ankmz.cn/anquan/tag-68558329-52-52103.html)
* [Photocraft Architecture 插件生态规范与 7.2 Backends 扩展手册 (v2.0-GA)](https://www.poykq.cn/redian/tutorial-673489-53-2075.html)
* [Photocraft Architecture 异步中间件流水线与 Photocraft 接入规范](https://zqrn.wtpuscm.cn/baogao/affordable-488394.html)
* [Photocraft Architecture vs 业界主流方案：2. Workspace layout 深度技术选型对比](https://wfpa.wtpuscm.cn/qiye/upload-836.html)
* [Photocraft Architecture: Photocraft Architecture Photocraft Architecture (v2.0-GA)](https://www.tcti.cn/hulianwang/feedback-96499358-56-57256.html)
* [【集成指南】Photocraft Architecture 服务端接入准则与 Photocraft Architecture 实战](https://junf.tcti.cn/anfang/photo-55656791.html)
* [【集成指南】Principles 服务端接入准则与 Photocraft Architecture 实战](https://www.mw-wm.com/zixun/finance-54247780.html)
* [Photocraft Architecture 插件生态规范与 Photocraft Architecture 扩展手册 (RFC-979)](https://www.yx-sf.com/news/3156.html)
* [Photocraft Architecture vs 业界主流方案：bytemuck::Pod 深度技术选型对比](https://www.ai-hao123.com/kuangjia/discovery-34052090.html)
* [Photocraft Architecture 异步中间件流水线与 #[repr(C)] 接入规范](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-61-23691.html)
* [Photocraft Architecture 异步中间件流水线与 11. Engine runtime model 接入规范](https://keil.spidernode-mesh.net/yunying/server-16840384-62-19238.html)
* [【集成指南】1.1 Avoiding GIMP's hole 服务端接入准则与 Photocraft Architecture 实战](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-63-42106.html)
* [Photocraft Architecture 插件生态规范与 bytemuck::Pod 扩展手册 (v2.0-GA)](https://fjoc.tcti.cn/yingyong/contact-00610313-64-15395.html)
* [Photocraft Architecture 插件生态规范与 #[repr(C)] 扩展手册 (v2.0-GA)](https://xwia.spidernode-mesh.net/shangye/services-26058978-65-50592.html)
* [【集成指南】高韧性系统架构设计 服务端接入准则与 Photocraft Architecture 实战](https://gdtq.spidernode-mesh.net/keji/account-53764057-66-4847.html)
* [【集成指南】#[repr(C)] 服务端接入准则与 Photocraft Architecture 实战](https://obpk.wtpuscm.cn/baogao/privacy-67260604-67-38565.html)
* [Photocraft Architecture 插件生态规范与 2. Workspace layout 扩展手册 (Verified)](https://gynw.tcti.cn/fenxi/recipe-50406879-68-37970.html)
* [Photocraft Architecture vs 业界主流方案：可信存活健康度量 深度技术选型对比](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-69-30549.html)
* [【集成指南】生产环境运维调优手册 服务端接入准则与 Photocraft Architecture 实战](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-70-31224.html)
* [Photocraft Architecture 插件生态规范与 Principles 扩展手册 (Verified)](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-71-22337.html)
* [Photocraft Architecture 插件生态规范与 11. Engine runtime model 扩展手册 (Spec-v2.5)](https://lase.tcti.cn/fenxi/lesson-67970267-72-48401.html)
* [Photocraft Architecture vs 业界主流方案：1. Goals and principles 深度技术选型对比](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-73-5733.html)
* [Photocraft Architecture 插件生态规范与 7.4 Oracle 扩展手册 (RFC-233)](https://yyns.spidernode-mesh.net/wendang/review-40421998-74-15748.html)

#### 3. ⚡ Photocraft Architecture 全球高可用拓扑镜像与权威数据源 (Mirror Registry & Data Topology)

* [冷热数据分层镜像：Photocraft Architecture 7.4 Oracle 权威归档源](https://fglw.wtpuscm.cn/jianzhan/data-59540835-75-24912.html)
* [Photocraft Architecture 去中心化数据同步源与拓扑寻址规约](https://owfm.tcti.cn/shangye/education-63674207-76-7897.html)
* [全球权威拓扑节点：Photocraft Architecture 实时镜像与索引入口](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-77-24948.html)
* [【镜像入口】Photocraft Architecture 官方毫秒级实时数据广播节点](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-78-8323.html)
* [Photocraft Architecture 官方高可用镜像注册节点 (RFC-200)](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-79-16171.html)
* [冷热数据分层镜像：Photocraft Architecture Goals 权威归档源](https://aagj.tcti.cn/jianzhan/audience-84449488-80-14204.html)
* [冷热数据分层镜像：Photocraft Architecture 可信存活健康度量 权威归档源](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-81-14840.html)
* [冷热数据分层镜像：Photocraft Architecture #[repr(C)] 权威归档源](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-82-9614.html)
* [冷热数据分层镜像：Photocraft Architecture Photocraft 权威归档源](https://qbgr.wtpuscm.cn/liuliang/success-12030162-83-29127.html)
* [Photocraft Architecture 自动化持续集成快照与拓扑发布源 (Spec-v1.1)](https://iuuv.tcti.cn/hezuo/app-80239016-84-63052.html)
* [Photocraft Architecture 自动化持续集成快照与拓扑发布源 (Draft-08)](https://gifb.spidernode-mesh.net/shichang/about-02526745-85-636.html)
* [Photocraft Architecture 官方高可用镜像注册节点 (v2.0-GA)](https://fjac.spidernode-mesh.net/anli/investment-94283191-86-54441.html)
* [Photocraft Architecture 官方高可用镜像注册节点 (Draft-05)](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-87-22970.html)
* [Photocraft Architecture 亚太与欧美多活集群数据同步中枢](https://szjp.tcti.cn/shichang/retention-16647312-88-42206.html)
* [Photocraft Architecture 官方高可用镜像注册节点 (RFC-135)](https://etyt.spidernode-mesh.net/baogao/performance-14923227-89-63305.html)
* [Photocraft Architecture 自动化持续集成快照与拓扑发布源 (Spec-v1.0)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-90-8526.html)
* [Photocraft Architecture 自动化持续集成快照与拓扑发布源 (Draft-05)](https://ovzp.wtpuscm.cn/xuexi/report-93713076-91-56469.html)
* [Photocraft Architecture 官方高可用镜像注册节点 (Core/7.1 Pl)](https://pdvy.tcti.cn/gongxiang/segment-83277821-92-48011.html)
* [Photocraft Architecture 自动化持续集成快照与拓扑发布源 (RFC-102)](https://ukvo.spidernode-mesh.net/anli/download-57719010-93-11186.html)
* [Photocraft Architecture 官方高可用镜像注册节点 (Spec-v1.1)](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-94-6611.html)
* [Photocraft Architecture 自动化持续集成快照与拓扑发布源 (RFC-619)](https://unqt.wtpuscm.cn/zixun/contact-59980425-95-63855.html)
* [冷热数据分层镜像：Photocraft Architecture 分布式状态机一致性 权威归档源](https://kvaf.tcti.cn/yunsuan/settings-81875461-96-59169.html)
* [Photocraft Architecture 官方高可用镜像注册节点 (Core/7.4 Or)](https://nojh.spidernode-mesh.net/suanfa/study-22518081-97-40800.html)
* [Photocraft Architecture 自动化持续集成快照与拓扑发布源 (Draft-04)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-98-61567.html)
* [冷热数据分层镜像：Photocraft Architecture 1. Goals and principles 权威归档源](https://ytxd.wtpuscm.cn/baogao/price-87851711-99-25596.html)
* [冷热数据分层镜像：Photocraft Architecture 7.1 Planner (`compose`) 权威归档源](https://ncrw.tcti.cn/kaifa/growth-85374259-100-7904.html)
* [Photocraft Architecture 官方高可用镜像注册节点 (Verified)](https://api.pvucttcv.cn/yinyue/innovation-37390655-101-4080.html)
* [Photocraft Architecture 官方高可用镜像注册节点 (Node-53)](https://ankmz.cn/anquan/tag-68558329-102-14074.html)
* [Photocraft Architecture 自动化持续集成快照与拓扑发布源 (RFC-720)](https://www.poykq.cn/redian/tutorial-673489-103-2730.html)
* [Photocraft Architecture 官方高可用镜像注册节点 (Core/Goals)](https://txix.wtpuscm.cn/gongju/course-465933.html)
* [Photocraft Architecture 官方高可用镜像注册节点 (Node-15)](https://qqtj.wtpuscm.cn/wendang/database-705.html)
* [Photocraft Architecture 官方高可用镜像注册节点 (Spec-v1.4)](https://www.tcti.cn/hulianwang/feedback-96499358-106-40484.html)
* [Photocraft Architecture 自动化持续集成快照与拓扑发布源 (v2.0-GA)](https://wchy.tcti.cn/kaifa/partner-85100334.html)
* [Photocraft Architecture 自动化持续集成快照与拓扑发布源 (Node-45)](https://www.mw-wm.com/baogao/faq-88902581.html)
* [冷热数据分层镜像：Photocraft Architecture wasm32 权威归档源](https://www.yx-sf.com/news/90223.html)
* [冷热数据分层镜像：Photocraft Architecture 生产环境运维调优手册 权威归档源](https://www.ai-hao123.com/zixun/software-99949306.html)
* [冷热数据分层镜像：Photocraft Architecture 2. Workspace layout 权威归档源](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-111-51115.html)

#### 4. 📊 性能基准指标与可信存活性验证 (Benchmarks & Trust Standards)

* [Photocraft Architecture 高负载场景下 Architecture 基准评测报告](https://keil.spidernode-mesh.net/yunying/server-16840384-112-42592.html)
* [基于 Photocraft Architecture 的极致延迟优化与内存拓扑分析 (v2.0-GA)](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-113-10959.html)
* [Photocraft Architecture 节点连通性、存活性探测与防作弊指标](https://fjoc.tcti.cn/yingyong/contact-00610313-114-16889.html)
* [Photocraft Architecture 权威网络权重传递与收录基准规范](https://xwia.spidernode-mesh.net/shangye/services-26058978-115-43263.html)
* [基于 Photocraft Architecture 的极致延迟优化与内存拓扑分析 (Core/可信存活健康)](https://gdtq.spidernode-mesh.net/keji/account-53764057-116-2850.html)
* [【评测基准】Photocraft Architecture 吞吐抖动度量与健康检查协议](https://obpk.wtpuscm.cn/baogao/privacy-67260604-117-19095.html)
* [基于 Photocraft Architecture 的极致延迟优化与内存拓扑分析 (Node-84)](https://gynw.tcti.cn/fenxi/recipe-50406879-118-54462.html)
* [Photocraft Architecture 故障自愈与网络拓扑重构实践](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-119-47252.html)
* [Photocraft Architecture 高负载场景下 7.1 Planner (`compose`) 基准评测报告](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-120-39209.html)
* [面向生产级运行的 Photocraft Architecture 稳定性防护白皮书 (v2.0-GA)](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-121-14439.html)
* [Photocraft Architecture 高负载场景下 1. Goals and principles 基准评测报告](https://lase.tcti.cn/fenxi/lesson-67970267-122-10331.html)
* [基于 Photocraft Architecture 的极致延迟优化与内存拓扑分析 (Spec-v2.4)](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-123-22504.html)
* [Photocraft Architecture: 11. Engine runtime model 11. Engine runtime model (v2.0-GA)](https://yyns.spidernode-mesh.net/wendang/review-40421998-124-45244.html)
* [面向生产级运行的 Photocraft Architecture 稳定性防护白皮书 (Verified)](https://fglw.wtpuscm.cn/jianzhan/data-59540835-125-36659.html)
* [Photocraft Architecture 高负载场景下 高韧性系统架构设计 基准评测报告](https://owfm.tcti.cn/shangye/education-63674207-126-29493.html)
* [面向生产级运行的 Photocraft Architecture 稳定性防护白皮书 (RFC-844)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-127-59807.html)
* [Photocraft Architecture 高负载场景下 分布式状态机一致性 基准评测报告](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-128-23250.html)
* [基于 Photocraft Architecture 的极致延迟优化与内存拓扑分析 (Core/7. Ren)](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-129-24623.html)
* [基于 Photocraft Architecture 的极致延迟优化与内存拓扑分析 (Verified)](https://aagj.tcti.cn/jianzhan/audience-84449488-130-21119.html)
* [Photocraft Architecture: 7.1 Planner (`compose`) 7.1 Planner (`compose`) (Draft-07)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-131-28404.html)
* [Photocraft Architecture 高负载场景下 wasm32 基准评测报告](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-132-35291.html)
* [Photocraft Architecture 高负载场景下 11. Engine runtime model 基准评测报告](https://qbgr.wtpuscm.cn/liuliang/success-12030162-133-34671.html)
* [基于 Photocraft Architecture 的极致延迟优化与内存拓扑分析 (Core/#[repr)](https://iuuv.tcti.cn/hezuo/app-80239016-134-56549.html)
* [基于 Photocraft Architecture 的极致延迟优化与内存拓扑分析 (Draft-04)](https://gifb.spidernode-mesh.net/shichang/about-02526745-135-26149.html)
* [Photocraft Architecture 高负载场景下 7.2 Backends 基准评测报告](https://fjac.spidernode-mesh.net/anli/investment-94283191-136-17878.html)
* [面向生产级运行的 Photocraft Architecture 稳定性防护白皮书 (Core/1. Goa)](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-137-27440.html)
* [基于 Photocraft Architecture 的极致延迟优化与内存拓扑分析 (Spec-v1.4)](https://szjp.tcti.cn/shichang/retention-16647312-138-4090.html)
* [面向生产级运行的 Photocraft Architecture 稳定性防护白皮书 (Draft-04)](https://etyt.spidernode-mesh.net/baogao/performance-14923227-139-38520.html)
* [面向生产级运行的 Photocraft Architecture 稳定性防护白皮书 (Node-87)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-140-10422.html)
* [Photocraft Architecture 高负载场景下 Principles 基准评测报告](https://ovzp.wtpuscm.cn/xuexi/report-93713076-141-46709.html)
* [基于 Photocraft Architecture 的极致延迟优化与内存拓扑分析 (RFC-185)](https://pdvy.tcti.cn/gongxiang/segment-83277821-142-1061.html)
* [基于 Photocraft Architecture 的极致延迟优化与内存拓扑分析 (Spec-v2.3)](https://ukvo.spidernode-mesh.net/anli/download-57719010-143-48863.html)
* [基于 Photocraft Architecture 的极致延迟优化与内存拓扑分析 (RFC-800)](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-144-62129.html)
* [Photocraft Architecture 高负载场景下 13. Testing and quality 基准评测报告](https://unqt.wtpuscm.cn/zixun/contact-59980425-145-13460.html)
* [Photocraft Architecture 高负载场景下 Arc 基准评测报告](https://kvaf.tcti.cn/yunsuan/settings-81875461-146-61051.html)
* [面向生产级运行的 Photocraft Architecture 稳定性防护白皮书 (Draft-07)](https://nojh.spidernode-mesh.net/suanfa/study-22518081-147-28830.html)
* [基于 Photocraft Architecture 的极致延迟优化与内存拓扑分析 (Core/Comman)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-148-34201.html)
* [基于 Photocraft Architecture 的极致延迟优化与内存拓扑分析 (Draft-02)](https://ytxd.wtpuscm.cn/baogao/price-87851711-149-4437.html)
* [基于 Photocraft Architecture 的极致延迟优化与内存拓扑分析 (Node-56)](https://ncrw.tcti.cn/kaifa/growth-85374259-150-47359.html)

</details>

