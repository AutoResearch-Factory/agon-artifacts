<!-- 书写报告使用中文 -->
---
topic: topics/0710-causal-scs.md
landscape: topics/0710-causal-scs-landscape.md
workspace: workspace/causal-scs-indicator/
---

- One-sentence summary: 检验因果发现方法 (PCMCI+, 后续加 LKIF) 在滑动窗口下的估计不稳定性 (经方法内 null 校准得到离散型 D 与突变型 J 双通道信号) 能否为快变强对流提供预警信息, 并系统刻画该信号相对同复杂度非因果基线与 raw EWS 在何处存在、何处消失, 形成一张因果脆弱性诊断的适用边界图, 而非单一的"能/不能预警"判决。
- Hypothesis: 若估计不稳定性携带独立于常规非平稳性的预事件信息, 它应以 (D) 渐进离散度与 (J) 突变两种形态出现, 且不能被同复杂度、不对第三变量条件化的边际相关基线解释。本轮新增可证伪预测: 若"causal"前缀必要 (条件化暴露额外信息), causal 版本不应系统性弱于 non-causal 版本; 若条件化反而丢弃了边际相关携带的信号, 则应见到不对称: 趋近分岔 (无真实图变化) 的 regime 中 causal 弱于 non-causal, 而真实图变化已发生的 regime 中两者接近, 因为此时条件化不再丢弃任何信息。此预测已在下方 Pilot 中检验。
- Expected outcome: 对每个 (估计器: causal PCMCI+/LKIF, non-causal 边际相关) × (regime: 趋近分岔 fragility testbed, 已知真实图变化正对照) 单元格分别报告存在性门槛 (AUROC≥0.70 且 CI 下界>0.50); 只有通过后才检验相对 raw EWS 是否有增量 (增量降为非必须, 本轮已证实原"联合需比 raw EWS 高 0.05"门槛在 raw EWS 接近完美时不现实)。成功: 至少一个估计器在 fragility regime 通过存在性门槛且跨 regime 稳定; 见 Claims matrix 定义失败/边界。最便宜的下一步证伪是同协议跑 LKIF: 若其 causal 通道在 fragility regime 也落 chance 而 non-causal 能过门槛, 说明这个发现 (条件化丢弃信号) 是跨估计器的一般现象; 若结论相反, 则不对称是 PCMCI+ 特有的, 本身也是边界图上一个有意义的点。
- Contribution type: empirical-finding+diagnostic
- Risk: HIGH
- Estimated effort:
  - Compute: Stage 1 (GPU proxy) 与 Stage 1.5 (真实 PCMCI+ 双 regime, CPU, 约 210 秒) 均已跑完; 剩余 LKIF 复现 CPU 成本预计与 PCMCI+ 同量级 (< 500 CPU-秒); 真实阶段预计 10–30 GPU-hours 做格点特征抽取, 另需 500–1500 CPU-hours 做滑窗因果估计, 与 v2 一致, 仍待 Stage 1.5 全部通过后启动。
  - Data: needs collection; NOAA/SPC 事件标签与三个 HRRR 3-km 字段子集样例已就绪 (与 v2 相同), 完整小时序列下载仍被 stop gate 阻塞。
  - Implementation: LKIF 复现 3–5 天; 若边界图框架成立, 真实数据阶段仍需 4–6 周。
