---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-active-collocation/
---

- One-sentence summary: 用自适应采样策略替代 FEX 的固定随机/均匀 collocation 点采样, 选择最能区分候选表达式的点来降低 RL reward 方差, 加速正确表达式的识别.
- Hypothesis: FEX 的 reward 评估经常在候选表达式一致同意的点上浪费计算; 自适应地选择候选表达式残差分歧最大的点可以更快地区分好坏表达式, 降低 controller 更新的梯度方差.
- Expected outcome: 成功: 自适应 collocation 在达到相同 score 阈值时所需的 RL 迭代次数减少 >30%; 失败: 自适应选点导致过拟合局部区域, 真实解的全局误差反而增大 -- 这本身也提供了关于 FEX score function 设计的洞察.
- Contribution type: method
- Risk: LOW
- Estimated effort:
  - Compute: 20 GPU-hours on A100
  - Data: available (FEX benchmark)
  - Implementation: 1-2 weeks
- Novelty quick-check: Active learning for PDE solving (DeepXDE 等) 已有自适应采样但用于 NN-based solver. 无人将 active collocation 与 expression tree RL search 结合. FEX 论文中采样策略是固定的 (Monte Carlo 均匀采样).
- Strongest objection: 自适应采样的额外开销可能抵消节省的 RL 迭代.
- Why we should do this: 这是一个低风险、模块化的改进, 可以与 FEX 的其他改进叠加使用; 且提供了关于 FEX score function 信息量的定量洞察.
- Pilot:
  - Setup: 在 Poisson 方程 d=10 上, (A) 标准 FEX (均匀采样 5000 点), (B) 每 50 iter 用 top-5 候选的残差最大分歧点替换 20% 的采样点, 单 seed 对比
  - Metric: 若 (B) 达到 score > 0.99 的迭代次数 < (A) 的 70%, 信号为正
  - Result: pending
  - Signal: SKIPPED

<deep-lit date="2026-06-16" scope="idea" rounds="1" new_wiki="8">

## Deep Lit 发现：Adaptive Collocation for PINN 已饱和，但 RL 表达式树搜索的 QBC 自适应采样完全空白

本次 deep-lit-tick 围绕 "fex-active-collocation" idea 展开系统性文献搜索（6 axes, 10 组 query + B0 web search），并通过 B7 反向扩展（references/cited/author/title 四维）验证饱和度。共精读 8 篇论文，全部写入 wiki。

### 核心发现

**1. PINN 自适应 collocation 领域已高度饱和（2022-2026）。** 8 篇论文涵盖该领域所有主流方法学：

| arXiv ID | 论文 | Year | 方法 | 与 idea 关系 |
|----------|------|------|------|-------------|
| [2504.12949](wiki/2504.12949.md) | RL-PINNs: RL-Driven Adaptive Sampling for PINNs | 2025 | DQN + function variation reward 驱动单轮自适应采样 | **最近 prior work**：首次将 RL 用于 PDE 自适应采样，但目标是 PINN 训练而非表达式树搜索 |
| [2404.07662](wiki/2404.07662.md) | PINNACLE: PINN Adaptive ColLocation | 2024 (ICLR) | NTK 特征谱 + 增广空间联合优化所有训练点类型 | 理论最深的方法，NTK convergence degree 准则可用于 FEX 采样诊断 |
| [2605.30910](wiki/2605.30910.md) | PINNs Failure Modes are Overfitting | 2026 | Double backprop 正则化抑制配点过拟合 | **支持 idea 动机**：PINN 失败 = 配点过拟合 → 更好的配点选择是根本解 |
| [2504.00910](wiki/2504.00910.md) | Provably Accurate Adaptive Sampling for PINNs | 2025 | Hessian-based quadrature + provable error bound | 提供可证明的采样理论框架，★-RAD 统一公式可用于系统性消融 |
| [2404.12282](wiki/2404.12282.md) | Guiding Information for Adaptive Collocation in PINNs | 2024 | 混合二阶偏导 U_xt / PDE_xt 作为信息探针 | 诚实报告失败案例，PDE_xt 思路（残差曲率 > 解曲率）有参考价值 |
| [2411.19632](wiki/2411.19632.md) | PACMANN: Point Adaptive Collocation Method | 2024 | Min-max 优化 + 梯度驱动点移动 | 高维表现最好（d=5 Poisson），但需要残差梯度计算（FEX 中不可用） |
| [2509.14198](wiki/2509.14198.md) | Variational Framework for Residual-Based Adaptivity | 2025 | 变分原理统一 RBA/RAD，指数势→L∞，二次势→L2 | 理论框架可指导 FEX 采样目标函数设计 |
| [2502.03963](wiki/2502.03963.md) | AL-PINN: Active Learning-Driven PINNs | 2025 (withdrawn) | MC Dropout uncertainty → 自适应采样 | 质量极低（已撤回），仅作方法学参考 |

