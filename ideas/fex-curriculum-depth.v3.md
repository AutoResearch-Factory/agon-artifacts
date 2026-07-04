---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-curriculum-depth/
---

- One-sentence summary: 把 FEX depth curriculum 收窄为 probe-gated module frontier: 浅层搜索只负责产生候选模块, held-out context probe 先检查它们能否在下一深度继续组合, 通过检查的 top-k 模块再作为软 macro prior 进入 controller; 若 probe 不优于 shallow reward, 论文改写成 FEX 可组合性边界诊断.
- Hypothesis: FEX 深树搜索的关键风险不是树深本身, 而是 shallow reward 会把可复用模块和短路近似混在一起. 对有组合结构的 PDE, 一个结构化的 transfer probe 应比 shallow reward 或 random ranking 更能预测模块在下一深度的收益; 对非组合 PDE, probe 应主动拒绝 locking, 避免把错误子表达式硬塞进后续 controller.
- Expected outcome: 成功信号是在 compositional PDE 上, probe-gated soft frontier 在 matched candidate/wall-clock 下比 flat FEX、原始 expanding tree、warm-start、greedy hard locking、random locking 少用 >=30% 候选评估达到同等 residual/L2, 且 transfer score 对 follow-up gain 的 rank correlation 明显高于 shallow reward. 最便宜反证是 transfer score 排序不优于 shallow reward 或 random ranking, 或 oracle module 在同一 probe 下也不能提升下一深度.
- Contribution type: method + diagnostic
- Contribution drift note: v2 contribution type 是 `method`; v3 保留 `method` 并新增 `diagnostic`, 因为这版把 cheap transfer probe 能否区分 reusable module 和 shallow shortcut 写成主问题. 没有删除 method/theory; topic 未声明 `preferred-contribution-types`, 因此不触发越界.
- Risk: HIGH
- Estimated effort:
  - Compute: 80-160 GPU-hours for controlled FEX sweeps and probe ablations; current pilots run on a single RTX 4060 Ti
  - Data: available; manufactured PDE collocation is generated on the fly
  - Implementation: 3-5 weeks
- Novelty quick-check: SPL (Sun et al., ICLR 2023) 有 MCTS module transplantation, 但没有 FEX/PDE residual/policy-gradient controller 中的 transfer probe; original FEX expanding tree 逐深搜索但各深度独立. VaSST soft symbolic trees 和 DSO/EGG-SR 覆盖 soft relaxation、RL symbolic search、equivalence pruning, 但不研究跨深度模块是否可迁移. 2026-06-17 的快速检索只再次找到 SymPlex/SSDE 等 PDE symbolic solvers, 没有发现同一 probe-gated FEX frontier.
- Strongest objection: v3 pilot 显示一个 naive tiny depth2 RL probe 比 shallow reward 更差; 如果更结构化的 held-out context probe 也失败, 这个 idea 不能作为加速方法成立.
- Why we should do this: v1/v2 已经证明 greedy curriculum 的正信号主要来自 shallow-first 和 undertrained flat baseline, 不是可靠的 grow-and-lock. v3 先检验 probe 是否可信; 正结果能支撑一个更稳的 FEX search 方法, 负结果也能说明 FEX 搜索空间里哪些组合结构不可被廉价利用.
- Review handling: 接受 reviewer 对 probe 定义、soft prior 注入和 k 选择不清楚的批评; v3 把 probe 写成核心机制, soft prior 只对通过 probe 的 macro candidates 加 logit bias, k 用 Occam-window/top-k 混合规则并允许 no-module fallback. 接受 evidence critique; pilot 新增 transfer-score rank correlation 和 oracle-module sanity check. 接受 outcome-realism critique; claim 限定到 compositional PDE, non-compositional case 只支持边界诊断. Pushback on "SPL/FEX already cover it": SPL 是 MCTS reward-driven transplantation, 原始 FEX 无跨深度迁移, 二者都没有 FEX/PDE 下的 transfer probe.
- Pilot:
  - Setup: 在 `radial_sine` d=2 上, 16-step depth1 pool 取 top 8 shallow candidates, 加 oracle `x^2+x^2` (`actions=[3,0,3]`), 每个 candidate 用相同随机种子做 3-step locked depth2 probe 和 12-step follow-up, 本机 RTX 4060 Ti.
  - Metric: positive 要求 Spearman(probe score, follow score) 至少比 Spearman(shallow score, follow score) 高 0.10, 且 probe 选出的 candidate follow score 不低于 shallow 选出的 candidate; oracle module 应在同一管线下有可见收益.
  - Result: 当前 tiny RL probe 为 NEGATIVE: shallow->follow Spearman = 0.667, probe->follow Spearman = 0.101; best follow score = 0.267, oracle `x^2+x^2` follow score = 0.258. v2 radial-sine 也只是 WEAK: curriculum/flat = 1.21, 但只比 depth1-only 高 1.6%.
  - Signal: NEGATIVE

