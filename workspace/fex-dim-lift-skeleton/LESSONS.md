# Lessons: fex-dim-lift-skeleton

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

## 人类嘱托

- [2026-07-01] 用户原文：
> 这里这几个 macro 说白了不就是模板吗？我背好了这几个解题套路去考试，试题被押中了我就会，不会就是 0 分，这不就是现在 STATE.md 的情况吗？
> 这个 idea 能够成立的背后的直觉就是"结构是广泛存在的"，进而我们可以在低维寻找结构，在高维应用结构，将不可解的高维问题变成可解的低维问题。
> 虽然结构广泛存在，但是我们也不能说把它写死成一套模板，规则性的东西是不可能解决所有问题的，这里的结构要是"学习出来的"，但是现在的这个 STATE.md，其结构是写死的，所以注定做不成。
> 自然界的结构不可能是规则能够覆盖的，它们必须是可学习的
> 整个"第二步（识别 macro）"的代码都要删除！全是垃圾！

当时情况：V1-V21 共 47 轮迭代，旧 macro library 方法在 Conservation 上成功但在所有其他方向碰壁——C3 无法分离、quartic 不 hard、Helmholtz d=3 搜索 0/5。用户诊断根因：模板匹配本质是"背题库考试"。Agent 注释：这一判断在科学上是准确的——47 轮实验的失败模式完全吻合模板匹配的固有局限。

## 可迁移经验

- [2026-07-02] Hard target 的结构性约束来自 FEX 离散搜索空间的 d-scaling，不来自连续参数维度。depth1 离散空间 ~243（2 unary 选择 × 1 binary + leaf），不随 d 变；depth2_sub ~324·d²，随 d 增长。因此 depth1 PDE（unary_replication 的领域）永远没有 activation gap——scratch 在 d=40 也能轻松通过。进一步，FEX unary 集 {sin,cos,exp,id,x²,x³,x⁴} 限制 depth2_sub hard targets 几乎只有 sin/cos of signed sums（Conservation 一族）。推论：追加 hard targets 不能靠换 PDE，要靠扩展 FEX 算子集或树深度。这是结构性限制，不是搜索不够。

- [2026-07-02] 中间标注有 bug 不等于整体结论无效——需要追溯 bug 是否在决策路径上。V22e parser 把 seed3 quartic 零系数分支误标 `is_active=true`，但模式选择逻辑只检查 transcendental root（L296），polynomial leaf 的 active 状态根本不在决策路径上。推论：audit 发现 bug 后，scientist 应先判断 bug 是否在 load-bearing path 上再决定 invalidate 范围——不是所有 bug 都 invalidate 所有结论。但同一个 bug 在扩展 primitive 时可能变成 load-bearing 的，所以必须修。

- [2026-07-02] Separable-sum（Σ g(x_i)）和 signed-sum（g(Σ c_i x_i)）的 activation gap 来源不同。Signed-sum 映射 depth2_sub，其离散搜索空间随 d 增长（~324·d²），d≥40 时 RL 探索不足 → 有 gap。Separable-sum 映射 depth1，其离散搜索空间固定（~243 种），高维 scratch 照样通过 → 没有 gap。推论：要让 tree insertion（unary_replication）展示 activation gap，候选 PDE 的 per-coordinate 函数 g 必须足够复杂以需要 depth2_sub+ 的表示——depth1 能搞定的结构不会有 gap。

- [2026-07-02] `tree` 配置名 + 表达式字符串扫描不等于 tree parser。V22d `auto_transfer.py` 用 `tree=="depth2_sub"` / `tree=="depth1"` 和 AST unary-call count 选择 transfer，结果看似移除了手写 `sin`，但仍没有重建 FEX 的 operator tree、`best_action`、活跃/常数子树和多 seed 稳定结构。教训：凡是 claim 说"从 tree 结构自动推导"，receipt 必须保存并解析 action-level node signature；配置名只能用来构造模板，不能直接决定科学分类。

- [2026-07-02] Leaf mixing 的 dimension consistency 在代数上不可能有意义地测试。证明：对 d_low×d_high 的 direct mapping W，当 d_mid ≥ d_low 时，总存在 W1(d_low×d_mid) 和 W2(d_mid×d_high) 使 W = W1@W2（取 W1=[I|0]，W2=[W;0] 即可）。因此 composed W1@W2 可表示任意 direct W，composition 没有约束力。这不是 test design 问题，而是 leaf mixing 作为线性叶替换的固有代数性质。推论：有意义的 dimension consistency 需要 transfer 机制本身对 composition 有结构约束，例如 tree insertion 中的树中间表示。教训：在设计 consistency test 前先检查被测对象的数学 DOF 是否允许非平凡约束。