**2. 无人将自适应 collocation 与 RL 驱动的表达式树搜索结合。** 这是本次 deep-lit 最关键的确认：所有 8 篇论文 + B7 反向扩展找到的数百篇参考文献中，自适应采样全部应用于 PINN（固定 NN 架构求解器），没有任何一篇在 RL 驱动的符号表达式树搜索中探索自适应 collocation。Idea 的核心 differentiator（QBC + RL expression tree search）**完全未被触及**。

**3. 关键对比：RL-PINNs vs fex-active-collocation**

| 维度 | RL-PINNs (2504.12949) | fex-active-collocation (本 idea) |
|------|----------------------|-------------------------------|
| 求解器类型 | PINN (固定 NN) | FEX (RL 搜索二叉树) |
| 采样目标 | 优化 NN 训练损失 | 降低 RL reward 方差 |
| 信息信号 | function variation (梯度无关) | 候选表达式间残差分歧 (QBC) |
| RL 角色 | DQN agent 选择采样点 | FEX Controller + 自适应采样为插件 |
| 理论保证 | 无 | 无 (可从 2509.14198 变分框架借理论) |
| 公开代码 | 无 | N/A |

**4. 可借用的方法论组件：**
- 2509.14198 的变分框架可直接映射到 QBC setting：候选残差分歧 → 势函数 → 采样分布 → reward variance bound
- 2605.30910 的 "配点过拟合" 诊断可为 NEGATIVE outcome（自适应采样导致过拟合）提供叙事支撑
- 2504.00910 的 ★-RAD 统一公式 p(x) ∝ γ(x)^τ + c 可用于系统消融不同信息源

### 撞车风险评估

| Claim | 撞车状态 | 证据 |
|-------|---------|------|
| QBC 自适应 collocation + RL 表达式树搜索 | ✅ 完全未被触及 | 8 篇 PINN 自适应采样 + B7 反向扩展 = 0 篇涉足 |
| RL 驱动的自适应采样 for PDE | ⚠️ RL-PINNs 存在但不构成撞车 | RL-PINNs 针对 PINN，方法学不可直接移植到 expression tree |
| 候选间残差分歧作为信息量判据 | ✅ 完全空白 | 所有 PINN 方法使用单模型残差/梯度/Hessian，无 committee-based |
| 自适应采样降低 policy gradient 方差 | ⚠️ 无直接证据但 RL 文献有相关讨论 | VIP (2602.01601)、Reinforce-Ada (2510.04996) 等验证了方差-效率 tradeoff |

### B7 发现的高价值补充候选（本轮未精读，交上层并入 landscape）

- **2207.10289** — "A Comprehensive Study of Adaptive Sampling for PINNs" (Wu et al. 2022, 698 citations): PINN 采样领域的标准参考文献，含 10 种采样方法的 6000+ 次模拟对比
- **2606.09949** — "OGAS: Learning Where to Simulate" (Cesar et al. 2026): 生成式主动采样 + 扩散模型指导 PDE 配置空间探索，方法论新颖但与 collocation 点选择正交
- **2502.07425** — "Foundation Model for PINNs: Multi-PDE Learning with Active Sampling" (Park 2025): 同 AL-PINN 作者，MC Dropout 多 PDE 主动采样
- **2509.01234** — "RAMS: Residual-based Adversarial-Gradient Moving Sample" (Ouyang et al. 2025): 梯度驱动采样点移动，首个应用于 operator learning 的自适应方法

</deep-lit>

<review date="2026-06-16">

## Novelty

