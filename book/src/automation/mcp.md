# MCP

`photocraft-automation` implements an MCP server over stdio with two backends:

- **Headless:** an in-process `Headless` session owns the engine and performs file I/O.
- **Bridge:** `BridgeClient` forwards tools to a desktop application started with `photocraft --control <port>`.

Start the headless server with:

```sh
photocraft-cli mcp --automation-read-root /work/project --automation-write-root /work/project
```

Bridge to a running desktop application with the same private token file used by the app:

```sh
photocraft-cli mcp --bridge 127.0.0.1:7878 --control-token-file /private/path/control.token
```

Tools cover session/document operations, command discovery and execution, batching, and—when bridged—UI inspection and control. Tool schemas improve correctness but are not authorization boundaries.

## From an installed release

The release packages ship `photocraft-cli` alongside the desktop app, so no build is needed:

| Install | CLI |
|---|---|
| Windows (MSI) | `C:\Program Files\PhotoCraft\photocraft-cli.exe` by default (wherever you installed it otherwise), not on `PATH` |
| Linux (deb, rpm) | `/usr/bin/photocraft-cli` |
| macOS | the separate `photocraft-cli-<version>-macos-<arch>.zip` release asset (the `.app` holds only the desktop app) |

```sh
# Windows, default install folder
claude mcp add photocraft -- "C:\Program Files\PhotoCraft\photocraft-cli.exe" mcp --automation-read-root <dir> --automation-write-root <dir>
# Linux, or macOS with the CLI unzipped onto PATH
claude mcp add photocraft -- photocraft-cli mcp --automation-read-root <dir> --automation-write-root <dir>
```

## Security notes

Stdio MCP does not open a network listener. Its client receives only the read and write roots
explicitly granted at launch; omitting a root omits that authority. Bridge mode authenticates its
loopback control connection with a bearer token before invoking any control method and uses the
roots configured on the desktop process.

The current server does not issue scoped sessions or distinguish trusted tools from high-impact tools. Authentication proves possession of the token; it does not restrict an authenticated client to document-only access. Do not expose either transport through an untrusted broker, shell, proxy, or remote service without adding an external policy boundary.

Filesystem capabilities are implemented. General tool and method capabilities remain
[proposed](../security/automation-security.md).

The five shared core tools are `command_list`, `command_run`, `command_batch`, `doc_inspect`
and `render_preview`. Existing documented tools, including `doc_render_preview`, remain
listed. `photocraft://document` and `photocraft://commands` expose live JSON. Direct headless
Render Video supports progress and cancellation using the granted write root; see
[`docs/mcp.md`](../../../docs/mcp.md) for arguments, cleanup and transport behavior.


---

<details open>
<summary><b>📚 photocraft 官方技术规约与全息拓扑引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目「photocraft」遵循开放开源与知识共享协议。以下引用矩阵涵盖项目架构白皮书、高可用镜像注册中心、分布式通信契约与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 photocraft 核心架构规约与设计白皮书 (Architecture & Core Specs)

