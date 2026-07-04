---
topic: topics/0616-fex.md
created: 2026-06-16
---

# FEX (Finite Expression Method) 及 RL 驱动符号回归的研究景观

## 1. FEX 方法族概述

Finite Expression Method (FEX) 由 Liang & Yang (2022, JMLR 2025) 提出, 核心思想是在有限个解析算子组成的函数空间 S_k 中搜索 PDE 的近似解. 搜索通过二叉树表示数学表达式, 每个节点为一元或二元算子, 用 RL (policy gradient) 训练 controller 生成高分算子序列, 再用 Adam + BFGS 两步优化参数. 理论上证明了 S_k 对 Holder 函数无维度灾难 (k = O(d^2 (log d + log 1/eps)^2)). 实验中在 Poisson、线性守恒律、非线性 Schrodinger 方程上达到机器精度 (约 1e-7), 远超 NN solver (1e-4 到 1e-2). 后续扩展包括:

- **FEX-PG** (Hardwick, Liang & Yang 2024, JCP 2025): 引入参数分组减少高维函数的系数数量, Taylor 近似加速积分项, 将 FEX 扩展到偏积分微分方程 (PIDE).
- **Multi-Scale FEX** (Hardwick & Yang 2025): 加入符号谱合成模块处理多尺度振荡, 重新设计线性输入层扩展表达力, 扩展到特征值问题. 但在 Conclusion 中明确指出 "widely separated frequency components pose particular difficulties, as such frequency separation introduces instability in the RL dynamics".
- **FEX for complex networks** (Song, Wang & Yang 2024): 将 FEX 应用于复杂网络动力学, 提出随机化快速算法把计算复杂度从 O(N^2) 降到 O(N).
- **FEX for committor problems** (Song, Cameron & Yang 2023): 用 FEX 计算高维 committor 函数, 能正确识别解的代数结构从而降维.
- **FEX for physical laws from data** (Jiang, Wang & Yang 2023): 将 FEX 从求解 PDE 扩展到从数据中发现物理定律.
- **H-FEX** (Lai, Liang & Wang 2025): 把 FEX 与 Hamiltonian 结构结合, 保证符号发现的 Hamiltonian 结构.
- **FEX for stochastic dynamics** (Liang, Wang & Xu 2025): 将 FEX 扩展到未知随机动力学的辨识.
- **FEX with TranNet** (Huynh, Bao, Yang et al. 2026): 用 Transformer 网络学习函数映射辅助 FEX.
- **FEX for turbulence** (Xu, Qi & Wang 2026): 将 FEX 用于湍流动力学并恢复高阶矩.
- **LLM + FEX** (Bhatnagar, Liang, Patel & Yang 2025): 用 LLM 预测 PDE 解中涉及的算子集合, 缩窄 FEX 搜索空间. 这是 Yang 组将 LLM 与 FEX 结合的第一步.

## 2. 竞争方法景观

符号回归 (SR) 领域近年经历了方法学的快速迭代, 主要分为以下几个流派:

**遗传编程 (GP) 及其改进**: PySR (Cranmer 2023) 仍是最广泛使用的 SR 工具, 在 SRBench++ (Franca et al. 2024, TEVC) 中表现领先. eggp (Franca & Kronberger 2025) 引入 equality graph 避免重复评估等价表达式. GP 的优势在于搜索多样性和成熟的工程实现, 劣势在于缺乏可学习的先验和收敛保证.

**MCTS 及 RL 搜索方法**: SPL (Sun et al. 2022) 首先将 MCTS 用于 SR. DGSR-MCTS 和 TPSR (Shojaee et al. 2023) 加入 policy network 引导搜索. SR-GPT (Li et al. 2024) 用 GPT 引导 MCTS 并相互优化. Huang et al. (2025) 提出改进的极端 bandit 策略和演化启发的状态跳跃. NRSR (Sun et al. 2025, AAAI) 引入噪声鲁棒的门控模块. EGRL-SR (Sun et al. 2026) 提出目标条件 RL 与经验回放. Sym-Q (Tian et al. 2025, Nature Communications) 用离线 RL 实现交互式 SR. 这些方法与 FEX 最接近, 但都面向一般 SR 而非 PDE 求解.

**预训练 Transformer 模型**: NeSymReS (Biggio et al. 2021, ICML) 首个大规模预训练 SR Transformer. SymbolicGPT (Valipour et al. 2021). End-to-end SR with Transformers (Kamienny et al. 2022). SNIP (Meidani et al. 2023, NeurIPS). **PROSE 系列 (Schaeffer 组, UCLA)**: PROSE (Liu, Zhang & Schaeffer 2023, Neural Networks 2024) [wiki](wiki/2309.16816.md) 用 Polish notation 编码 ODE 方程，双 Transformer encoder + hierarchical attention 同时预测算子和恢复符号方程; PROSE-PDE (Sun, Liu, Zhang & Schaeffer 2024) [wiki](wiki/2404.12355.md) 扩展到 1D PDE，**Skeleton 模式仅提供方程骨架（不知系数）即达到 1.06% 误差**; PROSE-FD (Liu et al. 2024) [wiki](wiki/2409.09811.md) 扩展到 2D 流体力学; PI-MFM (Zhu, Sun, Zhang & Schaeffer 2025) [wiki](wiki/2512.23056.md) 注入物理约束实现 zero-shot physics fine-tuning; BCAT (Liu, Sun & Schaeffer 2025) [wiki](wiki/2501.18972.md) 用 block causal transformer 做 next frame prediction. 详见 §20. MDLformer (ICLR 2025). 这些模型推理速度快但泛化能力有限, 对训练分布外的问题表现下降.

**LLM 驱动的 SR**: LLM-SR (Shojaee et al. 2024, 97 citations) 利用 LLM 科学先验生成方程骨架. ICSR (Merler et al. 2024, 35 citations) 用 LLM 迭代精化函数形式. DrSR (Wang et al. 2025) 结合数据驱动洞察与反思学习. LLM-Feynman (Song et al. 2025) 结合 MCTS. IGSR (Saveliev et al. 2026) 引入影响力引导. PiT-PO (Wang et al. 2026) 用 RL 微调 LLM. Symbolic-R1 (Hua et al. 2025) 直接微调 LLM 做 SR. LLM-SRBench (Shojaee et al. 2025, 47 citations) 提供防记忆的 benchmark. SR-LLM (Guo et al. 2025, PNAS) 用 LLM+RAG 生成符号片段作为 skeleton, 再用深度 RL (risk-seeking + SAC) 组装成完整表达式树, 是当前最接近 "LLM skeleton + RL 搜索" 的工作但面向一般 SR 而非 FEX/PDE 求解. [idea-reviewer, 2026-06-16]

**扩散模型方法**: Symbolic-Diffusion (Tymkow et al. 2025) 用 D3PM 离散扩散模型同时生成所有 token, 而非自回归生成.

## 3. 理论景观

- **SR 的 NP-hard 性**: Virgolin (2022, TMLR) 首次证明 SR 是 NP-hard. Song et al. (2024) 用 symbol graph 给出替代证明.
- **NN 优化中的维度灾难**: Na & Yang (2025) 证明浅层 NN 用梯度流训练时, population risk 的衰减率不快于 t^{-4r/(d-2r)}, 即优化计算本身存在维度灾难.
- **FEX 的理论保证**: 原始 FEX 论文证明 S_k 对 Holder 函数类无维度灾难 (approximation theory), 但 RL controller 在什么函数类上能以多项式步数收敛到 eps-最优 这个问题仍然开放.
- **缺失**: 没有任何论文证明 FEX 的搜索算法 (RL controller) 的收敛率或采样复杂度. 这是 Yang 学术脉络中最大的理论缺口.

## 4. 结构性缺口与机会

1. **FEX 搜索的计算复杂度理论完全缺失**. FEX 有 approximation theory (S_k 无 CoD), Na & Yang 有 optimization lower bound (NN 有 CoD), 但 FEX 自身的 RL 搜索是否能逃脱 CoD 没有任何结果. 正面 (poly 收敛) 或反面 (hardness reduction) 都是重大贡献.

2. **FEX 与 LLM 的结合刚刚起步**. Bhatnagar et al. (2025) 只做了算子集预测, 没有用 LLM 生成完整表达式骨架或引导 RL 搜索. LLM-SR (Shojaee et al.) 在一般 SR 上已展示 LLM 先验的强大, 但没人把这个范式移植到 PDE 求解场景.

3. **FEX 的搜索效率瓶颈未被系统诊断**. 原始论文观察到搜索时间随树深指数增长, Multi-Scale FEX 发现频率分离导致 RL 不稳定, 但没有系统的失效模式分类学.

4. **FEX 与预训练模型的整合空白**. 大量 SR 工作使用预训练模型 (NeSymReS, PROSE, SymbolicGPT) 做 warm-start 或骨架预测, 但 FEX 仍然从零开始搜索. 用预训练模型为 FEX 提供 controller 初始化或表达式先验是显然的方向. **2026-06-24 update**: Schaeffer PROSE-PDE 的 Skeleton 模式已证明仅提供方程骨架（不含系数）即可大幅改善 Transformer 算子学习 (1.06% vs 无骨架时退化), PI-MFM 进一步展示 zero-shot physics fine-tuning 仅用 PDE 残差即可适配到未见 PDE. 这两项工作为 "预训练模型 → FEX" 整合提供了最直接的实验范本和理论背书 (MNO scaling law 证明多算子共享表征不增加总体代价).

5. **FEX 在 PDE 之外的应用仍然有限**. FEX 被应用到网络动力学、committor 问题、流行病学等, 但在材料科学、化学反应动力学、金融衍生品定价等高维问题中的应用仍然空白.

6. **Curriculum / 分层搜索策略缺失**. SR 领域已有多项工作探索分层和课程学习搜索策略, 但 FEX 仍使用单一固定深度的树搜索, 没有从简到繁的搜索策略.

7. **对噪声和不完整数据的鲁棒性**. NRSR (Sun et al. 2025) 专门解决高噪声数据下的 SR, 而 FEX 的所有实验都在干净数据上进行. PDE 求解场景中数据可以通过采样生成因此噪声可控, 但从实验数据发现物理定律时噪声是核心挑战.

8. **EGG (equality graph) 等效类剪枝未被引入 FEX**. EGG-SR (Jiang et al. 2025) 证明了等价表达式剪枝可以收紧 MCTS 的 regret bound 和降低 DRL 梯度估计方差, 但 FEX 的 controller 没有利用符号等价性.

9. **Synchrosqueezed / butterfly 先验与 FEX 的缝合**. Yang 自己的博士论文工具 (SynLab, butterfly factorization) 提供频率估计, 可以作为 FEX 输入层的冻结先验, 解决 Multi-Scale FEX 明确承认的频率分离失效模式, 但至今没有论文做这件事.

## 5. Benchmark 与评测

SRBench++ (Franca et al. 2024) 是当前标准 benchmark, 主要面向一般 SR 任务. LLM-SRBench (Shojaee et al. 2025) 专门设计防 LLM 记忆. SRSD (Matsubara et al. 2022) 面向科学发现. 但没有专门面向 PDE 求解场景的 SR benchmark -- FEX 使用的是 PDE 求解的标准 benchmark (Poisson, conservation law, Schrodinger), 而非 SR benchmark. 这两个社区的评测体系仍然割裂.

