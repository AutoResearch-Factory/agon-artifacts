<!-- 书写报告使用中文 -->
---
topic: topics/0710-causal-scs.md
landscape: topics/0710-causal-scs-landscape.md
workspace: workspace/causal-scs-indicator/
---

- One-sentence summary: 在目标边、条件集维度和统计流程都相同的条件下，审计滑窗因果估计的不稳定性：区分真实边变化与混杂漂移，判断条件化何时提高或损害离散型 D / 突变型 J 的可探测性，以及这些信号能否超过原始变量 EWS。
- Hypothesis: 条件化指在检验目标边时控制第三变量。固定目标边、条件集维度、窗口和推断流程后，它的作用应主要取决于扰动来自真实边还是混杂机制，而不是“事件前/事件后”标签。真实目标边变化时，控制真实混杂应提高 D/J 的灵敏度；只有混杂机制漂移时，条件化可能去掉边际相关中的 D 信号，但这一损失未必适用于其他通道、时段或估计器。任何预警价值仍须独立超过完整的 variance+AC1 原始变量 EWS。
- Expected outcome: 当前只支持一个窄结论：D 稳定检测所测真实边变化，J 只在两个事件前的强变化单元过门槛；最大“仅边变化”单元中，控制真实混杂在事件前后都优于控制等维虚假变量；最大“仅混杂漂移”单元只有事件前 D 出现小幅损失，时段交互均不成立，所有估计器与原始变量 EWS 的配对差值都偏向后者。若要形成顶级 Earth-system/AI4Science empirical/diagnostic 稿件，必须在非退化可交换空对照、非线性/local-independence DGP 和第二估计器上复现这种扰动来源差异，再在去重风暴系统上证明严格提前量增量。最便宜的下一证伪是空对照失准，或第二估计器不能复现真实边变化下的收益；任一发生都继续停止完整 HRRR/radar 下载。
- Contribution type: empirical-finding+diagnostic
- Contribution drift note: v3 是 `empirical-finding+diagnostic`；v4 完全保留这两个类型，不新增 method、benchmark、application、theory 或 dataset，也未删除已有类型。
- Risk: HIGH
- Estimated effort:
  - Compute: v4 gate 实测为 1 个 L4 作业、2.13 秒核心计算；非退化空对照、非线性 DGP 和第二估计器复现预计低于 2 GPU-hours、约 50 CPU-hours。只有再过 gate 才投入真实阶段的 10--30 GPU-hours 特征抽取与 500--1500 CPU-hours 滑窗估计。
  - Data: needs collection；SPC 标签样例和三个 HRRR 3-km 字段子集已就绪，完整小时序列仍被停止门阻塞。
  - Implementation: 1--2 周完成可识别性复现；若通过，真实数据功效分析、storm-system 去重和验证另需 4--6 周。
