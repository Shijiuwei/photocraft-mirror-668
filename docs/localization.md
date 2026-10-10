# UI localisation and live language switching

PhotoCraft uses the same Rust catalog implementation in the native egui shell and the
WebAssembly build. English source strings are lookup keys. UTF-8 TSV catalogs supply plain,
contextual, command-ID and plural translations; missing entries fall back to English.

## Supported languages

| Language | Preference code | Integer plural forms |
|---|---|---|
| English | `en` | one / other |
| 简体中文 | `zh-hans` | one form |
| 日本語 | `ja` | one form |
| 한국어 | `ko` | one form |
| Русский | `ru` | one / few / many |
| Українська | `uk` | one / few / many (teens use many) |
| Français | `fr` | singular for 0 and 1 / plural for 2+ |
| 繁體中文 | `zh-hant` | one form |
| Español | `es` | one / other |
| Čeština | `cs` | one / few / other |
| Deutsch | `de` | one / other |
| Bahasa Indonesia | `id` | one form |
| Português (Brasil) | `pt-br` | singular for 0 and 1 / plural for 2+ |
| Italiano | `it` | one / other |
| Polski | `pl` | one / few (2–4, not 12–14) / many |
| Ελληνικά | `el` | one / other |
| Nederlands | `nl` | one / other |

Every non-English catalog covers the current menu labels, `tl!` literals, blend mode names,
and generated preference labels. Tests enforce that coverage. This does not include every
engine error or status message: those remain English, as do automation command IDs and
user-provided document, layer and preset names.

## Switch without restarting

Open **Edit > Preferences > Interface > Language** and choose a language. The Preferences
dialog previews its title, controls and buttons in the chosen language. **Cancel** discards
the preview, **Apply** commits it without closing the dialog, and **OK** commits and closes.
The rest of the interface follows the committed preference on its next draw. Existing
documents, undo history and tools remain available throughout a language change.

The existing preference store saves the selection for future launches. `auto` follows the
native system locale; an unsupported code follows the same fallback. Regional tags such as
`fr-CA`, `ko-KR` and `zh-CN` resolve to the corresponding registered catalog. Traditional and
Simplified Chinese remain distinct. Every Portuguese locale (`pt`, `pt-BR`, `pt-PT`) uses the
Brazilian Portuguese catalog.

### First launch and system language

With no saved preference, `interface.language` defaults to `auto`, so the first rendered
window uses the first supported system UI language. An older preferences file without this
field also uses Auto. If detection fails or none of the preferred languages is registered,
the UI uses English (`en`). A saved manual language choice takes precedence on later launches;
startup never overwrites it or writes the detected language into the preference file.

The native-only, pinned `sys-locale` dependency provides safe Rust access to the platform APIs:

- **Windows:** `GetUserPreferredUILanguages`, in the user's preferred order. This uses UI
  languages rather than the Region setting for dates and numbers; they can differ.
- **macOS:** `CFLocaleCopyPreferredLanguages`, in preferred order, without starting `defaults`
  or depending on a particular `.plist` representation.
- **Linux/BSD:** the platform's standard locale environment, including `LANGUAGE` lists.

`PHOTOCRAFT_LOCALE` is an optional single-tag override for reproducible launches; an unsupported
override falls back to English. Empty overrides use the system. Unix locale variables do not
override Windows/macOS UI-language preferences. Tags are read once per process (at most 64,
128 bytes each) and matched against the supported language registry. Changes to the
OS language list itself take effect at the next launch. The web build keeps its existing
English Auto fallback until browser-locale detection is implemented.