- [2026-07-02] 硬编码 unary 函数名和旧 MACRO_LIBRARY 查表在本质上是同一个错误。V22c Helmholtz pilot 的 `fit_tree_insertion_high_dim(..., unary_name="sin")` 看似"从 tree 生成 transfer"，但 `sin` 是 scientist 写在代码里的，不是 pipeline 从 tree 自动抽取的。auditor 正确将此定性为 oracle fit，不是 §5 要求的 learnable transfer。教训：每次实现新 transfer primitive，必须验证 PDE/macro/函数名称不出现在 pipeline 的决策路径中——只有 tree 的结构信息（节点类型、深度、子树关系）才是合法输入。

- [2026-07-02] Helmholtz separable Σsin(x_i) 不是 hard target。iter50 scientist 预测 depth1 scratch 在 d=40 会失败（"只有 2 项不能表达 40 项之和"），但实测 d10/d20/d40 全 5/5 HIT (worst 1.11e-6)。错在哪里：忽略了 FEX depth1 的每个 leaf 是 ALL d 维变量的线性组合（d=40 时有 80 个连续参数），加上 Helmholtz Δu+u=0 约束 |a|²=1 使优化 landscape 非常光滑，LBFGS 毫无压力。教训：activation gap 的根因是 RL 离散结构搜索的组合爆炸（unary/binary 算子选择随 d 增加），而非连续参数空间的维度。depth1 离散搜索空间只有 ~243 个组合，d 增加不影响；depth2_sub 的组合空间随 d 增长，d=40 时 RL 探索不足。推论：要让 tree insertion 有 activation gap，候选 PDE 必须迫使 FEX 使用高离散复杂度的树结构（如 depth2_sub 或 depth3），不能用 depth1 能搞定的结构。

- [2026-07-02] 选择 tree insertion 候选 PDE 时，结构类型必须与 transfer 机制匹配。Leaf mixing 处理 g(Σ c_i x_i) ("signed-sum"，一个 unary 包裹变量之和)；tree insertion 处理 Σ g(x_i) ("separable-sum"，变量各自过 unary 再求和)。两者在数学上对偶：前者是 unary(sum(leaves))，后者是 sum(unary(leaves))。FEX 的 depth2_sub 配置自然搜出 signed-sum，depth1 配置可能搜出 separable-sum（binary(unary(leaf), unary(leaf)) 正好是两个 unary 分支求和）。教训：transfer 机制不是一个万能方案，每种机制有对应的结构类型和 FEX 配置，实验设计必须把这三者对齐。

- [2026-07-01] Helmholtz d=3 search 0/5 HIT (best rel_l2=0.9949) 尽管解 sin(Σx_i) 与 Conservation 相同。原因：Helmholtz 是二阶椭圆 PDE（Laplacian），domain [0,2π]^d vs Conservation 的一阶 transport PDE 在 [-1,1]^d，PDE residual landscape 完全不同。教训：FEX RL controller 的 searchability 取决于 PDE residual 梯度景观，不仅取决于解的数学结构。

- [2026-07-01] UniSymNet (2505.06091) 的 bi-level 优化（外层结构搜索 + 内层参数优化）可直接套用于 §5 "屁股方案" (tree-level insertion)：外层搜索在哪个节点插入、用什么聚合方式，内层优化 transfer 参数。Gumbel-Softmax + BFGS 组合用于离散-连续混合优化，beam search 生成 top-k candidates。UniSymNet 不处理维度变化（d≤10），但 bi-level 架构可迁移。

- [2026-06-30] **preflight "no candidate" 不等于 "no candidate exists"——只等于已知池里没有。** V20 preflight 检查 6 个已知候选产出 0 selected，被 audit 升为 BLOCKER。但 preflight 不能否定尚未实现/测试的候选。Helmholtz 是不在 V20 池中的全新 PDE，C1-C3 预测可行。教训：exhaustive search 的 "exhaustive" 必须限定 scope——preflight 排除的是已知候选，不是物理世界中所有可能的 PDE。

- [2026-06-30] **route preflight 不能把容易跑的 toy route 排在 §5 真实目标前面。** V19 preflight 选择 `new_operator_scout`，但没有真实 PDE provenance 和强外部 baseline 阶段，导致 audit45 BLOCKER。教训：下一条 claim-bearing route 的 preflight 必须先检查人类 §5 的硬条件；operator/library scout 只能作为诊断，不能替代真实目标和 baseline。

