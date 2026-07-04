---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-prose-skeleton-convexity/
---

- One-sentence summary: 对 PROSE-PDE 的经验发现 (仅知骨架即可达到与已知方程几乎相同的预测精度) 给出理论解释: 证明对特定 PDE 类, 当符号骨架 (算子拓扑) 已知时, 系数优化问题变为 (强/弱) 凸的, 从而解释为什么 PROSE Skeleton 模式几乎不损失精度, 以及为什么 FEX 的 Adam+BFGS 系数优化阶段通常能快速收敛.

- Hypothesis: PROSE-PDE 的实验发现: Known 模式 (完整方程) 数据误差 0.92%, Skeleton 模式 (仅知结构) 数据误差 1.06%, 差距仅 0.14%. 这暗示: 对于 PROSE 训练涵盖的 PDE 类, 在正确骨架下系数优化是"容易的" (近凸的). FEX 的经验也支持这一点: 一旦 RL 找到正确的算子结构, Adam+BFGS 通常能快速收敛到高精度系数. 假设: 对线性 PDE (Poisson, advection-diffusion, wave) 和特定非线性 PDE (power-law nonlinearity), 可以证明在正确骨架下系数优化问题的 Hessian 是正定的 (严格凸), 或至少是 Polyak-Lojasiewicz 条件成立的 (保证梯度下降收敛). 推论: FEX 的搜索瓶颈完全在离散结构搜索, 不在连续系数优化, 这为 "PROSE 做结构 + FEX 做系数" 的管线提供理论保证.

- Expected outcome: 定理 1: 对 d 维线性椭圆 PDE, 若骨架为 sum-of-products 形式 (FEX S_k 空间中最常见的类型), 系数优化问题相对于 PDE 残差损失是 Polyak-Lojasiewicz 的, PL 常数仅依赖 PDE 的椭圆常数和骨架的树深. 定理 2: 对含 power-law 非线性的半线性 PDE, PL 条件在系数初始猜测与真值距离 < delta 时局部成立, delta 与非线性阶次成反比. 推论: PROSE 的 Skeleton 模式成功可以由 Transformer 逼近的一般性 + 系数优化的 PL 性联合解释. 失败信号: PL 条件只在极特殊的骨架形式下成立, 对一般 S_k 不成立.

- Contribution type: theory

- Risk: MEDIUM-HIGH

- Estimated effort:
  - Compute: 5 GPU-hours (数值验证 Hessian 正定性)
  - Data: available
  - Implementation: 3-6 weeks (数学证明 + 数值验证)

- Novelty quick-check: (1) FEX 原始论文证明了 S_k 的逼近论但未分析系数优化的凸性. (2) PROSE-PDE 论文只报告了 Skeleton 模式的数值结果, 未给理论解释. (3) Na & Yang (2025) 证明了 NN 优化的维度灾难下界, 但是 NN 的, 不涉及有限表达式的系数优化. (4) PDE 逆问题文献中有系数辨识的凸性分析, 但都是在给定 PDE 形式下辨识参数, 不涉及 S_k 空间或表达式树骨架. 差异化: 首次分析 "给定骨架下 PDE 符号求解的系数优化" 的凸性条件.

- Strongest objection: PL 条件可能只在极特殊的线性 PDE + 分离变量骨架下成立, 对一般非线性 PDE 和一般 S_k 骨架不成立, 使理论结果的实际指导意义有限.

- Why we should do this: 这在理论层面解释了两个社区各自观察到但未解释的现象: PROSE 的 "Skeleton 几乎等于 Known" 和 FEX 的 "Adam+BFGS 收敛很快". 统一解释将赋予两个方法各自的贡献以理论支撑, 也为 PROSE+FEX 混合管线 ("PROSE 做结构, FEX 做系数") 提供严格的正确性保证.

