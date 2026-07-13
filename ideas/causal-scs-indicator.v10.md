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

<review date="2026-07-12">

## Novelty

- Score: 3/10（Claude 与 codex 独立评估一致，与 v7-v9 一脉相承不变）
- Closest prior work: Ganesh, Beucler, DeMaria, Runge (2025), Multidata-PC SHIPS+ (arXiv:2510.02050)。Claude 与 codex 均重新核验其全文摘要：因果发现选出的预测因子被**增补**进标准 21 个 SHIPS predictors 并在真实业务数据上检验增量技能, 这一表述在 v9 review 中已被修正, v10 novelty quick-check 延续了修正后的准确表述。
- Key differentiator: 因果估计器与融合机制本身不主张新颖; 差异化仍是"在同历史、同下游参数量、含 sham 对照的原始时空场端到端模型上做条件增量审计"这一具体设计, 而非 SHIPS+ 式的联合增强预测因子集重训练。codex 独立核实并补充了一处需要收紧的表述: 本轮 novelty quick-check 把 Flora et al. (arXiv:2603.20250) 的 U-Net 称为"raw-tensor"先例, 但 codex 通读全文确认其实际输入是经过时间聚合/ensemble statistics 处理后的 63-channel 2D 张量, 并非完整 96h 原始历史张量; 下一版应把它改称"grid-based precedent", 不宜称为与 v10 完全同构的"raw-4D precedent"。这一纠正收窄但不消除既有差异化空间。

## Quality