- [2026-06-30] **evidence repair 的 manifest commit 必须包含生成脚本。** V18 两个 evidence-integrity artifact 都写 `git_commit=a98b4f1`，但脚本分别在后续 coder commit 才出现，导致 zero-context reviewer 无法用 manifest 的 commit+command 复现。教训：repair 类 run 不能先运行后提交脚本；manifest 至少记录包含脚本的 commit、script SHA256、exact command 和 input hashes。若是 post-run reanchor，必须显式标为 repair，而不是伪装成原 run commit。

- [2026-06-30] **taxonomy/evidence map 不能把 claim closure 变成 scientist wording decision。** V18 Claim E pack 一边说不闭合 Claim E，一边给出 `taxonomy_sufficiency` 路线让 scientist 接受 bounded wording 即闭合；这在 §5 “Claim 永远不许降级” 下是 scope rewrite 风险。教训：taxonomy 可以诊断 why evidence failed，不能替代新证据或人类明确 scope 决策。

- [2026-06-30] **claim-bearing `.json` 必须通过严格 JSON parser，Python 能读不等于证据可移植。** V16 lift/scratch raw receipts 含未加引号的 `NaN`/`Infinity`，Python `json.load` 可读但 `node JSON.parse` 拒绝。教训：receipt repair 必须包含 strict parser check；若 raw 文件保留 Python-only 数值常量，必须生成 strict derivative（带 source hash 和显式 pathology/status 字段）作为 claim source。

- [2026-06-30] **active GPU run 脱离 screen 时，scientist 不能只按 STATE/A3 判断状态。** V16 scratch 的 screen 不在 `screen -ls` 中，但 1202c 仍有 wrapper/fex_dim_lift pids 活着，且远端已产生本地没有的 d100 seed0 JSON。教训：active-run handoff 至少要三方一致（pid/process cwd+cmdline、result/checkpoint mtime、log tail），否则容易把 running 误标 collected 或重复发射同一 seed。

- [2026-06-30] **扩样本后从 negative 变成 boundary 的 family，要改分类轴而不是调阈值。** quartic 从 V13 的 4/10 到 V15 combined 26/40，V16 又显示 7/7 d100 lift 但 scratch d50 1/3 HIT。它既不是原先的干净负类，也不是第二 hard target；正确处理是拆成 searchable / liftable / hard / numerical-pathology 轴。教训：当 family 同时支持和反驳不同子 claim 时，taxonomy 比二分标签更接近 truth。

- [2026-06-29] **larger-n calibration 可能推翻标签本身，不能用 post-hoc 阈值补救。** V15 C3 从 50 cells 扩到 200 cells 后，quartic 从 V13 的 4/10 变成 combined 26/40，紧贴 schrodinger 29/40；预设 Wilson CI 分离失败。正确科学动作不是把 27/40 point threshold 当新证据，而是承认 quartic 原 negative label 错/至少是边界，回到 claim 轴重测 lift、scratch hardness 和 baseline。教训：当扩样本改变 family 性质判断时，先修分类和机制解释，再谈阈值。

- [2026-06-28] **经验条件的 calibration 可以诚实报告 CI overlap，不必硬追 p<0.05。** V13 C3 calibration 50 cells (5 families × 10 seeds)，5/10 threshold 100% 分类正确，point estimates 清晰分离 (margin 0.30)，但 Wilson 95% CIs overlap (schrodinger [0.40,0.89] vs quartic [0.17,0.69])。auditor 标 BLOCKER 要求选择"accept n=10 uncertainty"或"加种子"。科学判断：C3 是经验条件不是定理，calibration 的目的是证明阈值不是 cherry-picked，不是做 formal hypothesis test。诚实报告 CI overlap 比追 p<0.05 在科学上更值得尊重——reviewer 自己会判断这对论文叙事的影响。教训：empirical calibration ≠ statistical test；不要因为 CIs overlap 就觉得 calibration 失败了。

- [2026-06-28] **planned must-fix 还在 running/needs_impl 时不能送审。** V13 reviewer 给 6.3/not ready 的核心原因不是 SR d=100 证据弱，而是 C3 held-out searches 仍 running、calibration report 仍 needs_impl。教训：reviewer 明确要求的 must-fix 必须等 evidence chain collected、§4/A3/A6 一致后再进入 `needs_reviewer`；否则新增强证据也会被状态不完整抵消。

