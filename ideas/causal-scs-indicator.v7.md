<!-- 书写报告使用中文 -->
---
topic: topics/0710-causal-scs.md
landscape: topics/0710-causal-scs-landscape.md
workspace: workspace/causal-scs-indicator/
---

- One-sentence summary: 强对流环境预警探索，检验把 (D,J) 因果时序不稳定性诊断 (PCMCI+/LKIF/fixed-edge partial correlation 均可作载体) 融合进 raw-state EWS 特征后，能否得到优于"只用 raw EWS"的预警信息；因果方法本身是操作化工具，不是本轮创新落点。
- Hypothesis: (D,J) 单独与 raw EWS 单独的竞赛 (v1-v6，raw EWS 6/6 全胜) 是错误的比较框架，真正的问题是二者能否互补。预期：当 (D,J) 在某个扰动 regime 下确有真实标准信号 (即 v4-v6 已确认的 graph-only-max 格) 时，把它按恰当权重并入 raw EWS 可以拿到增量；但简单等权融合会用 (D,J) 的噪声稀释 raw EWS 的强信号，反而变差。当 (D,J) 本身只是 confounder-shift 伪影 (confound-only/mixed 格) 时，任何权重的融合都不该赢过 raw-alone。
- Expected outcome: 支持信号 = 存在一个经 held-out 拟合、有校准 CI 的加权/学习融合器，在 graph-only-max regime 上相对 raw-EWS-alone 取得 CI 下界 >0 的增量，且在 confound-shift regime 上不劣于 raw-alone。最便宜的证伪信号已经跑出：预注册的等权融合 (50/50) 在 16 格中 12 格 CI 完全为负 (含全部 confound-only/mixed 格)，其余 4 格 (graph-only-max) 2 格 CI 含零、2 格 CI 为负。等权融合从未显著赢过 raw-alone。
- Contribution type: empirical-finding+diagnostic+application
- Contribution drift note: v6 为 `empirical-finding+diagnostic`；v7 新增 `application`，因为 dispatcher 转达的 owner 澄清明确将本 idea 的核心价值定位为强对流预警的应用价值 (融合特征能否产出更好预警信息)，而非因果方法本身的新颖性。empirical-finding 与 diagnostic 均保留、无删除：diagnostic 支线 (估计器校准、跨方法方向一致性) 仍是判断"何时可以信任 (D,J) 特征去融合"的必要支撑证据，不是被砍掉的平行头衔。topic 未声明 `preferred-contribution-types`，三者并存不触发子集违规。
- Risk: HIGH
- Estimated effort:
  - Compute: v7 融合测试共 4.3s L4 (job 36909438)；held-out 融合器+扩展 null-calibration 预计 <1 GPU-hour；H2 真实 4D HRRR 融合测试约 10-30 GPU-hours、500-1500 CPU-hours (同 v6 估计)。
  - Data: HRRR/SPC smoke 资产已就绪；完整逐小时序列与雷达/地面边界标注仍受 stop gate 限制，本轮不下载。
  - Implementation: held-out 融合器预注册与实现 1 周；H2 标注与真实数据融合检验 4-6 周。
