# 剪枝蒸馏论文：蒸馏 Loss 设计对比

> 基于 `papers/` 目录下各论文原文（含 PPCL、HierarchicalPrune、Amber-Image、NanoFLUX、BRIDGING、E-MMDiT、Minitron/Nemotron Nano 2、Unified Pruning ViT、DPHuBERT、Staged TTS 等）逐篇抽取的蒸馏损失公式与设计要点。
> 整理日期：2026-09-18

---

## 0. 总览表

| 论文 | 模态 | 蒸馏对齐层位 | Loss 形式 | 独特设计 |
|---|---|---|---|---|
| **PPCL** | 扩散 MMDiT | 特征（hidden state） | L2 归一化 MSE | **非顺序输入**（student 吃 teacher 输出），区间独立 |
| **HierarchicalPrune** | 扩散 MMDiT | 速度场 + 特征 | 网络输出 MSE + 块特征 MSE | **反重要性加权**（重要块少更新/冻结）SGDistill |
| **Amber-Image** | 扩散 MMDiT | 特征（hidden state） | layer-wise MSE / 拼接隐藏态 MSE | 局部权重平均初始化 + **只训重初始化层** + 混合流 |
| **NanoFLUX** | 扩散 MMDiT | 网络输出 + 注意力头特征 | 输出 MSE + 头级特征 MSE | SVD 分析剪头；T5 编码器 block-wise 蒸馏 |
| **BRIDGING** | 扩散 U-Net | 网络输出（噪声区间） | 高噪声 MSE（repair）+ 对抗一步蒸馏 | **repair 桥接剪枝与步数蒸馏** |
| **E-MMDiT** | 扩散 MMDiT | 特征（与 DINOv2 对齐） | REPA 正则 + 对抗步蒸馏 | 从头设计，token 压缩 |
| **Minitron (Nemotron)** | LLM | logit 分布 | forward-KL（详见 Minitron §3） | 与 RL/偏好优化交替"回血" |
| **Unified Pruning ViT** | ViT | logit 分布（或末层特征） | L_CE + α·L_KL | PVTv2 改用特征 MSE（α=20） |
| **DPHuBERT** | 语音 SSL | 特征 | L1 + cosine 等权组合 | L0 正则联合剪枝蒸馏 |
| **Staged TTS** | 语音 TTS | 网络输出 | flow-matching 目标 | warm-start + 分阶段 + WER 门控 |
| **OBS-Diff** | 扩散 | 无蒸馏 | — | 免训练二阶 OBS |
| **WHAT MATTERS / CoCurve / Structural Sensitivity / ADAPTIVE PRUNING / ANALYZING FF** | LLM/ViT | 无蒸馏 | — | training-free |

---

## 1. 扩散模型系列

### 1.1 PPCL — 非顺序跨层特征蒸馏（arXiv:2511.16156）

**核心思想**：传统顺序蒸馏中，student 每层的输入来自"前一个被压缩的 student 层"，压缩误差逐层传播叠加，导致表征与 teacher 语义漂移。PPCL 让 student 层**直接吃 teacher 在剪枝区间前的输出**，切断误差链。

**深度剪枝（Stage 1.3）**——对每个冗余连续区间 [u, v]，用一层 student 复现整段 teacher：

$$L^{[u,v]}_{depth} = \left\| \operatorname{Norm}\big(S^{init}_u(T^D_{u-1})\big) - \operatorname{Norm}(T^D_v) \right\|_2^2, \qquad L_{depth} = \sum_i L^{[u_i,v_i]}_{depth}$$

- 输入 `T^D_{u-1}`：teacher 第 u−1 层输出（**非顺序**：不依赖前一个 student 层）
- 目标 `T^D_v`：teacher 第 v 层输出
- `Norm(·)`：沿特征维 **L2 归一化**，只对齐方向、稳定训练
- 各区间独立求和 → 模块化、即插即用