- Score: 6/10
- Closest prior work: RL-PINNs (Song 2025, arXiv:2504.12949) — RL-driven adaptive collocation for PINN-based PDE solvers; DISCOVER (Du, Chen & Zhang 2022, Phys Rev Research) — RL-based symbolic PDE discovery with expression trees (fixed sampling); Aikawa et al. (2023, JSAI) — dropout-based uncertainty (ensemble disagreement) for PINN collocation; PINNACLE (Lau et al. 2024, ICLR) — NTK-based joint collocation optimization; Deep Collocation Method (Weng, Mao & Shen 2025, arXiv:2502.17203) — greedy subspace construction with residual-guided rejection sampling.
- Key differentiator: 本 idea 的独特组合在于: (1) 自适应 collocation + (2) RL 驱动表达式树搜索 + (3) 候选表达式间残差分歧 (而非单模型残差) 作为信息量判据。三者各自有大量 prior art (adaptive collocation for PINNs / RL for symbolic PDE / ensemble disagreement in active learning), 但三者结合应用于 FEX 式的 PDE 搜索未见已有工作。新颖性本质上是增量式的交叉组合, 而非机制层面的突破。若实验揭示"分歧导向的采样比残差导向的采样在表达式树搜索中具有定性不同的行为"这一意外洞见, novelty 可升至 7/10; 若仅为 30% 加速, 则 novelty 有限。

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 5/10 | "自适应采样降低 controller 梯度方差"的论述缺乏严格论证。连接链: 自适应选点 → 更精确区分候选表达式 → 降低 reward 估计噪声 → 降低 policy gradient 方差 → 加速收敛。但每个箭头都需验证: (a) 残差分歧大的点是否确实携带更多区分信息? (b) 采样分布随迭代变化是否引入非平稳性, 反而增加方差? (c) FEX 的瓶颈是 reward 估计精度还是 controller 探索 (找到好的算子序列)? 后者可能才是主瓶颈, reward 降噪的边际收益有限。此外, 计算 inter-candidate residual divergence 的额外开销未与节省的 RL 迭代做成本-收益量化。 |
| Missing evidence signals | 4/10 | (1) 缺少"同等计算预算下更多均匀点"的 ablation——30% 加速可能是因为 adaptivity, 也可能仅仅因为替换的 20% 点在分歧区域恰好覆盖了均匀采样易忽略的关键区域; (2) 缺少维度 scaling 分析——自适应采样的优势应随维度升高而增大 (高维中均匀采样更浪费), 但 idea 未论证; (3) 缺少多 PDE 类型 (光滑 vs 激波 vs 振荡) 的预期行为差异分析; (4) 未讨论与 FEX 已有探索机制 (epsilon-greedy) 的交互——自适应选点可能降低探索多样性, 与 epsilon-greedy 产生负交互。 |
| Narrative | 5/10 | 当前 framing 是"将自适应采样应用到 FEX", 属于弱叙事。更锐利的 framing 应为"Query-by-Committee Active Learning for RL-based Symbolic PDE Solving"——将候选池自然解释为 committee, 分歧导向采样是 QBC 原则的自然实例化。这会将 idea 从 heuristic trick 升级为有原则的方法。此外, narrative 缺少"why now"——FEX 已有 2 年历史, 为什么现在才是做自适应采样的时机? 答案可能是 FEX 的搜索瓶颈刚刚被系统认识到 (见 landscape 中 gap 3), 或 RL 驱动的表达式树搜索刚刚达到需要精细调优 reward signal 的阶段。 |
| Venue contribution | 5/10 | 按 FEX 系列的发表历史 (JMLR, JCP), 合理的目标 venue 是 JMLR/JCP 而非 NeurIPS/ICML。若目标 NeurIPS/ICML: 30% 迭代减少的单项改进不够; 需要 (a) 一个理论洞见 (如"分歧导向采样等价于 policy gradient 方差的最优重要性采样"), 或 (b) 一个反直觉实验发现 (如"自适应采样改变了 controller 收敛到的表达式结构, 而不仅仅是加速了同一结构的收敛")。若目标 JMLR/JCP: 作为 FEX 的算法改进 + 系统实证分析, 有合理的发表空间, 但需更强的 ablation 和更广的 PDE 覆盖。 |
| Testability | 7/10 | Pilot 设计合理: Poisson d=10, (A) 标准 FEX vs (B) 每 50 iter 替换 20% 采样点, 单 seed。最便宜的证伪信号清晰: 若 (B) 的迭代次数不显著低于 (A) 的 70%, 则 idea 失败。局限: (1) 单 seed 不足——FEX 的随机性 (参数初始化、算子采样、MC 积分) 需要 multi-seed 才能区分信号和噪声; (2) 仅测试一个 PDE——光滑 Poisson 方程对自适应采样可能是 easiest case (残差均匀分布), 真正的区分能力应在有局部特征的 PDE (Burgers, Allen-Cahn) 上测试。 |
| Outcome realism | 6/10 | POSITIVE 的 30% 迭代减少是 aggressive but plausible。PINN 自适应采样文献中典型提升为 10-40% 的精度改进, 而非迭代减少——这两个指标不可直接类比。NEGATIVE 场景 (过拟合局部区域导致全局误差增大) 是真实且有趣的风险, 且提供了关于 FEX score function 设计的洞察——这一负面结果本身具有发表价值。NULL 场景 (加速不显著但不过拟合) 最无趣, 需要提前规划如何从 NULL 中提取有意义的结论。 |
| Contribution type compliance | n.a. | topic 未声明 preferred-contribution-types, 跳过检查。 |
| Overall Quality | 5/10 | Idea 具体、可测试、低风险, 但叙事的锐度和贡献的深度不足以支撑独立顶会投稿。最大价值在于: (a) 作为 FEX 工程改进池中的一个低风险模块化选项; (b) 提供关于 FEX score function 信息量的定量诊断——无论正面还是负面结果都有 insight。建议不要在未获得 pilot 信号前投入过多 narrative polishing; 先跑 pilot 获取信号, 再根据信号方向决定是包装为 method paper (positive) 还是 diagnostic paper (negative/null)。 |