- Novelty quick-check: RCV-PCMCI/VCDF (2410.19412/2602.21381) 用跨折 Consistency/Variability 过滤不稳定边，并未检验滑窗不稳定性在何种扰动来源下可识别；conditioning-depth graph instability (2606.01214) 通过改变条件集深度诊断隐藏混杂，却没有在等维条件集下比较扰动来源、时段和原始变量 EWS；PCMCI+ coherency score (2502.14719) 复用 CI 日志报告内部矛盾，不做事件判别；tvVAR-EWS (2205.07576) 用预测 VAR 的不确定性预警慢变生态转折。这里没有提出新的不稳定性分数。窄增量在于用真实混杂与虚假变量的等维对照及两阶段/分层推断表明：v3 的大幅损失在匹配后消失，真实边变化反而得到小而稳定的条件化收益，事件前后不构成边界。该发现目前只在一个线性 DGP 上成立。
- Strongest objection: 在线性高斯、固定目标边的场景中，“控制真实混杂有助于检测真实边变化”可能接近教科书结论；空对照又因 event/control 完全相同而机械得到 AUC 0.5。原始变量 EWS 全面占优，且没有真实风暴证据，因此当前结果更像一次严格的反证审计，还不是足以投稿的顶会贡献。
- Why we should do this: v3 的 `PCMCI+ - marginal = -0.222` 曾诱导出“因果条件化丢弃预事件信号”的强叙事，但它混合了父集搜索、显著性筛选、DGP、事件前后和窗口泄漏。v4 以很低的成本排除了这条归因，并说明什么证据出现后才值得投入真实数据。
- Pilot:
  - Setup: 在 NVIDIA L4 上运行预注册的四变量时变 VAR 因子实验：时段 `{pre,post}` × 真实边变化 `{0,0.2,0.4}` × 混杂漂移 `{0,0.25,0.5}`，每格 96 对共享随机创新的 event/control、8 个随机种子；32 个 controls 校准、64 对评估。主对比固定 `x_{t-1}->y_t`，使用相同数量的协变量，分别控制真实混杂 `z` 或独立虚假变量 `u`；400 次 calibration+evaluation bootstrap 后再重采样随机种子。事件前窗口全部在截止点前结束，事件后窗口全部在截止点后开始。
  - Metric: D/J 各自要求 AUROC >= 0.65 且 95% CI 下界 > 0.50；条件化损失/收益分别要求 `Delta=AUC_true-AUC_sham` 的区间完全小于/大于 0；时段 claim 要求至少两个非空单元的 `|Delta_post-Delta_pre|>=0.05` 且区间排除 0；预警 claim 还须以配对区间超过 variance+AC1 原始变量 EWS。
  - Result: 空对照单元为 AUC 0.500、Delta 0。最大“仅边变化”单元：事件前 `Delta_D=0.042 [0.023,0.067]`、`Delta_J=0.035 [0.013,0.054]`，事件后 `0.049 [0.020,0.075]` 与 `0.026 [0.004,0.049]`。最大“仅混杂漂移”单元只有事件前 D 为 `-0.029 [-0.055,-0.003]`；事件前 J、combined 及全部事件后 contrasts 包含 0。所有九个时段交互区间包含 0。在最大“仅边变化”单元中，true-D 仍落后原始变量 EWS：事件前 `-0.048 [-0.085,-0.018]`，事件后 `-0.065 [-0.104,-0.035]`；J 差值为 `-0.271` 与 `-0.302`。
  - Signal: NEGATIVE（generic conditioning penalty、时段边界、双族成功和独立预警价值均失败）+ 局部正诊断信号（条件化有助于 D/J 检测真实边变化）。

- Claims and Claims matrix:
  - C1（D 对真实边变化的灵敏度）: 在当前固定目标边的 DGP 中，D 在全部 12 个非零真实边变化单元通过 family gate；最大“仅边变化”单元的条件化收益在事件前后均成立。
  - C2（J 的窄边界）: J 只在两个事件前的强边变化单元通过 family gate；不得用 combined AUC 掩盖其余 J 失败。
  - C3（一般损失与时段边界均不成立）: “仅混杂漂移”条件下只有事件前 D 出现小幅损失；J/combined 及事件后损失不成立，所有时段交互均为 null。
  - C4（无独立预警价值）: 所测估计器不稳定性均未超过完整原始变量 EWS；不得声称强对流预警增量。
  - C5（归因边界）: v3 的大差值不能归因于条件化；本结果不等于真实 DAG 恢复、物理分岔或外部天气有效性。

  | Outcome | 允许的 claim |
  |---|---|
  | REPLICATED SOURCE BOUNDARY | 只有在非退化空对照、非线性 DGP 与第二估计器都复现后，才可声称扰动来源决定条件化对 D/J 的方向；事件前后仍须单独检验。 |
  | CURRENT PARTIAL | 仅声称当前线性、固定目标边的 DGP 中 D 稳定、J 窄、条件化帮助检测真实边变化，而一般损失、时段边界和原始变量增量均不成立。 |
  | ESTIMATOR-SPECIFIC | 若第二估计器不复现，只报告 fixed-edge partial-correlation 的边界，不推广到 PCMCI+/LKIF。 |
  | NULL / ARTIFACT | 若非退化空对照失准或扰动来源差异消失，结论降为测量装置 artifact，停止真实数据阶段。 |

- Narrative: 论文问题不再是“因果不稳定性会不会预警”，而是“比较估计器不稳定性时，何时能把差异归因于条件化”。当前结果给出一个可复现的反证：匹配统计流程后，v3 的一般损失与事件前后不对称消失；条件化对真实边变化有小收益，但原始状态已经包含更多判别信息。
- Experiments: (1) 用独立但同分布的 event/control 构造非退化可交换空对照，校验 type-I、等价界和 calibration coverage；(2) 在非线性/local-independence DGP 上复现同一 fixed-edge D/J estimand，并用固定 parent set 的官方 PCMCI+ 或可匹配的第二估计器复核；(3) 只有前两步通过，才对去重风暴系统做 prospective power、严格 1/3/6/12 h 提前量、cluster bootstrap/BH-FDR，并比较 CAPE/shear、完整原始变量 EWS、资料质量和稳健 causal-change baselines。多 DGP/多事件只提供 empirical/diagnostic 证据，不构建 benchmark。
- Assets status: v4 GPU gate、锁定环境、SPC 样例和 HRRR smoke subsets 已就绪；完整环境场下载因停止门继续暂停，异常与交接只见 `workspace/causal-scs-indicator/data/MANIFEST.md`。