评估视角: topic (`topics/0710-causal-scs.md`) 未声明 `target-venue`/`preferred-contribution-types`（Claude 与 codex 独立核实一致）。沿用 v7-v9 review 的"顶级 Earth-system methods / AI4Science / 强对流预测应用"稿件标准。

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 6/10（Claude 与 codex 独立评估一致，较 v9 的 4/10 明显提升） | 本轮核心修复经 Claude 独立读码（`cnn_fusion_helpers.py`）与 codex 独立复核一致确认为真实: `raw` 分支是完整 96h 原始张量 `fields.permute(...)` 直接喂入 `Conv3d`, 无任何手工聚合特征; `raw`/`sham`/`causal` 三条件通过 `inputs.shape[1]` 共享同一 `RawTensorCNN` 架构, `tests/test_cnn_fusion_helpers.py` 显式断言三者 `parameter_count` 相同 (687 in the tiny test config)。剩余缺口 (Claude 与 codex 独立收敛到几乎相同的清单): (1) v10b 架构是在看到 v10 (global pooling) 失败后选择的, 虽只改了一处 (`AdaptiveAvgPool3d(1)`→`(4,2,2)`, Claude 独立 diff 确认), 但仍是"看了结果再选架构", 需要未来一轮独立冻结架构复核; (2) 即便在 v10b 中, 8 个 seed 里仍有 1 个 (`20260901`) raw 分支训练损失停在 0.6931 (卡在 log(2)), Claude 独立计算发现该 seed 的 causal-minus-raw 增量 (+0.297) 是全部 8 个中最大的异常值, 拉高了汇总均值 (若剔除该 seed, 均值会从 0.065 降到约 0.032, 更接近零)——这一异常值目前被直接平均进汇总而未被诊断或标注, 但方向上是让 NULL 结论更保守而非制造虚高, 不构成 cherry-pick; (3) codex 独立指出一个 Claude 未充分权衡的更深问题: "capacity-matched" 只匹配了下游 CNN 的参数量, 未把外部 `(D,J)` 特征提取器本身对 true edge/true lag 的 oracle 知识计入比较, 即 `(D,J)` 通道享有"已知真实耦合位置和真实滞后"的信息优势, 这一优势不会被同参数量的 CNN 架构抵消; (4) Gaussian sham 只控制维度和随机噪声, 不控制"任何结构化、确定性的非因果 lag/covariance 表示"这一更强的替代解释 (idea 自己的 P2 计划已列入"raw+非因果 lag 特征"但本轮未运行)。 |
| Missing evidence signals | 4/10（Claude 与 codex 独立评估一致，较 v9 的 3/10 略有提升） | 仍缺: 独立冻结的现代时空架构族 (P2 路线图已提及 transformer/DiT, 但本轮不实现)、结构化非因果 lag-map 对照 (P2 计划中, 未跑)、更多 seeds 与预注册功效/精度停止规则 (当前 8-seed bootstrap 区间仍较宽, 对 0.02 的目标增量而言检验力有限)、pair-within-seed 两阶段重采样、真实 PCMCI+/LKIF (当前仍是 fixed-edge local regression proxy)、graph-null/familywise 校准 (v6 的 null 校准审计只覆盖了旧的 factorial_helpers pipeline, 未覆盖本轮新引入的 CNN+8-seed-bootstrap pipeline 本身, Claude 独立指出这一点)、storm-relative/advection-matched 真实执行、以及任何真实连续 4D storm 证据 (仍是 0%)。 |
| Narrative | 7/10（Claude 初评 8, 采纳 codex 更严格证据后下修为 7, 已收敛） | 自我纠错叙事真实且诚实 (下方 provenance 核验证实), 没有把首轮规则内 POSITIVE 包装成科学胜利, 明确把 v10b 标注为 post-result diagnostic, 这是本轮最大的叙事进步。但 codex 指出两处仍需收紧的措辞 (Claude 认同): (a) Expected outcome 中"最便宜证伪已完成"表述不准确——`+0.065 [-0.013,+0.154]` 是较宽的 NULL 区间, 并未真正证伪"存在实用增量"这一假设, 只是未能确认它; (b)"v10 首次直接回答 owner 的原始张量问题"这一表述应明确限定为"在单一合成 DGP、一个事后选择架构的微型 CNN 上"作答, 而非无条件地称为该问题的首次直接回答。 |
| Venue contribution | 3/10（Claude 与 codex 独立评估一致，较 v9 的 2/10 略有提升） | 仍是 oracle planted-edge 的 100% 合成 pilot, 无新方法也无真实气象发现。global-vs-position pooling 反转本身是有教学/诊断价值的 comparator-adequacy 案例, 可以成为未来论文中一项有力的消融实验, 但不能单独构成 top-venue 贡献。 |
| Testability | 6/10（Claude 初评 8, 采纳 codex 具体证据后下修为 6, 已收敛） | Expected outcome 对 REAL-4D 阶段的 Delta AUPRC/CI/校准/复现标准清楚, AUROC/AUPRC 指标本轮已统一 (v9 遗留问题已修复)。但 codex 指出一个 Claude 初评未捕捉的具体不一致: pilot 代码 (`summarize_cnn_results`) 把 `signal="NEGATIVE"` 的判据设为 `causal-minus-raw AUPRC CI 上界 < 0`, 而 idea 正文 Claims matrix 对"无实用增量"的判据是 CI 上界 `< 0.01`——这是两个不同的阈值 (statistical harm vs. practical non-inferiority margin), 当前文本未区分, 容易在下一阶段造成判定混淆。此外 n=8 的 seed-bootstrap 区间宽度 (约 ±0.08-0.09) 相对 0.02 的目标增量而言检验力有限, 使其作为"最便宜证伪"的效力打折扣。 |
| Outcome realism | 5/10（Claude 初评 7, codex 给 4, 部分采纳 codex 论据后收敛到 5） | `(D,J)` 是 raw tensor 的确定性函数这一信息论框架依然自洽, 且本轮结果与该框架预期一致, 值得肯定。但 codex 指出的关键点是: 即便在 oracle true edge/true lag、专门设计对 `(D,J)` 有利的合成条件下, 可信 raw comparator 之上仍只得到宽区间 NULL——十轮里已有约九轮持续得到零/负结果, 这本身是对"REAL-4D 阶段能达到 Delta AUPRC>=0.02 且 CI 下界>0.01"这一 Expected outcome 目标的现实性的实质性负面证据, 该目标当前看起来偏乐观 (虽非不可能)。 |
| Contribution type compliance | n.a. | idea types = {empirical-finding, diagnostic, application}；topic 未声明 `preferred-contribution-types`, 跳过, 不计入 Overall Quality 平均（Claude 与 codex 一致）。 |
| Overall Quality | 5/10（Claude 收敛后 (6+4+7+3+6+5)/6≈5.2→5, codex 直接给 5, 两者一致） | v10 是九轮以来在实验设计上最扎实的一轮: owner 反复要求的"真正原始张量端到端模型"缺口本轮被真实兑现并经独立验证; 但新发现的细节缺口 (capacity-matched 未覆盖 (D,J) 的 oracle 位置/滞后知识、sham 未覆盖结构化非因果替代解释、NEGATIVE 阈值与 Claims matrix 阈值不一致) 与 Quality 提升大体抵消, Overall Quality 相对 v9 (4/10) 仅小幅上升。 |

