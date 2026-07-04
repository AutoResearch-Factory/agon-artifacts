---
topic: topics/0616-fex.md
landscape: topics/0616-fex-landscape.md
workspace: workspace/fex-llm-skeleton/
---

- One-sentence summary: 用 LLM 生成完整的表达式树骨架 (而非仅算子集合) 作为 FEX RL controller 的 warm-start, 将 LLM 的科学先验与 FEX 的精确参数优化结合, 从根本上改变 FEX 的搜索范式.
- Hypothesis: Bhatnagar et al. (2503.09986) 已证明 LLM 能预测 PDE 解中的算子集合; 进一步让 LLM 生成完整骨架 (包括算子排列和树结构) 将比仅缩小算子集带来更大的搜索加速, 因为排列组合是搜索瓶颈的主要来源.
- Expected outcome: 成功: LLM 骨架 warm-start 在 FEX 标准 benchmark 上将搜索迭代次数减少 >70% (从 1000 次降到 <300 次) 且保持同等精度; 失败: LLM 生成的骨架在 FEX 的 BFGS 优化下频繁陷入局部极小, 说明骨架结构需要比算子集更高质量的预测.
- Contribution type: method
- Risk: MEDIUM
- Estimated effort:
  - Compute: 10 GPU-hours on A100 (LLM inference) + 10 GPU-hours (FEX search)
  - Data: available (使用 Bhatnagar et al. 的合成 PDE 数据集)
  - Implementation: 2-3 weeks
- Novelty quick-check: Bhatnagar et al. (2503.09986) 只做了算子集预测, 未生成骨架. LLM-SR (Shojaee et al. 2404.18400) 用 LLM 生成方程骨架但面向一般 SR 而非 PDE 求解, 且不与 RL 搜索整合. SR-GPT (Li et al. 2024) 用 GPT 引导 MCTS 但不是 FEX 的 policy gradient 框架. 本 idea 的差异化在于: (1) 面向 PDE 求解而非一般 SR, (2) 与 FEX 的 RL framework 深度整合而非替代.
- Strongest objection: LLM 可能记忆了常见 PDE 的解析解, 实验需要证明泛化到训练分布外的 PDE.
- Why we should do this: Yang 组已做了 LLM + FEX 的第一步 (算子集预测), 骨架生成是自然的下一步; 如果成功, 将 FEX 从"从零搜索"升级为"LLM 引导搜索", 对整个 symbolic PDE solving 社区有示范意义.
- Pilot:
  - Setup: 用 DeepSeek-chat 对 Poisson d=5 生成 top-5 表达式骨架 (postfix notation), 评估 LLM 生成的表达式质量
  - Metric: 若 LLM 生成的表达式经 BFGS 参数优化后 relative L2 error < 1e-3, 信号为正
  - Result: LLM 第一个候选 "x1 x^2 x2 x^2 x3 x^2 x4 x^2 x5 x^2 + + + + 2.5 *" 正确识别了 sum(x_i^2) 结构, 系数 2.5 (真值 0.5) 可通过 BFGS 修正. LLM 调用成本 435 tokens (约 $0.001). 参数优化实验仍在运行.
  - Signal: POSITIVE (LLM 能正确识别结构, 参数优化待完成)

<review date="2026-06-16">

## Novelty

- Score: 5/10
- Closest prior work: Bhatnagar et al. (2503.09986) — LLM operator set prediction for FEX; LLM-SR (Shojaee et al. 2404.18400) — LLM skeleton generation for general SR; SR-LLM (Guo et al. 2025, PNAS) — LLM+RAG 生成符号片段 + DRL 组装完整表达式树
- Key differentiator: 将 LLM 骨架生成与 FEX 的 policy-gradient RL controller 和 BFGS 参数优化做深度整合用于 PDE 求解, 而非像 LLM-SR 那样面向一般 SR 且不用 RL, 也非像 SR-LLM 那样自建 DRL 系统. 差异化的核心在于 (1) FEX 特有的二叉树表示和算子集, (2) warm-start 接入已有 RL controller 而非替代, (3) PDE 求解而非数据驱动的一般 SR. 但整体概念 "LLM 生成骨架 + 搜索" 已被 LLM-SR 和 SR-LLM 覆盖, 本 idea 主要是领域迁移和系统整合, 非干净的新方法机制. 若实验结果能揭示结构先验粒度 (算子集 vs 骨架 vs 完整表达式) 在 FEX 搜索中的相变规律, 则 novelty 可升至 7/10.

