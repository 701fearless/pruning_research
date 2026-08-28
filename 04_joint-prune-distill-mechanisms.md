# 04 — 剪枝与蒸馏"结合机制"总览（四机制分类 + 全量核验清单）

> 时间范围 2014–2026。覆盖领域：CNN / Transformer / NLP / CV / LLM / 语音 / 扩散 DiT / 文生图 / 视频 / GAN / SR / TTS / 检测 / 多模态。
> 核验方式：arXiv API（`export.arxiv.org/api/query`）逐条读取 `citation_title`/`authors`/`published` 比对；Semantic Scholar 辅助。WebSearch 生成摘要不采信（会编造作者与 arXiv ID）。
> 本文是 [01](01_kd-taxonomy.md) / [02](02_iterative-prune-distill.md) / [03](03_diffusion-prune-distill.md) 的机制侧重组，并补入"蒸馏 loss 作剪枝判据"线的核验结果。

---

## 0. 为什么是四个机制

剪枝流程有三个环节：**①决定剪谁（判据/敏感度）→ ②决定怎么剪（一次性 / 渐进 / 可微）→ ③决定怎么补回来（恢复）**。知识蒸馏（KD）可以嵌进其中任一环节，于是形成不同"结合位置"。去掉两个不算真联合的边界后，核心剩四个：

| 机制 | KD 嵌入位置 | 一句话 | 是否真联合 |
|---|---|---|---|
| **一·判据型** | ①决定剪谁 | 把 KD loss 当重要性/敏感度，剪"对 mimicking teacher 影响最小"的部分 | 是（判据级） |
| **二·恢复型** | ③决定怎么补回来 | 剪完用 KD loss（而非普通 CE/SGD）做 fine-tune/retrain | 否（顺序） |
| **三·交替/渐进型** | ②+③交织 | 剪一小步→蒸一下→再剪，沿途多教师 | 是（时序级） |
| **四·联合训练型** | ②同一优化循环 | 剪枝目标与 KD 在同一 loss、同一 step 同时优化 | 是（最强） |

**被剔除的两个边界**（不是真联合，单列于第 5 节）：
- 教师初始化 + 持续预训练（如 Sheared LLaMA）：**无显式 KD loss**，广义算"用教师恢复"但非经典 KD。
- 搜索式剪枝里的蒸馏：搜索只是壳，蒸馏本身仍归入上面四机制之一；多数搜索式剪枝（MetaPruning/AutoSlim/Once-for-All/DHP/GDP/Pruner-Zero/BESA）**根本不用蒸馏**。

> 注：一篇论文可同时落入多个机制（如 Knapsack+Inner KD 既把 KD 写进判据目标，又用 KD 做 fine-tune）。下文按"主导机制"归位，跨机制的在备注里标出。

---

## 机制一：判据型 —— KD loss 作为剪枝敏感度/重要性判据

**判定标准**：决定剪谁时，不用权值幅度、不用 Taylor/Hessian（基于 one-hot CE），而用 teacher-student 一致性损失或其梯度打分；或在可微剪枝里把 KD loss 直接写进 mask 优化目标。

### 1a. KD 梯度/损失增量直接当重要性分数

