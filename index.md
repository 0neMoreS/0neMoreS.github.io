---
layout: default
---

<h2 id="restir"><span class="project-number">01</span>ReSTIR · SPMIS</h2>

<div class="project-meta"><span>腾讯</span><span>引擎开发实习 · 2026.06 - 2026.09</span></div>

在 RTXDI 框架中复现 Stochastic Pairwise MIS（SPMIS）空间重采样，并构建等样本、等半径的对照评测。从 GRIS 的理论出发分析样本复用的有效性，再借助 Compatibility-Guided Neighbor Selection（CGNS）筛选复用 cell，最后将方法迁移至 ReSTIR GI。

<div class="result-grid">
  <div class="result-item"><strong>35.7<span>%</span></strong><p>DI 的 MAPE 降低<br><small>SPMIS 相比 Pairwise</small></p></div>
  <div class="result-item"><strong>8.4<span>%</span></strong><p>DI 的 MAPE 进一步降低<br><small>SPMIS + CGNS 相比 SPMIS</small></p></div>
  <div class="result-item"><strong>15.1<span>%</span></strong><p>GI 的 MAPE 累计降低<br><small>SPMIS + CGNS 相比 Pairwise</small></p></div>
</div>

### Direct Illumination 复现

在相同样本数和复用半径下，对比 Pairwise 与 SPMIS 的噪声和误差。通过共享 cell 中的候选样本，验证 SPMIS 空间重采样的效果，而不是通过增加样本预算获得收益。

<figure class="evidence-figure">
  <img src="./assets/restir-di-spmis.jpg" alt="Bistro 直接光对照：左侧为 Ground Truth，右侧为 GT、Pairwise 和 SPMIS 的局部放大图，MAPE 分别为 1.104 和 0.710。" loading="lazy" width="1080" height="451">
  <figcaption><span>DI / Pairwise → SPMIS</span></figcaption>
</figure>

### Compatibility-Guided 优化

复用样本经过 Shift Mapping 后，可能因为像素间几何差异失效。基于 GRIS 的理论基础，分析发现如果能尽量保证 Shift Mapping 前后的样本一致性，就能获得更好的结果。利用 CGNS 的兼容性启发式筛选复用 cell，就能很好地保证 Shift Mapping 前后的样本一致性，减少无效复用带来的方差。

<figure class="evidence-figure">
  <img src="./assets/restir-di-cgns.jpg" alt="Bistro 直接光兼容性筛选对照：GT、SPMIS、SPMIS 加 CGNS，MAPE 从 0.7997 降至 0.7329。" loading="lazy" width="1080" height="451">
  <figcaption><span>DI / SPMIS → SPMIS + CGNS</span></figcaption>
</figure>

### Global Illumination 迁移

将 SPMIS 与 CGNS 从直接光重采样迁移到 ReSTIR GI，并沿用对照评测验证间接光的噪声与误差变化。下图展示室内场景中灯具及局部反射区域的结果。

<figure class="evidence-figure">
  <img src="./assets/restir-gi-cgns.jpg" alt="室内间接光对照：GT、Pairwise、SPMIS、SPMIS 加 CGNS，MAPE 分别为 1.1358、0.9820 和 0.9647。" loading="lazy" width="1080" height="432">
  <figcaption><span>GI / Pairwise → SPMIS → SPMIS + CGNS</span></figcaption>
</figure>

<h2 id="neural-bvh"><span class="project-number">02</span>Neural BVH</h2>

<div class="project-meta"><span>腾讯</span><span>引擎开发实习 · 2026.05 - 2026.06</span></div>

将论文原本基于 CUDA 的光追框架改造成基于 OptiX 的硬件光追版本，建立纯硬件求交的对照基线。针对首版神经求交在性能与画质上均不及纯硬件的问题，从求交精度和 Divergence 两方面分析瓶颈，将方案调整为主光线走硬件、仅在次级光线上对部分表面使用神经求交。

### 性能收益的上限

神经 AABB 候选收集不能像三角交点查询一样，利用已知最近命中的距离持续缩短遍历范围。分段计时显示，神经方案仅硬件遍历 BVH 的部分就已耗时 **1.5 ms**，与纯硬件完整求交的 **1.4 ms** 基本相当。

<figure class="evidence-figure">
  <img src="./assets/neural-bvh-study.jpg" loading="lazy" width="1080" height="608">
</figure>

<h2 id="myvk"><span class="project-number">03</span>MyVK</h2>

<div class="project-meta"><span>个人项目 · 2025.11 — 2026.04</span></div>

基于 Vulkan API 从零实现实时渲染器，覆盖光照与阴影、资源绑定、可见性剔除等功能，并围绕实际画面问题与 GPU 开销进行优化。

### Tiled Light Culling

若每个片元都遍历全部光源，着色开销会随光源数量膨胀。为此用 Compute Shader 预先构建各屏幕分块（Tile）的有效光源列表，片元着色阶段只遍历所在 Tile 的光源，减少无效光照计算。