- Novelty quick-check: 方法新颖性非本轮重点，从简核查。Ganesh, Beucler et al. (2023, arXiv:2304.05294；2025 SHIPS 延伸 2510.02050) 用因果发现*筛选*特征后训练 ML 预测台风强度，是"因果衍生特征能否提升下游预测"最接近的先例，但其操作是剔除非因果特征，不是把因果不稳定性得分与既有特征加权融合，也不在强对流/滑窗设定下。未见工作把滑窗因果不稳定性诊断以加权/学习融合方式并入既有强对流 EWS 特征。
- Strongest objection: v7 唯一真正决定性的证据是"等权融合更差"这一负结果；探索性 alpha 扫描显示的"小权重融合可赢 raw-alone"完全没有 held-out 拟合和 bootstrap CI，可能只是对同一评估集的事后过拟合形状，不能证明存在真正泛化的融合器。且该形状本身建立在尚未对 graph-only-max 格做校准的 null-audit 基础上 (v6 只校准了 mixed 格)。
- Why we should do this: 无论结果如何都回答一个此前六轮从未问过的、更贴近 owner 目标的问题：因果时序特征能否*补充*而非*替代* raw EWS。若能，为强对流预警提供一个新的、可解释的融合指标；若不能 (等权已经是强证据)，则诚实地为"时序因果特征对这类预警任务几乎没有增量空间"提供一个干净的负面结论。
- Pilot:
  - Setup: 复用冻结的 v5 硬化 DGP (`factorial_helpers_v5.py`，未改动)，16 格 (phase×graph_strength{0,.4}×confound_shift{0,.5}×confound_observability{1,.5})，96 pairs/8 seeds/32 calibration/400 bootstrap，与 v4-v6 相同种子。新增 `fusion_helpers.py`：`raw_only` = raw variance+AC1 百分位均值 (与既有 `raw_ews`/`combined` 完全一致，作内建核对)；`fusion` = (conditioned_proxy 的 D,J + raw variance+AC1) 四个百分位分数等权均值；`delta_fusion_minus_raw` 用与 v4-v6 相同的层级 bootstrap 配对。预注册于 `PILOT_PLAN_V7.md`，commit `9c98283` 早于 GPU job 36909371/36909438 (L4，2.09-2.24s)。等权融合无需拟合/无过拟合风险，是本轮唯一的确证性预注册测试。
  - Metric: 决策规则固定于跑之前，primary cells = graph-only-max 两相 (D,J 唯一有真实标准信号的格)；两个 primary cell 的 Delta CI 下界 >0 记 POSITIVE；CI 含零记 NULL (不可判定，不能称"融合无效")；CI 上界 <0 记 NEGATIVE。v6 null audit 显示该 bootstrap 在混合格上过覆盖 (保守而非膨胀)，故决定性结果 (CI 完全在 0 一侧) 比校准修复前更可信，因为保守偏差让"恰好显著"更难由噪声产生。
  - Result: 等权融合 (预注册)：confound-only/mixed 共 8 个非空格，Delta CI 全为负 (如 pre/confound-only/obs1: −0.138 [−0.169,−0.110]；post/mixed/obs.5: −0.116 [−0.144,−0.085])。graph-only-max (primary，4 格)：pre 两格 CI 含零 (+0.0045/+0.0019)；post 两格 CI 全负 (−0.035、−0.048)。等权融合从未显著赢过 raw-alone。事后探索 (同日新增，非预注册，无 held-out/无 CI，仅描述性)：扫描 alpha·raw+(1−alpha)·causal，在全部 4 个 graph-only-max 格上最优 alpha 落在 0.7-0.9 且超过 raw-alone (+0.005 到 +0.021 AUC)；在全部 8 个 confound-only/mixed 格上最优 alpha 精确等于 1.0。只有 (D,J) 真有信号的格才可能被更细的加权救回，被污染的格无论怎么加权都不如 raw-alone，这个形状与既有机制解释一致。
  - Signal: NEGATIVE (等权融合，预注册结论)；探索性 alpha 扫描给出一个连贯、可解释、但未经校准的 WEAK 正面形状，值得作为下一步 held-out 融合器设计的依据，不构成独立证据。

<added-on-refine>
- Claims and Claims matrix:
  - C1: 等权融合 (D,J)+raw EWS 在全部 16 格中从未显著优于 raw-alone；12/16 格 (全部 confound-only/mixed) 显著更差；graph-only-max 的 4 格中 2 格 null、2 格显著更差。
  - C2: 探索性 alpha 扫描仅是描述性形状，不得称为"证明存在可泛化的融合增益"；其正向发现严格限定在 graph-only-max 格，且未做 held-out 拟合。
  - C3: v6 null-calibration 的过覆盖诊断只覆盖了 mixed 格，graph-only-max 格从未被独立校准，本轮的"决定性结果更可信"论证依赖一个尚未验证在 graph-only-max 格同样成立的假设。
  - C4: raw EWS 在全部非空 stress cell 上单独仍占优 (v1-v6 连续 6 轮 + 本轮再次确认)；LKIF 与 fixed-edge partial correlation 家族方向 3/4 相反、1/4 同向 (32 pairs，非决定性)。

  | Outcome | 允许的 claim |
  |---|---|
  | HELD-OUT POSITIVE | 只有在专门预注册的、held-out 拟合的加权/学习融合器上，graph-only-max 格 Delta CI 下界 >0 时，才可称"融合特征改进了预警信息"，且只在该 regime、该 DGP 上成立。 |
  | HELD-OUT NULL/NEGATIVE | 若 held-out 融合器仍不能赢，可诚实报告"时序因果不稳定性特征对该类合成 severe-convective-like 场景的预警任务几乎没有可提取的增量"，但不外推到全部因果特征或全部预警任务。 |
  | 当前状态 | UNCALIBRATED-EXPLORATORY：等权融合的 NEGATIVE 是确证性的；alpha 形状是假设生成的，禁止在 proposal 前当作证据引用。 |
