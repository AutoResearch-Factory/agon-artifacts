---
topic: topics/0710-causal-scs.md
landscape: topics/0710-causal-scs-landscape.md
workspace: workspace/causal-scs-indicator/
---
<!-- 书写报告使用中文 -->
- One-sentence summary: 在 HRRR 的真实 `pressure level × longitude × latitude × time` 场上，把跨气压层、跨邻近网格的因果效应不稳定性图 `(D,J)` 按小时级与天级 lag 分开提取，再检验它们能否在独立 storm-system 上为同输入、同容量的 raw-history 强对流预警基线提供增量；因果估计器是载体，不是方法创新。
- Hypothesis: 强对流前的 4D 环境不是一组彼此独立的单列时序。天气尺度强迫可能表现为 500-hPa 槽/涡度到 850/925-hPa 水汽与辐合的宽广 12-48 h 关系；边界层/对流尺度过程可能表现为 850/925-hPa 辐合到 250-hPa 辐散的局地 1-6 h 关系。若这些跨层、跨网格边的滑窗效应在事件前发生有位置和尺度选择性的 dispersion (`D`) 或 jump (`J`)，完整的 pressure-lag 空间图可能含有 raw state/history 未充分利用的预警信息。反之，若同容量 raw-history 或非因果 lag-map 基线吸收全部增量，这个负结果同样界定因果特征工程的适用边界。
- Expected outcome: 科学支持信号是：在嵌套 storm-system holdout 中，所有 edge/lag/权重选择只用内层折，`raw+(D,J)` 相对同 raw 输入、同 backbone/参数预算的 raw-history 模型取得 held-out `Delta AUPRC >= 0.02`，storm-cluster bootstrap 95% CI 下界 `>0.01`，且 Brier skill 下降与 ECE 增加均不超过 0.01。预注册 hour/day ablation 用来判断增量来自哪一尺度；只有两者都提供不可替代增量时才声称“跨尺度互补”。最便宜的实现证伪已完成：若已知 3 h/24 h 跨层边不能在合成 4D 场上被正确定位和分尺度，真实数据路线停止。v8 通过该实现门，但 held-out 合成融合选择 raw-only，尚无科学正信号。
- Contribution type: empirical-finding+diagnostic+application
- Contribution drift note: v7 已是 `empirical-finding+diagnostic+application`，v8 三类全部保留。`application` 表示 owner 指定的目标范围，只有真实 storm-system 预警证据才能最终挣得；当前合成与 schema smoke 不提前宣称完成应用贡献。topic 未声明 `preferred-contribution-types`。
- Risk: HIGH
- Estimated effort:
  - Compute: v8 smoke 为 13 s L4；解除 stop gate 后，3-date/120 h 4D mini 预计 2-5 GPU-hour、40-100 CPU-hour；确认性多 storm 实验约 20-50 GPU-hour、1000-3000 CPU-hour。
  - Data: 3 个 HRRR 等压面快照与 SPC schema 已有；连续逐小时等压面序列、匹配非事件和雷达标签仍需收集，当前 stop gate 不允许下载。
  - Implementation: 4D mini 1-2 周；storm-level 数据管线、标注和确认性评估 4-6 周。
- Novelty quick-check: Ganesh et al. 的 Multidata-PC SHIPS+ (arXiv:2510.02050) 已在真实 TC 业务数据上证明“因果发现衍生特征增强现有强 baseline”这一一般框架，故融合本身不新。ST-CND (Yu & Liang, arXiv:2606.17553) 已把空间因果网络用于地理临界点预警，并在解释性 logistic readout 中混合传统 EWS；其 headline 不是 pressure-resolved、multi-lag 特征对 severe-convective hazard baseline 的严格增量检验。剩余差异仅是 4D 等压面结构、小时/天级 lag 分解、`(D,J)` 特征及真实强对流端点，必须靠实证而非方法命名建立价值。
- Strongest objection: v7 的 50/50 失败只证伪等权组合，事后 alpha 又叠加了“先挑 graph-only 有利格、再在同一评估集挑权重”的双重选择；confound-only/mixed 格 raw AUC 已达 0.985-0.998，alpha=1.0 也可能只是天花板效应，而非混杂污染的独有证据；更根本地，4D `(D,J)` 图可能只是昂贵地重编码 raw 梯度与 lag correlation，当前 3 个 HRRR 快照不能反驳这一点。
- Why we should do this: 现有 v1-v7 全部只测单点/单列与单一窗口尺度，未触及 owner 指出的真正 gap。把垂直、水平和多尺度 lag 放进同一个增量预警检验后，正负结果都能回答“因果框架是否为真实 4D 强对流环境增加可用信息”。
- Pilot:
  - Setup: `PILOT_PLAN_V8.md` 在结果前冻结两部分：现有 3 个 HRRR 快照只验 1000/500-hPa `pressure × latitude-grid × longitude-grid` 读取与空间算子；32 对合成 event/control 使用 `{1000,850,500,250}` hPa、`16×12×96 h` 场，注入西邻格 500->850 的 24 h 边和 850->250 的 3 h 边。fixed-edge local regression 仅作实现 proxy；前 16 对选一个 raw-favoring alpha，后 16 对冻结评估。commit `b68a229` 早于 L4 job 36919944。
  - Metric: 实现 PASS 要求 3 h 与 24 h 都在各自候选 lag 中被选中，且真 lag 的 injected-region minus outside `(D+J)>0`；融合 AUC 只检查 held-out plumbing，不作为设计保证 DGP 上的科学门槛。HRRR 快照不连续，禁止计算 lag 或预警 skill。
  - Result: job 36919944 exit 0；HRRR 三份均解码为 `2x1059x1799`、1000/500-hPa U/V。合成小时/天级分别正确选中 3 h/24 h，空间 contrast 为 +0.04485/+0.02496，implementation PASS。训练折 alpha=0.6 与 1.0 同为 AUC 0.7656，预注册 tie-break 选 1.0；held-out raw/causal/fusion AUC=0.693/0.615/0.693，Delta=0。正确恢复结构未自动产生融合增量。
  - Signal: SKIPPED（科学证据）；PASS（4D 实现可行性）。