- [2026-06-28] **baseline 的“未测试”不能写成“不 feasible”。** V12 reviewer 接受 SR random d=50 成功但慢 4.78x，同时指出 d=100 未测就写 "Pipeline only feasible" 是 absence-of-testing overclaim。教训：只要某 baseline 在较低维度已经成功，目标维度必须真实跑或给出明确 no-hit bound；否则只能写 "untested / pending"，不能把空白格当失败证据。

- [2026-06-28] **activation boundary 不能凭直觉补点，必须插值实测。** V10 已知 d=20 scratch 3/3、d=50 scratch 0/3 后，直觉上 d=30 可能接近 failure boundary；V11 实测 d=30 仍 3/3 HIT（rel_l2 1.45e-6/3.53e-6/1.55e-6）。教训：FEX searchability 的维度退化不是线性平滑的，activation boundary 必须用 d30/d40/d50 这类插值 run 收紧，不能只用两端点外推。

- [2026-06-28] **composite subsumption 的双向性必须在代码中显式处理**。`fex_dim_lift.py` L1215 的 COMPOSITE_MACROS 检查只覆盖 composite→simpler 方向（low-dim 选 composite，high-dim 选 simpler child），遗漏了 reverse 方向（low-dim 选 simpler，high-dim composite 因额外自由度略优）。结果：12/87 false reject 全来自 poisson_sumsq/radial_quartic 的 reverse 情况。修复：加一行 `reverse_subsumes` 即可消除全部 FR 且不引入 FA（Gate-2 的 rel_l2 < 1e-3 独立保障安全）。教训：数学上的包含关系（A ⊂ B）在代码中必须双向检测——优化器选 B 不等于 A 错了。

- [2026-06-28] **经验枚举到 semi-formal proposition 的提升是最后一英里**。V10 reviewer 穷举分析 14 combos 是"正确的"但"停在枚举层面"——JSON 枚举本身不是可引用结论。论文需要一个 proposition：明确假设、逐一排除、引用数据。区别在于读者能在一个 proposition 中看到完整逻辑链而不是扫 14 行 JSON。Agent 注释：排除论证的数学部分（bounded/overflow/concentration/structural）都是基本分析，耗时来自把 JSON 数据翻译成可引用格式。

- [2026-06-28] **reviewer 要求改 claim scope 时，scientist 不能触碰 §5，只能把 evidence-bound scope 写进 §4/§6/A1。** V9 reviewer 建议修改 §5 的 "Claim 永远不许降级" 带来治理冲突；正确响应是 reject 任何 §5 rewrite，把 Conservation primary case、FEX 窗口极窄、Two-Gate 命名、topology prior 和 composite subsumption 做成可审计 evidence artifacts。教训：scope 不是靠口头解释解决，而是靠 claim→source→scope ceiling 的机器可查表解决。

- [2026-06-29] **blind recovery 的 criteria 代码必须用 PDE 质量判 verdict，不能用 strict name match**。blind_summary.json 写 verdict=FAIL 但实际 36/36 accept/reject 正确、0 lift-quality-FA。根因：criteria 检查 selected_macro == expected_macro，但 composite_sqsumx2_x2 数学上包含 sum_x2（α₁≈0 时退化为 sum_x2）。数据文件的自动 verdict 与科学结论不一致时，reviewer 只看 verdict=FAIL 就会否定。教训：blind test 的 criteria 应是"正确分类 + PDE 质量"，macro 名匹配只是 diagnostic。推广：任何 automated verdict 代码都必须与论文的 claim 定义一致——代码判据和论文判据是同一个东西，不能有两套。

- [2026-06-29] **probe-only 和 amortized 速度不能混着报**。V1-V8 连续 8 轮写 "pipeline ~1s"、"700-1250× faster"——全是 probe-only 数字。v8_gate_cleanup 实测后 scout 2109s + probe 57s = 2166s total，摊销 ~530s/dim。在 d≤10 vs PySR 仅 1-2×。诚实化后核心优势从"速度"转为"d≥50 可靠性"——这是更强的 claim（speed advantage 可被优化缩小，reliability advantage 是结构性的）。教训：claim 的数字要在实验一开始就对齐 fair comparison 的定义，不能等 8 轮后被 audit 推翻。