**HD-TLGP** (Cao, Liu, Wang, Xu, Ye, Tan, Jiang 2024, AAAI): 用遗传编程 (DEAP) + 自动微分做高维 PDE 可解释符号求解. 核心创新是结构迁移机制: 将 1D PDE 解析解通过变量替换 (x1→x2,...,xd) + 子树提取 + 硬编码乘法/加法扩展生成 Transferred Knowledge Base, 引导 GP 搜索. 在 Heat/Poisson/Advection 三类 PDE 上测到 d=3 (标题称"高维"), MSE 优于 PINN/PR-GPSR 但在 Poisson 上不如 FEM. 关键局限: 需要已知 1D 解析解; 扩展算子硬编码乘法 (适合分离变量型) 或加法; 仅到 d=3; GP 仍做完整 100 代搜索. 与 FEX 的维度骨架提升有表面相似但本质不同 (硬编码 vs 自动发现, d=3 vs d=100, GP 全搜索 vs 纯参数优化). [idea-reviewer, 2026-06-16]

## 6. Deep Lit 2026-06-16 新收录 (13 篇)

以下论文由 `deep-lit-tick --scope topic 0616-fex` 于 2026-06-16 系统性搜索并精读后收录。

### 6.1 直接竞品：RL/Transformer 驱动的符号 PDE 求解

- **SymPlex (2602.03816)** [wiki](wiki/2602.03816.md): Yesom Park, Stanley Osher (UCLA) 等, 2026.02. RL + Structure-Aware Transformer (SymFormer) 做符号 PDE 求解. 核心: tree-relative self-attention + grammar-constrained autoregressive decoding. 直接竞争 FEX——同样搜索表达式树, 但用 Transformer 学习策略替代 FEX 的 policy gradient. 报告 exact recovery of non-smooth and parametric PDE solutions. 与 FEX 关键差异: FEX 有 S_k 理论保证+Adam/BFGS 两步优化; SymPlex 无理论但 Transformer 更灵活. **撞车风险: 高**——若 idea 涉及 NN/Transformer/RL 组件需仔细对照. **该引**.

- **SSDE (2405.14620)** [wiki](wiki/2405.14620.md): Shu Wei, Lina Yu (CAS 半导体所) 等, 2024.05, ICML 2025. RL-based symbolic closed-form solver for ODE/PDE. 将 FEX 定位为 baseline 并声称大幅超越 (SSDE L_PHY ~1e-5 vs FEX L2 ~1e-1). 采用 RL + 符号表达式空间搜索, 与 FEX 方法学高度重叠. 局限性: 实验设置偏向 ODE; 无高维 PIDE 实验; 算子集灵活性不如 FEX. **撞车风险: 高**.

- **StruSR (2510.06635)**: Yunpeng Gong 等, 2025.10. GP-based SR + PINN Taylor guidance for PDE. 用 PINN 提取局部物理先验 (Taylor 展开) 引导 GP 进化. 9 citations. 与 FEX 方法论正交 (GP vs RL), 但 PINN+SR 融合范式正在逼近 FEX 性能.

### 6.2 LLM + 进化 / RL 驱动的符号回归

- **PiT-PO (2602.10576)** [wiki](wiki/2602.10576.md): Boxiao Wang, Jian Cheng (CASIA) 等, 2026.02. LLM+RL 微调做方程发现. 双约束: 物理有效性 + token 级惩罚抑制冗余. SOTA on SR benchmarks, 发现新湍流模型. 同组后续工作: **DrSR (2506.04282)** (15 citations, 双推理闭环), **SAGE-Fit (2605.23272)** (改善 SR 内层非凸参数优化). **撞车风险: 中**——面向一般 SR 而非 PDE 求解, 但 LLM+RL 骨架与 FEX 的 LLM 方向重叠.

- **FunctionEvolve (2606.07704)** [wiki](wiki/2606.07704.md): Zeyu Xia, Jun Zhu 等, 2026.06. 表达式树 + LLM 引导进化, 82.9% SA@50 on LLM-SRBench (4.5× baseline). 关键创新: structure-visible search (结构摘要+局部树编辑+结构感知系数拟合). **撞车风险: 中**——expression tree 表示与 FEX 一致, 但面向数据驱动 SR 而非 PDE 求解.

- **Deliberate Evolution (2606.04360)** [wiki](wiki/2606.04360.md): Xinyu Pang, Bo Han, Changshui Zhang (清华) 等, 2026.06, ICML 2026. Agentic reasoning for sample-efficient SR. 解耦符号生成与搜索控制, 自适应算子+诊断工具+反思记忆. 用 40% 采样预算超越 baselines. 消融显示诊断工具贡献最大. OOD 泛化差距惊人 (Chemistry: DE 1e1 vs baselines 最高 5e6). **撞车风险: 中**.

- **IGSR (2605.29184)** [wiki](wiki/2605.29184.md): Evgeny Saveliev, Mihaela van der Schaar (Cambridge) 等, 2026.05, ICML 2026. MCTS + LLM 项生成 + influence scores for pruning. 突破: 在真实生物学数据上发现 DNA methylation-RNA Polymerase II pausing 新关系 (湿实验验证). 局限: 受限于 f(x)=Σw_j ψ_j(x) 线性组合约束. **撞车风险: 中**——若做 LLM+MCTS+SR 则重叠严重.

- **DoLQ (2605.07323)** [wiki](wiki/2605.07323.md): Sumie Song 等, 2026.05, ICML 2026. Multi-agent LLM 引导 ODE 发现: Sampler Agent + Parameter Optimizer + Scientist Agent. 不涉及 RL. **撞车风险: 低**.

### 6.3 等价性剪枝与搜索效率

- **EGG-SR (2511.05849)** [wiki](wiki/2511.05849.md): Nan Jiang, Yexiang Xue (Purdue) 等, 2025.11, ICLR 2026. 将 e-graph 嵌入 MCTS/DRL/LLM 三类 SR 方法. 核心: Rao-Blackwellized gradient estimator + 等价类 backpropagation. **开源代码** github.com/jiangnanhugo/egg-sr. 与 landscape gap #8 高度对应. **撞车风险: 低**——EGG 模块可作 FEX 插件而非竞品.

### 6.4 生成式潜空间 & 符号蒸馏

- **GENSR (2602.20557)** [wiki](wiki/2602.20557.md): Qian Li, Yuntian Chen (东方理工) 等, 2026.02. CVAE + CMA-ES: 将离散 SR 搜索映射到连续潜空间. CVAE 训练使潜空间具有符号连续性和局部数值平滑性. Bayesian 视角: maximize p(Equ|Num) via ELBO. **撞车风险: 低**——搜索范式与 FEX 正交, 但为 landscape gap #4 (预训练+FEX) 提供范本.

- **SymTorch (2602.21307)** [wiki](wiki/2602.21307.md): E.S. Tan, Miles Cranmer (Cambridge) 等, 2026.02. PySR-powered symbolic distillation of neural networks. 从 PINN distill 出 PDE/ODE 精确闭式解 (含常数). 也做 SLIME (symbolic LIME) 和 transformer MLP 层的符号代理 (2-19% throughput 提升). **撞车风险: 低**——工具层重叠, 可作 FEX warm-start.

### 6.5 Neuro-Symbolic ODE 发现

- **Latent Grammar Flow (2604.16232)** [wiki](wiki/2604.16232.md): Karin Yu, Georgios Kissas (ETH) 等, 2026.04. Grammar-based 离散潜空间 + FSQ 嵌入 + behavioural Wasserstein loss + discrete flow matching. 支持多 predictor 引导采样. 局限: 仅 1D 合成 ODE. **撞车风险: 低**——方法论上游, 与 FEX 应用场景不重叠.

### 6.6 补遗: B0/B7 发现但未精读的相关论文

- **DISCOVER (2210.02181)**: Mengge Du 等, 2022. RL-based PDE symbolic discovery. 较早的 RL+PDE 工作.
- **EvoSR-LLM** (2026): 进化符号回归 + LLM 引导. 1 citation. arxiv ID 待查.
- **LLM-Meta-SR (2505.18602)**: Hengzhe Zhang 等, 2025. LLM 自动设计进化 SR 的选择算子. 2 citations.
- **Symbolic-Diffusion (2510.07570)**: Zachary Bastiani 等, 2025. D3PM 离散扩散做 SR.
- **Iterated Agent for SR (2510.08317)**: 2025. 迭代 agent 做 SR.
- **Engineering-Oriented SR (2603.19241)**: 2026. LLM as physics agents for constitutive law discovery.
- **Neuro-Symbolic AI for Analytical DE Solutions (2502.01476)**: 2025. 综述性质.
- **ERBench (2606.09276)**: 2026.06. Equation Recovery Benchmark, 强调 OOD 鲁棒性测试.
- **SRBench v2 (2505.03977)**: 2025.05. 扩展版 SR benchmark, 13 citations.
- **NetGP (IEEE CEC 2025)**: GP + Deep RL hybrid for PDE solutions.
- **Teaching the Teacher (2507.22767)**: 2025.07. GP-based symbolic distillation, teacher-student smoothness alignment.

## 7. 更新后的撞车风险矩阵

| FEX Claim / Gap | 2026-06-16 状态 | 关键竞品 |
|----------------|----------------|---------|
| S_k 理论保证 (无 CoD) | ✅ 唯一 | 无 |
| RL+表达式树搜索 PDE 解 | ⚠️ 双线竞争 | SymPlex, SSDE |
| 高维 PIDE (d=100) | ✅ 唯一 | 无 |
| LLM+FEX 骨架生成 | 🟢 完全空白 | PiT-PO/FunctionEvolve 在通用 SR 上探索但未移植 |
| Egg 剪枝 + FEX | 🟢 完全空白 | EGG-SR 提供可直接移植的模块 |
| 频率分离失效 | 🟢 完全空白 | 无 |
| 预训练模型+FEX warm-start | 🟢 完全空白 | GENSR/SymTorch 提供范式参考 |
| RL 搜索收敛率理论 | 🟢 完全空白 | 无 (FEX 最大理论缺口) |

## 8. Search-complexity addenda [idea-refiner, 2026-06-16]

### VSR-DPG (2402.00254)

Problem: high-dimensional symbolic regression has a large expression search space, and earlier vertical symbolic regression methods are hard to connect to neural policy learning. Method: models vertical SR as sequential grammar-rule decisions and trains a deep policy-gradient generator that builds equations along a reduced-to-full variable path. Results: reports stronger recovery of algebraic equations and ODE expressions than deep RL baselines and prior VSR variants. Relevance: this is a close example of policy-gradient expression-tree search, but it gives empirical acceleration rather than a convergence or sample-complexity theorem.

### CADSR (2406.06751)

Problem: risk-seeking deep symbolic regression can suffer from tail barriers where low-probability good expressions give weak or zero policy-gradient signal. Method: uses a decoder-only generator, frequency-domain attention, a BIC-style complexity-aware reward, and a ranking-based weighted policy update to stabilize search. Results: reports improved robustness and expression recovery across SR benchmarks, with code released. Relevance: it is useful background for reward-shaping and policy-update failure modes in expression search, but it does not address PDE residual search or prove FEX-style search complexity.

### VaSST (2602.23561)

Problem: symbolic regression search is discrete, multimodal, and usually lacks uncertainty quantification. Method: relaxes symbolic expression trees into soft symbolic trees and performs variational inference over operator and feature assignments. Results: reports improved structural recovery and predictive accuracy on simulated tasks and the Feynman Symbolic Regression Database within SRBench. Relevance: it is a recent continuous-relaxation alternative to discrete tree search, so it should be considered when arguing which part of expression search is intrinsically combinatorial.

### When Is Symbolic Regression Tractable? (ICML 2026 Poster, arxiv ID pending) [idea-reviewer, 2026-06-16]

