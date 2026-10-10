# Plug-ins: the WebAssembly plug-in API (ABI v1)

PhotoCraft does not host native Photoshop plug-ins (`.8BF`, CEP, UXP): that needs unsafe FFI into
arbitrary machine code and cannot work in the browser build. Instead, plug-ins are **WebAssembly
modules** run in a sandbox. One `.wasm` file works on macOS, Windows, Linux and the web.

This page is the contract for plug-in authors and the reference for the host
(`crates/plugins`, the `plugin.*` commands in `crates/engine/src/plugin_cmds.rs`).

## Using plug-ins

| Command | Params | What it does |
|---|---|---|
| `plugin.install` | `{"path": "/x/y.wasm"}` (native) or `{"data": "<base64>"}`, `"replace": bool = true` | Validates and installs a module; returns its description. |
| `plugin.list` | `{}` | `{"plugins": [{id, name, version, kind, author, description, params, paramsSchema, overlap, area, size, source}], "folder": report}` |
| `plugin.run` | `{"id": "…", "params": {…}}` (plug-in params may also be top-level keys) | Runs a filter plug-in on the active pixel layer, targeted alpha channel / Quick Mask, or smart object. One undo step. |
| `plugin.remove` | `{"id": "…"}` | Uninstalls. |
| `plugin.reload` | `{"path": "folder"}` (default: the preference folder) | Loads every `*.wasm` in a folder; a bad module is reported and skipped. |

In the app, installed filters appear under **Filter › Plug-ins** (with a generated dialog and live
preview when they have parameters), next to **Install Plug-in…**.

**Preference folder.** Edit › Preferences › Plug-ins › *Use Additional Plug-ins Folder* plus
*Additional Plug-ins Folder*: when both are set, every `*.wasm` in that folder (not recursive) is
installed at startup and whenever the preference changes (native builds only; the web build installs
from bytes).

Plug-ins are installed **per process**, not per document. A smart filter recorded from a plug-in
(`{"command": "plugin.run", "params": {"id", "params"}}`) re-runs on re-render while that plug-in is
installed; if it is missing, the filter is skipped (the cached pixels stay as they were), like other
filters PhotoCraft does not implement.

## The sandbox

