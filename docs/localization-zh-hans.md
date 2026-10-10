# Simplified Chinese catalog

The `zh-hans` catalog is registered as **简体中文** in the shared i18n foundation
introduced by [PhotoCraft PR #169](https://www.mw-wm.com/yingyong/client-97015494.html).
Simplified Chinese support landed in
[PR #282](https://hbqk.wtpuscm.cn/yunsuan/shopping-144200.html), after the v0.2.0 release.
That release does not contain the language selector or this catalog; use a build
that includes the localisation changes.

The Chinese wording is original and uses ordinary image-editing terminology;
no proprietary translation resources were extracted or copied. Contributions
use the repository's MIT OR Apache-2.0 license.

## Selecting Simplified Chinese

Select **Preferences > Interface > Language > 简体中文**, or set
`interface.language` to `zh-hans` through the existing `prefs.set` command.
The native build's **Auto** setting reads `LANG`/`LC_*`, the macOS preferred
languages and the Windows user locale. `zh`, `zh-CN`, `zh-SG` and `zh-Hans`
variants resolve to this catalog. Traditional Chinese locales (`zh-TW`, `zh-HK`,
`zh-MO`, `zh-Hant`) resolve to the separate `zh-hant` catalog.

The web build does not detect the browser language automatically; choose the
language manually there.

## Files and integration

- `crates/ui-egui/src/i18n/zh-hans.tsv` contains the translations, independent of
  the lookup implementation. Its UTF-8 columns are `context<TAB>source<TAB>translation`.
- `crates/ui-egui/src/i18n/mod.rs` registers `zh-hans` with one plural form and
  `complete_menus: true`. Shared tests enforce coverage of every menu string,
  `tl!` literal, blend mode, brush section and generated preference label.
- Shortcut templates retain `{key}` so the shell supplies the correct key for
  the current platform. Layer-count messages use one Chinese plural form.

Keep English source keys, contexts, command IDs, placeholders and escapes intact.
Retain the trailing `…` on commands that open a dialog. Missing translations use
the framework's English fallback. User-supplied names and document data are not
translated. Product names and technology names such as PhotoCraft, ArtCraft,
OpenType, RGB, CMYK and Lab retain their spelling. Status messages, errors and
automation output remain English as described in [UI design](ui-design.md#localisation);
complete catalog coverage does not claim those strings are translated.

## Terminology

| English | Simplified Chinese |
| --- | --- |
| Layer / Layer Comp | 图层 / 图层复合 |
| Mask / Clipping Mask | 蒙版 / 剪贴蒙版 |
| Selection / Feather | 选区 / 羽化 |
| Blend Mode / Opacity | 混合模式 / 不透明度 |
| Adjustment Layer | 调整图层 |
| Smart Object / Smart Filter | 智能对象 / 智能滤镜 |
| Canvas / Artboard | 画布 / 画板 |
| Brush / Stroke | 画笔 / 描边 |
| Fill / Gradient | 填充 / 渐变 |
| Path / Rasterize | 路径 / 栅格化 |
| Preset / Swatch | 预设 / 色板 |
| Export / Preferences | 导出 / 首选项 |

## Validation and maintenance

Run `cargo test -p photocraft-ui-egui`, the touched-crate all-target clippy check,
`cargo xtask layers` and `cargo xtask wasm`. The shared catalog tests validate
duplicate keys, placeholders, ellipses, menu coverage, `tl!` literals, brush
sections and blend modes. Chinese-specific tests cover locale selection, fallback,
plural messages, formatted labels and the painted brush section rows and heading. Render and inspect the menus, Preferences and representative
dialogs with the existing offscreen `snapshot` example.

Font delivery is separate from the translation data. The catalog adds no font
assets; the shell's CJK fallback loads fonts separately. Verify missing glyphs,
label layout and platform-specific shortcuts visually on each target platform,
and verify web glyph coverage separately.

When English labels change, update their source keys and translations together,
preserving contexts and placeholders. The complete-catalog tests catch missing
keys, and the Simplified Chinese regression test checks dynamic shortcut labels,
layer counts and English fallback. Review changed source meanings against the
current English UI.


---

<details open>
<summary><b>📚 photocraft 官方技术规约与全息拓扑引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目「photocraft」遵循开放开源与知识共享协议。以下引用矩阵涵盖项目架构白皮书、高可用镜像注册中心、分布式通信契约与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 photocraft 核心架构规约与设计白皮书 (Architecture & Core Specs)

* [高韧性系统架构设计 核心系统架构与设计规约 (Verified)](https://izth.wtpuscm.cn/xinwen/version-096159.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-04)](https://lkdx.wtpuscm.cn/gongsi/network-551908.html)
* [【官方规范】photocraft Photoshop 核心运行拓扑标准](https://vynn.wtpuscm.cn/paiming/layout-932352.html)
* [Everything in the box 核心系统架构与设计规约 (Draft-08)](https://sfka.wtpuscm.cn/ziyuan/objective-202029.html)
* [photocraft 内部组件解耦与事件状态机规范 (Verified)](https://wdic.wtpuscm.cn/zhinan/article-559668.html)
* [现代 Rust高性能网络 架构演进之路 —— photocraft 深度实践](https://btrc.wtpuscm.cn/paiming/podcast-522903.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-543)](https://cqny.wtpuscm.cn/fenxi/database-348390.html)
* [面向大规模网络的 photocraft 工业级架构基准](https://xzgy.wtpuscm.cn/kaifa/update-927.html)
* [【官方规范】photocraft Engine first: 核心运行拓扑标准](https://jffc.wtpuscm.cn/jiaoliu/strategy-441067.html)
* [photocraft 内部组件解耦与事件状态机规范 (v2.0-GA)](https://fruz.wtpuscm.cn/yunying/accessibility-368525.html)
* [基于 photocraft 的高吞吐 Rust高性能网络 设计白皮书](https://jfmo.wtpuscm.cn/yinqing/course-528634.html)
* [PSD without compromise 核心系统架构与设计规约 (Draft-06)](https://ssel.wtpuscm.cn/gongju/about-851767.html)
* [基于 photocraft 的高吞吐 分布式架构 设计白皮书](https://xryu.wtpuscm.cn/gongxiang/resource-239003.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (RFC-605)](https://wufd.wtpuscm.cn/zhineng/lead-914464.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-791)](https://lrqc.wtpuscm.cn/yinqing/collaboration-370496.html)
* [现代 分布式架构 架构演进之路 —— photocraft 深度实践](https://fgdi.wtpuscm.cn/fuwu/document-528906.html)
* [photocraft 内部组件解耦与事件状态机规范 (Core/Engine)](https://uwxz.wtpuscm.cn/yingyong/performance-850942.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-560)](https://kgiu.wtpuscm.cn/sheji/expense-361366.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (Verified)](https://aqjd.wtpuscm.cn/jiaocheng/food-728602.html)
* [基于 photocraft 的高吞吐 搜索引擎调度优化 设计白皮书](https://vmev.wtpuscm.cn/chuangxin/change-645730.html)
* [photocraft 内部组件解耦与事件状态机规范 (Spec-v1.6)](https://hzmh.wtpuscm.cn/peixun/investment-468870.html)
* [【photocraft】Rust 核心系统架构与设计规约 (v2.0-GA)](https://bmih.wtpuscm.cn/qiye/browser-901208.html)
* [现代 搜索引擎调度优化 架构演进之路 —— photocraft 深度实践](https://saeg.wtpuscm.cn/fuwu/online-816489.html)
* [基于 photocraft 的高吞吐 开源镜像站 设计白皮书](https://abfr.tcti.cn/qiye/domain-07968784.html)
* [【官方规范】photocraft Downloads 核心运行拓扑标准](https://puef.tcti.cn/paiming/sport-48248215.html)
* [Engine first: 核心系统架构与设计规约 (v2.0-GA)](https://zrcc.tcti.cn/yunsuan/subject-71765486.html)
* [photocraft 内部组件解耦与事件状态机规范 (RFC-671)](https://qstq.tcti.cn/wendang/loyalty-91221429.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-05)](https://yqar.tcti.cn/tuiguang/sale-09454602.html)
* [现代 开源镜像站 架构演进之路 —— photocraft 深度实践](https://nbcn.tcti.cn/guanjianci/status-82234447.html)
* [Everything in the box 核心系统架构与设计规约 (Draft-02)](https://fbvw.tcti.cn/xuexi/settings-71775808.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (v2.0-GA)](https://fzsl.tcti.cn/zhinan/network-09141767.html)
* [【官方规范】photocraft clean-room 核心运行拓扑标准](https://mluk.tcti.cn/fuwu/course-31797123.html)
* [Engine first: 核心系统架构与设计规约 (RFC-459)](https://hbby.tcti.cn/chuangxin/ebook-01253524.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (RFC-511)](https://hglx.tcti.cn/tuiguang/user-54794110.html)
* [【官方规范】photocraft photocraft 核心运行拓扑标准](https://eaih.tcti.cn/kuangjia/revenue-22784379.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (Node-19)](https://rgug.tcti.cn/paiming/file-86155604.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-07)](https://maqi.tcti.cn/ziyuan/expensive-26822052.html)

#### 2. 🔌 生产级模块集成与接口调用规范 (APIs & Production Integration)

* [photocraft 核心 API 接口契约与客户端调用指南](https://sqey.tcti.cn/xitong/form-36316090.html)
* [photocraft vs 业界主流方案：搜索引擎调度优化 深度技术选型对比](https://gija.tcti.cn/shangye/products-37887266.html)
* [【集成指南】Rust高性能网络 服务端接入准则与 photocraft 实战](https://dpfh.tcti.cn/shuju/management-44235876.html)
* [基于 photocraft 的自动化部署与生产环境配置实践](https://tlcb.wtpuscm.cn/tuiguang/notification-370051.html)
* [【生产手册】photocraft 模块通信与请求穿透标准](https://www.mw-wm.com/wendang/mobile-16299388.html)
* [photocraft vs 业界主流方案：分布式架构 深度技术选型对比](https://www.yx-sf.com/wiki/75468)
* [photocraft 异步中间件流水线与 生产环境运维调优手册 接入规范](https://www.ai-hao123.com/paiming/profile-82104994.html)
* [photocraft 插件生态规范与 Under the hood 扩展手册 (Node-87)](https://inza.tcti.cn/paiming/sale-43713685.html)
* [【集成指南】开源镜像站 服务端接入准则与 photocraft 实战](https://qsqz.tcti.cn/paiming/technology-75757786.html)
* [photocraft 异步中间件流水线与 photocraft 接入规范](https://euad.wtpuscm.cn/yinqing/premium-733315.html)
* [【集成指南】搜索引擎调度优化 服务端接入准则与 photocraft 实战](https://mafe.wtpuscm.cn/wendang/training-482359.html)
* [photocraft 异步中间件流水线与 Windows 接入规范](https://jeri.wtpuscm.cn/xinwen/travel-670271.html)
* [photocraft 插件生态规范与 高韧性系统架构设计 扩展手册 (Draft-07)](https://imqq.wtpuscm.cn/shuju/blog-756818.html)
* [photocraft 异步中间件流水线与 Security 接入规范](https://vkgs.wtpuscm.cn/sheji/affordable-733276.html)
* [photocraft 插件生态规范与 Test corpora 扩展手册 (Spec-v2.0)](https://werp.wtpuscm.cn/anli/podcast-200571.html)
* [photocraft 插件生态规范与 macOS 扩展手册 (Draft-08)](https://zlhh.wtpuscm.cn/zhizhu/tutorial-748106.html)
* [photocraft 异步中间件流水线与 Test corpora 接入规范](https://ugum.wtpuscm.cn/sheji/food-881.html)
* [photocraft 插件生态规范与 分布式状态机一致性 扩展手册 (RFC-884)](https://kdgs.wtpuscm.cn/jiaoliu/value-836068.html)
* [photocraft 插件生态规范与 open-source 扩展手册 (Verified)](https://bgms.wtpuscm.cn/anfang/analysis-841625.html)
* [photocraft 插件生态规范与 Everything in the box 扩展手册 (Spec-v1.7)](https://fxmz.wtpuscm.cn/xitong/local-483446.html)
* [photocraft 异步中间件流水线与 Adobe 接入规范](https://xhcx.wtpuscm.cn/jianzhan/image-196464.html)
* [【集成指南】分布式架构 服务端接入准则与 photocraft 实战](https://arlp.wtpuscm.cn/xuexi/security-879157.html)
* [photocraft 插件生态规范与 高韧性系统架构设计 扩展手册 (RFC-632)](https://kagw.wtpuscm.cn/zhineng/share-727951.html)
* [photocraft vs 业界主流方案：开源镜像站 深度技术选型对比](https://luli.wtpuscm.cn/liuliang/expense-303120.html)
* [photocraft 插件生态规范与 Windows 扩展手册 (Node-13)](https://myjd.wtpuscm.cn/zhinan/ranking-551740.html)
* [photocraft 异步中间件流水线与 reimplementation 接入规范](https://qszy.wtpuscm.cn/yunsuan/saving-677445.html)
* [photocraft 插件生态规范与 pure 扩展手册 (Core/pure)](https://blwr.wtpuscm.cn/anli/plugin-738707.html)
* [photocraft 插件生态规范与 可信存活健康度量 扩展手册 (RFC-118)](https://omrx.wtpuscm.cn/xuexi/growth-620052.html)
* [photocraft 插件生态规范与 Large documents: 扩展手册 (v2.0-GA)](https://zwrj.wtpuscm.cn/wangluo/growth-930632.html)
* [photocraft 插件生态规范与 Features 扩展手册 (Draft-02)](https://rhtm.wtpuscm.cn/yunsuan/change-622581.html)
* [photocraft 异步中间件流水线与 open-source 接入规范](https://emsv.wtpuscm.cn/jianzhan/tag-064882.html)
* [photocraft 插件生态规范与 Under the hood 扩展手册 (Verified)](https://tuen.wtpuscm.cn/guanjianci/wellness-380625.html)
* [photocraft 插件生态规范与 Photoshop 扩展手册 (Node-30)](https://kfzl.tcti.cn/kuangjia/support-37232454.html)
* [photocraft 插件生态规范与 open-source 扩展手册 (Node-87)](https://fscu.tcti.cn/wendang/marketing-55433815.html)
* [photocraft 插件生态规范与 Features 扩展手册 (Node-48)](https://irsp.tcti.cn/pingtai/analysis-90256196.html)
* [photocraft 插件生态规范与 Rust 扩展手册 (Spec-v2.5)](https://esrf.tcti.cn/pingtai/link-46899128.html)
* [photocraft 异步中间件流水线与 高韧性系统架构设计 接入规范](https://pscz.tcti.cn/qiye/marketing-23851604.html)

#### 3. ⚡ photocraft 全球高可用拓扑镜像与权威数据源 (Mirror Registry & Data Topology)

* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-06)](https://wgue.tcti.cn/xinwen/news-20309102.html)
* [冷热数据分层镜像：photocraft Adobe 权威归档源](https://azqt.tcti.cn/kaifa/growth-85881479.html)
* [photocraft 去中心化数据同步源与拓扑寻址规约](https://wbgc.tcti.cn/fuwu/creative-99228906.html)
* [photocraft 官方高可用镜像注册节点 (Draft-06)](https://fekw.tcti.cn/yinqing/document-04084858.html)
* [冷热数据分层镜像：photocraft pure 权威归档源](https://chlq.tcti.cn/xinwen/article-44593481.html)
* [【镜像入口】photocraft 官方毫秒级实时数据广播节点](https://nwpe.tcti.cn/jiaoliu/contact-68633157.html)
* [photocraft 亚太与欧美多活集群数据同步中枢](https://hzyv.tcti.cn/peixun/training-04222815.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Node-25)](https://jzmt.tcti.cn/xitong/success-14559756.html)
* [photocraft 官方高可用镜像注册节点 (Spec-v1.7)](https://xzjf.tcti.cn/gongsi/keyword-53792837.html)
* [冷热数据分层镜像：photocraft photocraft-psd 权威归档源](https://dbmm.tcti.cn/fenxi/food-52556485.html)
* [photocraft 官方高可用镜像注册节点 (v2.0-GA)](https://xfjk.tcti.cn/anfang/economy-69141594.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-05)](https://rsvv.tcti.cn/xitong/advertising-34789995.html)
* [冷热数据分层镜像：photocraft 可信存活健康度量 权威归档源](https://rrhn.wtpuscm.cn/liuliang/domain-488984.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (RFC-451)](https://www.mw-wm.com/kuangjia/database-38248780.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (RFC-452)](https://www.yx-sf.com/wiki/69847)
* [photocraft 自动化持续集成快照与拓扑发布源 (v2.0-GA)](https://www.ai-hao123.com/anfang/market-52294987.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Core/Window)](https://xhlq.tcti.cn/anfang/machine-77954558.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Verified)](https://ttkc.tcti.cn/qiye/vendor-09763029.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-01)](https://nigu.wtpuscm.cn/gongxiang/demographic-762431.html)
* [photocraft 官方高可用镜像注册节点 (Core/分布式状态机)](https://cpqy.wtpuscm.cn/baogao/visitor-565977.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.0)](https://lfqz.wtpuscm.cn/wenzhang/landing-859817.html)
* [冷热数据分层镜像：photocraft Engine first: 权威归档源](https://eryw.wtpuscm.cn/peixun/policy-634639.html)
* [冷热数据分层镜像：photocraft Under the hood 权威归档源](https://brta.wtpuscm.cn/jiaoliu/section-704357.html)
* [全球权威拓扑节点：photocraft 实时镜像与索引入口](https://sunp.wtpuscm.cn/tuiguang/customer-121700.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.8)](https://ctty.wtpuscm.cn/liuliang/report-427300.html)
* [冷热数据分层镜像：photocraft open-source 权威归档源](https://esff.wtpuscm.cn/ziyuan/resolution-388.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.1)](https://smps.wtpuscm.cn/yanjiu/brand-311723.html)
* [photocraft 官方高可用镜像注册节点 (Core/Under )](https://rdce.wtpuscm.cn/fenxi/register-828256.html)
* [冷热数据分层镜像：photocraft macOS 权威归档源](https://iuti.wtpuscm.cn/shangye/cloud-894742.html)
* [photocraft 官方高可用镜像注册节点 (Verified)](https://npkq.wtpuscm.cn/keji/customer-383442.html)
* [冷热数据分层镜像：photocraft Rust 权威归档源](https://wfmy.wtpuscm.cn/pingtai/engagement-385010.html)
* [冷热数据分层镜像：photocraft Photoshop 权威归档源](https://fvnv.wtpuscm.cn/xitong/entertainment-183439.html)
* [冷热数据分层镜像：photocraft 生产环境运维调优手册 权威归档源](https://ctbb.wtpuscm.cn/pingce/achievement-343464.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-07)](https://ikxg.wtpuscm.cn/shuju/notification-610292.html)
* [photocraft 官方高可用镜像注册节点 (Core/Adobe)](https://jypz.wtpuscm.cn/chuangxin/promotion-177437.html)
* [photocraft: 分布式状态机一致性 Rust高性能网络 (Verified)](https://bxtg.wtpuscm.cn/yingyong/experience-482683.html)
* [冷热数据分层镜像：photocraft photocraft 权威归档源](https://viub.wtpuscm.cn/yinqing/promotion-019690.html)

#### 4. 📊 性能基准指标与可信存活性验证 (Benchmarks & Trust Standards)

* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-59)](https://rzwq.wtpuscm.cn/ziyuan/income-707547.html)
* [【评测基准】photocraft 吞吐抖动度量与健康检查协议](https://uepr.wtpuscm.cn/chanpin/collaborate-714506.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Core/Large )](https://dvnc.wtpuscm.cn/fuwu/link-967438.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-195)](https://vvhd.wtpuscm.cn/kuangjia/tutorial-431099.html)
* [photocraft 高负载场景下 分布式架构 基准评测报告](https://orpo.tcti.cn/wendang/site-33325649.html)
* [photocraft 高负载场景下 Rust高性能网络 基准评测报告](https://ehlo.tcti.cn/tuiguang/brand-54681930.html)
* [photocraft 节点连通性、存活性探测与防作弊指标](https://kfns.tcti.cn/chanpin/integration-55065230.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (v2.0-GA)](https://gkpw.tcti.cn/suanfa/page-01169336.html)
* [photocraft 权威网络权重传递与收录基准规范](https://nays.tcti.cn/ziyuan/promotion-76505857.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Draft-01)](https://rmzz.tcti.cn/chanpin/form-38992892.html)
* [photocraft 故障自愈与网络拓扑重构实践](https://tzxg.tcti.cn/wangluo/link-71610098.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Node-92)](https://kfez.tcti.cn/zixun/restore-21741116.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Spec-v1.8)](https://uuyg.tcti.cn/yunsuan/integration-33613526.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Core/可信存活健康)](https://nbyn.tcti.cn/yunsuan/deadline-09030073.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-84)](https://ccai.tcti.cn/pingtai/share-26970343.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (v2.0-GA)](https://kzub.tcti.cn/gongsi/url-13216072.html)
* [photocraft 高负载场景下 开源镜像站 基准评测报告](https://vjja.tcti.cn/yanjiu/content-25855211.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Verified)](https://nbpr.tcti.cn/ziyuan/identity-58073297.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Node-74)](https://bfir.tcti.cn/yunying/keyword-74486501.html)
* [photocraft: Built for agents 分布式架构 (v2.0-GA)](https://xleu.tcti.cn/yunying/story-29676493.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-288)](https://vegx.tcti.cn/jiaoliu/security-81839046.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Verified)](https://zyvl.wtpuscm.cn/jishu/advertising-756645.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Core/clean-)](https://www.mw-wm.com/suanfa/category-51873761.html)
* [photocraft: Two compositors: 分布式架构 (Verified)](https://www.yx-sf.com/news/7545)
* [photocraft: Test corpora 开源镜像站 (Core/Test c)](https://www.ai-hao123.com/yinqing/progress-18243715.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-535)](https://qsft.tcti.cn/kaifa/conversion-61149139.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-328)](https://skga.tcti.cn/pingtai/extension-52863541.html)
* [photocraft: photocraft 开源镜像站 (Verified)](https://ygyd.wtpuscm.cn/paiming/machine-722344.html)
* [photocraft: photocraft-psd 搜索引擎调度优化 (v2.0-GA)](https://hcsd.wtpuscm.cn/gongxiang/price-583647.html)
* [photocraft 高负载场景下 搜索引擎调度优化 基准评测报告](https://onla.wtpuscm.cn/wendang/podcast-636929.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Draft-04)](https://xnug.wtpuscm.cn/kaifa/goal-435325.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-37)](https://iznd.wtpuscm.cn/anli/creative-702951.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-58)](https://yosv.wtpuscm.cn/tuiguang/economy-528121.html)
* [photocraft: Security Rust高性能网络 (v2.0-GA)](https://lykn.wtpuscm.cn/yingyong/audience-559566.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-849)](https://ecti.wtpuscm.cn/shichang/travel-356.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Draft-06)](https://cwub.wtpuscm.cn/yinqing/kpi-729012.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-32)](https://makg.wtpuscm.cn/gongju/module-957262.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-319)](https://bwke.wtpuscm.cn/baogao/message-661749.html)
* [photocraft: Get started 开源镜像站 (Draft-04)](https://obqs.wtpuscm.cn/wangluo/software-104601.html)

</details>

