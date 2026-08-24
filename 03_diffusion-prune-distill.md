# 03 — Diffusion 模型剪枝 + 蒸馏 SOTA

> 时间范围 2022–2026。核验方式：arXiv API（`export.arxiv.org`）逐条元数据验证。
> **状态**：方向 C（最核心）9 篇 + Bridging 全部核验完成；方向 B 核验 7 篇；方向 B/A 剩余条目因 arXiv + Semantic Scholar 双重 429 限流待补。

## 关键：必须分清三个方向（否则会混淆）

在 diffusion 文献里，「蒸馏」一词被占用为**少步推理加速**（方向B），与「剪枝后蒸馏恢复」的语义不同。真正切题的是**方向C**。

```
方向A：剪枝压缩 diffusion       → 压缩 U-Net/DiT 参数量/结构
方向B：蒸馏加速（少步推理）      → 把多步采样蒸馏成 1~4 步（diffusion 主流"蒸馏"指这个）
方向C：剪枝 + 蒸馏联合          → 先剪枝压缩再用蒸馏恢复/加速（本课题最核心，专门工作 2024 前稀少）
```

---

## 方向 C：剪枝 + 蒸馏联合（最核心，全部已核验 ✅）

**专门工作 2024 前稀少，2026 随 DiT / 视频架构爆发。** 下表 10 条均经 arXiv API 逐条核验，标题/作者/日期可直接引用。

