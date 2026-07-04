---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-failure-taxonomy/
---

- One-sentence summary: 通过控制性消融实验系统分类 FEX 搜索失败的模式 (频率分离、参数简并、树深不匹配、算子集缺失、非凸 landscape 等), 建立 FEX 的失效边界地图.
- Hypothesis: FEX 的搜索失败不是单一原因, 而是多种失效模式的组合; 通过系统分离各因素, 可以识别出主导失效模式并为每种模式设计针对性的缓解策略. 预计频率分离和参数简并是最常见的失效模式.
- Expected outcome: 成功: 建立包含 5-8 种失效模式的分类学, 每种模式有可复现的触发条件和定量指标 (如 controller entropy、score variance、参数 Hessian 条件数); 失败: 失效模式高度耦合无法分离, 说明 FEX 搜索的困难是整体性的而非可分解的 -- 这本身也是重要发现.
- Contribution type: diagnostic+empirical-finding
- Risk: LOW
- Estimated effort:
  - Compute: 50 GPU-hours on A100
  - Data: available (FEX benchmark + 设计的 stress-test PDE)
  - Implementation: 2-3 weeks
- Novelty quick-check: 无人对 RL-based symbolic regression 做过系统的失效模式分类. Multi-Scale FEX (2510.22497) 只识别了频率分离一种. SRBench++ (Franca et al. 2024) 对 GP-based SR 做了性能分析但不是失效模式分类.
- Strongest objection: 诊断性研究缺乏方法论贡献, 可能被审稿人视为"只是报告了一堆实验".
- Why we should do this: 这是 FEX 未来发展的路线图 -- 知道在哪里失败比知道在哪里成功更有价值. 每个识别出的失效模式都指向一个具体的改进方向, 为后续研究奠定基础.
- Pilot:
  - Setup: 在 3 个 PDE benchmark (Poisson, conservation law, oscillatory Helmholtz) 上运行 FEX 100 次, 记录每次搜索的 controller 概率分布演化、score 轨迹、最终表达式, 按成功/失败分组统计
  - Metric: 若能识别出 >= 3 种可复现的失效模式 (每种至少在 10% 的运行中出现), 信号为正
  - Result: pending
  - Signal: SKIPPED

<review date="2026-06-16">

## Novelty

- Score: 6/10
- Closest prior work: Multi-Scale Finite Expression Method (Hardwick & Yang, 2025, arXiv:2510.22497); The Inefficiency of Genetic Programming for Symbolic Regression (Kronberger et al., PPSN 2024, arXiv:2404.17292)
- Key differentiator: Multi-Scale FEX 仅识别了单一失效模式 (频率分离导致 RL 不稳定), Kronberger et al. 仅分析了 GP 的搜索低效 (等价表达式重复), 均未建立多模式分类学. 本 idea 是首个对 RL 驱动符号回归搜索做系统多模式失效分类的工作. 但方法论本身 (消融→分类) 属于已知范式, 新颖度集中在 domain application 层面.

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 6/10 | 候选失效模式 (频率分离、参数简并、树深不匹配、算子集缺失、非凸 landscape) 之间存在潜在因果重叠 — "参数简并"和"非凸 landscape"可能是同一底层问题的不同症状. 区分 representational failure (算子集/树深) 与 search failure (RL 收敛) 的逻辑框架尚未建立. |
| Missing evidence signals | 5/10 | 缺乏 oracle ablation 设计: oracle tree depth, oracle operator set, exhaustive small-tree search 作为上界对照; 缺乏 multi-start / global parameter fitting baseline 以区分参数优化失败与搜索失败; 缺乏 random controller baseline 以校准 RL controller 的边际贡献. |
| Narrative | 6/10 | "FEX 失效分类学"的当前叙事过于局限于 FEX 内部调试. 更锐利的叙事方向是 "RL 驱动符号回归的受控诊断框架" — 将贡献从 "为 FEX 画地图" 提升到 "建立 RL-based SR 失效分析的实验方法学". 此外, idea 自己承认了最强反对理由 (纯诊断缺乏方法贡献), 但未给出有效的叙事应对. |
| Venue contribution | 5/10 | 按 FEX 系列的发文 venue (JMLR/JCP) 标准, 纯诊断研究在缺乏方法论贡献或 benchmark artifact 时较难获得顶级接受. 若含 public stress-test benchmark + 1-2 个反直觉发现 (如 "频率分离并非主导失效模式, 参数简并才是"), 则贡献等级显著提升. 若 target venue 为 NeurIPS D&B track 则需 benchmark artifact. |
| Testability | 7/10 | Pilot 设计合理: 3 PDE × 100 次 FEX 运行, 按成功/失败分组统计, 阈值设为 ≥3 种可复现失效模式. 这是廉价的证伪信号. 但 "失效模式高度耦合" 这一 NULL 场景缺乏量化判据 — 什么统计量决定 "不可分离"? |
| Outcome realism | 6/10 | 50 GPU-hours 和 2-3 周实现时间 realistic. 但 5-8 种失效模式的预期可能过于乐观 — pilot 仅覆盖 3 个 PDE, 可能不足以暴露所有模式. 设计能特异性触发每种候选模式的 stress-test PDE 本身就是非平凡工作. |
| Contribution type compliance | N/A | topic 未声明 `preferred-contribution-types`, 跳过检查. |
| Overall Quality | 6/10 | 方向正确且可执行, 但当前版本在 causal isolation 设计、benchmark artifact 和顶层叙事方面存在显著弱点. 适合作为内部 pilot/诊断项目, 要成为顶会投稿还需 sharpening. |

