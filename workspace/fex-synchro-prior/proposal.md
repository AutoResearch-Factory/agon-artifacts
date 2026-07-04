<!-- 书写报告使用中文 -->
---
idea: fex-synchro-prior
title: "Frequency-Search Bottleneck Audit for RL Symbolic PDE Solvers, with Checked Spectral Priors as Search-Space Compilers"
version: 2
date: 2026-06-19
workspace: workspace/fex-synchro-prior/
---

## Problem Anchor (carried verbatim)

- **Bottom-line problem**: Multi-Scale FEX (2510.22497) 用 RL controller 在离散频率基 `{sin(3x)..sin(24x), cos(3x)..cos(24x)}` 上搜索, 再用输入层 `alpha_i` 连续微调精确频率。作者在 Conclusion 明确承认: 对 widely separated frequency components (如 `sin(10x1)sin(20x2)sin(30x3)`), "frequency separation introduces instability in the RL dynamics"。
- **Must-solve bottleneck**: 这种不稳定到底是不是**频率搜索本身**造成的? 还是被 expression-tree 结构搜索、`alpha`/系数的 Adam+BFGS 连续优化、reward 噪声、credit assignment 混淆? 没有任何工作把"频率"这一个轴隔离出来做因果诊断。
- **Success condition**: 用户能说"yes" 当且仅当: 在 d<=3、频率可观测、well-separated 的振荡 PDE 上, **真实 Multi-Scale FEX** 的 oracle-soft 频率先验比 standard FEX 用 <50% candidate evaluations 或 net wall-clock 达到同等 relative L2; 且 estimated-soft 先验在 overhead 计入、RHS decoy / nonlinear-mismatch / 非均匀采样控制下不退化为 blind FFT。
- **Constraints**: 本机 RTX 4060 Ti proxy 已完成; Gate 0 真实 FEX 预算 100-160 GPU-h; 无外部 data/权重; manufactured oscillatory PDE。

## Technical Gap

Multi-Scale FEX 已把多尺度周期算子放进 grammar, 但它没有回答频率失败的归因问题。真实 controller 的频率选择不是 v5 proxy 的 576 个 `(m,n)` dense pair, 而是每个相关节点从 `sin/cos(3x..24x)` 等 base periodic operator 中采样, 再由 `alpha_i` 连续缩放频率。这个 escape hatch 可能让"选错 base frequency"仍被连续优化救回来, 也可能因为 reward 噪声和 BFGS 早期拟合差而完全救不回来。v2 把这个动作空间错配写进核心诊断: Gate 0 必须同时测 oracle prior 是否加速搜索, 以及 `alpha_i` 是否已经足以消除频率动作瓶颈。

naive fix 仍然不够。扩大频率字典会增加 RL 动作空间; 连续 PINN 频谱先验修的是神经优化而不是离散 expression-tree 采样; blind RHS-FFT 在 manufactured RHS 上可能读到答案。v5 pilot 已构造 decoy RHS, naive FFT top-1 选中假峰 `(11,3)`、漏掉真 `(4,17)`、relative L2=1.0。因此最小缺失机制不是"再加一个频谱模块", 而是一个带泄漏控制的因果审计接口: 先用 oracle frequency prior 隔离频率动作轴, 再只在 oracle 通过后尝试 checked spectral prior。

2026-06-19 的增量检索没有发现直接 "spectral estimation -> soft frequency-action prior -> RL symbolic PDE/FEX search" 撞车。最近邻仍是 Multi-Scale FEX (2510.22497)、LLM+FEX operator-set pruning (2503.09986)、FEX+TranNet candidate-pool 扩展 (2604.22208)、SSDE/NetGP/StruSR 等 RL/GP PDE symbolic solvers, 以及 PiPRL (2506.22365) 这种跨域 "symbolic program prior -> RL" 先例。它们没有做频率动作轴的 oracle audit, 也没有把频谱估计编译成 FEX controller 的 soft frequency PMF。

## Method Thesis

- **One-sentence thesis**: 把频率视为 FEX controller 中可干预、可审计的动作轴: 用 oracle-soft prior 先回答"频率搜索是不是主瓶颈", 只在答案为 yes 时把通过验证的频谱估计编译成 width-capped soft frequency-action prior。
- **Why this is the smallest adequate intervention**: 不改 reward、不改 BFGS、不改 expression grammar、不训练新网络。新增机制只有一个 logits 偏置 `log p_prior(k)` 和一个固定预算的 validation gate。
- **Why this route is timely in the foundation-model era**: 这个 proposal 不需要额外 LLM。它复用 LLM+FEX 的高层经验: 外部预测可以缩小 FEX search space。但 LLM+FEX 做的是 hard 逐样本 operator-set pruning, 在 Poisson/Conservation Law 上报告 4.4x-6.0x iteration/time speedup; 它没有做 soft PMF 注入, 也没有处理 frequency-set。v2 将 soft 注入重新夺回为本方法的机制差异, 而不是把它误写成 prior art。