- Claims and Claims matrix: POSITIVE: 在有浅层可组合结构的 PDE 类上, held-out context transfer score 比 shallow reward 更能预测后续 module gain, probe-gated soft frontier 在 matched budget 下超过 flat/expanding/warm-start/hard/random locking. NULL: 若 probe 不优于 shallow reward, 只能声称 FEX depth curriculum 需要先做 shallow sufficiency test, 不能声称跨深度迁移有效. NEGATIVE: 若 oracle modules 也不能在下一深度产生收益, 结论应改为 FEX 当前 controller/locking 接口不支持稳定模块迁移.
- Narrative: 这篇论文不把 curriculum 当作技巧来卖. 它问一个更可审计的问题: FEX 搜索里有没有能被廉价探测的可组合子表达式, 以及这种探测能否安全地控制后续搜索.
- Experiments: 主实验只做方法证据和机制诊断, 不构建 benchmark. 核心比较包括 flat FEX、original expanding tree、depth1-only、warm-start without macro prior、greedy hard locking、probe-gated soft frontier、random macro prior. 必须报告 candidate evaluations、wall-clock、best residual、final relative-L2、transfer-score/follow-gain rank correlation、oracle-module ablation、soft-prior temperature/escape ablation、compositional vs non-compositional PDE split.
- Assets status: Poisson、radial-sine 和 v3 transfer-probe pilots 已完成且当前 probe 为 negative; 数据与结果交接见 `workspace/fex-curriculum-depth/data/MANIFEST.md`.

<review date="2026-06-17">

## Novelty

