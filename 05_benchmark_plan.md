# Benchmark 验证安排：DiT 文生图剪枝 + 蒸馏

> 目标：验证"剪枝 + 蒸馏"方法在 DiT 骨干上的有效性，并最终在 T2I MMDiT 上达到 SOTA。
> 策略：两里程碑递进——先在原版 DiT/ImageNet 验机制，再扩到 FLUX/SD3 验 T2I + 打 SOTA。

---

## 0. 总体策略与分工

```
里程碑 1（DiT / ImageNet）          里程碑 2（FLUX 或 SD3 / T2I）
─────────────────────────────       ─────────────────────────────
验"通用剪蒸机制"                     验"T2I/MMDiT/文本相关" + 打 SOTA
- 层重要性度量                        - 双/单流块剪枝、双流转单流
- 剪蒸 schedule                      - text stream / cross-attn 剪枝
- 特征蒸馏损失                       - velocity (flow-matching) 蒸馏
- 非顺序蒸馏                         - DPG/GenEval/OneIG 对标
对标 Bridging (FID 3.12)             对标 NanoFLUX / PPCL
```

**分工原则**：DiT 阶段验不到文本/双流部分，不当作 FLUX 的全包预演；FLUX 阶段才是 SOTA 声明落点。两层各司其职（与 PPCL/Amber-Image/NanoFLUX 多模型验证一致）。

---

## 1. 里程碑 1：原版 DiT / ImageNet

### 1.1 起点模型（教师/待剪）

- **DiT-XL/2**（Peebles & Xie，~675M，ImageNet-256 / 512 类条件）
- 架构特征：单流 DiT block + AdaLN（class label 调制）+ DDPM（ε-prediction）
- 选它理由：干净、小、FID 基准齐全、可与 Bridging/EDM2 直接对标

### 1.2 Benchmark 与指标

| Benchmark | 测什么 | 复现目标 |
|---|---|---|
| **ImageNet-256 类条件生成** | 标准 FID | 复现 DiT-XL/2 dense FID（约 2.27，250 步 DDPM） |
| **ImageNet-512（可选）** | 与 Bridging 同 setting | 复现 EDM2-XS dense FID 3.53，对标 Bridging 的 3.12 |

**指标**：
- 主：FID（clean-fid 实现，固定 Inception 权重）
- 辅：CLIP-Score、sFID、Precision/Recall
- 效率：参数量、FLOPs、推理延迟、采样步数 NFE

### 1.3 SOTA 对标

| 对象 | 角色 | 数字 |
|---|---|---|
| dense DiT-XL/2 | 上界 | FID ≈ 2.27（256）/ EDM2-XS 3.53（512） |
| **Bridging**（20% 剪 + repair + 一步 SiDA） | **直接竞品 SOTA** | FID **3.12**，1 NFE，98.8M |
| random / magnitude 剪枝 | 下界 | 自跑 |
| 只剪不蒸 | 消融 | 自跑 |

**里程碑 1 的"达标"**：在 ImageNet-512、相近参数/NFE 下 FID 接近或低于 Bridging 的 3.12；或在相近 FID 下参数/NFE 更优。

### 1.4 消融（DiT 上做满）

- [ ] 剪枝比例曲线（10/20/30/40/50%）FID
- [ ] 重要性度量对比：时间步敏感消融 vs CKA 连续区间 vs magnitude
- [ ] 蒸馏消融：只剪不蒸 / 先剪后蒸 / 剪蒸交织 / 联合
- [ ] 非顺序蒸馏 vs 逐层蒸馏（误差累积对照）
- [ ] 不同蒸馏目标：中间 hidden state vs ε-prediction vs 二者

### 1.5 里程碑 1 产出

- 一套 work 的剪枝 + 蒸馏 recipe（重要性度量 + schedule + 损失）
- ImageNet-512 上对 Bridging 的对比表
- 结论：方法在 DiT 骨干上有效（但尚未验文本/双流）

---

## 2. 里程碑 2：FLUX 或 SD3 / T2I（SOTA 落点）

### 2.1 起点模型（二选一为主，另一为辅）

| 模型 | 架构 | 用途 |
|---|---|---|
| **FLUX.1-dev**（~12B DiT，rectified flow，双/单流块） | MMDiT | **主结果 + 打 SOTA**（NanoFLUX/PPCL 同模型） |
| **SD3** / PixArt-α（0.6B，干净小 DiT） | MMDiT/DiT | 辅助消融、快速验证 MMDiT 部分 |

### 2.2 Benchmark 与指标（DiT 压缩三件套 + 人类偏好 + 效率）

