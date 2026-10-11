# Releasing PhotoCraft

Every push to the `release` branch runs `.github/workflows/release.yml`. The workflow builds
signed installers for macOS, Windows, Linux and the web, plus a FreeBSD tarball, then creates
or updates a **draft** GitHub Release named `PhotoCraft v<version>`. Nobody sees a draft until a
maintainer publishes it.

This is PhotoCraft's implementation of the shared
[release playbook](release-playbook.md). User-facing names say **PhotoCraft**. Files, binaries
and ids stay lowercase (`photocraft-<version>-<platform>-<arch>.<ext>`, `ai.storyteller.photocraft`).

## Cutting a release

1. **Bump the version** on `main`. The only place it lives is `[workspace.package] version`
   in the root `Cargo.toml`:

   ```sh
   cargo xtask version                 # prints the current version, e.g. 0.1.0
   cargo xtask version set 0.2.0       # or 0.2.0-rc.1; updates Cargo.toml and Cargo.lock
   ```

   Commit the change (`Cargo.toml` + `Cargo.lock`) through the normal review flow.
2. **Merge `main` into `release`** (or fast-forward it) and push. The workflow starts by itself.
3. **Wait for the draft.** After about 30 to 45 minutes (notarization is the slow part), the
   Releases page has a draft `PhotoCraft v0.2.0`, tagged `v0.2.0` on the pushed commit, with
   every artifact and `SHA256SUMS.txt`. The notes are generated from the merged PRs.
4. **Check it.** Download an installer or two and look at the job summaries. Any
   `::warning::` there means a signing secret was missing and that artifact is unsigned.
5. **Add the scorecard deltas** to the draft's notes: what moved in
   [`docs/scorecard.md`](scorecard.md) since the previous release. Compare the summary table
   and the performance table of both tags (`git diff v<previous> v<this> -- docs/scorecard.md`)
   and list each area's done / partial / missing change, the scenarios that newly meet their
   budget or got slower, corpus floors raised, and the change in settings that do nothing.
   Quote numbers with the baseline's machine and load average.
6. **Publish** the draft in the GitHub UI. Publishing creates the `v0.2.0` tag. Versions with a
   pre-release suffix (`-rc.1`) are marked as pre-releases.

Pushing to `release` again before you publish rebuilds the same draft and replaces its assets.
After the draft is published, the workflow refuses to touch that version again, so bump it first.

**Test runs:** *Actions → Release → Run workflow* runs the whole pipeline by hand. The optional
`version` input (such as `0.2.0-rc.1`) overrides `Cargo.toml` for that run only. The jobs apply
it with `cargo xtask version set` before building, so the binaries report it too. The run still
needs the `release` environment, which only the `release` branch can use, so pick that branch
in the dialog.

## What gets built

| Platform | Artifacts | Built on |
|---|---|---|
| macOS 11+ (universal: Apple silicon + Intel) | `photocraft-<v>-macos-universal.dmg`, `photocraft-cli-<v>-macos-universal.zip` | `macos-15` |
| Windows 10+ x64 | `photocraft-<v>-windows-x64.msi`, `photocraft-<v>-windows-x64-portable.zip` | `windows-latest` |
| Windows 10+ x86 (32-bit) | `photocraft-<v>-windows-x86.msi`, `photocraft-<v>-windows-x86-portable.zip` | `windows-latest` |
| Linux x86_64 | `photocraft-<v>-linux-x86_64.{AppImage,AppImage.zsync,deb,rpm,tar.gz,flatpak}` | `ubuntu-22.04` (Flatpak: `ubuntu-24.04`) |
| Linux aarch64 | `photocraft-<v>-linux-aarch64.{AppImage,AppImage.zsync,deb,rpm,tar.gz,flatpak}` | `ubuntu-22.04-arm` (Flatpak: `ubuntu-24.04-arm`) |
| FreeBSD 14 x86_64 | `photocraft-<v>-freebsd-x86_64.tar.gz` | FreeBSD 14.3 VM on `ubuntu-latest` |
| Web | `photocraft-web-<v>.zip` (static site; see [`packaging/web/README.md`](../packaging/web/README.md)) | `ubuntu-latest` |

Every binary reports its version, the commit and the build date: `photocraft --version`,
`photocraft-cli --version`, and *Help › About PhotoCraft*. CI sets `PHOTOCRAFT_BUILD_SHA` and
`PHOTOCRAFT_BUILD_DATE`, and `crates/engine/src/build_info.rs` reads them at compile time. A plain
`cargo build` doesn't set them and reports `0.2.0 (dev build)`.