## Contribution Drift

N/A (v1, 无前版)

## Alternative Framing

**Query-by-Committee Active Learning for RL-based Symbolic PDE Solving** — 将 FEX 的候选池 (capacity K=10) 自然解释为 committee, 候选间残差分歧是 QBC 原则 (Seung et al.) 的自然实例化。这一 framing 将 idea 从 "apply adaptive sampling to FEX" 的 heuristic 升级为"在 RL 表达式树搜索中实例化和验证 QBC 原则在 PDE 域的适用性", 使 idea 有更清晰的知识贡献锚点, 也更容易在 related work 中定位和 differentiate。同时, 无论结果是 POSITIVE (QBC works for expression tree search) 还是 NEGATIVE (QBC fails, revealing that expression-tree reward surfaces have different structure than classification), 都有发表价值。

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 候选间残差分歧导向的自适应 collocation 在 Poisson d=10 上将达到 score>0.99 所需 RL 迭代次数减少 >30%, 且该优势在至少 1 个额外 PDE (Burgers 或 Allen-Cahn) 上复现, 证明 QBC 原则在 RL 表达式树搜索中有效。 |
| NULL | 自适应 collocation 在 Poisson 上加速不显著 (<15%), 但在 Burgers (有激波, 残差分布不均匀) 上显示 >20% 加速——表明收益高度 PDE-dependent, 揭示了 FEX score function 的信息分布在不同 PDE 类型上的异质性。 |
| NEGATIVE | 自适应选点导致全局误差增大 (过拟合局部), 但该发现本身揭示了 FEX score function 的一个设计缺陷: 当前 score 对局部残差的敏感度过高, 建议在 score 中加入全局正则化项 (如候选表达式的 Sobolev 范数差异)。 |

## Likelihood-Impact Matrix

- Priority: Medium (5) = Likelihood: Medium x Impact: Medium
- Numeric score for ideas.xml: 5
- Rationale:
  - **Likelihood = Medium**: 有明确的实验路径 (pilot 设计合理, FEX 代码库可用, 20 GPU-hours 预算可行), 但做成 top-venue 级结果依赖若干条件成立: (a) 候选间残差分歧确实与信息增益相关 (而非仅仅反映候选在 trivial 区域的随机波动); (b) 自适应选点的计算开销 (需额外前向传播 top-K 候选、计算 pairwise 残差) 不超过总计算时间的 10-15%, 否则净收益消失; (c) 自适应采样引入的非平稳性不破坏 controller 的收敛 (policy gradient 对 reward 分布漂移敏感); (d) 收益在至少 2-3 个 PDE 类型上一致, 而非仅在 Poisson 上显著。四个条件均合理但不确定, 且任一失败都会将结果推向 NULL/NEGATIVE。不降为 Low 因为 pilot 成本低、信号获取快, 且 NULL/NEGATIVE 也有发表价值。
  - **Impact = Medium**: 在最乐观的成功情形下 (QBC 原则在 FEX 上验证有效, 且给出了可迁移的采样设计准则), 对 FEX 研究线有清楚推进——提供了一个可与其他 FEX 改进 (LLM skeleton, synchro prior 等) 叠加使用的采样模块, 以及关于"score function 的信息分布"的定量洞见。但不会改变更广泛领域的判断或范式: QBC 在分类/回归中的有效性是已知的, 将其成功应用于 PDE 表达式树搜索不会 surprise the community; 它更多是确认了已知原则在新区间的适用性, 而非发现新原则。若结果为 NEGATIVE (QBC 反而不如均匀采样), 冲击反而更大——会迫使社区重新思考"什么构成表达式树搜索中的 useful sample"。
  - Codex second opinion: codex session 未在时限内产出结构化 review (search loop 过长), 无法合并。本 review 的 Likelihood/Impact 判定完全基于 Claude 独立评估。

## Overall

- Priority: Medium
- Score: 5
- Comments: 这是一个结构完整、可测试、低风险的模块化改进 idea。主要短板: (a) 贡献深度不足以支撑独立顶会投稿——30% 迭代减少在 FEX 社区有价值, 但对外部审稿人来说不够 compelling; (b) 叙事的锐度不足——当前 framing 是 heuristic trick, 建议采用 Alternative Framing 升级为 QBC 原则的 domain 实例化和验证; (c) Pilot 需扩展到至少 2 个 PDE 类型 + multi-seed。建议: 先按 pilot 设计跑实验获取信号, 再根据信号方向决定下一步——POSITIVE 则扩展 PDE 覆盖并深化 QBC framing 写 method paper; NULL/NEGATIVE 则按 diagnostic paper 路线整理洞见。文件大小: <5 KB, 无超长警告。

</review>