| Benchmark | 测什么 | 来源 |
|---|---|---|
| **DPG-Bench** | 1065 密集语义 prompt，组合遵循 | Amber/PPCL/NanoFLUX |
| **GenEval** | 对象/数量/颜色/位置/属性绑定 | Amber/PPCL/NanoFLUX |
| **OneIG-Bench** | alignment/text/reasoning/style/diversity 中英 | Amber/PPCL/NanoFLUX |
| **HPSv3 / HPDv2** | 人类偏好 | NanoFLUX |
| 移动端延迟/显存 | 部署（若做端侧） | NanoFLUX |

**效率指标**：参数量、FLOPs、GPU 显存、推理延迟、采样步数。

> 不用 COCO 30K FID 作主指标——FLUX 在 COCO 上 FID 饱和且测不出组合语义，剪枝最易掉的恰是组合性，必须用 DPG/GenEval。

### 2.3 SOTA 对标（FLUX 线）

| 对象 | 角色 | 数字（talking.md 已核） |
|---|---|---|
| dense FLUX.1-dev | 上界 | 自复现 |
| **NanoFLUX**（~2.4B） | **直接竞品 SOTA** | OneIG≈42.1 / DPG≈75.5 / GenEval≈49.7 / HPSv3≈10.41，手机 512≈2.5s |
| **PPCL-FLUX**（~8B） | 竞品（大档） | ~74.4% 显存，平均性能降约 4.03% |
| 跨架构：SnapFusion/BK-SDM/LAPTOP-Diff | 对照 | U-Net 系（COCO FID） |
| random/magnitude、只剪不蒸 | 下界/消融 | 自跑 |

**里程碑 2 的"达标 = SOTA"**：在 FLUX.1-dev 上、与 NanoFLUX 相近参数/FLOPs 下，DPG/GenEval/OneIG 任一超过 NanoFLUX；或相近质量下参数/延迟更小。

### 2.4 里程碑 2 新增验证（DiT 阶段验不到的）

- [ ] 双/单流块剪枝：哪些双流块可删、双流转单流（对标 Amber-Image）
- [ ] text stream 剪枝：保留 QKV、其余换线性投影（对标 PPCL）
- [ ] cross-attention head 剪枝（文本条件路径）
- [ ] velocity（flow-matching）蒸馏损失（替代 ε）
- [ ] 时间步自适应 token 分辨率（若做端侧，对标 NanoFLUX）

---

## 3. 评测工具链（别自己实现）

- **FID**：`clean-fid`（或 `torch-fidelity`），固定 Inception 权重版本
- **CLIP-Score**：`open_clip` + ViT-L/14
- **DPG/GenEval/OneIG**：用各 benchmark 官方评测脚本，**照竞品论文的版本号复刻**，不换 Inception/CLIP 权重
- 全部指标**报竞品同版本**，否则不可比

---

## 4. 复现纪律（动创新前必做）

在跑任何剪枝+蒸馏创新前，先复现 dense 教师基线到接近原论文：

- [ ] 里程碑 1：DiT-XL/2 / EDM2-XS 在 ImageNet 上 FID 复现 ±0.3
- [ ] 里程碑 2：FLUX.1-dev 在 DPG/GenEval/OneIG 上复现到竞品报告的 dense 数字 ±合理误差

**理由**：FID/DPG 差 1.0 全是评测噪声，不复现教师基线则"我的数字变化来自方法还是评测 bug"无法分辨，结论全废。

---

## 5. 时间安排建议

| 阶段 | 内容 | 周期 |
|---|---|---|
| M1-a | 复现 DiT-XL/2 dense + 评测链 | 1–2 周 |
| M1-b | 实现剪枝 + 蒸馏 recipe + 消融 | 3–4 周 |
| M1-c | ImageNet-512 对标 Bridging 出表 | 1 周 |
| M2-a | 复现 FLUX.1-dev dense + DPG/GenEval/OneIG | 2–3 周 |
| M2-b | recipe 迁到 MMDiT + 双流/text/velocity 部分 | 4–6 周 |
| M2-c | 对标 NanoFLUX/PPCL 出主表 + SOTA 声明 | 2 周 |

**预算提醒**：FLUX dense 复现 + 多组剪枝实验是大头算力，M2 阶段至少预留 M1 的 3–5 倍 GPU 时。

---

## 6. 风险与诚实标注

1. **小模型冗余度低**：DiT-XL（675M）和 PixArt-α（0.6B）剪枝比大模型疼，结果可能偏悲观，不能直接外推到 FLUX 20B 级。论文需写明 limitation。
2. **DiT 阶段验不到文本/双流**：M1 通过不代表 M2 通过，M2 的 MMDiT 特有部分仍要从头验。
3. **benchmark 不可混用**：ImageNet FID（M1）与 DPG/GenEval（M2）数字不可比，SOTA 声明只落在 M2。
4. **未核实项**：NanoFLUX/PPCL 的精确 prompt 子集、采样配置（guidance scale、步数）需在 M2-a 阶段回原 repo 确认，否则 FID/DPD 不可比。