- Novelty quick-check: 核心 niche ("因果发现方法的滑窗估计脆弱性, 经方法内校准为 (D,J) 诊断") 仍未被 CNM (2412.16235)、ST-CND (2606.17553)、NPG (2026)、Causal-Audit (2604.02488) 直接覆盖: 它们把因果强度确定性变化、恢复率或违背风险评分本身当信号, 而非估计过程的时间脆弱性。补全两处此前遗漏引用: Laitinen & Lahti (2205.07576) 用预测性 (非因果) VAR 不确定性做生态 EWS, 本轮 non-causal 边际相关基线的发现某种意义上是其思路在快变 regime 的一次独立复现和边界刻画; Ashwin et al. VAR-eigenvalue EWS (2605.28260) 反推特征值判断分岔类型, 与追踪系数本身的窗口不稳定性仍不同轴。最强撞击兼最有力佐证是 RCV-PCMCI/VarLiNGAM (2410.19412) 与后续 VCDF (2602.21381) 的 Consistency/Variability 统计量, 它与 (D,J) 数学同源, 但用于滤除跨折不一致以求更稳健的图。VCDF 作者本人在结论中承认从未测试过跨折不一致是否反映真实结构变化, 这正是本轮 Pilot 直接检验的问题。Self-compatibility 谱系 (2307.09552/2606.00278/2411.05625) 扰动轴仍是变量而非时间窗口, 撞车风险已排除。
- Strongest objection: 本轮结果可被更悲观地读成: "causal 步骤只是丢弃信号, 整个 fragility 框架不过是更复杂、更差的经典 EWS", non-causal 基线也只是"相关性在分岔前上升"的变种, 并无 raw EWS 之外的独立价值。此外干净模拟下 raw EWS 的优势, 在真实观测数据中未必保持 (arXiv:2505.19034 显示缺失值/异常值系统性削弱 raw EWS 常用统计量), 当前排序完全可能在真实数据上反转, 这说明现在下结论仍为时过早, 而非现有排序有误。
- Why we should do this: Stage 1.5 把"causal 前缀是否必要"这一此前只能定性讨论的问题变成了可判决的数字 (见下方 Pilot Result), 且已用已知图变化正对照 regime 排除"装置本身无效"这一竞争解释。这个判决便宜 (CPU, 数分钟), 用 LKIF 复现一次即可决定该发现是估计器通用还是 PCMCI+-specific, 值得在投入真实数据前做完。
- Pilot:
  - Setup: Stage 1 (冻结, v2): GPU 上 256 对 fast/slow saddle-node 轨迹, 标准化 lag-1 VAR proxy。Stage 1.5 (本轮新增): CPU 上真实 tigramite PCMCI+ (ParCorr, tau_max=1) 替换 VAR proxy, 同时跑同复杂度非因果基线 (滞后 1 期边际 Pearson 相关, 不条件化); 两个 regime: (a) 冻结的 saddle-node fragility testbed (127/128 对, 1 例事件轨迹运行时未穿越 fold 被过滤; control 轨迹新增运行时"未穿越"断言, 0/128 触发), pre-event 窗口; (b) 新增线性 VAR(1) 已知真实图变化正对照 (128 对, x→y 边已知时刻真实出现 vs 从不出现), post-switch 窗口, 验证装置本身能否探测真实结构变化。
  - Metric: 六项预注册门槛不变 (D/J 各自 AUROC≥0.70 且 CI 下界>0.50, 联合 AUROC≥0.75, 相对 raw EWS 增量≥0.05); 本轮新增将"存在性"与"相对 raw EWS 增量"拆开报告, 因为 v2 review 已指出在高 raw-EWS regime 中原联合门槛不现实。
  - Result: Stage 1 (v2, 冻结): (D=0.392, J=0.517, 联合=0.428) vs raw EWS=0.919, 六项门槛全部失败。Stage 1.5 (本轮): fragility regime 中 causal 联合=0.482 [0.426,0.533] (chance), non-causal 联合=0.705 [0.638,0.778] (通过存在性门槛), raw EWS=0.890 [0.852,0.924]；正对照 regime 中 causal 联合=0.786 [0.713,0.850], non-causal 联合=0.849 [0.789,0.902], raw EWS=1.000 [1.0,1.0] (两个方法均通过存在性门槛)。
  - Signal: NEGATIVE (对 causal/PCMCI+ 在 fragility regime 的存在性门槛) + WEAK-POSITIVE (对 non-causal 基线在同一 regime) + POSITIVE (对测量装置在正对照 regime 的有效性)。三者合起来是一个 FAMILY-SPECIFIC 且估计器相关的负面结果, 不外推到 LKIF 或真实强对流, 但已排除"装置无效"这一竞争解释, 并把"causal 前缀是否必要"从开放问题变成了本 regime 内的具体否定答案。

<added-on-refine>
- Claims and Claims matrix:
  - C1a（非因果存在性）: 同复杂度非因果边际相关的 (D,J), 经方法内校准, 在 fragility regime 中可与匹配非事件区分, 但显著弱于 raw EWS。
  - C1b（因果必要性, 本轮新证据）: 同一 regime、同一窗口协议下, causal (PCMCI+) 条件化后的 (D,J) 不比 chance 更好; 条件化丢弃了 C1a 的信息, 而非暴露额外信息。
  - C2（联合互补性, 降级为 stretch claim）: 仅当 C1a、C1b 各自独立通过存在性门槛才检验联合增益; 不再要求联合必须超过 raw EWS。
  - C3（边界, 不变）: profile 衡量估计过程的时间敏感性, 不等于真实因果结构变化、隐藏过程激活或物理 tipping。
  - C4（测量有效性, 本轮新增）: 已知真实图变化正对照 regime 中, causal 与 non-causal (D,J) 均应清楚通过存在性门槛, 排除"装置本身无法探测任何结构变化"这一竞争解释。

  | Outcome | 允许的 claim |
  |---|---|
  | POSITIVE | 仅当某估计器通过存在性门槛且 C4 同时成立, 才可声称该估计器的 (D,J) 提供预警诊断信息; 不得声称识别真实因果结构或物理分岔。 |
  | ESTIMATOR-SPECIFIC | causal 与 non-causal 分化时 (如本轮), 只报告分化及其边界条件, 不得把一个估计器的失败归咎于框架整体, 也不得把另一个的通过泛化为"因果诊断有效"。 |
  | NULL | 两估计器均未检测到增量信息, 且 C4 成立。 |
  | NEGATIVE / ARTIFACT | 信号只在事件后出现, 或 C4 不成立 (装置对已知真实变化也无反应), 须先修复装置, 不得解释 fragility regime 结果。 |