<review date="2026-07-11">

## Novelty

- Score: 4/10 (Claude 与 codex 独立评分一致)
- Closest prior work:
  1. **Rothenhäusler, Meinshausen, Bühlmann, Peters, Anchor regression** (arXiv:1801.06229) 与 **Kostin, Jalaldoust, Bareinboim, Kpotufe, Yang, 有限样本 minimax 上下界** (arXiv:2606.12680) — 本轮 Claude 独立读取全文 wiki 后新增的最强理论近邻 (codex 本轮据其自述未使用 arxiv-tools skill, 改用官方摘要/PDF, 未覆盖此线)。1801.06229 的 Theorem 1 + Section 2.3 证明: 扰动作用于 Y/隐混淆而非协变量 X 时, 非因果估计量一致地、可证明地支配因果参数; 2606.12680 用 Le Cam 两点法把这一 population-level 结论升级为有限样本匹配上下界, 且其 `cor:linear-scm` 给出 margin=Θ(t²) 的解析标度律。v4 pilot 观测到的"控制真实混杂对真实边变化有小幅收益、对纯混杂漂移无收益/小幅损失"这一定性模式, 与这条决策论边界完全同构。**机制不同故非直接撞车** (population-level/i.i.d. 分布位移 minimax 决策论 vs 时序滑窗 CI 检验有限样本功效; 两篇 wiki 笔记均已明确写"撞车无"), 但意味着当前发现在"形状"上是可预测的, 而非意外。
  2. VCDF/RCV-PCMCI (2410.19412/2602.21381)、conditioning-depth graph instability (2606.01214)、coherency score (2502.14719)、Causal-CCP (2403.12677, codex 补充) — idea 自身 Novelty quick-check 已列出前三项并做区分, Claude 与 codex 独立核验一致认为区分成立; Causal-CCP 作为"因果变点检测"最直接 baseline 仍应在下一版正面比较 (codex 指出)。
- Key differentiator: 方法组合本身不新——fixed-edge partial-correlation ablation + factorial 强度扫描是标准的 negative-control/backdoor-adjustment 设计 (参见 landscape "T. Matched negative controls" 节), 且当前 DGP 中 `z` 是完全观测、线性驱动 `x,y` 的干净共同原因, 控制 `z` 消除混杂漂移、提高真实边变化信噪比属于教科书式 adjustment 效应, 不是意外发现 (Claude 与 codex 独立得出同一判断)。若"disturbance-source 决定条件化方向"这一模式能在非线性/local-independence DGP、第二估计器和真实风暴数据上稳定复现, novelty 会转移到 finding 本身——即把一条已知的 population-level 决策论边界迁移并证实在一个从未被检验过的领域 (滑窗时序、临近分岔、CI 检验有限样本功效) 中依然成立, 这与 idea 自身 iteration-3 idea-scope deep-lit 发现的 local-independence 文献空白 (Didelez 2008 本人标记为未解决的开放问题, 19 年未被填补) 是一致的潜在贡献点; 但当前证据 (单一线性 DGP、单一估计器族、零真实数据) 不足以支撑这一转移。

## Quality

