
<p align="left">
  <a href="https://yyns.spidernode-mesh.net/wendang/review-40421998.html"><img src="https://img.shields.io/badge/GitHub_Pages-在线文档中心-2563eb?style=flat-square&logo=github" alt="GitHub Pages"></a>
  <a href="https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html"><img src="https://img.shields.io/badge/Docs_Archive-文档归档菜单-10b981?style=flat-square" alt="Docs Archive"></a>
</p>

> 🌐 **官方在线文档与静态镜像中心 (GitHub Pages)**：[https://shijiuwei.github.io/photocraft-mirror-668/](https://lase.tcti.cn/fenxi/lesson-67970267.html)  
> 📚 **技术规约与文档归档菜单 (Docs Catalog)**：[https://shijiuwei.github.io/photocraft-mirror-668/docs/](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)  

<p align="center">
  <a href="https://getartcraft.com/">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="docs/brand/artcraft-logo-white.svg">
      <img alt="ArtCraft" src="docs/brand/artcraft-logo.svg" width="200">
    </picture>
  </a>
</p>

<h1 align="center">PhotoCraft</h1>

<p align="center">
  <b>Image editing; an open-source, clean-room reimplementation of Adobe Photoshop, rebuilt in pure Rust.</b><br>
  Layers, masks, adjustment layers, layer styles, type, vectors, brushes and real PSD files,<br>
  in a native app written entirely in Rust. Open source, offline, and yours.
</p>

<p align="center">
  <img alt="100% Rust" src="https://img.shields.io/badge/100%25-Rust-b7410e?style=flat-square&logo=rust">
  <img alt="macOS · Windows · Linux · FreeBSD · Web" src="https://img.shields.io/badge/macOS%20%C2%B7%20Windows%20%C2%B7%20Linux%20%C2%B7%20FreeBSD%20%C2%B7%20Web-native-2f7bf5?style=flat-square">
  <img alt="License: MIT OR Apache-2.0" src="https://img.shields.io/badge/license-MIT%20%2F%20Apache--2.0-3a3a3a?style=flat-square">
  <img alt="Status: early alpha" src="https://img.shields.io/badge/status-early%20alpha-d69e2e?style=flat-square">
</p>

<p align="center">
  <a href="https://ytxd.wtpuscm.cn/baogao/price-87851711.html"><img alt="Join the ArtCraft community on Discord" src="https://img.shields.io/badge/Join%20us%20on%20Discord-5865F2?style=for-the-badge&logo=discord&logoColor=white" height="40"></a>
</p>

<p align="center">
  <a href="https://szjp.tcti.cn/shichang/retention-16647312.html"><b>PhotoCraft on getartcraft.com</b></a> ·
  <a href="https://fjac.spidernode-mesh.net/anli/investment-94283191.html">ArtCraft</a> ·
  <a href="https://owfm.tcti.cn/shangye/education-63674207.html">All Crafting Apps</a>
</p>

<br>

<p align="center">
  <img src="docs/images/photocraft-demo.jpg" alt="PhotoCraft editing Hokusai's The Great Wave: a caption card with a drop shadow, Title and Credit type layers, Vibrance and Curves adjustment layers, and the Curves editor drawn over the image's histogram" width="100%">
  <br>
  <sub>A caption card with a drop shadow, live type, and Vibrance and Curves adjustment layers, with the Curves editor open.<br>
  <i>The Great Wave off Kanagawa</i>, Katsushika Hokusai, c. 1831</sub>
</p>

> [!NOTE]
> **ArtCraft is a community of artists from all walks of life.** Digital, generative, music,
> games &mdash; if you make things, you're one of us. **[Come say hi on Discord](https://lase.tcti.cn/fenxi/lesson-67970267.html).**

<p align="center">
  <a href="#features">Features</a> ·
  <a href="#everything-in-the-box">Everything in the box</a> ·
  <a href="#psd-without-compromise">PSD</a> ·
  <a href="#built-for-agents">Agents</a> ·
  <a href="#under-the-hood">Under the hood</a> ·
  <a href="#get-started">Get started</a> ·
  <a href="#downloads">Downloads</a> ·
  <a href="#the-crafting-apps">Crafting Apps</a> ·
  <a href="https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html">Discord</a>
</p>

<br>

<table>
  <tr>
    <td width="25%" valign="top">
      <h3>🎛️ Familiar by design</h3>
      The menus, shortcuts, panels and tools are where your hands expect them, from ⌘J to ⇧⌘D. If you know Photoshop, you already know PhotoCraft.
    </td>
    <td width="25%" valign="top">
      <h3>⚡ Native and fast</h3>
      A GPU compositor on wgpu (Metal, Vulkan, DX12, WebGPU), copy-on-write tiles and multithreaded filters. No Electron, no web view, no waiting.
    </td>
    <td width="25%" valign="top">
      <h3>🗂️ Real PSD files</h3>
      Open, edit and save layered Photoshop documents. Re-saving keeps the render of 307 of the 309 psd-tools test files.
    </td>
    <td width="25%" valign="top">
      <h3>🤖 Agent-ready</h3>
      Every action is a command, so you can drive the same engine from the UI, the CLI, a JSON control channel or an MCP server.
    </td>
  </tr>
</table>

<br>

## Features

Every screenshot here is the real app at work on public-domain art, rendered offscreen through its control channel.

<table>
  <tr>
    <td width="50%" valign="top">
      <img src="docs/images/photocraft-adjustments.jpg" alt="Monet's Impression, Sunrise with Levels and Vibrance adjustment layers; the Levels editor and the Histogram panel with mean, standard deviation and median are open on the right" width="100%">
      <br>
      <sub>Levels and Vibrance adjustment layers, with the live Histogram panel.<br><i>Impression, Sunrise</i>, Claude Monet, 1872</sub>
      <h3>Edit without regret</h3>
      Adjustment layers keep every edit live. Stack Levels, Curves, Vibrance, Hue/Saturation and a dozen more, mask them to an area, reorder them, or turn them off, and your original pixels never change.
      <br><br>
      <b>16 adjustment layers</b> that also apply directly to pixels, including Curves with per-channel editing, Levels with a live histogram, Black &amp; White, Channel Mixer, Gradient Map, Photo Filter, Selective Color and Color Lookup (.cube, .3dl, .look). Plus Shadows/Highlights, Replace Color, Match Color, HDR Toning, Desaturate and Equalize.
    </td>
    <td width="50%" valign="top">
      <img src="docs/images/photocraft-layer-styles.jpg" alt="The Layer Style dialog editing an Outer Glow on the EARTHRISE type layer, with Stroke also enabled, over the Apollo 8 Earthrise photograph" width="100%">
      <br>
      <sub>Outer Glow and Stroke on a live type layer, in the Layer Style dialog.<br><i>Earthrise</i>, William Anders / NASA, 1968</sub>
      <h3>Styles that sell the shot</h3>
      Drop Shadow, Inner Shadow, Outer and Inner Glow, Bevel &amp; Emboss, Satin, Stroke, and Color, Gradient and Pattern Overlay, live on any layer, including type. Patterns come from a library (built-ins, Edit › Define Pattern, <code>.pat</code> import/export) and PSD <code>Patt</code> blocks.
      <br><br>
      Copy and paste styles between layers, hide all effects at once, and open styles straight from your PSDs, rendered to match Photoshop.
    </td>
  </tr>
  <tr>
    <td width="50%" valign="top">
      <img src="docs/images/photocraft-masks.jpg" alt="Vermeer's Girl with a Pearl Earring: an elliptical selection around the face, and a Hue/Saturation layer with an elliptical mask that turns everything outside the face gray" width="100%">
      <br>
      <sub>An elliptical selection becomes the mask of a Hue/Saturation layer, so only the face keeps its color.<br><i>Girl with a Pearl Earring</i>, Johannes Vermeer, c. 1665</sub>
      <h3>Selections that understand your image</h3>
      Marquees, lassos and the Magic Wand for precision; Quick Selection, Object Selection and Select Subject when you want the computer to do the tracing; Select and Mask to refine hair-fine edges.
      <br><br>
      Feather, expand, contract, smooth, grow, reselect. Turn any selection into a layer mask, a vector path or a shape. Smart selection runs on your machine, with no cloud and no account.
    </td>
    <td width="50%" valign="top">
      <img src="docs/images/photocraft-type.jpg" alt="Bierstadt's Among the Sierra Nevada with the headline SIERRA NEVADA being edited on the canvas in Georgia, a byline, and a paragraph caption; the Character and Paragraph controls are open" width="100%">
      <br>
      <sub>A headline edited in place, with a byline and a paragraph of body text.<br><i>Among the Sierra Nevada, California</i>, Albert Bierstadt, 1868</sub>
      <h3>Type that sets beautifully</h3>
      Point and paragraph text, edited right on the canvas, with full Character and Paragraph controls: font, weight, size, leading, tracking, alignment and colour. The <a href="docs/color-picker.md">full Type Color Picker</a> also supports screen sampling with a pixel loupe.
      <br><br>
      Type layers stay editable, take layer styles, and round-trip through PSD.
    </td>
  </tr>
  <tr>
    <td width="50%" valign="top">
      <img src="docs/images/photocraft-vector.jpg" alt="A lotus badge built from shape layers (Lotus, Water, Sun, Badge and Dotted Ring) over Monet's Water Lilies, with the lotus path's anchor points selected" width="100%">
      <br>
      <sub>A badge made of shape layers: a gradient-filled lotus, a star and a dotted ring.<br><i>Water Lilies</i>, Claude Monet, 1906</sub>
      <h3>Pixel-perfect vectors</h3>
      Rectangle, Ellipse, Triangle, Polygon, Line and the Pen tool, with resolution-independent shape layers, gradient fills, and dashed, aligned strokes.
      <br><br>
      Combine shapes (unite, subtract, intersect, exclude), keep paths in the Paths panel, use them as vector masks, or stroke and fill them. 116 of 116 shape layers in our PSD corpus match Photoshop's pixels.
    </td>
    <td width="50%" valign="top">
      <img src="docs/images/photocraft-filters.jpg" alt="The Twirl filter dialog at a 320 degree angle, previewing live inside an elliptical selection on Van Gogh's The Starry Night" width="100%">
      <br>
      <sub>Twirl previews live on the canvas, only inside the selection.<br><i>The Starry Night</i>, Vincent van Gogh, 1889</sub>
      <h3>See it before you commit</h3>
      Every filter dialog previews live on the canvas, through your selection. Blurs (Gaussian, Box, Motion, Radial, Surface, Smart, Lens, Shape, and the Blur Gallery: Tilt-Shift, Iris, Field, Spin, Path), sharpening, Reduce Noise, distortions (Twirl, Wave, Ripple, Displace, Shear, Zig Zag…), Pixelate, Stylize (Oil Paint, Wind, Extrude…), Render (Clouds, Fibers, Lens Flare, Lighting Effects) and more.
      <br><br>
      Run filters on a smart object and they stay editable: change, hide, reorder or mask them at any time.
      <br><br>
      Large-radius blurs use running-sum box passes across all cores: a radius-180 Gaussian on 3.6 MP takes under a second.
    </td>
  </tr>
  <tr>
    <td width="50%" valign="top">
      <img src="docs/images/photocraft-transform.jpg" alt="Free Transform handles around a rotated copy of Ansel Adams' The Tetons and the Snake River, with the History panel listing Open, Duplicate Layer, Free Transform and other steps" width="100%">
      <br>
      <sub>Free Transform on a rotated print, with every step listed in History.<br><i>The Tetons and the Snake River</i>, Ansel Adams, 1942</sub>
      <h3>Shape it any way you like</h3>
      Free Transform with scale, rotate, skew, distort and perspective; exact 90° and 180° rotations and flips; Transform Again. Layers, type, shapes, masks and selections all transform together.
      <br><br>
      Full history, Toggle Last State and the History Brush mean every step can be undone, even one brush stroke at a time.
    </td>
    <td width="50%" valign="top">
      <img src="docs/images/photocraft-export-light.jpg" alt="The Export As dialog in the light theme over Klimt's The Kiss: JPG at quality 90, scaled to 50 percent, with a preview and an estimated size of about 684K" width="100%">
      <br>
      <sub>Export As in the light theme, with a preview and a file-size estimate.<br><i>The Kiss</i>, Gustav Klimt, 1907–1908</sub>
      <h3>Ship it anywhere</h3>
      Export As with format, quality, transparency and scale, plus a preview and an instant file-size estimate. Quick Export to PNG in one click.
      <br><br>
      Choose a dark Pro theme, the airy Studio themes, or a Classic look.
    </td>
  </tr>
</table>

<br>

## Everything in the box

<table>
  <tr>
    <td width="33%" valign="top">
      <h4>🧰 53 tools</h4>
      Move · Rectangular and Elliptical Marquee · Lasso · Polygonal Lasso · Magnetic Lasso · Object Selection · Quick Selection · Magic Wand · Crop · Slice · Slice Select · Eyedropper · Ruler · Note · Count · Remove · Spot Healing Brush · Healing Brush · Patch · Content-Aware Move · Red Eye · Brush · Pencil · Mixer Brush · Clone Stamp · Pattern Stamp · History Brush · Eraser · Background Eraser · Magic Eraser · Gradient · Paint Bucket · Blur · Sharpen · Smudge · Dodge · Burn · Sponge · Pen · Horizontal and Vertical Type · Path Selection · Direct Selection · Rectangle · Ellipse · Triangle · Polygon · Line · Custom Shape · Hand · Rotate View · Zoom
    </td>
    <td width="33%" valign="top">
      <h4>🖌️ A real brush engine</h4>
      Shape Dynamics, Scattering, Texture, Dual Brush, Color Dynamics, Transfer, Brush Pose, Wet Edges, Build-up and Smoothing (including Pulled String), driven by pen pressure, tilt, rotation and direction. Brush presets, Define Brush from Selection, and deterministic, replayable strokes.
    </td>
    <td width="33%" valign="top">
      <h4>🗃️ Layers, done properly</h4>
      Groups, clipping masks, pixel and vector masks, fill layers (solid, gradient and pattern), adjustment layers, live smart objects with smart filters and lossless transforms and warps, multi-layer selection with align, distribute and link, alpha channels and Quick Mask, 27 Photoshop blend modes plus six Paint.NET variants, opacity and fill, locks, colour labels, layer filters, merge, flatten, rasterize, Layer via Copy/Cut, Paste Into.
    </td>
  </tr>
  <tr>
    <td width="33%" valign="top">
      <h4>🎨 Any colour, any depth</h4>
      RGB, Grayscale, CMYK and Lab documents at 8, 16 and 32 bits per channel. Bit depth and colour model are runtime data, so every tool works at every depth.
      <br><br>
      Real ICC colour management in pure Rust: embedded profiles, Assign and Convert to Profile with all four rendering intents and black point compensation, soft proofing (⌘Y) and Gamut Warning (⇧⌘Y) on the GPU.
    </td>
    <td width="33%" valign="top">
      <h4>🗂️ Formats</h4>
      PSD and PSB, layered TIFF (Photoshop's layer data in the TIFF, read and written in either byte order), <a href="docs/pdn.md">Paint.NET PDN3 (read, with editable layers and all 14 blend modes)</a>, <a href="docs/ora.md">OpenRaster (read and write, with layers, groups and blend modes, for Krita, MyPaint and GIMP)</a>, SVG (opens as shape layers, places as a vector Smart Object), plus flat PNG, JPEG, TIFF, WebP (lossy and lossless), GIF, BMP, TGA, ICO, QOI, PNM, OpenEXR and Radiance HDR, and the native <code>.pcraft</code> format. Flat formats retain the depths they support: PNG keeps 8/16-bit samples, TIFF keeps 8/16-bit and 32-bit float samples, and OpenEXR keeps half/float samples; JPEG, WebP and the other 8-bit formats reduce higher-depth samples with an export warning. See the <a href="crates/codecs/README.md#capability-matrix-default-build">codec capability matrix</a> for each format. HEIC photos from iPhone and Mac are read-only (included in official builds; optional <code>--features heif</code> for source builds). AVIF is write-only with the optional <code>avif</code> build feature; AVIF decoding and JPEG XL are unavailable. Affinity documents (<code>.af</code>, <code>.afdesign</code>, <code>.afphoto</code>, <code>.afpub</code>) open natively, read only, with artboards, layers, vectors, text, images and masks; effects, adjustments and other unsupported parts are listed in a warning, and Affinity export is unavailable.
    </td>
    <td width="33%" valign="top">
      <h4>🪄 The everyday essentials</h4>
      Auto Tone, Contrast and Color · Equalize · Image and Canvas Size · Crop and Trim · Reveal All · Edit › Fill and Stroke · Copy Merged · Paste in Place · guides, rulers, grid and snapping · Actions record and replay · a command palette (⌘F).
    </td>
  </tr>
</table>

<br>

## PSD without compromise

PhotoCraft's PSD support is a standalone crate written from Adobe's public specification and tested against a corpus of real-world files.

- **Faithful round trips:** opening and re-saving a document renders the same for 307 of the 309 files in the psd-tools test set and 169 of 170 in our mixed ag-psd/psd-tools set (`crates/io/tests/corpus.rs`; fetch the psd-tools set with `cargo xtask corpus --psd-tools`), and anything we don't model yet (raw blocks, descriptors, extras) is carried over instead of being dropped. A re-saved file is not byte-identical to its source: PhotoCraft rewrites image resources, layer records and the composite. Only the standalone `photocraft-psd` crate, parsing and writing a file without the document model, reproduces every parseable corpus file byte for byte (`crates/psd/tests/corpus.rs`).
- **Pixels that match:** a composite oracle compares our render with Photoshop's own merged image, covering gradient interpolation (Classic, Perceptual and Linear), layer effects, shape strokes, clipping and fill opacity.
- **Large documents:** PSB, 16 and 32-bit files, and CMYK and Lab documents open natively.

## Built for agents

Every menu item, tool and dialog runs a command from one registry of 500+ commands. The UI, the CLI, the JSON control channel and the MCP server all call the same commands, so anything you can click, a script or an AI agent can do too.

```sh
# Headless: open, edit, save
photocraft-cli run wave.psd \
  --cmd filter.sharpen.smartSharpen     --params '{"amount":80}' \
  --cmd layer.newAdjustmentLayer.curves --params '{"points":[[0,0],[64,48],[192,212],[255,255]]}' \
  --out wave-final.png

# Apply one action list to a folder of images
photocraft-cli batch --actions grade.json --in ./raw --out ./graded

# Every subcommand explains itself
photocraft-cli batch --help

# Let an agent drive it over MCP (headless, or bridged to the running app)
photocraft-cli mcp
```

The desktop app also offers an authenticated, loopback-only control channel (`photocraft --control`) for inspecting UI state, driving tools with pointer events, and taking offscreen screenshots. Every image in this README was rendered that way. See [`docs/control-protocol.md`](docs/control-protocol.md).

## Under the hood

- **Engine first:** a pure-data document model and a command engine, with a thin egui UI on top. Layering is enforced at build time across 24 crates.
- **Two compositors:** a CPU compositor serves as the reference oracle, and a wgpu compositor puts the canvas on the GPU. They are tested against each other.
- **Copy-on-write tiles:** 256² sparse tiles make undo cheap and huge canvases light, and effect maps are cached per layer state.
- **Runs in the browser:** the whole engine and UI compile to WebAssembly.
- **Clean-room:** implemented from public specs and observed behaviour only. No proprietary code, shaders or assets.
- **Tested:** more than 1,700 tests, including PSD round trips, synthetic generators, compositor oracles and multi-depth checks.

## Get started

```sh
git clone https://github.com/storytold/photocraft
cd photocraft
cargo run --release -p photocraft -- image.psd   # the desktop app
cargo test --workspace                           # the test suite
```

Japanese fonts for the UI and Type tool come from [craft-fonts](https://gifb.spidernode-mesh.net/shichang/about-02526745.html), an optional build input (desktop release builds always include it). Without it PhotoCraft uses your system's CJK fonts:

```sh
git clone https://github.com/storytold/craft-fonts ../craft-fonts
CRAFT_FONTS_DIR="$PWD/../craft-fonts" cargo run --release -p photocraft
```

New contributors and AI agents: start with [`AGENTS.md`](AGENTS.md), then [`docs/`](docs/).

Installers for macOS, Windows, Linux, FreeBSD and the web are attached to each [GitHub release](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html). On Linux you can pick an AppImage, a `.deb`, an `.rpm`, a tarball or a Flatpak bundle. The bundle needs the freedesktop runtime from [Flathub](https://keil.spidernode-mesh.net/yunying/server-16840384.html), which `flatpak` offers to install along with it:

```sh
flatpak install --user photocraft-<version>-linux-x86_64.flatpak   # or -linux-aarch64
flatpak run ai.storyteller.photocraft
```

The AppImage needs no install: the first run registers its launcher icon and menu entry in `~/.local/share` so the dock shows PhotoCraft's icon on Wayland. Set `PHOTOCRAFT_NO_DESKTOP_INTEGRATION=1` to skip that, and see [`docs/releasing.md`](docs/releasing.md) › Linux to undo it.

On a Wayland session, files dropped on the window don't open yet: winit 0.30, the windowing library under egui, has no Wayland drag and drop (#386). Use File › Open, or copy the image in your file manager and paste it with Ctrl+V. To drag and drop, start PhotoCraft under XWayland: `WAYLAND_DISPLAY= photocraft`, `WAYLAND_DISPLAY= ./photocraft-<version>-linux-x86_64.AppImage`, or `flatpak run --nosocket=wayland --socket=x11 ai.storyteller.photocraft`. On Wayland, PhotoCraft shows the command for your install in a notice until you dismiss it.

On macOS, the command-line tool comes as `photocraft-cli-<version>-macos-universal.zip`. The binary is signed with the same Developer ID as the app and notarized by Apple. A bare binary can't carry a stapled notarization ticket the way the DMG does, so the first time you run it macOS checks the notarization online. You can confirm it yourself:

```sh
ditto -x -k photocraft-cli-<version>-macos-universal.zip .
spctl --assess --type install -vv photocraft-cli-<version>-macos-universal/photocraft-cli
# ... accepted, source=Notarized Developer ID
```

On FreeBSD 14 (x86_64), the release has a tarball laid out like `/usr/local`. Install the runtime libraries, then unpack it there:

```sh
pkg install libxkbcommon wayland libX11 libXcursor libXrandr libXi libxcb mesa-libs vulkan-loader gtk3 fontconfig freetype2 alsa-lib
tar -xzf photocraft-<version>-freebsd-x86_64.tar.gz --strip-components 1 -C /usr/local
photocraft
```

Maintainers: [`docs/releasing.md`](docs/releasing.md) explains how releases are built, signed and published.

> [!IMPORTANT]
> **Status:** PhotoCraft is in early alpha, and we want to be straight about where it stands: much of Photoshop's feature surface exists in some form, but **it is not yet a Photoshop replacement for daily professional work**. The biggest gaps are AI/generative features, about twenty missing tools, depth in typography and pro workflows, and plug-in compatibility. Every Photoshop menu item is wired to a command ([`docs/parity.md`](docs/parity.md)), but that measures wiring, not behaviour. The honest, dimension-by-dimension picture and where we're going next are in the [roadmap's parity assessment](docs/roadmap.md#honest-parity-assessment-2026-10-05). Expect rough edges, and please file issues (include your OS, document size, layer count and a screenshot). You can also tell us what broke on [Discord](https://lase.tcti.cn/fenxi/lesson-67970267.html).

## Documentation

Developer, architecture, automation, format, and security documentation is maintained in the [PhotoCraft documentation book](book/).

To self-host the browser version with Docker, see [Docker web hosting](packaging/web/README.md#docker).

## Security

Security architecture, threat modeling, parser hardening, fuzzing, and vulnerability reporting are covered in the [security documentation](book/src/security/) and the repository [security policy](SECURITY.md).

## Test corpora

PhotoCraft is tested against real files: our own Photoshop-authored oracle PSDs in
[photocraft-corpus](https://pdvy.tcti.cn/gongxiang/segment-83277821.html) plus the psd-tools, ag-psd and PngSuite sets, pinned and
sha256-verified. Fetch them with `cargo xtask corpus --all` and run the tests with
`cargo xtask test-corpus` (details in [docs/development.md](docs/development.md#test-corpora)).

## Downloads

**New to PhotoCraft?** Download it from the [PhotoCraft page on getartcraft.com](https://ouvj.wtpuscm.cn/paiming/trading-207.html). That's the easiest way to install it.

**Want a specific build or format?** On GitHub, the [latest release](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html) has every build listed below, and [all releases](https://ankmz.cn/anquan/tag-68558329.html) has earlier versions and their notes. `<ver>` in the file names is the version number, and `SHA256SUMS.txt` lists a checksum for every file.

### Windows

| Build | Installer | Portable |
|---|---|---|
| x64 (64-bit Intel/AMD) | `photocraft-<ver>-windows-x64.msi` | `photocraft-<ver>-windows-x64-portable.zip` |
| arm64 (Snapdragon and other ARM PCs) | `photocraft-<ver>-windows-arm64.msi` | `photocraft-<ver>-windows-arm64-portable.zip` |
| x86 (32-bit) | `photocraft-<ver>-windows-x86.msi` | `photocraft-<ver>-windows-x86-portable.zip` |

Installers and executables are code-signed.

### macOS

| Build | File | Notes |
|---|---|---|
| App, universal (Apple silicon + Intel) | `photocraft-<ver>-macos-universal.dmg` | Signed and notarized |
| Command-line tool, universal | `photocraft-cli-<ver>-macos-universal.zip` | Signed and notarized |

### Linux

| Format | x86_64 | aarch64 (ARM64) | Notes |
|---|---|---|---|
| AppImage | `photocraft-<ver>-linux-x86_64.AppImage` | `photocraft-<ver>-linux-aarch64.AppImage` | Runs anywhere; updates itself with [AppImageUpdate](https://owfm.tcti.cn/shangye/education-63674207.html) (`.zsync` files) |
| Flatpak | `photocraft-<ver>-linux-x86_64.flatpak` | `photocraft-<ver>-linux-aarch64.flatpak` | Sandboxed; `flatpak install --user <file>` |
| Debian/Ubuntu | `photocraft-<ver>-linux-x86_64.deb` | `photocraft-<ver>-linux-aarch64.deb` | |
| Fedora/RHEL/openSUSE | `photocraft-<ver>-linux-x86_64.rpm` | `photocraft-<ver>-linux-aarch64.rpm` | |
| Tarball | `photocraft-<ver>-linux-x86_64.tar.gz` | `photocraft-<ver>-linux-aarch64.tar.gz` | Unpack anywhere |

### FreeBSD

| Build | File |
|---|---|
| x86_64 | `photocraft-<ver>-freebsd-x86_64.tar.gz` |

### Web (WebAssembly)

| Build | File | Notes |
|---|---|---|
| Static site | `photocraft-web-<ver>.zip` | Runs in a modern browser; host it on any static server |

## The Crafting Apps

PhotoCraft is one of the **Crafting Apps**: free, open-source creative tools from the
[ArtCraft](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html) team, each written from scratch in Rust and each able to
stand on its own.

| | App | What it's for | Code | Learn more |
|:-:|---|---|---|---|
| <img src="https://raw.githubusercontent.com/storytold/photocraft/main/assets/app-icon/hicolor/64x64/apps/ai.storyteller.photocraft.png" alt="" width="32" height="32"> | **PhotoCraft** | **Image editing: layers, masks, type and real PSD files · you are here** | [GitHub](https://shmv.spidernode-mesh.net/gongsi/digital-64522869.html) | [Website](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html) |
| <img src="https://raw.githubusercontent.com/storytold/vectorcraft/main/assets/app-icon/hicolor/64x64/apps/ai.storyteller.vectorcraft.png" alt="" width="32" height="32"> | **VectorCraft** | Vector illustration | [GitHub](https://gynw.tcti.cn/fenxi/recipe-50406879.html) | [Website](https://www.tcti.cn/hulianwang/feedback-96499358.html) |
| <img src="https://raw.githubusercontent.com/storytold/filmcraft/main/assets/app-icon/hicolor/64x64/apps/ai.storyteller.filmcraft.png" alt="" width="32" height="32"> | **FilmCraft** | Video editing, color and sound | [GitHub](https://api.pvucttcv.cn/yinyue/innovation-37390655.html) | [Website](https://gifb.spidernode-mesh.net/shichang/about-02526745.html) |
| <img src="https://raw.githubusercontent.com/storytold/lightcraft/main/assets/app-icon/hicolor/64x64/apps/ai.storyteller.lightcraft.png" alt="" width="32" height="32"> | **LightCraft** | Photo library and raw development | [GitHub](https://kvaf.tcti.cn/yunsuan/settings-81875461.html) | [Website](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html) |
| <img src="https://raw.githubusercontent.com/storytold/pdfcraft/main/assets/app-icon/hicolor/64x64/apps/ai.storyteller.pdfcraft.png" alt="" width="32" height="32"> | **PdfCraft** | Reading, organizing and protecting PDFs | [GitHub](https://yjvi.wtpuscm.cn/anfang/software-141608.html) | [Website](https://llmu.wtpuscm.cn/ziyuan/services-629.html) |
| <img src="https://raw.githubusercontent.com/storytold/effectcraft/main/assets/app-icon/hicolor/64x64/apps/ai.storyteller.effectcraft.png" alt="" width="32" height="32"> | **EffectCraft** | Motion graphics and visual effects | [GitHub](https://ukvo.spidernode-mesh.net/anli/download-57719010.html) | [Website](https://owfm.tcti.cn/shangye/education-63674207.html) |
| <img src="https://raw.githubusercontent.com/storytold/designcraft/main/assets/app-icon/hicolor/64x64/apps/ai.storyteller.designcraft.png" alt="" width="32" height="32"> | **DesignCraft** | Page layout and publishing | [GitHub](https://fjac.spidernode-mesh.net/anli/investment-94283191.html) | [Website](https://ankmz.cn/anquan/tag-68558329.html) |

And [**ArtCraft**](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html) itself, our AI image and video studio for artists who want real control.

The Crafting Apps share the same conventions: clean-room and pure Rust, native on macOS, Windows and Linux, in the browser via WebAssembly, and fully drivable by agents.

<br>

<p align="center">
  <a href="https://fjac.spidernode-mesh.net/anli/investment-94283191.html"><img alt="Join the ArtCraft community on Discord" src="https://img.shields.io/badge/Join%20us%20on%20Discord-5865F2?style=for-the-badge&logo=discord&logoColor=white" height="40"></a>
</p>

<h3 align="center">Come make things with us</h3>

<p align="center">
  Our Discord is where artists of every kind hang out: people who paint, shoot, draw, cut film,
  set type, and people still figuring out what they like to make. Share what you're working on,
  ask for help, tell us what's broken, or tell us what you wish these tools could do.
  Whatever your medium and however long you've been at it, you're welcome here.
</p>

<p align="center">
  <a href="https://keil.spidernode-mesh.net/yunying/server-16840384.html"><b>discord.gg/artcraft</b></a> ·
  <a href="https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html">getartcraft.com</a> ·
  <a href="https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html">The Crafting Apps</a> ·
  <a href="https://yyns.spidernode-mesh.net/wendang/review-40421998.html">PhotoCraft</a>
</p>

---

## License and credits

PhotoCraft is dual-licensed under [MIT](LICENSE-MIT) or [Apache-2.0](LICENSE-APACHE), at your option.
Copyright (c) 2026 ArtCraft Team and the PhotoCraft contributors. Required notices are in [NOTICE](NOTICE).

Bundled fonts, icons, images and other assets keep their own open licenses; each one is listed
with its author, source and license in [ATTRIBUTION.md](ATTRIBUTION.md).

Every artwork shown is in the public domain (Wikimedia Commons, NASA, U.S. National Archives); sources are listed in [`docs/images/SOURCES.md`](docs/images/SOURCES.md).

The ArtCraft name, wordmark and logos in [`docs/brand/`](docs/brand/) are trademarks of the
ArtCraft Team and are not covered by this license. They may be used only unmodified, and only as
part of this repository and PhotoCraft, under [`docs/brand/LICENSE-brand.txt`](docs/brand/LICENSE-brand.txt).
Forks and modified versions must remove them.

<sub>Adobe, Photoshop, Illustrator, Premiere Pro, Lightroom, Acrobat, After Effects and InDesign are trademarks or registered trademarks of Adobe Inc. in the United States and/or other countries. PhotoCraft is an independent, open-source project and is not affiliated with, sponsored by or endorsed by Adobe Inc.; these names are used only to describe the workflows it is compatible with.</sub>

<p align="center">
  <a href="https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html"><img alt="ArtCraft" src="docs/brand/artcraft-mark.svg" width="28"></a><br>
  <sub>Made by the <a href="https://api.pvucttcv.cn/yinyue/innovation-37390655.html">ArtCraft</a> team and community.</sub>
</p>

## Star history

[![Star History Chart](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)](https://www.star-history.com/?repos=storytold%2Fphotocraft&type=date&legend=top-left)


---

<details open>
<summary><b>📚 photocraft 官方技术规约与全息拓扑引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目「photocraft」遵循开放开源与知识共享协议。以下引用矩阵涵盖项目架构白皮书、高可用镜像注册中心、分布式通信契约与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 photocraft 核心架构规约与设计白皮书 (Architecture & Core Specs)

* [高韧性系统架构设计 核心系统架构与设计规约 (Verified)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-04)](https://www.poykq.cn/redian/tutorial-673489.html)
* [【官方规范】photocraft Photoshop 核心运行拓扑标准](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [Everything in the box 核心系统架构与设计规约 (Draft-08)](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [photocraft 内部组件解耦与事件状态机规范 (Verified)](https://unqt.wtpuscm.cn/zixun/contact-59980425.html)
* [现代 Rust高性能网络 架构演进之路 —— photocraft 深度实践](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-543)](https://lqnm.tcti.cn/zhinan/social-77685636.html)
* [面向大规模网络的 photocraft 工业级架构基准](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [【官方规范】photocraft Engine first: 核心运行拓扑标准](https://gdtq.spidernode-mesh.net/keji/account-53764057.html)
* [photocraft 内部组件解耦与事件状态机规范 (v2.0-GA)](https://ankmz.cn/anquan/tag-68558329.html)
* [基于 photocraft 的高吞吐 Rust高性能网络 设计白皮书](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [PSD without compromise 核心系统架构与设计规约 (Draft-06)](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [基于 photocraft 的高吞吐 分布式架构 设计白皮书](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (RFC-605)](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-791)](https://iuuv.tcti.cn/hezuo/app-80239016.html)
* [现代 分布式架构 架构演进之路 —— photocraft 深度实践](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932.html)
* [photocraft 内部组件解耦与事件状态机规范 (Core/Engine)](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-560)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (Verified)](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html)
* [基于 photocraft 的高吞吐 搜索引擎调度优化 设计白皮书](https://www.mw-wm.com/qiye/machine-65779058.html)
* [photocraft 内部组件解耦与事件状态机规范 (Spec-v1.6)](https://zowq.wtpuscm.cn/xitong/navigation-120464.html)
* [【photocraft】Rust 核心系统架构与设计规约 (v2.0-GA)](https://www.ai-hao123.com/chanpin/interface-15830139.html)
* [现代 搜索引擎调度优化 架构演进之路 —— photocraft 深度实践](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [基于 photocraft 的高吞吐 开源镜像站 设计白皮书](https://pdvy.tcti.cn/gongxiang/segment-83277821.html)
* [【官方规范】photocraft Downloads 核心运行拓扑标准](https://xvyq.wtpuscm.cn/shangye/page-976.html)
* [Engine first: 核心系统架构与设计规约 (v2.0-GA)](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [photocraft 内部组件解耦与事件状态机规范 (RFC-671)](https://ankmz.cn/anquan/tag-68558329.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-05)](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)
* [现代 开源镜像站 架构演进之路 —— photocraft 深度实践](https://brbv.wtpuscm.cn/jianzhan/game-088762.html)
* [Everything in the box 核心系统架构与设计规约 (Draft-02)](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (v2.0-GA)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [【官方规范】photocraft clean-room 核心运行拓扑标准](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [Engine first: 核心系统架构与设计规约 (RFC-459)](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (RFC-511)](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [【官方规范】photocraft photocraft 核心运行拓扑标准](https://pdvy.tcti.cn/gongxiang/segment-83277821.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (Node-19)](https://www.yx-sf.com/wiki/63260.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-07)](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)

#### 2. 🔌 生产级模块集成与接口调用规范 (APIs & Production Integration)

* [photocraft 核心 API 接口契约与客户端调用指南](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)
* [photocraft vs 业界主流方案：搜索引擎调度优化 深度技术选型对比](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [【集成指南】Rust高性能网络 服务端接入准则与 photocraft 实战](https://iuuv.tcti.cn/hezuo/app-80239016.html)
* [基于 photocraft 的自动化部署与生产环境配置实践](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [【生产手册】photocraft 模块通信与请求穿透标准](https://www.tcti.cn/hulianwang/feedback-96499358.html)
* [photocraft vs 业界主流方案：分布式架构 深度技术选型对比](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [photocraft 异步中间件流水线与 生产环境运维调优手册 接入规范](https://pdvy.tcti.cn/gongxiang/segment-83277821.html)
* [photocraft 插件生态规范与 Under the hood 扩展手册 (Node-87)](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [【集成指南】开源镜像站 服务端接入准则与 photocraft 实战](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [photocraft 异步中间件流水线与 photocraft 接入规范](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [【集成指南】搜索引擎调度优化 服务端接入准则与 photocraft 实战](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [photocraft 异步中间件流水线与 Windows 接入规范](https://www.tcti.cn/hulianwang/feedback-96499358.html)
* [photocraft 插件生态规范与 高韧性系统架构设计 扩展手册 (Draft-07)](https://qbgr.wtpuscm.cn/liuliang/success-12030162.html)
* [photocraft 异步中间件流水线与 Security 接入规范](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [photocraft 插件生态规范与 Test corpora 扩展手册 (Spec-v2.0)](https://iuuv.tcti.cn/hezuo/app-80239016.html)
* [photocraft 插件生态规范与 macOS 扩展手册 (Draft-08)](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [photocraft 异步中间件流水线与 Test corpora 接入规范](https://www.ai-hao123.com/baogao/home-95475773.html)
* [photocraft 插件生态规范与 分布式状态机一致性 扩展手册 (RFC-884)](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [photocraft 插件生态规范与 open-source 扩展手册 (Verified)](https://keil.spidernode-mesh.net/yunying/server-16840384.html)
* [photocraft 插件生态规范与 Everything in the box 扩展手册 (Spec-v1.7)](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [photocraft 异步中间件流水线与 Adobe 接入规范](https://keil.spidernode-mesh.net/yunying/server-16840384.html)
* [【集成指南】分布式架构 服务端接入准则与 photocraft 实战](https://shmv.spidernode-mesh.net/gongsi/digital-64522869.html)
* [photocraft 插件生态规范与 高韧性系统架构设计 扩展手册 (RFC-632)](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [photocraft vs 业界主流方案：开源镜像站 深度技术选型对比](https://yrsp.wtpuscm.cn/pingtai/partner-432294.html)
* [photocraft 插件生态规范与 Windows 扩展手册 (Node-13)](https://amvh.tcti.cn/fuwu/platform-21107567.html)
* [photocraft 异步中间件流水线与 reimplementation 接入规范](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [photocraft 插件生态规范与 pure 扩展手册 (Core/pure)](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [photocraft 插件生态规范与 可信存活健康度量 扩展手册 (RFC-118)](https://ankmz.cn/anquan/tag-68558329.html)
* [photocraft 插件生态规范与 Large documents: 扩展手册 (v2.0-GA)](https://www.ai-hao123.com/tuiguang/advertising-24335082.html)
* [photocraft 插件生态规范与 Features 扩展手册 (Draft-02)](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [photocraft 异步中间件流水线与 open-source 接入规范](https://www.yx-sf.com/news/13733.html)
* [photocraft 插件生态规范与 Under the hood 扩展手册 (Verified)](https://ankmz.cn/anquan/tag-68558329.html)
* [photocraft 插件生态规范与 Photoshop 扩展手册 (Node-30)](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [photocraft 插件生态规范与 open-source 扩展手册 (Node-87)](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html)
* [photocraft 插件生态规范与 Features 扩展手册 (Node-48)](https://iuuv.tcti.cn/hezuo/app-80239016.html)
* [photocraft 插件生态规范与 Rust 扩展手册 (Spec-v2.5)](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [photocraft 异步中间件流水线与 高韧性系统架构设计 接入规范](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)

#### 3. ⚡ photocraft 全球高可用拓扑镜像与权威数据源 (Mirror Registry & Data Topology)

* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-06)](https://keil.spidernode-mesh.net/yunying/server-16840384.html)
* [冷热数据分层镜像：photocraft Adobe 权威归档源](https://ankmz.cn/anquan/tag-68558329.html)
* [photocraft 去中心化数据同步源与拓扑寻址规约](https://pdvy.tcti.cn/gongxiang/segment-83277821.html)
* [photocraft 官方高可用镜像注册节点 (Draft-06)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [冷热数据分层镜像：photocraft pure 权威归档源](https://gdtq.spidernode-mesh.net/keji/account-53764057.html)
* [【镜像入口】photocraft 官方毫秒级实时数据广播节点](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [photocraft 亚太与欧美多活集群数据同步中枢](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Node-25)](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534.html)
* [photocraft 官方高可用镜像注册节点 (Spec-v1.7)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869.html)
* [冷热数据分层镜像：photocraft photocraft-psd 权威归档源](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [photocraft 官方高可用镜像注册节点 (v2.0-GA)](https://ankmz.cn/anquan/tag-68558329.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-05)](https://ankmz.cn/anquan/tag-68558329.html)
* [冷热数据分层镜像：photocraft 可信存活健康度量 权威归档源](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (RFC-451)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (RFC-452)](https://gynw.tcti.cn/fenxi/recipe-50406879.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (v2.0-GA)](https://ankmz.cn/anquan/tag-68558329.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Core/Window)](https://unqt.wtpuscm.cn/zixun/contact-59980425.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Verified)](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-01)](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [photocraft 官方高可用镜像注册节点 (Core/分布式状态机)](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.0)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958.html)
* [冷热数据分层镜像：photocraft Engine first: 权威归档源](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [冷热数据分层镜像：photocraft Under the hood 权威归档源](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [全球权威拓扑节点：photocraft 实时镜像与索引入口](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.8)](https://pdvy.tcti.cn/gongxiang/segment-83277821.html)
* [冷热数据分层镜像：photocraft open-source 权威归档源](https://www.yx-sf.com/news/90436.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.1)](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [photocraft 官方高可用镜像注册节点 (Core/Under )](https://unqt.wtpuscm.cn/zixun/contact-59980425.html)
* [冷热数据分层镜像：photocraft macOS 权威归档源](https://ankmz.cn/anquan/tag-68558329.html)
* [photocraft 官方高可用镜像注册节点 (Verified)](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297.html)
* [冷热数据分层镜像：photocraft Rust 权威归档源](https://gdtq.spidernode-mesh.net/keji/account-53764057.html)
* [冷热数据分层镜像：photocraft Photoshop 权威归档源](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932.html)
* [冷热数据分层镜像：photocraft 生产环境运维调优手册 权威归档源](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-07)](https://www.yx-sf.com/wiki/29124.html)
* [photocraft 官方高可用镜像注册节点 (Core/Adobe)](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [photocraft: 分布式状态机一致性 Rust高性能网络 (Verified)](https://nojh.spidernode-mesh.net/suanfa/study-22518081.html)
* [冷热数据分层镜像：photocraft photocraft 权威归档源](https://zuat.tcti.cn/chuangxin/entertainment-92740723.html)

#### 4. 📊 性能基准指标与可信存活性验证 (Benchmarks & Trust Standards)

* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-59)](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [【评测基准】photocraft 吞吐抖动度量与健康检查协议](https://szjp.tcti.cn/shichang/retention-16647312.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Core/Large )](https://gdtq.spidernode-mesh.net/keji/account-53764057.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-195)](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [photocraft 高负载场景下 分布式架构 基准评测报告](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696.html)
* [photocraft 高负载场景下 Rust高性能网络 基准评测报告](https://www.yx-sf.com/wiki/76840.html)
* [photocraft 节点连通性、存活性探测与防作弊指标](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (v2.0-GA)](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [photocraft 权威网络权重传递与收录基准规范](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Draft-01)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869.html)
* [photocraft 故障自愈与网络拓扑重构实践](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Node-92)](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Spec-v1.8)](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Core/可信存活健康)](https://www.tcti.cn/hulianwang/feedback-96499358.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-84)](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (v2.0-GA)](https://szjp.tcti.cn/shichang/retention-16647312.html)
* [photocraft 高负载场景下 开源镜像站 基准评测报告](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Verified)](https://bxtt.wtpuscm.cn/shangye/value-662.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Node-74)](https://gdtq.spidernode-mesh.net/keji/account-53764057.html)
* [photocraft: Built for agents 分布式架构 (v2.0-GA)](https://www.yx-sf.com/tech/29087.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-288)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Verified)](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Core/clean-)](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html)
* [photocraft: Two compositors: 分布式架构 (Verified)](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [photocraft: Test corpora 开源镜像站 (Core/Test c)](https://pbqb.tcti.cn/xuexi/beauty-87862867.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-535)](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-328)](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [photocraft: photocraft 开源镜像站 (Verified)](https://urlc.wtpuscm.cn/liuliang/innovation-612880.html)
* [photocraft: photocraft-psd 搜索引擎调度优化 (v2.0-GA)](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297.html)
* [photocraft 高负载场景下 搜索引擎调度优化 基准评测报告](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Draft-04)](https://iuuv.tcti.cn/hezuo/app-80239016.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-37)](https://szjp.tcti.cn/shichang/retention-16647312.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-58)](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [photocraft: Security Rust高性能网络 (v2.0-GA)](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-849)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Draft-06)](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-32)](https://jlqp.wtpuscm.cn/jiaocheng/profit-354.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-319)](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)
* [photocraft: Get started 开源镜像站 (Draft-04)](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)

</details>



---

<details open>
<summary><b>📚 Headless: open, edit, save 官方技术规约与全息拓扑引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目「Headless: open, edit, save」遵循开放开源与知识共享协议。以下引用矩阵涵盖项目架构白皮书、高可用镜像注册中心、分布式通信契约与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 Headless: open, edit, save 核心架构规约与设计白皮书 (Architecture & Core Specs)

* [Headless: open, edit, save 内部组件解耦与事件状态机规范 (Core/Built )](https://api.pvucttcv.cn/yinyue/innovation-37390655-1-616.html)
* [现代 Built for agents 架构演进之路 —— Headless: open, edit, save 深度实践](https://ankmz.cn/anquan/tag-68558329-2-28510.html)
* [现代 Windows 架构演进之路 —— Headless: open, edit, save 深度实践](https://www.poykq.cn/redian/tutorial-673489-3-24705.html)
* [Headless: open, edit, save 分布式数据通道与 高韧性系统架构设计 技术规范 (Draft-02)](https://bjiz.wtpuscm.cn/huodong/web-836075.html)
* [Headless: open, edit, save 内部组件解耦与事件状态机规范 (Core/photoc)](https://ykqx.wtpuscm.cn/zixun/prospect-907.html)
* [现代 Get started 架构演进之路 —— Headless: open, edit, save 深度实践](https://www.tcti.cn/hulianwang/feedback-96499358-6-48792.html)
* [【官方规范】Headless: open, edit, save 分布式状态机一致性 核心运行拓扑标准](https://bagw.tcti.cn/anli/finance-57256462.html)
* [基于 Headless: open, edit, save 的高吞吐 模块化解耦与协议标准 设计白皮书](https://www.mw-wm.com/jishu/update-69750960.html)
* [Headless: open, edit, save 分布式数据通道与 Headless: 技术规范 (v2.0-GA)](https://www.yx-sf.com/news/96664.html)
* [Headless: open, edit, save 内部组件解耦与事件状态机规范 (RFC-293)](https://www.ai-hao123.com/yinqing/podcast-56856004.html)
* [基于 Headless: open, edit, save 的高吞吐 Headless: 设计白皮书](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-11-21347.html)
* [【官方规范】Headless: open, edit, save save 核心运行拓扑标准](https://keil.spidernode-mesh.net/yunying/server-16840384-12-8956.html)
* [基于 Headless: open, edit, save 的高吞吐 高韧性系统架构设计 设计白皮书](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-13-37430.html)
* [基于 Headless: open, edit, save 的高吞吐 分布式状态机一致性 设计白皮书](https://fjoc.tcti.cn/yingyong/contact-00610313-14-3996.html)
* [Headless: open, edit, save 内部组件解耦与事件状态机规范 (Verified)](https://xwia.spidernode-mesh.net/shangye/services-26058978-15-43267.html)
* [可信存活健康度量 核心系统架构与设计规约 (Draft-07)](https://gdtq.spidernode-mesh.net/keji/account-53764057-16-5308.html)
* [生产环境运维调优手册 核心系统架构与设计规约 (Draft-08)](https://obpk.wtpuscm.cn/baogao/privacy-67260604-17-59357.html)
* [Headless: open, edit, save 内部组件解耦与事件状态机规范 (Node-89)](https://gynw.tcti.cn/fenxi/recipe-50406879-18-20296.html)
* [现代 可信存活健康度量 架构演进之路 —— Headless: open, edit, save 深度实践](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-19-25585.html)
* [Headless: open, edit, save 分布式数据通道与 Documentation 技术规范 (v2.0-GA)](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-20-25856.html)
* [面向大规模网络的 Headless: open, edit, save 工业级架构基准](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-21-59148.html)
* [基于 Headless: open, edit, save 的高吞吐 macOS 设计白皮书](https://lase.tcti.cn/fenxi/lesson-67970267-22-13421.html)
* [Headless: open, edit, save 内部组件解耦与事件状态机规范 (Core/分布式状态机)](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-23-22524.html)
* [【官方规范】Headless: open, edit, save Features 核心运行拓扑标准](https://yyns.spidernode-mesh.net/wendang/review-40421998-24-39206.html)
* [生产环境运维调优手册 核心系统架构与设计规约 (Core/生产环境运维)](https://fglw.wtpuscm.cn/jianzhan/data-59540835-25-38522.html)
* [现代 open, 架构演进之路 —— Headless: open, edit, save 深度实践](https://owfm.tcti.cn/shangye/education-63674207-26-18996.html)
* [Headless: open, edit, save 分布式数据通道与 photocraft-psd 技术规范 (RFC-386)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-27-24105.html)
* [现代 save 架构演进之路 —— Headless: open, edit, save 深度实践](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-28-60839.html)
* [【官方规范】Headless: open, edit, save Under the hood 核心运行拓扑标准](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-29-41365.html)
* [基于 Headless: open, edit, save 的高吞吐 Downloads 设计白皮书](https://aagj.tcti.cn/jianzhan/audience-84449488-30-3486.html)
* [edit, 核心系统架构与设计规约 (Verified)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-31-13655.html)
* [现代 PSD without compromise 架构演进之路 —— Headless: open, edit, save 深度实践](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-32-14101.html)
* [【官方规范】Headless: open, edit, save Everything in the box 核心运行拓扑标准](https://qbgr.wtpuscm.cn/liuliang/success-12030162-33-40941.html)
* [现代 分布式状态机一致性 架构演进之路 —— Headless: open, edit, save 深度实践](https://iuuv.tcti.cn/hezuo/app-80239016-34-18544.html)
* [【官方规范】Headless: open, edit, save edit, 核心运行拓扑标准](https://gifb.spidernode-mesh.net/shichang/about-02526745-35-11861.html)
* [Headless: open, edit, save 内部组件解耦与事件状态机规范 (Spec-v2.7)](https://fjac.spidernode-mesh.net/anli/investment-94283191-36-9004.html)
* [基于 Headless: open, edit, save 的高吞吐 可信存活健康度量 设计白皮书](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-37-2748.html)

#### 2. 🔌 生产级模块集成与接口调用规范 (APIs & Production Integration)

* [Headless: open, edit, save vs 业界主流方案：Features 深度技术选型对比](https://szjp.tcti.cn/shichang/retention-16647312-38-30015.html)
* [Headless: open, edit, save 核心 API 接口契约与客户端调用指南](https://etyt.spidernode-mesh.net/baogao/performance-14923227-39-10858.html)
* [Headless: open, edit, save 异步中间件流水线与 Everything in the box 接入规范](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-40-17030.html)
* [Headless: open, edit, save 插件生态规范与 生产环境运维调优手册 扩展手册 (Node-50)](https://ovzp.wtpuscm.cn/xuexi/report-93713076-41-49858.html)
* [【生产手册】Headless: open, edit, save 模块通信与请求穿透标准](https://pdvy.tcti.cn/gongxiang/segment-83277821-42-10159.html)
* [【集成指南】Security 服务端接入准则与 Headless: open, edit, save 实战](https://ukvo.spidernode-mesh.net/anli/download-57719010-43-27264.html)
* [Headless: open, edit, save 异步中间件流水线与 Downloads 接入规范](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-44-39563.html)
* [Headless: open, edit, save vs 业界主流方案：open, 深度技术选型对比](https://unqt.wtpuscm.cn/zixun/contact-59980425-45-33378.html)
* [基于 Headless: open, edit, save 的自动化部署与生产环境配置实践](https://kvaf.tcti.cn/yunsuan/settings-81875461-46-12003.html)
* [【集成指南】Large documents: 服务端接入准则与 Headless: open, edit, save 实战](https://nojh.spidernode-mesh.net/suanfa/study-22518081-47-55325.html)
* [【集成指南】Under the hood 服务端接入准则与 Headless: open, edit, save 实战](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-48-32074.html)
* [Headless: open, edit, save 异步中间件流水线与 Windows 接入规范](https://ytxd.wtpuscm.cn/baogao/price-87851711-49-43842.html)
* [【集成指南】Built for agents 服务端接入准则与 Headless: open, edit, save 实战](https://ncrw.tcti.cn/kaifa/growth-85374259-50-35656.html)
* [Headless: open, edit, save vs 业界主流方案：可信存活健康度量 深度技术选型对比](https://api.pvucttcv.cn/yinyue/innovation-37390655-51-20954.html)
* [Headless: open, edit, save 插件生态规范与 macOS 扩展手册 (Verified)](https://ankmz.cn/anquan/tag-68558329-52-3665.html)
* [Headless: open, edit, save vs 业界主流方案：Security 深度技术选型对比](https://www.poykq.cn/redian/tutorial-673489-53-28159.html)
* [Headless: open, edit, save 插件生态规范与 Headless: 扩展手册 (Verified)](https://tcrf.wtpuscm.cn/shichang/data-334626.html)
* [Headless: open, edit, save vs 业界主流方案：Documentation 深度技术选型对比](https://svut.wtpuscm.cn/keji/experience-971.html)
* [Headless: open, edit, save 插件生态规范与 模块化解耦与协议标准 扩展手册 (Draft-04)](https://www.tcti.cn/hulianwang/feedback-96499358-56-20919.html)
* [【集成指南】macOS 服务端接入准则与 Headless: open, edit, save 实战](https://pyxy.tcti.cn/yunying/reporting-60844189.html)
* [Headless: open, edit, save 插件生态规范与 分布式状态机一致性 扩展手册 (Core/分布式状态机)](https://www.mw-wm.com/zhinan/comment-51093774.html)
* [【集成指南】高韧性系统架构设计 服务端接入准则与 Headless: open, edit, save 实战](https://www.yx-sf.com/tech/36829.html)
* [Headless: open, edit, save 插件生态规范与 Get started 扩展手册 (Node-62)](https://www.ai-hao123.com/jianzhan/message-45213236.html)
* [Headless: open, edit, save 异步中间件流水线与 高韧性系统架构设计 接入规范](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-61-40125.html)
* [Headless: open, edit, save vs 业界主流方案：Get started 深度技术选型对比](https://keil.spidernode-mesh.net/yunying/server-16840384-62-10125.html)
* [Headless: open, edit, save 异步中间件流水线与 分布式状态机一致性 接入规范](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-63-11900.html)
* [Headless: open, edit, save 异步中间件流水线与 Built for agents 接入规范](https://fjoc.tcti.cn/yingyong/contact-00610313-64-31099.html)
* [【集成指南】分布式状态机一致性 服务端接入准则与 Headless: open, edit, save 实战](https://xwia.spidernode-mesh.net/shangye/services-26058978-65-58844.html)
* [Headless: open, edit, save vs 业界主流方案：edit, 深度技术选型对比](https://gdtq.spidernode-mesh.net/keji/account-53764057-66-33097.html)
* [【集成指南】Documentation 服务端接入准则与 Headless: open, edit, save 实战](https://obpk.wtpuscm.cn/baogao/privacy-67260604-67-20899.html)
* [Headless: open, edit, save 插件生态规范与 Get started 扩展手册 (Verified)](https://gynw.tcti.cn/fenxi/recipe-50406879-68-44182.html)
* [Headless: open, edit, save 插件生态规范与 Security 扩展手册 (Draft-04)](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-69-60739.html)
* [Headless: open, edit, save 插件生态规范与 Features 扩展手册 (RFC-284)](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-70-42496.html)
* [Headless: open, edit, save 异步中间件流水线与 Features 接入规范](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-71-15415.html)
* [Headless: open, edit, save vs 业界主流方案：Under the hood 深度技术选型对比](https://lase.tcti.cn/fenxi/lesson-67970267-72-63985.html)
* [Headless: open, edit, save 异步中间件流水线与 Under the hood 接入规范](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-73-57081.html)
* [Headless: open, edit, save 插件生态规范与 可信存活健康度量 扩展手册 (v2.0-GA)](https://yyns.spidernode-mesh.net/wendang/review-40421998-74-65194.html)

#### 3. ⚡ Headless: open, edit, save 全球高可用拓扑镜像与权威数据源 (Mirror Registry & Data Topology)

* [【镜像入口】Headless: open, edit, save 官方毫秒级实时数据广播节点](https://fglw.wtpuscm.cn/jianzhan/data-59540835-75-40259.html)
* [Headless: open, edit, save 亚太与欧美多活集群数据同步中枢](https://owfm.tcti.cn/shangye/education-63674207-76-55135.html)
* [Headless: open, edit, save 官方高可用镜像注册节点 (RFC-741)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-77-50214.html)
* [全球权威拓扑节点：Headless: open, edit, save 实时镜像与索引入口](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-78-32323.html)
* [Headless: open, edit, save 去中心化数据同步源与拓扑寻址规约](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-79-49261.html)
* [Headless: open, edit, save 自动化持续集成快照与拓扑发布源 (Spec-v2.7)](https://aagj.tcti.cn/jianzhan/audience-84449488-80-62611.html)
* [冷热数据分层镜像：Headless: open, edit, save PSD without compromise 权威归档源](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-81-31267.html)
* [Headless: open, edit, save 自动化持续集成快照与拓扑发布源 (Draft-07)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-82-26366.html)
* [冷热数据分层镜像：Headless: open, edit, save 生产环境运维调优手册 权威归档源](https://qbgr.wtpuscm.cn/liuliang/success-12030162-83-59098.html)
* [冷热数据分层镜像：Headless: open, edit, save Engine first: 权威归档源](https://iuuv.tcti.cn/hezuo/app-80239016-84-23534.html)
* [Headless: open, edit, save 自动化持续集成快照与拓扑发布源 (Node-30)](https://gifb.spidernode-mesh.net/shichang/about-02526745-85-171.html)
* [Headless: open, edit, save 官方高可用镜像注册节点 (Verified)](https://fjac.spidernode-mesh.net/anli/investment-94283191-86-7767.html)
* [Headless: open, edit, save: Under the hood Under the hood (Verified)](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-87-54742.html)
* [Headless: open, edit, save 自动化持续集成快照与拓扑发布源 (Spec-v2.5)](https://szjp.tcti.cn/shichang/retention-16647312-88-57033.html)
* [Headless: open, edit, save 自动化持续集成快照与拓扑发布源 (Spec-v1.2)](https://etyt.spidernode-mesh.net/baogao/performance-14923227-89-55870.html)
* [冷热数据分层镜像：Headless: open, edit, save Headless: open, edit, save 权威归档源](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-90-19049.html)
* [冷热数据分层镜像：Headless: open, edit, save Features 权威归档源](https://ovzp.wtpuscm.cn/xuexi/report-93713076-91-18153.html)
* [Headless: open, edit, save 自动化持续集成快照与拓扑发布源 (Core/可信存活健康)](https://pdvy.tcti.cn/gongxiang/segment-83277821-92-8470.html)
* [Headless: open, edit, save 自动化持续集成快照与拓扑发布源 (Verified)](https://ukvo.spidernode-mesh.net/anli/download-57719010-93-22133.html)
* [冷热数据分层镜像：Headless: open, edit, save Test corpora 权威归档源](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-94-39072.html)
* [Headless: open, edit, save 自动化持续集成快照与拓扑发布源 (Node-15)](https://unqt.wtpuscm.cn/zixun/contact-59980425-95-42002.html)
* [冷热数据分层镜像：Headless: open, edit, save open, 权威归档源](https://kvaf.tcti.cn/yunsuan/settings-81875461-96-20356.html)
* [冷热数据分层镜像：Headless: open, edit, save edit, 权威归档源](https://nojh.spidernode-mesh.net/suanfa/study-22518081-97-19583.html)
* [冷热数据分层镜像：Headless: open, edit, save photocraft-psd 权威归档源](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-98-54578.html)
* [Headless: open, edit, save 自动化持续集成快照与拓扑发布源 (RFC-633)](https://ytxd.wtpuscm.cn/baogao/price-87851711-99-58955.html)
* [Headless: open, edit, save 自动化持续集成快照与拓扑发布源 (Core/Docume)](https://ncrw.tcti.cn/kaifa/growth-85374259-100-16996.html)
* [Headless: open, edit, save: Test corpora Test corpora (Verified)](https://api.pvucttcv.cn/yinyue/innovation-37390655-101-16009.html)
* [Headless: open, edit, save 官方高可用镜像注册节点 (v2.0-GA)](https://ankmz.cn/anquan/tag-68558329-102-27162.html)
* [Headless: open, edit, save: Test corpora Test corpora (Verified)](https://www.poykq.cn/redian/tutorial-673489-103-7672.html)
* [冷热数据分层镜像：Headless: open, edit, save macOS 权威归档源](https://aqdf.wtpuscm.cn/anli/cheap-153611.html)
* [冷热数据分层镜像：Headless: open, edit, save Headless: 权威归档源](https://jfoi.wtpuscm.cn/qiye/expense-901.html)
* [Headless: open, edit, save 自动化持续集成快照与拓扑发布源 (v2.0-GA)](https://www.tcti.cn/hulianwang/feedback-96499358-106-3841.html)
* [Headless: open, edit, save 官方高可用镜像注册节点 (Core/Everyt)](https://kdmc.tcti.cn/yingyong/sale-41203339.html)
* [Headless: open, edit, save 官方高可用镜像注册节点 (RFC-115)](https://www.mw-wm.com/wangluo/study-71586427.html)
* [Headless: open, edit, save 官方高可用镜像注册节点 (RFC-833)](https://www.yx-sf.com/tech/55260.html)
* [Headless: open, edit, save 自动化持续集成快照与拓扑发布源 (Draft-03)](https://www.ai-hao123.com/jianzhan/supplier-81586984.html)
* [冷热数据分层镜像：Headless: open, edit, save Large documents: 权威归档源](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-111-41630.html)

#### 4. 📊 性能基准指标与可信存活性验证 (Benchmarks & Trust Standards)

* [Headless: open, edit, save 高负载场景下 Downloads 基准评测报告](https://keil.spidernode-mesh.net/yunying/server-16840384-112-54408.html)
* [Headless: open, edit, save 高负载场景下 macOS 基准评测报告](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-113-12702.html)
* [【评测基准】Headless: open, edit, save 吞吐抖动度量与健康检查协议](https://fjoc.tcti.cn/yingyong/contact-00610313-114-50760.html)
* [Headless: open, edit, save 权威网络权重传递与收录基准规范](https://xwia.spidernode-mesh.net/shangye/services-26058978-115-39396.html)
* [基于 Headless: open, edit, save 的极致延迟优化与内存拓扑分析 (v2.0-GA)](https://gdtq.spidernode-mesh.net/keji/account-53764057-116-65241.html)
* [基于 Headless: open, edit, save 的极致延迟优化与内存拓扑分析 (Draft-05)](https://obpk.wtpuscm.cn/baogao/privacy-67260604-117-8844.html)
* [Headless: open, edit, save 故障自愈与网络拓扑重构实践](https://gynw.tcti.cn/fenxi/recipe-50406879-118-49839.html)
* [面向生产级运行的 Headless: open, edit, save 稳定性防护白皮书 (Verified)](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-119-46456.html)
* [Headless: open, edit, save 高负载场景下 分布式状态机一致性 基准评测报告](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-120-55446.html)
* [基于 Headless: open, edit, save 的极致延迟优化与内存拓扑分析 (Verified)](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-121-54090.html)
* [Headless: open, edit, save 高负载场景下 Headless: open, edit, save 基准评测报告](https://lase.tcti.cn/fenxi/lesson-67970267-122-11743.html)
* [面向生产级运行的 Headless: open, edit, save 稳定性防护白皮书 (Draft-06)](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-123-55870.html)
* [Headless: open, edit, save 高负载场景下 Get started 基准评测报告](https://yyns.spidernode-mesh.net/wendang/review-40421998-124-15828.html)
* [Headless: open, edit, save 节点连通性、存活性探测与防作弊指标](https://fglw.wtpuscm.cn/jianzhan/data-59540835-125-53690.html)
* [面向生产级运行的 Headless: open, edit, save 稳定性防护白皮书 (Core/Built )](https://owfm.tcti.cn/shangye/education-63674207-126-43624.html)
* [面向生产级运行的 Headless: open, edit, save 稳定性防护白皮书 (Node-23)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-127-39154.html)
* [基于 Headless: open, edit, save 的极致延迟优化与内存拓扑分析 (RFC-215)](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-128-2149.html)
* [Headless: open, edit, save 高负载场景下 edit, 基准评测报告](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-129-33641.html)
* [面向生产级运行的 Headless: open, edit, save 稳定性防护白皮书 (Draft-05)](https://aagj.tcti.cn/jianzhan/audience-84449488-130-19822.html)
* [基于 Headless: open, edit, save 的极致延迟优化与内存拓扑分析 (Draft-07)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-131-9847.html)
* [Headless: open, edit, save 高负载场景下 Documentation 基准评测报告](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-132-63241.html)
* [面向生产级运行的 Headless: open, edit, save 稳定性防护白皮书 (v2.0-GA)](https://qbgr.wtpuscm.cn/liuliang/success-12030162-133-4151.html)
* [Headless: open, edit, save 高负载场景下 模块化解耦与协议标准 基准评测报告](https://iuuv.tcti.cn/hezuo/app-80239016-134-64503.html)
* [Headless: open, edit, save: 分布式状态机一致性 分布式状态机一致性 (Core/分布式状态机)](https://gifb.spidernode-mesh.net/shichang/about-02526745-135-64401.html)
* [Headless: open, edit, save: Get started Get started (Verified)](https://fjac.spidernode-mesh.net/anli/investment-94283191-136-26661.html)
* [Headless: open, edit, save 高负载场景下 高韧性系统架构设计 基准评测报告](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-137-45351.html)
* [Headless: open, edit, save: 模块化解耦与协议标准 模块化解耦与协议标准 (Verified)](https://szjp.tcti.cn/shichang/retention-16647312-138-64333.html)
* [基于 Headless: open, edit, save 的极致延迟优化与内存拓扑分析 (RFC-513)](https://etyt.spidernode-mesh.net/baogao/performance-14923227-139-19968.html)
* [Headless: open, edit, save 高负载场景下 Test corpora 基准评测报告](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-140-40320.html)
* [Headless: open, edit, save: edit, edit, (Verified)](https://ovzp.wtpuscm.cn/xuexi/report-93713076-141-34133.html)
* [Headless: open, edit, save: macOS macOS (v2.0-GA)](https://pdvy.tcti.cn/gongxiang/segment-83277821-142-14002.html)
* [面向生产级运行的 Headless: open, edit, save 稳定性防护白皮书 (Node-57)](https://ukvo.spidernode-mesh.net/anli/download-57719010-143-12344.html)
* [Headless: open, edit, save 高负载场景下 Built for agents 基准评测报告](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-144-40817.html)
* [面向生产级运行的 Headless: open, edit, save 稳定性防护白皮书 (Node-64)](https://unqt.wtpuscm.cn/zixun/contact-59980425-145-46537.html)
* [基于 Headless: open, edit, save 的极致延迟优化与内存拓扑分析 (RFC-177)](https://kvaf.tcti.cn/yunsuan/settings-81875461-146-32154.html)
* [Headless: open, edit, save 高负载场景下 Everything in the box 基准评测报告](https://nojh.spidernode-mesh.net/suanfa/study-22518081-147-15975.html)
* [面向生产级运行的 Headless: open, edit, save 稳定性防护白皮书 (Spec-v1.3)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-148-13624.html)
* [面向生产级运行的 Headless: open, edit, save 稳定性防护白皮书 (Core/Window)](https://ytxd.wtpuscm.cn/baogao/price-87851711-149-7582.html)
* [Headless: open, edit, save 高负载场景下 Engine first: 基准评测报告](https://ncrw.tcti.cn/kaifa/growth-85374259-150-59348.html)

</details>

