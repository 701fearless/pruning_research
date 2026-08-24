# 01 — 剪枝后蒸馏方式分类与联合训练

> 核验方式：arXiv API（`export.arxiv.org`）+ Semantic Scholar API 逐条核验。WebSearch 对学术查询返回模型生成摘要（含错误作者 / arXiv ID），未采信。

---

## 方向一：剪枝后用于恢复性能的蒸馏方式分类

剪枝后恢复性能的蒸馏可按「迁移什么」分成几类。

### 1.1 Logit 蒸馏（软标签 / KL on logits）
- **Hinton KD（奠基）** — 用带温度的 softmax 软标签做 KL 损失，是所有 logit KD 的源头。
  - *Distilling the Knowledge in a Neural Network*, Hinton, Vinyals, Dean, NIPS 2014 DL Workshop. https://arxiv.org/abs/1503.02531

### 1.2 Feature / Hint 蒸馏（中间层特征对齐）
- **FitNets** — 用中间层「提示」对齐，让学生学老师的隐层表征，使更窄更深的网络可被训练。
  - *FitNets: Hints for Thin Deep Nets*, Romero et al., ICLR 2015. https://arxiv.org/abs/1412.6550

### 1.3 Relation 蒸馏（样本/特征间关系）
- **RKD** — 蒸馏样本对之间的距离关系（distance）和角度关系（angle），保持表征几何。
  - *Relational Knowledge Distillation*, Park, Kim, Lu, Cho, CVPR 2019. https://arxiv.org/abs/1904.05068
- **CRD** — 用对比学习把老师的表征空间结构传给学生，可视为关系蒸馏的对比化形式。
  - *Contrastive Representation Distillation*, Tian, Krishnan, Isola, ICLR 2020. https://arxiv.org/abs/1910.10699
- **MiniLM / MiniLMv2** — 蒸馏自注意力矩阵间的关系（student-teacher head 间 Pearson 相关），是 Transformer 关系蒸馏的代表作。
  - *MiniLM: Deep Self-Attention Distillation…*, Wang et al., 2020. https://arxiv.org/abs/2002.10957
  - *MiniLMv2: Multi-Head Self-Attention Relation Distillation…*, Wang et al., 2021. https://arxiv.org/abs/2012.15828

### 1.4 Self-Distillation（自蒸馏做恢复，无需外部教师）
- **EPSD** — 早期剪枝 + 自蒸馏：用剪枝前的模型自身作为教师引导剪枝后模型恢复，AAAI 2024。
  - *EPSD: Early Pruning with Self-Distillation for Efficient Model Compression*, Chen et al., AAAI 2024. https://arxiv.org/abs/2402.00084
- **Concurrent Pruning and Self-Distillation** — 同时做剪枝与自蒸馏。
  - *Deep Neural Compression Via Concurrent Pruning and Self-Distillation*, O'Neill et al., 2021. https://arxiv.org/abs/2109.15014
- **SDMPrune（面向 LLM）** — 自蒸馏 + MLP 剪枝，用于高效 LLM。
  - *SDMPrune: Self-Distillation MLP Pruning for Efficient Large Language Models*, Zhu & Shen, 2025. https://arxiv.org/abs/2506.11120
- **Self-Distillation as Performance Recovery for LLMs** — 直接以「自蒸馏作为 LLM 压缩后的性能恢复机制」为题，对抗压缩与灾难性遗忘，与本课题最贴合。
  - *Self-Distillation as a Performance Recovery Mechanism for LLMs…*, Liu et al., 2026. https://arxiv.org/abs/2604.15794
- *Revisiting Self-Distillation*, Pham et al., 2022. https://arxiv.org/abs/2206.08491

### 1.5 Combined / 多级 KD
- **TinyBERT** — embedding 层 + 隐藏态 + 注意力矩阵 + 预测层多级蒸馏，是「combined KD」的典型。见方向二 2.1。

---

## 方向二：剪枝 + 蒸馏联合训练相关方法

