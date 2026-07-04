---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-prose-defect-correct/
---

- One-sentence summary: 用 PROSE/PI-MFM 产出的 1-3% 精度数值解 u0 作为粗解, 令 FEX 仅搜索符号修正项 delta_u = u - u0, 测试修正项是否落在比完整解更低复杂度的 S_k 子空间中, 从而以更小的 FEX 树达到机器精度.

- Hypothesis: PROSE 给出的数值解 u0 已经捕捉了解的主要结构 (主频率、主非线性模式), 残差 delta_u = u - u0 应该更简单、更光滑. 数值分析中的 "defect correction" 方法正是基于此直觉: 粗解 u0 的残差 r0 = L[u0] - f 是一个新的 RHS, 关于 u0 的线性化算子 L'[u0] 的逆作用于 r0 给出修正项. 如果 delta_u 的 S_k 表示比 u 本身更浅 (需要更少的算子节点), 那么 FEX 搜索 delta_u 的树空间指数级地小于搜索 u 的树空间. 具体预测: 若 u 需要 depth-d 的 S_k 树, delta_u 在 PROSE 精度 eps_PROSE ~ 1e-2 下需要 depth-(d-1) 或更浅的树 (因为 delta_u ~ eps_PROSE * (低阶修正项)). 如果这个 "depth reduction" 成立, FEX 搜索 delta_u 的步数应该少一个数量级.

- Expected outcome: 在 5+ PDE 族上 (从 PROSE-PDE 和 FEX 论文取并集), 比较: (A) FEX 从零搜索 u, (B) FEX 搜索 delta_u = u - u_PROSE. 成功信号: 方案 B 的达标树深比方案 A 浅 1 层以上, 且搜索步数减少 50% 以上, 最终精度 (u0 + delta_u) 达到 FEX 的机器精度. 失败信号: delta_u 的 S_k 复杂度不低于 u (说明 PROSE 的误差不是 "低阶可分离" 的, 而是分布式的), 此时 defect correction 不提供搜索空间缩减.

- Contribution type: method+theory

- Risk: MEDIUM

- Estimated effort:
  - Compute: 60-100 GPU-hours (PROSE 推理 ~5h, FEX 搜索 u ~40h, FEX 搜索 delta_u ~40h, 分析 ~10h)
  - Data: available (PROSE-PDE 公开数据集 + FEX 标准 PDE)
  - Implementation: 2-3 weeks

- Novelty quick-check: (1) NOWS (2511.02481) 用 neural operator warm-start FEM 迭代求解器, 但是数值对数值, 不涉及符号. (2) SymTorch (2602.21307) 从 PINN distill 符号表达式, 但不是 defect correction. (3) 经典数值分析中 defect correction (Stetter, Zadunaisky) 是成熟技术, 但从未与符号搜索结合. (4) 没有工作研究过 "数值解的修正项在 S_k 空间中是否更简单" 这个问题. 差异化: 将经典数值分析的 defect correction 思想移植到符号 PDE 求解, 回答 "neural solver 的误差是否有低复杂度符号结构" 这个新问题.

- Strongest objection: PROSE 的误差可能是全局分布式的 (不可分离为低阶项), 使 delta_u 的 S_k 复杂度不低于 u, 此时 defect correction 无用.

- Why we should do this: 这是一个数值分析 + 符号计算的桥梁, 不是简单的 "用 PROSE 辅助 FEX". 它问了一个根本性问题: "neural PDE solver 的误差有没有可解析的结构?" 如果有, 这为所有 neural-symbolic 混合方法提供了理论基础; 如果没有, 这说明 neural solver 的误差本质上是不可符号化的, 这本身也是重要的负面发现.

