---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-prose-coarse-refine/
---

- One-sentence summary: 构建 PROSE-coarse + FEX-refine 双阶段 PDE 求解器, 其中 PROSE 快速给出数值近似解和结构假设, FEX 从 PROSE 的结构假设出发搜索精确符号解, 系统测量双阶段是否比各自单独使用在精度-速度 Pareto 前沿上占优.

- Hypothesis: PROSE 在 ~5ms 内给出 ~1-3% 精度的数值解和方程骨架, 但无法达到机器精度. FEX 能达到 ~1e-7 精度, 但从零搜索可能需要数百轮 RL. 假设 PROSE 的数值解能指导 FEX 的搜索起点 (从数值解的局部行为推断候选算子), 那么双阶段求解器应在 Pareto 前沿上严格优于两者单独使用. 具体机制: (1) PROSE 数值解在若干点的 Taylor 系数可以提示 FEX 哪些一元算子可能出现; (2) PROSE 符号解码的 top-k 骨架直接约束 FEX 搜索空间; (3) PROSE 数值解作为 FEX 的初始系数猜测 (通过在候选表达式上做最小二乘拟合). 核心问题: 这三种信息传递渠道中哪个贡献最大?

- Expected outcome: 在 PROSE-PDE 涵盖的 20 族 1D PDE 上, PROSE+FEX 双阶段在 wall-clock 相同条件下精度优于单独 FEX, 或在精度相同条件下 wall-clock 少于单独 FEX 30% 以上. 通过消融 (关闭三个信息传递渠道中的每一个) 确定主导渠道. 失败信号: PROSE 的结构预测在 FEX 精度标准下噪声太大, 双阶段不如 FEX 从零搜索 + 多 restart.

- Contribution type: method

- Risk: MEDIUM

- Estimated effort:
  - Compute: 80-150 GPU-hours
  - Data: available (PROSE-PDE 公开数据集)
  - Implementation: 3-4 weeks

- Novelty quick-check: (1) NOWS (2511.02481) 用 neural operator 为 FEM 迭代求解器做 warm-start, 但是数值求解器 warm-start 数值求解器, 不涉及符号搜索. (2) SymTorch (2602.21307) 从 PINN distill 出符号表达式, 是两阶段 (NN -> 符号) 但不含 RL 搜索. (3) 无工作将 PROSE 类 PDE 基础模型的输出 (数值+符号) 作为 RL 符号搜索的 warm-start. 差异化: 首次将 PDE 基础模型的多模态输出 (数值解 + 符号骨架 + 系数估计) 作为 RL 符号 PDE 搜索的分层 warm-start.

- Strongest objection: 增加 PROSE 推理阶段的工程复杂度可能不值得, 如果 FEX 用多 restart 或 LLM+FEX operator mask 就能达到类似加速.

- Why we should do this: 这是 FEX 和 PROSE 最自然的功能性桥梁. 它不是简单地 "apply X to Y", 而是问一个深层问题: 两种根本不同的 PDE 求解范式 (神经算子 vs RL 符号搜索) 能否在同一管线中发挥互补优势? 消融实验将揭示哪种信息传递 (数值 / 结构 / 系数) 最有价值, 为未来的混合求解器设计提供设计指南.

- Pilot:
  - Setup: 1D Burgers 方程, 5 个实例. PROSE 推理得到数值解 + top-3 骨架. 在每个骨架上用 scipy 做系数拟合, 取最小残差的骨架. 比较: (a) FEX from scratch, (b) FEX 从 PROSE 骨架出发只做系数搜索, (c) FEX 从 PROSE 数值解推断的 operator hint 出发搜索.
  - Metric: 方案 (b) 或 (c) 的 relative L2 < 1e-4 达标率 > 方案 (a) 的达标率.
  - Result: [待测]
  - Signal: [待测]

<review date="2026-06-25">

## Novelty

