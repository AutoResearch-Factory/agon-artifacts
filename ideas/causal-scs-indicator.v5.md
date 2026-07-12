<!-- 书写报告使用中文 -->
---
topic: topics/0710-causal-scs.md
landscape: topics/0710-causal-scs-landscape.md
workspace: workspace/causal-scs-indicator/
---

- One-sentence summary: 在目标边、条件集维度和统计流程都相同的条件下, 审计滑窗因果估计的不稳定性: 区分真实边变化与混杂漂移, 判断条件化何时提高或损害 D/J 的可探测性, 这一模式在部分可观测混杂、非线性机制下能否稳健, 以及在真实强对流环境场中可能揭示的领域新现象。
- Hypothesis: 条件化指在检验目标边时控制第三变量。固定目标边、条件集维度、窗口和推断流程后, 其作用应主要取决于扰动来自真实边还是混杂机制。v4 的混杂 `z` 完全观测且线性, pre/post 只是同一曲线的时间平移, 两处都可能人为保证了结论方向。v5 同时加固三处: 条件化只用含噪代理 `w`、边效应改为饱和非线性 `tanh`、pre/post 改为两套独立基线动力学。若模式在三处加固下同时存活, 说明它不是构造的人为产物; 但方向已被 anchor regression/有限样本 minimax 决策论边界预测 (见 Novelty quick-check), 更像迁移检验而非全新发现。
- Expected outcome: v5 GPU 加固实验 (36886062) 显示: graph-only-max 的条件化收益在部分可观测 (`corr(w,z)` 实测 0.70-0.78, 非标称值)、非线性边、独立 pre/post 动力学下依然存活, 两个 phase 的 Delta CI 均排除 0 (pre 0.029[0.004,0.047]; post 0.035[0.014,0.056]); confound-only-max 惩罚反而更不稳定, full observability 下两个 phase 均不显著。官方 `LK_Info_Flow` 在同一 v4 DGP 上的 smoke-scale 复现 (32 pairs/2 seeds) 方向相反: conditioned_true 未超过 sham/marginal, 与 v3 真实 PCMCI+ 一致，说明"条件化对真实边变化有益"可能是 fixed-edge partial correlation 的特有产物, 需同规模 PCMCI+/LKIF 复现才能下结论。raw EWS 仍全面占优。
- Contribution type: empirical-finding+diagnostic
- Contribution drift note: v4 是 `empirical-finding+diagnostic`; v5 完全保留, 不新增 method/benchmark/application/theory/dataset, 也未删除已有类型。real-data 阶段新增的 storm-mode/空间梯度探索分析臂仍在此框架内 (更多分层证据, 非新 contribution type)。
- Risk: HIGH
- Estimated effort:
  - Compute: v5 加固网关实测 2.48s L4 GPU + 12.5s CPU (LKIF smoke); 全规模 PCMCI+/LKIF 复现按 smoke 计时外推 <1 GPU-hour+<2 CPU-hours。真实阶段 (若通过) 沿用 v4 估计: 10-30 GPU-hours+500-1500 CPU-hours。
  - Data: needs collection；SPC 样例、HRRR smoke subsets 已就绪, 完整序列仍被停止门阻塞; 按 storm mode 分层还需 SPC/NWS 事后分类标注 (待补充)。
  - Implementation: 加固 DGP+LKIF smoke 已完成 (<1 周); 全规模双估计器复现预计 3-5 天; 若通过, 真实数据双臂另需 4-6 周。