## Contribution Drift (n >= 2)

- v_{n-1} (v9) contribution types: {empirical-finding, diagnostic, application}
- v_n (v10) contribution types: {empirical-finding, diagnostic, application}
- Status: unchanged
- Hard cap triggered: no（topic 未声明 `preferred-contribution-types`；即便声明, 本轮也无扩张/删除, Claude 与 codex 独立核验一致）

### v9 `<review>` concern 逐条复核（Claude 独立核验 + codex 独立核验，两者收敛一致）

| v9 concern | Status | Assessment |
|---|---|---|
| 对照必须是"原始 4D 场端到端训练的模型", v9 实际用的是 12 个手工聚合特征上的 LR/RF | **resolved** | Claude 独立读码确认 `cnn_fusion_helpers.py` 的 `raw` 分支是完整张量直接喂入 `Conv3d`, 无手工特征; codex 独立复核结论一致。v10b 的架构选择发生在看到 v10 失败之后, 是本轮唯一需要标注的限定。 |
| raw 只看最近 24h 而 `(D,J)` 隐含利用近 96h 历史 (时间窗口不对齐) | **resolved** | 三条件 (`raw`/`sham`/`causal`) 均读取完整 96h 张量, Claude 与 codex 独立核实一致。 |
| hour 族 12 个滚动窗口 vs day 族 7 个滚动窗口 (窗口数不对称) | **resolved** | Claude 独立读 `four_d_helpers.rolling_beta_maps` 与 `cnn_fusion_helpers.matched_beta_maps` 确认: 后者用 `common_lags = union(hour_lags, day_lags)` 强制两族共享同一组 7 个窗口终点 `{60,66,72,78,84,90,96}`, JSON `shared_window_count=7` 印证; codex 独立核实一致。 |
| 缺 dimension-matched sham/noncausal 对照 | **partially resolved** | Gaussian sham (维度/参数量匹配) 已完成并经 `parameter_count` 断言测试验证; 但"结构化非因果 lag 特征"对照仍只列入 P2 计划, 本轮未运行 (Claude 与 codex 独立指出同一缺口)。 |
| P0 单 seed、无 CI，与 P1 8-seed 不对称 | **resolved** | P0 本轮重算为 8-seed + seed-bootstrap CI, Claude 与 codex 独立核实数字与 JSON 完全一致。 |
| day J/D+J 错误 lag 结果高于真实 lag 未被披露为例外 | **resolved** | v10 明确写出 day J 在错误 lag=36h (0.619) 与真实 lag=24h (0.618) 几乎持平、不具 lag 特异性, Claude 独立核实该数字与 JSON `inside_injected__J_auroc_mean` 完全一致。 |
| RF raw mean AUROC 四舍五入错误 (0.770 应为 0.768)，最小单-seed delta 应为 -0.145 | **resolved** | Claude 独立从 `results/ml_fusion_pilot_results.json` 重新计算 RF raw mean = 0.76806640625 (四舍五入为 0.768, 非 0.770) 与最小 delta = -0.14453125 (四舍五入为 -0.145)，与 v10 正文一致。 |
| 遗漏错误 lag/区域外数值范围 0.456-0.642 | **resolved** | Claude 独立扫描 P0 全部错误 lag/outside 单元格, 确认最小值 0.456 (hour lag=1h outside)、最大值 0.642 (day lag=36h inside) 与 v10 正文范围完全一致。 |
| pilot 决策规则用 AUROC、正式 Expected outcome/Claims matrix 用 AUPRC 未统一 | **resolved** | v10 统一以 AUPRC 为主指标并计算 CI, AUROC 降为诊断性指标（Claude 与 codex 独立核实一致）；但 codex 另外发现一个新的、未被此条覆盖的阈值不一致问题（见 Testability）。 |
| storm-relative/advection 对照同时出现在 P1.5 与 P3，归属不清 | **resolved** | v10 明确把该对照的完整执行责任收拢到 P1.5, P3 不再重复列出（Claude 与 codex 独立核实一致）；但对照本身仍未实际执行。 |
| 缺 graph-null 校准、familywise 校正、功效分析 | **partially resolved** | 仍列为 P1.5 强制门, 本轮未新增证据；这类分阶段处理有其合理性, 但不能无限期推迟到确认性应用结果之后（codex 独立提出同一保留意见）。 |
| SHIPS+ 被误写为剪枝/替换算子 | **resolved** | codex 独立重新核验 SHIPS+ 全文摘要, 确认 v10 沿用的"增补"表述准确。 |
| Alternative Framing 建议 (收紧为多 seed P0 重刻画 + raw-tensor CNN/U-Net + sham + 因果特征三方比较) | **resolved, 已成为本轮蓝图** | v9 review 提出的两步走建议基本就是 v10 实际执行的设计, 不只是"未忽视", 而是被直接采纳为本轮工作蓝图。 |