| 方法 | 领域 | Venue/年 | arXiv | 机制要点（摘要原文佐证） |
|---|---|---|---|---|
| **Teacher-Guided One-Shot Pruning (Context-Aware KD)** | CV | IEEE BigData 2025 | [2511.16653](https://arxiv.org/abs/2511.16653) | "leverages gradient signals informed by the teacher during importance score calculation… Unlike prior approaches that apply KD as a post-pruning recovery step"——明说自己是判据、对比剪后恢复 |
| **Distillation-Based Channel Pruning (Alpha Matting)** | CV | ACCV 2022 | [2210.07760](https://arxiv.org/abs/2210.07760) | "remove channels having fewer impacts on mimicking the knowledge of a teacher network"——按剪掉后蒸馏损失增量选通道 |
| **SDMPrune** | LLM | arXiv 2025-06 | [2506.11120](https://arxiv.org/abs/2506.11120) | "introduce a self-distillation loss **during the pruning phase** (rather than post-training)… obtaining more accurate gradient information for pruning"——把 Taylor 里的 one-hot CE 换成 self-distill loss |
| **Concurrent Pruning and Self-Distillation (O'Neill)** | NLP | ECML-PKDD 2022 | [2109.15014](https://arxiv.org/abs/2109.15014) | "use distillation to **inform the pruning criteria**… cross-correlation objective for self-distilled pruning implicitly encourages sparse solutions" |
| **Knapsack Pruning with Inner Distillation** | CNN | NeurIPS 2020 | [2002.08258](https://arxiv.org/abs/2002.08258) | 剪枝建模为背包问题，目标函数含 inner distillation loss，KD loss 直接定义 neuron 重要性（兼跨机制二） |

### 1b. 可微/学习式剪枝里把 KD loss 塞进 mask 优化目标

| 方法 | 领域 | Venue/年 | arXiv | 机制要点 |
|---|---|---|---|---|
| **KDFS (Knowledge-driven Differential Filter Sampler)** | CNN | arXiv 2023-07 | [2307.00198](https://arxiv.org/abs/2307.00198) | 可微 sampler 学 binary mask，端到端目标含 dark knowledge distillation，蒸馏直接决定每个 filter 去留 |
| **MiR (Mimicking then Replacing)** | CV/通用 | arXiv 2022-01（CVPR 2022 相关） | [2201.02620](https://arxiv.org/abs/2201.02620) | 先让被剪模型模仿 teacher 的 penultimate 特征，模仿质量决定哪些层被替换/剪除，"mimic-then-prune"思路 |

### 1-边界（KD 参与流程但不是纯增量打分）

| 方法 | 领域 | arXiv | 为什么算边界 |
|---|---|---|---|
| **HFPrune** | LLM | [2603.08083](https://arxiv.org/abs/2603.08083) | 讨论并比较了 self-distillation criterion 作 Taylor 打分，但最终方法改用信息熵（不需 teacher），self-distill 只是被分析的 baseline |
| **EPSD** | CNN/通用 | [2402.00084](https://arxiv.org/abs/2402.00084) | self-distillation 用来筛选"可蒸馏权重"影响剪枝策略，但实际判据仍是 magnitude（兼跨机制二） |
| **LAPTOP-Diff** | Diffusion | [2404.11098](https://arxiv.org/abs/2404.11098) | 层判据是"可加性"准则；蒸馏用于 normalized feature distillation retraining，不是层判据本身（兼跨机制二） |
| **Bridging** | Diffusion | [2607.06335](https://arxiv.org/abs/2607.06335) | 蒸馏用于替代剪后恢复（teacher-aligned repair 桥接步蒸馏），剪枝本身用标准方法（边界偏否，兼跨机制二/四） |
| **Compressed Diffusion (Pruning with KD for T2I)** | Diffusion/T2I | **无 arXiv**（ICCV 2025 Workshop） | 标题含"Pruning with KD"，但无 arXiv 可核验，细节待原文 |
| **CLAD (Criterion Learner + Attention Distillation)** | CNN | **无 arXiv** | attention distillation 训练判据学习器，疑似 KD-as-criterion，无 arXiv 可核验 |

### 1-证伪（常被误归此，实为"仅剪后 KD 恢复"或"无 KD"）

| 方法 | arXiv | 真相 |
|---|---|---|
| Minitron v1 / v2 | [2407.14679](https://arxiv.org/abs/2407.14679) / [2408.11796](https://arxiv.org/abs/2408.11796) | 启发式深度/宽度剪枝，KD 仅剪后 retraining |
| Prune Your Model Before Distill It | [2109.14960](https://arxiv.org/abs/2109.14960) | 先剪后蒸馏，证的是"剪过的 teacher 有正则化作用"，非判据 |
| Compressing Multi-Task AD Model | [2511.05557](https://arxiv.org/abs/2511.05557) | Taylor 判据 + 剪后 feature KD，非判据 |
| PLATON / NISP / ResRep / Gate Decorator / DDP | [2206.12562](https://arxiv.org/abs/2206.12562) 等 | 判据与 KD 无关 |

---

## 机制二：恢复型 —— KD loss 作为剪后恢复/重训监督

**判定标准**：剪枝（一次性或迭代）完成后，恢复阶段用 KD loss（teacher 软标签 / 中间特征 / 关系）替代普通任务损失做 fine-tune/retrain。剪枝与蒸馏**顺序执行，不交替**。

### 2a. 先剪后蒸（两阶段管线，teacher = 外部/原 dense 模型）

| 方法 | 领域 | Venue/年 | arXiv | 机制要点 |
|---|---|---|---|---|
| **SnapFusion** | 文生图 U-Net | arXiv 2023-06 | [2306.00980](https://arxiv.org/abs/2306.00980) | 剪枝压缩 SD U-Net + 蒸馏恢复，两阶段管线 |
| **BK-SDM** | 文生图(SD) | arXiv 2023-05 | [2305.15798](https://arxiv.org/abs/2305.15798) | Stable Diffusion 轻量化，先剪后蒸 |
| **LAPTOP-Diff** | diffusion | arXiv 2024-04 | [2404.11098](https://arxiv.org/abs/2404.11098) | 层剪枝 + normalized feature distillation retraining（兼跨机制一边界） |
| **Minitron** | LLM | NeurIPS 2024 | [2407.14679](https://arxiv.org/abs/2407.14679) | depth/width/attention/MLP 剪枝 + KD retraining（teacher 为原 dense），最规范 recipe |
| **Knapsack + Inner KD** | CNN | NeurIPS 2020 | [2002.08258](https://arxiv.org/abs/2002.08258) | 剪后用 parent 网络 inner 特征蒸馏 fine-tune（兼跨机制一） |
| **Comb-Prune-Distill** | 视觉(CNN+ViT) | ITSC 2024 | [2408.03046](https://arxiv.org/abs/2408.03046) | 统一剪枝框架，"comb"解决层间依赖→剪枝→剪枝步内引入 KD 保留信息 |
| **Prune Your Model Before Distill It** | CNN | ECCV 2022 | [2109.14960](https://arxiv.org/abs/2109.14960) | 先剪后蒸，证剪过的 teacher 作正则化 |
| **Compressing Multi-Task AD Model** | 自动驾驶 | arXiv 2025 | [2511.05557](https://arxiv.org/abs/2511.05557) | Taylor 剪枝 + 剪后 feature-level KD（兼跨机制四：task-aware safe pruning + KD 同循环） |
| **Minitron (full/v2)** | LLM | NeurIPS 2024 | [2408.11796](https://arxiv.org/abs/2408.11796) | 2407.14679 的 extended 版，depth+width pruning → KD recovery 完整 recipe |
| **ShortOPD** | LLM | arXiv 2026 | [2607.13124](https://arxiv.org/abs/2607.13124) | short-to-long on-policy distillation 恢复被剪 LLM 的长程生成能力 |
| **From Pruning to Grafting** | LLM | arXiv 2024 | [2411.14507](https://arxiv.org/abs/2411.14507) | 剪后用 learnable layer fusion 将知识从被剪层动态重分配到保留层（蒸馏式恢复） |
| **Prompt-based Pruning** | 文生图 diffusion | ICLR 2025 | [2406.12042](https://arxiv.org/abs/2406.12042) | prompt-specific 剪枝 → fine-tune 恢复，每个 prompt 训一个专化子模型 |
| **TinySR** | 图像超分 diffusion | arXiv 2025(v3 2026-06) | [2508.17434](https://arxiv.org/abs/2508.17434) | 剪枝扩散 SR 模型 + adversarial compression 恢复，3.7× 加速、74% 参数削减 |
| **REAP** | LLM MoE | ICLR 2026 | [2510.13999](https://arxiv.org/abs/2510.13999) | router-weighted expert activation pruning → expert-wise KD 后蒸馏 |
| **Donut-MINT** | VLM(文档VQA) | ICDAR 2025 W | [2509.26235](https://arxiv.org/abs/2509.26235) | mechanistic interpretability 指导 attention head 剪枝 → KD 恢复 |
| **YOLOv8-SCD** | 检测(航拍) | arXiv 2025 | [2509.12918](https://arxiv.org/abs/2509.12918) | structured channel pruning → CWD channel-wise distillation |
| **Prune-Quantize-Distill** | 通用管线 | arXiv 2026 | [2604.04988](https://arxiv.org/abs/2604.04988) | 有序管线：unstructured pruning → INT8 量化 → KD 收尾 |

### 2b. 自蒸馏恢复（teacher = 剪枝前的自己，无需外部教师）

| 方法 | 领域 | Venue/年 | arXiv | 机制要点 |
|---|---|---|---|---|
| **Self-Data Distillation for Recovering Quality in Pruned LLMs** | LLM | MLSys 2025 | [2410.09982](https://arxiv.org/abs/2410.09982) | 剪枝后用自身未剪版本的生成做 self-data distillation 恢复 |
| **Self-Distillation as a Performance Recovery Mechanism for LLMs** | LLM | arXiv 2026 | [2604.15794](https://arxiv.org/abs/2604.15794) | 直接以"自蒸馏作为 LLM 压缩后性能恢复"为题，对抗灾难性遗忘 |
| **EPSD** | CNN/通用 | AAAI 2024 | [2402.00084](https://arxiv.org/abs/2402.00084) | 早期剪枝 + 自蒸馏恢复（兼跨机制一边界：策略层筛选可蒸馏权重） |
| **Revisiting Self-Distillation** | 通用 | arXiv 2022 | [2206.08491](https://arxiv.org/abs/2206.08491) | 自蒸馏理论参考 |
| **ZEDA** | LLM MoE | arXiv 2026 | [2605.18643](https://arxiv.org/abs/2605.18643) | zero-expert self-distillation adaptation，跳掉半数 active experts 后自蒸馏恢复 |

> 注：**SDMPrune / Concurrent+Self-Distill** 摘要明确 KD 进的是"判据/剪枝阶段"而非"剪后恢复"，故归机制一，不在此。

---

## 机制三：交替/渐进型 —— prune↔distill 时序交替

**判定标准**：不是一次性剪完再蒸，而是剪一小步→蒸一下→再剪，多轮交替；常伴有沿途 checkpoint 作"numerous teachers"或 ensemble。剪枝与蒸馏**时序交织但不同时**（与机制四的区别）。

| 方法 | 领域 | Venue/年 | arXiv | 机制要点 |
|---|---|---|---|---|
| **NutePrune** | LLM | arXiv 2024 | [2402.09773](https://arxiv.org/abs/2402.09773) | progressive 结构化剪枝 LLM，每步用原始完整模型作 KD teacher + 中间 checkpoint 作"numerous teachers"沿途监督 |
| **PPCD-GAN** | 条件 GAN | WACV 2022 | [2203.08456](https://arxiv.org/abs/2203.08456) | 大规模 cGAN 的 progressive pruning + class-aware KD 每步交错 |
| **DIPNet** | 图像超分 | NTIRE 2023 | [2304.07018](https://arxiv.org/abs/2304.07018) | 高分辨率增强输出作 teacher + 迭代网络剪枝 + multi-anchor 蒸馏 + progressive learning |
| **Paying more attention to snapshots** | CNN | BMVC 2020 | [2006.11487](https://arxiv.org/abs/2006.11487) | 迭代剪枝各 snapshot（带大学习率重启）构成 ensemble → ensemble distillation 回 compact |
| **DCASE acoustic scene** | 音频 | DCASE 2024 第一名 | [2410.20775](https://arxiv.org/abs/2410.20775) | Rep-Mobile + teacher KD + progressive pruning（多次小量剪优于单步） |
| **Staged Depth-Pruning Distillation** | TTS | arXiv 2026-07 | [2607.18662](https://arxiv.org/abs/2607.18662) | 标题直写"Staged Depth-Pruning Distillation"，分阶段深度剪枝+蒸馏把 flow-matching TTS teacher 压成紧凑合成器 |
| **SlimMoE** | LLM MoE | arXiv 2025 | [2506.18349](https://arxiv.org/abs/2506.18349) | multi-stage expert slimming + distillation，可压至 18%→9% |
| **AntGMM (LMM Compression)** | 多模态 LMM | ACM MM 2024 | [2312.05795](https://arxiv.org/abs/2312.05795) | multi-stage iterative pruning + distillation loss，专有 LMM 压缩（arXiv 2023-12，正式发表 2024） |

---

## 机制四：联合训练型 —— 剪枝目标 + KD 同 loss 同 step

**判定标准**：剪枝决策（mask/稀疏/结构选择）与 KD 在同一优化循环、同一损失里同时优化，而非交替、亦非先剪后蒸。这是和机制三的根本分界。含"搜索式+蒸馏真联合"子型。

### 4a. 同循环联合优化

| 方法 | 领域 | Venue/年 | arXiv | 机制要点 |
|---|---|---|---|---|
| **DPHuBERT** | 语音 SSL | INTERSPEECH 2023 | [2305.17651](https://arxiv.org/abs/2305.17651) | 标题直写"Joint Distillation and Pruning"，剪枝决策与 teacher SSL 的 KD 联合优化于一个训练循环 |
| **TuneComp** | 基础模型 | arXiv 2025 | [2505.21835](https://arxiv.org/abs/2505.21835) | 下游任务监督下"逐步蒸馏到剪枝后低秩结构"，KD+低秩+剪枝放进同一优化循环 |
| **Joint-DetNAS** | 目标检测 | CVPR 2021 | [2105.12971](https://arxiv.org/abs/2105.12971) | NAS+剪枝+dynamic distillation（弹性 teacher pool，progressive shrinking）三联合 |
| **Compresso** | LLM | ICLR 2024 | [2310.05015](https://arxiv.org/abs/2310.05015) | LoRA 嵌入 L0 正则的 instruction tuning，"collaborative prompt"让 dense LLM 全程作 soft teacher，联合非一次性 |
| **Dynamic-in-Few-Step** | 视频生成 | arXiv 2026-07 | [2607.06631](https://arxiv.org/abs/2607.06631) | 动态计算 + 少步蒸馏一体联合训练 |
| **Synergistic Effects of KD and Structured Pruning** | 语音 SSL | ICASSP 2025 | [2502.05837](https://arxiv.org/abs/2502.05837) | 显式研究 KD 与结构化剪枝"联合"的协同效应 |
| **Pluggable Pruning DiT (PPCL)** | DiT | CVPR 2026 | [2511.16156](https://arxiv.org/abs/2511.16156) | 剪 DiT 同时做 contiguous layer distillation 恢复/迁移中间特征 |
| **Pruning and Distilling MoE into Dense LMs** | LLM MoE | arXiv 2026-05 | [2605.28207](https://arxiv.org/abs/2605.28207) | MoE 剪枝+蒸馏成 dense LM，协同 |
| **TinyFusion** | DiT | **CVPR 2025** | [2412.01199](https://arxiv.org/abs/2412.01199) | end-to-end learnable depth pruning of DiT，训练代价 <7% 预训练成本 → 2× 加速；剪枝可学习、与恢复训练一体（真实 ID 已核验，非早先证伪的 2407.00011） |
| **Learnable Sparsity** | diffusion 生成 | arXiv 2024-12 | [2412.02852](https://arxiv.org/abs/2412.02852) | 可学习稀疏 mask + 生成质量恢复训练一体；低代价 diffusion 剪枝（旧名 "Effortless Efficiency" 已被作者改为现标题） |
| **SlimQwen** | LLM MoE | arXiv 2026 | [2605.08738](https://arxiv.org/abs/2605.08738) | MoE 预训练阶段渐进 prune + KD loss 作 backbone 约束联合进行 |
| **SPADE** | LLM-TTS | arXiv 2025 | [2509.20802](https://arxiv.org/abs/2509.20802) | structured pruning + adaptive distillation 联合，专为 LLM-TTS 系统 |
| **Multi-Task AD Compress** | 自动驾驶多任务 | arXiv 2025 | [2511.05557](https://arxiv.org/abs/2511.05557) | task-aware safe pruning + feature-level KD 同循环（兼跨机制二，BDD100k panoptic） |
| **Bridging** | diffusion | arXiv 2026-07 | [2607.06335](https://arxiv.org/abs/2607.06335) | 剪枝 + 步蒸馏一体（teacher-aligned repair 作桥），偏向联合但核验摘要把蒸馏定位为"替代剪后恢复"，故兼跨机制二/四（边界偏否） |

### 4b. 搜索式 + 蒸馏真联合（仅三篇）

| 方法 | 领域 | Venue/年 | arXiv | 机制要点 |
|---|---|---|---|---|
| **AMC** | CNN | ECCV 2018 | [1802.03494](https://arxiv.org/abs/1802.03494) | DDPG RL agent 选每层剪枝率，canonical 恢复为普通 SGD fine-tune（论文另报告 KD 增强变体，非 canonical） |
| **GAN Compression** | GAN | NeurIPS 2020 | [2003.08936](https://arxiv.org/abs/2003.08936) | NAS（权重共享）找高效架构 + 原 generator 多中间表征蒸馏给学生，NAS+蒸馏联合 |
| **Joint-DetNAS**（见上） | 检测 | CVPR 2021 | [2105.12971](https://arxiv.org/abs/2105.12971) | 搜索+剪枝+dynamic distillation 三联合代表 |

> 搜索式剪枝**但不使用蒸馏**的（边界，明确区分）：MetaPruning([1903.10258](https://arxiv.org/abs/1903.10258))、AutoSlim([1903.11728](https://arxiv.org/abs/1903.11728))、Once-for-All([1908.09791](https://arxiv.org/abs/1908.09791))、DHP([2003.13683](https://arxiv.org/abs/2003.13683))、GDP([2109.02220](https://arxiv.org/abs/2109.02220))、Pruner-Zero([2406.02924](https://arxiv.org/abs/2406.02924))、BESA([2402.16880](https://arxiv.org/abs/2402.16880))。

---

## 5. 边界 / 反例（不是真"剪枝+蒸馏联合"）

### 5a. 教师初始化 + 持续预训练（无显式 KD loss）

| 方法 | 领域 | Venue/年 | arXiv | 说明 |
|---|---|---|---|---|
| **Sheared LLaMA** | LLM | ICLR 2024 | [2310.06694](https://arxiv.org/abs/2310.06694) | targeted structured pruning + Dynamic Batch Loading + 持续预训练，从教师权重初始化、在教师派生数据配比上预训练。**原论文无显式 logit/feature KD loss**。常被误归入"剪枝+蒸馏"，实为剪枝+继续预训练。**重要反例**。 |

同类渐进但无蒸馏的还有：UPop([2301.13741](https://arxiv.org/abs/2301.13741))、GISP([2510.18030](https://arxiv.org/abs/2510.18030))。

### 5b. 一次性 / 免重训 / 无蒸馏的 LLM 剪枝（划清边界）

| 方法 | arXiv | 性质 |
|---|---|---|
| SparseGPT | [2301.00774](https://arxiv.org/abs/2301.00774) | 一次性，近似逆 Hessian 权重补偿，无重训，无蒸馏 |
| Wanda | [2306.11695](https://arxiv.org/abs/2306.11695) | 一次性，`|W|·‖X‖` 重要性，无任何更新 |
| SliceGPT | [2401.15024](https://arxiv.org/abs/2401.15024) | 结构化切片，靠正交不变性+PCA 保留性能，无蒸馏 |
| LLM-Pruner | [2305.11627](https://arxiv.org/abs/2305.11627) | 一次性结构化 + LoRA 恢复微调，非 KD |
| FLAP | [2312.11983](https://arxiv.org/abs/2312.11983) | 免重训，无蒸馏 |
| DuoGPT | [2506.20194](https://arxiv.org/abs/2506.20194) | activation-aware pruning + 训练免微调双稀疏，非 KD |
| OWL | [2310.05175](https://arxiv.org/abs/2310.05175) | 层级稀疏分配，恢复仍靠 SparseGPT 式补偿，非 KD |
| ShortGPT | [2403.03853](https://arxiv.org/abs/2403.03853) | 层剪枝，恢复靠少量微调，非 KD |
| LoRAPrune | [2305.18403](https://arxiv.org/abs/2305.18403) | LoRA 微调中剪枝，恢复靠 LoRA，非 KD |
| Pruner-Zero | [2406.02924](https://arxiv.org/abs/2406.02924) | 符号回归进化度量，靠更好度量减恢复负担，非 KD |

### 5c. diffusion 里 caching（非剪枝非蒸馏，常被混入）

- DeepCache([2312.00858](https://arxiv.org/abs/2312.00858))、T-GATE、Learning-to-Cache：缓存中间特征复用，**既非剪枝也非蒸馏**，已从机制四剔除。

### 5d. diffusion 里"蒸馏"指少步推理加速（语义不同，非剪枝恢复）

- 方向 B 全链路（Progressive Distillation → CM/LCM/Guided Distillation → ADD/DMD → DMD2/CTM/sCM/Mean Flows/Shortcut/Hyper-SD/PCM）：这是 diffusion 主流"蒸馏"含义，目的是把多步采样蒸成 1~4 步，与"剪枝后蒸馏恢复"不是一回事。仅 Dynamic-in-Few-Step / Bridging 把它和剪枝真正联合。

---

## 6. 按领域索引（便于按课题取用）

### 6a. Diffusion / DiT / 文生图 / 视频（你最关心的）

| 机制 | 方法 | arXiv | venue |
|---|---|---|---|
| 一·判据 | LAPTOP-Diff(边界)、Compressed Diffusion(无arxiv) | 2404.11098 / — | arXiv / ICCV 2025 W |
| 二·恢复 | SnapFusion、BK-SDM、LAPTOP-Diff、Prompt-based Pruning、TinySR | 2306.00980 / 2305.15798 / 2404.11098 / 2406.12042 / 2508.17434 | arXiv / arXiv / arXiv / ICLR 2025 / arXiv |
| 三·交替 | （专门工作稀少，TTS 侧有 Staged Depth-Pruning Distillation 2607.18662） | — | — |
| 四·联合 | Pluggable Pruning DiT (PPCL)、TinyFusion、Learnable Sparsity、Dynamic-in-Few-Step、Bridging | 2511.16156 / 2412.01199 / 2412.02852 / 2607.06631 / 2607.06335 | CVPR 2026 / CVPR 2025 / arXiv / arXiv / arXiv |
| A·纯剪枝 | ToMeSD、TinyFusion(亦可纯剪视角)、TAPE(仅剪枝无KD) | 候选 2303.17604 / 2412.01199 / 2605.17837 | arXiv / CVPR 2025 / arXiv |

> **关键空缺**：diffusion/文生图领域**没有"蒸馏 loss 当层/filter 判据"的专门工作**——LAPTOP-Diff 用可加性判据、Bridging 用步蒸馏替代恢复、TinyFusion 用可学习深度剪枝，都不是判据型。这是最值得填的空白。
> **TinyFusion 真实 ID 已确认 = 2412.01199（CVPR 2025）**，早先怀疑的 2407.00011 经核验证伪（实为数学论文）。

### 6b. LLM

| 机制 | 方法 | arXiv |
|---|---|---|
| 一·判据 | SDMPrune、Concurrent+Self-Distill、HFPrune(边界) | 2506.11120 / 2109.15014 / 2603.08083 |
| 二·恢复 | Minitron(2407.14679/2408.11796)、Self-Data Distillation、Self-Distill-as-Recovery、ShortOPD、From-Pruning-to-Grafting | 2407.14679 / 2410.09982 / 2604.15794 / 2607.13124 / 2411.14507 |
| 二·自蒸馏恢复 | ZEDA(MoE) | 2605.18643 |
| 三·交替 | NutePrune、SlimMoE | 2402.09773 / 2506.18349 |
| 四·联合 | Compresso、TuneComp、Pruning+Distilling MoE、SlimQwen | 2310.05015 / 2505.21835 / 2605.28207 / 2605.08738 |
| 二·恢复(MoE) | REAP | 2510.13999 |
| 边界反例 | Sheared LLaMA、SparseGPT、Wanda、SliceGPT、LLM-Pruner、FLAP、OWL、ShortGPT、LoRAPrune、DuoGPT、Pruner-Zero | 见 5a/5b |

### 6c. CNN / CV / 检测 / GAN / SR / 语音 / 音频 / TTS

| 领域 | 机制 | 方法 | arXiv |
|---|---|---|---|
| CNN | 一·判据 | Knapsack+Inner KD、KDFS、MiR | 2002.08258 / 2307.00198 / 2201.02620 |
| CNN | 一·判据 | Teacher-Guided One-Shot Pruning、Distillation-Based Channel Pruning | 2511.16653 / 2210.07760 |
| CNN | 三·交替 | Paying more attention to snapshots | 2006.11487 |
| CV/通用 | 一·判据(边界) | EPSD | 2402.00084 |
| 检测 | 四·联合 | Joint-DetNAS | 2105.12971 |
| GAN | 三·交替 | PPCD-GAN | 2203.08456 |
| GAN | 四·联合(搜索) | GAN Compression、AMC | 2003.08936 / 1802.03494 |
| SR | 三·交替 | DIPNet | 2304.07018 |
| 视觉(CNN+ViT) | 二·恢复 | Comb-Prune-Distill | 2408.03046 |
| 语音 SSL | 四·联合 | DPHuBERT、Synergistic Effects | 2305.17651 / 2502.05837 |
| 音频 | 三·交替 | DCASE acoustic scene | 2410.20775 |
| TTS | 三·交替 | Staged Depth-Pruning Distillation | 2607.18662 |

---

## 7. 关键观察

1. **四机制是"结合位置"的正交切分**：判据(①) / 恢复(③) / 交替(②③) / 联合(②同循环)。一篇论文可跨多个机制（Knapsuck、LAPTOP-Diff、EPSD、Bridging 都是跨机制案例）。
2. **"KD loss 作判据"线确证存在且横跨 CNN/LLM/NLP/CV**（7 篇确认），最干净的实现是 **KDFS**（可微 mask + dark knowledge）和 **Distillation-Based Channel Pruning**（按 mimicking impact 选通道）。
3. **LLM 的趋势是 self-distillation 替代 Taylor 里的 one-hot label**（SDMPrune、Concurrent-Self-Distill、HFPrune），本质是用完整输出分布算更准的梯度重要性。
4. **真正"同循环联合"最像教科书**：DPHuBERT（标题直写 Joint）、TuneComp（gradual distill-to-pruned）、NutePrune（progressive+numerous teachers）、Dynamic-in-Few-Step / Bridging（diffusion 侧 2026 才涌现）。
5. **diffusion/文生图在"判据型"上是空白**——可填的位置：把 KD 一致性损失增量当 DiT block / attention head / timestep 的剪枝判据。
6. **重要反例要牢记**：Sheared LLaMA（继续预训练非 KD）、搜索式剪枝主流（MetaPruning 等无蒸馏）、diffusion 的 caching（非剪枝非蒸馏）、一次性 LLM 剪枝（SparseGPT/Wanda 无恢复无蒸馏）。

---

## 8. 待补 / 限流阻塞

- [x] **TinyFusion 真实 arXiv ID**：已确认 = **2412.01199（CVPR 2025）**，候选 2407.00011 经核验证伪（实为数学论文 *Enhancing Computational Efficiency in Multiscale Systems…*）。
- [x] **2024–2026 全领域联合方法补全**：第二个 arXiv 核验 agent 已返回，16 篇新工作已并入上文相应机制（见第 9 节汇总）。
- [ ] **机制一**：Compressed Diffusion (ICCV 2025 Workshop) 与 CLAD 仍无 arXiv 可核验，需回原文/联系作者确认是否真属判据型。
- [ ] **机制四 diffusion 侧**：方向 B 少步蒸馏链 DMD2 / sCM / Mean Flows / Shortcut / Hyper-SD / PCM 部分 arXiv ID 因双重 429 限流待核（详见 [03](03_diffusion-prune-distill.md)）。
- [ ] **ToMeSD 真实 ID**（候选 2303.17604）待限流解除后核验。

## 9. 2024–2026 全领域补充（第二个核验 agent 返回，已并入上文）

下表为相对原 [01]/[02]/[03] 三篇**新核验**的工作（arXiv API 实测标题/作者/日期匹配）。已在机制一~四对应位置展开，此处按领域汇总以便检索。

### 9a. LLM / MoE 新增
| 方法 | arXiv | 机制 | 要点 |
|---|---|---|---|
| Minitron full/v2 | 2408.11796 | 二·恢复 | 2407.14679 的 extended 版，完整 prune+KD recipe |
| SlimQwen | 2605.08738 | 四·联合 | MoE 预训练阶段渐进 prune + KD 作 backbone 约束 |
| SlimMoE | 2506.18349 | 三·交替 | multi-stage expert slimming + distillation |
| ZEDA | 2605.18643 | 二·自蒸馏恢复 | 跳半数 experts 后 self-distillation 恢复 |
| REAP | 2510.13999 | 二·恢复 | router-weighted expert pruning → expert-wise KD，ICLR 2026 |
| ShortOPD | 2607.13124 | 二·恢复 | short-to-long on-policy distillation 恢复长程生成 |
| From Pruning to Grafting | 2411.14507 | 二·恢复 | learnable layer fusion 重分配被剪层知识 |

### 9b. Diffusion / DiT / 文生图 / SR 新增
| 方法 | arXiv | 机制 | 要点 |
|---|---|---|---|
| TinyFusion | 2412.01199 | 四·联合 | 可学习深度剪 DiT，CVPR 2025，<7% 预训练成本 |
| Learnable Sparsity | 2412.02852 | 四·联合 | 可学习稀疏 mask + 生成质量恢复一体（旧名 Effortless Efficiency） |
| Prompt-based Pruning | 2406.12042 | 二·恢复 | prompt-specific 剪枝+恢复，ICLR 2025 |
| TinySR | 2508.17434 | 二·恢复 | 扩散 SR 剪枝+对抗恢复，3.7× 加速 |

### 9c. 视频生成（补充）
| 方法 | arXiv | 机制 | 要点 |
|---|---|---|---|
| TAPE | 2605.17837 | A·纯剪枝(无KD) | training-free temporal-aware pruning，仅供对照 |

### 9d. 多模态 / VLM / 检测 / TTS / 通用管线 新增
| 方法 | arXiv | 机制 | 要点 |
|---|---|---|---|
| Donut-MINT | 2509.26235 | 二·恢复 | 可解释性指导 head 剪枝+KD，文档 VQA |
| AntGMM | 2312.05795 | 三·交替 | multi-stage iterative prune+distill，LMM，ACM MM 2024 |
| YOLOv8-SCD | 2509.12918 | 二·恢复 | structured pruning → CWD channel-wise distillation |
| SPADE | 2509.20802 | 四·联合 | structured pruning + adaptive distillation，LLM-TTS |
| Prune-Quantize-Distill | 2604.04988 | 二·恢复 | prune→quantize→KD 有序管线 |

### 9e. 证伪剔除（候选被 arXiv API 实测证伪，不属 prune+KD 联合）
| 候选 | arXiv | 真相 |
|---|---|---|
| "CompSRT" | 2601.21069 | 实测标题 *Any Orthogonal Transform Will Do*，旋转量化，无关 |
| HKD4VLM | 2506.13038 | 仅 KD 无结构剪枝 |
| Switch-KD | 2604.14629 | 仅 KD for VLM |
| MoEKD | 2603.13213 | 仅 KD for MoE 代码模型 |
| Distilling Knowledge in Data Pruning | 2403.07854 | 数据集剪枝，非模型压缩 |
| SlimLLM | 2505.22689 | 仅结构化剪枝无 KD |
| Prune&Comp | 2507.18212 | 剪枝+幅度补偿恢复无 KD |
| Designing Efficient DiT | 2502.14226 | 仅 distillation 设计原则 |