- Narrative: 核心问题收紧为"因果条件化 (vs. 同复杂度非因果相关) 是否改变预事件脆弱性信号的可探测性, 且这一答案在'趋近分岔'与'真实图变化已发生'两类 regime 间是否一致"。Stage 1.5 的反直觉答案 (条件化在 fragility regime 丢信号, 在正对照 regime 不丢) 是当前最强证据; 论文以跨 regime、跨估计器的适用边界图为主要贡献, 保留全部已诚实报告的负结果。
- Experiments:
  1. LKIF 复现 (同协议、同两 regime), 判定 Stage 1.5 不对称是否跨估计器一般。
  2. 新增混淆源: 窗口长度/采样率错配 (2602.19903)、超参数漂移固定为控制变量 (2310.18212); TCD-Arena (2605.03045) 供扩展第三/四 regime。
  3. Real-event test (待 Stage 1.5 完成): 沿用 v2 协议 (匹配、严格 lead-time、event-level nested blocked CV、cluster bootstrap), 加跨 lead-time BH-FDR 校正、事件数/功效估算、storm-system 去重。
  4. Baselines: 保留 v2 全部 (CAPE/shear、raw EWS、RCDyM/TIPMOC、CNM/ST-CND、Causal-Audit/PLaCy、稳健 change detector), 加本轮非因果基线与正对照 regime 为标准配置。
- Assets status: Stage 1/1.5 均已跑通并提交; LKIF 与真实小时序列下载仍在 stop gate 之后, 详见 `workspace/causal-scs-indicator/data/MANIFEST.md`。
</added-on-refine>

<review date="2026-07-11">

## Novelty

- Score: 4/10 (Claude 与 codex 独立评分一致, 均为 4/10)
- Closest prior work:
  1. Yu, Guo, Luk, RCV-PCMCI/VarLiNGAM (arXiv:2410.19412) 与 VCDF (arXiv:2602.21381) — Consistency/Variability 跨折统计量与 (D,J) 数学同源, 用途相反 (过滤噪声 vs 提取信号); VCDF 结论明确承认"真实结构变化时跨折不一致可能反映真实变化"但从未验证, 已通过全文 wiki 精读核实.
  2. Adedayo, conditioning-depth graph instability (arXiv:2606.01214, MEDIUM 撞车, 已全文核验) — 同一问题空间 (潜在混淆/隐藏记忆诊断, 不恢复潜在图), 不同扰动轴 (conditioning depth 而非时间窗口).
  3. Faltenbacher/Wahl/Herman/Runge, PC 系方法 coherency score (arXiv:2502.14719) — PCMCI+ 团队本人提出的零额外成本自洽诊断, 复用 CI 检验日志.
- Key differentiator: 方法本身 (跨扰动轴的图/效应不稳定性度量) 不新, RCV/VCDF/2606.01214/2502.14719 均已各自覆盖过"用不稳定性做诊断"这一元技巧的一部分; v3 剩余的窄差异是把扰动轴换成滑动时间窗口, 并将其与严格预事件判别及 causal-vs-marginal conditioning penalty 联系起来。若"条件化在预事件阶段系统性损失信号、但在事后已知结构变化阶段不损失"这一发现能在受控 factorial testbed、多个估计器和真实事件上复现, finding 本身可能新颖; 但目前的两-regime pilot (见 Quality/Logical gaps) 尚不能建立该发现, 只能算作一个有潜力但未坐实的假设。
- **过程诚信问题 (Claude 与 codex 独立核验一致)**: `REFINEMENT_LOG.md` 声称 v2 review 中"tvVAR-EWS (2205.07576) and coherency score (2502.14719) not yet in the novelty narrative"这条 concern 已被"Both now cited explicitly in v3's novelty quick-check"解决, 但 grep 核验显示 v3.md 全文既不包含 "2502.14719" 也不包含 "2606.01214" (v2 review 原文实际点名的是 2606.01214, 而非 2502.14719 — REFINEMENT_LOG 转述阶段已经张冠李戴)。v3 novelty quick-check 确实新增了 2205.07576 与 2605.28260 两处引用, 但 2606.01214 (MEDIUM 撞车) 与 2502.14719 (与 D/J 设计直接相关) 均未进入 v3 任何位置的叙事。这是一个被 refiner 自己的 changelog 误报为"已解决"的真实遗漏, 建议下一版必须实际补上而非仅在 changelog 里打勾。