## Alternative Framing

Claude 与 codex 收敛到同一个方向（codex 表述更精炼, Claude 采纳并补充一句落地路径）：把主线进一步收紧为"确定性气象衍生表示的 comparator-adequacy 与 sample-efficiency 审计"——不只是问"`(D,J)` 能否增益", 而是系统化研究"架构瓶颈 (如 global pooling 抹除位置信息) 在什么条件下会人为制造虚假增量、以及 `(D,J)` 在何种样本量/架构下仍可能有真实增益"。global-to-position pooling 反转已经是这一更宽框架下的第一个具体案例。这一重新框架不引入任何 topic `preferred-contribution-types` 之外的类型 (topic 本身未声明该字段)，但要让它把 Likelihood 从 Low 抬升到 Medium, 仍需要跨独立冻结架构、结构化非因果对照与真实 storm 证据的验证, 本轮尚未达到。

## Claims Discipline

Claude 与 codex 收敛后的版本（较 idea 正文 Claims matrix 更严格, 尤其区分了 statistical harm 与 practical non-inferiority margin）：

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 只有在**独立冻结**的原始时空模型 (而非本轮"看到失败后选择"的架构)、相同历史与容量预算、outer storm holdout、结构化非因果对照 (不只是 Gaussian sham)、校准不退步、跨 season/架构复现、且通过预设 Delta AUPRC 阈值后, 才能限定声称该 `(D,J)` 实现为这些模型提供了有限样本增量归纳偏置。不得声称"增加了信息"或"识别了真实大气因果机制"。 |
| NULL | 当前只能说: 在一个 planted-edge 合成 DGP、8 seeds、事后选择架构的位置保留微型 CNN 上, 观察到 causal-minus-raw 的正点估计但 CI 跨零 (`+0.065 [-0.013,+0.154]`)。不能声称 raw 模型已吸收全部信号, 也不能称该表示无效; 该结果甚至没有真正"证伪"实用增量假设, 只是未能确认它。 |
| NEGATIVE | 需要先冻结一个实用差异阈值 (如 0.01-0.02), 并以功效充分的 CI 排除该阈值, 且在多个独立冻结的合格 raw 架构与真实 storm splits 上复现, 才能称该具体表示在限定范围内没有实用增量。当前 pilot 代码把 `NEGATIVE` 定义为 CI 上界 `<0` (statistical harm), 与 Claims matrix 里"无实用增量"要求的 CI 上界 `<0.01` 不是同一个判据, 下一版应明确区分这两种不同强度的负面结论, 且都不得外推到全部因果表示。 |

## Likelihood-Impact Matrix

