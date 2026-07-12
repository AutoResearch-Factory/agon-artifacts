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

<review date="2026-07-12">

## Novelty

- Score: 3/10 (Claude 与 codex 独立评估一致)
- Closest prior work:
  1. **Ganesh, Beucler, DeMaria, Runge (2025), Multidata-PC SHIPS+** (arXiv:2510.02050) — 已在真实业务台风数据、真实技能指标上验证"因果发现衍生特征增强既有强 baseline"这一问题框架 (v7 review 已核验全文, 本轮结论不变)。
  2. **Yu, Liang (2026), ST-CND** (arXiv:2606.17553) — 空间因果网络诊断 + 传统 EWS 联合读出的最近邻先例, 但融合只是可解释性附属分析 (v7 review 已核验全文, 本轮结论不变)。
  3. **Wang, Varambally, Watson-Parris, Ma, Yu (2025), SPACY** (arXiv:2411.05331, ICML 2025) — 本轮新发现, 已读全文 (`main.tex` + `introduction.tex` + `experiments.tex`) 并追加到 landscape。变分推断联合学习空间因子与滞后/瞬时因果图, 在连续空间域下证明可辨识性, 真实全球月尺度气候网格数据 (MJO/NAO/AAO 遥相关) 上恢复已知因果结构。核验结论：这是又一个"气候网格尺度因果发现"竞争性先例, 但 (a) 只在纯水平 2D 网格上操作, 无气压层/垂直轴; (b) 月尺度遥相关滞后 (1-2 个月), 非小时/天尺度强对流; (c) 贡献是更好的因果结构恢复方法/可辨识性理论, 不是"不稳定性即信号"框架, 也不做 fusion-vs-raw 预警增量检验。不构成对本 idea 核心 claim 的直接撞车, 但进一步印证"气候网格因果发现"本身早已是拥挤赛道。
- Key differentiator: 加权/学习融合机制、causal estimator (PCMCI+/LKIF/fixed-edge proxy) 均明确非本 idea 的新颖性主张。v8 新增的跨气压层 + 双时间尺度 (小时/天) 空间图表示法，相对 SHIPS+ (无空间网格)、ST-CND (2D 地理网格, 无垂直层)、SPACY (2D 网格, 无垂直层, 月尺度) 三个最近先例而言是一个真实存在的技术差异化点——目前没有已知工作在**真正的多气压层 (垂直) x 水平网格 x 多尺度滞后**表示上做因果不稳定性诊断。但这一差异化点本轮仍未产出任何真实发现 (仍是 SKIPPED)，按 novelty-check 规则 ("apply X to Y" 不算新颖除非揭示意外洞见；若方法不新但 finding 会新则应明说) 本身不能救novelty：既没有意外洞见，也没有 finding，只是一个更精细的特征构造摆在同一个已被验证过的问题框架上。此外 Sha et al. (2023, arXiv:2310.06045, HRRR+SPC 真实数据上 CGAN+CNN 强对流概率预报) 和 Feldmann et al. (2024, arXiv:2406.09474, AI 全球模式在强对流环境垂直结构诊断变量上的技能评估) 两篇非因果背景文献 (本轮新发现, 已追加 landscape) 进一步确认"HRRR/SPC 强对流 ML 后处理"和"垂直结构对强对流环境评估很重要"这两个观察本身在应用域内均已是活跃、已有真实技能数字的赛道, 不是本 idea 能贡献的新洞察。

## Quality