评估视角: topic (`topics/0710-causal-scs.md`) 未声明 `target-venue` 或 `preferred-contribution-types`, 也无 `## Target venues` / `## Review standards` 小节, 沿用 v1-v3 review 的推断, 按顶级大气科学 / Earth-system methods / AI4Science 视角评估 (Claude 与 codex 一致)。

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 5/10 (Claude 初评 6-7, codex 5, 采纳 codex 更具体的核验后合并偏严) | v4 确实修复了 v3 最严重的归因混淆: fixed-edge partial correlation 隔离 `conditioned_true` (真实混杂 `z`) vs `conditioned_sham` (独立 sham 变量 `u`), 同一条边、同一协变量数、同一窗口/校准, Claude 独立读代码 (`factorial_helpers.py`) 确认成立; window audit 确认 pre 窗口全部 `end<=920`、post 窗口全部 `start>=1080`, 不再跨越 switch 点; 8 seeds + 两阶段 (calibration+evaluation) 分层 bootstrap 已实现。但 codex 指出且 Claude 认为成立的核心限制: 当前 source-direction 很大程度上由 DGP 构造本身保证——`z` 是完全观测、线性驱动 `x,y` 的共同原因, 控制它消除混杂漂移、提高真实边变化信噪比并不意外, 更接近教科书 adjustment 演示而非对 PCMCI+ 式 CI 检验在真实假设违背下脆弱性的压力测试; pre/post 只是同一条平滑转变曲线在同一窗口几何下的时间平移, `event_time` 本身没有独立的物理/动力学含义, 因此"phase boundary 不成立"这一发现更接近装置的时间平移不变性 sanity check, 而非一般性结论。另外 Claude 独立核验发现一处小的报告不精确: JSON 中 phase-interaction 实际是 9 个 cell × 3 channel = 27 个区间, idea 正文"所有九个时段交互区间包含 0"是对 9 个 cell 的概括, 严格计数应为 27 个区间 (全部确认包含 0, 无虚报, 仅计数表述可以更精确)。 |
| Missing evidence signals | 4/10 (Claude 与 codex 基本一致) | 仍需: 非退化可交换空对照的 type-I/coverage、非线性/local-independence DGP、第二估计器 (LKIF)、真实风暴的 prospective power/storm-system 去重/严格提前量。codex 新增两点 Claude 认为成立: (a) 当前只有 9 个高度重叠窗口、8 个相邻差分, J 的 90th percentile 缺 window/stride/quantile 敏感性检验; (b) 400 次 bootstrap 得到的 `bootstrap_p_two_sided=0` 应报告为有限分辨率上界 (如 `<1/400`), 而非精确零值, 且未做 multiplicity 校正 (9 cells × 3 channels × 多个 contrast 类型)。此外 Claude 独立指出两处此前分析未覆盖的缺口, 直接对应 dispatcher 本轮特别要求 (见下方"特别核查"节): (c) 完全未纳入 iteration-3 idea-scope deep-lit 发现的 anchor-regression/minimax 决策论理论支柱, 本可用于把"为什么条件化在这个方向上有效"从经验观察升级为理论驱动的可检验预测; (d) 真实数据阶段的实验设计 (Experiments 段) 只有"验证预警增量"这一条路径, 完全没有面向"可能的领域新发现"的探索性分析协议。 |
| Narrative | 6/10 (Claude 与 codex 一致) | "matched-attribution audit" (归因审计) 框架比 v3 的 causal-conditioning-penalty 故事诚实得多, 也如实保留了 raw EWS 全面占优的负结果, 是本轮最大的叙事进步。但如 codex 所指, 正文仍同时承载"合成统计审计"与"强对流预警"两条相距很远的故事, 当前最可信的结果其实是"匹配 estimand 和推断流程后, v3 的大幅 penalty 与 phase asymmetry 消失, 剩余收益很小、符合预期、且没有预警增量"这一更朴素的陈述。此外 Claude 独立指出: 叙事完全没有借力 iteration-3 deep-lit 发现的 anchor-regression/minimax 理论支柱 (本可以把当前发现从"我们观察到一个不对称"升级为"我们检验一条已知决策论边界是否在滑窗时序临近分岔场景中依然成立"), 也没有回应 topic 自身动机 ("这种不稳定性可能携带缺失过程参与系统演化的信息") 中隐含的发现导向框架, 见"特别核查"节。 |
| Venue contribution | 3/10 (Claude 采纳 codex 更严格的判定, 初评 4) | 当前只有一个四变量线性 VAR、一个 fixed-edge partial-correlation family、零真实天气证据。它验证了一套审计流程的自洽性, 但没有新方法、新理论、外部发现或优于现有 diagnostics 的实证结果。作为内部 stop/go gate 有价值, 作为顶级 Earth-system/AI4Science 投稿远远不足, 且比 v3 更清楚地暴露了"当前故事是'我们修好了自己的实验设计缺陷'而非面向外部的新知识"这一局限。 |
| Testability | 8/10 (Claude 初评 7, codex 9, 采纳 codex 的具体核验后上修) | codex 指出且 Claude 独立核验确认: workspace git log 显示预注册 commit `bce72b0` (2026-07-11 23:02:31) 早于结果 commit `6333d6a` (2026-07-11 23:14:41) 约 12 分钟, 且 `results/factorial_pilot_results.json` 的 `runtime.git_commit` 字段正是该预注册 commit hash——预注册纪律有明确的时间戳与哈希证据支撑, 不是自述。Expected outcome 给出的两个候选最便宜证伪 (非退化空对照失准; 第二估计器不能复现真实边变化下的收益) 顺序合理, 前者确实更便宜。 |
| Outcome realism | 5/10 (Claude 初评 7, codex 5, 采纳 codex 后下修) | 当前 pilot 的窄、负面结果描述现实且守纪律 (数字经 Claude 独立重跑 JSON 核验无虚报, 见下)。但如 codex 指出, 顶会路线最终依赖真实数据上出现目前没有任何证据支持的"排序反转"——最大 edge-only cell 中 true-conditioned D 仍落后 raw EWS 约 0.05-0.07, J 落后约 0.27-0.30, 3 轮迭代 (v2 Stage 1 GPU proxy、v3 Stage 1.5 真实 PCMCI+、v4 factorial pilot) 中 raw EWS 无一例外全面占优, 这是当前最大的、尚无缓解迹象的经验风险。 |
| Contribution type compliance | n.a. | idea types = {empirical-finding, diagnostic}; topic 未声明 `preferred-contribution-types`, 跳过, 不触发 hard cap (Claude 与 codex 一致)。 |
| Overall Quality | 5/10 (Claude 与 codex 一致; 6 项计分维度均值 5.0-5.2, 取整 5) | v4 是一次真实、可信的执行改进: fixed-edge 隔离设计、多 seed、两阶段 bootstrap、无窗口泄漏、预注册时间戳可核验, 均较 v3 (Overall 4/10) 有实质提升。但当前"发现"本质是一个匹配后消失的 artifact 排除 (v3 的大差值被证伪) 加一个教科书式 adjustment 效应, 缺乏新方法/新理论/外部数据支撑, 也未按 dispatcher 本轮特别要求利用已有的强理论支柱或设计面向真实数据的发现导向分析——比 v3 更干净, 但顶会实质贡献的缺口并未真正缩小。 |

