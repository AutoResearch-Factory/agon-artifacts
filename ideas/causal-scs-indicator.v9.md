<!-- 书写报告使用中文 -->
---
topic: topics/0710-causal-scs.md
landscape: topics/0710-causal-scs-landscape.md
workspace: workspace/causal-scs-indicator/
---
- One-sentence summary: 在 HRRR 的真实 `pressure level × longitude × latitude × time` 场上，检验跨气压层、跨邻近网格的因果不稳定性图 `(D,J)` 能否为已在同一原始场端到端训练的强对流预警分类器 (logistic regression / random forest / 小型 CNN) 提供该模型自身学不到的增量，而非与手工统计基线比较；因果估计器是载体，不是方法创新。
- Hypothesis: 强对流预警领域越来越多直接在原始场上端到端训练 ML 模型 (梯度提升树、随机森林、CNN)，但不能默认这类模型已吸收原始场全部可用信息。若跨气压层、跨邻近网格的因果不稳定性在小时级与天级两个滑窗尺度上捕捉到该模型自身表示学不到的信息，把 `(D,J)` 作为额外通道加入同容量、同数据的模型应带来独立增量。两个 lag band 只是滑窗尺度标签，不是已确认的物理机制 (v8 review 指出"1-6h=边界层"/"12-48h=天气尺度"命名易被误读为已验证机制，本版收回该命名，仅作待检验切片)。若已训练的 raw-only 分类器已吸收全部增量，这个负结果同样界定适用边界。
- Expected outcome: 在嵌套 storm-system holdout 中，所有 edge/lag/权重/超参数选择只用内层折，`raw+(D,J)` 相对同输入、同架构、同数据的**已训练** raw-history 分类器 (非统计基线) 取得 held-out `Delta AUPRC >= 0.02`，storm-cluster bootstrap 95% CI 下界 `>0.01`，Brier/ECE 不退步，并跨分类器架构复现。本轮最便宜证伪已完成，是九轮以来首次替换统计基线：P0 确认 `(D,J)` 在真实注入 lag/区域内确有非平凡事件分离信号 (D AUROC 0.71 hour-scale, 0.70 day-scale)；P1 中 logistic regression/random forest 替代百分位统计基线后，raw-only 基线本身显著变强 (mean AUROC 0.75-0.77，超过 v8 的 0.693)，但加入 `(D,J)` 后两分类器族 held-out Delta AUROC 95% CI 均跨零 (NULL)。
- Contribution type: empirical-finding+diagnostic+application
- Contribution drift note: v8 是 `empirical-finding+diagnostic+application`，v9 三类全部保留，无新增无删除。`application` 仍待真实 storm-system 证据挣得，本轮训练分类器仍在合成场上运行。topic 未声明 `preferred-contribution-types`。
- Risk: HIGH
- Estimated effort:
  - Compute: v9 pilot 为几秒 CPU + L4 GPU job 内 <20s；4D mini 仍需 2-5 GPU-hour、40-100 CPU-hour；P1.5 校准约 10-30 CPU-hour；确认性 P2 约 20-50 GPU-hour、1000-3000 CPU-hour。
  - Data: 3 个 HRRR 快照与 SPC schema 已有；连续逐小时序列、匹配非事件标签、风暴移动矢量仍需收集，stop gate 不允许下载。
  - Implementation: 4D mini + P1.5 校准 2-3 周；storm-level 管线、标注、分类器与确认性评估 4-6 周。
