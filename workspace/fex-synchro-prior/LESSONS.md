# Lessons: fex-synchro-prior

<!--
LESSONS.md: 本 workspace 内沉淀的可迁移知识总账.
与同目录另两份文档三足互补:
- STATE.md 是一篇 mini-文章 (送审快照), 记载了最后会呈现在文章上的精华
- experiment-log.md 答 "这个 workspace 经历了什么" (时间倒序流水账), 记录了所有细节
- LESSONS.md 补全 STATE.md 和 experiment-log.md 中间缺失的层次, 是提炼后的可迁移知识

该记什么:
1. 人类嘱托: 用户在本 workspace 工作过程中的指令, 每条带 [YYYY-MM-DD]. 记录人类嘱托时必须记录用户原文; 只允许修正明显 typo, 不得改写、概括、翻译、润色或重排。可在原文后另写"当时情况"和"Agent 注释", 但必须明确标注为 agent 注释, 不得替代、扩展或冒充用户原文。
2. 可迁移经验: 调参经验, 工程 trick, 失败的教训
3. 被搁置的路线: 之前哪些路线觉得重要性不高/性价比不高被搁置了, 将来有空可以继续做, 或者作为其他路线的参考

保持言简意赅, 并且随着新的知识的加入不断整理合并, 就像对 memory 的操作一样.
不要滥用加粗, 处处加粗等于没加粗.
不要删除本注释, 一直保留作为本文件的填写指引.
-->

## 工程经验

### Multi-Scale FEX 实现陷阱 [2026-06-19]

1. Reward clamp 不能硬设上界。helmholtz_sine 的 PDE 误差 O(1e5)，clamp=100 会压平所有梯度。用 nan_to_num 替代。
2. Alpha_i（频率缩放参数）在 LBFGS 下容易爆炸。seed1 alpha 发散到 -12618。每步 Adam/LBFGS 后 clamp alpha 到 [0.01, 20.0]。
3. CandidatePool（top-10 action/error pairs）必须持久化到 checkpoint。否则 resume 会丢失所有候选。
4. 外部 timeout 和 Python timeout 不能设成同一值。搜索阶段消耗几乎全部时间后，进程在 finetune 前被杀。外部 timeout 应 > 搜索 + finetune 总时间。

### 控制组设计经验 [2026-06-19/20]

init_only（仅初始化 alpha 到真实频率）的 search curve 与 standard byte-for-byte 相同——因为 alpha 不参与 RL policy 的 action selection。init_only 只是一个 finetune-phase control，不是 search-phase control。设计控制组时要确认干预是否作用于目标阶段。

### Arnold 服务器注意事项 [2026-06-19]

- RTX 8000 是 Turing 架构，不支持 BF16。FEX controller 很小，fp32 即可。
- Python stdout 通过 tee 时 block-buffered，train.log 只有 warning。真数据在 train.jsonl。未来 run 加 `PYTHONUNBUFFERED=1`。
- 14400s wrapper timeout 对 1000 epoch search + 10 candidates finetune 勉强够用。搜索慢的 seed（12.4s/epoch）会超时。建议 18000s。

## 科学经验

### Soft prior > hard prior [2026-06-20]

oracle_hard（强制正确频率）2/3 失败一个 seed（s0: 1.82e-3），方差 1397x。oracle_soft（logit bias）3/3 全部机器精度，方差 1.2x。可能原因：hard constraint 消除了频率动作的探索，但 soft bias 在引导频率的同时保持了 exploration，间接帮助结构搜索收敛。这个 "soft > hard" 的发现可能比 "频率是瓶颈" 本身更有趣。

### 3-seed → 8-seed 叙事翻转 [2026-06-21]