评估视角：topic (`topics/0710-causal-scs.md`) frontmatter 与正文均未声明 `target-venue`/`preferred-contribution-types` (Claude 与 codex 独立核实一致)。按 owner 的应用价值定位，视角沿用 v7 review 的"顶级 Earth-system methods / AI4Science / 强对流预测应用"稿件标准。

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 5/10 (Claude 初评 6, 采纳 codex 更具体的四点拆解后下修为 5) | 正面：v7 review 指出的三处问题 (10/16 计数、PILOT_PLAN_V7 内部矛盾、三项限制补充) 经 Claude 独立对照原始 JSON/git log 核验、codex 独立复核，**全部确认已修复** (见下方"v7 concern 逐条复核")。v8 全部新增数字 (hour/day lag 选中值、空间 contrast +0.04485/+0.02496、train alpha tie 0.7656、held-out raw/causal/fusion 0.693/0.615/0.693/Delta=0、HRRR shape 2x1059x1799) 经 Claude 逐一与 `results/four_d_smoke_results.json`/`results/fusion_pilot_results.json` 原始字段核对，**零误差、完全一致**——这是 8 轮以来数字保真度最好的一轮。但 codex 指出四个新缺口，Claude 核实后认为均成立：(a) "1-6h=边界层"/"12-48h=天气尺度" 只是待验证的标签，滞后时长本身不能识别物理 regime，idea 文本目前把尺度标签写得像是已确认的机制归属；(b) 西邻网格边很容易只是平流/风暴移动/资料同化传播的伪装，而非真实跨层因果耦合，P1/P2 设计尚无 storm-relative/advection-matched 对照来排除这一点；(c) 120h 在 48h lag 下只有 9 个高度重叠的滚动窗口，这是"可定义 D/J"的算术下限，不是 PCMCI+/LKIF 统计稳定估计的下限，且不同 lag 对应不同窗口数，会机械地系统性偏置跨 lag 的 D/J (尤其 J=最大 jump 统计量) 可比性；(d) P2/P3 排序问题 (见下)。另外，v8 自身的 `PILOT_PLAN_V8.md` 又出现了本轮的新一次"预注册后自行发现算术错误"(原文写 72h 实际需要 120h)——这是连续第二轮 (v7: 12/16 vs 10/16；v8: 72h vs 120h) 出现事后纠正，每次都被诚实披露，但频率本身值得作为过程纪律信号记录。 |
| Missing evidence signals | 3/10 (Claude 初评 2, 采纳 codex 补充的具体缺口上修为 3) | 沿用 v4-v7 连续未解决的缺口 (全规模 PCMCI+/LKIF、Causal-CCP baseline、bootstrap multiplicity 校正、graph-only-max 格独立 null 校准) 之外，codex 独立指出且 Claude 核实成立的新缺口：真实数据阶段仍缺 lag-specific null、advection/风暴移动对照、独立 regime 标签、正式 power analysis，以及"同样看到完整 120h 历史"的强 raw 模型对照——这些在拟议的 P2 决定性对照组设计中目前完全没有着落。4D smoke 本身也比 v4-v7 的 fusion pilot 少了一个 sham/非因果对照通道 (只有 raw/causal/fusion 三列，没有v4-v7一直带着的 sham-conditioned 基线)，是该具体 artifact 上的一个小退步，虽然只是实现 smoke 不是科学检验。真实数据仍是 0%：3 个 HRRR 快照只做了 schema check，无一 real lag/predictor-label pair/skill estimate。 |
| Narrative | 6/10 (Claude 初评 6, codex 给 7, 综合取 6) | 相对 v7 有真实改善：精确回应了 owner "pressure x lon x lat x time" 的第二次澄清，boundary F1 已被明确降级为 secondary endpoint，primary 改为 2-6h severe-hazard skill (v7 review 指出的端点错位已解决，见下)。Claims matrix 也诚实区分了 implementation PASS 与 scientific SKIPPED。但 codex 指出且 Claude 认同的薄弱处：'synoptic-scale causal map'/'boundary-layer-scale causal map' 这类命名容易让读者把"滞后尺度标签"误当成"已识别的物理机制"，更准确的主线应是"pressure-resolved multi-lag estimator-instability 表示的增量效用与失效边界审计"，把尺度标签严格限定为待检验的解释性切片而非已确认机制。此外，8 轮以来 (v1-v8) 因果特征在任何合成设置下都从未在预注册规则下击败 raw baseline，这个持续的负面积累本身也在弱化"继续做 idea 级别文字打磨"的边际价值。 |
| Venue contribution | 2/10 (Claude 与 codex 独立评估一致) | 本轮新增证据只是一次 schema check 加一次设计保证的 synthetic implementation smoke，完全不足以构成任何顶会/顶刊的独立贡献，即便是最宽松的应用价值标准。只有 P2 在多季节独立 storm systems 上真正越过预设实用阈值、或给出功效充足且跨估计器复现的明确负边界，才可能成为强贡献。 |
| Testability | 7/10 (Claude 初评 6, codex 给 8, 综合取 7) | Expected outcome 给出了具体的 Delta AUPRC、CI、Brier/ECE、嵌套选择规则与 hour/day ablation，已完成的 known-edge 4D smoke 也是一个设计合理、真正可能失败 (若空间/滞后门未通过则会真的 FAIL) 的最便宜实现证伪，边界诚实标注为"只能证伪实现，不能证伪科学假设"。扣分点：P1 仍缺数值稳定性/窗口重叠 gate，且 stop-gate 解除规则、P2 前置校准规则均未冻结。 |
| Outcome realism | 4/10 (Claude 初评 3, codex 给 4, 综合取 4) | 同一份 raw history 的因果衍生表示，在信息论上不可能凭空增加信息，收益只能来自有限样本下更好的归纳偏置；raw EWS/raw-history 在 8 轮里的每一次合成检验 (包括本轮新的 4D held-out fusion, alpha=1.0 被选中、Delta=0) 都保持占优，confound-only/mixed 格 raw AUC 天花板 (0.985-0.998) 进一步压缩现实中因果特征能贡献的增量空间。绝对 Delta AUPRC>=0.02、CI 下界>0.01 的门槛因此合理但极具挑战性。 |
| Contribution type compliance | n.a. | idea types = {empirical-finding, diagnostic, application}；topic 未声明 `preferred-contribution-types`，跳过，不触发 hard cap (Claude 与 codex 一致)。 |
| Overall Quality | 4/10 (Claude 独立初评 (5+3+6+2+7+4)/6=4.5→4, codex 独立给出 5/10；分歧在 1 分以内, 未构成需要标注的 Likelihood/Impact 级别分歧) | v8 在数字保真度、v7 遗留问题修复、owner 第二次澄清的实质回应上都有可核实的真实进步，是 8 轮里执行纪律最好的一轮；但科学证据基础完全没有移动 (仍 100% 合成)，且新暴露了 P2/P3 排序问题 (见下)、lag-window 可比性问题与"标签当机制"的表述风险。 |

