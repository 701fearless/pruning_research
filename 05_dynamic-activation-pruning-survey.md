# 05 — 动态激活结合剪枝 论文调研（2024–2026）

> 三个子方向：(A) 不同推理步数激活不同大小模型；(B) 不同剪枝阶段/比例下恢复步数/强度不同；(C) 对时间步 t 的敏感度测试/加权指导剪枝。
>
> **核验方式**：每条 arXiv ID 均经 arXiv API（`export.arxiv.org`）真实返回的 title/作者/日期核验，由主流程二次直读确认（非仅依赖 agent 转述）。WebSearch 仅作发现线索，其给出的作者/ID 不予信任。
>
> **venue 诚实声明**：arXiv API 不返回发表 venue。下表 venue 列中带「据 web」者为多源网页佐证但未经本流程独立核验，引用前建议用 OpenReview/DBLP 复核；标「未核验」者表示未找到任何 venue 佐证。年份依据 API `published` 日期，前缀 25xx=2025、26xx=2026。

---

## 方向 A：不同推理步数激活不同大小模型（timestep-level routing / dynamic sub-network）

按 t 路由到不同剪枝程度/宽度/专家子网络。最贴切定义的是 ALTER 与 DiffPruning（剪枝程度随 t 变）；DyDiT 系宽度级；SparseDiT/DDiT 为 token/patch 粒度；MoDE/eDiff-I 为专家模型级。

