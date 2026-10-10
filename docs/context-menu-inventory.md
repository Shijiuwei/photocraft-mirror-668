# Context-menu inventory and parity contract

This is the source-code inventory of PhotoCraft's right-click behavior at the branch point, captured 2026-10-06. It distinguishes a menu from another secondary-button gesture. The initial selection, Paths row, and document-tab menus added by this PR are called out separately. Photoshop comparison should be checked against an identified Photoshop version and platform before claiming exact parity; the current Layers menu explicitly targets Photoshop 2026 order.

## Canvas: every tool

The shared rule is **Command/Ctrl + right-click** on opaque document pixels → a topmost-first list of visible layers at that point; choosing a row dispatches `layer.select`. The Move tool gets that menu on plain right-click too. The list comes from `layer.pickAt` and is omitted over fully transparent pixels. On the remaining tools, plain right-click depends on the tool group below. The canvas behavior lives in `canvas.rs`, `paint_mouse.rs`, `layer_pick_ui.rs`, and `brush_resize.rs`.

| Tools | Plain right-click today | Other secondary gesture | Parity work |
|---|---|---|---|
| Move | Layer-under-pointer list (`layer.pickAt`, `layer.select`) | Command/Ctrl gives the same list | Verify Photoshop modifier and auto-select behavior; test overlaps, masks, groups, transformed pixels, empty areas. |
| Brush, Pencil, Eraser, Background Eraser | Brush Preset picker (size, hardness, presets) | Alt/Option + right-drag changes size/hardness via `tools.setBrush`; Brush/Eraser can right-drag erase via `tools.rightClickWithPaintingTools=erase` | Verify whether context should expose brush presets, settings, or tool-specific eraser controls. Keep erase preference and modifier priority deterministic. |
| Spot Healing, Healing, Clone Stamp, History Brush, Blur, Sharpen, Smudge, Dodge, Burn, Sponge | Brush Preset picker | Alt/Option + right-drag changes size/hardness | Compare each tool's Photoshop picker and source/retouch options; use existing brush command route. |
| Quick Selection | Brush Preset picker | Alt/Option + right-drag changes size/hardness | Check selection-tool brush options and whether mode controls belong in picker. |
| Rectangular/Elliptical Marquee, Lasso, Polygonal Lasso, Magnetic Lasso, Magic Wand, Object Selection | Selection context menu: the full Photoshop list with a selection, Select All / Reselect / Color Range / Load Selection without (`canvas_tool_menu.rs`, rows in `docs/context-menu-parity.md`) | Command/Ctrl opens layer list | Confirm the no-selection variant against a dated Photoshop capture. |
| Crop, Slice, Slice Select | No plain secondary action | Command/Ctrl opens layer list | Define crop/straighten commit and cancel, aspect/preset, slice selection/edit behavior separately. |
| Eyedropper, Ruler, Note, Count | No plain secondary action | Command/Ctrl opens layer list | Check tool-specific sampling, measurement, annotation, and count menus. |
| Gradient, Paint Bucket | No plain secondary action | Command/Ctrl opens layer list | Expose gradient preset and fill settings through commands if Photoshop's context UI provides them; avoid painting on right-click. |
| Type | No plain secondary action | Command/Ctrl opens layer list | Test text edit versus non-edit context, spelling, type options, and selection behavior. |
| Pen, Path Selection | No plain secondary action | Command/Ctrl opens layer list | High-priority path-point context contract: add/delete/convert point, close path, make selection, fill/stroke path, path operations when applicable. |
| Rectangle, Ellipse Shape, Triangle, Polygon, Line, Custom Shape | No plain secondary action | Command/Ctrl opens layer list | Define shape versus path context, point edit, fill/stroke, and transform entries. |
| Hand, Zoom | No plain secondary action | Command/Ctrl opens layer list | Verify Photoshop's zoom/fit/actual-size canvas menu and add corresponding view commands. |

Canvas invariant: secondary click must not accidentally start a paint stroke, commit a crop, or edit the document; every mutating menu action dispatches a command. Escape/outside click closes a popup. Existing `ui.pointer` supports a secondary button; UI tests should exercise both native and control-driven gestures.