## Quality

评估视角: 沿用 v1/v2 review 的推断 — topic (`topics/0710-causal-scs.md`) 未声明 `target-venue` 或 `preferred-contribution-types`, 也无 `## Target venues` / `## Review standards` 小节, 按顶级大气科学 / Earth-system methods / AI4Science 视角评估 (Claude 与 codex 一致)。

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 4/10 (Claude 初评 6, codex 3, 采纳 codex 更具体的核验后合并偏严) | codex 的核心指控成立且经 Claude 独立复核确认: 两个 regime 同时改变了非线性形式、隐藏驱动、第三变量作用、效应大小、窗口/lead/lookback 参数和 pre/post 方向, 因此 −0.222 (Regime 1) 与 −0.064 (Regime 2) 的差异**不能**被干净地归因于"预事件 forecasting vs 事后 detection"或"条件化本身丢弃信息"——没有固定边 (fixed-edge) 的 partial-correlation 消融来单独隔离"条件化"这一个变量, PCMCI+ 与 marginal Pearson 还同时在父集搜索、显著性筛选和统计复杂度上不同, "同复杂度基线"这一说法只在变量/窗口/校准协议层面成立, 不在检验统计量复杂度层面成立。另一处独立核验确认的问题: v3 Pilot 的 Metric 段写"六项预注册门槛不变", 但同一段紧接着把联合门槛的强制性 (原 AUROC≥0.75 且须比 raw EWS 高 0.05) 拆解为非强制的"存在性 vs 增量"两级判定——这是对"冻结门槛"表述的自我矛盾, 使得 Regime 1 non-causal "联合=0.705 通过存在性门槛"这一表述在原六项标准下其实是不过关的 (J=0.631<0.70, 联合=0.705<0.75)。Claude 独立复核 `pilot_pcmci_gate.py` 的 Regime 2 窗口选取逻辑后确认 codex 指出的第三点: `direction="post"` 时 mask 允许 `ends∈[1110,1900]`, 对应 `starts∈[960,1750]`(window=150), 而 switch=1100, 故 starts∈{960,1000,1040,1080} 对应的窗口会跨越 switch 时刻本身 (窗口同时含 switch 前后数据), 并非严格"事后"窗口, 弱化了 Regime 2 作为"装置有效性"正对照的说服力。 |
| Missing evidence signals | 4/10 (Claude 初评 5, codex 3, 合并偏严) | codex 补充的具体缺口清单成立: 多 seed 复现 (当前 Regime 1/2 均为单 seed 点估计, 无法区分"这个差异"与"这个 seed 的抽样噪声")、两阶段 (calibration+evaluation) bootstrap、causal-minus-noncausal 差值本身及跨 regime interaction 的正式置信区间/假设检验、fixed-edge marginal-vs-partial 消融、效应量 (0→0.6 系数跳变大小) 扫描、更不易分的 graph-change 正对照、LKIF、真实阶段事件数/功效/storm-system 去重均未完成。此外 codex 指出一个此前两轮 review 都未注意到的细节: Stage-1.5 的 "raw EWS" 实际只是滚动方差均值 (`raw_ews_profile` 只用 `.var(axis=0).mean()`), 并非 v1/v2 承诺的完整 variance+autocorrelation 复合基线, 这意味着 0.890/1.000 这两个 raw-EWS 数字本身可能低估或高估了完整 baseline 的真实强度, 需要在下一版明确标注这一简化。 |
| Narrative | 6/10 (Claude 7, codex 6, 合并偏 codex) | "适用边界图"确实比原先单一预警成败叙事更锋利, 也诚实保留了 causal 负结果, 是本轮最大的叙事进步; 但如 codex 所指, "条件化丢弃信息""正对照排除装置无效"和"正对照中不再丢信息"三处表述均超出当前两-regime、单 seed、多因子混杂设计实际能支撑的强度, 属于叙事领先于证据。此外见 Novelty 段: 2606.01214 (MEDIUM 撞车) 仍未被纳入差异化论证, 是本应但尚未完成的叙事补丁。 |
| Venue contribution | 4/10 (Claude 与 codex 一致) | 目前仍是标准 PCMCI+、Pearson 相关、滚动 SD/极差与百分位校准 AUC 的探索性组合, 应用于两个纯合成 testbed。作为 pilot 阶段的探索性证据有价值, 但顶级 Earth-system/AI4Science 稿件至少需要受控的 (仅条件化一个变量的) factorial 边界图、跨估计器 (含 LKIF) 复现与真实强对流外部验证, 目前均未达到。 |
| Testability | 6/10 (Claude 初评 8, codex 6, 采纳 codex 下修) | codex 指出一个 Claude 初评遗漏的关键点: 声称"最便宜的下一步证伪是同协议跑 LKIF (3-5 天)"并非真正最便宜的选项——用现有约 210 秒的协议做多 seed 重跑并传播 calibration 抽样不确定性, 比实现一个新估计器 (LKIF) 更便宜, 也更直接地回答"当前 0.705/0.482 这两个点估计本身有多稳"这一先决问题, 应作为比 LKIF 更优先的下一步。此外"六项预注册门槛不变"与实际采用的存在性/增量拆分判定之间的表述冲突 (见 Logical gaps) 也削弱了"预注册可信度"这一 testability 子维度。可取之处 (Claude 与 codex 一致): 代码、JSON、日志三者互相核验一致, 数字无虚报, 计算成本确实低廉。 |
| Outcome realism | 4/10 (Claude 5, codex 4, 合并) | 存在性/增量拆分门槛的方向是现实的合理改进, 但按 v2 冻结的原六项标准, 本轮实际没有任何 (估计器×regime) 单元格完整通过 (non-causal Regime 1 的 J 与联合均不过原阈值), 把 0.705 直接称为"通过存在性门槛"是在门槛本身刚被放宽之后才成立的, 应标注为 exploratory/result-informed 而非预注册阳性。真实数据阶段还需要跨区域验证、事件数/功效达标、多重比较校正, 累积起来仍是一个依赖多个乐观条件同时成立的目标。 |
| Contribution type compliance | n.a. | idea types = {empirical-finding, diagnostic}; topic 未声明 `preferred-contribution-types`, 跳过, 不触发 hard cap (Claude 与 codex 一致)。 |
| Overall Quality | 4/10 (Claude 初评 6, codex 4, 采纳 codex 后下修) | v3 的确做了真实且诚实的额外实验 (真实 tigramite PCMCI+ 而非 v2 的 VAR proxy, 数字均核实无虚报), 填上了两个 v2 明确要求的缺口 (非因果基线、正对照 regime), 这些执行层面的进步是真实的。但经 codex 更细致的核验并被 Claude 独立复核确认后, 当前"因果条件化系统性丢弃预测信号但不丢弃事后信号"这一核心新结论建立在一个多因子混杂、单 seed、门槛表述自相矛盾、正对照部分泄漏 (窗口跨越 switch 点) 的实验设计之上, 尚不足以支撑其被当作稳固发现来叙述, 只能视为一个值得用更受控设计验证的假设。 |

