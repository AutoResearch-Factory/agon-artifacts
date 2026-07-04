---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-prose-skel-condition/
---

- One-sentence summary: 定义骨架条件数 kappa(T) = sigma_max / sigma_min (骨架 T 在 collocation 点处的变量投影残差 Jacobian 的条件数), 证明 kappa(T) 决定了 FEX 的 Adam+BFGS 在正确骨架上的收敛速率, 并用它作为 PROSE 骨架输出的可计算 accept/reject 判据.

- Hypothesis: FEX 的两步优化中, "好骨架但系数找不到" (Good-Structure-Bad-Score, GSBS) 是 DSO 框架 (2505.10762) 已识别但未解决的失效模式. 假设: GSBS 现象可以用骨架条件数 kappa(T) 完全预测 -- 当 kappa(T) > kappa_critical 时, Adam+BFGS 的收敛步数指数增长, GSBS 必然发生; 当 kappa(T) < kappa_critical 时, 收敛步数与 kappa 成线性关系. 这意味着: (1) 对 FEX: 在 RL 搜索中优先探索低 kappa 骨架可以避免 GSBS; (2) 对 PROSE: 骨架条件数可以作为符号解码质量的新评估指标, 比表达式正确率更有信息量; (3) 对两者的桥梁: kappa 是一个可以从 PROSE 骨架计算、用于 FEX 搜索决策的通用量.

- Expected outcome: (1) 定理: 对椭圆 PDE 的 sum-of-products 骨架, Adam 步数 T_converge <= C * kappa(T)^2 * log(1/eps). (2) 实验: 在 5+ PDE 族上生成 1000+ 骨架 (正确骨架+随机变异骨架), 计算 kappa(T) 和 FEX 实际收敛步数, 验证 T_converge vs kappa 的幂律关系. (3) 应用: 将 kappa(T) 作为 PROSE 骨架的 accept/reject 判据, 在 FEX 搜索循环中跳过 kappa > kappa_critical 的骨架, 测量搜索效率提升. 成功信号: kappa(T) 与 FEX 收敛步数的 Spearman rho > 0.7. 失败信号: kappa(T) 与收敛步数无显著相关 (rho < 0.3), 说明 GSBS 的原因不是条件数.

- Contribution type: method+theory

- Risk: MEDIUM

- Estimated effort:
  - Compute: 50-80 GPU-hours (骨架生成 ~5h, kappa 计算 ~5h, FEX 搜索 ~40h, 对照 ~20h)
  - Data: available (FEX 标准 PDE benchmark)
  - Implementation: 2-3 weeks

- Novelty quick-check: (1) DSO (2505.10762) 识别了 GSBS 现象但未给出预测性指标. (2) VarPro (variable projection) 文献分析过非线性最小二乘的条件数, 但未应用于 PDE 残差或符号搜索. (3) PROSE+SymPy (2409.11609) 用 SMC 粒子滤波精炼系数, 但没有条件数作为搜索引导. (4) SAGE-Fit (2605.23272) 改善 SR 的系数优化, 但从参数化拟合角度, 不是条件数引导. 差异化: 首次用条件数理论连接 "骨架质量" 和 "系数可优化性", 同时适用于 FEX 和 PROSE.

- Strongest objection: 条件数只度量线性化残差的局部性质, 对非线性骨架 (含频率参数) 可能在 BFGS 的搜索轨迹上变化剧烈, 使得初始 kappa 不能预测最终收敛.

- Why we should do this: 这给 FEX-PROSE 桥梁提供了一个可计算的"接口": PROSE 输出骨架 + kappa, FEX 根据 kappa 决定是否接受. 这不是 "apply X to Y", 而是定义了两个方法之间信息传递的质量度量. 同时, kappa 理论本身对 FEX 有独立价值 (解释 GSBS), 对 PROSE 也有独立价值 (新的骨架评估指标), 所以两个社区都会 care.