- Narrative: 把 v6 "跨估计器可识别性审计" 的 framing 换回 owner 原始动机：因果时序特征能否改进强对流预警，而非估计器谁更可信。真实 4D SCS 数据重新升级为核心目标 (非 v6 的 held-out 证伪测试)：held-out 融合器一旦在合成数据上验证，下一步是在真实 HRRR/雷达数据上重复同一"融合 vs raw-alone"检验。H2 (D/J 空间梯度 vs 中尺度边界) 因此重新定义为该检验在真实数据上最便宜的切入点 (静态快照，不需长时间窗口基础设施)，而非独立于预警价值的边界探测分支：用 D/J 空间梯度与既有湿度/θe/风场梯度做同一套"融合 vs baseline-alone" F1 比较。方法论新颖性 (anchor regression/minimax) 保持 v6 定性类比措辞，不再深挖。
- Experiments: P1 (must-run，先于真实数据) 把等权换成 held-out 拟合的融合器：用不相交于评估集的一半 seeds/pairs 拟合 alpha 或岭回归权重 (先验偏向 raw-only)，另一半报告 CI；同时把 v6 的 exchangeable-null 校准扩展到 graph-only-max 格 (v6 review 专项核查 4 指出的缺口)，这是本轮"决定性结果更可信"论证成立的前提。P2 (待 P1 过门) 用同一 held-out 融合协议跑 PCMCI+/LKIF 版本的 (D,J)，检验结论是否只是 fixed-edge partial correlation 的特例。P3 (H2，待 P1-P2 过门) 定义 `S_DJ` 空间梯度，与既有湿度/θe梯度、风场辐合、开发集拟合原始场融合三个 baseline 做同一套 F1 比较，标注协议沿用 v6 已写定条款 (双标注者盲于 predictor、60-min lead、6-km tolerance、cluster bootstrap、95% CI 下界>0 才算增量)。H1 (storm-mode 分层) 继续保持探索项。
- Assets status: HRRR/SPC smoke 资产与 v7 融合结果均已就绪 (job 36909371/36909438)；完整数据获取仍受 stop gate 限制，未在本轮下载；细节见 `workspace/causal-scs-indicator/data/MANIFEST.md`。
</added-on-refine>

<review date="2026-07-12">

## Novelty

- Score: 3/10 (Claude 独立初评 4, 经 codex 提供的 arxiv 全文核验后下修为 3, 采纳 codex 判断)
- Closest prior work:
  1. **Ganesh, Beucler, DeMaria, Runge (2025), Multidata-PC SHIPS+** (arXiv:2510.02050) — Claude 用 `agon:arxiv-tools` 重新全文核验 (`main.tex`) 后发现, 该文核心实验实为**特征增强/融合**, 而非landscape 原条目所述的单纯"筛选": 明确构造 "SHIPS+" = 原 21 个 SHIPS 业务预报因子 + 新发现的因果筛选预报因子, 直接对比 "original SHIPS" 与 "SHIPS+" 模型的 test R^2, 并在独立真实 2022-2024 北大西洋 TC 个例上做业务化验证 (相对既有业务基线的 MAE 降幅), 全部预报时效均有提升。这是"因果衍生特征增强既有强 baseline 能否优于 baseline-alone"这一确切问题在真实数据、真实技能指标下的既有先例, 且作者组与 landscape 已有条目重叠 (Beucler)。已追加更正到 `topics/0710-causal-scs-landscape.md` ([idea-reviewer, 2026-07-12], append-only)。
  2. **Yu, Liang (2026), ST-CND** (arXiv:2606.17553, 本轮新发现, 已入 landscape) — 最近邻的"融合"先例, 但其融合 (因果网络特征 + 传统 EWS/RQA 统计量联合逻辑回归) 明确标注"仅用于可解释性分析", 不是与四个基线对比的 headline 结果; 领域是地理/空间临界点 (SST/AMOC), 非强对流。