- Novelty quick-check: 因果估计器与融合机制本身从不主张新颖；新颖性收窄为一个具体应用空白，即是否有人检验过"跨气压层、多尺度滞后的因果时空不稳定性，能否为已在原始 4D 场端到端训练的强对流预警模型提供该模型自身学不到的信息"。Ganesh SHIPS+ (arXiv:2510.02050, 前身 arXiv:2304.05294) 证明因果衍生特征能增强既有强 baseline，但机制是剪枝/替换 SHIPS 算子而非融合进已训练模型，域是 TC 强度而非强对流。SPACY (arXiv:2411.05331) 与 ST-CND (arXiv:2606.17553) 证明气候网格因果发现是活跃赛道，但前者是月尺度纯水平结构恢复，后者的融合只是可解释性附属分析，都不做 fusion-vs-trained-baseline 检验。Sha et al. (arXiv:2310.06045) 与 Feldmann et al. (arXiv:2406.09474) 确认 HRRR/SPC 强对流 ML 与 AI 模式垂直结构评估都已是活跃赛道，但不涉及因果不稳定性特征。判断标准是分析角度是否被填补，不是因果方法是否新颖 (与 v1-v8 立场一致)；差异化本身不能替代实证。
- Strongest objection: 换成真训练分类器后 raw-only 基线显著变强 (mean AUROC 0.75-0.77，高于 v8 的 0.693)，说明黑箱模型确实能吸收比手工 EWS 统计量更多的信息，这也让 `(D,J)` 更难提供增量，v9 结果 (CI 跨零) 暂未提供正面证据。更根本的问题是，西邻网格边可能只是风暴平流/移动或资料同化传播的伪影，而非真实跨层因果耦合；当前设计 (含历版) 都没有 storm-relative 坐标变换或平流匹配对照，无法排除这一解释，这是尚未解决的已知局限，storm-relative 变换只是候选后续步骤，不是已完成的工作。4D `(D,J)` 图也可能只是昂贵地重编码 raw 梯度与 lag correlation；如果 v9 的负结果成立，这反而支持这一顾虑，而不是反驳它。
- Why we should do this: v1-v8 都用一个从未真正训练过的统计基线，可能低估了原始场自身的信息量。v9 是九轮以来第一次用真正训练的分类器作对照，直接回应 owner 反复澄清的核心问题：因果时空特征能否补充黑箱模型自己学不到的信息，正负结果都能回答这一此前未被正确检验的问题。
- Pilot:
  - Setup: `PILOT_PLAN_V9.md` 复用 v8 冻结的 `four_d_helpers.py`（未改动）。P0：对已知边合成 4D 场计算 `(D,J)` 相对标签的 AUROC，覆盖全部候选 lag 与注入区域内外。P1：用 12 个更丰富的 raw 场特征 (4 气压层 × 窗口均值/标准差/端点差) 训练 `LogisticRegression` 与 `RandomForestClassifier` (冻结超参数)，比较 raw-only 与 raw+4 个因果特征，首 16 对训练/后 16 对留出，覆盖 8 个独立场实现。commit `1f1c18d` 早于 L4 job 36936250。
  - Metric: P0 只求诚实描述信号存在与否，无 pass/fail 阈值。P1 要求至少一个分类器的 held-out Delta AUROC 95% (seed bootstrap) CI 下界 `>0` 才算 POSITIVE，两分类器 CI 上界均 `<0` 才算 NEGATIVE，否则 NULL。
  - Result: P0 显示 `(D,J)` 在真实注入 lag/区域内有真实分离信号 (D AUROC 0.707 @3h、0.699 @24h，明显高于错误 lag 或区域外的约 0.48-0.64)。P1 中已训练 raw-only 基线 mean AUROC 0.753 (logreg)/0.770 (RF)，比 v8 统计基线 (0.693) 更强；加入 `(D,J)` 后 mean Delta AUROC 为 -0.010 [-0.028,+0.010] (logreg) 与 -0.023 [-0.062,+0.012] (RF)，两者 CI 均跨零，单种子方差不小 (RF 最低单种子 -0.144)。
  - Signal: P0 POSITIVE（信号存在，非预警增量声称）；P1 NULL（合成域内、训练分类器比较；不是 REAL-4D 结论）。

