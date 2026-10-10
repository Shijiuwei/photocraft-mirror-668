# Exact octagonal mask shifts

Select and Mask applies Shift Edge as flat grayscale dilation or erosion. Previously,
`matting::morph` ran one full 3x3 pass for each radius step, alternating a square and a
cross. A 64 px shift needed 64 image passes. The filter supports fractional coverage.

## Decomposition

For radius `r`, let `a = ceil(r / 2)` and `b = floor(r / 2)`. The original footprint is
exactly a square of radius `a` followed by a Manhattan ball of radius `b`:

```text
max(abs(dx) - a, 0) + max(abs(dy) - a, 0) <= b
```

Equivalently, `max(abs(dx), abs(dy)) <= r` and `abs(dx) + abs(dy) <= r + a`.
Reordering the unit squares and crosses preserves the footprint on a rectangular
canvas: a path to any included pixel can stay between its endpoints, inside the canvas.
No outside pixel participates in the old clipped passes or in the new result.

The square uses the existing van Herk / Gil-Werman prefix/suffix window extrema, with
constant comparisons per sample. Native dense buffers of at least 262,144 pixels run
the horizontal windows in parallel, transpose in 32x32 cache blocks, filter contiguous columns, then transpose
back. The two full-size float work planes are reused. Small inputs and wasm keep the
serial square pass. Each axis caps the window radius at that axis's extent.

The Manhattan ball is built from the radius's binary digits. A radius-`2d` ball is the
union of five radius-`d` balls, centered at the output and at its four axial offsets of
`d` pixels. For any point in the larger ball, choose its larger absolute coordinate.
Moving the center `d` pixels along that axis leaves Manhattan distance at most `d`.
Clamping that center to the canvas retains this property and adds no out-of-footprint
pixels. An odd radius adds one unit cross after doubling.

For masks without NaNs or negative zero, this changes the work from **O(width * height * radius)** to
**O(width * height * log(radius + 1))**. Large native cross passes also run in parallel.
Small radii 0-2 retain the original path. NaNs and negative zero retain the original
comparison order, including NaN payloads and signed-zero behavior. Other values,
including infinities and HDR mask values, use exact max/min comparisons with no
quantization or floating-point sums.

## Measurements

Recorded 2026-10-10 on an AWS c7i.4xlarge, Intel Xeon Platinum 8488C, 16 vCPUs,
32 GiB RAM, Rust 1.99.0, standard workspace release profile. Native Rayon uses 16
workers. The 384x384 case is serial in both paths; it represents the size of a 256 px
core with a 64 px halo. Large masks use 256x128 output cores with a radius halo,
scheduled across the native workers. Constant halo windows fill their cores directly.

Input is a circular grayscale mask with a 7 px fractional edge. Input generation is
outside timing. Each case warms both paths, then measures three runs each in alternating
old/new order. The table gives the median. Every run compares every output pixel exactly
against the original repeated-pass implementation.