## Contribution Drift (n >= 2 only; n=1 写 N/A)

- v_{n-1} (v3) contribution types: {empirical-finding, diagnostic}
- v_n (v4) contribution types: {empirical-finding, diagnostic}
- Status: unchanged
- Hard cap triggered: no (topic 未声明 `preferred-contribution-types`; REFINEMENT_LOG.md 与 idea 正文的 "Contribution drift note" 均明确声明未新增/删除任何类型; Claude 与 codex 独立核验一致)

v3 review concern-by-concern 复核 (Claude 与 codex 独立复核后合并, 二者判定高度一致):

| v3 concern | Status | Assessment |
|---|---|---|
| 两个 regime 同时改变大量因素 (多因子混杂) | resolved | 已改为单一四变量 time-varying VAR skeleton, 仅 phase × graph-strength × confound-shift 三因子系统变化, 其余 (窗口、检验统计量、DGP 骨架) 固定。 |
| PCMCI+ vs Pearson 未隔离"条件化" | resolved | 主对比改为同一 fixed-edge partial correlation, 分别条件化真实混杂 `z` 与独立 sham 变量 `u`, 协变量数相同, Claude 独立读代码确认。 |
| "六项预注册门槛不变"与实际存在性/增量拆分自相矛盾 | resolved | PILOT_PLAN_V4.md 明确将 v3 全部结果重新定性为 exploratory, v4 门槛在结果产出前 12 分钟即冻结提交 (git 时间戳独立核验)。 |
| Regime 2 部分窗口跨越 switch 点 | resolved | window_audit 确认 pre `end<=920`、post `start>=1080`, 满足 `event_time±lead=1000±80`, Claude 独立重算确认。 |
| 单 seed, 无法区分"差异"与"抽样噪声" | resolved | 8 独立 seeds, 报告 seed-level AUC 与分层 bootstrap 区间。 |
| bootstrap 未传播 calibration-null 抽样不确定性 | resolved | calibration 与 evaluation 均在每次 bootstrap 中重采样, 再做 seed 层重采样。 |
| 缺 fixed-edge 消融与效应量扫描 | resolved | graph-strength {0,0.2,0.4} × confound-shift {0,0.25,0.5} 平滑扫描, 含真 null cell。 |
| 正对照过于简单 (0→0.6 硬跳变) | resolved | 已替换为连续强度扫描, 最大 edge-only cell 不再是 trivial AUC=1。 |
| raw EWS 只用滚动方差, 未含 AC1 | resolved | 已用 variance+AC1 双特征复合 baseline, 但仍是 y-only 单变量 baseline, 非完整多变量 operational diagnostics (codex 指出的残留限制)。 |
| D/J 被 combined AUC 掩盖 (只有 D 过关却笼统称"存在性通过") | resolved | D/J/combined 分栏报告, D 在 12/12 非零 graph cells 过 family gate, J 仅 2/12, 未被掩盖。 |
| 跨方法 percentile 可比性、LKIF 未验证 | partially resolved | 当前 claims 已限定在 fixed-edge statistic family 内, 避免无依据跨方法泛化; LKIF 仍未实现。 |
| 真实阶段事件数/功效/storm dedup 缺失 | ignored (有据的阶段性推迟, 非遗忘) | 把大下载放在 synthetic gate 之后是合理的资源排序决策, 但不能消解顶会证据缺口, 下一版若通过 gate 必须重新提上日程。 |
| 适用边界图 framing | resolved | 已转为 disturbance-source (真实边变化 vs 混淆漂移) × phase 的归因审计, 比 v3 的两点对比更精确。 |
| 2606.01214/2502.14719 被 REFINEMENT_LOG 误报为已解决 | resolved | v4.md 正文 (Novelty quick-check 行) 现已实际包含并区分二者, Claude 与 codex 独立 grep/阅读核验一致, REFINEMENT_LOG.md 也主动承认了 v3 的误报。 |
| 更便宜的下一步应是多 seed 重跑而非 LKIF | resolved | v4 PILOT_PLAN 明确采纳此排序, 先完成多 seed/bootstrap 稳定 estimand, LKIF 押后。 |

