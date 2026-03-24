---
layout: default
---

欢迎来到我的主页！这里集中展示了我的所有代表作品和项目。

---

### MyVk

项目概述：基于 Vulkan API 从零构建实时渲染程序，重点实践显存管理、指令同步与渲染管线组织。

#### Frustum Culling 性能优化
核心实现：通过 Frustum Culling 剔除视区外几何体，降低 CPU Draw Call 与 GPU 渲染负载。

<div align="center">
    <video width="100%" controls>
    <source src="./assets/myvk-culling.mp4" type="video/mp4">
    </video>
</div>

结果说明：黄色线框为相机 Frustum，绿色线框为 AABB，蓝色线框为 OBB，红色线框为被剔除几何体（剔除计算基于 AABB）。

#### Bindless Texture 资源绑定优化
核心实现：将纹理绑定改为动态数组 `Textures[]` + Push Constant 索引方式，实现 Bindless Texture。

代码片段：

{% highlight cpp %}
layout(set=2, binding=1) uniform sampler2D Textures[];

layout(push_constant) uniform Push {
    uint MATERIAL_INDEX;
} push;
{% endhighlight %}

#### PBR + IBL 光照管线
核心实现：构建 PBR 光照管线并集成 IBL 预积分器，提升场景环境光真实感。

<div align="center">
    <video width="100%" controls>
    <source src="./assets/myvk-pbr.mp4" type="video/mp4">
    </video>
</div>

<div align="center">
    <img src="./assets/myvk-ibl.png" width="75%">
</div>

结果说明：RGBE 编码格式预积分结果中，最左侧为 Diffuse 项，后续为不同 Roughness Level 下的 Specular 项。

#### Cascade Shadow Map

Sun Light的阴影计算使用了Cascade Shadow Map。首先根据相机的frustum，划分了四个cascade层级，然后，利用划分的层级，计算出一个包围球，基于这个包围球的半径，构建出四个ortho矩阵，最后，利用这四个矩阵，计算出shadowmap

<div align="center">
    <video width="100%" controls>
    <source src="./assets/myvk-cascade.mp4" type="video/mp4">
    </video>
</div>

在采样阶段，为了避免切换cascade层级时产生的走样，我提前采样下一等级的cascade shadow map，并和当前cascade层级采样结果进行混合，以实现无缝切换cascade层级的效果

<div align="center">
    <video width="100%" controls>
    <source src="./assets/myvk-cascade-debug.mp4" type="video/mp4">
    </video>
</div>

该视频展示了在不同相机距离下选择不同的cascade层级

#### PCSS

Spot Light 和 Sphere Light 的阴影采样使用了PCSS算法。我的PCSS计算分为三个阶段，首先shader会在着色点附近向周围20个方向做采样，统计着色点附近的blocker数量和其对应的平均深度；接下来，根据统计结果估算Penumbra，最后根据Penumbra确定pcf采样核大小，pcf也进行20次均匀采样

<div align="center">
    <img src="./assets/myvk-No-Filtering.png" width="75%">
</div>

<div align="center">
    <img src="./assets/myvk-PCF.png" width="75%">
</div>

<div align="center">
    <img src="./assets/myvk-PCSS.png" width="75%">
</div>

三张图从上至下分别是完全不用滤波，使用PCF和使用PCSS。对比这三张图，我们可以观察到完全不用滤波的阴影边缘十分锐利，而使用PCF无论阴影距离光源远还是近，阴影边缘都被模糊了，最后PCSS在距离光源近的部分锐利，而距离光源远的部分模糊，更符合真实世界的观察

#### 屏幕空间 Light Culling

首先把屏幕空间沿着xy方向划分成若干个Tile，再遍历整个屏幕空间所有tile。在遍历的过程中，对于每个光源，我把以位置为圆心，limit为半径的圆投影到屏幕空间，然后看该投影是否在Tile中，如果在则说明这个光源对当前tile有影响

<div align="center">
    <video width="100%" controls>
    <source src="./assets/myvk-light-culling.mp4" type="video/mp4">
    </video>
</div>

在这个场景中，我复制了48块包含25个Sphere Light的平面，然后为相机制作了一个动画，从视角中只包含部分光源，逐渐到包含全部光源

<div align="center">
    <img src="./assets/myvk-close-light-culling.png" width="75%">
</div>

<div align="center">
    <img src="./assets/myvk-open-light-culling.png" width="75%">
</div>

参考分析图表，我们可以观察到打开Light Sort之后，峰值frame time几乎下降了一半。在起始阶段视角内光源数量少的时候，下降的更多，甚至frame time只有不开启Light Sort的十分之一，这充分证明了分Tile剔除光源的有效性

---

### Unity 6 + Compute Shader SSR

项目概述：在 Unity 6 渲染框架下实现 Compute Shader 版本 SSR。

<div align="center">
    <video width="100%" controls>
    <source src="./assets/unity-SSR.mp4" type="video/mp4">
    </video>
</div>

---

### MyPT（个人项目）

项目概述：基于现代 C++ 构建离线渲染系统，围绕全局光照算法验证 PBR 理论。

关键实现：
1. PBRT 风格模块化架构：拆分几何、材质、积分器等子系统，提升全局光照算法拓展性。
2. NEE + MIS 收敛优化：加速 Diffuse 材质直接光照收敛，减少高频噪点。
3. Photon Mapping 间接光照：弥补纯路径追踪在高频信号下的局限，高效计算复杂间接光照。

材料：架构图、基础光追 32spp vs NEE 32spp（待补充）。

<div align="center">
    <img src="./assets/mtpt-caustics-pt-8h.png" width="75%">
</div>

<div align="center">
    <img src="./assets/mypt-caustics-pm-32spp.png" width="75%">
</div>

结果对比：纯 Path Tracing（8h）仍有明显噪点；结合 Photon Mapping 的 Path Tracing（32spp，约 40min）可显著改善复杂间接光照质量。

---

### Bernstein Bounds for Caustics, SIGGRAPH 2025

项目概述：论文提出面向焦散渲染的新算法，分为预计算与渲染两个阶段，通过保守边界与重要性采样提升采样效率。

<div align="center">
    <img src="./assets/sig-fig_dragon.png">
</div>

#### Python 到 C++ 的工程重构
我的贡献：将算法原型由 Python 重构为现代 C++，优化内存与计算逻辑，实测性能贴合理论预期。

<div align="center">
    <img src="./assets/sig-arch.png">
</div>

架构说明：抽象父类 Bounder 负责公共流程（预计算数据准备、栈空间初始化、结果输出）；BVP 类负责 Bernstein 基多项式初始化与边界计算，并通过随机数测试验证可靠性。

优化细节：在原始实现中，多项式运算大量依赖运行期确定边界的 for 循环矩阵计算，分支预测开销明显。针对固定高光路径类型（如单次反射/折射），可在编译期通过模板特化确定矩阵维度与循环边界，使代码更接近顺序执行，提升流水线效率。另外，约束方程在计算边界前使用幂基多项式，其矩阵天然为上三角结构。模板特化时可省略下半矩阵计算，带来接近 50% 的性能提升。