- Priority: Medium = Likelihood: Low x Impact: High
- Numeric score for ideas.xml: 5
- Rationale:
  - Likelihood: Claude 与 codex 独立评估完全一致, 判定 Low, 无分歧。理由: v10 修复了 owner 反复要求的最严重 comparator 缺陷 (真正原始张量端到端 CNN), 但即便在 oracle true edge/true lag、专为 `(D,J)` 设计有利条件的合成场景下, 可信 raw comparator 之上仍只得到宽区间 NULL——这是新增的、对最终成功概率不利的证据, 而非仅仅"尚未证实"。真实阶段还需要连续 HRRR/SPC 数据解除 stop gate、真正的 PCMCI+/LKIF、storm-relative/advection 控制的真实执行、结构化非因果对照、graph-null/familywise 校准、独立冻结的架构、以及跨 >=2 季节/>=50 个独立 storm systems 的复现同时成立, 条件连乘且新发现的 capacity-matching 遗留问题意味着下一版设计仍需继续打磨。
  - Impact: Claude 与 codex 独立评估完全一致, 判定 High, 无分歧。理由: 若最终在严格独立冻结的 raw-tensor baseline 上跨季节、跨架构获得真实增量, 或建立起功效充分、可推广的失败边界, 将实质回答"因果衍生的时空不稳定性能否为现代天气 ML 提供该模型自身学不到的归纳偏置"这一问题, 对该研究线有明确价值; 但 SHIPS+ 已确立"因果特征增强既有强预测器"这一路线的可行性先例, 本工作本身不提出新方法也不提供新信息来源, 故不到 Exceptional。
  - 查表: Low x High = Medium (5)。topic 未声明 `preferred-contribution-types`, hard cap 不适用。Claude 与 codex 在 Likelihood 和 Impact 两个轴上均无分歧, 无需额外标注; 与 v9 review 的 Likelihood/Impact 判断相同, 故 numeric score 与 v9 (5) 持平——Quality 的实质提升未改变 Likelihood/Impact 这两个独立轴的判断。

## Overall

- Priority: Medium
- Score: 5
- Comments: v10 是十轮以来在实验设计执行上最扎实的一轮——Claude 与 codex 独立核验了完整的 git 预注册时序 (commit `d706fc1`→job `36939149`→commit `74e7cb0` 记录该结果并预注册位置保留架构→job `36939215`→commit `05d47ad` 记录该结果)、JSON 数值 (global 0.547/0.543/0.746, position 0.690/0.515/0.755, causal-minus-raw `+0.065 [-0.013,+0.154]`, causal-minus-sham `+0.240 [0.164,+0.312]`)、以及全部九项 v9 review 具体问题的修正, 均未发现伪造时序或选择性报告数字的迹象; "第一版全局池化 CNN 因弱 raw 训练制造虚高增量、换架构后重跑" 的自我纠错叙事经独立核验为真, 不是事后包装。但本轮结论仍是"弱 raw comparator 会把 NULL 制造成表面 POSITIVE", 而不是 `(D,J)` 已提供预警增量——十轮里已有约九轮在不同公平性水平的对照下得到零/负结果, 这是不利于 REAL-4D 阶段最终成功的实质性证据, 而非仅仅"尚未验证"。Claude 与 codex 在 Likelihood、Impact 两个轴上完全一致, 无需标注分歧; Quality 子项上 codex 在 Testability、Outcome realism 上给出比 Claude 初评更严格的具体证据 (NEGATIVE 阈值与 Claims matrix 阈值不一致、oracle 条件下持续 NULL 对 Expected outcome 现实性的负面含义), Claude 采纳后下修收敛, Overall Quality 最终与 codex 一致 (5/10)。就整体方向而言: idea 阶段已经把 owner 提出的核心方法论问题 (真实端到端模型对照) 真实、诚实地回答了一遍, 剩余的具体缺口 (独立冻结架构、结构化非因果对照、真实数据、校准) 更适合作为 proposal/experiment 阶段的具体任务清单, 而非继续在 idea 阶段做第十一轮相似规模的合成 pilot 打磨。

</review>

<deep-lit-integration date="2026-07-13" trigger="proposal v1 review (verdict: REVISE), idea-scope deep-lit-tick, 23 papers across 5 waves">

本轮由 proposal v1 review 指出的三个 CRITICAL 缺口触发, 完整报告见 `ideas/causal-scs-indicator-deep-lit-report-2026-07-13.md`。核心结论摘要（供下一轮 proposal-refiner 直接采纳）：

