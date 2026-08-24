# 02 — 迭代 / 渐进式剪枝 + 蒸馏恢复（prune↔distill 交替）

> 不限制模型类型（CNN / Transformer / NLP / CV / LLM / 语音 / SR / GAN / TTS / 扩散均覆盖）。
> 核验方式：直接 `curl arxiv.org/abs/<id>` 读取 `citation_*` 元数据；venue 优先取 arXiv comments 字段。

## 核心判断（先说结论）

经典剪枝-重训谱系（Han / NetAdapt / AMC / ThiNet / Network Slimming / Lottery Ticket 等）的恢复步骤**几乎全部是「原始任务上的普通 SGD 微调」，并不使用知识蒸馏**。蒸馏是另一条独立谱系（Hinton 2015）。真正「把 KD 显式嵌入迭代剪枝循环」的工作主要集中在 2020 年之后，尤其是 2023–2026 的语音 / 检测 / SR / LLM 领域。

---

## 子方向 1：经典迭代剪枝-重训（恢复 = 普通 SGD fine-tune，**无 KD**）

| # | 论文 | Venue/年 | arXiv | 核心思路 |
|---|---|---|---|---|
| 1 | Optimal Brain Damage (LeCun et al.) | NeurIPS 1989 | 无（pre-arXiv） | 二阶（Hessian 对角）显著性剪权重后继续训练，恢复=普通 backprop 重训 |
| 2 | Learning both Weights and Connections for Efficient NNs (Han et al.) | NeurIPS 2015 | [1506.02626](https://arxiv.org/abs/1506.02626) | 剪枝-重训教科书流程：训练→按权值幅度剪→普通 SGD 微调恢复，可多轮 prune–retrain，**无蒸馏** |
| 3 | Deep Compression (Han et al.) | ICLR 2016 oral | [1510.00149](https://arxiv.org/abs/1510.00149) | 幅度剪枝+训练量化+Huffman 三阶段，恢复均为普通 SGD 微调 |
| 4 | Pruning Filters for Efficient ConvNets (Li et al.) | ICLR 2017 | [1608.08710](https://arxiv.org/abs/1608.08710) | 按 filter L1-norm 剪通道，逐层 fine-tune |
| 5 | Pruning CNNs for Resource Efficient Inference (Molchanov et al.) | ICLR 2017 | [1611.06440](https://arxiv.org/abs/1611.06440) | 一阶 Taylor 近似估计剪 filter 的损失增量，贪心逐层 prune–fine-tune |
| 6 | ThiNet (Luo et al.) | **ICCV 2017**（非 ICLR） | [1707.06342](https://arxiv.org/abs/1707.06342) | 基于「下一层激活统计」的 filter 剪枝，贪心 prune+fine-tune |
| 7 | Network Slimming (Liu et al.) | ICCV 2017 spotlight | [1708.06519](https://arxiv.org/abs/1708.06519) | BN 上加 L1 稀疏的 channel scaling factor，剪 scale→0 通道后微调，天然迭代 |
| 8 | Dynamic Network Surgery (Guo et al.) | **NIPS 2016**（非 ECCV） | [1608.04493](https://arxiv.org/abs/1608.04493) | 剪枝 + splicing（重新加回有利连接）集成进 SGD 训练循环，拓扑动态演化 |
| 9 | NetAdapt (Yang et al.) | ECCV 2018 | [1804.03230](https://arxiv.org/abs/1804.03230) | 逐层自适应选剪枝率满足延迟预算，每层剪后短 fine-tune 再测精度反馈下一步，普通 fine-tune |
| 10 | FPGM (He et al.) | CVPR 2019 | [1811.00250](https://arxiv.org/abs/1811.00250) | 剪「最接近几何中位数」的冗余 filter，迭代剪+fine-tune |
| 11 | Lottery Ticket Hypothesis (Frankle & Carbin) | ICLR 2019 Best Paper | [1803.03635](https://arxiv.org/abs/1803.03635) | 「迭代幅度剪枝 (IMP)」命名性工作：反复剪最小幅度权重并从原始初始化重训到收敛（weight rewinding），**无蒸馏** |

> **小结**：子方向 1 的恢复机制**同质为普通 SGD fine-tune / retrain**。这是与子方向 2 的根本分界。

---

## 子方向 2：把知识蒸馏显式嵌入迭代/渐进剪枝循环（本调研的核心）

| # | 论文 | 领域 | Venue/年 | arXiv | 机制（如何结合迭代+蒸馏） |
|---|---|---|---|---|---|
| 12 | **DPHuBERT: Joint Distillation and Pruning…** (Peng et al.) | 语音 SSL | INTERSPEECH 2023 | [2305.17651](https://arxiv.org/abs/2305.17651) | **标题直写 "Joint Distillation and Pruning"**——剪枝决策与来自 teacher SSL 的 KD 联合优化于一个训练循环，而非 prune-then-finetune |
| 13 | **NutePrune: Progressive Pruning with Numerous Teachers** (Li et al.) | LLM | arXiv 2024 | [2402.09773](https://arxiv.org/abs/2402.09773) | **渐进（迭代）结构化剪枝 LLM**，每一步用原始完整模型作 KD teacher，并以中间剪枝 checkpoint 作「numerous teachers」沿途监督 |
| 14 | **DIPNet: Efficiency Distillation and Iterative Pruning for SR** (Yu et al.) | 图像超分 | NTIRE 2023 | [2304.07018](https://arxiv.org/abs/2304.07018) | 高分辨率增强输出作 teacher + 迭代网络剪枝 + multi-anchor 蒸馏 + progressive learning |
| 15 | **Compact LMs via Pruning+KD (Minitron)** (Muralidharan et al., NVIDIA) | LLM | NeurIPS 2024 | [2407.14679](https://arxiv.org/abs/2407.14679) | depth/width/attention/MLP 剪枝 + 基于 KD 的 retraining（teacher 为原 dense 模型），最规范 recipe |
| 16 | **TuneComp: Joint Fine-tuning and Compression** (Chen et al., MERL) | 基础模型 | arXiv 2025 | [2505.21835](https://arxiv.org/abs/2505.21835) | 在下游任务监督下「逐步蒸馏到剪枝后的低秩结构」，把 KD、低秩、剪枝放进一个优化循环 |
| 17 | **Joint-DetNAS** (Yao et al.) | 目标检测 | **CVPR 2021** | [2105.12971](https://arxiv.org/abs/2105.12971) | 把 NAS、剪枝、**dynamic distillation**（弹性 teacher pool，progressive shrinking 训练）放进同一搜索循环 |
| 18 | **PPCD-GAN: Progressive Pruning + Class-Aware Distillation** (Vo et al.) | 条件 GAN | **WACV 2022**（非 CVPR） | [2203.08456](https://arxiv.org/abs/2203.08456) | 大规模 cGAN 的**渐进剪枝** + **class-aware KD** 在每一步交错 |
| 19 | **Pluggable Pruning with Contiguous Layer Distillation (DiT)** (Ma et al.) | 扩散 DiT | **CVPR 2026** | [2511.16156](https://arxiv.org/abs/2511.16156) | 剪 DiT 同时做 **contiguous layer distillation** 恢复/迁移中间特征（亦属 [03](03_diffusion-prune-distill.md) 方向C） |
| 20 | **Compresso** (Guo et al., MSRA) | LLM | arXiv 2023 / ICLR 2024 | [2310.05015](https://arxiv.org/abs/2310.05015) | 把 LoRA 嵌入 L0 正则的 instruction tuning，「collaborative prompt」让 dense LLM 在剪枝全程作 soft teacher——**联合**而非一次性 |
| 21 | **Self-Data Distillation for Recovering Quality in Pruned LLMs** (Thangarasa et al.) | LLM | **MLSys 2025**（非 EMNLP） | [2410.09982](https://arxiv.org/abs/2410.09982) | 剪枝后用自身未剪版本的生成做 self-data distillation 恢复质量 |
| 22 | **Synergistic Effects of KD and Structured Pruning** (Shiva Kumar et al.) | 语音 SSL | **ICASSP 2025** | [2502.05837](https://arxiv.org/abs/2502.05837) | 显式研究 KD 与结构化剪枝**联合**用于语音 SSL 的协同效应 |
| 23 | **Paying more attention to snapshots of Iterative Pruning** (Le et al.) | CNN | **BMVC 2020** | [2006.11487](https://arxiv.org/abs/2006.11487) | 把迭代剪枝过程中各 snapshot（带大学习率重启）构成强 ensemble，再 **ensemble distillation** 回 compact 模型 |
| 24 | **Data-Efficient Acoustic Scene via Distilling and Progressive Pruning** (Han) | 音频 | DCASE 2024 Challenge 第一名（ICASSP 2025 投稿） | [2410.20775](https://arxiv.org/abs/2410.20775) | Rep-Mobile 架构 + teacher KD + **progressive pruning**（多次小量剪优于单步） |
| 25 | **Staged Depth-Pruning Distillation of Flow-Matching TTS Teacher** | TTS | arXiv 2026/07（极新） | [2607.18662](https://arxiv.org/abs/2607.18662) | 标题直写 **"Staged Depth-Pruning Distillation"**——分阶段深度剪枝+蒸馏把 flow-matching TTS teacher 压成紧凑印地语合成器 |
| 26 | **Pruning and Distilling MoE into Dense LMs** (Kim et al.) | LLM MoE | arXiv 2026/05（极新） | [2605.28207](https://arxiv.org/abs/2605.28207) | 把 MoE 模型**剪枝+蒸馏**成 dense LM，剪枝与蒸馏协同 |
| 27 | **Knapsack Pruning with Inner Distillation** (Aflalo et al.) | CNN | arXiv 2020 | [2002.08258](https://arxiv.org/abs/2002.08258) | 剪枝形式化为 Knapsack，剪后用 parent 网络**内层特征蒸馏 (Inner KD)** fine-tune；剪枝为一次性（非迭代）——边界 |
| 28 | **Comb, Prune, Distill** (Schmitt et al.) | 视觉 | ITSC 2024 | [2408.03046](https://arxiv.org/abs/2408.03046) | 统一剪枝框架（CNN+Transformer），「comb」解决层间依赖→剪枝→剪枝步内引入 KD 保留信息；联合但非显式迭代 |

---

## 子方向 3：自动化/搜索式（NAS/RL/可微）剪枝 + 蒸馏

> 注意：本方向多数「搜索式剪枝」**并不使用蒸馏**。下面明确区分。

**真正结合搜索+蒸馏：**

| # | 论文 | Venue/年 | arXiv | 核心思路 |
|---|---|---|---|---|
| 29 | **AMC: AutoML for Model Compression** (He/Han et al.) | ECCV 2018 | [1802.03494](https://arxiv.org/abs/1802.03494) | DDPG RL agent 选每层剪枝率，reward 为剪后 fine-tune 的精度。canonical 恢复为普通 SGD fine-tune（论文另报告 KD 增强变体，非 canonical）。搜索迭代在 RL 循环内 |
| 30 | **GAN Compression** (Li/Han et al.) | NeurIPS 2020 | [2003.08936](https://arxiv.org/abs/2003.08936) | cGAN 压缩框架，NAS（权重共享）找高效架构 + 把原 generator 多个中间表征蒸馏给学生。**NAS + 蒸馏**联合 |
| 31 | **Joint-DetNAS**（见 #17） | CVPR 2021 | [2105.12971](https://arxiv.org/abs/2105.12971) | 搜索+剪枝+dynamic distillation 三联合的代表 |

**搜索/自动化但明确无蒸馏（边界标注）：**

| # | 论文 | Venue | arXiv | 是否蒸馏 |
|---|---|---|---|---|
| 32 | MetaPruning (Liu et al.) | ICCV 2019 | [1903.10258](https://arxiv.org/abs/1903.10258) | 否（明确「搜索时无需 fine-tune」） |
| 33 | AutoSlim (Yu, Huang) | arXiv 2019 | [1903.11728](https://arxiv.org/abs/1903.11728) | 否 |
| 34 | Once-for-All (Cai, Gan, Han) | ICLR 2020 | [1908.09791](https://arxiv.org/abs/1908.09791) | 否（progressive shrinking 但无 KD） |
| 35 | DHP: Differentiable Meta Pruning via HyperNetworks (Li et al.) | ECCV 2020 | [2003.13683](https://arxiv.org/abs/2003.13683) | 否 |
| 36 | GDP: Gates with Differentiable Polarization (Guo et al.) | ICCV 2021 | [2109.02220](https://arxiv.org/abs/2109.02220) | 否 |
| 37 | Pruner-Zero (Dong et al.) | ICML 2024 | [2406.02924](https://arxiv.org/abs/2406.02924) | 否（进化搜索 symbolic metric） |
| 38 | BESA (Xu et al.) | ICLR 2024 | [2402.16880](https://arxiv.org/abs/2402.16880) | 否（blockwise 可微重建损失） |

> 纠错：MetaPruning 正确 arXiv 是 **1903.10258**（非 1903.06362，后者是天文学论文）；DHP 全称是 "Differentiable **Meta** Pruning"（非 "Hierarchical"）；GDP 全称是 "Gates with Differentiable Polarization"（非 "Global Dimension Pruning"）。

---

## 子方向 4：分阶段/渐进/课程式剪枝 + 蒸馏监督

> 本方向与子方向 2 高度重叠；此处列最清晰的「分阶段+蒸馏」实例，并附「渐进但无蒸馏」的诚实对照。

**带蒸馏监督的：**
- **NutePrune (#13)**：progressive + numerous teachers，LLM 代表。
- **PPCD-GAN (#18)**：progressive pruning + class-aware distillation，GAN 代表。
- **DIPNet (#14)**：iterative pruning + progressive learning + multi-anchor distillation，SR 代表。
- **Paying more attention snapshots (#23)**：iterative pruning snapshots → ensemble distillation，CV 代表。
- **DCASE acoustic scene (#24)**：progressive pruning + KD，音频代表。
- **Staged Depth-Pruning Distillation (#25)**：staged depth pruning + distillation，TTS 代表。

**渐进/迭代但「无蒸馏」的诚实对照（避免误归类）：**

| # | 论文 | Venue | arXiv | 说明 |
|---|---|---|---|---|
| 39 | **Sheared LLaMA** (Xia et al.) | ICLR 2024 | [2310.06694](https://arxiv.org/abs/2310.06694) | LLaMA2-7B 目标结构化剪到 1.3B/2.7B + 动态 batch 加载的 **continued pretraining**。是迭代/渐进剪枝，但**不使用 KD**，而用动态数据组成恢复。**重要反例**：常被误归入「剪枝+蒸馏」，实为剪枝+继续预训练 |
| 40 | **UPop: Unified and Progressive Pruning for VL Transformers** (Shi et al.) | **ICML 2023**（非 CVPR） | [2301.13741](https://arxiv.org/abs/2301.13741) | VL Transformer 统一+渐进搜索 subnet 并 retrain。**渐进但无蒸馏** |
| 41 | **GISP / From Local to Global** | **ACL 2026 Main** | [2510.18030](https://arxiv.org/abs/2510.18030) | Global Iterative Structured Pruning，按一阶 loss 重要性 + block 归一化迭代剪 attention heads 与 MLP channels。**迭代但无蒸馏**，post-training 无需中间 fine-tune |

---

## 对照组：一次性 / 免重训 / 无蒸馏的 LLM 剪枝（用于划清边界）

| # | 论文 | Venue | arXiv | 性质 |
|---|---|---|---|---|
| 42 | SparseGPT (Frantar, Alistarh) | ICML 2023 | [2301.00774](https://arxiv.org/abs/2301.00774) | 一次性，无重训，无蒸馏 |
| 43 | Wanda (Sun, Liu, Bair, Kolter) | ICLR 2024 | [2306.11695](https://arxiv.org/abs/2306.11695) | 一次性，无重训，无蒸馏 |
| 44 | SliceGPT (Ashkboos et al.) | ICLR 2024 | [2401.15024](https://arxiv.org/abs/2401.15024) | 一次性，无蒸馏 |
| 45 | LLM-Pruner (Ma, Fang, Wang) | NeurIPS 2023 | [2305.11627](https://arxiv.org/abs/2305.11627) | 一次性结构化 + LoRA 恢复（非 KD，非迭代） |
| 46 | FLAP (An et al.) | AAAI 2024 | [2312.11983](https://arxiv.org/abs/2312.11983) | 免重训，无蒸馏 |

## 基础 KD 参考（非剪枝工作，但是蒸馏谱系的起点）
- **Distilling the Knowledge in a Neural Network**, Hinton, Vinyals, Dean, NIPS 2014 DL Workshop. [1503.02531](https://arxiv.org/abs/1503.02531)
  - KD 奠基性工作（temperature-scaled soft target + KL 损失）。**不是剪枝论文**，但是子方向 2 所有工作的 teacher 信号来源。明确：Han 的 Deep Compression 与此是两条独立谱系，无 Han 版「Distilling the Knowledge」。

---

## 关键结论

1. **子方向 1（经典迭代剪枝-重训）整体同质为「普通 SGD fine-tune，无蒸馏」**——Han、NetAdapt、AMC(canonical)、ThiNet、Network Slimming、Lottery Ticket 等皆如此。这是与子方向 2 的根本分界。
2. **真正「把 KD 显式嵌入迭代剪枝循环」的工作集中于 2020 年后**，且横跨语音（DPHuBERT、Synergistic-Effects）、检测（Joint-DetNAS）、GAN（PPCD-GAN）、SR（DIPNet）、扩散（Pluggable-Pruning-DiT）、TTS（Staged Depth-Pruning Distillation）、LLM（NutePrune、Minitron、TuneComp、Compresso、Self-Data-Distillation、Pruning+Distilling-MoE）。LLM 那一波是 2024–2026 才涌现的。
3. **子方向 3（搜索式+蒸馏）的「真联合」仅 AMC、GAN Compression、Joint-DetNAS 三篇**；MetaPruning/AutoSlim/Once-for-All/DHP/GDP/Pruner-Zero/BESA 等主流搜索式剪枝**并不使用蒸馏**，需明确区分。
4. **重要反例**：Sheared LLaMA（ICLR 2024）虽是渐进/迭代 LLM 剪枝的旗舰，但用**继续预训练 + 动态数据**恢复，**无 KD**；UPop、GISP 同为渐进但无蒸馏。常被误归入「剪枝+蒸馏」，实非。
5. **最像「教科书式 prune↔distill 交替/联合」的代表作**：DPHuBERT（标题直写 Joint Distillation+Pruning）、TuneComp（joint fine-tune+compress by gradual distill-to-pruned）、NutePrune（progressive+numerous teachers）、PPCD-GAN（progressive+class-aware KD）、Staged Depth-Pruning Distillation（staged+distillation）、Paying-More-Attention（iterative-pruning-snapshots→ensemble-distillation）。

## 纠错记录
ThiNet 是 ICCV 2017 非 ICLR 2017；Dynamic Network Surgery 是 NIPS 2016 非 ECCV 2016；PPCD-GAN 是 WACV 2022 非 CVPR 2022；UPop 是 ICML 2023 非 CVPR 2023；Self-Data Distillation 是 MLSys 2025 非 EMNLP 2024；Joint-DetNAS 是 CVPR 2021（非仅 arXiv）；MetaPruning 正确 arXiv 为 1903.10258；NetAdapt 为 1804.03230；Dynamic Network Surgery 为 1608.04493。BESA/SparseGPT/Sheared-LLaMA/DHP/GAN-Compression 的 venue 因 arXiv comments 为空而依公认记录（已交叉验证）。

## 未找到专门工作的子方向（诚实标注）
- **"Distillation-driven pruning" 作为精确剪枝论文标题**：未找到专门工作。
- **精确标题为 "Joint Pruning and Knowledge Distillation" 的论文**：未找到与该完全同名的专门论文。最接近「joint」的是 DPHuBERT (#12) 与 TuneComp (#16)。
- **"Mimic then Prune" / "Prune and Mimic" 作为论文标题**：未找到 arXiv 匹配。
- **"Annealed pruning + distillation" 作为一个命名方法**：未找到。
- **"Curriculum-aware pruning" 作为专门论文**：未找到。
- **AFP / Padé / Delta-LLM / SLIP / Polyak & Jude 等**：未找到可核实剪枝+蒸馏工作。