### 2.1 Transformer / LM 的多级蒸馏（可与剪枝联用）
- **TinyBERT** — 两阶段（通用蒸馏 + 任务特定蒸馏）+ transformer 多层蒸馏（embedding/hidden/attention/prediction）。是 BERT 压缩-蒸馏的代表，常被作为「剪枝后/窄化后蒸馏」的基线。
  - *TinyBERT: Distilling BERT for Natural Language Understanding*, Jiao et al., Findings of EMNLP 2020. https://arxiv.org/abs/1909.10351
  - 注：网上流传的 TinyBERT 作者列表常有误，正确作者为 **Xiaoqi Jiao, Yichun Yin, Lifeng Shang, Xin Jiang, Xiao Chen, Linlin Li, Fang Wang, Qun Liu**（已核实）。
- **AD-KD（很可能是你提到的「ADS」）** — 把「归因（attribution）」作为额外监督做语言模型压缩蒸馏，ACL 2023。
  - *AD-KD: Attribution-Driven Knowledge Distillation for Language Model Compression*, Wu et al., ACL 2023. https://arxiv.org/abs/2305.10010
  - 未在 arXiv 上找到名为「ADS」的剪枝/蒸馏论文，最近匹配即 AD-KD。

### 2.2 LLM 蒸馏（逆向 KL / 在策略）—— 与剪枝结合的常用蒸馏骨干
- **MiniLLM** — 用 reverse KLD（mode-seeking）+ 策略梯度做白盒 LLM 蒸馏，比前向 KL 更适合生成任务，ICLR 2024。
  - *MiniLLM: On-Policy Distillation of Large Language Models*, Gu, Dong, Wei, Huang, ICLR 2024. https://arxiv.org/abs/2306.08543
- **GKD（Agarwal 等）** — 提出广义知识蒸馏 GKD：学生在自己生成的序列上训练、用老师反馈，解决训练/推理分布失配。LLM 在策略蒸馏的奠基工作之一，ICLR 2024。
  - *On-Policy Distillation of Language Models: Learning from Self-Generated Mistakes*, Agarwal, Vieillard, Zhou, Stanczyk, Ramos, ICLR 2024. https://arxiv.org/abs/2306.13649
  - 另有一篇同名缩写 **GKD**（Tan 等，ACL 2023 工业轨，面向大规模预训练 LM 的通用 KD 框架），https://arxiv.org/abs/2306.06629 ，二者不同，引用时注意区分。
- **DSKD** — 双空间（logit + 表示）通用 KD 框架，2025。
  - *A Dual-Space Framework for General Knowledge Distillation of Large Language Models*, Zhang et al., 2025. https://arxiv.org/abs/2504.11426

### 2.3 真正「联合」剪枝 + 蒸馏的工作
- **DPHuBERT** — 名字里就是 Joint Distillation and Pruning，对自监督语音模型同时做蒸馏与剪枝，INTERSPEECH 2023。
  - *DPHuBERT: Joint Distillation and Pruning of Self-Supervised Speech Models*, Peng et al., INTERSPEECH 2023. https://arxiv.org/abs/2305.17651
- **Joint-DetNAS** — NAS + 剪枝 + 动态蒸馏联合用于检测器，CVPR 2021。
  - *Joint-DetNAS: Upgrade Your Detector with NAS, Pruning and Dynamic Distillation*, Yao et al., CVPR 2021. https://arxiv.org/abs/2105.12971
- **SKILL** — 相似度感知蒸馏用于语音自监督，ICASSP 2024 workshop。
  - *SKILL: Similarity-aware Knowledge distILLation for Speech Self-Supervised Learning*, Zampierin et al., ICASSP 2024 SASB Workshop. https://arxiv.org/abs/2402.16830
- **TuneComp** — 大基础模型的联合微调与压缩，2025（preliminary）。
  - *TuneComp: Joint Fine-tuning and Compression for Large Foundation Models*, Chen et al., 2025. https://arxiv.org/abs/2505.21835