## Quality

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 5/10 | "排列组合是主要搜索瓶颈" 缺乏直接证据; "骨架比算子集带来更大加速" 是合理猜想但未经 pilot 验证; warm-start 如何输入 FEX policy-gradient controller 的具体接口未说明. |
| Missing evidence signals | 3/10 | 缺少: (1) oracle skeleton 上界实验, (2) operator-set-only warm-start 对照 (Bhatnagar et al.), (3) random/corrupted skeleton 对照, (4) 多 seed 稳定性评估, (5) OOD/防记忆 PDE 测试, (6) wall-clock 时间而非仅 iteration 计数, (7) top-k skeleton 聚合策略, (8) 与 LLM-SR/ICSR/SR-GPT 等竞争方法的比较. Pilot 仅测了一个 trivial PDE (Poisson d=5), 完全不构成证据链. |
| Narrative | 5/10 | "Bhatnagar 的下一步" 叙事很顺, 但也因此缺乏 surprise. 当前 framing 像工程延伸而非研究贡献. 若能改为 "FEX 搜索中结构先验粒度的作用规律", 故事会更有科学问题驱动感. |
| Venue contribution | 4/10 | 按 NeurIPS/ICML 标准, 仅在 FEX benchmark 上实现 70% iteration reduction 不足以构成顶会贡献. 方法创新 (LLM skeleton 用于 FEX) 是增量式改进, 需要极强实验洞察 (如先验粒度相图、失效模式分类学、反直觉发现) 或理论分析才能达到顶会门槛. 若目标为 JCP 或更专门的 venue, 评分可升至 6/10. |
| Testability | 6/10 | POSITIVE 信号 (70% iteration reduction) 具体可测; pilot 廉价 (435 tokens, ~$0.001). 但 NULL 信号定义过窄 — "BFGS 陷入局部极小" 只是可能失败模式之一; 缺少 "骨架不优于 operator-set only" 这一更关键的 falsifying signal. 且 Poisson 太简单, 极易被 LLM 记忆, 不能在 pilot 阶段当决定性证据. |
| Outcome realism | 5/10 | 70% iteration reduction 在简单 benchmark 上可能实现, 但保持同等精度、跨 PDE 泛化、扣除 LLM inference 成本和多候选探索 overhead 后能否成立存疑. FEX 搜索本身已被 Multi-Scale FEX 报告为不稳定, LLM 先验在分布偏移时可能反而引入 bias. 2-3 周实现可行; 10 GPU-hours LLM inference 偏乐观 (若需测试多样化 PDE 集). |
| Contribution type compliance | n.a. | topic 未声明 preferred-contribution-types, 跳过检查. |
| Overall Quality | 5/10 | 方向可做, 是 Yang 组的自然延伸, 但在当前粒度下更接近组内诊断/工程实验而非独立顶会投稿. 最大短板是 novelty 压缩 (SR-LLM 已做 LLM skeleton + RL) 和实验设计不足以产生意外发现. |

## Contribution Drift (n >= 2 only; n=1 写 N/A)

N/A

## Alternative Framing

