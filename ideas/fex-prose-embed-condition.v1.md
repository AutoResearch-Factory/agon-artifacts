---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-prose-embed-condition/
---

- One-sentence summary: 用 PROSE 预训练的符号编码器 (4 层 self-attention, 在 512K PDE 系统上训练) 作为冻结特征提取器, 将其输出的 PDE 结构嵌入向量作为 FEX controller 的条件输入, 测试跨社区的表示迁移是否能改善 FEX 搜索.

- Hypothesis: PROSE 的符号编码器在 512K 个 PDE 系统上训练, 学到了 PDE 结构的丰富表示 (算子类型、非线性度、耦合模式等). FEX 的 controller 是一个无条件 MLP, 不接收任何 PDE 信息, 每次从零搜索. 假设: 将 PROSE 符号编码器的输出 (一个 512 维向量, 编码了 PDE 的结构特征) 作为 FEX controller 的额外条件输入, FEX 的搜索效率应该提升, 因为 controller 现在"知道"自己在解什么类型的 PDE. 这不同于 fex-foundation-model (用自定义的 20 维统计特征) -- PROSE 的表示是在大规模 PDE 数据上端到端学到的, 信息量远大于手工特征.

- Expected outcome: 在 PROSE-PDE 涵盖的 PDE 族上, PROSE-conditioned FEX controller 的搜索步数比无条件 FEX 少 20% 以上 (paired Wilcoxon p < 0.05). 与 fex-foundation-model 的 20 维手工统计特征对照, PROSE embedding 应在复杂 PDE (KdV, conservation law) 上优势更大 (因为手工特征可能遗漏非线性结构信息). 失败信号: PROSE embedding 与 FEX 的 RL 奖励信号 (PDE 残差) 不兼容, 导致 controller 学到的表示在梯度更新中被破坏.

- Contribution type: method

- Risk: MEDIUM

- Estimated effort:
  - Compute: 40-80 GPU-hours (PROSE 编码器推理 ~2h, FEX 训练 5 PDE 族 x 10 实例 x 3 seeds ~50h, 对照 ~20h)
  - Data: available (PROSE-PDE 公开代码和权重, FEX 标准 PDE)
  - Implementation: 2-3 weeks

- Novelty quick-check: (1) fex-foundation-model (本项目已有) 用 20 维手工统计特征条件化 FEX controller, v2 pilot 显示 PDE 残差改善 11% 但 p=0.25. 差异: 用预训练的 deep 表示替代手工特征, 信息量和质量应更高. (2) FormulaGPT (2404.06330) 将 RL 搜索蒸馏到 Transformer 做 in-context 策略, 但不是冻结预训练表示做条件化. (3) DGSR (2401.00282) 预训练条件生成模型但不用 PROSE 类 PDE 专用编码器. 差异化: 首次用跨社区的预训练 PDE 表示 (Schaeffer 组的 PROSE 编码器) 条件化另一个社区的搜索方法 (Yang 组的 FEX controller).

- Strongest objection: PROSE 符号编码器的训练目标是为 PROSE 自己的 fusion/decoder 服务, 学到的表示可能对 FEX 的 RL 优化目标不相关甚至有害 -- 这是跨任务 feature transfer 的经典风险.

- Why we should do this: 这是两个连通分支之间最优雅的表示级桥梁: 用 Schaeffer 组的预训练表示增强 Yang 组的搜索方法, 不修改任何一方的核心算法, 只连接接口. 如果有效, 这证明 PDE 结构的通用表示可以跨方法论迁移; 如果失效, 失效分析本身就是有价值的: "为什么为 Transformer 预测训练的 PDE 表示不能帮助 RL 搜索?"