- Score: 4/10
- Closest prior work: 内部 sibling `fex-prose-skeleton-warm` (本批次, 几乎同题); LLM+FEX operator-set prediction (Bhatnagar et al. 2503.09986); PROSE-PDE Skeleton mode (2404.12355); PROSE+SymPy 系数精炼 (2409.11609); FEX+TranNet (2604.22208).
- Key differentiator: 唯一相对新的点是把 PROSE 的**三类输出 (数值解 / top-k 骨架 / 系数估计) 并列作为 FEX 的分层 warm-start 并做通道消融**. 但拆成 claim 看, 三条通道各自都已被覆盖: (A) 骨架通道 = sibling `fex-prose-skeleton-warm` + `fex-llm-skeleton` + Bhatnagar, 只是把"骨架来源"从 LLM 换成 PROSE decoder; (B) 系数通道 (在候选式上做最小二乘) = PROSE+SymPy 的 SMC 粒子滤波系数精炼 (2409.11609, 同 Schaeffer 组) + `fex-llm-skeleton` Gate 2 的 SAGE-Fit; (C) 数值解 Taylor 系数 → operator hint 是唯一未被直接做过的子点, 但高度 speculative 且 idea 自己也只给了一句话. 整体属于"组合三条已知 warm-start 通道 + 消融"的工程式贡献, 而非新方法范式. novelty 只能来自消融**结论**本身 (哪条通道最有价值), 而非方法.

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 4/10 | 最大跳跃: 从 PROSE 的 ~1-3% 数值误差 / 99.94% 训练内 token-validity 推到"足以指导 FEX 达到 1e-7 符号精度". idea 自己的 failure-signal 也承认这点. 更关键: Yang 组**自己**的 FEX+TranNet (2604.22208) 已实证 FEX 对候选/骨架精度极敏感——真算子在池中时 ~1e-7, 移除后误差暴涨 ~4 个数量级, 局部行为不像的替代算子伤害更大. 这直接削弱"PROSE 近似骨架够用"的前提. 通道 C (Taylor 系数 → 算子身份) 完全 hand-wave, 未说明局部 Taylor 数据如何区分 exp vs 高次多项式. |
| Missing evidence signals | 3/10 | Pilot 全部 `[待测]`, 零信号. 决定性 baseline 未进 pilot: Bhatnagar operator-set prior、FEX from-scratch + multi-restart、idea 自己承认的替代方案 (LLM+FEX operator mask). 还需要: top-k 骨架在 FEX 精度下的等价率、operator hint 的 precision/recall、系数初始化相对随机初始化的收敛增益. |
| Narrative | 5/10 | "FEX 与 PROSE 最自然的功能性桥梁 / 哪条通道贡献最大"是连贯故事, 但是**组合+消融**故事, 不是"答案无论哪个方向都重要"的故事. 三条通道并列堆叠, 主线容易塌成工程拼接. 比 sibling `fex-prose-skeleton-warm` 更宽但不更锋利. |
| Venue contribution | 3/10 | 对 AAAI/ICLR/JMLR/Neural Networks: "把 PROSE 三类输出作为 FEX warm-start 并消融"偏增量. 更成熟的 sibling `fex-llm-skeleton` (已 3 轮 refine + 真实 pilot + 分层 Gate + 锋利叙事) 当前定标在 5; 本 v1 在 pilot / 机制 specification / baseline / 叙事每一轴都严格弱于该 sibling. |
| Testability | 7/10 | 这是 idea 的强项: pilot metric ("方案 b/c 的 rel-L2<1e-4 达标率 > 方案 a") 在 5 个 Burgers 实例上是干净廉价的可证伪信号. 失败信号也明确. 唯一问题是完全未执行, 且 cheapest falsifier 应更小 (先证 PROSE 骨架/hint 不降低 FEX 成功率即可). |
| Outcome realism | 4/10 | 80-150 GPU-h / 3-4 周合理. 但">30% wall-clock 节省 / 严格 Pareto 占优"过强: (1) PROSE 推理 overhead 侵蚀 wall-clock; (2) operator-set prior 已大幅缩小搜索空间, 全骨架边际增益不确定; (3) PROSE 骨架噪声在 1e-7 标准下未知. 若骨架错或系数 hint 噪声大, 双阶段可能输给 FEX restart / LLM mask. |
| Contribution type compliance | n.a. | topic 未声明 `preferred-contribution-types`? 否——topic 声明 `[method, theory, application]`. idea 声明 `method` ⊆ 该集合, 合规 (评分 10, 但本行按模板在无越界时记 n.a. 不计入 Overall 平均). |
| Overall Quality | 4/10 | well-formed 但前提存疑且零 pilot 信号. 可做成一篇方法/诊断论文, 但当前 top-venue 说服力完全依赖一个尚未证明、且与 sibling/prior 高度冗余的实证发现. |

## Contribution Drift (n >= 2 only; n=1 写 N/A)

- N/A (n=1, 首版无 drift 检查)

## Alternative Framing

当前 framing 不是最锋利. 两个方向:
1. (与 codex 一致) 收紧为 **"Which PROSE signal, if any, is a reliable search prior for exact symbolic PDE solving?"** —— 贡献仍是 method, 但重点从"造一个双阶段求解器"变成"可证伪的机制诊断", 并能明确区分 skeleton-only sibling.
2. 把唯一差异化的通道 C 单独拎出: **"一个粗粒度神经数值解的局部微分签名, 能否比解码器自己的符号骨架更好地预测 FEX 算子集?"** —— 这是 channel A/B 之外真正没人做过的问题. 但它会把 idea 推向 diagnostic 并与 sibling `fex-prose-approx-hierarchy` 在正则性-方法选择轴上部分重叠.

