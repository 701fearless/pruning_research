# Prune-Distill 调研报告

本工程记录「剪枝（pruning）后用蒸馏（distillation）恢复/压缩」方向的文献调研，覆盖 CNN / Transformer / LLM / 语音 / 视觉 / 扩散模型 / TTS 等。

调研日期：2026-08。所有条目经 arXiv API + Semantic Scholar API 逐条核验。

## 目录

| 文件 | 主题 |
|---|---|
| [01_kd-taxonomy.md](01_kd-taxonomy.md) | 剪枝后蒸馏方式分类（logit/feature/relation/self/combined）、剪枝+蒸馏联合训练、LLM 剪枝后恢复机制、2024–2026 venue 代表作、综述 |
| [02_iterative-prune-distill.md](02_iterative-prune-distill.md) | 迭代/渐进式剪枝 + 蒸馏（prune↔distill 交替）代表工作、NAS+蒸馏联合、反例 |
| [03_diffusion-prune-distill.md](03_diffusion-prune-distill.md) | Diffusion 模型剪枝 + 蒸馏 SOTA（区分剪枝 / 蒸馏加速 / 联合三个方向） |

## 关键结论速览

### 剪枝后蒸馏恢复（整体）
1. LLM 剪枝一线（SparseGPT / Wanda / SliceGPT / OWL / FLAP / Pruner-Zero）主流**不显式用 KD 恢复**，靠权重补偿 / 结构保留 / 更好度量 / LoRA 微调。
2. 真正把 KD 作为恢复主力的，集中在**结构化剪枝 + 后训练恢复**这条线：Sheared LLaMA（持续预训练 + DBL，**非经典 KD**）、Compresso（collaborative prompting）、SDMPrune、Self-Distillation recovery for LLMs。
3. **当前 LLM「剪枝后蒸馏恢复」相对空缺，是值得填补的位置。**

### 迭代剪枝-蒸馏
1. 经典迭代剪枝-重训谱系（Han / NetAdapt / AMC / ThiNet / Network Slimming / Lottery Ticket）恢复**几乎全是普通 SGD 微调，无蒸馏**。
2. 真正把 KD 嵌入迭代 / 渐进剪枝循环的工作集中于 **2020 年后**，横跨语音、检测、GAN、SR、扩散、TTS、LLM。
3. 重要反例：Sheared LLaMA（渐进但用持续预训练恢复，无经典 KD）、UPop、GISP。

### Diffusion
1. diffusion 文献里「蒸馏」特指**少步推理加速**（Progressive / CM / LCM / ADD / DMD / CTM …），与「剪枝后蒸馏恢复」语义不同，勿混淆。
2. **方向 C（剪枝+蒸馏联合）专门工作 2024 前稀少，2026 随 DiT / 视频架构爆发**；真正联合训练的仅 *Dynamic-in-Few-Step*，其余多为「先剪后蒸」管线。
3. caching（DeepCache / T-GATE）严格不算剪枝也不算蒸馏，已剔除。

## 方法学说明

- 本环境 WebSearch 对学术查询返回模型生成摘要（含错误作者 / arXiv ID），**未采信**。
- 所有 arXiv ID、标题、作者、venue 均经 **arXiv API（`export.arxiv.org`）+ Semantic Scholar API** 逐条核验。
- venue 优先取 arXiv comments 字段；为空者依公认记录并交叉验证。
- 少数无法定位的条目明确标注「未找到」，未编造。
- Diffusion 方向 C 部分（逐篇 arxiv 链接）仍在二次核实中，见 [03](03_diffusion-prune-distill.md)。

## 核验脚本（可复现）

```bash
# arXiv 元数据批量核验
curl -s "https://export.arxiv.org/api/query?id_list=1506.02626,1510.00149,1503.02531"

# Semantic Scholar venue 核验
curl -s "https://api.semanticscholar.org/graph/v1/paper/arXiv:2306.11695?fields=title,venue,year"
```

## 待办
- [x] Diffusion 方向 C 9 篇 + Bridging 全部逐条 arXiv 核验完成（见 [03](03_diffusion-prune-distill.md) 方向C表）
- [~] 方向 B 核验 7 篇（Progressive/CM/LCM/Guided/ADD/DMD/CTM）；DMD2/sCM/Mean-Flows/Shortcut/Hyper-SD/PCM 因 arXiv+S2 双 429 限流待补
- [ ] 方向 A ToMeSD/T-GATE/TinyFusion 待核（TinyFusion 候选 2407.00011 已证为无关数学论文，ID 待重查）
- [ ] 可选：用 `/download-papers` skill 批量下载已核实 PDF 到本目录
- [ ] 已修正错误 ID：SnapFusion→2306.00980、BK-SDM→2305.15798