- Pilot:
  - Setup: 1D Poisson, 30 个骨架 (5 correct sin, 5 over-param sin+cos, 5 Fourier K=1-5, 5 polynomial d=2-6, 5 exp-type, 5 mixed sin+poly). 100 点梯形积分. L-BFGS-B 优化系数, 记录 n_feval. 有限差分 Jacobian SVD 算 kappa.
  - Metric: Spearman rho(kappa, n_feval) > 0.5.
  - Result: Overall Spearman rho=0.534 (p=0.002). Well-posed subset (smin>0) rho=0.689 (p=0.004). Per-category medians: correct kappa=2.2/nfev=48, overparam kappa=2e10/nfev=85, fourier kappa=9.0/nfev=48, poly kappa=1e12/nfev=130, exp kappa=1e12/nfev=120, mixed kappa=11.6/nfev=72. kappa 清晰分离正确(~1-10) vs 错误(~1e12) 骨架.
  - Signal: POSITIVE. kappa 与收敛步数显著相关 (p=0.002), 且能区分正确/错误骨架. 在 well-posed 骨架内, rho=0.689 更强, 说明 kappa 在正确结构类内也能区分 "好优化" vs "难优化" 的骨架.

<review date="2026-06-25">

## TL;DR

首次 review (n=1). 这是 0616-fex 在 2026-06-24/25 批量产出的 fex-prose-* 家族成员, 是其中 condition-number 路线的"承载版本"——sibling fex-prose-struct-coeff-sep 的 review 明确把"可救理论核"委托给了本 idea. 叙事干净, pilot 真的跑了 (区别于 embed-condition 的零 pilot), 但 **headline 的三块支柱逐条被已发表文献预占, 其中一篇 (Kronberger 2209.00942) 是本 idea 完全没引、几乎逐字的先例**, 而支撑 headline 的 pilot 信号经审查是 **representability + 参数个数的混淆 artifact, 不是 conditioning 律**. Score: Low (3), 不建议作为独立"新方法/新定理" idea 推进; 若要续命必须 reframe 成"可标定可迁移 kappa 阈值作为搜索剪枝 gate"并与 SAGE-Fit 正面 head-to-head 对照 (见 Alternative Framing).

## Novelty

- Score: 2/10
- 三块支柱逐条核验 (独立 novelty-check subagent 与本 reviewer 结论一致):

  1. **Claim 1a — "定理 T_converge <= C kappa^2 log(1/eps)": LOW (textbook).** "梯度法/Gauss-Newton 在 kappa-conditioned 最小二乘上的迭代复杂度 = O(kappa·log(1/eps)) 或 O(kappa^2·log(1/eps))" 是凸优化标准结论 (Nesterov / Boyd-Vandenberghe / Nocedal-Wright). 把它换写到"骨架 VarPro 残差 Jacobian"上不增加任何新数学. 这与 sibling skeleton-convexity / struct-coeff-sep 被判 3/10 的同一病因 (VarPro/separable-NLS + 教科书结论换皮) 同构.

  2. **Claim 1b — "kappa 经验上预测 SR 系数拟合难度": LOW. 直接预占者 = Kronberger, "Local Optimization Often is Ill-conditioned in Genetic Programming for Symbolic Regression", arXiv:2209.00942 (EuroGP 2022) —— 本 idea 完全未引.** 该文已经 (a) 用 SVD 算 SR 常数拟合 Jacobian 的条件数, (b) 把高 kappa 与 NLS 收敛变慢直接挂钩, (c) 明确建议"用 SVD 检查解的 conditioning"作为筛查信号. 后续 de França & Kronberger 2023 / Kronberger & de França 2025 (J. Symbolic Computation) 进一步显示"去过参数化 (降 kappa) 改善收敛速度". 本 idea 的 Novelty quick-check 第 (2) 条写 "VarPro 文献分析过非线性最小二乘的条件数, 但未应用于 PDE 残差或符号搜索"——**这句话是事实错误**: 2209.00942 正是把条件数应用于符号搜索的系数拟合. 唯一未被逐字占据的薄片是 kappa-vs-iteration 的 power-law/Spearman 散点这一具体"打包方式", 机制和方向已完全已知.

  3. **Claim 2 — "kappa 作 accept/reject gate 跳过病态骨架": LOW–MEDIUM.** Kronberger 2209.00942 已建议把 conditioning 当筛查信号、把降条件数当设计目标; 把它接成"优化前硬阈值 gate 省搜索预算"只是其上的小工程增量.

  4. **GSBS 归因错误 + Claim 2 被 SAGE-Fit 预占.** idea 反复把 "Good-Structure-Bad-Score (GSBS)" 归给 "DSO (2505.10762) 已识别但未解决". 核验 wiki: GSBS / "Good Structure, Bad Score" 是 **SAGE-Fit (When Good Equations Get Bad Scores, Wang et al., AAAI 2026, arXiv:2605.23272)** 提出的, DSO 2505.10762 只在"差异化方向"里一句话提及 (wiki:127); 而且 SAGE-Fit 的**整篇贡献就是解决 GSBS**, 并非"未解决". 更致命: SAGE-Fit **已经把 GSBS 诊断为 VarPro 式 ill-conditioning** ("确定性维度坍缩, 消除线性-非线性耦合导致的 ill-conditioning", wiki:26), 并用 **Tree-Directed Variable Projection + Projected Gauss-Newton/TRF** 修复 (wiki:21-43). 本 idea 的核心论点"GSBS 由 VarPro 残差 Jacobian 的 kappa 预测/解释"基本是 SAGE-Fit 自己诊断的重述, 而 SAGE-Fit 恰是本 idea 的目标会议 (AAAI). idea 的 quick-check 第 (4) 条把 SAGE-Fit 描述成"从参数化拟合角度, 不是条件数引导"——也是对 prior 的弱化误读.

  5. **Claim 3 — "kappa 作 PROSE→FEX 桥梁的接口量": MEDIUM, 但是 plumbing 不是 principle.** 没有论文用 conditioning 量耦合 PROSE-PDE 输出与 FEX 接受判据, 这个系统配对未被占据. 但 FEX 本就用 minimized loss S(e)=(1+L)^{-1} 给候选打分, 加一个 conditioning 滤波是 incremental 集成, 非新原理. 且这条还叠加下面的对象错误.