- Pilot:
  - Setup: 1D Poisson 3 case (A: single sin, B: two modes, C: sin+poly). 3 粗解类型: FEM 3-elem, FEM 5-elem, Fourier 1-mode. 固定表达式模板 depth 0/1/2, 多起点最小二乘拟合, MSE<1e-6 为达标.
  - Metric: delta_u 达标所需 depth < u* 达标所需 depth.
  - Result: Fourier 粗解有效: Case A u* depth=1 / delta_u depth=0 (BETTER); Case B u* depth=2 / delta_u depth=1 (BETTER); Case C tie (depth=2 both). FEM 粗解失败: 分段线性 u0 引入结点 kinks, delta_u 在光滑模板下永远不达标 (MSE floor ~1e-3).
  - Signal: CONDITIONAL POSITIVE. Defect correction 在粗解与目标同属光滑函数类时有效 (Fourier 去模式降 1 层深度), 在粗解引入异类结构时反效 (FEM kinks). 关键发现: 粗解必须与搜索语法同属函数类, 否则 defect correction 引入不可表达的结构. 这对 PROSE (输出光滑函数) + FEX (搜索光滑 S_k) 组合是好消息, 因为 PROSE 的数值解是光滑的.

<review date="2026-06-25" reviewer="claude">

## Object-Alignment Check (gating — 先于评分；本批次专杀机制)

本批次 FEX-PROSE 桥梁已连续三个 idea (`fex-prose-search-duality` / `fex-prose-entropy-bound` / `fex-prose-gibbs-equiv`, 均 score=1) 撞同一 fatal premise: 把 PROSE 当成"在**解空间** S_k 上放概率质量 / 输出**解骨架** token 分布"的对象, 而事实是 PROSE 的 Symbol Decoder 自回归生成的是**控制方程** (核验 `wiki/2404.12355.md`:21/37/43, `wiki/2309.16816.md`:50/376)。`gibbs-equiv` review (本仓 `ideas/fex-prose-gibbs-equiv.v1.md`:102) 已立硬规则: 后续推该桥梁**必须先确认 PROSE 输出空间与 FEX 对象空间是否对齐**。

**本 idea 通过该 gate, 是本批次第一个不踩对象陷阱的桥梁 idea。** 区别是决定性的, 不可懒惰套用 sibling kill:
- 被杀 sibling 用的是 PROSE 的 **Symbol Decoder** (输出方程 token 分布) ⇒ 锚在不存在/错空间的对象上。
- 本 idea 用的是 PROSE 的 **Data Decoder** 输出的**数值解场 u0(x,t)** (核验 `wiki/2404.12355.md`:20 "Data Decoder ... 独立 query time points 作为评估点", :44 Skeleton 模式数据相对误差 **1.06%**, R²=0.998; `wiki/2309.16816.md`:50 "数据解码器直接预测 ... 比用学习到的方程直接求解更准确 4.59% vs 14.69%")。这个 u0 **真实存在且正是 ~1% 精度的光滑数值解**, 恰好是 idea 需要的粗解。
- 故 idea 的对象 (`delta_u = u_exact - u0`, u0 = neural 数值解场) 在**同一函数空间** (PDE 解空间) 内有良定义, `delta_u` 的 S_k 复杂度问题是 well-posed 的。**sibling 的 kill 理由不迁移到本 idea。** 这是必须如实记录的关键判别。

**结论: 通过 object gate, 进入正常评分。** 但 idea **自己的 novelty quick-check 漏掉了真正的近邻 prior** (见下), 且 load-bearing hypothesis 已被 prior 在印 + 被主流理论 (spectral bias) 反向预测 + 被 idea 自己的 pilot 半证伪。

## Novelty