当前 framing 是 "LLM skeleton warm-start FEX", 过于像 "apply X to Y". 更锋利的 framing: **"FEX 搜索中结构先验粒度的相图: 算子集合 vs 子树/骨架 vs 完整表达式, 在何种 PDE 家族上帮助或伤害搜索"**. 这个 framing 将贡献从弱方法改成机制诊断 — 无论结果如何 (skeleton 胜出、算子集已足够、骨架有害) 都有发表价值, 因为回答了一个结构性问题. 同时自然引入 anti-memorization/OOD 测试、oracle 上界、不同先验粒度的 ablation, 实验设计更系统.

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 在明确限定的 FEX PDE benchmark 和 anti-memorization/OOD 设置下, LLM 生成的完整骨架先验能比 operator-set-only prior 和 from-scratch FEX 更高效地达到同等误差; **不能声称** "根本改变了 symbolic PDE solving 的搜索范式" 或推广到所有 SR 方法. |
| NULL | 若 skeleton 不显著优于 operator-set-only prior, 只能声称 "在当前 FEX 框架和测试 PDE 上完整骨架先验未提供超越算子集先验的额外增益", **不能否定** LLM 科学先验的整体价值或 LLM+SR 的总路线. |
| NEGATIVE | 若 skeleton 频繁误导 BFGS/RL 导致性能退化, 可声称 "硬骨架先验对 FEX 的 policy gradient 搜索过于脆弱, FEX 更适合软 prior、ensemble prior 或可恢复的 controller bias 形式", **不能声称** LLM 不适合 PDE 求解. |

## Likelihood-Impact Matrix

- Priority: Medium (5) = Likelihood: Medium x Impact: Medium
- Numeric score for ideas.xml: 5
- Rationale: 
  - Likelihood (Medium): 有明确成稿路径 (LLM skeleton generation → FEX integration → controlled experiments), Yang 组有 FEX 基础设施和 Bhatnagar et al. 作为直接前驱, pilot 已初步验证 LLM 能生成正确骨架. 但依赖关键条件成立: (1) LLM skeletons 在 anti-memorization/OOD PDE 上仍正确, (2) skeleton 提供的额外结构信息确实比 operator-set-only 带来显著加速, (3) 实验结果足够系统以至于即便 null 也有发表价值. 若按本文当前 framing (纯方法), likelihood 降至 Low; 若采用 Alternative Framing 的相图设计, likelihood 升至 Medium.
  - Impact (Medium): 如果成功, 会对 FEX/符号 PDE 求解这条研究线产生清楚价值, 并可为 LLM+SR 社区提供关于 "先验粒度" 的实证规律. 但不太可能改变整个 SR 或 LLM-SR 领域的判断, 因为 FEX 本身是 niche 方法.
  - Claude 与 codex 对 Likelihood 存在分歧 (Claude: Medium, codex: Low, >=1 level), 分歧来源于对 "top-venue 可能性" 的评估差异: codex 认为当前 framing 下 novelty 被强烈压缩, 而 Claude 认为通过 reframing + 系统实验设计可以扭转. Impact 双方一致 (Medium). 综合后取 Medium/Medium → Medium (5).

## Overall

- Priority: Medium
- Score: 5
- Comments: 这是一个 Yang 组内值得快速试的工程/诊断方向, 不应包装成独立的顶会方法创新. 最大价值不是 "LLM 给 FEX 写骨架", 而是系统回答 "FEX 到底需要多强、多硬的结构先验" — 这个 framing 会将 idea 从弱方法贡献升级为有发表价值的机制诊断. 建议: (1) 采纳 Alternative Framing, 将实验设计为 multi-granularity prior comparison; (2) 在 landscape 中已补充 SR-LLM (Guo et al. 2025, PNAS), 该工作是当前最接近 "LLM skeleton + RL 搜索" 的工作, 需在 related work 中明确 differentiate; (3) pilot 需扩展到 anti-memorization test (如 LLM-SRBench 风格的合成 PDE) 才能作为 POSITIVE gateway. Claude 与 codex 在 Likelihood 上分歧 >=1 level (Claude: Medium, codex: Low).

</review>

<deep-lit-update date="2026-06-16">

## 新发现的相关文献 (deep-lit-tick 0616-fex, 2026-06-16)

以下文献在系统性调研中新发现, 与本 idea 直接相关:

### 最直接相关 (LLM + 进化/RL 驱动 SR)