- **PROSE 对象混淆 (家族杀手, 本 idea 部分中招).** Claim 3 / headline 把 kappa 写成"PROSE 骨架输出的可计算 accept/reject 判据"、"可以从 PROSE 骨架计算". 但 PROSE-PDE 的 Symbol Decoder 输出的是**控制方程**的 Polish notation (wiki/2404.12355.md:21,31), Skeleton 模式指"方程结构已知、系数未知", 它**根本不做系数优化**——数值解由 Data Decoder 直接预测 (wiki:20,44). FEX 求解时**已经持有控制方程**, 它搜索的是**解**的表达式树. 因此"在 PROSE 骨架上建残差 Jacobian 算 kappa 来 accept/reject" 这个动作的对象是错位的: PROSE 不输出可供 FEX 系数优化的**解骨架**. 这正是 killed 掉 embed-condition / entropy-bound / search-duality / skeleton-convexity 四个 sibling 的同一 conflation. (kappa 理论本身可定义在 FEX 自己的候选解树上, 那部分不犯此错; 但一旦把 PROSE 写进 accept/reject 回路, Claim 3 就塌.)

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 3/10 | (1) **pilot 的 POSITIVE 信号是 artifact, 非 conditioning 律** (详见 Missing evidence). (2) 定理把"初始 kappa"与"收敛步数"挂钩, 但 idea 自己的 Strongest objection 已承认非线性 (频率参数) 骨架沿 BFGS 轨迹 kappa 剧变, 初始 kappa 不能预测最终收敛——而 sibling skeleton-convexity 的 pilot **已实测证实**这点 (a_sin 含非线性频率, cvx@2.0=20%, 全局非凸). 即"kappa 在最优点 (pilot 测的) 与 kappa 沿轨迹 (定理需要的) 是两回事", 这个 gap 没弥合. (3) kappa^2 还是 kappa 的指数本身就 method-dependent (一阶法 kappa^2, Gauss-Newton 局部 kappa), idea 写死 kappa^2 缺依据. |
| Missing evidence signals | 3/10 | **pilot 的 rho 被两个混淆量驱动, 经不起审查**: (a) `condition_number()` 在 smin→0 时把 kappa 钳到 smax/(1e-12·smax)=1e12 (pilot_condition.py:265). poly/exp 全部 5×2 个骨架的 kappa 都正好是 1e12——这不是测出来的条件数, 是**硬编码天花板**, 因为这些骨架的残差 Jacobian 有真零方向 (smin=0): 它们**根本无法表示 sin** (final_loss 9.2/22.6/48.7, 见 results.txt). 所以"kappa 区分正确 vs 错误骨架" 大半是 **representability 指标 (smin=0) 在 fire, 不是 conditioning 预测优化难度**. 把这些钳位类别计入 overall rho=0.534 是污染. (b) 诚实子集是 well-posed (rho=0.689, n=15), 但它**跨类别**混 correct/fourier-K/mixed, 而 n_feval 随**参数个数**涨 (fourier_K5: 5 params→378 evals), kappa 也随 K 涨 (K^2). 即 kappa 与 n_feval 同被潜变量"骨架规模/参数个数"驱动——这是 near-tautological 混淆, 与 skeleton-convexity review 标记的同型. (c) **方向错**: pilot 在**最优点**事后测 kappa, 但 Claim 1 定理 (从起点预测收敛) 和 Claim 2 (优化前 accept/reject) 需要**优化前/独立于优化**的 kappa. pilot 完全没测预测/因果方向. (d) 缺最关键对照: 与 SAGE-Fit 的 VarPro+refit (just-refit-everything) head-to-head——没有它无法证明"kappa-gate"比"全部重拟合"更省. |
| Narrative | 6/10 | "kappa 作 FEX-PROSE 桥梁的可计算接口" 叙事干净有吸引力, landscape §23/§504 有铺垫. 比 embed-condition 实在 (真跑了 pilot). 但"双重独立价值 (对 FEX 解释 GSBS / 对 PROSE 新评估指标)"两条在 prior 下都站不住: 解释 GSBS 被 SAGE-Fit 占, kappa 作骨架评估被 Kronberger 占. |
| Venue contribution | 2/10 | 以 AAAI/JMLR/Neural Networks/ICLR 标准: kappa-predicts-convergence 撞 Kronberger 2209.00942 + 教科书; kappa-diagnoses-GSBS + VarPro-cure 撞 SAGE-Fit 2605.23272 (AAAI 2026, 同会议). 即便 positive 也读作"已知机制的 PDE-residual 实例化 + 一个未发表过的 Spearman 散点", 审稿人判已知结论重述. 唯一防御白区 (可标定可迁移 kappa 阈值 + 对 SAGE-Fit/refit-all 的系统对照) 窄且 incremental. |
| Testability | 6/10 | 有明确 falsifier (rho<0.3 则 GSBS 非条件数). pilot 可跑且便宜. 但当前 falsifier 测的是**错的方向** (最优点 kappa vs 难度), 且阈值用单点 rho 而非 effect-size+CI; 真正该测的便宜 gate 是"优化前 kappa 能否预测该骨架最终能否被拟合到目标精度", 且必须扣除参数个数这个混淆 (partial correlation / 同 np 分层). |
| Outcome realism | 4/10 | "5+ PDE 族 1000+ 骨架 + FEX 实际收敛步数, 验证幂律" 在 50-80 GPU-h 内可行 (区别于 embed-condition 依赖不存在的 PROSE checkpoint——本 idea 的 kappa 计算不依赖 PROSE 权重, 这点更扎实). 但"幂律"预期偏乐观: pilot 已显示 kappa 在 well-posed 内主要随参数个数变, 扣除后 partial 相关可能远弱; 且 Claim 3 一旦真要"从 PROSE 骨架算 kappa", 又撞 PROSE 不输出解骨架的对象问题. |
| Contribution type compliance | n.a. | idea type = method+theory; topic 0616-fex preferred-contribution-types = [method, theory, application], 合规. (此维度不计入 Overall 平均.) |
| Overall Quality | 3.5/10 | 比同家族零-pilot 的 embed-condition 实在, 但 headline 的实证支柱 (pilot 信号) 是 representability+参数个数 artifact, 理论支柱是教科书+VarPro 换皮, 而 prior 端被 Kronberger 2209.00942 (未引、近逐字) + SAGE-Fit 2605.23272 (GSBS 原主、已用 VarPro 解决) 双重预占. |