- [2026-06-28] **reviewer 连续说"reframe 不算 fix"时，必须做一个改变核心证据基础的实验**。V4-V8 五轮同样两个结构性阻塞器（单 hard target + target-specific grammar），V5-V7 用穷举分析、narrative reframe、PySR 回应——全部是 reframe/lightweight fix，分数 5.8→5.4→5.7→5.5→5.2 无上升趋势。V8 reviewer 明确警告"再一次 reframe 将终止审查"。正确做法：blind recovery 用 systematic library 替换 target-specific macros——直接改变核心指标（0 FA）的证据性质（从"预装答案"到"systematic 枚举中的 selection"）。Agent 注释：V5 reviewer 首次提出 target-specific grammar 时就应该做 blind recovery，拖了 4 轮花了 $70+ GPU 和大量 reviewer/auditor 时间。

- [2026-06-27] **Gate 冗余性发现可以反转论文叙事**。Gate-2 被 Gate-1 完全覆盖（624 cells 完全一致）表面上削弱"three-gate design"，但科学上揭示了搜索稳定性蕴含 fit 质量。呈现为 discovery（"我们发现多 seed 一致性是最强的质量保证信号"）而非 design flaw，论文反而更可信。

- [2026-06-27] **coder 的科学解读不可信——scientist 必须从原始数据独立判断**。iter24 coder 称"Gate-3 过度保守、不防 FA"，实际 Gate-3 正确拒绝 45 cells（PDE rl2 1.2e-3~1.72e-2，阈值的 1.2×–17×）。如果 scientist 照搬 coder 解读移除 Gate-3，pipeline 将接受 PDE 误差达 1.72% 的 lift。与 iter20 "SECOND HARD TARGET CONFIRMED" 事件同类。教训：coder 的 "claim framing" 必须回到原始 JSON 验证，不能 pass-through。
- [2026-06-27] **gate/certificate 项目的 ablation 常揭示出意料之外的冗余**。Gate-2（低维 fit check）在 Gate-1+Gate-3 存在时边际贡献为零（full=no_gate2 在所有 624 cells 完全一致）。科学解释：搜索稳定性（Gate-1）蕴含 fit 质量——多 seed 同 macro 意味着 fit 已经好了。这是一个可发表的 finding——pipeline 的有效机制比预设的更精简。论文不必隐藏冗余，应呈现为 discovery。
- [2026-06-27] **no_search ambiguity 的三重来源是 pipeline 必要性的最强论证**。57/156 ambiguous 不仅来自浓度测度（21%），精确库 families 也占 42%（composite_sqsumx2_x2 与 sum_x2 结构相似在高维 PDE 拟合中不可区分）。这比 "浓度测度是唯一问题" 的叙事更强——即使 target 精确在库里，直接高维拟合也不可靠。
- [2026-06-27] **reviewer 说"单 hard target 不够 NeurIPS"时，穷举搜索+分析表比继续碰运气找第二个更有效**。V5-V9 花 $93 GPU 尝试 12+ hard target 候选全 negative。正确策略是用穷举分析（unary 集 × depth × d→∞ growth 行为）把"找不到第二个"从弱点变成"可行窗口极窄"的科学发现。同时用 gate ablation 证明 pipeline 的科学必要性——比找更多 hard target 更直接地回应 "target-specific macros" 质疑。Agent 注释：reviewer 在 MUST-001 中显式给出了 "claim 精确定界" 的 alternative path，应该更早识别到这是优选路线。
- [2026-06-27] **Gate-3 ablation 这类 proposal 承诺的实验必须在送审前执行，否则会被 reviewer 反复升级**。Proposal S1 承诺 Gate-3 ablation 但从 V1 到 V5 都未执行，V3/V5 reviewer 两次标为 must-fix。简单实验（~3-5 GPU-h）本可以在 V1 就做完，拖延导致审稿信誉受损。教训：proposal 中的实验承诺（ablation/baseline/control）应在第一轮就安排为 P0。
- [2026-06-27] **dispatcher 在 coder round 结束后不能直接切 needs_auditor**。coder 产出 claim-bearing 结果后必须先回到 scientist，由 scientist 整合 STATE/LESSONS/MANIFEST 并决定继续实验还是送审；auditor 只检查整合质量。若 auditor 发现 coder 结果未经 scientist 整合就进入 audit，应标为治理 BLOCKER，要求 dispatcher 回派 scientist，不得代 scientist 做科学整合。
- [2026-06-27] **coder 偏离 scientist plan 时，若产出有价值则接受但记录治理教训**。iter21 coder 将 A1 Task Group B 从 "写 SR feasibility analysis" 升级为 "跑 SR d=50 实验"（2000 trials, 42min, $0.46 GPU）。产出推翻了 scientist 的 "SR 高维 infeasible" 假设，得到了更强的实证证据（SR d=50 feasible but 2500× more expensive）。教训：(a) coder 应事先请示 scientist 而非自行升级 plan scope；(b) 但若结果有价值，scientist 应接受并更新叙事，而非机械执行原 plan。Agent 注释：此事件暴露了治理流程的灰色地带——plan 是 scientist 的科学判断，但 coder 可能有更准确的技术直觉。未来应在 A1 中显式写 "若发现 plan 假设有误，coder 应停止并请示而非自行改道"。
- [2026-06-27] **scientist 必须独立验证 coder 的 claim framing**。Coder 在 V7 声称 "SECOND HARD TARGET CONFIRMED"，scientist (iter20) 直接接受未做独立验证。iter20 auditor 指出 cos_anchor_sum 与 Conservation 数学同构（同一 1D 守恒律 PDE 的 sin/cos 变体）——这不是独立第二 hard target，而是同一 PDE family 的跨函数验证。教训：coder 产出数字 → scientist 必须独立评估科学意义，不能直接接受 coder 的叙事。Agent 注释：这是 iter21 最昂贵的教训——从 "two hard targets" 退到 "one hard target + cross-function validation"，论文叙事需要实质性修改。
- [2026-06-27] **更长的 RL 训练在某些 target 上有害**。R12 Conservation ep60 5/5 HIT → ep120 2/5 HIT（3/5 seeds 完整 policy collapse，finetuning 无法恢复）。但 cos target 在 ep120 下 4/5 HIT。结论：(a) ep 不是越多越好——某些 target 的 loss landscape 在长 RL 后对 finetuning 不友好；(b) 不要假设最佳 search config 跨 target 通用——每个 target 应扫 ep 范围。Agent 注释：这个发现对将来 FEX target scout 有直接影响——始终报告 multi-epoch 结果，不要只报告 best config。
- [2026-06-27] **audit 文件跨 branch 丢失是系统性 bug，不是偶发事件**。iter18 audit 在 `route/v5-second-hard-target` 创建，iter19 audit 在 `route/v6-hard-target-candidates` 创建——两者在后续 branch 中都缺失。根因：route branch 从 main 分出时，在这些 branch 上创建的 audit 不在 main 的祖先链上。修复需要治理规则：每次创建新 route branch 时显式复制所有 audit 文件；或将 audit 始终 commit 到 main（在 merge 前手动 `git checkout main -- audits/`）。Agent 注释：这已经是第二次发生（iter18+iter19），必须在下一条 route 创建前建立脚本/检查。
- [2026-06-27] **signed-sum PDE family 的 searchability 对 tree depth 极其敏感**。R12 Conservation depth1 0/5 vs depth2_sub 5/5；V6 5 candidate depth1 全 0/5；V7 cos depth2_sub 2/3。经验规则：signed-sum 结构 `f(linear_combo)` 需要 depth2_sub (unary(binary(linear,linear)))，depth1 (unary(linear)) 只能表达 `f(x_i)` 或 `f(c*x_i)` 但无法聚合所有变量。对于新的 signed-sum candidate，depth2_sub 是强制配置，不是优化选项。
- [2026-06-27] **FEX unary 算子集严重限制 hard target 空间**。当前 unary={sin, cos, exp, id, x², x³, x⁴, 0, 1}。去除 sin/cos（同一 PDE family）和 Σx²/d 型（浓度测度）后，候选空间仅余：(a) x³ of signed sum (cubic); (b) x⁴ of signed sum (quartic); (c) exp of signed sum (数值不稳定)。Agent 注释：这是 honest limitation，论文应明确写出算子集限制是找不到更多 hard target 的根本原因。