## Contribution Drift (n >= 2 only; n=1 写 N/A)

N/A (v1)

## Alternative Framing

当前 framing: "FEX 失效分类学" — 将贡献锚定在 FEX 这一特定方法上.
建议替代 framing: **"A Controlled Experimental Framework for Diagnosing Failure Modes in RL-Guided Symbolic Regression"** — 将核心贡献从 taxonomy product 转变为 diagnosis methodology: 如何设计受控消融实验来分离 representational insufficiency / score miscalibration / parameter-optimization failure / policy-collapse failure 四类正交失效轴. 附带一个 public stress-test PDE suite 作为 benchmark artifact. 这一 framing 将贡献从 "给 FEX 画了一张诊断地图" 提升到 "建立了 RL 驱动 SR 诊断的方法学标准", 显著增强 top-venue 可行性.

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 在受控 stress-test PDE 上, FEX 搜索失败可分解为 4-6 种有可区分触发条件和定量指标的操作性失效模式; 该分类学为每种模式指明了针对性改进方向. |
| NULL | 候选失效模式在操作层面高度耦合, 无法被受控消融分离; 这表明 FEX/RL-SR 搜索困难具有整体性, 分治策略可能不适用. |
| NEGATIVE | 识别的失效模式在 seed/task 间不可复现, 或无法预测新的 stress-test PDE 上的失效表现; 说明当前诊断指标的区分效力不足. |

## Likelihood-Impact Matrix

- Priority: Medium (5) = Likelihood: Medium x Impact: Medium
- Numeric score for ideas.xml: 5
- Rationale: **Likelihood=Medium**: FEX 代码库已存在, pilot 设计合理且成本可控, 执行路径清晰. 但做成 top-venue 级结果依赖若干条件: (a) 失效模式在操作层面可分离; (b) 能设计出有效 trigger 各模式的 stress-test PDE; (c) 定量指标能有效区分不同模式. 这些条件在首次尝试中未必全部满足. **Impact=Medium**: 成功的分类学将显著指导 FEX 改进路线, 对 RL 驱动 SR 社区有参考价值; 但作为诊断性贡献, 不会改变领域范式或开启新研究路线.

## Overall

- Priority: Medium
- Score: 5
- Comments: 本 review 与 codex second opinion 在 Likelihood 上存在 1 级分歧 (Claude: Medium, codex: Low). codex 认为当前版本缺乏 method/benchmark artifact/surprising pilot signal 导致 top-venue Likelihood 为 Low. Claude 认为按 FEX 系列的 target venue (JMLR/JCP) 标准, 诊断性研究有一定发表空间, 且 pilot 设计可行; 但同意 codex 的核心关切: 纯诊断分类学若不附 benchmark artifact 或反直觉发现则贡献偏弱. 建议: 优先跑 pilot (3 PDE × 100 seeds) 获取初步信号; 若 pilot 揭示 ≥3 种可区分失效模式, 再按 alternative framing 方向 refine 并补充 stress-test suite 作为 public artifact.

#if file_size_kB > 10
超长警告: 文件 ~3 KB, 未超上限
#endif