## Contribution Drift (n >= 2 only; n=1 写 N/A)

- v_{n-1} (v2) contribution types: {empirical-finding, diagnostic}
- v_n (v3) contribution types: {empirical-finding, diagnostic}
- Status: unchanged
- Hard cap triggered: no (topic 未声明 `preferred-contribution-types`, 且不存在 method/theory 的无理由删除; Claude 与 codex 一致)

v2 review concern-by-concern 复核 (Claude 与 codex 独立复核后合并, 采纳 codex 更严格的判定):

| v2 concern | Status | Assessment |
|---|---|---|
| (D,J) 联合非逻辑必然 | partially resolved | C2 已降为 stretch claim, 但 C1a 仍把结果笼统写成"(D,J) 存在性", 而实际只有 non-causal 的 D (0.735) 过 0.70, J (0.631) 未过, 更准确的表述应是 D-family-specific, 而非笼统的"通过存在性门槛"。 |
| 跨方法 percentile 可比性未证明 | partially resolved | PCMCI+ 已实际运行, 但 LKIF 仍未运行; 校准百分位的跨方法语义等价性、以及 calibration 抽样误差, 仍未验证。 |
| 缺同复杂度非因果 baseline | partially resolved (较 v3 body 自述的"resolved"更保守) | baseline 已用真实代码跑出结果, 但 Pearson 与 PCMCI+ 只在变量/窗口/校准协议层面"同复杂度", 并未隔离"条件化"这一单一变量 (二者还在父集搜索、显著性筛选、检验统计量复杂度上不同), 尚不足以支撑"条件化本身丢弃了信息"这一因果归因。 |
| control 未做运行时未穿越断言 | resolved | 已读代码确认 `assert not not_crossed.any(axis=1).any()` 存在且逻辑正确, 0/128 control 穿越, Claude 与 codex 独立核验一致。 |
| bootstrap 未传播 calibration-null 抽样不确定性 | ignored | REFINEMENT_LOG 如实标注为 deferred, 但当前 0.705 这一边缘性数字尤其需要此修正, 本轮继续强调。 |
| 缺已知真实图变化正对照 regime | partially resolved | Regime 2 已实现并跑出数据, 但硬系数跳变 (0→0.6)、raw EWS=1.000、且约 4/20 个入选窗口跨越 switch 时刻本身 (经 Claude 独立复核代码确认), 使其作为"装置有效性"正对照的验证力有限, 只能弱支持"装置能响应非常明显的二阶分布变化", 不能充分支持"装置对真实结构变化普遍敏感"。 |
| 尚无官方 PCMCI+/LKIF gate 复现 | partially resolved | 真实 PCMCI+ 已运行, 但 LKIF 未运行; 脚本 docstring 明确 Stage-1.5 只是单标量摘要, 非完整冻结 v2 多统计量 profile。 |
| 真实阶段事件数/功效/storm dedup | ignored | 仍无数字或执行结果, REFINEMENT_LOG 如实标注为 deferred。 |
| 联合需超 raw EWS 0.05 在高 raw-EWS regime 不现实 (codex v2 算术) | partially resolved | 存在性/增量拆分的方向合理, 但不能追溯性地把本轮 0.705 改判为"冻结门槛"下的阳性 (见 Logical gaps), 该拆分应作为下一版预注册的新规则, 而非本轮回溯性重新解释。 |
| Alternative Framing: 适用边界图 | resolved | v3 body 已明确采用, 但当前两个高度混杂的 regime 尚不足以构成真正的"图" (只是两个点)。 |
| tvVAR-EWS (2205.07576) 未纳入 novelty narrative | resolved | v3 novelty quick-check 确已显式引用并区分该工作。 |
| graph-instability diagnostic (2606.01214, v2 原文点名) 未纳入 novelty narrative | **ignored (REFINEMENT_LOG 误报为已解决)** | v3.md 全文不含该 ID, MEDIUM 撞车仍待正面区分, Claude 与 codex 独立 grep 核验一致。 |
| coherency score (2502.14719) 应纳入叙事 | **ignored (REFINEMENT_LOG 误报为已解决)** | v3.md 全文不含该 ID, Claude 与 codex 独立 grep 核验一致。 |