- [2026-06-27] V5 三候选全部失败的根因是选型错误：exp_gaussian_x2、cos_sumsq_x2、rational_1px2 都是 Σx²/d 型函数，高维时浓度测度（Σx²/d→const）使它们变得平凡。Conservation 用 signed sum Σxi（增长 O(d)）避免了浓度测度。设计 hard target candidate 的核心判据是 argument 是否随维度 growth——O(d) growth 的 signed sum 可以，O(1) concentration 的 squared sum 不行。Agent 注释：V6 候选池全部基于此教训。
- [2026-06-27] 浓度测度（concentration of measure）既可以是致命弱点，也可以是可发表的科学发现。V5 non-perturbative 两族 d=100 全 ACCEPT 表面上 gate 失去了区分力，但科学上揭示了 gate 的物理边界。把 limitation 写成"我们发现了 gate 的有效范围及其物理根源"而非"gate 失败了"，论文反而更可信。Agent 注释：对 gate/certificate 类项目，发现 limitation 的物理根源是加分项。
- [2026-06-27] **search configuration 必须匹配 target structure 的树复杂度**。R12 Conservation 证明 signed-sum 类需要 depth2_sub（unary(binary(linear, linear))）才能搜到 sin_anchor。V6 scout 沿用 depth1 导致 0/5 全 FAIL——depth1 只能搜 squared-sum 类（unary(leaf) 即 x²），对 signed-sum 结构性无能。通用规则：target PDE 的 skeleton 需要什么树深度来表达 → scout 必须用那个深度。Agent 注释：这是 V6 最昂贵的教训——4.53 GPU-h 浪费在一个已知无效的配置上。今后 scientist 在 A1 中必须显式标注"为什么这个深度足够表达目标结构"。
- [2026-06-27] **sinc(x)=sin(x)/x 在 FEX 默认算子集中不可搜**。FEX unary 集有 sin/cos 但无 sinc，binary 集有 +/*/- 但无除法 `/`。sinc 需要 sin 和除法的组合，控制器极难偶然发现。V6 sinc "HIT" 实质是 quadratic Taylor 近似——低维 rel_l2 蒙混过关但高维 lift 必然崩溃。教训：scout 前必须先检查目标函数是否在算子集的表达能力范围内。如果目标需要不在库中的算子，要么加算子，要么换 target。Agent 注释：这个问题在 V6 A1 设计时应该被发现——科学家应检查 FEX function.py 的 unary/binary 列表再选定候选。
- [2026-06-27] **git branch 切换会导致 audit 报告丢失**。iter18 audit 在 `route/v5-second-hard-target` 分支创建（commit c9926da），但 `route/v6-hard-target-candidates` 从 main 分支分出时该 commit 不在祖先链上，`audits/audit_iter18_20260627_0600.md` 在 V6 branch 上不存在。治理教训：audit 应在 merge 回 main 后、或显式跨 branch 复制。
- [2026-06-27] 当 audit→scientist→coder 治理链被外部事件（如 reviewer 返回）打断时，被跳过的 audit 必须在下一轮显式补回应。iter18 因 V4 reviewer 返回跳过了 iter16 audit response，被 iter18 auditor 升为 BLOCKER。治理纪律：无论什么外部事件，audit response 不能被跳过。
- [2026-06-26] 当 reviewer 说"所有 accept-path 全部 exact-library"时，先检查已有数据里是否有被低估的证据。R7 NB comp_pert_x4/comp_pert_pw 是真解不在 grammar 内的 150 probes，但 V3 只把它们叫"NB calibration"。重新组织为"非精确库 accept-path 证据"后直接回应 reviewer，不需要跑新实验。数据的叙事定位比数据本身更重要。
- [2026-06-26] 代码中有 active rejection path 但文档否认（tie_cv L1006-1008），即使该 path 从未触发也会被 reviewer 当成 code integrity 问题。gate 定义必须与代码实现完全一致，任何"informational-only"标签必须真的在代码里是 no-op。
- [2026-06-26] Evidence pack 不能只写人类友好的 `sub_keys` 标签；zero-context reviewer 会按路径直接查 JSON。以后 claim-to-source artifact 必须同时写 source path、actual JSON dot-path、value、rounding rule 和 existence check。
- [2026-06-26] Coder 收集完 claim-bearing 结果后，scientist 必须在同一轮把数字搬进 STATE §4/§6/A1，而不是只在 experiment-log 里留流水。连续 stale STATE 会让后续 agent 误读当前证据，且 auditor 会把它升级成 CRITICAL。
- [2026-06-26] 对 §5 真实目标，多个 target 的证据强度必须分层。R12 Conservation 是 clean primary（5/5 scout → d=100 lift → d≥50 scratch 0/6 → NN d=100 fail）；Schrodinger 是 strong lift but borderline hardness（d=50 scratch 1/3, d=100 0/0）。下一轮应先补边界 target，而不是把两个 target 混写成同强度结论。
- [2026-06-26] `data/MANIFEST.md` 也会变 stale。R12 baseline 已经 collected 27/30，但 MANIFEST 仍写 needs_sync/26 cells；scientist 读结果后要同步资产账本，否则后续 coder/auditor 会把旧资产状态当成当前事实。
- [2026-06-25] 对 §5 真实目标，连续两轮 low-d searchability 失败后必须停止同族续跑。R10+R11 product-sine 从 `{2,3,5}` 到 `{10,20,30}` 共 0/36 hits，说明问题是 target family 不适合作为当前 pipeline 的成功目标，而不是 omega 还没调好；下一步应换 family，并把 product-sine 作为 searchability failure 证据保留。
- [2026-06-25] 对“真实/文献失败 PDE”不要只挑最难目标。目标要先同时满足三门：低维 vanilla FEX 能搜到、同一 skeleton 能真实来源 lift、高维 scratch 有稳定性或成本缺口。没有第一门，后两门全是 analytic coverage 或 wasted compute。
- [2026-06-22] 对 §5 这类真实目标，必须先把 "low-d searchable" 作为独立硬门。R10 把 low-d scout 和 high-d scratch 放在同一个 18000s 脚本里，结果 Phase 1 0/9 失败并吞掉 Phase 2；以后 target scout、baseline failure、lift validation 要拆成独立 run 和独立 timeout。
- [2026-06-22] 对 grammar scaling / threshold report，NB reference families 不能混进 true expected-reject denominator。一个 JSON artifact 写 `pass=false` 就会污染后续接力，即使 raw facts 是 true-reject FA 0；报告脚本必须显式分 `expected_accept` / `expected_reject` / `nb_reference`。
- [2026-06-22] 对有限 macro grammar 的 scaling 质疑，不能只手动加一堆候选宏。更稳的是借 COSINE(2604.12806) 的 feedback-driven library expansion 思路：根据 residual / near-miss / coefficient importance 逐步加 2-3 个候选，并同步报告 expected-reject FA rate 和 accept recall。
- [2026-06-22] 当 reviewer 要求 external baseline 时，"官方代码存在"之后必须跑真实代码 anchor，不能继续用自设计 oracle 顶替。HD-TLGP 官方 repo 是脚本式 Poisson/Heat/Advection 入口，d≤3 anchor 足以证明我们认真比较；d>3 不要承诺。
- [2026-06-22] 对 near-boundary gate 校准，可借 PACE 思路把宽松阈值当候选发现工具，但最终 claim-bearing 阈值必须冻结。探索 rel-RMSE 0.05 可以帮助看边界，最终 accept 仍用 r3 冻结的 0.02 和 probe rel-L2 1e-3。
- [2026-06-21] 外部 baseline 缺失被 reviewer 升为 must-fix 时，deep-lit 论文里的 baseline 数字只能当实验设计锚点，不能替代本项目 matched-compute 可执行 baseline。SIGS(2502.01476) 提供 HD-TLGP/SSDE 低维 PDE 性能参考，但它未测 d>3；本项目仍必须自己跑 HD-TLGP-style 或 PSR-style baseline。
- [2026-06-20] 对 gate/certificate 项目，只报告"当前阈值 0 false accept"不够。必须报告 flip threshold：阈值放宽到多少会让 near-boundary control 翻成 false accept，并把 searchability failure 与 gate rejection 分开统计。
- [2026-06-19] 对 lift/certificate 类项目，`source="analytic_low_dim_skeleton"` 只能证明 macro library 覆盖了某个解析结构，不能证明真实 generator 搜到该结构后可 lift。STATE 和 proposal 里必须把 analytic injection 与 real generator output 分开标注。
- [2026-06-19] 成本 claim 不能从 short-budget smoke 推出。只要主 claim 包含 amortization 或 scalability，第一轮实验就要先跑 fair matched-budget from-scratch baseline，并把它设为 kill-switch。
- [2026-06-19] `manifest.json` 里的 `git_commit="unknown"` 会让整轮结果变成诊断信号，哪怕数字看起来很好。run wrapper 必须 fail-fast 检查 commit/command/server/GPU，而不是 silent fallback。
- [2026-06-19] speedup 必须分清 probe-only 和 pipeline-amortized。单个高维 probe 秒级不等于整体 pipeline 秒级；低维多 seed search 和失败 trial 都要进 break-even。
- [2026-06-19] exact-library family 的 d=100 近零误差主要证明"正确宏在库里且搜索找到了"，不能单独证明 audit 的近似能力。必须配 approximate-match/out-of-library falsifier。