</review>

<deep-lit date="2026-06-16" scope="idea" rounds="2" papers_read="7">

## 文献饱和确认

围绕 fex-failure-taxonomy 的 6 个搜索轴 (方法/应用/数据/评估/失败模式/对抗) 进行了 2 轮系统搜索。已读 7 篇新论文 (此前该 idea 无专属 deep-lit)，全部写入 wiki。R2 仅产出 1 篇强相关论文，B7 反向扩展产生的新候选均为已读或 SR 无关，文献搜索达到饱和。

**核心发现**: 无任何已有工作对 FEX/RL-SR 搜索失败做过系统多模式分类。fex-failure-taxonomy 的 novelty 依然完整。但单一维度的 failure analysis 已有多篇工作覆盖不同侧面，必须做 ≥4 种失效模式的受控消融才能维持 novelty。

## 新收录论文及解读

### 直接撞车风险 (HIGH)

1. **EGRL-SR (2601.14693)** — Jianwen Sun et al., 2026.01 [[wiki](wiki/2601.14693.md)]
   - **关键发现**: 明确诊断了 SR 搜索失败的根本原因——error-based guidance 导致结构不同的表达式获得相似分数，搜索方向模糊。用 GCRL + HER + APSR 二值奖励 + structure-guided exploration 系统性地解决此问题。
   - **与 fex-failure-taxonomy 的关系**: 这是当前最接近 "为什么 SR 搜索会失败" 的系统分析。但其 focus 是 solution (如何修) 而非 taxonomy (如何分类)。fex-failure-taxonomy 需要将其作为 baseline，但在 "分类学 + 诊断方法论" 层面区分——EGRL-SR 只覆盖了 error ambiguity 这一种 failure cause，未系统分类。
   - **可复用方法**: all-point satisfaction binary reward, HER 轨迹重标注, structure-guided heuristic exploration

2. **NRSR (2501.01085)** — Chenglu Sun et al., 2025.01 [[wiki](wiki/2501.01085.md)]
   - **关键发现**: 系统分析了 SR 在高噪声/无关变量下的 failure。L0 gating module + PPO + MPE entropy bonus。消融: No-NGM 导致恢复率从 89% 降至 33%。
   - **与 fex-failure-taxonomy 的关系**: 直接 hit 了 "噪声/无关特征作为可控 failure trigger" 这一实验设计思路。但其 noise 定义仅限于无关变量 (synthetic)，未覆盖真实测量噪声或 PDE 特定噪声。fex-failure-taxonomy 可以在更广的 noise taxonomy (采样噪声/离散误差/边界条件扰动) 上扩展。
   - **可复用方法**: L0/Binary Concrete gating, mixed path entropy, action-mask 集成

### 方法学模板 (MEDIUM-LOW)