- Key differentiator: 加权/学习融合机制本身完全不新 (v7 故意选用最简单的等权平均, 且明确降低方法新颖性权重); 潜在差异仅剩滑窗因果不稳定性 (D,J) 这一特征构造方式本身、以及强对流 (而非热带气旋/地理临界点) 这一具体域。"用因果衍生特征增强既有强 baseline, 检验增强后是否优于 baseline-alone"这一**问题框架**已经在紧邻域 (TC 强度预报) 用真实数据、真实业务技能指标验证过 (SHIPS+), 不能再算作本 idea 的独立新颖贡献; "Applying X (D,J 融合) to Y (severe convection)" 若无意外洞见不构成顶会新颖性, 目前也未产生意外洞见 (等权融合失败、事后 alpha 形状均属预期内的直觉解释)。

## Quality

评估视角: topic (`topics/0710-causal-scs.md`) frontmatter 与正文均未声明 `target-venue`/`preferred-contribution-types` (Claude 与 codex 独立核实一致)。鉴于 owner 澄清将核心问题转向"强对流预警应用价值", 本轮视角相应从 v1-v6 的"因果推断方法学"稍微偏移至"顶级 Earth-system methods / AI4Science / 大气预测应用"稿件标准 (Claude 与 codex 一致)。

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 3/10 (Claude 初评 5, 经 codex 更细致的拆解后下修采纳 3) | (a) **数字错误**: idea 的 "Expected outcome" 行与 Claims C1 均称"16 格中 12 格 CI 完全为负 (含全部 confound-only/mixed 格)", Claude 与 codex 独立从 `results/fusion_pilot_results.json` 重算后一致得出正确值为 **10/16** CI 严格为负 (8 个 confound-only/mixed + 2 个 post/graph-only-max), **6/16** CI 含零 (2 个 pre/graph-only-max 真实空 + 4 个 g=0,h=0 完全退化格, 后者 delta 恒为 0, 根本未施加任何扰动, 不应被计入"负"这一桶)。git 提交 `c5da5c6` 的 commit message 给出第三种、同样错误的表述 ("null in the remaining 4 graph-only-max cells", 但实际 4 个 primary cell 中只有 2 个是 null, 另 2 个 post 相significantly negative)。idea 自身更靠下的 Result 明细段落与 `EXPERIMENT_LOG.md`/`README.md` 的表格反而是准确的 (10/6 拆分), 与文件自己的头部摘要、Claims 段自相矛盾。定性结论 (等权融合从未显著赢过 raw-alone, 0/16 格 CI 下界 >0) 不受影响, 但这是与 v4-v6 review 反复抓到的同类"小数字失准"问题同源的新一例。(b) **决策规则表述与自身定义不符**: `PILOT_PLAN_V7.md` 与 idea Metric 行都把 primary cells 定义为 "graph-only-max, both phases, obs∈{1,.5}" (=4 格, 与 JSON 中 `is_primary_cell=true` 计数一致), 却又说 "两个 primary cell"/"both primary cells" 的 CI 判定规则, 数字上暗示只有 2 格。(c) **等权失败并不证伪主假设** (codex 提出, Claude 认同): Expected outcome 的正面支持信号是"存在某个经 held-out 拟合的加权/学习融合器能赢", 而已跑的确证性检验只是 50/50 这一个特定权重, 逻辑上只能证伪"等权有效", 不能单独作为主假设的判定性反例, idea 文本对此边界交代不够清晰。(d) alpha 扫描存在双重选择偏差 (codex 提出, Claude 核实成立): graph-only-max 本身是此前 6 轮里按 (D,J) 表现挑出的"有利格", 再在同一评估集上扫 alpha, 叠加了两层事后选择, 不只是"无 held-out"这一层已披露的局限。(e) confound-only/mixed 格 "最优 alpha 恰为 1.0" 被 idea 叙事完全归因于"(D,J) 被 confounder 污染", 但这些格 raw baseline AUC 已高达 0.985-0.998, 天花板效应本身也足以解释"任何非 raw 权重都不会更好", 两种机制未被区分。 |
| Missing evidence signals | 2/10 (Claude 初评 3, 采纳 codex 更严格的 2) | 沿用 v4-v6 连续未解决的缺口 (全规模 PCMCI+/LKIF、Causal-CCP baseline、window/stride/J-quantile 敏感性、bootstrap multiplicity 校正、graph-only-max 格独立 null 校准) 之外, 本轮新增了尚未偿还的欠账 (held-out 拟合融合器、主要 cell 聚合判定规则、模型选择冻结/equivalence-futility gate); codex 进一步指出仍缺一个"同输入同容量的非因果时序特征 baseline"和"强 raw-history/operational baseline"——这在拟议的真实数据阶段 (H2/P3) 会是决定性的对照组, 目前完全没有设计。真实数据仍是 0% (仅 3 天 SPC 报告 + 3 个 HRRR 快照做 schema smoke test, MANIFEST 明确写"not used to claim pilot skill")。工程纪律上还有一处新退步: v4/v5/v6 每轮新增的 helper 模块都配有对应 `tests/test_*.py`, 而 v7 新增的 `fusion_helpers.py` 是第一个没有配套单元测试的模块 (git log 核实), idea 中未披露此退步。 |
| Narrative | 5/10 (Claude 初评 6, 结合 codex 指出的 P3 端点错位后下修为 5) | 相对 v6 有真实、实质的改善: headline、Strongest objection、P1 都切实转向"融合能否带来增量"而非"谁的估计更可信", 不是简单换标签。但 (a) P3 的实际 estimand 是中尺度边界 (outflow/dryline) 的 F1, 这本质上是一个边界探测/诊断任务, 不等价于"提前预测冰雹/龙卷/大风等强对流灾害"这一 owner 真正关心的预警问题, narrative 里"H2 是融合检验在真实数据上最便宜的切入点"这一定位与其实际 estimand 之间存在错位 (codex 指出, Claude 认同); (b) "UNCALIBRATED-EXPLORATORY" 与"等权融合的 NEGATIVE 是确证性的 (decisive/confirmatory)"两个表述在同一 Claims 段并存, 口径存在张力——calibration gap 被恰当地当作必要支撑性 caveat 而非重新变成审计主线是对的, 但"未校准"与"确证性"两个词同时用于描述同一结果, 需要更精确的措辞。 |
| Venue contribution | 2/10 (Claude 初评 3, 采纳 codex 更严格的 2) | 即使按应用价值 venue 的更宽松标准衡量, 本轮新增的全部证据是"一个任意选择的 50/50 组合在合成 DGP 上失败"加上"同一评估集上一个事后 alpha 扫描的小幅增益"——没有方法创新、没有真实数据证据、也没有一个可推广的意外发现, 不足以构成任何顶级 venue 的独立贡献, 即便是 workshop track。 |
| Testability | 5/10 (Claude 初评 7, 采纳 codex 指出的关键缺陷后下修为 5) | Expected outcome 确有一个已经跑出的最便宜信号 (预注册等权融合), 且 Claude 独立复算确认其方向可信 (raw_only 与既有 raw_ews 逐格 bit-exact 一致, alpha=0.5 与等权结果一致, prereg commit 严格早于 GPU job)。但 codex 指出的问题成立: 这个已跑测试只回答了"50/50 是否有效", 并未给出"是否存在某个更优权重"这一真正主假设的判定性检验; decision rule 缺一个跨 4 个 primary cell 的联合聚合规则、缺最小实际效应阈值、缺模型选择冻结。这些缺口使"cheapest falsifying signal"的覆盖面比 idea 文本暗示的要窄。 |
| Outcome realism | 4/10 (Claude 与 codex 一致) | raw EWS 在全部 16 格标准竞赛中连续 7 轮 (含本轮再确认) 保持独立占优; 即便最乐观地假设 held-out 融合器在 graph-only-max 格站得住, 距离"real HRRR/radar 数据上真实验证的预警增量"仍需要多个条件依次成立 (held-out 融合器胜出 AND 通过 graph-only-max 校准 AND 跨 PCMCI+/LKIF 复现 AND 真实数据迁移成功), 且 confound-only/mixed 格的 raw AUC 天花板效应可能进一步压缩现实中能拿到的增量空间。 |
| Contribution type compliance | n.a. | idea types = {empirical-finding, diagnostic, application}; topic 未声明 `preferred-contribution-types`, 跳过, 不触发 hard cap (Claude 与 codex 一致)。 |
| Overall Quality | 4/10 (Claude 独立初评 (5+3+6+3+7+4)/6≈4.67→5, 采纳 codex 更严格的拆解后合并为 4/10; codex 独立给出 4/10, 一致) | reframe 方向正确、诚实纪律总体延续 (等权 NEGATIVE 结果被准确执行、alpha 扫描被恰当标注为非确证性), 但本轮新增了一组真实的数字/表述失准 (12 vs 10, primary cell 计数), 证据仍完全停留在同一合成 AR-skeleton 装置上, 且 P3 端点与"预警"目标之间出现新的错位。 |

