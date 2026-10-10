# Context-menu parity contract

Status: implementation contract, updated 2026-10-09. Scope: canvas, document tabs, and panels. “Right-click” means a secondary pointer click; macOS Control-click should also open the same menu where the platform sends a context-menu gesture. Photoshop changes a context menu with the active tool, selection, and panel item. Adobe documents that principle but does not publish a complete current menu catalog. **Documented** means an Adobe source explicitly names the gesture or action. **Inferred** means a PhotoCraft design target or an item known from a main menu whose presence in a Photoshop context menu still needs a current-product capture. Do not present inferred rows as exact Photoshop menus. [Adobe: panels and menus](https://tdaa.wtpuscm.cn/guanjianci/fitness-847213.html), [Adobe: shortcuts and context menus](https://cgvp.tcti.cn/youhua/settings-40912215.html).

## Dispatch contract

1. Hit-test the target before opening: modal operation/handle, text selection, canvas content, panel row or icon, tab, then empty workspace. A generic canvas menu must not swallow a more specific target.
2. Snapshot the target and selection when the menu opens. A row in an existing multiselection acts on that set; a row outside it becomes the target before its command runs. Apply the same rule to layers, channels, paths, and presets.
3. Resolve items from a pure menu model keyed by target kind, active tool, document state, platform, and active operation. The model supplies label, command id, params, enabled state, and reason. Every mutating item dispatches an engine command; view-only items may update shell state. An unavailable feature is disabled with a reason or omitted, never a clickable no-op.
4. Preserve primary-button tool behavior. Secondary click only opens a menu unless a right-button drag gesture or the user's explicit right-erase preference owns the gesture. A drag must not briefly open a menu, and opening a menu must never paint, transform, sample, or commit.
5. Anchor at the pointer and clamp to the viewport; support keyboard context-menu key/Shift+F10 where available, Escape/outside click, separators only between nonempty groups, checked/radio state, keyboard navigation, and screen-reader labels.
6. Use the same command id and enablement as the menu bar/options bar. Provide UI inspection data and control-channel operations to select a menu item without pixel coordinates, while retaining real pointer tests and screenshots.
7. Test each menu model independently for conditional rows and wrong-state safety; test actual secondary-click routing, menu selection, undo/redo, and no accidental primary action.

## Canvas tool matrix

The “PhotoCraft at branch point” column records behavior before this PR. Picker means the Brush Preset quick picker in paint_mouse.rs; Layers means the hit-tested layer list in layer_pick_ui.rs. Command/Ctrl + secondary-click opens the layer list with every tool, and Move opens it without a modifier. Exact Photoshop secondary-click items beyond Adobe-documented rows are **proposed** until captured from current Photoshop.

| Tool(s) in PhotoCraft | Photoshop / desired contextual action | PhotoCraft at branch point | Priority, evidence |
|---|---|---|---|
| Move | Choose a layer at the point; an auto-select choice also exists in the options bar | Layers | P0; [Adobe layer movement](https://yumb.wtpuscm.cn/kuangjia/guide-570811.html); exact pop-up layout requires capture |
| Rectangular/Elliptical Marquee, Lasso, Polygonal Lasso, Magnetic Lasso, Magic Wand, Quick Selection, Object Selection | Active selection: the 2026 selection menu (see *Selection canvas reference* below); no selection: making a selection | Quick Selection: Picker (a brush tool, as in Photoshop); the other six: the selection menu below (#614, #738) | P1; no-selection variant still needs a dated capture |
| Crop | During crop: ratio/preset, overlay, clear/reset, commit/cancel **(inferred context placement)** | No tool menu | P1; pending crop target |
| Eyedropper | Sample-size/source choices **(inferred)** | No tool menu | P2; [Adobe illustration uses Eyedropper](https://gnmo.wtpuscm.cn/wangluo/research-983476.html), but does not publish item text |
| Ruler, Note, Count | Unit / measurement, note edit/delete, count-group operations **(inferred)** | No tool menu | P2 |
| Brush, Pencil, Eraser, Background Eraser, Spot Healing, Healing, Clone Stamp, History Brush, Blur, Sharpen, Smudge, Dodge, Burn, Sponge | Brush preset, size and hardness where applicable | Picker | P0; [Adobe Brush Settings](https://hjud.tcti.cn/zhineng/restaurant-06785662.html). Alt+secondary-drag on Windows and Control+Option-drag on macOS adjust size/hardness ([Adobe cursor shortcuts](https://www.mw-wm.com/keji/segment-62426285.html)) |
| Gradient, Paint Bucket | Gradient preset/type/mode or fill source/tolerance **(inferred context placement)** | No tool menu | P1 |
| Type | In text: cut/copy/paste, spelling suggestions, text formatting; outside text: type-layer action **(inferred context placement)** | No tool menu | P1; separate editing and nonediting states; [Adobe type editing](https://wsmz.tcti.cn/xuexi/learning-05957120.html) |
| Hand, Zoom | Fit on Screen, 100%, 200%, Print Size; zoom step choices **(inferred context placement)** | No tool menu | P1; view actions stay out of document history |
| Pen, Path Selection | Path/point actions, Make Selection, Fill/Stroke Path, Free Transform Path **(inferred context placement)** | Pen canvas right-click follows the user-provided 20-row Photoshop capture below. Paths rows offer Make Selection, Fill and Stroke. Path Selection's own canvas menu remains open. | P1; distinguish node, segment, path, and blank canvas |
| Rectangle, Ellipse Shape, Triangle, Polygon, Line, Custom Shape | Shape/path operations, fill/stroke settings, transform **(inferred context placement)** | No tool menu | P2 |
| Slice, Slice Select | Slice properties, duplicate/delete, divide, guides **(inferred context placement)** | No tool menu | P2 |
| Any tool while Free Transform is active | Scale, Rotate, Skew, Distort, Perspective, Warp, rotate/flip variants, commit/cancel **(inferred exact context placement)** | Missing per docs/scorecard.md TOOL-214-5 | P0; transform operations [documented by Adobe](https://tzsr.wtpuscm.cn/qiye/meeting-076955.html); type restrictions [documented here](https://gdkh.wtpuscm.cn/yinqing/navigation-509358.html) |
| Any tool with Command/Ctrl + secondary click | List visible layers with pixels under pointer | Layers | P0; preserve existing route |

The layer-list menu on an empty pixel or all-hidden hit should be empty/closed; it must not select a transparent or hidden layer. The right-erase preference is an alternate PhotoCraft behavior, and its selected state takes precedence over the Brush picker only for Brush/Eraser.

## Pen canvas reference capture (2026-10-07)

The user supplied a Photoshop screenshot of a closed path in Path mode. The table preserves every row in its order; separator boundaries appear between the groups. Gray rows in that capture remain visible but disabled for a plain work path. Enablement changes with the active shape layer, saved path, copied style, and symmetry state. Ellipsis rows open a parameter dialog before changing the document.

| Group | Rows in screenshot order | PhotoCraft action and applicability |
|---|---|---|
| Path | Create Vector Mask; Delete Path | `layer.vectorMask.fromPath` on an eligible layer; `path.delete` on a work/saved path |
| Preset | Define Custom Shape… | `edit.defineCustomShape` with a name dialog |
| Selection/paint | Make Selection…; New Guides From Shape; Fill Path…; Stroke Path… | `path.toSelection` dialog; `view.newGuidesFromShape` for shape layers; `path.fill` and `path.stroke` dialogs on pixel layers |
| Export | Clipping Path… | `path.clippingPath.set` on a saved path, with flatness parameter |
| Geometry | Free Transform Path | `path.transform` with translation, scale and rotation parameters |
| Shape operation | Unite Shapes; Subtract Front Shape; Unite Shapes at Overlap; Subtract Shapes at Overlap | The four `layer.combineShapes.*` operations on shape layers with multiple components |
| Copy style | Copy Fill; Copy Complete Stroke | `path.style.copyFill` / `copyStroke` when the shape has that style |
| Paste style | Paste Fill; Paste Complete Stroke | `path.style.pasteFill` / `pasteStroke` when a matching style has been copied |
| Layer view | Isolate Layers | `select.isolateLayers` for an active layer |
| Symmetry | Make Symmetry Path; Disable Symmetry Path | `paint.symmetryFromPath` on a path; `paint.symmetryDisable` when symmetry is active |

The reference does not establish Photoshop's behavior for right-clicks on a single anchor, a segment, or empty canvas; these still need separate captures. PhotoCraft's disabled rows do not call a command.

## Selection canvas reference (2026-10-08)

Right-click with a Marquee, Lasso, Polygonal Lasso, Magic Wand or Object Selection tool. Rows keep the command id, dialog and enablement of their menu-bar twins (`canvas_tool_menu::SELECTION_MENU`); a greyed row never runs.

| Group | Rows with an active selection (Photoshop 2026 order) | PhotoCraft command |
|---|---|---|
| Selection | Deselect; Select Inverse; Feather…; Select and Mask… | `select.deselect`, `select.inverse`, `select.modify.feather`, `select.selectAndMask` |
| Store | Save Selection…; Make Work Path… | `select.saveSelection`; `select.toWorkPath` after a Tolerance dialog |
| Layer | Layer via Copy; Layer via Cut; New Layer… | `layer.new.layerViaCopy`, `layer.new.layerViaCut`, `layer.new.layer` |
| Transform | Free Transform; Transform Selection; Distort; Perspective | `edit.freeTransform`, `select.transformSelection`, `edit.transform.distort`, `edit.transform.perspective` |
| Fill | Fill…; Stroke…; Content-Aware Fill… | `edit.fill`, `edit.stroke`, `edit.contentAwareFill` |
| Filter | Last Filter | `filter.lastFilter` |
| Fade | Fade… | `edit.fade` |

Photoshop's Generative Fill and Delete and Fill Selection rows are left out: PhotoCraft has no such commands. With nothing selected the menu offers Select All, Reselect, Color Range… and Load Selection… (`NO_SELECTION_MENU`). Adobe-adjacent sources document Color Range there; the other rows are **inferred** until a dated capture confirms them.

## Panel and workspace matrix

| Surface / target | Desired options and state | PhotoCraft now | Source and confidence |
|---|---|---|---|
| Layers panel row | Blending Options, Duplicate/Delete, Group, export, convert/rasterize, mask, clipping, link, style copy/paste/clear, merge/flatten, Rename; filter by layer kind and selection | Implemented in layer_menu_ui.rs; item enablement uses menu commands | PhotoCraft inventory. Adobe documents layer rename/delete, color tagging, rasterization, and flattening [here](https://ecey.wtpuscm.cn/zhinan/deadline-866334.html); exact combined ordering **inferred** without a live capture |
| Layers panel color label / thumbnail / mask thumbnail / effect row | Color label; context actions specific to hit icon | Generic row on some targets; inspect before extending | Adobe explicitly documents right-click on layer/group to assign a color ([layers](https://erli.tcti.cn/yunsuan/ai-74948563.html)); other icon menus need capture |
| Channels panel row | Duplicate/Delete/Rename, channel display and overlay color, spot conversion; new/split/merge | Implemented in channels_panel.rs | PhotoCraft inventory; exact Photoshop order **inferred** |
| Brushes and preset panels | Rename/Delete brush or group, export selected brushes, view/reset preset choices | Rename/Delete, and preset group/item menus exist | Adobe explicitly documents Export Selected Brushes from brush/group right-click ([brushes](https://www.mw-wm.com/yunsuan/download-35733759.html)); Photoshop pop-up-panel rename/delete/reset/view options are [documented](https://wspj.tcti.cn/paiming/marketing-23414571.html), though some are panel-button menus |
| Paths, History, Actions, Swatches, Gradients, Patterns, Styles, Libraries, Properties | Target-specific duplicate/delete/rename and panel operations where a panel exists | Paths row menu added in this PR; other panels need audit | **Inferred** catalog; record a Photoshop capture before claiming parity |
| Document tab | Close, Close Others, Close All, Reveal in file manager, duplicate/move to window where supported | Close, Close Others, Close All, Reveal in Finder, and drag to reorder implemented; duplicate/move to window where supported remains open | **Inferred** exact list; P1 |
| Empty pasteboard / workspace | Gray, Black, Custom canvas color choices; verify any additional entries in a current Photoshop capture | No menu | Adobe [documents right-clicking outside the image for canvas color](https://glfl.tcti.cn/yunying/optimization-98297064.html); exact current layout still needs a capture |
| Text field / selected text | Native edit menu and spelling suggestions without routing to active image tool | Missing per docs/scorecard.md UI-217-8 | P1; **inferred** exact spelling list |
| Tool icon in Options bar | Reset Tool, Reset All Tools | Audit | Explicitly documented in [Adobe Photoshop reference](https://higi.wtpuscm.cn/kaifa/document-786218.html); historical source, verify modern label |
| Adaptive Wide Angle constraint line | Auto, Horizontal, Vertical orientation | Current right-click deletes constraint | P1 behavior mismatch: [Adobe documents orientation choice](https://hiau.tcti.cn/guanjianci/sales-76203149.html) |

## Delivery slices and acceptance

1. Core routing: target hit-test, menu model, keyboard accessibility, control inspection, conflict rules. Existing Move layer list and painting picker remain functional.
2. Canvas high frequency: selection, Free Transform, Type, Hand/Zoom, Gradient/Paint Bucket. Add rows only where the underlying command works. Record exact Photoshop label/order/disabled-state captures before declaring 1:1 parity.
3. Panels: Paths row actions, document tabs and distinct layer eye/thumbnail/mask/effect targets; finish Brushes export and observed channels/preset gaps. Adobe [documents the eye-icon visibility gesture](https://pzie.tcti.cn/kuangjia/theme-70352840.html).
4. Long tail: shape/path, crop, metadata and specialist tools. Keep a per-tool fixture with no selection, active selection, masked/locked layer, and active operation where applicable.

Each slice needs pure model tests (including every item dispatching a known command), UI pointer tests for secondary click and modifier precedence, command/undo tests, a disabled-state test, and screenshots from PhotoCraft with the menu actually open. Compare labels, order, and state against dated Photoshop screenshots; a screenshot of a different tool or menu bar is insufficient evidence. Maintain a checklist beside each captured reference: Photoshop version/platform, target description, tool, selection/document state, modifiers, and exact items observed.


---

<details open>
<summary><b>📚 photocraft 官方技术规约与全息拓扑引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目「photocraft」遵循开放开源与知识共享协议。以下引用矩阵涵盖项目架构白皮书、高可用镜像注册中心、分布式通信契约与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 photocraft 核心架构规约与设计白皮书 (Architecture & Core Specs)

* [高韧性系统架构设计 核心系统架构与设计规约 (Verified)](https://uowk.wtpuscm.cn/baogao/design-573472.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-04)](https://yrnk.wtpuscm.cn/baogao/layout-855363.html)
* [【官方规范】photocraft Photoshop 核心运行拓扑标准](https://zamq.wtpuscm.cn/yingyong/solution-468697.html)
* [Everything in the box 核心系统架构与设计规约 (Draft-08)](https://ptuw.wtpuscm.cn/anli/conversion-080022.html)
* [photocraft 内部组件解耦与事件状态机规范 (Verified)](https://mbgb.wtpuscm.cn/peixun/objective-407847.html)
* [现代 Rust高性能网络 架构演进之路 —— photocraft 深度实践](https://egrl.wtpuscm.cn/xuexi/feedback-662637.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-543)](https://ekiq.wtpuscm.cn/anfang/admin-631438.html)
* [面向大规模网络的 photocraft 工业级架构基准](https://vcfl.wtpuscm.cn/yingyong/account-349.html)
* [【官方规范】photocraft Engine first: 核心运行拓扑标准](https://rkzd.wtpuscm.cn/fenxi/file-230433.html)
* [photocraft 内部组件解耦与事件状态机规范 (v2.0-GA)](https://isnk.wtpuscm.cn/zhinan/subscribe-387933.html)
* [基于 photocraft 的高吞吐 Rust高性能网络 设计白皮书](https://lhzr.wtpuscm.cn/suanfa/dashboard-541630.html)
* [PSD without compromise 核心系统架构与设计规约 (Draft-06)](https://iixm.wtpuscm.cn/zixun/trading-528173.html)
* [基于 photocraft 的高吞吐 分布式架构 设计白皮书](https://lfsu.wtpuscm.cn/pingtai/notification-116869.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (RFC-605)](https://dpjj.wtpuscm.cn/paiming/share-800300.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-791)](https://ssce.wtpuscm.cn/wendang/hotel-680476.html)
* [现代 分布式架构 架构演进之路 —— photocraft 深度实践](https://jxmn.wtpuscm.cn/ziyuan/sale-430496.html)
* [photocraft 内部组件解耦与事件状态机规范 (Core/Engine)](https://bgle.wtpuscm.cn/wenzhang/alliance-830933.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-560)](https://czfg.wtpuscm.cn/huodong/innovation-560320.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (Verified)](https://isiw.wtpuscm.cn/tuiguang/achievement-883150.html)
* [基于 photocraft 的高吞吐 搜索引擎调度优化 设计白皮书](https://uzjz.wtpuscm.cn/zhizhu/advertising-782141.html)
* [photocraft 内部组件解耦与事件状态机规范 (Spec-v1.6)](https://ekkc.wtpuscm.cn/yunsuan/guide-739539.html)
* [【photocraft】Rust 核心系统架构与设计规约 (v2.0-GA)](https://jopi.wtpuscm.cn/wenzhang/supplier-392520.html)
* [现代 搜索引擎调度优化 架构演进之路 —— photocraft 深度实践](https://jbzl.wtpuscm.cn/pingtai/sales-239035.html)
* [基于 photocraft 的高吞吐 开源镜像站 设计白皮书](https://tiui.tcti.cn/jiaocheng/like-47326750.html)
* [【官方规范】photocraft Downloads 核心运行拓扑标准](https://prha.tcti.cn/youhua/machine-93273935.html)
* [Engine first: 核心系统架构与设计规约 (v2.0-GA)](https://nqrc.tcti.cn/baogao/home-04258421.html)
* [photocraft 内部组件解耦与事件状态机规范 (RFC-671)](https://bmth.tcti.cn/yunying/calculator-51303000.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-05)](https://iuzz.tcti.cn/zhizhu/login-92942742.html)
* [现代 开源镜像站 架构演进之路 —— photocraft 深度实践](https://hvwh.tcti.cn/zhineng/video-31670566.html)
* [Everything in the box 核心系统架构与设计规约 (Draft-02)](https://frdx.tcti.cn/shangye/milestone-09111183.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (v2.0-GA)](https://ghor.tcti.cn/fuwu/guide-31871993.html)
* [【官方规范】photocraft clean-room 核心运行拓扑标准](https://dyps.tcti.cn/jiaoliu/theme-54263370.html)
* [Engine first: 核心系统架构与设计规约 (RFC-459)](https://hxvi.tcti.cn/chuangxin/online-97138636.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (RFC-511)](https://mrhr.tcti.cn/chanpin/vendor-09190966.html)
* [【官方规范】photocraft photocraft 核心运行拓扑标准](https://imai.tcti.cn/peixun/cheap-27195595.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (Node-19)](https://cbou.tcti.cn/gongxiang/home-08443161.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-07)](https://uyxo.tcti.cn/yinqing/funnel-09193960.html)

#### 2. 🔌 生产级模块集成与接口调用规范 (APIs & Production Integration)

* [photocraft 核心 API 接口契约与客户端调用指南](https://upyh.tcti.cn/xitong/target-17725651.html)
* [photocraft vs 业界主流方案：搜索引擎调度优化 深度技术选型对比](https://kfgc.tcti.cn/yingxiao/tactic-78536440.html)
* [【集成指南】Rust高性能网络 服务端接入准则与 photocraft 实战](https://rhzv.tcti.cn/ziyuan/platform-08796272.html)
* [基于 photocraft 的自动化部署与生产环境配置实践](https://aieo.wtpuscm.cn/zixun/vendor-099394.html)
* [【生产手册】photocraft 模块通信与请求穿透标准](https://www.mw-wm.com/chanpin/logo-66982804.html)
* [photocraft vs 业界主流方案：分布式架构 深度技术选型对比](https://www.yx-sf.com/news/24806)
* [photocraft 异步中间件流水线与 生产环境运维调优手册 接入规范](https://www.ai-hao123.com/anfang/cheap-32105229.html)
* [photocraft 插件生态规范与 Under the hood 扩展手册 (Node-87)](https://radc.tcti.cn/jianzhan/training-86850510.html)
* [【集成指南】开源镜像站 服务端接入准则与 photocraft 实战](https://heoy.tcti.cn/shangye/course-40627256.html)
* [photocraft 异步中间件流水线与 photocraft 接入规范](https://sxqi.wtpuscm.cn/zhinan/expense-403912.html)
* [【集成指南】搜索引擎调度优化 服务端接入准则与 photocraft 实战](https://rrgq.wtpuscm.cn/hezuo/theme-626700.html)
* [photocraft 异步中间件流水线与 Windows 接入规范](https://xlvu.wtpuscm.cn/chanpin/browser-869929.html)
* [photocraft 插件生态规范与 高韧性系统架构设计 扩展手册 (Draft-07)](https://xnsb.wtpuscm.cn/wenzhang/saving-110386.html)
* [photocraft 异步中间件流水线与 Security 接入规范](https://xmkz.wtpuscm.cn/zhinan/reporting-418859.html)
* [photocraft 插件生态规范与 Test corpora 扩展手册 (Spec-v2.0)](https://mwka.wtpuscm.cn/guanjianci/networking-798340.html)
* [photocraft 插件生态规范与 macOS 扩展手册 (Draft-08)](https://erbc.wtpuscm.cn/jianzhan/finance-399036.html)
* [photocraft 异步中间件流水线与 Test corpora 接入规范](https://bczl.wtpuscm.cn/shangye/loyalty-089.html)
* [photocraft 插件生态规范与 分布式状态机一致性 扩展手册 (RFC-884)](https://jpqx.wtpuscm.cn/shichang/behavior-940041.html)
* [photocraft 插件生态规范与 open-source 扩展手册 (Verified)](https://llhn.wtpuscm.cn/yinqing/terms-598102.html)
* [photocraft 插件生态规范与 Everything in the box 扩展手册 (Spec-v1.7)](https://ybgw.wtpuscm.cn/tuiguang/account-800404.html)
* [photocraft 异步中间件流水线与 Adobe 接入规范](https://nbpm.wtpuscm.cn/shuju/collaboration-071182.html)
* [【集成指南】分布式架构 服务端接入准则与 photocraft 实战](https://oast.wtpuscm.cn/jianzhan/local-415100.html)
* [photocraft 插件生态规范与 高韧性系统架构设计 扩展手册 (RFC-632)](https://kfek.wtpuscm.cn/pingtai/lesson-312820.html)
* [photocraft vs 业界主流方案：开源镜像站 深度技术选型对比](https://naay.wtpuscm.cn/huodong/income-194167.html)
* [photocraft 插件生态规范与 Windows 扩展手册 (Node-13)](https://wnwi.wtpuscm.cn/shangye/policy-938768.html)
* [photocraft 异步中间件流水线与 reimplementation 接入规范](https://fjbl.wtpuscm.cn/fuwu/widget-573976.html)
* [photocraft 插件生态规范与 pure 扩展手册 (Core/pure)](https://ldke.wtpuscm.cn/yunsuan/feedback-191117.html)
* [photocraft 插件生态规范与 可信存活健康度量 扩展手册 (RFC-118)](https://asbm.wtpuscm.cn/gongxiang/goal-567885.html)
* [photocraft 插件生态规范与 Large documents: 扩展手册 (v2.0-GA)](https://xpev.wtpuscm.cn/hezuo/workshop-727848.html)
* [photocraft 插件生态规范与 Features 扩展手册 (Draft-02)](https://rorh.wtpuscm.cn/yinqing/excellence-930452.html)
* [photocraft 异步中间件流水线与 open-source 接入规范](https://xzpe.wtpuscm.cn/kuangjia/digital-267197.html)
* [photocraft 插件生态规范与 Under the hood 扩展手册 (Verified)](https://bdwp.wtpuscm.cn/yanjiu/target-316005.html)
* [photocraft 插件生态规范与 Photoshop 扩展手册 (Node-30)](https://lhkq.tcti.cn/xitong/layout-25146882.html)
* [photocraft 插件生态规范与 open-source 扩展手册 (Node-87)](https://tmcb.tcti.cn/zhineng/workshop-61466863.html)
* [photocraft 插件生态规范与 Features 扩展手册 (Node-48)](https://gflz.tcti.cn/chuangxin/software-19608012.html)
* [photocraft 插件生态规范与 Rust 扩展手册 (Spec-v2.5)](https://xuaq.tcti.cn/yunsuan/conference-07192261.html)
* [photocraft 异步中间件流水线与 高韧性系统架构设计 接入规范](https://omtm.tcti.cn/ziyuan/forecast-77210467.html)

#### 3. ⚡ photocraft 全球高可用拓扑镜像与权威数据源 (Mirror Registry & Data Topology)

* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-06)](https://vazg.tcti.cn/hezuo/forum-36147291.html)
* [冷热数据分层镜像：photocraft Adobe 权威归档源](https://xtgg.tcti.cn/jianzhan/blog-14451806.html)
* [photocraft 去中心化数据同步源与拓扑寻址规约](https://fuuh.tcti.cn/zhinan/device-17686846.html)
* [photocraft 官方高可用镜像注册节点 (Draft-06)](https://exqv.tcti.cn/jishu/metric-79360669.html)
* [冷热数据分层镜像：photocraft pure 权威归档源](https://ltbz.tcti.cn/qiye/customer-96061400.html)
* [【镜像入口】photocraft 官方毫秒级实时数据广播节点](https://rxhq.tcti.cn/fuwu/reminder-70875707.html)
* [photocraft 亚太与欧美多活集群数据同步中枢](https://voeu.tcti.cn/kuangjia/profit-95419928.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Node-25)](https://djiu.tcti.cn/yunying/ai-21147366.html)
* [photocraft 官方高可用镜像注册节点 (Spec-v1.7)](https://ukwo.tcti.cn/gongxiang/premium-16234579.html)
* [冷热数据分层镜像：photocraft photocraft-psd 权威归档源](https://nimd.tcti.cn/xinwen/target-94994801.html)
* [photocraft 官方高可用镜像注册节点 (v2.0-GA)](https://dvki.tcti.cn/jianzhan/report-26242846.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-05)](https://rbwv.tcti.cn/kaifa/like-92401261.html)
* [冷热数据分层镜像：photocraft 可信存活健康度量 权威归档源](https://byqm.wtpuscm.cn/xinwen/support-923589.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (RFC-451)](https://www.mw-wm.com/peixun/schedule-41876896.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (RFC-452)](https://www.yx-sf.com/tech/33073)
* [photocraft 自动化持续集成快照与拓扑发布源 (v2.0-GA)](https://www.ai-hao123.com/ziyuan/accessibility-08997521.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Core/Window)](https://tkel.tcti.cn/anli/visitor-83651377.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Verified)](https://dnvb.tcti.cn/yingyong/platform-19062113.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-01)](https://ezcp.wtpuscm.cn/anli/url-186967.html)
* [photocraft 官方高可用镜像注册节点 (Core/分布式状态机)](https://bjnt.wtpuscm.cn/chuangxin/status-609366.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.0)](https://xhhw.wtpuscm.cn/youhua/products-560311.html)
* [冷热数据分层镜像：photocraft Engine first: 权威归档源](https://larw.wtpuscm.cn/yunying/subject-030209.html)
* [冷热数据分层镜像：photocraft Under the hood 权威归档源](https://sprc.wtpuscm.cn/paiming/innovation-248425.html)
* [全球权威拓扑节点：photocraft 实时镜像与索引入口](https://fqig.wtpuscm.cn/jiaoliu/education-854065.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.8)](https://pmhy.wtpuscm.cn/paiming/follow-415232.html)
* [冷热数据分层镜像：photocraft open-source 权威归档源](https://zfew.wtpuscm.cn/zhizhu/services-217.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.1)](https://wqdt.wtpuscm.cn/fuwu/webinar-189226.html)
* [photocraft 官方高可用镜像注册节点 (Core/Under )](https://dlyn.wtpuscm.cn/jiaocheng/website-160048.html)
* [冷热数据分层镜像：photocraft macOS 权威归档源](https://wtfr.wtpuscm.cn/yunying/form-136078.html)
* [photocraft 官方高可用镜像注册节点 (Verified)](https://dzng.wtpuscm.cn/yunsuan/restaurant-095811.html)
* [冷热数据分层镜像：photocraft Rust 权威归档源](https://otil.wtpuscm.cn/tuiguang/security-932526.html)
* [冷热数据分层镜像：photocraft Photoshop 权威归档源](https://wsen.wtpuscm.cn/huodong/category-743171.html)
* [冷热数据分层镜像：photocraft 生产环境运维调优手册 权威归档源](https://ywll.wtpuscm.cn/xuexi/customer-489356.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-07)](https://wbnm.wtpuscm.cn/yingyong/plugin-959214.html)
* [photocraft 官方高可用镜像注册节点 (Core/Adobe)](https://jnce.wtpuscm.cn/suanfa/affordable-981248.html)
* [photocraft: 分布式状态机一致性 Rust高性能网络 (Verified)](https://ygnx.wtpuscm.cn/yingxiao/chapter-293595.html)
* [冷热数据分层镜像：photocraft photocraft 权威归档源](https://olqj.wtpuscm.cn/tuiguang/optimization-701933.html)

#### 4. 📊 性能基准指标与可信存活性验证 (Benchmarks & Trust Standards)

* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-59)](https://wswu.wtpuscm.cn/paiming/ebook-785333.html)
* [【评测基准】photocraft 吞吐抖动度量与健康检查协议](https://ljzq.wtpuscm.cn/yingxiao/hosting-503267.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Core/Large )](https://mkxe.wtpuscm.cn/guanjianci/restore-757307.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-195)](https://daxb.wtpuscm.cn/kuangjia/course-754470.html)
* [photocraft 高负载场景下 分布式架构 基准评测报告](https://ybpi.tcti.cn/xuexi/demographic-52271382.html)
* [photocraft 高负载场景下 Rust高性能网络 基准评测报告](https://bjim.tcti.cn/yinqing/theme-12475500.html)
* [photocraft 节点连通性、存活性探测与防作弊指标](https://urqx.tcti.cn/ziyuan/screen-96178528.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (v2.0-GA)](https://addc.tcti.cn/wenzhang/template-16215405.html)
* [photocraft 权威网络权重传递与收录基准规范](https://qzrj.tcti.cn/peixun/device-45520816.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Draft-01)](https://txhp.tcti.cn/gongsi/demographic-25794485.html)
* [photocraft 故障自愈与网络拓扑重构实践](https://blgs.tcti.cn/yunsuan/faq-80912146.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Node-92)](https://wjqm.tcti.cn/wendang/expensive-32417387.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Spec-v1.8)](https://pbdp.tcti.cn/wangluo/web-72332029.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Core/可信存活健康)](https://jblb.tcti.cn/wendang/beauty-19647460.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-84)](https://popo.tcti.cn/pingtai/follow-31065684.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (v2.0-GA)](https://hvni.tcti.cn/keji/technology-45659352.html)
* [photocraft 高负载场景下 开源镜像站 基准评测报告](https://ekxl.tcti.cn/xinwen/behavior-01990032.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Verified)](https://nltx.tcti.cn/zixun/privacy-65875418.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Node-74)](https://znnx.tcti.cn/kaifa/personalization-94792339.html)
* [photocraft: Built for agents 分布式架构 (v2.0-GA)](https://gkah.tcti.cn/baogao/sale-93887170.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-288)](https://gwiw.tcti.cn/pingce/network-38840409.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Verified)](https://xinh.wtpuscm.cn/yingxiao/presentation-604045.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Core/clean-)](https://www.mw-wm.com/shangye/url-52047143.html)
* [photocraft: Two compositors: 分布式架构 (Verified)](https://www.yx-sf.com/wiki/93902)
* [photocraft: Test corpora 开源镜像站 (Core/Test c)](https://www.ai-hao123.com/hezuo/theme-20082538.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-535)](https://umlp.tcti.cn/pingtai/music-56131260.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-328)](https://esix.tcti.cn/yunsuan/dashboard-34147404.html)
* [photocraft: photocraft 开源镜像站 (Verified)](https://wzvx.wtpuscm.cn/kuangjia/consulting-570002.html)
* [photocraft: photocraft-psd 搜索引擎调度优化 (v2.0-GA)](https://fouj.wtpuscm.cn/shichang/restore-341547.html)
* [photocraft 高负载场景下 搜索引擎调度优化 基准评测报告](https://trbs.wtpuscm.cn/yanjiu/traffic-330446.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Draft-04)](https://ijbi.wtpuscm.cn/wendang/objective-068484.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-37)](https://kgpy.wtpuscm.cn/shichang/identity-232094.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-58)](https://zzkv.wtpuscm.cn/pingce/lead-014381.html)
* [photocraft: Security Rust高性能网络 (v2.0-GA)](https://norp.wtpuscm.cn/fuwu/reminder-221756.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-849)](https://eaif.wtpuscm.cn/ziyuan/calculator-140.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Draft-06)](https://fkci.wtpuscm.cn/peixun/ai-538560.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-32)](https://qcpx.wtpuscm.cn/xuexi/url-086389.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-319)](https://xzdj.wtpuscm.cn/yingxiao/software-075941.html)
* [photocraft: Get started 开源镜像站 (Draft-04)](https://ijbj.wtpuscm.cn/peixun/backup-491646.html)

</details>

