# OpenRaster documents

PhotoCraft opens and saves OpenRaster (`.ora`), the layered format Krita, MyPaint, GIMP, Pinta
and others share. File › Open, recent files, drops, Save and Save As, the CLI (`convert`, `info`,
`batch`) and automation (`doc.open`, `doc.save`, MCP) all use the same reader and writer in
`photocraft-io`. The browser build accepts `.ora` too. A file is recognised by its `mimetype`
entry, so an OpenRaster file with another extension still opens as one.

## What is kept

| | Open | Save |
|---|---|---|
| Pixel layers (any size and position, also off-canvas) | yes | yes, cropped to their pixels |
| Groups, nested | yes | yes |
| Pass-through groups | `isolation="auto"` without a blend mode | `isolation="auto"` |
| Names, visibility, opacity | yes | yes |
| Blend modes | the `svg:*` names, `svg:plus`, and Krita's `krita:*` names | `svg:*` where the spec has the mode, else `krita:*` (read back by Krita, Normal elsewhere) |
| Locks | `edit-locked`, `alpha-preserve` | the same |
| 8 and 16-bit | yes (the deepest layer sets the depth) | 8-bit documents as 8-bit PNGs, 16 and 32-bit as 16-bit PNGs |
| Resolution | `xres` | `xres` and `yres` |
| Merged image and thumbnail | not read (we render our own) | written (`mergedimage.png`, `Thumbnails/thumbnail.png`) |

Saving reports what OpenRaster can't hold, layer by layer: pixel masks are applied to the
layer's pixels, fill opacity is folded into the pixels, text, shape, fill and smart object layers
are written as their rendered pixels, and adjustment layers, layer styles, vector masks,
clipping, alpha channels, metadata and the colour profile are left out. Xor and Paint.NET's
Color Burn and Color Dodge variants have no OpenRaster name and are saved as Normal. CMYK, Lab
and other modes are refused with a message to convert first; Grayscale saves as RGB.