- Novelty quick-check: RCV-PCMCI/VCDF (2410.19412/2602.21381) 过滤不稳定边但不检验扰动来源; graph-instability (2606.01214) 变条件集深度但不比较扰动来源; coherency score (2502.14719) 复用 CI 日志不做事件判别。Siddique et al. (2026, submitted ESD, Conditioners-LKIF) 对 LKIF 做混杂强度扫描, 是最近的方法论近邻, 但用固定系数静态 VAR, 无滑窗/转折点框架。核心模式与 anchor regression (1801.06229) + 有限样本 minimax (2606.12680) 的决策论边界同构: 扰动作用于协变量本身 (对应"真实边变化") vs 作用于混杂/隐变量 (对应"混杂漂移") 决定谁被支配。v5 是把这条 population-level/i.i.d. 边界迁移到滑窗时序、部分可观测混杂场景下检验其是否依然成立, 而非提出新边界。
- Strongest objection: 三点。(1) 模式即便在加固 DGP 下存活, 方向已被 anchor regression/minimax 理论预测, "意外性"很低; 真正新颖处只在于检验该边界是否迁移到一个从未被检验过的领域 (滑窗、临近分岔、部分可观测混杂), 而 v5 尚未找到与该预测背离的场景。(2) 官方 LKIF 在同一 DGP 上的 smoke-scale 复现方向相反, 与 v3 真实 PCMCI+ 一致, 说明该模式可能是 fixed-edge partial correlation 的特有产物而非跨估计器现象，比 v4 被指出的问题更严重， 全规模双估计器复现是不可再推迟的下一步。(3) raw EWS 在四轮迭代中无一例外全面占优, 尚无证据支持真实数据上的排序反转。
- Why we should do this: v4 排除了"条件化系统性丢失预事件信号"这一强叙事, 但被指出构造可能保证了结论方向、pre/post 缺乏独立物理意义, 且未回应"跑在真实大气数据上能发现什么"。v5 用最小加固压力测试前两点, 并把真实数据阶段扩展为验证+探索, 给出两个可证伪的领域假设。
- Pilot:
  - Setup: `factorial_helpers_v5.py` 在 v4 冻结骨架上加固三处 (含噪代理混杂、tanh 非线性边、pre/post 独立基线动力学), 在决策性角点 (null/graph-only-max/confound-only-max) x phase x 混杂可观测度 `{1.0,0.5}` 上重跑同规模 (96 pairs/8 seeds/400 次两阶段+分层 bootstrap)。另用官方 `LK_Info_Flow` 包 (github.com/YinengRong/LKIF, 需 numpy<2.0) 在冻结 v4 DGP 三个决策 cell 上做 smoke-scale (32 pairs/2 seeds) 复现。
  - Metric: 同 v4 门槛 (AUC>=0.65 且 CI 下界>0.50), Delta 收益/惩罚要求 CI 完全大于/小于 0; 新增 oracle-proxy gap 与跨估计器方向一致性检验。
  - Result: graph-only-max 收益两个 phase、两档可观测度下 CI 均排除 0 (pre 0.037[0.008,0.069]→obs0.5 时 0.029[0.004,0.047]; post 0.054[0.031,0.082]→0.035[0.014,0.056])。confound-only-max 惩罚仅在 post 且 obs=0.5 时显著 (-0.030[-0.053,-0.007]), 其余三格含 0, 比 v4 更不稳定。oracle-proxy gap 对边检测不显著, 对混杂漂移检测在低可观测度下显著 (0.028[0.008,0.050])。LKIF smoke: 4 个非空 cell 的 conditioned_true 点估计全部 <= sham/marginal, 但 32 pairs 下 CI 大幅重叠, 非决定性。raw EWS 仍全面占优 (0.88-0.99)。
  - Signal: NEGATIVE (预警增量、跨估计器普适性仍不成立) + 局部正诊断信号在加固 DGP 下更稳健存活, 跨估计器一致性从假设变为待全规模复现裁决的开放问题。

<!-- 注意: 不要将 <added-on-refine> 这个 xml tag 写到报告中, 它只是模板标记, 用于指示 refine 阶段新增的字段 -->

- Claims and Claims matrix:
  - C1 (D 对真实边变化的稳健性): D 通过 graph-only-max family gate, 收益在部分可观测混杂、非线性边、独立 pre/post 动力学下依然显著, 不是 v4 DGP 构造的人为产物。
  - C2 (J 的窄边界): J 仍只在部分强边变化 cell 通过 family gate, 不得用 combined AUC 掩盖。
  - C3 (混杂漂移惩罚更加脆弱): 加固后惩罚仅在一个 phase x 可观测度组合显著, 比 v4 更不稳定, 不得声称一般性惩罚。
  - C4 (无独立预警价值): 所测估计器均未超过完整原始变量 EWS。
  - C5 (归因边界): 不等于真实 DAG 恢复、物理分岔或外部天气有效性。
  - C6 (估计器普适性未决): graph-only 收益在官方 LKIF smoke-scale 复现中方向相反, 与 v3 真实 PCMCI+ 一致; 全规模复现前不得声称跨估计器一般性。

  | Outcome | 允许的 claim |
  |---|---|
  | HARDENING-ROBUST | 仅限 partial-correlation 家族: 三处加固后 graph-only 收益依然显著, 不依赖清洁混杂/线性/同曲线平移三个构造假设; 不得称跨估计器已确认。 |
  | ESTIMATOR-SPECIFIC | 若全规模 LKIF/PCMCI+ 复现仍方向相反, 只报告 fixed-edge partial-correlation 边界, 不推广到 LKIF/PCMCI+。 |
  | ESTIMATOR-CONFIRMED | 若全规模复现方向一致, 可声称该模式跨两个真实公开估计器成立; 尚无证据支持。 |
  | NULL/ARTIFACT | 若空对照失准或加固后模式消失, 降为测量装置 artifact, 停止真实数据阶段。 |