- Claims and Claims matrix:
  - C1 (v7-v8, 历史结果不变，详见 `EXPERIMENT_LOG.md`): 等权融合 10/16 格 Delta CI 严格为负、0/16 显著为正，只否定 50/50 规则；alpha sweep 事后无 CI；软件能在已知合成场上形成 `(D,J)` 图并分辨 3h/24h lag，3 个真实 HRRR 快照只证明空间数据通路。不得称已发现真实大气因果结构或预警技能。
  - C2 (v9 本轮新增): 首次用真正训练的分类器做对照：(a) P0 确认 `(D,J)` 在真实注入 lag/区域内确有非平凡事件分离信号，排除"这只是空指标"的最弱反驳；(b) P1 显示，在这一更强的 trained raw baseline 之上叠加 `(D,J)`，两个分类器族的 held-out Delta AUROC 95% CI 均跨零。只说明 32-训练案例、单一合成 DGP、两种架构下未观察到可靠增量；不能外推到真实大气数据、更大样本或其他架构 (如 CNN)。
  - C3: 只有真实 storm-level nested holdout 越过预设实用阈值，且跨分类器架构复现，才允许声称有增量预警技能；因果机制识别不在允许 claim 内。

  | Outcome | 允许的 claim |
  |---|---|
  | REAL-4D POSITIVE | `Delta AUPRC` 点估计与 CI 同时越过阈值、校准不退步且跨 season/分类器架构复现时，限定声称 `(D,J)` 相对同输入同容量的**已训练** raw-history 模型有增量预警技能。 |
  | REAL-4D NULL/NEGATIVE | CI 含零仅称不可判定；CI 上界 `<0.01` 且对照有功效，可称该实现无实用增量；均不外推到其他因果表示或架构。 |
  | 当前状态 | 50/50 SYNTHETIC NEGATIVE (v7)；4D IMPLEMENTATION PASS (v8)；v9 P0 特征-事件 POSITIVE、P1 TRAINED-CLASSIFIER FUSION NULL；REAL-4D 仍 SKIPPED。 |
- Narrative: 主线是把 `S(t,lag-band,pressure-pair,lon,lat,D/J)` 作为一个已训练预警分类器的附加输入通道，问其是否提供该分类器自身学不到的增量，而非问因果估计器本身是否精妙。hour/day lag band 只是解释增量来源的预注册切片，不是已确认机制标签；边界 F1 仍是次要定位端点。
- Experiments:
  - P0（已完成）: v8 HRRR schema + 已知边合成 4D smoke；v9 加入特征-事件关系刻画与训练分类器融合 smoke，只决定该角度是否值得走向真实数据。
  - P1（stop gate 后的最小真实验证）: 3 个日期各取 120h event/non-event chip，固定 `{1000,925,850,500,250}` hPa，跑 `{1,3,6}` h 与 `{12,24,36,48}` h lag，用 PCMCI+/LKIF 生成图，训练同容量 raw-history 分类器与 raw+causal 融合分类器，只报告 lag profile 与点估计，不做三日 skill claim。
  - P1.5（新增，确认性应用前的强制门，回应 v8 review 的排序问题）: graph-only-max null 校准、familywise 校正、window/stride 敏感性、功效分析、storm-relative/平流匹配对照，这几项须在 P2 之前全部通过。
  - P2（确认性应用，门槛为通过 P1.5）: `>=2` seasons、`>=50` 独立 storm systems，2-6h 内 36km 的 tornado/hail/wind，outer storm holdout，cluster bootstrap；比较已训练 raw-history 分类器、非因果 lag 特征分类器、raw+`(D,J)` 融合分类器，三者共享输入/架构/参数预算。
  - P3（归因与稳健性）: full 4D 对比 single-column/horizontal-only/hour-only/day-only；检验强迫 regime 假说 (不预先命名机制)；storm-relative 坐标变换纳入本阶段以排除平流伪影；边界 overlap 为次要端点。
- Assets status: HRRR/SPC smoke 资产、v7-v9 合成结果与环境已就绪 (scikit-learn 已加入依赖)；连续 4D 数据仍被 stop gate 阻塞，细节见 `workspace/causal-scs-indicator/data/MANIFEST.md`。

<review date="2026-07-12">

## Novelty