- Pilot:
  - Setup: 1D Poisson -u''=f, u(0)=u(1)=0, 真解 u*=sin(pi*x). 5 种骨架: (a) c1*sin(c2*x), (b) c1*x*(1-x)+c2*x^2*(1-x)^2, (c) c1*sin(c2*x)+c3*cos(c4*x), (d) c1*exp(c2*x)*sin(c3*x), (e) sum_k c_k*sin(k*pi*x). 多起点 Nelder-Mead+BFGS 优化, 有限差分 Hessian, 50 条随机 1D 切片测凸性.
  - Metric: 最优处 Hessian 正定 (min eigenvalue > 0); 小半径 (0.2) 切片凸性 > 90%.
  - Result: (a) loss=1.5e-32, min_eig=28.0, PD=yes, cvx@0.2=100%, cvx@2.0=20%. (b) loss=2.9e-2, min_eig=1.6, PD=yes, cvx@0.2=100%, cvx@2.0=100%. (c) loss=7.5e-33, min_eig=6.4e-33, PD=no(PSD), cvx@0.2=100%, cvx@2.0=44%. (d) loss=1.2e-20, min_eig=5.9, PD=yes, cvx@0.2=100%, cvx@2.0=26%. (e) loss=7.5e-20, min_eig=97.4, PD=yes, cvx@0.2=100%, cvx@2.0=100%.
  - Signal: POSITIVE (局部凸性). 所有骨架在最优解附近都是严格凸的 (100% 小半径切片凸), 支持 "系数优化在正确骨架附近是容易的". 但全局凸性仅对系数线性骨架 (b,e) 成立. 非线性频率参数骨架 (a,c,d) 全局非凸, 说明 BFGS 仍依赖好的初始化. 关键发现: "凸性是局部性质, 不是全局性质, 除非骨架对系数是线性的" -- 这直接回答了 PROSE Skeleton 模式为何有效 (PROSE 的 Transformer 隐式学到了好的初始化).

<review date="2026-06-24">

## Novelty

- Score: 3/10
- Closest prior work: Variable Projection / Separable Nonlinear Least Squares (Golub & Pereyra 2003, Inverse Problems 19:R1-R26; Dong & Yang, "Numerical Approximation of PDEs by a Variable Projection Method with ANNs", CMAME 2022 / arXiv:2201.09989)
- Key differentiator: 真正的差异只在 framing(把已知数学事实贴到 FEX 固定骨架 + PROSE Skeleton 现象上),而非数学内核。"线性 PDE + 系数线性骨架 ⟹ 残差最小二乘严格凸" 是 separable NLS / 椭圆系数反问题 / PINN 残差极小化三条文献的经典共识(VarPro 把线性系数闭式消去得到更好条件的 reduced problem;Harrach 2021 把线性椭圆系数辨识写成唯一可解凸 SDP)。Theorem 1 在系数线性情形基本是这一事实的复述。"非线性骨架最优点附近局部 PL" 是 LPLR(locally PL region)的通用结论——任何非退化最优点(Hessian PD)都自动局部 PL,这正是 pilot 的 cvx@0.2=100%,不是定理而是教科书推论。SR 社区已命名 "Good Structure, Bad Score" 现象(SAGE-Fit 2605.23272),且明确指出根因是非线性算子使内层高度非凸——与本 idea 乐观叙事相反。

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 3/10 | 致命的对象混淆:PROSE Skeleton 模式根本不做系数优化——它是 Transformer 在 "给定无系数骨架" 条件下直接预测数值解(2404.12355 Table:1.06% vs 0.92%),管线里没有任何 Adam+BFGS 系数拟合环节。FEX 才做系数优化。一个 "系数优化 PL" 定理在数学上无法解释 PROSE 的 0.14% gap;Expected outcome 里 "PROSE Skeleton 成功 = Transformer 逼近 + 系数优化 PL" 这条解释链不成立。 |
| Missing evidence signals | 4/10 | 必须把 VarPro / SAGE-Fit / 椭圆系数反问题凸化文献放到中心正面对齐(landscape §25 已补);并补 FEX 真实 S_k 骨架(含一元/二元算子的非线性频率/尺度参数)的 Hessian 谱 / 局部极小 / 条件数统计,而非仅 5 个 1D toy 骨架的切片。 |
| Narrative | 4/10 | "PROSE 成功 + FEX 快收敛 = 系数优化 PL" 很抓人但跳步过大;且 pilot 自己证伪了强版本——全局凸性只在系数线性骨架(b,e)成立,正确结构的 a_sin(含非线性频率 c2)全局非凸(cvx@2.0=20%)。 |
| Venue contribution | 3/10 | 对 AAAI/JMLR/Neural Networks/ICLR:系数线性骨架的强凸会被直接视为 VarPro / 最小二乘满秩条件的翻版;非线性骨架若只能给 "非退化最优点附近局部 PL",理论增量过弱,审稿人会判为已知机制的重述。 |
| Missing evidence signals (cheapest falsifier) | 7/10 | Expected outcome 含明确便宜 falsifier(一般 S_k 不满足 PL),pilot 已给出非线性频率参数全局非凸的反例信号——这条诚实且可执行。 |
| Outcome realism | 3/10 | "PL 常数仅依赖椭圆常数和树深" 不现实:它至少还依赖基函数相关性 / 配点设计 / 边界条件 / 参数尺度 / 骨架条件数(VarPro 文献明确)。最 "干净" 的 Theorem 1 落在系数线性的平凡情形,而 FEX 真正搜索的非线性骨架恰好是没有干净结果的区域。 |
| Contribution type compliance | 10/10 | idea types {theory} ⊆ preferred-contribution-types {method, theory, application}: yes |
| Overall Quality | 3.5/10 | 有可救的理论核(FEX 内层优化相图),但当前 claim 过宽、prior 压力极大、PROSE 解释链断裂。 |