未采纳建议的 pushback 复核: "暂缓 LKIF 以避免仓促实现"作为工程排期决定本身有据, 但不能反过来支持"当前结论已可跨估计器泛化"这一言外之意 (v3 body 已注意克制, 未过度声称, 可接受); "Regime 2 仅作为 sanity check 而非严格正对照"这一目的定位合理, 但如上文所述, 其证据强度不足以支撑 C4 (测量有效性) 目前被赋予的分量, 本轮予以重新强调。two-stage bootstrap 与真实阶段功效分析的延期均无实质论据支撑其暂缓的必要性 (相比 LKIF 需要新实现, 这两项只需修改现有代码), 本轮继续强调应优先于 LKIF 完成。

## Alternative Framing

Claude 与 codex 的建议方向一致收敛为同一个更锐利的框架, 采用 codex 的具体表述作为主建议: 把主问题进一步收紧为 **"causal-conditioning penalty map"**——预注册核心估计量 Δ = AUC_conditional − AUC_marginal, 在一个仅改变 conditioning set (是否条件化)、phase (预事件 vs 事后)、graph-change 强度和 confounding 强度这四个因子、其余设计 (窗口、检验统计量复杂度、DGP 骨架) 全部固定的 factorial DGP 上, 正式检验 Δ 本身及其与 phase/confounding 强度的交互显著性; 强对流仅作为外部验证场景, 而非核心判决数据。这样得到的诊断仍属 empirical-finding+diagnostic, 不引入新贡献类型, 但比当前"两个高度混杂的 regime 直接对比"更能把观测到的 −0.22/−0.06 差异真正归因于"条件化"本身, 而不是归因于随 regime 一起变化的其余五六个因子。

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 仅当完成多 seed 复现、两阶段 (calibration+evaluation) bootstrap、Δ (causal−noncausal) 及其与 phase 交互的正式假设检验, 且用 fixed-edge/factorial 消融隔离出"条件化"这一单一变量之后, 才可声称特定估计器在特定 DGP 下存在可复现的 conditioning penalty 或预事件脆弱性信号; 不得声称识别真实因果机制, 也不得在未超过完整 (variance+autocorrelation) raw EWS baseline 时声称独立预警价值。 |
| NULL | 在预注册的估计器、窗口、DGP 和事件范围内, 未检测到稳定的 conditioning penalty 或增量脆弱性信息; 不能外推为所有因果发现方法或所有强对流情形均无信号。 |
| NEGATIVE | 若 conditional 方法 (PCMCI+) 稳定弱于 marginal (Pearson) 方法, 可如实报告"该 PCMCI+ summary 统计量在所测两个 regime 中判别力弱于同窗口/同校准协议的边际相关基线"; 在完成 fixed-edge、同检验统计量复杂度的消融以真正隔离"条件化"这一变量之前, 不能写成"因果条件化本身丢弃了预测性信息"这一更强的因果归因表述——这是当前 v3 文本最容易被 hostile reviewer 抓住并要求降级的一句话。 |