- Score: 3/10 (Claude 与 codex 独立评估一致，与 v8 不变)
- Closest prior work:
  1. **Ganesh, Beucler, DeMaria, Runge (2025), Multidata-PC SHIPS+** (arXiv:2510.02050) — v7/v8 已通读全文，本轮 Claude 与 codex 独立复核其精确机制表述：**v9 idea 正文称 SHIPS+ 的机制是"剪枝/替换 SHIPS 算子"，这一表述不准确**。Claude 独立核验其摘要原文为 "We build an extended predictor set (SHIPS+) by adding selected features to the standard SHIPS predictors"——即因果发现选出的新预测因子是**增补**进已有 SHIPS 预测因子集 (augmentation)，而非剪枝或替换。codex 独立指出同一问题。这一表述纠正会**收窄**而非消除 v9 与 SHIPS+ 的差异化空间：v9 仍有一个真实、更精细的设计差异 (在一个已独立训练好的 raw-only 模型之上做受控的增量 ablation，而非像 SHIPS+ 那样直接构建一个联合增强预测因子集重新训练), 但 v9 novelty quick-check 现有措辞夸大了这一差异, 下一版应改为准确表述。
  2. **Yu, Liang (2026), ST-CND** (arXiv:2606.17553) 与 **Wang et al. (2025), SPACY** (arXiv:2411.05331) — v7/v8 已通读全文，结论不变：占据相邻的空间因果诊断/网格因果发现赛道，但不做 fusion-vs-trained-baseline 检验。
- Key differentiator: 因果估计器与融合机制本身不主张新颖；差异化收窄为一个具体应用空白——"因果时空不稳定性能否为一个已在原始 4D 场端到端训练的强对流预警模型提供增量"。但如 Quality 部分所述，v9 本轮实际测试的对照 (12 个手工聚合特征上的 LogisticRegression/RandomForest) **并非**严格意义上"在原始 4D 场端到端训练"的模型，故这一差异化目前仍是未兑现的设计意图，而非已完成的实证检验。本轮新增背景文献 (见 landscape 末尾 [idea-reviewer, 2026-07-12] 条目)：Flora et al. (2026, arXiv:2603.20250) 展示 U-Net 已在 WoFS 原始网格张量上真实端到端训练用于 2-6h 强对流预警，是未来 P2 阶段构建"真正 raw-tensor baseline"时可参照的具体先例；(2025, arXiv:2512.24440) 独立佐证"已训练端到端天气模型内部会自发学到物理有意义结构"这一前提在别处也有证据，均不构成对本 idea 的直接撞车。

## Quality