- Score: 3/10
- Closest prior work (idea 的 quick-check 全部漏掉, 三篇各击穿一层):
  1. **VIPER-R1 "Symbolic Residual Realignment (SR²)"** (2508.17380, Liu et al., 2025-08; web 核验标题/作者/日期属实) —— **几乎逐字 scoop 了 headline 机制**: 先给高置信 symbolic ansatz, 再对 **residual** 调外部 symbolic regression 工具, 最终 = ansatz + SR(residual), 明确以"physicist's perturbation analysis"为动机。这正是本 idea 的 "搜 delta_u 而非 u" + "残差落在更小搜索空间"。差异仅在: SR² 的 base 是 **symbolic VLM 预测** (非数值/neural 解场), 域是 **ODE/运动学公式发现** (1D trajectory, 非空间 PDE 场), search engine 非 FEX。
  2. **NNPT "Neural Network Perturbation Theory: Learning Residual Corrections from Exact Solutions"** (2512.01558, Chen/Shen/Fain/Nussinov, 2025-12, 2026-06 revised; web 核验属实) —— **idea 的 load-bearing Hypothesis 已在印**: "analytically subtracting known exact solutions" 后 "predicts residual perturbations", 并明确 argue 修正项**更低复杂度/更少参数** ("leaving only statistically smooth corrections requiring fewer parameters", "47% reduction from peak" 网络容量), 跨 PDE/量子/多体泛化。差异: NNPT 的 baseline 是**解析微扰解** (非 ~1% neural 解), corrector 是**神经网络**而非符号回归——但"correction 比 full solution 简单"这个**核心定性主张已被它公开 claim**。
  3. **MC² "Monte Carlo Correction for Fast Elliptic PDE Solving"** (2605.09288, 2026-05) —— 同 meta-move: 廉价 Walk-on-Spheres 解的残差 "carries structure ... consistent across PDE instances and recoverable by a learned operator", 量化为 44.10 dB vs 噪声去噪器 <27 dB 的 17 dB "information content lower bound"。差异: classical MC solver, learned operator 而非符号, dB 而非符号复杂度。