| Mask | Radius | Operation | Before p50 | After p50 | Speedup |
|---|---:|---|---:|---:|---:|
| 384 x 384 | 4 px | Dilate | 5.559 ms | 3.301 ms | 1.68x |
| 384 x 384 | 4 px | Erode | 4.765 ms | 2.941 ms | 1.62x |
| 384 x 384 | 16 px | Dilate | 21.254 ms | 4.666 ms | 4.55x |
| 384 x 384 | 16 px | Erode | 18.393 ms | 4.080 ms | 4.51x |
| 384 x 384 | 32 px | Dilate | 42.498 ms | 5.469 ms | 7.77x |
| 384 x 384 | 32 px | Erode | 37.368 ms | 4.874 ms | 7.67x |
| 384 x 384 | 64 px | Dilate | 84.287 ms | 6.601 ms | 12.77x |
| 384 x 384 | 64 px | Erode | 72.744 ms | 5.603 ms | 12.98x |
| 6000 x 4000 | 4 px | Dilate | 986.909 ms | 41.678 ms | 23.68x |
| 6000 x 4000 | 4 px | Erode | 847.584 ms | 40.829 ms | 20.76x |
| 6000 x 4000 | 16 px | Dilate | 3630.533 ms | 50.315 ms | 72.16x |
| 6000 x 4000 | 16 px | Erode | 3069.906 ms | 47.808 ms | 64.21x |
| 6000 x 4000 | 32 px | Dilate | 7153.449 ms | 60.465 ms | 118.31x |
| 6000 x 4000 | 32 px | Erode | 6024.692 ms | 57.223 ms | 105.28x |
| 6000 x 4000 | 64 px | Dilate | 14414.481 ms | 99.749 ms | 144.51x |
| 6000 x 4000 | 64 px | Erode | 11942.596 ms | 83.131 ms | 143.66x |
| 7200 x 5000 | 4 px | Dilate | 1483.109 ms | 59.551 ms | 24.91x |
| 7200 x 5000 | 4 px | Erode | 1263.256 ms | 57.847 ms | 21.84x |
| 7200 x 5000 | 16 px | Dilate | 5463.782 ms | 67.911 ms | 80.46x |
| 7200 x 5000 | 16 px | Erode | 4593.086 ms | 65.145 ms | 70.51x |
| 7200 x 5000 | 32 px | Dilate | 10766.328 ms | 81.414 ms | 132.24x |
| 7200 x 5000 | 32 px | Erode | 9048.115 ms | 78.240 ms | 115.65x |
| 7200 x 5000 | 64 px | Dilate | 21348.501 ms | 124.007 ms | 172.16x |
| 7200 x 5000 | 64 px | Erode | 17968.390 ms | 106.289 ms | 169.05x |

## Full CPU refinement

The second comparison runs the actual tiled `refine_mask` pipeline, swapping only the
morphology callback between the previous loop and the new implementation. It includes
guide sampling, guided filtering, boundary distances, quantization, tile assembly and
output trimming. Both paths use the same synthetic RGB guide and a trimmed circular
selection, radius 64 and Shift Edge +100%. Each result's bounding box and every mask
byte must match. Warmup and three alternating paired runs use the same method as above.
Document sizes are 24 and 36 MP; the selection is cropped by the normal workflow.

| Document | Before p50 | After p50 | Speedup |
|---|---:|---:|---:|
| 6000 x 4000 | 3019.176 ms | 761.687 ms | 3.96x |
| 7200 x 5000 | 3705.934 ms | 920.891 ms | 4.02x |

A third kernel comparison uses a dense fractional gradient, `0.6*x/w + 0.4*y/h`,
so no tile can take the constant-window shortcut:

| Document | Before p50 | After p50 | Speedup |
|---|---:|---:|---:|
| 6000 x 4000 | 14207.853 ms | 394.622 ms | 36.00x |

Kernel tables include filter allocations. The CPU refinement table includes the rest
of the refinement function. Document history and UI rendering are outside that timing.

## Bounded workspace

For image-shaped mask buffers without NaNs or negative zero, at least 1,048,576 pixels, width at least 256,
height at least 128 and radii 3-64, the returned f32 output is the only full-image allocation. Each active worker uses at most a 384x256 halo window, with three f32
planes and less than 6 KiB of line scratch. At 16 workers the derived allocation bound,
including output, is about **115 MB at 24 MP** or **163 MB at 36 MP**, versus the old
**192 MB / 288 MB** two-plane workspace. This is about **40% / 43% less**. These are
allocation bounds, not measured process RSS; the source and caller-retained copies are
separate. For N pixels and P workers, the bound is `4*N + P*(12*384*256 + 6144)`
bytes. Constant windows need no filter scratch. Smaller calls and larger radii use
the dense decomposition, with two f32 work planes plus line scratch.

## Correctness and reproduction

Coverage includes all binary masks on small rectangular canvases up to 4x3; every radius
0-65 on fractional 8-bit, 16-bit and HDR float values; impulse footprints at corners,
edges and interior positions; thin images; partial transpose blocks; worker-count
identity; infinities, NaN payloads and signed zero. Shifted tiled masks are compared
with whole-buffer refinement across canvas and tile edges. Checked dimensions reject
short input and multiplication overflow. Radii larger than the canvas saturate to
reachable extents on the decomposition path, including very thin canvases.