1. **PiT-PO (2602.10576)**: Boxiao Wang, Jian Cheng (CASIA) 等, 2026.02. LLM+RL 微调做方程发现. 双约束 (物理有效性 + token 级惩罚). SOTA on SR benchmarks. 与本 idea 的关键差异: PiT-PO 直接用 RL 微调 LLM 生成完整方程, 而本 idea 是用 LLM 生成骨架再交 FEX controller 搜索. PiT-PO 证明 LLM+RL 可行, 但未针对 PDE 求解且不与 FEX 整合.

2. **FunctionEvolve (2606.07704)**: Zeyu Xia, Jun Zhu 等, 2026.06. Expression tree + LLM 引导进化, 82.9% SA@50 on LLM-SRBench. 核心: structure-visible search (结构摘要+局部树编辑+结构感知系数拟合). **高度相关**——expression tree representation 与本 idea 一致, 结构感知系数拟合直击 "Good Structure, Bad Score" 问题. 差异: 面向通用 SR 数据而非 PDE 求解, 且用进化而非 FEX RL.

3. **Deliberate Evolution (2606.04360)**: Xinyu Pang, Bo Han (清华) 等, 2026.06, ICML 2026. Agentic SR: 解耦符号生成与搜索控制, 自适应算子+诊断工具+反思记忆, 40% 采样预算超越 baselines. 关键启发: diagnostic tools (结构诊断+误差归因) 可帮 LLM skeleton 在 FEX 搜索中自适应调整.

4. **DrSR (2506.04282)**: Runxian Wang, Boxiao Wang (同 PiT-PO 组), 2025.06, 15 citations. LLM + dual reasoning (数据理解+反思学习) 闭环. 与本 idea 的互补: DrSR 的数据理解模块可帮 LLM skeleton 阶段更好地理解 PDE 数据特征.

### 竞争格局更新

5. **SR-LLM 的竞争压力加大**: FunctionEvolve (82.9% SA@50) 和 Deliberate Evolution (40% budget) 在 LLM-SRBench 上大幅推高了 SOTA, 说明 "LLM 生成表达式 + 搜索" 范式正在快速成熟. 本 idea 必须强调 **PDE 求解特有的物理约束 (BC/IC/方程形式) 和多尺度结构** 来差异化, 否则易被视为 LLM-SR 在 PDE 上的简单应用.

6. **SAGE-Fit (2605.23272)** 的启示: 同 PiT-PO 组, 证明 "Good Structure, Bad Score" 是 LLM skeleton 方法的关键瓶颈 — LLM 生成的正确骨架可能因非凸参数优化被错误低估. 本 idea 的 pilot 已观察到类似问题 (LLM 正确识别 sum(x_i^2) 但系数偏差). SAGE-Fit 的结构感知拟合可作为 FEX BFGS 阶段的改进插件.

### 对 idea 评估的影响

- **Novelty 压力增大**: FunctionEvolve 和 Deliberate Evolution 显著收窄了 "LLM 生成表达式骨架" 的 novelty 空间. 采用 reviewer 建议的 Alternative Framing ("结构先验粒度的相图") 变得更为紧迫.
- **实验基线必须更新**: 需加入 PiT-PO 和 FunctionEvolve 作为 baseline, 或至少详细 differentiate.
- **工具可用性提升**: EGG-SR e-graph 模块 (github.com/jiangnanhugo/egg-sr) 可直接用于 FEX 表达式的等价性检测, SAGE-Fit 提供即插即用的系数优化改进.

</deep-lit-update>

<deep-lit-update date="2026-06-16" scope="idea">
  
## Per-Idea Deep Lit 新发现 (11 篇)

以下论文由 `deep-lit-tick --scope idea 0616-fex --idea fex-llm-skeleton` 于 2026-06-16 精读后收录。

### 最直接相关：LLM 先验 + 搜索