**缺口(1) HRRRCast(2507.05658)/StormCast(2408.10958)能否作 P2 raw-only 骨干预训练初始化**：精读后判定两者均非"拿来即用"。HRRRCast 有三处非平凡不匹配，其中最关键的是**单时刻输入单时刻输出、零 96h 历史维度**——用作骨干需另建时序聚合模块，直接违反 proposal 已声明的"不构建新时空骨干"non-goal。StormCast 是单体架构无可拆卸的分层编码器。同一作者 Pathak 2026 年最新后续作 Stormscope(2601.17268) 气压层覆盖更差、历史堆叠仍远不足 96h，同样不能解决该问题。最终骨干选型排序：**自建自监督预训练 > HRRRCast > StormCast > Stormscope(仅架构参考)**。若采用微调路径，probe head 复杂度应默认锚定线性/1-2层MLP(2508.17903 a fortiori证据：同类严重天气post-processing任务中，样本量大于我们时CNN/UNet仍会过拟合跑输简单模型)，且"训练折内部验证选骨干"步骤不可省略(2506.19088)。

**缺口(2) 小样本功效分析/storm-cluster bootstrap/AUPRC跨prevalence比较**：确认是真实、未被现成方案解决的方法论空白(WoFS 校准谱系 2012.00679→WoFSCast→2603.20250 三代都未解决)。综合 4 篇聚类稳健推断文献(2406.00650/2205.03288/2301.04522/2604.02000)可组装出具体的 storm-cluster bootstrap 可信度协议(阳性storm数功效分析、leave-one-storm-out杠杆诊断、双层嵌套覆盖率校准、day/season-block稳健性检查)。**更重要的发现**：McDermott et al.(2401.06091, NeurIPS, Tier S, 155引用)证明"AUPRC类别不平衡下总优于AUROC"是广泛误引的说法，AUPRC会系统性偏向高流行率子群体——这比 review 已指出的 prevalence-mismatch 缺口更根本，proposal review 建议的 PR-Gain(2007.01905)修补必要但不充分。建议下一版把 **Delta AUROC 提升为跨 IID/regime-迁移 holdout 比较的并列主指标**(AUROC 的流行率不变性有定理保证，AUPRC 没有)，同一 holdout 内 raw vs causal 比较仍用 AUPRC。

**缺口(3) 因果/物理诊断特征与预训练大模型融合先例**：精读 TabPFN-CFM(2606.26467, 34个tex文件+全部24条参考文献逐条核验)后确认**不构成先例**——纯 i.i.d. 表格因果基础模型，架构上是单模型联合学习结构+效应，与本 idea"独立因果发现方法算特征→注入另一预训练骨干"的两阶段融合设计完全不同轴，零引用天气/气候基础模型或时空因果发现方法。proposal"该角度目前无人做过"的 novelty 论证完整存活，且证据强度高于仅凭关键词搜索的此前版本。

**对下一版 proposal 的具体行动项**：(a) Frontier Leverage 修补应改为"P2 优先自建自监督预训练骨干，HRRRCast 微调路径列为备选并明确标注三处架构不匹配需先解决"，而非直接采纳 review 原建议的"冻结 HRRRCast/StormCast 编码器"；(b) Validation Focus 修补应采纳上述 storm-cluster bootstrap 协议五要点，并把 AUROC 提升为跨 holdout 比较的并列主指标；(c) Frontier Leverage 与 Novelty 段落可各补一句引用 2401.06091 与 2606.26467 分别处理"更根本的指标质疑已被正面回应"与"融合先例经核验确认不存在"。

</deep-lit-integration>

<deep-lit-integration date="2026-07-13" trigger="proposal v2 review (verdict: REVISE), idea-scope deep-lit-tick round 2, 16 papers across 2 rounds + 1 saturation-check round">

本轮由 proposal v2 review 的 CRITICAL 发现触发：v2 设计是自监督预训练后冻结 encoder、只训练一个线性 probe 做下游分类，encoder 本身从未接收任务梯度，与 idea Problem Anchor 锚定的"端到端训练的原始 raw CNN"产生操作性漂移。完整报告见 `ideas/causal-scs-indicator-deep-lit-report-2026-07-13-iteration2.md`。核心结论摘要（供下一轮 proposal-refiner 直接采纳）：