### 2.4 关于 AFP、Padé（明确说明）
- **AFP**：在 arXiv 上用 `ti:AFP`、`all:AFP AND all:pruning AND all:distillation`、`all:"activation-aware" AND all:pruning`、`all:adversarial AND all:"knowledge distillation" AND all:pruning` 等多组查询搜索，**未找到**在剪枝+蒸馏语境下名为 "AFP" 的论文。搜到的相近项是 *AWP: Activation-Aware Weight Pruning*（ICML 2025 workshop，https://arxiv.org/abs/2506.10205 ）和 *ADMP: Adversarial Double Masks Based Pruning*（2020，https://arxiv.org/abs/2006.04127 ），但二者都不是「AFP」。**未找到，需提供来源以确认。**
- **Padé**：arXiv 上标题含 "Pade" 的论文全部是数学/物理/天文（宇宙学、大地测量、正交多项式等），**没有任何 Padé 近似用于剪枝/蒸馏恢复的论文**。**未找到，需提供来源。**

---

## 方向三：LLM 时代剪枝后是否做蒸馏恢复

LLM 剪枝一线工作里，「恢复」手段差别很大——有的是权重补偿、有的是结构保留、有的是 LoRA 微调、有的是持续预训练/蒸馏。

| 方法 | 剪枝类型 | 恢复机制（是否用 KD） | 论文 / venue / 链接 |
|---|---|---|---|
| **SparseGPT** | 一次性非结构化 | 用近似逆 Hessian 做**权重补偿**（同行内剩余权重重解），**非 KD** | Frantar & Alistarh, ICML 2023. https://arxiv.org/abs/2301.00774 |
| **Wanda** | 一次性非结构化 | `|W|·‖X‖` 重要性，**不做任何权重更新/恢复** | Sun, Liu, Bair, Kolter, ICLR 2024. https://arxiv.org/abs/2306.11695 |
| **SliceGPT** | 结构化（行列切片） | 用正交计算不变性 + PCA 找低方差维度切片，**靠结构本身保留性能**，后接少量微调；非 KD | Ashkboos et al., ICLR 2024. https://arxiv.org/abs/2401.15024 |
| **LLM-Pruner** | 结构化（依赖感知） | 50k 样本/2–3 天内做 **LoRA 恢复微调**；原论文非显式 KD | Ma, Fang, Wang, NeurIPS 2023. https://arxiv.org/abs/2305.11627 |
| **Sheared LLaMA** | 结构化（到目标形状） | **targeted structured pruning + Dynamic Batch Loading (DBL) + 持续预训练**；从教师权重初始化、在教师派生数据配比上预训练。**原论文未使用显式 logit/feature KD 损失**——所谓「蒸馏策略」更准确的表述是「教师初始化 + 持续预训练恢复」。若宽义地把「用教师数据恢复教师能力」算作蒸馏，它属于这一类，但不是经典 KD loss | Xia, Gao, Zeng, Chen, ICLR 2024. https://arxiv.org/abs/2310.06694 |
| **Compresso** | 结构化 | 「**collaborative prompting**」范式：资源高效剪枝 + 协作式 prompting 做任务无关恢复（prompting 形式的自/教师引导恢复，非经典 logit KD） | Guo, Xu, Zhang, Yang, ICLR 2024. https://arxiv.org/abs/2310.05015 |
| **OWL** | 非结构化（层级稀疏分配） | 「outlier-weighed」层级稀疏分配，让高稀疏下掉更少；恢复仍靠 SparseGPT 式补偿，非 KD | Yin et al., ICML 2024. https://arxiv.org/abs/2310.05175 |
| **FLAP** | 结构化（自适应） | 基于「波动」的重要性做结构化剪枝 + 恢复微调；非显式 KD | An et al., AAAI 2024. https://arxiv.org/abs/2312.11983 |
| **Pruner-Zero** | 结构化（符号度量） | 用符号回归进化剪枝度量，靠更好度量减少恢复负担；非 KD | Dong et al., ICML 2024. https://arxiv.org/abs/2406.02924 |
| **LoRAPrune** | 结构化 | 在 LoRA 微调中剪枝，恢复靠 LoRA 训练本身；非显式 KD | Zhang et al., ACL 2024 Findings. https://arxiv.org/abs/2305.18403 |
| **ShortGPT** | 层剪枝 | 基于 BiGEMM 冗余评分删层，恢复靠少量微调；非 KD | Men et al., 2024 (preprint). https://arxiv.org/abs/2403.03853 |
| **DuoGPT** | 结构化 + 非结构化 | activation-aware pruning + 训练免微调的双稀疏；非 KD | Yin et al., NeurIPS 2025. https://arxiv.org/abs/2506.20194 |