未采纳建议的 pushback 复核 (Claude 与 codex 一致): "先稳定 matched estimand 再跑 LKIF"与"强对流暂不作为 headline"两条 pushback 均有据, 已在 PILOT_PLAN_V4.md 中给出明确理由并被本轮结果印证 (v3 的大差值确实被证伪, 若先跑 LKIF 会在一个已知有缺陷的对比协议上浪费实现成本)。但如 codex 指出, 这不能反过来支持"第二估计器或真实数据验证已经不再必要"这一言外之意——v4 body 本身未过度声称, 可接受, 但下一版若通过 synthetic gate 必须立即重新提上日程, 不得继续无限期推后。

## 特别核查：真实 4D SCS 环境数据应用与潜在领域新发现 (dispatcher 本轮特别指定核查项)

**结论: v4 未充分回应这一要求, 构成需要 v5 继续补齐的明确缺口。**

Dispatcher 在委托 v4 refine 时明确要求"需要考虑到方法在实际 SCS 多变量 4D 环境数据的分析应用以及可能的领域新发现", 且据 session memory 记录, 用户在授权自主迭代时进一步明确要求下一轮"push the next refine toward real SCS multivariate 4D environmental-data application and possible domain-level findings, leveraging the strong theoretical grounding iteration 3's deep-lit found"——即同时点名了 (a) 真实数据应用路径与 (b) 利用 iteration-3 deep-lit 发现的理论支柱两项具体要求。Claude 独立核查 v4.md 正文、`REFINEMENT_LOG.md` 与 `PILOT_PLAN_V4.md` 后确认两项均未落实:

1. **真实数据应用路径完全是验证导向, 没有发现导向的分析协议**: v4 正文 Expected outcome 与 Experiments 段落中, 真实数据阶段 (Experiment 3) 唯一规划的产出是"对去重风暴系统做 prospective power、严格 1/3/6/12 h 提前量、cluster bootstrap/BH-FDR, 并比较 CAPE/shear、完整原始变量 EWS、资料质量和稳健 causal-change baselines"——这是一套完整的"diagnostic 是否有预警增量"验证协议, 但没有任何一句话规划"如果 (D,J) 或 conditioning-penalty 在真实 HRRR 4D 场上呈现某种空间/时间结构, 应如何反过来解读为一种此前未知的大气过程线索"。topic 文件 (`topics/0710-causal-scs.md`) 自身的研究动机原文写道: "这种由假设违背导致的效应不稳定, 可能携带了'缺失过程'参与系统演化的信息"——这是一个明确的发现导向假设, 但 v3、v4 两版均已将其收缩为纯粹的"预警增量是否存在"这一验证问题, 未见任何探索性设计 (例如: 按 storm mode/region/season 分层比较 conditioning-penalty 的空间分布是否对应已知或未知的对流触发机制差异)。
2. **iteration-3 deep-lit 发现的强理论支柱未被纳入**: idea-scope deep-lit iteration 3 (见 v3.md 末尾 `<deep-lit-integration>` 区块) 明确发现了两组高价值理论支柱——(i) anchor-regression/IRM/minimax 决策论边界 (1801.06229/2606.12680/2502.02710/2010.05761 等 7 条独立技术路线), 精确刻画了"条件化何时帮助、何时被非因果方法碾压"这一现象的一般规律; (ii) local independence 文献空白 (Didelez 2008 本人标记为开放问题, 19 年未解决)。这两条本可以把 v4 的 Novelty quick-check、Strongest objection 或 Narrative 段落从"我们观察到一个不对称"升级为"我们检验一条已知决策论边界/一个已知理论空白是否在滑窗时序、临近分岔场景中依然成立"——但 Claude 独立 grep 核验确认, v4.md 正文完全未出现这些 arXiv ID 或与之对应的论证。

Codex 本轮的 second opinion 未独立覆盖此项 (codex 未使用 arxiv-tools skill, 未读取这些 wiki 笔记, 也未看到 session memory 中用户的具体措辞), 因此这是 Claude 本轮独立提出、且有两处独立证据 (dispatcher 当前指令 + session memory 记录的 v4 委托原文) 支撑的核查结论, 不依赖 codex 佐证。建议 v5 refine 必须同时补齐这两点, 而不能仅继续在 synthetic gate 内打磨设计——否则会连续第二轮 (v3→v4→v5) 回避 dispatcher 明确交办的任务。

## Alternative Framing

三个方向, 均不引入 preferred-contribution-types 之外的贡献类型 (topic 未声明该字段, 故均不受 hard cap 约束), 优先级从低到高:

1. **codex 提议 (最小改动)**: 收紧为"causal-instability indicator 的 matched-attribution audit"——复现一个看似很大的 conditioning penalty, 再证明它在固定 estimand/条件集维度/推断流程后消失, 定义什么证据才允许把滑窗不稳定性归因于真实机制变化。仍是 empirical-finding+diagnostic, 能改善叙事诚实度, 但如 codex 自己指出, 仅审计自己早期 pilot 不足以改变顶会判断。
2. **Claude 提议 (理论锚定)**: 借力 iteration-3 deep-lit 发现的 anchor-regression margin 标度律 (`cor:linear-scm`, Δ=Θ(t²)), 把当前离散的 3×3 强度网格改造为单旋钮连续扰动强度扫描, 显式检验观测到的 AUC gap 是否遵循这一已知的决策论标度律——把"我们发现了一个不对称"升级为"我们检验一条已知有限样本 minimax 边界是否迁移到滑窗时序临近分岔场景", 理论落点更锋利, 且直接回应 codex 对当前"DGP 构造保证了方向"这一批评 (标度律本身就预测了方向, 关键是检验其定量形式是否也成立)。
3. **Claude 提议 (直接回应 dispatcher 本轮要求, 优先级最高)**: 把真实数据阶段从纯验证 (是否超过 raw EWS) 扩展为验证+探索并重——在通过 synthetic gate 后, 除既定的预警增量检验外, 增加一个探索性分析臂: 按已知 storm mode (superstorm/mesoscale convective system/discrete supercell 等)、区域、季节分层, 检验 conditioning-penalty 的"edge-change-dominant" vs "confounder-drift-dominant"分布是否对应已知或未被文献记录的对流触发机制差异, 直接回应 topic 自身"不稳定性可能携带缺失过程信息"这一原始动机, 也是 dispatcher 本轮明确要求但 v4 未落实的部分。

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 仅能声称: 在这一线性、稳定、完全观测混杂的 fixed-edge VAR 中, 条件化真实混杂对强边变化的 D/J 判别 AUC 相对等维 sham 条件化有小幅收益 (D 在 12/12 非零 graph cells 过 family gate, J 仅在两个 pre/强变化 cells 过 gate)。不得声称一般性 causal-discovery 优势、物理机制识别或强对流预警价值, 也不得声称已利用 iteration-3 发现的理论支柱验证了其在此场景下的迁移性 (因为尚未这样设计实验)。 |
| NULL | 可声称"在预注册精度下未检测到 phase interaction"(27 个区间全部包含 0), 以及混淆漂移场景下多数 J/combined contrasts 未过 gate; 不能把包含 0 的区间写成严格等价或效应不存在, 机械 null cell (event≡control) 只能证明实现自洽, 不能证明真实 type-I 校准。 |
| NEGATIVE | 可声称 generic conditioning penalty、phase boundary、双族稳定成功和独立于 raw EWS 的预警增量均未获支持; 在全部非空 estimator/channel-minus-raw contrasts 中点估计与区间均偏向 raw EWS。不能外推为所有因果不稳定性诊断均无用, 也不能把 synthetic 负结果写成强对流领域的否定结论。 |