## Contribution Drift (n >= 2)

- v_{n-1} (v7) contribution types: {empirical-finding, diagnostic, application}
- v_n (v8) contribution types: {empirical-finding, diagnostic, application}
- Status: unchanged
- Hard cap triggered: no (topic 未声明 `preferred-contribution-types`；即便声明，本轮也无扩张/删除，Claude 与 codex 独立核验一致)

### v7 `<review>` concern 逐条复核 (Claude 独立核验 + codex 独立核验，两者收敛一致)

| v7 concern | Status | Assessment |
|---|---|---|
| "12/16 negative" 算术错误 | **resolved** | Claude 用 Python 独立重算 `results/fusion_pilot_results.json` 16 个 cell：10 个严格为负、6 个含零 (其中 4 个是 `g=0,h=0` 完全退化格)、0 个显著为正；4 个 primary (`graph-only-max`) 格中 2 负 2 零。与 v8 idea 正文 C1 的表述逐字匹配。`EXPERIMENT_LOG.md` 有明确的 "Correction ledger" 段落，承认历史 commit `c5da5c6` 的错误表述且不篡改历史记录，只在新记录中更正——这是诚实、可审计的纠正方式，不是掩盖。codex 独立复核结论一致。 |
| `PILOT_PLAN_V7.md` "Contribution scope unchanged: empirical-finding + diagnostic" 与 idea 三元声明矛盾 | **resolved** | Claude 用 `git show b68a229 -- PILOT_PLAN_V7.md` 核验：该矛盾行在 v8 的同一预注册 commit 中被改写为 "Contribution target: empirical-finding + diagnostic + application...it remains contingent on real storm-system evidence and is not earned by this synthetic pilot"，commit message 明确写"align the application contribution target"。原矛盾 commit `9c98283` 未被重写，历史保留，纠正记录在案。codex 独立复核结论一致。 |
| 三项待补充限制 (等权失败不证伪主假设 / alpha 扫描双重事后选择 / confound 格 alpha=1 的 raw-AUC 天花板替代解释) | **resolved** | 三点均已原样出现在 v8 的 Strongest objection 与 Claims C2 中，Claude 核实措辞与 v7 review 原文一致。codex 独立复核结论一致。 |
| P3 (H2) 端点 (边界 F1) 与"预警"目标错位 | **resolved** | `REFINEMENT_LOG.md` v8 段落明确 "Boundary F1 is not a warning endpoint | Accept | Severe-hazard skill at a fixed lead becomes primary; boundary overlap is a secondary mechanism/localization endpoint."，v8 Experiments 段落也已体现。 |
| `fusion_helpers.py` 无配套单元测试 | **resolved** | `git log` 显示 `tests/test_fusion_helpers.py` 在 v8 的预注册 commit `b68a229` 中首次出现 (v7 review 时确实不存在，该项 v7 批评本身准确)，是本轮偿还的技术债。 |

