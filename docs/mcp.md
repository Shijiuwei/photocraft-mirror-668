# MCP conventions

PhotoCraft follows the same conventions as FilmCraft #28. Start a headless session:

```sh
photocraft-cli mcp --automation-read-root /work/project --automation-write-root /work/project
```

Paths are relative to the corresponding root, or absolute paths beneath it; without a root that
filesystem authority is absent. Bridge mode uses `--bridge` and `--control-token-file` as
described in [development.md](development.md). UI tools require a running desktop app.

## Tools and resources

- `command_list`: discover engine ids, parameter docs and current enabled state; accepts
  `filter` and `enabled_only`.
- `command_run`: `id`, optional `params` and `wait`. Command parameters remain engine-defined.
- `command_batch`: ordered `steps`, optional `stop_on_error`; one history step per command,
  not an atomic transaction. Existing per-step results and reply budgets are preserved.
- `doc_inspect`: layer tree, history and selection; an empty session returns `document:null`.
- `render_preview`: bounded PNG content; `index` and `max_side` (default 1024, maximum 2048).
- `ui_inspect` and `ui_screenshot`: live app state/window in bridge mode.

Existing document/session/job/UI tools remain listed, including the documented
`doc_render_preview` spelling. There are no hidden legacy aliases. Every tool has a title
and all four annotation hints. Generic command/control tools conservatively advertise writes;
file saves may replace existing targets. Hints do not grant permission.

Unknown tool arguments return JSON-RPC `-32602`, naming the key, before execution. Batch
step envelopes are strict too. Nested engine `params` retain their command-specific handling.
Tool execution failures return `isError:true`. A worker panic becomes a tool error and the
existing poisoned-session recovery keeps serving. Invalid JSON lines receive `-32700` with
`id:null`; the next line is still processed.