- Pilot:
  - Setup: 用 PROSE-PDE 的预训练符号编码器 (从公开 checkpoint 加载, 冻结权重) 对 10 个 PDE (Poisson, Advection, Burgers, KdV, Wave) 生成 512 维嵌入向量. 训练一个小 FEX controller (3 层 MLP, 128 hidden), 输入 = PROSE embedding, 输出 = 算子动作分布. 在 Poisson 和 Advection 上训练, 在 Burgers 上测试 (zero-shot 迁移).
  - Metric: conditioned controller 在 Burgers 上的 solve rate > unconditioned controller 的 solve rate, 差异 > 10pp.
  - Result: [待测]
  - Signal: [待测]

<review date="2026-06-25">

## Novelty

- Score: 3/10
- Closest prior work:
  1. **LLM+FEX (Bhatnagar, Liang, Patel & Yang 2025, arXiv 2503.09986)** — Yang 组**自己**的工作, 用在 PDE 数据上 fine-tune 的 transformer (LLaMA-3 8B) 读取**控制方程** (PDE type / RHS / BC) 来通知 FEX. 与本 idea 同一目标 (用预训练模型加速 FEX 搜索), 同一信号源 (方程侧). 差异仅在 conditioning 机制: 2503.09986 用输出做 operator-set pruning (hard mask), 本 idea 用 ~512 维 soft embedding 喂 controller MLP. 这是同一被占据问题上的机制变体.
  2. **A Unified Framework for Deep Symbolic Regression (Landajuela et al., NeurIPS 2022)** — set-transformer 把问题编码成 latent 来 condition/initialize RL RNN controller (p(τ|θ,ψ,(X,y))). "预训练 encoder embedding → condition RL 符号 controller" 的范式已存在; 本 idea 只换了 encoder (PROSE vs set-transformer) 和 frozen vs end-to-end.
  3. **fex-foundation-model (本项目 score=5)** — 已经测试了**几乎相同的核心假设** (用 PDE 特征 condition FEX controller 加速搜索). 唯一区别是特征来源: fex-foundation-model 用 20 维手工统计特征, 本 idea 用 PROSE 冻结 embedding.
- Key differentiator: "用 PROSE 这个跨社区预训练模型作冻结特征提取器" + "soft 512 维 embedding 替代手工特征/operator mask" 确实没有逐字先例. 但本质是 "Apply X (pretrained encoder conditioning) to Y (FEX controller)" 的**双重变体** — 既是 2503.09986 的机制变体, 又是 fex-foundation-model 的特征来源变体. 两条 differentiator 都是工程替换, 不是新机制或新问题. 独立 novelty-check subagent 判定 MEDIUM-LOW, 与本 reviewer 一致.

## 核心概念缺陷: 方程 embedding ≠ 解结构 embedding

这是本 idea 最严重的问题, 高于一般 novelty/quality 关切.

PROSE-PDE 的 Symbol Encoder (wiki/2404.12355.md §管线 step 4) **输入的是控制方程本身** (Polish notation of the PDE), 它的 embedding 编码的是**方程结构**. 但 FEX 求解 PDE 时**已经知道控制方程** (algorithm 拿到了算子和 RHS), 它搜索的是**解的表达式树**. 因此:

- 本 idea 写 "PROSE ... 输出的 PDE 结构嵌入向量 ... 编码了 PDE 的结构特征 (算子类型、非线性度、耦合模式等)" 并假设这能告诉 controller "自己在解什么类型的 PDE". 但 controller 通过 PDE 输入**已经知道**算子类型/非线性度 — PROSE embedding 在这里只是把 FEX **本来就持有的方程 token** 做了一次有损 re-encoding. 这是接近**同义反复 (near-tautological)** 的低信息条件.
- 真正有用的信号是 "给定方程, **解**里会出现哪些算子" (方程结构 → 解算子的映射). 而 PROSE 的 frozen Symbol Encoder **从未为这个目标训练过** — 它训练目标是与 data 融合后预测数值解/重建方程, 不是预测解的算子集. 2503.09986 之所以有效 (报告 FEX 搜索轮数减 40-60%), 正是因为它**显式 fine-tune** 一个模型去输出**解所需的算子**. 本 idea 用一个没学过这个映射的冻结 embedding, 凭什么提供这个信号, 没有给出论证.
- idea 自己的 "Strongest objection" (PROSE 表示为自己的 fusion/decoder 服务, 对 FEX RL 目标可能不相关甚至有害) 已经触及这一点, 但停在 "跨任务 feature transfer 的经典风险", **没有意识到更尖锐的问题不是"transfer 会不会失败", 而是"被 transfer 的 embedding 编码的根本是错的东西 (方程而非解)"**.

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 3/10 | (1) 方程 embedding ≠ 解结构 (见上), 核心机制建立在 conflation 上. (2) "PROSE embedding 信息量远大于手工特征" 是**未论证断言** — 信息量大不等于对 FEX RL 目标**相关**; 一个为别的任务训练的高维表示可能恰恰把无关方差喂给 controller. (3) PROSE Symbol Encoder 输出的是 token **序列** embedding 而非单个 512 维向量, 如何 pool 成固定向量 (mean? CLS? 末 token?) 是未提及的设计决策, 且 pooling 方式直接决定信号质量. (4) idea 假设 frozen embedding 与 RL reward "不兼容" 只是失败信号之一, 但没讨论 conditioning 网络如何把 512 维注入一个原本极小的 MLP controller 而不淹没 policy gradient. |
| Missing evidence signals | 2/10 | (1) **Result/Signal 全是 [待测]** — pilot 完全没跑, 这是当前最大短板. fex-foundation-model 至少有 v2 PDE pilot (尽管 p=0.25). 本 idea 零 empirical signal. (2) 缺最关键的 sanity probe: PROSE frozen embedding 对 "解算子" 的**线性可分性/top-k recall** — 在投入任何 FEX 实验前, 应先验证 "PROSE 方程 embedding 能否线性预测解里出现的算子". 若不能, 整个 idea 当场否决. (3) 缺与 2503.09986 operator-mask 的对照 (必须). (4) 缺与手工特征 (fex-foundation-model) 的同条件对照. |
| Narrative | 5/10 | "两个连通分支之间最优雅的表示级桥梁" 叙事清晰且有吸引力, landscape §23 也铺垫了 FEX-PROSE bridge. 但叙事优雅掩盖了机制不成立的风险. "如果失效, 失效分析本身有价值 (为什么为 Transformer 预测训练的 PDE 表示不能帮助 RL 搜索?)" 是合理的 fallback framing, 但这个 "为什么" 在概念层面**已经能部分回答** (因为编码的是方程不是解), 不需要花 40-80 GPU-h 才发现. |
| Venue contribution | 3/10 | 以 NeurIPS/ICML/ICLR/AAAI 标准: 单纯 "用 PROSE embedding condition FEX" 既撞 2503.09986 (同组) 又撞 fex-foundation-model (同项目), 即使 positive 也读作 ablation 而非 contribution. 以 JMLR/Neural Networks 标准: 若能给出 "哪种 PDE 表示 (手工 / operator-mask / frozen-foundation-embedding) 对 FEX 最有效" 的系统对照 + 清晰的失效边界, 有边际发表价值, 但这要求把 idea 重构为对照研究 (见 Alternative Framing). |
| Testability | 6/10 | 有明确 falsifier (Burgers zero-shot solve rate diff > 10pp). 但 (a) pilot 依赖一个**可能不存在的 PROSE checkpoint** (见 Outcome realism), 当前 falsifier 实际上跑不起来; (b) 用 solve-rate-difference 而非 effect-size+CI, 单点 10pp 阈值在小样本下噪声大. 建议先做不依赖 FEX 的 "embedding → 解算子" 可分性 probe 作为最便宜 gate. |
| Outcome realism | 2/10 | **资产可用性是硬伤**: idea 的 pilot 写 "从公开 checkpoint 加载, 冻结权重", compute 估算 "PROSE 编码器推理 ~2h". 但 wiki/2404.12355.md R4 明确记 "未提供训练好的 checkpoint", github felix-lyx/prose 顶层未见可下载权重 (sibling fex-prose-skeleton-warm 也只敢说**数据集**公开). 若 checkpoint 不存在, 必须**从头训练 PROSE-PDE** (单卡 4090 ~4.5h × 完整 pipeline + 数据生成 512K 系统), 这**完全不在** 40-80 GPU-h 预算内, 且把 idea 从 "2-3 周" 变成数月. 此外预期 (Burgers zero-shot solve rate +10pp) 在 fex-foundation-model PDE pilot (solve rate 两组均 1.2%, p=0.25) 的实证背景下偏乐观; 2509.19849 的 pretrained-SR-OOD-鸿沟进一步压低跨族 zero-shot 期望. |
| Contribution type compliance | n.a. | idea type = method; topic 0616-fex 声明 preferred-contribution-types: [method, theory, application] — method ∈ 列表, 合规. (此维度按惯例不计入 Overall Quality 平均) |
| Overall Quality | 3.5/10 | 叙事和 testability 尚可, 但核心机制建立在 "方程 embedding ≈ 解结构先验" 的 conflation 上, pilot 零 empirical signal, 关键资产 (PROSE checkpoint) 可能不存在导致预算严重低估, 且双重撞车 (2503.09986 + fex-foundation-model). |