<figure>
  <video controls preload="metadata" playsinline>
    <source src="./assets/myvk-light-culling.mp4" type="video/mp4">
  </video>
  <figcaption>Light Culling 测试场景：48 块平面，每块带有 25 个 Sphere Light</figcaption>
</figure>

相机沿动画路径移动，视野内光源数量从少量增加到全部，用于观察不同负载下的性能变化。

<div class="comparison-grid comparison-grid--two">
  <figure><img src="./assets/myvk-close-light-culling.png" alt="关闭 Tiled Light Culling 的性能记录。" loading="lazy" width="840" height="480"><figcaption>关闭 Light Culling</figcaption></figure>
  <figure><img src="./assets/myvk-open-light-culling.png" alt="开启 Tiled Light Culling 的性能记录。" loading="lazy" width="840" height="480"><figcaption>开启 Light Culling</figcaption></figure>
</div>

<div class="takeaway"><p>特性开关对照中，峰值 frame time 接近减半；视野内光源较少时最低可降至未开启时的约十分之一。测试场景下获得约 <strong>2 — 10 倍</strong>性能提升。</p></div>

### PCSS

Spot Light 与 Sphere Light 的阴影采用 PCSS：先通过 Blocker Search 估计遮挡体平均深度，再估算半影宽度，最后自适应调整 PCF 采样核。Blocker Search 与 PCF Filtering 均使用 20 次采样。

<figure><img src="./assets/myvk-No-Filtering.png" alt="无滤波阴影，边缘较硬。" loading="lazy" width="1922" height="1112"><figcaption>无滤波 / 阴影边缘过硬</figcaption></figure>
<figure><img src="./assets/myvk-PCF.png" alt="PCF 阴影，近处和远处统一模糊。" loading="lazy" width="1922" height="1112"><figcaption>PCF / 统一模糊阴影边缘</figcaption></figure>
<figure><img src="./assets/myvk-PCSS.png" alt="PCSS 阴影，近处锐利、远处柔和。" loading="lazy" width="1922" height="1112"><figcaption>PCSS / 近处更锐利，远处更柔和</figcaption></figure>

### Cascaded Shadow Map

Sun Light 使用 4 级 CSM。针对相机旋转引起的投影包围盒尺寸变化，以视锥体的外接包围球固定正交投影尺寸；针对相机平移产生的亚像素抖动，通过 Texel Snapping 将光源投影位移吸附到整数纹理像素步长。同时在相邻级联的交界处混合采样，平滑级联切换，并借助 RenderDoc 可视化验证。

<figure><video controls preload="metadata" playsinline><source src="./assets/myvk-cascade.mp4" type="video/mp4"></video><figcaption>相机运动时的级联过渡</figcaption></figure>
<figure><video controls preload="metadata" playsinline><source src="./assets/myvk-cascade-debug.mp4" type="video/mp4"></video><figcaption>Debug 视图 / Cascade 层级选择</figcaption></figure>

### PBR + IBL

基于 Karis Split-Sum 近似实现 IBL 预积分流程，包括 Diffuse Irradiance、Specular Prefilter 与 BRDF LUT。针对 GGX 重要性采样在高频区域产生的 Firefly，按样本的 Solid Angle 动态选择环境贴图 Mip 层级，抑制预滤波噪声。

<figure><video controls preload="metadata" playsinline><source src="./assets/myvk-pbr.mp4" type="video/mp4"></video><figcaption>PBR 着色管线</figcaption></figure>
<figure class="figure-compact"><img src="./assets/myvk-ibl.png" alt="IBL 预积分：Diffuse 项及不同 Roughness 下的 Specular 项。" loading="lazy" width="1504" height="3072"><figcaption>左侧为 Diffuse 项，后续为不同 Roughness 下的 Specular 项</figcaption></figure>

### Frustum Culling

基于相机 Frustum 与物体 AABB / OBB 进行相交测试，提前剔除视锥外几何体，减少 CPU Draw Call 提交与 GPU 无效开销。

<figure><video controls preload="metadata" playsinline><source src="./assets/myvk-culling.mp4" type="video/mp4"></video><figcaption>黄色：Frustum · 绿色：AABB · 蓝色：OBB · 红色：被剔除几何体</figcaption></figure>

### Bindless Texture

将逐材质纹理绑定改为动态纹理数组，通过 Push Constant 传递材质索引，在着色阶段按索引访问纹理，降低 descriptor 切换频率。

{% highlight glsl %}
layout(set=2, binding=1) uniform sampler2D Textures[];

layout(push_constant) uniform Push {
    uint MATERIAL_INDEX;
} push;
{% endhighlight %}

<h2 id="mypt"><span class="project-number">04</span>MyPT</h2>

<div class="project-meta"><span>个人项目 · 2025.03 — 2025.10</span></div>

基于现代 C++ 实现离线渲染系统，围绕 NEE、MIS 与 Photon Mapping 进行可验证实验，以 PBRT v3 的参考图像检验渲染结果。

### 架构搭建