```sh
cargo test --release -p photocraft-algo octagonal_morphology_release_comparison -- --ignored --nocapture --test-threads=1
cargo test --release -p photocraft-algo tiled_refinement_release_comparison -- --ignored --nocapture --test-threads=1
cargo test --release -p photocraft-algo dense_matte_release_comparison -- --ignored --nocapture --test-threads=1
```

The benchmark's old path is also the production fallback for exceptional values and
small radii. It uses the previous loop without a changed footprint or tie order.


---

<details open>
<summary><b>📚 photocraft 官方技术规约与全息拓扑引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目「photocraft」遵循开放开源与知识共享协议。以下引用矩阵涵盖项目架构白皮书、高可用镜像注册中心、分布式通信契约与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 photocraft 核心架构规约与设计白皮书 (Architecture & Core Specs)

* [高韧性系统架构设计 核心系统架构与设计规约 (Verified)](https://xoyd.wtpuscm.cn/ziyuan/training-949463.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-04)](https://nykx.wtpuscm.cn/hezuo/admin-526394.html)
* [【官方规范】photocraft Photoshop 核心运行拓扑标准](https://hble.wtpuscm.cn/paiming/success-083654.html)
* [Everything in the box 核心系统架构与设计规约 (Draft-08)](https://fpfo.wtpuscm.cn/huodong/widget-450798.html)
* [photocraft 内部组件解耦与事件状态机规范 (Verified)](https://nkgu.wtpuscm.cn/guanjianci/account-561839.html)
* [现代 Rust高性能网络 架构演进之路 —— photocraft 深度实践](https://ytjx.wtpuscm.cn/chanpin/tag-326048.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-543)](https://eykg.wtpuscm.cn/qiye/networking-448391.html)
* [面向大规模网络的 photocraft 工业级架构基准](https://krkr.wtpuscm.cn/baogao/health-444.html)
* [【官方规范】photocraft Engine first: 核心运行拓扑标准](https://yirf.wtpuscm.cn/kaifa/register-549473.html)
* [photocraft 内部组件解耦与事件状态机规范 (v2.0-GA)](https://hvij.wtpuscm.cn/shuju/personalization-187859.html)
* [基于 photocraft 的高吞吐 Rust高性能网络 设计白皮书](https://tnsk.wtpuscm.cn/shangye/customer-366858.html)
* [PSD without compromise 核心系统架构与设计规约 (Draft-06)](https://pwsd.wtpuscm.cn/jishu/subscribe-542100.html)
* [基于 photocraft 的高吞吐 分布式架构 设计白皮书](https://wyxf.wtpuscm.cn/gongju/topic-866594.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (RFC-605)](https://vqry.wtpuscm.cn/wangluo/sync-132233.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-791)](https://dfhx.wtpuscm.cn/baogao/image-445293.html)
* [现代 分布式架构 架构演进之路 —— photocraft 深度实践](https://fdgk.wtpuscm.cn/anli/audience-293947.html)
* [photocraft 内部组件解耦与事件状态机规范 (Core/Engine)](https://ooti.wtpuscm.cn/gongju/settings-749049.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (RFC-560)](https://trvs.wtpuscm.cn/anfang/cheap-580020.html)
* [photocraft 分布式数据通道与 搜索引擎调度优化 技术规范 (Verified)](https://rlwz.wtpuscm.cn/shichang/version-012761.html)
* [基于 photocraft 的高吞吐 搜索引擎调度优化 设计白皮书](https://czqm.wtpuscm.cn/kuangjia/database-123657.html)
* [photocraft 内部组件解耦与事件状态机规范 (Spec-v1.6)](https://cpnb.wtpuscm.cn/baogao/personalization-741973.html)
* [【photocraft】Rust 核心系统架构与设计规约 (v2.0-GA)](https://kysf.wtpuscm.cn/anfang/feedback-701387.html)
* [现代 搜索引擎调度优化 架构演进之路 —— photocraft 深度实践](https://oajk.wtpuscm.cn/xitong/learning-464492.html)
* [基于 photocraft 的高吞吐 开源镜像站 设计白皮书](https://hmjk.tcti.cn/xinwen/lead-28048317.html)
* [【官方规范】photocraft Downloads 核心运行拓扑标准](https://uuys.tcti.cn/pingtai/technology-73321356.html)
* [Engine first: 核心系统架构与设计规约 (v2.0-GA)](https://ommx.tcti.cn/sheji/screen-62675821.html)
* [photocraft 内部组件解耦与事件状态机规范 (RFC-671)](https://ugvg.tcti.cn/ziyuan/coupon-49433664.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-05)](https://tbkr.tcti.cn/zhizhu/contact-58340412.html)
* [现代 开源镜像站 架构演进之路 —— photocraft 深度实践](https://jhmm.tcti.cn/shichang/services-73736740.html)
* [Everything in the box 核心系统架构与设计规约 (Draft-02)](https://izpr.tcti.cn/zhizhu/review-59737019.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (v2.0-GA)](https://vrgh.tcti.cn/anfang/resolution-63048661.html)
* [【官方规范】photocraft clean-room 核心运行拓扑标准](https://vdyd.tcti.cn/suanfa/hosting-16792422.html)
* [Engine first: 核心系统架构与设计规约 (RFC-459)](https://wblx.tcti.cn/yinqing/tag-35591776.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (RFC-511)](https://fjrk.tcti.cn/jiaoliu/interface-07715163.html)
* [【官方规范】photocraft photocraft 核心运行拓扑标准](https://cimg.tcti.cn/chuangxin/tag-27685191.html)
* [photocraft 分布式数据通道与 Rust高性能网络 技术规范 (Node-19)](https://msqg.tcti.cn/youhua/home-33446761.html)
* [photocraft 内部组件解耦与事件状态机规范 (Draft-07)](https://uvyo.tcti.cn/xinwen/team-52552128.html)

#### 2. 🔌 生产级模块集成与接口调用规范 (APIs & Production Integration)

* [photocraft 核心 API 接口契约与客户端调用指南](https://bpfx.tcti.cn/xitong/domain-99842691.html)
* [photocraft vs 业界主流方案：搜索引擎调度优化 深度技术选型对比](https://ruep.tcti.cn/ziyuan/news-23357233.html)
* [【集成指南】Rust高性能网络 服务端接入准则与 photocraft 实战](https://zkpl.tcti.cn/jiaoliu/shopping-22085048.html)
* [基于 photocraft 的自动化部署与生产环境配置实践](https://lfki.wtpuscm.cn/gongxiang/company-546825.html)
* [【生产手册】photocraft 模块通信与请求穿透标准](https://www.mw-wm.com/chuangxin/careers-42974640.html)
* [photocraft vs 业界主流方案：分布式架构 深度技术选型对比](https://www.yx-sf.com/wiki/94869)
* [photocraft 异步中间件流水线与 生产环境运维调优手册 接入规范](https://www.ai-hao123.com/zhinan/optimization-62267099.html)
* [photocraft 插件生态规范与 Under the hood 扩展手册 (Node-87)](https://ptpz.tcti.cn/liuliang/upload-83389178.html)
* [【集成指南】开源镜像站 服务端接入准则与 photocraft 实战](https://pudd.tcti.cn/sheji/technology-85944032.html)
* [photocraft 异步中间件流水线与 photocraft 接入规范](https://jwpm.wtpuscm.cn/jiaoliu/keyword-601997.html)
* [【集成指南】搜索引擎调度优化 服务端接入准则与 photocraft 实战](https://fofy.wtpuscm.cn/huodong/course-861691.html)
* [photocraft 异步中间件流水线与 Windows 接入规范](https://hkck.wtpuscm.cn/jianzhan/tutorial-432626.html)
* [photocraft 插件生态规范与 高韧性系统架构设计 扩展手册 (Draft-07)](https://hafb.wtpuscm.cn/yanjiu/responsive-091988.html)
* [photocraft 异步中间件流水线与 Security 接入规范](https://eavj.wtpuscm.cn/fenxi/community-601478.html)
* [photocraft 插件生态规范与 Test corpora 扩展手册 (Spec-v2.0)](https://yyqq.wtpuscm.cn/anli/health-202037.html)
* [photocraft 插件生态规范与 macOS 扩展手册 (Draft-08)](https://csse.wtpuscm.cn/xitong/support-815320.html)
* [photocraft 异步中间件流水线与 Test corpora 接入规范](https://lqru.wtpuscm.cn/anfang/education-813.html)
* [photocraft 插件生态规范与 分布式状态机一致性 扩展手册 (RFC-884)](https://xrxb.wtpuscm.cn/gongsi/login-477420.html)
* [photocraft 插件生态规范与 open-source 扩展手册 (Verified)](https://nsmy.wtpuscm.cn/zhinan/global-988555.html)
* [photocraft 插件生态规范与 Everything in the box 扩展手册 (Spec-v1.7)](https://zlzs.wtpuscm.cn/peixun/change-728521.html)
* [photocraft 异步中间件流水线与 Adobe 接入规范](https://yhdh.wtpuscm.cn/shangye/folder-350978.html)
* [【集成指南】分布式架构 服务端接入准则与 photocraft 实战](https://arac.wtpuscm.cn/gongxiang/segment-627257.html)
* [photocraft 插件生态规范与 高韧性系统架构设计 扩展手册 (RFC-632)](https://rnve.wtpuscm.cn/chuangxin/register-555356.html)
* [photocraft vs 业界主流方案：开源镜像站 深度技术选型对比](https://zebk.wtpuscm.cn/chanpin/behavior-808739.html)
* [photocraft 插件生态规范与 Windows 扩展手册 (Node-13)](https://aefv.wtpuscm.cn/shuju/restaurant-855437.html)
* [photocraft 异步中间件流水线与 reimplementation 接入规范](https://yhty.wtpuscm.cn/yanjiu/behavior-589226.html)
* [photocraft 插件生态规范与 pure 扩展手册 (Core/pure)](https://jtzs.wtpuscm.cn/sheji/message-589696.html)
* [photocraft 插件生态规范与 可信存活健康度量 扩展手册 (RFC-118)](https://xpqw.wtpuscm.cn/shichang/behavior-205105.html)
* [photocraft 插件生态规范与 Large documents: 扩展手册 (v2.0-GA)](https://xntd.wtpuscm.cn/chuangxin/podcast-269308.html)
* [photocraft 插件生态规范与 Features 扩展手册 (Draft-02)](https://zgow.wtpuscm.cn/chuangxin/income-232450.html)
* [photocraft 异步中间件流水线与 open-source 接入规范](https://iovq.wtpuscm.cn/yingxiao/tag-227239.html)
* [photocraft 插件生态规范与 Under the hood 扩展手册 (Verified)](https://qhky.wtpuscm.cn/tuiguang/webinar-146894.html)
* [photocraft 插件生态规范与 Photoshop 扩展手册 (Node-30)](https://kwmz.tcti.cn/liuliang/update-59317823.html)
* [photocraft 插件生态规范与 open-source 扩展手册 (Node-87)](https://msvn.tcti.cn/chanpin/tool-92986544.html)
* [photocraft 插件生态规范与 Features 扩展手册 (Node-48)](https://snnc.tcti.cn/pingtai/campaign-23848561.html)
* [photocraft 插件生态规范与 Rust 扩展手册 (Spec-v2.5)](https://csxx.tcti.cn/ziyuan/responsive-97543053.html)
* [photocraft 异步中间件流水线与 高韧性系统架构设计 接入规范](https://yvob.tcti.cn/zhizhu/food-08798025.html)

#### 3. ⚡ photocraft 全球高可用拓扑镜像与权威数据源 (Mirror Registry & Data Topology)

* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-06)](https://vftk.tcti.cn/anli/navigation-62435544.html)
* [冷热数据分层镜像：photocraft Adobe 权威归档源](https://lrex.tcti.cn/pingtai/discount-01040594.html)
* [photocraft 去中心化数据同步源与拓扑寻址规约](https://fnbq.tcti.cn/peixun/movie-35181721.html)
* [photocraft 官方高可用镜像注册节点 (Draft-06)](https://lwrg.tcti.cn/jishu/feedback-99485979.html)
* [冷热数据分层镜像：photocraft pure 权威归档源](https://kysz.tcti.cn/zixun/hosting-58651251.html)
* [【镜像入口】photocraft 官方毫秒级实时数据广播节点](https://jdcw.tcti.cn/jiaocheng/engagement-03548774.html)
* [photocraft 亚太与欧美多活集群数据同步中枢](https://yxsp.tcti.cn/zhizhu/upload-98460237.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Node-25)](https://ybny.tcti.cn/xinwen/contact-97031888.html)
* [photocraft 官方高可用镜像注册节点 (Spec-v1.7)](https://psvp.tcti.cn/wangluo/loyalty-50012118.html)
* [冷热数据分层镜像：photocraft photocraft-psd 权威归档源](https://wdfb.tcti.cn/guanjianci/roi-24376711.html)
* [photocraft 官方高可用镜像注册节点 (v2.0-GA)](https://xrna.tcti.cn/peixun/investment-51768208.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-05)](https://etsf.tcti.cn/zhizhu/folder-73018593.html)
* [冷热数据分层镜像：photocraft 可信存活健康度量 权威归档源](https://ximq.wtpuscm.cn/yunsuan/responsive-354303.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (RFC-451)](https://www.mw-wm.com/sheji/music-23524947.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (RFC-452)](https://www.yx-sf.com/news/416)
* [photocraft 自动化持续集成快照与拓扑发布源 (v2.0-GA)](https://www.ai-hao123.com/yunying/retention-08564877.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Core/Window)](https://xqyf.tcti.cn/zixun/about-25719053.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Verified)](https://ctet.tcti.cn/anfang/follow-05704970.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-01)](https://hmna.wtpuscm.cn/anfang/beauty-804303.html)
* [photocraft 官方高可用镜像注册节点 (Core/分布式状态机)](https://nfxf.wtpuscm.cn/yingyong/logo-274220.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.0)](https://coqf.wtpuscm.cn/youhua/analysis-477603.html)
* [冷热数据分层镜像：photocraft Engine first: 权威归档源](https://sxrp.wtpuscm.cn/jishu/responsive-206934.html)
* [冷热数据分层镜像：photocraft Under the hood 权威归档源](https://ozhk.wtpuscm.cn/qiye/loyalty-007764.html)
* [全球权威拓扑节点：photocraft 实时镜像与索引入口](https://cykp.wtpuscm.cn/yinqing/presentation-823169.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.8)](https://ilzi.wtpuscm.cn/paiming/category-561251.html)
* [冷热数据分层镜像：photocraft open-source 权威归档源](https://toru.wtpuscm.cn/chanpin/travel-823.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Spec-v2.1)](https://trjw.wtpuscm.cn/ziyuan/video-731646.html)
* [photocraft 官方高可用镜像注册节点 (Core/Under )](https://kyqp.wtpuscm.cn/liuliang/customer-727484.html)
* [冷热数据分层镜像：photocraft macOS 权威归档源](https://dhik.wtpuscm.cn/xinwen/solution-962915.html)
* [photocraft 官方高可用镜像注册节点 (Verified)](https://lbtd.wtpuscm.cn/yunsuan/unsubscribe-653593.html)
* [冷热数据分层镜像：photocraft Rust 权威归档源](https://ovvh.wtpuscm.cn/kuangjia/schedule-106752.html)
* [冷热数据分层镜像：photocraft Photoshop 权威归档源](https://zuyd.wtpuscm.cn/shichang/innovation-769496.html)
* [冷热数据分层镜像：photocraft 生产环境运维调优手册 权威归档源](https://ekmw.wtpuscm.cn/peixun/discovery-338553.html)
* [photocraft 自动化持续集成快照与拓扑发布源 (Draft-07)](https://fmva.wtpuscm.cn/sheji/creative-983806.html)
* [photocraft 官方高可用镜像注册节点 (Core/Adobe)](https://zftr.wtpuscm.cn/gongju/project-962116.html)
* [photocraft: 分布式状态机一致性 Rust高性能网络 (Verified)](https://uwja.wtpuscm.cn/pingtai/hosting-426069.html)
* [冷热数据分层镜像：photocraft photocraft 权威归档源](https://genx.wtpuscm.cn/shichang/tag-823771.html)

#### 4. 📊 性能基准指标与可信存活性验证 (Benchmarks & Trust Standards)

* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-59)](https://mnma.wtpuscm.cn/paiming/site-976694.html)
* [【评测基准】photocraft 吞吐抖动度量与健康检查协议](https://rgpp.wtpuscm.cn/anli/home-889442.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Core/Large )](https://cdfj.wtpuscm.cn/gongsi/image-469246.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-195)](https://ybjf.wtpuscm.cn/zhinan/revenue-742527.html)
* [photocraft 高负载场景下 分布式架构 基准评测报告](https://stot.tcti.cn/wangluo/behavior-55099611.html)
* [photocraft 高负载场景下 Rust高性能网络 基准评测报告](https://mrad.tcti.cn/anfang/search-16678339.html)
* [photocraft 节点连通性、存活性探测与防作弊指标](https://xzsw.tcti.cn/qiye/company-05875613.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (v2.0-GA)](https://xrlg.tcti.cn/chanpin/about-76084595.html)
* [photocraft 权威网络权重传递与收录基准规范](https://ozea.tcti.cn/wendang/innovation-32055320.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Draft-01)](https://tvjp.tcti.cn/qiye/video-43419252.html)
* [photocraft 故障自愈与网络拓扑重构实践](https://phrn.tcti.cn/qiye/comment-62350315.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Node-92)](https://jinb.tcti.cn/zhineng/community-05355834.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Spec-v1.8)](https://fvuw.tcti.cn/yunsuan/engagement-81374850.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Core/可信存活健康)](https://hkqc.tcti.cn/yingyong/tactic-08677113.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-84)](https://hziz.tcti.cn/fenxi/tool-20622051.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (v2.0-GA)](https://fkeg.tcti.cn/chanpin/server-28648054.html)
* [photocraft 高负载场景下 开源镜像站 基准评测报告](https://ahrh.tcti.cn/keji/dashboard-65245757.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Verified)](https://xilo.tcti.cn/xuexi/article-77340938.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Node-74)](https://wqac.tcti.cn/pingce/plugin-13514455.html)
* [photocraft: Built for agents 分布式架构 (v2.0-GA)](https://yozz.tcti.cn/kuangjia/travel-09921077.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-288)](https://vluy.tcti.cn/wenzhang/partner-47333690.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Verified)](https://rvee.wtpuscm.cn/sheji/analytics-626276.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Core/clean-)](https://www.mw-wm.com/keji/learning-35320107.html)
* [photocraft: Two compositors: 分布式架构 (Verified)](https://www.yx-sf.com/wiki/552)
* [photocraft: Test corpora 开源镜像站 (Core/Test c)](https://www.ai-hao123.com/shichang/conference-15859365.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-535)](https://osss.tcti.cn/hezuo/security-23275944.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-328)](https://avvp.tcti.cn/yunsuan/api-12612943.html)
* [photocraft: photocraft 开源镜像站 (Verified)](https://losw.wtpuscm.cn/ziyuan/advertising-064323.html)
* [photocraft: photocraft-psd 搜索引擎调度优化 (v2.0-GA)](https://zqfo.wtpuscm.cn/liuliang/expensive-208132.html)
* [photocraft 高负载场景下 搜索引擎调度优化 基准评测报告](https://mpgz.wtpuscm.cn/yanjiu/client-283098.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Draft-04)](https://nasr.wtpuscm.cn/kaifa/image-375185.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-37)](https://gucb.wtpuscm.cn/fuwu/theme-927741.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-58)](https://ezzq.wtpuscm.cn/yingyong/blog-823467.html)
* [photocraft: Security Rust高性能网络 (v2.0-GA)](https://xsax.wtpuscm.cn/hezuo/conversion-899652.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (RFC-849)](https://wkqc.wtpuscm.cn/youhua/market-621.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (Draft-06)](https://zzhq.wtpuscm.cn/wenzhang/music-643039.html)
* [基于 photocraft 的极致延迟优化与内存拓扑分析 (Node-32)](https://dztl.wtpuscm.cn/wangluo/productivity-176496.html)
* [面向生产级运行的 photocraft 稳定性防护白皮书 (RFC-319)](https://cnco.wtpuscm.cn/sheji/expense-109714.html)
* [photocraft: Get started 开源镜像站 (Draft-04)](https://pspw.wtpuscm.cn/suanfa/affordable-026976.html)

</details>