1. **KeplerAgent (2602.12259)** [wiki](wiki/2602.12259.md): Jianwei Yang, Rose Yu (UCSD) 等, 2026.02. Physics-guided LLM agent for equation discovery. LLM 不直接猜方程，而是先调用 symmetry discovery 等物理工具提取结构，再据此配置 PySINDy/PySR 的函数库和结构约束。DiffEq benchmark 上 symbolic accuracy 达 75% (clean) / 45% (noisy)，远超 PySR 和 LLM-SR。**核心启示**: "LLM 提取先验 → SR 后端" 的范式已由 KeplerAgent 在物理方程发现上验证，fex-llm-skeleton 必须差异化：FEX 特有的二叉树 RL controller + PDE 求解（非参数估计）+ S_k 理论保证。可借鉴其 workspace/experience log 机制和 symmetry→structure 的思路。

2. **SR-Scientist (2510.11661)** [wiki](wiki/2510.11661.md): Shijie Xia, Pengfei Liu 等, 2025.10. LLM agent 用 data_analyzer 和 equation_evaluator 工具做长程方程发现，experience buffer 保存历史最优方程做功能性 warm-start，GRPO 自训练。LSR-Synth 上超越 baseline 6-35%。**核心启示**: experience buffer 是另一种形式的 warm-start——与我们的 "LLM skeleton 注入 FEX controller" 互补。GRPO 自训练机制可考虑用于 FEX controller 的 self-play。

3. **STRIDE (2605.17790)** [wiki](wiki/2605.17790.md): Jiarui Su, Songjun Tu 等, 2026.05. 多角色 agent 框架：data-aware generation + AST 混合参数拟合 + critic-executor 局部修复 + TF-IDF 语义记忆。直击 "Good Structure, Bad Score" 问题——可靠拟合反馈和结构多样记忆显著提升 ID/OOD 方程恢复。**核心启示**: critic-executor 修复机制可直接借鉴到 LLM skeleton 验证阶段——当 FEX BFGS 给 skeleton 低分时，先让 critic 诊断是骨架错误还是参数优化失败，再决定修复还是丢弃。

4. **LLM-ODE (2603.20910)** [wiki](wiki/2603.20910.md): Amirmohammad Ziaei Bideh, Jonathan Gryak, 2026.03. LLM 从精英候选提取模式，生成新表达式骨架，BFGS 拟合，Pareto front 选择。91 个动力系统上整体优于 PySR 和 ODEFormer。**核心启示**: LLM 引导进化搜索的具体实现，但面向 ODE 系统辨识而非 PDE 求解，且用 GP 而非 RL。

### 结构修复与编辑

5. **EditSR (2606.07915)** [wiki](wiki/2606.07915.md): Da Li 等, 2026.06. 两层框架：NeSymReS 一次生成表达式 skeleton + 预训练 Tagger-Editor Rectifier 做语法受限 subtree 编辑。把修补路径构造成 supervised state-transition chains，推理时少量 edit steps 即可修复。**核心启示**: 预训练语法约束编辑器的思路可直接用于 LLM skeleton 后处理——FEX 搜索前先过 Rectifier 修复语法或结构错误。

### Yang 组相关

6. **Domain-Aware Symbolic Priors (2503.09592)** [wiki](wiki/2503.09592.md): Sikai Huang, **Haizhao Yang** 等, 2025.03. **Yang 组工作！** 从 arXiv 公式语料提取领域符号先验分布（root/depth/width/operator 共现），以 KL regularization、hard mask 和结构块字典注入 tree-RNN 表达式生成。**核心启示**: (1) Yang 组已在探索 "结构先验引导表达式生成"，fex-llm-skeleton 是这条线的自然延伸；(2) 该文的 prior 是从语料统计的，我们的是 LLM 生成的——"统计 prior vs 语义 prior" 本身就是值得消融的维度；(3) 该文使用 tree-RNN 而非 LLM，说明 Yang 组内部可能对 LLM 在该场景的适用性持观望态度。