- Score: 6/10
- Closest prior work: SPL (Sun et al., ICLR 2023) — MCTS-based module transplantation; VaSST (Roy et al., 2026) — soft symbolic trees via VI, no cross-depth transfer; SDSR (Lin et al., IEEE MIND 2025) — curriculum learning for multi-decoder SR, no transfer probe.
- Key differentiator: 在 FEX 的 RL policy-gradient controller + PDE residual 这一特定场景下, 用 held-out context transfer probe 区分 reusable module 与 shallow shortcut, 再做 probe-gated soft macro prior 注入. SPL 做 module transplantation 但基于 MCTS reward 而非 FEX 的 policy gradient controller; VaSST 做 soft tree 连续松弛但无跨深度模块迁移; SDSR 做表达式长度 curriculum 但无 transfer probe. Delta 真实但偏窄——"soft over hard locking"和"test before commit"是通用直觉, 贡献在 FEX/PDE 场景下具体的 probe-gated soft frontier 机制设计与配套的诊断框架.

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 6/10 | v3 相比 v2 在三个方面改善: probe 从模糊概念变成核心机制、soft prior 限定为 logit bias、k 有了 Occam-window 规则雏形. 剩余 gap: (a) "held-out context probe" 的操作化定义仍不完整——held-out 指什么 (独立 RL run? 独立模型? 独立 budget?)、probe 用什么 metric (partial residual? locked-tree completion gain?) 和多少 budget 算"便宜"; (b) probe 目前只有 "naive tiny depth2 RL probe" 的 NEGATIVE 结果, 如果"更结构化的 held-out context probe"也只是另一个 RL probe 变体, 那"更结构化"的论证力不够; (c) soft macro prior 注入后 controller 如何利用 logit bias 进行搜索 (bias 作用于所有深度还是仅下一层? 与 exploration 如何 trade off?) 未交代. |
| Missing evidence signals | 5/10 | v3 pilot 为系统 NEGATIVE: probe->follow Spearman 仅 0.101 (vs shallow->follow 0.667), oracle `x^2+x^2` follow score 0.258 与 best follow 0.267 基本持平. 最急需的证据: (a) 至少 2-3 种不同 probe 设计 (不只 RL probe, 也可考虑 analytical probe、residual-based probe、或基于 controller ensemble 的不确定性 probe) 在同一管线上的对比; (b) 在 compositional PDE 上 oracle module 是否有清晰正收益 (当前 tied); (c) matched wall-clock comparison 仍未实施. |
| Narrative | 7/10 | v3 叙事显著优于 v2: 从 "curriculum trick" 升级为 "FEX 是否具有可被廉价探测的组合结构". 诊断 fallback 诚实且合理. 剩余弱点: paper 同时在卖"加速方法"和"诊断工具"两个故事, 审稿人可能认为诊断是 post-hoc 退路而非主力贡献; 建议在 paper 中将诊断作为 primary framing、"若 probe 有效则同时得到加速"作为 secondary gain. |
| Venue contribution | 5/10 | topic 未声明 target-venue. 按 FEX 方法论文惯常投稿目标 (JMLR/ICML/NeurIPS) 推断. 当前 NEGATIVE pilot 使 positive method claim 无支撑. 纯诊断论文若只能说明 "FEX 的某一种 probe 不行", 不足以支撑顶会——需要将诊断泛化为更宽的 "RL symbolic tree search 的可组合性审计方法" 或在足够多 PDE/gate 设置上展示 pattern. |
| Testability | 9/10 | Expected outcome 有清晰最便宜反证: transfer score 排序不优于 shallow reward 或 random ranking——这个信号已经在 v3 pilot 上触发了. 若后续不同 probe 设计全部失败, 也很容易得出干净的 null conclusion. |
| Outcome realism | 5/10 | claims 已合理限定到 compositional PDE、>=30% 候选评估缩减 (比 v2 的 50% 更保守). 但当前 empirical 证据强烈指向 positive path 不太可能: naive probe 远差于 shallow reward, oracle module 无清晰收益. 若"更结构化的 probe"也不奏效, positive outcome 非常不可能. |
| Contribution type compliance | N/A | topic 0616-fex 未声明 `preferred-contribution-types`, 跳过. Idea 声明 types: method + diagnostic, 自身一致. |
| Overall Quality | 6/10 | v3 在 scientific clarity、claims discipline 和 testability 上比 v2 更进一步 (5→6). 但 empirical 信号反而恶化了——从 WEAK (v2 radial-sine curriculum 1.21x) 到 NEGATIVE (v3 probe Spearman 0.101). 目前是一个 well-posed falsification study, 不是一个有初步实证的方法. 若下一步能在不同 probe 设计和更多 PDE 上生产出正信号, Quality 可再上一个台阶. |

## Contribution Drift

- v_{n-1} contribution types: {method}
- v_n contribution types: {method, diagnostic}
- Status: expanded(+diagnostic)
- Hard cap triggered: no. Topic 0616-fex 未声明 `preferred-contribution-types`. Diagnostic 的加入在 idea body 中有明确理由 (Contribution drift note 说明"因为这版把 cheap transfer probe 能否区分 reusable module 和 shallow shortcut 写成主问题"), 接受. 未删除 method, 无 silent downgrade.

## Alternative Framing