- Narrative: 论文问题仍是"比较估计器不稳定性时, 何时能把差异归因于条件化", 但 v5 把"结论是否只是 DGP 人为构造"这一反对意见转成三处同时加固的压力测试, 并把真实数据阶段从单纯验证扩展为验证+探索。若这一模式在真实 HRRR 4D 场上重现, 其价值不止于预警, 还能检验两个具体、可证伪的领域假设: (H1) 若某类对流环境中 CAPE-shear 协同变化主要由天气尺度强迫 (如高空急流) 决定, 条件化收益应接近 confound-only-max 的窄边界 (小/不显著); 若主要来自边界层/冷池反馈等直接因果通路 (如 MCS 冷池增强下游切变), 收益应接近 graph-only-max (显著为正)。"条件化收益大小"由此变成可按已知风暴模态 (孤立超级单体/MCS-QLCS/脉冲对流) 分层检验的新诊断量, 而非只是预警特征。(H2) 若在 HRRR 格点场上逐格点计算 (D,J), 收益/惩罚的空间梯度是否勾勒出已知中尺度边界 (出流边界、干线，静态 CAPE/shear 场未必直接捕捉), 这会把因果条件化不稳定性变成独立于预警价值的边界探测工具。两个假设都是关于大气科学本身的可证伪陈述, 不依赖诊断能否跑赢 raw EWS。
- Experiments: (1) 非退化可交换空对照的 type-I/coverage 校验 (仍待办)。(2) 加固 DGP 已完成决策性角点; 若资源允许补齐 v4 原始 3x3 网格内部点位。(3) 全规模 PCMCI+/LKIF 复现 (96 pairs/8 seeds/完整 bootstrap), 现在是最高优先级、不可再推迟的下一步，当前只有 LKIF smoke-scale (32 pairs/2 seeds, 方向相反但非决定性) 结果。(4) 只有 (1)-(3) 都通过, 才对去重风暴系统做验证臂 (prospective power、严格提前量、cluster bootstrap/BH-FDR, 比较 CAPE/shear 与原始变量 EWS, 同 v4) 与探索臂并重: 按已知 storm mode 分层比较条件化收益是否对应 H1, 逐格点 D/J 空间梯度是否对应已知中尺度边界 (H2)。多 DGP/多 storm-mode 只提供 empirical/diagnostic 证据, 不构建 benchmark。
- Assets status: v5 加固 GPU 网关、官方 LKIF smoke 复现均已跑通; 完整环境场下载仍被停止门阻塞, 见 `workspace/causal-scs-indicator/data/MANIFEST.md`。

<review date="2026-07-12">

## Novelty

- Score: 4/10 (Claude 4, codex 3,差距在正常评审波动范围内, 均落在同一"仍不够"档位, 不算 Likelihood/Impact 意义上的分歧)
- Closest prior work:
  1. **Rothenhäusler et al., Anchor regression** (1801.06229) + **Kostin et al., 有限样本 minimax** (2606.12680) — 与 v4 一致, 仍是核心模式最直接的理论近邻 (population-level 决策论边界 vs 本 idea 的滑窗时序、部分可观测混杂场景)。
  2. **Siddique et al. (2026, submitted ESD), "Benchmarking Conditioners in Liang–Kleeman Information Flow: Application to Land–Atmosphere Interactions"** — codex 独立核查后指出, 该文并非只有 idea Novelty quick-check 所述的"固定系数静态 VAR 附录", 其主体已经在真实 land-atmosphere 站点/再分析数据上分析 bivariate vs. conditioned LKIF divergence 的时空/环境-regime 依赖并给出 compound-stress 领域结论。Claude 核实 `topics/0710-causal-scs-landscape.md` 的 AK 节确实已记录这一真实数据结果, 但 idea 正文自己的 Novelty quick-check 一句话只强调了"静态 VAR, 无滑窗框架", 客观上把这个最近的方法论近邻描述得比实际更远——这是本轮一处真实的、可修的表述精度问题, 而非撞车。
  3. **Causal-CCP (2403.12677)** — v4 review已指出应作为直接 baseline 正面比较, v5 仍未采纳, codex 再次强调, Claude 同意应在下一版列为 must-fix。