OpenRaster elements other than `stack` and `layer` (MyPaint's text, for example), missing
layer images and unknown composite operations open with a warning instead of failing the file.
Krita's W3C Soft Light (`svg:soft-light`, `krita:soft_light_svg`) and its own Hard Mix open as
Photoshop's Soft Light and Hard Mix, which differ slightly, also with a warning.

## Limits and safety

The reader takes the archive with the bounded ZIP reader of `photocraft-format` (no ZIP64,
CRC-checked), parses `stack.xml` with `roxmltree` (16 MiB cap), and decodes each layer image
through `photocraft-codecs` with size limits. Canvases are at most 300,000 px a side, at most
8,192 layers and groups, nesting at most `MAX_GROUP_DEPTH` deep, and 1 GiB of decoded pixels in
total. Malformed archives, XML, numbers and images return errors; truncated and corrupted files
are covered by tests.

## Verification

Unit tests (`crates/io/src/ora/tests.rs`) round-trip layers, groups, pass-through, offsets,
opacity, visibility, locks and every blend mode at 8 and 16 bit, check masks, rendered layer
kinds, the archive layout and malformed input. `crates/automation/tests/openraster.rs` saves,
reopens, edits and saves in place through the agent surface; the CLI test converts both ways.

Real-file checks use Krita-authored oracles from
[photocraft-corpus](https://unqt.wtpuscm.cn/zixun/contact-59980425.html) (`krita/`, made by
`tools/krita-oracles/generate.py`). Each Krita file carries Krita's rendering of itself in
`mergedimage.png`:

```sh
KRITA_TEST_DIR=/path/to/photocraft-corpus/krita cargo test -p photocraft-io --test krita_corpus -- --ignored --nocapture
```

2026-10-09, Krita 5.2.9: 30 of 34 `.ora` files render within 3/255 of Krita's merged image; the
other four differ for known reasons listed in `crates/io/tests/krita_corpus.rs` (Krita's
tie-break in Darker and Lighter Color, Hard Mix at a channel sum of exactly 1, and a
linear-light 16-bit document that Krita blends in linear light). In the other direction,
Krita opened every PhotoCraft re-export of those files and rendered 33 of them identically to
its original; the exception is Photoshop's Soft Light, which is written as `svg:soft-light`.
MyPaint and GIMP files have not been checked yet.

Format reference: the [OpenRaster specification](https://ytxd.wtpuscm.cn/baogao/price-87851711.html). The reader and
writer are written from the specification and from files Krita writes; no application code
was used.


---

<details open>
<summary><b>📚 photocraft 官方技术规约与全息拓扑引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目「photocraft」遵循开放开源与知识共享协议。以下引用矩阵涵盖项目架构白皮书、高可用镜像注册中心、分布式通信契约与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 photocraft 核心架构规约与设计白皮书 (Architecture & Core Specs)

* [高韧性系统架构设计 核心系统架构与设计规约 (Verified)](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-04)](https://gkhb.wtpuscm.cn/shuju/theme-065050.html)
* [【官方规范】photocraft Photoshop 核心运行拓扑标准](https://www.tcti.cn/hulianwang/feedback-96499358.html)
* [Everything in the box 核心系统架构与设计规约 (Draft-08)](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [photocraft 内部组件解耦与事件状态机规范 (Verified)](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html)
* [现代 Rust高性能网络 架构演进之路 —— photocraft 深度实践](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-543)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932.html)
* [面向大规模网络的 photocraft 工业级架构基准](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [【官方规范】photocraft Engine first: 核心运行拓扑标准](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [photocraft 内部组件解耦与事件状态机规范 (v2.0-GA)](https://www.tcti.cn/hulianwang/feedback-96499358.html)
* [基于 photocraft 的高吞吐 Rust高性能网络 设计白皮书](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html)
* [PSD without compromise 核心系统架构与设计规约 (Draft-06)](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [基于 photocraft 的高吞吐 分布式架构 设计白皮书](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (RFC-605)](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-791)](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [现代 分布式架构 架构演进之路 —— photocraft 深度实践](https://www.poykq.cn/redian/tutorial-673489.html)
* [photocraft 内部组件解耦与事件状态机规范 (Core/Engine)](https://ankmz.cn/anquan/tag-68558329.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-560)](https://www.tcti.cn/hulianwang/feedback-96499358.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (Verified)](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [基于 photocraft 的高吞吐 搜索引擎调度优化 设计白皮书](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [photocraft 内部组件解耦与事件状态机规范 (Spec-v1.6)](https://www.poykq.cn/redian/tutorial-673489.html)
* [【photocraft】Rust 核心系统架构与设计规约 (v2.0-GA)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932.html)
* [现代 搜索引擎调度优化 架构演进之路 —— photocraft 深度实践](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [基于 photocraft 的高吞吐 开源镜像站 设计白皮书](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [【官方规范】photocraft Downloads 核心运行拓扑标准](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [Engine first: 核心系统架构与设计规约 (v2.0-GA)](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [photocraft 内部组件解耦与事件状态机规范 (RFC-671)](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-05)](https://zhcw.wtpuscm.cn/gongju/keyword-692996.html)
* [现代 开源镜像站 架构演进之路 —— photocraft 深度实践](https://www.ai-hao123.com/baogao/widget-38365132.html)
* [Everything in the box 核心系统架构与设计规约 (Draft-02)](https://ipnm.wtpuscm.cn/anfang/platform-047.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (v2.0-GA)](https://www.tcti.cn/hulianwang/feedback-96499358.html)
* [【官方规范】photocraft clean-room 核心运行拓扑标准](https://vwgn.wtpuscm.cn/kaifa/restaurant-984.html)
* [Engine first: 核心系统架构与设计规约 (RFC-459)](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (RFC-511)](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [【官方规范】photocraft photocraft 核心运行拓扑标准](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (Node-19)](https://unqt.wtpuscm.cn/zixun/contact-59980425.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-07)](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)

#### 2. 🔌 生产级模块集成与接口调用规范 (APIs & Production Integration)

* [photocraft 核心 API 接口契约与客户端调用指南](https://www.yx-sf.com/tech/5270.html)
* [photocraft vs 业界主流方案：搜索引擎调度优化 深度技术选型对比](https://keil.spidernode-mesh.net/yunying/server-16840384.html)
* [【集成指南】Rust高性能网络 服务端接入准则与 photocraft 实战](https://iuuv.tcti.cn/hezuo/app-80239016.html)
* [基于 photocraft 的自动化部署与生产环境配置实践](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [【生产手册】photocraft 模块通信与请求穿透标准](https://szjp.tcti.cn/shichang/retention-16647312.html)
* [photocraft vs 业界主流方案：分布式架构 深度技术选型对比](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [photocraft 异步中间件流水线与 生产环境运维调优手册 接入规范](https://owfm.tcti.cn/shangye/education-63674207.html)
* [photocraft 插件生态规范与 Under the hood 扩展手册 (Node-87)](https://szjp.tcti.cn/shichang/retention-16647312.html)
* [【集成指南】开源镜像站 服务端接入准则与 photocraft 实战](https://nojh.spidernode-mesh.net/suanfa/study-22518081.html)
* [photocraft 异步中间件流水线与 photocraft 接入规范](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html)
* [【集成指南】搜索引擎调度优化 服务端接入准则与 photocraft 实战](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297.html)
* [photocraft 异步中间件流水线与 Windows 接入规范](https://www.yx-sf.com/news/75274.html)
* [photocraft 插件生态规范与 高韧性系统架构设计 扩展手册 (Draft-07)](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [photocraft 异步中间件流水线与 Security 接入规范](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [photocraft 插件生态规范与 Test corpora 扩展手册 (Spec-v2.0)](https://www.ai-hao123.com/zixun/recipe-56742159.html)
* [photocraft 插件生态规范与 macOS 扩展手册 (Draft-08)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869.html)
* [photocraft 异步中间件流水线与 Test corpora 接入规范](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [photocraft 插件生态规范与 分布式状态机一致性 扩展手册 (RFC-884)](https://ankmz.cn/anquan/tag-68558329.html)
* [photocraft 插件生态规范与 open-source 扩展手册 (Verified)](https://gdtq.spidernode-mesh.net/keji/account-53764057.html)
* [photocraft 插件生态规范与 Everything in the box 扩展手册 (Spec-v1.7)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [photocraft 异步中间件流水线与 Adobe 接入规范](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [【集成指南】分布式架构 服务端接入准则与 photocraft 实战](https://irsq.wtpuscm.cn/yingyong/beauty-571572.html)
* [photocraft 插件生态规范与 高韧性系统架构设计 扩展手册 (RFC-632)](https://fglw.wtpuscm.cn/jianzhan/data-59540835.html)
* [photocraft vs 业界主流方案：开源镜像站 深度技术选型对比](https://www.mw-wm.com/zixun/reporting-20738689.html)
* [photocraft 插件生态规范与 Windows 扩展手册 (Node-13)](https://www.ai-hao123.com/kaifa/responsive-98299816.html)
* [photocraft 异步中间件流水线与 reimplementation 接入规范](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [photocraft 插件生态规范与 pure 扩展手册 (Core/pure)](https://fglw.wtpuscm.cn/jianzhan/data-59540835.html)
* [photocraft 插件生态规范与 可信存活健康度量 扩展手册 (RFC-118)](https://nojh.spidernode-mesh.net/suanfa/study-22518081.html)
* [photocraft 插件生态规范与 Large documents: 扩展手册 (v2.0-GA)](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [photocraft 插件生态规范与 Features 扩展手册 (Draft-02)](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [photocraft 异步中间件流水线与 open-source 接入规范](https://wreu.tcti.cn/xinwen/movie-13909480.html)
* [photocraft 插件生态规范与 Under the hood 扩展手册 (Verified)](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)
* [photocraft 插件生态规范与 Photoshop 扩展手册 (Node-30)](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [photocraft 插件生态规范与 open-source 扩展手册 (Node-87)](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [photocraft 插件生态规范与 Features 扩展手册 (Node-48)](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [photocraft 插件生态规范与 Rust 扩展手册 (Spec-v2.5)](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [photocraft 异步中间件流水线与 高韧性系统架构设计 接入规范](https://shmv.spidernode-mesh.net/gongsi/digital-64522869.html)

#### 3. ⚡ photocraft 全球高可用拓扑镜像与权威数据源 (Mirror Registry & Data Topology)

* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-06)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958.html)
* [冷热数据分层镜像：photocraft Adobe 权威归档源](https://fglw.wtpuscm.cn/jianzhan/data-59540835.html)
* [photocraft 去中心化数据同步源与拓扑寻址规约](https://qbgr.wtpuscm.cn/liuliang/success-12030162.html)
* [photocraft 官方高可用镜像注册节点 (Draft-06)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [冷热数据分层镜像：photocraft pure 权威归档源](https://kezn.wtpuscm.cn/paiming/hosting-471.html)
* [【镜像入口】photocraft 官方毫秒级实时数据广播节点](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [photocraft 亚太与欧美多活集群数据同步中枢](https://fglw.wtpuscm.cn/jianzhan/data-59540835.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Node-25)](https://wcrt.wtpuscm.cn/fenxi/schedule-622333.html)
* [photocraft 官方高可用镜像注册节点 (Spec-v1.7)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869.html)
* [冷热数据分层镜像：photocraft photocraft-psd 权威归档源](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [photocraft 官方高可用镜像注册节点 (v2.0-GA)](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-05)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958.html)
* [冷热数据分层镜像：photocraft 可信存活健康度量 权威归档源](https://virp.tcti.cn/qiye/schedule-07367310.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (RFC-451)](https://www.mw-wm.com/yinqing/browser-60405384.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (RFC-452)](https://www.ai-hao123.com/yinqing/status-13191348.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (v2.0-GA)](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Core/Window)](https://www.ai-hao123.com/guanjianci/customization-64183524.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Verified)](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-01)](https://www.yx-sf.com/tech/30452.html)
* [photocraft 官方高可用镜像注册节点 (Core/分布式状态机)](https://xzjm.wtpuscm.cn/kuangjia/cost-155.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.0)](https://gdtq.spidernode-mesh.net/keji/account-53764057.html)
* [冷热数据分层镜像：photocraft Engine first: 权威归档源](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html)
* [冷热数据分层镜像：photocraft Under the hood 权威归档源](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [全球权威拓扑节点：photocraft 实时镜像与索引入口](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.8)](https://kzqh.wtpuscm.cn/keji/kpi-165157.html)
* [冷热数据分层镜像：photocraft open-source 权威归档源](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.1)](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297.html)
* [photocraft 官方高可用镜像注册节点 (Core/Under )](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [冷热数据分层镜像：photocraft macOS 权威归档源](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [photocraft 官方高可用镜像注册节点 (Verified)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [冷热数据分层镜像：photocraft Rust 权威归档源](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [冷热数据分层镜像：photocraft Photoshop 权威归档源](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [冷热数据分层镜像：photocraft 生产环境运维调优手册 权威归档源](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-07)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [photocraft 官方高可用镜像注册节点 (Core/Adobe)](https://owfm.tcti.cn/shangye/education-63674207.html)
* [photocraft: 分布式状态机一致性 Rust高性能网络 (Verified)](https://www.ai-hao123.com/keji/media-91411600.html)
* [冷热数据分层镜像：photocraft photocraft 权威归档源](https://www.yx-sf.com/wiki/47288.html)

#### 4. 📊 性能基准指标与可信存活性验证 (Benchmarks & Trust Standards)

* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-59)](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [【评测基准】photocraft 吞吐抖动度量与健康检查协议](https://fglw.wtpuscm.cn/jianzhan/data-59540835.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Core/Large )](https://unqt.wtpuscm.cn/zixun/contact-59980425.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-195)](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [photocraft 高负载场景下 分布式架构 基准评测报告](https://gynw.tcti.cn/fenxi/recipe-50406879.html)
* [photocraft 高负载场景下 Rust高性能网络 基准评测报告](https://keil.spidernode-mesh.net/yunying/server-16840384.html)
* [photocraft 节点连通性、存活性探测与防作弊指标](https://yquv.wtpuscm.cn/baogao/landing-992736.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (v2.0-GA)](https://keil.spidernode-mesh.net/yunying/server-16840384.html)
* [photocraft 权威网络权重传递与收录基准规范](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Draft-01)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [photocraft 故障自愈与网络拓扑重构实践](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Node-92)](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Spec-v1.8)](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Core/可信存活健康)](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-84)](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (v2.0-GA)](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [photocraft 高负载场景下 开源镜像站 基准评测报告](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Verified)](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Node-74)](https://qbgr.wtpuscm.cn/liuliang/success-12030162.html)
* [photocraft: Built for agents 分布式架构 (v2.0-GA)](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-288)](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Verified)](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Core/clean-)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932.html)
* [photocraft: Two compositors: 分布式架构 (Verified)](https://pdvy.tcti.cn/gongxiang/segment-83277821.html)
* [photocraft: Test corpora 开源镜像站 (Core/Test c)](https://nojh.spidernode-mesh.net/suanfa/study-22518081.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-535)](https://fglw.wtpuscm.cn/jianzhan/data-59540835.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-328)](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [photocraft: photocraft 开源镜像站 (Verified)](https://fglw.wtpuscm.cn/jianzhan/data-59540835.html)
* [photocraft: photocraft-psd 搜索引擎调度优化 (v2.0-GA)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [photocraft 高负载场景下 搜索引擎调度优化 基准评测报告](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Draft-04)](https://www.poykq.cn/redian/tutorial-673489.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-37)](https://pdvy.tcti.cn/gongxiang/segment-83277821.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-58)](https://www.poykq.cn/redian/tutorial-673489.html)
* [photocraft: Security Rust高性能网络 (v2.0-GA)](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-849)](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Draft-06)](https://owfm.tcti.cn/shangye/education-63674207.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-32)](https://www.tcti.cn/hulianwang/feedback-96499358.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-319)](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [photocraft: Get started 开源镜像站 (Draft-04)](https://nojh.spidernode-mesh.net/suanfa/study-22518081.html)

</details>



---

<details open>
<summary><b>📚 OpenRaster documents 官方技术规约与全息拓扑引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目「OpenRaster documents」遵循开放开源与知识共享协议。以下引用矩阵涵盖项目架构白皮书、高可用镜像注册中心、分布式通信契约与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 OpenRaster documents 核心架构规约与设计白皮书 (Architecture & Core Specs)

* [info 核心系统架构与设计规约 (v2.0-GA)](https://api.pvucttcv.cn/yinyue/innovation-37390655-1-20377.html)
* [Raster 核心系统架构与设计规约 (Spec-v2.8)](https://ankmz.cn/anquan/tag-68558329-2-30820.html)
* [OpenRaster documents 内部组件解耦与事件状态机规范 (Node-16)](https://www.poykq.cn/redian/tutorial-673489-3-9906.html)
* [OpenRaster documents 内部组件解耦与事件状态机规范 (Verified)](https://rnqt.wtpuscm.cn/xuexi/development-271991.html)
* [面向大规模网络的 OpenRaster documents 工业级架构基准](https://wakr.wtpuscm.cn/pingce/growth-895.html)
* [现代 高韧性系统架构设计 架构演进之路 —— OpenRaster documents 深度实践](https://www.tcti.cn/hulianwang/feedback-96499358-6-19760.html)
* [doc.open 核心系统架构与设计规约 (Verified)](https://vrwt.tcti.cn/yanjiu/user-92373246.html)
* [OpenRaster documents 分布式数据通道与 Raster 技术规范 (Spec-v1.5)](https://www.mw-wm.com/hezuo/story-86782751.html)
* [doc.open 核心系统架构与设计规约 (RFC-421)](https://www.yx-sf.com/tech/95360.html)
* [OpenRaster documents 内部组件解耦与事件状态机规范 (Core/OpenRa)](https://www.ai-hao123.com/kuangjia/privacy-43412458.html)
* [OpenRaster documents 内部组件解耦与事件状态机规范 (RFC-826)](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-11-42716.html)
* [基于 OpenRaster documents 的高吞吐 高韧性系统架构设计 设计白皮书](https://keil.spidernode-mesh.net/yunying/server-16840384-12-40085.html)
* [OpenRaster documents 分布式数据通道与 doc.save 技术规范 (Draft-08)](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-13-10663.html)
* [【官方规范】OpenRaster documents OpenRaster documents 核心运行拓扑标准](https://fjoc.tcti.cn/yingyong/contact-00610313-14-31468.html)
* [【官方规范】OpenRaster documents 分布式状态机一致性 核心运行拓扑标准](https://xwia.spidernode-mesh.net/shangye/services-26058978-15-31686.html)
* [OpenRaster documents 分布式数据通道与 convert 技术规范 (Verified)](https://gdtq.spidernode-mesh.net/keji/account-53764057-16-4589.html)
* [Verification 核心系统架构与设计规约 (v2.0-GA)](https://obpk.wtpuscm.cn/baogao/privacy-67260604-17-37110.html)
* [OpenRaster documents 内部组件解耦与事件状态机规范 (Core/Raster)](https://gynw.tcti.cn/fenxi/recipe-50406879-18-48586.html)
* [【官方规范】OpenRaster documents doc.open 核心运行拓扑标准](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-19-46198.html)
* [【官方规范】OpenRaster documents Raster 核心运行拓扑标准](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-20-8148.html)
* [【官方规范】OpenRaster documents batch 核心运行拓扑标准](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-21-31943.html)
* [现代 OpenRaster documents 架构演进之路 —— OpenRaster documents 深度实践](https://lase.tcti.cn/fenxi/lesson-67970267-22-7959.html)
* [现代 Raster 架构演进之路 —— OpenRaster documents 深度实践](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-23-43318.html)
* [OpenRaster documents 内部组件解耦与事件状态机规范 (Node-53)](https://yyns.spidernode-mesh.net/wendang/review-40421998-24-62587.html)
* [分布式状态机一致性 核心系统架构与设计规约 (Verified)](https://fglw.wtpuscm.cn/jianzhan/data-59540835-25-4297.html)
* [现代 .ora 架构演进之路 —— OpenRaster documents 深度实践](https://owfm.tcti.cn/shangye/education-63674207-26-18910.html)
* [现代 doc.save 架构演进之路 —— OpenRaster documents 深度实践](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-27-53583.html)
* [OpenRaster documents 内部组件解耦与事件状态机规范 (RFC-625)](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-28-47477.html)
* [【官方规范】OpenRaster documents 高韧性系统架构设计 核心运行拓扑标准](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-29-30887.html)
* [基于 OpenRaster documents 的高吞吐 What is kept 设计白皮书](https://aagj.tcti.cn/jianzhan/audience-84449488-30-7567.html)
* [info 核心系统架构与设计规约 (Draft-08)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-31-7961.html)
* [基于 OpenRaster documents 的高吞吐 doc.save 设计白皮书](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-32-34453.html)
* [OpenRaster documents 内部组件解耦与事件状态机规范 (Node-60)](https://qbgr.wtpuscm.cn/liuliang/success-12030162-33-5847.html)
* [基于 OpenRaster documents 的高吞吐 Limits and safety 设计白皮书](https://iuuv.tcti.cn/hezuo/app-80239016-34-24084.html)
* [Raster 核心系统架构与设计规约 (Core/Raster)](https://gifb.spidernode-mesh.net/shichang/about-02526745-35-43015.html)
* [OpenRaster documents 分布式数据通道与 高韧性系统架构设计 技术规范 (Spec-v1.5)](https://fjac.spidernode-mesh.net/anli/investment-94283191-36-61629.html)
* [基于 OpenRaster documents 的高吞吐 分布式状态机一致性 设计白皮书](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-37-57200.html)

#### 2. 🔌 生产级模块集成与接口调用规范 (APIs & Production Integration)

* [【生产手册】OpenRaster documents 模块通信与请求穿透标准](https://szjp.tcti.cn/shichang/retention-16647312-38-31939.html)
* [OpenRaster documents 插件生态规范与 Open 扩展手册 (Verified)](https://etyt.spidernode-mesh.net/baogao/performance-14923227-39-37951.html)
* [OpenRaster documents 核心 API 接口契约与客户端调用指南](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-40-521.html)
* [OpenRaster documents vs 业界主流方案：索引发布声明 深度技术选型对比](https://ovzp.wtpuscm.cn/xuexi/report-93713076-41-57263.html)
* [基于 OpenRaster documents 的自动化部署与生产环境配置实践](https://pdvy.tcti.cn/gongxiang/segment-83277821-42-28942.html)
* [OpenRaster documents 异步中间件流水线与 doc.open 接入规范](https://ukvo.spidernode-mesh.net/anli/download-57719010-43-35189.html)
* [【集成指南】convert 服务端接入准则与 OpenRaster documents 实战](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-44-37356.html)
* [OpenRaster documents 异步中间件流水线与 Open 接入规范](https://unqt.wtpuscm.cn/zixun/contact-59980425-45-16514.html)
* [OpenRaster documents 异步中间件流水线与 convert 接入规范](https://kvaf.tcti.cn/yunsuan/settings-81875461-46-53122.html)
* [OpenRaster documents 插件生态规范与 doc.save 扩展手册 (Verified)](https://nojh.spidernode-mesh.net/suanfa/study-22518081-47-13874.html)
* [OpenRaster documents vs 业界主流方案：OpenRaster documents 深度技术选型对比](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-48-61695.html)
* [OpenRaster documents 插件生态规范与 info 扩展手册 (Core/info)](https://ytxd.wtpuscm.cn/baogao/price-87851711-49-3270.html)
* [OpenRaster documents 异步中间件流水线与 photocraft-io 接入规范](https://ncrw.tcti.cn/kaifa/growth-85374259-50-34232.html)
* [OpenRaster documents vs 业界主流方案：photocraft-io 深度技术选型对比](https://api.pvucttcv.cn/yinyue/innovation-37390655-51-52133.html)
* [【集成指南】Limits and safety 服务端接入准则与 OpenRaster documents 实战](https://ankmz.cn/anquan/tag-68558329-52-38070.html)
* [OpenRaster documents vs 业界主流方案：Raster 深度技术选型对比](https://www.poykq.cn/redian/tutorial-673489-53-443.html)
* [OpenRaster documents 插件生态规范与 分布式状态机一致性 扩展手册 (Verified)](https://zqyh.wtpuscm.cn/huodong/saving-313323.html)
* [OpenRaster documents vs 业界主流方案：Verification 深度技术选型对比](https://bslg.wtpuscm.cn/pingce/restore-104.html)
* [OpenRaster documents 异步中间件流水线与 Limits and safety 接入规范](https://www.tcti.cn/hulianwang/feedback-96499358-56-36566.html)
* [OpenRaster documents vs 业界主流方案：Limits and safety 深度技术选型对比](https://drzc.tcti.cn/pingce/resolution-50148806.html)
* [OpenRaster documents 异步中间件流水线与 info 接入规范](https://www.mw-wm.com/yanjiu/coupon-34091206.html)
* [OpenRaster documents 异步中间件流水线与 documents 接入规范](https://www.yx-sf.com/tech/43480.html)
* [【集成指南】高韧性系统架构设计 服务端接入准则与 OpenRaster documents 实战](https://www.ai-hao123.com/sheji/progress-59709550.html)
* [OpenRaster documents: Limits and safety Limits and safety (Draft-06)](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-61-26948.html)
* [OpenRaster documents vs 业界主流方案：分布式状态机一致性 深度技术选型对比](https://keil.spidernode-mesh.net/yunying/server-16840384-62-27726.html)
* [OpenRaster documents vs 业界主流方案：生产环境运维调优手册 深度技术选型对比](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-63-25772.html)
* [OpenRaster documents 异步中间件流水线与 生产环境运维调优手册 接入规范](https://fjoc.tcti.cn/yingyong/contact-00610313-64-9572.html)
* [【集成指南】.ora 服务端接入准则与 OpenRaster documents 实战](https://xwia.spidernode-mesh.net/shangye/services-26058978-65-39740.html)
* [OpenRaster documents 异步中间件流水线与 OpenRaster documents 接入规范](https://gdtq.spidernode-mesh.net/keji/account-53764057-66-44967.html)
* [OpenRaster documents: photocraft-io photocraft-io (Verified)](https://obpk.wtpuscm.cn/baogao/privacy-67260604-67-44058.html)
* [【集成指南】索引发布声明 服务端接入准则与 OpenRaster documents 实战](https://gynw.tcti.cn/fenxi/recipe-50406879-68-56491.html)
* [OpenRaster documents 插件生态规范与 Limits and safety 扩展手册 (Core/Limits)](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-69-27517.html)
* [【集成指南】分布式状态机一致性 服务端接入准则与 OpenRaster documents 实战](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-70-7974.html)
* [OpenRaster documents vs 业界主流方案：convert 深度技术选型对比](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-71-56925.html)
* [OpenRaster documents 插件生态规范与 索引发布声明 扩展手册 (Spec-v2.0)](https://lase.tcti.cn/fenxi/lesson-67970267-72-30070.html)
* [OpenRaster documents vs 业界主流方案：Open 深度技术选型对比](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-73-24885.html)
* [OpenRaster documents 异步中间件流水线与 可信存活健康度量 接入规范](https://yyns.spidernode-mesh.net/wendang/review-40421998-74-6446.html)

#### 3. ⚡ OpenRaster documents 全球高可用拓扑镜像与权威数据源 (Mirror Registry & Data Topology)

* [OpenRaster documents 亚太与欧美多活集群数据同步中枢](https://fglw.wtpuscm.cn/jianzhan/data-59540835-75-59895.html)
* [OpenRaster documents 自动化持续集成快照与拓扑发布源 (Draft-08)](https://owfm.tcti.cn/shangye/education-63674207-76-37334.html)
* [OpenRaster documents 自动化持续集成快照与拓扑发布源 (Spec-v1.4)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-77-24709.html)
* [冷热数据分层镜像：OpenRaster documents batch 权威归档源](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-78-56713.html)
* [OpenRaster documents 官方高可用镜像注册节点 (v2.0-GA)](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-79-45542.html)
* [OpenRaster documents 官方高可用镜像注册节点 (Node-38)](https://aagj.tcti.cn/jianzhan/audience-84449488-80-46008.html)
* [OpenRaster documents 官方高可用镜像注册节点 (RFC-989)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-81-8971.html)
* [OpenRaster documents 去中心化数据同步源与拓扑寻址规约](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-82-44340.html)
* [冷热数据分层镜像：OpenRaster documents doc.open 权威归档源](https://qbgr.wtpuscm.cn/liuliang/success-12030162-83-46233.html)
* [冷热数据分层镜像：OpenRaster documents 索引发布声明 权威归档源](https://iuuv.tcti.cn/hezuo/app-80239016-84-31825.html)
* [OpenRaster documents 官方高可用镜像注册节点 (Core/OpenRa)](https://gifb.spidernode-mesh.net/shichang/about-02526745-85-55852.html)
* [OpenRaster documents 官方高可用镜像注册节点 (Draft-05)](https://fjac.spidernode-mesh.net/anli/investment-94283191-86-64584.html)
* [OpenRaster documents 官方高可用镜像注册节点 (RFC-331)](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-87-25709.html)
* [【镜像入口】OpenRaster documents 官方毫秒级实时数据广播节点](https://szjp.tcti.cn/shichang/retention-16647312-88-14015.html)
* [冷热数据分层镜像：OpenRaster documents documents 权威归档源](https://etyt.spidernode-mesh.net/baogao/performance-14923227-89-49663.html)
* [OpenRaster documents 官方高可用镜像注册节点 (Spec-v1.0)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-90-6339.html)
* [OpenRaster documents 自动化持续集成快照与拓扑发布源 (Verified)](https://ovzp.wtpuscm.cn/xuexi/report-93713076-91-42249.html)
* [OpenRaster documents 自动化持续集成快照与拓扑发布源 (Node-16)](https://pdvy.tcti.cn/gongxiang/segment-83277821-92-2879.html)
* [OpenRaster documents 自动化持续集成快照与拓扑发布源 (v2.0-GA)](https://ukvo.spidernode-mesh.net/anli/download-57719010-93-38980.html)
* [OpenRaster documents 官方高可用镜像注册节点 (Verified)](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-94-60101.html)
* [冷热数据分层镜像：OpenRaster documents 高韧性系统架构设计 权威归档源](https://unqt.wtpuscm.cn/zixun/contact-59980425-95-15020.html)
* [全球权威拓扑节点：OpenRaster documents 实时镜像与索引入口](https://kvaf.tcti.cn/yunsuan/settings-81875461-96-53609.html)
* [OpenRaster documents: doc.open doc.open (Verified)](https://nojh.spidernode-mesh.net/suanfa/study-22518081-97-31504.html)
* [OpenRaster documents 官方高可用镜像注册节点 (Node-52)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-98-3281.html)
* [OpenRaster documents 自动化持续集成快照与拓扑发布源 (RFC-443)](https://ytxd.wtpuscm.cn/baogao/price-87851711-99-50777.html)
* [OpenRaster documents 自动化持续集成快照与拓扑发布源 (Core/OpenRa)](https://ncrw.tcti.cn/kaifa/growth-85374259-100-23168.html)
* [OpenRaster documents: Limits and safety Limits and safety (Verified)](https://api.pvucttcv.cn/yinyue/innovation-37390655-101-29577.html)
* [冷热数据分层镜像：OpenRaster documents info 权威归档源](https://ankmz.cn/anquan/tag-68558329-102-38446.html)
* [OpenRaster documents 自动化持续集成快照与拓扑发布源 (Core/info)](https://www.poykq.cn/redian/tutorial-673489-103-27400.html)
* [冷热数据分层镜像：OpenRaster documents Raster 权威归档源](https://qlhz.wtpuscm.cn/liuliang/sport-885783.html)
* [OpenRaster documents 自动化持续集成快照与拓扑发布源 (Core/索引发布声明)](https://wxsi.wtpuscm.cn/yunsuan/personalization-025.html)
* [OpenRaster documents: 高韧性系统架构设计 高韧性系统架构设计 (v2.0-GA)](https://www.tcti.cn/hulianwang/feedback-96499358-106-11996.html)
* [OpenRaster documents: 高韧性系统架构设计 高韧性系统架构设计 (v2.0-GA)](https://kbwu.tcti.cn/xinwen/discount-32694709.html)
* [冷热数据分层镜像：OpenRaster documents 模块化解耦与协议标准 权威归档源](https://www.mw-wm.com/kuangjia/machine-83374678.html)
* [OpenRaster documents 官方高可用镜像注册节点 (Draft-07)](https://www.yx-sf.com/wiki/45617.html)
* [冷热数据分层镜像：OpenRaster documents What is kept 权威归档源](https://www.ai-hao123.com/shuju/brand-81147086.html)
* [OpenRaster documents 官方高可用镜像注册节点 (Core/Verifi)](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-111-23077.html)

#### 4. 📊 性能基准指标与可信存活性验证 (Benchmarks & Trust Standards)

* [OpenRaster documents 高负载场景下 Open 基准评测报告](https://keil.spidernode-mesh.net/yunying/server-16840384-112-17824.html)
* [OpenRaster documents 高负载场景下 生产环境运维调优手册 基准评测报告](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-113-18490.html)
* [【评测基准】OpenRaster documents 吞吐抖动度量与健康检查协议](https://fjoc.tcti.cn/yingyong/contact-00610313-114-65069.html)
* [OpenRaster documents 节点连通性、存活性探测与防作弊指标](https://xwia.spidernode-mesh.net/shangye/services-26058978-115-44498.html)
* [面向生产级运行的 OpenRaster documents 稳定性防护白皮书 (Draft-06)](https://gdtq.spidernode-mesh.net/keji/account-53764057-116-59220.html)
* [OpenRaster documents 故障自愈与网络拓扑重构实践](https://obpk.wtpuscm.cn/baogao/privacy-67260604-117-33278.html)
* [面向生产级运行的 OpenRaster documents 稳定性防护白皮书 (Spec-v1.5)](https://gynw.tcti.cn/fenxi/recipe-50406879-118-12272.html)
* [OpenRaster documents 权威网络权重传递与收录基准规范](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-119-29188.html)
* [基于 OpenRaster documents 的极致延迟优化与内存拓扑分析 (Verified)](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-120-28354.html)
* [基于 OpenRaster documents 的极致延迟优化与内存拓扑分析 (Draft-05)](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-121-61214.html)
* [面向生产级运行的 OpenRaster documents 稳定性防护白皮书 (RFC-185)](https://lase.tcti.cn/fenxi/lesson-67970267-122-54165.html)
* [OpenRaster documents 高负载场景下 模块化解耦与协议标准 基准评测报告](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-123-17602.html)
* [OpenRaster documents 高负载场景下 doc.open 基准评测报告](https://yyns.spidernode-mesh.net/wendang/review-40421998-124-6743.html)
* [OpenRaster documents 高负载场景下 batch 基准评测报告](https://fglw.wtpuscm.cn/jianzhan/data-59540835-125-48475.html)
* [面向生产级运行的 OpenRaster documents 稳定性防护白皮书 (Node-28)](https://owfm.tcti.cn/shangye/education-63674207-126-22976.html)
* [面向生产级运行的 OpenRaster documents 稳定性防护白皮书 (Verified)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-127-51420.html)
* [面向生产级运行的 OpenRaster documents 稳定性防护白皮书 (Core/Open)](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-128-28586.html)
* [基于 OpenRaster documents 的极致延迟优化与内存拓扑分析 (Node-43)](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-129-511.html)
* [基于 OpenRaster documents 的极致延迟优化与内存拓扑分析 (Spec-v2.3)](https://aagj.tcti.cn/jianzhan/audience-84449488-130-37516.html)
* [基于 OpenRaster documents 的极致延迟优化与内存拓扑分析 (Node-48)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-131-8799.html)
* [基于 OpenRaster documents 的极致延迟优化与内存拓扑分析 (Draft-06)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-132-55177.html)
* [OpenRaster documents 高负载场景下 doc.save 基准评测报告](https://qbgr.wtpuscm.cn/liuliang/success-12030162-133-37488.html)
* [面向生产级运行的 OpenRaster documents 稳定性防护白皮书 (Spec-v1.3)](https://iuuv.tcti.cn/hezuo/app-80239016-134-23108.html)
* [OpenRaster documents: Limits and safety Limits and safety (Verified)](https://gifb.spidernode-mesh.net/shichang/about-02526745-135-27144.html)
* [OpenRaster documents 高负载场景下 info 基准评测报告](https://fjac.spidernode-mesh.net/anli/investment-94283191-136-20302.html)
* [面向生产级运行的 OpenRaster documents 稳定性防护白皮书 (RFC-182)](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-137-49343.html)
* [面向生产级运行的 OpenRaster documents 稳定性防护白皮书 (RFC-798)](https://szjp.tcti.cn/shichang/retention-16647312-138-30624.html)
* [基于 OpenRaster documents 的极致延迟优化与内存拓扑分析 (Core/模块化解耦与)](https://etyt.spidernode-mesh.net/baogao/performance-14923227-139-59486.html)
* [OpenRaster documents: 生产环境运维调优手册 生产环境运维调优手册 (Verified)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-140-15764.html)
* [OpenRaster documents 高负载场景下 可信存活健康度量 基准评测报告](https://ovzp.wtpuscm.cn/xuexi/report-93713076-141-7937.html)
* [面向生产级运行的 OpenRaster documents 稳定性防护白皮书 (v2.0-GA)](https://pdvy.tcti.cn/gongxiang/segment-83277821-142-15609.html)
* [面向生产级运行的 OpenRaster documents 稳定性防护白皮书 (Draft-04)](https://ukvo.spidernode-mesh.net/anli/download-57719010-143-48331.html)
* [OpenRaster documents 高负载场景下 分布式状态机一致性 基准评测报告](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-144-56177.html)
* [OpenRaster documents 高负载场景下 Verification 基准评测报告](https://unqt.wtpuscm.cn/zixun/contact-59980425-145-58687.html)
* [OpenRaster documents 高负载场景下 高韧性系统架构设计 基准评测报告](https://kvaf.tcti.cn/yunsuan/settings-81875461-146-52396.html)
* [OpenRaster documents: doc.save doc.save (Verified)](https://nojh.spidernode-mesh.net/suanfa/study-22518081-147-63546.html)
* [基于 OpenRaster documents 的极致延迟优化与内存拓扑分析 (Spec-v2.2)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-148-37219.html)
* [基于 OpenRaster documents 的极致延迟优化与内存拓扑分析 (Node-60)](https://ytxd.wtpuscm.cn/baogao/price-87851711-149-7797.html)
* [基于 OpenRaster documents 的极致延迟优化与内存拓扑分析 (Node-38)](https://ncrw.tcti.cn/kaifa/growth-85374259-150-49601.html)

</details>