- Claims and Claims matrix:
  - C1: v7 等权融合的准确结果是 10/16 格 Delta CI 严格为负、6/16 含零（其中 4 格是 `g=0,h=0` 无扰动退化格），0/16 显著为正；4 个 graph-only-max 主格中 2 负、2 含零。它只否定该 DGP/proxy 下的 50/50 规则。
  - C2: v7 alpha sweep 仅用于生成假设，既无 held-out/CI，又受双重事后选择；confound/mixed 格 alpha=1 同时兼容“因果通道受污染”和“raw baseline 天花板”两种解释，现有数据不能区分。
  - C3: v8 只证明软件能在已知合成场上形成 pressure-lon-lat-time `(D,J)` 图并分辨 3 h/24 h lag；3 个真实 HRRR 快照只证明空间数据通路。不得称已发现真实大气因果结构、机制或预警技能。
  - C4: 只有真实 storm-level nested holdout 越过预设实用阈值，才允许声称“在指定 hazard、lead time、数据和模型类中有增量预警技能”；因果机制识别不在允许 claim 内。

  | Outcome | 允许的 claim |
  |---|---|
  | REAL-4D POSITIVE | `Delta AUPRC` 点估计与 CI 同时越过阈值、校准不退步且跨 season/estimator 复现时，限定声称 pressure-resolved multi-lag `(D,J)` 特征对该预警任务有增量。 |
  | REAL-4D NULL | 仅称当前设计不可判定；只有功效充足且 CI 完全落入预注册 `[-0.01,+0.01]` 等效区间，才称增量小于实际意义。 |
  | REAL-4D NEGATIVE | 若增量 CI 上界 `<0.01` 且 raw/noncausal controls 有功效，可称该特征在明确范围内没有达到实用增量；不外推到所有因果表示。 |
  | 当前状态 | 50/50 SYNTHETIC NEGATIVE；4D IMPLEMENTATION PASS；REAL-4D SCIENTIFIC SKIPPED。 |
- Narrative: 主线不是另做一篇“空间边界探测”论文，而是把 `S(t,lag-band,pressure-pair,lon,lat,D/J)` 作为预警模型的附加通道，问其是否提升真实 hazard skill。垂直/水平 pattern 和 hour/day lag 是解释增量来自何处的机制切片；边界 F1 仅为次要定位端点，不能替代龙卷/冰雹/大风预警。
- Experiments:
  - P0（已完成）: v8 真实 HRRR pressure-grid schema + 已知边合成 4D smoke；结果只决定实现是否可继续。
  - P1（stop gate 后的最小真实验证）: 为现有 3 个日期各取 120 h 逐小时 event chip 与匹配 non-event chip，固定 `{1000,925,850,500,250}` hPa、U/V/T/RH/omega/geopotential 和物理稀疏 edge library；分别跑 `{1,3,6}` h 与 `{12,24,36,48}` h。固定 24 h window/6 h stride 后，48 h lag 仍有 9 个滚动窗口可定义 D/J。用 PCMCI+/LKIF 与同复杂度 partial-correlation proxy 生成图，只报告 lag profile、runtime 与 raw/noncausal 对照点估计，不做三日 skill claim。
  - P2（确认性应用）: 至少 2 seasons、由预先功效分析确定且不少于 50 个独立 storm systems；primary label 为 2-6 h 内 36 km 的 tornado/hail/wind，outer storm holdout、inner edge/lag/alpha/HPO，cluster bootstrap。比较 operational/raw-history、dimension-matched 非因果 lag-correlation/gradient maps、以及 raw+(D,J)，共享输入时段、backbone 与参数预算。
  - P3（归因与稳健性）: full 4D 对比 single-column、horizontal-only、hour-only、day-only；检验低层辐合->高层辐散是否随位置系统变化，以及 hour/day 差值能否在独立 synoptic/boundary 标签下区分强迫 regime。同步做 graph-only null calibration、window/stride/quantile 敏感性、familywise correction、PCMCI+/LKIF 复现；中尺度边界 overlap 为 secondary endpoint。
- Assets status: 3 个 HRRR/SPC smoke 资产、v7/v8 结果与锁定的 GRIB 环境已就绪；连续 4D 数据仍被 stop gate 阻塞，细节见 `workspace/causal-scs-indicator/data/MANIFEST.md`。