**宽度剪枝（Stage 2）**——对每个被瘦身层 j ∈ R_txt ∪ R_ffn：

$$L^j_{width} = \left\| \operatorname{Norm}\big(S^j_{width}(T^D_{j-1})\big) - \operatorname{Norm}(T^D_j) \right\|_2^2 + L^j_{linear}$$

$$L^j_{linear} = \begin{cases} \|z^{txt}_j - T^z_j\|_2^2 + \|h^{txt}_j - T^{txt}_j\|_2^2 & j \in R_{txt} \\[4pt] \|h^{img}_j - T^{img}_j\|_2^2 + \|h^{txt}_j - T^{txt}_j\|_2^2 & j \in R_{ffn}\end{cases}$$

- 第二项 `L_linear` 是**普通逐点 MSE**（不加归一化），把被替换的流输出直接拉向 teacher 对应层输出

**消融证据（Qwen-Image 20B，Table 2）**：顺序蒸馏 baseline 掉点 18.2% → +线性探针选层 14.5% → **+非顺序蒸馏 5.22%**（最大单项贡献）→ +文本流瘦身 3.79% → +FFN 瘦身 4.91% → +全参微调 **2.61%**。

---

### 1.2 HierarchicalPrune — SGDistill 灵敏度引导蒸馏（arXiv:2508.04663）

**总损失**：`L = L_feat + L_KD`