## Contribution Drift (n >= 2)

- n = 1 (仅 v1), 不适用. 首次 review.

## Sibling 重叠分析 (本 idea 在 fex-prose-* 家族中的定位)

0616-fex topic 下 2026-06-24 一次性产出 8 个 fex-prose-* idea, 本 idea 是其中之一. 与本 idea 直接重叠/竞争的:

- **fex-foundation-model (score 5)**: 核心假设几乎相同 (condition FEX controller 加速搜索), 仅特征来源不同 (手工 20 维 vs PROSE embedding). 本 idea 本质是 fex-foundation-model 的一个 "特征来源消融臂". **强烈建议合并**: 把 "PROSE frozen embedding" 作为 fex-foundation-model 实验矩阵里的一个 feature variant (与手工特征、operator-mask 并列对照), 而非独立 idea. 独立存在会稀释两者. 
- **fex-prose-skeleton-warm (score 0)**: 用 PROSE **符号解码器**生成骨架候选 (FEX 做系数优化). 比本 idea 信息路径更直接 (解码器输出的是解结构而非方程 embedding), 概念上更站得住. 本 idea 用 encoder embedding 是更弱的信号路径.
- **fex-prose-entropy-bound (score 应较高, pilot POSITIVE)**: 给 PROSE-prior → FEX 搜索效率建立理论关系, 已有 synthetic pilot R²=0.93. 本 idea 是它的一个实证特例但缺理论.

## Claims Discipline

idea 当前无 claims matrix. 若推进 (或合并入 fex-foundation-model), 应限定:

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 在限定 PDE families / FEX tree depth / compute budget 下, **特定 pooling 的 PROSE frozen embedding** 作为 controller 条件能显著降低搜索步数, 且在同条件下**优于手工特征和 operator-mask baseline**. 不能声称 "跨社区表示通用迁移". 必须先证 embedding 对解算子有可分信息. |
| NULL | 当前 frozen PROSE embedding / pooling / controller 注入方式不足以提供有效条件; 仅排除 "frozen PROSE-encoder embedding 直接 condition" 这一条, 不否定 operator-mask (2503.09986 已 positive) 或解码器骨架 (skeleton-warm) 路线. |
| NEGATIVE | frozen 方程 embedding 引入无关方差/有害偏置, 降低 solve rate. 可形成 "为预测/重建训练的 PDE 表示编码的是方程而非解, 因此不是 FEX 搜索的有效先验" 的失效边界结论 — 但注意这个结论在概念层面已可部分预判, empirical 只是确认. |

## Alternative Framing