当前 framing 已较锐 ("FEX 搜索里有没有能被廉价探测的可组合子表达式"), 但可再推一步: 把 paper 标题写成 **"A Held-Out Composability Test for RL Symbolic Tree Search"**, 主 claim 是"held-out probe 能否在 commit 之前区分可复用模块和短路近似". 加速只是下游结果, 诊断才是主贡献. 这样做的好处: (a) NULL/NEGATIVE 结果天然成立 (证明 probe 不行本身就是发现); (b) 审稿人不会因"加速仅为 30%"而挑战 novelty; (c) 如果有至少一个 probe 成功了, 那就是 method + diagnostic 双赢.

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 在 compositional PDE 类上, held-out context transfer score 的 rank correlation 显著高于 shallow reward, probe-gated soft macro prior 在 matched candidate/wall-clock 下比 flat FEX / expanding tree / warm-start / hard locking / random locking 少用 >=30% 候选评估达到同等 residual/L2. |
| NULL | 若 probe 在多个设计和 PDE 上均不优于 shallow reward 或 random ranking, claim 限定为 "FEX 在测试的 PDE 类上未展示可被廉价探测的跨深度模块迁移, depth curriculum 需先过 shallow-sufficiency gate, 不能默认 grow-and-lock 有效". |
| NEGATIVE | 若 oracle modules 在多 probe / 多 PDE 下也无法产生下一深度收益, claim 应为 "FEX 当前 controller/locking 接口不支持稳定模块迁移", 但不能声称 FEX search space 普遍缺乏组合结构 (可能只是接口/表示问题). |

## Likelihood-Impact Matrix

- Priority: Medium = Likelihood: Low x Impact: High
- Numeric score for ideas.xml: 5
- Rationale: **Likelihood: Low** — 顶会级成果依赖几个目前不成立的 key conditions: (a) transfer probe 必须显著优于 shallow reward (当前 Spearman 0.101 vs 0.667 反向); (b) oracle modules 必须在同一管线下有清晰的下游收益 (当前 tied); (c) 至少 2-3 个 compositional PDE 上需出现正信号. 如果"更结构化的 probe"不过是 RL probe 的变体, 其改善空间有限. 唯一的 chance 是发现一个结构上根本不同的 probe (如利用 PDE residual 结构、controller ensemble 不确定性、或 analytical transfer measure) 能捕捉当前 probe 完全错过的信号. **Impact: High** — 若成立, 直接为 FEX 和 RL-driven symbolic tree search 提供一个可操作的 composability litmus test, 改变"盲目 grow-and-lock"的搜索策略, 并开启 "SR search space compositionality" 这条研究线. 但范围限于 FEX/PDE/RL-tree-search, 不会改写整个 SR 领域的判断 (非 Exceptional). Claude 与 codex 对 Likelihood 和 Impact 判断一致, 无分歧.

## Overall

- Priority: Medium
- Score: 5
- Comments: v3 相比 v2 的 scientific clarity 提升是实质的——narrative 从 generic curriculum trick 收敛到 well-posed composability question, claims 边界更诚实, pilot 加入了 rank correlation 和 oracle sanity check. 但 empirical 证据恶化 (NEGATIVE pilot) 使得 positive path 的前景比 v2 更暗淡. 当前 score 持平 v2 (5), 因为 clarity gain 被 evidence deterioration 抵消. 建议下一步不继续 refine idea 文本, 而是优先在实验层面测试当前"更结构化的 held-out context probe"能否逆转 NEGATIVE 信号; 如果能, 回来更新 pilot 后 score 有望升到 7; 如果不能, 需要严肃考虑是否以 diagnostic-only paper 收尾或 pivot 到其他方向. v2 review 中 concern audit: probe 定义/soft prior/top-k → partially resolved (概念更清但未操作化); transfer rank correlation + oracle sanity → resolved as planned but currently negative; matched wall-clock → 未完成; narrative + cheap falsifier → resolved.

</review>