评估视角：topic (`topics/0710-causal-scs.md`) 未声明 `target-venue`/`preferred-contribution-types` (Claude 与 codex 独立核实一致)。沿用 v7-v8 review 的"顶级 Earth-system methods / AI4Science / 强对流预测应用"稿件标准。

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 4/10 (Claude 初评 5，codex 给 3，综合采纳 codex 具体证据后下修为 4) | **本轮最核心的发现 (Claude 与 codex 独立收敛一致)**：owner 第三次澄清明确要求对照是"在原始 4D 场上端到端训练的分类器 (而非手工统计特征)"，但 v9 P1 实际训练的 raw-only 分支输入是 12 个手工聚合特征 (每层窗口 mean/std/endpoint-delta 的 top-decile-mean)，空间结构、时间顺序和绝大多数网格值在训练前已被压缩掉——这仍是手工特征工程，不是"端到端在原始张量上训练"，也未使用 idea 自己在 One-sentence summary/Hypothesis 中列出的候选之一 (CNN)。Claude 独立读码确认两个 codex 未展开但同样成立的具体子问题：(a) **时间窗口不对齐**——`_raw_ml_features`(`ml_fusion_helpers.py`) 只取最近 24 h (`fields[..., -config.window_hours:]`)，而 `(D,J)` 特征来自横跨近全部 96 h 的滚动窗口回归 (`rolling_beta_maps`)，raw 分支实际看到的历史远短于因果通道隐含利用的历史，这一信息不对等本身就可能压低 raw baseline 的公平上限；(b) **day-scale 边的 J/D+J 通道不满足真实 lag 特异性**：P0 JSON 显示 day 边错误 lag=36h 的 `J`=0.6338、`D+J`=0.6387 均**高于**真实 lag=24h 的 `J`=0.5820、`D+J`=0.6309——P0 "POSITIVE"结论只对 D 通道成立，对 J 和 D+J 通道不成立，idea 正文未点明这一区分。此外 raw(12)/fusion(16) 特征维度不匹配，缺 v4 review 以来反复要求的 dimension-matched sham/noncausal 对照。 |
| Missing evidence signals | 3/10 (Claude 与 codex 独立评估一致) | 最关键缺口：同一完整历史、同一空间分辨率、同容量的小型 CNN/U-Net raw-tensor baseline 与 raw+sham 维度匹配对照 (codex 具体点出，Claude 认同)。P0 仅用单一 seed (`20260901`) 计算 36 个 AUROC，无 bootstrap CI、无 multiplicity 校正，而 P1 已经为 8 个 seed 重新计算过 maps，把 P0 扩展到 8-seed 几乎零增量成本却未做——这是本轮一个容易弥补的疏漏。仍缺真实 PCMCI+/LKIF (仍是 fixed-edge local regression proxy)、lag-specific null、有效窗口数/重叠校正 (见下)、storm-relative/advection 对照的实际执行、真实 storm 证据。 |
| Narrative | 5/10 (Claude 初评 6，codex 给 5，采纳 codex 更严格的读法下修为 5) | v9 正确地把主线改为"conditional incremental information audit"，诚实报告 P1 为 NULL 而非 NEGATIVE，收回 hour/day 机制命名 (v8 review 要求已落实)。但 One-sentence summary 与 Hypothesis 两处均反复使用"在同一原始场端到端训练的...分类器"这一措辞描述实际是手工特征上的 LR/RF，这不是孤立的一次性用词失误，而是贯穿全文的核心叙事定位，构成事实性失准 (Claude 与 codex 独立认定)。另外 Experiments 段落中 storm-relative/advection 对照同时出现在 P1.5 ("须在 P2 之前全部通过") 与 P3 ("纳入本阶段以排除平流伪影")，未说明这一最大的未解决技术风险究竟由哪个阶段负责解决 (Claude 独立发现，codex 独立指出同一处歧义)。 |
| Venue contribution | 2/10 (Claude 初评 3，codex 给 2，采纳更严格评估下修为 2) | 仍是单一、标签依赖的 coupling 被设计植入的 synthetic DGP；即便 P1 设计完全公平，也只是 pilot 级证据，不足以构成任何顶会/顶刊独立贡献。相对 v8 (纯 implementation smoke) 本轮确有真实的假设检验内容 (P0+P1 拆解)，但科学证据基础仍是 100% 合成。 |
| Testability | 5/10 (Claude 初评 7，codex 给 5，采纳 codex 具体证据下修为 5) | Expected outcome 对 REAL-4D 阶段给出的 Delta AUPRC/CI/校准/storm holdout 标准是清楚的，但本轮"最便宜证伪"实际使用的决策规则是 Delta **AUROC** 的 seed-bootstrap CI (`ml_fusion_helpers.py`只对 `delta_auroc` 算 CI)，而正式 Expected outcome 与 Claims matrix 的 primary 指标是 Delta **AUPRC**——JSON 里有 AUPRC 点估计但没有对应 CI，两套指标未统一 (Claude 独立核验 JSON 结构确认，codex 独立指出同一问题)。且由于 P1 未使用 raw tensor，该 pilot 客观上没有直接检验 owner 的核心问题，只检验了一个更弱的替代问题。 |
| Outcome realism | 4/10 (Claude 与 codex 独立评估一致) | `(D,J)` 是同一 raw history 的确定性衍生量，信息论上不能凭空增加信息，收益只能来自有限样本下更好的归纳偏置。本轮即便在对因果特征有利的条件下 (oracle true lag、比 raw 分支更长的隐含历史窗口、比 v8 更丰富的 raw 特征集)，两个分类器族仍是负点估计且 CI 跨零——这使 REAL-4D 阶段 `Delta AUPRC>=0.02`、CI 下界 `>0.01` 的门槛前景偏低，但不是不可能 (v9 尚未在真正 raw-tensor 对照上检验过)。 |
| Contribution type compliance | n.a. | idea types = {empirical-finding, diagnostic, application}；topic 未声明 `preferred-contribution-types`，跳过，不计入 Overall Quality 平均 (Claude 与 codex 一致)。 |
| Overall Quality | 4/10 (Claude 独立初评 (5+3+6+3+7+4)/6≈4.7→5，codex 给 (3+3+5+2+5+4)/6≈3.7→4；采纳 codex 具体可验证证据后 Claude 收敛到 (4+3+5+2+5+4)/6≈3.8→4，与 codex 一致) | v9 在问题排序 (P0 先于 P1)、机制命名收回、诚实标注 NULL 而非 NEGATIVE 上都是真实进步，但本轮最重要的 owner 澄清 (对照必须是"原始场端到端训练"而非手工特征) 实际上仍未被真正兑现，这是本轮 Overall Quality 未能超过 v8 的核心原因。 |