oracle_soft h7pi 的 3-seed 数据（variance 1.2x, 3/3 成功）看起来像"消除方差"。8 seeds 暴露了 s5 的完全失败（variance 2747x, 6/8 成功），叙事从"消除方差"变成"提高收敛率"。教训：小样本下的 100% 成功率极易误导。
更值得记录的是：s5 的 first_hit=eval 2 证明 oracle 成功引导了频率，但搜索仍然失败——这是频率和结构两个独立失败模式的因果分离证据，比"全部成功"更有论文价值。

### 频率距离 vs 性能降解 [2026-06-21]

base set {3..24} 范围内的目标频率（h5pi=15.7, h7pi=22.0）oracle_soft 有效。超出 base set 时，性能随距离降解：
- 44% 超出 (h11pi): oracle_soft 有 ~12% 部分优势（mean relL2 0.84 vs 0.95），但不收敛
- 149% 超出 (h19pi): oracle_soft ≈ standard，无优势
这个降解梯度可以作为论文 scope discussion 的定量依据。

### 主判据选择影响统计结论 [2026-06-21]

search convergence（search_error<10）和 relL2<1e-5 给出不同的 standard 计数（h7pi: 1/8 vs 2/8），因为 standard-s7 搜索未收敛但 finetune 偶然救回。选择主判据要基于干预的作用阶段（oracle 作用于搜索），而非对自己最有利的数字。

### Search convergence 应使用 min search_error 而非 best-candidate search_error [2026-06-22]