**结论性观察**：在 LLM 剪枝一线（SparseGPT/Wanda/SliceGPT/OWL/FLAP/Pruner-Zero 等）**主流并不显式使用 KD 来恢复**——它们靠权重补偿、结构保留、更好度量或 LoRA 微调。真正把「蒸馏」作为恢复主力的 LLM 工作，集中在**结构化剪枝 + 后训练恢复**这条线：Sheared LLaMA（持续预训练+DBL）、Compresso（collaborative prompting）、以及方向一里的 SDMPrune / Self-Distillation recovery for LLMs。若你想做「剪枝后蒸馏恢复」课题，这正是当前相对空缺、值得填补的位置。

---

## 方向四：2024–2026 NeurIPS/ICML/ICLR/ACL/EMNLP 等的代表作（已核实 venue）

### ICLR 2024（剪枝/蒸馏簇）
- SliceGPT — https://arxiv.org/abs/2401.15024
- Wanda — https://arxiv.org/abs/2306.11695 （Semantic Scholar 核实 venue = ICLR）
- Sheared LLaMA — https://arxiv.org/abs/2310.06694
- Compresso — https://arxiv.org/abs/2310.05015
- MiniLLM — https://arxiv.org/abs/2306.08543
- GKD / On-Policy Distillation (Agarwal 等) — https://arxiv.org/abs/2306.13649 （comment 明确 "Accepted at ICLR 2024"）

### ICML 2024
- OWL — https://arxiv.org/abs/2310.05175 （comment "Published at ICML 2024"）
- Pruner-Zero — https://arxiv.org/abs/2406.02924 （comment "Accepted by ICML2024"）

### AAAI 2024
- FLAP — https://arxiv.org/abs/2312.11983 （comment "Accepted to AAAI 2024"）
- EPSD（自蒸馏剪枝恢复） — https://arxiv.org/abs/2402.00084 （comment "accepted by AAAI 2024"）

### ACL 2023 / 2024
- AD-KD — https://arxiv.org/abs/2305.10010 （comment "Accepted to ACL 2023 Main Conference"）
- GKD（Tan 等，通用 KD 框架） — https://arxiv.org/abs/2306.06629 （comment "accepted for ACL 2023 industry track"）
- LoRAPrune — https://arxiv.org/abs/2305.18403 （comment "accepted by acl 2024 findings"）

### NeurIPS 2023 / 2025
- LLM-Pruner — https://arxiv.org/abs/2305.11627 （comment "Accepted at NeurIPS 2023"）
- DuoGPT — https://arxiv.org/abs/2506.20194 （comment "NeurIPS 2025"）

### ICML 2026 / COLM 2026（最新）
- Entropy-Aware On-Policy Distillation of Language Models — https://arxiv.org/abs/2603.07079 （comment "ICML 2026"）
- Batch-wise Adaptive Pruning: Periodic Neuron Activation-Aware Weight Pruning — https://arxiv.org/abs/2608.14003 （comment "Accepted at COLM 2026"）
- SOD: Step-wise On-policy Distillation for Small LM Agents — https://arxiv.org/abs/2605.07725 （2026）
- EasyOPD: An Easy-to-use On-Policy Distillation Framework for LLMs — https://arxiv.org/abs/2607.11012 （2026）

### ICML 2023
- SparseGPT — https://arxiv.org/abs/2301.00774

### INTERSPEECH 2023
- DPHuBERT（联合剪枝+蒸馏） — https://arxiv.org/abs/2305.17651

