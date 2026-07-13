<!-- 书写报告使用中文 -->
---
topic: topics/0710-causal-scs.md
landscape: topics/0710-causal-scs-landscape.md
workspace: workspace/causal-scs-indicator/
---

- One-sentence summary: 在同一段 96h `pressure level × longitude × latitude × time` 场上，用容量匹配的 raw-only、raw+sham、raw+`(D,J)` CNN 检验多尺度因果不稳定性图能否提供原始网格端到端模型尚未学到的预警增量；因果估计器只是载体。
- Hypothesis: `(D,J)` 是 raw history 的确定性函数，不会凭空增加信息；可能的收益只能来自有限样本下更合适的归纳偏置。若跨层、邻格的效应离散度 D 与跳变 J 捕捉到普通 CNN 难学的演化模式，加入它们应优于同架构 raw-only 和维度匹配 sham。hour/day 只是 lag 切片，不预设物理机制；若可靠增量消失，则得到该表示的适用边界。
- Expected outcome: REAL-4D 的主指标统一为 AUPRC。在 nested storm holdout 中，所有选择只用内层折；仅当 raw+`(D,J)` 相对 raw-only 的 `Delta AUPRC >=0.02`、storm-cluster bootstrap 95% CI 下界 `>0.01`，同时优于 sham、Brier/ECE 不退步并跨 season/架构复现，才称有实用增量。最便宜证伪已完成：真正吃 96h grid tensor 的位置保留 CNN 上，raw AUPRC 0.690、raw+`(D,J)` 0.755，但 paired delta `+0.065 [-0.013,+0.154]`，结论为 NULL。
- Contribution type: empirical-finding+diagnostic+application
- Contribution drift note: v9 的三类贡献全部保留，无新增或删除；`application` 仍须真实 storm-system 证据挣得。topic 未声明 `preferred-contribution-types`。
- Risk: HIGH
- Estimated effort:
  - Compute: v10 两次 L4 运行共 56 秒（约 0.016 GPU-hour）；真实 mini 约 2-5 GPU-hour，确认性 P2 约 20-50 GPU-hour、1000-3000 CPU-hour。
  - Data: 3 个 HRRR 快照与 SPC schema 已有；连续逐小时场、匹配非事件标签和风暴移动矢量仍受 stop gate 阻塞。
  - Implementation: 真实 mini 与 P1.5 约 2-3 周；多季节 storm-level 管线约 4-6 周。
- Novelty quick-check: 不主张因果估计器或融合机制新颖。Ganesh et al. SHIPS+ (arXiv:2510.02050) 是最近邻：它把因果发现选出的特征**增补**到标准 SHIPS predictors，而非 v9 误写的剪枝/替换；本工作改问 SCS 原始网格 CNN 上、容量与 sham 受控的时空不稳定性条件增量。Flora et al. (arXiv:2603.20250) 已证明 U-Net 可在 WoFS 网格张量上端到端做 2-6h 强对流预警，但不含因果不稳定性审计。差异化是待验证的应用问题，不等于贡献已经成立。
- Strongest objection: v10 的 `(D,J)` 来自同一 raw tensor；首个全局池化 CNN 因 raw 训练损失停在 0.693 而产生虚高的 `+0.199` AUPRC 增量，说明弱对照会制造乐观结论。位置保留 CNN 虽让 raw AUPRC 升至 0.690，增量 CI 仍跨零，且架构是在看到首轮失败后选择。另一个未解风险是邻格边可能只是平流、风暴移动或资料同化传播；合成场不能排除它。
- Why we should do this: v10 首次直接回答 owner 的原始张量问题，并展示 comparator adequacy 会改变结论。无论真实阶段得到稳定正增量还是功效充分的零结果，都能约束“因果衍生表示是否为天气 CNN 提供额外归纳偏置”。
- Pilot:
  - Setup: `PILOT_PLAN_V10.md` 预注册 8-seed、32 train/32 test 的 3D CNN；raw+zero、raw+Gaussian-sham、raw+`(D,J)` 共享完整 96h 输入和参数量，hour/day 共用 7 个窗口。首轮 global pooling 后，`PILOT_PLAN_V10B.md` 另行预注册保留 `4×2×2` 粗时空位置的诊断性复核；两次均在 L4 实跑。
  - Metric: AUPRC 为主、AUROC 为诊断；POSITIVE 要求 causal-minus-raw 与 causal-minus-sham 的 seed-bootstrap CI 下界均 `>0`，causal-minus-raw CI 上界 `<0` 才为 NEGATIVE，否则 NULL。
  - Result: P0 从 v9 单 seed/no CI 扩为 8 seed：D 在真实 lag/区域内为 0.688 [0.655,0.721]（3h）和 0.723 [0.708,0.737]（24h）。day J 仍不具 lag 特异性：错误 36h 为 0.619，真实 24h 为 0.618；D+J 则为 0.619 vs 0.671。位置保留 CNN 中 sham/raw/causal AUPRC 为 0.515/0.690/0.755；causal-minus-sham `+0.240 [0.164,0.312]`，causal-minus-raw `+0.065 [-0.013,0.154]`。
  - Signal: P0 仅 D 为清晰 POSITIVE，day J 不支持 lag-specific claim；P1 raw-tensor CNN 为 NULL。全局池化首轮的规则内 POSITIVE 仅诊断弱 comparator，不作为主结果。