| # | 方法 | 完整标题 | 作者 | 年月 | venue | arXiv | 核验 | 一句话主题 |
|---|---|---|---|---|---|---|---|---|
| A1 | **ALTER** | ALTER: All-in-One Layer Pruning and Temporal Expert Routing for Efficient Diffusion Generation | Xiaomeng Yang, Lei Lu, Qihui Fan, … Yanzhi Wang, Xuan Zhang, Shangqian Gao | 2025-05 | 未核验 | [2505.21817](https://arxiv.org/abs/2505.21817) | ✅ | 用 hypernetwork 把 UNet 转成时序专家混合，按 timestep 动态生成层剪枝决策并路由到不同剪枝程度子网络（SD2.1 仅 25.9% MACs）。**最贴切** |
| A2 | **DiffPruning** | Mixture of Efficient Diffusion Experts Through Automatic Interval and Sub-Network Selection | Alireza Ganjdanesh, Yan Kang, Yuchen Liu, Richard Zhang, Zhe Lin, Heng Huang | 2024-09 | 据 web NeurIPS 2024 | [2409.15557](https://arxiv.org/abs/2409.15557) | ✅ | 把去噪 timestep 聚类成区间，每区间单独微调出 depth/width 弹性可变专家子网络，端到端选每步用哪套配置。**最贴切** |
| A3 | **DyDiT** | Dynamic Diffusion Transformer | Wangbo Zhao, Yizeng Han, Jiasheng Tang, Kai Wang, Yibing Song, Gao Huang, Fan Wang, Yang You | 2024-10 | 据 web ICLR 2025 | [2410.03456](https://arxiv.org/abs/2410.03456) | ✅ | Timestep-wise Dynamic Width 按生成时间步自适应调整模型宽度（激活不同 head/MLP 通道子集），DiT-XL 减 51% FLOPs。 |
| A4 | **DyDiT++** | DyDiT++: Diffusion Transformers with Timestep and Spatial Dynamics for Efficient Visual Generation | Wangbo Zhao, Yizeng Han, Jiasheng Tang, Kai Wang, Hao Luo, Yibing Song, Gao Huang, Fan Wang, Yang You | 2025-04 | 未核验 | [2504.06803](https://arxiv.org/abs/2504.06803) | ✅ | DyDiT 期刊扩展版，timestep+空间动态推广到 FLUX/Latte/视频及 TD-LoRA。 |
| A5 | **SparseDiT** | SparseDiT: Token Sparsification for Efficient Diffusion Transformer | Shuning Chang, Pichao Wang, Jiasheng Tang, Fan Wang, Yi Yang | 2024-12 | 据 web NeurIPS 2025 | [2412.06028](https://arxiv.org/abs/2412.06028) | ✅ | 时间维度随去噪阶段动态调节 token 密度（早期少、后期多），DiT-XL 减 55% FLOPs。属"步数级不同计算量"的 token 级实现。 |
| A6 | **MoDE** | Efficient Diffusion Transformer Policies with Mixture of Expert Denoisers for Multitask Learning | Moritz Reuss, Jyothish Pari, Pulkit Agrawal, Rudolf Lioutikov | 2024-12 | 未核验 | [2412.12953](https://arxiv.org/abs/2412.12953) | ✅ | 稀疏专家层 + 噪声条件路由，不同噪声级/时间步激活不同专家子网络，减 40% 活跃参数、90% FLOPs（机器人 diffusion policy）。 |
| A7 | **eDiff-I** | eDiff-I: Text-to-Image Diffusion Models with an Ensemble of Expert Denoisers | Yogesh Balaji, Seungjun Nah, … Ming-Yu Liu (NVIDIA) | 2022-11 | 未核验（奠基工作，早于2024） | [2211.01324](https://arxiv.org/abs/2211.01324) | ✅ | 训练一组分别专精不同采样阶段的专家 denoiser，不同时间步用不同模型。该思路的早期代表。 |
| A8 | **DDiT** | DDiT: Dynamic Patch Scheduling for Efficient Diffusion Transformers | Dahye Kim, Deepti Ghadiyaram, Raghudeep Gadde | 2026-02 | 未核验 | [2602.16968](https://arxiv.org/abs/2602.16968) | ✅ | 按 denoising timestep 动态切换 patch 尺寸（早期粗 patch 建结构、后期细 patch 补细节），FLUX-1.dev 3.52× 加速。 |

### 方向 A 边缘相关（不严格切题，列出供参考）
- ⚠️ **SODA** [2603.07057] 按 timestep/layer/module 敏感度做 caching+pruning 混合调度，偏 caching。
- ⚠️ **LazyDiT** [2412.12444] lazy learning 复用前步结果跳过计算，caching 类。
- ⚠️ **DRiffusion** [2603.25872] draft-and-refine 并行/speculative 类，非按 t 路由子网络。

---

## 方向 B：不同剪枝阶段/比例下自适应恢复（步数/强度/数据量随进度变化）

注意：本方向目前命中的多为 **LLM/VLM 剪枝恢复**，diffusion 语境下专门工作仍稀少。最贴合"按剪枝进度自适应调整恢复调度"的是 ShortOPD、M2M-DC、IDEA Prune、NutePrune、Cascaded Multi-Granularity；其余偏"恢复方法本身"。

| # | 方法 | 完整标题 | 作者 | 年月 | venue | arXiv | 核验 | 一句话主题 |
|---|---|---|---|---|---|---|---|---|
| B1 | **ShortOPD** | ShortOPD: Recovering Pruned LLMs with Short-to-Long On-Policy Distillation | Qingyu Zhang, Qianhao Yuan, Hongyu Lin, … Ming Xu, Jiarui Li | 2026-07 | 未核验 | [2607.13124](https://arxiv.org/abs/2607.13124) | ✅ | 短→长 on-policy 蒸馏恢复调度，按策略当前可用长度自适应分配 rollout 预算，避免早期恢复预算浪费于低信息后缀。**切题** |
| B2 | **M2M-DC** | MI-to-Mid Distilled Compression (M2M-DC): … Progressive Inner Slicing Approach to Model Compression | Lionel Levine, Haniyeh Ehsani Oskouie, Sajjad Ghiasvand, Majid Sarrafzadeh | 2025-11 | 未核验 | [2511.06842](https://arxiv.org/abs/2511.06842) | ✅ | 信息引导块剪枝 + 渐进式内层切片 + 分阶段 KD 交替，切片间穿插短 KD 阶段，恢复量随进度调整。**切题** |
| B3 | **IDEA Prune** | IDEA Prune: An Integrated Enlarge-and-Prune Pipeline in Generative Language Model Pretraining | Yixiao Li, Xianzhi Du, Ajay Jaiswal, … Tuo Zhao, Chong Wang, Jianyu Wang | 2025-03 | 未核验 | [2503.05920](https://arxiv.org/abs/2503.05920) | ✅ | enlarge/剪枝/recovery 统一在单一 cosine 退火 LR 调度下，迭代式结构化剪枝使 recovery 与剪枝进度耦合。**切题** |
| B4 | **NutePrune** | NutePrune: Efficient Progressive Pruning with Numerous Teachers for Large Language Models | Shengrui Li, Junzhe Chen, Xueting Han, Jing Bai | 2024-02 | 据 web ICLR 2024 | [2402.09773](https://arxiv.org/abs/2402.09773) | ✅ | 渐进式剪枝中用多个不同容量"教师"逐步指导学生，教师强度随剪枝进度变化。**切题** |
| B5 | **SlimQwen** | SlimQwen: Exploring the Pruning and Distillation in Large MoE Model Pre-training | Shengkun Tang, Zekun Wang, Bo Zheng, … Zhiqiang Shen, Dayiheng Liu | 2026-05 | 未核验 | [2605.08738](https://arxiv.org/abs/2605.08738) | ✅ | MoE 预训练下系统对比剪枝+蒸馏，结论"渐进式剪枝调度优于一次性压缩"，并提 multi-token prediction 蒸馏。 |
| B6 | **PASER** | PASER: Post-Training Data Selection for Efficient Pruned Large Language Model Recovery | Bowei He, Lihao Yin, Hui-Ling Zhen, … Chen Ma | 2025-02 | 未核验 | [2502.12594](https://arxiv.org/abs/2502.12594) | ✅ | 按各能力退化程度自适应分配恢复数据预算到不同语义簇，优先挑下降最多的样本（自适应恢复数据量）。**切题** |
| B7 | **Cascaded Multi-Granularity Pruning** | Cascaded Multi-Granularity Pruning for On-Device LLM Inference in Industrial IoT | Jinghan Wang, Yanjun Chen, Wei Zhang, … Gaoliang Peng | 2026-06 | 未核验 | [2606.26861](https://arxiv.org/abs/2606.26861) | ✅ | 粗→细级联剪枝（层/头/FFN），阶段间插入轻量 low-rank recovery 重估重要性，每阶段恢复量随粒度调整。**切题** |
| B8 | **Equivariant-Aware Pruning** | Equivariant-Aware Structured Pruning for Efficient Edge Deployment: … Adaptive Fine-Tuning | Mohammed Alnemari | 2025-11 | 未核验 | [2511.17242](https://arxiv.org/abs/2511.17242) | ✅ | 自适应微调——准确率下降超 2% 自动触发，配 early stopping+LR scheduling 恢复（触发与强度自适应）。偏 G-CNN 边缘部署，弱切题 |
| B9 | **RLRC** | RLRC: Reinforcement Learning-based Recovery for Compressed Vision-Language-Action Models | Yuxuan Chen, Yixin Han, Yize Huang, Xiao Li | 2025-06 | 未核验 | [2506.17639](https://arxiv.org/abs/2506.17639) | ✅ | 三阶段剪枝-SFT/RL recovery-量化管线，critic warm-up+BC loss 正则稳定 RL 恢复阶段。偏恢复方法本身 |
| B10 | **STARFISH** | STARFISH: faST Accuracy Recovery in pruned networks From Internal State Healing | Shir Maon, Odelia Melamed, Adi Shamir | 2026-05 | 未核验 | [2606.01126](https://arxiv.org/abs/2606.01126) | ✅ | 剪枝后用极小无标签校准集对齐原始网络内部状态表示恢复，校准量随剪枝激进度自适应（75% 剪枝仅 0.4% 图像）。 |
| B11 | **SPADE** | SPADE: Structured Pruning and Adaptive Distillation for Efficient LLM-TTS | Tan Dat Nguyen, Jaehun Kim, Ji-Hoon Kim, … Joon Son Chung | 2025-09 | 未核验 | [2509.20802](https://arxiv.org/abs/2509.20802) | ✅ | WER 引导层重要性剪枝 + 多级 KD 恢复自回归连贯性（"Adaptive Distillation"多级恢复）。 |
| B12 | **Self-Data Distillation** | Self-Data Distillation for Recovering Quality in Pruned Large Language Models | Vithursan Thangarasa, Ganesh Venkatesh, Mike Lasby, … Sean Lie | 2024-10 | 未核验 | [2410.09982](https://arxiv.org/abs/2410.09982) | ✅ | 用原始未剪枝模型生成自蒸馏数据集恢复剪枝质量，缓解 SFT 灾难性遗忘。偏恢复方法 |

### 方向 B 边界相关（无显式"按进度自适应恢复调度"，标边界）
- ⚠️ **Boomerang Distillation** [2510.05064] 先蒸到小模型再渐进式回嵌教师层重建中间尺寸，无显式 KD loss 调度。
- ⚠️ **SHIFT-LLM** [2608.25068] 训练免剪枝后用线性残差适配器+可选 PEFT 校正，恢复偏静态校正。
- ⚠️ **LVLM Structural Pruning Study** [2604.24380] 经验研究不同剪枝级别下 SFT+hidden-state 蒸馏恢复差异，实证非方法。
- ⚠️ **Data-Free KD Recovery** [2511.20702] DeepInversion 合成数据做 data-free KD 恢复，无进度自适应调度。

---

## 方向 C：对时间步 t 的敏感度测试/加权指导剪枝

直接切题（测各 t 敏感度→按 t 加权打分剪枝）5 篇，边界相关 4 篇。

| # | 方法 | 完整标题 | 作者 | 年月 | venue | arXiv | 核验 | 一句话主题 |
|---|---|---|---|---|---|---|---|---|
| C1 | **OBS-Diff** | OBS-Diff: Accurate Pruning For Diffusion Models in One-Shot | Junhan Zhu, Hesong Wang, Mingluo Su, Zefang Wang, Huan Wang | 2025-10 | 据 web ICLR 2026 | [2510.06751](https://arxiv.org/abs/2510.06751) | ✅ | one-shot 训练免剪枝；提出 timestep-aware Hessian 构造，用对数递减加权方案给更早时间步更大权重以抑制误差累积。**最贴切** |
| C2 | **AT-EDM** | Attention-Driven Training-Free Efficiency Enhancement of Diffusion Models | Hongjie Wang, Difan Liu, Yan Kang, … Niraj K. Jha, Yuchen Liu | 2024-05 | 据 web CVPR 2024 | [2405.05252](https://arxiv.org/abs/2405.05252) | ✅ | 用 attention map 做运行时 token 剪枝，提出 DSAP（Denoising-Steps-Aware Pruning）跨不同去噪步调整剪枝预算。**切题** |
| C3 | **Early-Bird Diffusion** | Early-Bird Diffusion: Investigating and Leveraging Timestep-Aware Early-Bird Tickets in Diffusion Models for Efficient Training | Lexington Whalen, Zhenbang Du, Haoran You, Chaojian Li, Sixu Li, Yingyan Lin | 2025-04 | 未核验 | [2504.09606](https://arxiv.org/abs/2504.09606) | ✅ | 发现 diffusion early-bird tickets，让 timestep-aware EB tickets 按各时间步区域重要性自适应稀疏度（关键区域保守、非关键激进）。**切题** |
| C4 | **SparseDiT**（同 A5） | SparseDiT: Token Sparsification for Efficient Diffusion Transformer | Shuning Chang, Pichao Wang, Jiasheng Tang, Fan Wang, Yi Yang | 2024-12 | 据 web NeurIPS 2025 | [2412.06028](https://arxiv.org/abs/2412.06028) | ✅ | 时间维度随去噪阶段动态调节 token 密度，属 timestep-aware token 剪枝。跨方向 A/C |
| C5 | **Timestep-Aware Block Masking** | Timestep-Aware Block Masking for Efficient Diffusion Model Inference | Haodong He, Yuan Gao, Weizhong Zhang, Gui-Song Xia | 2026-03 | 未核验 | [2603.19939](https://arxiv.org/abs/2603.19939) | ✅ | 为每个时间步学独立 block mask，引入 timestep-aware loss scaling 优先保护敏感去噪阶段，剪掉冗余时空依赖。**切题** |

### 方向 C 边界相关
- ⚠️ **DiP-GO** [2410.16942] 据 web NeurIPS 2024 — 剪枝转 SubNet 搜索，基于相邻 t 特征相似性建 SuperNet，偏"利用相邻 t 相似性做子网搜索"而非"测 t 敏感度→t 加权"。
- ⚠️ **SiTo**（⚠️ 无 arXiv）据 web AAAI 2025 — 相似性 token 剪枝，处理跨时间步重复剪同一 token 问题；arXiv 未检索到，无 ID，引用前需自查原文。
- ⚠️ **TFMQ-DM** [2311.16503] 据 web CVPR 2024（扩展版 TPAMI 2025）— **量化非剪枝**，但为 diffusion 时间步敏感度分析奠基工作（指出时间步嵌入对扰动更敏感导致轨迹偏离），方法论高度相关。
- ⚠️ **APT** [2608.25380] 软硬件协同加速器，attention probability 引导剪枝+Timestep-Aware FlashAttention，偏硬件，timestep 未直接用于剪枝打分加权。

---

## 跨方向观察

1. **最贴合三问的"主力论文"**：
   - 步数级激活不同大小子网络 → **ALTER (2505.21817)**、**DiffPruning (2409.15557)**、**DyDiT (2410.03456)**
   - 剪枝阶段自适应恢复 → **ShortOPD (2607.13124)**、**M2M-DC (2511.06842)**、**IDEA Prune (2503.05920)**、**NutePrune (2402.09773)**、**PASER (2502.12594)**
   - 时间步 t 敏感度加权剪枝 → **OBS-Diff (2510.06751)**、**AT-EDM (2405.05252)**、**Early-Bird Diffusion (2504.09606)**、**Timestep-Aware Block Masking (2603.19939)**

2. **领域分布**：方向 A、C 以 **diffusion/DiT** 为主；方向 B 目前以 **LLM/VLM** 为主——diffusion 语境下"按剪枝进度自适应恢复"的专门工作仍稀少，这是可切入的研究空白。

3. **重叠**：SparseDiT (2412.06028) 同时属方向 A（步数级不同计算量）与 C（timestep-aware 剪枝）。

4. **可信度**：全部 32 个 arXiv ID 经 arXiv API 真实核验；venue 列除"据 web"标注外均未独立核验（arXiv API 不提供 venue）。引用 venue 前请用 OpenReview/DBLP 复核。

## 待办
- [ ] 用 DBLP/OpenReview 复核"据 web"标注的 venue（DyDiT ICLR2025 / SparseDiT NeurIPS2025 / AT-EDM CVPR2024 / DiP-GO NeurIPS2024 / OBS-Diff ICLR2026 / NutePrune ICLR2024 / TFMQ-DM CVPR2024）
- [ ] SiTo 的正式发表出处（arXiv 无收录）