7. **Multi-Scale FEX (2510.22497)** [wiki](wiki/2510.22497.md): Gareth Hardwick, **Haizhao Yang**, 2025.10. FEX 扩展到高频振荡 PDE + 复杂孔洞域 + 特征值问题。符号谱合成 + 可学习频率缩放 + Rayleigh quotient 初始化。**明确讨论**频率分离导致的 RL 动力学不稳定性——这是 LLM skeleton 可直接针对解决的失效模式：LLM 先验可在频率分离场景下提供比随机初始化更稳定的骨架，缓解 RL 训练不稳定。

### 搜索效率与理论

8. **Improved MCTS for SR (2509.15929)** [wiki](wiki/2509.15929.md): Zhengyao Huang 等, 2025.09. Extreme bandit (UCB-extreme 关注 best-expression 而非 mean) + evolution-inspired state-jumping (mutation/crossover)。消融显示 state-jumping 是真正主贡献。**核心启示**: FEX RL controller 可借鉴 extreme bandit 策略——当前 FEX 的 policy gradient 优化的是 mean reward，但在 SR 中一个极高分的表达式比 100 个中等分更有价值。

9. **Generalization Bounds of SR with GP (2604.17402)** [wiki](wiki/2604.17402.md): M. Nomura, Isao Ono 等, 2026.04. 表达式树泛化界：误差 = 结构选择项 (log|T|) + 常数拟合项 (RG√(s/m))。**核心启示**: LLM skeleton 提供了更小的 |T|（结构复杂度降低），相当于减小了结构选择项的搜索空间——这给出了 "warm-start 为什么应该 work" 的理论直觉。

### LLM 微调做 SR

10. **SymbArena / Symbolic-R1 (2508.09897)** [wiki](wiki/2508.09897.md): Yingfan Hua 等, 2025.08. 148K 方程、1.83B tokens 微调 Qwen2.5-7B-Instruct，LoRA + Form-GRPO + HER 多轮推理。首个 LLM 在 R² 和 form consistency 上超越传统数值 SR 方法。**核心启示**: (1) 微调后的 LLM 做 SR 可超越传统方法，说明 LLM 确实有超越记忆的 SR 能力；(2) SymbArena 数据集可用于 fex-llm-skeleton 的 LLM 领域微调。

### 物理先验融合

11. **StruSR (2510.06635)** [wiki](wiki/2510.06635.md): Yunpeng Gong 等, 2025.10. PINN 提取局部 Taylor 系数 → mask attribution 评估子树贡献 → 引导 GP mutation/crossover。PINN 物理先验 + SR 的融合范式，不同于 LLM 先验但目标类似。**核心启示**: 物理先验可以通过多种来源注入 SR——LLM (语义)、PINN (导数)、语料统计 (频率)——不同先验源在不同 PDE 家族上的相对效力本身就是一个值得研究的问题。

### 对 idea 的影响（更新）

- **Novelty 收窄但仍有空间**: KeplerAgent 和 STRIDE 展示了 "LLM agent + SR" 的快速成熟，但它们都面向**数据驱动方程发现**（给定观测数据恢复方程），而非 FEX 的 **PDE 求解**（给定 PDE 和 BC/IC 找解的闭式表达式）。这是关键的差异化锚点。
- **Alternative Framing 更紧迫**: 按原 "apply LLM skeleton to FEX" framing，novelty 已被 KeplerAgent 严重挤压。按 "结构先验粒度的相图" framing，所有 SR/agent 工作都只是不同粒度的 prior，fex-llm-skeleton 成为唯一系统比较这些粒度的研究。
- **实验必须覆盖的 ablation axes 更新**: (a) 统计 prior (2503.09592) vs 语义 prior (LLM skeleton) vs 物理 prior (KeplerAgent style); (b) skeleton 粒度: operator-set-only vs subtree-pattern vs full skeleton vs oracle; (c) skeleton 注入方式: hard initialization vs soft controller bias vs grammar constraint; (d) OOD/anti-memorization test (LLM-SRBench 风格 PDE 变体).
- **新工具可用**: EditSR 的预训练语法约束编辑器、EGG-SR 的 e-graph 等价性剪枝、SAGE-Fit 的结构感知系数拟合均可作为 fex-llm-skeleton 的组件或 baseline。

</deep-lit-update>