## Contribution Drift (n >= 2 only; n=1 写 N/A)

N/A (n=1)

## Alternative Framing

收缩成 "固定 FEX 表达式骨架后的内层参数优化相图":线性进入残差的参数 → VarPro/满秩强凸或 PL;非线性内部参数 → 局部 PL 条件 + 全局非凸反例 + 条件数刻画。PROSE 只作 motivation,绝不作为 "被解释对象"。即便如此,核心结果仍高度依赖 VarPro,增量是 "把 separable-NLS 相图实例化到 FEX 算子树并给出失效边界",属局部诊断而非新理论路线。

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 在限定 PDE 类 + 限定骨架 + 满秩/可辨识条件下,可声称 FEX 内层参数拟合全局凸(系数线性)或最优点附近局部 PL(非线性),故结构搜索命中正确骨架后系数优化相对容易。不可外推到一般 S_k。 |
| NULL | 只能声称 "未发现一般 S_k 的统一 PL 条件";不可声称 "FEX 瓶颈完全不在连续优化",更不可声称解释了 PROSE Skeleton 的 0.14% gap(对象不同)。 |
| NEGATIVE | 若简单正确骨架也出现局部极小 / 鞍点 / 病态 Hessian,应否定 "给定骨架后系数优化普遍容易",转向 initialization / VarPro / multi-start 诊断叙事(这本身仍是有价值的负面边界)。 |

## Likelihood-Impact Matrix

- Priority: Low = Likelihood: Low x Impact: Medium
- Numeric score for ideas.xml: 3
- Rationale:
  - Likelihood = Low:正面 Theorem 1 很可能被 VarPro / 椭圆系数反问题凸化文献覆盖(数学内核非新);广义 PL 已被 pilot 自身的非线性骨架反例击穿(全局非凸),只剩 "局部 PL = 非退化最优点" 这一通用且弱的结论。要做成 top-venue theory 需要超出 VarPro 的实质新机制,目前看不到。
  - Impact = Medium:即使最乐观地按 Alternative Framing 做成 "FEX 内层优化相图",它主要是局部方法论 / 诊断推进(指导 FEX 何时可省 RL、何时必须好初始化),不改变领域共识也不开新路线;且最吸引人的 PROSE-解释 headline 在对象层面就不成立。

## Overall

- Priority: Low
- Score: 3
- Comments: 不建议按当前 "解释 PROSE Skeleton" 版本推进——对象混淆使 headline 不成立,且数学内核被 VarPro/separable-NLS 与椭圆系数反问题凸化文献直接覆盖,false-novelty 风险高。若要续命,必须收缩为 FEX inner-loop 优化相图并把 VarPro / SR 常数优化文献放在正中心,但增量仍偏弱。**Claude 与 codex 对 Impact 有 1 level 分歧**:codex 判 Impact=High(认为严谨刻画 FEX 内层何时易/何时必然非凸会直接影响搜索设计与 PROSE→FEX 叙事)给出 Low×High=Medium(5);Claude 判 Impact=Medium(最佳情形仍是 VarPro 相图的局部实例化 + 诊断,且 PROSE headline 不成立)给出 Low×Medium=Low(3)。两者均同意 Likelihood=Low、Novelty 3-4/10、且 "解释 PROSE Skeleton" 版本不可推进。取较保守的合并 Impact=Medium,最终 score=3。

</review>