3. **Discovery under Hypothesis Redundancy (2606.14386)** — Li C. Xia, Baoxun Wang, 2026.06 [[wiki](wiki/2606.14386.md)]
   - **关键发现**: 提出 Search Compression Hypothesis——非局部探索仅在谱压缩、正交逃逸、残差信号对齐三个几何条件同时成立时有收益。在 SR benchmarks 上验证了 discovery saturation。提供 effective rank 和 escape distance 作为诊断指标。
   - **与 fex-failure-taxonomy 的关系**: 为 FEX 搜索饱和 (failure mode #5) 提供了形式化理论框架。可将其 geometric conditions 作为 FEX 搜索是否进入 "无效探索区" 的 diagnostic signal。
   - **可复用方法**: effective rank, escape distance, Predictive Novelty (PredNovelty), Residual Signal Alignment (RSA)

4. **PAGER (2309.10977)** — Thiagarajan et al., 2023.09 [[wiki](wiki/2309.10977.md)]
   - **关键发现**: 提出 anchored training + manifold non-conformity 的系统性 failure characterization。核心发现: epistemic uncertainty 必要但不足——OOS/OOD 可能低不确定但高误差。
   - **与 fex-failure-taxonomy 的关系**: 其 "风险分档" (risk regime) 方法论可直接迁移为 FEX search failure 的分类框架。Score_1 (forward anchor) + Score_2 (reverse anchor) 的双信号设计可作为诊断模板。
   - **可复用方法**: anchored training, manifold non-conformity scores, risk regime 分档

5. **EPO (2509.22576)** — Wujiang Xu et al., 2025.09 [[wiki](wiki/2509.22576.md)]
   - **关键发现**: 识别了 RL 中 exploration-exploitation cascade failure: 早期 premature convergence → 晚期 policy collapse。Entropy smoothing regularizer 防止熵剧烈波动。
   - **与 fex-failure-taxonomy 的关系**: 为 failure mode #6 (policy collapse/entropy degeneration) 提供了命名和诊断方法学。FEX controller 的 entropy trajectory 可以按 EPO 的框架分析。
   - **可复用方法**: entropy smoothing regularizer, trajectory-level entropy diagnostics, 历史平均熵 corridor

6. **CDE (2509.09675)** — Runpeng Dai et al., 2025.09 [[wiki](wiki/2509.09675.md)]
   - **关键发现**: 识别 calibration collapse 作为 RLVR failure mode。Actor PPL + critic multi-head variance 作为 curiosity/diagnostic signal。
   - **与 fex-failure-taxonomy 的关系**: 为 FEX controller 的 calibration 诊断提供了具体指标设计模板。CDE 的 "calibration collapse" 概念在 FEX 中可能表现为 controller 对某些表达式子空间过度自信。
   - **可复用方法**: actor PPL bonus, multi-head critic variance, calibration collapse detection

7. **Beyond Inference-Time Search (2605.18374)** — Massoudi et al., 2026.05 [[wiki](wiki/2605.18374.md)]
   - **关键发现**: 在受控 SDS 基准上设计暴露特定 failure mode (heuristic trap) 的实验。Negative ablations 揭示了反直觉发现: standard stabilizers 反效果, soft feasibility gate 失败。
   - **与 fex-failure-taxonomy 的关系**: 为 fex-failure-taxonomy 的 stress-test PDE 设计提供了实验方法学范本——如何设计能特异性触发某种 failure mode 的受控环境。
   - **可复用方法**: controlled failure mode exposure design, negative ablation methodology, compile-once evaluation

## 更新后的撞车风险矩阵

| 候选失效模式 | 6/16 状态 | 最接近竞品 | 空间 |
|------------|----------|-----------|------|
| 频率分离 | 🟢 仅 Multi-Scale FEX 识别 | Multi-Scale FEX (已读 2510.22497) | 仅识别，未分类 |
| 参数简并 | 🟡 SAGE-Fit 分析了但目标是修 | SAGE-Fit (已读 2605.23272) | solution vs diagnosis |
| 树深不匹配 | 🟢 完全空白 | 无 | 完整贡献 |
| 算子集缺失 | 🟢 完全空白 | 无 | 完整贡献 |
| 非凸 landscape | 🟡 Kronberger GP 分析 + SAGE-Fit | Kronberger (已读 2404.17292) | 无 RL-specific |
| Policy collapse | 🟡 EPO/CDE 在 LLM RL 中分析 | EPO (2509.22576), CDE (2509.09675) | 无 SR-specific |
| 搜索饱和 | 🟡 Discovery Bottleneck Theory | 2606.14386 | 无 FEX 实验 |
| 噪声敏感性 | 🟡 NRSR 仅无关变量噪声 | NRSR (2501.01085) | 无 PDE 噪声分类 |

**判定**: ≥4 种失效模式在 FEX 上下文中完全未被系统研究。fex-failure-taxonomy 的 novelty 安全。但必须在 artifact (stress-test PDE suite + 可复现指标) 和叙事 (从 taxonomy product → diagnosis methodology) 上 sharpening，否则 top-venue 可行性受限。

## 对 idea design 的影响

1. **Stress-test PDE 设计**: 借鉴 2605.18374 的受控 failure mode 暴露方法学，为每种候选 failure mode 设计特异性触发 PDE（而非仅 3 个通用 benchmark）
2. **诊断指标**: 从 EPO (entropy corridor), CDE (calibration), PAGER (risk regime), 2606.14386 (effective rank) 中选取/改造定量指标
3. **区分 representational vs search failure**: PAGER 的双信号设计启发了 "representational insufficiency vs search failure" 的逻辑区分框架
4. **Oracle baselines**: 补充 oracle tree depth / oracle operator set / exhaustive small-tree search 作为上界对照（reviewer 建议）
5. **NULL 场景判据**: 定义 "失效模式不可分离" 的量化标准——例如各模式间的条件互信息 < 阈值

</deep-lit>