## Contribution Drift (n >= 2)

- v_{n-1} (v6) contribution types: {empirical-finding, diagnostic}
- v_n (v7) contribution types: {empirical-finding, diagnostic, application}
- Status: expanded(+application)
- Hard cap triggered: no (topic 未声明 `preferred-contribution-types`, Claude 与 codex 独立核验一致)

扩张评估 (Claude 与 codex 独立收敛一致): drift note 给出的理由 (dispatcher 转达的 owner 澄清明确将核心价值定位为强对流预警应用价值) 是**站得住的、有据的**理由, 不是 refiner 自行编造的 scope creep, 因此不构成"越界扩张"或"silent downgrade"意义上的违规。但这一扩张**目前尚未被本轮证据挣得**: 本轮全部实验依旧在与 v4-v6 完全相同的合成 AR-skeleton DGP 上运行 (`factorial_helpers_v5.py` 未改动), 零真实 HRRR/SPC/radar predictor-label pair 被使用。更值得注意的是 (codex 独立发现, Claude 核实属实): 本轮自己的预注册文档 `workspace/causal-scs-indicator/PILOT_PLAN_V7.md` 在"Frozen decision rule"一节明确写着 "**Contribution scope unchanged: `empirical-finding + diagnostic`**. This is application-value evidence for the reframed core question, not a new method, estimator, or benchmark" ——也就是说, 本轮实验设计本身仍然认为贡献类型集合"unchanged" (不含 application), 而 idea.md 的 "Contribution type" 字段却已经声明为三元集合。这是本轮产出内部的一处真实不一致, 支持"owner 的动机澄清可以改变研究目标, 但不能自动生成 application 贡献"这一判断——建议下一版要么把 idea.md 的贡献类型改回与 PILOT_PLAN_V7.md 一致的二元集合 (更诚实), 要么明确说明为何 idea.md 层面提前声明了 PILOT_PLAN 尚未追认的类型。