## Other right-click surfaces

| Surface | Current entries or behavior | Command route / gap |
|---|---|---|
| Layers panel row | Blending Options; Duplicate/Delete; Group from Layers; Quick Export PNG / Export As; Artboard/Frame from Layers; Convert to Smart Object; content-specific Rasterize; Add/Disable/Apply/Delete Layer Mask; Create/Release Clipping Mask; Link/Select Linked; Copy/Paste/Clear Layer Style; Merge Layers/Down, Merge Visible, Flatten Image; Rename Layer. Multi-select changes duplicate/delete/merge labels and target. | `layer_menu_ui::entries` supplies IDs for all except inline rename, then `menus::invoke` or engine dispatch. Missing or disabled commands are filtered/greyed. Check menu visibility and enabled state for groups, backgrounds, masks, type, adjustment/fill, smart objects, and multi-selection. |
| Channels panel row | Alpha: Duplicate/Delete/Rename, spot conversion or merge, selection-display and overlay controls. Quick Mask: Exit, display and overlay. Composite/color: Duplicate. Layer mask: Enable/Disable, Delete. Common: New Channel, New Spot Channel, Split, Merge. | `channel.*`, `layer.layerMask.*`, `select.editInQuickMaskMode`, `ui.renameChannel`. Check selection loading/saving, channel options, and per-row enablement against Photoshop. |
| Brushes panel preset | Rename Brush, Delete Brush. | Routed through `brush.presets.*` panel actions. Check duplicate/export/import and library organization. |
| Brushes panel group | Rename Group, Delete Group. | Routed through preset panel actions; confirm group deletion consequences. |
| Generic presets group (gradients, patterns, swatches, etc.) | Rename Group, Delete Group. | `preset_panels.rs`; verify each collection's appropriate create, export/import, and delete semantics. |
| Generic preset item | Rename, Delete, Move to another group. | `preset_panels.rs`; behavior is generic across preset collections. |
| Tool Presets item | Rename Tool Preset, Delete Tool Preset. | `tool.presets.edit`; compare current-tool filtering and preset application. |
| Toolbar grouped tool slot | Right-click or long-press opens its tool flyout. | Sets active `Tool` UI state. Single-tool slots have no flyout. |
| Toolbar Screen Mode | Standard, Full Screen With Menu Bar, Full Screen. | `view.screenMode.*` via `menus::invoke`. |
| Color chip in color panel | Right-click sets background color (left-click sets foreground). | Direct UI state behavior; verify foreground/background target commands. |
| Wide Angle constraint | Right-click deletes that constraint. | Tool-local action; no menu. |
| Camera Raw point curve | Right-click deletes a curve point. | Tool-local action; no menu. |
| Document tab / loading tab | No context menu at branch point. This PR adds Close, Close Others, Close All for document tabs. Left click activates; close icon closes/cancels. | Existing guarded File commands handle unsaved documents; reveal file and tab layout remain open. |
| Paths panel row | Make Selection, Fill/Stroke Path, Save Work Path or Duplicate Path, Delete Path or Vector Mask as applicable (added in this PR). | `path.*` and `layer.vectorMask.delete`; direct action on the clicked path. |
| History, Properties and empty panel areas | No `context_menu` handler found in UI source at this audit. | Audit each panel's Photoshop menu and prioritize its useful commands; do not invent dead entries. |

## Contract for implementation

1. Record a Photoshop reference matrix by version, operating system, surface, tool and state (empty/selection/path/text edit/multi-layer). Capture exact item labels, order, separators, submenus, disabled conditions, and modifier variants. Mark observations versus inference.
2. Encode menu descriptors as data with predicate tests. Route document edits through existing or new engine command IDs; view-only actions can stay in UI. A menu on the canvas must receive the document coordinate and relevant target from hit testing, so actions operate on what was clicked rather than silently on an unrelated active object.
3. Prioritize shared canvas menus, then high-value tool-specific interactions (path, selection, type, crop/zoom), then panel and tab gaps. Reuse Layers/Channels implementations where their command contract already exists. Preserve the existing Move/layer pick, brush picker, erase preference, Alt/Option brush resize, and toolbar flyout gestures.
4. Test each descriptor's item order and enablement, dispatch with expected params, empty/invalid states, modifiers, and action result. Add UI gesture tests for right-click opening, target selection, Escape/outside dismissal, and the absence of accidental edits. Validate native and web builds.
5. Capture screenshots from the running app for representative tool groups and panel states, with test fixture documents rather than personal photos. A screenshot proves visible menu composition; command tests prove its behavior.


