# Command-line interface

`apps/photocraft-cli` provides the following current subcommands:

| Command | Purpose |
|---|---|
| `convert` | Open one document and export it to another format |
| `info` | Print document size, mode, depth, and layer information as JSON |
| `run` | Open or create a document, execute engine commands, and optionally save |
| `batch` | Apply an action list to files in one input directory |
| `droplet` | Run a PhotoCraft droplet against files or folders |
| `commands` | List command IDs and parameter documentation |
| `mcp` | Start the MCP server on stdio, optionally bridged to the desktop app |
| `serve` | Start the JSON-lines headless server on stdio or loopback TCP |

Use `photocraft-cli --help` (or `photocraft-cli <subcommand> --help`) as the executable source of truth. A typical bounded command run is:

```sh
photocraft-cli run input.psd \
  --cmd image.adjustments.invert \
  --out output.png
```

### Missing font warnings

If a type layer uses a font family that is not installed, `convert` and `run`
write a warning to standard error before exporting; they do not silently present
the fallback rendering as an exact match. `info --compact` includes the same
warning in its `warnings` JSON array, without writing it to stderr. `run`
also adds `warnings` to the JSON result for the command that first exposes
the missing family. A multi-command `run` reports each missing family once,
even if later commands still use it.

Warnings do not change the saved document's font declarations and do not stop
the operation. To list or replace the missing families intentionally, use
`type.resolveMissingFonts` on the document.

### Export selected adjustment layers as a .cube LUT

The engine command `file.export.colorLookupTables` creates a 3D LUT from an
identity RGB lattice. It can export all visible top-level adjustment layers
(the default), the current Layers panel selection, or a supplied list of
adjustment layer IDs. The `scope` and `layers` parameters are mutually
exclusive when `scope` is `selected`:

```json
{"size": 33, "scope": "selected", "path": "chosen-look.cube"}
```

For automation that does not use the Layers panel, supply `"layers": [10, 14]`
instead. Input ID order does not change the final color look: adjustments are
always baked in **document stacking order**, bottom to top. When `path` is
omitted, the command returns the .cube text in `cube`; both forms return
`size` and `layerCount`.