更可辩护的两条路:
1. **合并消融**: 取消独立 idea, 把 PROSE frozen embedding 并入 fex-foundation-model 的 "feature source" 对照轴 (handcrafted-20d / PROSE-frozen-embed / 2503.09986-operator-mask / oracle-operator-set). 这样 positive/null/negative 都能写成 "哪种 PDE 表示对 FEX 条件化最有效" 的系统结论, 且**自动包含强 baseline**.
2. **改信号路径**: 若坚持用 PROSE, 改用**符号解码器输出 (解骨架)** 而非 encoder embedding (即 fex-prose-skeleton-warm 路线), 因为解码器输出的是解结构, 不犯方程-vs-解的 conflation. encoder-embedding 路线是三条 PROSE→FEX 路径里信号最弱的.

## Likelihood-Impact Matrix

- Priority: Low (3) = Likelihood: Low x Impact: Medium
- Numeric score for ideas.xml: 3
- Rationale:
  - Likelihood = Low: 核心机制建立在方程-vs-解 conflation 上, frozen PROSE 方程 embedding 是否携带 "解算子" 信号未经任何验证 (零 pilot); 即便携带, 还要在小 controller 上不淹没 policy gradient、pooling 设计正确、并击败两个已有 baseline (operator-mask + 手工特征). 叠加 PROSE checkpoint 可能不存在导致需从头训练. 这是 "需要多个关键且互相独立的条件同时成立, 且其中一个 (信号路径正确性) 概念上就存疑".
  - Impact = Medium: 若 positive, 它给 "跨社区预训练表示能 condition 符号搜索" 提供一个数据点, 对 FEX 研究线有局部价值. 但因为同时被 2503.09986 (同组) 和 fex-foundation-model (同项目) 夹击, 即使成功也偏 incremental/ablation, 不构成独立 high-impact 叙事. 若 negative, 失效结论概念上已可部分预判, 增量信息有限. 故 Impact 封顶 Medium (不到 fex-foundation-model 的 High — 后者至少是 "FEX 首个 foundation model" 的独立叙事载体, 本 idea 是其特征臂).
  - Claude 判断: Likelihood=Low, Impact=Medium. (独立 novelty-check subagent 亦判 MEDIUM-LOW novelty, 一致.)

## Overall

- Priority: Low
- Score: 3
- Comments: 这是 0616-fex 在 2026-06-24 批量产出的 8 个 fex-prose-* idea 之一, 首次 review. 叙事 ("FEX-PROSE 表示级桥梁") 优雅且 landscape 有铺垫, testability 框架尚可. 但三个硬伤压低评分: (1) **核心概念 conflation** — PROSE Symbol Encoder 编码的是**控制方程**, FEX 求解时已持有方程, 它搜索的是**解**; 用方程 embedding 去 condition 解搜索是近同义反复的低信息条件, 而真正有用的 "方程→解算子" 映射 PROSE frozen encoder 从未训练过 (2503.09986 正是显式 fine-tune 这个映射才 work). (2) **双重撞车** — 既是 Yang 组自己 2503.09986 的机制变体 (soft embedding vs operator mask), 又是本项目 fex-foundation-model 的特征来源变体 (PROSE embed vs 手工 20 维). (3) **零 pilot + 资产风险** — Result/Signal 全 [待测], 且 pilot 依赖一个 wiki 明确记为 "未提供" 的 PROSE checkpoint, 若需从头训练则 40-80 GPU-h 预算严重低估. **建议**: 不作为独立 idea 推进, 而是 (a) 合并进 fex-foundation-model 作为 "PROSE-frozen-embedding" 特征对照臂 (自动获得强 baseline + 系统对照叙事), 或 (b) 若坚持用 PROSE, 转向信号更直接的 fex-prose-skeleton-warm (用解码器输出解骨架, 不犯方程-vs-解 conflation). 若仍要独立推进, **decisive 最便宜 gate**: 先做不依赖 FEX 的 probe — "PROSE frozen 方程 embedding 能否线性预测/top-k recall 解里出现的算子", 若不能则当场否决, 省下全部 FEX 实验预算.

</review>