<figure><img src="./assets/mypt-arch.svg" alt="MyPT 的渲染系统架构，参考 PBRT v3。" loading="lazy" width="1356" height="708"><figcaption>渲染系统架构 / 参考 PBRT v3</figcaption></figure>

### NEE + MIS

朴素路径追踪在漫反射表面依赖 BRDF 余弦采样，难以命中小光源，容易产生大量零贡献样本。引入 NEE 显式采样光源，再以 Balance Heuristic 按概率密度融合光源采样与 BRDF 采样，使两种策略互补。

<div class="comparison-grid comparison-grid--three">
  <figure><img src="./assets/mypt-nee-gt.png" alt="PBRT v3 的 Cornell Box Ground Truth。" loading="lazy" width="1024" height="1024"><figcaption><strong>Ground Truth</strong>PBRT v3</figcaption></figure>
  <figure><img src="./assets/mypt-nee-mis-256spp.png" alt="MyPT NEE 加 MIS，256 spp，relMSE 0.0028。" loading="lazy" width="1024" height="1024"><figcaption><strong>NEE + MIS</strong>256 spp · relMSE 0.0028</figcaption></figure>
  <figure><img src="./assets/mypt-naive-pt-256spp.png" alt="MyPT 朴素 Path Tracing，256 spp，relMSE 0.1613。" loading="lazy" width="1024" height="1024"><figcaption><strong>朴素 Path Tracing</strong>256 spp · relMSE 0.1613</figcaption></figure>
</div>

<div class="takeaway"><p>相同采样预算下，relMSE 降低 <strong>98.26%</strong>。</p></div>

### Photon Mapping

镜面聚焦产生的焦散属于高频光路，朴素路径追踪难以有效采到。实现两趟 Photon Mapping：从光源发射光子，分别构建全局光子图与焦散光子图；渲染时在漫反射表面做密度估计，以 K 近邻确定自适应核半径，缓解光子疏密不均时的噪点与模糊。

<div class="comparison-grid comparison-grid--three">
  <figure><img src="./assets/mypt-photon-gt.png" alt="PBRT v3 SPPM 的焦散场景 Ground Truth。" loading="lazy" width="1024" height="1024"><figcaption><strong>Ground Truth</strong>PBRT v3 · SPPM</figcaption></figure>
  <figure><img src="./assets/mypt-photon-20min.png" alt="MyPT Photon Mapping 焦散结果，渲染时间 20 分钟。" loading="lazy" width="1024" height="1024"><figcaption><strong>Photon Mapping</strong>20 min</figcaption></figure>
  <figure><img src="./assets/mypt-naive-pt-20min.png" alt="MyPT 朴素 Path Tracing 焦散结果，渲染时间 20 分钟。" loading="lazy" width="1024" height="1024"><figcaption><strong>朴素 Path Tracing</strong>20 min</figcaption></figure>
</div>

<h2 id="bernstein"><span class="project-number">05</span>Bernstein Bounds for Caustics</h2>

<div class="project-meta"><span>SIGGRAPH 2025</span><span>NJU Meta Graphics · 2024.11 — 2025.02</span><span>Python / C++</span></div>

为焦散渲染中每个三角形组贡献的辐照度计算保守边界，只对可能产生高贡献的光路组合进行采样，缩小搜索范围。我负责将算法从原型落地为高性能 C++ 完整实现。

<figure><img src="./assets/sig-fig_dragon.png" alt="Bernstein Bounds for Caustics 的 Dragon 焦散渲染结果。" loading="lazy" width="2599" height="811"><figcaption>Bernstein Bounds for Caustics / SIGGRAPH 2025</figcaption></figure>

### 渐进式实现

先用 Python 搭建 2D 原型，与密集采样逐点比对验证正确性，再迁移扩展到 C++ 的 4D 完整版本。将算法理解偏差与实现、性能问题分阶段暴露，降低复杂算法的调试成本。

<figure><img src="./assets/sig-arch.svg" alt="Bernstein Bounds 算法实现架构：Bounder 公共流程及 BVP 多项式计算。" loading="lazy" width="1792" height="1021"><figcaption>算法实现架构 / 公共流程与多项式计算分离</figcaption></figure>

### 模板化优化

将多项式度数改为编译期固定的模板参数，使循环边界成为常量，便于编译器自动向量化；针对高频出现的低度数稀疏组合，使用模板全特化提供快路径。结合实际运算量估算理论耗时，并与实测对照检查额外实现开销。

<h2 id="unity-ssr"><span class="project-number">06</span>Unity · Compute Shader SSR</h2>

<div class="project-meta"><span>Unity 6 / Compute Shader</span><span>屏幕空间反射</span></div>

在 Unity 6 渲染框架下实现 Compute Shader 版本 SSR。通过分 Tile 调度与组内共享内存复用邻域采样数据，减少重复访存并提升反射追踪效率。

<figure><video controls preload="metadata" playsinline><source src="./assets/unity-SSR.mp4" type="video/mp4"></video><figcaption>Compute Shader SSR / 屏幕空间反射展示</figcaption></figure>