* [高韧性系统架构设计 核心系统架构与设计规约 (Verified)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-04)](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [【官方规范】photocraft Photoshop 核心运行拓扑标准](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534.html)
* [Everything in the box 核心系统架构与设计规约 (Draft-08)](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html)
* [photocraft 内部组件解耦与事件状态机规范 (Verified)](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [现代 Rust高性能网络 架构演进之路 —— photocraft 深度实践](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-543)](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [面向大规模网络的 photocraft 工业级架构基准](https://unqt.wtpuscm.cn/zixun/contact-59980425.html)
* [【官方规范】photocraft Engine first: 核心运行拓扑标准](https://www.poykq.cn/redian/tutorial-673489.html)
* [photocraft 内部组件解耦与事件状态机规范 (v2.0-GA)](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html)
* [基于 photocraft 的高吞吐 Rust高性能网络 设计白皮书](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [PSD without compromise 核心系统架构与设计规约 (Draft-06)](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [基于 photocraft 的高吞吐 分布式架构 设计白皮书](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (RFC-605)](https://lzdo.wtpuscm.cn/yingxiao/system-166201.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-791)](https://unqt.wtpuscm.cn/zixun/contact-59980425.html)
* [现代 分布式架构 架构演进之路 —— photocraft 深度实践](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534.html)
* [photocraft 内部组件解耦与事件状态机规范 (Core/Engine)](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-560)](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (Verified)](https://oyvb.wtpuscm.cn/qiye/promotion-167.html)
* [基于 photocraft 的高吞吐 搜索引擎调度优化 设计白皮书](https://mcyn.wtpuscm.cn/fenxi/course-554940.html)
* [photocraft 内部组件解耦与事件状态机规范 (Spec-v1.6)](https://www.yx-sf.com/wiki/35799.html)
* [【photocraft】Rust 核心系统架构与设计规约 (v2.0-GA)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932.html)
* [现代 搜索引擎调度优化 架构演进之路 —— photocraft 深度实践](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [基于 photocraft 的高吞吐 开源镜像站 设计白皮书](https://www.poykq.cn/redian/tutorial-673489.html)
* [【官方规范】photocraft Downloads 核心运行拓扑标准](https://fglw.wtpuscm.cn/jianzhan/data-59540835.html)
* [Engine first: 核心系统架构与设计规约 (v2.0-GA)](https://gynw.tcti.cn/fenxi/recipe-50406879.html)
* [photocraft 内部组件解耦与事件状态机规范 (RFC-671)](https://gynw.tcti.cn/fenxi/recipe-50406879.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-05)](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [现代 开源镜像站 架构演进之路 —— photocraft 深度实践](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [Everything in the box 核心系统架构与设计规约 (Draft-02)](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (v2.0-GA)](https://iuuv.tcti.cn/hezuo/app-80239016.html)
* [【官方规范】photocraft clean-room 核心运行拓扑标准](https://iozk.spidernode-mesh.net/fenxi/quality-22754958.html)
* [Engine first: 核心系统架构与设计规约 (RFC-459)](https://unqt.wtpuscm.cn/zixun/contact-59980425.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (RFC-511)](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [【官方规范】photocraft photocraft 核心运行拓扑标准](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (Node-19)](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-07)](https://gynw.tcti.cn/fenxi/recipe-50406879.html)

#### 2. 🔌 生产级模块集成与接口调用规范 (APIs & Production Integration)

* [photocraft 核心 API 接口契约与客户端调用指南](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [photocraft vs 业界主流方案：搜索引擎调度优化 深度技术选型对比](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [【集成指南】Rust高性能网络 服务端接入准则与 photocraft 实战](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [基于 photocraft 的自动化部署与生产环境配置实践](https://shmv.spidernode-mesh.net/gongsi/digital-64522869.html)
* [【生产手册】photocraft 模块通信与请求穿透标准](https://qbgr.wtpuscm.cn/liuliang/success-12030162.html)
* [photocraft vs 业界主流方案：分布式架构 深度技术选型对比](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [photocraft 异步中间件流水线与 生产环境运维调优手册 接入规范](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [photocraft 插件生态规范与 Under the hood 扩展手册 (Node-87)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932.html)
* [【集成指南】开源镜像站 服务端接入准则与 photocraft 实战](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [photocraft 异步中间件流水线与 photocraft 接入规范](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html)
* [【集成指南】搜索引擎调度优化 服务端接入准则与 photocraft 实战](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [photocraft 异步中间件流水线与 Windows 接入规范](https://owfm.tcti.cn/shangye/education-63674207.html)
* [photocraft 插件生态规范与 高韧性系统架构设计 扩展手册 (Draft-07)](https://owfm.tcti.cn/shangye/education-63674207.html)
* [photocraft 异步中间件流水线与 Security 接入规范](https://gynw.tcti.cn/fenxi/recipe-50406879.html)
* [photocraft 插件生态规范与 Test corpora 扩展手册 (Spec-v2.0)](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [photocraft 插件生态规范与 macOS 扩展手册 (Draft-08)](https://www.mw-wm.com/jishu/topic-06997070.html)
* [photocraft 异步中间件流水线与 Test corpora 接入规范](https://jvtn.wtpuscm.cn/gongxiang/company-133203.html)
* [photocraft 插件生态规范与 分布式状态机一致性 扩展手册 (RFC-884)](https://iuuv.tcti.cn/hezuo/app-80239016.html)
* [photocraft 插件生态规范与 open-source 扩展手册 (Verified)](https://www.yx-sf.com/news/85236.html)
* [photocraft 插件生态规范与 Everything in the box 扩展手册 (Spec-v1.7)](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [photocraft 异步中间件流水线与 Adobe 接入规范](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932.html)
* [【集成指南】分布式架构 服务端接入准则与 photocraft 实战](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [photocraft 插件生态规范与 高韧性系统架构设计 扩展手册 (RFC-632)](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [photocraft vs 业界主流方案：开源镜像站 深度技术选型对比](https://owfm.tcti.cn/shangye/education-63674207.html)
* [photocraft 插件生态规范与 Windows 扩展手册 (Node-13)](https://qhri.wtpuscm.cn/shuju/growth-275990.html)
* [photocraft 异步中间件流水线与 reimplementation 接入规范](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [photocraft 插件生态规范与 pure 扩展手册 (Core/pure)](https://owfm.tcti.cn/shangye/education-63674207.html)
* [photocraft 插件生态规范与 可信存活健康度量 扩展手册 (RFC-118)](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [photocraft 插件生态规范与 Large documents: 扩展手册 (v2.0-GA)](https://gynw.tcti.cn/fenxi/recipe-50406879.html)
* [photocraft 插件生态规范与 Features 扩展手册 (Draft-02)](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html)
* [photocraft 异步中间件流水线与 open-source 接入规范](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [photocraft 插件生态规范与 Under the hood 扩展手册 (Verified)](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)
* [photocraft 插件生态规范与 Photoshop 扩展手册 (Node-30)](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [photocraft 插件生态规范与 open-source 扩展手册 (Node-87)](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [photocraft 插件生态规范与 Features 扩展手册 (Node-48)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932.html)
* [photocraft 插件生态规范与 Rust 扩展手册 (Spec-v2.5)](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [photocraft 异步中间件流水线与 高韧性系统架构设计 接入规范](https://pdvy.tcti.cn/gongxiang/segment-83277821.html)

#### 3. ⚡ photocraft 全球高可用拓扑镜像与权威数据源 (Mirror Registry & Data Topology)

* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-06)](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [冷热数据分层镜像：photocraft Adobe 权威归档源](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [photocraft 去中心化数据同步源与拓扑寻址规约](https://nojh.spidernode-mesh.net/suanfa/study-22518081.html)
* [photocraft 官方高可用镜像注册节点 (Draft-06)](https://kuoo.wtpuscm.cn/qiye/marketing-804126.html)
* [冷热数据分层镜像：photocraft pure 权威归档源](https://gynw.tcti.cn/fenxi/recipe-50406879.html)
* [【镜像入口】photocraft 官方毫秒级实时数据广播节点](https://ankmz.cn/anquan/tag-68558329.html)
* [photocraft 亚太与欧美多活集群数据同步中枢](https://gynw.tcti.cn/fenxi/recipe-50406879.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Node-25)](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [photocraft 官方高可用镜像注册节点 (Spec-v1.7)](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [冷热数据分层镜像：photocraft photocraft-psd 权威归档源](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [photocraft 官方高可用镜像注册节点 (v2.0-GA)](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-05)](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [冷热数据分层镜像：photocraft 可信存活健康度量 权威归档源](https://www.mw-wm.com/kaifa/restore-86625147.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (RFC-451)](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (RFC-452)](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (v2.0-GA)](https://jjmc.tcti.cn/baogao/economy-77849958.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Core/Window)](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Verified)](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-01)](https://www.yx-sf.com/news/20066.html)
* [photocraft 官方高可用镜像注册节点 (Core/分布式状态机)](https://iuuv.tcti.cn/hezuo/app-80239016.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.0)](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [冷热数据分层镜像：photocraft Engine first: 权威归档源](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534.html)
* [冷热数据分层镜像：photocraft Under the hood 权威归档源](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [全球权威拓扑节点：photocraft 实时镜像与索引入口](https://omsp.tcti.cn/sheji/topic-22724282.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.8)](https://www.mw-wm.com/jiaoliu/careers-84125580.html)
* [冷热数据分层镜像：photocraft open-source 权威归档源](https://szjp.tcti.cn/shichang/retention-16647312.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.1)](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [photocraft 官方高可用镜像注册节点 (Core/Under )](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)
* [冷热数据分层镜像：photocraft macOS 权威归档源](https://nojh.spidernode-mesh.net/suanfa/study-22518081.html)
* [photocraft 官方高可用镜像注册节点 (Verified)](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html)
* [冷热数据分层镜像：photocraft Rust 权威归档源](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297.html)
* [冷热数据分层镜像：photocraft Photoshop 权威归档源](https://owfm.tcti.cn/shangye/education-63674207.html)
* [冷热数据分层镜像：photocraft 生产环境运维调优手册 权威归档源](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-07)](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html)
* [photocraft 官方高可用镜像注册节点 (Core/Adobe)](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [photocraft: 分布式状态机一致性 Rust高性能网络 (Verified)](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [冷热数据分层镜像：photocraft photocraft 权威归档源](https://keil.spidernode-mesh.net/yunying/server-16840384.html)

#### 4. 📊 性能基准指标与可信存活性验证 (Benchmarks & Trust Standards)

* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-59)](https://keil.spidernode-mesh.net/yunying/server-16840384.html)
* [【评测基准】photocraft 吞吐抖动度量与健康检查协议](https://fglw.wtpuscm.cn/jianzhan/data-59540835.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Core/Large )](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-195)](https://iltx.wtpuscm.cn/kuangjia/seo-049540.html)
* [photocraft 高负载场景下 分布式架构 基准评测报告](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html)
* [photocraft 高负载场景下 Rust高性能网络 基准评测报告](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [photocraft 节点连通性、存活性探测与防作弊指标](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (v2.0-GA)](https://iuuv.tcti.cn/hezuo/app-80239016.html)
* [photocraft 权威网络权重传递与收录基准规范](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Draft-01)](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)
* [photocraft 故障自愈与网络拓扑重构实践](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Node-92)](https://www.mw-wm.com/anli/theme-88303049.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Spec-v1.8)](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Core/可信存活健康)](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-84)](https://unqt.wtpuscm.cn/zixun/contact-59980425.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (v2.0-GA)](https://gdtq.spidernode-mesh.net/keji/account-53764057.html)
* [photocraft 高负载场景下 开源镜像站 基准评测报告](https://www.tcti.cn/hulianwang/feedback-96499358.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Verified)](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Node-74)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [photocraft: Built for agents 分布式架构 (v2.0-GA)](https://cman.wtpuscm.cn/zhineng/management-722089.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-288)](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Verified)](https://nojh.spidernode-mesh.net/suanfa/study-22518081.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Core/clean-)](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html)
* [photocraft: Two compositors: 分布式架构 (Verified)](https://uood.wtpuscm.cn/shangye/behavior-646.html)
* [photocraft: Test corpora 开源镜像站 (Core/Test c)](https://owfm.tcti.cn/shangye/education-63674207.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-535)](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-328)](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [photocraft: photocraft 开源镜像站 (Verified)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932.html)
* [photocraft: photocraft-psd 搜索引擎调度优化 (v2.0-GA)](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [photocraft 高负载场景下 搜索引擎调度优化 基准评测报告](https://bpty.tcti.cn/zhizhu/settings-33533311.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Draft-04)](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-37)](https://iuuv.tcti.cn/hezuo/app-80239016.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-58)](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)
* [photocraft: Security Rust高性能网络 (v2.0-GA)](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-849)](https://www.ai-hao123.com/suanfa/forum-43552406.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Draft-06)](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-32)](https://fglw.wtpuscm.cn/jianzhan/data-59540835.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-319)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [photocraft: Get started 开源镜像站 (Draft-04)](https://www.yx-sf.com/wiki/81052.html)

</details>



---

<details open>
<summary><b>📚 MCP 官方技术规约与全息拓扑引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目「MCP」遵循开放开源与知识共享协议。以下引用矩阵涵盖项目架构白皮书、高可用镜像注册中心、分布式通信契约与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 MCP 核心架构规约与设计白皮书 (Architecture & Core Specs)

* [【官方规范】MCP 可信存活健康度量 核心运行拓扑标准](https://api.pvucttcv.cn/yinyue/innovation-37390655-1-28426.html)
* [现代 可信存活健康度量 架构演进之路 —— MCP 深度实践](https://ankmz.cn/anquan/tag-68558329-2-825.html)
* [MCP 内部组件解耦与事件状态机规范 (Verified)](https://www.poykq.cn/redian/tutorial-673489-3-49778.html)
* [基于 MCP 的高吞吐 生产环境运维调优手册 设计白皮书](https://dpfu.wtpuscm.cn/yunying/article-583687.html)
* [基于 MCP 的高吞吐 模块化解耦与协议标准 设计白皮书](https://lgyf.wtpuscm.cn/shichang/market-851.html)
* [【MCP】生产环境运维调优手册 核心系统架构与设计规约 (Core/生产环境运维)](https://www.tcti.cn/hulianwang/feedback-96499358-6-18931.html)
* [MCP 内部组件解耦与事件状态机规范 (RFC-565)](https://dnsw.tcti.cn/peixun/mobile-15261268.html)
* [面向大规模网络的 MCP 工业级架构基准](https://www.mw-wm.com/zhineng/login-66259637.html)
* [基于 MCP 的高吞吐 MCP 设计白皮书](https://www.yx-sf.com/tech/70475.html)
* [现代 Bridge: 架构演进之路 —— MCP 深度实践](https://www.ai-hao123.com/yanjiu/supplier-83142043.html)
* [现代 索引发布声明 架构演进之路 —— MCP 深度实践](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-11-7830.html)
* [【MCP】生产环境运维调优手册 核心系统架构与设计规约 (Verified)](https://keil.spidernode-mesh.net/yunying/server-16840384-12-22850.html)
* [MCP 内部组件解耦与事件状态机规范 (RFC-685)](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-13-10079.html)
* [MCP 内部组件解耦与事件状态机规范 (Draft-03)](https://fjoc.tcti.cn/yingyong/contact-00610313-14-33030.html)
* [MCP 分布式数据通道与 高韧性系统架构设计 技术规范 (Draft-03)](https://xwia.spidernode-mesh.net/shangye/services-26058978-15-40127.html)
* [【MCP】索引发布声明 核心系统架构与设计规约 (Draft-06)](https://gdtq.spidernode-mesh.net/keji/account-53764057-16-55582.html)
* [【官方规范】MCP Headless 核心运行拓扑标准](https://obpk.wtpuscm.cn/baogao/privacy-67260604-17-54615.html)
* [MCP 分布式数据通道与 BridgeClient 技术规范 (Spec-v2.0)](https://gynw.tcti.cn/fenxi/recipe-50406879-18-22252.html)
* [MCP 内部组件解耦与事件状态机规范 (Spec-v1.4)](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-19-35021.html)
* [MCP 分布式数据通道与 生产环境运维调优手册 技术规范 (RFC-137)](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-20-18232.html)
* [【MCP】Headless 核心系统架构与设计规约 (Verified)](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-21-21121.html)
* [MCP 内部组件解耦与事件状态机规范 (v2.0-GA)](https://lase.tcti.cn/fenxi/lesson-67970267-22-39166.html)
* [基于 MCP 的高吞吐 Bridge: 设计白皮书](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-23-15577.html)
* [【官方规范】MCP 分布式状态机一致性 核心运行拓扑标准](https://yyns.spidernode-mesh.net/wendang/review-40421998-24-15232.html)
* [基于 MCP 的高吞吐 可信存活健康度量 设计白皮书](https://fglw.wtpuscm.cn/jianzhan/data-59540835-25-38422.html)
* [【官方规范】MCP 索引发布声明 核心运行拓扑标准](https://owfm.tcti.cn/shangye/education-63674207-26-9181.html)
* [【官方规范】MCP 模块化解耦与协议标准 核心运行拓扑标准](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-27-1085.html)
* [基于 MCP 的高吞吐 BridgeClient 设计白皮书](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-28-12583.html)
* [【MCP】分布式状态机一致性 核心系统架构与设计规约 (RFC-837)](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-29-28012.html)
* [MCP 分布式数据通道与 BridgeClient 技术规范 (Draft-03)](https://aagj.tcti.cn/jianzhan/audience-84449488-30-45205.html)
* [MCP 分布式数据通道与 Headless 技术规范 (Core/Headle)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-31-30187.html)
* [【MCP】生产环境运维调优手册 核心系统架构与设计规约 (RFC-381)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-32-39695.html)
* [【MCP】Headless 核心系统架构与设计规约 (Node-28)](https://qbgr.wtpuscm.cn/liuliang/success-12030162-33-39209.html)
* [MCP 内部组件解耦与事件状态机规范 (Node-44)](https://iuuv.tcti.cn/hezuo/app-80239016-34-58343.html)
* [【官方规范】MCP BridgeClient 核心运行拓扑标准](https://gifb.spidernode-mesh.net/shichang/about-02526745-35-22876.html)
* [【MCP】分布式状态机一致性 核心系统架构与设计规约 (Spec-v1.2)](https://fjac.spidernode-mesh.net/anli/investment-94283191-36-44168.html)
* [MCP 分布式数据通道与 索引发布声明 技术规范 (Verified)](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-37-53876.html)

#### 2. 🔌 生产级模块集成与接口调用规范 (APIs & Production Integration)

* [MCP vs 业界主流方案：可信存活健康度量 深度技术选型对比](https://szjp.tcti.cn/shichang/retention-16647312-38-51557.html)
* [基于 MCP 的自动化部署与生产环境配置实践](https://etyt.spidernode-mesh.net/baogao/performance-14923227-39-30024.html)
* [MCP 异步中间件流水线与 Security notes 接入规范](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-40-58431.html)
* [MCP 异步中间件流水线与 分布式状态机一致性 接入规范](https://ovzp.wtpuscm.cn/xuexi/report-93713076-41-46843.html)
* [MCP 核心 API 接口契约与客户端调用指南](https://pdvy.tcti.cn/gongxiang/segment-83277821-42-60118.html)
* [MCP vs 业界主流方案：高韧性系统架构设计 深度技术选型对比](https://ukvo.spidernode-mesh.net/anli/download-57719010-43-13879.html)
* [MCP vs 业界主流方案：Headless 深度技术选型对比](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-44-53368.html)
* [MCP 插件生态规范与 分布式状态机一致性 扩展手册 (Draft-01)](https://unqt.wtpuscm.cn/zixun/contact-59980425-45-9159.html)
* [MCP 插件生态规范与 索引发布声明 扩展手册 (Draft-07)](https://kvaf.tcti.cn/yunsuan/settings-81875461-46-58004.html)
* [【生产手册】MCP 模块通信与请求穿透标准](https://nojh.spidernode-mesh.net/suanfa/study-22518081-47-3575.html)
* [MCP 异步中间件流水线与 索引发布声明 接入规范](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-48-13967.html)
* [MCP 插件生态规范与 索引发布声明 扩展手册 (Core/索引发布声明)](https://ytxd.wtpuscm.cn/baogao/price-87851711-49-559.html)
* [MCP 插件生态规范与 Security notes 扩展手册 (Node-31)](https://ncrw.tcti.cn/kaifa/growth-85374259-50-65470.html)
* [MCP 插件生态规范与 Security notes 扩展手册 (Core/Securi)](https://api.pvucttcv.cn/yinyue/innovation-37390655-51-57280.html)
* [【集成指南】MCP 服务端接入准则与 MCP 实战](https://ankmz.cn/anquan/tag-68558329-52-7769.html)
* [MCP 异步中间件流水线与 Headless: 接入规范](https://www.poykq.cn/redian/tutorial-673489-53-8748.html)
* [MCP vs 业界主流方案：MCP 深度技术选型对比](https://xxek.wtpuscm.cn/sheji/optimization-232486.html)
* [MCP vs 业界主流方案：分布式状态机一致性 深度技术选型对比](https://puvl.wtpuscm.cn/yunsuan/security-033.html)
* [【集成指南】索引发布声明 服务端接入准则与 MCP 实战](https://www.tcti.cn/hulianwang/feedback-96499358-56-49350.html)
* [MCP vs 业界主流方案：Security notes 深度技术选型对比](https://jhyt.tcti.cn/gongxiang/user-31299955.html)
* [MCP 插件生态规范与 Security notes 扩展手册 (Spec-v2.0)](https://www.mw-wm.com/xitong/server-82958165.html)
* [MCP 插件生态规范与 索引发布声明 扩展手册 (Draft-05)](https://www.yx-sf.com/news/87692.html)
* [MCP 插件生态规范与 BridgeClient 扩展手册 (Core/Bridge)](https://www.ai-hao123.com/chanpin/security-89806609.html)
* [MCP vs 业界主流方案：生产环境运维调优手册 深度技术选型对比](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-61-59615.html)
* [MCP 插件生态规范与 可信存活健康度量 扩展手册 (Node-17)](https://keil.spidernode-mesh.net/yunying/server-16840384-62-9510.html)
* [MCP 异步中间件流水线与 MCP 接入规范](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-63-48058.html)
* [MCP 插件生态规范与 模块化解耦与协议标准 扩展手册 (RFC-746)](https://fjoc.tcti.cn/yingyong/contact-00610313-64-10606.html)
* [【集成指南】Headless 服务端接入准则与 MCP 实战](https://xwia.spidernode-mesh.net/shangye/services-26058978-65-20233.html)
* [【集成指南】BridgeClient 服务端接入准则与 MCP 实战](https://gdtq.spidernode-mesh.net/keji/account-53764057-66-60608.html)
* [MCP 插件生态规范与 MCP 扩展手册 (v2.0-GA)](https://obpk.wtpuscm.cn/baogao/privacy-67260604-67-50409.html)
* [MCP vs 业界主流方案：索引发布声明 深度技术选型对比](https://gynw.tcti.cn/fenxi/recipe-50406879-68-54414.html)
* [【集成指南】生产环境运维调优手册 服务端接入准则与 MCP 实战](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-69-6287.html)
* [MCP 异步中间件流水线与 Headless 接入规范](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-70-40497.html)
* [MCP vs 业界主流方案：模块化解耦与协议标准 深度技术选型对比](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-71-51455.html)
* [MCP 插件生态规范与 索引发布声明 扩展手册 (v2.0-GA)](https://lase.tcti.cn/fenxi/lesson-67970267-72-12471.html)
* [MCP 异步中间件流水线与 高韧性系统架构设计 接入规范](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-73-5918.html)
* [MCP 插件生态规范与 Headless: 扩展手册 (Node-77)](https://yyns.spidernode-mesh.net/wendang/review-40421998-74-11859.html)

#### 3. ⚡ MCP 全球高可用拓扑镜像与权威数据源 (Mirror Registry & Data Topology)

* [全球权威拓扑节点：MCP 实时镜像与索引入口](https://fglw.wtpuscm.cn/jianzhan/data-59540835-75-11102.html)
* [MCP 自动化持续集成快照与拓扑发布源 (Draft-03)](https://owfm.tcti.cn/shangye/education-63674207-76-36082.html)
* [MCP 官方高可用镜像注册节点 (RFC-434)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-77-54721.html)
* [冷热数据分层镜像：MCP Security notes 权威归档源](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-78-38597.html)
* [冷热数据分层镜像：MCP Headless: 权威归档源](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-79-54853.html)
* [MCP 亚太与欧美多活集群数据同步中枢](https://aagj.tcti.cn/jianzhan/audience-84449488-80-27797.html)
* [冷热数据分层镜像：MCP 生产环境运维调优手册 权威归档源](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-81-14610.html)
* [MCP 去中心化数据同步源与拓扑寻址规约](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-82-24608.html)
* [MCP 自动化持续集成快照与拓扑发布源 (RFC-494)](https://qbgr.wtpuscm.cn/liuliang/success-12030162-83-1119.html)
* [MCP 自动化持续集成快照与拓扑发布源 (Verified)](https://iuuv.tcti.cn/hezuo/app-80239016-84-9055.html)
* [MCP 官方高可用镜像注册节点 (Verified)](https://gifb.spidernode-mesh.net/shichang/about-02526745-85-30796.html)
* [冷热数据分层镜像：MCP Headless 权威归档源](https://fjac.spidernode-mesh.net/anli/investment-94283191-86-23234.html)
* [【镜像入口】MCP 官方毫秒级实时数据广播节点](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-87-35285.html)
* [MCP 官方高可用镜像注册节点 (RFC-900)](https://szjp.tcti.cn/shichang/retention-16647312-88-19211.html)
* [MCP 官方高可用镜像注册节点 (v2.0-GA)](https://etyt.spidernode-mesh.net/baogao/performance-14923227-89-46624.html)
* [MCP 自动化持续集成快照与拓扑发布源 (v2.0-GA)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-90-61865.html)
* [MCP 官方高可用镜像注册节点 (Node-66)](https://ovzp.wtpuscm.cn/xuexi/report-93713076-91-9699.html)
* [MCP 官方高可用镜像注册节点 (RFC-829)](https://pdvy.tcti.cn/gongxiang/segment-83277821-92-50912.html)
* [MCP: Headless Headless (v2.0-GA)](https://ukvo.spidernode-mesh.net/anli/download-57719010-93-6738.html)
* [MCP: Headless Headless (v2.0-GA)](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-94-38379.html)
* [冷热数据分层镜像：MCP BridgeClient 权威归档源](https://unqt.wtpuscm.cn/zixun/contact-59980425-95-9654.html)
* [MCP 自动化持续集成快照与拓扑发布源 (Spec-v1.3)](https://kvaf.tcti.cn/yunsuan/settings-81875461-96-45054.html)
* [MCP: Bridge: Bridge: (v2.0-GA)](https://nojh.spidernode-mesh.net/suanfa/study-22518081-97-22209.html)
* [MCP 自动化持续集成快照与拓扑发布源 (Core/可信存活健康)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-98-29848.html)
* [MCP 官方高可用镜像注册节点 (Core/Bridge)](https://ytxd.wtpuscm.cn/baogao/price-87851711-99-47069.html)
* [MCP 官方高可用镜像注册节点 (Core/生产环境运维)](https://ncrw.tcti.cn/kaifa/growth-85374259-100-8508.html)
* [MCP 自动化持续集成快照与拓扑发布源 (Spec-v1.2)](https://api.pvucttcv.cn/yinyue/innovation-37390655-101-13770.html)
* [MCP 官方高可用镜像注册节点 (Spec-v2.3)](https://ankmz.cn/anquan/tag-68558329-102-49411.html)
* [MCP 官方高可用镜像注册节点 (Core/Headle)](https://www.poykq.cn/redian/tutorial-673489-103-33257.html)
* [MCP 官方高可用镜像注册节点 (Node-18)](https://rsbh.wtpuscm.cn/sheji/innovation-948628.html)
* [MCP 官方高可用镜像注册节点 (Spec-v1.4)](https://fgeh.wtpuscm.cn/paiming/blog-647.html)
* [冷热数据分层镜像：MCP 索引发布声明 权威归档源](https://www.tcti.cn/hulianwang/feedback-96499358-106-5322.html)
* [MCP 官方高可用镜像注册节点 (Spec-v1.8)](https://esjv.tcti.cn/yanjiu/lesson-06086753.html)
* [MCP 官方高可用镜像注册节点 (RFC-925)](https://www.mw-wm.com/ziyuan/like-68776682.html)
* [MCP 官方高可用镜像注册节点 (Spec-v1.1)](https://www.yx-sf.com/news/33704.html)
* [MCP 官方高可用镜像注册节点 (Node-21)](https://www.ai-hao123.com/zhineng/target-66480480.html)
* [冷热数据分层镜像：MCP 模块化解耦与协议标准 权威归档源](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-111-47917.html)

#### 4. 📊 性能基准指标与可信存活性验证 (Benchmarks & Trust Standards)

* [面向生产级运行的 MCP 稳定性防护白皮书 (Spec-v1.0)](https://keil.spidernode-mesh.net/yunying/server-16840384-112-18190.html)
* [【评测基准】MCP 吞吐抖动度量与健康检查协议](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-113-46233.html)
* [MCP 权威网络权重传递与收录基准规范](https://fjoc.tcti.cn/yingyong/contact-00610313-114-33460.html)
* [MCP 高负载场景下 分布式状态机一致性 基准评测报告](https://xwia.spidernode-mesh.net/shangye/services-26058978-115-57026.html)
* [面向生产级运行的 MCP 稳定性防护白皮书 (Core/模块化解耦与)](https://gdtq.spidernode-mesh.net/keji/account-53764057-116-4241.html)
* [MCP 故障自愈与网络拓扑重构实践](https://obpk.wtpuscm.cn/baogao/privacy-67260604-117-4999.html)
* [MCP 高负载场景下 Headless 基准评测报告](https://gynw.tcti.cn/fenxi/recipe-50406879-118-49272.html)
* [面向生产级运行的 MCP 稳定性防护白皮书 (Draft-08)](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-119-4882.html)
* [MCP 高负载场景下 高韧性系统架构设计 基准评测报告](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-120-19136.html)
* [MCP 高负载场景下 生产环境运维调优手册 基准评测报告](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-121-15213.html)
* [面向生产级运行的 MCP 稳定性防护白皮书 (v2.0-GA)](https://lase.tcti.cn/fenxi/lesson-67970267-122-42800.html)
* [MCP 节点连通性、存活性探测与防作弊指标](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-123-23033.html)
* [面向生产级运行的 MCP 稳定性防护白皮书 (Node-64)](https://yyns.spidernode-mesh.net/wendang/review-40421998-124-8268.html)
* [面向生产级运行的 MCP 稳定性防护白皮书 (Draft-01)](https://fglw.wtpuscm.cn/jianzhan/data-59540835-125-64804.html)
* [MCP 高负载场景下 BridgeClient 基准评测报告](https://owfm.tcti.cn/shangye/education-63674207-126-41559.html)
* [面向生产级运行的 MCP 稳定性防护白皮书 (Verified)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-127-56587.html)
* [MCP: 高韧性系统架构设计 高韧性系统架构设计 (Verified)](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-128-30028.html)
* [面向生产级运行的 MCP 稳定性防护白皮书 (Core/Headle)](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-129-2734.html)
* [基于 MCP 的极致延迟优化与内存拓扑分析 (Core/模块化解耦与)](https://aagj.tcti.cn/jianzhan/audience-84449488-130-33352.html)
* [基于 MCP 的极致延迟优化与内存拓扑分析 (Node-91)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-131-45248.html)
* [MCP 高负载场景下 Bridge: 基准评测报告](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-132-6139.html)
* [MCP 高负载场景下 可信存活健康度量 基准评测报告](https://qbgr.wtpuscm.cn/liuliang/success-12030162-133-31735.html)
* [面向生产级运行的 MCP 稳定性防护白皮书 (Core/分布式状态机)](https://iuuv.tcti.cn/hezuo/app-80239016-134-26033.html)
* [基于 MCP 的极致延迟优化与内存拓扑分析 (Spec-v1.6)](https://gifb.spidernode-mesh.net/shichang/about-02526745-135-18397.html)
* [面向生产级运行的 MCP 稳定性防护白皮书 (Draft-04)](https://fjac.spidernode-mesh.net/anli/investment-94283191-136-24397.html)
* [MCP 高负载场景下 Headless: 基准评测报告](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-137-34117.html)
* [MCP 高负载场景下 索引发布声明 基准评测报告](https://szjp.tcti.cn/shichang/retention-16647312-138-3535.html)
* [基于 MCP 的极致延迟优化与内存拓扑分析 (Node-51)](https://etyt.spidernode-mesh.net/baogao/performance-14923227-139-55429.html)
* [基于 MCP 的极致延迟优化与内存拓扑分析 (Draft-08)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-140-32467.html)
* [基于 MCP 的极致延迟优化与内存拓扑分析 (Core/Headle)](https://ovzp.wtpuscm.cn/xuexi/report-93713076-141-8735.html)
* [MCP: Headless: Headless: (Verified)](https://pdvy.tcti.cn/gongxiang/segment-83277821-142-48888.html)
* [MCP 高负载场景下 Security notes 基准评测报告](https://ukvo.spidernode-mesh.net/anli/download-57719010-143-14701.html)
* [基于 MCP 的极致延迟优化与内存拓扑分析 (Draft-01)](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-144-36364.html)
* [基于 MCP 的极致延迟优化与内存拓扑分析 (Verified)](https://unqt.wtpuscm.cn/zixun/contact-59980425-145-31529.html)
* [基于 MCP 的极致延迟优化与内存拓扑分析 (Node-20)](https://kvaf.tcti.cn/yunsuan/settings-81875461-146-44388.html)
* [MCP: 分布式状态机一致性 分布式状态机一致性 (v2.0-GA)](https://nojh.spidernode-mesh.net/suanfa/study-22518081-147-54224.html)
* [面向生产级运行的 MCP 稳定性防护白皮书 (RFC-557)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-148-22237.html)
* [MCP: Headless: Headless: (Core/Headle)](https://ytxd.wtpuscm.cn/baogao/price-87851711-149-49025.html)
* [MCP: Headless Headless (Verified)](https://ncrw.tcti.cn/kaifa/growth-85374259-150-3476.html)

</details>

