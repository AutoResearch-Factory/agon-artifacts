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