**新发现的未解决/新增问题** (codex 独立发现，Claude 核实成立，非 v7 review 已提出的范围)：P2 (确认性应用) 与 P3 (归因与稳健性) 的顺序有误——graph-only null calibration、familywise correction、window/stride 敏感性、PCMCI+/LKIF 复现这些**理应作为 P2 决定性检验前置门槛**的校准工作，被 v8 Experiments 段落排在 P3 (P2 之后)。合理顺序应是：P1 (schema/吞吐量) → 新增 P1.5 (lag-specific null、平流/地形控制、窗口稳定性、graph-only calibration、冻结 edge library 与算力预算) → P2 (多风暴确认性检验) → P3 (机制归因与次要稳健性)。当前顺序直接从三日期 P1 跳到 P2 不够稳妥，建议下一版调整。

## Alternative Framing

Claude 与 codex 收敛到互补的两点建议（均不需要引入 topic `preferred-contribution-types` 之外的类型，topic 本身未声明该字段）：**(1, codex 主张, Claude 认同) 收紧主张边界**：把主线严格限定为"对 pressure-resolved multi-lag estimator-instability 表示做预注册的增量效用与失效边界审计"，hour/day 和气压层配对仅作为解释异质性的预注册切片，不预先把 1-6h/12-48h 命名为"边界层"/"天气尺度"这类已识别的物理机制——这样可以避免 Narrative 中指出的"尺度标签当机制归属"风险，且不改变已声明的 contribution types。**(2, Claude 主张，codex 未提出但不冲突) 降低对昂贵确认性应用的路径依赖**：鉴于 8 轮里因果特征在任何合成设置下都从未在预注册规则下击败 raw baseline，且 P2/P3 的多季节、>=50 storm systems 确认性应用仍被 stop gate 完全阻塞，一个更容易在现有 3 个日期 120h chip 上短期兑现的备选框架是——即使 (D,J) 融合始终不能提供预警增量，pressure-lag 空间图本身能否提供一个物理上可解释的、可复现的多尺度强迫 regime (天气尺度 vs 边界层尺度) 定位，并与已知的环境/风暴模态转换相关联？这仍落在 `{empirical-finding, diagnostic}` 范围内 (不需要 application)，且不必等待 P2/P3 的巨额算力与数据授权即可部分兑现，可作为 P1 阶段一个更快出结果的次要产出，而非替代当前主线。

## Claims Discipline

Claude 与 codex 独立复核后收敛一致 (与 idea 正文 Claims 段落及 JSON 核对无出入)：

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 只有在预注册的、嵌套 storm-system holdout 中，Delta AUPRC 点估计与 CI 同时越过预设实用阈值、校准指标 (Brier/ECE) 不退步，且跨 season、estimator (PCMCI+/LKIF/fixed-edge proxy) 与 raw-capacity 敏感性复现，才能声称该 pressure-resolved multi-lag `(D,J)` 表示在指定数据、hazard、lead time 和模型类下提供了增量样本外预警技能。不得声称"产生了新信息"、"识别了真实因果机制"或"普遍改进了强对流预警"。 |
| NULL | CI 含零仅说明当前设计不可判定；只有功效充足且 CI 完全落入预注册等效区间，才可称增量小于实际意义阈值。 |
| NEGATIVE | 若 CI 上界低于实用阈值且 raw/noncausal 对照具备充分功效，可称该具体 pressure-lag `(D,J)` 实现在明确范围内没有实用增量，甚至可能主动损害技能；不得外推到全部因果表示、估计器或强对流预警任务。当前证据只支持"实现能在设计保证的合成结构上恢复已知边并分尺度，且该 smoke 的冻结融合规则选择了 raw-only、Delta=0"，不支持任何真实大气或预警结论。 |