| # | 方法 | 完整标题 | 作者 | 日期 | arXiv | 核验 |
|---|---|---|---|---|---|---|
| 1 | **SnapFusion** | SnapFusion: Text-to-Image Diffusion Model on Mobile Devices within Two Seconds | Yanyu Li, Huan Wang, Qing Jin, Ju Hu, Pavlo Chemerys, Yun Fu et al. | 2023-06 | [2306.00980](https://arxiv.org/abs/2306.00980) | ✅ |
| 2 | **LAPTOP-Diff** | LAPTOP-Diff: Layer Pruning and Normalized Distillation for Compressing Diffusion Models | Dingkun Zhang, Sijia Li, Chen Chen, Qingsong Xie, Haonan Lu | 2024-04 | [2404.11098](https://arxiv.org/abs/2404.11098) | ✅ |
| 3 | **BK-SDM** | BK-SDM: A Lightweight, Fast, and Cheap Version of Stable Diffusion | Bo-Kyeong Kim, Hyoung-Kyu Song, Thibault Castells, Shinkook Choi | 2023-05 | [2305.15798](https://arxiv.org/abs/2305.15798) | ✅ |
| 4 | **PPCL / Pluggable Pruning DiT** | Pluggable Pruning with Contiguous Layer Distillation for Diffusion Transformers | Jian Ma, Qirong Peng, Xujie Zhu, Peixing Xie, Chen Chen, Haonan Lu | 2025-11（CVPR 2026） | [2511.16156](https://arxiv.org/abs/2511.16156) | ✅ |
| 5 | **NanoFLUX** | NanoFLUX: Distillation-Driven Compression of Large Text-to-Image Generation Models for Mobile Devices | Ruchika Chavhan, Malcolm Chadwick, Alberto Gil Couto Pimentel Ramos, Luca Morreale, Mehdi Noroozi, Abhinav Mehrotra | 2026-02 | [2602.06879](https://arxiv.org/abs/2602.06879) | ✅ |
| 6 | **Amber-Image** | Amber-Image: Efficient Compression of Large-Scale Diffusion Transformers | Chaojie Yang, Tian Li, Yue Zhang, Jun Gao | 2026-02 | [2602.17047](https://arxiv.org/abs/2602.17047) | ✅ |
| 7 | **PARE** | PARE: Pruning and Adaptive Routing for Efficient Video Generation | Yutong Wang, Yunke Wang, Tianfan Xue, Yu Qiao, Yaohui Wang, Xinyuan Chen et al. | 2026-05 | [2605.27336](https://arxiv.org/abs/2605.27336) | ✅ |
| 8 | **Dynamic-in-Few-Step** | Dynamic-in-Few-Step: Unifying Dynamic Computation and Few-Step Distillation for Efficient Video Generation | Yu Cheng, Siyue Yao, Zhongang Qi, Shanyan Guan, Wei Li, Fajie Yuan | 2026-07 | [2607.06631](https://arxiv.org/abs/2607.06631) | ✅ |
| 9 | **MobileWan** | MobileWan: Closing the Quality Gap for Mobile Video Diffusion | Mohsen Ghafoorian, Denis Korzhenkov, Adil Karjauv, Ioannis Lelekas, Noor Fathima, Spyridon Stasis et al.（共 12 人） | 2026-07 | [2607.06173](https://arxiv.org/abs/2607.06173) | ✅ |
| 10 | **Bridging（最切题）** | Bridging Diffusion Pruning and Step Distillation with Teacher-Aligned Repair | Jincheng Ying, Li Wenlin, Minghui Xu, Yinhao Xiao | 2026-07 | [2607.06335](https://arxiv.org/abs/2607.06335) | ✅ |

### 关键区分
- **仅 *Dynamic-in-Few-Step* 与 *Bridging* 真正联合训练**（动态计算+少步蒸馏 / 剪枝+步蒸馏一体），其余 SnapFusion / LAPTOP-Diff / BK-SDM 等多为「先剪后蒸」两阶段管线。
- **PPCL（#4）即 [02](02_iterative-prune-distill.md) 子方向2 #19 的 Pluggable Pruning DiT**，同一篇（2511.16156, CVPR 2026），跨两个报告出现，此处统一标注。
- **caching ≠ pruning ≠ distillation**：DeepCache / Learning-to-Cache / T-GATE 是 caching（缓存中间特征复用），既非剪枝也非蒸馏，已从方向C剔除。

---

## 方向 B：蒸馏加速（diffusion 的主流「蒸馏」谱系）

从 Progressive Distillation → CM / LCM / Guided Distillation → ADD / DMD → DMD2 / CTM / sCM / Mean Flows / Shortcut / Hyper-SD / PCM 全链路。

| 方法 | 完整标题 | 作者 | 日期/Venue | arXiv | 核验 |
|---|---|---|---|---|---|
| **Progressive Distillation** | Progressive Distillation for Fast Sampling of Diffusion Models | Salimans & Ho | ICLR 2022 | [2202.00512](https://arxiv.org/abs/2202.00512) | ✅ |
| **Consistency Models (CM)** | Consistency Models | Yang Song, Prafulla Dhariwal, Mark Chen, Ilya Sutskever | 2023-03 / ICML 2023 | [2303.01469](https://arxiv.org/abs/2303.01469) | ✅ |
| **LCM** | Latent Consistency Models: Synthesizing High-Resolution Images with Few-Step Inference | Simian Luo, Yiqin Tan, Longbo Huang, Jian Li, Hang Zhao | 2023-10 | [2310.04378](https://arxiv.org/abs/2310.04378) | ✅ |
| **Guided Distillation** | On Distillation of Guided Diffusion Models | Chenlin Meng, Robin Rombach, Ruiqi Gao, Diederik P. Kingma, Stefano Ermon, Jonathan Ho | 2022-10 / CVPR 2023 | [2210.03142](https://arxiv.org/abs/2210.03142) | ✅ |
| **ADD** | Adversarial Diffusion Distillation | Axel Sauer, Dominik Lorenz, Andreas Blattmann, Robin Rombach | 2023-11 | [2311.17042](https://arxiv.org/abs/2311.17042) | ✅ |
| **DMD** | One-Step Diffusion with Distribution Matching Distillation | Yin et al. | ICML 2024 | [2311.18828](https://arxiv.org/abs/2311.18828) | ✅ |
| **CTM** | Consistency Trajectory Models | — | 2023 | [2310.02279](https://arxiv.org/abs/2310.02279) | ✅ |
| **DMD2** | （候选 *Improved Distribution Matching Distillation*，标题待核） | — | 2024 | 候选 2405.14867 | ⏳ 429 |
| **sCM** | （候选，标题待核） | — | 2024 | 候选 2410.11081 | ⏳ 429 |
| **Mean Flows** | （候选，标题待核） | — | 2025 | 候选 2505.13447 | ⏳ 429 |
| **Shortcut** | （候选，标题待核） | — | 2024 | 候选 2410.12557 | ⏳ 429 |
| **Hyper-SD** | （候选，标题待核） | — | 2024 | 候选 2404.13686 | ⏳ 429 |
| **PCM** | （候选 *Phased Consistency Model*，待核） | — | 2024 | 候选 2405.18407 | ⏳ 429 |

> ⏳ = arXiv + Semantic Scholar 双重 429 限流，候选 ID 未核验，引用前需回原文确认。

---

## 方向 A：剪枝压缩 diffusion

覆盖技术路线：token merging（ToMeSD 系）、结构化通道/block 剪枝、timestep-aware 动态剪枝、one-shot 剪枝（OBS-Diff）、命名 pruner（Diff-Prune / DiP-GO）、attention/temporal 剪枝、稀疏/MoE 重构。

> 注：方向A 的多数代表性工作同时归入方向C（BK-SDM / SnapFusion / LAPTOP-Diff / Pluggable Pruning DiT / Amber-Image / PARE 等标题含 pruning+distillation），见方向C表。下表仅列方向A 专属（纯剪枝 / caching 边界）条目。

| 方法 | 说明 | arXiv | 核验 |
|---|---|---|---|
| **ToMeSD** | token merging 用于 Stable Diffusion（纯剪枝/合并 token，非蒸馏） | 候选 2303.17604 | ⏳ 429 |
| **T-GATE** | temporal gating（**caching 类，边界**） | 候选 2404.02747 | ⏳ 429 |
| **TinyFusion** | DiT block-wise 剪枝（ICLR 2025 区间） | 候选 2412.01199 | ⏳ 429 |
| **DeepCache** | 特征缓存复用（**非剪枝、非蒸馏，边界，已剔除方向C**） | [2312.00858](https://arxiv.org/abs/2312.00858) | ✅ |

> **重要**：TinyFusion 候选 ID 2407.00011 经核验实为无关数学论文（*Enhancing Computational Efficiency in Multiscale Systems…*），**并非 TinyFusion**——该候选 ID 错误，真实 ID 待限流解除后重查。

---

## 未找到项（诚实标注）
- **SLaP**：未找到 diffusion 剪枝语境下的该名工作。
- **RAFT（diffusion 剪枝版）**：未找到。
- **ImproveD**：未找到。
- **独立 SD-Turbo 论文**：SD-Turbo 仅有 Stability AI 官方 blog，**无 arXiv 论文**。

---

## 诚实修正记录
- **SnapFusion**：我初稿误写 arXiv 2312.00496，经核验正确为 **2306.00980**（已更正）。
- **BK-SDM**：我初稿误写 arXiv 2305.15730，经核验正确为 **2305.15798**（已更正）。
- 用户原始记忆中的 3 个错误 ID 已更正：Progressive Distillation（→ 2202.00512）、DMD（→ 2311.18828）、CTM（→ 2310.02279）。
- **TinyFusion 候选 2407.00011 证伪**：实为数学论文，需重查真实 ID。
- 明确区分 caching vs pruning vs distillation（DeepCache / T-GATE 非 C）。
- 仅 Dynamic-in-Few-Step / Bridging 真正联合训练，其余为「先剪后蒸」管线。

## 待办（逐篇链接补全，受 API 限流阻塞）
- [ ] 方向B 剩余 6 篇（DMD2 / sCM / Mean Flows / Shortcut / Hyper-SD / PCM）逐条核验
- [ ] 方向A ToMeSD / T-GATE / TinyFusion 逐条核验（TinyFusion 真实 ID 需重查）
- [ ] 限流解除后（通常数小时内）用 arXiv API 完成上表 ⏳ 行，届时本文件为完整版