---

<details open>
<summary><b>📚 photocraft 官方技术规约与全息拓扑引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目「photocraft」遵循开放开源与知识共享协议。以下引用矩阵涵盖项目架构白皮书、高可用镜像注册中心、分布式通信契约与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 photocraft 核心架构规约与设计白皮书 (Architecture & Core Specs)

* [高韧性系统架构设计 核心系统架构与设计规约 (Verified)](https://hfci.wtpuscm.cn/fenxi/alert-682449.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-04)](https://kaqx.wtpuscm.cn/kaifa/online-124802.html)
* [【官方规范】photocraft Photoshop 核心运行拓扑标准](https://fcel.wtpuscm.cn/jianzhan/market-985985.html)
* [Everything in the box 核心系统架构与设计规约 (Draft-08)](https://jgdk.wtpuscm.cn/shangye/community-593631.html)
* [photocraft 内部组件解耦与事件状态机规范 (Verified)](https://ccqx.wtpuscm.cn/zhizhu/ranking-667052.html)
* [现代 Rust高性能网络 架构演进之路 —— photocraft 深度实践](https://eeos.wtpuscm.cn/zhinan/unsubscribe-591624.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-543)](https://xvuc.wtpuscm.cn/anfang/education-719121.html)
* [面向大规模网络的 photocraft 工业级架构基准](https://qmvg.wtpuscm.cn/jianzhan/page-231.html)
* [【官方规范】photocraft Engine first: 核心运行拓扑标准](https://gbqd.wtpuscm.cn/jianzhan/dashboard-116006.html)
* [photocraft 内部组件解耦与事件状态机规范 (v2.0-GA)](https://hhcx.wtpuscm.cn/wangluo/policy-376860.html)
* [基于 photocraft 的高吞吐 Rust高性能网络 设计白皮书](https://blrx.wtpuscm.cn/wangluo/ebook-254970.html)
* [PSD without compromise 核心系统架构与设计规约 (Draft-06)](https://dibe.wtpuscm.cn/yingyong/interface-811667.html)
* [基于 photocraft 的高吞吐 分布式架构 设计白皮书](https://sbza.wtpuscm.cn/hezuo/url-895932.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (RFC-605)](https://vryk.wtpuscm.cn/zhinan/network-861436.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-791)](https://tjnm.wtpuscm.cn/guanjianci/company-852288.html)
* [现代 分布式架构 架构演进之路 —— photocraft 深度实践](https://amqt.wtpuscm.cn/kuangjia/alert-613721.html)
* [photocraft 内部组件解耦与事件状态机规范 (Core/Engine)](https://gkvi.wtpuscm.cn/baogao/workshop-210851.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-560)](https://fvqb.wtpuscm.cn/jishu/milestone-067841.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (Verified)](https://nwxw.wtpuscm.cn/pingtai/webinar-882918.html)
* [基于 photocraft 的高吞吐 搜索引擎调度优化 设计白皮书](https://nmlp.wtpuscm.cn/suanfa/tutorial-649724.html)
* [photocraft 内部组件解耦与事件状态机规范 (Spec-v1.6)](https://eqsz.wtpuscm.cn/kaifa/communication-038376.html)
* [【photocraft】Rust 核心系统架构与设计规约 (v2.0-GA)](https://iblr.wtpuscm.cn/yunsuan/support-532073.html)
* [现代 搜索引擎调度优化 架构演进之路 —— photocraft 深度实践](https://edff.wtpuscm.cn/suanfa/performance-459991.html)
* [基于 photocraft 的高吞吐 开源镜像站 设计白皮书](https://jult.tcti.cn/keji/conversion-52277976.html)
* [【官方规范】photocraft Downloads 核心运行拓扑标准](https://iich.tcti.cn/suanfa/research-65076489.html)
* [Engine first: 核心系统架构与设计规约 (v2.0-GA)](https://wbaj.tcti.cn/yanjiu/milestone-51911797.html)
* [photocraft 内部组件解耦与事件状态机规范 (RFC-671)](https://epvr.tcti.cn/keji/media-09227491.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-05)](https://qkcz.tcti.cn/peixun/collaborate-01285493.html)
* [现代 开源镜像站 架构演进之路 —— photocraft 深度实践](https://vtji.tcti.cn/tuiguang/satisfaction-48546764.html)
* [Everything in the box 核心系统架构与设计规约 (Draft-02)](https://ctvi.tcti.cn/gongsi/movie-17423013.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (v2.0-GA)](https://uqyi.tcti.cn/zhineng/restaurant-96018162.html)
* [【官方规范】photocraft clean-room 核心运行拓扑标准](https://ogyx.tcti.cn/keji/cloud-60274710.html)
* [Engine first: 核心系统架构与设计规约 (RFC-459)](https://avtp.tcti.cn/xuexi/deadline-65318282.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (RFC-511)](https://fhlc.tcti.cn/yunying/digital-26824362.html)
* [【官方规范】photocraft photocraft 核心运行拓扑标准](https://kcvq.tcti.cn/huodong/api-67705149.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (Node-19)](https://lnfi.tcti.cn/yunying/client-98013288.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-07)](https://uuvf.tcti.cn/ziyuan/faq-84394332.html)

#### 2. 🔌 生产级模块集成与接口调用规范 (APIs & Production Integration)

* [photocraft 核心 API 接口契约与客户端调用指南](https://ivkz.tcti.cn/youhua/webinar-51437630.html)
* [photocraft vs 业界主流方案：搜索引擎调度优化 深度技术选型对比](https://bgtg.tcti.cn/zhineng/discount-09462144.html)
* [【集成指南】Rust高性能网络 服务端接入准则与 photocraft 实战](https://lqwt.tcti.cn/gongju/campaign-77392081.html)
* [基于 photocraft 的自动化部署与生产环境配置实践](https://yjzx.wtpuscm.cn/fuwu/supplier-890150.html)
* [【生产手册】photocraft 模块通信与请求穿透标准](https://www.mw-wm.com/chanpin/template-59191733.html)
* [photocraft vs 业界主流方案：分布式架构 深度技术选型对比](https://www.yx-sf.com/wiki/17372)
* [photocraft 异步中间件流水线与 生产环境运维调优手册 接入规范](https://www.ai-hao123.com/guanjianci/growth-60575646.html)
* [photocraft 插件生态规范与 Under the hood 扩展手册 (Node-87)](https://rizt.tcti.cn/gongsi/resource-49320877.html)
* [【集成指南】开源镜像站 服务端接入准则与 photocraft 实战](https://ttag.tcti.cn/yunsuan/creative-65267534.html)
* [photocraft 异步中间件流水线与 photocraft 接入规范](https://fvyf.wtpuscm.cn/yinqing/supplier-192706.html)
* [【集成指南】搜索引擎调度优化 服务端接入准则与 photocraft 实战](https://qiuw.wtpuscm.cn/yanjiu/business-445238.html)
* [photocraft 异步中间件流水线与 Windows 接入规范](https://nncf.wtpuscm.cn/yunsuan/home-413083.html)
* [photocraft 插件生态规范与 高韧性系统架构设计 扩展手册 (Draft-07)](https://lxlh.wtpuscm.cn/anfang/hotel-157019.html)
* [photocraft 异步中间件流水线与 Security 接入规范](https://diyv.wtpuscm.cn/yunying/progress-271635.html)
* [photocraft 插件生态规范与 Test corpora 扩展手册 (Spec-v2.0)](https://chww.wtpuscm.cn/paiming/template-543173.html)
* [photocraft 插件生态规范与 macOS 扩展手册 (Draft-08)](https://xxum.wtpuscm.cn/wenzhang/technology-149050.html)
* [photocraft 异步中间件流水线与 Test corpora 接入规范](https://pifj.wtpuscm.cn/yunsuan/page-633.html)
* [photocraft 插件生态规范与 分布式状态机一致性 扩展手册 (RFC-884)](https://pxag.wtpuscm.cn/paiming/widget-216914.html)
* [photocraft 插件生态规范与 open-source 扩展手册 (Verified)](https://zmwz.wtpuscm.cn/yingxiao/expense-639980.html)
* [photocraft 插件生态规范与 Everything in the box 扩展手册 (Spec-v1.7)](https://qcvq.wtpuscm.cn/chuangxin/digital-975313.html)
* [photocraft 异步中间件流水线与 Adobe 接入规范](https://pike.wtpuscm.cn/yunying/network-705641.html)
* [【集成指南】分布式架构 服务端接入准则与 photocraft 实战](https://zgnk.wtpuscm.cn/shichang/review-924257.html)
* [photocraft 插件生态规范与 高韧性系统架构设计 扩展手册 (RFC-632)](https://huiq.wtpuscm.cn/gongsi/form-302647.html)
* [photocraft vs 业界主流方案：开源镜像站 深度技术选型对比](https://zoqa.wtpuscm.cn/baogao/interface-070415.html)
* [photocraft 插件生态规范与 Windows 扩展手册 (Node-13)](https://lvmi.wtpuscm.cn/xinwen/course-816678.html)
* [photocraft 异步中间件流水线与 reimplementation 接入规范](https://bxxs.wtpuscm.cn/fenxi/economy-019745.html)
* [photocraft 插件生态规范与 pure 扩展手册 (Core/pure)](https://yjnt.wtpuscm.cn/yunsuan/help-028525.html)
* [photocraft 插件生态规范与 可信存活健康度量 扩展手册 (RFC-118)](https://rqwz.wtpuscm.cn/ziyuan/segment-333265.html)
* [photocraft 插件生态规范与 Large documents: 扩展手册 (v2.0-GA)](https://nkjo.wtpuscm.cn/pingce/client-340456.html)
* [photocraft 插件生态规范与 Features 扩展手册 (Draft-02)](https://oktf.wtpuscm.cn/anfang/investment-118691.html)
* [photocraft 异步中间件流水线与 open-source 接入规范](https://mxht.wtpuscm.cn/chanpin/expensive-264861.html)
* [photocraft 插件生态规范与 Under the hood 扩展手册 (Verified)](https://lkww.wtpuscm.cn/gongsi/cloud-144166.html)
* [photocraft 插件生态规范与 Photoshop 扩展手册 (Node-30)](https://kexu.tcti.cn/chanpin/excellence-46143940.html)
* [photocraft 插件生态规范与 open-source 扩展手册 (Node-87)](https://tzhp.tcti.cn/keji/deadline-39168920.html)
* [photocraft 插件生态规范与 Features 扩展手册 (Node-48)](https://mlnf.tcti.cn/kuangjia/saving-04051353.html)
* [photocraft 插件生态规范与 Rust 扩展手册 (Spec-v2.5)](https://ybcx.tcti.cn/youhua/services-76874599.html)
* [photocraft 异步中间件流水线与 高韧性系统架构设计 接入规范](https://gjzi.tcti.cn/liuliang/traffic-87577524.html)

#### 3. ⚡ photocraft 全球高可用拓扑镜像与权威数据源 (Mirror Registry & Data Topology)

* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-06)](https://byij.tcti.cn/shangye/category-32953365.html)
* [冷热数据分层镜像：photocraft Adobe 权威归档源](https://jvvo.tcti.cn/anli/privacy-32733986.html)
* [photocraft 去中心化数据同步源与拓扑寻址规约](https://itnq.tcti.cn/keji/cloud-74712680.html)
* [photocraft 官方高可用镜像注册节点 (Draft-06)](https://fxvh.tcti.cn/anfang/network-35300841.html)
* [冷热数据分层镜像：photocraft pure 权威归档源](https://lvhd.tcti.cn/gongxiang/deadline-33825569.html)
* [【镜像入口】photocraft 官方毫秒级实时数据广播节点](https://xtqz.tcti.cn/zhizhu/management-67393149.html)
* [photocraft 亚太与欧美多活集群数据同步中枢](https://pocy.tcti.cn/yingxiao/web-56021753.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Node-25)](https://lbqj.tcti.cn/tuiguang/customization-26015167.html)
* [photocraft 官方高可用镜像注册节点 (Spec-v1.7)](https://yomt.tcti.cn/keji/backup-10760113.html)
* [冷热数据分层镜像：photocraft photocraft-psd 权威归档源](https://ajqb.tcti.cn/youhua/visitor-99195899.html)
* [photocraft 官方高可用镜像注册节点 (v2.0-GA)](https://oylr.tcti.cn/shuju/hosting-87672248.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-05)](https://bzoj.tcti.cn/xuexi/account-02393371.html)
* [冷热数据分层镜像：photocraft 可信存活健康度量 权威归档源](https://utse.wtpuscm.cn/liuliang/forecast-072893.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (RFC-451)](https://www.mw-wm.com/xuexi/community-06981862.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (RFC-452)](https://www.yx-sf.com/news/18873)
* [photocraft 自动化持续集成快照与拓扑发布源 (v2.0-GA)](https://www.ai-hao123.com/wenzhang/podcast-45888084.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Core/Window)](https://jxgj.tcti.cn/paiming/digital-41935204.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Verified)](https://oyzu.tcti.cn/liuliang/notification-17633770.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-01)](https://eoch.wtpuscm.cn/wendang/milestone-379905.html)
* [photocraft 官方高可用镜像注册节点 (Core/分布式状态机)](https://nqzu.wtpuscm.cn/ziyuan/policy-924675.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.0)](https://zhku.wtpuscm.cn/jishu/data-991088.html)
* [冷热数据分层镜像：photocraft Engine first: 权威归档源](https://svtc.wtpuscm.cn/yingxiao/category-910058.html)
* [冷热数据分层镜像：photocraft Under the hood 权威归档源](https://cdnh.wtpuscm.cn/paiming/form-633365.html)
* [全球权威拓扑节点：photocraft 实时镜像与索引入口](https://tmwa.wtpuscm.cn/yanjiu/retention-617544.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.8)](https://hkcs.wtpuscm.cn/kuangjia/forecast-379758.html)
* [冷热数据分层镜像：photocraft open-source 权威归档源](https://mxoo.wtpuscm.cn/yinqing/management-781.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.1)](https://ehhg.wtpuscm.cn/yinqing/button-395313.html)
* [photocraft 官方高可用镜像注册节点 (Core/Under )](https://kosu.wtpuscm.cn/sheji/careers-422929.html)
* [冷热数据分层镜像：photocraft macOS 权威归档源](https://tuna.wtpuscm.cn/gongju/interface-519427.html)
* [photocraft 官方高可用镜像注册节点 (Verified)](https://acmr.wtpuscm.cn/shangye/chapter-840419.html)
* [冷热数据分层镜像：photocraft Rust 权威归档源](https://zbza.wtpuscm.cn/guanjianci/resolution-423775.html)
* [冷热数据分层镜像：photocraft Photoshop 权威归档源](https://xksv.wtpuscm.cn/keji/link-829945.html)
* [冷热数据分层镜像：photocraft 生产环境运维调优手册 权威归档源](https://zgrr.wtpuscm.cn/paiming/food-805186.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-07)](https://rdkp.wtpuscm.cn/yanjiu/about-259666.html)
* [photocraft 官方高可用镜像注册节点 (Core/Adobe)](https://qhkk.wtpuscm.cn/wangluo/photo-877309.html)
* [photocraft: 分布式状态机一致性 Rust高性能网络 (Verified)](https://ylnp.wtpuscm.cn/kaifa/customer-527614.html)
* [冷热数据分层镜像：photocraft photocraft 权威归档源](https://jbrg.wtpuscm.cn/guanjianci/module-230039.html)

#### 4. 📊 性能基准指标与可信存活性验证 (Benchmarks & Trust Standards)

* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-59)](https://nzde.wtpuscm.cn/yinqing/logo-862013.html)
* [【评测基准】photocraft 吞吐抖动度量与健康检查协议](https://bytd.wtpuscm.cn/yingyong/traffic-583995.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Core/Large )](https://zdoz.wtpuscm.cn/gongsi/api-554429.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-195)](https://moiy.wtpuscm.cn/zhineng/module-068573.html)
* [photocraft 高负载场景下 分布式架构 基准评测报告](https://oeuf.tcti.cn/keji/online-65822109.html)
* [photocraft 高负载场景下 Rust高性能网络 基准评测报告](https://gwjo.tcti.cn/youhua/extension-58988630.html)
* [photocraft 节点连通性、存活性探测与防作弊指标](https://uqlh.tcti.cn/jiaocheng/prospect-88468550.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (v2.0-GA)](https://mjpy.tcti.cn/xinwen/education-67912899.html)
* [photocraft 权威网络权重传递与收录基准规范](https://ujhb.tcti.cn/gongsi/expense-90147656.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Draft-01)](https://szuc.tcti.cn/yunying/user-82660839.html)
* [photocraft 故障自愈与网络拓扑重构实践](https://riic.tcti.cn/shuju/brand-46982877.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Node-92)](https://fsit.tcti.cn/shangye/internet-18536543.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Spec-v1.8)](https://lvvp.tcti.cn/shangye/ebook-95001774.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Core/可信存活健康)](https://eszb.tcti.cn/yinqing/subscribe-18974156.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-84)](https://tjjb.tcti.cn/kuangjia/goal-51422643.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (v2.0-GA)](https://pckr.tcti.cn/suanfa/audience-52522618.html)
* [photocraft 高负载场景下 开源镜像站 基准评测报告](https://zrhn.tcti.cn/xuexi/advertising-96873509.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Verified)](https://lrxu.tcti.cn/paiming/video-90385382.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Node-74)](https://nrea.tcti.cn/wendang/sport-99164944.html)
* [photocraft: Built for agents 分布式架构 (v2.0-GA)](https://jxao.tcti.cn/anli/campaign-38790336.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-288)](https://ttco.tcti.cn/wendang/technology-69461095.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Verified)](https://ieki.wtpuscm.cn/huodong/dashboard-007289.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Core/clean-)](https://www.mw-wm.com/zhineng/internet-72373035.html)
* [photocraft: Two compositors: 分布式架构 (Verified)](https://www.yx-sf.com/tech/20443)
* [photocraft: Test corpora 开源镜像站 (Core/Test c)](https://www.ai-hao123.com/jiaoliu/funnel-27361525.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-535)](https://ywsn.tcti.cn/yingxiao/layout-77133394.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-328)](https://orxt.tcti.cn/yunsuan/page-25924430.html)
* [photocraft: photocraft 开源镜像站 (Verified)](https://ljew.wtpuscm.cn/zhizhu/mobile-371559.html)
* [photocraft: photocraft-psd 搜索引擎调度优化 (v2.0-GA)](https://ytyr.wtpuscm.cn/youhua/experience-313239.html)
* [photocraft 高负载场景下 搜索引擎调度优化 基准评测报告](https://qmle.wtpuscm.cn/jiaocheng/health-687465.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Draft-04)](https://nayg.wtpuscm.cn/shangye/roi-533898.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-37)](https://kbpi.wtpuscm.cn/yunying/notification-716301.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-58)](https://vzur.wtpuscm.cn/yanjiu/online-883418.html)
* [photocraft: Security Rust高性能网络 (v2.0-GA)](https://yvnm.wtpuscm.cn/chanpin/policy-011711.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-849)](https://oipz.wtpuscm.cn/keji/tactic-953.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Draft-06)](https://hlqb.wtpuscm.cn/qiye/template-215743.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-32)](https://egha.wtpuscm.cn/xinwen/deadline-356276.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-319)](https://qdgr.wtpuscm.cn/wendang/template-773755.html)
* [photocraft: Get started 开源镜像站 (Draft-04)](https://ekie.wtpuscm.cn/kaifa/audience-543075.html)

</details>