## Contribution Focus

- **Dominant contribution**: 频率动作轴的因果瓶颈审计协议。oracle-hard / oracle-soft / init-only / standard 的真实 Multi-Scale FEX 对照决定"频率搜索是主瓶颈"这一命题是否成立。
- **Optional supporting contribution**: Gate 0 通过后, checked spectral prior 作为 search-space compiler: 频谱估计 -> 固定预算验证 -> bounded soft PMF -> FEX controller。
- **Explicit non-contributions**: 不做新 PDE benchmark; 不提出新 FFT/NCPSD/SST 算法; 不训练 frequency predictor; 不把 LLM 加进 pipeline; 不声称 blind RHS-FFT 安全; 不声称一般 PDE 求解鲁棒性。

## Proposed Method

### Complexity Budget

- **Frozen / reused backbone**: Multi-Scale FEX 的 risk-seeking PG controller、periodic operator set、`alpha_i` 输入缩放、coarse Adam+BFGS、candidate pool、fine tuning 全部冻结。实现前须先复现 Multi-Scale FEX 的 manufactured sine benchmark 到同量级误差, 否则不进入 Gate 0。
- **New trainable components**: 零新增可训练参数。`p_prior(k)` 由 oracle 或现成估计器给出, 只作为 controller 频率节点 logits 偏置。
- **Tempting additions intentionally not used**: 不学一个 neural frequency predictor; 不引入 e-graph/EGG 剪枝; 不做 LLM skeleton; 不把 dense grid 当成主方法。dense-grid 与 true-skeleton-capacity 只作 conditional appendix sanity check。

### System Overview

```mermaid
graph LR
    A["Sampled RHS or early residual"] --> B["Estimator selector"]
    B --> C["Fixed-budget validation gate"]
    C --> D["Width-capped soft frequency prior"]
    D --> E["FEX frequency-node logits"]
    E --> F["Frozen PG search plus Adam/BFGS"]
    F --> G["Expression and relative L2"]
    C -. "fails" .-> H["Uniform frequency prior"]
    H --> E
```

### Core Mechanism

- **Input / output**: 输入是 PDE RHS samples 或早期 residual field; 输出是 base-frequency operator 上的离散分布 `p_prior(k)`。oracle prior 直接由 ground-truth frequency tuple 构造; estimated prior 由 FFT/NCPSD/early-residual DFT/CWT-SST 候选构造。
- **Architecture or policy**: controller 不变。对频率节点的 logits 做 `z_k <- z_k + lambda_prior log(p_prior(k)+eps)`。hard prior 是 diagnostic extreme, soft prior 是主候选, init-only 只改变 `alpha_i` 初值不改变采样 PMF。
- **Training signal / loss**: 无新 loss。reward 仍为原 FEX `S(e)=(1+L(e))^{-1}`。
- **Candidate-validation budget**: validation 不是自由搜索。每个 estimator 每个 PDE instance 最多评估
  `B_val = min(32, ceil(0.10 * |K_dense|))`
  个 frequency tuple, 只做轻量 constant/coarse fit, 不允许完整 controller rollout。`B_val` 全部计入 candidate-evaluation 和 net wall-clock。若需要超过该预算才能过 gate, 结果记为 DEGENERATE 或 NULL-A, 不能写 method success。
- **Why this is the main novelty**: novelty 不在频谱估计本身, 而在三个绑定约束: oracle audit 先于方法 claim; soft PMF 注入只作用于频率动作; validation 和 prior width 都有硬预算, 防止退化为 brute-force dense grid。

### Optional Supporting Component

- **Only include if truly necessary**: estimator selector 只在 Gate 0 oracle-soft 明确优于 standard 后激活。
- **Input / output**: FFT 与 NCPSD 是第一批 estimator; early-residual DFT 在 RHS decoy 或 nonlinear mismatch 时触发; CWT-SST 只在局部振荡或非均匀采样导致前两者 width >25% 时触发。
- **Training signal / loss**: 无训练。selector 根据 validation loss、prior width 和 decoy/mismatch gates 选择或拒绝 estimator。
- **Why it does not create contribution sprawl**: selector 不是新算法, 只是防止 blind FFT 泄漏和过宽 prior 的安全阀。主论文仍由 oracle audit 决定。

### Modern Primitive Usage