- Claims and Claims matrix:
  - C1（历史纠错）: v9 只测试了 12 个手工聚合特征上的 LR/RF，不是端到端 raw-grid 模型。其 raw mean AUROC 为 0.753/0.768（LR/RF），RF 最小单-seed delta 为 -0.145；错误 lag 或区域外 P0 值覆盖 0.456-0.642。v9 P0 单 seed 无 CI，而 P1 有 8 seed CI；day 边错误 36h 的 J/D+J=0.634/0.639 高于真实 24h 的 0.582/0.631。v9 AUROC 仅是历史 tabular screen，不再与正式 AUPRC gate 混用。
  - C2（v10）: 已完成真正 raw-tensor、等历史、等窗口、等容量并含 sham 的 CNN 检验。P0 支持 D 通道的时空定位，不支持 day J 的 lag 特异性；可信 CNN 对 raw 的增量为正点估计但 CI 跨零，只能称 NULL。不得用首轮弱 raw CNN 声称可靠增量。
  - C3: 只有 P1.5 全通过且真实 storm-level 结果越过预设 AUPRC、校准和复现门槛，才允许声称增量预警技能；任何结果都不识别真实大气因果机制。

  | Outcome | 允许的 claim |
  |---|---|
  | REAL-4D POSITIVE | 限定声称该 `(D,J)` 实现为同历史、同容量的已训练 raw-grid 模型提供增量技能。 |
  | REAL-4D NULL/NEGATIVE | CI 含零只称不可判定；功效充分且 CI 上界 `<0.01` 时，称该实现无实用增量，不外推到其他表示。 |
  | 当前状态 | SYNTHETIC P0：D POSITIVE/day J non-specific；raw-tensor fusion：NULL；REAL-4D：SKIPPED。 |
- Narrative: 论文主线是 conditional-information audit：先确认各 `(D,J)` 通道是否定位植入信号，再问它们能否补充一个合格的 raw-grid 模型。v10 的关键教训不是“CNN 已被击败”，而是弱 raw comparator 会把 NULL 变成表面 POSITIVE。
- Experiments:
  - P0（已完成）: 8-seed 合成 4D 场，逐 lag/区域/通道报告 AUROC 与 CI；hour/day 使用相同 7 个窗口。
  - P1（stop gate 后真实 mini）: 3 个日期各取 120h event/non-event chip；冻结 raw-tensor CNN、sham 与 raw+`(D,J)`，只作 lag profile 和管线验证，不作三日 skill claim。
  - P1.5（P2 强制门）: graph-null 与 familywise 校正、等窗口敏感性、功效分析，以及**完整** storm-relative 坐标/平流匹配对照；该阶段独占平流风险，不再把同一任务列入 P3。
  - P2（确认性应用）: `>=2` seasons、`>=50` 独立 storm systems，2-6h/36km hazard，outer storm holdout；比较 raw、raw+sham、raw+非因果 lag 特征、raw+`(D,J)`，共享历史与预算。模型 comparator 集不限于本轮的位置保留 CNN：v10 只是验证"能否在一个合格的端到端模型上观察到增量"这一问题的第一个可信实现，P2 起应视为可扩展的模型谱系，后续可纳入更大容量/更现代的时空架构 (如 transformer-based 时空模型、DiT 等扩散式架构) 复核结论是否只是当前 CNN 归纳偏置的特例；本轮不实现这些架构，仅作路线图标注。
  - P3（归因）: 在通过 P1.5 后比较 full 4D、single-column、horizontal-only、hour-only、day-only，并分析不预命名的 regime 差异；边界 overlap 仅为次要端点。
- Assets status: 合成 4D、v10 CNN 代码/结果及 HRRR/SPC smoke 资产已就绪；连续真实 4D 数据仍受 stop gate 阻塞，详见 `workspace/causal-scs-indicator/data/MANIFEST.md`。
