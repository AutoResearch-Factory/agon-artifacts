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
- Novelty quick-check: 因果估计器与融合机制本身从不主张新颖；新颖性收窄为一个具体应用空白——是否有人检验过"跨气压层、多尺度滞后的因果时空不稳定性，能否为已在原始 4D 场端到端训练的强对流预警模型提供该模型自身学不到的信息"。Ganesh SHIPS+ (arXiv:2510.02050, 前身 arXiv:2304.05294) 证明因果衍生特征能增强既有强 baseline，但机制是剪枝/替换 SHIPS 算子而非融合进已训练模型，域是 TC 强度而非强对流。SPACY (arXiv:2411.05331) 与 ST-CND (arXiv:2606.17553) 证明气候网格因果发现是活跃赛道，但前者是月尺度纯水平结构恢复，后者的融合只是可解释性附属分析，都不做 fusion-vs-trained-baseline 检验。Sha et al. (arXiv:2310.06045) 与 Feldmann et al. (arXiv:2406.09474) 确认 HRRR/SPC 强对流 ML 与 AI 模式垂直结构评估都已是活跃赛道，但不涉及因果不稳定性特征。判断标准是分析角度是否被填补，不是因果方法是否新颖 (与 v1-v8 立场一致)；差异化本身不能替代实证。
- Strongest objection: 换成真训练分类器后 raw-only 基线显著变强 (mean AUROC 0.75-0.77，高于 v8 的 0.693)——印证黑箱模型能吸收比手工 EWS 统计量更多的信息，也让 `(D,J)` 更难提供增量，v9 结果 (CI 跨零) 暂未提供正面证据。更根本地，西邻网格边可能只是风暴平流/移动或资料同化传播的伪影而非真实跨层因果耦合；当前设计 (含历版) 都没有 storm-relative 坐标变换或平流匹配对照，无法排除这一解释——尚未解决的已知局限，storm-relative 变换是候选后续步骤而非已完成工作。4D `(D,J)` 图也可能只是昂贵地重编码 raw 梯度与 lag correlation；v9 的负结果若成立，反而支持这一顾虑而非反驳它。
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
  - P1.5（新增，确认性应用前的强制门，回应 v8 review 的排序问题）: graph-only-max null 校准、familywise 校正、window/stride 敏感性、功效分析、storm-relative/平流匹配对照——须在 P2 之前全部通过。
  - P2（确认性应用，门槛为通过 P1.5）: `>=2` seasons、`>=50` 独立 storm systems，2-6h 内 36km 的 tornado/hail/wind，outer storm holdout，cluster bootstrap；比较已训练 raw-history 分类器、非因果 lag 特征分类器、raw+`(D,J)` 融合分类器，三者共享输入/架构/参数预算。
  - P3（归因与稳健性）: full 4D 对比 single-column/horizontal-only/hour-only/day-only；检验强迫 regime 假说 (不预先命名机制)；storm-relative 坐标变换纳入本阶段以排除平流伪影；边界 overlap 为次要端点。
- Assets status: HRRR/SPC smoke 资产、v7-v9 合成结果与环境已就绪 (scikit-learn 已加入依赖)；连续 4D 数据仍被 stop gate 阻塞，细节见 `workspace/causal-scs-indicator/data/MANIFEST.md`。