Authors: Adil Soubki, Miles Cranmer. Problem: SR is known to be NP-hard, yet SR software routinely finds good models without exhaustive search. Method: studies SR through parameterized complexity theory, analyzing tractability relative to expression depth, tree size, and number of primitives/variables. Results: (1) SR is FPT when parameterized by expression depth or tree size, explaining why bounded-complexity search succeeds; (2) SR is W[1]-hard when parameterized by number of variables or primitives, identifying selection as the source of intractability; (3) lower bounds under ETH, approximation hardness, and polynomial kernel exclusion. Relevance: directly engages the theoretical complexity of expression-tree search but via parameterized complexity (W-hierarchy/FPT), NOT via RL policy-gradient convergence or PDE residual search. Preempts part of any "SR search space counting" claim but leaves open the FEX-specific question of RL controller step/sample complexity and the ∃R-hardness of real-coefficient feasibility.

## 9. Dim-lift addenda [idea-reviewer, 2026-06-16]

### NMIPS (2602.11630)

Problem: PDE families share identical mathematical structure but differ in parameters (e.g., diffusion coefficient, advection speed). Traditional symbolic regression solves each instance independently. Method: proposes Neuro-assisted Multitasking Symbolic PDE Solver (NMIPS) — uses gene expression programming (C-ADF chromosome) with multifactorial optimization (MFO) to simultaneously solve multiple PDE instances in a family. Devises an affine transfer method to align populations from different PDE tasks, transferring learned symbolic structure across parameter variations. Results: tested on 1D Advection, 1D Burgers', 1D Advection-Diffusion, 2D Advection, 2D Navier-Stokes, 3D Advection. Reports up to ~35.7% MSE improvement over baselines (SP-GPSR, DSR, GNOT). Key limitation: transfers across parameter instances within the SAME dimension only — does not address cross-dimension lifting. Relevance: NMIPS validates the broader paradigm of symbolic skeleton transfer in PDE solving but operates orthogonally to our idea (same-dimension parameter transfer via GP+MFO vs. cross-dimension skeleton lifting via FEX RL + macro inference + parameter-only optimization). Should be cited as related work demonstrating that structured transfer in symbolic PDE solving is effective.

## 10. Curriculum-depth addenda [idea-refiner, 2026-06-16]

### FePySR (2605.12704)

Problem: complex symbolic regression often needs reusable nonlinear feature modules before the final equation becomes searchable. Method: trains a heterogeneous feature-mapping network to extract candidate nonlinear features, ranks/simplifies them with SymPy, then feeds the enriched feature set to PySR. Results: reports 97.75% recovery on Nguyen, 85.78% recovery on 36 complex generated equations versus 20.24% for PySR and 36.11% for DSO, and 24/100 exact recoveries on a biological ODE case where PySR recovers none. Relevance: close in spirit to search-space reduction through modules, but it is two-stage neural feature extraction plus GP over observational SR data, not depth-staged FEX policy-gradient search over PDE residuals.

## 11. Adaptive-activation / continuous-relaxation addenda [idea-refiner, 2026-06-16]

### MetaSymNet (2311.07326)

Problem: standard neural networks fit data but do not expose concise symbolic formulas, and fixed architectures can leave redundant structure. Method: builds a tree-like symbolic network whose architecture can expand or contract and whose PANGU meta activation evolves into basic functions during training before the network is simplified into an expression. Results: reports comparisons against four symbolic regression baselines on more than 10 public datasets with 222 formulas, with stronger fitting, extrapolation, and lower structure complexity than MLP under matched fit. Relevance: this is the closest adaptive-activation symbolic-regression prior, but it is general data-driven SR rather than PDE residual solving, and it does not use an adaptive activation as a proposal that is then validated by a finite FEX expression-tree search.

### Smooth Symbolic Regression (2108.03274)

Problem: discrete symbolic regression creates abrupt changes in candidate expressions, making the search surface hard to analyze with continuous optimization tools. Method: proposes a transformation that maps symbolic regression instances into smoother real-valued optimization problems. Results: presents a procedure and exploratory analysis for studying smoothed SR search spaces rather than a PDE solver or FEX-style controller. Relevance: useful background for continuous relaxations of symbolic search; it supports treating spline probes as proposal mechanisms, while leaving open whether the final validated object remains a finite expression.

### AutoSINDy (2605.09696) [idea-reviewer, 2026-06-16]

Problem: SINDy requires a pre-specified basis function library, and a mismatched library causes discovery to fail. Method: proposes a "Discovery-then-Solve" framework using PySR (genetic programming) to automatically generate candidate basis functions from bootstrapped data, then curates and filters them (decomposition + collinearity analysis), and finally feeds the curated library into SINDy for sparse PDE identification. Results: reports 92.8% ground-truth recovery rate across canonical nonlinear ODE/PDE systems. Relevance: addresses the same high-level problem of operator-library misspecification, but uses GP-based SR for candidate generation and SINDy for validation rather than a differentiable spline probe and FEX RL expression-tree search. The paradigm (auto-generate → curate → solve) is similar in spirit but fundamentally different in mechanism — no differentiable probe, no finite expression tree RL search, no PDE residual-based solving.

## 12. Active-collocation addenda [idea-refiner, 2026-06-16]

### RL-PINNs (2504.12949)

Problem: PINN training can waste collocation points in uninformative regions, especially when solutions have peaks or wave fronts. Method: trains a DQN sampler that moves through the PDE domain and keeps points with high learned function variation, then retrains the PINN on the selected set. Results: reports lower relative L2 error than uniform, RAR, and RAD on six PDEs, with sampling time below 2% of total runtime. Relevance: it is the closest "RL for adaptive PDE sampling" prior, but the solver is a fixed neural network, not an RL expression-tree search.

### PINNACLE (2404.07662)

Problem: PINNs need to choose PDE, boundary, initial-condition, and experimental points under a fixed training budget. Method: uses augmented-space empirical NTK features and a convergence-degree score to select points, with sampling and K-means variants. Results: reports stronger accuracy and lower point budgets on several forward, inverse, and transfer PDE problems; the ICLR version includes ablations and public code. Relevance: it gives a principled point-selection baseline for PINNs, but its NTK criterion depends on a neural-network training dynamic absent from finite expression search.

### Aikawa, Ueda, and Tanaka (New Generation Computing 2024)

Problem: fixed PINN collocation sets can miss regions where the current solution estimate is uncertain. Method: uses dropout-based Bayesian uncertainty to actively and sequentially add collocation points during PINN training. Results: published as "Improving the Efficiency of Training Physics-Informed Neural Networks Using Active Learning", New Generation Computing 42(4):739-760. Relevance: it is a direct prior for uncertainty-driven PINN collocation, but its uncertainty comes from one neural model rather than disagreement among symbolic candidate expressions.

### Deep Collocation Method (2502.17203)

Problem: one-shot deep-network PDE solves can be inaccurate and hard to terminate with error control. Method: builds a growing space of single-hidden-layer neural basis functions guided by equation residuals, solves a collocation least-squares problem, and uses posterior error indicators to stop sequential updates. Results: the paper reports high accuracy and robustness across several PDE experiments, with adaptive collocation and parameter initialization as supporting pieces. Relevance: it is another residual-guided adaptive collocation method, but it constructs a neural basis subspace rather than searching finite symbolic trees.

### PINNs Failure Modes are Overfitting (2605.30910)

Problem: several PINN failures previously attributed to optimization or precision may instead be overfitting to collocation points. Method: monitors train/test residual divergence and adds double-backpropagation penalties on PDE, boundary, and initial-condition residual terms. Results: reports order-of-magnitude error reductions and up to 23x fewer collocation points on canonical PDEs. Relevance: it is useful background for diagnosing when adaptive collocation overfits local residual regions, although it does not study symbolic PDE solvers.

### Variational residual-based adaptivity (2509.14198)

Problem: residual-based adaptive sampling and weighting in neural PDE solvers lacked a common objective-level explanation. Method: derives a variational framework in which tilted sampling distributions arise from Gibbs/Laplace principles, with exponential and quadratic potentials corresponding to different residual objectives. Results: reports gains for PINNs and operator-learning models, and analyzes signal-to-noise changes during training. Relevance: it provides a clean language for residual-based sampling, but its theory targets continuous neural optimization rather than policy-gradient expression search.

## 13. Deep Lit 2026-06-16 续扫 — Round 1 & 2 (11 篇)

以下 11 篇论文由 `deep-lit-tick --scope topic 0616-fex` 于 2026-06-16 第二轮系统性搜索并精读后收录。本轮搜索覆盖 method/application/benchmark/evaluator/failure-mode/competition 六轴 + 第一轮 5 篇 B7 反向扩展 + 第二轮 6 篇 B7 反向扩展。第二轮 B7 产出 0 篇新 2025+ 论文，文献调研饱和。

### 13.1 Benchmark 与评测体系

- **MDBench (2509.20529)** [wiki](wiki/2509.20529.md): Amirmohammad Ziaei Bideh 等, 2025.09. 开源动力系统模型发现评测框架，含 63 ODE + 14 PDE 数据集 (Heat Laser, Navier-Stokes Cylinder, Reaction-Diffusion Cylinder 等)。12 类方法系统比较：PDE 上 linear methods 更稳，ODE 上 GP 误差更低，linear models 对噪声更鲁棒。引入 equation fidelity 指标超越 NMSE——低 NMSE 可掩盖错误方程结构。对 FEX 的直接价值：提供首个系统的 PDE discovery benchmark，FEX 可在此评测并暴露对比 SINDy/GP 的优劣势。**撞车风险: 低**（工具层，非方法论竞品）。

- **Data-driven discovery of governing DEs review (2606.09638)** [wiki](wiki/2606.09638.md): 2026.06 综述。核心贡献：结构复杂度×系数复杂度可发现性相图 + REO (representation-evaluation-optimization) 统一抽象。强调领域从 canonical rediscovery 走向真实噪声/不完全观测/多尺度/closure model。引入 solvability metric 和 RealPDEBench。与 MDBench 互补形成评测体系。**撞车风险: 低**（综述，为 FEX 评测框架设计提供地图）。

### 13.2 深度符号优化框架

- **DSO: Deep Symbolic Optimization (2505.10762)** [wiki](wiki/2505.10762.md): Conor Hayes, Brenden Petersen, Mikel Landajuela 等 (LLNL), 2025.05. DSO 框架综述章节。将符号回归表述为自回归 token 序列决策→RNN 采样表达式树→RSPG 聚焦 top-quantile→in-situ 约束+常数优化+PQT+GP seeding+Wikipedia/Set Transformer 预训练。uDSO 在 Nguyen/SRBench 上 SOTA。对 FEX 的核心价值：(a) RSPG 可与 FEX policy gradient 对比，(b) in-situ constraint 机制可移植到 FEX controller，(c) GP seeding 为 FEX warm-start 提供范本，(d) Good-Structure-Bad-Score 现象为 FEX 搜索诊断提供概念框架。**撞车风险: 高**——若课题涉及 RL+表达式树搜索则必须引用为核心 prior。

### 13.3 等价性剪枝

- **eggp: Equality Graphs for GP Symbolic Regression (2501.17848)** [wiki](wiki/2501.17848.md): Franca, Kronberger 等, 2025.01. 在 GP 中引入 equality graph 记录全历史访问表达式及其等价形式，改造 subtree crossover/mutation 使新 offspring 避开已访问等价表达式。SRBench AUC 0.56 (vs Operon 0.50, PySR 0.53)。与 EGG-SR (2511.05849) 互补：EGG-SR 将 e-graph 嵌入 MCTS/DRL/LLM，eggp 嵌入 GP。两者均为 FEX gap #8 (EGG 剪枝) 提供可直接移植的模块。**撞车风险: 低**（可作 FEX plugin 而非竞品）。