## Likelihood-Impact Matrix

- Priority: Medium = Likelihood: Low x Impact: High
- Numeric score for ideas.xml: 5
- Rationale:
  - Likelihood: Claude 与 codex 一致判定 Low, 无分歧。理由: v4 已使实验可信 (fixed-edge 隔离、多 seed、两阶段 bootstrap、无窗口泄漏、预注册时间戳可核验), 但可信结果本身是"小而符合预期的条件化收益 + 无 phase boundary + raw EWS 全面胜出"。达到顶会仍需要非退化空对照、非线性/local-independence DGP、第二估计器和真实风暴严格提前量增量依次通过——尤其最终需要目前没有任何证据支持的真实数据"排序反转"(3 轮迭代中 raw EWS 无一例外全面占优); 加上 dispatcher 本轮指出且 Claude 独立确认的两个新缺口 (理论支柱未纳入、真实数据发现导向设计缺失), 需要补齐的工作量不减反增, 不是常规工程补强可以完成的, 故为 Low 而非 Medium。
  - Impact: Claude 与 codex 一致判定 High, 无分歧。理由: 若最乐观情形成立——disturbance-source 决定条件化方向这一模式在多 DGP、多估计器、真实强对流数据上都稳定复现, 且不稳定性诊断在严格提前量上提供 raw/operational baseline 之外的增量——会为"因果诊断目标 vs 预测/forecasting 目标何时不一致"这条活跃研究线提供一个从未在滑窗时序/临近分岔场景中被检验过的清晰边界, 具备发表价值和叙事影响力; 但方法论工具 (fixed-edge 隔离、negative-control 设计) 和其理论解释 (anchor-regression/minimax 决策论边界) 均已存在于相邻文献, 问题域也相对窄, 不到 Exceptional (不会颠覆性开辟新方向或推翻强共识)。
  - 查表: Low x High = Medium (5)。topic 无 `preferred-contribution-types` 声明, hard cap 不适用, numeric score 不截断。与 v2/v3 review 的 Low×High=5 结论一致, 但推导路径已从"假设性推断"→"部分实证 (v3)"→"更干净但更朴素的实证 (v4)": 证据质量在提高, 但顶会门槛与经验风险 (raw EWS 全面占优) 均未松动。

## Overall

- Priority: Medium
- Score: 5
- Comments: v4 是一次真实、可信的执行改进——fixed-edge 隔离设计、多 seed、两阶段 bootstrap、无窗口泄漏、预注册时间戳可独立核验 (git log 确认 commit 顺序), Overall Quality 由 v3 的 4/10 提升到 5/10, Claude 与 codex 独立评分高度一致, 无 >=1 level 分歧。但 v4 的核心新"发现"——控制真实混杂对真实边变化有小收益、对纯混淆漂移无收益——本质上是一个匹配后消失的 v3 artifact 排除加一个教科书式 adjustment 效应, 其方向已被 anchor-regression/minimax 决策论文献 (Claude 本轮独立核验, codex 未覆盖) 预测, 不是意外结果; 更重要的是, v4 完全没有回应 dispatcher 本轮明确交办的两项任务——(1) 真实 4D SCS 数据应用路径的探索性/发现导向设计, (2) 利用 iteration-3 deep-lit 发现的理论支柱——这是本轮独立确认的新缺口 (见"特别核查"节), 建议 v5 must-fix, 而非继续在 synthetic gate 内打磨。Likelihood-Impact 数值与 v2/v3 一致 (Low×High=5), 因为经验风险 (raw EWS 全面占优) 与顶会证据缺口均未随本轮改进而缩小。

</review>