$$L_{KD} = \mathbb{E}_{t, x}\big[\, \|v_{\theta'}(x_t, t) - v_{\theta}(x_t, t)\|^2 \,\big]$$

$$L_{feat} = \mathbb{E}\Big[ \sum_{i \in \mathcal{B}^*} \|f^i_{\theta'}(x_t, t) - f^i_{\theta}(x_t, t)\|^2 \Big]$$

- `L_KD`：**速度场 MSE**——student 与 teacher 的去噪速度场（flow-matching 的 v）逐点对齐（Eq.3）
- `L_feat`：对被选更新块集合 B\* 的**块特征 MSE**（Eq.4）

**SGDistill（反直觉但有效的设计）**：分析发现"重要性越高的块对变化越敏感"，激进剪枝下更新这些高重要性块反而有害（即便有位置权重 PWP 仍退化 31.9%）。因此对每个块**按重要性反比缩放参数更新**（scale = 1/ΔP，越重要更新越小、甚至冻结），把更新集中到低敏感区域。

**效果**：激进剪枝（30%）下质量退化从 31.9% → **10.1%**；配合 INT4 量化，SD3.5-L-Turbo 内存 15.8→3.24GB（−79.5%），FLUX.1-Schnell 22.6→4.44GB（−80.4%）。

---

### 1.3 Amber-Image — 局部权重平均 + 隐藏态 MSE（arXiv:2602.17047）

**10B 变体（深度剪枝 + 双流转单流）**：
- **局部权重平均初始化**（Local Weight Averaging）：把被剪掉的 teacher 层权重按位置加权平均，初始化相邻 student 层，缓解剪枝后 loss spike
- **定向 layer-wise 蒸馏**：只有**被重新初始化的层**参与训练，其余冻结；蒸馏目标 = teacher 末层在被剪簇对应位置的隐藏态，让 student 层**隐式重建整段累积效果**

**6B 变体（30 层，10 双流 + 20 单流）**：
- 前 10 个双流层**冻结**作语义锚点；后 20 个单流层对齐 teacher 的**双流拼接隐藏态**：

$$H^{(l)}_{target} = \operatorname{Concat}\big(H^{(l,teacher)}_{text},\; H^{(l,teacher)}_{image}\big), \qquad L_{MSE} = \|H^{(l)}_{student} - H^{(l)}_{target}\|_2^2$$

- 单流权重用 teacher 的**图像流投影权重初始化**（W_shared = W_image，因图像流是空间结构主通路）
- 最后轻量全参微调（标准扩散 loss）收尾

**特点**：整条 10B+6B 管线 <2,000 GPU 时，压掉 70% 参数，DPG/GenEval 反而**反超教师** Qwen-Image。

---

### 1.4 NanoFLUX — 输出蒸馏 + 注意力头特征 + T5 编码器蒸馏（arXiv:2602.06879)

**DiT 蒸馏（核心）**：

$$L_{distill}(x_t) = \|\, f_S(x_t, t, p; \theta_S) - f_T(x_t, t, p; \theta_T) \,\|_2^2$$

- 直接对**网络输出**（flow-matching 去噪预测）做逐 timestep MSE
- 因为剪头数（24→16），特征级 loss 不可直接加，改在**注意力头级**做特征对齐——把每个头的输出按 token 平均后再比较（H_T 与 H_S 头数不同）：

$$L_{features} = \sum_{l} \left\| \frac{1}{H_T}\sum_{h} o^{l,h}_T(x_t) - \frac{1}{H_S}\sum_{h} o^{l,h}_S(x_t) \right\|$$

**T5 文本编码器蒸馏（5B→330M，分两阶段）**：
1. 两层 MLP 对齐输出维度，**prompt embedding MSE** warmup
2. **block-wise 蒸馏**：采样截止时间步 t̂，t < t̂ 的梯度截断（避免高噪声区间梯度不稳），只监督前 3 个 transformer 块的 prompt 隐藏态 MSE

**效果**：17B→2.4B（约 7×），移动端 512px 约 2.5s；对比 Wang et al. 每 timestep 都蒸馏的做法，block-wise 设计更稳。

---

### 1.5 BRIDGING — repair 桥接 + 一步蒸馏（arXiv:2607.06335）

**洞察**：剪枝后直接做一步蒸馏会失败（FID 112.56）——因为剪枝使 student 在一步生成所用的高噪声区间偏离 teacher，蒸馏损失在该区间的梯度无用。需要一个**短的 teacher-alignment repair 桥接阶段**。

**Repair 损失**（固定高噪声水平 σ_r，真实图像 latent）：

$$L_{repair}(G_S) = \mathbb{E}_{x,y,\epsilon}\big[\, \|G_S(\tilde{z}, \sigma_r, y) - T(\tilde{z}, \sigma_r, y)\|_2^2 \,\big], \qquad \tilde{z} = z + \sigma_r \epsilon$$

- 只在高噪声区对齐（σ_r = max{σ_init, σ_1..σ_K}，实验里 = 2.5），**不是完整重训**，几十步 Adam
- 本质：把剪枝后的生成器推进到"蒸馏损失梯度有用"的 basin

**之后**：用修复好的 checkpoint 初始化一步蒸馏 θ_0 = θ_align，loss D = **SiDA**（score identity + 真实图像 + 对抗训练）。

**效果**：20% 剪枝 + repair + 一步：98.8M 参数、1 NFE、**FID 3.12（反低于教师 3.53）**；无 repair 直接 SiDA = 112.56（崩）。该 repair 也可配 Diff-Instruct（CIFAR-10 验证通用性）。

---

### 1.6 E-MMDiT — REPA 正则 + 步骤蒸馏（arXiv:2510.27135）

- **REPA**：训练时把模型隐藏态与 **DINOv2 特征**做表示对齐（作正则项，不是主要训练目标）
- **步骤蒸馏**：adversarial distillation（Nitro-1），20 步教师 → 1–4 步学生，1M 合成数据
- 注意：E-MMDiT 是**从头设计**的轻量模型（304M），非后剪枝，蒸馏作用是把多步教师压成少步学生

### 1.7 OBS-Diff — 无蒸馏（arXiv:2510.06751）

- **完全免训练**：one-shot，二阶 OBS saliency + 时间步感知 Hessian，无任何蒸馏/微调
- 代表"免训练"路线的 SOTA：SD3.5-L 60% 稀疏 FID 29.15（vs Wanda 48.80）

---

## 2. LLM 系列

### 2.1 Minitron (Nemotron Nano 2) — forward-KL logit 蒸馏（arXiv:2508.14444）

- **logit-based，仅前向 KL**（准确系数与是否含 CE 项详见 Minitron 原文 §3）：

$$L = D_{KL}\big(P_{teacher} \,\|\, P_{student}\big)$$

- teacher = 对齐后的 12B 模型；student = 剪枝后的 9B；词表不变，logits 可直接比
- **与 RL 交替"回血"**：深度剪后 KD ~60B@8k → 宽度剪后 KD ~50B@8k + ~25B@49k + ~1B@262k → DPO → GRPO → **KD ~0.4B@262k（恢复 RL 掉点）** → RLHF → 0.5 线性模型合并
- 效果：12B→9B，比 Qwen3-8B 快 3–6× 且精度略超

### 2.2 免蒸馏的 LLM 剪枝（training-free）

| 论文 | 打分 | 为什么能免蒸馏 |
|---|---|---|
| **WHAT MATTERS** (2406.15786) | 模块输入/输出余弦相似度 S=1−cos(X,Y) | 注意力层固有冗余，one-shot 移除即可 |
| **CoCurve** (2607.17568) | 二阶 Fisher **co-pruning 曲率**（边+节点） | 全局贪心选最少"联合损害"单元 |
| **Structural Sensitivity** (2603.20991) | Lyapunov 收缩因子 \|ρ−1\| | 两次前向排序，物理删层 |

---

## 3. ViT 系列

### 3.1 Unified Pruning ViT — soft logit 蒸馏（arXiv:2111.15127）

$$L = L_{CE}(y, p) + \alpha\, L_{KL}(q, p)$$

- 原模型作教师，**classic soft distillation**；对比 hard-label 蒸馏、soft+patch 蒸馏，soft 最优
- **PVTv2 场景改用倒数第二层（penultimate）特征 MSE，α=20**——直接蒸馏 logits 不如蒸馏深层特征
- 效果：UP-DeiT-S top-1 81.56%（比 DeiT-S 高 1.7pp）、3.03× 加速；UP-DeiT-T 甚至不蒸馏也更强

### 3.2 ADAPTIVE PRUNING — 无蒸馏

- 微分包含 mask 解路径一次搜索得多种稀疏度，只做少量微调，不走蒸馏路线

---

## 4. 语音 / 其他

### 4.1 DPHuBERT — layer-to-layer 蒸馏（arXiv:2305.17651）

- **对齐层位**：中间层特征，匹配层集合 S = {0, 4, 8, 12}
- **Loss**：L1 + **cosine 距离等权组合**（非 MSE）
- **联合目标**：min E[ L(f(x_k), f(x_k; θ̃)) ] + λ‖θ̃‖₀，用 hard-concrete（L0 正则）+ 增广拉格朗日求稀疏约束，**剪枝与蒸馏在同一目标里联合**（这是它和"先剪后蒸"路线的区别）
- 两步：Step1 联合剪+蒸，Step2 纯蒸馏微调（Step2 必要：Step1 中正则项与蒸馏互相竞争）
- 效果：HuBERT-Base 94.68M→23.59M，SUPERB 10 任务 8 项优于纯蒸馏方法

### 4.2 Staged Depth-Pruning TTS — flow-matching 目标 + warm-start（arXiv:2607.18662）

- 剪枝后**从教师权重 warm-start**（非 block 张量 1:1 拷贝，只收缩深度）
- 每阶段用 **conditional flow-matching 目标**在教师生成的语料（17.6h）上微调；LR 5e-5、500 步 warmup、EMA、Euler ODE
- **ASR-WER 门控**：上一阶段 WER 没恢复就不继续剪下一级
- 效果：249M 未见句 WER 0.00；131–249M 保留教师约 96% 自然度，102M 出现容量悬崖

### 4.3 ANALYZING FEED-FORWARD BLOCKS — 无蒸馏（纯分析）

- 用 norm-based + Integrated Gradients 归因，发现 FF 块约占参数 2/3、存在冗余并与残差组件互相抵消——为层剪枝提供依据

---

## 5. 横向对比要点

### 5.1 对齐层位分三档（监督信号往哪对齐）

| 层位 | 代表论文 | 特点 |
|---|---|---|
| **logit 分布** | Minitron、Unified Pruning ViT | 最上层、软信息最丰富；要求 student 与 teacher 词表/输出维度一致 |
| **特征（hidden state）** | PPCL、Amber-Image、NanoFLUX、DPHuBERT、Unified-PVTv2、E-MMDiT(REPA) | 能传递中间表征；要选对齐哪些层，PPCL 用它配合非顺序输入 |
| **网络输出（速度场/去噪预测）** | HierarchicalPrune、NanoFLUX、BRIDGING repair、Staged TTS | 扩散模型天然把"输出对齐"当作 loss 主体（速度场 MSE） |

> 扩散模型里 logit/输出层就是去噪预测，所以"输出级 MSE"（HierarchicalPrune、NanoFLUX）本质就是匹配去噪场，与 LLM 的 logit 蒸馏相对应。

### 5.2 蒸馏输入构造：顺序 vs 非顺序

- **顺序**：student 吃 student（传统，误差传播）
- **非顺序**：student 吃 teacher（PPCL：直接砍掉 9 个点的掉点；Amber-Image：冻结锚点层、只训重初始化层；BRIDGING：repair 也在 teacher 输入上对齐）
- 结论：**剪枝越激进，越该让 student 在 teacher 的表征空间里"跳跃"训练**。

### 5.3 蒸馏权重/范围调节（防止过度更新伤到关键结构）

| 论文 | 机制 |
|---|---|
| **HierarchicalPrune** | 反重要性加权 1/ΔP，重要块少更新/冻结 |
| **PPCL** | 区间独立、L2 归一化只对齐方向 |
| **Amber-Image** | 只训重初始化/单流层，其余冻结当锚点 |
| **NanoFLUX** | 高噪声时间步截断梯度 |
| **BRIDGING** | 只在高噪声区做短 repair，不重训全谱 |

### 5.4 与其他阶段/技术的耦合

- **与 RL 对齐**：Nemotron 把 KD 穿插在 DPO/GRPO/RLHF 之间当"回血"；HierarchicalPrune 目标模型本身是 Turbo（已对齐）版
- **与步数蒸馏**：BRIDGING 明确"剪枝恢复"和"步数蒸馏"要桥接，直接连会崩
- **与量化**：HierarchicalPrune 蒸馏 + INT4；Amber-Image 蒸馏后轻量微调
- **免训练路线**：OBS-Diff / CoCurve / WHAT MATTERS 完全不蒸馏——省资源，但上限一般低于蒸馏（蒸馏可以反超教师：Amber、BRIDGING、Unified-PViT）

### 5.5 对扩散剪枝的启示

1. **速度场/去噪输出 MSE 是对齐扩散模型的最自然选择**（HierarchicalPrune、NanoFLUX、BRIDGING 全部用输出级 MSE），而 LLM 用 logit-KL；
2. **噪声时间步需要特殊处理**：OBS-Diff 用时间步加权 Hessian、NanoFLUX 用截止时间步、BRIDGING 固定高噪声 σ_r——"在哪噪声区间对齐"本身就是设计变量；
3. **特征级蒸馏（hidden state）通常与输出级蒸馏配合**（HierarchicalPrune 的 L_feat + L_KD，PPCL 的 L_width + L_linear），单独用一类往往不够；
4. **越激进的剪枝越需要"保护关键层"的机制**（反重要性加权 / 冻结锚点 / 局部训练），这是从 10% 掉点迈进 3% 掉点的关键。