- **Which primitive is used**: RL primitive 是 FEX 的 risk-seeking policy-gradient controller。
- **Exact role**: 被诊断和被注入先验的对象, 不是新加入的组件。
- **Why not an LLM / learned predictor**: LLM+FEX 已证明 hard operator-set pruning 可加速 search-space selection, 但本问题有可观测频谱信号。经典 FFT/NCPSD/SST 更便宜、更可审计, 且能直接做 decoy leakage test。只有当经典 estimator 在 Gate 2 系统失败时, 才有理由另立 learned predictor 作为后续工作。

### Integration into Base Generator / Downstream Pipeline

v2 在 FEX controller 采样前插入一个 prior adapter。adapter 只读取频率候选, 不读取 exact solution, 不修改 tree grammar, 不修改 operator reward, 不修改 BFGS。推理顺序为: 估计或 oracle 构造 `p_prior` -> fixed-budget validation -> width cap -> logits bias -> frozen controller 采样 -> 原 FEX coarse/fine tuning。

Gate 0 必须记录真实 controller 的 frequency-action cardinality: base periodic operator 数、sin/cos family、维度选择、以及实际搜索中每个频率节点的 entropy。还必须加 `alpha` audit: free-alpha vs freeze-alpha 对照, 报告选错 base frequency 后 `alpha_i` 能补偿多少。这个 audit 决定 v5 proxy 是否高估了真实频率瓶颈。

### Training Plan

1. **Pre-gate implementation calibration**: 基于 FEX-PG 复刻 Multi-Scale input layer: per-dim `alpha_i`, `sin/cos(3x..24x)`, 加法/乘法 binary choice, risk-seeking PG, coarse Adam+BFGS。先在 Multi-Scale FEX paper 的 sine manufactured benchmark 上达到同量级 relative L2 (目标 `<=1e-5`, 或复刻具体 domain 时在 paper 数字的 10x 内)。不过关则停止, 不跑 oracle 对照。
2. **Gate 0 oracle audit**: 主表四臂: standard / oracle-hard / oracle-soft / init-only, 3-5 seeds, equal candidate budget 和 equal wall-clock 两条预算线。dense-grid 与 true-skeleton-capacity 只在结果模糊时进 appendix。
3. **Gate 1 injection ablation**: 比较 logits soft bias、hard pruning、reward bonus、init-only。预期 soft bias 在轻微频率误差下比 hard pruning 稳。
4. **Gate 2 estimator selector**: 先 FFT+NCPSD; 若 decoy/nonlinear mismatch 暴露 RHS leakage, 加 early-residual DFT; 若局部/非均匀采样失败, 再加 CWT-SST。
5. **Gate 3 robustness**: RHS decoy、人为 forcing 峰、nonlinear RHS/solution spectrum mismatch、非均匀采样、频率扰动。每个结果同时报 estimator overhead、validation cost 和 selected prior width。

### Failure Modes and Diagnostics

- **NULL-B: 频率不是主瓶颈**: oracle-soft 不优于 standard。诊断: first-true-frequency-hit 已改善但 final L2 不变, 说明 bottleneck 在 coefficient coupling/reward/credit assignment; first-hit 也不改善, 说明真实 action space 已足够小或 `alpha` 已补偿。
- **Proxy overstates bottleneck**: v5 proxy 用 576-pair dense grid, 真实 FEX 是较小 base-frequency operator + free `alpha_i`。诊断: base action cardinality log + freeze/free-alpha 对照。
- **LEAKAGE-FAIL**: blind RHS FFT 只在 manufactured RHS 暴露解频率时有效。decoy 峰失败和 nonlinear spectrum mismatch 分开报告, 不能互相替代。
- **DEGENERATE prior**: selected prior width >25% dense grid 时不能声称 search-space reduction; >35% 直接判 DEGENERATE。
- **Validation becomes brute force**: `B_val` 超过 `min(32, 10% dense)` 或用完整 controller rollout 做 validation 时, run 标为 invalid。
- **Implementation-invalid**: 自实现 Multi-Scale FEX 不能复现 paper benchmark, 则不允许解释 standard vs oracle 的差距。

### Novelty and Elegance Argument

最接近的 prior 边界如下。Multi-Scale FEX 加了 spectral operator set 和 `alpha_i`, 但没有隔离 frequency action 是否是不稳定根因。LLM+FEX 把 LLM 预测的 operator-set hard prune 成小搜索空间, 报告 4.4x-6.0x 加速; 它不是 soft PMF prior, 也不处理 frequency-set。FEX+TranNet 扩 candidate pool, 并显示 FEX 对候选池质量敏感; 它不提供频率估计或因果 oracle gate。SSDE、NetGP、StruSR、spatio-temporal reward PDE-SR 都是 PDE symbolic search 竞争线, 但不做 oscillatory frequency-action prior。MSPINN/PRISMA/FRES/cross-attention spectral-bias 方法修连续 neural solver, 不缩小 RL expression-tree 的离散频率动作空间。