## Contribution Drift (n >= 2)

n = 1 (仅 v1), 不适用. 首次 review.

## Sibling / 家族重叠分析

本 idea 在 fex-prose-* 家族中是 **condition-number 路线**, 与以下直接竞争/重叠:

- **fex-prose-struct-coeff-sep (score 1, archived)**: 其 review 把"可救理论核"**委托**给本 idea ("可救理论核已被 skel-condition 承载"). 但本 review 证明该理论核 (VarPro/条件数) 本身被 Kronberger+SAGE-Fit 预占——所以那次委托的落点也是空的. 两 idea 共享同一 VarPro 病因.
- **fex-prose-skeleton-convexity (score 3)**: 共用同一 1D Poisson + 同型骨架的 pilot 框架; convexity 测 Hessian 谱, 本 idea 测残差 Jacobian 条件数——数学上紧密相关 (Hessian≈J^T J 的谱与 J 的奇异值平方对应). convexity 的 pilot **已实测**非线性骨架全局非凸 (cvx@2.0=20%), 直接削弱本 idea"初始 kappa 预测收敛"的定理可行性. 二者应合并讨论而非各自独立.
- **fex-prose-entropy-bound / embed-condition / search-duality (score 1/3/1)**: 均因 PROSE 对象混淆被压低; 本 idea 的 Claim 3 (kappa 从 PROSE 骨架算) 部分中同一招.