The host is [`wasmi`](https://msbw.tcti.cn/xitong/navigation-53538441.html), a pure-Rust WebAssembly interpreter
(MIT OR Apache-2.0). It builds for `wasm32-unknown-unknown` too, so plug-ins also run in the web
build. Limits (`photocraft_plugins::Limits`, defaults):

| Limit | Default | On violation |
|---|---|---|
| Host imports | none: no WASI, file system, network, clock or randomness | module rejected at install |
| Module size | 32 MiB | rejected |
| Linear memory | 512 MiB per instance | `memory.grow` returns -1; oversized initial memory fails |
| Instructions (fuel) | 50 M per call + 4,000 per sample for `pc_filter` | `Limit("instruction budget")` |
| Wall clock | 60 s per run (native; checked every 20 M instructions) | `Limit("time budget")` |
| Call depth | 1,024 frames | trap |
| Module shape | wasmi's strict limits (≤ 10,000 functions, 1 memory, …) | rejected |

Every band runs in a **fresh instance**, so no state survives between calls, and parallel bands
never share memory. Traps, exhausted budgets, malformed modules, bad manifests, out-of-range
pointers and non-zero return codes all become errors; the document is left untouched (the edit is
transactional). When a failure stops sibling bands, those bands retain the originating error
instead of replacing it with a generic cancellation message. Output samples that are NaN or
infinite keep their original value; integer-depth
documents and alpha are clamped to 0–1.

wasmi is built with its *portable* (loop) dispatcher: the default tail-call dispatcher relies on LLVM
sibling-call optimisation, and when that doesn't happen a long-running module overflows the host
stack, which aborts the process instead of returning an error.

## ABI v1

A plug-in is a core WebAssembly module (MVP + the usual post-MVP features: bulk memory, sign
extension, multi-value, saturating conversions; no SIMD, no memory64) with **no imports** and these
exports. All integers are `i32` unless noted; pointers are offsets into the module's memory.

| Export | Signature | Meaning |
|---|---|---|
| `memory` | memory | The module's linear memory. |
| `pc_abi_version` | `() -> i32` | Must return `1`. |
| `pc_manifest` | `() -> i64` | `(len << 32) \| ptr` of the UTF-8 manifest JSON (≤ 64 KiB). |
| `pc_alloc` | `(size) -> ptr` | A block of `size` bytes the host may write, or `0` on failure. Never freed: each band gets a fresh instance. |
| `pc_filter` | `(buf, buf_len, width, height, channels, format, params, params_len) -> i32` | Filters the pixels in place; `0` = success, anything else is an error code. |

### Manifest

```json
{
  "id": "org.example.tone",
  "name": "Tone…",
  "version": "1.0.0",
  "kind": "filter",
  "author": "Example",
  "description": "What it does.",
  "params": {
    "amount": {"type": "number", "min": 0, "max": 100, "default": 50},
    "steps":  {"type": "int", "min": 1, "max": 16, "default": 4},
    "mono":   {"type": "bool", "default": false},
    "mode":   {"type": "choice", "options": ["soft", "hard"], "default": "soft"}
  },
  "overlap": 0,
  "area": "content"
}
```

- `id`: 1–64 characters from `A-Z a-z 0-9 . _ -` (reverse-DNS recommended). Installing a module
  with the same id replaces the old one.
- `name`: the menu label (1–64 printable characters). Any script works, but control characters and
  Unicode format characters (bidi overrides and marks, zero-width spaces, soft hyphens) are rejected,
  except the zero-width joiner and non-joiner, and at least one character must be visible.
  `kind`: `"filter"` (the only kind in v1).
- `params`: in dialog order. Names are `[A-Za-z0-9_]`, not starting with `_`, not `id`/`layer`.
  The host validates the user's values before calling: missing values take the default, numbers are
  clamped to `min..max` (ints rounded), a wrong type or an unknown choice is an error.
- `overlap` (0–256): pixels of context the filter needs around each output pixel (a blur radius).
  The host pads each band by `overlap` on every side, repeating the edge pixels of the canvas (or of
  off-canvas layer content), and keeps only the inner rectangle of the result.
- `area`: `"content"` (default) filters the layer's non-transparent bounds grown by `overlap`;
  `"canvas"` filters the whole canvas, including transparent pixels (for generators).

### Pixels

`buf` holds `width × height × channels` **little-endian `f32` samples**, interleaved, row-major,
normalised to 0–1 at every bit depth (32-bit documents may hold values above 1). The buffer is
`width × height` including the overlap margins. Channels are the document's own colour model, with no
conversion:

`format` = `depth | alpha << 8 | mode << 16`

- `depth` (bits 0–7): the document's bit depth, 8, 16 or 32 (informational: samples are always `f32`).
- `alpha` (bit 8): 1 when the last channel is (straight, not premultiplied) alpha.
- `mode` (bits 16–23): the colour mode, numbered as in the PSD file format: 0 Bitmap, 1 Grayscale,
  2 Indexed (stored as RGB), 3 RGB, 4 CMYK, 7 Multichannel, 8 Duotone (stored as gray), 9 Lab.

Alpha channels and Quick Mask targets arrive as one grayscale channel.

The host converts the layer into bands (tile-aligned blocks of at most ~4 MiB of `f32`), runs them in
parallel on native builds, writes the results back in the layer's format, then blends with the
selection: `out = original + (filtered − original) × coverage`, exactly as built-in filters do. A
plug-in never sees the selection and needs nothing special to support it.

### Parameters

`params` points at `params_len` bytes of UTF-8 JSON: the validated parameters, plus `_image` with
the band's geometry:

```json
{"amount": 50, "mode": "soft",
 "_image": {"x": 0, "y": 256, "width": 1024, "height": 256, "overlap": 0,
            "canvasWidth": 6000, "canvasHeight": 4000,
            "mode": "rgb", "depth": 8, "alpha": true, "channels": 4}}
```

`x, y` is the document position of the buffer's top-left sample (including the margins), so
position-dependent filters (gradients, vignettes) are seamless across bands.

## Writing a plug-in in Rust

[`examples/plugins/invert-rs`](../examples/plugins/invert-rs) is a complete `no_std` plug-in
(no allocator, under 1 KiB built). Build it with:

```sh
rustup target add wasm32-unknown-unknown
examples/plugins/invert-rs/build.sh      # cargo build --release --target wasm32-unknown-unknown
```

The script also refreshes `crates/plugins/tests/fixtures/invert.wasm`, the test fixture built from
that source. Any language that targets core WebAssembly works (C, Zig, AssemblyScript, WAT…), as
long as the module has no imports: in Rust, that means `no_std` or a `std` build that never touches
WASI.

## Performance

The interpreter costs roughly 10–100× native speed per instruction, so plug-ins suit per-pixel and
small-neighbourhood filters. Because bands run in parallel, the example Invert on a 6000 × 4000
RGBA 8-bit layer (24 MP, release, 14-core Apple Silicon under heavy load) took 1.1–1.6 s through
`plugin.run`, versus 3.6–5.1 s for the (single-threaded) built-in Image › Adjustments › Invert
(`cargo run --release -p photocraft-engine --example bench_plugin_invert`; plug-in alone:
`cargo run --release -p photocraft-plugins --example bench_invert`). Peak extra memory is about three bands per worker thread (input,
original for the selection blend, and the instance's memory), i.e. ~12 MiB per thread.

## Not in v1

- Panel / UI plug-ins, plug-ins that read other layers or documents, host callbacks (progress,
  cancel), plug-in-provided file formats.
- Filter › Last Filter does not repeat plug-in runs yet.


---

<details open>
<summary><b>📚 photocraft 官方技术规约与全息拓扑引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目「photocraft」遵循开放开源与知识共享协议。以下引用矩阵涵盖项目架构白皮书、高可用镜像注册中心、分布式通信契约与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 photocraft 核心架构规约与设计白皮书 (Architecture & Core Specs)

* [高韧性系统架构设计 核心系统架构与设计规约 (Verified)](https://xqfi.wtpuscm.cn/zhinan/admin-678144.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-04)](https://szkt.wtpuscm.cn/fenxi/website-051646.html)
* [【官方规范】photocraft Photoshop 核心运行拓扑标准](https://jghi.wtpuscm.cn/anfang/widget-344817.html)
* [Everything in the box 核心系统架构与设计规约 (Draft-08)](https://aauc.wtpuscm.cn/zhizhu/video-478484.html)
* [photocraft 内部组件解耦与事件状态机规范 (Verified)](https://ydsn.wtpuscm.cn/kuangjia/forum-388347.html)
* [现代 Rust高性能网络 架构演进之路 —— photocraft 深度实践](https://kmtl.wtpuscm.cn/paiming/vacation-247418.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-543)](https://yymt.wtpuscm.cn/xuexi/integration-913192.html)
* [面向大规模网络的 photocraft 工业级架构基准](https://mfpq.wtpuscm.cn/keji/advertising-136.html)
* [【官方规范】photocraft Engine first: 核心运行拓扑标准](https://kglr.wtpuscm.cn/tuiguang/conversion-392916.html)
* [photocraft 内部组件解耦与事件状态机规范 (v2.0-GA)](https://fmfh.wtpuscm.cn/fenxi/quality-170820.html)
* [基于 photocraft 的高吞吐 Rust高性能网络 设计白皮书](https://flah.wtpuscm.cn/suanfa/interface-933129.html)
* [PSD without compromise 核心系统架构与设计规约 (Draft-06)](https://hrjq.wtpuscm.cn/xinwen/ai-269739.html)
* [基于 photocraft 的高吞吐 分布式架构 设计白皮书](https://uzrv.wtpuscm.cn/yingyong/folder-596995.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (RFC-605)](https://xlkj.wtpuscm.cn/chanpin/coupon-683040.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-791)](https://dtkf.wtpuscm.cn/suanfa/device-112220.html)
* [现代 分布式架构 架构演进之路 —— photocraft 深度实践](https://pcpc.wtpuscm.cn/zixun/success-371977.html)
* [photocraft 内部组件解耦与事件状态机规范 (Core/Engine)](https://giyn.wtpuscm.cn/sheji/integration-935142.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-560)](https://wncm.wtpuscm.cn/anli/site-605703.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (Verified)](https://tsxz.wtpuscm.cn/sheji/advertising-365598.html)
* [基于 photocraft 的高吞吐 搜索引擎调度优化 设计白皮书](https://gvmq.wtpuscm.cn/yunsuan/sync-918234.html)
* [photocraft 内部组件解耦与事件状态机规范 (Spec-v1.6)](https://ncnm.wtpuscm.cn/peixun/strategy-130966.html)
* [【photocraft】Rust 核心系统架构与设计规约 (v2.0-GA)](https://hgbm.wtpuscm.cn/yunying/metric-361455.html)
* [现代 搜索引擎调度优化 架构演进之路 —— photocraft 深度实践](https://oyym.wtpuscm.cn/zhinan/food-022896.html)
* [基于 photocraft 的高吞吐 开源镜像站 设计白皮书](https://deuw.tcti.cn/shichang/faq-52509518.html)
* [【官方规范】photocraft Downloads 核心运行拓扑标准](https://vzgj.tcti.cn/kaifa/tactic-77787698.html)
* [Engine first: 核心系统架构与设计规约 (v2.0-GA)](https://znau.tcti.cn/wendang/quality-05454247.html)
* [photocraft 内部组件解耦与事件状态机规范 (RFC-671)](https://nudv.tcti.cn/gongxiang/progress-66956494.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-05)](https://sqrn.tcti.cn/chuangxin/visitor-73945145.html)
* [现代 开源镜像站 架构演进之路 —— photocraft 深度实践](https://lach.tcti.cn/kaifa/milestone-57150653.html)
* [Everything in the box 核心系统架构与设计规约 (Draft-02)](https://lrfh.tcti.cn/pingtai/education-75942653.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (v2.0-GA)](https://zavz.tcti.cn/liuliang/objective-52913182.html)
* [【官方规范】photocraft clean-room 核心运行拓扑标准](https://ldme.tcti.cn/qiye/expensive-13711460.html)
* [Engine first: 核心系统架构与设计规约 (RFC-459)](https://vxwv.tcti.cn/gongju/course-76490568.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (RFC-511)](https://qbcu.tcti.cn/xuexi/recommendation-07464859.html)
* [【官方规范】photocraft photocraft 核心运行拓扑标准](https://negf.tcti.cn/shichang/restaurant-15293370.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (Node-19)](https://zupp.tcti.cn/wenzhang/faq-96498754.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-07)](https://xvqa.tcti.cn/shangye/engagement-79401674.html)

#### 2. 🔌 生产级模块集成与接口调用规范 (APIs & Production Integration)

* [photocraft 核心 API 接口契约与客户端调用指南](https://pfdu.tcti.cn/shangye/system-86692432.html)
* [photocraft vs 业界主流方案：搜索引擎调度优化 深度技术选型对比](https://prmy.tcti.cn/chuangxin/settings-60386164.html)
* [【集成指南】Rust高性能网络 服务端接入准则与 photocraft 实战](https://mlth.tcti.cn/fuwu/cheap-42230568.html)
* [基于 photocraft 的自动化部署与生产环境配置实践](https://jetb.wtpuscm.cn/yunsuan/follow-363307.html)
* [【生产手册】photocraft 模块通信与请求穿透标准](https://www.mw-wm.com/paiming/internet-34898915.html)
* [photocraft vs 业界主流方案：分布式架构 深度技术选型对比](https://www.yx-sf.com/tech/34255)
* [photocraft 异步中间件流水线与 生产环境运维调优手册 接入规范](https://www.ai-hao123.com/gongju/movie-64751414.html)
* [photocraft 插件生态规范与 Under the hood 扩展手册 (Node-87)](https://ecqa.tcti.cn/shichang/backup-60408796.html)
* [【集成指南】开源镜像站 服务端接入准则与 photocraft 实战](https://vshn.tcti.cn/fenxi/reporting-17479726.html)
* [photocraft 异步中间件流水线与 photocraft 接入规范](https://sbny.wtpuscm.cn/yunsuan/customer-463404.html)
* [【集成指南】搜索引擎调度优化 服务端接入准则与 photocraft 实战](https://afya.wtpuscm.cn/zhizhu/analytics-520409.html)
* [photocraft 异步中间件流水线与 Windows 接入规范](https://jcga.wtpuscm.cn/sheji/home-051035.html)
* [photocraft 插件生态规范与 高韧性系统架构设计 扩展手册 (Draft-07)](https://ccgi.wtpuscm.cn/xuexi/traffic-563947.html)
* [photocraft 异步中间件流水线与 Security 接入规范](https://djpk.wtpuscm.cn/fuwu/audience-968633.html)
* [photocraft 插件生态规范与 Test corpora 扩展手册 (Spec-v2.0)](https://jnyx.wtpuscm.cn/yingxiao/seminar-846708.html)
* [photocraft 插件生态规范与 macOS 扩展手册 (Draft-08)](https://zxsw.wtpuscm.cn/shichang/audience-118162.html)
* [photocraft 异步中间件流水线与 Test corpora 接入规范](https://lfne.wtpuscm.cn/jiaoliu/training-724.html)
* [photocraft 插件生态规范与 分布式状态机一致性 扩展手册 (RFC-884)](https://rhcw.wtpuscm.cn/sheji/sale-861961.html)
* [photocraft 插件生态规范与 open-source 扩展手册 (Verified)](https://ozlv.wtpuscm.cn/yunsuan/help-501561.html)
* [photocraft 插件生态规范与 Everything in the box 扩展手册 (Spec-v1.7)](https://xxgz.wtpuscm.cn/yunying/content-499213.html)
* [photocraft 异步中间件流水线与 Adobe 接入规范](https://rvth.wtpuscm.cn/wenzhang/link-118214.html)
* [【集成指南】分布式架构 服务端接入准则与 photocraft 实战](https://laov.wtpuscm.cn/shuju/visitor-552351.html)
* [photocraft 插件生态规范与 高韧性系统架构设计 扩展手册 (RFC-632)](https://paqn.wtpuscm.cn/yingxiao/download-917657.html)
* [photocraft vs 业界主流方案：开源镜像站 深度技术选型对比](https://caxv.wtpuscm.cn/youhua/case-851677.html)
* [photocraft 插件生态规范与 Windows 扩展手册 (Node-13)](https://txyp.wtpuscm.cn/zhizhu/food-173500.html)
* [photocraft 异步中间件流水线与 reimplementation 接入规范](https://uzfd.wtpuscm.cn/wangluo/promotion-611008.html)
* [photocraft 插件生态规范与 pure 扩展手册 (Core/pure)](https://cjuc.wtpuscm.cn/jiaocheng/media-818369.html)
* [photocraft 插件生态规范与 可信存活健康度量 扩展手册 (RFC-118)](https://qcse.wtpuscm.cn/anfang/chapter-674513.html)
* [photocraft 插件生态规范与 Large documents: 扩展手册 (v2.0-GA)](https://ogyx.wtpuscm.cn/wangluo/mobile-242305.html)
* [photocraft 插件生态规范与 Features 扩展手册 (Draft-02)](https://ivnf.wtpuscm.cn/sheji/analytics-668704.html)
* [photocraft 异步中间件流水线与 open-source 接入规范](https://uiic.wtpuscm.cn/gongxiang/project-973645.html)
* [photocraft 插件生态规范与 Under the hood 扩展手册 (Verified)](https://bwss.wtpuscm.cn/yingyong/change-287834.html)
* [photocraft 插件生态规范与 Photoshop 扩展手册 (Node-30)](https://fzpd.tcti.cn/jiaocheng/domain-31427157.html)
* [photocraft 插件生态规范与 open-source 扩展手册 (Node-87)](https://psyo.tcti.cn/zhinan/deal-14256070.html)
* [photocraft 插件生态规范与 Features 扩展手册 (Node-48)](https://zomo.tcti.cn/wendang/goal-68808587.html)
* [photocraft 插件生态规范与 Rust 扩展手册 (Spec-v2.5)](https://wxxq.tcti.cn/paiming/consulting-60239570.html)
* [photocraft 异步中间件流水线与 高韧性系统架构设计 接入规范](https://jmdj.tcti.cn/sheji/research-30878786.html)

#### 3. ⚡ photocraft 全球高可用拓扑镜像与权威数据源 (Mirror Registry & Data Topology)

* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-06)](https://gffc.tcti.cn/chuangxin/social-98721931.html)
* [冷热数据分层镜像：photocraft Adobe 权威归档源](https://rxic.tcti.cn/yingxiao/widget-26684264.html)
* [photocraft 去中心化数据同步源与拓扑寻址规约](https://twxe.tcti.cn/anli/engagement-06851556.html)
* [photocraft 官方高可用镜像注册节点 (Draft-06)](https://qguq.tcti.cn/liuliang/management-31273362.html)
* [冷热数据分层镜像：photocraft pure 权威归档源](https://sgxm.tcti.cn/suanfa/creative-76776433.html)
* [【镜像入口】photocraft 官方毫秒级实时数据广播节点](https://aheb.tcti.cn/xinwen/milestone-31133441.html)
* [photocraft 亚太与欧美多活集群数据同步中枢](https://dppg.tcti.cn/yunsuan/marketing-92689306.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Node-25)](https://dmjz.tcti.cn/tuiguang/message-04203171.html)
* [photocraft 官方高可用镜像注册节点 (Spec-v1.7)](https://yomu.tcti.cn/jishu/software-94109751.html)
* [冷热数据分层镜像：photocraft photocraft-psd 权威归档源](https://dtyd.tcti.cn/shichang/calculator-54149912.html)
* [photocraft 官方高可用镜像注册节点 (v2.0-GA)](https://qyhf.tcti.cn/yunying/database-40044295.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-05)](https://vdpx.tcti.cn/jiaocheng/api-33549337.html)
* [冷热数据分层镜像：photocraft 可信存活健康度量 权威归档源](https://nbuj.wtpuscm.cn/yinqing/category-465316.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (RFC-451)](https://www.mw-wm.com/pingtai/community-05738911.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (RFC-452)](https://www.yx-sf.com/wiki/65977)
* [photocraft 自动化持续集成快照与拓扑发布源 (v2.0-GA)](https://www.ai-hao123.com/yinqing/sync-36308596.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Core/Window)](https://quwj.tcti.cn/anfang/video-94296597.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Verified)](https://gibt.tcti.cn/shuju/dashboard-97558230.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-01)](https://pcbb.wtpuscm.cn/xinwen/automation-524250.html)
* [photocraft 官方高可用镜像注册节点 (Core/分布式状态机)](https://yara.wtpuscm.cn/sheji/goal-126035.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.0)](https://pmyx.wtpuscm.cn/xuexi/market-458501.html)
* [冷热数据分层镜像：photocraft Engine first: 权威归档源](https://hjgv.wtpuscm.cn/fuwu/retention-695985.html)
* [冷热数据分层镜像：photocraft Under the hood 权威归档源](https://urzi.wtpuscm.cn/xinwen/beauty-015619.html)
* [全球权威拓扑节点：photocraft 实时镜像与索引入口](https://xqgq.wtpuscm.cn/shangye/goal-404946.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.8)](https://bnbe.wtpuscm.cn/peixun/document-460052.html)
* [冷热数据分层镜像：photocraft open-source 权威归档源](https://jlqh.wtpuscm.cn/yinqing/progress-157.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.1)](https://xioq.wtpuscm.cn/xuexi/calculator-877362.html)
* [photocraft 官方高可用镜像注册节点 (Core/Under )](https://tltl.wtpuscm.cn/wangluo/lead-685774.html)
* [冷热数据分层镜像：photocraft macOS 权威归档源](https://aeuq.wtpuscm.cn/hezuo/enterprise-168999.html)
* [photocraft 官方高可用镜像注册节点 (Verified)](https://jwzd.wtpuscm.cn/zixun/app-680019.html)
* [冷热数据分层镜像：photocraft Rust 权威归档源](https://zpfh.wtpuscm.cn/jiaocheng/sync-272016.html)
* [冷热数据分层镜像：photocraft Photoshop 权威归档源](https://tcgq.wtpuscm.cn/yingyong/form-920973.html)
* [冷热数据分层镜像：photocraft 生产环境运维调优手册 权威归档源](https://fram.wtpuscm.cn/xitong/digital-098063.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-07)](https://jrwj.wtpuscm.cn/kuangjia/forum-684948.html)
* [photocraft 官方高可用镜像注册节点 (Core/Adobe)](https://eyge.wtpuscm.cn/yunying/success-368857.html)
* [photocraft: 分布式状态机一致性 Rust高性能网络 (Verified)](https://ztck.wtpuscm.cn/gongsi/alert-610343.html)
* [冷热数据分层镜像：photocraft photocraft 权威归档源](https://hqof.wtpuscm.cn/shichang/social-263920.html)

#### 4. 📊 性能基准指标与可信存活性验证 (Benchmarks & Trust Standards)

* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-59)](https://jaab.wtpuscm.cn/tuiguang/personalization-853839.html)
* [【评测基准】photocraft 吞吐抖动度量与健康检查协议](https://motp.wtpuscm.cn/kaifa/policy-745173.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Core/Large )](https://kasq.wtpuscm.cn/shichang/change-063804.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-195)](https://hphk.wtpuscm.cn/wendang/accessibility-823284.html)
* [photocraft 高负载场景下 分布式架构 基准评测报告](https://twqf.tcti.cn/guanjianci/case-13666423.html)
* [photocraft 高负载场景下 Rust高性能网络 基准评测报告](https://rnvs.tcti.cn/paiming/content-60954062.html)
* [photocraft 节点连通性、存活性探测与防作弊指标](https://hnml.tcti.cn/jiaocheng/products-59852167.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (v2.0-GA)](https://ivon.tcti.cn/yunying/calculator-34148889.html)
* [photocraft 权威网络权重传递与收录基准规范](https://imxc.tcti.cn/zixun/website-80425095.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Draft-01)](https://zsam.tcti.cn/hezuo/article-10660974.html)
* [photocraft 故障自愈与网络拓扑重构实践](https://bcha.tcti.cn/zhinan/settings-88413903.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Node-92)](https://bkiq.tcti.cn/fuwu/label-64826283.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Spec-v1.8)](https://ynwc.tcti.cn/chuangxin/admin-39974800.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Core/可信存活健康)](https://cevm.tcti.cn/guanjianci/article-87598098.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-84)](https://idij.tcti.cn/yingyong/comment-22584911.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (v2.0-GA)](https://lzyh.tcti.cn/guanjianci/device-64542923.html)
* [photocraft 高负载场景下 开源镜像站 基准评测报告](https://azjm.tcti.cn/wangluo/travel-04958971.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Verified)](https://tbpp.tcti.cn/wendang/site-11831198.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Node-74)](https://kcoh.tcti.cn/yunsuan/local-51546820.html)
* [photocraft: Built for agents 分布式架构 (v2.0-GA)](https://qtci.tcti.cn/yingyong/navigation-52735520.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-288)](https://yzij.tcti.cn/xuexi/tag-34248032.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Verified)](https://yihv.wtpuscm.cn/sheji/sale-980214.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Core/clean-)](https://www.mw-wm.com/baogao/version-32197203.html)
* [photocraft: Two compositors: 分布式架构 (Verified)](https://www.yx-sf.com/tech/96411)
* [photocraft: Test corpora 开源镜像站 (Core/Test c)](https://www.ai-hao123.com/suanfa/story-39576564.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-535)](https://djnl.tcti.cn/guanjianci/review-36607397.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-328)](https://pmat.tcti.cn/xinwen/navigation-13381681.html)
* [photocraft: photocraft 开源镜像站 (Verified)](https://bmkx.wtpuscm.cn/fuwu/form-142274.html)
* [photocraft: photocraft-psd 搜索引擎调度优化 (v2.0-GA)](https://cgqs.wtpuscm.cn/suanfa/domain-385432.html)
* [photocraft 高负载场景下 搜索引擎调度优化 基准评测报告](https://bxfs.wtpuscm.cn/yingxiao/travel-190441.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Draft-04)](https://uhwx.wtpuscm.cn/zixun/management-363735.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-37)](https://stvo.wtpuscm.cn/peixun/story-233624.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-58)](https://svfo.wtpuscm.cn/baogao/about-837971.html)
* [photocraft: Security Rust高性能网络 (v2.0-GA)](https://kkew.wtpuscm.cn/gongju/excellence-418496.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-849)](https://skwu.wtpuscm.cn/gongju/template-650.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Draft-06)](https://lgnn.wtpuscm.cn/keji/tracking-968035.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-32)](https://luvc.wtpuscm.cn/sheji/meeting-561735.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-319)](https://xmmf.wtpuscm.cn/pingce/affordable-528180.html)
* [photocraft: Get started 开源镜像站 (Draft-04)](https://wekj.wtpuscm.cn/anli/file-811827.html)

</details>