因此 v2 的故事保持单一: 一个参数为零、单注入点、先诊断后修复的 frequency-action audit。若 Gate 0 失败, method claim 自动关闭; 这比堆一个更大的 FEX 系统更像可被顶会审稿人信任的机制论文。

## Claim-Driven Validation Sketch

### Claim 1: 频率动作搜索是 Multi-Scale FEX wide-frequency failure 的可测量瓶颈

- **Minimal experiment**: Pre-gate calibration 后, 在 d<=3 well-separated oscillatory PDE 上跑真实 Multi-Scale FEX 四臂: standard / oracle-hard / oracle-soft / init-only。3-5 seeds, equal candidate 和 equal wall-clock。
- **Baselines / ablations**: init-only 隔离 "只给 `alpha` 初值"; freeze-alpha vs free-alpha 隔离连续频率补偿; dense-grid/true-skeleton-capacity 仅作 appendix ambiguity check。
- **Metric**: candidate-evaluations-to-target relative L2, net wall-clock-to-target, frequency-action entropy, first true-frequency hit, final relative L2。
- **Expected evidence**: POSITIVE = oracle-soft 用 <50% candidate eval 或 net wall-clock 达同等 L2, 且 first-hit/entropy 同向改善。NULL-B = oracle-soft 近似 standard, 或 free-alpha 已消除 base-frequency miss。

### Claim 2: Checked spectra can be compiled into a bounded soft frequency prior without leaking the answer

- **Minimal experiment**: Gate 0 通过后, 在同一任务上比较 oracle-soft、FFT-soft、NCPSD-soft、early-residual-soft、conditional CWT-SST-soft, 并带 RHS decoy 与 nonlinear mismatch controls。
- **Baselines / ablations**: blind RHS-FFT 是 unsafe baseline; hard pruning 和 reward bonus 是 injection ablation; learned frequency predictor 不进主表, 只作为 "classical estimator failed" 后续路线。
- **Metric**: net eval/wall-clock after estimator overhead and `B_val`, selected prior width, decoy/mismatch pass rate, relative L2。
- **Expected evidence**: estimated-soft 保留 oracle gain 的主要部分, selected width <=25%, validation cost <= `B_val`, decoy 和 nonlinear mismatch 不失败。若 oracle 有效但 estimator 弱, verdict 是 NULL-A, 不是 method success。

## Paper Outline

- **Section 1**: Multi-Scale FEX 的 wide-frequency instability and the missing causal attribution.
- **Section 2**: Frequency-action audit: action cardinality, `alpha` compensation, oracle-hard/oracle-soft/init-only gates.
- **Section 3**: Checked spectral prior: fixed-budget validation, width cap, soft logits injection, leakage gates.
- **Section 4**: Experiments: calibration pre-gate, Gate 0 main table, Gate 1 injection ablation, Gate 2/3 estimator and robustness.
- **Key figures**: Fig 1 = true-controller Gate 0 four-arm curves with entropy/first-hit; Fig 2 = `alpha` compensation audit; Fig 3 = estimator gain vs prior width with decoy/nonlinear rows.

## Compute and Timeline Estimate

- **Estimated GPU-hours**: 100-160 GPU-h for claim-bearing experiments after a runnable Multi-Scale FEX exists. Reimplementation calibration has a separate stop budget: 5-10 engineering days and up to 30 GPU-h; if it cannot reproduce the paper benchmark by then, stop rather than run invalid oracle comparisons.
- **Data / annotation cost**: 无外部数据或模型权重。使用 manufactured oscillatory Poisson/Helmholtz-style PDE, RHS/residual samples, 和 paper benchmark reproduction cases。
- **Timeline**: 3-5 weeks realistic calendar: 1-2 weeks implementation/calibration, 2-3 GPU days for Gate 0, 1 week for estimator/robustness gates, remaining time for audit plots and write-up.

## Data / Asset Handoff Status

- `workspace/fex-synchro-prior/data/MANIFEST.md` 确认无外部 dataset/model, 无 active 或 interrupted download。
- 已有本地资产: `scripts/pilot_spectral_prior.py`, `run_pilot.sh`, `results/pilot_summary.json`, real-FEX-family smoke scripts/results, `NOTES.md`。
- 当前证据边界: v5 CuPy/PG proxy 和 Helmholtz smoke 是 POSITIVE_BOUNDED, 但不是 Multi-Scale FEX Gate 0。v2 不把这些 pilot 当作 method success, 只当作实现前的 cheap signal 和 leakage warning。
