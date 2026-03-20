---
layout: default
---

# 你好，我是李博轩

欢迎来到我的个人主页。这里主要记录我的图形学学习路径、项目实践与研究经历。

---

## 项目经历

### MyVk

- 基于 Vulkan API 从零构建实时渲染程序，实践内存管理与指令同步机制

#### Frustum Culling 性能优化
- 结合 Frustum Culling 剔除视区外几何体，降低 CPU Draw Call 与 GPU 渲染负载

<div align="center">
    <video width="100%" controls>
    <source src="./assets/myvk-culling.mp4" type="video/mp4">
    </video>
</div>

- 视频中黄色线框是相机Frustum，绿色线框是AABB，蓝色线框OBB，红色线框代表被Frustum Culling剔除的几何体（Frustum Culling计算基于AABB）

#### Bindless Texture 资源绑定优化
- 结合动态数组 Textures[] 和 Push_Constant，运行时动态索引 Texture 资源，实现 Bindless Texture
```cpp
layout(set=2,binding=1) uniform sampler2D Textures[];

layout(push_constant) uniform Push {
    uint MATERIAL_INDEX;
} push;
```

#### PBR + IBL 光照管线
- 实现 PBR 管线并集成 IBL 预积分器，提升场景环境光真实感。

<div align="center">
    <video width="100%" controls>
    <source src="./assets/myvk-pbr.mp4" type="video/mp4">
    </video>
</div>

<div align="center">
    <img src="./assets/myvk-ibl.png" width="50%">
</div>

- RGBE编码格式的预积分结果，其中最左侧是 Diffuse 项的预积分结果，后续几张是 Specualr 项在不同 Roughness Level 下的预积分结果

#### CSM + PCSS 阴影方案
- 核心实现：通过 CSM 缓解透视走样，并用 PCSS 优化 Spot Light / Sphere Light 阴影，实现动态软阴影。
- A3 report

#### 屏幕空间 Light Culling
- 核心实现：实现基于屏幕空间的 Light Culling，仅计算有效光源，提升多光源场景性能。
- A3 report

### Unity 6 + Compute Shader SSR

- 核心实现：接入 Unity 6 渲染 API，编写 Compute Shader 实现 SSR，提升反射表现与光线步进效率。
- ETC desktop

### MyPT（个人项目）

- 基于现代 C++ 构建离线渲染系统，系统实现并验证 PBR 理论。

#### PBRT 风格模块化架构
- 核心实现：参考 PBRT 架构拆分几何、材质、积分器等子系统，提升全局光照算法拓展性。
- 画个架构图

#### NEE + MIS 收敛优化
- 核心实现：集成 NEE 与 MIS，加速 Diffuse 材质直接光照收敛，减少高频噪点。
- 基础光追 32spp vs NEE 32spp

#### Photon Mapping 间接光照
- 实现 Photon Mapping，弥补纯路径追踪在高频信号下的局限，高效计算 Diffuse 表面的复杂间接光照。

<div align="center">
    <img src="./assets/mtpt-caustics-pt-8h.png" width="50%">
</div>

<div align="center">
    <img src="./assets/mypt-caustics-pm-32spp.png" width="50%">
</div>

- 其中第一张图是纯 Path Tracing 运行8h后得到的结果，第二张图是结合了 Photon Mapping 的 Path Tracing 结果，使用32spp，运行时间约40min。可以明显观察到，尽管已经充分提高采样率，传统 Path Tracing 带来噪点还是非常明显。相对地，Photon Mapping 可以更好地处理这类复杂间接光照场景

### Bernstein Bounds for Caustics, SIGGRAPH 2025

- 在这篇文章中，我们提出了一种新的算法以优化在渲染焦散场景时的种种问题。总体上看，该算法分为预计算和渲染两个阶段。预计算阶段，我们首先利用有理化的高光路径约束方程和 Bernstein 基多项式的性质预计算出单束高光路径几何位置和 Irradiance 的保守边界；随后，由于渲染阶段采样的对象是图元元组（如一组三角形面片），我们再将之前计算的边界推广到光束所属图元元组在给定着色点产生的 Irradiance 边界。渲染时，我们根据预计算 Irradiance 边界结果，对符合几何边界条件的图元元组全集进行重要性采样（Importance Sampling），使得高贡献图元元组更容易被采样到。

#### Python 到 C++ 的工程重构
- 我在该项目中的贡献主要集中于将算法原型由 Python 重构为现代 C++，优化内存与计算逻辑，实测性能贴合理论预期。

<div align="center">
    <img src="./assets/sig-arch.png" width="50%">
</div>

- 总体代码架构如上，其中抽象父类 Bounder 包含了子类的共同流程，具体来说，包括预计算数据的准备，栈空间的初始化和最后的结果输出。Bounder 依赖BVP类，它们主要包含 Bernstein 基多项式的初始化及边界计算工作，经过充分的随机数测试，为后续算法实现提供可靠基础。

- 注意到BVP类都是基于模板实现的，这是因为在代码中，我们用多维数组来模拟多项式不同变量的系数和次数，在未进行优化之前，计算过程中主要的算力资源消耗是多项式运算中反复的 for 循环矩阵计算。在进行这些 for 循环时，其终止条件需要在运行期确定，为 CPU 的分支预测带来了不确定性，会导致不小的性能损耗。而根据我们推导的约束方程，对于特定的高光路径种类，比如单次反射或者单次折射，其计算过程中变量的次数是固定的，改变的仅仅只有系数。这就意味着对于特定的高光路径种类，我们可以对其进行模板特化，在编译期确定多项式矩阵大小，进而确定 for 循环终止条件，让原本需要进行分支预测的代码变成顺序执行的代码，最大化 CPU 流水线效率。

- 除此以外，使用模板特化的好处还有很多，比如我们计算约束方程时使用的是幂基多项式，最后计算边界才将其转化成 Bernstein 基多项式。而幂基多项式对应的矩阵都是上三角矩阵，这意味着在模板特化时可以省略下半矩阵，从而获得近乎 50% 的性能提升。