## Contribution Drift (n >= 2)

- v_{n-1} (v8) contribution types: {empirical-finding, diagnostic, application}
- v_n (v9) contribution types: {empirical-finding, diagnostic, application}
- Status: unchanged
- Hard cap triggered: no (topic 未声明 `preferred-contribution-types`；即便声明，本轮也无扩张/删除，Claude 与 codex 独立核验一致)

### v8 `<review>` concern 逐条复核 (Claude 独立核验 + codex 独立核验，两者收敛一致)

| v8 concern | Status | Assessment |
|---|---|---|
| "1-6h=边界层"/"12-48h=天气尺度"命名易被误读为已验证机制 | **resolved** | v9 Hypothesis 明确收回该命名，仅作"待检验切片"，Claude 与 codex 独立核实一致。 |
| 西邻网格边可能只是平流/风暴移动/资料同化传播的伪影 | **partially resolved** | Strongest objection 诚实保留此风险为未解决限局，P1.5 把 storm-relative/advection-matched 对照列为 P2 前置门；但对照尚未实际执行，且 P3 又把 storm-relative 坐标变换列为"本阶段"任务，构成该风险究竟由 P1.5 还是 P3 解决的歧义 (Claude 与 codex 独立发现同一处)。 |
| P2 (确认性应用) 排在 P3 (归因与稳健性/校准) 之前，排序有误 | **resolved** | v9 新增 P1.5，明确 gate P2，顺序已修复，Claude 与 codex 独立核实一致。 |
| 120h 在 48h lag 下只有 9 个高度重叠滚动窗口；不同 lag 对应不同窗口数会系统性偏置跨 lag D/J 可比性 | **partially resolved / largely ignored** | Claude 独立验证 `rolling_beta_maps` 代码：hour 族 (`max(hour_lags)=6`) 产生 12 个滚动窗口，day 族 (`max(day_lags)=36`) 只产生 7 个滚动窗口，两族窗口数仍不同，该结构性差异未被修复，也未在 Strongest Objection 或别处被重新提及。P1.5 列出"window/stride 敏感性"作为待办，但没有给出具体的等效窗口数规则或数值 gate。 |
| 缺一个真正看到完整 raw history 的、经过真正训练的强对照模型 | **claimed resolved, actually ignored** | v9 REFINEMENT_LOG 声称"Accept, 已用真正训练的分类器重新设计"，但实际实现仍是最近 24h 窗口上的 12 个手工聚合统计量，不是原始 4D 张量，也不是 CNN——这是 Claude 与 codex 独立收敛认定的本轮最严重未解决项 (见 Logical gaps)。 |
| 缺 graph-null 校准、familywise 校正、功效分析、非因果/sham 对照 | **partially resolved** | 多数被写入 P1.5/P2 待办，但均未实际执行、无冻结数值阈值；当前 P1 的"任一分类器 CI 下界>0 即 POSITIVE"规则本身也没有跨分类器 multiplicity 校正 (codex 指出，Claude 认同)。 |
| 真实数据证据仍是 0%，venue contribution 未挣得 | **acknowledged, not resolved** | 仍是 100% 合成；v9 没有把 application 声称为已完成，如实标注 SKIPPED/待验证，非隐瞒。 |
| Alternative Framing 建议 1 (收紧为 incremental-utility/failure-boundary audit，不预注机制名) | **resolved** | v9 主叙事已采纳。 |
| Alternative Framing 建议 2 (v8 review, Claude 主张的"regime localization 作为更快替代主线") | **partially resolved, pushback 有据** | v9 未采纳为主线，仅保留于 P3 作次要归因问题。鉴于 owner 明确要求先回答"trained-raw baseline 之上的条件增量"这一问题，refiner 拒绝把更廉价的替代框架提升为主线是有理有据的，不算未回应。 |
| 持续 8 轮负面证据应被正面承认，而非继续做纯文字打磨 | **resolved** | v9 明确承认 9 轮以来因果特征从未在预注册规则下击败 raw baseline，并将本轮 P1 结果诚实判为 NULL 而非直接宣称阶段性胜利。 |