### v6 `<review>` concern-by-concern 复核 (Claude 与 codex 独立复核后合并)

| v6 concern | Status | Assessment |
|---|---|---|
| "跨估计器可识别性" headline 措辞过强 | resolved | v7 完全放弃该 headline, 改为 fusion-vs-alone 问题, 该批评的具体对象已不存在。 |
| graph-only-max 格从未被独立 null 校准 (v6 专项核查 4) | acknowledged, not resolved | v7 明确重申此缺口并列入 P1, 但零执行; 用 4 秒的方向性 pilot 结果作为"决定性结果更可信"的支撑论证在 Claude 与 codex 看来都略显宽松——mixed 格的 overcoverage 校准来自不同 cell、不同 estimand, 尚未验证同一保守偏差方向在 graph-only-max 格(信噪比迥异)上是否依然成立。 |
| 全规模 PCMCI+/LKIF、Causal-CCP baseline (v3→v6 连续 ignored/deferred) | ignored/deferred (第 4-6 轮) | 推迟到 P2, 有据 (需先有 held-out 融合协议), 但零新执行; 目前不能称三种 carrier "interchangeable"。 |
| window/stride/J-quantile 敏感性 (v2→v6 连续 ignored) | ignored (第 6 轮) | 仍只是 P1 文字承诺, 无低成本试点执行。 |
| bootstrap multiplicity 校正 (v2→v6 连续 ignored) | ignored (第 6 轮) | 同上, 且新增了跨 4 个 primary cell 的聚合判定规则缺失问题。 |
| H2 标注协议缺口 (一致性阈值/仲裁/功效分析/baseline 模型细节, v6 专项核查 5) | ignored | v7 未在这些具体缺口上补充任何内容, 协议原样继承。 |
| v6 review 自己建议的 Alternative Framing ("calibration stress-test + estimator-specific failure map") | superseded, 有据 | 被 owner 澄清替换为 fusion-vs-alone 框架; 有明确、更高优先级的理由记录在 `REFINEMENT_LOG.md` 的"Direction from owner"表中, 接受。 |
| v6 review 的"收敛判断" (建议不再 refine, 直接执行 P1 全规模复现) | 未遵循, 但有据 | owner 的动机澄清构成合理的 anchor reset 理由, 接受这一次偏离; 但需指出: 促使 v6 建议收敛的证据缺口 (全规模复现、校准修复) 一个都没有缩小, 反而新增了 held-out 融合器与 graph-only-max 校准两项。Claude 与 codex 一致认为: 从现在起再做 v8 式的 idea 文字打磨边际收益已经很低, 下一步应是执行, 而非继续 refine。 |
| raw EWS 连续占优, 无真实数据缓解 | unresolved, 诚实保留 | 第 7 轮 (v1-v6 + 本轮) 仍无一例外, idea 未回避这一事实。 |