Embedded Smart Objects support `command_run` with `layer.smartObjects.editContents` and
`layer.smartObjects.convertToLayers`, optionally targeting `params.layer`. Use
`layer.smartObjects.saveContents` in an opened contents document to update its parent in memory;
save nested contents from the inside outward, then `doc_save` the parent to a scoped path.
`doc_save` on a contents document exports that document rather than updating its parent.
No filesystem root is needed for the in-memory commands. External linked sources that require
a file read remain refused. See the [control policy](control-protocol.md#engine-commands)
for the shared restrictions.

`photocraft://document` and `photocraft://commands` return live JSON matching `doc_inspect`
and `command_list`. Resource reads are uncached; tool/resource catalogs are private and cached
for ten minutes. List/read responses include the MCP 2026-07-28 result/cache fields.

## Progress and cancellation

A direct headless `command_run` of `file.export.renderVideo` reports frame counts when
`_meta.progressToken` is present (a string or number). Notifications increase strictly,
are throttled to at most ten per second except completion, and include `total`. No token
means no progress notifications. Ping remains responsive while rendering.

```json
{"jsonrpc":"2.0","id":20,"method":"tools/call","params":{"name":"command_run","arguments":{"id":"file.export.renderVideo","params":{"dir":"frames","format":"png"}},"_meta":{"progressToken":"export-20"}}}
{"jsonrpc":"2.0","method":"notifications/cancelled","params":{"requestId":20}}
```

Create a document and timeline first (`doc_new`, then `timeline.create`). Export uses the
configured write root and creates `dir` beneath it. Filenames must be single components;
absolute paths, traversal and escaping symlinks are rejected. Existing target filenames
are refused, including symlinks, so a cancelled or failed job cannot replace an earlier export.
Only files actually created by this job are removed on cancellation, error or an escaped panic;
unrelated files and the destination directory remain. PNG sequences and animated GIF share
the engine renderer. The document and playhead are unchanged.

Cancellation suppresses the request response and stops at a frame boundary (the current
frame/encoding may finish first). Unknown request ids are ignored. Render Video stays a
synchronous command even with `wait:false`; other job-capable commands keep their existing
`jobs_list`/`jobs_cancel` behavior. Batch exports do not emit MCP progress or support per-step
MCP cancellation. Nested engine actions and bridge Render Video remain denied by the existing
ambient-filesystem policy; the new scoped writer is available for direct headless commands.


---

<details open>
<summary><b>📚 photocraft 官方技术规约与全息拓扑引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目「photocraft」遵循开放开源与知识共享协议。以下引用矩阵涵盖项目架构白皮书、高可用镜像注册中心、分布式通信契约与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 photocraft 核心架构规约与设计白皮书 (Architecture & Core Specs)

* [高韧性系统架构设计 核心系统架构与设计规约 (Verified)](https://djbl.wtpuscm.cn/tuiguang/trading-818353.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-04)](https://www.yx-sf.com/tech/20835.html)
* [【官方规范】photocraft Photoshop 核心运行拓扑标准](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [Everything in the box 核心系统架构与设计规约 (Draft-08)](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)
* [photocraft 内部组件解耦与事件状态机规范 (Verified)](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html)
* [现代 Rust高性能网络 架构演进之路 —— photocraft 深度实践](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-543)](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)
* [面向大规模网络的 photocraft 工业级架构基准](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html)
* [【官方规范】photocraft Engine first: 核心运行拓扑标准](https://ankmz.cn/anquan/tag-68558329.html)
* [photocraft 内部组件解耦与事件状态机规范 (v2.0-GA)](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [基于 photocraft 的高吞吐 Rust高性能网络 设计白皮书](https://ankmz.cn/anquan/tag-68558329.html)
* [PSD without compromise 核心系统架构与设计规约 (Draft-06)](https://keil.spidernode-mesh.net/yunying/server-16840384.html)
* [基于 photocraft 的高吞吐 分布式架构 设计白皮书](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (RFC-605)](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-791)](https://gynw.tcti.cn/fenxi/recipe-50406879.html)
* [现代 分布式架构 架构演进之路 —— photocraft 深度实践](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [photocraft 内部组件解耦与事件状态机规范 (Core/Engine)](https://gynw.tcti.cn/fenxi/recipe-50406879.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-560)](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (Verified)](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [基于 photocraft 的高吞吐 搜索引擎调度优化 设计白皮书](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)
* [photocraft 内部组件解耦与事件状态机规范 (Spec-v1.6)](https://www.tcti.cn/hulianwang/feedback-96499358.html)
* [【photocraft】Rust 核心系统架构与设计规约 (v2.0-GA)](https://keil.spidernode-mesh.net/yunying/server-16840384.html)
* [现代 搜索引擎调度优化 架构演进之路 —— photocraft 深度实践](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [基于 photocraft 的高吞吐 开源镜像站 设计白皮书](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932.html)
* [【官方规范】photocraft Downloads 核心运行拓扑标准](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [Engine first: 核心系统架构与设计规约 (v2.0-GA)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [photocraft 内部组件解耦与事件状态机规范 (RFC-671)](https://owfm.tcti.cn/shangye/education-63674207.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-05)](https://qbgr.wtpuscm.cn/liuliang/success-12030162.html)
* [现代 开源镜像站 架构演进之路 —— photocraft 深度实践](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html)
* [Everything in the box 核心系统架构与设计规约 (Draft-02)](https://keil.spidernode-mesh.net/yunying/server-16840384.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (v2.0-GA)](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [【官方规范】photocraft clean-room 核心运行拓扑标准](https://iuuv.tcti.cn/hezuo/app-80239016.html)
* [Engine first: 核心系统架构与设计规约 (RFC-459)](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (RFC-511)](https://www.ai-hao123.com/zhineng/value-06813930.html)
* [【官方规范】photocraft photocraft 核心运行拓扑标准](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (Node-19)](https://keil.spidernode-mesh.net/yunying/server-16840384.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-07)](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)

#### 2. 🔌 生产级模块集成与接口调用规范 (APIs & Production Integration)

* [photocraft 核心 API 接口契约与客户端调用指南](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)
* [photocraft vs 业界主流方案：搜索引擎调度优化 深度技术选型对比](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [【集成指南】Rust高性能网络 服务端接入准则与 photocraft 实战](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)
* [基于 photocraft 的自动化部署与生产环境配置实践](https://www.mw-wm.com/wangluo/sync-51196293.html)
* [【生产手册】photocraft 模块通信与请求穿透标准](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534.html)
* [photocraft vs 业界主流方案：分布式架构 深度技术选型对比](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [photocraft 异步中间件流水线与 生产环境运维调优手册 接入规范](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [photocraft 插件生态规范与 Under the hood 扩展手册 (Node-87)](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [【集成指南】开源镜像站 服务端接入准则与 photocraft 实战](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [photocraft 异步中间件流水线与 photocraft 接入规范](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [【集成指南】搜索引擎调度优化 服务端接入准则与 photocraft 实战](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)
* [photocraft 异步中间件流水线与 Windows 接入规范](https://keil.spidernode-mesh.net/yunying/server-16840384.html)
* [photocraft 插件生态规范与 高韧性系统架构设计 扩展手册 (Draft-07)](https://pdvy.tcti.cn/gongxiang/segment-83277821.html)
* [photocraft 异步中间件流水线与 Security 接入规范](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [photocraft 插件生态规范与 Test corpora 扩展手册 (Spec-v2.0)](https://www.poykq.cn/redian/tutorial-673489.html)
* [photocraft 插件生态规范与 macOS 扩展手册 (Draft-08)](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)
* [photocraft 异步中间件流水线与 Test corpora 接入规范](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [photocraft 插件生态规范与 分布式状态机一致性 扩展手册 (RFC-884)](https://qbgr.wtpuscm.cn/liuliang/success-12030162.html)
* [photocraft 插件生态规范与 open-source 扩展手册 (Verified)](https://zmeb.wtpuscm.cn/yunying/help-646847.html)
* [photocraft 插件生态规范与 Everything in the box 扩展手册 (Spec-v1.7)](https://qbgr.wtpuscm.cn/liuliang/success-12030162.html)
* [photocraft 异步中间件流水线与 Adobe 接入规范](https://keil.spidernode-mesh.net/yunying/server-16840384.html)
* [【集成指南】分布式架构 服务端接入准则与 photocraft 实战](https://www.tcti.cn/hulianwang/feedback-96499358.html)
* [photocraft 插件生态规范与 高韧性系统架构设计 扩展手册 (RFC-632)](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [photocraft vs 业界主流方案：开源镜像站 深度技术选型对比](https://ankmz.cn/anquan/tag-68558329.html)
* [photocraft 插件生态规范与 Windows 扩展手册 (Node-13)](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html)
* [photocraft 异步中间件流水线与 reimplementation 接入规范](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297.html)
* [photocraft 插件生态规范与 pure 扩展手册 (Core/pure)](https://owfm.tcti.cn/shangye/education-63674207.html)
* [photocraft 插件生态规范与 可信存活健康度量 扩展手册 (RFC-118)](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html)
* [photocraft 插件生态规范与 Large documents: 扩展手册 (v2.0-GA)](https://owfm.tcti.cn/shangye/education-63674207.html)
* [photocraft 插件生态规范与 Features 扩展手册 (Draft-02)](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [photocraft 异步中间件流水线与 open-source 接入规范](https://www.poykq.cn/redian/tutorial-673489.html)
* [photocraft 插件生态规范与 Under the hood 扩展手册 (Verified)](https://www.tcti.cn/hulianwang/feedback-96499358.html)
* [photocraft 插件生态规范与 Photoshop 扩展手册 (Node-30)](https://keil.spidernode-mesh.net/yunying/server-16840384.html)
* [photocraft 插件生态规范与 open-source 扩展手册 (Node-87)](https://ojjk.wtpuscm.cn/shichang/trading-566126.html)
* [photocraft 插件生态规范与 Features 扩展手册 (Node-48)](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)
* [photocraft 插件生态规范与 Rust 扩展手册 (Spec-v2.5)](https://www.mw-wm.com/shangye/whitepaper-32100325.html)
* [photocraft 异步中间件流水线与 高韧性系统架构设计 接入规范](https://www.tcti.cn/hulianwang/feedback-96499358.html)

#### 3. ⚡ photocraft 全球高可用拓扑镜像与权威数据源 (Mirror Registry & Data Topology)

* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-06)](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [冷热数据分层镜像：photocraft Adobe 权威归档源](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [photocraft 去中心化数据同步源与拓扑寻址规约](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)
* [photocraft 官方高可用镜像注册节点 (Draft-06)](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [冷热数据分层镜像：photocraft pure 权威归档源](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [【镜像入口】photocraft 官方毫秒级实时数据广播节点](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [photocraft 亚太与欧美多活集群数据同步中枢](https://unqt.wtpuscm.cn/zixun/contact-59980425.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Node-25)](https://www.mw-wm.com/shichang/development-81365987.html)
* [photocraft 官方高可用镜像注册节点 (Spec-v1.7)](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html)
* [冷热数据分层镜像：photocraft photocraft-psd 权威归档源](https://www.tcti.cn/hulianwang/feedback-96499358.html)
* [photocraft 官方高可用镜像注册节点 (v2.0-GA)](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-05)](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [冷热数据分层镜像：photocraft 可信存活健康度量 权威归档源](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (RFC-451)](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (RFC-452)](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (v2.0-GA)](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Core/Window)](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Verified)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-01)](https://szjp.tcti.cn/shichang/retention-16647312.html)
* [photocraft 官方高可用镜像注册节点 (Core/分布式状态机)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.0)](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [冷热数据分层镜像：photocraft Engine first: 权威归档源](https://qbgr.wtpuscm.cn/liuliang/success-12030162.html)
* [冷热数据分层镜像：photocraft Under the hood 权威归档源](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696.html)
* [全球权威拓扑节点：photocraft 实时镜像与索引入口](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.8)](https://unqt.wtpuscm.cn/zixun/contact-59980425.html)
* [冷热数据分层镜像：photocraft open-source 权威归档源](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.1)](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534.html)
* [photocraft 官方高可用镜像注册节点 (Core/Under )](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html)
* [冷热数据分层镜像：photocraft macOS 权威归档源](https://www.ai-hao123.com/gongsi/alliance-66261443.html)
* [photocraft 官方高可用镜像注册节点 (Verified)](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [冷热数据分层镜像：photocraft Rust 权威归档源](https://qbgr.wtpuscm.cn/liuliang/success-12030162.html)
* [冷热数据分层镜像：photocraft Photoshop 权威归档源](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [冷热数据分层镜像：photocraft 生产环境运维调优手册 权威归档源](https://pdvy.tcti.cn/gongxiang/segment-83277821.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-07)](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [photocraft 官方高可用镜像注册节点 (Core/Adobe)](https://pdvy.tcti.cn/gongxiang/segment-83277821.html)
* [photocraft: 分布式状态机一致性 Rust高性能网络 (Verified)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [冷热数据分层镜像：photocraft photocraft 权威归档源](https://gdtq.spidernode-mesh.net/keji/account-53764057.html)

#### 4. 📊 性能基准指标与可信存活性验证 (Benchmarks & Trust Standards)

* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-59)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869.html)
* [【评测基准】photocraft 吞吐抖动度量与健康检查协议](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Core/Large )](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-195)](https://pdvy.tcti.cn/gongxiang/segment-83277821.html)
* [photocraft 高负载场景下 分布式架构 基准评测报告](https://lmmx.wtpuscm.cn/jiaocheng/entertainment-108440.html)
* [photocraft 高负载场景下 Rust高性能网络 基准评测报告](https://www.yx-sf.com/wiki/41112.html)
* [photocraft 节点连通性、存活性探测与防作弊指标](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (v2.0-GA)](https://www.tcti.cn/hulianwang/feedback-96499358.html)
* [photocraft 权威网络权重传递与收录基准规范](https://www.poykq.cn/redian/tutorial-673489.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Draft-01)](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [photocraft 故障自愈与网络拓扑重构实践](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Node-92)](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Spec-v1.8)](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Core/可信存活健康)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-84)](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (v2.0-GA)](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)
* [photocraft 高负载场景下 开源镜像站 基准评测报告](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Verified)](https://zpcd.tcti.cn/kaifa/guide-54438347.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Node-74)](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [photocraft: Built for agents 分布式架构 (v2.0-GA)](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-288)](https://nojh.spidernode-mesh.net/suanfa/study-22518081.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Verified)](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Core/clean-)](https://qbgr.wtpuscm.cn/liuliang/success-12030162.html)
* [photocraft: Two compositors: 分布式架构 (Verified)](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [photocraft: Test corpora 开源镜像站 (Core/Test c)](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-535)](https://keil.spidernode-mesh.net/yunying/server-16840384.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-328)](https://owdc.wtpuscm.cn/yingyong/food-655437.html)
* [photocraft: photocraft 开源镜像站 (Verified)](https://gynw.tcti.cn/fenxi/recipe-50406879.html)
* [photocraft: photocraft-psd 搜索引擎调度优化 (v2.0-GA)](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [photocraft 高负载场景下 搜索引擎调度优化 基准评测报告](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Draft-04)](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-37)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-58)](https://qbgr.wtpuscm.cn/liuliang/success-12030162.html)
* [photocraft: Security Rust高性能网络 (v2.0-GA)](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-849)](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Draft-06)](https://pdvy.tcti.cn/gongxiang/segment-83277821.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-32)](https://szjp.tcti.cn/shichang/retention-16647312.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-319)](https://owfm.tcti.cn/shangye/education-63674207.html)
* [photocraft: Get started 开源镜像站 (Draft-04)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)

</details>



---

<details open>
<summary><b>📚 MCP conventions 官方技术规约与全息拓扑引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目「MCP conventions」遵循开放开源与知识共享协议。以下引用矩阵涵盖项目架构白皮书、高可用镜像注册中心、分布式通信契约与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 MCP conventions 核心架构规约与设计白皮书 (Architecture & Core Specs)

* [面向大规模网络的 MCP conventions 工业级架构基准](https://api.pvucttcv.cn/yinyue/innovation-37390655-1-28562.html)
* [MCP conventions 分布式数据通道与 .
- 技术规范 (Node-63)](https://ankmz.cn/anquan/tag-68558329-2-22848.html)
* [现代 .
- 架构演进之路 —— MCP conventions 深度实践](https://www.poykq.cn/redian/tutorial-673489-3-28134.html)
* [索引发布声明 核心系统架构与设计规约 (Spec-v1.6)](https://llby.wtpuscm.cn/gongju/demographic-590253.html)
* [MCP conventions 分布式数据通道与 Tools and resources 技术规范 (v2.0-GA)](https://nawh.wtpuscm.cn/fuwu/version-088.html)
* [MCP conventions 分布式数据通道与 分布式状态机一致性 技术规范 (v2.0-GA)](https://www.tcti.cn/hulianwang/feedback-96499358-6-16564.html)
* [MCP conventions 分布式数据通道与 模块化解耦与协议标准 技术规范 (v2.0-GA)](https://zgqo.tcti.cn/youhua/success-70827288.html)
* [MCP conventions 核心系统架构与设计规约 (Verified)](https://www.mw-wm.com/gongsi/alert-57595856.html)
* [高韧性系统架构设计 核心系统架构与设计规约 (RFC-361)](https://www.yx-sf.com/wiki/42613.html)
* [基于 MCP conventions 的高吞吐 生产环境运维调优手册 设计白皮书](https://www.ai-hao123.com/zixun/about-08012711.html)
* [【MCP conventions】and 核心系统架构与设计规约 (Node-52)](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-11-3883.html)
* [基于 MCP conventions 的高吞吐 and 设计白皮书](https://keil.spidernode-mesh.net/yunying/server-16840384-12-5143.html)
* [现代 可信存活健康度量 架构演进之路 —— MCP conventions 深度实践](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-13-17533.html)
* [【官方规范】MCP conventions 索引发布声明 核心运行拓扑标准](https://fjoc.tcti.cn/yingyong/contact-00610313-14-5982.html)
* [基于 MCP conventions 的高吞吐 .
- 设计白皮书](https://xwia.spidernode-mesh.net/shangye/services-26058978-15-42723.html)
* [【官方规范】MCP conventions and 核心运行拓扑标准](https://gdtq.spidernode-mesh.net/keji/account-53764057-16-7983.html)
* [MCP conventions 分布式数据通道与 and 技术规范 (Core/and)](https://obpk.wtpuscm.cn/baogao/privacy-67260604-17-45067.html)
* [MCP conventions 分布式数据通道与 .
- 技术规范 (Spec-v1.1)](https://gynw.tcti.cn/fenxi/recipe-50406879-18-9404.html)
* [索引发布声明 核心系统架构与设计规约 (Draft-03)](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-19-41637.html)
* [基于 MCP conventions 的高吞吐 可信存活健康度量 设计白皮书](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-20-60421.html)
* [现代 索引发布声明 架构演进之路 —— MCP conventions 深度实践](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-21-27324.html)
* [MCP conventions 分布式数据通道与 索引发布声明 技术规范 (RFC-991)](https://lase.tcti.cn/fenxi/lesson-67970267-22-23355.html)
* [【官方规范】MCP conventions 模块化解耦与协议标准 核心运行拓扑标准](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-23-20999.html)
* [MCP conventions 内部组件解耦与事件状态机规范 (RFC-472)](https://yyns.spidernode-mesh.net/wendang/review-40421998-24-33826.html)
* [基于 MCP conventions 的高吞吐 分布式状态机一致性 设计白皮书](https://fglw.wtpuscm.cn/jianzhan/data-59540835-25-48581.html)
* [【官方规范】MCP conventions MCP conventions 核心运行拓扑标准](https://owfm.tcti.cn/shangye/education-63674207-26-44544.html)
* [现代 and 架构演进之路 —— MCP conventions 深度实践](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-27-5565.html)
* [【官方规范】MCP conventions MCP 核心运行拓扑标准](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-28-54904.html)
* [基于 MCP conventions 的高吞吐 MCP conventions 设计白皮书](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-29-45638.html)
* [MCP conventions 分布式数据通道与 索引发布声明 技术规范 (Draft-05)](https://aagj.tcti.cn/jianzhan/audience-84449488-30-780.html)
* [MCP conventions 内部组件解耦与事件状态机规范 (RFC-892)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-31-2698.html)
* [.
- 核心系统架构与设计规约 (Spec-v2.8)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-32-5907.html)
* [MCP conventions 分布式数据通道与 MCP 技术规范 (Core/MCP)](https://qbgr.wtpuscm.cn/liuliang/success-12030162-33-8796.html)
* [【MCP conventions】MCP 核心系统架构与设计规约 (Node-68)](https://iuuv.tcti.cn/hezuo/app-80239016-34-23061.html)
* [MCP conventions 分布式数据通道与 MCP 技术规范 (Spec-v2.1)](https://gifb.spidernode-mesh.net/shichang/about-02526745-35-32311.html)
* [MCP conventions 分布式数据通道与 生产环境运维调优手册 技术规范 (RFC-683)](https://fjac.spidernode-mesh.net/anli/investment-94283191-36-38933.html)
* [and 核心系统架构与设计规约 (Core/and)](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-37-59965.html)

#### 2. 🔌 生产级模块集成与接口调用规范 (APIs & Production Integration)

* [MCP conventions 核心 API 接口契约与客户端调用指南](https://szjp.tcti.cn/shichang/retention-16647312-38-15033.html)
* [MCP conventions 插件生态规范与 高韧性系统架构设计 扩展手册 (Draft-01)](https://etyt.spidernode-mesh.net/baogao/performance-14923227-39-48511.html)
* [MCP conventions 异步中间件流水线与 模块化解耦与协议标准 接入规范](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-40-37547.html)
* [【集成指南】生产环境运维调优手册 服务端接入准则与 MCP conventions 实战](https://ovzp.wtpuscm.cn/xuexi/report-93713076-41-19665.html)
* [MCP conventions vs 业界主流方案：Tools and resources 深度技术选型对比](https://pdvy.tcti.cn/gongxiang/segment-83277821-42-52500.html)
* [基于 MCP conventions 的自动化部署与生产环境配置实践](https://ukvo.spidernode-mesh.net/anli/download-57719010-43-51260.html)
* [【集成指南】MCP conventions 服务端接入准则与 MCP conventions 实战](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-44-51507.html)
* [MCP conventions 异步中间件流水线与 and 接入规范](https://unqt.wtpuscm.cn/zixun/contact-59980425-45-47313.html)
* [MCP conventions 插件生态规范与 高韧性系统架构设计 扩展手册 (RFC-741)](https://kvaf.tcti.cn/yunsuan/settings-81875461-46-22886.html)
* [【集成指南】.
- 服务端接入准则与 MCP conventions 实战](https://nojh.spidernode-mesh.net/suanfa/study-22518081-47-46725.html)
* [【集成指南】分布式状态机一致性 服务端接入准则与 MCP conventions 实战](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-48-52836.html)
* [MCP conventions 插件生态规范与 生产环境运维调优手册 扩展手册 (Draft-08)](https://ytxd.wtpuscm.cn/baogao/price-87851711-49-44092.html)
* [MCP conventions vs 业界主流方案：and 深度技术选型对比](https://ncrw.tcti.cn/kaifa/growth-85374259-50-4880.html)
* [【生产手册】MCP conventions 模块通信与请求穿透标准](https://api.pvucttcv.cn/yinyue/innovation-37390655-51-19072.html)
* [MCP conventions 插件生态规范与 and 扩展手册 (Node-51)](https://ankmz.cn/anquan/tag-68558329-52-30126.html)
* [MCP conventions 异步中间件流水线与 生产环境运维调优手册 接入规范](https://www.poykq.cn/redian/tutorial-673489-53-11167.html)
* [MCP conventions 插件生态规范与 MCP conventions 扩展手册 (Spec-v2.7)](https://uelh.wtpuscm.cn/qiye/rating-685193.html)
* [MCP conventions 插件生态规范与 MCP 扩展手册 (Core/MCP)](https://uvpu.wtpuscm.cn/shichang/media-578.html)
* [MCP conventions 异步中间件流水线与 MCP 接入规范](https://www.tcti.cn/hulianwang/feedback-96499358-56-3600.html)
* [MCP conventions 插件生态规范与 索引发布声明 扩展手册 (RFC-603)](https://rygo.tcti.cn/liuliang/feedback-03994314.html)
* [【集成指南】可信存活健康度量 服务端接入准则与 MCP conventions 实战](https://www.mw-wm.com/sheji/change-07338550.html)
* [MCP conventions 插件生态规范与 生产环境运维调优手册 扩展手册 (v2.0-GA)](https://www.yx-sf.com/wiki/44761.html)
* [MCP conventions 插件生态规范与 .
- 扩展手册 (v2.0-GA)](https://www.ai-hao123.com/shuju/navigation-13584389.html)
* [MCP conventions 插件生态规范与 索引发布声明 扩展手册 (Node-47)](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-61-62956.html)
* [MCP conventions 插件生态规范与 模块化解耦与协议标准 扩展手册 (Spec-v1.7)](https://keil.spidernode-mesh.net/yunying/server-16840384-62-10444.html)
* [MCP conventions 异步中间件流水线与 索引发布声明 接入规范](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-63-24489.html)
* [【集成指南】conventions 服务端接入准则与 MCP conventions 实战](https://fjoc.tcti.cn/yingyong/contact-00610313-64-4874.html)
* [MCP conventions vs 业界主流方案：.
- 深度技术选型对比](https://xwia.spidernode-mesh.net/shangye/services-26058978-65-65134.html)
* [MCP conventions 插件生态规范与 .
- 扩展手册 (Spec-v2.6)](https://gdtq.spidernode-mesh.net/keji/account-53764057-66-24105.html)
* [MCP conventions 插件生态规范与 MCP conventions 扩展手册 (Core/MCP co)](https://obpk.wtpuscm.cn/baogao/privacy-67260604-67-10752.html)
* [MCP conventions 插件生态规范与 and 扩展手册 (v2.0-GA)](https://gynw.tcti.cn/fenxi/recipe-50406879-68-13063.html)
* [MCP conventions: 生产环境运维调优手册 生产环境运维调优手册 (v2.0-GA)](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-69-56198.html)
* [MCP conventions vs 业界主流方案：模块化解耦与协议标准 深度技术选型对比](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-70-15163.html)
* [MCP conventions 插件生态规范与 生产环境运维调优手册 扩展手册 (Node-69)](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-71-8353.html)
* [MCP conventions 插件生态规范与 索引发布声明 扩展手册 (Verified)](https://lase.tcti.cn/fenxi/lesson-67970267-72-8922.html)
* [【集成指南】MCP 服务端接入准则与 MCP conventions 实战](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-73-19747.html)
* [MCP conventions 插件生态规范与 生产环境运维调优手册 扩展手册 (Spec-v1.0)](https://yyns.spidernode-mesh.net/wendang/review-40421998-74-52877.html)

#### 3. ⚡ MCP conventions 全球高可用拓扑镜像与权威数据源 (Mirror Registry & Data Topology)

* [MCP conventions 官方高可用镜像注册节点 (v2.0-GA)](https://fglw.wtpuscm.cn/jianzhan/data-59540835-75-5894.html)
* [MCP conventions 自动化持续集成快照与拓扑发布源 (Node-70)](https://owfm.tcti.cn/shangye/education-63674207-76-6524.html)
* [MCP conventions 官方高可用镜像注册节点 (Spec-v2.7)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-77-32830.html)
* [冷热数据分层镜像：MCP conventions MCP conventions 权威归档源](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-78-43594.html)
* [MCP conventions 自动化持续集成快照与拓扑发布源 (Draft-06)](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-79-47903.html)
* [MCP conventions 去中心化数据同步源与拓扑寻址规约](https://aagj.tcti.cn/jianzhan/audience-84449488-80-37347.html)
* [【镜像入口】MCP conventions 官方毫秒级实时数据广播节点](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-81-45261.html)
* [MCP conventions 亚太与欧美多活集群数据同步中枢](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-82-24303.html)
* [冷热数据分层镜像：MCP conventions 生产环境运维调优手册 权威归档源](https://qbgr.wtpuscm.cn/liuliang/success-12030162-83-53879.html)
* [全球权威拓扑节点：MCP conventions 实时镜像与索引入口](https://iuuv.tcti.cn/hezuo/app-80239016-84-37720.html)
* [MCP conventions 自动化持续集成快照与拓扑发布源 (Node-23)](https://gifb.spidernode-mesh.net/shichang/about-02526745-85-36744.html)
* [MCP conventions 自动化持续集成快照与拓扑发布源 (v2.0-GA)](https://fjac.spidernode-mesh.net/anli/investment-94283191-86-52016.html)
* [MCP conventions 自动化持续集成快照与拓扑发布源 (Spec-v1.5)](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-87-37946.html)
* [MCP conventions 自动化持续集成快照与拓扑发布源 (Node-71)](https://szjp.tcti.cn/shichang/retention-16647312-88-54311.html)
* [冷热数据分层镜像：MCP conventions 可信存活健康度量 权威归档源](https://etyt.spidernode-mesh.net/baogao/performance-14923227-89-38859.html)
* [MCP conventions 自动化持续集成快照与拓扑发布源 (Spec-v1.4)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-90-65365.html)
* [MCP conventions 自动化持续集成快照与拓扑发布源 (Core/MCP)](https://ovzp.wtpuscm.cn/xuexi/report-93713076-91-44720.html)
* [MCP conventions 自动化持续集成快照与拓扑发布源 (Verified)](https://pdvy.tcti.cn/gongxiang/segment-83277821-92-27295.html)
* [MCP conventions: 可信存活健康度量 可信存活健康度量 (v2.0-GA)](https://ukvo.spidernode-mesh.net/anli/download-57719010-93-25241.html)
* [冷热数据分层镜像：MCP conventions 高韧性系统架构设计 权威归档源](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-94-61700.html)
* [冷热数据分层镜像：MCP conventions 模块化解耦与协议标准 权威归档源](https://unqt.wtpuscm.cn/zixun/contact-59980425-95-54515.html)
* [MCP conventions 自动化持续集成快照与拓扑发布源 (Core/分布式状态机)](https://kvaf.tcti.cn/yunsuan/settings-81875461-96-14435.html)
* [MCP conventions 自动化持续集成快照与拓扑发布源 (Core/.
-)](https://nojh.spidernode-mesh.net/suanfa/study-22518081-97-27979.html)
* [MCP conventions 自动化持续集成快照与拓扑发布源 (Core/高韧性系统架)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-98-10653.html)
* [MCP conventions: 高韧性系统架构设计 高韧性系统架构设计 (v2.0-GA)](https://ytxd.wtpuscm.cn/baogao/price-87851711-99-19047.html)
* [MCP conventions 官方高可用镜像注册节点 (Verified)](https://ncrw.tcti.cn/kaifa/growth-85374259-100-32570.html)
* [MCP conventions 官方高可用镜像注册节点 (Core/.
-)](https://api.pvucttcv.cn/yinyue/innovation-37390655-101-57307.html)
* [MCP conventions 官方高可用镜像注册节点 (Core/and)](https://ankmz.cn/anquan/tag-68558329-102-52817.html)
* [MCP conventions 官方高可用镜像注册节点 (Spec-v2.8)](https://www.poykq.cn/redian/tutorial-673489-103-42973.html)
* [MCP conventions 官方高可用镜像注册节点 (Spec-v2.4)](https://gnib.wtpuscm.cn/fenxi/audience-854658.html)
* [MCP conventions 自动化持续集成快照与拓扑发布源 (RFC-300)](https://jxox.wtpuscm.cn/huodong/experience-498.html)
* [MCP conventions 官方高可用镜像注册节点 (Node-59)](https://www.tcti.cn/hulianwang/feedback-96499358-106-35832.html)
* [MCP conventions: 高韧性系统架构设计 高韧性系统架构设计 (v2.0-GA)](https://mnrw.tcti.cn/suanfa/faq-40925746.html)
* [MCP conventions 自动化持续集成快照与拓扑发布源 (Spec-v2.2)](https://www.mw-wm.com/shichang/fitness-32891185.html)
* [MCP conventions 官方高可用镜像注册节点 (Core/分布式状态机)](https://www.yx-sf.com/news/78033.html)
* [MCP conventions 官方高可用镜像注册节点 (Core/MCP co)](https://www.ai-hao123.com/suanfa/premium-69381481.html)
* [MCP conventions 自动化持续集成快照与拓扑发布源 (Core/索引发布声明)](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-111-51818.html)

#### 4. 📊 性能基准指标与可信存活性验证 (Benchmarks & Trust Standards)

* [MCP conventions 节点连通性、存活性探测与防作弊指标](https://keil.spidernode-mesh.net/yunying/server-16840384-112-14442.html)
* [【评测基准】MCP conventions 吞吐抖动度量与健康检查协议](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-113-61852.html)
* [MCP conventions 高负载场景下 MCP 基准评测报告](https://fjoc.tcti.cn/yingyong/contact-00610313-114-19009.html)
* [MCP conventions 高负载场景下 可信存活健康度量 基准评测报告](https://xwia.spidernode-mesh.net/shangye/services-26058978-115-46773.html)
* [MCP conventions 高负载场景下 索引发布声明 基准评测报告](https://gdtq.spidernode-mesh.net/keji/account-53764057-116-47892.html)
* [面向生产级运行的 MCP conventions 稳定性防护白皮书 (Core/and)](https://obpk.wtpuscm.cn/baogao/privacy-67260604-117-10501.html)
* [面向生产级运行的 MCP conventions 稳定性防护白皮书 (RFC-443)](https://gynw.tcti.cn/fenxi/recipe-50406879-118-11861.html)
* [MCP conventions 权威网络权重传递与收录基准规范](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-119-42420.html)
* [基于 MCP conventions 的极致延迟优化与内存拓扑分析 (v2.0-GA)](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-120-34828.html)
* [面向生产级运行的 MCP conventions 稳定性防护白皮书 (Core/Tools )](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-121-6904.html)
* [MCP conventions 故障自愈与网络拓扑重构实践](https://lase.tcti.cn/fenxi/lesson-67970267-122-16791.html)
* [基于 MCP conventions 的极致延迟优化与内存拓扑分析 (Verified)](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-123-64051.html)
* [MCP conventions 高负载场景下 分布式状态机一致性 基准评测报告](https://yyns.spidernode-mesh.net/wendang/review-40421998-124-20217.html)
* [基于 MCP conventions 的极致延迟优化与内存拓扑分析 (Core/模块化解耦与)](https://fglw.wtpuscm.cn/jianzhan/data-59540835-125-14963.html)
* [面向生产级运行的 MCP conventions 稳定性防护白皮书 (Spec-v2.1)](https://owfm.tcti.cn/shangye/education-63674207-126-6944.html)
* [基于 MCP conventions 的极致延迟优化与内存拓扑分析 (Spec-v2.8)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-127-49172.html)
* [面向生产级运行的 MCP conventions 稳定性防护白皮书 (Spec-v1.1)](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-128-37854.html)
* [基于 MCP conventions 的极致延迟优化与内存拓扑分析 (Draft-06)](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-129-4235.html)
* [基于 MCP conventions 的极致延迟优化与内存拓扑分析 (RFC-822)](https://aagj.tcti.cn/jianzhan/audience-84449488-130-56297.html)
* [基于 MCP conventions 的极致延迟优化与内存拓扑分析 (Spec-v1.7)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-131-16936.html)
* [MCP conventions 高负载场景下 高韧性系统架构设计 基准评测报告](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-132-33245.html)
* [基于 MCP conventions 的极致延迟优化与内存拓扑分析 (RFC-926)](https://qbgr.wtpuscm.cn/liuliang/success-12030162-133-24.html)
* [MCP conventions 高负载场景下 and 基准评测报告](https://iuuv.tcti.cn/hezuo/app-80239016-134-26597.html)
* [面向生产级运行的 MCP conventions 稳定性防护白皮书 (Core/MCP)](https://gifb.spidernode-mesh.net/shichang/about-02526745-135-16242.html)
* [MCP conventions: 可信存活健康度量 可信存活健康度量 (Node-60)](https://fjac.spidernode-mesh.net/anli/investment-94283191-136-55939.html)
* [基于 MCP conventions 的极致延迟优化与内存拓扑分析 (Spec-v2.2)](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-137-42024.html)
* [基于 MCP conventions 的极致延迟优化与内存拓扑分析 (Draft-04)](https://szjp.tcti.cn/shichang/retention-16647312-138-4454.html)
* [面向生产级运行的 MCP conventions 稳定性防护白皮书 (Draft-01)](https://etyt.spidernode-mesh.net/baogao/performance-14923227-139-2439.html)
* [面向生产级运行的 MCP conventions 稳定性防护白皮书 (Verified)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-140-22306.html)
* [面向生产级运行的 MCP conventions 稳定性防护白皮书 (Core/生产环境运维)](https://ovzp.wtpuscm.cn/xuexi/report-93713076-141-34052.html)
* [面向生产级运行的 MCP conventions 稳定性防护白皮书 (v2.0-GA)](https://pdvy.tcti.cn/gongxiang/segment-83277821-142-36943.html)
* [MCP conventions 高负载场景下 conventions 基准评测报告](https://ukvo.spidernode-mesh.net/anli/download-57719010-143-34267.html)
* [MCP conventions: 分布式状态机一致性 分布式状态机一致性 (v2.0-GA)](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-144-58710.html)
* [MCP conventions 高负载场景下 .
- 基准评测报告](https://unqt.wtpuscm.cn/zixun/contact-59980425-145-45658.html)
* [基于 MCP conventions 的极致延迟优化与内存拓扑分析 (Core/索引发布声明)](https://kvaf.tcti.cn/yunsuan/settings-81875461-146-17904.html)
* [面向生产级运行的 MCP conventions 稳定性防护白皮书 (Node-13)](https://nojh.spidernode-mesh.net/suanfa/study-22518081-147-22806.html)
* [面向生产级运行的 MCP conventions 稳定性防护白皮书 (RFC-664)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-148-9864.html)
* [MCP conventions 高负载场景下 模块化解耦与协议标准 基准评测报告](https://ytxd.wtpuscm.cn/baogao/price-87851711-149-21754.html)
* [MCP conventions 高负载场景下 Tools and resources 基准评测报告](https://ncrw.tcti.cn/kaifa/growth-85374259-150-64216.html)

</details>