## Alternative Framing

Claude 与 codex 收敛到同一个建议 (均不需要引入 topic `preferred-contribution-types` 之外的类型，topic 本身未声明该字段)：把当前 P1 明确重新命名为"trained tabular-summary-feature fusion stress test"，不要在 One-sentence summary/Hypothesis 中暗示 owner 的"raw-4D 端到端训练模型"问题已经被回答。真正贴合 owner 意图的下一版主线应是一个两阶段 conditional-information audit：(1) 先用多 seed、matched-pair bootstrap CI 重新刻画 P0 (对 D、J、D+J 三通道分别给出可信区间，明确日边 J/D+J 当前不满足 lag 特异性)；(2) 再在完全相同的信息时间窗口与模型容量约束下，比较 raw-tensor CNN/U-Net (如 arXiv:2603.20250 WoFS 先例的量级)、raw+dimension-matched-sham 与 raw+`(D,J)` 三者。这不改变已声明的 contribution types，但会显著提高该 idea 下一版实验设计对 owner 核心问题的直接检验力度。

## Claims Discipline

Claude 与 codex 收敛后在 idea 正文基础上收紧的版本：

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | P0 目前只在单一 seed、无 CI 的描述性意义上支持"`D` 通道在真实注入 lag/区域内有分离信号"；对 `J`/`D+J` 通道、尤其是 day 边，当前数据不支持真实 lag 特异性 (错误 lag=36h 的 J/D+J 反而更高)，需多 seed bootstrap 后才能升级为可信声称。REAL-4D 阶段只有在**真正的 raw-tensor 对照** (而非手工聚合特征分类器)、相同历史时间窗、匹配容量、nested storm holdout、P1.5 全部通过、Delta AUPRC 点估计与 CI 同时越过预设阈值且跨 season/architecture 复现时，才能限定声称 `(D,J)` 提供了该模型自身学不到的增量预警技能。不得声称"产生了新信息"或"识别了真实因果机制"。 |
| NULL | 当前证据只支持："在 32-training-case、单一合成 DGP、两个 tabular 分类器 (输入为 12 个手工工程化 raw 聚合特征，非原始张量) 上，加入 oracle-true-lag 的 `(D,J)` 未观察到可靠的 held-out Delta AUROC 增量"。不能写成"已训练的 raw-4D end-to-end 模型已吸收全部增量"——这一表述目前没有被真正检验过。REAL-4D 阶段 CI 含零仍只说明当前设计不可判定。 |
| NEGATIVE | 只有在公平的 raw-tensor comparator (而非手工特征代理)、功效充分、且 Delta AUPRC CI 上界低于预注册实用阈值时，才能称该具体 `(D,J)` 实现在限定数据/lead time/架构下没有实用增量，且不外推到全部因果表示。当前两个负点估计 + CI 跨零的组合，因对照本身不是 owner 要求的对照类型，尚不构成 NEGATIVE，只能算 NULL。 |