未采纳建议的 pushback 复核 (Claude 与 codex 一致): 放弃 v6 的 identifiability headline、把 anchor regression/minimax 理论支线降为次要, 均有据, 接受。把等权融合结果在 graph-only-max 校准缺口尚未修复前当作"低成本方向性 pilot"来跑, 亦可接受。但把相邻 mixed-cell 的 overcoverage 当作对 graph-only-max fusion-delta estimand 同样成立的确认性保护、以及继续把 window/stride/multiplicity/H2 功效分析全部搁置, 均缺乏充分依据, 本轮重新强调为 must-fix。

## Alternative Framing

Claude 与 codex 收敛到互补但不完全相同的建议, 以 codex 的端点级建议为主, Claude 的机制级建议为补充: **(1, 主) 把主线 endpoint 收紧为**: 在独立 storm systems 上, 明确 lead time 下, (D,J) 工程特征能否在同输入、同容量的强 raw-history/operational baseline 之上, 提供达到预设最小实际效用阈值的增量强对流灾害 (冰雹/龙卷/大风) 预测或决策价值, 以及该增量在哪些 regime 消失; 中尺度边界 F1 (当前 P3/H2 的实际 estimand) 降级为机制性次要端点, 不再作为"预警检验"的主载体, 从而修复本轮 Narrative 中指出的端点错位。**(2, 补充) 机制上**, 与其用固定 alpha 或全局拟合权重做融合, 不如让 v6 已建立的诊断/校准机制本身成为融合器的一部分 (例如用校准后的 confound-contamination 信号做自适应/门控加权, 而非常数或全局拟合的 alpha), 这样"diagnostic"与"application"两个贡献类型会通过融合机制本身直接耦合, 而不是并列但松散的两条线。这两点都不需要引入 topic `preferred-contribution-types` 之外的类型 (topic 未声明该字段)。

## Claims Discipline

采纳 codex 更精确的表述 (Claude 核实与原始 JSON 一致):

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 只有在预注册的、嵌套 held-out storm-system/独立 DGP 评估中, 增强模型相对同输入同容量 raw-only 模型的增量 CI 下界超过预先设定的最小实际效用阈值, 且校准/决策指标同步改善, 才能声称在指定数据、灾害类型、lead time 和模型类别下存在增量预警技能。不得声称"产生了新信息"、"识别了因果机制"或"普遍改进了强对流预警"。 |
| NULL | CI 含零只说明当前设计无法判定, 除非预先定义等效边界 (equivalence margin) 且功效充足、CI 完全落入该边界内, 否则不能称"融合无效"或"没有增量"。 |
| NEGATIVE | 当前只能确凿声称: 在该 AR-skeleton 合成 DGP、fixed-edge partial correlation 代理和 50/50 等权规则下, 16 格中 10 格 CI 严格为负、6 格 CI 含零 (其中 4 格为完全退化的无扰动格); 4 个 graph-only-max 主格中 2 个显著更差、2 个含零。不得外推到经过优化的融合器、PCMCI+/LKIF、真实强对流数据, 或任何其他因果衍生特征。若未来 held-out 融合器的增量 CI 上界仍低于预设最小实际效用阈值, 才可在此明确范围内声称"该类因果不稳定性特征对该预警任务几乎没有可提取的增量"。 |