- Key differentiator: 把"扰动作用于协变量 (真实边) vs. 混杂/隐变量"这一已知决策论边界迁移到滑窗时序、临近分岔、部分可观测混杂场景下做压力测试, 这条路径本身仍是迁移检验而非新边界 (idea 自己承认)。H1/H2 若能被严谨执行, 理论上能把这一压力测试从"合成审计"延伸到"真实大气数据上的可证伪领域发现", 但目前二者都只是设计, 零真实数据证据。

## Quality

评估视角: topic (`topics/0710-causal-scs.md`) 仍未声明 `target-venue`/`preferred-contribution-types`, 沿用 v1-v4 推断, 按顶级大气科学/Earth-system methods/AI4Science 视角评估 (Claude 与 codex 一致)。

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 4/10 (Claude 与 codex 一致) | 三处 DGP 加固真实可信 (Claude 独立读 `factorial_helpers_v5.py` 确认: 噪声代理 `w`、`tanh` 非线性边、pre/post 独立 AR 系数+噪声尺度), 但"不是构造 artifact"这一headline仍偏强: (a) **codex 独立发现、Claude 用原始 JSON 复核确认为真的新问题**——raw `stress_pilot_results.json` 实际是 graph_strength×confound_shift 的完整 2×2 交叉 (16 cells), 而不只是 PILOT_PLAN_V5 声称的"决策性角点"; 其中被 idea 正文完全未提及的"混合扰动"格 (g=0.4 且 h=0.5 同时非零) 在 obs=1.0 时收益仍显著 (pre combined Δ=0.039 [0.001,0.073]; post 0.059 [0.030,0.089]), 但在更贴近真实大气 (真实边变化与混杂漂移大概率同时发生且混杂只能部分观测) 的 obs=0.5 时收益转为不显著 (pre 0.020 [-0.009,0.046]; post 0.017 [-0.009,0.043], 两者 CI 均含 0)。这恰好是对"加固后模式依然稳健存活"这一headline 最不利、也从未被正文引用的一格。(b) Hypothesis 段"方向已被 anchor regression/有限样本 minimax 决策论边界预测"一句用词偏强 (codex 指出, Claude 同意): 未给出任何数学桥接或对 `cor:linear-scm` 的 Θ(t²) 标度律做定量检验, 更准确的措辞应是"与该边界定性一致/迁移检验", 而非"已被预测"。(c) LKIF 部分见下方"Missing evidence signals"与专项核查。 |
| Missing evidence signals | 3/10 (Claude 与 codex 一致) | 沿用 v4 尚未解决项 (非退化空对照 type-I/coverage、window/stride/J-quantile 敏感性、bootstrap 分辨率与 multiplicity 校正、Causal-CCP 正面对比, 均 ignored) 之外, 本轮新增: 全规模 PCMCI+/LKIF 复现仍未执行 (只有 smoke-scale); H1/H2 均缺乏可操作化协议——无独立 storm-mode/机制 ground-truth 标注方案、无预注册 estimand、无相对既有边界探测算法 (dryline/outflow 已有湿度/风场梯度方法, 见下方专项核查) 的增量比较基线、无功效分析。 |
| Narrative | 5/10 (Claude 与 codex 一致, 较 v4 6/10 下调) | 叙事确有实质进步: 理论支柱与两个可证伪领域假设首次进入正文, 负结果 (raw EWS 全面占优) 继续被诚实保留。但本轮新增三处具体的表述精度问题拉低本项分数: 混合扰动格未被提及 (见 Logical gaps)、anchor-regression"预测"措辞偏强、以及下方专项核查确认的 LKIF"全部四格"过度概括。三者单独看都不算严重, 但共同效果是让叙事显得比证据更整齐、更一致。 |
| Venue contribution | 3/10 (Claude 与 codex 一致, 较 v4 3/10 持平) | 仍是单一 AR skeleton 上的 fixed-edge partial-correlation 结果 + 一次 underpowered smoke-scale 第二估计器复现, 零真实 4D 大气数据。本轮增加的理论 framing 和领域假设设计提升了"未来能不能做成"的可信度 (见 Likelihood-Impact), 但不改变"当前快照"作为顶会投稿的实质分量。 |
| Testability | 6/10 (Claude 与 codex 一致, 较 v4 8/10 下调) | 阈值/门槛延续 v4 的精确预注册风格, 代码可复现、原始 JSON 可独立核验 (本轮全部数字经 Claude 独立用 Python 重新计算校验, 除下述两处外均准确)。但 v4 曾被专门称赞的"预注册先于结果"git 证据链本轮断裂: Claude 独立 `git log` 核验确认, `PILOT_PLAN_V5.md`/`factorial_helpers_v5.py`/两份结果 JSON 全部只出现在同一次事后提交 `0fe5e8e` 中 (无 v4 式的"prereg commit 早于 result commit 12 分钟"式独立时间戳证据); 更值得注意的是, `stress_pilot_results.json` 里动态写入的 `runtime.git_commit` 字段记录的是旧提交 `6333d6a` (v4 的结果提交), 而非任何 v5 提交, 且本地文件 mtime 显示 `PILOT_PLAN_V5.md` (23:54:23) 实际晚于 `stress_pilot_results.json` (23:49:39)。这不能证明事后设计 (GPU 结果可能来自远程 SLURM 节点、mtime 经同步后不可靠), 但确实意味着这一轮不再具备 v4 那种可独立核验的"计划先于结果"证据, 是一项具体的、可指出的纪律倒退。 |
| Outcome realism | 4/10 (Claude 与 codex 一致, 较 v4 5/10 下调) | raw EWS 连续第 5 轮 (v2/v3/v3.5/v4/v5) 全面占优 (0.88-0.99), 无一例外; 达到顶会仍完全依赖目前没有任何证据支持的真实数据"排序反转"。H1/H2 若被朴素执行, 其最可能结果 (per Claude 与 codex 独立文献检索) 是重新确认已被气象学证实的 storm-mode/CAPE-shear 关联与既有边界探测方法的已知能力, 而非产出意外发现——这是新增的现实性折扣, 而非仅延续 v4 的旧折扣。 |
| Contribution type compliance | n.a. | idea types = {empirical-finding, diagnostic}; topic 未声明 `preferred-contribution-types`, 跳过, 不触发 hard cap (Claude 与 codex 一致)。 |
| Overall Quality | 4/10 (Claude 与 codex 一致; 6 项计分维度均值 25/6≈4.17, 取整 4; 较 v4 5/10 下调) | 一个不直观但站得住的结论: v5 做了真实、有价值的新工作 (DGP 加固、理论整合、领域假设设计、第二估计器实现), 但同一轮新引入了三项独立、可核验的表述精度问题 (混合格未报告、LKIF"全部四格"误报、anchor-regression"预测"措辞偏强) 和一项纪律倒退 (预注册时间证据链断裂), 三者合计的扣分超过了叙事/理论进步的加分, 使 Overall Quality 相对 v4 净下降而非上升。 |