- Key differentiator (idea 还剩的真实白区, 但很窄): (a) base 是**空间 PDE 场的 neural/spectral 数值解** (非 symbolic ansatz、非 analytic 微扰、非 1D trajectory); (b) corrector 是 **FEX = RL 组合树搜索**, 故"搜 delta_u"是**离散算子树空间缩减**这一更尖锐、更可证伪的 efficiency claim (SR²/GWAgent 报的只是 SR-engine 加速); (c) 没人用**符号树复杂度**这把尺子量过 neural PDE solver 误差。但这三点都是 "把已发表的 residual-first 概念 (SR²) instantiate 到 (neural base + FEX) 新组合", 接近 idea-quality.md 判的 **apply-X-to-Y**——X = SR² 的 residual-first SR (2025-08 已发表), Y = FEX/PROSE。idea 自称的 novelty ("数值分析 defect correction 首次与符号搜索结合 / 没人研究过数值解修正项是否更简单") 在 VIPER-R1 与 NNPT 面前**不成立**: 符号残差搜索 (SR²) 与"修正项更低复杂度"(NNPT) 都已在印。

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 4/10 | (a) **核心 Hypothesis 与主流理论冲突且 idea 未正面处理**: spectral bias / F-Principle (Rahaman 2019; SpecBoost 2404.07200 "残差能量集中在**高频**", FNO low-freq bias) 预测——neural solver 已抓住主导低频模式, 残差恰是**剩下的高频/振荡难部分**, 即 delta_u 可能比 u **更难**符号化, 而非更简单。idea 的 Hypothesis (delta_u ~ eps_PROSE × 低阶修正, depth 降 1 层) 是**逆着既有理论下注**, 但 idea 全文没引用也没回应 spectral bias——这是最大的逻辑缺口。(b) **idea 自己的 pilot 已半证伪乐观假设**: 9 个 (case×coarse) 组合里仅 **2 个 BETTER** (Fourier Case A/B), 1 个 TIE (Case C mixed), **6 个 WORSE** (FEM 全部, delta_u 因 P1 分段线性 kink 永不达标, MSE floor ~1e-3)。即"depth 降 1 层"只在"粗解与目标同属光滑谱基"的窄regime成立, 且一旦有多项式 tail (Case C) 就退化为 tie。(c) pilot 的"好消息"推断有跳跃: "PROSE 输出光滑 ⇒ 对 FEX-PROSE 是好消息" 忽略了**光滑 ≠ 低 S_k 复杂度**——FEM 失败证明 base 必须同属搜索语法函数类, 但 PROSE 的 Data Decoder 输出是 **8 层 cross-attention 的 DeepONet 式 branch-trunk 数值场** (`wiki/2404.12355.md`:20), 它是光滑的但**未必落在 FEX 的 S_k = {sin/exp/poly 有限组合} 内**; 若 PROSE 误差含 attention 基函数残留的非 S_k 结构, 就是 FEM-kink 失败模式的连续版。 |
| Missing evidence signals | 4/10 | pilot **诚实**地 surface 了双向结果 (设计上就要暴露 FEM 失败), 这是加分。但: (i) pilot 用 **fixed least-squares 模板拟合**, 不是真 FEX RL 搜索——"depth 代理"测的是"某深度下是否存在好拟合", 不是"FEX controller 实际搜到 delta_u 的迭代数"; idea 的 headline ("搜索步数减少 50%+") 在 pilot 里**根本没测**。(ii) pilot 用人造 Fourier-1mode / FEM 粗解, **完全没有 PROSE**——而 idea 的真实主张依赖 PROSE 误差的具体谱结构, 这从未被实例化。(iii) 缺决定性对照: 没把 **NNPT (2512.01558)** 当正面 baseline (它已 claim residual 更低复杂度), 也没把 **VIPER-R1 SR² (2508.17380)** 当 scoop 对照——不引则 reviewer 一眼判 "residual-first SR 已发表"。 |
| Narrative | 5/10 | "neural solver 误差有没有可解析结构? 有则为所有 neural-symbolic 混合奠基, 没有则是重要负面发现" 这个 reframe 抓人且 falsifiable, 是 idea 最强的部分。但 (i) "首次桥接数值分析 defect correction 与符号搜索" 的卖点被 SR²/NNPT 击穿; (ii) 负面结果的"价值"被 spectral bias 文献**预期** (领域已半信残差是难部分), 故 NEGATIVE 不 surprising; (iii) 正面结果在理论上 disfavored。叙事从"开放问题"退到"已被半回答的问题加一把符号尺子"。 |
| Venue contribution | 3/10 | 对 AAAI/JMLR/Neural Networks/ICLR: method deliverable (FEX-as-defect-corrector for neural base) 是 SR² residual-first 机制换 search engine + 换域, theory deliverable ("delta_u 的 depth-reduction") 已被 NNPT 定性 claim 且被 spectral bias 反向预测。剩下可发表的硬核只有 (c) "符号树复杂度尺子量 neural 误差"这一**测量贡献**, 但这是窄 gap, 单独撑不起 top venue method+theory 双 claim。 |
| Testability (cheapest falsifier) | 7/10 | falsifier 写得**好且具体**: "delta_u 达标 depth < u 达标 depth & 搜索步数 −50%+"。pilot 已经是 cheapest falsifier 的雏形且**已给出偏负的答案** (2/9 BETTER)。这是 idea 的相对强项——它**可被快速杀死**, 且 pilot 已经在杀它。给 7 分因为 falsifier 清晰可执行。 |
| Outcome realism | 4/10 | 60–100 GPU-h / 2–3 周预算合理。但 **headline 量级 claim ("搜索步数少一个数量级") 与 pilot 证据矛盾**: pilot 最好情形也只降 1 层 depth 且仅 2/9 组合 BETTER, "指数级缩小搜索空间"在 mixed/非谱基 PDE 上无证据。真实 PROSE 误差大概率含非 S_k 结构 (attention 基函数残留), 落入 FEM-kink 连续版失败模式的风险高。最可能的真实结果是 **CONDITIONAL/NULL** (只在纯谱解 PDE 上小幅 better), 不改变领域判断。 |
| Contribution type compliance | 10/10 | idea types {method, theory} ⊆ topic `0616-fex.md` preferred {method, theory, application}: yes (不计入平均)。 |
| Overall Quality | 4/10 | 六要素齐全, pilot 诚实且 falsifiable 是真加分 (区别于被杀 sibling 的纯合成 pilot); 但三处 load-bearing 环节受损: Hypothesis 与 spectral bias 冲突且未回应、被 NNPT 定性在印 scoop、被自己 pilot 半证伪 (2/9 BETTER); headline 机制被 VIPER-R1 SR² 发表 scoop; "指数搜索缩减"量级无 pilot 支撑。 |

## Contribution Drift (n >= 2 only; n=1 写 N/A)

N/A (n=1, 首版无 drift 检查)

## Alternative Framing

idea **没被对象错误击穿** (区别于 sibling), 故有可救路径, 但都需要重新定 framing 以避开 SR²/NNPT 并正面拥抱 spectral bias:

1. **从"method (加速 FEX)" 转为"measurement / diagnostic (符号复杂度尺子)"**: 放弃"指数搜索缩减"的强 efficiency claim (被 pilot 证据反对 + 被 SR² scoop), 改为系统性**实证刻画**: 对 N 个 PDE 族 × M 个 neural/spectral solver (FNO / DeepONet / PROSE / Fourier-truncation), 测 delta_u 的 **FEX 最小达标树深 vs u 的树深**, 并**与残差谱 (高频能量占比) 关联**——回答"何时 neural 误差是低 S_k 复杂度的, 何时不是, 由什么 (谱集中度/函数类匹配) 决定"。这把 idea 从"被 disfavored 的正面 method"变成"领域缺失的 diagnostic 测量", contribution type 退到纯 application/empirical 但**可发表且诚实** (符号复杂度尺子是真白区, 见 novelty subagent angle 5)。必须正面引 VIPER-R1/NNPT/SpecBoost 并 differentiate。

2. **若坚持 method**: 唯一非平凡 delta 是把 pilot 已发现的失效条件 ("base 必须同属搜索语法函数类") 升级为**可计算的 accept/reject 判据**——即"给定 neural solver 输出, 预判 delta_u 是否落在 FEX 的 S_k 内、defect correction 是否会加速"。但这与 sibling `fex-prose-skel-condition` (score 待定, 用骨架条件数 kappa(T) 做 PROSE 输出 accept/reject) 的**判据型 framing 高度重叠**, 需明确区分 (skel-condition 判"系数可优化性", 本路径判"残差可符号化性"), 否则 contribution drift 风险。

3. **不可救的部分**: "首次结合数值 defect correction 与符号搜索" (SR² 已做)、"修正项更简单" 作为新发现 (NNPT 已 claim)、"指数搜索缩减" (pilot 反对) ——这三个卖点必须从 headline 删除。

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 至多可声称"在**粗解与目标同属光滑谱基**的 PDE 子类上, 对 neural/spectral 数值解做 FEX 符号 defect correction 可降低达标树深 1 层"; **不能**声称首创 residual-first 符号搜索 (VIPER-R1 SR² 2508.17380 已发表), **不能**声称首次发现"修正项更低复杂度" (NNPT 2512.01558 已 claim), **不能**声称"指数级 / 一个数量级"搜索缩减 (pilot 仅 2/9 BETTER 且最多降 1 层)。 |
| NULL | 可声称"在含多项式/非谱 tail 或函数类不匹配的 PDE 上, delta_u 的 S_k 复杂度不低于 u, defect correction 不提供搜索缩减"——这已是 pilot Case C (tie) + FEM (worse) 的实测结论, 是**最可能的真实结果**, 且与 spectral bias 预测一致。 |
| NEGATIVE | 可诚实声称"neural PDE solver 误差在 S_k 意义下**不可低复杂度压缩** (谱上是高频分布式的)"——但此结论已被 spectral bias / F-Principle **预期**, 故 surprise 值低; 且需说明这与 NNPT 的"解析微扰 baseline 下修正更简单"为何不矛盾 (函数类匹配 vs 不匹配)。 |

## Likelihood-Impact Matrix

- Priority: Weak Refine / Borderline (3) = Likelihood: Low x Impact: Low-Medium
- Numeric score for ideas.xml: 3
- Rationale:
  - Likelihood (Low): 做成 top-venue **method+theory** 双 claim 概率低——headline 机制被 VIPER-R1 SR² (2508.17380, 2025-08) 发表 scoop, load-bearing "修正更简单"被 NNPT (2512.01558, 2025-12) 在印 claim, "指数搜索缩减"被 idea 自己 pilot 半证伪 (2/9 BETTER, FEM 全 WORSE), 且核心 Hypothesis 逆 spectral bias 主流理论而 idea 未回应。**但显著高于被杀 sibling 的 Very Low**: 因为 (i) 通过 object gate (用真实存在的 PROSE Data Decoder 数值解, 非错空间对象), (ii) pilot 诚实可证伪且已在做 falsification, (iii) "符号复杂度尺子量 neural 误差"是真白区 (novelty subagent angle 5 确认无直接命中)。故不是 kill-class, 而是 borderline。
  - Impact (Low-Medium): 若退到 diagnostic framing 做成, "何时 neural solver 误差可符号化"对 neural-symbolic 混合方法有中等参考价值; 但当前 method framing 下增量小 (SR² 换 engine + 换域)。