无论哪个 framing, 都改变不了"三条通道每条都已知可用、骨架通道与 sibling 几乎同题"这一底层冗余.

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 仅能声称: 在测试的 1D PROSE-PDE 分布、同 wall-clock 预算、对照 Bhatnagar operator-set / FEX-restart / LLM-mask 的前提下, PROSE 某条 (或某组合) 信号通道把 FEX 达标所需 evaluations 或 wall-clock 降低. **不能**声称双阶段对两者"严格 Pareto 占优", 也不能外推到 FEX 主战场 (高维 d=100) ——PROSE 仅在 1D 上训练. |
| NULL | 仅能声称: 在测试 PDE 族和 FEX 框架下, PROSE 的低误差数值预测/骨架不等价于可用的 exact-symbolic-search prior, FEX 只需 (或已能从) 粗粒度算子先验获得同等增益; 三通道消融的相图本身是贡献. |
| NEGATIVE | 仅能声称: 硬约束 PROSE 骨架或 noisy operator hint 会损害 FEX 搜索 (低于 operator-set prior 或 from-scratch+restart), 故 PROSE 信号需软化/fallback 才能安全注入. **不能**声称 PROSE 对 PDE 求解无价值——可能只是注入机制需改进. |

## Likelihood-Impact Matrix

- Priority: Low (3) = Likelihood: Low x Impact: Medium
- Numeric score for ideas.xml: 3
- Rationale:
  - Likelihood (Low): 做成 **top-venue 级**结果需要消融在三条**各自已被验证**的通道里翻出一个非平凡赢家, 且需要 PROSE 的近似骨架在 FEX 1e-7 标准下存活——而 2604.22208 (组内自己的工作) 提供了相反的直接证据. 加上零 pilot 信号、且被更成熟的 sibling `fex-llm-skeleton` (定标 5) 在每一轴严格压制. 做成一篇**可发表诊断论文**是可能的, 但顶会概率低. (注: 工程上"能跑通"≠ Likelihood; Likelihood 是"做成顶会结果的概率".)
  - Impact (Medium): 若完全做成 (干净的三通道相图 + 指出哪条 PROSE 信号是可靠 prior), 对混合求解器设计和 FEX–PROSE 桥梁有清楚发表价值, 但属于局部推进/诊断, 不会单独改变领域判断. FEX 本身 niche, 桥梁是一条设计线.
  - **Claude vs codex 分歧 (>= 1 level)**: codex 判 Likelihood=Medium → Score 5; 本 review 判 Likelihood=Low → Score 3. 分歧根源: codex 的 Medium 主要建立在"实现路径清楚 + 数据可用"上, 但这是**工程可行性**, 而 idea-reviewer 的 Likelihood 定义是"能做成 top-venue 级结果的可能性". 鉴于 (a) 零 pilot, (b) 被 5 分 sibling 严格 dominate, (c) 前提被组内证据反驳, 合并后取 Likelihood=Low 更可辩护. Impact 双方一致=Medium.

## Overall

- Priority: Low
- Score: 3
- Comments: 不越界, 但 novelty 被内部 sibling `fex-prose-skeleton-warm` (几乎同题) + LLM+FEX (2503.09986) + PROSE-PDE Skeleton (2404.12355) + PROSE+SymPy 系数精炼 (2409.11609) + FEX+TranNet (2604.22208) 五面夹击; 三条信息通道各自已知可用, 故"方法"贡献塌成工程拼接, 真正价值只能来自尚未证明的消融发现, 而该发现还与 sibling 高度冗余. Claude 与 codex 对 Likelihood 分歧 1 level (本 review 取 Low=3, codex 取 Medium=5; 见 Matrix 段). 建议: 本批次 9 个 fex-prose-* idea 里, 应优先推进已 3 轮精炼、有真实 pilot、叙事更锋利的 `fex-llm-skeleton`; 若仍要保留 PROSE 方向, 把本 idea 砍到只剩通道 C (数值解微分签名 → 算子集预测) 这个真正空白的子问题, 并先跑 pilot 拿信号. 另: novelty quick-check 引用的 NOWS (2511.02481) 未在本仓 wiki 收录、本轮也未能经 arxiv 核验, 作为对比引用风险可接受, 但若日后升格为 proposal 需核实该 ID 真实性.

</review>