## Claims Discipline

idea 当前无 claims matrix. 若推进 (或并入 SAGE-Fit-style 对照研究), 应限定:

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 在限定 PDE 族 / FEX tree depth / 固定参数个数分层下, **优化前**计算的骨架残差 Jacobian 条件数 kappa 在**扣除参数个数混淆后**仍显著预测系数拟合达标所需迭代; 且 kappa-阈值 gate 在同等最终精度下比"全部重拟合 (SAGE-Fit/lm_trf_multi)"更省总搜索预算. **不能**声称"首次用条件数连接骨架质量与可优化性" (Kronberger 2209.00942 已做), 不能声称"解释 GSBS" 为新 (SAGE-Fit 已做), 不能把 kappा 写成"从 PROSE 骨架计算" (PROSE 不输出解骨架). |
| NULL | 扣除参数个数后 kappa 与迭代数无显著偏相关, 或 kappa-gate 不优于 refit-all; 仅排除"kappa 作 FEX 搜索剪枝信号"这一条, 不否定 SAGE-Fit 的 VarPro 修复 (已 positive). |
| NEGATIVE | 最优点 kappa 与沿轨迹 conditioning 脱节 (非线性骨架), 初始 kappa 不能预测收敛——形成"GSBS 的成因不是初始 conditioning 而是 landscape 多吸引域 (SAGE-Fit 的 needle-in-a-basin)"的失效边界, 但注意这个结论 SAGE-Fit 的 disconnected-basins 刻画已部分给出. |

## Alternative Framing

更可辩护的两条路 (均需大幅降 novelty 声明):
1. **可标定可迁移 kappa 阈值 + 对照研究**: 取消"新定理/新桥梁"声明, 把贡献收缩为一个干净的经验研究——"优化前 kappa (扣除参数个数) 作为 FEX/SR 搜索的廉价剪枝 gate, 在多 PDE 族上标定可迁移阈值, 并与 (a) SAGE-Fit VarPro+refit, (b) refit-all, (c) FEX 裸 loss-score 做 head-to-head". 必须把 Kronberger 2209.00942 与 SAGE-Fit 2605.23272 放正中心引用. 增量是"阈值的可迁移性 + 省预算量化", 属局部方法论. 
2. **彻底剥离 PROSE**: kappa 只定义在 **FEX 自己的候选解树**上 (FEX 持有方程、autograd 自算残差, 对象正确), 不写"从 PROSE 骨架算 kappa". 这样避开对象混淆, 但 Claim 3 (桥梁叙事) 随之消失, idea 退化为"FEX 内层优化的 conditioning 诊断", 与 skeleton-convexity/struct-coeff-sep 高度重叠, 应三者合并.

## Likelihood-Impact Matrix