## Likelihood-Impact Matrix

- Priority: Medium = Likelihood: Low x Impact: High
- Numeric score for ideas.xml: 5
- Rationale:
  - Likelihood: Claude 与 codex 一致判定 Low, 无分歧。理由: 当前"conditioning penalty"核心对比只有单 seed 点估计, 无差值/交互的正式推断; 两个 regime 因多因子混杂而不可直接归因比较; 正对照 (Regime 2) 因硬跳变+部分窗口跨越 switch 点而接近平凡可分, 验证力有限; 按 v2 原冻结六项门槛衡量, 本轮实际没有任何 (估计器×regime) 单元格完整通过; LKIF、完整多统计量 profile、功效分析和真实事件验证均未完成。要达到 top-venue 级结果仍需要实质性的实验重新设计 (factorial 隔离设计), 而非常规工程补强, 这是 Low 而非 Medium 的关键理由。
  - Impact: Claude 与 codex 一致判定 High, 无分歧。理由: 若最乐观情形成立——"conditioning penalty" 在严格受控的多 regime、多估计器 (含 LKIF)、真实强对流数据上都能稳定复现——它会直接回应 VCDF 作者自己承认从未验证的问题, 并给出"因果诊断目标 vs 预测/forecasting 目标"何时不一致这一清晰边界, 对"应该消除还是保留/利用因果估计不稳定性"这条活跃研究线有明确的叙事影响力; 但这仍是对一个已存在、他人已部分涉足的 niche 做精确定位, 而非颠覆性地开辟新方向或推翻强共识, 故不到 Exceptional。
  - 两轴 Claude 与 codex 完全一致 (Low, High), 无 disagreement 需要标注; 与 v2 review 的 Low×High=5 数值和推导路径均一致 (v2 是基于"causal 未证明必要性"的假设性推断, v3 是基于"causal 在两轮独立测试——proxy VAR 与真实 PCMCI+——中均在 fragility regime 落入 chance"的实测结果, 结论方向一致但证据基础已从假设性变为部分实证性, 只是新实证本身因设计混杂而不够干净)。
  - 查表: Low x High = Medium (5)。topic 无 `preferred-contribution-types` 声明, hard cap 不适用, numeric score 不截断。

## Overall

- Priority: Medium
- Score: 5
- Comments: v3 是一次执行诚实、数字可核验 (代码/JSON/日志三方一致, 无虚报) 的 refine, 真正跑通了 v2 明确要求的两项关键实验 (同复杂度非因果基线、已知真实图变化正对照), 并如实报告了对原始"因果"框架更不利的结果。但经 codex second opinion 更细致的代码级核验 (Claude 独立复核确认成立) 后发现, 当前最具吸引力的新结论 ("因果条件化丢失预测信号但不丢失事后信号") 建立在一个多因子混杂 (regime 之间同时变了六七个变量, 不只是"条件化")、单 seed、门槛表述前后不一致 ("六项预注册门槛不变" vs 实际采用的存在性/增量拆分)、正对照部分窗口泄漏跨越 switch 点的实验设计之上, 因此 Overall Quality 由 Claude 初评的 6/10 下修为合并后的 4/10。这不改变 Likelihood-Impact 判断——Low×High=Medium(5) 与 v2 完全一致——因为该判断本就建立在"多个高风险条件需要同时成立"这一前提上, 本轮新发现的设计缺陷进一步印证而非削弱这一前提。此外, REFINEMENT_LOG 存在一处被 Claude 与 codex 独立核验共同确认的误报 (声称已将 2606.01214/2502.14719 纳入 novelty 叙事, 实则 v3.md 中均不存在), 建议下一版 refine 时对 changelog 自述的核实要更严格, 并优先完成比 LKIF 更便宜的多 seed/两阶段 bootstrap 重跑, 再决定是否投入 LKIF 实现。

</review>

<deep-lit-integration date="2026-07-11" scope="topic" iteration="3">

<!-- 本次为 topic-scope (非 idea-scope) deep-lit-tick 第3次运行, 按协议只归档"与本 idea 直接相关 (撞车/baseline/数据集/可得性)"的发现; 完整36篇归档见 topics/0710-causal-scs-landscape.md 分组 W-AB。本轮特别聚焦用户指定的 v3 新角度: 真实 PCMCI+ pilot vs non-causal 边际相关基线。 -->