### 13.4 模型选择与过拟合防治

- **TIR SR with Multi-Objective Optimization (2501.01905)** [wiki](wiki/2501.01905.md): Franca, 2025.01. 将 TIR 符号回归从单目标 accuracy 改为 accuracy+节点数双目标 NSGA-II。核心发现：仅多目标搜索不够，需在 Pareto front 上选接近最佳精度但更小的模型 (TIRMOO-Sel-points) 才能缓解小数据过拟合。对 FEX：为 FEX 表达式选择提供多目标/复杂度约束范式，但 FEX 的核心过拟合问题在 PDE residual 而非 SRBench 数据。**撞车风险: 低**。

- **MDL + Multi-Objective GP for SR (2605.22374)** [wiki](wiki/2605.22374.md): 2026.05. 比较 AIC/BIC/BIC_SR/DL/FBF 五类选择准则 + MO-Length/MO-DL/SO 三种策略。核心发现：先多目标 GP 保留 NLL/length Pareto front，再 DL/FBF/BIC_SR 后处理选择 → 更短且泛化更稳；直接把 DL 当单目标 fitness 反因复杂度压力过强而 underfit。对 FEX：为 FEX 搜索中的表达式选择提供 MDL 视角，但 FEX 的 Adam+BFGS 两步优化已内建复杂度偏好。**撞车风险: 低**。

### 13.5 噪声鲁棒 PDE 发现

- **ANN-PYSR: Noise-Robust PDE Discovery (2506.17908)** [wiki](wiki/2506.17908.md): 2025.06. Savitzky-Golay/Gaussian 平滑→residual attention NN 拟合响应函数和导数→PySR 做 SR 恢复 PDE。覆盖 Burgers/KdV/Klein-Gordon/Convection-Diffusion/Chaffee-Infante/2D Wave。在合成 sparse/noisy benchmark 上比 DLGA/R-DLGA 快很多。直接对应 FEX gap #7（噪声鲁棒性）。对 FEX 的启发：(a) 两级过滤 (平滑+attention) 可作为 FEX 数据预处理管线，(b) PySR 作为 FEX 的 baseline 对比。**撞车风险: 低**（工具层互补，非方法论竞品）。

### 13.6 LLM + 优化驱动 PDE 发现

- **NeuroSym-BO (2601.00088)** [wiki](wiki/2601.00088.md): 2026.01. LLM 生成候选 PDE→STRidge 数值评价→Bayesian Optimization (EI) 动态选择下一轮指令。在 Burgers/Fisher/Chafee/Divide/Allen-Cahn 上测试 Llama-3.2-3B。对 FEX gap #2 (LLM+FEX 骨架生成) 提供新思路：BO 驱动的指令调优可替代 LLM 直接生成，但当前仅在 1D PDE 且 LLM pipeline 脆弱。**撞车风险: 低**（LLM pipeline 与 FEX 方法学正交）。

### 13.7 搜索策略与表示

- **T-SHRED (2506.15881)** [wiki](wiki/2506.15881.md): Alexey Yermakov, J.N. Kutz 等, 2025.06. Transformer 替换 SHRED RNN→SINDy-Attention 使每个 head 学习可解释线性潜空间 ODE。诚实发现：next-step RMSE 上 GRU-SHRED 仍最优，T-SHRED 价值在潜空间解释和非自回归 one-shot 长程 rollout。对 FEX：SINDy-Attention 的"可解释潜空间正则化"思想可启发 FEX 搜索过程中的符号约束，但 T-SHRED 面向稀疏传感重构而非 PDE 符号求解。**撞车风险: 低**。

- **Exhaustive Symbolic Regression (2507.13033)** [wiki](wiki/2507.13033.md): Harry Desmond, 2025.07. 在给定算子集和复杂度上限内穷举表达式树，用 MDL 统一精度/复杂度/参数编码为 nats。在 feynman 和三个天体物理问题上找到优于文献标准的低描述长度函数。对 FEX 的直接价值：(a) 穷举搜索为 FEX controller 提供 ground truth oracle——在某复杂度上限内 RL 是否找到最优表达式？(b) MDL 为 FEX 的表达式选择提供 principled 准则。但穷举不 scale 到 FEX 的 d=100 树。**撞车风险: 低**。

- **Multi-view SR in Physical Sciences (2509.10500)** [wiki](wiki/2509.10500.md): Franca, Kronberger 等, 2025.09. 比较 Operon/PySR/phy-SO/eggp 四种 MvSR 实现于 5 个真实科学多视角数据集。核心发现：MvSR 在共享表达式结构、每 view 独立参数下可找到准确且简单的公式，但参数数目/loss 聚合/uncertainty 处理显著影响科学可用性。对 FEX：(a) MvSR 范式可启发 FEX 对多 PDE 实例共享骨架，(b) 为 fex-dim-lift-skeleton 和 fex-active-collocation 提供方法参考。**撞车风险: 低**。

### 13.8 补遗：B7 发现但无 arXiv preprint 的相关论文

- **T-NNGP (AAAI 2025)**: Lulu Cao, Zexin Lin, Kay Chen Tan. GP+DRL 混合框架求解多物理场 PDE，含 universal decoupling strategy。与 NetGP 同组。无 arXiv preprint。
- **GraphDSR (Neural Networks 2025)**: Jingyi Liu, Lina Yu 等. 用 DAG 表示数学表达式，generative GNN 采样+RL 训练。110 benchmark 评测。无 arXiv preprint。
- **SDSR (IEEE MIND 2025)**: Zexin Lin, Yunpeng Gong 等. 多 decoder LSTM 同时生成耦合 PDE 系统符号解，curriculum learning 渐进增长表达式长度。无 arXiv preprint。

## 14. 更新后的撞车风险矩阵 (2026-06-16 续扫后)