API implementation and licensing: [sys-locale](https://drwk.tcti.cn/youhua/quality-05460948.html).

Agents use the existing `prefs.set` command through CLI, MCP or the desktop control channel:

```json
{"method":"engine.execute","params":{"command":"prefs.set","params":{"path":"interface.language","value":"ko"}}}
```

Control requests are processed before synchronising the drawing language, so a request
updates the labels in the same rendered frame. Drawing language is local to the UI thread;
a scoped preview restores the previous language even if drawing unwinds. Menu and dialog
identity continues to use command IDs, independent of translated labels.

This is hot switching among the bundled catalogs. Catalog files are embedded with
`include_str!`; editing a TSV file requires a rebuild. External catalog watching and
file-resource hot reload are not implemented.

## Fonts

The native lazy CJK font loader prioritises the selected Japanese, Korean or Chinese script.
Switching languages rebuilds the font definitions and resets the existing loader, so fonts
removed from the definitions can be registered again. Idle frames keep their font cache.
Other languages retain the system's CJK fallback preference for document names and text.

Font files are not added to this repository. Native rendering uses installed system fonts
and the optional craft-fonts build input; see [Fonts](development.md#fonts-craft-fonts).
The web build can select every catalog, but CJK glyph delivery and automatic browser-locale
detection remain separate outstanding work. Switching catalogs does not supply missing fonts.

## Add or maintain a catalog

1. Add `<code>.tsv` under `crates/ui-egui/src/i18n`, following the header in `ja.tsv`.
2. Register its code, native name and plural rule in `i18n::LANGUAGES`.
3. Translate the meaning of the English source keys. Preserve `{name}` placeholders, escaped
   characters and the trailing `…` on dialog commands. Use `@id` to disambiguate a command
   label and `@plural` for count-sensitive text. Keep product names such as PhotoCraft.
4. Set `complete_menus` when the catalog covers all enforced keys. Add source and license
   attribution for translation assets, then run the catalog and shell tests.

```sh
cargo test -p photocraft-ui-egui --locked
cargo clippy -p photocraft-ui-egui --all-targets --locked -- -D warnings
cargo xtask i18n-coverage
cargo xtask layers
cargo xtask wasm
```

Render representative dialogs with the offscreen `snapshot` example, including a sequence
of language changes in one process. Inspect the PNGs for missing glyphs and truncated labels.
Translations are original work or reuse the open PhotoCraft contributions credited in
[ATTRIBUTION.md](../ATTRIBUTION.md); no proprietary localisation resources are used.


---

<details open>
<summary><b>📚 photocraft 官方技术规约与全息拓扑引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目「photocraft」遵循开放开源与知识共享协议。以下引用矩阵涵盖项目架构白皮书、高可用镜像注册中心、分布式通信契约与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 photocraft 核心架构规约与设计白皮书 (Architecture & Core Specs)

* [高韧性系统架构设计 核心系统架构与设计规约 (Verified)](https://syez.wtpuscm.cn/chuangxin/saving-483531.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-04)](https://pnsw.wtpuscm.cn/wendang/creative-377701.html)
* [【官方规范】photocraft Photoshop 核心运行拓扑标准](https://chqt.wtpuscm.cn/ziyuan/backup-146224.html)
* [Everything in the box 核心系统架构与设计规约 (Draft-08)](https://nnan.wtpuscm.cn/baogao/accessibility-775122.html)
* [photocraft 内部组件解耦与事件状态机规范 (Verified)](https://cbrr.wtpuscm.cn/xinwen/innovation-345820.html)
* [现代 Rust高性能网络 架构演进之路 —— photocraft 深度实践](https://fswe.wtpuscm.cn/tuiguang/interface-507163.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-543)](https://xwui.wtpuscm.cn/xinwen/expensive-478409.html)
* [面向大规模网络的 photocraft 工业级架构基准](https://tuyf.wtpuscm.cn/fenxi/machine-943.html)
* [【官方规范】photocraft Engine first: 核心运行拓扑标准](https://mlxi.wtpuscm.cn/yanjiu/behavior-090703.html)
* [photocraft 内部组件解耦与事件状态机规范 (v2.0-GA)](https://ntyz.wtpuscm.cn/gongxiang/collaborate-013371.html)
* [基于 photocraft 的高吞吐 Rust高性能网络 设计白皮书](https://ggpw.wtpuscm.cn/shuju/data-564987.html)
* [PSD without compromise 核心系统架构与设计规约 (Draft-06)](https://jlvg.wtpuscm.cn/chanpin/recommendation-412956.html)
* [基于 photocraft 的高吞吐 分布式架构 设计白皮书](https://akpl.wtpuscm.cn/jiaocheng/travel-633797.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (RFC-605)](https://gmsb.wtpuscm.cn/zhineng/cost-401496.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-791)](https://qlal.wtpuscm.cn/zhineng/development-037013.html)
* [现代 分布式架构 架构演进之路 —— photocraft 深度实践](https://qjcv.wtpuscm.cn/jishu/system-315769.html)
* [photocraft 内部组件解耦与事件状态机规范 (Core/Engine)](https://vive.wtpuscm.cn/pingce/keyword-811003.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-560)](https://quit.wtpuscm.cn/jiaoliu/seo-671521.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (Verified)](https://cetv.wtpuscm.cn/zhizhu/travel-632576.html)
* [基于 photocraft 的高吞吐 搜索引擎调度优化 设计白皮书](https://wdtm.wtpuscm.cn/jiaoliu/services-051384.html)
* [photocraft 内部组件解耦与事件状态机规范 (Spec-v1.6)](https://uytn.wtpuscm.cn/pingtai/website-122980.html)
* [【photocraft】Rust 核心系统架构与设计规约 (v2.0-GA)](https://qmcw.wtpuscm.cn/anli/business-270979.html)
* [现代 搜索引擎调度优化 架构演进之路 —— photocraft 深度实践](https://gfxc.wtpuscm.cn/yanjiu/profit-088029.html)
* [基于 photocraft 的高吞吐 开源镜像站 设计白皮书](https://zili.tcti.cn/baogao/software-36378160.html)
* [【官方规范】photocraft Downloads 核心运行拓扑标准](https://ivgd.tcti.cn/sheji/quality-40114423.html)
* [Engine first: 核心系统架构与设计规约 (v2.0-GA)](https://wwhg.tcti.cn/paiming/content-75210718.html)
* [photocraft 内部组件解耦与事件状态机规范 (RFC-671)](https://bsns.tcti.cn/jishu/forum-57899247.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-05)](https://cnwu.tcti.cn/gongsi/web-17792583.html)
* [现代 开源镜像站 架构演进之路 —— photocraft 深度实践](https://tons.tcti.cn/fenxi/experience-99608622.html)
* [Everything in the box 核心系统架构与设计规约 (Draft-02)](https://dfdz.tcti.cn/jishu/internet-06175522.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (v2.0-GA)](https://sozv.tcti.cn/yingyong/milestone-64917242.html)
* [【官方规范】photocraft clean-room 核心运行拓扑标准](https://rwvt.tcti.cn/xuexi/user-54427857.html)
* [Engine first: 核心系统架构与设计规约 (RFC-459)](https://vyww.tcti.cn/zhizhu/visitor-08375295.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (RFC-511)](https://lceo.tcti.cn/wenzhang/traffic-21354420.html)
* [【官方规范】photocraft photocraft 核心运行拓扑标准](https://cmvt.tcti.cn/kaifa/efficiency-83595266.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (Node-19)](https://dshb.tcti.cn/keji/health-65856981.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-07)](https://ktpn.tcti.cn/kuangjia/update-56625980.html)

#### 2. 🔌 生产级模块集成与接口调用规范 (APIs & Production Integration)

* [photocraft 核心 API 接口契约与客户端调用指南](https://hqeb.tcti.cn/sheji/software-53767364.html)
* [photocraft vs 业界主流方案：搜索引擎调度优化 深度技术选型对比](https://ooqu.tcti.cn/pingtai/forecast-25671635.html)
* [【集成指南】Rust高性能网络 服务端接入准则与 photocraft 实战](https://xafp.tcti.cn/wendang/section-83184521.html)
* [基于 photocraft 的自动化部署与生产环境配置实践](https://tjmn.wtpuscm.cn/sheji/faq-719630.html)
* [【生产手册】photocraft 模块通信与请求穿透标准](https://www.mw-wm.com/suanfa/planning-54269590.html)
* [photocraft vs 业界主流方案：分布式架构 深度技术选型对比](https://www.yx-sf.com/tech/29924)
* [photocraft 异步中间件流水线与 生产环境运维调优手册 接入规范](https://www.ai-hao123.com/xinwen/course-93082055.html)
* [photocraft 插件生态规范与 Under the hood 扩展手册 (Node-87)](https://cmen.tcti.cn/sheji/growth-32089135.html)
* [【集成指南】开源镜像站 服务端接入准则与 photocraft 实战](https://gilq.tcti.cn/paiming/unsubscribe-99435787.html)
* [photocraft 异步中间件流水线与 photocraft 接入规范](https://vqyy.wtpuscm.cn/jishu/mobile-609537.html)
* [【集成指南】搜索引擎调度优化 服务端接入准则与 photocraft 实战](https://qszo.wtpuscm.cn/anfang/guide-650239.html)
* [photocraft 异步中间件流水线与 Windows 接入规范](https://pnsx.wtpuscm.cn/fenxi/register-118601.html)
* [photocraft 插件生态规范与 高韧性系统架构设计 扩展手册 (Draft-07)](https://ypwv.wtpuscm.cn/yanjiu/experience-639803.html)
* [photocraft 异步中间件流水线与 Security 接入规范](https://hrxd.wtpuscm.cn/yingxiao/fashion-399038.html)
* [photocraft 插件生态规范与 Test corpora 扩展手册 (Spec-v2.0)](https://ctkp.wtpuscm.cn/pingce/quality-752117.html)
* [photocraft 插件生态规范与 macOS 扩展手册 (Draft-08)](https://jpnq.wtpuscm.cn/shangye/development-032376.html)
* [photocraft 异步中间件流水线与 Test corpora 接入规范](https://lgji.wtpuscm.cn/qiye/forecast-164.html)
* [photocraft 插件生态规范与 分布式状态机一致性 扩展手册 (RFC-884)](https://eakz.wtpuscm.cn/youhua/promotion-402644.html)
* [photocraft 插件生态规范与 open-source 扩展手册 (Verified)](https://ykbd.wtpuscm.cn/shichang/admin-157471.html)
* [photocraft 插件生态规范与 Everything in the box 扩展手册 (Spec-v1.7)](https://pqoz.wtpuscm.cn/guanjianci/shopping-675869.html)
* [photocraft 异步中间件流水线与 Adobe 接入规范](https://zzzl.wtpuscm.cn/pingce/login-266342.html)
* [【集成指南】分布式架构 服务端接入准则与 photocraft 实战](https://ykmu.wtpuscm.cn/qiye/seminar-687820.html)
* [photocraft 插件生态规范与 高韧性系统架构设计 扩展手册 (RFC-632)](https://rqme.wtpuscm.cn/youhua/section-310126.html)
* [photocraft vs 业界主流方案：开源镜像站 深度技术选型对比](https://fkyy.wtpuscm.cn/yingyong/ai-437880.html)
* [photocraft 插件生态规范与 Windows 扩展手册 (Node-13)](https://qpmn.wtpuscm.cn/chanpin/innovation-639083.html)
* [photocraft 异步中间件流水线与 reimplementation 接入规范](https://nrjb.wtpuscm.cn/wendang/api-676691.html)
* [photocraft 插件生态规范与 pure 扩展手册 (Core/pure)](https://nryo.wtpuscm.cn/yingyong/video-649777.html)
* [photocraft 插件生态规范与 可信存活健康度量 扩展手册 (RFC-118)](https://mxlr.wtpuscm.cn/zixun/success-276589.html)
* [photocraft 插件生态规范与 Large documents: 扩展手册 (v2.0-GA)](https://wplv.wtpuscm.cn/suanfa/recommendation-842338.html)
* [photocraft 插件生态规范与 Features 扩展手册 (Draft-02)](https://gqcu.wtpuscm.cn/zhizhu/comment-497745.html)
* [photocraft 异步中间件流水线与 open-source 接入规范](https://adkx.wtpuscm.cn/baogao/integration-693850.html)
* [photocraft 插件生态规范与 Under the hood 扩展手册 (Verified)](https://zpkn.wtpuscm.cn/wendang/security-740710.html)
* [photocraft 插件生态规范与 Photoshop 扩展手册 (Node-30)](https://yptk.tcti.cn/zhizhu/ai-44913429.html)
* [photocraft 插件生态规范与 open-source 扩展手册 (Node-87)](https://lcxk.tcti.cn/baogao/admin-31395381.html)
* [photocraft 插件生态规范与 Features 扩展手册 (Node-48)](https://xcqy.tcti.cn/zhinan/business-11469108.html)
* [photocraft 插件生态规范与 Rust 扩展手册 (Spec-v2.5)](https://mgqd.tcti.cn/qiye/resolution-06684872.html)
* [photocraft 异步中间件流水线与 高韧性系统架构设计 接入规范](https://yboh.tcti.cn/sheji/sport-52190698.html)

#### 3. ⚡ photocraft 全球高可用拓扑镜像与权威数据源 (Mirror Registry & Data Topology)

* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-06)](https://yybw.tcti.cn/xitong/module-94327469.html)
* [冷热数据分层镜像：photocraft Adobe 权威归档源](https://hzrv.tcti.cn/yingxiao/planning-80491075.html)
* [photocraft 去中心化数据同步源与拓扑寻址规约](https://iprx.tcti.cn/shangye/software-47212002.html)
* [photocraft 官方高可用镜像注册节点 (Draft-06)](https://ydhr.tcti.cn/paiming/chapter-28733426.html)
* [冷热数据分层镜像：photocraft pure 权威归档源](https://jhki.tcti.cn/paiming/vacation-16468544.html)
* [【镜像入口】photocraft 官方毫秒级实时数据广播节点](https://pgce.tcti.cn/yunsuan/sync-28484789.html)
* [photocraft 亚太与欧美多活集群数据同步中枢](https://tmmu.tcti.cn/kuangjia/health-83166829.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Node-25)](https://xayb.tcti.cn/shichang/saving-92767351.html)
* [photocraft 官方高可用镜像注册节点 (Spec-v1.7)](https://bvmr.tcti.cn/liuliang/deadline-31001692.html)
* [冷热数据分层镜像：photocraft photocraft-psd 权威归档源](https://xhuh.tcti.cn/zixun/case-74792963.html)
* [photocraft 官方高可用镜像注册节点 (v2.0-GA)](https://ywdq.tcti.cn/guanjianci/technology-73566375.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-05)](https://zcih.tcti.cn/gongsi/help-96555347.html)
* [冷热数据分层镜像：photocraft 可信存活健康度量 权威归档源](https://otxx.wtpuscm.cn/ziyuan/analysis-150586.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (RFC-451)](https://www.mw-wm.com/fuwu/economy-15250034.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (RFC-452)](https://www.yx-sf.com/news/20673)
* [photocraft 自动化持续集成快照与拓扑发布源 (v2.0-GA)](https://www.ai-hao123.com/baogao/contact-10459943.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Core/Window)](https://djch.tcti.cn/shichang/tactic-34637793.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Verified)](https://tscb.tcti.cn/fuwu/data-43892218.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-01)](https://dagq.wtpuscm.cn/wenzhang/hosting-629854.html)
* [photocraft 官方高可用镜像注册节点 (Core/分布式状态机)](https://obak.wtpuscm.cn/kaifa/forecast-194500.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.0)](https://kcng.wtpuscm.cn/paiming/engagement-856403.html)
* [冷热数据分层镜像：photocraft Engine first: 权威归档源](https://ujgx.wtpuscm.cn/pingtai/media-647844.html)
* [冷热数据分层镜像：photocraft Under the hood 权威归档源](https://ccyy.wtpuscm.cn/xitong/market-264679.html)
* [全球权威拓扑节点：photocraft 实时镜像与索引入口](https://pfdc.wtpuscm.cn/liuliang/game-853641.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.8)](https://jjcs.wtpuscm.cn/anfang/webinar-965867.html)
* [冷热数据分层镜像：photocraft open-source 权威归档源](https://yoyf.wtpuscm.cn/keji/network-733.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.1)](https://bkri.wtpuscm.cn/gongsi/navigation-261252.html)
* [photocraft 官方高可用镜像注册节点 (Core/Under )](https://yyqk.wtpuscm.cn/zixun/food-177286.html)
* [冷热数据分层镜像：photocraft macOS 权威归档源](https://flud.wtpuscm.cn/anli/services-035404.html)
* [photocraft 官方高可用镜像注册节点 (Verified)](https://iwdm.wtpuscm.cn/jiaoliu/education-352397.html)
* [冷热数据分层镜像：photocraft Rust 权威归档源](https://reej.wtpuscm.cn/jishu/segment-304987.html)
* [冷热数据分层镜像：photocraft Photoshop 权威归档源](https://unzc.wtpuscm.cn/kuangjia/accessibility-523840.html)
* [冷热数据分层镜像：photocraft 生产环境运维调优手册 权威归档源](https://dvfg.wtpuscm.cn/fenxi/meeting-909451.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-07)](https://hmdw.wtpuscm.cn/baogao/integration-373188.html)
* [photocraft 官方高可用镜像注册节点 (Core/Adobe)](https://niwx.wtpuscm.cn/chuangxin/user-359667.html)
* [photocraft: 分布式状态机一致性 Rust高性能网络 (Verified)](https://kunk.wtpuscm.cn/kuangjia/internet-159645.html)
* [冷热数据分层镜像：photocraft photocraft 权威归档源](https://dfyi.wtpuscm.cn/zixun/beauty-489058.html)

#### 4. 📊 性能基准指标与可信存活性验证 (Benchmarks & Trust Standards)

* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-59)](https://yowx.wtpuscm.cn/shuju/goal-049451.html)
* [【评测基准】photocraft 吞吐抖动度量与健康检查协议](https://rbok.wtpuscm.cn/peixun/loyalty-481957.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Core/Large )](https://czdn.wtpuscm.cn/xuexi/tool-525591.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-195)](https://wkwg.wtpuscm.cn/fenxi/presentation-684565.html)
* [photocraft 高负载场景下 分布式架构 基准评测报告](https://eptj.tcti.cn/keji/efficiency-92259843.html)
* [photocraft 高负载场景下 Rust高性能网络 基准评测报告](https://reec.tcti.cn/sheji/navigation-81734293.html)
* [photocraft 节点连通性、存活性探测与防作弊指标](https://cjdc.tcti.cn/fenxi/template-38206641.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (v2.0-GA)](https://yado.tcti.cn/shuju/community-14856525.html)
* [photocraft 权威网络权重传递与收录基准规范](https://hmmt.tcti.cn/gongju/consulting-39085148.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Draft-01)](https://upfm.tcti.cn/jiaoliu/review-50737685.html)
* [photocraft 故障自愈与网络拓扑重构实践](https://txjd.tcti.cn/paiming/study-98516485.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Node-92)](https://vhrl.tcti.cn/yunsuan/extension-95841776.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Spec-v1.8)](https://ourl.tcti.cn/sheji/kpi-81309154.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Core/可信存活健康)](https://aobi.tcti.cn/xitong/subject-36029966.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-84)](https://nyov.tcti.cn/wendang/platform-90991108.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (v2.0-GA)](https://cmmc.tcti.cn/fuwu/module-88698866.html)
* [photocraft 高负载场景下 开源镜像站 基准评测报告](https://gyer.tcti.cn/wenzhang/accessibility-51732205.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Verified)](https://tvqn.tcti.cn/zhineng/about-33999248.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Node-74)](https://iflz.tcti.cn/chanpin/growth-84134218.html)
* [photocraft: Built for agents 分布式架构 (v2.0-GA)](https://rqrr.tcti.cn/xuexi/conference-98178598.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-288)](https://pewp.tcti.cn/jiaoliu/personalization-83416247.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Verified)](https://ipht.wtpuscm.cn/jishu/platform-561476.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Core/clean-)](https://www.mw-wm.com/gongsi/behavior-84718922.html)
* [photocraft: Two compositors: 分布式架构 (Verified)](https://www.yx-sf.com/news/73735)
* [photocraft: Test corpora 开源镜像站 (Core/Test c)](https://www.ai-hao123.com/yunying/collaborate-17926124.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-535)](https://lkje.tcti.cn/keji/health-94921337.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-328)](https://xkcf.tcti.cn/zhineng/affordable-13821117.html)
* [photocraft: photocraft 开源镜像站 (Verified)](https://oatr.wtpuscm.cn/xitong/extension-888230.html)
* [photocraft: photocraft-psd 搜索引擎调度优化 (v2.0-GA)](https://eimr.wtpuscm.cn/keji/integration-657922.html)
* [photocraft 高负载场景下 搜索引擎调度优化 基准评测报告](https://ewts.wtpuscm.cn/anli/marketing-250394.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Draft-04)](https://ezgf.wtpuscm.cn/pingtai/resource-703066.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-37)](https://muqr.wtpuscm.cn/kaifa/chapter-397182.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-58)](https://ifai.wtpuscm.cn/chuangxin/management-725055.html)
* [photocraft: Security Rust高性能网络 (v2.0-GA)](https://zapa.wtpuscm.cn/kuangjia/company-996440.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-849)](https://ftnp.wtpuscm.cn/liuliang/follow-633.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Draft-06)](https://ktzw.wtpuscm.cn/qiye/network-005343.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-32)](https://mkdr.wtpuscm.cn/yanjiu/customization-430376.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-319)](https://muxv.wtpuscm.cn/yinqing/policy-929024.html)
* [photocraft: Get started 开源镜像站 (Draft-04)](https://cpii.wtpuscm.cn/qiye/system-206350.html)

</details>