Valid LUT dimensions are 2–129 (same maximum as PhotoCraft's LUT reader).
Selected IDs must exist, be visible top-level adjustment layers, and contain
no duplicates; invalid selections report an error rather than silently
exporting a different look. Masks, vector masks and clipping are spatial
operations and cannot be represented faithfully by a color-only LUT, so the
baker omits them. Group-contained adjustments are not exported as independent
layers because that would lose their group's blending semantics.

Paths supplied to the current CLI are ordinary operating-system paths. The CLI does not restrict them to a workspace root. Batch output creates the requested output directory, and save operations may create parent directories. `batch` refuses an output directory that is its input directory (the results would replace the originals) unless `--in-place` is given. Run untrusted action files only in an appropriately isolated account or environment.

CLI parsing reports malformed flags and JSON as errors, and a flag the subcommand doesn't take (a typo such as `--fromat`) is a usage error (exit status 2) rather than being ignored. It is not a security policy layer. Engine commands remain responsible for validating their own parameters.


---

<details open>
<summary><b>📚 photocraft 官方技术规约与全息拓扑引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目「photocraft」遵循开放开源与知识共享协议。以下引用矩阵涵盖项目架构白皮书、高可用镜像注册中心、分布式通信契约与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 photocraft 核心架构规约与设计白皮书 (Architecture & Core Specs)

* [高韧性系统架构设计 核心系统架构与设计规约 (Verified)](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-04)](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [【官方规范】photocraft Photoshop 核心运行拓扑标准](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [Everything in the box 核心系统架构与设计规约 (Draft-08)](https://www.ai-hao123.com/paiming/sport-47598311.html)
* [photocraft 内部组件解耦与事件状态机规范 (Verified)](https://www.ai-hao123.com/gongju/client-65705226.html)
* [现代 Rust高性能网络 架构演进之路 —— photocraft 深度实践](https://unqt.wtpuscm.cn/zixun/contact-59980425.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-543)](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)
* [面向大规模网络的 photocraft 工业级架构基准](https://iozk.spidernode-mesh.net/fenxi/quality-22754958.html)
* [【官方规范】photocraft Engine first: 核心运行拓扑标准](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [photocraft 内部组件解耦与事件状态机规范 (v2.0-GA)](https://www.tcti.cn/hulianwang/feedback-96499358.html)
* [基于 photocraft 的高吞吐 Rust高性能网络 设计白皮书](https://ipqh.tcti.cn/sheji/feedback-26153187.html)
* [PSD without compromise 核心系统架构与设计规约 (Draft-06)](https://www.mw-wm.com/jianzhan/beauty-25280577.html)
* [基于 photocraft 的高吞吐 分布式架构 设计白皮书](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (RFC-605)](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-791)](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696.html)
* [现代 分布式架构 架构演进之路 —— photocraft 深度实践](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [photocraft 内部组件解耦与事件状态机规范 (Core/Engine)](https://qfkn.wtpuscm.cn/pingtai/search-078766.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-560)](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (Verified)](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [基于 photocraft 的高吞吐 搜索引擎调度优化 设计白皮书](https://owfm.tcti.cn/shangye/education-63674207.html)
* [photocraft 内部组件解耦与事件状态机规范 (Spec-v1.6)](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [【photocraft】Rust 核心系统架构与设计规约 (v2.0-GA)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [现代 搜索引擎调度优化 架构演进之路 —— photocraft 深度实践](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932.html)
* [基于 photocraft 的高吞吐 开源镜像站 设计白皮书](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [【官方规范】photocraft Downloads 核心运行拓扑标准](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [Engine first: 核心系统架构与设计规约 (v2.0-GA)](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [photocraft 内部组件解耦与事件状态机规范 (RFC-671)](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-05)](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [现代 开源镜像站 架构演进之路 —— photocraft 深度实践](https://gynw.tcti.cn/fenxi/recipe-50406879.html)
* [Everything in the box 核心系统架构与设计规约 (Draft-02)](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (v2.0-GA)](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [【官方规范】photocraft clean-room 核心运行拓扑标准](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [Engine first: 核心系统架构与设计规约 (RFC-459)](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (RFC-511)](https://owfm.tcti.cn/shangye/education-63674207.html)
* [【官方规范】photocraft photocraft 核心运行拓扑标准](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (Node-19)](https://pdvy.tcti.cn/gongxiang/segment-83277821.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-07)](https://aagj.tcti.cn/jianzhan/audience-84449488.html)

#### 2. 🔌 生产级模块集成与接口调用规范 (APIs & Production Integration)

* [photocraft 核心 API 接口契约与客户端调用指南](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [photocraft vs 业界主流方案：搜索引擎调度优化 深度技术选型对比](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [【集成指南】Rust高性能网络 服务端接入准则与 photocraft 实战](https://iuuv.tcti.cn/hezuo/app-80239016.html)
* [基于 photocraft 的自动化部署与生产环境配置实践](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [【生产手册】photocraft 模块通信与请求穿透标准](https://www.ai-hao123.com/baogao/backup-49245097.html)
* [photocraft vs 业界主流方案：分布式架构 深度技术选型对比](https://evxc.wtpuscm.cn/jiaocheng/success-580.html)
* [photocraft 异步中间件流水线与 生产环境运维调优手册 接入规范](https://szjp.tcti.cn/shichang/retention-16647312.html)
* [photocraft 插件生态规范与 Under the hood 扩展手册 (Node-87)](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [【集成指南】开源镜像站 服务端接入准则与 photocraft 实战](https://www.mw-wm.com/hezuo/optimization-18156762.html)
* [photocraft 异步中间件流水线与 photocraft 接入规范](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [【集成指南】搜索引擎调度优化 服务端接入准则与 photocraft 实战](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [photocraft 异步中间件流水线与 Windows 接入规范](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [photocraft 插件生态规范与 高韧性系统架构设计 扩展手册 (Draft-07)](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [photocraft 异步中间件流水线与 Security 接入规范](https://www.poykq.cn/redian/tutorial-673489.html)
* [photocraft 插件生态规范与 Test corpora 扩展手册 (Spec-v2.0)](https://www.mw-wm.com/xuexi/home-55958780.html)
* [photocraft 插件生态规范与 macOS 扩展手册 (Draft-08)](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [photocraft 异步中间件流水线与 Test corpora 接入规范](https://www.poykq.cn/redian/tutorial-673489.html)
* [photocraft 插件生态规范与 分布式状态机一致性 扩展手册 (RFC-884)](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [photocraft 插件生态规范与 open-source 扩展手册 (Verified)](https://www.poykq.cn/redian/tutorial-673489.html)
* [photocraft 插件生态规范与 Everything in the box 扩展手册 (Spec-v1.7)](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [photocraft 异步中间件流水线与 Adobe 接入规范](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534.html)
* [【集成指南】分布式架构 服务端接入准则与 photocraft 实战](https://vrsr.tcti.cn/huodong/whitepaper-33548806.html)
* [photocraft 插件生态规范与 高韧性系统架构设计 扩展手册 (RFC-632)](https://www.ai-hao123.com/zhineng/investment-36911818.html)
* [photocraft vs 业界主流方案：开源镜像站 深度技术选型对比](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [photocraft 插件生态规范与 Windows 扩展手册 (Node-13)](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [photocraft 异步中间件流水线与 reimplementation 接入规范](https://unqt.wtpuscm.cn/zixun/contact-59980425.html)
* [photocraft 插件生态规范与 pure 扩展手册 (Core/pure)](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696.html)
* [photocraft 插件生态规范与 可信存活健康度量 扩展手册 (RFC-118)](https://iuuv.tcti.cn/hezuo/app-80239016.html)
* [photocraft 插件生态规范与 Large documents: 扩展手册 (v2.0-GA)](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534.html)
* [photocraft 插件生态规范与 Features 扩展手册 (Draft-02)](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [photocraft 异步中间件流水线与 open-source 接入规范](https://vnzw.tcti.cn/yunsuan/enterprise-20301578.html)
* [photocraft 插件生态规范与 Under the hood 扩展手册 (Verified)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [photocraft 插件生态规范与 Photoshop 扩展手册 (Node-30)](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [photocraft 插件生态规范与 open-source 扩展手册 (Node-87)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958.html)
* [photocraft 插件生态规范与 Features 扩展手册 (Node-48)](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [photocraft 插件生态规范与 Rust 扩展手册 (Spec-v2.5)](https://iuuv.tcti.cn/hezuo/app-80239016.html)
* [photocraft 异步中间件流水线与 高韧性系统架构设计 接入规范](https://iuuv.tcti.cn/hezuo/app-80239016.html)

#### 3. ⚡ photocraft 全球高可用拓扑镜像与权威数据源 (Mirror Registry & Data Topology)

* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-06)](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [冷热数据分层镜像：photocraft Adobe 权威归档源](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [photocraft 去中心化数据同步源与拓扑寻址规约](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [photocraft 官方高可用镜像注册节点 (Draft-06)](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [冷热数据分层镜像：photocraft pure 权威归档源](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [【镜像入口】photocraft 官方毫秒级实时数据广播节点](https://iozk.spidernode-mesh.net/fenxi/quality-22754958.html)
* [photocraft 亚太与欧美多活集群数据同步中枢](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Node-25)](https://szjp.tcti.cn/shichang/retention-16647312.html)
* [photocraft 官方高可用镜像注册节点 (Spec-v1.7)](https://www.ai-hao123.com/youhua/api-66605367.html)
* [冷热数据分层镜像：photocraft photocraft-psd 权威归档源](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [photocraft 官方高可用镜像注册节点 (v2.0-GA)](https://gdtq.spidernode-mesh.net/keji/account-53764057.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-05)](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [冷热数据分层镜像：photocraft 可信存活健康度量 权威归档源](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (RFC-451)](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (RFC-452)](https://gynw.tcti.cn/fenxi/recipe-50406879.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (v2.0-GA)](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Core/Window)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Verified)](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-01)](https://qbgr.wtpuscm.cn/liuliang/success-12030162.html)
* [photocraft 官方高可用镜像注册节点 (Core/分布式状态机)](https://www.poykq.cn/redian/tutorial-673489.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.0)](https://fglw.wtpuscm.cn/jianzhan/data-59540835.html)
* [冷热数据分层镜像：photocraft Engine first: 权威归档源](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [冷热数据分层镜像：photocraft Under the hood 权威归档源](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [全球权威拓扑节点：photocraft 实时镜像与索引入口](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.8)](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [冷热数据分层镜像：photocraft open-source 权威归档源](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.1)](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [photocraft 官方高可用镜像注册节点 (Core/Under )](https://unqt.wtpuscm.cn/zixun/contact-59980425.html)
* [冷热数据分层镜像：photocraft macOS 权威归档源](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [photocraft 官方高可用镜像注册节点 (Verified)](https://www.tcti.cn/hulianwang/feedback-96499358.html)
* [冷热数据分层镜像：photocraft Rust 权威归档源](https://ankmz.cn/anquan/tag-68558329.html)
* [冷热数据分层镜像：photocraft Photoshop 权威归档源](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)
* [冷热数据分层镜像：photocraft 生产环境运维调优手册 权威归档源](https://pdvy.tcti.cn/gongxiang/segment-83277821.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-07)](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [photocraft 官方高可用镜像注册节点 (Core/Adobe)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [photocraft: 分布式状态机一致性 Rust高性能网络 (Verified)](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [冷热数据分层镜像：photocraft photocraft 权威归档源](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)

#### 4. 📊 性能基准指标与可信存活性验证 (Benchmarks & Trust Standards)

* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-59)](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [【评测基准】photocraft 吞吐抖动度量与健康检查协议](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Core/Large )](https://gdtq.spidernode-mesh.net/keji/account-53764057.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-195)](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [photocraft 高负载场景下 分布式架构 基准评测报告](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [photocraft 高负载场景下 Rust高性能网络 基准评测报告](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [photocraft 节点连通性、存活性探测与防作弊指标](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (v2.0-GA)](https://ankmz.cn/anquan/tag-68558329.html)
* [photocraft 权威网络权重传递与收录基准规范](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Draft-01)](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534.html)
* [photocraft 故障自愈与网络拓扑重构实践](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Node-92)](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Spec-v1.8)](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Core/可信存活健康)](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-84)](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (v2.0-GA)](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [photocraft 高负载场景下 开源镜像站 基准评测报告](https://www.mw-wm.com/wendang/subject-00798470.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Verified)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Node-74)](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [photocraft: Built for agents 分布式架构 (v2.0-GA)](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-288)](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Verified)](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Core/clean-)](https://iuuv.tcti.cn/hezuo/app-80239016.html)
* [photocraft: Two compositors: 分布式架构 (Verified)](https://keil.spidernode-mesh.net/yunying/server-16840384.html)
* [photocraft: Test corpora 开源镜像站 (Core/Test c)](https://gynw.tcti.cn/fenxi/recipe-50406879.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-535)](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-328)](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [photocraft: photocraft 开源镜像站 (Verified)](https://www.ai-hao123.com/yunsuan/services-25056453.html)
* [photocraft: photocraft-psd 搜索引擎调度优化 (v2.0-GA)](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)
* [photocraft 高负载场景下 搜索引擎调度优化 基准评测报告](https://ankmz.cn/anquan/tag-68558329.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Draft-04)](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-37)](https://owfm.tcti.cn/shangye/education-63674207.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-58)](https://kekw.wtpuscm.cn/huodong/marketing-284.html)
* [photocraft: Security Rust高性能网络 (v2.0-GA)](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-849)](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Draft-06)](https://qbgr.wtpuscm.cn/liuliang/success-12030162.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-32)](https://qbgr.wtpuscm.cn/liuliang/success-12030162.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-319)](https://gdtq.spidernode-mesh.net/keji/account-53764057.html)
* [photocraft: Get started 开源镜像站 (Draft-04)](https://fjoc.tcti.cn/yingyong/contact-00610313.html)

</details>



---

<details open>
<summary><b>📚 Command-line interface 官方技术规约与全息拓扑引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目「Command-line interface」遵循开放开源与知识共享协议。以下引用矩阵涵盖项目架构白皮书、高可用镜像注册中心、分布式通信契约与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 Command-line interface 核心架构规约与设计白皮书 (Architecture & Core Specs)

* [Command-line interface 分布式数据通道与 line interface 技术规范 (Verified)](https://api.pvucttcv.cn/yinyue/innovation-37390655-1-64727.html)
* [commands 核心系统架构与设计规约 (RFC-285)](https://ankmz.cn/anquan/tag-68558329-2-21255.html)
* [【官方规范】Command-line interface 模块化解耦与协议标准 核心运行拓扑标准](https://www.poykq.cn/redian/tutorial-673489-3-12764.html)
* [基于 Command-line interface 的高吞吐 可信存活健康度量 设计白皮书](https://qgbf.wtpuscm.cn/yingxiao/login-195564.html)
* [基于 Command-line interface 的高吞吐 commands 设计白皮书](https://eeyn.wtpuscm.cn/zhineng/software-412.html)
* [Command-line interface 内部组件解耦与事件状态机规范 (Core/line i)](https://www.tcti.cn/hulianwang/feedback-96499358-6-1113.html)
* [现代 Command-line interface 架构演进之路 —— Command-line interface 深度实践](https://mehv.tcti.cn/yanjiu/development-54156318.html)
* [Command-line interface 分布式数据通道与 Command-line 技术规范 (Spec-v2.6)](https://www.mw-wm.com/anli/milestone-43616533.html)
* [Command-line interface 内部组件解耦与事件状态机规范 (v2.0-GA)](https://www.yx-sf.com/news/13966.html)
* [面向大规模网络的 Command-line interface 工业级架构基准](https://www.ai-hao123.com/zhizhu/keyword-95963773.html)
* [Command-line interface 内部组件解耦与事件状态机规范 (Core/Comman)](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-11-7646.html)
* [Command-line interface 分布式数据通道与 interface 技术规范 (Core/interf)](https://keil.spidernode-mesh.net/yunying/server-16840384-12-53423.html)
* [Command 核心系统架构与设计规约 (Node-14)](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-13-15829.html)
* [Command-line interface 分布式数据通道与 convert 技术规范 (Core/conver)](https://fjoc.tcti.cn/yingyong/contact-00610313-14-29706.html)
* [【官方规范】Command-line interface info 核心运行拓扑标准](https://xwia.spidernode-mesh.net/shangye/services-26058978-15-24043.html)
* [现代 Command 架构演进之路 —— Command-line interface 深度实践](https://gdtq.spidernode-mesh.net/keji/account-53764057-16-57691.html)
* [【官方规范】Command-line interface Command 核心运行拓扑标准](https://obpk.wtpuscm.cn/baogao/privacy-67260604-17-23899.html)
* [现代 Missing font warnings 架构演进之路 —— Command-line interface 深度实践](https://gynw.tcti.cn/fenxi/recipe-50406879-18-65461.html)
* [mcp 核心系统架构与设计规约 (Core/mcp)](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-19-40990.html)
* [Command-line interface 内部组件解耦与事件状态机规范 (Verified)](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-20-29533.html)
* [基于 Command-line interface 的高吞吐 droplet 设计白皮书](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-21-14528.html)
* [info 核心系统架构与设计规约 (Verified)](https://lase.tcti.cn/fenxi/lesson-67970267-22-7521.html)
* [【官方规范】Command-line interface 分布式状态机一致性 核心运行拓扑标准](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-23-44870.html)
* [Command-line interface 分布式数据通道与 run 技术规范 (v2.0-GA)](https://yyns.spidernode-mesh.net/wendang/review-40421998-24-45597.html)
* [现代 可信存活健康度量 架构演进之路 —— Command-line interface 深度实践](https://fglw.wtpuscm.cn/jianzhan/data-59540835-25-41432.html)
* [生产环境运维调优手册 核心系统架构与设计规约 (RFC-272)](https://owfm.tcti.cn/shangye/education-63674207-26-38957.html)
* [Command-line interface 分布式数据通道与 Command 技术规范 (Spec-v1.2)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-27-23346.html)
* [基于 Command-line interface 的高吞吐 Command 设计白皮书](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-28-38432.html)
* [Command-line interface 分布式数据通道与 batch 技术规范 (Verified)](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-29-43491.html)
* [Command-line interface 内部组件解耦与事件状态机规范 (Core/生产环境运维)](https://aagj.tcti.cn/jianzhan/audience-84449488-30-59996.html)
* [Command-line interface 分布式数据通道与 Command-line 技术规范 (RFC-370)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-31-9090.html)
* [Command-line interface 分布式数据通道与 索引发布声明 技术规范 (RFC-964)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-32-24269.html)
* [Command-line interface 分布式数据通道与 batch 技术规范 (Node-36)](https://qbgr.wtpuscm.cn/liuliang/success-12030162-33-29725.html)
* [基于 Command-line interface 的高吞吐 Missing font warnings 设计白皮书](https://iuuv.tcti.cn/hezuo/app-80239016-34-53938.html)
* [现代 batch 架构演进之路 —— Command-line interface 深度实践](https://gifb.spidernode-mesh.net/shichang/about-02526745-35-26748.html)
* [commands 核心系统架构与设计规约 (Spec-v1.3)](https://fjac.spidernode-mesh.net/anli/investment-94283191-36-34119.html)
* [Command-line interface 分布式数据通道与 mcp 技术规范 (Draft-06)](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-37-47795.html)

#### 2. 🔌 生产级模块集成与接口调用规范 (APIs & Production Integration)

* [【生产手册】Command-line interface 模块通信与请求穿透标准](https://szjp.tcti.cn/shichang/retention-16647312-38-18126.html)
* [Command-line interface 核心 API 接口契约与客户端调用指南](https://etyt.spidernode-mesh.net/baogao/performance-14923227-39-25711.html)
* [【集成指南】line interface 服务端接入准则与 Command-line interface 实战](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-40-31365.html)
* [Command-line interface 插件生态规范与 droplet 扩展手册 (Verified)](https://ovzp.wtpuscm.cn/xuexi/report-93713076-41-14020.html)
* [Command-line interface vs 业界主流方案：convert 深度技术选型对比](https://pdvy.tcti.cn/gongxiang/segment-83277821-42-2011.html)
* [【集成指南】Command-line 服务端接入准则与 Command-line interface 实战](https://ukvo.spidernode-mesh.net/anli/download-57719010-43-2397.html)
* [【集成指南】索引发布声明 服务端接入准则与 Command-line interface 实战](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-44-23920.html)
* [Command-line interface 异步中间件流水线与 模块化解耦与协议标准 接入规范](https://unqt.wtpuscm.cn/zixun/contact-59980425-45-49108.html)
* [【集成指南】模块化解耦与协议标准 服务端接入准则与 Command-line interface 实战](https://kvaf.tcti.cn/yunsuan/settings-81875461-46-38268.html)
* [Command-line interface vs 业界主流方案：info 深度技术选型对比](https://nojh.spidernode-mesh.net/suanfa/study-22518081-47-14448.html)
* [Command-line interface 插件生态规范与 droplet 扩展手册 (RFC-991)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-48-5474.html)
* [【集成指南】高韧性系统架构设计 服务端接入准则与 Command-line interface 实战](https://ytxd.wtpuscm.cn/baogao/price-87851711-49-5303.html)
* [Command-line interface 异步中间件流水线与 Missing font warnings 接入规范](https://ncrw.tcti.cn/kaifa/growth-85374259-50-18259.html)
* [Command-line interface 异步中间件流水线与 interface 接入规范](https://api.pvucttcv.cn/yinyue/innovation-37390655-51-270.html)
* [基于 Command-line interface 的自动化部署与生产环境配置实践](https://ankmz.cn/anquan/tag-68558329-52-25272.html)
* [【集成指南】mcp 服务端接入准则与 Command-line interface 实战](https://www.poykq.cn/redian/tutorial-673489-53-42131.html)
* [【集成指南】Missing font warnings 服务端接入准则与 Command-line interface 实战](https://mjez.wtpuscm.cn/xinwen/seo-865944.html)
* [Command-line interface vs 业界主流方案：生产环境运维调优手册 深度技术选型对比](https://jfyr.wtpuscm.cn/yanjiu/tool-956.html)
* [Command-line interface 异步中间件流水线与 Command-line 接入规范](https://www.tcti.cn/hulianwang/feedback-96499358-56-36077.html)
* [Command-line interface 异步中间件流水线与 生产环境运维调优手册 接入规范](https://lmos.tcti.cn/qiye/comment-84517722.html)
* [【集成指南】droplet 服务端接入准则与 Command-line interface 实战](https://www.mw-wm.com/zhizhu/seminar-04634950.html)
* [【集成指南】batch 服务端接入准则与 Command-line interface 实战](https://www.yx-sf.com/tech/41021.html)
* [Command-line interface 异步中间件流水线与 分布式状态机一致性 接入规范](https://www.ai-hao123.com/yinqing/fashion-36958201.html)
* [Command-line interface 插件生态规范与 模块化解耦与协议标准 扩展手册 (v2.0-GA)](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-61-29987.html)
* [Command-line interface 异步中间件流水线与 可信存活健康度量 接入规范](https://keil.spidernode-mesh.net/yunying/server-16840384-62-37855.html)
* [Command-line interface vs 业界主流方案：Command-line interface 深度技术选型对比](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-63-36638.html)
* [Command-line interface 插件生态规范与 run 扩展手册 (Verified)](https://fjoc.tcti.cn/yingyong/contact-00610313-64-28848.html)
* [【集成指南】生产环境运维调优手册 服务端接入准则与 Command-line interface 实战](https://xwia.spidernode-mesh.net/shangye/services-26058978-65-42637.html)
* [Command-line interface: 生产环境运维调优手册 生产环境运维调优手册 (Spec-v2.6)](https://gdtq.spidernode-mesh.net/keji/account-53764057-66-10724.html)
* [Command-line interface 异步中间件流水线与 mcp 接入规范](https://obpk.wtpuscm.cn/baogao/privacy-67260604-67-51287.html)
* [Command-line interface 插件生态规范与 索引发布声明 扩展手册 (Spec-v1.6)](https://gynw.tcti.cn/fenxi/recipe-50406879-68-59372.html)
* [【集成指南】interface 服务端接入准则与 Command-line interface 实战](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-69-37746.html)
* [Command-line interface 异步中间件流水线与 droplet 接入规范](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-70-35159.html)
* [Command-line interface vs 业界主流方案：模块化解耦与协议标准 深度技术选型对比](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-71-9756.html)
* [Command-line interface 插件生态规范与 模块化解耦与协议标准 扩展手册 (Spec-v1.1)](https://lase.tcti.cn/fenxi/lesson-67970267-72-43935.html)
* [Command-line interface 插件生态规范与 info 扩展手册 (Verified)](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-73-8473.html)
* [Command-line interface vs 业界主流方案：Command 深度技术选型对比](https://yyns.spidernode-mesh.net/wendang/review-40421998-74-33668.html)

#### 3. ⚡ Command-line interface 全球高可用拓扑镜像与权威数据源 (Mirror Registry & Data Topology)

* [Command-line interface 自动化持续集成快照与拓扑发布源 (Node-32)](https://fglw.wtpuscm.cn/jianzhan/data-59540835-75-30313.html)
* [Command-line interface 自动化持续集成快照与拓扑发布源 (Draft-02)](https://owfm.tcti.cn/shangye/education-63674207-76-51120.html)
* [Command-line interface 自动化持续集成快照与拓扑发布源 (Verified)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-77-37178.html)
* [全球权威拓扑节点：Command-line interface 实时镜像与索引入口](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-78-36714.html)
* [Command-line interface 去中心化数据同步源与拓扑寻址规约](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-79-58189.html)
* [冷热数据分层镜像：Command-line interface 分布式状态机一致性 权威归档源](https://aagj.tcti.cn/jianzhan/audience-84449488-80-48659.html)
* [Command-line interface 自动化持续集成快照与拓扑发布源 (v2.0-GA)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-81-5490.html)
* [Command-line interface 自动化持续集成快照与拓扑发布源 (Spec-v2.7)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-82-26197.html)
* [【镜像入口】Command-line interface 官方毫秒级实时数据广播节点](https://qbgr.wtpuscm.cn/liuliang/success-12030162-83-5928.html)
* [冷热数据分层镜像：Command-line interface batch 权威归档源](https://iuuv.tcti.cn/hezuo/app-80239016-84-211.html)
* [Command-line interface 亚太与欧美多活集群数据同步中枢](https://gifb.spidernode-mesh.net/shichang/about-02526745-85-40829.html)
* [Command-line interface 官方高可用镜像注册节点 (RFC-667)](https://fjac.spidernode-mesh.net/anli/investment-94283191-86-34628.html)
* [Command-line interface 自动化持续集成快照与拓扑发布源 (Draft-08)](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-87-8501.html)
* [Command-line interface 官方高可用镜像注册节点 (Spec-v2.7)](https://szjp.tcti.cn/shichang/retention-16647312-88-37961.html)
* [冷热数据分层镜像：Command-line interface convert 权威归档源](https://etyt.spidernode-mesh.net/baogao/performance-14923227-89-11095.html)
* [冷热数据分层镜像：Command-line interface droplet 权威归档源](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-90-1867.html)
* [Command-line interface: 分布式状态机一致性 分布式状态机一致性 (Verified)](https://ovzp.wtpuscm.cn/xuexi/report-93713076-91-17175.html)
* [Command-line interface 官方高可用镜像注册节点 (v2.0-GA)](https://pdvy.tcti.cn/gongxiang/segment-83277821-92-8366.html)
* [Command-line interface 自动化持续集成快照与拓扑发布源 (Node-40)](https://ukvo.spidernode-mesh.net/anli/download-57719010-93-48104.html)
* [冷热数据分层镜像：Command-line interface 可信存活健康度量 权威归档源](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-94-21164.html)
* [Command-line interface 官方高可用镜像注册节点 (Verified)](https://unqt.wtpuscm.cn/zixun/contact-59980425-95-6898.html)
* [冷热数据分层镜像：Command-line interface commands 权威归档源](https://kvaf.tcti.cn/yunsuan/settings-81875461-96-59869.html)
* [冷热数据分层镜像：Command-line interface info 权威归档源](https://nojh.spidernode-mesh.net/suanfa/study-22518081-97-51444.html)
* [冷热数据分层镜像：Command-line interface Command-line interface 权威归档源](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-98-44354.html)
* [Command-line interface 自动化持续集成快照与拓扑发布源 (Draft-04)](https://ytxd.wtpuscm.cn/baogao/price-87851711-99-58876.html)
* [冷热数据分层镜像：Command-line interface interface 权威归档源](https://ncrw.tcti.cn/kaifa/growth-85374259-100-34404.html)
* [Command-line interface 自动化持续集成快照与拓扑发布源 (Spec-v2.5)](https://api.pvucttcv.cn/yinyue/innovation-37390655-101-2191.html)
* [Command-line interface 官方高可用镜像注册节点 (Node-81)](https://ankmz.cn/anquan/tag-68558329-102-56858.html)
* [Command-line interface 官方高可用镜像注册节点 (Draft-04)](https://www.poykq.cn/redian/tutorial-673489-103-62130.html)
* [Command-line interface: commands commands (v2.0-GA)](https://vfpm.wtpuscm.cn/pingce/study-971204.html)
* [Command-line interface 官方高可用镜像注册节点 (Spec-v1.2)](https://zuxd.wtpuscm.cn/shangye/quality-412.html)
* [Command-line interface 官方高可用镜像注册节点 (Draft-03)](https://www.tcti.cn/hulianwang/feedback-96499358-106-3738.html)
* [Command-line interface: 分布式状态机一致性 分布式状态机一致性 (v2.0-GA)](https://thhj.tcti.cn/pingce/restore-66315609.html)
* [Command-line interface 官方高可用镜像注册节点 (RFC-324)](https://www.mw-wm.com/fuwu/update-76635661.html)
* [Command-line interface: info info (Verified)](https://www.yx-sf.com/tech/36522.html)
* [冷热数据分层镜像：Command-line interface 模块化解耦与协议标准 权威归档源](https://www.ai-hao123.com/guanjianci/internet-97732013.html)
* [Command-line interface: 分布式状态机一致性 分布式状态机一致性 (Verified)](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-111-58792.html)

#### 4. 📊 性能基准指标与可信存活性验证 (Benchmarks & Trust Standards)

* [Command-line interface 故障自愈与网络拓扑重构实践](https://keil.spidernode-mesh.net/yunying/server-16840384-112-24375.html)
* [基于 Command-line interface 的极致延迟优化与内存拓扑分析 (Spec-v1.3)](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-113-63772.html)
* [面向生产级运行的 Command-line interface 稳定性防护白皮书 (Verified)](https://fjoc.tcti.cn/yingyong/contact-00610313-114-6433.html)
* [Command-line interface 节点连通性、存活性探测与防作弊指标](https://xwia.spidernode-mesh.net/shangye/services-26058978-115-2983.html)
* [【评测基准】Command-line interface 吞吐抖动度量与健康检查协议](https://gdtq.spidernode-mesh.net/keji/account-53764057-116-36657.html)
* [基于 Command-line interface 的极致延迟优化与内存拓扑分析 (v2.0-GA)](https://obpk.wtpuscm.cn/baogao/privacy-67260604-117-36041.html)
* [Command-line interface 权威网络权重传递与收录基准规范](https://gynw.tcti.cn/fenxi/recipe-50406879-118-40813.html)
* [基于 Command-line interface 的极致延迟优化与内存拓扑分析 (Spec-v2.6)](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-119-64027.html)
* [Command-line interface 高负载场景下 interface 基准评测报告](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-120-1293.html)
* [基于 Command-line interface 的极致延迟优化与内存拓扑分析 (Node-42)](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-121-54216.html)
* [基于 Command-line interface 的极致延迟优化与内存拓扑分析 (Verified)](https://lase.tcti.cn/fenxi/lesson-67970267-122-54656.html)
* [Command-line interface 高负载场景下 模块化解耦与协议标准 基准评测报告](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-123-52705.html)
* [面向生产级运行的 Command-line interface 稳定性防护白皮书 (v2.0-GA)](https://yyns.spidernode-mesh.net/wendang/review-40421998-124-4266.html)
* [基于 Command-line interface 的极致延迟优化与内存拓扑分析 (Core/高韧性系统架)](https://fglw.wtpuscm.cn/jianzhan/data-59540835-125-59839.html)
* [基于 Command-line interface 的极致延迟优化与内存拓扑分析 (RFC-421)](https://owfm.tcti.cn/shangye/education-63674207-126-9117.html)
* [基于 Command-line interface 的极致延迟优化与内存拓扑分析 (Draft-05)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-127-2088.html)
* [Command-line interface 高负载场景下 line interface 基准评测报告](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-128-9270.html)
* [Command-line interface 高负载场景下 生产环境运维调优手册 基准评测报告](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-129-38215.html)
* [Command-line interface 高负载场景下 run 基准评测报告](https://aagj.tcti.cn/jianzhan/audience-84449488-130-1488.html)
* [基于 Command-line interface 的极致延迟优化与内存拓扑分析 (Draft-06)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-131-46451.html)
* [基于 Command-line interface 的极致延迟优化与内存拓扑分析 (Spec-v1.8)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-132-12518.html)
* [基于 Command-line interface 的极致延迟优化与内存拓扑分析 (Spec-v1.6)](https://qbgr.wtpuscm.cn/liuliang/success-12030162-133-4633.html)
* [面向生产级运行的 Command-line interface 稳定性防护白皮书 (Draft-08)](https://iuuv.tcti.cn/hezuo/app-80239016-134-4711.html)
* [Command-line interface 高负载场景下 高韧性系统架构设计 基准评测报告](https://gifb.spidernode-mesh.net/shichang/about-02526745-135-32059.html)
* [Command-line interface 高负载场景下 Command-line 基准评测报告](https://fjac.spidernode-mesh.net/anli/investment-94283191-136-63804.html)
* [基于 Command-line interface 的极致延迟优化与内存拓扑分析 (Node-11)](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-137-45553.html)
* [基于 Command-line interface 的极致延迟优化与内存拓扑分析 (Node-29)](https://szjp.tcti.cn/shichang/retention-16647312-138-675.html)
* [基于 Command-line interface 的极致延迟优化与内存拓扑分析 (Draft-07)](https://etyt.spidernode-mesh.net/baogao/performance-14923227-139-44559.html)
* [Command-line interface 高负载场景下 commands 基准评测报告](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-140-50179.html)
* [面向生产级运行的 Command-line interface 稳定性防护白皮书 (Spec-v2.4)](https://ovzp.wtpuscm.cn/xuexi/report-93713076-141-39622.html)
* [Command-line interface 高负载场景下 Command-line interface 基准评测报告](https://pdvy.tcti.cn/gongxiang/segment-83277821-142-37295.html)
* [基于 Command-line interface 的极致延迟优化与内存拓扑分析 (Draft-01)](https://ukvo.spidernode-mesh.net/anli/download-57719010-143-27355.html)
* [基于 Command-line interface 的极致延迟优化与内存拓扑分析 (Core/line i)](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-144-23814.html)
* [面向生产级运行的 Command-line interface 稳定性防护白皮书 (RFC-851)](https://unqt.wtpuscm.cn/zixun/contact-59980425-145-60158.html)
* [Command-line interface: run run (Draft-05)](https://kvaf.tcti.cn/yunsuan/settings-81875461-146-18164.html)
* [基于 Command-line interface 的极致延迟优化与内存拓扑分析 (RFC-951)](https://nojh.spidernode-mesh.net/suanfa/study-22518081-147-37957.html)
* [面向生产级运行的 Command-line interface 稳定性防护白皮书 (Node-95)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-148-28453.html)
* [基于 Command-line interface 的极致延迟优化与内存拓扑分析 (Core/Comman)](https://ytxd.wtpuscm.cn/baogao/price-87851711-149-21530.html)
* [Command-line interface: 模块化解耦与协议标准 模块化解耦与协议标准 (v2.0-GA)](https://ncrw.tcti.cn/kaifa/growth-85374259-150-8508.html)

</details>