## Likelihood-Impact Matrix

- Priority: Medium = Likelihood: Low x Impact: High
- Numeric score for ideas.xml: 5
- Rationale:
  - Likelihood: Claude 与 codex 独立评估完全一致，判定 Low，无分歧。理由：8 轮以来因果特征在任何合成设置 (包括本轮新的 4D 版本) 下都从未在预注册规则下击败 raw baseline；要走到 top-venue 级结果，仍需要依次成立：连续真实 4D 数据获取 (受 stop gate 阻塞)、可靠的短窗高维因果估计 (面对 grid-scale PCMCI 已知的样本量/维度诅咒风险)、lag/window null 校准、平流/地形控制、严格 raw-history/operational baseline 对照、公平算力比较，以及最终跨 >=50 个独立 storm systems 的泛化——条件连乘，且新发现的 P2/P3 排序问题意味着即使执行也需要先修正实验设计顺序。Likelihood 维持 Low，与 v1-v7 一致。
  - Impact: Claude 与 codex 独立评估完全一致，判定 High，无分歧。理由：若最乐观情形成立——pressure-resolved hour/day `(D,J)` 表示在跨季节独立 storm systems 上稳定提升严格 held-out 的强对流预警技能，并给出可信的失效边界——会为"因果特征工程能否切实改进天气预警"这条研究线提供一个有价值的正面或负面锚点。但 SHIPS+、ST-CND、SPACY 均已在紧邻域证明"因果驱动的空间/特征方法"这一般路线可行且有人在做，Sha et al./Feldmann et al. 也已证明"HRRR+SPC 强对流 ML"与"AI 模式垂直结构技能评估"本身是活跃、已有真实数字的赛道，故不到 Exceptional。
  - 查表：Low x High = Medium (5)。topic 未声明 `preferred-contribution-types`，hard cap 不适用，numeric score 不截断。Claude 与 codex 在 Likelihood 和 Impact 两个轴上均无分歧，无需额外标注。

## Overall

- Priority: Medium
- Score: 5
- Comments: v8 是这个 idea 的第 8 轮，也是执行纪律最好的一轮——Claude 独立对照原始 JSON/git log 逐条核验，v7 review 指出的三处具体问题 (10/16 计数、PILOT_PLAN_V7 内部矛盾、三项限制补充) 全部确认真实修复，且本轮所有新增数字 (4D smoke 的滞后选择、空间 contrast、held-out AUC) 与原始结果文件零误差一致，是 8 轮里数字保真度最好的一次。v8 对 owner 第二次动机澄清 (真正的 4D pressure x lon x lat x time 结构与多尺度滞后) 的回应也是实质性的工程投入 (4 层气压、双时间尺度独立门控、真实 HRRR schema 校验)，不是术语换皮，但对 owner 实际关心的问题 (因果分析框架能为真实 4D 强对流环境提供什么) 仍交出 0% 真实证据——这一点被诚实标注为 SKIPPED，没有被夸大。Claude 与 codex 独立发现的新问题 (P2/P3 校准顺序颠倒、lag 窗口数不匹配导致的 D/J 跨尺度不可比、尺度标签容易被读成机制归属) 建议在下一版或 proposal 阶段吸收，而不是继续做纯文字级别的 refine——8 轮里因果特征从未在任何预注册规则下击败 raw baseline，这个持续的负面证据本身已经足够充分，继续 idea 级别打磨的边际价值很低，下一步的关键是先补齐 P1.5 校准/可执行性 gate 再决定是否投入昂贵的 P2 确认性实验。Claude 与 codex 在本轮 Likelihood、Impact 两个判断上完全一致，无分歧。

</review>