| FEX Claim / Gap | 2026-06-16 状态 | 关键竞品 | 变化 |
|----------------|----------------|---------|------|
| S_k 理论保证 (无 CoD) | ✅ 唯一 | 无 | — |
| RL+表达式树搜索 PDE 解 | ⚠️ 双线竞争 | SymPlex, SSDE, DSO | +DSO (LLNL 框架) |
| 高维 PIDE (d=100) | ✅ 唯一 | 无 | — |
| LLM+FEX 骨架生成 | 🟢 完全空白 | PiT-PO, FunctionEvolve, NeuroSym-BO | +NeuroSym-BO (BO pipeline) |
| Egg 剪枝 + FEX | 🟢 完全空白 | EGG-SR, eggp 均提供可直接移植模块 | +eggp (GP side) |
| 频率分离失效 | 🟢 完全空白 | 无 | — |
| 预训练模型+FEX warm-start | 🟢 完全空白 | DSO GP seeding, GENSR, SymTorch | +DSO seeding |
| RL 搜索收敛率理论 | 🟢 完全空白 | Exhaustive SR oracle 可标定上界 | +oracle benchmark |
| 噪声鲁棒性 (gap #7) | 🟡 有新基线 | ANN-PYSR, NRSR | +ANN-PYSR |
| PDE 符号求解 benchmark | 🟡 有新工具 | MDBench, ERBench, RealPDEBench | +MDBench |
| 搜索表达式选择/过拟合 | 🟡 有新方法 | MDL-GP, TIRMOO, BIC_SR | +MDL 选择准则 |

## 15. Dim-lift OpenReview addendum [idea-refiner, 2026-06-17]

### Projective Symbolic Regression (PSR)

Problem: Applying symbolic regression directly to high-dimensional PDE solution data is intractable because the expression and data spaces grow quickly with dimension. Method: PSR generates low-dimensional projections by fixing subsets of variables, runs local symbolic regression on each projection, then treats the local expressions as high-level tokens for a global symbolic regression stage constrained by PDE residual. Results: the OpenReview submission reports improved predictive accuracy and interpretable compositional solutions over conventional SR-style baselines; as of the checked OpenReview page, it is listed as submitted to ICLR 2026 and modified on 2026-02-11, not as an accepted poster/oral. Relevance: this is a close prior for low-dimensional-to-high-dimensional symbolic PDE solving, but its mechanism is projection decomposition plus global SR composition rather than direct invariant-macro inference from a learned skeleton.

## 16. Deep Lit 2026-06-17 新收录 (5 篇)

以下论文由 `deep-lit-tick --scope topic 0616-fex` 于 2026-06-17 系统性搜索并精读后收录。本轮 6 axes 搜索 + B7 反向扩展发现文献高度饱和（132 篇已读 + 仅 5 篇新 wiki），确认 FEX 竞争格局无重大变化。

### 16.1 FEX 族内扩展（Yang 组）

- **H-FEX (2506.20607)** [wiki](wiki/2506.20607.md): Jasen Lai, Senwei Liang, Chunmei Wang, 2025.06. 将 FEX 适配到哈密顿系统。核心创新：(a) 将辛积分器嵌入 FEX 的 RL 搜索循环，强制哈密顿动力学约束；(b) 引入 interaction node 捕捉多体系统中位置坐标之间的成对相互作用项（如 1/||qi-qj||）。在非可分系统和三体问题（12 维）上精确恢复解析形式（系数误差 <1%），远超 SINDy/HNN/SRNN/SANN。坦承依赖预定义树结构和 operator 字典。**撞车风险: 无**——修改的是 FEX 应用层而非核心搜索机制。interaction node 设计模式可作 FEX 领域扩展参考。

- **FEX-DM (2504.07085)** [wiki](wiki/2504.07085.md): Senwei Liang, Chunmei Wang, Xingjian Xu, 2025.04. 两阶段 SDE 识别框架：Stage 1 用 FEX (RL 符号回归) 恢复确定性漂移项 μ(x_t)；Stage 2 用免训练条件扩散模型 (TF-CDM) 学习扩散项分布。5 个案例均成功恢复漂移表达式，外推预测精度远超纯 NN 方法。仅 90 轮 RL 迭代即恢复 OU 过程 (1.1989 - 0.9953x_t vs 真值 1.2 - x_t)。**撞车风险: 无**——FEX 应用实例，非方法论改进。展示了符号回归+生成模型混合框架的强大外推能力。

### 16.2 竞争方法景观

- **GP-SDE (2603.09597)** [wiki](wiki/2603.09597.md): Sigur de Vries, Sander Keemink, Marcel van Gerven (Radboud), 2026.03. 首次将 GP 应用于 SDE 符号发现。用 MLE 作为适应度函数，JAX/Kozax 框架联合进化漂移+扩散项。关键发现：(a) 高维 (Lorenz96, d=10-20) 和混沌系统中显著超越 KM-SR（维度灾难）；(b) 多步积分扩展使稀疏采样下仍准确恢复；(c) 显式学习扩散项反而改善漂移识别——对 FEX 是重要设计原则。可推广到 SPDE。诚实讨论 identifiability 问题和高斯噪声假设。**撞车风险: 低**——系统辨识 vs FEX 的特征提取，但 Kozax GP 库、MLE 适应度函数和多步积分技巧可复用。方法论 insight：显式建模噪声提升确定性分量恢复。

- **FIND (2509.07303)** [wiki](wiki/2509.07303.md): Tingxiong Xiao 等 (清华 Suo 组), 2025.09. 统一科学发现框架：Buckingham Pi 定理设计潜变量层 + Taylor 定理设计表达式层 + SHAP/PPMCC 筛选 + 量纲不变性约束 + C2F 网格搜索 + 多级优化，搜索迭代从数亿次降至数十次。11 个数据集验证，从 NASA 行星数据发现 58 个物理公式（57 个理论可证）。**撞车风险: 低**——物理约束+网格搜索 vs FEX RL 学习搜索策略，方法路线正交。DI 约束和两阶段分解可启发 FEX 引入 action space masking 和 hierarchical RL。应在 FEX 论文中引用，论证 FEX 无需量纲先验的优势场景。

- **FISR-EQL (2604.14569)** [wiki](wiki/2604.14569.md): Jiazhe Li, Chenyu Wu 等, 2026.04. 将 Equation Learner 符号网络嵌入伴随方法 PDE 约束场反演，端到端可解释湍流模型修正。消除传统两阶段 FIML 的目标失配。在 SST 湍流模型上提取出仅 9 个非零参数的显式表达式。反直觉发现：显式解析表达式在输入分布偏移下比神经网络更鲁棒（FAITH Hill 案例中 NN 几乎不激活）。**撞车风险: 低**——面向湍流建模的工程应用，与 FEX PDE 求解场景不重叠。端到端嵌入思想和解析表达式鲁棒性论点可借鉴。

### 16.3 B7 发现但未深读的相关论文

- **FEX-Turbulence (2605.10687)**: Xingjian Xu, Di Qi, Chunmei Wang, 2026.05. FEX 用于湍流动力学，两阶段框架：FEX 发现确定性动力学闭式表达式 → 生成模型学习残差随机分量。可恢复 5 阶矩。Yang 组 FEX 最新扩展，已记载于 §1。

### 16.4 更新后的撞车风险矩阵 (2026-06-17)

| FEX Claim / Gap | 2026-06-17 状态 | 关键竞品 | 变化 |
|----------------|----------------|---------|------|
| S_k 理论保证 (无 CoD) | ✅ 唯一 | 无 | — |
| RL+表达式树搜索 PDE 解 | ⚠️ 双线竞争 | SymPlex, SSDE, DSO | — |
| 高维 PIDE (d=100) | ✅ 唯一 | 无 | — |
| LLM+FEX 骨架生成 | 🟢 完全空白 | PiT-PO, FunctionEvolve, NeuroSym-BO | — |
| Egg 剪枝 + FEX | 🟢 完全空白 | EGG-SR, eggp | — |
| 频率分离失效 | 🟢 完全空白 | 无 | — |
| 预训练模型+FEX warm-start | 🟢 完全空白 | DSO GP seeding, GENSR, SymTorch | — |
| RL 搜索收敛率理论 | 🟢 完全空白 | Exhaustive SR oracle | — |
| 噪声鲁棒性 (gap #7) | 🟡 有新基线 | ANN-PYSR, NRSR, GP-SDE | +GP-SDE (SDE 场景) |
| PDE 符号求解 benchmark | 🟡 有新工具 | MDBench, ERBench, RealPDEBench | — |
| 搜索表达式选择/过拟合 | 🟡 有新方法 | MDL-GP, TIRMOO, BIC_SR | — |
| FEX 扩展到 Hamiltonian | ✅ FEX 已有 | H-FEX (Yang 组) | +H-FEX (自身扩展) |
| FEX 扩展到 SDE | ✅ FEX 已有 | FEX-DM (Yang 组) | +FEX-DM (自身扩展) |
| 端到端符号 PDE 修正 | 🟡 有新方法 | FISR-EQL | +FISR-EQL (工程应用) |
| 物理约束搜索空间缩减 | 🟡 有新方法 | FIND | +FIND (量纲约束) |

## 17. RL-PDE reward shaping addendum [idea-refiner, 2026-06-17]

### Deep Reinforcement Learning-Based Symbolic Regression for PDE Discovery Using Spatio-Temporal Rewards

Problem: PDE symbolic discovery from data can miss temporal dynamics when the reward is based only on static residual or pointwise fit. Method: the paper builds a deep symbolic regression framework with a dense encoder-decoder that extracts temporal-dynamics information, then injects this information through a spatio-temporal reward for reinforcement-learning-based PDE symbolic regression. Results: on a Cahn-Hilliard example, it reports better identification accuracy than baselines with less data, plus a parameter-sensitivity analysis. Relevance: this is another RL-based PDE symbolic regression line and should be cited when comparing reward-shaping approaches, but it does not study FEX, oscillatory PDE solving, or frequency-estimation priors.

## 18. Permutation-invariant SR representation addendum [idea-reviewer, 2026-06-17]

### GSR: Graph-based Symbolic Regression with Invariance and Constraint Encoding

Ziyu Xiang, Kenna Ashen, Xiaofeng Qian, Xiaoning Qian (Texas A&M), NeurIPS 2025. Problem: existing SR methods suffer from redundant representations that break permutation invariance (e.g., a+b vs b+a are treated as different expressions) and sparse rewards when constraints can only be evaluated on complete expressions. Method: proposes expression graphs (EGs) — permutation-invariant DAG representations that encode expression equivalences via a term-rewriting system (TRS) with commutativity normalization. Uses hybrid neural-guided MCTS (hnMCTS) with constraint-informed neural guidance on EGs. Results: theoretical proofs of EG uniqueness and hnMCTS convergence; strong empirical results on synthetic and real-world scientific datasets. Relevance: the permutation invariance studied here is **within-expression commutativity** (a+b vs b+a), NOT cross-dimension exchangeability macro inference. GSR addresses representation efficiency for general SR search; our fex-dim-lift-skeleton addresses whether a learned skeleton across dimensions can be certified as liftable. No overlap in mechanism or claim, but GSR should be cited as related work on permutation-invariant SR representation.

## 19. Deep Lit 2026-06-17 续扫 — Round 3 (5 篇)

以下 5 篇论文由 `deep-lit-tick --scope topic 0616-fex` 于 2026-06-17 第三轮系统性搜索并精读后收录。本轮 6 axes 搜索 + B7 反向扩展（20 调用）验证文献高度饱和（155 篇已读，仅 5 篇通过筛选）。B7 产出 3 篇低相关度论文未达精读门槛，本轮自然终止。

### 19.1 FEX 族内扩展（Yang 组，需补 wiki）

- **FEX+TranNet (2604.22208)** [wiki](wiki/2604.22208.md): P. Huynh, Feng Bao, Haizhao Yang, Ahmed Zytoon, 2026.04. 将 FEX 候选算子池从手工解析函数扩展到 TransNet 训练的浅层 NN 候选。60D Poisson / 60D reaction-diffusion / 55D semilinear elliptic 上测试。核心发现：候选池含真实结构时误差可达 1e-7；移除 TN_x² 等关键候选后误差上升 2-3 个数量级——暴露 FEX 对候选池质量的极端敏感性。TransNet 候选在没有 ground truth 结构时产生光滑但欠精确的解（~1e-3），但对 FEX 算子选择启发式提供了新的 regularization 思路。**撞车风险: 无**——Yang 组自身扩展，非竞品。为 FEX gap #4（预训练模型+FEX warm-start）提供半成品范本。

- **LLM+FEX (2503.09986)** [wiki](wiki/2503.09986.md): Rohan Bhatnagar, Ling Liang, Krish Patel, Haizhao Yang, 2025.03. 用合成 PDE 数据训练 LLaMA/T5/BART 预测 FEX 解析解所需算子集合（以 postfix 表示）。198K 样本（随机表达式树→反推 PDE RHS + BC）。LLaMA-8B 最佳：12 epoch 后 average mismatch rate 0.56%。将预测软先验（probability mask）注入 FEX 控制器使其偏向高概率算子，Poisson 和 Linear Conservation Law 上搜索轮数减少约 40-60%。局限：仅覆盖 Poisson/Conservation Law；依赖合成训练数据（非真实 PDE 分布）；LLM 推理开销约 2-5s/query。**撞车风险: 无**——Yang 组 FEX+LLM 第一步，已在 landscape §1 记载。方法论 insight：soft operator prior 机制可作为 FEX 引入预训练先验的通用接口。

### 19.2 竞争方法景观

- **GWAgent (2605.11280)** [wiki](wiki/2605.11280.md): Tousif Islam, D. Wadekar, T. Venumadhav 等, 2026.05. LLM/coding agent 在可验证仿真闭环中构造偏心双黑洞引力波的可解释解析 surrogate。核心范式：先物理 ansatz 拆主结构 → agent 搜索 residual polynomial-Fourier ridge basis → held-out mismatch/phase error/runtime/likelihood 四重验证 gate。核心教训（对 FEX）：(a) residual-first 搜索比全结构搜索更稳健；(b) validation gate 必须是 domain-level 的独立审查（不仅是表达式拟合 loss）；(c) 失败模式分类学（缓存、数据泄漏、错用 validation metric、物理公式写错）应直接写入 agent 设计。**撞车风险: 低**——GW 物理场景 vs FEX PDE 求解，但 agentic verifier + residual-first search 范式可直接移植到 FEX workflow。该引。

- **Numerical Spectrum Linking (2510.23078)** [wiki](wiki/2510.23078.md): Phonepaserth Sisaykeo, S. Muramatsu, 2025.10. Chebyshev 节点采样 + DCT-II → Koopman 矩阵 → 方程驱动 Koopman vs 数据驱动 Koopman 特征值匹配 → PDE 识别。仅覆盖 Advection-X 和 Burgers 1D 案例。局限性：不处理非线性算子选择（回归做在矩阵层面而非符号层面）、对噪声敏感、纯数据驱动无物理约束。**撞车风险: 低**——与 FEX 方法论正交（谱匹配 vs RL 符号搜索），但为 PDE 识别提供了不依赖候选库的替代路径，应作为 related work 引用。

- **ASyMOB (2505.23851)** [wiki](wiki/2505.23851.md): Michael Shalyt, Rotem Elimelech, I. Kaminer, 2025.05. 35,368 个符号数学问题的反记忆 benchmark（积分/极限/ODE/级数/超几何），经 symbolic/numeric/equivalence 三类扰动。核心发现：minor perturbation 下大多数 LLM 性能崩溃，top system 出现 regime shift。**撞车风险: 低**——面向 LLM 符号数学评测，非 PDE 求解。方法论启示：equivalence-preserving transformation 的扰动测试方法论可移植到 FEX benchmark 设计（测试 FEX 对等价表达式重排的鲁棒性）。

### 19.3 B7 发现但未精读的低相关度论文

- **Decoding PDEs (2510.05278)**: Paloma García-de-Herreros 等, 2025. decoder-only LM 跨模态适配 PDE。1 citation。
- **SciML Agents (2509.09936)**: S. Gaonkar 等, 2025. "Write the Solver, Not the Solution"——LLM agent 生成数值求解器代码。6 citations。
- **Hash Consing for Symbolic Computation (2509.20534)**: Bowen Zhu 等, 2025. Julia 符号计算中 hash consing 消除表达式冗余。加速最高 100x。与 EGG-SR 的 e-graph 等价性剪枝互补。

### 19.4 更新后的撞车风险矩阵 (2026-06-17 Round 3)

| FEX Claim / Gap | 2026-06-17 状态 | 关键竞品 | 变化 |
|----------------|----------------|---------|------|
| S_k 理论保证 (无 CoD) | ✅ 唯一 | 无 | — |
| RL+表达式树搜索 PDE 解 | ⚠️ 双线竞争 | SymPlex, SSDE, DSO | — |
| 高维 PIDE (d=100) | ✅ 唯一 | 无 | — |
| LLM+FEX 骨架生成 | 🟡 Yang 组已有初步工作 | LLM+FEX (2503.09986) | 已有内部 baseline |
| Egg 剪枝 + FEX | 🟢 完全空白 | EGG-SR, eggp | — |
| 频率分离失效 | 🟢 完全空白 | 无 | — |
| 预训练模型+FEX warm-start | 🟡 Yang 组已有初步工作 | FEX+TranNet (2604.22208) | 已有内部 baseline |
| RL 搜索收敛率理论 | 🟢 完全空白 | Exhaustive SR oracle | — |
| Agentic verifier / validation gate | 🟡 有新范式参考 | GWAgent (2605.11280) | +agentic verifier |
| 候选池质量敏感性 | 🟡 有新诊断 | FEX+TranNet 消融实验 | +candidate pool diagnosis |

## 20. Hayden Schaeffer PROSE 系列补全 [deep-lit-tick, 2026-06-24]

以下 9 篇论文由 `deep-lit-tick --scope topic 补全 Hayden Schaeffer 的 PROSE 相关工作` 于 2026-06-24 补录。此前 landscape 仅在 §2 一句话提及 PROSE (Liu, Zhang & Schaeffer 2023)，未精读。本轮系统性搜索 Schaeffer 的 PROSE 系列及其理论/应用扩展，填补该作者线的调研盲区。

### 20.1 PROSE 核心系列（Schaeffer 组，符号+算子双模态 PDE 基础模型）

- **PROSE (2309.16816)** [wiki](wiki/2309.16816.md): Yuxuan Liu, Zecheng Zhang, Hayden Schaeffer, 2023.09, Neural Networks 2024. 原始论文。用 Polish notation 将 ODE 方程编码为 token 序列，两个独立 transformer encoder 分别处理数值轨迹和符号表达式，hierarchical attention 做特征融合，双 decoder 分别输出算子预测和符号方程。在 25 个 ODE 族上训练，符号解码器可恢复控制方程。**关键对 FEX 的意义**：PROSE 的符号编码方式（Polish notation token 序列）与 FEX 的二叉树表示本质同构——都是表达式树的序列化。PROSE 用预训练 Transformer 做符号回归，而 FEX 用 RL policy gradient 搜索。两种方法的 warm-start 整合是 landscape gap #4 的核心候选路径。24 citations。

- **PROSE-PDE (2404.12355)** [wiki](wiki/2404.12355.md): Jingmin Sun, Yuxuan Liu, Zecheng Zhang, Hayden Schaeffer, 2024.04. PROSE 扩展到 1D 时间相关非线性 PDE。20 族 PDE、512K 系统训练。关键新发现：(1) 在 Known 模式（完整方程已知）下数据误差 0.92%；(2) **Skeleton 模式（仅知方程结构不知系数）下数据误差 1.06%**——证明即使系数未知，仅提供符号骨架即可大幅改善预测；(3) 三项外推实验展示 zero-shot 泛化到未见 PDE。这直接验证了 "LLM 生成骨架 → FEX 搜索系数" 范式的可行性。51 citations。**撞车风险: 中**——PROSE-PDE 的 Skeleton 模式与 fex-llm-skeleton idea 的 "LLM 预测骨架 + FEX 搜索系数" 高度相似，但 PROSE-PDE 用 Transformer 做算子学习（数值解预测），而 FEX 用 RL 搜索解析解。**必须引用并详细区分**。

- **PROSE-FD (2409.09811)** [wiki](wiki/2409.09811.md): Yuxuan Liu, Jingmin Sun, Xin He, Griffin Pinney, Zecheng Zhang, Hayden Schaeffer, 2024.09. PROSE 扩展到 2D 流体力学：169M 参数，6 类方程（浅水波、可压/不可压 NS），13 个数据集、60K+ 轨迹。patch-based encoder-decoder，非自回归一次预测 10 步。在标准 benchmark 上超越 MPP、DPOT 等。对 FEX：展示了符号编码 + 数据融合在高维 PDE 上的 scale-up 能力，但 PROSE-FD 是纯数值预测器而非解析解搜索器。32 citations。

- **BCAT (2501.18972)** [wiki](wiki/2501.18972.md): Yuxuan Liu, Jingmin Sun, Hayden Schaeffer, 2025.01. PROSE 家族最新成员。引入 block causal transformer 架构做 next frame prediction（同帧内双向 spatial attention + 帧间因果）。在 6 个流体力学下游任务上平均相对误差 1.18%。核心工程贡献：next frame prediction 比 next token prediction 精度提升 3.5x，推理加速 185x。对 FEX：BCAT 放弃了符号模态，转向纯数据驱动自回归——标志着 Schaeffer 组在实用 PDE 求解中认为符号编码的边际收益不如架构优化。15 citations。

### 20.2 PROSE 多模态扩展

- **PROSE + Text (2502.06026)** [wiki](wiki/2502.06026.md): Elisa Negrini, Yuxuan Liu, Liu Yang, Stanley Osher, Hayden Schaeffer, 2025.02. 将 PROSE 的符号模态替换/扩展为自然语言文本描述。GPT-2 backbone，52 个参数化 ODE/PDE 训练。in-distribution 误差 <3.3%，文本描述 F1>0.93。**对 FEX 的重要性**：展示了符号编码并非唯一的方程描述方式——文本描述在 symbolic representation 不完整时可作为替代。但文本训练数据由 GPT-4 生成，本质是 LLM 蒸馏而非真正物理理解。7 citations。

- **PROSE + SymPy (2409.11609)** [wiki](wiki/2409.11609.md): Derek Jollie, Jingmin Sun, Zecheng Zhang, Hayden Schaeffer, 2024.09. 用 SymPy 自动标准化 PDE 符号编码，解决等价表达式产生不同 token 序列的问题（类似 EGG-SR 的等价性剪枝问题）。SymPy tree 降低 L² 误差从 2.18% 到 1.42%。引入 SMC 粒子滤波做系数精炼。**对 FEX 的重要性**：(1) SymPy 标准化可直接移植到 FEX 的候选表达式去重；(2) 粒子滤波系数精炼可补充 FEX 的 Adam+BFGS 两步优化。6 citations。

- **PI-MFM (2512.23056)** [wiki](wiki/2512.23056.md): Min Zhu, Jingmin Sun, Zecheng Zhang, Hayden Schaeffer 等, 2025.12. 将物理约束（PDE 残差 + IC/BC）注入 PROSE 预训练。核心：自动从 Polish notation 解析表达式树并向量化计算 PDE 残差损失。在稀疏标注（4×16 网格）下 L² 误差 ~1% vs 纯数据驱动 ~11.5%。**关键发现：zero-shot physics-informed fine-tuning**——从 physics-informed 预训练模型出发，仅用 PDE 残差和 IC/BC（无任何标注解数据）即可快速适配到未见 PDE，误差降到 ~1%，显著优于从零开始的 physics-only 训练。**对 FEX 的重要性**：PI-MFM 的 PDE 残差自动计算机制与 FEX 的 RL 奖励函数（PDE 残差最小化）异曲同工。PI-MFM 用 Transformer + 梯度下降优化残差，FEX 用 RL + Adam/BFGS 优化残差。两者的整合（PI-MFM 预训练 → FEX 精搜）是 gap #4 最直接的实现路径。5 citations。

### 20.3 PROSE 理论基础

- **LeMON (2408.16168)** [wiki](wiki/2408.16168.md): Jingmin Sun, Zecheng Zhang, Hayden Schaeffer, 2024.08. 将 MAML meta-learning 应用于多算子学习。关键发现：预训练数据多样性（多族 PDE）是泛化的关键驱动力。通过 LeMONS 模块引入符号编码做 task-specific 自适应。**对 FEX 的启发**：FEX 目前每个 PDE 从零搜索，LeMON 的 meta-learning + 多族预训练范式可为 FEX 提供跨 PDE 族的 warm-start 策略。

- **MNO/MONet (2510.25379)** [wiki](wiki/2510.25379.md): Adrien Weihs, Jingmin Sun, Zecheng Zhang, Hayden Schaeffer, 2025.10. 多算子学习的统一理论框架。证明了 MNO 对 Lipschitz 多算子映射的万能逼近定理和显式 scaling law（网络大小与精度的关系）。关键理论结果：多任务共享表征不增加总体代价——多算子学习与单算子学习遵循相同的 scaling law。**对 FEX 的重要性**：为 "预训练模型 + FEX" 整合提供理论背书——如果多算子预训练不增加表达成本，那么用 PROSE 类模型为 FEX 提供 warm-start 在理论上是免费的。

### 20.4 B7 发现但未精读的其他 Schaeffer 论文（2024+）

- **ViCON (2411.16063)**: Yadi Cao, Yuxuan Liu, Liu Yang, Rose Yu, Hayden Schaeffer, Stanley Osher, 2024.11. Vision In-Context Operator Networks，patch-wise 2D 处理。面向流体力学 benchmark。与 FEX 方法论不重叠。
- **2605.22724**: Adrien Weihs, Hayden Schaeffer, 2026.05. 多任务 neural operator 的 near-optimal rates 理论。与 2510.25379 互补。
- **2604.01961**: Adrien Weihs, Hayden Schaeffer, 2026.04. MNO 的泛化界与统计保证。与 2510.25379 互补。
- **2512.17884**: Xinyue Yu, Hayden Schaeffer, 2025.12. RRFF-FEM: 算子学习中的正则化随机 Fourier 特征。面向去噪算子学习，非符号回归。
- **2604.00333**: Liyao Lyu, Xinyue Yu, Hayden Schaeffer, 2026.04. MVNN: 学习 McKean-Vlasov 动力学。面向粒子系统，非 PDE 符号求解。

### 20.5 更新后的撞车风险矩阵 (2026-06-24 Schaeffer PROSE 补全后)

| FEX Claim / Gap | 2026-06-24 状态 | 关键竞品 | 变化 |
|----------------|----------------|---------|------|
| S_k 理论保证 (无 CoD) | ✅ 唯一 | 无 | — |
| RL+表达式树搜索 PDE 解 | ⚠️ 双线竞争 | SymPlex, SSDE, DSO | — |
| 高维 PIDE (d=100) | ✅ 唯一 | 无 | — |
| LLM+FEX 骨架生成 | 🟡 Yang 组已有初步工作 | LLM+FEX (2503.09986) | — |
| 符号骨架 warm-start | ⚠️ **Schaeffer PROSE-PDE Skeleton 模式是直接 prior** | PROSE-PDE Skeleton mode (2404.12355) | +PROSE Skeleton |
| Egg 剪枝 + FEX | 🟢 完全空白 | EGG-SR, eggp | — |
| 频率分离失效 | 🟢 完全空白 | 无 | — |
| 预训练模型+FEX warm-start | ⚠️ **PROSE/PI-MFM 提供直接可用的预训练 backbone** | PI-MFM (2512.23056), LeMON (2408.16168) | +PI-MFM +LeMON |
| RL 搜索收敛率理论 | 🟢 完全空白 | Exhaustive SR oracle | — |
| 多算子 scaling law 理论 | 🟡 有新理论 | MNO/MONet (2510.25379) | +MNO 理论 |
| 符号标准化 / 等价性 | 🟡 有新工具 | PROSE+SymPy (2409.11609), EGG-SR, eggp | +SymPy 标准化 |
| PDE 残差自动计算 | 🟡 有新框架 | PI-MFM 自动 PDE 残差 | +PI-MFM |

## 21. Schaeffer PROSE 连通分支穷尽 [deep-lit-tick, 2026-06-24]

§20 补录了 PROSE 核心系列 9 篇。本节穷尽该连通分支的剩余部分：Schaeffer 组的 operator learning 基础设施、理论框架、以及 PROSE 直接竞品/上游 PDE 基础模型。共 23 篇新 wiki（R3-R6）。

### 21.1 PROSE 竞品：PDE Foundation Models

- **ICON (2304.07993)** [wiki](wiki/2304.07993.md): Liu Yang, Siting Liu, Tingwei Meng, S. Osher, 2023.04. In-Context Operator Networks，用 data prompts 做 few-shot 算子学习。PROSE 的直接上游——PROSE-PDE 和 PROSE-FD 都引用 ICON 作为 in-context learning 范式来源。126 citations。
- **ICON-PDE (2401.07364)** [wiki](wiki/2401.07364.md): Liu Yang, Stanley Osher, 2024.01. ICON 扩展到 1D 非线性守恒律。PROSE-PDE 的直接竞品。38 citations。
- **MPP (2310.02994)** [wiki](wiki/2310.02994.md): Michael McCabe 等 (Flatiron), 2023.10. Multiple Physics Pretraining，PROSE-FD 的主要 baseline。在多物理场景上预训练 ViT。103 citations。
- **PDEformer (2402.12652)** [wiki](wiki/2402.12652.md): Zhanhong Ye 等 (MSRA/PKU), 2024.02. 1D PDE 基础模型，用 DAG 表示 PDE 并嵌入 Transformer。PROSE-PDE 的直接竞品。48 citations。
- **Poseidon (2405.19101)** [wiki](wiki/2405.19101.md): Maximilian Herde 等 (ETH), 2024.05. 高效 PDE 基础模型，多尺度 latent Transformer。PROSE-FD 的竞品。

### 21.2 Schaeffer 组 Operator Learning 基础设施

- **BelNet (2212.07336)** [wiki](wiki/2212.07336.md): Zecheng Zhang, W.T. Leung, Hayden Schaeffer, 2022.12. Mesh-free neural operator，用 Bernstein/Legendre basis 做 localized 表征。PROSE 的早期基础架构。
- **D2NO (2310.18888)** [wiki](wiki/2310.18888.md): Zecheng Zhang, Christian Moya, Lu Lu, Guang Lin, Hayden Schaeffer, 2023.10. Distributed Deep Neural Operators，处理异构输入函数空间。为 MODNO 和 PROSE 提供分布式架构范式。
- **MODNO (2404.02892)** [wiki](wiki/2404.02892.md): Zecheng Zhang, 2024.04. Multi Operator Learning with Distributed Neural Operators，直接喂入 PROSE-PDE。20 citations。
- **DeepONet-MOE (2411.07239)** [wiki](wiki/2411.07239.md): Zecheng Zhang, Christian Moya, Lu Lu, Guang Lin, Hayden Schaeffer, 2024.11. DeepONet 做多算子外推 + physics-informed fine-tuning。PROSE 的 DeepONet 对应物。
- **FERN (2510.26962)** [wiki](wiki/2510.26962.md): Zecheng Zhang, Hao Liu, Guosheng Fu, Hayden Schaeffer, Guang Lin, 2025.10. 有限元表征网络，将 FEM basis 嵌入 neural operator。
- **ViCON (2411.16063)** [wiki](wiki/2411.16063.md): Yadi Cao, Yuxuan Liu, Liu Yang, Rose Yu, Hayden Schaeffer, Stanley Osher, 2024.11. Vision In-Context Operator Networks，patch-wise 处理 2D 数据。PROSE-FD 的 in-context 竞品。
- **Conformalized-DeepONet (2402.15406)** [wiki](wiki/2402.15406.md): Christian Moya, A. Mollaali, Zecheng Zhang, Lu Lu, Guang Lin, 2024.02. 无分布假设的 DeepONet 不确定性量化。
- **Discretization-invariant DON (2307.09738)** [wiki](wiki/2307.09738.md): Zecheng Zhang, W.T. Leung, Hayden Schaeffer, 2023.07. 分析 DeepONet 的离散不变性。

### 21.3 Schaeffer 组 Operator Learning 理论

- **Neural Scaling Laws (2410.00357)** [wiki](wiki/2410.00357.md): Hao Liu, Zecheng Zhang, Wenjing Liao, Hayden Schaeffer, 2024.10. Deep ReLU / Deep Operator Network 的 scaling law 理论。证明 DON 逼近率和 Monte Carlo 误差界。
- **MNO Generalization Bounds (2604.01961)** [wiki](wiki/2604.01961.md): Adrien Weihs, Hayden Schaeffer, 2026.04. MNO 的 Rademacher/PAC-Bayes 泛化界。
- **MNO Near-Optimal Rates (2605.22724)** [wiki](wiki/2605.22724.md): Adrien Weihs, Hayden Schaeffer, 2026.05. 多任务 neural operator 的 minimax-optimal rates。证明 multi-task 共享不增额外代价。
- **RRFF-FEM (2512.17884)** [wiki](wiki/2512.17884.md): Xinyue Yu, Hayden Schaeffer, 2025.12. 正则化随机 Fourier 特征 + 有限元重建做算子学习。噪声鲁棒。
- **Cauchy Random Features (2503.00300)** [wiki](wiki/2503.00300.md): Chunyang Liao, Deanna Needell, Hayden Schaeffer, 2025.03. Cauchy 分布随机特征做 Sobolev 空间算子学习。

### 21.4 Schaeffer 组应用拓展

- **Bayesian DeepONet (2308.14188)** [wiki](wiki/2308.14188.md): Zecheng Zhang, Christian Moya, W.T. Leung, Guang Lin, Hayden Schaeffer, 2023.08. Bayesian 深度算子学习做多尺度 PDE 同态到精细映射。
- **Random Feature Models for Dynamics (2212.05591)** [wiki](wiki/2212.05591.md): Yuxuan Liu, S. McCalla, Hayden Schaeffer, 2022.12. 随机特征模型学习交互动力学。PROSE 方向的早期探索。
- **MVNN (2604.00333)** [wiki](wiki/2604.00333.md): Liyao Lyu, Xinyue Yu, Hayden Schaeffer, 2026.04. Measure-valued NN 学习 McKean-Vlasov 粒子动力学。
- **Multiscale Nudging (2606.06809)** [wiki](wiki/2606.06809.md): Liyao Lyu, Xinyue Yu, Hayden Schaeffer, 2026.06. 从宏观观测到微观粒子动力学的数据同化。Schaeffer 最新工作。
- **NAMO (2602.17080)** [wiki](wiki/2602.17080.md): Minxin Zhang, Yuxuan Liu, Hayden Schaeffer, 2026.02. Adam+Muon 正交化动量优化器。BCAT 使用 Muon 的理论基础。

## 22. RL 搜索预训练相关补录 [idea-reviewer, 2026-06-24]

- **FormulaGPT (2404.06330)**: Yanjie Li, Weijun Li, Lina Yu 等 (CAS 半导体所), 2024.04. 将 DSR/DSO 的 RL 搜索历史（表达式序列+R^2 奖励序列）收集为 1.5M 训练数据，训练 SetTransformer encoder + Transformer decoder 做上下文内策略更新。推理时给新数据 [X,y] 自回归生成 RL 搜索过程。在 SRBench 上 R^2 综合 SOTA，噪声鲁棒性优于 NeSymReS/SNIP，通用性优于纯预训练模型。核心机制：将 RL 蒸馏为 Transformer 的 in-context learning，而非传统意义的 controller 权重预训练+RL 微调。面向一般 SR 而非 PDE 求解。3 citations。

- **IGEP Transfer Learning for GEP SR (2406.05166)** [idea-reviewer, 2026-06-25]: M. Reissmann, Yuan Fang, Andrew S. H. Ooi, R. Sandberg, 2024.06 (arXiv) / 2025 Physics of Fluids. 首次将 transfer learning 系统化引入符号回归进化搜索 (GEP)。核心机制：从先前已优化问题上收集表达式，训练 NLP 语言模型识别其中反复出现的模式/子结构，用训练好的语言模型生成更优的初始候选解。在 CFD 湍流建模场景 (弯管流) 验证：从简单建筑块流 (canonical duct + periodic hill) 预训练，迁移到复杂三维弯管流，相比随机初始化提升收敛速度和解质量。4 citations。特点：(a) 面向 GEP 进化搜索而非 RL policy gradient, (b) warm-start 的是候选表达式种群而非 controller 网络权重, (c) 面向 turbulence closure 而非 PDE 解析解搜索。

## 23. FEX-PROSE 两个连通分支之间的桥梁分析 [idea-creator, 2026-06-24]

### 23.1 两个连通分支的结构性对比

学术文献网络中, 以 Haizhao Yang 的 FEX 为中心和以 Hayden Schaeffer 的 PROSE 为中心存在两个几乎不相连的连通分支。两组研究人员在做高度相关的事情 (符号+数值 PDE 求解), 但使用不同方法论、不同术语体系、不同评测标准, 且几乎不互引 (仅在 LLM+FEX 的 2503.09986 中引用了 PROSE 作为背景)。

| 维度 | FEX (Yang 组) | PROSE (Schaeffer 组) |
|------|-------------|---------------------|
| 核心方法 | RL policy gradient 搜索表达式树 | Transformer 多模态编码+解码 |
| 表示 | 二叉树 (一元/二元算子节点) | Polish notation token 序列 |
| 输出 | 精确符号表达式 (~1e-7 精度) | 数值解 (~1-3% 误差) + 低精度符号 |
| 理论保证 | S_k 无维度灾难 (Holder 函数) | MNO 万能逼近+scaling law |
| 搜索策略 | 每个 PDE 从零 RL 搜索 | 多 PDE 族预训练, zero-shot 迁移 |
| 维度 | 高维 (d=100) | 低维 (1D-2D) |
| 系数优化 | Adam + BFGS 两步精确优化 | Transformer 隐式学习 |
| PDE 残差 | 作为 RL 奖励信号 | PI-MFM: 作为训练损失 |
| 术语 | controller, action space, reward | encoder, decoder, fusion, operator |

### 23.2 桥梁机会

两个连通分支的互补性极强: FEX 的精确符号搜索 + PROSE 的跨 PDE 表示学习。核心未被探索的桥梁方向:

1. **PROSE 预训练表示 -> FEX 搜索先验**: PROSE 的 fusion layer 已经学到了跨 PDE 族的数值-符号联合表示。这些表示能否作为 FEX controller 的条件输入或 warm-start, 替代 FEX 从零搜索?

2. **FEX 精确解 -> PROSE 训练数据质量**: FEX 能在某些 PDE 上找到机器精度的解析解。这些精确解能否作为 PROSE 的高质量训练/验证标签, 替代数值求解器的低精度近似?

3. **S_k 理论 + MNO 理论的统一**: FEX 的 S_k 逼近论证明有限表达式无维度灾难, MNO 的 scaling law 证明多任务共享不增额外代价。两者能否在同一框架下统一, 给出"符号+数值联合求解"的理论保证?

4. **PI-MFM 残差计算 -> FEX 奖励信号**: PI-MFM 实现了从 Polish notation 自动解析表达式树并向量化计算 PDE 残差的管线。这与 FEX 的 RL 奖励函数 (PDE 残差最小化) 异曲同工但技术路线不同。统一两者的残差计算可能揭示新的优化路径。

5. **PROSE Skeleton 模式 -> FEX 搜索空间约束**: PROSE-PDE 已证明仅知骨架 (不含系数) 即可达 1.06% 数据误差。如果 PROSE 能为 FEX 提供高质量骨架候选, FEX 只需做系数优化 (Adam+BFGS), 搜索空间从指数级降到连续优化。

6. **表达式树 vs Polish notation 的表示等价性**: 两者在数学上同构, 但在计算上有不同的归纳偏置。这种等价性是否意味着 FEX 的 RL 搜索和 PROSE 的 Transformer 生成在某种意义上是同一搜索问题的对偶视角?

### 23.3 当前无人探索的确认

截至 2026-06-24, 在 arXiv, Google Scholar, Semantic Scholar 上搜索 "FEX" + "PROSE", "Haizhao Yang" + "Hayden Schaeffer", "finite expression method" + "multi-operator learning" 等组合, 均未找到任何连接两个连通分支的论文。唯一的交叉引用是 LLM+FEX (2503.09986) 在背景部分提及 PROSE, 但未做任何技术整合。SymPlex (2602.03816) 同时引用了 FEX 和 PROSE 作为 baseline, 但 SymPlex 本身是独立的第三条路线, 并未架起 FEX-PROSE 之间的桥梁。

## 24. 标签保真度 / 训练数据来源相关补录 [idea-reviewer, 2026-06-24]

以下两篇与 "neural PDE solver 的训练标签精度/来源" 主题直接相关, 是评估 "用更精确符号解标签训练 neural operator" 类 idea 时的关键 prior。

- **Neural Emulator Superiority (2510.23111)**: Felix Koehler, Nils Thuerey (TUM), 2025.10, NeurIPS 2025. 核心反直觉发现: 用低保真数值求解器数据训练的 neural emulator, 在对照高保真 reference 评测时, 精度可以**超过其训练标签的保真度** (emulator superiority)。机制: 网络的归纳偏置对动力学起正则化作用, 隐式学到比训练数据更物理、误差累积更友好的解 (如 Burgers 激波传播: 粗求解器未充分解析激波, 但 UNet emulator 学到更接近高保真解的激波传播)。结果高度 regime-dependent (依赖具体 PDE、低保真求解器误差特性、架构、训练目标、评测指标、时间跨度)。同时警告: 用数值模拟器数据当 ground truth 的 benchmark 会不公平地惩罚正确学到物理的 emulator。**对本课题的意义**: "降低标签噪声 ⇒ 降低预测误差" 这一朴素单调假设正是此文质疑的对象; 任何主张"更精确标签必然改善 neural operator 训练"的 idea 必须正面回应此结果。

- **PISR — Physics-Informed Symbolic Regression for PDEs** (Neural Computing and Applications 2025, springer 10.1007/s00521-025-11450-9): PINN + 符号回归 (GP) 混合, 用 PINN 预测解作为 SR 输入数据推导解析近似解。报告 BIC-based PISR 在 Laplace 方程上恢复**精确解**, Burgers R²=0.998, 非线性波 R²=0.957。范式方向为 NN → 符号 (与 SymTorch 2602.21307 同向), 即从已训练 NN 蒸馏符号解, 而非用符号解反过来训练 NN。**撞车风险: 低** (工具层与方向均与"符号解→neural operator 训练标签"相反)。

## 25. 给定结构下系数优化的凸性 / 可辨识性相关补录 [idea-reviewer, 2026-06-24]

以下是分析 "给定符号骨架 (算子拓扑固定) 后, 连续系数优化问题的优化景观 (凸性 / PL / 条件数)" 这一主题的关键 prior。该主题横跨 separable nonlinear least squares、PDE 系数反问题、SR 常数优化三条经典文献线。

- **Variable Projection / Separable Nonlinear Least Squares (Golub & Pereyra 2003, Inverse Problems 19:R1-R26; Dong & Yang, CMAME 2022, arXiv:2201.09989)**: 经典框架, 处理"模型 = 非线性基函数的线性组合" min ‖y − Φ(α)a‖² 形式的问题。核心结构性事实: 线性系数 a 与非线性参数 α 可分离, 消去线性部分得到只含 α 的 reduced problem, 条件更好收敛更快。直接对应"骨架对系数线性 ⟹ 问题可分/凸, 非线性 ⟹ 不可分"。Dong & Yang 2022 把 VarPro 用于 PDE: 线性 PDE + 线性输出层 ⟹ separable (线性部分闭式解); 非线性 PDE/BC ⟹ 全部参数变非线性, 需 Newton-VarPro。**与 FEX 的关系**: FEX 的系数优化本质就是 separable NLS——一元/二元算子节点的频率/尺度参数是非线性参数, 顶层组合系数是线性参数。任何"给定骨架后系数优化凸性"的分析都必须建立在 VarPro 这一已有框架上, 并说明 delta 相对 VarPro 已知结论 (线性部分平凡凸, 非线性部分一般非凸) 的增量。

- **Elliptic coefficient identification via convex energy functional (Knowles-type convex functional; Harrach 2021, arXiv:2105.11440 convex SDP reformulation)**: PDE 系数反问题文献中已有大量工作证明: 对线性椭圆 PDE 的系数辨识, 可构造凸能量泛函 (无需系数正则性即得 well-posedness), 或重写为唯一可解的凸非线性半定规划。共识: 线性方程 + 系数线性表示 ⟹ 残差最小二乘严格凸; 非线性系数反问题一般非凸且依赖好初值。

- **SR 常数优化 benchmark (arXiv:2412.02126; Current Challenges of SR, arXiv:2512.01682; Kommenda et al. 2019 GPEM)**: SR 社区共识——给定表达式结构后的常数 (系数) 优化普遍非凸、依赖初值、问题相关无统一最优方法; Kommenda 用 Levenberg-Marquardt + autodiff 做 local search 并明确给出成功/失败例子。**SAGE-Fit (2605.23272)** 进一步命名了 "Good Structure, Bad Score" 现象——正确结构因系数拟合不充分而得低分, 根因是非线性算子使内层高度非凸; SAGE-Fit 为纯算法改进 (multi-start/staged), 无凸性/PL 定理。

## 26. RL 符号搜索的 value function / baseline / 部分树估值相关补录 [idea-reviewer, 2026-06-25]

以下是分析 "为 RL 符号表达式树搜索引入 value function / critic / baseline 降低 policy gradient 方差, 或对部分(未完成)表达式树估值" 这一主题的关键 prior。该主题是任何"给 FEX 加 critic / dense reward shaping"类 idea 的核心 prior 线。注: FEX 原始 controller 是固定拓扑树上的单步联合采样 (contextual bandit, 单次 forward 吐出所有节点算子分布), 而非逐 token 的 sequential MDP——这一结构差异决定了许多面向 autoregressive 序列生成的 baseline/critic 技术不能直接照搬。

- **DSR / DSO (Petersen et al., ICLR 2021, arXiv:1912.04871; DSO 综述 arXiv:2505.10762)**: RL 符号回归的奠基工作, 自回归生成表达式 token 序列。显式讨论 baseline 作为 control variate 降低 REINFORCE 方差, 并把"value function 估计"列为标准 baseline 选择之一; 招牌贡献 risk-seeking policy gradient (RSPG) 本身就是一种 (1−ε)-quantile baseline 形式的方差削减/聚焦机制。"用 baseline/critic 给符号树 PG 降方差"在此已是标准机械。

- **QuantFactor REINFORCE (Zhao et al., arXiv:2409.05144)**: 在符号表达式树 (formulaic alpha, 算术算子树, 结构与 FEX 同构) 上做 RL, 提出方差有上界的 greedy baseline 并给出 REINFORCE 方差的显式上界, 明确讨论丢弃 vs 保留 critic/value network。是"baseline/critic 降方差"在 SR 兄弟领域的直接实例。

- **SR-GPT (Li et al., arXiv:2401.14424)**: MCTS + 策略-价值网络做 SR, 网络输出 (p, v) = N_θ(s), 其中 state value v 被逐字描述为"从当前节点继续向下选择最终能达到好结果的程度"——即从**部分**表达式树预测最终质量, 是 learned partial-tree value 的直接先例。

- **TPSR (Shojaee et al., NeurIPS 2023, arXiv:2303.06833) / DGSR-MCTS (arXiv:2302.11223)**: 用 MCTS 引导预训练 Transformer 的解码, 对部分树估值。关键区分: 这两者用 **beam-search / MC rollout 补全**来估部分树价值并算真实 reward, 而非 learned-from-data 的 critic。因此"用 learned critic 直接预测最终残差、替代 rollout、且在 PDE 场景"是相对更窄、占据更少的格子, 但"部分树估值"概念本身已被 MCTS-for-SR 一族占据。

- **程序合成的 partial-program value head (Write-Execute-Assess, Ellis et al., NeurIPS 2019, arXiv:1906.04604; Neural Program Synthesis by Self-Learning, arXiv:1910.05865)**: 训练 value function 评估"已写出的部分程序的前景", 预测从部分状态出发的最终 (折扣) 结果以避免 rollout。AlphaZero 式 value head 按构造即做此事。说明"从部分语法树预测最终质量"是程序合成/搜索的成熟范式。

- **PDE foundation model + reward / RL (Towards Reasoning for PDE Foundation Models, arXiv:2509.02846)**: reward-model 驱动的 inference-time scaling 用于 PDE 基础模型, 明确把"构建基于 RL 的方法"列为 future work 并指出 PDE 的 local reward signal 难构造。是"PDE 基础模型 + reward/RL"最近的相邻线, 但不涉及符号搜索 / FEX / value estimation。

## 22. 固定基系数 operator learning / 符号-basis decoder [idea-reviewer, 2026-06-25]

以下论文由 idea-reviewer 在审 idea 期间经 web (arXiv/Scholar/S2) 检索发现, 本仓尚无 wiki, 仅据 abstract/标题摘要登记, **未经 deep-lit 全文核验**。它们界定了 "neural operator 的 decoder 输出固定基系数而非逐点场值" 这一已成熟的范式, 是 0616-fex 中任何 "用符号/逼近论函数类约束 operator 输出层" 方向的直接 prior。

- **FB-C2CNet — Learning Operators through Coefficient Mappings in Fixed Basis Spaces (arXiv:2510.10350, 2025.10)**: 明确提出 "fixed-basis coefficient-to-coefficient (FB-C2C) operator learning" 范式——operator 的输入输出均表示为固定基 (random features / FEM, 并论及 Chebyshev/Legendre/RBF/wavelet) 的展开系数, 网络学系数→系数的映射而非节点值。已在高维 Poisson 上测试, 论证 random-feature 基比 FEM 基更能缓解 CoD。是 "固定基系数 decoder + 高维 PDE" 这一格子最接近、占据最多的工作。

- **Spectral Neural Operators (SNO, arXiv:2205.10573)**: 将有限组输入 Chebyshev/Fourier 系数映射到输出系数的 neural operator, 即固定谱基的系数 decoder。

- **FERN (arXiv:2510.26962) / Finite Operator Learning (FOL)**: decoder 预测 (可训练或固定) FEM 基的系数, 输出函数类被 "硬连线" 到给定基。

- **SEDONet (arXiv:2512.09165)**: Chebyshev-dictionary DeepONet——以 Chebyshev 字典为 trunk 的 DeepONet 变体。

- **MoE Softens the Curse of Dimensionality in Operator Learning (arXiv:2404.09101)**: 用 Mixture-of-Experts + 叶节点基展开 (Fourier / 分片多项式) 在理论上缓解 operator learning 的 CoD。与 Deep Operator Learning Lessens the Curse of Dimensionality (arXiv:2301.12227) 同属 "neural operator 借低维结构 / 基展开逃 CoD" 的理论线。
