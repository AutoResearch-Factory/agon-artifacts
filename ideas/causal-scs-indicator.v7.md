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
