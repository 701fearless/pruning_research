# 查找扩散模型压缩论文

> 为我查找| # | 方法 | 完整标题 | 作者 | 日期 | arXiv | 核验 |
> |---|---|---|---|---|---|---|
> | 1 | **SnapFusion** | SnapFusion: Text-to-Image Diffusion Model on Mobile Devices within Two Seconds | Yanyu Li, Huan Wang, Qing Jin, Ju Hu, Pavlo Chemerys, Yun Fu et al. | 2023-06 | [2306.00980](https://arxiv.org/abs/2306.00980) | ✅ |
> | 2 | **LAPTOP-Diff** | LAPTOP-Diff: Layer Pruning and Normalized Distillation for Compressing Diffusion Models | Dingkun Zhang, Sijia Li, Chen Chen, Qingsong Xie, Haonan Lu | 2024-04 | [2404.11098](https://arxiv.org/abs/2404.11098) | ✅ |
> | 3 | **BK-SDM** | BK-SDM: A Lightweight, Fast, and Cheap Version of Stable Diffusion | Bo-Kyeong Kim, Hyoung-Kyu Song, Thibault Castells, Shinkook Choi | 2023-05 | [2305.15798](https://arxiv.org/abs/2305.15798) | ✅ |
> | 4 | **PPCL / Pluggable Pruning DiT** | Pluggable Pruning with Contiguous Layer Distillation for Diffusion Transformers | Jian Ma, Qirong Peng, Xujie Zhu, Peixing Xie, Chen Chen, Haonan Lu | 2025-11（CVPR 2026） | [2511.16156](https://arxiv.org/abs/2511.16156) | ✅ |
> | 5 | **NanoFLUX** | NanoFLUX: Distillation-Driven Compression of Large Text-to-Image Generation Models for Mobile Devices | Ruchika Chavhan, Malcolm Chadwick, Alberto Gil Couto Pimentel Ramos, Luca Morreale, Mehdi Noroozi, Abhinav Mehrotra | 2026-02 | [2602.06879](https://arxiv.org/abs/2602.06879) | ✅ |
> | 6 | **Amber-Image** | Amber-Image: Efficient Compression of Large-Scale Diffusion Transformers | Chaojie Yang, Tian Li, Yue Zhang, Jun Gao | 2026-02 | [2602.17047](https://arxiv.org/abs/2602.17047) | ✅ |
> | 7 | **PARE** | PARE: Pruning and Adaptive Routing for Efficient Video Generation | Yutong Wang, Yunke Wang, Tianfan Xue, Yu Qiao, Yaohui Wang, Xinyuan Chen et al. | 2026-05 | [2605.27336](https://arxiv.org/abs/2605.27336) | ✅ |
> | 8 | **Dynamic-in-Few-Step** | Dynamic-in-Few-Step: Unifying Dynamic Computation and Few-Step Distillation for Efficient Video Generation | Yu Cheng, Siyue Yao, Zhongang Qi, Shanyan Guan, Wei Li, Fajie Yuan | 2026-07 | [2607.06631](https://arxiv.org/abs/2607.06631) | ✅ |
> | 9 | **MobileWan** | MobileWan: Closing the Quality Gap for Mobile Video Diffusion | Mohsen Ghafoorian, Denis Korzhenkov, Adil Karjauv, Ioannis Lelekas, Noor Fathima, Spyridon Stasis et al.（共 12 人） | 2026-07 | [2607.06173](https://arxiv.org/abs/2607.06173) | ✅ |
> | 10 | **Bridging（最切题）** | Bridging Diffusion Pruning and Step Distillation with Teacher-Aligned Repair | Jincheng Ying, Li Wenlin, Minghui Xu, Yinhao Xiao | 2026-07 | [2607.06335](https://arxiv.org/abs/2607.06335) | ✅ |这些所有文献，概括文章的主要内容：针对于什么模型、用了什么技术、做了什么实验

<details><summary>4 previous messages</summary>

> 我会逐篇核对 arXiv 摘要与论文页面，提炼三个维度：针对的模型、核心技术、实验设置与结果。对 2026 年条目也会以论文当前公开版本为准，并标注尚未公开细节的地方。
>
> <details><summary>Searched the web</summary>
>
> Searched the web for `site:arxiv.org/abs/2306.00980 SnapFusion ...`
>
> Searched the web for `site:arxiv.org/abs/2602.06879 NanoFLUX ...`
>
> Searched the web for `site:arxiv.org/abs/2607.06173 MobileWan ...`
>
> </details>
>
> 下面按“目标模型—核心技术—实验与结果”概括。需要注意：第 4—10 篇是 2025/2026 年 arXiv 预印本，实验结论以其当前公开版本为准。
>
> 1. **SnapFusion**
>
>    - **针对模型**：Stable Diffusion 类文本到图像扩散模型，重点面向移动端部署。
>    - **核心技术**：
>      - 重新设计轻量化 U-Net，删除冗余结构；
>      - 通过数据蒸馏降低图像解码器开销；
>      - 改进少步数蒸馏，加入更合适的训练策略和 classifier-free guidance 正则化。
>    - **实验**：在 MS-COCO 文本到图像任务上，与 Stable Diffusion v1.5 的 50 步采样比较。SnapFusion 使用 8 步采样时，FID 和 CLIP 均优于 50 步 SD v1.5，并实现移动端约 2 秒生成。[论文](https://arxiv.org/abs/2306.00980)
>
> 2. **LAPTOP-Diff**
>
>    - **针对模型**：Stable Diffusion v1.5 和 SDXL 的 U-Net。
>    - **核心技术**：
>      - 自动化层剪枝，避免人工指定删除哪些层；
>      - 提出具有可加性的 one-shot 层重要性准则；
>      - 使用归一化特征蒸馏，缓解不同特征损失量纲不一致导致的训练失衡。
>    - **实验**：在 SDXL 和 SD v1.5 上进行 50% 层级压缩，并使用 PickScore 等指标评估。50% 剪枝时，PickScore 最多下降约 4.0%，而对比方法最低下降约 8.2%，同时测试了模型延迟、计算量及生成质量。[论文](https://arxiv.org/abs/2404.11098)
>
> 3. **BK-SDM**
>
>    - **针对模型**：Stable Diffusion v1.4 和 v2.1-base 的 U-Net。
>    - **核心技术**：
>      - 结构化删除 U-Net 中的 residual blocks 和 attention blocks；
>      - 采用特征蒸馏恢复剪枝后的生成能力；
>      - 重点验证在有限训练资源下进行压缩和蒸馏的可行性。
>    - **实验**：
>      - 模型大小、MACs 和推理延迟降低约 30%–50%；
>      - 仅使用约 13 个 A100 GPU 天和小规模数据进行蒸馏，而原始模型训练成本超过 6000 个 A100 天；
>      - 在零样本 MS-COCO 上与大型文本到图像模型比较；
>      - 进一步测试个性化生成、image-to-image translation；
>      - 在边缘设备上实现约 4 秒推理。[论文](https://arxiv.org/abs/2305.15798)
>
> 4. **PPCL / Pluggable Pruning DiT**
>
>    - **针对模型**：Diffusion Transformer，实验覆盖多种多模态 DiT/MMDiT 架构。
>    - **核心技术**：
>      - 用线性探测和相似度指标的一阶变化趋势识别冗余的连续层区间；
>      - 同时支持深度剪枝和宽度剪枝；
>      - 提出 teacher-student 交替蒸馏，使一个训练过程适配多个剪枝比例；
>      - 因此不必针对每个压缩配置重新训练。
>    - **实验**：在多种多模态扩散 Transformer 上验证。相较完整模型，参数量最多减少 50%，关键客观指标下降低于 3%，同时保持较好的图像生成质量。[论文](https://arxiv.org/abs/2511.16156)
>
> 5. **NanoFLUX**
>
>    - **针对模型**：FLUX.1-Schnell 及其大规模文本到图像 Flow Matching/扩散 Transformer，目标是移动端部署。
>    - **核心技术**：
>      - 将约 17B
>
> Stream error
>
> Reconnecting 1/5
> stream disconnected before completion: idle timeout waiting for SSE
</details>

下面按“针对的模型—核心技术—实验与结论”概括。文中“实验结果”以各 arXiv 摘要和公开页面为准；部分 2026 年论文仍是预印本，实验细节可能会随版本更新。

1. **SnapFusion**
   论文：[arXiv:2306.00980](https://arxiv.org/abs/2306.00980)

   - **针对模型**：Stable Diffusion 类文本到图像扩散模型，重点面向移动端部署。
   - **核心技术**：
     - 重新设计轻量级 U-Net，删除冗余结构；
     - 通过数据蒸馏降低图像解码器计算量；
     - 改进步数蒸馏，将多步采样压缩到少量去噪步骤；
     - 在蒸馏中加入 classifier-free guidance 相关正则化。
   - **实验**：
     - 在 MS-COCO 文本到图像任务上测试；
     - 使用 8 步采样与 Stable Diffusion v1.5 的 50 步采样比较；
     - FID 和 CLIP 得分优于 50 步的 SD v1.5；
     - 在移动设备上实现约 2 秒内生成图像。
   - **主要贡献**：较早系统性展示了“轻量网络结构 + 步数蒸馏”在移动端文本生图中的可行性。

2. **LAPTOP-Diff**
   论文：[arXiv:2404.11098](https://arxiv.org/abs/2404.11098)

   - **针对模型**：Stable Diffusion v1.5 和 SDXL 的 U-Net。
   - **核心技术**：
     - 自动化层剪枝，不再依赖人工选择删除哪些层；
     - 提出一次性层重要性评估准则；
     - 利用指标的可加性，使一次剪枝结果能够较好预测实际性能；
     - 提出归一化特征蒸馏，缓解不同特征层损失数值尺度不平衡的问题。
   - **实验**：
     - 在 SDXL 和 SD v1.5 上进行不同剪枝比例实验；
     - 使用 PickScore 等文本到图像质量指标；
     - 在 50% 剪枝比例下，PickScore 最小下降约 4.0%，而对比方法至少下降约 8.2%；
     - 还比较了参数量、计算量和生成质量。
   - **主要贡献**：把 U-Net 层删除从人工经验设计转为自动化结构剪枝，并通过归一化蒸馏提高压缩后恢复能力。

3. **BK-SDM**
   论文：[arXiv:2305.15798](https://arxiv.org/abs/2305.15798)

   - **针对模型**：Stable Diffusion v1.4 和 v2.1-base 的 U-Net。
   - **核心技术**：
     - 对 U-Net 中的 ResNet block 和 attention block 进行结构化 block pruning；
     - 使用 feature distillation 让剪枝后的模型模仿原始 SD；
     - 通过少量数据和有限算力完成再训练。
   - **实验**：
     - 模型大小、MACs 和推理延迟降低约 30%～50%；
     - 在零样本 MS-COCO 文本到图像任务上与大型模型比较；
     - 测试个性化生成和 image-to-image translation；
     - 原始 SD 模型训练需要超过 6000 个 A100 天，而 BK-SDM 蒸馏再训练约 13 个 A100 天；
     - 在边缘设备上实现约 4 秒推理。
   - **主要贡献**：证明 Stable Diffusion 可以通过 block pruning 和轻量蒸馏，在较低训练成本下获得实用的小型版本。

4. **PPCL / Pluggable Pruning with Contiguous Layer Distillation**
   论文：[arXiv:2511.16156](https://arxiv.org/abs/2511.16156)

   - **针对模型**：Diffusion Transformer，尤其是多模态 DiT/MMDiT 类文本到图像模型。
   - **核心技术**：
     - 通过线性探测和相似度指标的一阶变化趋势，自动寻找冗余的连续层区间；
     - 同时支持深度剪枝和宽度剪枝；
     - 使用 teacher-student 交替蒸馏；
     - 一个训练过程支持多种剪枝比例，避免每种结构都重新训练。
   - **实验**：
     - 在多个多模态 Diffusion Transformer 架构上进行测试；
     - 参数量最多降低 50%；
     - 关键客观指标下降低于 3%；
     - 与不同剪枝比例和其他压缩方法进行比较。
   - **主要贡献**：提供可插拔、可适配不同压缩比例的 DiT 结构化剪枝框架，重点解决“每个压缩结构都要单独训练”的问题。

5. **NanoFLUX**
   论文：[arXiv:2602.06879](https://arxiv.org/abs/2602.06879)

   - **针对模型**：FLUX.1-Schnell 文本到图像 flow-matching 模型，目标是移动端部署。
   - **核心技术**：
     - 从约 17B 规模的 FLUX.1-Schnell 蒸馏出约 2.4B 模型；
     - 剪除 Diffusion Transformer 中的冗余组件；
     - 引入基于 ResNet 的 token 下采样，使中间层处理低分辨率 token；
     - 文本编码器蒸馏利用去噪器早期层的视觉信号；
     - 采用渐进式压缩流程逐步恢复质量。
   - **实验**：
     - 主要测试 512×512 文本到图像生成；
     - 在移动设备上约 2.5 秒生成一张图像；
     - 比较压缩前后的模型规模、延迟和视觉质量。
   - **主要贡献**：将 FLUX 级别的大模型压缩到移动端可运行规模，同时保留较高文本对齐和图像质量。摘要中还描述了从约 12B Transformer 主体压缩到约 2B 的过程。

6. **Amber-Image**
   论文：[arXiv:2602.17047](https://arxiv.org/abs/2602.17047)

   - **针对模型**：Qwen-Image 的 60 层双流 MMDiT 文本到图像模型。
   - **核心技术**：
     - **Amber-Image-10B**：
       - 根据时间步敏感性进行深度剪枝；
       - 被保留层通过局部权重平均重新初始化；
       - 采用逐层蒸馏和全参数微调。
     - **Amber-Image-6B**：
       - 将深层双流结构转换为单流结构；
       - 单流部分从 image branch 初始化；
       - 使用渐进式蒸馏和轻量微调。
   - **实验**：
     - 在 DPG-Bench、LongText-Bench 等基准上测试；
     - 参数量最多减少约 70%；
     - 从 10B 压缩到 6B 的完整流程使用少于 2000 GPU 小时；
     - 在图像保真度和文字渲染方面接近更大的模型。
   - **主要贡献**：针对大规模 MMDiT 的深度压缩和双流到单流转换，强调不需要从头训练，也不需要大规模数据工程。

7. **PARE**
   论文：[arXiv:2605.27336](https://arxiv.org/abs/2605.27336)

   - **针对模型**：Wan2.1-14B Video Diffusion Transformer，覆盖文本到视频和图像到视频。
   - **核心技术**：
     - **宽度剪枝**：区分空间 attention head 和时间 attention head，避免过早删除影响运动建模的 temporal heads；
     - **深度动态路由**：训练轻量 router，根据去噪时间步和当前视频内容动态决定执行哪些 block；
     - 先进行宽度剪枝和蒸馏，再联合优化剪枝后的学生模型与路由器；
     - 可与 step distillation 组合。
   - **实验**：
     - 在 Wan2.1-14B 的 text-to-video 和 image-to-video 任务上测试；
     - 使用 VBench 的多个维度评价画质、运动一致性和文本对齐；
     - 显著降低每个去噪步骤的计算量；
     - 与 step distillation 结合后可进一步加速。
   - **主要贡献**：从固定结构压缩转向“输入和时间步自适应”的视频生成计算。

8. **Dynamic-in-Few-Step**
   论文：[arXiv:2607.06631](https://arxiv.org/abs/2607.06631)

   - **针对模型**：视频扩散模型，实验重点为 Wan-14B。
   - **核心技术**：
     - 将动态结构稀疏化直接融入少步蒸馏；
     - 为不同去噪时间步学习不同的子模型，形成 step-specific Mixture-of-Models；
     - 提出 Progressive Training Strategy，逐步稳定结构决策；
     - 使用 Output Rollout Mechanism，使不同时间步的输出保持一致；
     - 设计专用推理引擎执行动态子模型。
   - **实验**：
     - 与 4 步蒸馏模型结合；
     - 在 Wan-14B 上额外减少约 24% 的每步 FLOPs；
     - 获得约 1.2 倍额外墙钟时间加速；
     - 相比 50 步教师模型达到约 30 倍整体加速；
     - 在保持有竞争力的视频质量的同时降低计算量。
   - **主要贡献**：联合优化“采样步数”和“每一步使用多少网络结构”，而不是先固定网络再做少步蒸馏。

9. **MobileWan**
   论文：[arXiv:2607.06173](https://arxiv.org/abs/2607.06173)

   - **针对模型**：Wan2.2-5B 视频扩散 Transformer，目标是商业移动设备。
   - **核心技术**：
     - 通过 recurrence distillation 将视频生成重构为分块自回归过程；
     - 使用 chunk-wise 处理和 causal linear attention，使注意力计算保持近似恒定内存；
     - 将模型推理形式转化为类似 RNN 的递归计算；
     - 使用可学习 attention-head pruning，通过二值 head gate 和稀疏性目标进行端到端剪枝；
     - 结合采样步数蒸馏和内存优化的 VAE 解码。
   - **实验**：
     - 在移动硬件上生成 5 秒、480×832、16 FPS 视频；
     - 端到端延迟约 20 秒；
     - VBench 得分 83.79；
     - 同时评估视频质量、运动连贯性、内存占用和移动端延迟。
   - **主要贡献**：不是简单地把模型缩小到 0.4B～1.8B，而是证明 5B 规模模型也可以通过递归重构和结构化压缩部署到手机。

10. **Bridging Diffusion Pruning and Step Distillation with Teacher-Aligned Repair**
    论文：[arXiv:2607.06335](https://arxiv.org/abs/2607.06335)

    - **针对模型**：EDM2-XS 图像扩散模型，实验使用 ImageNet 512×512。
    - **核心技术**：
      - 研究剪枝和一步蒸馏能否直接串联；
      - 发现剪枝模型直接进行 SiDA/一步蒸馏会产生不可用样本；
      - 在二者之间加入短暂的 teacher-aligned repair 阶段；
      - 使用带噪真实图像 latent，使剪枝后的学生先重新对齐教师生成分布；
      - 再进行 one-step distillation。
    - **实验**：
      - 原始 EDM2-XS：124.713M 参数、63 次网络评估、FID 3.53；
      - 20% 剪枝后：98.826M 参数、1 次网络评估、FID 3.12；
      - 30% 剪枝后：88.029M 参数、1 次网络评估、FID 4.26；
      - 证明“剪枝—修复—一步蒸馏”优于直接将剪枝模型交给一步蒸馏。
    - **主要贡献**：最直接地研究网络剪枝与采样步数蒸馏的衔接问题，提出一个很短但关键的教师对齐修复阶段。

**整体脉络**

这些工作可以分成三类：

- **文本到图像模型压缩**：SnapFusion、BK-SDM、LAPTOP-Diff、PPCL、NanoFLUX、Amber-Image；
- **视频扩散模型压缩与动态计算**：PARE、Dynamic-in-Few-Step、MobileWan；
- **剪枝与步数蒸馏的联合流程**：Bridging。

技术路线也逐渐演进：

1. 从删除 U-Net block 或 layer 的静态剪枝开始；
2. 发展到自动层搜索、宽度/深度联合剪枝；
3. 再发展到根据输入内容和去噪时间步动态执行网络；
4. 最后将结构压缩、步数蒸馏、教师对齐和移动端推理引擎统一起来。

因此，这组论文共同关注的问题是：如何在尽量保持生成质量的情况下，同时减少模型参数量、FLOPs、采样步数、显存和实际设备延迟。

> 这些论文的benchmark是什么？

<details><summary>2 previous messages</summary>

> 我会把“benchmark”区分为数据集/基准套件与指标，并逐篇补齐；对于摘要没有明确写出的论文，我会标注“公开摘要未说明”，避免把训练数据或指标误当成 benchmark。
>
> <details><summary>Searched the web</summary>
>
> Searched the web for `SnapFusion MS-COCO benchmark FID CLIP ...`
>
> Searched the web for `NanoFLUX benchmark evaluation dataset metrics ...`
>
> Searched the web for `"PARE" "VBench" Wan2.1-14B ...`
>
> </details>
</details>

这些论文使用的 benchmark 不完全相同。可以按图像生成、视频生成和系统效率三类来看。

| 方法 | 主要 benchmark / 数据集 | 主要指标 |
|---|---|---|
| **SnapFusion** | **MS-COCO 2014 validation**，通常使用 30K caption-image pairs；移动端还进行真实设备延迟测试 | FID、CLIP Score、去噪步数、端到端延迟。论文比较 8 步 SnapFusion 与 50 步 SD v1.5。 |
| **LAPTOP-Diff** | **MS-COCO**；在 **Stable Diffusion v1.5 和 SDXL** 上测试。论文重点使用 PickScore 评价压缩后的文本到图像质量 | PickScore、参数量、MACs、推理延迟、剪枝比例。50% 剪枝时重点报告 PickScore 下降。 |
| **BK-SDM** | **MS-COCO 30K zero-shot text-to-image**；此外测试个性化生成和 image-to-image translation | FID、IS、CLIP Score、模型大小、MACs、延迟；还报告边缘设备推理时间。公开代码明确使用 COCO 30K 和 41K real-image statistics。 |
| **PPCL** | 主要是 **Qwen-Image / MMDiT 类模型**的文本到图像评测；论文还将方法扩展到 **FLUX.1 Lite** 等 DiT 架构 | 参数量、计算量、图像质量和文本对齐指标；论文摘要强调 50% 参数压缩、关键客观指标下降小于 3%。公开摘要没有完整列出所有 benchmark 名称。 |
| **NanoFLUX** | **GenEval** 是公开资料中明确提到的主要文本到图像 benchmark；同时进行 512×512 移动端生成测试 | GenEval 的组合性/语义正确性指标、图像质量、参数量、延迟、移动端内存和运行时间。 |
| **Amber-Image** | **DPG-Bench**、**GenEval**、**LongText-Bench**、**CVTG-2K**；其中前两者偏通用文本到图像和复杂提示词，后两者偏文字渲染 | DPG-Bench/GenEval 总体分数，LongText-Bench 和 CVTG-2K 的中英文文字渲染分数，以及参数量和训练成本。 |
| **PARE** | **VBench**；在 **Wan2.1-14B** 上分别测试 text-to-video 和 image-to-video | VBench 各维度质量、时间一致性、运动合理性、文本对齐，以及 FLOPs、参数量和每步计算量。 |
| **Dynamic-in-Few-Step** | 主要在 **Wan-14B** 上做视频生成实验；公开摘要没有给出额外独立数据集名称 | 视频生成质量、4 步蒸馏后的 FLOPs、墙钟时间、相对 50 步教师模型的加速比。摘要明确报告额外减少 24% 每步 FLOPs 和约 30 倍整体加速。 |
| **MobileWan** | **VBench**，同时与 Wan2.2、Open-Sora、CogVideoX、LTX-Video、SANA-Video 等视频模型比较；另外进行商业移动设备测试 | VBench Quality、Semantic、Total 分数，视频分辨率、FPS、端到端延迟、峰值内存。官方项目页报告 Total VBench 83.79。 |
| **Bridging** | **ImageNet 512×512** 扩散生成 benchmark，模型为 EDM2-XS | FID、参数量、NFE/network evaluations。原始 EDM2-XS 为 63 次网络评估、FID 3.53；20% 剪枝加一步蒸馏后为 1 次评估、FID 3.12。 |

补充说明：

- **MS-COCO** 主要用于文本到图像质量和文本图像对齐；
- **GenEval、DPG-Bench** 更关注组合性提示词、实体属性和关系是否正确；
- **LongText-Bench、CVTG-2K** 主要用于图像中的长文本和多语言文字渲染；
- **VBench** 是视频生成综合 benchmark，包含视觉质量、语义一致性、运动和时间一致性等维度；
- **ImageNet-512** 在 Bridging 中主要用于标准图像扩散模型的 FID 比较；
- 移动端论文中的手机延迟、显存和内存占用属于 **deployment/system benchmark**，不是传统意义上的生成质量数据集。

从 benchmark 覆盖范围看，**SnapFusion、BK-SDM、LAPTOP-Diff** 的评测最偏传统 Stable Diffusion 文本生图；**NanoFLUX、Amber-Image、PPCL** 更偏新一代 DiT/MMDiT 压缩；**PARE、Dynamic-in-Few-Step、MobileWan** 则以 Wan 系列视频模型和 VBench 为主；**Bridging** 的实验规模最集中，核心就是 ImageNet-512 上的 FID 与网络评估次数。

> Amber-Image
> 论文：arXiv:2602.17047
> Bridging Diffusion Pruning and Step Distillation with Teacher-Aligned Repair
> 论文：arXiv:2607.06335
> PPCL / Pluggable Pruning with Contiguous Layer Distillation
> 论文：arXiv:2511.16156
> NanoFLUX
> 论文：arXiv:2602.06879
> 再帮我更详细地总结一下这四篇文章的主要内容，包括关键公式、技术方法和流程、关键点、benchmark

<details><summary>2 previous messages</summary>

> 我会进一步查看这四篇论文的正文/公式，而不只依赖摘要。重点整理：问题定义、压缩目标、损失函数、训练阶段、推理流程、消融/关键结论，以及 benchmark 的具体设置；若某篇正文暂时无法抓取，我会明确区分“论文明确给出”和“根据摘要可确认”。
>
> <details><summary>Searched the web</summary>
>
> Searched the web for `https://arxiv.org/html/2602.17047`
>
> Searched the web for `'GenEval'`
>
> Searched the web for `'Qwen-Image'`
>
> </details>
</details>

下面按“模型对象 → 方法流程 → 关键公式 → 关键实验”展开。公式使用论文中的核心形式，并做了适度简化。

## 1. Amber-Image

论文：[arXiv:2602.17047](https://arxiv.org/abs/2602.17047)

### 1.1 针对的模型

Amber-Image 从 **Qwen-Image** 压缩而来：

- 原始模型：约 20B 参数；
- 主干：60 层双流 MMDiT；
- image stream 和 text stream 分别处理图像、文本 token；
- 文本编码器 Qwen2.5-VL-7B 和 VAE 保持不变；
- 压缩对象主要是 60 层 MMDiT backbone。

最终得到两个模型：

- **Amber-Image-10B**：通过深度剪枝得到，约保留 30 层；
- **Amber-Image-6B**：在 10B 模型基础上，将后部双流结构改为单流结构。

### 1.2 Amber-Image-10B：时间步敏感的深度剪枝

#### 层重要性评估

对第 \(l\) 层进行消融，令其残差分支失效：

\[
\tilde{h}_{l+1}
=
(1-m_l)\mathcal{F}_l(h_l)+h_l
\]

其中：

- \(\mathcal{F}_l\)：第 \(l\) 个 Transformer block；
- \(m_l=1\)：删除该层；
- \(m_l=0\)：保留该层。

比较原始模型和删除第 \(l\) 层后的噪声预测：

\[
\delta_l(z_t,t,p)
=
\epsilon_\theta(z_t,t,p)
-
\epsilon_{\theta\setminus\{l\}}(z_t,t,p)
\]

然后在多个 prompt 和多个扩散时间步上累积误差：

\[
I_l(\mathcal P)
=
\frac{1}{|\mathcal P|}
\sum_{p\in\mathcal P}
\frac{
\sum_{t\in\mathcal T_{\rm sub}}
\omega_t
\|\delta_l(z_t,t,p)\|_2^2
}{
\sum_{t\in\mathcal T_{\rm sub}}\omega_t
}
\]

其中 \(\omega_t\) 是时间步权重。论文强调，高噪声阶段更影响全局语义和布局，因此对相应时间步使用更高权重。\(I_l\) 越小，说明删除该层对噪声预测影响越小，越适合剪枝。

#### 局部权重平均初始化

删除连续层后，不是简单复制某一层，而是将被删除层邻近的权重平均到保留层：

\[
W_l^{\rm init}
=
\frac{1}{k+1}
\sum_{j=0}^{k}W_{l+j}^{\rm orig}
\]

其中 \(k\) 是保留层 \(l\) 后面连续被删除的层数。

直观上，这相当于用一个保留层近似多个相邻层的累积变换，减小剪枝后模型的初始损失突增。

#### 两阶段恢复

1. **针对性层蒸馏**：只训练通过局部权重平均重新初始化的层，其余层冻结；
2. **全参数微调**：解冻所有层，使用标准扩散训练目标继续恢复生成质量。

论文中的核心思想是：先让新初始化层补偿删除层的功能，再让整个模型重新协调。

### 1.3 Amber-Image-6B：深层双流到单流

10B 模型的前 10 层继续保留双流，后 20 层改为共享的单流结构：

- 第 1～10 层：text/image 双流；
- 第 11～30 层：单流；
- 单流层使用 image branch 的权重初始化：

\[
W_{\rm shared}=W_{\rm image},
\quad
W\in\{W_Q,W_K,W_V,W_O\}
\]

单流层的蒸馏目标是把教师模型两个分支的隐藏状态拼接起来：

\[
H_{\rm target}^{(l)}
=
\operatorname{Concat}
\left(
H_{\rm text}^{(l,\rm teacher)},
H_{\rm image}^{(l,\rm teacher)}
\right)
\]

然后最小化：

\[
\mathcal L_{\rm MSE}
=
\left\|
H_{\rm student}^{(l)}
-
H_{\rm target}^{(l)}
\right\|_2^2
\]

前 10 层冻结，先训练后 20 个单流层；之后再做一次轻量全参数微调。

### 1.4 训练流程

整体流程为：

```text
Qwen-Image 20B / 60层双流 MMDiT
        ↓
时间步敏感的层消融评估
        ↓
删除30个低重要性层
        ↓
局部权重平均初始化
        ↓
层级蒸馏 + 全参数微调
        ↓
Amber-Image-10B
        ↓
后20层双流 → 单流
        ↓
image-stream 初始化
        ↓
隐藏状态蒸馏 + 轻量微调
        ↓
Amber-Image-6B
```

论文使用约 100 万图文对进行恢复训练；层重要性评估使用 1 万个样本。整个 10B 到 6B 的流程约消耗 1,872 GPU hours。

### 1.5 Benchmark

通用文本到图像：

- **DPG-Bench**：1,065 个密集语义 prompt，评价 Global、Entity、Attribute、Relation 等；
- **GenEval**：Single Object、Two Objects、Counting、Colors、Position、Attribute Binding；
- **OneIG-Bench**：Alignment、Style、Diversity、Text、Reasoning，支持中英文。

文字渲染：

- **LongText-Bench**：英文和中文长文本；
- **CVTG-2K**：复杂文字布局和字符完整性。

论文报告 Amber-Image-10B/6B 在 DPG-Bench、GenEval 上具有很强的组合语义能力；在 LongText-Bench 和 CVTG-2K 上，10B 版本的文字渲染优于多种更大模型。OneIG-Bench 中，Text 接近教师，但 Style 和 Diversity 仍有损失。

---

## 2. Bridging Diffusion Pruning and Step Distillation with Teacher-Aligned Repair

论文：[arXiv:2607.06335](https://arxiv.org/abs/2607.06335)

### 2.1 研究问题

论文研究的是：

> 剪枝之后，能否直接跳过传统的长时间 post-pruning retraining，直接进行一步蒸馏？

作者发现不能直接替换：

```text
剪枝模型 → 直接 SiDA 一步蒸馏
```

会导致训练初期样本崩溃，因为剪枝模型在教师模型所在的噪声空间中偏差太大，蒸馏梯度不稳定。于是引入一个短暂的 **Teacher-Aligned Repair** 阶段。

### 2.2 模型与压缩对象

实验使用：

- 教师模型：EDM2-XS；
- 任务：ImageNet-512 类条件图像生成；
- 剪枝结构：U-Net 的通道宽度和 residual/attention branch；
- 目标：得到一次网络调用即可生成图像的 compact generator。

紧凑模型仍保持教师模型的输入输出接口：

\[
\hat z
=
G_\theta(\eta,\sigma_{\rm init},y)
\]

其中：

- \(\eta\)：初始噪声；
- \(\sigma_{\rm init}\)：初始噪声尺度；
- \(y\)：类别标签；
- \(\hat z\)：生成的 latent。

### 2.3 Teacher-aware block sensitivity

剪枝前，先在真实图像 latent 上测量每个 block 的重要性。

给定真实图像 \(x_i\)：

\[
z_i=E(x_i),
\qquad
\tilde z_i=z_i+\sigma^\star\epsilon_i
\]

教师输出为：

\[
t_i=T(\tilde z_i,\sigma^\star,y_i)
\]

完整模型的局部教师对齐误差：

\[
\hat\varepsilon_{\sigma^\star}(G)
=
\frac{1}{N}
\sum_i
\left\|
G(\tilde z_i,\sigma^\star,y_i)-t_i
\right\|_2^2
\]

暂时删除 block \(b\) 后，重要性定义为：

\[
I(b)
=
\hat\varepsilon_{\sigma^\star}(G^{-b})
-
\hat\varepsilon_{\sigma^\star}(G)
\]

\(I(b)\) 越小，说明删除该 block 对教师输出影响越小，因此优先删除。ImageNet 实验使用 \(\sigma^\star=2.5\)。

### 2.4 Noisy-real teacher alignment

剪枝后，用真实图像 latent 加噪，而不是用模型自己生成的样本：

\[
\tilde z=z+\sigma_r\epsilon
\]

修复损失为：

\[
\mathcal L_{\rm repair}
=
\mathbb E_{x,y,\epsilon}
\left[
\left\|
G_{\mathcal S}(\tilde z,\sigma_r,y)
-
T(\tilde z,\sigma_r,y)
\right\|_2^2
\right]
\]

参数更新为：

\[
\theta_{\rm align}
=
\theta_{\mathcal S}
-
\tau
\nabla_\theta
\mathcal L_{\rm repair}
\]

噪声尺度选择为：

\[
\sigma_r
=
\max\{
\sigma_{\rm init},
\sigma_1,\ldots,\sigma_K
\}
\]

在 ImageNet 实验中 \(\sigma_{\rm init}=2.5\)，因此 repair 主要对齐高噪声起点。其作用不是重新训练一个完整多步扩散模型，而只是将剪枝模型拉回教师模型附近的有效蒸馏区域。

### 2.5 一步蒸馏

修复后再初始化 SiDA：

\[
\theta_0\leftarrow\theta_{\rm align}
\]

\[
\theta^\star
=
\arg\min_{\theta:\theta_0=\theta_{\rm align}}
\mathcal D(G_\theta;T,\mathcal A)
\]

论文实现中 \(\mathcal D\) 使用 SiDA。教师 score network 冻结，fake-score network 用于一步生成器训练。论文还说明，该 repair bridge 原则上可以接入其他教师型一步蒸馏损失。

### 2.6 完整流程

```text
EDM2-XS 教师
    ↓
测量每个 block 的 teacher-alignment sensitivity
    ↓
删除低重要性 block / channel
    ↓
复制或切片教师权重
    ↓
在真实 ImageNet latent + 高噪声上进行 repair
    ↓
以 repaired checkpoint 初始化 SiDA
    ↓
训练一次生成器
    ↓
1 NFE compact generator
```

### 2.7 Benchmark 与结果

主要 benchmark：

- **ImageNet-512**：类别条件图像生成；
- **CIFAR-10**：额外测试 repair 是否能与 Diff-Instruct 配合。

ImageNet-512 主要指标：

- FID；
- 参数量；
- NFE/network evaluations；
- RTX 3090 上的推理时间。

关键结果：

| 模型 | NFE | 参数量 | FID | 单图时间 |
|---|---:|---:|---:|---:|
| EDM2-XS | 63 | 124.713M | 3.53 | 3390.8 ms |
| 20% pruning + repair + SiDA | 1 | 98.826M | 3.12 | 62.7 ms |
| 30% pruning + repair + SiDA | 1 | 88.029M | 4.26 | 51.6 ms |

消融显示：

- 直接 SiDA、无 repair：FID 112.56；
- 仅有限 repair：FID 17.23；
- 完整的 teacher-aware block pruning + 高噪声 repair + SiDA：FID 约 3.16。

这篇论文最关键的结论是：**剪枝和一步蒸馏之间存在初始化分布鸿沟，短暂的教师对齐阶段可以填补这个鸿沟。**

---

## 3. PPCL：Pluggable Pruning with Contiguous Layer Distillation

论文：[arXiv:2511.16156](https://arxiv.org/abs/2511.16156)

### 3.1 针对的模型

PPCL 面向 MMDiT/DiT 文本到图像模型，重点实验模型为：

- **Qwen-Image**：20B、60 层 MMDiT；
- **FLUX.1-dev**；
- **FLUX.1 Lite**；
- 还测试 Qwen-Image-Edit 等图像编辑模型。

目标是同时进行：

- depth-wise pruning；
- width-wise pruning；
- 多个压缩比例共用一个蒸馏过程。

### 3.2 为什么剪连续层

作者观察到：

- 相邻层具有功能耦合；
- 一段连续层可能共同完成一个语义变换；
- 删除连续层往往比删除不连续层更容易通过蒸馏恢复；
- text stream 的跨层表示相似度较高，存在冗余；
- 一些 FFN 可以用线性投影近似。

因此 PPCL 不直接按照单层 importance 排序，而是先找出可替代的连续层区间。

### 3.3 线性探测与冗余区间检测

对每个教师层训练一个轻量线性 probe \(l_i\)，拟合该层的输出。然后用 probe 替换连续层，构造 surrogate model。

对于起始层 \(u\) 和候选终点 \(k\)，计算 CKA：

\[
\operatorname{cka}(u,k)
=
\operatorname{CKA}
\left(
T_u(T_{u-1}^{X}),
T_{u:k}^{[u\rightarrow k]}(T_{u-1}^{X})
\right)
\]

再定义一阶差分：

\[
\Delta(u,k)
=
-
\left(
\operatorname{cka}(u,k)
-
\operatorname{cka}(u,k-1)
\right)
\]

当 \(\Delta(u,k)\) 先下降后上升时，把该拐点视为冗余区间终点：

\[
v
=
\min
\left\{
k-1:
\Delta(u,k)>\Delta(u,k-1)
\right\}
\]

得到多个连续冗余区间 \(\mathcal I\)。

### 3.4 连续层深度剪枝与蒸馏

对于冗余区间 \([u,v]\)，将学生层初始化为教师第 \(u\) 层，并让它直接拟合教师第 \(v\) 层的表示：

\[
\mathcal L_{\rm depth}^{[u,v]}
=
\left\|
\operatorname{Norm}
\left(
S_{\rm init}^{u}(T_{u-1}^{D})
\right)
-
\operatorname{Norm}
\left(
T_v^{D}
\right)
\right\|_2^2
\]

总深度蒸馏损失为：

\[
\mathcal L_{\rm depth}
=
\sum_{i=1}^{n}
\mathcal L_{\rm depth}^{[u_i,v_i]}
\]

这里的重点是 **非顺序蒸馏**：每个连续区间单独训练，避免早期层误差逐层累积。

### 3.5 宽度剪枝

PPCL 主要压缩两类宽度冗余：

1. text stream 的部分层；
2. image/text stream 中可替代的 FFN。

对于 text stream，保留 QKV，但将其余部分替换为轻量线性投影：

\[
z_p^{txt}=l_p^z(z_r^{txt}),
\qquad
h_p^{txt}=l_p^h(h_r^{txt})
\]

对于 FFN：

\[
h_q^{img}=l_q^{img}(g_q^{img}),
\qquad
h_q^{txt}=l_q^{txt}(g_q^{txt})
\]

宽度蒸馏损失：

\[
\mathcal L_{\rm width}
=
\sum_{j\in\mathcal R_{txt}\cup\mathcal R_{ffn}}
\left(
\mathcal L_{\rm width}^{j}
+
\mathcal L_{\rm linear}^{j}
\right)
\]

其中：

\[
\mathcal L_{\rm width}^{j}
=
\left\|
\operatorname{Norm}
\left(
S_{\rm width}^{j}(T_{j-1}^{D})
\right)
-
\operatorname{Norm}
\left(
T_j^{D}
\right)
\right\|_2^2
\]

\[
\mathcal L_{\rm linear}^{j}
=
\begin{cases}
\|z_j^{txt}-T_j^z\|_2^2+
\|h_j^{txt}-T_j^{txt}\|_2^2,
&j\in\mathcal R_{txt}\\[3pt]
\|h_j^{img}-T_j^{img}\|_2^2+
\|h_j^{txt}-T_j^{txt}\|_2^2,
&j\in\mathcal R_{ffn}
\end{cases}
\]

### 3.6 PPCL 流程

```text
教师 MMDiT
   ↓
为每层训练 linear probe
   ↓
用 CKA 一阶差分检测连续冗余区间
   ↓
连续层深度剪枝 + 非顺序特征蒸馏
   ↓
检测 text-stream / FFN 宽度冗余
   ↓
线性投影替换 + 宽度蒸馏
   ↓
全参数微调
   ↓
按需替换部分教师层
   ↓
得到 6.5B、8B、10B、12B、14B 等不同压缩版本
```

“Pluggable”指的是：可以通过重新插入部分教师层，在同一压缩框架中得到不同参数规模，而不必完全重新设计网络。

### 3.7 Benchmark 与结果

训练数据：

- 从 LAION-2B-en 抽取 100,000 张图像；
- 用 Qwen2.5-VL 生成更详细 caption；
- 再用 Qwen-Image 形成图文训练对。

评测 benchmark：

- **DPG**：密集 prompt 组合语义；
- **GenEval**：对象、数量、颜色、位置、属性绑定；
- **OneIG-Bench**：英文/中文的 alignment、text、reasoning、style、diversity；
- **T2I-CompBench**：B-VQA、UniDet、S-CoT、空间和数值理解；
- **LongText-Bench**：中英文长文本渲染。

效率指标：

- 参数量；
- GPU memory；
- latency；
- 平均性能下降比例。

Qwen-Image 结果中：

- 20B → 14B：约 79.4% 显存，平均性能下降约 0.42%；
- 20B → 12B：约 71.4% 显存，平均下降约 3.03%；
- 20B → 10B：约 66.9% 显存，平均下降约 6.88%；
- 10B 加额外 fine-tuning 后，平均下降约 3.29%。

FLUX.1-dev 结果中，PPCL-8B 约 74.4% 显存，平均性能下降约 4.03%。论文还指出，PPCL 对过长文本和小尺寸文字仍存在明显局限。

---

## 4. NanoFLUX

论文：[arXiv:2602.06879](https://arxiv.org/abs/2602.06879)

### 4.1 针对的模型

NanoFLUX 针对 **FLUX.1-Schnell**：

- 完整 FLUX pipeline：约 17B；
- diffusion transformer：约 12B；
- T5-XXL text encoder：约 5B；
- NanoFLUX 最终约 2.4B：
  - 约 2B diffusion transformer；
  - 330M T5-Large text encoder。

FLUX.1-Schnell 是 rectified-flow/flow-matching 模型，原本已经通过步数蒸馏支持 4 步推理。NanoFLUX 的重点不是重新做 step distillation，而是压缩网络结构和文本编码器。

### 4.2 Flow matching 基础公式

对真实 latent \(x_0\) 和高斯噪声 \(\epsilon\)：

\[
x_t=(1-t)x_0+t\epsilon
\]

目标速度场：

\[
v_t=\frac{dx_t}{dt}=\epsilon-x_0
\]

模型 \(f(x_t,t,p;\theta)\) 通过 flow matching 目标训练：

\[
\mathcal L_{\rm FM}
=
\mathbb E_{x_0,\epsilon,t}
\left[
\left\|
f(x_t,t,p;\theta)
-
(\epsilon-x_0)
\right\|_2^2
\right]
\]

推理时使用 Euler 更新：

\[
\hat{x}_{j-1}
=
\hat{x}_{j}
+
\Delta t_j
f(\hat{x}_{j},t,p;\theta)
\]

### 4.3 Progressive transformer compression

NanoFLUX 的压缩链条是：

```text
12B transformer
   ↓ 注意力头剪枝
5B
   ↓ 每个 head 的特征维度压缩
3B
   ↓ 深度剪枝 / 合并冗余 block
2.5B
   ↓ Static LayerNorm
1.8B transformer
```

#### 阶段 C1：12B → 5B

对 attention 输出：

\[
\operatorname{softmax}(QK^\top)V
\]

做 SVD 分析。教师有 24 个 attention heads，但实验发现保留前 16 个 head 已能较好重建输出，因此：

- \(H:24\rightarrow16\)；
- hidden dimension：3072 → 2048；
- 参数规模约 12B → 5B。

#### 阶段 C2：5B → 3B

对每个 attention head 的特征做 SVD，只保留主要奇异成分：

- head dimension：128 → 96；
- hidden dimension：2048 → 1536；
- 参数规模约 5B → 3B。

#### 阶段 C3：3B → 2.5B

分析 Transformer block 输入输出的余弦相似度，发现部分 single-stream blocks 的输入输出变化很小，因此对连续冗余层进行深度压缩或参数合并。

#### 阶段 C4：Static LayerNorm

作者观察到 AdaLN 系数在校准集上的方差较小，因此将部分动态 LayerNorm/AdaLN 参数静态化，进一步降低计算和参数开销。

### 4.4 蒸馏损失

每个压缩阶段都采用教师-学生 flow prediction 蒸馏：

\[
\mathcal L_{\rm distill}(x_t)
=
\left\|
f_S(x_t,t,p;\theta_S)
-
f_T(x_t,t,p;\theta_T)
\right\|_2^2
\]

训练方式是渐进式的：

- 5B 主要由 12B 教师蒸馏；
- 3B 先由 5B 教师蒸馏，再由 12B 教师继续校正；
- 2.5B 先由 3B 蒸馏，再由 12B 教师校正；
- 最终版本还要压缩文本编码器。

### 4.5 Progressive Token Downsampling

为降低 MMDiT self-attention 的 \(O(T^2)\) 复杂度，NanoFLUX 使用 ResNet 下采样模块：

\[
D:
\mathbb R^{\sqrt T\times\sqrt T\times d}
\rightarrow
\mathbb R^{\frac{\sqrt T}{2}\times\frac{\sqrt T}{2}\times d}
\]

token 数量从 \(T\) 变为 \(T/4\)。

对应的上采样模块：

\[
U:
\mathbb R^{\frac{\sqrt T}{2}\times\frac{\sqrt T}{2}\times d}
\rightarrow
\mathbb R^{\sqrt T\times\sqrt T\times d}
\]

设计为：

- 前部 block 使用高分辨率 token；
- 中间 23 个 block 使用低分辨率 token；
- 后部恢复到高分辨率；
- 早期高噪声时间步使用下采样；
- 后期细节生成阶段恢复全分辨率。

令时间步阈值为 \(t_{\rm thresh}\)：

- \(t\le t_{\rm thresh}\)：使用 \(D/U\)；
- \(t>t_{\rm thresh}\)：跳过下采样，保持高分辨率。

### 4.6 文本编码器蒸馏

FLUX 使用：

- T5-XXL：5B；
- CLIP-Large：约 120M。

NanoFLUX 将 T5-XXL 压缩到 330M 的 T5-Large。其特点是：不仅让小文本编码器模仿大文本编码器，还利用去噪器早期层的视觉信号辅助文本表示对齐。

训练时主要比较：

- 教师/学生文本 embedding；
- 早期 denoiser hidden states；
- 最终 flow/velocity prediction。

因此 NanoFLUX 的压缩不仅是“剪 transformer”，还包括 **文本条件路径的压缩**。

### 4.7 数据与 Benchmark

训练数据：

- Ye-PoP 数据集，约 480K 图像；
- 用 FLUX.1-Schnell 重新生成训练图像；
- 使用 Qwen-Image 生成短描述、中等描述、长描述、对象标签和细粒度 caption。

主要 benchmark：

- **OneIG-Bench**：Alignment、Style、Diversity；由于训练数据几乎不包含文字渲染，因此不计入 Text 和 Reasoning 类别；
- **DPG-Bench**：复杂 prompt 遵循；
- **GenEval**：对象、数量、颜色、位置、属性绑定；
- **HPDv3 / HPSv3**：人类偏好分数；
- **Samsung S25 Ultra**：真实移动端延迟测试。

最终 NanoFLUX：

- DiT 约 2B；
- T5-Large 330M；
- 10 步推理；
- OneIG 约 42.1；
- DPG 约 75.5；
- GenEval 约 49.7；
- HPSv3 约 10.41。

渐进压缩过程中，模型规模从 12B 降到 1.8B 时，GenEval 从约 66.0 降至约 53.1；加入 token downsampling 和文本编码器压缩后，质量进一步下降，但换来了移动端效率。

论文报告 NanoFLUX 在移动设备上生成 512×512 图像约需 2.5 秒。其核心取舍是：允许一定文本/组合性质量损失，换取显著的模型规模、attention token 数和实际延迟下降。

---

## 四篇论文的核心差异

| 论文 | 主要压缩对象 | 是否动态计算 | 是否做步数蒸馏 | 最核心的技术点 |
|---|---|---:|---:|---|
| Amber-Image | Qwen-Image 的 MMDiT 层和双流结构 | 否 | 主要是特征/层蒸馏 | 时间步敏感层剪枝、局部权重平均、深层双流转单流 |
| Bridging | EDM2-XS U-Net branch/channel | 否 | 是，一步 SiDA | 在剪枝与一步蒸馏之间加入高噪声真实 latent 对齐 |
| PPCL | Qwen-Image/FLUX 的连续层、text stream、FFN | 否 | 结构/特征蒸馏 | CKA 一阶差分寻找连续冗余区间，支持可插拔压缩比例 |
| NanoFLUX | FLUX transformer、attention head、token、T5 编码器 | 部分，按时间步 token 分辨率变化 | 主要是结构蒸馏，不是核心 step distillation | 渐进式 12B→2B 压缩、token 下采样、文本编码器蒸馏、移动端部署 |

从研究侧重点看：

- **Amber-Image** 关注 MMDiT 的架构演化；
- **PPCL** 关注通用的 DiT 结构化剪枝框架；
- **NanoFLUX** 关注端到端移动端部署；
- **Bridging** 关注剪枝和少步/一步蒸馏之间的训练衔接问题。

> 我没看懂，你分别总结一下这四篇文章什么时候用了剪枝？怎么剪的？什么时候进行了蒸馏？怎么蒸馏的？以及为什么要这样做

可以把四篇论文都理解成同一个问题：

> 先删除模型中“可能不重要的部分”，再用一个更大的教师模型教被压缩的学生模型恢复能力。

区别主要在于：剪什么、剪的依据是什么、蒸馏发生在剪枝前还是剪枝后，以及蒸馏的目标是恢复中间特征、噪声预测，还是直接恢复最终图像质量。

---

## 1. Amber-Image

论文：[arXiv:2602.17047](https://arxiv.org/abs/2602.17047)

### 它什么时候剪枝？

Amber-Image 有两次结构压缩。

第一阶段从 Qwen-Image 20B 开始：

```text
Qwen-Image 20B，60层双流 MMDiT
        ↓
删除30层
        ↓
Amber-Image-10B，约30层双流 MMDiT
```

第二阶段从 Amber-Image-10B 开始：

```text
前10层：保留 text/image 双流
后20层：双流改成单流
        ↓
Amber-Image-6B
```

### 怎么剪？

#### 第一次：剪层

对每一层单独做消融：

1. 暂时关闭第 \(l\) 层；
2. 输入相同的 latent、prompt 和时间步；
3. 比较原始模型和删除该层后的噪声预测；
4. 预测变化小的层被认为不重要；
5. 删除重要性最低的 30 层。

重要性大致是：

\[
I_l
=
\mathbb E
\left[
\|\epsilon_{\rm teacher}
-
\epsilon_{\rm layer\ removed}\|_2^2
\right]
\]

注意它不是只在一个时间步上判断，而是考虑多个扩散时间步，并对不同时间步加权。原因是不同时间步负责不同内容：

- 高噪声阶段：决定整体布局和语义；
- 低噪声阶段：决定纹理和细节。

#### 第二次：双流改单流

MMDiT 原来有：

```text
text stream
image stream
```

Amber-Image 保留前 10 层双流，因为前面需要分别理解文本和图像；在后 20 层，文本和图像特征已经高度融合，因此让它们共用一个 stream。

### 什么时候蒸馏？

Amber-Image 在剪枝后进行蒸馏，分两步。

#### Amber-Image-10B 的蒸馏

删除层之后，保留层中有些层需要承担原来多个层的功能。作者先用局部权重平均初始化这些层：

\[
W_l^{init}
=
\frac{1}{k+1}
\sum_{j=0}^{k}W_{l+j}
\]

然后：

1. 冻结原本没有被重新初始化的层；
2. 只训练新初始化的层；
3. 让学生层的 hidden state 接近教师对应区域的 hidden state；
4. 再解冻全部模型，做一次全参数微调。

#### Amber-Image-6B 的蒸馏

双流转单流以后，学生单流需要同时表达原来的 text stream 和 image stream，因此教师目标是：

\[
H_{\rm target}
=
\operatorname{Concat}
(H_{\rm text},H_{\rm image})
\]

学生单流输出通过 MSE 拟合这个拼接后的教师表示：

\[
\mathcal L
=
\|H_{\rm student}-H_{\rm target}\|_2^2
\]

最后再进行轻量全参数微调。

### 为什么这样做？

如果直接删层：

- 剩下的层没有学会承担被删层的功能；
- hidden state 分布会突然改变；
- 生成质量和文本对齐能力会下降。

所以 Amber-Image 的思路是：

> 先用结构剪枝减少计算，再用中间特征蒸馏让保留层学会“代替”被删除层。

它采用“前部双流、后部单流”的原因是：

- 前部需要保留模态独立性；
- 后部文本和图像表示已经高度融合，继续使用双流存在冗余。

---

## 2. PPCL

论文：[arXiv:2511.16156](https://arxiv.org/abs/2511.16156)

PPCL 和 Amber-Image 最大的不同是：它不是简单地“先全部剪完，再统一蒸馏”，而是把剪枝和蒸馏结合成一个结构搜索和训练流程。

### 它什么时候剪枝？

PPCL 有两种剪枝：

```text
第一步：深度剪枝
第二步：宽度剪枝
最后：全参数微调
```

针对的主要模型是：

- Qwen-Image；
- FLUX.1-dev；
- FLUX.1 Lite。

### 怎么剪？

#### 第一步：寻找连续冗余层

PPCL 认为相邻层可能共同完成一个功能，所以重点寻找：

```text
第3～4层
第15～24层
第42～43层
```

这种连续的冗余区间，而不是单独删除随机层。

它的流程是：

1. 给每个教师层训练一个轻量 linear probe；
2. 用 probe 模拟连续层被替代后的输出；
3. 比较模拟输出和原始教师输出的 CKA 相似度；
4. 观察 CKA 变化趋势；
5. CKA 变化先减小、后增大的位置，被认为是冗余区间边界；
6. 删除这些连续区间。

可以简单理解为：

> 如果连续几层被一个简单的线性映射近似后，整体表示变化仍然很小，那么这些层可能是冗余的。

#### 第二步：宽度剪枝

PPCL 还压缩两类宽度：

1. **text stream 的冗余部分**；
2. **FFN 的冗余部分**。

对于 text stream，不是完全删除整个层，而是保留 QKV，将其他复杂计算替换成轻量线性投影：

\[
z_p^{txt}=l_p^z(z_r^{txt}),
\qquad
h_p^{txt}=l_p^h(h_r^{txt})
\]

对于 FFN，也用较小的线性投影代替原始 FFN：

\[
h_q^{img}=l_q^{img}(g_q^{img}),
\qquad
h_q^{txt}=l_q^{txt}(g_q^{txt})
\]

### 什么时候蒸馏？

PPCL 在每次结构压缩时都进行蒸馏。

#### 连续层剪枝时

假设删除教师的第 \(u\) 到 \(v\) 层，学生保留一个对应层，让它直接学习：

- 删除前的教师表示；
- 删除区间结束后的教师表示。

损失为：

\[
\mathcal L_{\rm depth}
=
\left\|
\operatorname{Norm}(H_{\rm student})
-
\operatorname{Norm}(H_{\rm teacher}^{v})
\right\|_2^2
\]

这里使用归一化是为了减少不同层特征尺度不同带来的影响。

#### 宽度剪枝时

学生线性投影不仅要匹配教师的最终输出，还要匹配中间表示：

\[
\mathcal L_{\rm width}
=
\mathcal L_{\rm feature}
+
\mathcal L_{\rm linear}
\]

其中：

- \(\mathcal L_{\rm feature}\)：学生和教师层输出的差异；
- \(\mathcal L_{\rm linear}\)：被替代的 text/FFN 表示差异。

最后再进行全参数微调。

### 为什么这样做？

PPCL 主要想解决两个问题。

#### 为什么剪连续层？

相邻层通常有功能联系，删除连续层并不一定比删除随机层更危险。反过来，连续层之间可能存在功能重复，因此连续删除更容易形成实际的结构压缩。

#### 为什么要用非顺序蒸馏？

如果按传统方式一层一层蒸馏：

```text
学生第1层 → 教师第1层
学生第2层 → 教师第2层
学生第3层 → 教师第3层
```

早期层的小误差会不断传递，最后导致累积偏差。

PPCL 让一个学生层直接学习一个连续教师区间的效果：

```text
学生一层 → 教师连续多层
```

这样可以减少误差累积。

#### 为什么称为 Pluggable？

PPCL 可以通过重新插入部分教师层，得到不同模型规模：

```text
20B → 14B
20B → 12B
20B → 10B
```

不需要为每一个压缩比例重新设计完整算法。

---

## 3. NanoFLUX

论文：[arXiv:2602.06879](https://arxiv.org/abs/2602.06879)

NanoFLUX 的压缩过程最像“逐级缩小模型”。它不是一次性剪掉很多模块，而是每压缩一级，就蒸馏一次。

### 它什么时候剪枝？

整体流程是：

```text
FLUX transformer 12B
        ↓ 剪 attention heads
5B
        ↓ 剪 attention 特征维度
3B
        ↓ 剪/合并冗余 block
2.5B
        ↓ 静态化 AdaLN
1.8B
        ↓ 加入 token downsampling
约2B NanoFLUX
        ↓
T5-XXL 5B → T5-Large 330M
```

最终完整 pipeline 约 2.4B 参数。

### 怎么剪？

#### 第一步：剪 attention heads

FLUX.1-Schnell 的 transformer 有 24 个 attention heads。作者对 attention 输出做 SVD：

\[
\operatorname{softmax}(QK^\top)V
\]

发现保留前 16 个 head 仍然可以近似原始输出，因此：

```text
24 heads → 16 heads
3072 hidden dimension → 2048
12B → 5B
```

#### 第二步：剪每个 head 的特征维度

5B 模型中，每个 head 原本维度为 128。作者继续对每个 head 的特征做 SVD，保留主要奇异成分：

```text
head dimension：128 → 96
5B → 3B
```

#### 第三步：删除或合并冗余 block

分析 transformer block 的输入输出余弦相似度：

\[
\cos(h_{\rm input},h_{\rm output})
\]

如果一个 block 的输入和输出非常相似，说明该 block 进行的变换较小，可能具有冗余。

因此：

```text
3B → 2.5B
```

#### 第四步：静态化 LayerNorm/AdaLN

作者观察到 AdaLN 系数在校准样本上的变化较小，因此将部分动态系数预先计算，减少推理时的额外开销：

```text
2.5B → 1.8B
```

#### 第五步：token downsampling

在中间 transformer block 使用 ResNet 下采样：

\[
T\rightarrow T/4
\]

也就是将二维 token 网格的高和宽各缩小一半，token 总数变成原来的四分之一。

但不是所有时间步都下采样：

- 高噪声早期阶段：主要生成大致结构，可以使用低分辨率 token；
- 低噪声后期阶段：需要生成纹理和文字，恢复高分辨率 token。

### 什么时候蒸馏？

NanoFLUX 每完成一次结构压缩，就进行教师-学生蒸馏。

统一的核心损失是 flow prediction 蒸馏：

\[
\mathcal L_{\rm distill}
=
\left\|
f_S(x_t,t,p)
-
f_T(x_t,t,p)
\right\|_2^2
\]

蒸馏关系是逐级变化的：

```text
5B 学 12B
3B 先学 5B，再学 12B
2.5B 先学 3B，再学 12B
1.8B 继续由大模型校正
```

这样做的原因是：如果 12B 直接教 1.8B，学生容量太小，训练会困难；逐级蒸馏能降低教师和学生之间的能力差距。

### 文本编码器怎么蒸馏？

FLUX 的 5B T5-XXL 太大，因此 NanoFLUX 还将：

```text
T5-XXL 5B → T5-Large 330M
```

文本编码器蒸馏不仅比较文本 embedding，还利用去噪器早期层的视觉隐藏状态，使小文本编码器生成的条件更加适合压缩后的视觉 transformer。

### 为什么这样做？

NanoFLUX 同时面临三种瓶颈：

1. transformer 参数量太大；
2. attention token 太多，计算复杂度高；
3. T5-XXL 文本编码器本身也很大。

因此它分别处理：

- attention head 冗余；
- head 内部特征维度冗余；
- transformer block 冗余；
- AdaLN 动态计算；
- token 数量；
- 文本编码器规模。

它不是单纯追求最低 FID，而是追求：

> 在移动设备上，以可接受的图像质量实现低延迟。

---

## 4. Bridging 与其他三篇的关键区别

这篇论文的剪枝和蒸馏顺序最明确：

```text
先剪枝
  ↓
教师对齐修复
  ↓
一步蒸馏
```

而其他三篇基本是：

```text
剪枝
  ↓
特征/输出蒸馏
  ↓
微调
```

Bridging 特别指出，直接这样做：

```text
剪枝模型 → 一步蒸馏
```

通常会失败，因为剪枝模型还没有处于教师可蒸馏的局部区域。

所以它加入：

```text
剪枝模型 → 在真实 noisy latent 上匹配教师 → 一步蒸馏
```

这里的 repair 不是完整训练，也不是让模型恢复成一个多步扩散模型，而只是让它在一步生成的高噪声起点附近先和教师对齐。

---

## 最简对照

| 论文 | 什么时候剪 | 怎么剪 | 什么时候蒸馏 | 蒸馏什么 | 为什么 |
|---|---|---|---|---|---|
| Amber-Image | 先剪层，再双流改单流 | 时间步敏感层消融；后部双流改单流 | 每次结构变化后 | hidden states、噪声预测、标准扩散目标 | 让保留层承担被删层功能 |
| PPCL | 深度剪枝后再宽度剪枝 | CKA 一阶差分找连续冗余区间；线性投影替代 text/FFN | 剪枝过程中同步进行 | 连续层输出、中间特征、线性投影输出 | 避免层间误差累积，并支持多种压缩比例 |
| NanoFLUX | 逐级压缩 | attention head、head dimension、block、AdaLN、token、T5 | 每一级压缩后 | flow/velocity prediction，文本和视觉表示 | 逐级降低师生能力差距，适合移动端 |
| Bridging | 先剪 U-Net | teacher-aware block/channel pruning | 剪枝后先 repair，再一步 SiDA | 高噪声真实 latent 上的教师 denoising 输出，随后生成器输出 | 解决“剪枝模型不能直接进行一步蒸馏”的初始化问题 |

> 1.局部权重平均初始化是个什么操作？我没懂Amber-Image-10B 的蒸馏过程。\(W_l^{init}
> =
> \frac{1}{k+1}
> \sum_{j=0}^{k}W_{l+j}\)是谁的层？教师还是学生？教师从哪里来？学生从哪里来？教师目标又是什么意思？ 2.NanoFLUX 每完成一次结构压缩，就进行教师-学生蒸馏。 这个时候教师和学生是指什么？ 4.你还没见过Bridging的剪枝蒸馏技术路线

你问到的是这三篇里最容易混淆的地方。核心是区分：

- **原始教师模型**
- **剪枝后学生模型**
- **学生的初始化权重**
- **蒸馏目标**

---

# 1. Amber-Image-10B：局部权重平均和蒸馏过程

论文：[arXiv:2602.17047](https://arxiv.org/abs/2602.17047)

## 1.1 教师和学生分别是什么？

原始模型是 Qwen-Image：

```text
教师模型 T：原始 Qwen-Image
20B 参数，60 层双流 MMDiT
```

学生模型是从教师模型剪出来的：

```text
学生模型 S：剪枝后的 Qwen-Image
约 10B 参数，约 30 层双流 MMDiT
```

教师模型保留完整的 60 层，参数被冻结；学生模型只有 30 层，需要重新训练。

```text
完整教师 T
60层，冻结
   ↓ 剪枝和初始化
学生 S
30层，可训练
```

论文明确说明：Amber-Image-10B 是从 Qwen-Image 删除 30 层后得到的，教师仍然是完整 Qwen-Image。

---

## 1.2 “局部权重平均”到底做了什么？

假设原始教师有如下连续层：

```text
教师层：
L1, L2, L3, L4, L5, L6
```

如果经过重要性评估后，决定删除：

```text
删除 L3、L4
```

那么学生不能简单变成：

```text
学生层：
L1, L2, L5, L6
```

因为原始的 L2 后面原本紧接着 L3、L4、L5，现在一下跳到了 L5，功能可能不连续。

Amber-Image 对保留下来的邻近层进行初始化融合：

```text
学生第2层 ≈ 平均(教师 L2, L3, L4)
```

公式是：

\[
W_l^{init}
=
\frac{1}{k+1}
\sum_{j=0}^{k}W_{l+j}^{orig}
\]

这里：

- \(W_{l+j}^{orig}\)：**原始完整教师模型中的权重**；
- \(W_l^{init}\)：**学生模型第 \(l\) 层的初始化权重**；
- \(k\)：该学生层后面连续被删除的教师层数量。

如果 L3、L4 被删除，那么 \(k=2\)：

\[
W_2^{student,init}
=
\frac{
W_2^{teacher}
+
W_3^{teacher}
+
W_4^{teacher}
}{3}
\]

因此，这个公式右边的 \(W\) 都来自教师；公式左边是学生层初始化后的权重。

更准确地说：

> 局部权重平均不是在训练过程中平均教师和学生，而是在构造学生模型时，用教师相邻层的权重平均值初始化学生保留下来的层。

论文给出的形式正是将保留层及其后面连续被删除层的原始权重做平均。

---

## 1.3 为什么是“后面的层”也参与平均？

假设删除的是教师层 \(L_3,L_4\)，那么保留层 \(L_2\) 需要承担一部分原来由 \(L_3,L_4\) 完成的变换，因此使用：

\[
L_2^{student}
\leftarrow
\operatorname{Average}(L_2^{teacher},L_3^{teacher},L_4^{teacher})
\]

这相当于把连续的多个 Transformer block 合并成一个“平均 block”。

它不是严格的函数等价，因为：

\[
F_4(F_3(F_2(x)))
\neq
F_{\rm avg}(x)
\]

但相邻层通常承担逐步 refinement 功能，权重平均可以给学生提供一个比随机初始化或直接复制更好的起点。

---

## 1.4 Amber-Image-10B 的完整蒸馏过程

### 第一步：完整教师模型评估层重要性

对原始 Qwen-Image 的每一层做一次消融：

```text
教师 T 正常运行
教师 T 删除第 l 层后运行
比较两个噪声预测
```

重要性定义可以写成：

\[
I_l
=
\mathbb E
\left[
\omega_t
\left\|
\epsilon_T(z_t,t,p)
-
\epsilon_{T\setminus l}(z_t,t,p)
\right\|_2^2
\right]
\]

重要性低的层被选中删除。

### 第二步：构造学生模型

例如：

```text
教师：60层
删除：30层
学生：30层
```

构造学生时：

- 没有被删除的层，直接从教师复制；
- 如果某个保留层后面紧跟若干被删除层，则用局部权重平均初始化；
- 教师完整模型保持不变。

可以表示为：

```text
教师 T：
L1 → L2 → L3 → L4 → L5 → L6

删除 L3、L4 后：

学生 S：
L1' → L2' → L5' → L6'

其中：
L1' = T.L1
L2' = average(T.L2, T.L3, T.L4)
L5' = T.L5
L6' = T.L6
```

### 第三步：只训练“新初始化”的学生层

局部平均得到的层并不一定已经正确，因此先进行 targeted distillation：

```text
教师 T：完整60层，冻结
学生 S：30层
只训练局部权重平均产生的学生层
其他直接复制的学生层先冻结
```

教师目标不是教师的参数，而是教师的**中间 hidden state**。

例如，教师执行到被删除区间的末端 \(L_4\)，得到：

\[
H_T^{(4)}
\]

学生用一个合并后的层执行相应部分，得到：

\[
H_S^{(2)}
\]

训练目标是：

\[
\mathcal L_{\rm hidden}
=
\left\|
H_S^{(2)}
-
H_T^{(4)}
\right\|_2^2
\]

也就是说：

> 学生的一层，要学会近似教师连续多层之后的表示。

这里“教师目标”指的是教师模型在相同输入、相同 latent、相同时间步、相同 prompt 下产生的目标 hidden state，而不是一个新的标签数据集。

### 第四步：全参数微调

当重新初始化的学生层已经基本对齐后：

```text
解冻学生所有层
继续训练学生
使用标准 diffusion loss
```

这一步让：

- 原封不动复制的层；
- 局部平均初始化的层；
- 剪枝后改变了连接关系的层；

重新协调到同一个表示空间中。

因此完整流程是：

```text
完整 Qwen-Image 教师 T
        ↓
层消融评估
        ↓
删除30层
        ↓
从 T 构造学生 S
        ↓
局部权重平均初始化 S 的部分层
        ↓
冻结 T
        ↓
只蒸馏被重新初始化的 S 层
        ↓
解冻 S 全部层
        ↓
标准 diffusion loss 微调
        ↓
Amber-Image-10B
```

---

## 1.5 “教师目标”到底是什么意思？

这里有三种可能的目标，不能混为一谈：

### 参数目标

不属于蒸馏损失。

例如：

\[
W_l^{student}
\leftarrow
\frac{W_l^{teacher}+W_{l+1}^{teacher}}{2}
\]

这是初始化操作，不是训练目标。

### 中间特征目标

Amber-Image 的 targeted distillation 使用的是这一类：

\[
H_S^{(l)}
\approx
H_T^{(v)}
\]

学生层输出去拟合教师相应层或删除区间末端的 hidden state。

### 扩散输出目标

全参数微调阶段使用标准扩散目标，例如噪声/速度预测：

\[
\mathcal L_{\rm diff}
=
\left\|
\epsilon_S(z_t,t,p)
-
\epsilon_{\rm target}
\right\|_2^2
\]

因此可以概括为：

```text
局部权重平均：初始化学生参数
层级蒸馏：匹配教师中间特征
全参数微调：恢复最终扩散生成能力
```

---

# 2. NanoFLUX：每一阶段的教师和学生是谁？

论文：[arXiv:2602.06879](https://arxiv.org/abs/2602.06879)

NanoFLUX 使用的是“渐进式教师-学生压缩”，不是一次性让 2B 模型直接学习 12B 模型。

## 2.1 第一阶段：12B → 5B

```text
教师：FLUX.1-Schnell 的 12B diffusion transformer
学生：剪掉 attention heads 后的 5B transformer
```

压缩方式：

```text
24 attention heads → 16 heads
hidden dimension 3072 → 2048
```

然后训练学生，让它模仿 12B 教师：

\[
\mathcal L_{\rm distill}
=
\left\|
f_{5B}(x_t,t,p)
-
f_{12B}(x_t,t,p)
\right\|_2^2
\]

这里的 \(f\) 是 flow/velocity prediction。

---

## 2.2 第二阶段：5B → 3B

```text
教师：已经训练好的 5B 学生
学生：进一步缩小后的 3B 模型
```

压缩方式：

```text
每个 attention head 的特征维度：
128 → 96
```

训练时论文使用两级教师：

1. 先让 3B 学习 5B；
2. 再让 3B 继续接受原始 12B 教师校正。

所以是：

```text
3B 学 5B
3B 再学 12B
```

为什么不直接只学 12B？

因为：

- 5B 和 3B 的容量更接近；
- 训练初期更容易收敛；
- 原始 12B 可以在后面纠正逐级压缩造成的误差。

---

## 2.3 第三阶段：3B → 2.5B

```text
教师：训练好的 3B 模型
学生：删除或合并冗余 block 后的 2.5B 模型
```

之后继续由 12B FLUX 教师进行校正：

```text
2.5B 学 3B
2.5B 再学 12B
```

论文中的逻辑是：

```text
直接 12B → 2.5B：能力差距太大
先 12B → 5B → 3B → 2.5B：逐级减小能力差距
```

---

## 2.4 第四阶段：2.5B → 1.8B

```text
教师：2.5B 模型
学生：静态化 AdaLN 后的 1.8B 模型
```

这一步不是典型的“删除整个层”，而是利用 AdaLN 系数在校准集上变化较小的特点，将部分动态调制预计算或静态化。

仍然需要蒸馏，使静态化后的学生 flow prediction 接近教师。

---

## 2.5 Token downsampling 阶段的教师和学生

Token downsampling 不是重新构造一个完全独立的模型，而是在 2.5B 或 1.8B 模型中加入：

```text
高分辨率 token
    ↓ ResNet downsampling
低分辨率 token，数量变为1/4
    ↓ 中间 block 处理
ResNet upsampling
    ↓
恢复高分辨率 token
```

这一阶段：

```text
教师：未使用 token downsampling 的 2.5B/1.8B 模型
学生：加入 token downsampling 的对应模型
```

学生需要模仿教师的 flow 输出和图像质量。

这里的重点不是减少参数，而是减少 attention 的 token 数量，从而降低：

\[
O(T^2)
\]

的 self-attention 计算量。

论文还根据时间步决定是否下采样：

- 早期高噪声：可以低分辨率；
- 后期低噪声：恢复高分辨率，保留纹理和文字细节。

---

## 2.6 文本编码器蒸馏阶段

FLUX 的文本编码器是 T5-XXL：

```text
教师：T5-XXL，约5B
学生：T5-Large，约330M
```

学生文本编码器不仅需要输出接近教师的文本 embedding，还要让后面的压缩 diffusion transformer 能正确使用这些文本条件。

因此可以理解为：

```text
T5-XXL 生成文本条件
        ↓
T5-Large 学习近似这些文本条件
        ↓
压缩后的 diffusion transformer 使用 T5-Large
```

论文还利用去噪器早期层的视觉 hidden state 辅助文本编码器蒸馏，使文本表示更适合图像生成。

---

## 2.7 NanoFLUX 的教师学生关系总结

| 阶段 | 教师 | 学生 |
|---|---|---|
| 12B → 5B | FLUX.1-Schnell 12B DiT | 5B，减少 attention heads |
| 5B → 3B | 已训练 5B，后续还有 12B 校正 | 3B，降低 head dimension |
| 3B → 2.5B | 已训练 3B，后续还有 12B 校正 | 2.5B，合并/删除冗余 block |
| 2.5B → 1.8B | 2.5B | 静态化 AdaLN 后的 1.8B |
| Token downsampling | 未下采样版本 | 加入 token 下采样版本 |
| 文本编码器 | T5-XXL 5B | T5-Large 330M |

所以 NanoFLUX 的核心不是：

```text
一个教师 → 一个学生
```

而是：

```text
12B → 5B → 3B → 2.5B → 1.8B
```

每一步的上一代模型通常是近距离教师，原始 12B 模型则在若干阶段作为全局质量教师。

---

# 3. Bridging：完整的剪枝-蒸馏技术路线

论文：[arXiv:2607.06335](https://arxiv.org/abs/2607.06335)

这篇论文的路线与 Amber-Image、NanoFLUX 不同。它关注的不是普通的“剪枝后恢复多步模型”，而是：

> 剪枝后能不能直接把模型变成一步生成器？

答案是：

```text
剪枝 → 直接一步蒸馏：失败
剪枝 → 教师对齐修复 → 一步蒸馏：成功
```

## 3.1 教师和学生是谁？

```text
教师 T：完整 EDM2-XS
学生 G：剪枝后的 compact generator
最终学生 G*：一步生成模型
```

教师 EDM2-XS：

- 约 124.713M 参数；
- 需要 63 次 network evaluations；
- ImageNet-512 FID 3.53。

学生：

- 结构上缩小 U-Net；
- 目标是一次 network evaluation 生成图像；
- 不是先恢复成一个小型 63-step diffusion model。

---

## 3.2 第一步：构造候选学生架构

Bridging 使用两类结构压缩：

### 通道宽度压缩

例如将原始各分辨率通道宽度：

\[
(c_1,c_2,c_3,c_4)
\]

替换为更小的配置，例如：

\[
(104,208,384,512)
\]

或：

\[
(104,208,312,416)
\]

教师权重通过切片复制到学生对应通道。

### 分支/block 删除

原始 block 可以简化表示为：

\[
h_{b+1}
=
\operatorname{MPsum}
\big(
h_b,R_b(h_b,e)
\big)
+
A_b(h_b)
\]

其中：

- \(R_b\)：residual branch；
- \(A_b\)：attention branch；
- \(e\)：类别/噪声 embedding。

剪枝后，某些 block 的 residual 或 attention branch 被实际删除，而不是保留零权重：

```text
原始：
skip + residual + attention

剪枝后：
只保留 skip/resample 路径
```

这可以真正减少推理时的参数和计算。

---

## 3.3 第二步：用教师对齐误差决定剪谁

先用真实 ImageNet 图像构造 noisy latent：

\[
z_i=E(x_i)
\]

\[
\tilde z_i=z_i+\sigma^\star\epsilon_i
\]

教师目标：

\[
t_i=T(\tilde z_i,\sigma^\star,y_i)
\]

完整候选学生和教师之间的误差：

\[
\hat\varepsilon_{\sigma^\star}(G)
=
\frac{1}{N}
\sum_i
\left\|
G(\tilde z_i,\sigma^\star,y_i)-t_i
\right\|_2^2
\]

对每个候选 block \(b\)，临时删除它，得到 \(G^{-b}\)，计算：

\[
I(b)
=
\frac{1}{N}\sum_i
\left[
\|G^{-b}(\tilde z_i,\sigma^\star,y_i)-t_i\|_2^2
-
\|G(\tilde z_i,\sigma^\star,y_i)-t_i\|_2^2
\right]
\]

含义是：

- \(I(b)\) 小：删掉 block 后仍然接近教师，适合剪；
- \(I(b)\) 大：删掉 block 后偏离教师很多，不应剪。

这和普通按权重大小剪枝不同。它衡量的是：

> 删除某个 block 后，学生与教师在实际去噪状态上的关系会恶化多少。

---

## 3.4 第三步：为什么不能直接一步蒸馏？

直接路线是：

```text
完整教师 T
    ↓ 剪枝
剪枝模型 G0
    ↓ 直接 SiDA
一步生成器
```

作者发现这样会失败：

- 剪枝模型的 denoising field 已经严重偏离教师；
- SiDA 看到的学生初始状态不在有效蒸馏区域；
- 训练早期样本崩溃；
- FID 可能达到 112.56。

论文用局部教师对齐误差表示这个问题：

\[
\varepsilon(G;\mu)
=
\mathbb E_{(\tilde z,\sigma,y)\sim\mu}
\left[
\|G(\tilde z,\sigma,y)-T(\tilde z,\sigma,y)\|_2^2
\right]
\]

如果剪枝后的 \(G_0\) 在蒸馏使用的 noisy latent 分布上误差很大，那么一步蒸馏的梯度就不可靠。

---

## 3.5 第四步：Teacher-aligned repair

Repair 的目的不是重新训练完整扩散模型，而是短暂地让剪枝学生靠近教师。

真实 latent：

\[
z=E(x)
\]

加噪：

\[
\tilde z=z+\sigma_r\epsilon
\]

修复损失：

\[
\mathcal L_{\rm repair}
=
\mathbb E
\left[
\left\|
G_{\mathcal S}(\tilde z,\sigma_r,y)
-
T(\tilde z,\sigma_r,y)
\right\|_2^2
\right]
\]

参数更新：

\[
\theta_{\rm align}
=
\theta_{\mathcal S}
-
\tau\nabla_\theta
\mathcal L_{\rm repair}
\]

其中：

\[
\sigma_r
=
\max
\{
\sigma_{\rm init},\sigma_1,\ldots,\sigma_K
\}
\]

ImageNet 实验中：

\[
\sigma_r=\sigma_{\rm init}=2.5
\]

也就是说，repair 专门对齐一步生成的高噪声起点。

重要区别：

```text
Repair 不是：
剪枝后重新训练一个63步扩散模型

Repair 是：
用少量更新，让剪枝模型在高噪声区域先接近教师
```

---

## 3.6 第五步：修复后进行一步 SiDA 蒸馏

repair 完成后：

```text
G0
 ↓ 高噪声真实 latent 上的教师对齐
Galign
 ↓ SiDA
G*
```

初始化：

\[
\theta_0\leftarrow\theta_{\rm align}
\]

再优化：

\[
\theta^\star
=
\arg\min_{\theta:\theta_0=\theta_{\rm align}}
\mathcal D(G_\theta;T,\mathcal A)
\]

论文使用 SiDA 作为一步蒸馏方法。教师的 true-score network 冻结，fake-score network 用于训练生成器。

可以把 SiDA 简化理解为：

```text
学生一次生成样本
        ↓
在噪声空间中加噪
        ↓
比较教师 score 和 fake-score
        ↓
更新学生，使其生成分布接近教师
```

论文给出的简化生成器损失包含：

\[
T(\hat z_\sigma,\sigma,y)
-
F_\psi(\hat z_\sigma,\sigma,y)
\]

即教师 score 和 fake score 的差异。

---

## 3.7 Bridging 的完整路线

```text
完整 EDM2-XS 教师 T
124.713M，63 NFE
        ↓
准备真实 ImageNet 校准图像
        ↓
编码成 latent 并加噪
        ↓
计算每个 block 的 teacher-alignment sensitivity
        ↓
删除低重要性 branch/block/channel
        ↓
复制或切片教师权重
        ↓
得到剪枝模型 G0
        ↓
直接 SiDA 会失败
        ↓
在真实 noisy latent、σ=2.5 上做 repair
        ↓
得到 Galign
        ↓
以 Galign 初始化 SiDA
        ↓
训练一步生成器 G*
        ↓
1 NFE
```

最终结果：

| 模型 | 参数量 | NFE | FID |
|---|---:|---:|---:|
| EDM2-XS 教师 | 124.713M | 63 | 3.53 |
| 20% 剪枝 + repair + SiDA | 98.826M | 1 | 3.12 |
| 30% 剪枝 + repair + SiDA | 88.029M | 1 | 4.26 |

这篇论文的真正贡献不是某一种新的 block importance，而是发现并解决了：

> 剪枝学生虽然结构上已经变小，但它还没有进入适合一步蒸馏的参数区域。

Repair 就是这个“进入有效蒸馏区域”的过渡步骤。

---

# 四篇论文最终对照

| 论文 | 教师 | 学生 | 剪枝时机 | 蒸馏时机 | 蒸馏目标 |
|---|---|---|---|---|---|
| Amber-Image | 完整 Qwen-Image 20B | 30层/10B，再到6B | 先剪层，再双流改单流 | 每次结构压缩后 | hidden states + diffusion loss |
| PPCL | 完整 Qwen-Image 或 FLUX | 不同规模的 DiT 学生 | 连续层剪枝，再 text/FFN 宽度剪枝 | 剪枝过程中同步 | 连续层输出、中间特征、线性投影 |
| NanoFLUX | 上一级更大的压缩模型，必要时原始12B校正 | 5B、3B、2.5B、1.8B、最终2B | 逐级剪 attention/head/block/token/encoder | 每一级压缩后 | flow/velocity prediction、文本/视觉表示 |
| Bridging | 完整 EDM2-XS | 剪枝后的一步生成器 | 先按教师对齐误差剪 block/channel | 先 repair，再一步 SiDA | noisy latent 上的教师 denoising 输出，再生成分布 |

最关键的区别可以简化成：

```text
Amber-Image：
剪枝 → 中间特征蒸馏 → 全参数微调

PPCL：
连续层/宽度剪枝 ↔ 局部蒸馏 → 全参数微调

NanoFLUX：
12B → 5B → 3B → 2.5B → 1.8B
每一级压缩后蒸馏

Bridging：
剪枝 → 教师对齐 repair → 一步蒸馏
```