## Likelihood-Impact Matrix

- Priority: Medium = Likelihood: Low x Impact: High
- Numeric score for ideas.xml: 5
- Rationale:
  - Likelihood: Claude 与 codex 独立评估完全一致, 判定 Low, 无分歧。理由: 唯一已确认的新结果是"任意 50/50 融合失败"这一否定性发现; 正面信号仅是同一评估集上的事后 alpha 增益 (+0.005 至 +0.021 AUC), 不构成独立证据。要走到 top-venue 级结果, 需要依次: 完成 held-out 拟合融合器的预注册检验、完成 graph-only-max 格的独立 null 校准、跨 PCMCI+/LKIF 复现结论、构建并击败同输入同容量的非因果/operational baseline、并最终在真实 storm systems 上达到预设实际效用阈值——条件连乘, 且 raw EWS 优势连续 7 轮无缓解迹象、confound-only/mixed 格的 raw AUC 天花板效应可能进一步压缩现实增量空间。Likelihood 维持 Low, 与 v1-v6 一致。
  - Impact: Claude 与 codex 独立评估完全一致, 判定 High, 无分歧。理由: 若最乐观情形成立——(D,J) 类因果不稳定性特征在跨季节、跨风暴类型、跨估计器的真实 4D 数据上稳定提升严格 held-out 的强对流预警技能, 并给出可信的 regime-specific 信任边界——会为"因果特征工程能否切实改进天气预警"这条此前从未被清晰验证过的研究线提供一个有价值的正面或负面锚点, 具备顶会/顶刊发表价值。但 SHIPS+ (2510.02050) 已在紧邻域用真实数据证明了"因果衍生特征增强 baseline 可行"这一更一般的问题, ST-CND 也已在另一个紧邻域探索类似融合, 说明这条路线本身并非无人涉足的全新方向, 故不到 Exceptional。
  - 查表: Low x High = Medium (5)。topic 未声明 `preferred-contribution-types`, hard cap 不适用, numeric score 不截断。Claude 与 codex 在 Likelihood 和 Impact 两个轴上均无分歧 (均为 Low/High), 无需在 Comments 中额外标注分歧。

## Overall

- Priority: Medium
- Score: 5
- Comments: v7 是这个 idea 的第 7 轮, 也是第一次真正跳出"v1-v6 (D,J)-alone vs raw-alone 竞赛"这一错误问题框架, 转向 owner 实际关心的"融合能否改进预警"——Claude 与 codex 独立评估都认为这次 anchor reset 方向正确、比 v6 的跨估计器审计更贴近可行且有价值的下一步, 且预注册纪律 (prereg commit 早于 GPU job、raw_only 与既有结果 bit-exact 复现、alpha 扫描被诚实标注为非确证性) 总体延续了 v4-v6 建立起来的良好习惯。但本轮同时新增了一组不应在第 7 轮还出现的问题: (1) "12/16" 这一核心计数在 idea 头部摘要、Claims C1 和 git commit message 三处一致地算错 (正确值 10/16), 与文件自己更下方的明细段落自相矛盾; (2) `application` 贡献类型的声明超前于证据, 且与本轮自己的 `PILOT_PLAN_V7.md` "contribution scope unchanged" 表述直接冲突; (3) 新增的 fusion_helpers.py 是四轮以来第一个没有配套单元测试的模块; (4) P3 (H2) 的实际 estimand (边界 F1) 与"预警"目标之间出现新的错位。这些都是可以在下一轮低成本修正的具体问题, 不影响 Likelihood-Impact 判断, 但建议在推进到 proposal 之前先修正数字口径、把 idea.md 贡献类型声明与 PILOT_PLAN_V7.md 对齐、并明确 P3 端点与预警目标的关系, 而不是继续做第 8 轮式的文字/框架打磨——Claude 与 codex 一致认为, 从现在起唯一能真正推进 Likelihood 判断的是执行 (held-out 融合器 + graph-only-max 校准 + 真实数据验证), 而非继续 idea-level 的 refine。

</review>