### Findings of EMNLP 2020
- TinyBERT — https://arxiv.org/abs/1909.10351

---

## 综述（2023–2024，便于切入文献）
- *A Survey on Model Compression for Large Language Models*, Zhu et al., TACL. https://arxiv.org/abs/2308.07633
- *A Comprehensive Survey of Compression Algorithms for Language Models*, Park et al., 2024. https://arxiv.org/abs/2401.15347
- *Model Compression and Efficient Inference for Large Language Models: A Survey*, Wang et al., 2024. https://arxiv.org/abs/2402.09748
- *A Survey on Transformer Compression*, Tang et al., 2024. https://arxiv.org/abs/2402.05964
- *A Survey on Deep Neural Network Pruning*, Cheng, Zhang, Shi, TPAMI. https://arxiv.org/abs/2308.06767

---

## 简要分类总结表

| 维度 | 代表方法 | 是否显式 KD 恢复 | venue / 年份 |
|---|---|---|---|
| Logit KD | Hinton KD | 是 | NIPS-W 2014 |
| Feature/Hint KD | FitNets | 是 | ICLR 2015 |
| Relation KD | RKD, CRD, MiniLM/v2 | 是 | CVPR19 / ICLR20 / 2020-21 |
| Self-distillation 恢复 | EPSD, Concurrent Prune+Self-Distill, SDMPrune, Self-Distill Recovery for LLMs | 是 | AAAI24 / 2021 / 2025 / 2026 |
| Combined 多级 KD | TinyBERT | 是 | Findings EMNLP 2020 |
| LLM 在策略蒸馏 | MiniLLM, GKD(Agarwal), DSKD | 是 | ICLR24 / ICLR24 / 2025 |
| 联合剪枝+蒸馏 | DPHuBERT, Joint-DetNAS, SKILL, TuneComp | 是 | INTERSPEECH23 / CVPR21 / ICASSP24-W / 2025 |
| LLM 剪枝(权重补偿恢复) | SparseGPT, OWL | 否（补偿） | ICML23 / ICML24 |
| LLM 剪枝(无恢复) | Wanda | 否 | ICLR24 |
| LLM 剪枝(结构保留) | SliceGPT | 否 | ICLR24 |
| LLM 剪枝(LoRA/微调恢复) | LLM-Pruner, LoRAPrune, FLAP, ShortGPT | 否 | NeurIPS23 / ACL24F / AAAI24 / 2024 |
| LLM 剪枝(持续预训练恢复) | Sheared LLaMA | 弱/否（持续预训练+DBL，非经典KD） | ICLR24 |
| LLM 剪枝(prompting恢复) | Compresso | 弱（collaborative prompting） | ICLR24 |
| LLM 剪枝(度量搜索) | Pruner-Zero | 否 | ICML24 |
| LLM 剪枝(训练免) | DuoGPT | 否 | NeurIPS25 |
| 最早进 LLM 在策略蒸馏(2026) | Entropy-Aware OPD, SOD, EasyOPD | 是 | ICML26 / 2026 |

---

## 几点提醒
1. **AFP 与 Padé 在 arXiv 上未找到**对应剪枝+蒸馏论文；AD-KD（ACL 2023）是「ADS」最可能的指代。若你有这两项的具体出处，提供链接后再补核实。
2. **Sheared LLaMA 的「蒸馏策略」**：原论文核心是 targeted structured pruning + Dynamic Batch Loading + 持续预训练，**并没有显式 logit/feature KD 损失**；它属于「教师初始化 + 数据驱动恢复」。若二手资料称其用 KD，需回到原文确认。
3. 环境里 WebSearch 不可靠（会生成错误作者/arXiv ID），上述全部条目都用 arXiv API 的 `id_list`/`search_query` 直接拉取真实 metadata 做了交叉核对；Semantic Scholar 验证了 Wanda 的 venue=ICLR（其余 ICLR 2024 项的 comment 字段虽为空，但与社区/OpenReview 记录一致，可信）。