Every desktop build job (macOS, Windows, Linux, FreeBSD) also checks out [craft-fonts](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
at the commit in `CRAFT_FONTS_REF` (top of `release.yml`) and builds with `CRAFT_FONTS_DIR` and
`CRAFT_FONTS_REQUIRED=1`, so desktop releases embed its Japanese fonts (the web build embeds none: see
`docs/development.md` › Fonts) and fail rather than ship without them. The packages carry each font's licence as
`OFL-<family>.txt` (`copy_font_licences` in `packaging/env.sh`; the portable zip on Windows;
`Contents/Resources/Licenses` in the macOS app). Bump the pin deliberately, together with the one
in `ci.yml`. Rules: `../craftrules/standards/fonts.md`; build option: `docs/development.md` › Fonts.

### macOS

`packaging/macos/package.sh` builds `aarch64-apple-darwin` and `x86_64-apple-darwin` with
`MACOSX_DEPLOYMENT_TARGET=11.0`, joins them with `lipo`, and assembles `PhotoCraft.app`:

- `Info.plist` is generated from `Info.plist.in`. The bundle id is `ai.storyteller.photocraft`.
  The plist sets `LSMinimumSystemVersion` 11.0, `NSHighResolutionCapable`, and document types:
  `.pcraft` (Owner), plus PSD/PSB and the image formats PhotoCraft reads (Alternate, so it never
  takes over Preview's defaults). The icon is `assets/app-icon/photocraft.icns`.
- **Signing** goes inside-out with the hardened runtime and a secure timestamp. The executable
  is signed first, then the bundle. There's no `--deep` on the final signature. The
  entitlements (`entitlements.plist`) are deliberately empty.
- **Notarization:** the app is zipped and sent with `xcrun notarytool submit --wait`, then the
  ticket is stapled to the app. The app goes on a DMG (`hdiutil`, with an `Applications` link
  to drag onto). The DMG is signed, notarized and stapled too. The script checks the results
  with `codesign --verify --strict`, `stapler validate` and `spctl -a -vvv`.
  Its Finder window (background, icon size and positions) comes from
  [`packaging/macos/dmg/`](../packaging/macos/dmg/README.md), and its volume is named `PhotoCraft`
  without the version, which the window's background needs; the DMG file name keeps the version.
- **CLI:** the universal `photocraft-cli` is signed with the same Developer ID, the hardened
  runtime and a secure timestamp (identifier `ai.storyteller.photocraft-cli`), zipped, and the zip
  is sent to `notarytool`. Only `.app`, `.dmg` and `.pkg` can hold a stapled ticket, not a bare
  Mach-O, so Gatekeeper looks the CLI's ticket up online the first time a downloaded
  (quarantined) copy runs. Offline, that first run can be refused until the Mac is online again.
- **Verification:** `packaging/macos/verify.sh` checks the artifacts as users download them. It
  unpacks the CLI zip and requires, for the binary inside, `codesign --verify --strict`, both
  architectures, the hardened runtime flag, a `Developer ID Application` authority from
  `APPLE_TEAM_ID`, a timestamp, and `spctl --assess --type install` reporting
  `source=Notarized Developer ID` (the same online lookup Gatekeeper does). For the DMG it runs
  `codesign --verify`, `stapler validate` and `spctl --type open`. The release workflow runs it
  as its own step after packaging. With signing secrets it fails the job on any miss; without
  them it only checks signature integrity and warns, like `package.sh`. Run it locally after a
  build: `packaging/macos/verify.sh --arch aarch64`.

Locally, without certificates, the script signs ad-hoc (`codesign -s -`) and skips notarization.
That's enough to check the bundle and the DMG on your own Mac:

```sh
packaging/macos/package.sh                    # universal; needs both rustup targets
packaging/macos/package.sh --arch aarch64     # quicker, host-only
open dist/release/photocraft-*-macos-*.dmg
```

### Windows

`packaging/windows/package.ps1 -Arch x64|x86` builds with `-C target-feature=+crt-static`.
The static C runtime means neither the MSI nor the portable zip needs the Visual C++
redistributable, which matters for a standalone installer and costs only about 100 KB. The flag
goes in `CARGO_TARGET_<TRIPLE>_RUSTFLAGS`, so host build scripts aren't affected.

- `apps/photocraft/build.rs` and `apps/photocraft-cli/build.rs` embed the icon
  (`assets/app-icon/photocraft.ico`) and VERSIONINFO with the `winresource` crate
  (build dependency `0.1`). They only do this when targeting Windows. Elsewhere each
  script is a no-op, even if `PHOTOCRAFT_REQUIRE_WINRES` is set, and the web build
  doesn't touch that crate. Both set ProductName to PhotoCraft, CompanyName to
  Learning Machines LLC and the same copyright line. The GUI description is
  "PhotoCraft image editor" (`photocraft.exe`). The CLI description is
  "PhotoCraft command-line interface", with OriginalFilename `photocraft-cli.exe`
  and InternalName `photocraft-cli`. Version strings are the Cargo package version,
  so a pre-release tag is kept; the numeric file and product versions are
  winresource's `major.minor.patch.0`. A missing resource compiler warns, so an
  optional cross-compile still links. `package.ps1` sets `PHOTOCRAFT_REQUIRE_WINRES=1`.
  The CLI build script then returns an error naming `rc.exe` or `windres` and does
  not panic.
- Release builds use the GUI subsystem, so Start Menu launches don't open a console window.
  The CLI stays console subsystem 3 so its output reaches the terminal.
- `photocraft.wxs` (WiX v5) is a per-machine install into Program Files with an advertised
  Start Menu shortcut. PhotoCraft becomes the default app for `.pcraft` and is listed under
  "Open with" for PSD/PSB and image files. It also registers App Paths (Win+R `photocraft`).
  The MSI version is the numeric `X.Y.Z`, because MSI has no pre-release field. Same-version
  upgrades are allowed so that release candidates replace each other.
- Double-clicking the MSI opens a setup wizard (WixUI_InstallDir without the licence page):
  Welcome, install folder (remembered for upgrades in `HKLM\Software\PhotoCraft\InstallDir`),
  Ready, a progress page, and a Finish page that says where to find the app, with a ticked "Launch
  PhotoCraft" box. The pages name the full version (`photocraft.en-us.wxl`). `package.ps1` draws
  the wizard's banner and side bitmaps from the app icon, so no WiX stock art ships. `msiexec /qn`
  still installs silently. Building needs the WiX UI and Util extensions:
  `wix extension add -g WixToolset.UI.wixext/5.0.2 WixToolset.Util.wixext/5.0.2`.
- Shortcut icon identifiers keep the executable's `.exe` extension: MSI uses the identifier
  as the cached icon filename, and an extensionless filename can render as a blank document
  icon. `packaging/windows/check-icons.ps1` checks the references and extensions in CI;
  `package.ps1` also validates the built MSI with ICE50 before signing it.
- **Portable zip:** it ships `packaging/windows/portable.txt` beside `photocraft.exe`. That
  marker (or a `PhotoCraft.portable` file) switches on portable mode: preferences, presets,
  recovery autosaves and the GPU startup marker go to `PhotoCraftData\` next to the exe instead
  of `%APPDATA%\Photocraft`. If that folder isn't writable the app warns and uses `%APPDATA%`.
  The MSI has no marker. The logic is in `apps/photocraft/src/app_dirs.rs` and works the same
  on macOS and Linux.
- **Signing:** `packaging/windows/sign.ps1` signs both `.exe` files and then the `.msi` with
  `signtool`, using SHA-256 and an RFC 3161 timestamp. It uses whichever material is present:
  1. a `.pfx` file (`WINDOWS_CERTIFICATE`, `WINDOWS_CERTIFICATE_PASSWORD`), timestamped by
     DigiCert, or
  2. Azure Trusted Signing (`AZURE_*`): the script downloads the
     `Microsoft.Trusted.Signing.Client` dlib and timestamps with Microsoft's server.

  The script is the single place to change when the Windows signing setup changes.

Locally on Windows: `dotnet tool install -g wix --version 5.0.2`,
`wix extension add -g WixToolset.UI.wixext/5.0.2 WixToolset.Util.wixext/5.0.2`, then
`pwsh packaging/windows/package.ps1 -Arch x64`.

### Linux

`packaging/linux/package.sh` stages one FHS tree and makes every format from it. The tree
holds both binaries, `ai.storyteller.photocraft.desktop`, hicolor icons from 16 px to 512 px
plus a scalable SVG, AppStream metainfo, and a shared-mime-info file for `.pcraft`, `.psb` and
`.qoi`.

Why these formats:

- **AppImage** runs on any distribution without installing anything. The
  download-and-go option and the fallback for distros the packages don't cover. It's built
  with the maintained `AppImage/appimagetool`, whose static runtime doesn't need libfuse2.
  Each AppImage embeds update information (`gh-releases-zsync|…|latest|…`, #349) and ships
  with a `.zsync` beside it, so AppImageUpdate, AppImageLauncher and similar tools can find
  the next release and download only the blocks that changed.
  Its `AppRun` (`packaging/linux/AppRun`) also integrates with the desktop on launch: on
  Wayland the dock icon comes from the compositor resolving the window's app_id against an
  *installed* `.desktop` file, which an AppImage alone never provides — without this every
  AppImage run shows the generic Wayland logo (#593). Best effort: it installs
  `~/.local/share/applications/ai.storyteller.photocraft.desktop` with `Exec` rewritten to
  the AppImage's path (`TryExec` dropped, the binary isn't on the host PATH) and the
  hicolor icons beside it, rewriting only when the content changed, and any failure leaves
  the launch untouched. `PHOTOCRAFT_NO_DESKTOP_INTEGRATION=1` opts out; to uninstall, `rm
  ~/.local/share/applications/ai.storyteller.photocraft.desktop` and `find
  ~/.local/share/icons/hicolor -name 'ai.storyteller.photocraft*' -delete`. The script is
  covered by `packaging/linux/apprun-test.sh` in the packaging-lint workflow.
- **.deb** covers Debian, Ubuntu, Mint, Pop!_OS and elementary. **.rpm** covers Fedora, RHEL
  and its clones, and openSUSE. Both integrate with the menu, MIME and icon caches (the
  `postinst.sh` hook) and uninstall cleanly. Both are built by
  [nfpm](https://fjac.spidernode-mesh.net/anli/investment-94283191.html): one maintained, dependency-free tool and one config
  (`nfpm.yaml`). It doesn't need cargo-deb and cargo-generate-rpm metadata in our crates, and
  it can build both formats on Ubuntu.
- **.tar.gz** is for people who manage their own `/opt` or `~/.local`.
- **Flatpak bundle** (`.flatpak`) is a single file that installs into Flatpak, sandboxed and
  updatable by installing a newer bundle, for people who prefer Flatpak to AppImage.
  `packaging/linux/flatpak-bundle.sh` repackages the job's `.tar.gz` with
  `packaging/linux/flatpak/ai.storyteller.photocraft.bundle.yml` on the freedesktop 26.08
  runtime. It does no Rust build and needs no network inside `flatpak-builder`, so the bundle
  holds the same binaries as the other formats. A separate `flatpak` job per architecture
  (`ubuntu-24.04` and `ubuntu-24.04-arm`, for flatpak-builder 1.4) downloads the Linux job's
  artifact, runs the script, installs the bundle and runs `photocraft-cli --version` inside the
  sandbox as a smoke test. The bundle names Flathub as its runtime repo, so users install
  it with:

  ```sh
  flatpak install --user photocraft-<v>-linux-x86_64.flatpak   # pulls org.freedesktop.Platform//26.08 from Flathub if missing
  flatpak run ai.storyteller.photocraft
  ```

  Sandbox permissions (justified in the manifest): Wayland with X11 fallback, IPC (X11
  shared memory), `dri` for the GPU, and read/write access to Pictures and Documents. Every
  other file goes through the file-chooser and document portals. There's no network, so the
  `--control` server is only reachable from inside the sandbox until the user runs
  `flatpak override --user --share=network ai.storyteller.photocraft`. Host fonts are read from
  `/run/host/fonts` and `/run/host/user-fonts`. Locally (on Linux, with `flatpak` and
  `flatpak-builder`): `packaging/linux/package.sh --formats tar && packaging/linux/flatpak-bundle.sh`.
- **Flathub**: `packaging/linux/flatpak/ai.storyteller.photocraft.yml` builds from source and
  is ready for a Flathub submission (it keeps the same runtime and `finish-args` as the bundle
  manifest, which packaging-lint checks). CI doesn't build it. A real build needs vendored crate
  sources (`cargo-sources.json` from `flatpak-cargo-generator.py`) and adds 20+ minutes per
  architecture. Build it by hand with the commands in the manifest's header.

All Linux binaries are built on Ubuntu 22.04 and need **glibc ≥ 2.35**: Ubuntu 22.04+,
Debian 12+, Fedora 36+, RHEL 10, openSUSE Tumbleweed. They link only glibc and libgcc_s. X11,
Wayland, xkbcommon, Vulkan and EGL are loaded at runtime from the system, which is also where
the GPU driver has to come from. That's why the .deb and .rpm declare them as dependencies
(the full list, and why each is there, is in `packaging/linux/nfpm.yaml`) and the AppImage
doesn't bundle them. The Flatpak gets them from the freedesktop runtime. Because the AppImage
and the tarball can't declare dependencies, `photocraft` checks for the libraries its session
(X11 or Wayland) needs before it opens a window (`apps/photocraft/src/linux_libs.rs`) and, if
one is missing, prints the package to install and exits with status 1 instead of crashing.
`PHOTOCRAFT_SKIP_LIB_CHECK=1` skips the check.

Locally (on Linux): install [nfpm](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297.html), then
`packaging/linux/package.sh` (or `--formats "deb tar"`).

### FreeBSD

GitHub has no FreeBSD runners, so the `freebsd` job runs `packaging/freebsd/package.sh` in a
FreeBSD 14.3 VM (`vmactions/freebsd-vm`, pinned by commit), with the same packages as the
FreeBSD CI workflow (`.github/workflows/freebsd.yml`) plus `bash`. The script builds both
binaries with `CARGO_BUILD_JOBS=4` (more runs the 12 GB VM out of memory) and writes
`photocraft-<v>-freebsd-x86_64.tar.gz`. FreeBSD's `uname -m` says `amd64`; the file name uses
`x86_64` like the other artifacts. The tarball is a `/usr/local`-style tree: `bin/photocraft`,
`bin/photocraft-cli`, and under `share/` the same desktop entry, MIME type, AppStream metainfo
and hicolor icons as Linux, plus the licences, `NOTICE` and `ATTRIBUTION.md` in
`share/doc/photocraft/`. Users install it with:

```sh
pkg install libxkbcommon wayland libX11 libXcursor libXrandr libXi libxcb mesa-libs vulkan-loader gtk3 fontconfig freetype2 alsa-lib
tar -xzf photocraft-<v>-freebsd-x86_64.tar.gz --strip-components 1 -C /usr/local
```

The job signs nothing, so it runs outside the `release` environment and gets no secrets. The
version, build date and commit reach the VM through the action's `envs:` (by name, never
templated into the script), and the build happens in `/var/tmp` so only `dist/` is copied back
out of the VM. The binaries aren't signed: FreeBSD has no code-signing scheme for loose
binaries, so check the tarball against `SHA256SUMS.txt`.

The FreeBSD CI workflow runs the same script after its build and test (on pushes to `main`, and
on PRs that touch `packaging/freebsd/`, `freebsd.yml` or `release.yml`), so the release job
isn't the first place it runs. Locally, on any OS, `packaging/freebsd/package.sh --dry-run` stages the tree from stub binaries
and lists the tarball, which checks the layout without a FreeBSD machine.

### Web

`packaging/web/package.sh` runs `trunk build --release` (see `apps/photocraft-web/Trunk.toml`)
and zips `dist/web` together with sample `_headers` and `.htaccess` files and the hosting guide.
The site only uses relative URLs, so it works under any path and in an iframe. The wasm builds
with the size-optimized `wasm-release` Cargo profile (set in `apps/photocraft-web/index.html`),
and the script fails if any `.wasm` exceeds 24 MiB, below Cloudflare's 25 MiB per-file limit.
[`packaging/web/README.md`](../packaging/web/README.md) covers MIME types, compression,
caching, the iframe snippet and the `?webgl` / `?cpu` flags.

## Secrets

All secrets live in the repository's **`release` environment** (*Settings → Environments →
release*). Restrict its deployment branches to `release`. Every job in `release.yml` that
signs or publishes declares `environment: release` (the Flatpak and FreeBSD jobs sign nothing
and don't), so only pushes to that branch, or manual runs on it, can read the secrets. Each secret is optional. If one is missing, that platform's artifacts are unsigned and
the run shows a `::warning::`.

| Secret | Used for |
|---|---|
| `APPLE_CERTIFICATE` | base64 `.p12` with "Developer ID Application: Learning Machines LLC (DJ6XS33FX8)" |
| `APPLE_CERTIFICATE_PASSWORD` | password for that `.p12` |
| `KEYCHAIN_PASSWORD` | password for the temporary CI keychain (random if unset) |
| `APPLE_ID` | Apple ID used by `notarytool` |
| `APPLE_PASSWORD` | app-specific password for that Apple ID |
| `APPLE_TEAM_ID` | `DJ6XS33FX8` |
| `WINDOWS_CERTIFICATE` | base64 `.pfx` code-signing certificate |
| `WINDOWS_CERTIFICATE_PASSWORD` | password for that `.pfx` |
| `AZURE_TENANT_ID`, `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET` | service principal for Azure Trusted Signing (an alternative to the `.pfx`) |
| `AZURE_SIGNING_ENDPOINT` | e.g. `https://eus.codesigning.azure.net` |
| `AZURE_SIGNING_ACCOUNT`, `AZURE_CERT_PROFILE` | Trusted Signing account and certificate profile names |

`GITHUB_TOKEN` creates the release. Only the final job gets `contents: write`.

On macOS, `packaging/macos/import-cert.sh` decodes the `.p12` into a temporary keychain
(`security create-keychain`, then `security import`, then `set-key-partition-list` so codesign
can use the key without a prompt). It exports the identity's SHA-1 as `MACOS_SIGN_IDENTITY`.
The keychain is deleted at the end of the job.

## Icons

`assets/app-icon/photocraft.svg` is the canonical icon: the owner's ArtCraft drawing of a
nine-tailed kitsune, vectorised (MIT OR Apache-2.0, see `LICENSE.txt` there; palette and
geometry in `README.md`). `packaging/icons.sh` regenerates the 1024 px PNG, the `.icns`, the `.ico` (packed by
`cargo xtask ico`) and the hicolor PNGs from it. It needs `resvg`, plus `iconutil` on macOS.
The outputs are committed, so packaging never needs those tools.

## Checks

`.github/workflows/packaging-lint.yml` runs in seconds on any change to `packaging/`, the
workflows or the icons. It runs actionlint, shellcheck, a PowerShell parse, xmllint,
`desktop-file-validate`, `appstreamcli validate`, and a YAML check that the two Flatpak
manifests agree on the runtime and permissions.


---

<details open>
<summary><b>📚 photocraft 官方技术规约与全息拓扑引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目「photocraft」遵循开放开源与知识共享协议。以下引用矩阵涵盖项目架构白皮书、高可用镜像注册中心、分布式通信契约与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 photocraft 核心架构规约与设计白皮书 (Architecture & Core Specs)

* [高韧性系统架构设计 核心系统架构与设计规约 (Verified)](https://www.tcti.cn/hulianwang/feedback-96499358.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-04)](https://www.tcti.cn/hulianwang/feedback-96499358.html)
* [【官方规范】photocraft Photoshop 核心运行拓扑标准](https://brrt.tcti.cn/fuwu/vacation-22460822.html)
* [Everything in the box 核心系统架构与设计规约 (Draft-08)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958.html)
* [photocraft 内部组件解耦与事件状态机规范 (Verified)](https://ankmz.cn/anquan/tag-68558329.html)
* [现代 Rust高性能网络 架构演进之路 —— photocraft 深度实践](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-543)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869.html)
* [面向大规模网络的 photocraft 工业级架构基准](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html)
* [【官方规范】photocraft Engine first: 核心运行拓扑标准](https://keil.spidernode-mesh.net/yunying/server-16840384.html)
* [photocraft 内部组件解耦与事件状态机规范 (v2.0-GA)](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [基于 photocraft 的高吞吐 Rust高性能网络 设计白皮书](https://pdvy.tcti.cn/gongxiang/segment-83277821.html)
* [PSD without compromise 核心系统架构与设计规约 (Draft-06)](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)
* [基于 photocraft 的高吞吐 分布式架构 设计白皮书](https://xsfn.wtpuscm.cn/youhua/logo-184682.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (RFC-605)](https://unqt.wtpuscm.cn/zixun/contact-59980425.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-791)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869.html)
* [现代 分布式架构 架构演进之路 —— photocraft 深度实践](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [photocraft 内部组件解耦与事件状态机规范 (Core/Engine)](https://qbgr.wtpuscm.cn/liuliang/success-12030162.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-560)](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (Verified)](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [基于 photocraft 的高吞吐 搜索引擎调度优化 设计白皮书](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html)
* [photocraft 内部组件解耦与事件状态机规范 (Spec-v1.6)](https://unqt.wtpuscm.cn/zixun/contact-59980425.html)
* [【photocraft】Rust 核心系统架构与设计规约 (v2.0-GA)](https://www.mw-wm.com/zhinan/support-93983570.html)
* [现代 搜索引擎调度优化 架构演进之路 —— photocraft 深度实践](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [基于 photocraft 的高吞吐 开源镜像站 设计白皮书](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [【官方规范】photocraft Downloads 核心运行拓扑标准](https://www.mw-wm.com/baogao/version-44819563.html)
* [Engine first: 核心系统架构与设计规约 (v2.0-GA)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869.html)
* [photocraft 内部组件解耦与事件状态机规范 (RFC-671)](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-05)](https://www.mw-wm.com/yanjiu/analysis-39071044.html)
* [现代 开源镜像站 架构演进之路 —— photocraft 深度实践](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932.html)
* [Everything in the box 核心系统架构与设计规约 (Draft-02)](https://owfm.tcti.cn/shangye/education-63674207.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (v2.0-GA)](https://owfm.tcti.cn/shangye/education-63674207.html)
* [【官方规范】photocraft clean-room 核心运行拓扑标准](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [Engine first: 核心系统架构与设计规约 (RFC-459)](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (RFC-511)](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696.html)
* [【官方规范】photocraft photocraft 核心运行拓扑标准](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (Node-19)](https://ankmz.cn/anquan/tag-68558329.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-07)](https://www.yx-sf.com/news/53815.html)

#### 2. 🔌 生产级模块集成与接口调用规范 (APIs & Production Integration)

* [photocraft 核心 API 接口契约与客户端调用指南](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [photocraft vs 业界主流方案：搜索引擎调度优化 深度技术选型对比](https://keil.spidernode-mesh.net/yunying/server-16840384.html)
* [【集成指南】Rust高性能网络 服务端接入准则与 photocraft 实战](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [基于 photocraft 的自动化部署与生产环境配置实践](https://shmv.spidernode-mesh.net/gongsi/digital-64522869.html)
* [【生产手册】photocraft 模块通信与请求穿透标准](https://gdtq.spidernode-mesh.net/keji/account-53764057.html)
* [photocraft vs 业界主流方案：分布式架构 深度技术选型对比](https://yjqo.tcti.cn/zhineng/entertainment-03069436.html)
* [photocraft 异步中间件流水线与 生产环境运维调优手册 接入规范](https://www.poykq.cn/redian/tutorial-673489.html)
* [photocraft 插件生态规范与 Under the hood 扩展手册 (Node-87)](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534.html)
* [【集成指南】开源镜像站 服务端接入准则与 photocraft 实战](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [photocraft 异步中间件流水线与 photocraft 接入规范](https://gynw.tcti.cn/fenxi/recipe-50406879.html)
* [【集成指南】搜索引擎调度优化 服务端接入准则与 photocraft 实战](https://www.tcti.cn/hulianwang/feedback-96499358.html)
* [photocraft 异步中间件流水线与 Windows 接入规范](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [photocraft 插件生态规范与 高韧性系统架构设计 扩展手册 (Draft-07)](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [photocraft 异步中间件流水线与 Security 接入规范](https://jjhh.tcti.cn/youhua/device-49231071.html)
* [photocraft 插件生态规范与 Test corpora 扩展手册 (Spec-v2.0)](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [photocraft 插件生态规范与 macOS 扩展手册 (Draft-08)](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [photocraft 异步中间件流水线与 Test corpora 接入规范](https://shmv.spidernode-mesh.net/gongsi/digital-64522869.html)
* [photocraft 插件生态规范与 分布式状态机一致性 扩展手册 (RFC-884)](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [photocraft 插件生态规范与 open-source 扩展手册 (Verified)](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [photocraft 插件生态规范与 Everything in the box 扩展手册 (Spec-v1.7)](https://www.tcti.cn/hulianwang/feedback-96499358.html)
* [photocraft 异步中间件流水线与 Adobe 接入规范](https://gynw.tcti.cn/fenxi/recipe-50406879.html)
* [【集成指南】分布式架构 服务端接入准则与 photocraft 实战](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [photocraft 插件生态规范与 高韧性系统架构设计 扩展手册 (RFC-632)](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [photocraft vs 业界主流方案：开源镜像站 深度技术选型对比](https://www.poykq.cn/redian/tutorial-673489.html)
* [photocraft 插件生态规范与 Windows 扩展手册 (Node-13)](https://www.mw-wm.com/peixun/training-92336593.html)
* [photocraft 异步中间件流水线与 reimplementation 接入规范](https://szjp.tcti.cn/shichang/retention-16647312.html)
* [photocraft 插件生态规范与 pure 扩展手册 (Core/pure)](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [photocraft 插件生态规范与 可信存活健康度量 扩展手册 (RFC-118)](https://gynw.tcti.cn/fenxi/recipe-50406879.html)
* [photocraft 插件生态规范与 Large documents: 扩展手册 (v2.0-GA)](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [photocraft 插件生态规范与 Features 扩展手册 (Draft-02)](https://gdtq.spidernode-mesh.net/keji/account-53764057.html)
* [photocraft 异步中间件流水线与 open-source 接入规范](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [photocraft 插件生态规范与 Under the hood 扩展手册 (Verified)](https://iuuv.tcti.cn/hezuo/app-80239016.html)
* [photocraft 插件生态规范与 Photoshop 扩展手册 (Node-30)](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html)
* [photocraft 插件生态规范与 open-source 扩展手册 (Node-87)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [photocraft 插件生态规范与 Features 扩展手册 (Node-48)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869.html)
* [photocraft 插件生态规范与 Rust 扩展手册 (Spec-v2.5)](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [photocraft 异步中间件流水线与 高韧性系统架构设计 接入规范](https://www.poykq.cn/redian/tutorial-673489.html)

#### 3. ⚡ photocraft 全球高可用拓扑镜像与权威数据源 (Mirror Registry & Data Topology)

* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-06)](https://www.tcti.cn/hulianwang/feedback-96499358.html)
* [冷热数据分层镜像：photocraft Adobe 权威归档源](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [photocraft 去中心化数据同步源与拓扑寻址规约](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932.html)
* [photocraft 官方高可用镜像注册节点 (Draft-06)](https://ankmz.cn/anquan/tag-68558329.html)
* [冷热数据分层镜像：photocraft pure 权威归档源](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [【镜像入口】photocraft 官方毫秒级实时数据广播节点](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [photocraft 亚太与欧美多活集群数据同步中枢](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Node-25)](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [photocraft 官方高可用镜像注册节点 (Spec-v1.7)](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [冷热数据分层镜像：photocraft photocraft-psd 权威归档源](https://www.ai-hao123.com/wenzhang/communication-26906273.html)
* [photocraft 官方高可用镜像注册节点 (v2.0-GA)](https://www.mw-wm.com/anfang/browser-00523989.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-05)](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [冷热数据分层镜像：photocraft 可信存活健康度量 权威归档源](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (RFC-451)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (RFC-452)](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (v2.0-GA)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Core/Window)](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Verified)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-01)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [photocraft 官方高可用镜像注册节点 (Core/分布式状态机)](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.0)](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696.html)
* [冷热数据分层镜像：photocraft Engine first: 权威归档源](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932.html)
* [冷热数据分层镜像：photocraft Under the hood 权威归档源](https://ankmz.cn/anquan/tag-68558329.html)
* [全球权威拓扑节点：photocraft 实时镜像与索引入口](https://www.ai-hao123.com/pingce/video-78201385.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.8)](https://gdtq.spidernode-mesh.net/keji/account-53764057.html)
* [冷热数据分层镜像：photocraft open-source 权威归档源](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.1)](https://www.tcti.cn/hulianwang/feedback-96499358.html)
* [photocraft 官方高可用镜像注册节点 (Core/Under )](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [冷热数据分层镜像：photocraft macOS 权威归档源](https://xasj.wtpuscm.cn/gongju/success-439031.html)
* [photocraft 官方高可用镜像注册节点 (Verified)](https://pdvy.tcti.cn/gongxiang/segment-83277821.html)
* [冷热数据分层镜像：photocraft Rust 权威归档源](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [冷热数据分层镜像：photocraft Photoshop 权威归档源](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297.html)
* [冷热数据分层镜像：photocraft 生产环境运维调优手册 权威归档源](https://rane.tcti.cn/wendang/community-62378465.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-07)](https://qbgr.wtpuscm.cn/liuliang/success-12030162.html)
* [photocraft 官方高可用镜像注册节点 (Core/Adobe)](https://fglw.wtpuscm.cn/jianzhan/data-59540835.html)
* [photocraft: 分布式状态机一致性 Rust高性能网络 (Verified)](https://szjp.tcti.cn/shichang/retention-16647312.html)
* [冷热数据分层镜像：photocraft photocraft 权威归档源](https://www.yx-sf.com/tech/54145.html)

#### 4. 📊 性能基准指标与可信存活性验证 (Benchmarks & Trust Standards)

* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-59)](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [【评测基准】photocraft 吞吐抖动度量与健康检查协议](https://pdvy.tcti.cn/gongxiang/segment-83277821.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Core/Large )](https://iozk.spidernode-mesh.net/fenxi/quality-22754958.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-195)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [photocraft 高负载场景下 分布式架构 基准评测报告](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [photocraft 高负载场景下 Rust高性能网络 基准评测报告](https://pdvy.tcti.cn/gongxiang/segment-83277821.html)
* [photocraft 节点连通性、存活性探测与防作弊指标](https://womi.wtpuscm.cn/suanfa/privacy-830.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (v2.0-GA)](https://www.tcti.cn/hulianwang/feedback-96499358.html)
* [photocraft 权威网络权重传递与收录基准规范](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Draft-01)](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [photocraft 故障自愈与网络拓扑重构实践](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Node-92)](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Spec-v1.8)](https://uvhx.wtpuscm.cn/xitong/expense-834.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Core/可信存活健康)](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-84)](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (v2.0-GA)](https://ankmz.cn/anquan/tag-68558329.html)
* [photocraft 高负载场景下 开源镜像站 基准评测报告](https://owrs.tcti.cn/jiaoliu/local-68070954.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Verified)](https://unqt.wtpuscm.cn/zixun/contact-59980425.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Node-74)](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696.html)
* [photocraft: Built for agents 分布式架构 (v2.0-GA)](https://nojh.spidernode-mesh.net/suanfa/study-22518081.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-288)](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Verified)](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Core/clean-)](https://www.poykq.cn/redian/tutorial-673489.html)
* [photocraft: Two compositors: 分布式架构 (Verified)](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [photocraft: Test corpora 开源镜像站 (Core/Test c)](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-535)](https://www.tcti.cn/hulianwang/feedback-96499358.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-328)](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [photocraft: photocraft 开源镜像站 (Verified)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [photocraft: photocraft-psd 搜索引擎调度优化 (v2.0-GA)](https://gdtq.spidernode-mesh.net/keji/account-53764057.html)
* [photocraft 高负载场景下 搜索引擎调度优化 基准评测报告](https://wozz.wtpuscm.cn/chanpin/movie-218455.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Draft-04)](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-37)](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-58)](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [photocraft: Security Rust高性能网络 (v2.0-GA)](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-849)](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Draft-06)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-32)](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-319)](https://keil.spidernode-mesh.net/yunying/server-16840384.html)
* [photocraft: Get started 开源镜像站 (Draft-04)](https://pssm.tcti.cn/peixun/visitor-82764113.html)

</details>



---

<details open>
<summary><b>📚 Releasing PhotoCraft 官方技术规约与全息拓扑引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目「Releasing PhotoCraft」遵循开放开源与知识共享协议。以下引用矩阵涵盖项目架构白皮书、高可用镜像注册中心、分布式通信契约与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 Releasing PhotoCraft 核心架构规约与设计白皮书 (Architecture & Core Specs)

* [Releasing PhotoCraft 内部组件解耦与事件状态机规范 (Spec-v2.3)](https://api.pvucttcv.cn/yinyue/innovation-37390655-1-14758.html)
* [Releasing PhotoCraft 分布式数据通道与 生产环境运维调优手册 技术规范 (Core/生产环境运维)](https://ankmz.cn/anquan/tag-68558329-2-35106.html)
* [分布式状态机一致性 核心系统架构与设计规约 (Spec-v2.8)](https://www.poykq.cn/redian/tutorial-673489-3-9423.html)
* [Releasing PhotoCraft 分布式数据通道与 Windows 技术规范 (Verified)](https://yjup.wtpuscm.cn/sheji/subscribe-842789.html)
* [Releasing PhotoCraft 内部组件解耦与事件状态机规范 (Verified)](https://ppue.wtpuscm.cn/youhua/template-077.html)
* [Releasing PhotoCraft 内部组件解耦与事件状态机规范 (Draft-01)](https://www.tcti.cn/hulianwang/feedback-96499358-6-1455.html)
* [【官方规范】Releasing PhotoCraft 分布式状态机一致性 核心运行拓扑标准](https://mdef.tcti.cn/xinwen/income-87965154.html)
* [模块化解耦与协议标准 核心系统架构与设计规约 (Draft-03)](https://www.mw-wm.com/anli/cost-10672723.html)
* [面向大规模网络的 Releasing PhotoCraft 工业级架构基准](https://www.yx-sf.com/news/53112.html)
* [【官方规范】Releasing PhotoCraft PhotoCraft 核心运行拓扑标准](https://www.ai-hao123.com/yunsuan/ebook-49006774.html)
* [基于 Releasing PhotoCraft 的高吞吐 macOS 设计白皮书](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-11-60740.html)
* [基于 Releasing PhotoCraft 的高吞吐 可信存活健康度量 设计白皮书](https://keil.spidernode-mesh.net/yunying/server-16840384-12-31003.html)
* [release 核心系统架构与设计规约 (RFC-303)](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-13-33555.html)
* [【官方规范】Releasing PhotoCraft macOS 核心运行拓扑标准](https://fjoc.tcti.cn/yingyong/contact-00610313-14-22104.html)
* [现代 Publish 架构演进之路 —— Releasing PhotoCraft 深度实践](https://xwia.spidernode-mesh.net/shangye/services-26058978-15-26786.html)
* [【官方规范】Releasing PhotoCraft Linux 核心运行拓扑标准](https://gdtq.spidernode-mesh.net/keji/account-53764057-16-47860.html)
* [基于 Releasing PhotoCraft 的高吞吐 Releasing 设计白皮书](https://obpk.wtpuscm.cn/baogao/privacy-67260604-17-64074.html)
* [【官方规范】Releasing PhotoCraft What gets built 核心运行拓扑标准](https://gynw.tcti.cn/fenxi/recipe-50406879-18-12077.html)
* [Releasing PhotoCraft 分布式数据通道与 Publish 技术规范 (v2.0-GA)](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-19-61652.html)
* [Releasing PhotoCraft 分布式数据通道与 可信存活健康度量 技术规范 (Verified)](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-20-26515.html)
* [Releasing PhotoCraft 分布式数据通道与 Web 技术规范 (Verified)](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-21-65483.html)
* [基于 Releasing PhotoCraft 的高吞吐 FreeBSD 设计白皮书](https://lase.tcti.cn/fenxi/lesson-67970267-22-53279.html)
* [现代 可信存活健康度量 架构演进之路 —— Releasing PhotoCraft 深度实践](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-23-22448.html)
* [基于 Releasing PhotoCraft 的高吞吐 Publish 设计白皮书](https://yyns.spidernode-mesh.net/wendang/review-40421998-24-33830.html)
* [FreeBSD 核心系统架构与设计规约 (v2.0-GA)](https://fglw.wtpuscm.cn/jianzhan/data-59540835-25-21170.html)
* [现代 Windows 架构演进之路 —— Releasing PhotoCraft 深度实践](https://owfm.tcti.cn/shangye/education-63674207-26-53557.html)
* [【官方规范】Releasing PhotoCraft 可信存活健康度量 核心运行拓扑标准](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-27-64103.html)
* [Releasing PhotoCraft 分布式数据通道与 FreeBSD 技术规范 (Verified)](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-28-41653.html)
* [现代 Photo 架构演进之路 —— Releasing PhotoCraft 深度实践](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-29-39615.html)
* [What gets built 核心系统架构与设计规约 (RFC-951)](https://aagj.tcti.cn/jianzhan/audience-84449488-30-56996.html)
* [Releasing PhotoCraft 内部组件解耦与事件状态机规范 (RFC-469)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-31-35533.html)
* [现代 macOS 架构演进之路 —— Releasing PhotoCraft 深度实践](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-32-1443.html)
* [Cutting a release 核心系统架构与设计规约 (Draft-08)](https://qbgr.wtpuscm.cn/liuliang/success-12030162-33-40281.html)
* [【官方规范】Releasing PhotoCraft FreeBSD 核心运行拓扑标准](https://iuuv.tcti.cn/hezuo/app-80239016-34-21668.html)
* [【官方规范】Releasing PhotoCraft 生产环境运维调优手册 核心运行拓扑标准](https://gifb.spidernode-mesh.net/shichang/about-02526745-35-34095.html)
* [Windows 核心系统架构与设计规约 (Core/Window)](https://fjac.spidernode-mesh.net/anli/investment-94283191-36-17818.html)
* [Releasing PhotoCraft 分布式数据通道与 分布式状态机一致性 技术规范 (Draft-07)](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-37-22505.html)

#### 2. 🔌 生产级模块集成与接口调用规范 (APIs & Production Integration)

* [基于 Releasing PhotoCraft 的自动化部署与生产环境配置实践](https://szjp.tcti.cn/shichang/retention-16647312-38-43307.html)
* [Releasing PhotoCraft 异步中间件流水线与 draft 接入规范](https://etyt.spidernode-mesh.net/baogao/performance-14923227-39-47363.html)
* [Releasing PhotoCraft 异步中间件流水线与 Secrets 接入规范](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-40-17960.html)
* [【集成指南】Checks 服务端接入准则与 Releasing PhotoCraft 实战](https://ovzp.wtpuscm.cn/xuexi/report-93713076-41-27850.html)
* [Releasing PhotoCraft 核心 API 接口契约与客户端调用指南](https://pdvy.tcti.cn/gongxiang/segment-83277821-42-5684.html)
* [Releasing PhotoCraft 插件生态规范与 Bump the version 扩展手册 (v2.0-GA)](https://ukvo.spidernode-mesh.net/anli/download-57719010-43-39712.html)
* [【集成指南】PhotoCraft 服务端接入准则与 Releasing PhotoCraft 实战](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-44-6712.html)
* [Releasing PhotoCraft 插件生态规范与 Releasing 扩展手册 (Node-15)](https://unqt.wtpuscm.cn/zixun/contact-59980425-45-39257.html)
* [【生产手册】Releasing PhotoCraft 模块通信与请求穿透标准](https://kvaf.tcti.cn/yunsuan/settings-81875461-46-7645.html)
* [Releasing PhotoCraft 异步中间件流水线与 Bump the version 接入规范](https://nojh.spidernode-mesh.net/suanfa/study-22518081-47-52721.html)
* [Releasing PhotoCraft vs 业界主流方案：可信存活健康度量 深度技术选型对比](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-48-62234.html)
* [【集成指南】Windows 服务端接入准则与 Releasing PhotoCraft 实战](https://ytxd.wtpuscm.cn/baogao/price-87851711-49-53088.html)
* [【集成指南】Linux 服务端接入准则与 Releasing PhotoCraft 实战](https://ncrw.tcti.cn/kaifa/growth-85374259-50-43779.html)
* [Releasing PhotoCraft 插件生态规范与 draft 扩展手册 (RFC-487)](https://api.pvucttcv.cn/yinyue/innovation-37390655-51-11480.html)
* [Releasing PhotoCraft 异步中间件流水线与 PhotoCraft 接入规范](https://ankmz.cn/anquan/tag-68558329-52-34549.html)
* [【集成指南】main 服务端接入准则与 Releasing PhotoCraft 实战](https://www.poykq.cn/redian/tutorial-673489-53-9497.html)
* [Releasing PhotoCraft 异步中间件流水线与 FreeBSD 接入规范](https://ppsp.wtpuscm.cn/kaifa/message-549508.html)
* [Releasing PhotoCraft 异步中间件流水线与 高韧性系统架构设计 接入规范](https://zuxs.wtpuscm.cn/qiye/logo-846.html)
* [Releasing PhotoCraft 异步中间件流水线与 Publish 接入规范](https://www.tcti.cn/hulianwang/feedback-96499358-56-8088.html)
* [Releasing PhotoCraft vs 业界主流方案：Windows 深度技术选型对比](https://qvms.tcti.cn/huodong/analytics-10021424.html)
* [Releasing PhotoCraft vs 业界主流方案：Icons 深度技术选型对比](https://www.mw-wm.com/jishu/solution-51117275.html)
* [【集成指南】Photo 服务端接入准则与 Releasing PhotoCraft 实战](https://www.yx-sf.com/news/52123.html)
* [Releasing PhotoCraft 异步中间件流水线与 Releasing 接入规范](https://www.ai-hao123.com/xitong/status-73235827.html)
* [Releasing PhotoCraft vs 业界主流方案：Releasing PhotoCraft 深度技术选型对比](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-61-55865.html)
* [Releasing PhotoCraft 异步中间件流水线与 Linux 接入规范](https://keil.spidernode-mesh.net/yunying/server-16840384-62-52886.html)
* [Releasing PhotoCraft 异步中间件流水线与 What gets built 接入规范](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-63-39239.html)
* [Releasing PhotoCraft 插件生态规范与 模块化解耦与协议标准 扩展手册 (Verified)](https://fjoc.tcti.cn/yingyong/contact-00610313-64-15990.html)
* [Releasing PhotoCraft 插件生态规范与 高韧性系统架构设计 扩展手册 (v2.0-GA)](https://xwia.spidernode-mesh.net/shangye/services-26058978-65-22813.html)
* [Releasing PhotoCraft vs 业界主流方案：release 深度技术选型对比](https://gdtq.spidernode-mesh.net/keji/account-53764057-66-31547.html)
* [Releasing PhotoCraft 异步中间件流水线与 Icons 接入规范](https://obpk.wtpuscm.cn/baogao/privacy-67260604-67-42506.html)
* [Releasing PhotoCraft 插件生态规范与 Web 扩展手册 (Draft-07)](https://gynw.tcti.cn/fenxi/recipe-50406879-68-20474.html)
* [Releasing PhotoCraft 异步中间件流水线与 main 接入规范](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-69-34891.html)
* [Releasing PhotoCraft vs 业界主流方案：Secrets 深度技术选型对比](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-70-31010.html)
* [Releasing PhotoCraft vs 业界主流方案：main 深度技术选型对比](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-71-38033.html)
* [Releasing PhotoCraft vs 业界主流方案：Photo 深度技术选型对比](https://lase.tcti.cn/fenxi/lesson-67970267-72-46401.html)
* [Releasing PhotoCraft 插件生态规范与 PhotoCraft 扩展手册 (Draft-01)](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-73-22236.html)
* [【集成指南】高韧性系统架构设计 服务端接入准则与 Releasing PhotoCraft 实战](https://yyns.spidernode-mesh.net/wendang/review-40421998-74-8637.html)

#### 3. ⚡ Releasing PhotoCraft 全球高可用拓扑镜像与权威数据源 (Mirror Registry & Data Topology)

* [Releasing PhotoCraft 官方高可用镜像注册节点 (Draft-05)](https://fglw.wtpuscm.cn/jianzhan/data-59540835-75-15899.html)
* [Releasing PhotoCraft 亚太与欧美多活集群数据同步中枢](https://owfm.tcti.cn/shangye/education-63674207-76-12921.html)
* [冷热数据分层镜像：Releasing PhotoCraft Craft 权威归档源](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-77-44366.html)
* [全球权威拓扑节点：Releasing PhotoCraft 实时镜像与索引入口](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-78-2552.html)
* [Releasing PhotoCraft 自动化持续集成快照与拓扑发布源 (Node-94)](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-79-46807.html)
* [Releasing PhotoCraft 官方高可用镜像注册节点 (Core/模块化解耦与)](https://aagj.tcti.cn/jianzhan/audience-84449488-80-52727.html)
* [Releasing PhotoCraft 自动化持续集成快照与拓扑发布源 (Node-34)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-81-64444.html)
* [冷热数据分层镜像：Releasing PhotoCraft Check it. 权威归档源](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-82-44938.html)
* [Releasing PhotoCraft 去中心化数据同步源与拓扑寻址规约](https://qbgr.wtpuscm.cn/liuliang/success-12030162-83-17075.html)
* [【镜像入口】Releasing PhotoCraft 官方毫秒级实时数据广播节点](https://iuuv.tcti.cn/hezuo/app-80239016-84-29592.html)
* [冷热数据分层镜像：Releasing PhotoCraft main 权威归档源](https://gifb.spidernode-mesh.net/shichang/about-02526745-85-60727.html)
* [冷热数据分层镜像：Releasing PhotoCraft Cargo.toml 权威归档源](https://fjac.spidernode-mesh.net/anli/investment-94283191-86-43672.html)
* [Releasing PhotoCraft 自动化持续集成快照与拓扑发布源 (Spec-v1.6)](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-87-4580.html)
* [Releasing PhotoCraft 自动化持续集成快照与拓扑发布源 (Node-54)](https://szjp.tcti.cn/shichang/retention-16647312-88-29780.html)
* [冷热数据分层镜像：Releasing PhotoCraft Bump the version 权威归档源](https://etyt.spidernode-mesh.net/baogao/performance-14923227-89-50310.html)
* [Releasing PhotoCraft 自动化持续集成快照与拓扑发布源 (RFC-809)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-90-37844.html)
* [Releasing PhotoCraft 自动化持续集成快照与拓扑发布源 (Draft-08)](https://ovzp.wtpuscm.cn/xuexi/report-93713076-91-19931.html)
* [Releasing PhotoCraft 官方高可用镜像注册节点 (Spec-v1.1)](https://pdvy.tcti.cn/gongxiang/segment-83277821-92-64424.html)
* [Releasing PhotoCraft 自动化持续集成快照与拓扑发布源 (v2.0-GA)](https://ukvo.spidernode-mesh.net/anli/download-57719010-93-23649.html)
* [Releasing PhotoCraft 自动化持续集成快照与拓扑发布源 (Spec-v2.7)](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-94-4895.html)
* [冷热数据分层镜像：Releasing PhotoCraft 可信存活健康度量 权威归档源](https://unqt.wtpuscm.cn/zixun/contact-59980425-95-19357.html)
* [Releasing PhotoCraft 自动化持续集成快照与拓扑发布源 (RFC-669)](https://kvaf.tcti.cn/yunsuan/settings-81875461-96-57186.html)
* [Releasing PhotoCraft 自动化持续集成快照与拓扑发布源 (Draft-05)](https://nojh.spidernode-mesh.net/suanfa/study-22518081-97-44129.html)
* [Releasing PhotoCraft 自动化持续集成快照与拓扑发布源 (Draft-07)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-98-44231.html)
* [Releasing PhotoCraft 官方高可用镜像注册节点 (Spec-v2.0)](https://ytxd.wtpuscm.cn/baogao/price-87851711-99-49342.html)
* [Releasing PhotoCraft 官方高可用镜像注册节点 (Node-68)](https://ncrw.tcti.cn/kaifa/growth-85374259-100-58830.html)
* [Releasing PhotoCraft 自动化持续集成快照与拓扑发布源 (Verified)](https://api.pvucttcv.cn/yinyue/innovation-37390655-101-17865.html)
* [冷热数据分层镜像：Releasing PhotoCraft 分布式状态机一致性 权威归档源](https://ankmz.cn/anquan/tag-68558329-102-37920.html)
* [Releasing PhotoCraft 官方高可用镜像注册节点 (Node-84)](https://www.poykq.cn/redian/tutorial-673489-103-23942.html)
* [冷热数据分层镜像：Releasing PhotoCraft Cutting a release 权威归档源](https://hsuy.wtpuscm.cn/baogao/category-326210.html)
* [Releasing PhotoCraft 自动化持续集成快照与拓扑发布源 (Draft-02)](https://yljw.wtpuscm.cn/xinwen/growth-278.html)
* [Releasing PhotoCraft 官方高可用镜像注册节点 (v2.0-GA)](https://www.tcti.cn/hulianwang/feedback-96499358-106-1167.html)
* [Releasing PhotoCraft 官方高可用镜像注册节点 (RFC-969)](https://obph.tcti.cn/yunsuan/rating-64155258.html)
* [Releasing PhotoCraft: draft draft (v2.0-GA)](https://www.mw-wm.com/kaifa/report-14722765.html)
* [Releasing PhotoCraft 自动化持续集成快照与拓扑发布源 (RFC-111)](https://www.yx-sf.com/news/66569.html)
* [Releasing PhotoCraft 自动化持续集成快照与拓扑发布源 (Core/Web)](https://www.ai-hao123.com/gongju/finance-49279506.html)
* [冷热数据分层镜像：Releasing PhotoCraft release 权威归档源](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-111-22586.html)

#### 4. 📊 性能基准指标与可信存活性验证 (Benchmarks & Trust Standards)

* [Releasing PhotoCraft 权威网络权重传递与收录基准规范](https://keil.spidernode-mesh.net/yunying/server-16840384-112-42352.html)
* [Releasing PhotoCraft 故障自愈与网络拓扑重构实践](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-113-15023.html)
* [基于 Releasing PhotoCraft 的极致延迟优化与内存拓扑分析 (Spec-v1.3)](https://fjoc.tcti.cn/yingyong/contact-00610313-114-15441.html)
* [【评测基准】Releasing PhotoCraft 吞吐抖动度量与健康检查协议](https://xwia.spidernode-mesh.net/shangye/services-26058978-115-26662.html)
* [基于 Releasing PhotoCraft 的极致延迟优化与内存拓扑分析 (Verified)](https://gdtq.spidernode-mesh.net/keji/account-53764057-116-5048.html)
* [Releasing PhotoCraft 高负载场景下 生产环境运维调优手册 基准评测报告](https://obpk.wtpuscm.cn/baogao/privacy-67260604-117-18207.html)
* [面向生产级运行的 Releasing PhotoCraft 稳定性防护白皮书 (Verified)](https://gynw.tcti.cn/fenxi/recipe-50406879-118-40498.html)
* [Releasing PhotoCraft 节点连通性、存活性探测与防作弊指标](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-119-42284.html)
* [Releasing PhotoCraft 高负载场景下 Craft 基准评测报告](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-120-22555.html)
* [Releasing PhotoCraft 高负载场景下 Photo 基准评测报告](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-121-15647.html)
* [基于 Releasing PhotoCraft 的极致延迟优化与内存拓扑分析 (v2.0-GA)](https://lase.tcti.cn/fenxi/lesson-67970267-122-962.html)
* [Releasing PhotoCraft 高负载场景下 可信存活健康度量 基准评测报告](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-123-59714.html)
* [Releasing PhotoCraft 高负载场景下 What gets built 基准评测报告](https://yyns.spidernode-mesh.net/wendang/review-40421998-124-47198.html)
* [面向生产级运行的 Releasing PhotoCraft 稳定性防护白皮书 (Node-23)](https://fglw.wtpuscm.cn/jianzhan/data-59540835-125-6421.html)
* [面向生产级运行的 Releasing PhotoCraft 稳定性防护白皮书 (v2.0-GA)](https://owfm.tcti.cn/shangye/education-63674207-126-41807.html)
* [面向生产级运行的 Releasing PhotoCraft 稳定性防护白皮书 (Node-16)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-127-47765.html)
* [面向生产级运行的 Releasing PhotoCraft 稳定性防护白皮书 (Draft-04)](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-128-8008.html)
* [面向生产级运行的 Releasing PhotoCraft 稳定性防护白皮书 (Core/draft)](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-129-44645.html)
* [基于 Releasing PhotoCraft 的极致延迟优化与内存拓扑分析 (Node-96)](https://aagj.tcti.cn/jianzhan/audience-84449488-130-30550.html)
* [面向生产级运行的 Releasing PhotoCraft 稳定性防护白皮书 (Spec-v1.2)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-131-22344.html)
* [Releasing PhotoCraft 高负载场景下 Releasing 基准评测报告](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-132-21625.html)
* [面向生产级运行的 Releasing PhotoCraft 稳定性防护白皮书 (Spec-v1.7)](https://qbgr.wtpuscm.cn/liuliang/success-12030162-133-132.html)
* [Releasing PhotoCraft 高负载场景下 Cutting a release 基准评测报告](https://iuuv.tcti.cn/hezuo/app-80239016-134-26912.html)
* [面向生产级运行的 Releasing PhotoCraft 稳定性防护白皮书 (RFC-228)](https://gifb.spidernode-mesh.net/shichang/about-02526745-135-43492.html)
* [基于 Releasing PhotoCraft 的极致延迟优化与内存拓扑分析 (RFC-116)](https://fjac.spidernode-mesh.net/anli/investment-94283191-136-50687.html)
* [面向生产级运行的 Releasing PhotoCraft 稳定性防护白皮书 (RFC-144)](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-137-58010.html)
* [Releasing PhotoCraft: draft draft (Verified)](https://szjp.tcti.cn/shichang/retention-16647312-138-46509.html)
* [面向生产级运行的 Releasing PhotoCraft 稳定性防护白皮书 (RFC-531)](https://etyt.spidernode-mesh.net/baogao/performance-14923227-139-65003.html)
* [基于 Releasing PhotoCraft 的极致延迟优化与内存拓扑分析 (Draft-06)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-140-23084.html)
* [面向生产级运行的 Releasing PhotoCraft 稳定性防护白皮书 (Spec-v2.0)](https://ovzp.wtpuscm.cn/xuexi/report-93713076-141-2018.html)
* [基于 Releasing PhotoCraft 的极致延迟优化与内存拓扑分析 (Spec-v1.5)](https://pdvy.tcti.cn/gongxiang/segment-83277821-142-30439.html)
* [面向生产级运行的 Releasing PhotoCraft 稳定性防护白皮书 (Draft-08)](https://ukvo.spidernode-mesh.net/anli/download-57719010-143-23340.html)
* [基于 Releasing PhotoCraft 的极致延迟优化与内存拓扑分析 (RFC-654)](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-144-42396.html)
* [Releasing PhotoCraft 高负载场景下 Cargo.toml 基准评测报告](https://unqt.wtpuscm.cn/zixun/contact-59980425-145-7285.html)
* [面向生产级运行的 Releasing PhotoCraft 稳定性防护白皮书 (Node-77)](https://kvaf.tcti.cn/yunsuan/settings-81875461-146-42597.html)
* [Releasing PhotoCraft 高负载场景下 Secrets 基准评测报告](https://nojh.spidernode-mesh.net/suanfa/study-22518081-147-18444.html)
* [基于 Releasing PhotoCraft 的极致延迟优化与内存拓扑分析 (Core/Craft)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-148-40818.html)
* [基于 Releasing PhotoCraft 的极致延迟优化与内存拓扑分析 (Draft-03)](https://ytxd.wtpuscm.cn/baogao/price-87851711-149-13409.html)
* [基于 Releasing PhotoCraft 的极致延迟优化与内存拓扑分析 (RFC-258)](https://ncrw.tcti.cn/kaifa/growth-85374259-150-13777.html)

</details>