## Contribution Drift (n >= 2 only; n=1 写 N/A)

- v_{n-1} (v4) contribution types: {empirical-finding, diagnostic}
- v_n (v5) contribution types: {empirical-finding, diagnostic}
- Status: unchanged
- Hard cap triggered: no (topic 未声明 `preferred-contribution-types`; idea 正文 "Contribution drift note" 与 REFINEMENT_LOG.md 均未提及新增/删除类型; Claude 与 codex 独立核验一致; 唯一的流程瑕疵是 `REFINEMENT_LOG.md` 本轮未追加"v5 refinement decisions"小节 (仅到 v4 为止), 设计决策改记录在 `PILOT_PLAN_V5.md`, 内容等价, 不影响 drift 判定)

v4 `<review>` concern-by-concern 复核 (Claude 与 codex 独立复核后合并):

| v4 concern / 特别核查项 | Status | Assessment |
|---|---|---|
| (dispatcher 特别核查 a) 真实 4D SCS 数据应用路径需含发现导向假设 | partially resolved | H1/H2 两个可证伪领域假设已写入 Narrative/Experiments, 真实数据阶段从纯验证扩展为验证+探索, 是真实的设计进步; 但零操作化协议 (无独立 ground-truth 标注、无预注册 estimand、无相对既有方法的增量基线)、零真实数据证据, 连续第三轮 (v3→v4→v5) 仍是纯设计层面的推进。 |
| (dispatcher 特别核查 b) 利用 iteration-3 deep-lit 发现的 anchor-regression/IRM 理论支柱 | partially resolved | 已实质进入 Hypothesis/Novelty quick-check/Strongest objection, 不再是 v4 那样完全缺席; 但连接停留在定性类比层面, "方向已被...预测"措辞偏强 (未做 Θ(t²) 标度律的定量检验), 更准确的是"理论类比/迁移检验"。 |
| 非退化可交换空对照 type-I/coverage | ignored | Experiments 段仍列为"待办", 是最便宜的下一 validity falsifier, 无合理 pushback 支持继续推迟。 |
| 非线性/local-independence DGP | partially resolved | `tanh` 非线性已实现; local-independence 框架仍未触及。 |
| 第二估计器 (LKIF) | partially resolved, 且新增报告精度问题 | 官方包 smoke-scale 复现已完成 (较 v4 的"deferred"是真实进步), 但全规模复现仍推迟, 且 smoke 结果的"全部四格方向相反"表述被 Claude 与 codex 独立核验证伪 (见下方专项核查)。 |
| 真实风暴功效/去重/严格提前量 | ignored/deferred, 有据 | 停止大下载的资源排序 pushback 成立 (synthetic gate 应先过), 但顶会证据缺口客观上仍未缩小。 |
| J 的 window/stride/quantile 敏感性 | ignored | 沿用同一 9 窗口/90th-percentile 定义, 无 sensitivity 分析。 |
| bootstrap 分辨率与 multiplicity 校正 | ignored | 仍是 400 次 bootstrap + 多 cell×channel 对比, 未做 multiplicity 调整。 |
| Causal-CCP 正面 baseline | ignored | v4 review 已指出, v5 仍未采纳, 无合理 pushback, 本轮重新强调为 must-fix。 |
| 连续扰动强度/Θ(t²) 标度律检验 (v4 review Alternative Framing #2) | ignored | v5 只跑离散角点, 未检验理论定量预测; idea md 未给出不采纳的理由, 本轮重新强调。 |
| raw EWS 全面占优, 需真实数据排序反转 | unresolved, 诚实承认 | v5 继续如实报告 0.88-0.99, 无缓解证据, 风险随迭代次数累积。 |
| Contribution scope 保持 empirical-finding+diagnostic | resolved | 无新增/删除类型, 正文 drift note 属实。 |

未采纳建议的 pushback 复核 (Claude 与 codex 一致): "先完成加固 DGP 压力测试, 再排 LKIF/真实数据"这一执行顺序有据, 接受; 但"用 smoke-scale 而非全规模 LKIF"和"继续推迟非退化空对照、window/stride 敏感性、Causal-CCP baseline、Θ(t²) 标度律检验"均缺乏在 idea md 中给出的明确理由, 不能视为有据的阶段性推迟, 应在下一版 must-fix。

## 专项核查 (dispatcher 本轮指定)

**1. LKIF"方向相反"发现是否被诚实、准确呈现**: **不是**。idea 正文 (Pilot Result) 与 `EXPERIMENT_LOG.md` 均写"4 个非空 cell 的 conditioned_true 点估计全部 <= sham/marginal"/"at or below sham/marginal in all four non-null cells"。Claude 与 codex 独立读取 `results/lkif_smoke_results.json` 原始数值后一致确认此说法为假: graph-only-max/post 格的 combined AUC 为 conditioned_true=0.9331 > conditioned_sham=0.9301、marginal=0.9303——方向与另外 3 格 (均 true < sham/marginal) 相反, 且恰好与加固后 partial-correlation 家族"graph-only-max 收益"同向。准确表述应为"3/4 格方向相反, 1 格 (post, graph-only-max) 方向一致, 32 pairs 下三者 CI 高度重叠, 点估计方向本身不构成配对显著性检验"。这是"过度声称"而非"淡化"——把非决定性、非一致的结果包装成了整齐的"全部相反", 且被掩盖的那一格恰好是加固后 partial-correlation 家族全部 cell 中效应量最大的一格 (Δ=0.054), 如果被如实呈现, 反而会削弱"LKIF 系统性不复现"这一叙事的整洁度。数据可信度本身没有问题 (原始数值真实、未捏造), 问题在于文字概括。

**2. H1/H2 可证伪性与科学价值独立评估**:
- **H1** (条件化收益按风暴模态分层, 映射到"天气尺度强迫 vs. 冷池反馈"机制): 表层可证伪 ("不同 storm mode 间条件化收益是否有差异"可以测), 但其真正声称的机制归因不可清洁证伪——storm mode 本身很可能是环境场与对流反馈共同作用的下游结果而非机制的独立随机标签, 按 mode 分层比较存在选择偏差风险。Claude 与 codex 独立文献检索均确认: 不同 storm mode 的 CAPE-shear 组合分布、以及 mode 演变对边界强迫/切变的依赖关系, 已是气象学界确立的知识 (如 storm-mode/CAPE-shear 关系研究、cold-pool 驱动对流起始的因果图分析 Hirt et al. 2020 QJRMS)。因此 H1 的朴素正结果大概率只是用新仪器重新确认已知气象学, 而非交付意外洞见, 除非补充独立于 storm-mode 分类本身的机制标签。科学价值: 中低, 除非重新设计。
- **H2** (逐格点 D/J 空间梯度对应已知中尺度边界): 更干净地可证伪——可用独立雷达/地面分析边界标注、object-based 距离或 IoU 指标、留出个例做盲测。但现有 dryline/outflow-boundary 探测算法已直接利用湿度梯度、风场切变等原始场, 因此 D/J 梯度与已知边界重合本身并不意外; 唯有证明其相对这些既有原始场梯度方法具有可量化的增量 (更早探测、更准确定位、或揭示原始场未捕捉的新边界), 正结果才具有真正的科学价值——而这一"增量对比"目前未被写入实验设计。科学价值: 中, 可通过补充增量基线对比大幅提升。

综合看, H2 现阶段是比 H1 更值得投入的领域假设; 若只能选一个作为下一版重点, 应优先操作化 H2 并显式加入相对原始场梯度方法的增量基线要求。

**3. 真实 4D SCS 应用路径与理论支柱是否实质补齐 (dispatcher 特别核查, 承接 v4 review)**: 实质推进但未完全补齐, 详见上方 concern-by-concern 表前两行。总结: 这不是第三轮回避, 而是"设计层面部分兑现, 证据层面仍是零"——dispatcher 若要求下一版继续推进, 应明确要求 H1/H2 至少一个的操作化协议 (标注方案+估计量+基线) 而非仅有假设陈述。

## Alternative Framing

Claude 与 codex 独立收敛到同一方向, 且比 v4 review 提出的三个选项更聚焦: 将 headline 收紧为"conditioner-sensitivity 的跨估计器可识别性/失效边界审计"——完整披露全部 16 格 (含此前未报告的混合扰动格), 把 fixed-edge partial correlation、PCMCI+、LKIF 三者的一致/不一致本身作为核心贡献 ("何时 conditioner divergence 是物理信号、何时只是 estimator artifact"), 真实 SCS 数据阶段降级为相对既有 storm-mode/边界 diagnostics 的 held-out 证伪测试, 而不是与 estimator-audit 并重的第二条headline故事。这样既不需要引入 preferred-contribution-types 之外的类型 (topic 未声明该字段, 不受 hard cap 约束), 也把 H1 (科学价值较弱) 自然降级为探索性附加项, H2 (科学价值较强) 作为更有希望的候选主线。

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 仅能声称: 在本轮加固的单一 AR skeleton 中, graph-only-max 角点 (真实边变化, 无混杂漂移) 的条件化收益在两个 phase、两档可观测度下对 fixed-edge partial correlation 家族显著为正; 该收益在"混合扰动"格 (真实边变化与混杂漂移同时发生) 的部分可观测条件下并不稳健 (CI 含 0), 不得笼统声称"加固后模式全面存活"。不得声称跨估计器一般性 (LKIF smoke 3/4 格方向相反), 不得声称 anchor-regression/minimax 理论已经"预测"该方向 (只能称定性一致), 不得声称 H1/H2 已获得任何真实数据支持 (二者仍是未执行的设计)。 |
| NULL | 可声称混合扰动格在低可观测度下的收益、以及 LKIF smoke 三个非空格的方向差异, 在当前样本规模下均未达到决定性 (CI 宽/高度重叠), 不能写成效应不存在或方向确证反转。 |
| NEGATIVE | 可声称 raw EWS 连续 5 轮实证中全面占优, 无一例外; LKIF smoke 未能跨 phase 一致复现 partial-correlation 家族的 graph-only 收益 (3 负 1 正)。不得因此判定所有因果不稳定性诊断均无效, 也不得把这一 synthetic/smoke-scale 结果外推为强对流领域定论。 |

## Likelihood-Impact Matrix

- Priority: Medium = Likelihood: Low x Impact: High
- Numeric score for ideas.xml: 5
- Rationale:
  - Likelihood: Claude 与 codex 一致判定 Low, 无分歧。理由: v5 的加固实验和第二估计器执行是真实工程进步, 但本轮新独立发现的两项证据 (混合扰动格在低可观测度下收益消失、LKIF smoke 3/4 格反向且被过度概括为 4/4) 恰恰指向"跨条件/跨估计器一致性"这一顶会必需的关键环节比 v4 review 判断的更脆弱, 而非更稳固; 加上非退化空对照、window/stride 敏感性、multiplicity 校正、Causal-CCP baseline、H1/H2 操作化协议均仍未完成, 以及连续 5 轮 raw EWS 全面占优仍无缓解迹象, 整体判断维持 Low (与 v2/v3/v4 一致)。
  - Impact: Claude 与 codex 一致判定 High, 无分歧。理由不变于 v4: 若最乐观情形——跨 DGP、跨估计器、跨真实强对流数据都稳定复现"扰动来源决定条件化方向", 且诊断在严格提前量上提供 raw baseline 之外的增量, H1/H2 也交付超出已知气象学的新发现——会为因果诊断目标与预测目标何时分歧这条研究线提供一个从未在滑窗时序场景中被检验过的清晰边界, 具备顶会发表价值; 但工具与理论解释均已存在于相邻文献, 问题域相对窄, 不到 Exceptional。
  - 查表: Low x High = Medium (5)。topic 无 `preferred-contribution-types` 声明, hard cap 不适用, numeric score 不截断。与 v2/v3/v4 review 的 Low×High=5 结论完全一致——本轮新证据没有改变象限, 只是让"为什么仍是 Low"的理由更充分、更具体。

## Overall

- Priority: Medium
- Score: 5
- Comments: v5 在理论整合 (anchor-regression/minimax 首次进入正文)、领域假设设计 (H1/H2)、第二估计器实现 (LKIF 官方包 smoke-scale) 三个方向上都做了 dispatcher 本轮明确要求的真实工作, 不是第三轮空转; 但 Claude 与 codex 独立核验一致发现三项新的、具体的报告精度问题——(1) 未报告的"混合扰动"格在更贴近真实大气的部分可观测条件下收益消失, (2) LKIF smoke 结果被概括为"全部四格方向相反", 实际 3/4, 且被掩盖的一格恰是效应量最大的一格, (3) anchor-regression"预测"措辞超出了实际完成的定性类比——外加预注册时间证据链断裂 (v4 有独立 git 时间戳证明, v5 无)。四者合计使 Overall Quality 相对 v4 净下降 (5→4/10), 但 Likelihood-Impact 象限保持 Low×High=Medium(5) 不变, 与 v2/v3/v4 一致, Claude 与 codex 本轮在 Likelihood/Impact 两个轴上均无分歧。下一版 must-fix: 全量披露 16 格结果 (含混合扰动格)、改正 LKIF 表述为"3/4 格反向"、将 anchor-regression 措辞降级为"类比"、恢复可独立核验的预注册时间证据链、以及至少操作化 H2 (含相对既有边界探测方法的增量基线)。

</review>