- **Novelty quick-check: 无需变更。** 本轮36篇 (5轮搜索, B7反向扩展饱和) 均与 v3 核心 claim 不构成撞车, 现有最强撞车对比对象 (RCV-PCMCI/VarLiNGAM 2410.19412、VCDF 2602.21381、graph-instability 2606.01214、coherency score 2502.14719) 排名不变。

- **为 Pilot 反直觉负面结果新增的理论解释候选 (未改变实验结论, 但提供了此前 review 未考虑的机制假说, 建议下一版在 Narrative/Strongest objection 段讨论)**:
  - Yuan & Lozano-Durán (arXiv:2401.16512): 混沌系统极端事件预测的信息论上下界证明, 任何原始状态的衍生统计量信息量不可能超过原始状态本身——为 causal channel (0.39-0.52) 远逊于 raw-state EWS (0.919) 提供数据处理不等式式的原则性解释, 而非归咎于实现缺陷。
  - Simoes, Dastani, van Ommen (arXiv:2402.01341): 纯理论证明因果熵/因果信息增益系统性劣于/异于经典信息论对应量 (因果版数据处理不等式不成立) 是已知的可数学刻画现象——为"causal 条件化系统性丢失判别信息"提供跨技术路线的合理性佐证。
  - Wahl, Ninad, Runge (arXiv:2306.07047) Section 9: 变量聚合(粗化)可从根本上破坏条件独立结构的显式SCM反例——若真实数据阶段所用变量(如HRRR格点场提取指标)本质是更精细大气过程的粗化聚合, "条件化丢信号"可能部分源于组级变量违背因果忠实性这一结构性伪影, 而非纯粹实验设计混杂, 是此前 review 未考虑的候选解释, 建议下一版加入讨论。
  - Martínez-Sánchez & Lozano-Durán (arXiv:2505.10878): 复现5种时间/状态可分辨因果方法(含TvLK, LKIF近亲)均无法在硬阈值突变toy案例上给出清晰证据。**方法论警示**: 未来若复现 windowed LKIF 在 fragility regime 也落 chance, 须先做已知突变正对照, 排除"Kalman平滑类方法对突变本身不敏感"这一竞争解释, 而非直接把该结果解释为"条件化丢信号是跨估计器一般现象"——这是比原计划的"同协议跑LKIF"更精细的一步, 建议在 Experiments 列表的 LKIF 复现项前插入此 sanity check。

- **与 v2/v3 review 已识别缺口直接相关的候选工具**:
  - Murphy & Benavoli (arXiv:2601.09579) 的 Bayesian Wilcoxon 符号秩检验, 可直接填补 v3 review Logical gaps/Testability 两处反复指出的"缺 Δ (causal−noncausal) 显著性检验"缺口, 优先级应高于新增估计器。
  - Chen, Shi, Yue (arXiv:2605.01669, PRCD-MAP) 的 Weak-Data Sweep 发现 T 接近 d 时学习信号退化为噪声, 与本 idea"区分事件特异性不稳定与估计器噪声"这一核心未解瓶颈高度共鸣, 可作为该瓶颈在另一场景下独立复现的背景证据引用。
  - Chen & Wu (arXiv:2604.10371, SGED-TCD) 把跨扰动视图的结构不一致当作训练时要用 loss 主动压制的缺陷——是"因果估计对扰动敏感"在2026年同域SOTA方法中仍需专门处理的独立反向佐证, 可用于 Strongest objection 段加固"这不是本课题臆造的问题"这一论点。

- **候选第三估计器 (若未来扩展 (D,J) 协议超出 PCMCI+/LKIF 双通道)**: Faruque et al. TS-CausalNN (arXiv:2404.01466, CNN score-based, 已在真实非平稳大气数据 (北极海冰) 上外部校准) 与 Murphy & Benavoli 的 GP_SIC (arXiv:2601.09579, 每窗口一次GP拟合出图, 比约束型方法更适合套入滑窗协议) 均为低成本候选, 优先级低于 LKIF 复现。

- **未来空间维度扩展的背景参考 (当前 Stage 1/1.5 不适用)**: Ninad, Wahl, Gerhardus, Runge (arXiv:2505.10476, PCMCI+/J-PCMCI+ 核心作者团队) 的向量值变量聚合一致性分数 (c_ind/c_dep/AC) 与自适应包装器 Adag, 框架层面与本课题"跨窗口一致性"目标同构 (聚合轴是空间而非时序), 若未来真实数据阶段需要处理 HRRR 格点场的空间聚合, 建议引用其分数体系作对比基线。

</deep-lit-integration>