## Likelihood-Impact Matrix

- Priority: Medium = Likelihood: Low x Impact: High
- Numeric score for ideas.xml: 5
- Rationale:
  - Likelihood: Claude 与 codex 独立评估完全一致，判定 Low，无分歧。理由：owner 要求的核心对照 (真正 raw-tensor/端到端模型) 本轮仍未运行；即便在对因果特征相对有利的条件 (oracle true lag、比 raw 分支更长的隐含历史窗口、比 v8 更丰富的 raw 特征集) 下，本轮合成 fusion 检验仍是 NULL。真实阶段还需要短窗高维因果估计、advection 排除的真实执行、P1.5 全部校准通过、连续 HRRR/SPC 数据获取解除 stop gate、跨 >=50 个独立 storm systems 泛化同时成立——条件连乘，且新暴露的时间窗口不对齐/维度不匹配/日边 lag 特异性缺失等问题意味着下一版还需要先修正实验设计本身。
  - Impact: Claude 与 codex 独立评估完全一致，判定 High，无分歧。理由：若最终在严格的 raw-tensor baseline 之上，跨季节、跨架构获得真实、可信的增量 (或给出功效充分的清晰负面边界)，将直接回答"因果衍生表示能否为现代天气 ML 提供该模型自身学不到的归纳偏置"这一问题，对相关研究线有明确价值；但 SHIPS+ 已证明"因果发现特征增强既有强预测模型"这一一般路线可行 (且本轮 Claude 核实其机制是特征增补而非剪枝/替换，比 v9 novelty quick-check 描述的更接近 v9 自己的融合范式)，ST-CND/SPACY 证明气候网格因果发现本身是拥挤赛道，故不到 Exceptional。
  - 查表：Low x High = Medium (5)。topic 未声明 `preferred-contribution-types`，hard cap 不适用。Claude 与 codex 在 Likelihood 和 Impact 两个轴上均无分歧，无需额外标注。

## Overall

- Priority: Medium
- Score: 5
- Comments: v9 是九轮以来在实验排序 (P0 先于 P1，回应 owner 第四次澄清) 和诚实结果判定 (NULL 而非 NEGATIVE 或夸大的 POSITIVE) 上执行最好的一轮，也是第一次跳出纯统计基线、引入真正训练的分类器。但 Claude 与 codex 独立收敛认定：owner 第三次澄清中最核心的要求——对照必须是"在原始 4D 场端到端训练的模型，而非手工统计特征"——本轮实际上仍未被兑现 (P1 的 raw 分支是 12 个手工聚合特征上的 LR/RF，还有相对因果通道更短的隐含历史窗口)，这意味着本轮 P1 的 NULL 结果回答的是一个比 owner 实际问题更弱的替代问题。这不是文字包装问题而是实验设计问题，建议下一版把 P1 明确重新定位为"trained tabular-summary baseline"的中间检验，并把真正的 raw-tensor/CNN 对照列为下一版的 P0.5 或 P1 必做项，而不是继续在当前设计上做文字微调。Claude 与 codex 在 Likelihood、Impact 两个轴上完全一致，无需标注分歧；Quality 各子项 codex 普遍比 Claude 初评更严格，Claude 采纳其具体可验证证据 (SHIPS+ 机制核验、day 边 J/D+J 非 lag-specific、时间窗口不对齐、AUROC/AUPRC 指标不统一) 后下修收敛，Overall Quality 最终与 codex 一致 (4/10)。

超长警告: 文件 10.1 KB，超过 10 KB 上限约 1.1%

</review>
