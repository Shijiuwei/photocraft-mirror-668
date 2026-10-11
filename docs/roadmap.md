# Roadmap

Status legend: ✅ done · 🟡 in progress · ⬜ not started. Updated 2026-10-08.

**Parity metrics.** `cargo xtask parity` measures how much of Photoshop's menu tree is *wired to a
command* and writes [`parity.md`](parity.md). It does **not** measure whether those commands behave
like Photoshop, feel right, or survive real files. Read it next to the PSD composite oracle, the
test count, and the [honest parity assessment](#honest-parity-assessment-2026-10-05) below, which
is the reference answer to "how close are we really". The weighted feature estimate in
[`parity-estimate.md`](parity-estimate.md) (~82%) counts feature surface and overstates user-facing
readiness; treat it as an upper bound.

| M | Status | Where we are |
|---|---|---|
| M0 Skeleton | ✅ | workspace, xtask (layers / wasm / ci / stats / corpus / parity), CI workflow |
| M1 Foundation | ✅ | geom, color (27 blend modes), raster (COW tiles, any depth), doc, ops, cms (ICC) |
| M2 PSD v1 | ✅ | photocraft-psd: 134/135 real files byte-exact round trip |
| M3 Viewer app | ✅ | egui shell (Pro / Studio / Classic themes), 13+ codecs, native and web (trunk) builds |
| M4 Native format + engine | ✅ | 500+ commands, `.pcraft` (incremental, autosave, crash recovery), CLI, persistent preferences |
| M5 GPU compositor | 🟡 | wgpu compositor drives the canvas, layer effects, vector masks, artboards, pattern fills and every clip case included (≤1/255 vs CPU); Multichannel documents fall back to the CPU |
| M6 Paint + select | 🟡 | brush engine, all selection tools, multi-layer selection, snapping + smart guides, free transform + warp; stylus pressure on Windows (WM_POINTER via winit), the web (Pointer Events, with tilt/twist), macOS (AppKit event monitor: pressure, tilt, rotation, eraser end) and Linux X11 (XInput2 raw valuators); on Linux Wayland a pen opens the window through Xwayland (native tablet-v2 still open, #79) |
| M7 Adjust + filters | 🟡 | 16 adjustment layers + destructive-only adjustments, 70+ filters incl. Blur Gallery, Actions record/replay, Fade |
| M8 PSD v2 | 🟡 | adjustments (incl. Selective Color, Color Lookup), fills, effects, patterns, text, shapes, smart objects, alpha channels; oracle 111/170 |
| M9 Text, vector, styles | 🟡 | type engine + Warp Text, shapes / pen / paths, all 10 effects on CPU and GPU (parity ≤1/255, 30/31 corpus effect files on the GPU) |
| M10 Smart features | 🟡 | classical Select Subject / Object, content-aware fill and scale, healing, auto-align / auto-blend; ML backend not started |
| M11 Automation + formats | 🟡 | MCP (headless + live bridge), batch, Image Processor, prefs over MCP; DoD test passes (10 agent tasks over MCP, `automation/tests/agent_tasks.rs`); since 2026-10-07 the CLI rejects unknown flags and answers `<subcommand> --help`, batch runs report same-name outputs instead of overwriting them, MCP `command_batch` steps can start background jobs, and headless saves never flatten over the opened file; JP2 / DICOM / DPX / C2PA pending |
| M12 Pro parity | 🟡 | CMYK / Lab / Indexed / Bitmap / Duotone, ICC + soft proofing, channels + Quick Mask, smart-object stack modes, artboards, layer comps; print, HDR, photomerge, timeline pending |

**Menu parity: 532 / 625 (85.1%)** on 2026-10-01, up from 224 (35.8%) the day before. See [`parity.md`](parity.md).

2026-10-08: Affinity `.af`, `.afdesign`, `.afphoto` and `.afpub` documents (container versions
8–12) open **natively** ([format notes](../crates/affinity/README.md)): pages and artboards, layers
and groups, curves and geometric shapes with fills, gradients and strokes, text as type layers,
placed images as smart objects, pixel layers and masks, as an 8-bit RGB document without a source
save path. Measured against the thumbnail Affinity embeds in each file, 21 pinned public documents
(`cargo xtask corpus --affinity`) differ by 0–4.6 of 255 on average. Layer effects, adjustments,
live filters, brush strokes, special shapes, master pages and CMYK/Lab/16-bit document colour are
approximated or left out, each with a warning; damaged or unknown files fall back to the embedded
preview. Affinity writing is not implemented: no Affinity installation was available to check
written files, so `.af` export stays unsupported.

2026-10-09: `corpus/affinity/` now has 39 pinned documents and 21 PNGs (20 rendered references and
one bitmap-fill texture): the prior 21 public documents plus 18 CC0 samples for #1606. The new
samples compare against their exported PNGs;
mean differences range from 0.44/255 (conical gradients) to 86.05/255 (RGB/32 reduced to 8-bit).
Clouds, hearts, cogs, callouts, arrows, double stars, tears, crescents, diamonds and circular segments
now import as parametric shapes; the special-shapes sample measures 5.68/255 against its PNG. The
remaining gaps in #1606 stay tracked with per-sample corpus ceilings.

## Honest parity assessment (2026-10-05)

This is the reference answer to "how close are we to Photoshop parity, really". Agents: read it
before picking work. Update it (with dated measurements) when the numbers move; don't restate the
menu-parity number in its place.

**Measured numbers live in the [scorecard](scorecard.md)** (`cargo xtask scorecard`, kept current
by CI): performance scenarios against their budgets, corpus floors, per-area checklists with
done / partial / missing counts, and the number of settings that do nothing. Where the scorecard
and the estimates below disagree, the scorecard's numbers supersede them. As of 2026-10-05 it
counts 67 of 129 preferences that nothing reads; 3 of the 25 budgeted performance scenarios meet
their #209–#211 targets (first baseline, taken on a heavily loaded machine), and the 150-layer
nudge scenario crashes the GPU compositor; the open area issues' checklists (#203–#220) are
almost entirely missing or partial.

2026-10-07: UI Font Size now applies to interface text, including CJK fallbacks, independently
of display and canvas zoom (#532). The current preference audit drops from 59 to 58 unread
settings out of 135; see the regenerated scorecard.

2026-10-09: Output-preserving Radial Blur direct sampling measured on a local
i7-9750H, 12 workers, 24 MP RGBA8, Good quality, amount 1: Spin 66.10 → 16.43 s
(4.02×), Zoom 19.19 → 5.35 s (3.59×), one paired run each, exact output equality.
These are local measurements at amount 1; see [method and limits](radial-blur-performance.md).

2026-10-10: native selection distance transforms measured 7.59–10.57x faster on six
24–36 MP synthetic masks on an AWS c7i.4xlarge (16 workers, three paired release runs).
The transform also corrects f32 envelope errors past coordinate 4096, including nonzero
distances at selected pixels. The parallel path needs an additional four bytes per pixel;
see [measurements, correctness and limits](selection-distance-performance.md).

**Bottom line.** Two days after 0.2.0 we had merged ~96 PRs and closed ~48 issues, but **real
Photoshop parity is still well below 50%**. The biggest gaps are AI, missing tools, professional
workflow depth and the plug-in ecosystem. Most fixes since 0.2.0 have passed our tests but have
**not yet been validated by users**, and this week proved our tests miss what users hit:
on first public use they found broken basics (text selection offset, 214 shortcut failures, panels
resizing themselves, an immovable crop frame, folders that wouldn't collapse, lag on layout PSDs)
that all counted as "live" in `parity.md`.

**Overall.** Feature surface: roughly **60–70%** of what a typical Photoshop user touches exists in
some form. "A professional could switch for daily work": roughly **25–35%**. True 1:1 parity is
**many months** of focused work, and some areas need product decisions rather than effort.
Confidence: moderate — the next users of 0.2.x will move these numbers either way.

### By dimension

2026-10-10: [Shape stroke controls](shape-strokes.md) expose existing Solid/Dashed/Dotted,
custom dash/gap, offset, caps, joins, alignment, miter-limit and opacity capabilities in
Shape/Pen options and Properties. Properties stages a cached canvas preview before one
undoable edit. Gradient fill controls from #1053 remain open; the Line tool still draws a
filled bar. This extends UI reachability and does not claim Photoshop-authored stroke parity.

| Dimension | Measured / evidence (2026-10-05) | Grade | Notes |
|---|---|---|---|
| Menu wiring | 626/626 menu items dispatch a command (`parity.md`) | high but shallow | Says nothing about behaviour. |
| PSD fidelity (rendering) | Corpus oracle 115/170 (68%): 30 differ, 26 have no usable reference, 1 import error. 2026-10-07: io corpus 146/170, psd-tools corpus 236/309 (was 229: Advanced Blending knockouts), Photoshop oracles 132/258 | medium | Push to 170/170 under way (effects/strokes, multi-instance effects, 16/32-bit and colour modes, references for skipped files). |
| PSD round trip | 169/169 re-import identically; every adjustment layer and blend mode round-trips | high (within corpus) | Floors in `crates/io/tests/corpus.rs`; raise, never lower. Corpora: `cargo xtask corpus --all` (ours: https://github.com/storytold/photocraft-corpus). |
| OpenRaster read/write | 2026-10-09: 34 Krita 5.2.9-authored `.ora` files (27 blend modes, groups, pass-through, offsets, opacity, visibility, masks, 16-bit, gray) render within 3/255 of Krita's merged image (30) or differ for known reasons (4: Krita's lighter/darker-colour tie-break, Hard Mix at exactly 1, a linear-light 16-bit document); Krita re-opens our exports and renders them identically to its originals (33/34; Soft Light is written as `svg:soft-light`) | medium (one authoring app) | MyPaint and GIMP files untested. Masks are applied to pixels and layer styles dropped on save (reported). See [OpenRaster](ora.md). |
| Paint.NET import | 2026-10-09: 14/14 local PDN3 textures preserve editable layers exactly through `.pcraft` and render within 2.142/255 of full-size previews; a supplied 5.x document (1280×720, 11 layers) passes editing/history/native saves and matches its thumbnail within 0.418/255 mean; all 14 blend modes exercised synthetically | medium (limited corpus) | Import only; metadata omitted. More current-version files and full-size exports needed. See [PDN support](pdn.md). |
| Smart filters / text / effect shapes in PSDs | Measured on our Photoshop-authored set (https://github.com/storytold/photocraft-corpus, `corpus/photoshop`, 258 files): see the per-group floors in `crates/io/tests/corpus.rs` and `crates/engine/tests/photoshop_oracles.rs`. Smart objects and smart filters now survive PSD save and open (41 corpus files, 206 smart objects, round trip strict; Photoshop opens our exports with live filters); re-rendering Photoshop's smart filters with ours matches 5/30 (was 1/30) | low–medium | Remaining re-render gaps are filter maths (Gaussian/Motion Blur, Unsharp Mask, Emboss, Add Noise RNG) and bicubic placement. |
| Core editing (layers, masks, selections, adjustments, filters, transforms) | Broad engine coverage; many interaction bugs fixed after 0.2.0 (adjustment dialogs, Curves, crop, Move/Transform modifiers, gesture origin) | medium | Fixes not yet user-validated. |
| UI / UX polish | Shortcut audit 214 → 0 failures; dock, Layers rows and menus reworked; first visual-QA sweep found 14 defects (#147–#157). 2026-10-07: the keyboard-only shortcuts with no menu item (⌥[ ⌥] ⌥, ⌥. layer navigation, ⇧⌥[ ⇧⌥] to extend the selection, 1–0 for opacity and ⇧ for flow or fill, ⇧[ ⇧] hardness, ⌥⌘T to transform a copy and ⌥⇧⌘T to step and repeat), ⌥-click colour sampling with painting tools, double-click a Layers row for Layer Style, File › New from Clipboard, a centred main window and remembered Liquify settings (#352, #417, #350, #368, #419, #418). 2026-10-09: canvas zoom is physical at any display scale — 100% is one document pixel per physical display pixel, and every screen-space overlay, cursor, scrollbar, navigator and fit follows it (#1943) | low–medium | Needs recurring visual QA with realistic documents. |
| Tools | 2026-10-09: Pencil, Mixer Brush, Patch, Content-Aware Move, Vertical Type, Pattern Stamp and Rotate View are toolbar tools. Remaining missing tools include Red Eye, Art History Brush, Freeform/Curvature Pen, Add/Delete Anchor Point tools, single row/column marquee, Color Sampler, Perspective Crop, type masks and Frame | low–medium | Patch has a live healing preview. Content-Aware Move is partial: Move/Extend, Structure/Color and Sample All Layers exist; Transform On Drop and a live result preview remain missing (`TOOL-213-4` in the scorecard). Magic/Background Eraser added; live gradients in progress (#180). Magnetic Lasso added 2026-10-08 (live-wire edge tracing, Width/Contrast/Frequency, `select.magneticLasso`). Pattern Stamp added 2026-10-08 (S flyout, `paint.patternStamp`, Aligned / Impressionist). Rotate View (2026-10-09) turns the canvas camera around its centre without rewriting pixels; Reset View and Match Rotation copy the angle. Crop rotation (2026-10-09, #1792): dragging outside the crop frame turns it (⇧ 15° steps, angle readout), and ↵ rotates the document and crops in one undo step (`image.crop` `angle`); the frame turns over the image as in Photoshop's Classic Mode, and the options-bar Straighten is still missing. |
| Painting | Brush model and Brush Settings panel near Photoshop; .abr/.grd import; persistent presets; pen pressure/tilt on Windows, web, macOS and X11 | medium | Native Wayland pen input open (#79; a pen opens the window through Xwayland meanwhile); X11 pressure confirmed by a user, macOS not yet verified on tablet hardware. |
| Text / typography | Engine works; caret placement and size editing fixed; OpenType features, text-on-path editing, composer parity partial | medium-low | Measure with the Photoshop-authored set. |
| Colour management | Colour-managed canvas (document → monitor), embedded CMYK profiles, linear EXR/HDR, 16-bit float canvas | medium-high | Monitor profile follows only at launch. |
| Performance | 14k+ px on the GPU at ~⅓ the memory; adjustment preview 285 ms → 4–9 ms; font-size edits 297 ms → 4.6 ms; 2026-10-07: 30 MP TIFF open (banded, parallel strip/tile decode) Deflate 345 → 32 ms, LZW 428 → 43 ms, BigTIFF and every IFD readable; 2026-10-09: 4.2 MP Indexed Color, 256 colours, full-resolution CPU preview p50 3018 → 1061 ms (Ryzen AI 7 350, three measured runs; GPU upload excluded) | medium-high on rasters | Complex layout documents still laggy (#125/#128); >16384 px GPU tiling in progress (#49). Native Indexed Color previews run on one background worker with stale-result rejection; large palettes can still take about a second to compute. Web previews remain synchronous. |
| Stability | Never-crash lint series, crash guard, `panic_hunt` fuzzing in the gate | medium-high | No field crash data yet. |
| Camera RAW | DNG, CR2, Sony ARW (lossless + compressed), Nikon NEF (lossless + lossy compressed), RW2, uncompressed ORF; 2026-10-09: uncompressed Fujifilm RAF, Bayer and X-Trans (edge-directed X-Trans demosaic; 26 MP X-Trans develop 306 ms on 12 cores) | medium | CR3, compressed RAF, Nikon "lossy after split" and calibrated colour for non-DNG cameras (no Fuji colour calibration: neutral fallback) still open (#50). |
| AI / generative | none | ~0% | Deferred by decision (#41). |
| Ecosystem | Sandboxed WebAssembly plug-ins instead of .8BF; no ExtendScript/UXP/.atn; no Adobe Fonts/Libraries/cloud docs | low | By design for 8BF; scripting compatibility open. |
| Platforms | macOS (notarized), Windows, Linux (AppImage/deb/rpm/Flatpak bundle), web | medium-high | Flathub later (#173); Windows signing material pending. |
| Localisation | 2026-10-07: 10 UI languages; menu, `tl!`, blend mode, preference and brush-section coverage enforced by tests; live switching and scoped Preferences previews | medium | Engine errors/status messages still partly English; CJK web fonts, browser-locale detection, and RTL remain open. |

2026-10-08: Motion Blur adds adaptive FFT convolution for wide streaks while retaining the
row kernel merged in [#902](https://lase.tcti.cn/fenxi/lesson-67970267.html) for shorter
streaks and unsupported cases. Native release medians of three paired runs on an Intel
i7-9750H with 12 Rayon workers, synthetic RGBA8 at 30° / distance 1000: 24 MP
22.594 → 9.785 s (2.31×), and 1.5 MP
2.455 → 0.592 s (4.15×).
Full-image comparisons differ by at most one U8 code level; distance 64/256 retain the
byte-identical row output. These measure decode/halo/filter/write/prune, excluding UI proxy
creation, compositing and upload; the UI remains synchronous. FFT work is scheduled within
a conservative 512 MiB working-set estimate, excluding stored Surface tiles and row fallback
memory. The crossover is a heuristic; images dominated by near-cutoff alpha can need extra
scalar work. These measurements do not establish Photoshop filter parity.

2026-10-10: Proximity Match search medians on an AWS c7i.4xlarge improved 1.54–3.82x
for four synthetic stroke regions (50–300 px holes), with identical source displacements.
Bounding the mask table to the hole and pruning losing SSD candidates reduced table storage
by about 90–93 percent there. Full 24–36 MP kernel inputs improved 4.60–6.24x, but normal
engine strokes already use a cropped region. See [method and limits](proximity-match-performance.md).

2026-10-09: the Remove tool is in the J flyout (`paint.remove`, a background job). A stroke
around an object also removes what it encloses. The fill is non-local patch completion with
texture features (A. Newson et al., IPOL 2017: `crates/algo/src/nonlocal.rs`), finished by a
best-patch copy and gradient-domain seam hiding. On 20 holes in public-domain photos
(`remove_quality`, `docs/development.md` › Remove Tool quality) the fill keeps 0.89 of the
original's texture (Wexler/PatchMatch completion as Content-Aware Fill uses it: 0.54). On a 24 MP
document a 100 px ring takes 120 ms and a 1000 px ring 2.5 s (`remove_bench`). It is not
generative: large objects over complex structure fill less convincingly than Photoshop's AI mode
(#41).

2026-10-08: the tools update above is checked against the current toolbar groups in
[`panels.rs`](../crates/ui-egui/src/panels.rs), the Pencil and tool-cycle tests in
[`pencil_tests.rs`](../crates/ui-egui/src/pencil_tests.rs), Patch's live-preview test in
[`patch_preview.rs`](../crates/ui-egui/src/patch_preview.rs), and Vertical Type's point/paragraph
creation and undo test in [`type_tool_tests.rs`](../crates/ui-egui/src/type_tool_tests.rs).
These implementations remove the tools from the missing list; they do not establish Photoshop
behavioural parity. The historical estimates and corpus measurements in this assessment are
unchanged; current measured floors remain in the scorecard.

2026-10-07: Camera Raw PSD mapping covers relative custom white balance, Light/Presence,
parametric and four point curves, HSL, Color Grading, sharpening/noise detail, grain and numeric
post-crop vignette controls. Two revisions of one supplied Photoshop ACR 18.4 PSD preserve their
filter descriptors without edits. The updated revision stays opaque because of active manual
Optics and unverified vignette style; a descriptor projection verifies its supported controls.
Synthetic 8/16/32-bit PSD → edit → `.pcraft` → PSD tests preserve the filter stack, mask and
undo/redo. Unmapped Camera Raw fields/versions remain opaque; Photoshop
acceptance of generated exports and pixel parity are still unverified. Corpus floors above are
unchanged.

2026-10-10: Select and Mask's Shift Edge now computes the exact octagonal footprint
with a square window extreme plus a dyadically built Manhattan ball. On a c7i.4xlarge
(16 vCPUs), full tiled CPU refinement at radius 64 / Shift Edge +100% takes
3.019 s -> 0.762 s at 24 MP (3.96x),
3.706 s -> 0.921 s at 36 MP (4.02x).
Three alternating paired release runs include guide sampling, guided filtering,
boundary distances, quantization and assembly. Every output mask matches exactly.
The dense fractional 24 MP radius-64 dilation kernel improves 36.00x;
circular soft-mask kernels improve 20.76-172.16x at 24-36 MP, radii 4-64.
Bounded halo tiles reduce the large-mask workspace allocation bound from 192/288 MB
to about 115/163 MB, including output and 16 workers' scratch, excluding the source.
See [method and results](octagonal-mask-performance.md).

### Where we're going (priority order)

1. **Ship the fixes:** cut 0.2.1 once the current batch lands, so users validate them.
2. **PSD fidelity to 170/170** with round trips, plus the Photoshop-authored reference set for smart
   filters, the text engine and effect shapes (https://github.com/storytold/photocraft-corpus, fetched by
   `cargo xtask corpus --all`); enforce floors in `corpus.rs`.
3. **Workflow acceptance tests:** 30–50 real tasks (retouch a portrait, social post with text and
   effects, composite with masks and adjustment layers, CMYK print prep…) scripted end to end and
   checked against Photoshop's output on every build. Their pass rate becomes the headline parity
   number.
4. **Tool coverage and workflow depth:** Pen variants (Direct Selection landed, #790) and
   Perspective Crop; finish Content-Aware Move's Transform On Drop and live result preview.
   Pencil, Mixer Brush, Patch, Vertical Type, Pattern Stamp and Rotate View are already available on the toolbar.
5. **Complex-document performance** (#125/#128) and GPU tiling beyond the texture limit (#49).
6. **Recurring visual QA** (`cargo run -p photocraft-engine --example designer_psd`) and fast
   turnaround on user reports (OS, document size, layer count, screenshot).
7. Later / needs decisions: generative AI backend (#41), scripting compatibility (ExtendScript /
   UXP / .atn), Flathub (#173), Wayland pen pressure (#79).

## Current focus (infrastructure before the long tail)

Landed on 2026-10-01:
- multi-layer selection, live smart objects + smart filters, alpha channels + Quick Mask;
- patterns, Warp, preferences, snapping, ~33 filters, the remaining core adjustments;
- Layer Comps and Artboards, GPU layer effects, Liquify / Puppet Warp / Perspective Warp;
- the PSD fidelity pass (oracle 102 → 111), the M11 MCP acceptance test, and the release pipeline
  (`docs/releasing.md`).

Next:
1. **Follow "Where we're going" in the assessment above.** 0.2.0 shipped signed and notarized on
   2026-10-05 (`docs/releasing.md`); Windows code-signing material still needs to be obtained.
2. **Fidelity**: the PSD oracle (113/170). Modern Brightness/Contrast and grayscale Levels
   curves; chisel-soft / stroke-emboss bevel shapes; Photoshop's 8-bit blend rounding; non-Normal
   modes in Lab documents; smart-filter maths against Photoshop's (re-render oracle 5/30).
3. **GPU**: only Multichannel documents and regions over the texture limit fall back to the CPU.
4. **Vanishing Point, Camera Raw / Lens Correction, Face-Aware Liquify** (needs a landmark model).
5. **Panels**: Patterns, Styles, Glyphs, Character/Paragraph Styles, Timeline; Custom Shape tool.
6. Print, Photomerge, Merge to HDR, video layers.

## Milestone definitions

Each milestone has a **definition of done (DoD)** and must leave `main` green on all Tier-1 platforms plus a wasm build.

| M | Name | Scope (key items) | DoD / acceptance |
|---|---|---|---|
| **M0** | Skeleton | Workspace, all crate stubs, lints, `xtask` (layers check, ci), CI matrix, testkit | `cargo test --workspace` green; `cargo check --target wasm32-unknown-unknown -p photocraft-engine -p photocraft-ui-egui` green; layering check passes |
| **M1** | Foundation | geom, color (formats, blend math for all 27 modes, sRGB/linear), raster (sparse COW tiles, U8/U16/F32), doc (full type model incl. CMYK/Lab/adjust/smart), ops (history) | Property tests (proptest) on tile COW and history; blend-mode reference tests against published formulas |
| **M2** | PSD v1 | `photocraft-psd`: header, resources, layer records, channel data (raw/RLE/ZIP/ZIP+pred), masks, groups (lsct), unicode names, unknown-block passthrough, merged image, PSB; writer | Round-trip byte-stability tests; synthetic PSD generator tests; fuzz target; ≥150 unit tests |
| **M3** | Viewer app | codecs (png/jpeg/tiff/webp/gif/bmp/tga/pnm/qoi/exr/hdr, all read+write), CPU compositor, egui shell (menu from registry, canvas pan/zoom, layers panel, history), open/save, native + web build | Opens PNG/JPEG/PSD, shows layers, toggles visibility, undo/redo, saves PNG/PSD; web build loads a file from the browser |
| **M4** | Native format + engine | `.pcraft` bundle, command registry with schemas, jobs/cancellation, snapshots via arc-swap, CLI (`convert`, `run`, `inspect`) | CLI parity tests: every GUI command also runs headless |
| **M5** | GPU compositor | wgpu planner backend, tile residency, parity tests vs CPU (all blend modes), viewport on GPU, mips | GPU vs CPU ≤1/255; pan/zoom 60 fps on a 100 MP / 20-layer document |
| **M6** | Paint + select | platform input (pen: macOS/Windows/Linux/web), brush engine v1, eraser, marquee/lasso/wand, move, free transform, crop | Brush latency <1 frame; selection-restricted paint tests |
| **M7** | Adjust + filters | 16 adjustment layers, first 30 filters, schema-generated dialogs + live preview, actions record/replay | Golden tests at 8/16/32f; action replay determinism |
| **M8** | PSD v2 | adjustment layers, fill layers, lfx2 effects (descriptor parser), text (EngineData) preserve+render, smart objects, vector masks | Composite-oracle pass rate ≥90% on corpus |
| **M9** | Text, vector, styles | parley text layers, shapes/pen/paths, all 10 layer effects on GPU | Visual goldens; PSD text round-trip |
| **M10** | Smart features | `ml` (ort native / ort-web on the web), Select Subject/Object/Sky, Remove BG, Remove tool, content-aware fill, healing, AI denoise, RAW develop | Quality benchmarks on a public dataset; timing budgets |
| **M11** | Automation + formats | MCP server, batch, scripting, remaining formats (JP2, DICOM, DPX…), C2PA | An agent completes 10 scripted edit tasks via MCP |
| **M12** | Pro parity | CMYK/Lab UI, print, HDR display, photomerge/HDR merge, timeline, layer comps, artboards, symmetry, neural filters | `xtask parity` ≥ 90% of Photoshop menu checklist |


---

<details open>
<summary><b>📚 photocraft 官方技术规约与全息拓扑引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目「photocraft」遵循开放开源与知识共享协议。以下引用矩阵涵盖项目架构白皮书、高可用镜像注册中心、分布式通信契约与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 photocraft 核心架构规约与设计白皮书 (Architecture & Core Specs)

* [高韧性系统架构设计 核心系统架构与设计规约 (Verified)](https://www.yx-sf.com/tech/30028.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-04)](https://ankmz.cn/anquan/tag-68558329.html)
* [【官方规范】photocraft Photoshop 核心运行拓扑标准](https://fglw.wtpuscm.cn/jianzhan/data-59540835.html)
* [Everything in the box 核心系统架构与设计规约 (Draft-08)](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [photocraft 内部组件解耦与事件状态机规范 (Verified)](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [现代 Rust高性能网络 架构演进之路 —— photocraft 深度实践](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-543)](https://nojh.spidernode-mesh.net/suanfa/study-22518081.html)
* [面向大规模网络的 photocraft 工业级架构基准](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [【官方规范】photocraft Engine first: 核心运行拓扑标准](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [photocraft 内部组件解耦与事件状态机规范 (v2.0-GA)](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [基于 photocraft 的高吞吐 Rust高性能网络 设计白皮书](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696.html)
* [PSD without compromise 核心系统架构与设计规约 (Draft-06)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869.html)
* [基于 photocraft 的高吞吐 分布式架构 设计白皮书](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (RFC-605)](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-791)](https://www.tcti.cn/hulianwang/feedback-96499358.html)
* [现代 分布式架构 架构演进之路 —— photocraft 深度实践](https://unqt.wtpuscm.cn/zixun/contact-59980425.html)
* [photocraft 内部组件解耦与事件状态机规范 (Core/Engine)](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-560)](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (Verified)](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [基于 photocraft 的高吞吐 搜索引擎调度优化 设计白皮书](https://iozk.spidernode-mesh.net/fenxi/quality-22754958.html)
* [photocraft 内部组件解耦与事件状态机规范 (Spec-v1.6)](https://owfm.tcti.cn/shangye/education-63674207.html)
* [【photocraft】Rust 核心系统架构与设计规约 (v2.0-GA)](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534.html)
* [现代 搜索引擎调度优化 架构演进之路 —— photocraft 深度实践](https://www.poykq.cn/redian/tutorial-673489.html)
* [基于 photocraft 的高吞吐 开源镜像站 设计白皮书](https://www.mw-wm.com/anfang/mobile-67262414.html)
* [【官方规范】photocraft Downloads 核心运行拓扑标准](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [Engine first: 核心系统架构与设计规约 (v2.0-GA)](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [photocraft 内部组件解耦与事件状态机规范 (RFC-671)](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-05)](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [现代 开源镜像站 架构演进之路 —— photocraft 深度实践](https://iozk.spidernode-mesh.net/fenxi/quality-22754958.html)
* [Everything in the box 核心系统架构与设计规约 (Draft-02)](https://pdvy.tcti.cn/gongxiang/segment-83277821.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (v2.0-GA)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [【官方规范】photocraft clean-room 核心运行拓扑标准](https://www.yx-sf.com/news/82919.html)
* [Engine first: 核心系统架构与设计规约 (RFC-459)](https://rwya.wtpuscm.cn/jishu/finance-327.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (RFC-511)](https://www.tcti.cn/hulianwang/feedback-96499358.html)
* [【官方规范】photocraft photocraft 核心运行拓扑标准](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (Node-19)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-07)](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)

#### 2. 🔌 生产级模块集成与接口调用规范 (APIs & Production Integration)

* [photocraft 核心 API 接口契约与客户端调用指南](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [photocraft vs 业界主流方案：搜索引擎调度优化 深度技术选型对比](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [【集成指南】Rust高性能网络 服务端接入准则与 photocraft 实战](https://rimv.wtpuscm.cn/paiming/contact-144686.html)
* [基于 photocraft 的自动化部署与生产环境配置实践](https://iozk.spidernode-mesh.net/fenxi/quality-22754958.html)
* [【生产手册】photocraft 模块通信与请求穿透标准](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html)
* [photocraft vs 业界主流方案：分布式架构 深度技术选型对比](https://www.yx-sf.com/tech/98146.html)
* [photocraft 异步中间件流水线与 生产环境运维调优手册 接入规范](https://www.mw-wm.com/xuexi/creative-85406191.html)
* [photocraft 插件生态规范与 Under the hood 扩展手册 (Node-87)](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)
* [【集成指南】开源镜像站 服务端接入准则与 photocraft 实战](https://owfm.tcti.cn/shangye/education-63674207.html)
* [photocraft 异步中间件流水线与 photocraft 接入规范](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [【集成指南】搜索引擎调度优化 服务端接入准则与 photocraft 实战](https://pdvy.tcti.cn/gongxiang/segment-83277821.html)
* [photocraft 异步中间件流水线与 Windows 接入规范](https://www.poykq.cn/redian/tutorial-673489.html)
* [photocraft 插件生态规范与 高韧性系统架构设计 扩展手册 (Draft-07)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932.html)
* [photocraft 异步中间件流水线与 Security 接入规范](https://szjp.tcti.cn/shichang/retention-16647312.html)
* [photocraft 插件生态规范与 Test corpora 扩展手册 (Spec-v2.0)](https://www.poykq.cn/redian/tutorial-673489.html)
* [photocraft 插件生态规范与 macOS 扩展手册 (Draft-08)](https://pltj.wtpuscm.cn/paiming/tag-535963.html)
* [photocraft 异步中间件流水线与 Test corpora 接入规范](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [photocraft 插件生态规范与 分布式状态机一致性 扩展手册 (RFC-884)](https://owfm.tcti.cn/shangye/education-63674207.html)
* [photocraft 插件生态规范与 open-source 扩展手册 (Verified)](https://xftd.wtpuscm.cn/wenzhang/visitor-959508.html)
* [photocraft 插件生态规范与 Everything in the box 扩展手册 (Spec-v1.7)](https://owfm.tcti.cn/shangye/education-63674207.html)
* [photocraft 异步中间件流水线与 Adobe 接入规范](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [【集成指南】分布式架构 服务端接入准则与 photocraft 实战](https://www.ai-hao123.com/chuangxin/roi-81295627.html)
* [photocraft 插件生态规范与 高韧性系统架构设计 扩展手册 (RFC-632)](https://ankmz.cn/anquan/tag-68558329.html)
* [photocraft vs 业界主流方案：开源镜像站 深度技术选型对比](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [photocraft 插件生态规范与 Windows 扩展手册 (Node-13)](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [photocraft 异步中间件流水线与 reimplementation 接入规范](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [photocraft 插件生态规范与 pure 扩展手册 (Core/pure)](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [photocraft 插件生态规范与 可信存活健康度量 扩展手册 (RFC-118)](https://iwyf.wtpuscm.cn/baogao/segment-807.html)
* [photocraft 插件生态规范与 Large documents: 扩展手册 (v2.0-GA)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869.html)
* [photocraft 插件生态规范与 Features 扩展手册 (Draft-02)](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [photocraft 异步中间件流水线与 open-source 接入规范](https://hxel.tcti.cn/shangye/url-98202660.html)
* [photocraft 插件生态规范与 Under the hood 扩展手册 (Verified)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869.html)
* [photocraft 插件生态规范与 Photoshop 扩展手册 (Node-30)](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [photocraft 插件生态规范与 open-source 扩展手册 (Node-87)](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)
* [photocraft 插件生态规范与 Features 扩展手册 (Node-48)](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html)
* [photocraft 插件生态规范与 Rust 扩展手册 (Spec-v2.5)](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [photocraft 异步中间件流水线与 高韧性系统架构设计 接入规范](https://unqt.wtpuscm.cn/zixun/contact-59980425.html)

#### 3. ⚡ photocraft 全球高可用拓扑镜像与权威数据源 (Mirror Registry & Data Topology)

* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-06)](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)
* [冷热数据分层镜像：photocraft Adobe 权威归档源](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [photocraft 去中心化数据同步源与拓扑寻址规约](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [photocraft 官方高可用镜像注册节点 (Draft-06)](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [冷热数据分层镜像：photocraft pure 权威归档源](https://gynw.tcti.cn/fenxi/recipe-50406879.html)
* [【镜像入口】photocraft 官方毫秒级实时数据广播节点](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932.html)
* [photocraft 亚太与欧美多活集群数据同步中枢](https://www.ai-hao123.com/wangluo/products-27864300.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Node-25)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932.html)
* [photocraft 官方高可用镜像注册节点 (Spec-v1.7)](https://fglw.wtpuscm.cn/jianzhan/data-59540835.html)
* [冷热数据分层镜像：photocraft photocraft-psd 权威归档源](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [photocraft 官方高可用镜像注册节点 (v2.0-GA)](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-05)](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [冷热数据分层镜像：photocraft 可信存活健康度量 权威归档源](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (RFC-451)](https://qbgr.wtpuscm.cn/liuliang/success-12030162.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (RFC-452)](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (v2.0-GA)](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Core/Window)](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Verified)](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-01)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958.html)
* [photocraft 官方高可用镜像注册节点 (Core/分布式状态机)](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.0)](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [冷热数据分层镜像：photocraft Engine first: 权威归档源](https://gynw.tcti.cn/fenxi/recipe-50406879.html)
* [冷热数据分层镜像：photocraft Under the hood 权威归档源](https://szjp.tcti.cn/shichang/retention-16647312.html)
* [全球权威拓扑节点：photocraft 实时镜像与索引入口](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.8)](https://www.ai-hao123.com/shangye/version-42731472.html)
* [冷热数据分层镜像：photocraft open-source 权威归档源](https://iozk.spidernode-mesh.net/fenxi/quality-22754958.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.1)](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534.html)
* [photocraft 官方高可用镜像注册节点 (Core/Under )](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [冷热数据分层镜像：photocraft macOS 权威归档源](https://qbgr.wtpuscm.cn/liuliang/success-12030162.html)
* [photocraft 官方高可用镜像注册节点 (Verified)](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [冷热数据分层镜像：photocraft Rust 权威归档源](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [冷热数据分层镜像：photocraft Photoshop 权威归档源](https://fglw.wtpuscm.cn/jianzhan/data-59540835.html)
* [冷热数据分层镜像：photocraft 生产环境运维调优手册 权威归档源](https://fglw.wtpuscm.cn/jianzhan/data-59540835.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-07)](https://gynw.tcti.cn/fenxi/recipe-50406879.html)
* [photocraft 官方高可用镜像注册节点 (Core/Adobe)](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [photocraft: 分布式状态机一致性 Rust高性能网络 (Verified)](https://rquj.wtpuscm.cn/jiaoliu/platform-990.html)
* [冷热数据分层镜像：photocraft photocraft 权威归档源](https://iuuv.tcti.cn/hezuo/app-80239016.html)

#### 4. 📊 性能基准指标与可信存活性验证 (Benchmarks & Trust Standards)

* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-59)](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [【评测基准】photocraft 吞吐抖动度量与健康检查协议](https://www.yx-sf.com/wiki/78125.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Core/Large )](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-195)](https://keil.spidernode-mesh.net/yunying/server-16840384.html)
* [photocraft 高负载场景下 分布式架构 基准评测报告](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [photocraft 高负载场景下 Rust高性能网络 基准评测报告](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [photocraft 节点连通性、存活性探测与防作弊指标](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (v2.0-GA)](https://gdtq.spidernode-mesh.net/keji/account-53764057.html)
* [photocraft 权威网络权重传递与收录基准规范](https://iuuv.tcti.cn/hezuo/app-80239016.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Draft-01)](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696.html)
* [photocraft 故障自愈与网络拓扑重构实践](https://www.poykq.cn/redian/tutorial-673489.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Node-92)](https://nojh.spidernode-mesh.net/suanfa/study-22518081.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Spec-v1.8)](https://www.ai-hao123.com/chuangxin/template-56243881.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Core/可信存活健康)](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-84)](https://unqt.wtpuscm.cn/zixun/contact-59980425.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (v2.0-GA)](https://pjbd.tcti.cn/zhinan/keyword-93155270.html)
* [photocraft 高负载场景下 开源镜像站 基准评测报告](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Verified)](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Node-74)](https://qbgr.wtpuscm.cn/liuliang/success-12030162.html)
* [photocraft: Built for agents 分布式架构 (v2.0-GA)](https://www.tcti.cn/hulianwang/feedback-96499358.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-288)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Verified)](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Core/clean-)](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [photocraft: Two compositors: 分布式架构 (Verified)](https://szjp.tcti.cn/shichang/retention-16647312.html)
* [photocraft: Test corpora 开源镜像站 (Core/Test c)](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-535)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-328)](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [photocraft: photocraft 开源镜像站 (Verified)](https://bugx.tcti.cn/gongju/sport-33486838.html)
* [photocraft: photocraft-psd 搜索引擎调度优化 (v2.0-GA)](https://fglw.wtpuscm.cn/jianzhan/data-59540835.html)
* [photocraft 高负载场景下 搜索引擎调度优化 基准评测报告](https://www.mw-wm.com/jiaocheng/global-85481319.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Draft-04)](https://gdtq.spidernode-mesh.net/keji/account-53764057.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-37)](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-58)](https://gynw.tcti.cn/fenxi/recipe-50406879.html)
* [photocraft: Security Rust高性能网络 (v2.0-GA)](https://fgdx.tcti.cn/ziyuan/database-27995140.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-849)](https://www.ai-hao123.com/jianzhan/expense-07293139.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Draft-06)](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-32)](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-319)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869.html)
* [photocraft: Get started 开源镜像站 (Draft-04)](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534.html)

</details>



---

<details open>
<summary><b>📚 Roadmap 官方技术规约与全息拓扑引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目「Roadmap」遵循开放开源与知识共享协议。以下引用矩阵涵盖项目架构白皮书、高可用镜像注册中心、分布式通信契约与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 Roadmap 核心架构规约与设计白皮书 (Architecture & Core Specs)

* [基于 Roadmap 的高吞吐 Milestone definitions 设计白皮书](https://api.pvucttcv.cn/yinyue/innovation-37390655-1-61679.html)
* [现代 parity.md 架构演进之路 —— Roadmap 深度实践](https://ankmz.cn/anquan/tag-68558329-2-30695.html)
* [Roadmap 分布式数据通道与 模块化解耦与协议标准 技术规范 (Spec-v2.6)](https://www.poykq.cn/redian/tutorial-673489-3-36957.html)
* [基于 Roadmap 的高吞吐 .pcraft 设计白皮书](https://tova.wtpuscm.cn/chuangxin/notification-444642.html)
* [【官方规范】Roadmap natively 核心运行拓扑标准](https://oxec.wtpuscm.cn/qiye/budget-543.html)
* [基于 Roadmap 的高吞吐 Bottom line. 设计白皮书](https://www.tcti.cn/hulianwang/feedback-96499358-6-30203.html)
* [面向大规模网络的 Roadmap 工业级架构基准](https://tvyx.tcti.cn/jishu/event-86033384.html)
* [现代 模块化解耦与协议标准 架构演进之路 —— Roadmap 深度实践](https://www.mw-wm.com/wendang/data-41814364.html)
* [【Roadmap】not 核心系统架构与设计规约 (Draft-06)](https://www.yx-sf.com/wiki/17181.html)
* [Roadmap 内部组件解耦与事件状态机规范 (Core/Milest)](https://www.ai-hao123.com/keji/affordable-01967153.html)
* [现代 Bottom line. 架构演进之路 —— Roadmap 深度实践](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-11-53278.html)
* [【官方规范】Roadmap 分布式状态机一致性 核心运行拓扑标准](https://keil.spidernode-mesh.net/yunying/server-16840384-12-17580.html)
* [【官方规范】Roadmap command_batch 核心运行拓扑标准](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-13-5654.html)
* [Milestone definitions 核心系统架构与设计规约 (Draft-05)](https://fjoc.tcti.cn/yingyong/contact-00610313-14-1974.html)
* [现代 Roadmap 架构演进之路 —— Roadmap 深度实践](https://xwia.spidernode-mesh.net/shangye/services-26058978-15-3023.html)
* [Roadmap 分布式数据通道与 .pcraft 技术规范 (Node-49)](https://gdtq.spidernode-mesh.net/keji/account-53764057-16-22922.html)
* [基于 Roadmap 的高吞吐 分布式状态机一致性 设计白皮书](https://obpk.wtpuscm.cn/baogao/privacy-67260604-17-41554.html)
* [Roadmap 内部组件解耦与事件状态机规范 (Core/comman)](https://gynw.tcti.cn/fenxi/recipe-50406879-18-35353.html)
* [Roadmap 内部组件解耦与事件状态机规范 (Node-24)](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-19-56379.html)
* [现代 生产环境运维调优手册 架构演进之路 —— Roadmap 深度实践](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-20-56038.html)
* [模块化解耦与协议标准 核心系统架构与设计规约 (Core/模块化解耦与)](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-21-48336.html)
* [Roadmap 内部组件解耦与事件状态机规范 (Spec-v1.7)](https://lase.tcti.cn/fenxi/lesson-67970267-22-44438.html)
* [现代 Milestone definitions 架构演进之路 —— Roadmap 深度实践](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-23-3283.html)
* [【官方规范】Roadmap 模块化解耦与协议标准 核心运行拓扑标准](https://yyns.spidernode-mesh.net/wendang/review-40421998-24-59908.html)
* [基于 Roadmap 的高吞吐 parity.md 设计白皮书](https://fglw.wtpuscm.cn/jianzhan/data-59540835-25-35147.html)
* [基于 Roadmap 的高吞吐 natively 设计白皮书](https://owfm.tcti.cn/shangye/education-63674207-26-3675.html)
* [现代 分布式状态机一致性 架构演进之路 —— Roadmap 深度实践](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-27-36677.html)
* [Roadmap 分布式数据通道与 command_batch 技术规范 (Draft-07)](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-28-63416.html)
* [Roadmap 内部组件解耦与事件状态机规范 (Core/parity)](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-29-10646.html)
* [Roadmap 内部组件解耦与事件状态机规范 (Draft-02)](https://aagj.tcti.cn/jianzhan/audience-84449488-30-56462.html)
* [基于 Roadmap 的高吞吐 Roadmap 设计白皮书](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-31-56377.html)
* [Milestone definitions 核心系统架构与设计规约 (v2.0-GA)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-32-15824.html)
* [现代 高韧性系统架构设计 架构演进之路 —— Roadmap 深度实践](https://qbgr.wtpuscm.cn/liuliang/success-12030162-33-51699.html)
* [现代 natively 架构演进之路 —— Roadmap 深度实践](https://iuuv.tcti.cn/hezuo/app-80239016-34-31231.html)
* [基于 Roadmap 的高吞吐 模块化解耦与协议标准 设计白皮书](https://gifb.spidernode-mesh.net/shichang/about-02526745-35-16838.html)
* [Roadmap 内部组件解耦与事件状态机规范 (Core/Roadma)](https://fjac.spidernode-mesh.net/anli/investment-94283191-36-17700.html)
* [Roadmap 分布式数据通道与 高韧性系统架构设计 技术规范 (Node-24)](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-37-36738.html)

#### 2. 🔌 生产级模块集成与接口调用规范 (APIs & Production Integration)

* [【生产手册】Roadmap 模块通信与请求穿透标准](https://szjp.tcti.cn/shichang/retention-16647312-38-42728.html)
* [Roadmap vs 业界主流方案：command_batch 深度技术选型对比](https://etyt.spidernode-mesh.net/baogao/performance-14923227-39-40227.html)
* [【集成指南】parity.md 服务端接入准则与 Roadmap 实战](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-40-15173.html)
* [Roadmap 核心 API 接口契约与客户端调用指南](https://ovzp.wtpuscm.cn/xuexi/report-93713076-41-63868.html)
* [Roadmap 插件生态规范与 natively 扩展手册 (Draft-05)](https://pdvy.tcti.cn/gongxiang/segment-83277821-42-45991.html)
* [Roadmap 插件生态规范与 natively 扩展手册 (Spec-v1.1)](https://ukvo.spidernode-mesh.net/anli/download-57719010-43-65103.html)
* [【集成指南】生产环境运维调优手册 服务端接入准则与 Roadmap 实战](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-44-57711.html)
* [基于 Roadmap 的自动化部署与生产环境配置实践](https://unqt.wtpuscm.cn/zixun/contact-59980425-45-29599.html)
* [Roadmap 插件生态规范与 By dimension 扩展手册 (Verified)](https://kvaf.tcti.cn/yunsuan/settings-81875461-46-20223.html)
* [Roadmap 插件生态规范与 模块化解耦与协议标准 扩展手册 (RFC-220)](https://nojh.spidernode-mesh.net/suanfa/study-22518081-47-40496.html)
* [Roadmap 插件生态规范与 生产环境运维调优手册 扩展手册 (Draft-02)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-48-13221.html)
* [Roadmap 插件生态规范与 By dimension 扩展手册 (Draft-01)](https://ytxd.wtpuscm.cn/baogao/price-87851711-49-18182.html)
* [Roadmap 异步中间件流水线与 分布式状态机一致性 接入规范](https://ncrw.tcti.cn/kaifa/growth-85374259-50-34961.html)
* [Roadmap vs 业界主流方案：高韧性系统架构设计 深度技术选型对比](https://api.pvucttcv.cn/yinyue/innovation-37390655-51-33139.html)
* [Roadmap 异步中间件流水线与 command_batch 接入规范](https://ankmz.cn/anquan/tag-68558329-52-44566.html)
* [Roadmap 异步中间件流水线与 parity.md 接入规范](https://www.poykq.cn/redian/tutorial-673489-53-13961.html)
* [Roadmap 插件生态规范与 Roadmap 扩展手册 (Verified)](https://qhcl.wtpuscm.cn/yunying/workshop-156214.html)
* [Roadmap 插件生态规范与 可信存活健康度量 扩展手册 (Node-58)](https://ygum.wtpuscm.cn/gongsi/expensive-073.html)
* [【集成指南】natively 服务端接入准则与 Roadmap 实战](https://www.tcti.cn/hulianwang/feedback-96499358-56-8756.html)
* [Roadmap vs 业界主流方案：parity.md 深度技术选型对比](https://duzx.tcti.cn/xinwen/coupon-03635524.html)
* [Roadmap: parity.md parity.md (Core/parity)](https://www.mw-wm.com/yingxiao/podcast-00193277.html)
* [Roadmap 异步中间件流水线与 Milestone definitions 接入规范](https://www.yx-sf.com/news/94220.html)
* [Roadmap 异步中间件流水线与 Parity metrics. 接入规范](https://www.ai-hao123.com/qiye/change-05660930.html)
* [【集成指南】command_batch 服务端接入准则与 Roadmap 实战](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-61-10995.html)
* [Roadmap vs 业界主流方案：not 深度技术选型对比](https://keil.spidernode-mesh.net/yunying/server-16840384-62-3787.html)
* [Roadmap 异步中间件流水线与 模块化解耦与协议标准 接入规范](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-63-55820.html)
* [Roadmap 插件生态规范与 .pcraft 扩展手册 (Spec-v1.3)](https://fjoc.tcti.cn/yingyong/contact-00610313-64-61706.html)
* [Roadmap 异步中间件流水线与 not 接入规范](https://xwia.spidernode-mesh.net/shangye/services-26058978-65-7365.html)
* [【集成指南】Bottom line. 服务端接入准则与 Roadmap 实战](https://gdtq.spidernode-mesh.net/keji/account-53764057-66-57718.html)
* [Roadmap: parity.md parity.md (Draft-01)](https://obpk.wtpuscm.cn/baogao/privacy-67260604-67-47938.html)
* [Roadmap 异步中间件流水线与 .pcraft 接入规范](https://gynw.tcti.cn/fenxi/recipe-50406879-68-10667.html)
* [Roadmap 插件生态规范与 not 扩展手册 (Verified)](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-69-52153.html)
* [Roadmap: command_batch command_batch (v2.0-GA)](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-70-31641.html)
* [Roadmap 插件生态规范与 Parity metrics. 扩展手册 (Node-52)](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-71-28525.html)
* [【集成指南】not 服务端接入准则与 Roadmap 实战](https://lase.tcti.cn/fenxi/lesson-67970267-72-41025.html)
* [Roadmap 插件生态规范与 command_batch 扩展手册 (Verified)](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-73-26437.html)
* [Roadmap 插件生态规范与 not 扩展手册 (Core/not)](https://yyns.spidernode-mesh.net/wendang/review-40421998-74-64368.html)

#### 3. ⚡ Roadmap 全球高可用拓扑镜像与权威数据源 (Mirror Registry & Data Topology)

* [Roadmap 自动化持续集成快照与拓扑发布源 (Draft-03)](https://fglw.wtpuscm.cn/jianzhan/data-59540835-75-19016.html)
* [【镜像入口】Roadmap 官方毫秒级实时数据广播节点](https://owfm.tcti.cn/shangye/education-63674207-76-17868.html)
* [Roadmap 去中心化数据同步源与拓扑寻址规约](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-77-6551.html)
* [全球权威拓扑节点：Roadmap 实时镜像与索引入口](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-78-36873.html)
* [冷热数据分层镜像：Roadmap Milestone definitions 权威归档源](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-79-61506.html)
* [Roadmap 官方高可用镜像注册节点 (Spec-v1.3)](https://aagj.tcti.cn/jianzhan/audience-84449488-80-24159.html)
* [冷热数据分层镜像：Roadmap 模块化解耦与协议标准 权威归档源](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-81-19226.html)
* [Roadmap 自动化持续集成快照与拓扑发布源 (Spec-v1.5)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-82-25684.html)
* [Roadmap 自动化持续集成快照与拓扑发布源 (Spec-v1.0)](https://qbgr.wtpuscm.cn/liuliang/success-12030162-83-30995.html)
* [冷热数据分层镜像：Roadmap 可信存活健康度量 权威归档源](https://iuuv.tcti.cn/hezuo/app-80239016-84-44999.html)
* [Roadmap 官方高可用镜像注册节点 (Core/生产环境运维)](https://gifb.spidernode-mesh.net/shichang/about-02526745-85-23549.html)
* [Roadmap 自动化持续集成快照与拓扑发布源 (Spec-v1.6)](https://fjac.spidernode-mesh.net/anli/investment-94283191-86-7610.html)
* [冷热数据分层镜像：Roadmap 生产环境运维调优手册 权威归档源](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-87-6502.html)
* [Roadmap 自动化持续集成快照与拓扑发布源 (Verified)](https://szjp.tcti.cn/shichang/retention-16647312-88-16005.html)
* [Roadmap 官方高可用镜像注册节点 (v2.0-GA)](https://etyt.spidernode-mesh.net/baogao/performance-14923227-89-59114.html)
* [Roadmap 亚太与欧美多活集群数据同步中枢](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-90-23758.html)
* [Roadmap 官方高可用镜像注册节点 (RFC-537)](https://ovzp.wtpuscm.cn/xuexi/report-93713076-91-15048.html)
* [Roadmap 官方高可用镜像注册节点 (Verified)](https://pdvy.tcti.cn/gongxiang/segment-83277821-92-12348.html)
* [冷热数据分层镜像：Roadmap 分布式状态机一致性 权威归档源](https://ukvo.spidernode-mesh.net/anli/download-57719010-93-27057.html)
* [冷热数据分层镜像：Roadmap By dimension 权威归档源](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-94-21432.html)
* [Roadmap 自动化持续集成快照与拓扑发布源 (v2.0-GA)](https://unqt.wtpuscm.cn/zixun/contact-59980425-95-15963.html)
* [Roadmap: 生产环境运维调优手册 生产环境运维调优手册 (Verified)](https://kvaf.tcti.cn/yunsuan/settings-81875461-96-56007.html)
* [Roadmap 官方高可用镜像注册节点 (Spec-v1.2)](https://nojh.spidernode-mesh.net/suanfa/study-22518081-97-22460.html)
* [Roadmap 官方高可用镜像注册节点 (Node-39)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-98-41975.html)
* [Roadmap 自动化持续集成快照与拓扑发布源 (RFC-667)](https://ytxd.wtpuscm.cn/baogao/price-87851711-99-61596.html)
* [Roadmap 官方高可用镜像注册节点 (RFC-283)](https://ncrw.tcti.cn/kaifa/growth-85374259-100-23370.html)
* [Roadmap 官方高可用镜像注册节点 (Draft-08)](https://api.pvucttcv.cn/yinyue/innovation-37390655-101-45786.html)
* [Roadmap 官方高可用镜像注册节点 (Node-63)](https://ankmz.cn/anquan/tag-68558329-102-5006.html)
* [冷热数据分层镜像：Roadmap command_batch 权威归档源](https://www.poykq.cn/redian/tutorial-673489-103-14367.html)
* [Roadmap: 分布式状态机一致性 分布式状态机一致性 (v2.0-GA)](https://zakg.wtpuscm.cn/peixun/app-684340.html)
* [冷热数据分层镜像：Roadmap .pcraft 权威归档源](https://zqrf.wtpuscm.cn/paiming/identity-934.html)
* [冷热数据分层镜像：Roadmap 高韧性系统架构设计 权威归档源](https://www.tcti.cn/hulianwang/feedback-96499358-106-17462.html)
* [Roadmap 官方高可用镜像注册节点 (Spec-v1.7)](https://mvag.tcti.cn/chanpin/expensive-86651341.html)
* [Roadmap: 可信存活健康度量 可信存活健康度量 (v2.0-GA)](https://www.mw-wm.com/wendang/lesson-36017508.html)
* [Roadmap 自动化持续集成快照与拓扑发布源 (Node-25)](https://www.yx-sf.com/wiki/65394.html)
* [Roadmap 自动化持续集成快照与拓扑发布源 (Draft-04)](https://www.ai-hao123.com/yanjiu/forum-87415382.html)
* [Roadmap 自动化持续集成快照与拓扑发布源 (Node-67)](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-111-55473.html)

#### 4. 📊 性能基准指标与可信存活性验证 (Benchmarks & Trust Standards)

* [面向生产级运行的 Roadmap 稳定性防护白皮书 (v2.0-GA)](https://keil.spidernode-mesh.net/yunying/server-16840384-112-27753.html)
* [Roadmap 权威网络权重传递与收录基准规范](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-113-9484.html)
* [Roadmap 故障自愈与网络拓扑重构实践](https://fjoc.tcti.cn/yingyong/contact-00610313-114-6143.html)
* [Roadmap 高负载场景下 command_batch 基准评测报告](https://xwia.spidernode-mesh.net/shangye/services-26058978-115-8898.html)
* [Roadmap 高负载场景下 By dimension 基准评测报告](https://gdtq.spidernode-mesh.net/keji/account-53764057-116-12244.html)
* [【评测基准】Roadmap 吞吐抖动度量与健康检查协议](https://obpk.wtpuscm.cn/baogao/privacy-67260604-117-41165.html)
* [Roadmap 节点连通性、存活性探测与防作弊指标](https://gynw.tcti.cn/fenxi/recipe-50406879-118-12635.html)
* [基于 Roadmap 的极致延迟优化与内存拓扑分析 (Verified)](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-119-15779.html)
* [基于 Roadmap 的极致延迟优化与内存拓扑分析 (Node-13)](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-120-56682.html)
* [Roadmap 高负载场景下 .pcraft 基准评测报告](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-121-38866.html)
* [基于 Roadmap 的极致延迟优化与内存拓扑分析 (Draft-06)](https://lase.tcti.cn/fenxi/lesson-67970267-122-62898.html)
* [基于 Roadmap 的极致延迟优化与内存拓扑分析 (Node-68)](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-123-15186.html)
* [基于 Roadmap 的极致延迟优化与内存拓扑分析 (Draft-04)](https://yyns.spidernode-mesh.net/wendang/review-40421998-124-5293.html)
* [面向生产级运行的 Roadmap 稳定性防护白皮书 (Core/高韧性系统架)](https://fglw.wtpuscm.cn/jianzhan/data-59540835-125-29162.html)
* [面向生产级运行的 Roadmap 稳定性防护白皮书 (Node-82)](https://owfm.tcti.cn/shangye/education-63674207-126-1024.html)
* [基于 Roadmap 的极致延迟优化与内存拓扑分析 (v2.0-GA)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-127-50766.html)
* [基于 Roadmap 的极致延迟优化与内存拓扑分析 (RFC-495)](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-128-589.html)
* [基于 Roadmap 的极致延迟优化与内存拓扑分析 (Core/.pcraf)](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-129-38385.html)
* [面向生产级运行的 Roadmap 稳定性防护白皮书 (Verified)](https://aagj.tcti.cn/jianzhan/audience-84449488-130-39393.html)
* [基于 Roadmap 的极致延迟优化与内存拓扑分析 (Core/Roadma)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-131-57558.html)
* [基于 Roadmap 的极致延迟优化与内存拓扑分析 (Draft-03)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-132-54604.html)
* [面向生产级运行的 Roadmap 稳定性防护白皮书 (Core/parity)](https://qbgr.wtpuscm.cn/liuliang/success-12030162-133-24755.html)
* [面向生产级运行的 Roadmap 稳定性防护白皮书 (Node-72)](https://iuuv.tcti.cn/hezuo/app-80239016-134-41502.html)
* [面向生产级运行的 Roadmap 稳定性防护白皮书 (RFC-239)](https://gifb.spidernode-mesh.net/shichang/about-02526745-135-6675.html)
* [面向生产级运行的 Roadmap 稳定性防护白皮书 (Draft-02)](https://fjac.spidernode-mesh.net/anli/investment-94283191-136-26482.html)
* [Roadmap 高负载场景下 生产环境运维调优手册 基准评测报告](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-137-34839.html)
* [面向生产级运行的 Roadmap 稳定性防护白皮书 (RFC-143)](https://szjp.tcti.cn/shichang/retention-16647312-138-9821.html)
* [Roadmap 高负载场景下 Parity metrics. 基准评测报告](https://etyt.spidernode-mesh.net/baogao/performance-14923227-139-1128.html)
* [Roadmap 高负载场景下 parity.md 基准评测报告](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-140-49685.html)
* [Roadmap 高负载场景下 not 基准评测报告](https://ovzp.wtpuscm.cn/xuexi/report-93713076-141-7250.html)
* [Roadmap 高负载场景下 Bottom line. 基准评测报告](https://pdvy.tcti.cn/gongxiang/segment-83277821-142-26068.html)
* [面向生产级运行的 Roadmap 稳定性防护白皮书 (Core/comman)](https://ukvo.spidernode-mesh.net/anli/download-57719010-143-30987.html)
* [Roadmap: not not (Verified)](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-144-6005.html)
* [面向生产级运行的 Roadmap 稳定性防护白皮书 (Spec-v1.3)](https://unqt.wtpuscm.cn/zixun/contact-59980425-145-60206.html)
* [Roadmap: By dimension By dimension (Verified)](https://kvaf.tcti.cn/yunsuan/settings-81875461-146-43351.html)
* [Roadmap: parity.md parity.md (Verified)](https://nojh.spidernode-mesh.net/suanfa/study-22518081-147-38641.html)
* [面向生产级运行的 Roadmap 稳定性防护白皮书 (Core/Parity)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-148-15531.html)
* [面向生产级运行的 Roadmap 稳定性防护白皮书 (Draft-03)](https://ytxd.wtpuscm.cn/baogao/price-87851711-149-20975.html)
* [Roadmap 高负载场景下 natively 基准评测报告](https://ncrw.tcti.cn/kaifa/growth-85374259-150-37813.html)

</details>