**子问题(1) frozen probe vs. 端到端微调，小样本时空遥感/天气场景**：无文献支持"默认冻结"。LP-FT（Kumar et al. 2022, arXiv:2202.10054，ICLR，理论+10基准）证明全量微调会扭曲预训练特征、纯冻结探测留下可改进空间，二者之间存在近零成本、有理论保证的更优策略——先线性探测再全量微调，同时改善 ID/OOD 精度；NLP 领域（2301.12715）独立证实"全量微调破坏任务无关特征"这一机制跨领域成立。**最锋利的一条证据**：v2 proposal 自己援引为灵感来源的 Prithvi WxC 原文（2409.13598，本轮精读其消融/局限章节）从未使用"冻结+纯线性 probe"——其 3 个下游任务的适配配方都是"冻结核心 + 多层新卷积块 + 残差融合"，比 v2 当前 `Linear(1088,1)` 复杂得多，且论文自陈"仅微调新头效果不佳，需要针对性架构改造"；proposal 当前设计比它自己的灵感来源更保守，而后者已自认这种保守做法效果不佳。WeatherPEFT/SFAS（2509.22020，已有 wiki，本轮复核）提供第三条路——Fisher 引导选择性微调，backbone-agnostic，0.1%-4% 参数量追平/反超全量微调，但其 Fisher 估计仅在数千级样本验证过，`<=50-100 storm` regime 未经验证，采纳前需先做小规模稳定性检查。

**子问题(2) masked temporal prediction vs. masked reconstruction pretext**：未找到任何论文（含 Prithvi WxC 自身）对"重建 vs 预测未来"做受控消融——Prithvi WxC 论文自陈"no ablations isolating individual design choices"（掩码策略/气候态目标/架构改动三者同时变化，无法归因）。这是真实文献空白而非检索遗漏。支持 codex 建议方向的是间接证据：SatVision-TOA（2411.17000）纯空间重建预训练的成功高度依赖"下游任务不需要时序推理"这一前提（其任务是单帧 2D 云检索），而本课题下游信号本质是 96h 时序演化；MAM4WF（2409.20117）证明"预测未来帧"范式在 SEVIR（强对流邻近数据集）上有效，但属全监督判别式预测器，非"预训练+小样本微调"框架。**建议表述**：下一版若采纳 masked-temporal-prediction 或 reconstruction+forecasting 混合目标，应定位为"任务对齐的合理设计选择，有间接类比证据支持"，不宜声称"已被文献证明优于 reconstruction-only"。

**子问题(3) 因果/物理衍生特征 + 微调后（非冻结）预训练模型融合先例**：两轮定向搜索 + 一轮跨领域广谱搜索（Granger 特征融合/因果图嵌入辅助输入/transfer entropy 特征融合/handcrafted+deep 混合融合四种独立措辞）均未发现直接先例——证据基础从此前"仅 TabPFN-CFM 一篇最接近但不构成先例"扩展到"跨四种措辞两轮搜索均未命中"，进一步加固 proposal Novelty and Elegance Argument"该两阶段设计目前无人做过"的论断。唯一可迁移的是架构模式而非概念先例：Prithvi-CAFE（2601.02315）的"轻量 adapter 微调的基础模型分支 + 处理互补信息的并行 CNN 分支 + 逐尺度注意力门控融合"模式，可用于升级 v10 pilot 现有 raw 分支与 `(D,J)` 分支间的 naive concat/mean 融合，与选择哪种 frozen/fine-tune 策略正交。Prithvi-EO-2.0 的 GPP 发现（全流程微调表征模型以 R² +20% 反超工程特征 baseline）是诚实的反向提醒：不能预设因果衍生特征一定比训练充分的表征模型多带来信息，支持 proposal 现有"诚实证伪、不预设正结果"的判据设计不需要改动。

**对下一版 proposal 的具体行动项**：(a) 采纳 codex 建议的核心修复方向，具体化为 LP-FT（SSL 初始化后解冻整个 encoder，在 outer-training storms 上联合微调 encoder+probe），作为零新增结构的默认路径；SFAS 式选择性微调列为备选，但须先补一次 Gate 0 阶段的小样本 Fisher 稳定性检查；(b) Modern Primitive Usage 若改用 masked-temporal-prediction/混合目标，措辞须收紧为"任务对齐的设计选择"而非"文献证明更优"；(c) Core Mechanism 的 raw/因果双分支融合方式可参考 Prithvi-CAFE 的注意力门控多尺度融合模式升级现有 concat 设计；(d) Novelty and Elegance Argument 可补一句：经本轮四措辞跨领域广谱搜索，因果特征+微调预训练模型融合仍无直接先例，证据强度已进一步加固。

</deep-lit-integration>