- Priority: Low (3) = Likelihood: Low x Impact: Medium
- Numeric score for ideas.xml: 3
- Rationale:
  - Likelihood = Low: headline 需要多个互相独立的条件同时成立——(i) kappa 在扣除参数个数混淆后仍预测难度 (pilot 当前信号大半是 representability 钳位 + 参数个数 artifact, 未证), (ii) 初始/优化前 kappa 能预测沿轨迹收敛 (sibling convexity pilot 已实测非线性骨架全局非凸, 反向证据), (iii) 即便成立还要击败 SAGE-Fit 的 VarPro+refit 才有方法价值, (iv) Claim 3 桥梁还要 PROSE 真能输出可算 kappa 的解骨架 (对象错). 任一条都脆弱, 其中 (i)(ii) 是概念/实证层面已存疑.
  - Impact = Medium: 若 positive, 给"conditioning 作符号搜索剪枝信号"添一个 PDE-residual 数据点 + 一个未发表的 kappa-iteration 标定, 对 FEX 研究线有局部价值. 但 kappa-predicts-convergence 被 Kronberger 2209.00942 占、GSBS+VarPro-cure 被 SAGE-Fit 占, 即便成功也偏 incremental/已知机制实例化, 不构成独立 high-impact 叙事; 若 negative, 失效结论 (SAGE-Fit 的 disconnected-basins) 已部分预判, 增量有限. 故 Impact 封顶 Medium.
  - Claude 判断: Likelihood=Low, Impact=Medium. 独立 novelty-check subagent 亦判三条 claim 均 LOW (Claim 3 MEDIUM-but-plumbing), 与本 reviewer 一致.

## Overall

- Priority: Low
- Score: 3
- Comments: 首次 review. 这是 0616-fex condition-number 路线的"承载版本", 叙事 (kappa 作 FEX-PROSE 可计算接口) 干净, 且 **难得地真跑了 pilot** (优于同家族零-pilot 的 embed-condition). 但三个硬伤压低评分: (1) **headline 三块支柱逐条被已发表文献预占**——"kappa 预测收敛"撞 Kronberger arXiv:2209.00942 (EuroGP 2022, **本 idea 完全未引、几乎逐字的先例**: 已用 SVD 算 SR 系数拟合 Jacobian 条件数并挂钩收敛变慢) + 凸优化教科书; "kappa 诊断 GSBS + VarPro 修复"撞 SAGE-Fit arXiv:2605.23272 (AAAI 2026, GSBS 概念原主, **已把 GSBS 诊断为 VarPro 式 ill-conditioning 并用 Tree-Directed VarPro 修复**). idea 的 Novelty quick-check 对这两篇都有**事实层面的误述** (说 VarPro 文献"未应用于符号搜索"、说 GSBS 是 DSO"未解决"、把 SAGE-Fit 说成"非条件数引导"). (2) **pilot 的 POSITIVE 信号经审查是 artifact**: overall rho 大半由 poly/exp 类的 **kappa 硬钳天花板 (1e12, 因 smin=0 即无法表示 sin 的 representability 失败)** 驱动, 而 well-posed 子集的 rho 又被**参数个数**这个潜变量混淆 (kappa 与 n_feval 同随骨架规模涨); 且 pilot 在最优点事后测 kappa, 而定理/应用需要优化前的预测性 kappa——方向都没对. (3) **Claim 3 部分中 PROSE 对象混淆家族杀手**: PROSE Symbol Decoder 输出控制方程不输出解骨架 (FEX 已持方程、搜索的是解), "从 PROSE 骨架算 kappa 做 accept/reject"对象错位. **建议**: 不作为独立"新方法/新定理" idea 推进. 若续命, 按 Alternative Framing (1) 重构为"可标定可迁移 kappa 阈值作为搜索剪枝 gate"的经验对照研究, 把 Kronberger 2209.00942 + SAGE-Fit 2605.23272 放正中心, 并与 SAGE-Fit-VarPro / refit-all 做 head-to-head; 同时按 (2) 彻底剥离 PROSE 接口, kappa 只定义在 FEX 自己的候选解树上——但此时与 skeleton-convexity / struct-coeff-sep 高度重叠, 三者应合并为单一"FEX 内层 conditioning 诊断"工作而非各自独立. **最便宜的 decisive gate**: 先在 well-posed 骨架上做"扣除参数个数后 kappa 与迭代数的偏相关", 若偏相关消失则 headline 当场否决, 省下全部 FEX 实验预算.

</review>