FEX 搜索阶段生成多个候选表达式树。按 relL2（finetune 后）选的最佳候选的 search_error 不一定是搜索阶段的最佳结果。正确定义 search convergence = min(all candidates' search_error) < 10。这个区别在 h7pi-s3 和 s6 上造成了 6/8 vs 7/8 的误差——搜索阶段确实找到了低误差候选，只是 finetune 碰巧选了另一个候选。

### 频率保持（retention）vs 频率发现（finding）[2026-06-22]

Gate 0 最重要的机制发现。h4pi 和 h8pi 的 first_hits 显示 standard 在 epoch 0 的前几次评估内就找到了正确频率（h4pi: eval 3,5,5; h8pi: eval 2,3,1），但 2/3 seeds 在后续 RL 探索中丢失了。Oracle_soft 的价值不是帮找频率（它的 first_hit 差不多快），而是在整个搜索过程中持续 bias logits 防止 RL exploration drift 导致频率遗忘。这比"搜索空间太大"更有深度——它揭示了 RL exploration-exploitation tradeoff 在频率动作上的根本缺陷。

### 三种独立失败模式 [2026-06-22]

RL 符号 PDE 搜索的失败可以分成三类独立模式：(a) 频率漂移——找到频率但 RL 探索中丢失（standard 最常见）；(b) 结构搜索失败——频率保持成功但 expression tree 结构搜索崩溃（h7pi-s5, first_hit=eval 2 but min_se=520）；(c) finetune 不完全——搜索收敛但 Adam/BFGS 未收敛到机器精度（h7pi-s4, min_se=0.40 but relL2=4.4e-5）。这个分类学对论文有价值。

### Risk-seeking PG 的 top-ε 过滤天然导致 retention failure [2026-06-22]

来自 lit-feed 论文 2510.18927 (BAPO) 和 2505.22617 的洞察：FEX 的 risk-seeking PG 只保留 top-ε 样本更新 controller。当正确频率暂时被遗忘（概率下降）后，即使该频率偶尔被采样且获得高 reward，它出现在 top-ε 中的概率也极低（因为 batch 中大部分样本来自当前高概率动作）。这是 Cov(log π, π·A) > 0 的具体机制：高概率动作的 advantage 被系统性放大，低概率动作即使有正 advantage 也被过滤。Oracle_soft 的 logit bias 人为维持正确频率的概率，使其不落入 top-ε 过滤的死区。

### REPO 作为 oracle_soft 的潜在替代 [2026-06-22]

来自 lit-feed 2603.11682：REPO（修改 advantage 引入零显存熵控制）可作为 FEX 频率节点的 entropy regularizer，不需要知道正确频率。如果我们将来需要一个 estimated-prior-free 的 retention 机制，REPO 是候选。但当前阶段，oracle_soft 作为 diagnostic tool 已经足够。

### 维度效应 vs PDE 类型效应 [2026-06-22]

1D convdiff (dim=1) 上 standard 也收敛，看起来像"Oracle 只对 helmholtz 有效"。实际上所有 helmholtz 实验都是 2D (dim=2, freqs=[kπ,kπ])，搜索空间 20²=400 组合。1D 只有 20 个选择，retention 不是瓶颈。"Cross-PDE" 实验必须控制维度——不能拿 1D 和 2D 比。

### Tfp 轨迹比收敛率更有解释力 [2026-06-22]

收敛率 (16/17 vs 4/17) 说明 oracle 有效但不解释为什么。Tfp 轨迹直接展示了因果机制：早期 4.3x 概率差异 → 表达树结构承诺 → 不可逆。更关键的是 standard 最终也到 0.50 tfp，说明"找到频率"不是问题——"保持频率"才是。这比"搜索空间太大"的直觉解释更精确、更不平凡。

### Soft/hard 最优注入强度取决于 PDE 结构搜索复杂度 [2026-06-22]

Helmholtz: soft(16/17) > hard(8/12)。Convdiff2d: hard(3/3 机器精度) > soft(1/3)。关键区别是 PDE 给定正确频率后的表达树结构搜索难度。Helmholtz 的对称 Laplacian 允许多种可行树结构，soft 保留的探索有助于发现；convdiff 的 advection 项约束更强，可行结构更少，hard forcing 消除的频率探索噪声不再伤害反而更干净。这说明"soft always better"是 helmholtz-specific 的偶然，真正的规律是"最优注入强度 = f(PDE 结构复杂度)"。论文中应避免简单声称 soft>hard，而应呈现这个交互效应。

### 注入延迟比最终频率概率更关键 [2026-06-23]

Gate 1 reward_bonus 到 ep199 的 target_freq_prob 可接近 oracle_soft，但 ep9 仍是 standard 水平（0.083 vs oracle_soft 0.34），因此搜索阶段 0/3 收敛。结论：对 FEX controller 的频率先验，早期 action-channel logits bias 比 reward-channel shaping 更有效；报告 TFP 时必须按时间轴看，不只看最终值。

### search_error 与 relL2 会严重脱钩 [2026-06-23]

CP-s2 cand0 search_error=0.111 但 relL2=0.214；cand9 search_error=558 但 relL2=9.06e-7。search convergence 仍适合作为搜索阶段判据，但不能替代最终质量；以后所有 Gate 1/2 表都要同时报告 candidate-level search_error、relL2、rank 和是否含目标频率算子。

### Checked FFT prior 的第一轮真实信号 [2026-06-23]

Gate 2 h7pi clean 2/3 search convergence，ep9 TFP=0.510，高于 oracle_soft 的 0.343。原因不是 oracle 弱，而是 width cap 把 support 收到 {21,24} 两个 base；这给更强早期引导，也带来更高估计误差敏感性。s2 final TFP=0.902 仍失败，说明频率锁定后仍可能死在结构搜索。

### NCPSD 当前不是可用 estimator [2026-06-23]

Gate 2 smoke 中 NCPSD 在 h7pi、h8pi、convdiff2d 都未覆盖目标 base；FFT 全部覆盖。后续不要把 "spectral estimator" 泛化成多个 estimator 都有效。当前 claim-bearing estimator 只能是 checked FFT，NCPSD 是 negative baseline。

### Decoy control 需要分层 [2026-06-23]

远峰 decoy（freq=12, amp=2000）被 validation gate 正确拒绝，但这只证明最简单泄漏能挡住。近目标 decoy、多峰 decoy、低幅 decoy 的风险不同；尤其 near-target decoy 可能被 width-capped support 吸收，必须看 selected bases、validation residual 和最终搜索是否退化。

### Width cap 会改变 soft prior 的有效强度 [2026-06-23]

Gate 2 的 `width_cap=0.25` 只保留 2/8 个 base frequency，因此 validated estimated-soft 在行为上比 oracle_soft 更像 hard/near-hard prior。这个浓缩解释了 h7/h8 的 ep9 TFP 高于 oracle_soft，也解释了 convdiff2d 上 estimated_soft 的 median relL2≈oracle_hard。以后不能把 "estimated_soft" 直接等同于 oracle_soft；必须报告 selected_bases、width_fraction，并做 width sensitivity。

### Validation gate 固定阈值要实测灰区 [2026-06-23] → 已实测 [2026-06-24]

~~当前灰区未测试。~~ 已完成：boundary sweep 36 scenarios 找到 7 grey-zone (res 0.30-0.70)；两个最危险场景 (decoy24-amp400, decoy21-amp1500) 在真实 controller 上 6/6 机器精度。Validation gate 不检测所有 contamination，但通过 gate 的 contamination 无害——因为 width_cap 保证 selected bases 仍覆盖 target。Claim 从 "gate rejects contamination" 重构为 "method tolerates near-target; gate blocks far-freq"。

### 近目标 contamination 是无害的——机制鲁棒性 > 检测安全 [2026-06-24]

Grey-zone 实验最重要的教训：validation gate 对 near-target moderate-amp decoy 完全透明（ACCEPTED），但 6/6 seeds 仍达到机器精度。原因是 width_cap 的几何性质——当 decoy freq 在 target 邻域内时，即使 FFT 估计被 contaminate，width_cap 选出的 bases 仍覆盖 target base。这说明 method 的鲁棒性来自 spectral locality 的内在性质，不依赖检测能力。论文叙事应强调机制鲁棒性而非检测能力。

### Resume seeds 的 first_hit 不可靠 [2026-06-24]

cd2d-cap0125-s2 的 first_hit 被报告为 ep900/eval5400，但 ep9 TFP=0.9995 与其他 cap0.125 seeds 完全一致。原因是该 seed 在 ep949 超时后从 ep899 resume，first_hit 检测从 resume 起点重新开始计数。后续报告 first_hit 时应排除 resume seeds 或标注。

### Width-cap 确认不敏感 [2026-06-24; corrected 2026-06-25]

cap 0.125（1/8 base ≈ oracle_hard）和 cap 0.50（4/8 base ≈ oracle_soft）在 3 PDE 上均有效。Reviewer v1 纠正了 balanced denominator：按每个 cell 3 seeds 统计，3 caps × 3 PDE 是 25/27 search conv、23/27 machine precision；旧的 24/27 混用了 h7pi cap=0.25 的 8-seed扩展和 3-seed cell。结论仍是 width 不脆弱，但数字必须按同一 denominator 报告。

### Gate validation 不能偷看 final scoring target [2026-06-25]

Reviewer v1 指出 Gate 2 `validated_estimated_soft` 用 `true_sol` 做 validation，导致 "checked without leaking" claim 失效。以后所有 deployable gate 必须声明 input channel：gate 可以用 PDE residual、BC、observed RHS 和固定预算候选拟合；final relL2 可以用真解评分，但不能反流到 gate。2512.21319、2606.12050、2507.02227 提供残差/误差认证的理论垫脚石。

### RHS-only decoy gate 有可辨识性边界 [2026-06-25]

如果 decoy 真的污染了 observed RHS，任何只看 observed RHS 的 validation 都可能把 decoy 当成真实 PDE 频率；这不是实现 bug，而是信息论边界。下一轮必须区分三件事：clean PDE 的 no-true-sol validation、solution/RHS spectrum mismatch 的 nonlinear 控制、以及 adversarial RHS contamination 的不可辨识边界。不能再把 true_sol-based decoy rejection 写成安全证据。

### Retention 证据优先用 action-level TFP 而非输出 KL [2026-06-25]

lit-feed 2601.21669 支持 expected-return mode collapse 的结构性解释；2605.28860 提醒 output KL 不能代表内部保持。对 FEX controller，最直接的 retention 证据仍是 per-node target_freq_prob trajectory、target_freq_sampled 和 first-hit/late-loss 分离；KL_from_init 只能做辅证。

### Multi-frequency 先做显式 capacity probe [2026-06-27]

mf-depth3-capacity-oracle 15/15 全 fail 说明“oracle 也失败”不能自动解释成频率 prior 无效。对组合表达式任务，必须先绕过 controller 做 fixed-action / fixed-alpha capacity probe：若显式表达式都无法到 relL2≤1e-5，根因是 compute graph、leaf isolation、alpha scaling 或 grammar；若显式表达式能过但搜索失败，根因才是 multi-component credit assignment。

### 重建 result.json 不能承载 gate 行为 claim [2026-06-27]

gate2-residval-decoy-greywidth 的 f12-s0/s1 只有重建的 result.json，没有 train.jsonl 或 `estimator.validation` trace；`decoy_rejected=true` 是推断，不是程序证据。以后凡是 claim-bearing gate 行为必须有 validation residual、tested tuples、passed/fallback 字段；重建文件只能当结果恢复线索，不能当 gate 主证据。

### Harder PDE 的 n=1 叙事可以被 n=5 完全推翻 [2026-06-27; confirmed 2026-06-28]

gate3 nonlinear gamma=10/100 seed0 给出很强但互相矛盾的信号：g10 prior 大胜 standard，g100 estimated 最好但 standard 又优于 oracle。n=5 结果：g100 的 arm 排序完全反转——oracle 4/5 machine 是最好的，VES 2/5 中等，standard 1/5 最差。iter12 的 "VES best, oracle worst" 全是 seed0 outlier。n=1 信号只能触发下一轮，绝不能写进 claim。

### 估计失败 vs 搜索 noise 要看 estimator 数据 [2026-06-28]

g100 VES 从 g10 的 4/5 降到 2/5，直觉以为是 FFT 估计在高非线性下失效。但检查 g100 VES 全部 5 seeds 的 estimator 数据：angular_freq=22.09≈7π (正确)，gate 全 PASS (res=0.075<<0.5)，selected_bases=[21,24] (与 g10 完全相同)。估计器是对的，降级来自高非线性下搜索 noise 放大——更难的 PDE 让 controller 在相同 prior 下更容易 wander。论文叙事不能写 "estimation fails at high gamma"，应该写 "estimation is robust, but search difficulty increases"。

### 穷尽低成本修复再宣布 fundamental bottleneck [2026-06-28]

mf-additive 15/15 FAIL 时，本能反应是"optimizer fundamentally 不行，需要重大架构改进"。但 capacity probe 证明 relL2=1.83e-6 可达，os s3 到 1.90e-3 已改善 200x。此时应先试：(a) 延长 finetune 到 50K-100K，(b) 换 L-BFGS，(c) multi-stage optimize，(d) 增加 coarse eval 步数。每项 <5 GPU-h，全做也只 <25 GPU-h。在这些低垂果实全 FAIL 之前不能声称瓶颈是 fundamental。

## 搁置路线

### Gate 1-3 全流程 [2026-06-22] → Gate 1+2 已完成 [2026-06-24]

~~Proposal 要求 Gate 1-3。Reviewer 4.3/10 指出缺失。~~ Gate 1 injection ablation 已完成（4 arms × h7pi）。Gate 2 checked FFT prior 已完成（cross-freq, cross-PDE, width, greyzone）。Gate 3 剩余: nonuniform sampling + nonlinear RHS，作为 scope limitation 标注。