- **本轮为单评 (Claude)** + 2 个 novelty subagent (web 核验 VIPER-R1/NNPT/MC²/SpecBoost 均属实)。核心判别 (本 idea **不踩** sibling 对象陷阱) 可由本仓 `wiki/2404.12355.md`:20/44 (Data Decoder 输出 ~1% 数值解) 直接核验, 非判断分歧。三条减分 (SR² scoop / NNPT 在印 / pilot 半证伪 + spectral bias 冲突) 均有 web-核验的外部 prior 或仓内 pilot 证据支撑。

## Overall

- Priority: Weak Refine (borderline; 若 refine 必须改 framing, 否则 Archive)
- Score: 3
- Comments: **Borderline (3), 本批次 FEX-PROSE 桥梁中第一个不踩对象陷阱的 idea, 但 headline 被双重 scoop + 自身 pilot 半证伪。**

  **关键正面判别 (必须如实记录, 不可懒套 sibling kill)**: 被杀的三个 sibling (`search-duality`/`entropy-bound`/`gibbs-equiv`) 都锚在 PROSE 的 **Symbol Decoder** (输出方程 token 分布, 错空间) 上, 故 ill-defined。本 idea 锚在 PROSE 的 **Data Decoder 数值解场 u0** (核验 `wiki/2404.12355.md`:20/44, Skeleton 模式 1.06% L²)——这个 u0 **真实存在且是光滑数值解**, `delta_u = u - u0` 在 PDE 解空间内良定义。**sibling 的 fatal-premise kill 理由不迁移到本 idea**; object gate 通过。

  **但三条独立减分使其止步 borderline**: (1) **headline 机制被发表 scoop** —— VIPER-R1 "Symbolic Residual Realignment" (2508.17380, 2025-08, web 核验) 已做"给 ansatz → 对 **residual** 跑 symbolic regression → 加回"并以"perturbation analysis / 残差是更小搜索空间"为动机, 与本 idea "搜 delta_u 而非 u" 几乎同构 (差异仅 base 类型/域/engine)。(2) **load-bearing Hypothesis 已在印 + 逆主流理论** —— NNPT (2512.01558, 2025-12, web 核验) 已 claim "subtract exact solution, 修正项更 smooth / 更少参数 (47% 容量降)"; 而 spectral bias / F-Principle / SpecBoost (2404.07200) 主流结果反向预测 neural 残差是**高频难部分**, delta_u 可能比 u **更难**符号化——idea 全文未引未回应此冲突。(3) **idea 自己 pilot 半证伪乐观假设** —— 9 组合仅 **2 个 BETTER** (Fourier 谱基), 1 TIE (mixed), **6 个 WORSE** (FEM kink 全失败); "depth 降 1 层"只在窄谱基 regime 成立, "指数/一个数量级搜索缩减"无任何 pilot 支撑。另: pilot 用 fixed-template 拟合非真 FEX 搜索, 且**不含 PROSE**, headline 的 "PROSE 误差是否低 S_k 复杂度" 从未实例化 (真实 PROSE Data Decoder 是 attention branch-trunk 场, 光滑但未必落在 S_k 内, 是 FEM-kink 失败的连续版风险)。

  **建议**: 不直接 kill (区别于 sibling, 因 object 正确 + pilot 诚实可证伪 + 符号复杂度尺子是真白区), 但**当前 method+theory framing 不可送 coding**。若 refine, 唯一可救路径是退到 **diagnostic/measurement framing** (Alternative Framing #1): 放弃"加速 FEX / 指数缩减"强 claim, 改做"系统实证 + 谱关联: 何时 neural/spectral solver 误差是低 S_k 复杂度的, 由函数类匹配/谱集中度决定", 并**强制正面引用 VIPER-R1 SR² / NNPT / SpecBoost 做 differentiation**。若不改 framing, 则与 sibling 一同 Archive。注意 Alternative #2 (可计算 accept/reject 判据) 与 sibling `fex-prose-skel-condition` framing 重叠, 需防 contribution drift。

</review>
