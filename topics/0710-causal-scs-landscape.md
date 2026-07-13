# Landscape: 因果效应不稳定性作为强对流预警指标

<!-- 本文件由 idea-reviewer 在审查 causal-scs-indicator idea 时创建 (原 idea-creator 阶段被跳过, 未生成 landscape). 以下条目仅客观描述相关论文做了什么, 不涉及与具体 idea 的关联判断. Append-only, 后续贡献者请勿删改已有条目. -->

## [idea-reviewer, 2026-07-10]

- **Bian, Wang, Leng, Lin, Shi (2025), "Utilizing Causal Network Markers to Identify Tipping Points ahead of Critical Transition"** (arXiv:2412.16235, Advanced Science). 提出 causal network marker (CNM) 框架: 用 Granger causality (CNM-GC) 和 transfer entropy (CNM-TE) 量化节点间方向性因果强度, 用 K-means 将节点分为 dominant group (DG) / non-dominant group (NDG). 理论证明 (线性化系统 + 特征值分解) 当系统趋近 codimension-one 分岔点时, DG→NDG 方向的 Granger causality 强度趋于 0, 而 marker CNM = |DG||NDG| / Σcs(DG→NDG) 相应发散, 以此作为早期预警信号. 在五基因调控网络、生态互惠网络、Turing 反应扩散网络三类 benchmark 及癫痫发作 iEEG 真实数据上验证, 预测精度优于传统 DNB (dynamical network biomarker, 基于方差/自相关). 机制是"真实因果强度随分岔确定性地趋于 0/发散", 而非"因果估计过程本身在假设违背下变得不稳定".

- **NPG (2026), "Quantitative Comparison of Causal Inference Methods for Climate Tipping Points"** (npg.copernicus.org/articles/33/313/2026/, 原预印本 EGUsphere 2025-6258, 未查到独立 arXiv ID). 比较 PCMCI、LKIF (Liang-Kleeman Information Flow)、GCSS (Granger Causality for State Space Models) 三种方法在气候临界点场景 (AMOC-北极夏季海冰范围交互) 上的表现, 数据来自三次非线性随机微分方程生成的分岔动力学合成序列, 用 Matthews Correlation Coefficient 与 ground truth 比较. 核心发现: 方法表现依数据特征分化明显 (LKIF 适合小样本弱耦合, GCSS 适合大样本强信号, PCMCI 中庸但灵活), 且**当分岔事件落在被分析窗口内时, 所有方法都可靠地失效** ("all methods fail reliably when tipping events occur within analyzed data"). 该发现将跨方法分歧/失效视为需要规避的局限性, 而非可利用的预警信号本身.

- **Ruiz, Arana-Catania, Ardila, Ventura (2026), "Causal-Audit: A Framework for Risk Assessment of Assumption Violations in Time-Series Causal Discovery"** (arXiv:2604.02488). 提出 Causal-Audit 框架, 对五类假设族 (stationarity, irregularity, persistence, nonlinearity, confounding proxies) 计算 effect-size 诊断量, 聚合为四个带不确定区间的校准风险分数, 并用 abstention-aware 决策策略, 仅在证据支持可靠推断时推荐使用 PCMCI+ / VAR-based Granger causality. 在 500 个合成 DGP (10 类假设违背) 上校准 (AUROC>0.95), 21 个外部 benchmark (TimeGraph/CausalTime) 上一致. 该框架把"假设违背程度"当作决定是否信任/弃权因果推断结果的风险指标, 而非当作物理系统状态转换的预警信号.

- **Mameche, Cornanguer, Ninad, Vreeken (2025), "SpaceTime: Causal Discovery from Non-Stationary Time Series"** (arXiv:2501.10235). 统一时序因果图发现、regime changepoint 重建、跨空间/时间不变因果关系分区三个任务, 用 Minimum Description Length 构造一致性得分, 在河流径流与生物圈-大气交互真实数据上验证. 提供 regime changepoint 检测能力, 但不涉及"用因果关系不稳定性预测未来临界事件"的框架。

## [idea-reviewer via codex second opinion, 2026-07-10]

- **Yu, Liang (2026), "SpatioTemporal Causal Network Diagnostics for Geographic Tipping Point Early Warning"** (arXiv:2606.17553). 提出 ST-CND 框架: 用 transfer entropy 推断空间节点间信息流拓扑 (替代固定欧氏邻域), 用 dynamic mode decomposition 估计每个候选子网络的局部恢复率, 再结合"高内部波动 + 高内部同步 + 低外部耦合"三signal 识别最脆弱子网络, 以抑制空间相关噪声导致的假警报. 在合成分岔数据及 Indo-Pacific SST / North Atlantic AMOC 两个真实观测 benchmark 上验证, AMOC 任务 AUROC 0.783, 关键子网络 IoU 0.378, 优于 recurrence-network 和 lambda-AR1 基线. 该工作把"因果网络拓扑 + 局部恢复率"组合为空间早期预警指标, 应用于地理尺度的临界点 (生态系统/气候子系统/冰盖), 属于因果网络诊断类早期预警框架谱系中与本 topic 邻近的又一新近工作 (已通过 arxiv-tools 核验存在, 摘要与标题一致).

## [deep-lit-tick --scope topic, 2026-07-10]

<!-- 本轮 deep-lit-tick 精读 24 篇 (4 轮, B4 饱和: 第 4 轮新论文 2510.19138 的 top_related 全部落入已读集). reader=claude(opus-4-8; 配置的 deepseek CLI claude-ds 本机未安装, 按 env 规则 fallback 到同接口的 claude). 每篇均有全文 wiki: $ARXIV_WIKI_DIR/<id>.md 含 "Read by: 0710-causal-scs". 下按角色分组, 撞车风险 = 对 idea causal-scs-indicator 核心 claim 的威胁度. 2412.16235/2604.02488/2501.10235/2606.17553 上文已有条目, 此处为全文精读后的补充. -->

### A. 确定性机制早期预警 (最接近的 prior, 必引且必须机制区分)

- **Bian et al. (2024), CNM** (arXiv:2412.16235, R-Score B/15). 全文精读补充: 理论建立在**不动点线性化 + Jacobian 对角化 + 慢变近平衡**假设上, 证明趋近分岔 (max eigenvalue→1) 时 DG→NDG 因果强度**确定性**趋 0、marker 发散; 全部案例慢变. **撞车 HIGH 但机制可分**: CNM=真实因果强度确定性变化 (动力学机制); 本 idea=估计过程在假设违背下的统计脆弱性 (认识论机制). 可复用 DG/NDG K-means 分组、CNM-GC/CNM-TE 作确定性机制基线、Turing 121 格点空间基准.
- **Yu & Liang (2026), ST-CND** (arXiv:2606.17553, R-Score B/15). 全文精读补充: TE(KSG)+IAAFT surrogate+BH-FDR 重建有向拓扑 → Graph-DMD 估领头衰减率 → DNSD 三信号选最脆弱子网. **DNSD 与 CNM 的 DNB 是同一比值形式**, 与 CNM 并列两根 related-work 支柱; 两轴正交—ST-CND 主动 surrogate/FDR **压制** TE 噪声, 本 idea 把估计不稳定性**当信号**. 全在慢变气候域, **明列 PCMCI 为 future work**. 可复用 IAAFT+BH-FDR 显著性协议 (正好证伪"不稳定性是否伪影"objection).

### B. 快变强迫 / 尺度错配下 EWS 失效 (动机弹药, 也是最大威胁)

- **Ritchie, Kachhara, Ashwin (2026), 几何 R-tipping 预警** (arXiv:2605.16128, R-Score A/21). 证明**经典 CSD (方差/自相关/回复率) 在快速强迫下 AUC≈0.5 失效**, 提出到 R-tipping 阈值的带符号距离. 撞车 LOW (几何/model-based). 双刃: 是"经典 EWS 在快变失效"的动机弹药, 又要求本 idea 证 fragility 优于 R-tipping. 局限: 需已知 ODE+未来强迫剖面 (本 idea 数据驱动是优势). 可复用 ROC/AUC + optimal-vs-fixed-threshold 评测法.
- **Ashwin, Bastiaansen et al. (2025), accelerating cascades** (arXiv:2506.01981, R-Score A/20). **本轮对 idea 威胁最大**: 定量证明经典 CSD-EWS 技能≈随机 (AUC≈0.5) 于**"加速级联"(慢上游驱动快下游, 尤其 downstream-within-upstream)**—正是强对流 (快中尺度对流被慢天气尺度环境驱动) 的 regime. 把 reviewer 的"快变 vs 慢变尺度错配"**从担忧升级为有反例有机制的定量结论**. 给外推失效四因子清单 (fragility 须排除的混淆项). 可复用有限时域 ROC/AUC ensemble 框架、耦合一维 bistable 上下游模型 (比耦合 Lorenz 更干净的 Stage-1 testbed). **idea 下一版必须正面回应.**

### C. 稳健因果变点检测 (idea "突变型指标" 分支的 SOTA baseline, 必须超越)

- **Gao, Kocaoglu et al. (2025 AISTATS), Causal-RuLSIF CPD** (arXiv:2407.07290, R-Score A/20). PCMCI 父集切 IID 段 + 动态 RuLSIF 散度峰值. 检测**因果机制突变**而非联合分布漂移. soft/hard change 对应"条件分布变/DAG 边变"两类指标. PE 散度序列 = 现成连续机制散度信号. 撞车 MEDIUM (取向相反). 局限: 离散/二值、瞬时突变、须无隐混杂.
- **Gavioli-Akilagun, Wood, Quinzan (2026), kernel/copula 因果变点** (arXiv:2605.05809, R-Score A/18). Q 统计量**刻意设计为对 confounder/非平稳漂移不变**—正好过滤掉本 idea 想利用的 fragility. **撞车 MEDIUM-HIGH: idea 必须超越的 clean baseline**. 高价值副产品: **25 个 null 的负对照分类学 (NCL/NIV/NMD/NNS/NCF/NPO) = codex review 要求的"匹配非平稳但不 tipping 负对照"现成模板**.
- **MEND (arXiv:2505.12023, R-Score B/13)**. model-X CRT + NN 蒸馏 + fused-lasso 检测 p(Y|X) 变点, Type-I 精确控制. 撞车 LOW. 可用: 优于 PELT/BOCPD/CUSUM; **model-X CRT 重采样 = 区分真实变化 vs 估计伪影的 null/surrogate 校准模板**.
- **Huang, Peters, Pfister (2024), Causal-CCP** (arXiv:2403.12677, R-Score A/19). 用 invariance 定义"因果变点": 仅 Y|X 因果机制改变才触发. causal stability loss = 逐时刻连续指标. **撞车 HIGH (线性/offline/iid 下已占据"因果结构变化指示器")**. 差异化: 真实数据/非线性/在线/隐混淆鲁棒/时间依赖. 最直接 baseline.

### D. 课题核心因果方法 (基础设施/工具, 无 novelty 撞车; idea v1 全部未引, 须补)

- **Runge (2020 UAI), PCMCI+** (arXiv:2003.03685, R-Score A/21). 核心工具定义源. **自相关对 PCMCI+ 是助力**. 假设**因果充分** (不处理隐混杂). 可复用 tigramite、(a,N,T,tau_max) 扫描作 Stage-1 分歧基线、effect-size 定理为 fragility score 提供解析动机.
- **Günther, Ninad, Runge (2023 UAI), J-PCMCI+** (arXiv:2306.12896, R-Score A/20). 多数据集 + 隐 context 混杂. idea 点名方法, 适配 Stage-2 多个例池化. **关键: 固定小 T、增大 M 时 FPR 膨胀不收敛 (Nickell 有限样本偏差)—把 idea 最强反驳"不稳定=有限样本伪影"具体化**, 为必建的 matched 负对照/样本量校正基线提供理论依据.
- **Gerhardus & Runge (2020 NeurIPS), LPCMCI** (arXiv:2007.01884). 补 PCMCI+ 隐混杂短板, Stage-2 候选. **effect size = min|ParCorr| = within-method 标量 fragility 度量** (回应 reviewer"跨方法量纲不可比"批评). interaction-info effect-size 定理给"自相关↑→effect size↓→CI 失效"解析机制 (平稳假设下, 非平稳需补).
- **Liang (2021), 多元 LKIF** (arXiv:2104.11360). 仅样本协方差的线性 ML 估计器 + 归一化 tau + Fisher 显著性. 抗噪声/抗同步/抗混杂, 极低算力. LKIF 腿的必引基础. 反引文 **NPG(2026) 已发现分岔落窗口内时 LKIF/PCMCI/GCSS 全部失效**—正是 idea 想反用的现象.
- **Lien (2024), LKIF+LIM** (arXiv:2409.06797). 用 LIM 稳健估计 A 再算信息流. 实证**白 vs 有色噪声假设给出符号/大小都不同的因果 = "因果估计对假设敏感"的直接证据**. Caveat: LIM 稳态+线性+月尺度与快变中尺度错配.
- **Colored-LIM (arXiv:2402.15184, R-Score A/17)**. **A 对 lag ρ 敏感 (作者当 nuisance 去修, 本 idea 反用作信号 = 现成差异化支点)**. 可作 sliding-window 线性动力估计器, 追踪 A/τ/特征值漂移 = 不稳定性指标. SIS+colored+分岔仿真 = Stage-1 testbed.

### E. 不稳定性量化 / 稳定化工具 (哲学对立面: 把不稳定当噪声平均掉)

- **Debeire et al. (2024 CLeaR), Bagged-PCMCI+** (arXiv:2306.08946, R-Score A/19). 保 lag 的 bootstrap 套 PCMCI+ → **per-link bootstrap 频率 = 现成不稳定性量化装置** (tigramite 可直出). 陷阱: 置信度理论建立在平稳性上、40-60% 区间正偏 → 非平稳滑窗须用负对照排除该估计器固有偏差.
- **Wu et al. (2026), DAGgr** (arXiv:2605.18633, R-Score A/17). **边重要性分数 s_ij = 把 PCMCI+ (CI 统计量) vs LKIF (信息流) 压到共同 [0,1] 尺度的加权边频率 = 课题缺失的"跨方法分歧度量"候选定义** (解决量纲不可比); 相邻窗口 s_ij 变化 = 结构不稳定指标. 是姐妹子话题 Causal Discovery Uncertainty 正中靶心.
- **Ruiz et al. (2026), Causal-Audit** (arXiv:2604.02488, R-Score A/19). 全文精读补充: 三阶段流水线把违背算成四维校准风险分 (AUROC>0.95) 用于弃权. **哲学对立面: 证明违背诱导的不稳定是"可校准噪声"→ idea 必须新增 baseline, 证 fragility 指标对 SCS 预测力超出其四维风险分所能解释**. 可复用 AssumptionAuditor 作滑窗诊断引擎.
- **hidden-confounding via graph instability (arXiv:2606.01214, R-Score B)**. 同一问题空间 (给潜混杂诊断指标), 信号不同 (图不稳定 D_p vs 因果强度幅值). 撞车 MEDIUM. 提供**写全但未执行的 surrogate-null bootstrap 校准协议** (order-p Markov null + moving-block bootstrap → p̂_global/BH-FDR)—idea 可搬这套 recipe 把指标升级为"有校准的假设检验".
- **Zhang et al. (2026 ICLR sub), InvarGC** (arXiv:2510.19138, R-Score B/13). 跨环境不变性处理潜混杂+未知干预, 把机制漂移正则化掉求稳. **是 idea strongest objection 的活体证据: shift-aware 方法能干净吸收干预+混杂 → skeptic 会说不稳定只是 misspecification; idea 须证信号在 shift-aware 方法后仍存活, 或强对流违背其 A2/A3**. 其 top_related 全部落入本轮已读集 = B4 饱和信号.
- **Mameche et al. (2025 AAAI), SpaceTime** (arXiv:2501.10235, R-Score A/21). 全文精读补充: MDL score-based, GP残差→PELT 变点 + 核检验判机制差异. "正确建模非平稳"阵营. 可复用 MDL δE 作第 4 种估计器加入跨方法分歧; GP残差→PELT 实现突变型指标; FLUXNET 检测 2003 热浪=慢变尺度先例 (暴露快/慢错配).
- **CausalDynamics (arXiv:2505.16620, R-Score S/23)**. 14693 真值因果图 + 即插即用引擎, 评 10 SOTA. **Stage-1 合成引擎 (耦合混沌+隐混杂+噪声+time-lag+真值图, 但无参数漂移/分岔穿越需自加); 10 方法同数据分歧 = 现成"跨方法固有分歧基线"**. 双刃: 证明这些方法在**静态平稳混淆下就已近随机**, 加重"不稳定升高只是估计噪声"的最强反驳 (与 NPG 同源).

### F. 空间 EWS 区分转变类型

- **Sanders & Bastiaansen (2025)** (arXiv:2510.01959, R-Score A/21). 从转变前噪声时空数据拟合线性 reaction-diffusion PDE (SINDy), 用 dispersion relation λ(k)+主导波数 k* 区分空间齐次 tipping (k*=0) vs Turing (k*≠0), 解决经典 CSD 的 ambiguity. **撞车 MEDIUM-HIGH (若 idea 主打"区分转变类型"则直接撞, 必引)**; 机制不同 (谱/算子回归). 可复用 Klausmeier benchmark (解析 ground-truth SN/Turing/重合三 case).

### G. 强对流应用 (撞车检查: 均非因果 → 立靶/动机)

- **Ganesh, Beucler, DeMaria, Runge (2025), Multidata-PC SHIPS** (arXiv:2510.02050, R-Score S/24). 多数据 PCMCI 从 SHIPS 选因果预报因子提升 TC 强度预报. **撞车 LOW 且正交 (求稳路线). 核心可复用: pc_alpha 扫描 × 7 折的折间计频 = 现成的不稳定性度量装置 (本文阈值化求稳, 我们读取波动本身)**; 多数据范式可迁到多 SCS 个例.
- **FuXi-Nowcast (arXiv:2512.08974)**. 环境→严重对流最强 DL nowcasting, **主动声明"这不是 physical/causal attribution"**—最有价值的立靶抓手 (撞车不成立: 它做预报系统, 我们做因果指标). 可复用候选因果变量池 (CAPE/CIN/TPW/wind divergence/bulk shear/dewpoint 梯度/θe)、CSI+neighborhood maxpool 验证协议、dryline CI/squall-line 个例. 动机: 引其自述局限论证"SCS 环境驱动的因果归因仍是开放问题, ablation-sensitivity ≠ causal".

## [deep-lit-tick --scope idea causal-scs-indicator, 2026-07-11]

<!-- 本轮 idea-scope deep-lit-tick 精读 26 篇 (4 轮, B4 饱和), 全部在 topic-level 24 篇之外; 由 dispatcher 汇总并入本文件 (idea-scope 规则不直接写 landscape). 每篇均有全文 wiki, 分组延续 idea 报告的"三支柱"结构. 详见 ideas/causal-scs-indicator-deep-lit-report-2026-07-11.md 的完整撞车矩阵. -->

### H. 机制区分理论支撑 (支柱 a: "统计脆弱性 ≠ 确定性因果强度变化" 的困难程度)

- **Shah & Peters (2020 Annals of Statistics), GCM / No-Free-Lunch** (arXiv:1804.07203). 证明**任何非参数条件独立性检验在假设违背下都不可能同时保持 well-calibrated 且有功效** (Theorem NFL) —— 给"因果估计器在假设违背下天然不稳定"提供第一性原理依据, 而非本 idea 的临时假设. 撞车 LOW (纯理论工具), 但是三篇理论支撑之一, proposal 必引.
- **Rabel & Runge (2025), HCCD context-specific 因果图发现** (arXiv:2511.21537). **Impossibility Result B**: 在无观测 context 下, "因果强度变化"与"检验统计量固有不稳定"在一般条件下不可判别分离. 把本 idea"机制区分"支柱从经验断言升级为已知困难问题, proposal 需正面承认此边界而非回避. 撞车 LOW-MEDIUM (理论工具非直接竞品).
- **He, Pogodin, Li, Deka, Gretton, Sutherland (2025), Hardness of CI Testing in Practice** (arXiv:2512.14000). **Type-I 误差膨胀定理**: 有限样本下 CI 检验的名义显著性水平系统性失真, 为"观测到的不稳定性是回归/检验伪影"这一竞争解释提供精确的排除工具 (可反过来证明信号不是纯伪影). 撞车 LOW.
- **Bergen, Sejdinovic, Didelez (2026), GKCM (Generalised Kernel Covariance Measure)** (arXiv:2604.03721). 联合嵌入版 GCM 的 Type-I 膨胀分析, 补齐"有限样本伪影"证据链第 4 种机制. 撞车 LOW.
- **Faller, Vankadara, Mastakouri, Locatello, Janzing (2023), Self-Compatibility** (arXiv:2307.09552). 提出 self-compatibility 作为无 ground-truth 时评估因果发现的框架 —— **本 idea"估计脆弱性"概念的形式化语言来源**, 也是 novelty check 最初标记的最大风险点 (若已扩展到时序场景则直接撞车). 撞车 LOW-MEDIUM.
- **Jahn & Janzing (2026), Evaluating Bivariate Causal Statements via Mutual Compatibility** (arXiv:2606.00278). 将 self-compatibility 扩展到双变量因果陈述评估, **仍限于 iid/横截面场景, 未触及时序/滑动窗口**——独立确认 novelty 最大风险点未被占据. 撞车 LOW.
- **Schkoda, Faller, Blöbaum, Janzing (2024), Leave-One-Variable-Out (LOVO) 交叉验证** (arXiv:2411.05625). 提供另一个 self-compatibility 谱系的自评指标 (LOVO cross-validation error), **第二篇独立确认该谱系未扩展到时序**, 同时给出候选 fragility 自评量. 撞车 LOW-MEDIUM.
- **Nanavati, Vreeken, Kaltenpoth (2026), StruBI (Structural Bias Identification)** (arXiv:2606.18834). 用 average markov interventions (AMI) + 祖先覆盖率区分混杂偏差与选择偏差来源. 可作机制区分 (confounding-driven vs selection-driven instability) 的候选判据. 撞车 LOW-MEDIUM.

### I. 快慢尺度错配新增混淆源 (支柱 b: 威胁加剧, 须新增负对照)

- **Ritchie, Bastiaansen, von der Heydt, Ashwin (2025), coupling and timescales for interacting tipping elements** (arXiv:2509.03996). 揭示 **UaDB (unequal-and-different-basins) regime 下, 几何/时间尺度本身即可导致因果方向系统性误判**——独立于任何统计噪声的第 5 种混淆渠道, 是 arXiv:2506.01981 (topic-level 已读, "本轮威胁最大") 的姊妹篇. Proposal 必须新增此负对照. 撞车 LOW (机制威胁而非竞品占位).
- **Li, Zhu, Zhao, Zhao, Zhang, Duan, Lin (2026), RCDyM (Reservoir Computing + Dynamical Measures)** (arXiv:2603.14944). 目前最全面的非因果 EWS baseline (整合多种动力学指标 + reservoir computing 超前预测). 其局限本身印证: **即便最前沿的非因果方法也未解决快慢尺度错配问题**, 侧面支持本 idea 仍有空间, 但也是须超越的强 baseline 之一. 撞车 LOW.
- **Masuda (2026), Detecting tipping points from sample variance alone** (arXiv:2602.10817, TIPMOC). 纯方差幂律标度检测分岔, 提供第 5 个稳健经典 EWS baseline, 附带**无分岔负对照设计模板**可直接复用. 撞车 LOW.
- **Montagna, Mastakouri, Eulig, Noceti, Rosasco, Janzing, Aragam, Locatello (2023), score-matching 假设违背鲁棒性 benchmark** (arXiv:2310.13387). 系统评测多种假设违背下 score-based 因果发现的鲁棒性, 核心发现: **non-iid (而非分岔临近本身) 是唯一能让几乎所有因果发现方法同时崩溃的场景**——独立于分岔的第 5 种混淆源, proposal 须设计负对照排除. 撞车 LOW-MEDIUM.

### J. 须超越的 robust baseline (支柱 c: 名单扩至 5 篇)

- **Tusoni, Masi, Coletta, Glielmo, Arrigoni, Bartolini (2025), PLaCy: Robust Causal Discovery with Power-Laws** (arXiv:2507.12257). 用幂律噪声建模提升因果发现对重尾/非高斯扰动的鲁棒性. **新增第 5 个"须超越"的稳健化因果发现 baseline** (此前名单: 2605.05809/2403.12677/2407.07290/2604.02488). 撞车 MEDIUM.

### K. 因果发现/CI 检验基础方法学 (idea v1 未引, 须补引的奠基谱系)

- **Huang, Zhang, Zhang, Ramsey, Sanchez-Romero, Glymour, Schölkopf (2019 UAI), CD-NOD** (arXiv:1903.01672). 异质/非平稳因果发现的奠基作之一, **认识论立场与本 idea 相反** (主动建模非平稳性以恢复正确因果图, 而非把不稳定性当信号), novelty check 此前遗漏此谱系, 须补引以证明差异化. 撞车 LOW-MEDIUM.
- **Zhang, Peters, Janzing, Schölkopf (2011 UAI), 核条件独立性检验 (KCI)** (arXiv:1202.3775). 方法基础必引链最上游节点, 后续多篇 (SplitKCI/GKCM/CI-hardness 系列) 均建立于此. 撞车无 (基础工具).
- **Pogodin, Schrab, Li, Sutherland, Gretton (2024), Practical Kernel Tests of CI (SplitKCI)** (arXiv:2402.13196). 给出 KCI 偏差的理论刻画, 为"有限样本估计伪影"提供机制性支撑 (呼应 J-PCMCI+ 的 Nickell FPR 膨胀发现). 撞车 LOW.
- **Wieck-Sosa, Haddad, Ramdas (2025), 单实现非平稳非线性时间序列的条件独立性检验** (arXiv:2504.21647, dGCM)。可作为 PCMCI+ 的候选插件检验器, 处理单一非平稳实现的统计推断. 撞车 LOW.
- **Zhang, Y. (2026), 高维非平稳时间序列独立性检验** (arXiv:2606.08498) 及其姊妹篇 **ANOVA for High-dimensional Non-stationary Time Series** (arXiv:2509.09079). 提供比"重叠窗口 Cohen's d"更严格的显著性检验候选, 可用于给 fragility score 的窗口对比配上正式的统计检验. 撞车 LOW.
- **Ferdous, Gani et al. (2023), CDANs: 自相关+非平稳时间序列的时序因果发现** (arXiv:2302.03246). PCMCI+ 在非平稳场景下的直接竞品, 检测"changing modules"而非本 idea 关注的估计脆弱性. 撞车 MEDIUM.

### L. 数据集/Benchmark (Stage-1/1.5 中间基准候选)

- **Stein, Shadaydeh, Blunk, Penzel, Denzler (2025), CausalRivers** (arXiv:2503.17452). 真实世界大规模因果发现 benchmark (含洪水分布偏移事件), PCMCI 在真实数据上仅中等表现——现实基线参照. 撞车 LOW.
- **Ferdous, Hossain, Gani (2025), TimeGraph** (arXiv:2506.01361, idea-scope 精读补充 topic-level 已读条目)。非平稳/混杂噪声下因果发现的合成 benchmark, 提供"地板基线"参照. 撞车 LOW.
- **Cheng, Wang, Xiao, Zhong, Suo, He (2023), CausalTime** (arXiv:2310.01753)。真实感生成时间序列因果发现 benchmark, **地理先验图模板可迁移到 3km 网格**, 可作 Stage 1.5 中间基准. 撞车 LOW.
- **Fu, Huang, Li, Zheng, Ng, Chen, Hu, Zhang (2025), CaDRe: 气候隐藏动态因果结构学习** (arXiv:2501.12500)。气候领域隐藏动态因果框架, 但**无 sub-daily (中尺度对流时间尺度) 实验佐证**——快变尺度上仍是空白, 支持本 idea 的迁移新颖性. 撞车 LOW.

### M. 强对流气候背景资料 (变量池/物理量来源, 非因果, 无撞车)

- **Li, Chavas, Reed, Dawson (2020), ERA5/CAM6 SLS 环境气候学** (arXiv:2005.05489)。强对流环境的再分析气候学, 候选环境变量池来源 (CAPE/shear 等标准 diagnostics 的观测基线). 撞车无.
- **Chavas, Dawson (2020), 理想化强对流探空模型** (arXiv:2004.11636)。给出 bulk diagnostics (CAPE/shear) 不足以刻画环境演化的具体实证数字 (以 3MAY99 个例为例), 直接支持"现有 diagnostics 信息维度有限"的研究动机. 撞车无.

## [idea-reviewer, 2026-07-11]

<!-- v2 审查阶段补充搜索, 通过 web search + arxiv.org abstract 页核验 (非全文精读, 因初筛后机制均与本 topic 核心因果发现/因果效应估计机制不同, 未构成新颖性直接威胁, 故未触发"必须读全文"门槛). -->

- **Ashwin et al. (2026), "Early warnings of critical transitions through vector autoregression: lessons from multiscale systems"** (arXiv:2605.28260). 提出用向量自回归 (VAR) 同时拟合多条时间序列以提取系统特征值, 从而在传统单变量方差/滞后自相关早期预警信号因振荡行为而失效的多尺度系统 (fold、亚临界 Hopf、含额外时间尺度分离的 singular Hopf 分岔) 中, 不仅判断稳定性还能识别即将发生的分岔类型. 机制是从 VAR 拟合系数中反推特征值 (确定性动力学重构), 而非追踪 VAR 系数本身在滑动窗口下的方差/漂移/不稳定性; 未使用因果推断框架, 也未处理观测假设违背下的估计器脆弱性.

- **Smith, Morr, Schötz, Boers (2026), "Estimating the Resilience of Non-Stationary Systems"** (arXiv:2604.24345; 作者更正见下方 2026-07-11 全文精读条目, 原条目误署名"Franzke et al.")。提出基于 Langevin 方程回归形式化的方法, 在强季节性驱动等非平稳背景下估计地球系统分量 (如全球植被) 的韧性/稳定性, 可原生处理数据缺口、不规则采样、时变观测不确定性, 并可扩展到空间系统. 是传统自相关类韧性估计量的替代品, 核心仍是"经典临界慢化框架下的韧性指标", 不涉及因果推断, 也不将估计过程本身的不稳定性当作信号来源.

- **(2026), "Early Detection of Latent Microstructure Regimes in Limit Order Books"** (arXiv:2604.20949)。针对限价订单簿, 用三 regime (稳定→潜伏恶化→压力) 因果数据生成过程形式化"潜伏建立期"窗口, 结合多路互补信号 MAX 聚合、rising-edge 条件和自适应阈值构造触发式探测器, 在 200 次仿真中平均提前 18.6±3.2 个时间步、precision 接近完美地检测出压力事件, 优于经典变点检测和微观结构基线. 应用领域 (金融市场微观结构) 与机制 (多路原始信号触发探测, 非因果效应/因果图估计过程的不稳定性) 均与因果发现文献不同, 但概念上是"潜伏期早于可观测应激期存在预测窗口"这一元叙事在另一领域的实现.

## [idea-reviewer via codex second opinion, 2026-07-11]

- **Laitinen, Lahti (2022), "Probabilistic Multivariate Early Warning Signals"** (arXiv:2205.07576)。提出用概率化 (贝叶斯/正则化) 向量自回归 (VAR) 模型作为多变量早期预警指标, 论证其相对传统单变量方差/自相关指标的优势在于更充分利用多变量信息、对参数的正则化处理及对不确定性的显式建模; 在多物种生态模型的仿真 benchmark 上验证检测灵敏度提升. 核心机制是概率 VAR 模型拟合过程本身的不确定性 (regularization/uncertainty treatment), 而非因果推断框架; 系统仍是经典 (慢变) 临界转换场景, 未处理快变强迫/非因果假设违背下的估计器脆弱性, 也未涉及因果发现方法 (PCMCI+/LKIF 等).

## [deep-lit-tick --scope topic, 2026-07-11 iteration 2]

<!-- 本轮 deep-lit-tick 第 2 次对本 topic 运行 (第 1 次已读 24 篇 + idea-scope 26 篇, 均在 already_read_ids 中去重). 本轮精读 34 篇 (5 轮, B4 饱和: 第 5 轮仅剩 1 篇独立候选 MOSAIC 且其 B7 反向扩展未产生任何新的、非已读、非切题的候选, 判定饱和). reader=claude(opus-4-8; 配置的 deepseek CLI claude-ds 本机未安装, 按 env 规则 fallback 到同接口的 claude). 每篇均有全文 wiki: $ARXIV_WIKI_DIR/<id>.md 含 "Read by: 0710-causal-scs". 核心结论: 本轮 34 篇中 0 篇构成 novelty 撞车 (最接近的 2511.04361 经全文精读证实为内部矛盾的空壳 workshop 摘要, 非真实竞品); 反而绝大多数 (>20 篇) 独立佐证了 idea 已冻结的 Stage-1 pilot 负面结果 (raw-state EWS baseline AUROC 0.919 远超因果 D/J 双通道 AUROC 0.428) 具有真实的方法论根源, 而非仅是仿真设置的偶然产物. 按主题分四组归档如下. -->

### N. EWS 可靠性/观测量选择/不确定性谱系 (最大簇, Boers/Morr/Bathiany/Ashwin/Lohmann 研究群体, 与 idea 立场相反但高度互补)

<!-- 本组 18 篇构成一个高度互引的紧密文献簇, 核心元问题与 idea 同构: "经典 EWS 指标的表观信号, 有多少是真实动力学、有多少是估计过程/观测选择的伪影?" 但本组一致选择"消除/校正/规避"这一混淆, 与 idea"保留并利用"的立场正好互补而非竞争。反引文网络在本轮末端已闭合 (2310.05587 反引文经核实全部落入已读集), 是本组饱和的直接证据. -->

- **Hobden, Ritchie, Ashwin (2026), VAR multiscale EWS** (arXiv:2605.28260, R-Score 未评级). 首次为多尺度奇异 Hopf 分岔给出定量 VAR 特征值 EWS, 发现 non-leading eigenvalue 才携带正确失稳趋势 (leading eigenvalue 和 AR(1) 均误导). 撞车无: 追求"更准的特征值估计"而非"估计不稳定性即信号", 适用域 (慢强迫经典分岔) 与 idea (快强迫强对流) 相反.
- **Suerhoff, Morr, Bathiany et al. (2026), 周期强迫振荡崩溃预警** (arXiv:2603.26537, R-Score A/18). 负结果: 经典 Floquet 乘子在慢强迫极限下指数趋零、无法预警; 提出相位漂移指标 (SVM+CV 验证). 方法论同构 (Stage-1 合成评测协议可复用), 撞车无.
- **Ben-Yami, Skiba, Bathiany, Boers (2023), AMOC CSD 不确定性传播** (arXiv:2303.06448)。用官方观测不确定性集合 + 修正 B21 的代理生成方法论缺陷, 证明"必须做数据感知 null model", 是本课题坚持匹配 null 校准纪律的活案例. 撞车无.
- **Ben-Yami, Boers et al. (2024 Sci Adv), tipping time 外推不确定性** (arXiv:2309.08521)。拆分建模/观测代表性/预处理三类不确定性, 证明崩溃时间外推可从 2050 跨到无穷, 但"探测是否失稳"本身相对稳健——为 idea pilot 选择判别式框架而非回归式外推提供支撑. 撞车无.
- **Shin, Boers, Kug (2025), 观测-模式 AMOC CSD 对齐** (arXiv:2503.22111)。用 ΔMOV 指标对齐观测 (1990s 出现 CSD) 与 CESM2 (2040s 才出现) 的 50 年错位, 是 raw-state EWS baseline 在真实顶级 tipping 场景的胜利案例, 佐证 pilot 负面结果非偶然. 撞车无.
- **Lohmann, Gottwald (2025), Fokker-Planck 观测量选择** (arXiv:2506.11735, R-Score A/18)。用反向 Fokker-Planck 次主导特征函数 (diffusion map) 构造最优 CSD 观测量, 明确承认准平稳假设对快速强迫系统失效——独立佐证 idea 放弃经典 CSD 框架转向强对流专属信号这一路线选择. 撞车无.
- **Morr, Riechers, Rydin Gorjão, Boers (2024 PRR), 多维 Kramers-Moyal CSD** (arXiv:2308.16773)。漂移雅可比特征值负实部泛化方差/AC1 到时变/状态相关噪声, 是 2605.28260 同谱系更早先驱. 撞车无.
- **Smith, Morr, Schötz, Boers (2026), Langevin 回归非平稳韧性估计** (arXiv:2604.24345; 更正: 此前 idea-review 阶段摘要级搜索误署名"Franzke et al.")。IRLS 稳健回归原生处理数据缺口/时变不确定性, NGRIP 案例部分推翻同一通讯作者 2018 年的正面 CSD 结论, 是"未校正采样偏差制造假阳性 EWS"的真实反例. 精读判定: 此前 novelty check 把本文归入"估计不确定性当信号"新颖性空间的判断不准确 (本文是诚实误差传播, 非追踪估计过程不稳定性), 建议下一版恢复相应新颖性空间的完整性. 撞车无, 立场相反.
- **Boers 组 (2026), AMOC 崩溃时间预测批判** (arXiv:2604.20341)。系统性技术批判 DD23 (Nature Comm): 三次替代模型反转外推结论、bootstrap CI 实际覆盖率仅 0-45%、两个具体代码 bug 导致显著性反转. 其"结构证伪"实验设计 (平淡模型合成数据检验有趣模型是否误报) 可直接移植为 idea Stage-1 负对照模板. 是本课题已追踪的 Boers 组 AMOC 批判链条 (2303.06448→2309.08521→2603.26537) 最新一环. 撞车无.
- **Liu, Morr, Bathiany, Blaschke, Qian, Diao, Smith, Boers (2025), 数据缺口扭曲 CSD 指标** (arXiv:2505.19034, R-Score S/22)。证明 λ_AC1/λ_Var 一致性几乎完全由首末数据点决定, 缺失值/异常值系统性偏置指标. 直接警示: idea 自己的 (D,J) 双通道联合 AUROC 也应检查是否受边界伪影主导, 且 idea 冻结 pilot 用的是无缺失干净模拟——可部分解释为何 raw-EWS baseline 分数偏高 (外部效度缺口, 建议下一版 refinement 标注). 撞车无.
- **Morr, Boers, Ashwin (2023), 交叉耦合噪声欺骗性 CSD 趋势** (arXiv:2311.18597, R-Score S/23)。解析证明: 即使确定性动力学完全解耦, 噪声跨维度耦合可使 6 种独立 EWS 方法 (含 4 种"现代"方法) 全部给出方向相反的欺骗性趋势. 对 idea Stage-1 耦合 Lorenz 验证有直接警示: 因果效应不稳定性本身也可能是"观测到错误变量组合"的伪迹. 撞车无.
- **Morr, Boers (2024 PRX), 红噪声 CSD 检测 ACS/PSD-fit** (arXiv:2310.05587)。联合拟合目标参数与 nuisance 噪声参数以分离真实失稳与红噪声伪影, 优于方差/AC1/GLSAR. **反引文核实显示该子方向 (CSD 鲁棒性/红噪声) 搜索已趋近饱和** (7 篇高相关反引文均已入库), 是本轮 B4 判定的关键证据之一. 撞车无.
- **Lohmann (2025), AMOC 观测量选择与分岔时间外推偏差** (arXiv:2512.17142, R-Score A/20)。三箱 AMOC 模型证明: 看似自然的观测量测不到 EWS 信号, 正交"盐度化"变量才对齐; 正规型标度假设仅极靠近分岔才成立, 不同观测量给出方向相反的偏差. 撞车无.
- **Shi, Serdukova, Zheng, Petrovskii, Lucarini et al. (2026), committor 几何 EWS** (arXiv:2603.08861)。基于 committor 函数的确定性 PDE 几何构造 EWS_geom, 弱噪声极限下与 log(MFPT) 渐近仿射, 强噪声/短窗口下仍可计算 (经典方差/AC1 因样本不足失效时). 与 idea 数据驱动因果估计器脆弱性诊断对偶 (信号来源相反: 真实分界面几何 vs 估计过程不稳定性), 应归入 EWS 元技巧家族"确定性 committor-几何路径"子类, 与 2605.28260"预测性 VAR-特征值路径"并列. 撞车无.
- **Morr, Kuehn, Datseris (2025), 统一韧性指标计算框架** (arXiv:2509.19609)。7 种韧性概念统一为雅可比+吸引域几何定义, 核心发现不同韧性指标在不同转变机制下矛盾. 认识论起点与 idea 完全相反 ("已知模型" vs "未知模型+纯观测"), 可作仿真阶段零成本诊断/消融工具. 撞车无.
- **Datseris, Lohmann, Hamilton, Haqq-Misra (2026), 高维系统 intermingledness** (arXiv:2604.09661)。IA-DBSCAN 聚类+intermingledness 指标区分"分岔诱导 tipping"(需低 intermingledness 诊断变量) 与"噪声诱导转变"(需高 intermingledness). 为解释 pilot 负面结果提供候选假说: saddle-node testbed 属分岔诱导型, raw-state 变量可能天然贴近最优判别特征, 因果效应波动未必与吸引域几何对齐. 撞车无.
- **Datseris, Rossi, Wagemakers (2023), RAFM/Attractors.jl 全局稳定性延拓** (arXiv:2304.12786, R-Score A/20)。已知 ODE 前提下追踪吸引子吸引域分数, 是 2509.19609 的方法论源头, 可直接为 idea 耦合 testbed 计算吸引域分数 ground truth. 引用的 Schultz 2017 反例 (吸引域分数不保证平滑趋零) 为"单一 EWS 指标不可靠"提供独立证据. 撞车无.
- **Enache, Kozak, Wunderling, Vollmer (2024), 鞍结分岔安全/危险 overshoot 边界** (arXiv:2401.07712, R-Score A/21)。严格推导 overshoot 深浅两种 regime 下 tipping 的解析安全/危险边界, 深越界时安全区远小于早期反平方根律预测. 直接回应 v2 review 指出的"control 轨迹缺少运行时未穿越验证、依赖经验参数试凑"的逻辑缺口, 其闭式边界公式可替代当前 0.28<0.385 经验参数构造方式, 为 Strongest Objection (matched controls 加入后信号可能消失) 提供更严格匹配轨迹对的构造工具. 撞车无.

### O. 因果发现/时序因果方法新增竞品与基础设施 (idea 核心方法轴的同期相关工作)

- **Faruque, Ali, Zheng et al. (2026), TTCD transformer 非平稳因果发现** (arXiv:2605.08111, R-Score B/15)。非平稳注意力+去平稳化因子块+瞬时块无环约束, 在 5 数据集上大幅超基线. 代表"主动消除非平稳性以获得更准因果图"的主流立场, 恰为 idea"保留假设违背导致的不稳定性并当信号"立场的镜像对立面. 若被复用逐窗口重训练, 训练方差本身可能被误认成结构脆弱性信号——idea 现有五类混淆解释外的候选第六类. 撞车无, 建议引用作主流对照.
- **Bi, Pan, Jiang, Sun, Ma, Wang (2025 NeurIPS), UnCLe 动态因果发现** (arXiv:2511.03168)。参数共享 TCN+事后置换扰动求真正动态因果图, TVSEM/ND8 合成基准正是 review 指缺失的 positive control regime, 可借用为 Stage-1 第三 regime. 撞车无 (求"估准"因果图 vs 求"估计不稳定性能否作信号", 二者正交).
- **Das, Chakraborty, Maulik (2026), SC3D 两阶段可微分因果发现** (arXiv:2602.02830, R-Score A/19)。谱半径惩罚+2-cycle 惩罚精炼结构, 五类测试床全面对比 7 基线. 明确声明"显式时变因果图/在线设定"仍是 future work, 是同赛道最新论文 (2026-02) 亲口承认"窗口不稳定性量化为信号"仍是空白的直接文本证据. 撞车无, 是 novelty 论证的有力旁证.
- **Thumm, Chen (2026), CausalTimePrior 干预时序先验** (arXiv:2603.11090, R-Score A/19)。regime-switching TSCM 生成器正是 review 指缺失的"已知真实 DAG 会变化的 positive control regime", 建议接入 Stage-1 作第二 regime. 撞车无.
- **Thumm (2025), Causal Regime Detection in Energy Markets** (arXiv:2511.04361)。**精读后降级**: 全文仅 196 行, 无 Results/Experiments section, 摘要中性能声明被作者自己注释掉但结论段落同样声明原样保留 (内部矛盾), 结论明确承认"未来工作包括实证验证". R-Score Tier C 下限 (7/25), 不构成需要对比的 baseline. 此前 B7 反向扩展中因"causal regime detection"表述与本课题高度相似而被列为最高优先级候选, 全文精读后证实只是概念相似的空壳摘要, 非真实竞品——是本轮"标题诱导的假阳性撞车警报, 全文精读后排除"的典型案例.

### P. 因果效应/SDE 可辨识性理论谱系 (更贴近姊妹课题 "Causal Discovery Uncertainty", 本课题背景素材)

<!-- 本组 9 篇是一条独立的数理统计/可辨识性理论文献线 (OU 过程、图连续 Lyapunov 模型、SDE 参数可辨识性、因果表征学习), 多篇读者独立指出"更适合姊妹课题 Causal Discovery Uncertainty, 而非本课题"。全部与 idea 核心 claim (有限样本滑窗因果估计脆弱性经验诊断) 不撞车, 因为它们关注总体层面模型可辨识性理论而非有限样本估计过程本身。 -->

- **LIACS Leiden (2026), OU 过程因果边符号可辨识性** (arXiv:2603.08311, R-Score S/24, UAI 2026)。放松扩散矩阵已知假设, 证明符号可辨识/不可辨识/部分可辨识三分类. 隐变量导致的结构性不可辨识是现有五类混淆外的候选第六类. 撞车无.
- **Guan et al. (2024), APPEX 边际快照 SDE 识别** (arXiv:2410.22729, R-Score A/21, JMLR revision)。证明线性 SDE 在仅边际快照观测下几乎总可识别, 但假设 SDE 参数全程恒定——与 idea 恰要捕捉参数漂移本身近乎对立. 撞车无.
- **ICLR 2025, 签名核 CI 检验 SigKer** (arXiv:2402.18477)。路径空间首个不依赖密度存在性的一致 CI 检验, 证明环存在时完整图发现信息论不可能. 撞车无, 可作黑盒统计量补充方法分歧对比集.
- **Recke, Hansen (2026), 图连续 Lyapunov 模型可辨识与估计** (arXiv:2603.17142)。非高斯噪声+高阶累积量可使 M (因果图) 可辨识到公共比例因子; 可辨识不等于小样本好估计. 撞车无.
- **ICLR 2026, 干预 SDE 可辨识性** (arXiv:2505.15987)。r 个干预识别漂移矩阵的精确界; 证伪了 APPEX 反引文线索——未把可识别性推广到临界转变场景. 撞车无.
- **Wang et al. (2026 NeurIPS preprint), 潜在 SDE 可辨识性 (扩散偏移)** (arXiv:2606.28228)。两环境扩散协方差方差比互异即可识别隐坐标到置换缩放; Hardanger 悬索桥真实数据跨种子验证, 是最贴近 causal-scs 真实数据验证阶段的方法论模板. 撞车无.
- **Améndola, Boege, Hollering, Misra (2025), 图连续 Lyapunov 模型结构可辨识性** (arXiv:2510.04985)。GCLM 等价类严格加细贝叶斯网络 Markov 等价类, 结构可辨识率 96.9% (n=6) 远超 8.1%. 撞车无, 对姊妹课题更相关.
- **Fan, Zhang, Cheng (2026 ICML), TRACE 连续机制演化轨迹恢复** (arXiv:2601.21135, R-Score S/22 边界)。K 个 atomic mechanism 概率单纯形连续凸组合替代离散切换; K_active 增大时"结构学错"与"分解阶段几何病态"两种失效清晰分离, 其对照消融范式直接回应 review 中"因果估计器不稳定性是否优于同复杂度非因果模型"的未解缺口. 撞车无 (需训练时已知纯域标签, 与强对流场景不符).
- **NeurIPS 2026 投稿, MOSAIC 稀疏加性模块发现** (arXiv:2605.05524)。两阶段稀疏加性时序 VAE, regime-association 是全数据集一次性静态分类, 无滑动窗口轨迹, 与 idea (D,J) 双通道非同类客体. 撞车无, Stage-1-only VAE 可作非因果脆弱性基线.

### Q. EWS 失败模式新增证据 (非因果领域, 加固"经典指标结构性失效"动机)

- **Truong, Truong (2026), Entropy Collapse 普适失效模式** (arXiv:2512.12381)。证明反馈放大系统的熵坍缩是一阶 (不连续) 相变, 经典 CSD 框架结构性失效 (无预警). 精读发现可信度瑕疵 (自指引用、S2/正文引用数不一致), 仅引用可独立验证的解析定理. 独立支持 idea"不假定经典临界慢化"的立场, 其 Remark 承认 CSD 对 fold 型二阶转换仍适用, 恰解释 idea pilot 中 fold testbed 上 raw EWS 高达 0.919 的结果. 撞车无.
- **Goltsev, Dorogovtsev (2026), 网络渗流 susceptibility EWS** (arXiv:2602.10060, R-Score A/19)。观察者节点 susceptibility 在渗流临界点发散, 推广到 k-core 崩塌/相依链路网络. 研究对象、驱动机制 (缓变准静态 vs 强对流快强迫) 均不同, 是经典 CSD 范式的严谨范例, 可作背景引用划清边界. 撞车无.

## [deep-lit-tick --scope idea causal-scs-indicator, 2026-07-11 iteration 2]

<!-- 本轮 idea-scope deep-lit-tick 第 2 次对本 idea 运行 (第 1 次已读 26 篇). 精读 18 篇 (5 轮, B4 饱和), 全部在此前 84 篇已读集之外, 由 dispatcher 汇总并入本文件 (idea-scope 规则不直接写 landscape). 聚焦用户指定的 v2 idea 四个具体设计点: 双族 D/J profile、synthetic stop gate、matched negative controls、PCMCI+/LKIF. 详见 ideas/causal-scs-indicator-deep-lit-report-2026-07-11-iteration2.md 的完整撞车矩阵. -->

### R. 双族 (D,J) profile 同构/镜像工作 (novelty 佐证核心)

- **Yu, Guo, Luk (2024), Robust Time Series Causal Discovery for Agent-Based Model Validation (RCV-PCMCI)** (arXiv:2410.19412)。提出 Consistency/Variability 统计量, 与本 idea 的 (D,J) 双族在数学形式上同源, 但用途相反——用于滤除跨折不一致性以求更稳健的因果图, 而非把该不一致性本身当作事件预警信号. 撞车 LOW (方向镜像).
- **Yu, Guo, Luk (2026), VCDF: A Validated Consensus-Driven Framework for Time Series Causal Discovery** (arXiv:2602.21381)。同一作者簇后续版, 扩展 consensus-driven 验证框架. **作者本人在结论中明确承认从未测试过跨折不一致性是否可能反映真实结构变化**——是本轮迄今最直接的 novelty 佐证: 该子领域的原创作者自己标记了本 idea 想填补的确切空白. 撞车 LOW。
- **Faltenbacher, Wahl, Herman, Runge (2025), How PC-based Methods Err: Towards Better Reporting of Assumption Violations and Small Sample Errors** (arXiv:2502.14719)。PCMCI+ 方法本身的作者团队提出零额外计算成本的"coherency score"内部自洽性诊断, 复用已执行的 CI 检验日志区分统计噪声与结构性假设违反. 候选第三诊断通道 (可能补充或替代当前二维 D/J 设计, 需与 codex second opinion 讨论是否引入会削弱"双通道正是为检验 C2 互补性"这一论证). 撞车 LOW, 与 PCMCI+/LKIF 设计点也直接相关.
- **Wahl, Runge (2024), Separation-Based Distance Measures for Causal Graphs** (arXiv:2402.04952)。提出一族基于图分离陈述的距离度量, 与 Tigramite 兼容, 可用于严格化 J (突变/edge 变化) 通道的定义. 撞车 LOW.

### S. Synthetic stop gate 现成基础设施

- **Stein, Penzel, Piater, Denzler (2026), TCD-Arena: Assessing Robustness of Time Series Causal Discovery Methods Against Assumption Violations** (arXiv:2605.03045, R-Score S/22)。33 类假设违背 × 16 regime × 10 方法的现成基准, 含真实 DAG 变化的正对照生成器 (V_stat/V_coef) 以及缺失/样本量/观测噪声混淆生成器 (V_mcar/mar/mnar/V_length/V_obs) —— 正是 v2 review 指出缺失的 "已知真实 DAG 变化的 positive control regime" 与混淆生成器库. 撞车 LOW (评测基准而非预警框架).
- **Butler, Machlanski, Dimitrakopoulos, Tsaftaris (2026), Rethinking Chronological Causal Discovery with Signal Processing** (arXiv:2602.19903)。实证 GC/PCMCI/DYNOTEARS 对窗口长度/采样率 (Q,k) 高度敏感 ("因果 Nyquist" 现象), 提出候选第七类混淆源 (相变现象可能被误判为真实结构变化). 双变量延迟滤波器设置可作低成本合成 testbed. 撞车 LOW.
- **Machlanski, Samothrakis, Clarke (2023), Robustness of Algorithms for Causal Structure Learning to Hyperparameter Choice** (arXiv:2310.18212)。证明超参数误设下的鲁棒性剖面与 oracle 最优表现近乎正交, 提出 BEST/WORST/DEFAULT/SIM_MEAN 四态框架, 建议移植用于隔离超参数漂移与真实窗口驱动信号 (gate 设计应固定超参数控制). 撞车 LOW.

### T. Matched negative controls 方法论

- **Petersen (2024), Are You Doing Better Than Random Guessing? A Call for Using Negative Controls When Evaluating Causal Discovery Algorithms** (arXiv:2412.10039)。提出超几何精确检验 + 模拟负对照的四步评估管线, 直接呼应本 idea 已有的 paired-trajectory 负对照设计, 可借鉴其统计严谨性. 撞车 LOW.
- **Kummerfeld, Lim, Shi (2022), Data-driven Automated Negative Control Estimation (DANCE)** (arXiv:2210.00528)。用 vanishing tetrad test 从数据中自动搜索并验证横截面负对照变量. 应用场景 (横截面因果推断) 与本 idea (时序滑窗) 不同, 仅方法论姿态可类比. 撞车 LOW.
- **Tsoumas, Bormpoudakis, Sitokonstantinou, Askitopoulos, Kalogeras, Kontoes, Athanasiadis (2025), Positive-Unlabeled Learning for Control Group Construction in Observational Causal Inference** (arXiv:2507.14528)。用 PU-learning 在无天然对照组时构造高置信度对照组. 真实数据阶段 (而非当前 synthetic gate 阶段) 的候选工具. 撞车 LOW.

### U. PCMCI+/LKIF 镜像对照与应用背景 (均加固 "PCMCI+ 在假设违背下确实脆弱" 这一动机前提)

- **Stanishevska, Guikema (2025), Operational early warning of thunderstorm-driven power outages from open data** (arXiv:2510.03959)。两阶段机器学习做雷暴驱动停电早期预警, 其"causal"仅指无泄漏特征工程, 非结构因果发现. 应用领域高度相关 (强对流驱动的下游影响预警) 但方法不撞车; 该研究本身也报告 NEGATIVE 结果, 侧面印证此类预警任务的真实难度. 撞车无.
- **Paldino, Bontempi (2025), Causal Discovery in Multivariate Time Series through Mutual Information Featurization (TD2C)** (arXiv:2508.01848)。监督式因果发现, 主动消除假设违背以求更准的图, 方向与本 idea 正相反 (镜像对照). 撞车 LOW.
- **Fesanghary, Havaldar (2026), GRACE: Gated Refinement for Accurate Causal Edge Discovery in High-Dimensional Time Series** (arXiv:2606.23880)。L0 门控 + bootstrap 多数投票丢弃窗口不稳定性以求精炼骨架, 坐实了此前已读的 Bagged-PCMCI+ (arXiv:2306.08946) 这一"把不稳定性平均掉"的路线仍在持续发展. 撞车 LOW.
- **Stein, Shadaydeh, Denzler (2024), Embracing the black box: Heading towards foundation models for causal discovery from time series data** (arXiv:2402.09305)。监督端到端映射因果图的立场论文, 方向正交于本 idea, 报告诚实 (含负结果). 撞车无.
- **Sanchez, Machlanski, McDonagh, Tsaftaris (2025), Causal Ordering for Structure Learning from Time Series (DOTS)** (arXiv:2510.24639)。多噪声尺度扩散排序软投票求更准结构, 方向正交. 撞车无.
- **Machlanski, Samothrakis, Clarke (2023), The Challenges of Hyperparameter Tuning for Accurate Causal Effect Estimation** (arXiv:2303.01412)。横截面 CATE 估计的超参数/指标选择敏感性研究, 非时序因果发现, 仅作方法论警示背景. 撞车无.

### V. 背景类比 (非因果, EWS/物理背景)

- **Dylewsky, Anand, Bauch (2024), Early Warning Signals for Bifurcations Embedded in High Dimensions** (arXiv:2402.10300)。神经网络嵌入高维观测的经典 (慢变 regime) EWS, "silent catastrophe" 概念 (高维中隐藏的失稳信号) 可类比. 撞车 LOW.
- **Bar, Banerjee, Casals, Catalan, Rodríguez-Viejo (2025), Disorder-aided Early Warning Signals: Predicting Catastrophic Shifts in Athermal Systems** (arXiv:2509.01601)。凝聚态物理中无序如何拉长预警窗口, 直觉上可类比环境异质性对指标可靠性的影响. 撞车无.

**待 records-only 引用 (无 arXiv ID, 无法通过 arxiv-tool 全文精读, 仅摘要级记录)**: Miersch, Günther, Runge et al. (2025, DOI:10.1175/aies-d-24-0114.1) 对 PCMCI+ 在 45 个真实流域上的鲁棒性评测; Gamella, Peters, Bühlmann (2025, DOI:10.1038/s42256-024-00964-x) 用 Causal Chambers 显示 PCMCI+ 在真实因果链上表现近乎随机猜测。两篇均与本 idea 自身 Stage-1 NEGATIVE pilot 结果高度平行, 是独立的真实世界证据来源, 但受限于 arxiv-tools 红线未能下载全文, 如实标注此限制。

## [deep-lit-tick --scope topic, 2026-07-11 iteration 3]

<!-- 本轮 topic-scope deep-lit-tick 第 3 次对本 topic 运行 (第1次24篇+第2次34篇topic-scope, 加idea-scope第1次26篇+第2次18篇, 已读基线102篇, 已加载 already_read_ids 去重)。本轮精读 36 篇 (5轮: 8+8+8+6+6, B4 饱和: 第5轮候选池已明显转向纯理论/跨领域通用因果发现方法, B7反向扩展多次把已读论文误报为"新发现"或仅在已探索透彻的相邻簇——SURD/PID信息论分解、结构不确定性/误设基准、通用TSCD方法——内部循环, 无一篇构成新的直接claim重叠)。reader=claude(opus-4-8; claude-ds 本机未安装, 按约定 fallback)。每篇均有全文 wiki: $ARXIV_WIKI_DIR/<id>.md 含 "Read by: 0710-causal-scs"。核心结论: 36 篇中 **0 篇构成 novelty 撞车**——idea v3 novelty quick-check 现有最强撞车对比对象 (RCV-PCMCI/VarLiNGAM 2410.19412、VCDF 2602.21381、graph-instability 2606.01214、coherency score 2502.14719) 排名不变, 无需更新。但本轮意外收获: 至少 4 篇 (2401.16512/2402.01341/2306.07047/2505.10878) 为 idea v3 pilot 已观测到的反直觉负面结果 (causal/PCMCI+ 条件化在 fragility regime 判别力落 chance, 弱于 non-causal 边际相关基线) 提供了此前未被考虑的、来自不同技术路线的理论解释候选, 已同步写入 `ideas/causal-scs-indicator.v3.md` 的新增 `<deep-lit-integration>` 区块。用户本轮特别要求聚焦 v3 新引入的"真实 PCMCI+ pilot vs non-causal 边际相关基线"这一角度, 本轮搜索关键词与选题据此针对性加权 (conditioning penalty、结构不确定性、鲁棒性基准、信息论因果分解四个方向)。按主题分六组归档如下。 -->

### W. 因果分解/信息论谱系 (SURD/PID 家族及理论支撑, 静态单快照分解, 与"滑窗估计不稳定性作信号"正交但对 pilot 结果有解释力)

<!-- 本组9篇构成一条独立但内部高度互引的信息论因果分解文献线 (SURD 及其扩展、PID 变体、因果熵理论)。全部为静态单窗口/单快照分析, 无滑动窗口或时序不稳定性维度, 与 idea 核心 claim 不撞车; 但其中数篇为 idea v3 pilot 的反直觉负面结果提供了独立的理论解释候选, 价值高于常规背景引用。 -->

- **Martínez-Sánchez, Arranz, Lozano-Durán (2024), SURD (Synergistic-Unique-Redundant Decomposition)** (arXiv:2405.12411, R-Score S/22)。逐状态 specific mutual information 比特增量把变量子集对目标的因果贡献分解为冗余/独有/协同三部分, 首次给出可量化的"因果泄漏"(未观测变量贡献), 15+ benchmark 全面优于 CGC/CTE/CCM/PCMCI, 代码开源、81 篇反引文跨领域采用。撞车无(静态单窗口分解)。因果泄漏可作与现有 dispersion/jump 正交的新诊断 channel。
- **Yuan, Lozano-Durán (2024), Limits to extreme event forecasting in chaotic systems** (arXiv:2401.16512, R-Score A/18)。混沌系统极端事件预测误差的精确 Bayes 最优解 + 与建模方法无关的信息论上下界 (cost-sensitive Fano's/Hellman's 不等式), 三源误差分解(初始条件/未观测变量/模型次优)。撞车无(纯信息论工具, 无因果图)。**为 idea v3 pilot NEGATIVE 结果(causal channel AUROC 0.39-0.52 远低于 raw-state EWS 0.919)提供数据处理不等式式的原则性解释**: 任何原始状态的衍生统计量(含因果不稳定性分数)信息量不可能超过原始状态本身, 建议写入 EXPERIMENT_LOG。
- **Martínez-Sánchez, Lozano-Durán (2025), Cause-and-effect approach to turbulence forecasting** (arXiv:2509.25065, R-Score B/16)。SURD+MINE(神经互信息估计器)做湍流预测输入变量选择, DNS 槽道流真实数据验证。撞车无。验证"causal vs 同复杂度相关基线"对照范式与本课题 pilot 思路相通, 独立佐证该实验设计范式的合理性。
- **Martínez-Sánchez, Lozano-Durán (2025), Observational causality by states and interaction type** (arXiv:2505.10878, R-Score S/23)。SURD 逐状态(而非聚合)扩展, 代码开源, 证明 5 种现有时间/状态可分辨因果方法(含时变 Liang-Kleeman 信息流 TvLK)均无法在 toy 案例复原 ground truth。撞车无("状态"轴是变量瞬时取值, 非时间窗口)。**关键警示**: TvLK(LKIF 近亲, 线性/高斯/Kalman平滑)对硬阈值突变 toy 案例未给出清晰证据——若 idea 未来复现 windowed LKIF 在 fragility regime 也落 chance, 需先做已知突变正对照, 排除"Kalman 平滑类方法对突变不敏感"这一竞争解释, 而非直接归因于"条件化丢信号"是跨估计器现象。引出 RPCMCI(Saggioro et al. 2020, arXiv:2007.00267, PCMCI 家族 regime 方法)为更省成本的状态分解候选。
- **Yang, Wang, Zhang (2026), PEID (Partial Effective Information Decomposition)** (arXiv:2605.03267, R-Score A/18)。最大熵干预使源变量独立后做 PID, 扩展到含协同超边的多尺度粗粒化与连续系统, 应用杭州空气质量预测。撞车无。与 SURD 同域两条不同技术路线, 互为对照; Flexibility+environment synergy 分解可作候选第三诊断方向。
- **Jansma (2025), Decomposing Interventional Causality into Synergistic, Redundant, and Unique Components** (arXiv:2501.11447, R-Score A/19)。把 PID 推广到干预性因果效应(MACE), 用反链格 Möbius 反演, 与 SURD 逐点批判性对比。撞车无(假设效应已识别, 不处理估计不确定性/时间滑窗)。
- **Faes, Mijatović, Pernice, Marinazzo, Stramaglia, Antonacci (2026), PDGC: Dissecting Spectral Granger Causality through PID** (arXiv:2603.07634, R-Score A/19)。谱域 Granger causality + PID 分解, 心血管-脑血管-呼吸网络真实数据验证(直立性晕厥 vs 健康对照)。撞车无(单一平稳区段内分解, 从未把估计不稳定性当信号)。IAAFT surrogate + 百分位显著性检验方法论与本课题 null-percentile-calibration 路数一致, 可作旁证; 有限样本 GC 偏差/方差仿真刻画可作背景引用。
- **Simoes, Dastani, van Ommen (2024), Fundamental Properties of Causal Entropy and Information Gain** (arXiv:2402.01341, R-Score A/20)。纯理论证明因果熵可超观测熵上界、因果信息增益不对称可为负、因果版数据处理不等式不成立。撞车无(不涉及因果发现算法/时间窗口)。**为 v3 核心发现(PCMCI+条件化系统性丢失判别信息)提供跨技术路线的理论合理性佐证**: "因果版本信息量系统性劣于/异于非因果对应量"是因果信息论中已知的可数学刻画现象, 而非本课题实现缺陷, 可作 novelty quick-check 补充引用(但不能替代 v3 review 要求的 fixed-edge 消融等实证验证)。
- **Wahl, Ninad, Runge (2023), Foundations of Causal Discovery on Groups of Variables** (arXiv:2306.07047, R-Score S/23)。纯理论: 因果 Markov 性质/邻接忠实性无条件从微观变量传递到分组(宏观)变量, 但完整忠实性一般不传递(含反例)。撞车无(静态分组变量假设传递理论)。**Section 9 降维陷阱为 idea 最反直觉发现(条件化在 fragility regime 判别力落 chance)提供此前未考虑的候选解释**: 若真实数据阶段所用变量(如 HRRR 格点场提取指标)本质是更精细大气过程的粗化聚合, 条件化"丢信号"可能部分源于组级变量违背因果忠实性这一结构性伪影, 而非纯粹实验设计混杂; 建议下一版 idea 在讨论机制解释时引用本文 Section 9。

### X. 结构不确定性/误设/鲁棒性基准谱系 (与 v3 "causal vs non-causal 存在性门槛"直接相关的方法论背景)

- **Yi, Shen, Wu, Chen, Wang, Yu (2026), CausalCompass** (arXiv:2602.07915, R-Score A/19)。NeurIPS2026投稿, 11种TSCD方法×8类假设违背×110000+次实验鲁棒性基准, 核心结论"无单一方法全场景最优, 深度学习方法总体最稳健"。撞车无(问"准确率在违背下掉多少", idea问"估计不稳定性本身能否作独立诊断信号", 完全不同轴)。确认 PCMCI 在该文献中准确率中游, 印证 idea 选 PCMCI+ 出于可解释性而非 SOTA 图恢复; 同作者 i.i.d. 前作已一并读入(见下条), 构成完整研究纲领, "违背下准确率排行榜"角度已趋饱和。
- **Yi, He, Chen, Kang, Wang, Yu (2025), The robustness of differentiable Causal Discovery in misspecified Scenarios** (arXiv:2510.12503, R-Score A/19)。CausalCompass 的 i.i.d. 前作, 12种方法×8类违背×7万+次实验, 发现可微分方法几乎全场景最优除尺度变化场景, 用噪声比 r 给出理论解释。撞车无。评估基于 oracle tuning(每数据集独调至真值最优超参)是最大可信度扣分点, 若课题想做"无 oracle 访问的近似指标"或"非 oracle 超参数场景稳健性评估协议", 构成有价值差异化空间。
- **Strieder, Drton (2024), Dual Likelihood for Causal Inference under Structure Uncertainty** (arXiv:2402.08328, R-Score A/20)。结构不确定性下总因果效应置信域的 dual likelihood 闭式解, 比前作 LRT 快 2 个数量级。撞车无, 问题方向近乎相反(本文求"结构不确定性不污染效应推断"的严格统计保证且要求已知/静态/可识别查询, idea 明确将"识别真实大气DAG"列为 non-goal)。可作"点估计因果效应需要传播结构不确定性"背景论点引用。
- **Padh, Li, Casolo, Kilbertus (2025), Your Assumed DAG is Wrong and Here's How To Deal With It** (arXiv:2502.17030, R-Score A/19)。已知 sure/forbidden 边下相容 DAG 集合上因果查询上下界, 梯度优化(DAGMA无环约束+Gumbel-Softmax)替代暴力枚举。撞车无(静态单快照, 属 topic 已排除的姊妹课题 Causal Discovery Uncertainty 范畴)。PC 多排列 sure/forbidden 边提取机制可类比量化滑窗骨架不确定性, 作为 (D,J) 外第三诊断通道候选, 但优化开销与当前预算冲突, 暂不建议采纳。
- **Zhang, Chen, Yao, Wang (2026), Consistency evaluation of benchmarks used for causal discovery** (arXiv:2606.01789, R-Score A/18)。首次系统评估11个因果发现基准(Sachs/Child/Alarm/Asia等)ground-truth图与当前领域文献一致性, 用LLM判断38081篇论文支持/矛盾, 发现即便最成熟的 Sachs 基准仍有 26.7% 不一致率, 老基准普遍更差。撞车无(审计静态基准图质量, 与本课题滑窗时序不稳定性完全不同领域/方法)。**为 topic 声明的 non-goal("不以恢复真实大气DAG为目标")提供最直接实证佐证**: 若医学领域数十年研究的 ground-truth 基准都难维持文献一致性, 以恢复真实大气 DAG 为评价目标更站不住脚。

### Y. 时序/非平稳因果发现新方法 (候选估计器/工具池, 准确率轴与 idea 不稳定性轴正交)

- **Faruque, Ali, Zheng, Wang (2024), TS-CausalNN** (arXiv:2404.01466, R-Score A/18)。并行 Causal Conv2D 支路 score-based 深度学习时序因果发现, 北极海冰真实数据验证非平稳性, 优于 PCMCI(+)/NOTEARS-MLP 多数指标。撞车无(让因果图点估计更准, 无滑窗/扰动/bootstrap稳定性分析)。候选第三估计器接入现有(D,J)协议; 自曝 best-of-N 评测弱点印证"缺多seed/bootstrap"批评的普遍性。
- **Gao, Addanki, Yu, Rossi, Kocaoglu (2024), Causal Discovery in Semi-Stationary Time Series (PCMCI_Ω)** (arXiv:2407.07291, R-Score A/20)。处理机制随时间周期性确定性重复变化的"半平稳"时序, 标准PCMCI得父集超集后逐变量猜周期重做CI检验。撞车无(仅处理已知/待估计但精确周期性的机制变化, 不覆盖非周期性单次结构突变)。"稀疏度反推正确时间切分"trick 可作结构变化诊断信号来源候选, 但需与其强假设(硬机制切换、遍历Markov链)划清边界; 姊妹篇 Causal-RuLSIF CPD(2407.07290)已在 topic-level iteration 1 读过。
- **Chen, Wu (2026), SGED-TCD** (arXiv:2604.10371, R-Score B/13)。显式结构门控张量+双扰动视图稳定性正则+扰动效应对齐损失, 应用中国东部/北部热浪-空气污染复合极端月尺度早期预警, 优于persistence/相关性/Granger-VAR基线。撞车无(问题设置正交)。**本文把跨扰动视图的结构不一致当作训练时要用 L_stable 正则主动压制的缺陷, 而 idea 把跨滑动时间窗口的因果估计不稳定当作要提取利用的信号本身——这个对比证明"因果估计对扰动敏感"是2026年同域SOTA方法仍需专门处理的真实现象, 已纳入下方 idea 集成区块**。
- **Chen, Shi, Yue (2026), Causal Discovery from Heteroscedastic Stochastic Dynamical Systems under Imperfect Physical Models (SCD)** (arXiv:2602.04907, R-Score A/17)。已知(可能有偏)ODE机制放入SDE漂移项, 未知因果图编码进乘性扩散项, 证明ODE先验有偏(加性误差δ)时恢复保证仍成立(给出显式容忍阈值)。撞车无。稳定化拟似然技巧、误设定压力测试实验设计模式可复用。
- **Ouyang, Zhang, Zhuang, Shu, Guo, Yang (2026), PTCD** (arXiv:2605.26759, R-Score A/17)。时序因果发现预训练框架, 双尺度注意力+干预pretext task+因果mixup, 跨数据集泛化超过AERCA/CUTS+。撞车无(跨数据集准确度 vs 时序不稳定性)。SCM假设时不变因果关系与检测结构失稳的前提相悖, 未经验证不可直接套用。
- **Blöbaum, Balasubramanian, Kasiviswanathan (2026), FoundCause** (arXiv:2606.17516, R-Score B/14)。amortized因果发现Transformer, 横截面i.i.d.数据单次前向(<2秒)出DAG+混淆矩阵, 15个真实基准AUROC全面超越经典方法。撞车无(横截面无时间/滞后结构, 问题设定完全正交)。推理速度快理论上可大幅降低review要求的多seed/两阶段bootstrap复现成本, 但需显式lag-augmentation改造, 否则丢失PCMCI+滞后条件独立结构。
- **(2026), PRCD-MAP** (arXiv:2605.01669, R-Score A/19)。结构VAR+MAP框架融合可靠性未知的外部因果先验, 逐边可学习empirical-Bayes温度校准, CausalTime真实基准验证。撞车无(静态单次先验融合 vs 动态滑窗不稳定性, 且不涉及外部先验)。**Weak-Data Sweep 发现 T 接近 d 时学习信号退化为噪声, 与 idea README 明确的"区分事件特异性不稳定与估计器噪声"这一待解决瓶颈高度共鸣**, 为 pilot Regime 1 causal 通道落 chance 附近提供另一场景下的佐证背景。
- **(2026), EML-CD** (arXiv:2606.05942, R-Score B/16)。符号算子二叉树表示因果机制边, 两阶段框架联合恢复DAG结构和逐边闭式方程, Sachs数据SHD持平PC/GES但精确率最高、召回率低。撞车无(静态横截面i.i.d.数据机制可解释性工作)。用R²不对称启发式代理替代残差独立性检验, 与idea均触及"启发式代理/判别力替代严格检验可能引入未隔离混淆"这一批评视角, 可跨领域简要提及。

### Z. 强对流/地球科学应用与综述背景

- **Ali, Hasan, Li, Faruque, Sampath, Huang, Gani, Wang (2024), Causality for Earth Science — A Review** (arXiv:2404.05746, R-Score A/18)。系统综述时序/时空因果方法及地球科学应用, 覆盖PCMCI/PCMCI+/LPCMCI/LiNGAM/Granger/TCDF等方法及数据集/工具箱清单。撞车无(通篇把confounding/non-stationarity当需规避的偏差来源, 从未讨论假设违背导致的效应不稳定性本身可作信号——这正是本课题核心创新点, 互补而非竞争)。全文未提severe convective storm/tornado/hail应用, 也未提LKIF方法; 提供Tigramite/CauseMe/NAM数据集/TPR-FPR-SHD-SID-AOC评估指标等可直接复用的背景资源。
- **Cerutti (2025), Methodological Insights into Structural Causal Modelling and Uncertainty-Aware Forecasting for Economic Indicators** (arXiv:2509.07036, R-Score B/15)。LPCMCI+GPDC分析美国宏观经济指标, 结合Chronos零样本预测+Beta-Binomial覆盖率校准。撞车无(静态单次因果图, 从未做滚动重估计, 完全不涉及"因果图不稳定性"这一核心假设)。GPDC在条件集大、样本小时退化的警示, 对idea未来若尝试GPDC替代线性ParCorr/PCMCI+是直接警示(事件前窗口正是小样本场景)。

### AA. 分组变量/多域/空间因果推断 (topic 未来若扩展到空间网格数据的候选背景)

- **Li, Zhang, Zhou (2025), Spatio-Temporal Hierarchical Causal Models (ST-HCM)** (arXiv:2511.20558, R-Score A/18)。层级因果模型时空扩展+坍缩定理(unit数趋无穷时ST-HCGM收敛到ST-DCM), 芝加哥交通真实数据验证。撞车无(追求"拿到正确点估计", idea追求"估计不稳定性本身能否作信号")。**最有价值的差异化论点**: 本文把估计方差当噪声、越小越好, idea把估计方差(及跳变)当信号本身——适合写入 motivation 段落, argue 现有ST因果识别文献默认追求点估计正确性, 没人问过在线场景下估计不确定性的瞬态膨胀是否携带状态转移信息。
- **Jalaldoust, Salehkaleybar, Kiyavash (2025), Multi-Domain Causal Discovery in Bijective Causal Models** (arXiv:2504.21261, R-Score A/19)。多域设置下假设因果函数不变、噪声跨域变化的双射生成机制(BGM), 统一ANM/LiNGAM/post-nonlinear/location-scale四类经典噪声模型。撞车无, 方向相反(本文假设机制不变去恢复真实图, idea假设机制可能被违背且不关心恢复真实图)。跨域独立性检验若把窗口视为domain, 或可作"隔离条件化"消融的候选形式化, 但需每窗口完整条件密度估计, 成本高于现有PCMCI+/LKIF协议。
- **Ninad, Wahl, Gerhardus, Runge (2025), 向量值变量因果发现与一致性引导聚合** (arXiv:2505.10476, R-Score A/17)。**PCMCI+/J-PCMCI+核心作者团队工作**, 比较逐分量/聚合/向量化三条因果发现路线, 提出无真值下的聚合一致性分数(c_ind/c_dep/AC)及自适应包装器Adag。撞车无(空间维度聚合, 非时序滞后边际化, 默认宏观因果充分性)。框架层面强相关(无真值一致性指标+自适应降维包装器, 与本课题"跨窗口一致性"目标类似), 建议未来扩展到空间网格数据时引用其分数体系和Adag模式作对比基线。

### AB. 静态方向判定/贝叶斯决策/专家先验融合/其他因果发现方法学 (topic 已知方法谱系的补充背景)

- **Kaptein (2025), 双变量因果方向不确定性下的贝叶斯决策理论** (arXiv:2507.23495, R-Score A/17)。证明结构后验不确定性适中时贝叶斯模型平均比模型选择决策更优, 6400次仿真验证。撞车无(静态单次决策优化 vs 时序早期预警)。"有限样本下结构后验更不确定"与idea pilot小窗口PCMCI+接近chance的观察有方法论共鸣。
- **Hiremath, Janzing, Faller, Blöbaum, Kirschbaum, Kasiviswanathan, Gan (2025), Guess2Graph** (arXiv:2510.14488, R-Score A/18)。专家/LLM猜测引导PC系算法CI检验执行顺序而非替代检验结果, 保持统计一致性。撞车无(meta-level"何时该信任猜测"与本课题不同轴, 互补)。
- **Murphy, Benavoli (2026), 核方法Granger因果统一 + GP_SIC + 同期GC识别** (arXiv:2601.09579, R-Score A/19)。统一KGC/lsNGC为KPCR特例, 提出GP_SIC score-based方法及首个纯GC同期识别算法。撞车无(object-level方法准确率 vs meta-level估计不稳定性, 问题空间正交)。GP_SIC是topic候选score-based方法现成低成本候选; **贝叶斯Wilcoxon符号秩检验可直接填补v3 review反复指出的"缺Δ显著性检验"缺口**。
- **Zhou, Wang, He, Zhou, Olya, Kocaoglu, Ribeiro (2025), DAGPA** (arXiv:2510.22031, R-Score S/22)。percolation理论+soft logic使d-separation可微, 真实Sachs数据CI-MCC远超PC/NOTEARS/DAGMA。撞车无(静态单快照方法)。已更正2603.24436对本文可扩展性的错误转述——实际O(d^5), d=50需GPU 12小时, 不适合topic滑窗重复估计。
- **Gladyshev, Alechina, Dastani, Doder, Logan (2025), Temporal Causal Reasoning with SEM** (arXiv:2501.10190, R-Score A/20)。纯理论时序因果逻辑CPLTL, 假设因果模型已知/无同时刻依赖。撞车无(假设模型给定, 不涉及从数据学习/估计, 回避真实数据违背假设这一核心问题)。仅背景术语层面价值(Full Time/Window/Summary Causal Graph标准词汇脉络)。
- **Sadeghi, AbdAlmageed (2026), 因果表征学习基准/复现审计** (arXiv:2603.17405, R-Score B/16)。批判CRL数据集/评估指标, 复现CausalVAE代码10次发现方差扩大5-10倍。撞车无(静态图像VAE解耦评估, 零涉及时序/tipping point)。复现方差实证可作本课题NEGATIVE结果讨论的跨领域参考。
- **Telcs, Kurbucz, Jakovác (2024), df-causality** (arXiv:2410.19469, R-Score B/15)。条件方差随约束数增加饱和读出"自由度", 同时覆盖确定性和随机系统。撞车无(全新底层因果发现算法本身, 目标是平稳完整数据下正确判定因果结构)。要求无环图+单步滞后+平稳性, 与真实强对流环境冲突, 仅适合模拟阶段补充对照。
- **Kafantaris (2026), "Enes Causal Discovery"** (arXiv:2603.24436, R-Score C/7)。质量差的单作者论文, 双专家MoE做因果模式分类, Recall/F1远逊PC算法(50节点MM上PC Recall=0.735 vs 仅0.135), 作者自认"cherry picks results"。撞车无(生物信息学蛋白质网络应用域, 方法论立场相反)。**不建议引用**, 唯一价值是其引用的dagpa2025(即上方2510.22031)经独立核验澄清了错误的可扩展性转述。

## [deep-lit-tick --scope idea causal-scs-indicator, 2026-07-11 iteration 3]

<!-- 本轮 idea-scope deep-lit-tick 第 3 次对本 idea 运行, 精读 51 篇 (7 轮, B4 饱和), 由 dispatcher 汇总并入本文件 (idea-scope 规则不直接写 landscape)。本轮聚焦回答 v3 review 提出的两个具体问题: (a) 能否设计更干净的受控对比隔离 Delta = AUC_conditional - AUC_marginal; (b) 是否有"因果条件化丢弃预测信号"这一现象的理论解释。全部 51 篇均为理论/机制类背景文献, 0 篇构成 idea 核心 niche 的 novelty 撞车 (最强撞车对象 RCV-PCMCI/VCDF/graph-instability/coherency-score 排名不变)。详见 ideas/causal-scs-indicator-deep-lit-report-2026-07-11-iteration3.md。 -->

### AC. Markov Blanket 预测最优性 vs 因果 parent set (机制支柱: 为何条件化丢信号)

- **Steyerberg 学派综述, Directed Acyclic Graphs and causal thinking in clinical risk prediction modeling** (arXiv:2002.09414)。证明 Markov Blanket (parents+children+parents-of-children) 是预测最优集, parents-only (=causal) 模型校准误差全场第二差; children 携带的信号是真实反向读出而非伪相关。撞车无, 是本轮解释"因果条件化丢弃预测信号"最直接的机制支撑之一。
- **因果特征选择综述** (arXiv:1911.07147, Yu et al.)。CI 检验所需样本量随条件集大小指数增长, 是本课题"PCMasking"失效模式的正式命名来源。撞车无。
- **A Unified View of Causal and Non-causal Feature Selection** (arXiv:1802.05844)。证明因果 FS 与非因果 FS 是同一目标函数 (互信息最大化=Markov Blanket) 的不同求解路径; 因果法条件集阶数指数增长 vs 非因果法阶数≤2, 小样本高维时非因果法几乎全面胜出。撞车无。
- **Peters, Bühlmann, Meinshausen (2016), Causal inference using invariant prediction (ICP)** (arXiv:1501.01332)。ICP 原始论文, Theorem 1 只保证 FPR 不保证功效; 显式必要性反例——检验矩与扰动矩错配 (如仅二阶矩变化) 导致功效归零。撞车无, 是"条件化在特定错配下会归零功效"这一机制最早的数学证明。
- **Heinze-Deml, Peters, Meinshausen (2018), Invariant Causal Prediction for Nonlinear Models** (arXiv:1706.08576)。非线性 ICP, 给出三个具体的 CI 检验功效盲区数学反例。撞车无。
- **Rojas-Carulla, Schölkopf, Turner, Peters (2018), Invariant Models for Causal Transfer Learning** (arXiv:1507.05333)。anchor regression/IRM 共同理论先驱; Type II 错误导致非因果变量被误判为不变的具体机制, 提供两个可迁移的受控 factorial 设计模板。撞车无。
- **Arjovsky, Bottou, Gulrajani, Lopez-Paz (2019), Invariant Risk Minimization (IRM)** (arXiv:1907.02893)。IRM 奠基论文, 其自身合成实验报告 ICP (与 PCMCI+ 同源方法) 表现"保守"(漏检真父节点)。撞车无。
- **Rosenfeld, Ravikumar, Risteski (2021), The Risks of Invariant Risk Minimization** (arXiv:2010.05761)。证明训练环境数 E ≤ 环境特征维度 d_e 时 IRM 全局最优解被非因果解严格支配, 且是"近乎默认状态"; 构造性反例显示伪不变预测器在扰动方向反转时可劣于随机猜测。撞车无, 提供可迁移的环境数/维度比×扰动方向双轴受控设计模板。
- **Kamath, Tangella, Sutherland, Srebro (2021), Does Invariant Risk Minimization Capture Invariance?** (arXiv:2101.01134, R-Score S 24/25)。证明 IRM 线性松弛类别本身系统性选错不变预测器, 与环境数无关; 不变集内选择失败 (骨架正确≠下游 OOD 鲁棒)。撞车无, 与上一条正交, 是独立的第 9 种机制候选。
- **Schölkopf, Janzing, Peters, Sgouritsa, Zhang, Mooij (2012), On Causal and Anticausal Learning** (arXiv:1206.6471)。因果/反因果学习奠基工作, 额外信息是否可用取决于其作用在因果链哪一端, 是 anchor regression minimax 边界的定性雏形。撞车无。

### AD. Anchor regression / Minimax 决策论边界 (本轮最锋利的理论支撑)

- **Rothenhäusler, Meinshausen, Bühlmann, Peters (2021), Anchor regression: heterogeneous data meet causality** (arXiv:1801.06229)。证明扰动作用于 Y/隐混淆 H (而非协变量 X) 时, 非因果方法一致地、可证明地支配因果参数, 即使因果参数完全正确。撞车无, 是理解 idea v3 pilot 结果最核心的一篇理论支撑。
- **How Useful is Causal Invariance for Domain Adaptation in Finite-Sample Settings?** (arXiv:2606.12680)。用 Le Cam 两点法证明匹配的有限样本 minimax 上下界: margin Δ≲1/n 时无算法能从因果不变性获益 (信息论硬下界, 非仅经验观察)。撞车无, 把上一条的定性结论升级为定量下界。
- **Achievable distributional robustness when the robust risk is only partially identified** (arXiv:2502.02710)。把 anchor regression 推广到"测试扰动含未见方向": 只要存在任意强度未见方向分量即存在不可消除的线性惩罚项; 全部扰动落在未见方向时 anchor regression 与 OLS 风险增长率完全相同, K562 真实基因数据验证。撞车无。
- **Bühlmann (2020 Neyman Lecture), Invariance, Causality and Robustness** (arXiv:1812.08233)。综述明确声明 ICP Theorem 1 不提供功效保证; Table 1 因子设计 (模型复杂度×扰动强度×learner) 展示相对增益从 +83% 翻转到 -100.8%, 可直接借鉴为"隔离 Delta"的受控设计模板。撞车无, 是本轮为回答 review 具体问题找到的最直接可用资产之一。

### AE. 因果方法反而占优的边界条件 (互补对照, 证明结论是 regime-dependent 而非单向)

- **Pfister, Bühlmann, Peters (2019), Learning stable and predictive structures in kinetic systems (CausalKinetiX)** (arXiv:1810.11776)。证明环境异质性充分时 (条件 C3 唯一性) 因果方法碾压非因果方法, 一致性定理精确刻画"因果何时赢"。撞车无, 是本轮唯一独立证明"因果法反而占优"边界条件的文献, 与 idea 结果互补而非矛盾。
- **Domain Adaptation by Using Causal Inference to Predict Invariant Conditional Distributions** (arXiv:1707.06422)。提供可直接迁移的 γ(扰动强度 0.1→100)×N(样本量) 受控扫描: γ=0.1 时因果方法无优势甚至更差, γ≥1 时优势单调增大。撞车无, 是本轮回答"如何设计受控对比"最直接的现成模板之一。

### AF. 计算复杂度/信息论视角 (第 4 类独立机制)

- **Is Spurious Correlation Removal Always Learnable?** (arXiv:2606.12930)。证明与样本量无关的计算-统计缺口: 某些不变子空间统计上可识别但多项式时间算法恢复困难 (类比 Planted Clique); 给出显式相变阈值。撞车无, 是"条件化丢信号"的第 4 种独立机制候选 (计算难度而非样本量或功效)。

### AG. 大规模实证审计 (独立佐证 fragility 现象非孤例)

- **Do causal predictors generalize better to new domains?** (arXiv:2402.09891)。16 个真实表格数据集、超 50 万模型: 全特征预测器 16/16 Pareto 支配纯因果特征预测器; ICP/PC/FCI 真实数据产出率近零。撞车无, 是本轮最强的大规模实证佐证之一。
- **Gulrajani, Lopez-Paz (2021), In Search of Lost Domain Generalization (DomainBed)** (arXiv:2007.01434)。近 4.6 万网络严格协议下 IRM 平均分反而略低于 ERM; IRM 唯一决定性胜利建立在信息泄漏的 oracle 调参协议上。撞车无。
- **Empirical or Invariant Risk Minimization? A Sample Complexity Perspective** (arXiv:2010.16412)。证明 IRM 渐近不劣于 ERM 但小样本 (N<10000) 时打平甚至略差, 是 conditioning 有限样本功效损失的实例化。撞车无。

### AH. 横截面理论谱系背景补充 (大部分为重复确认, 排除误读风险)

- 以下 13 篇均为横截面因果不变性/分布泛化理论谱系的补充背景, 逐篇精读后判定与 idea 核心 claim 不撞车, 贡献主要是排除误读风险或提供次要背景: **A causal viewpoint on prediction model performance under case-mix shift** (arXiv:2409.01444, AUC=P(X|Y) 纯泛函, 问题轴不同不可直接套用); **Distribution Shift Is Key to Learning Invariant Prediction** (arXiv:2601.12296, 疑似误用 Fano 引理方向, R-Score C/11); **Domain adaptation under structural causal models** (arXiv:2010.15764, 区分"多环境平均"与"单次多变量条件化"是不同操作轴); **A causal framework for distribution generalization** (arXiv:2006.07433, confounding-removing 干预下因果解 minimax 最优, 纯截面无时序); **Functional structural equation models with out-of-sample guarantees** (arXiv:2503.20072) 与 **Functional worst risk minimization** (arXiv:2412.00412, 两篇是同一期刊修订对, anchor regression 推广到无穷维 Hilbert 空间); **Partial Transportability for Domain Generalization** (arXiv:2503.23605, 反直觉: 纯非因果 descendant 特征可在 worst-case 泛化上击败真实因果 parent set, 关键在机制是否跨域不变而非是否因果); **Bengio et al., A Meta-Transfer Objective for Learning to Disentangle Causal Mechanisms** (arXiv:1901.10912, 参数计数给出第 7 种独立机制: 因果分解仅需 O(N) 重估计, 错误分解需 O(N²)); **Schölkopf et al., Towards Causal Representation Learning** (arXiv:2102.11107, sparse mechanism shift 综述, SMS 是未证明假设非定理); **Adaptive-CaRe** (arXiv:2602.06611, ALARM 低数据场景因果约束 MLP 明显劣于无约束 MLP 的独立负结果); **Causal Feature Selection via Orthogonal Search** (arXiv:2007.02938, 单因素 factorial ablation 模板); **Two generalizations of Markov blankets** (arXiv:1903.03538, 形式化区分预测最优 Markov blanket vs 干预导向因果集); **Improving TabPFN's Synthetic Data Generation by Integrating Causal Structure** (arXiv:2603.10254, 真实 DAG 已知时因果条件化稳定获益, 但换成 PC-stable 发现的图后收益消失甚至转负——"因果图本身被估计"这一层的独立佐证)。

### AI. Local Independence 支线 (连续时间/点过程原生因果发现理论, 全新方向, 高强度 novelty 证据)

<!-- 这条支线由 round5 的 2001.06208 触发, round6-7 系统性追索, 结论是一个稳定、可作 novelty 证据的文献空白。 -->

- **Peters, Bauer, Pfister (2022), Causal models for dynamical systems** (arXiv:2001.06208)。概念性框架, 非正式断言"faithfulness 假设在动力系统中似乎不再站得住脚", 触发本支线。撞车无。
- **Conditional Local Independence Testing for Itô processes** (arXiv:2506.07844)。连续时间 Itô 过程 local independence 检验, 关键渐近机制要求样本量 N→∞ 与时间步长 δ→0 联合发生, 单纯增大 N 会导致 Type I error 发散——与离散时间因果发现渐近理论的本质区别。撞车无。
- **Mogensen, Malinsky, Hansen (2022), Faithful graphical representations of local independence** (arXiv:2310.13796)。Local independence 图 faithfulness 公理化 (transitivity conditions D0-D3); 全文不讨论非平稳/趋近分岔; 数值实验显示 PC 类"多个小条件集"策略 (与 PCMCI+ 邻接搜索同构) 比单次大条件集更易因 (近) faithfulness 违反出错。撞车无。
- **Local Independence Testing for Point Processes** (arXiv:2110.12709)。Hawkes 点过程边缘化 (潜变量) 会破坏一阶局部独立检验的渐近水平 (非仅功效); 全文限定平稳过程。撞车无。
- **Nonparametric conditional local independence testing** (arXiv:2203.13559)。首个 CLI 非参数检验, 给出比"趋近分岔"更精确的机制候选——non-collapsibility (对满足 CLI 的完整系统的边缘化子系统做参数化 CI 检验会完全丧失渐近水平)。撞车无。
- **Weak equivalence of local independence graphs** (arXiv:2302.12541)。局部独立图 Markov 等价判定是 coNP-complete (经典 ADMG 是多项式); 全文确认零涉及 bifurcation/非平稳/faithfulness。撞车无。
- **An Asymmetric Independence Model for Causal Discovery on Path Spaces** (arXiv:2503.09859)。自我定位为 local independence 的替代框架 (批评其"除计数过程外无法实践检验"); stationary-only, 对"更干净受控对比"问题是明确负结果。撞车无。
- **Detecting Mutual Excitations in Non-Stationary Hawkes Processes** (arXiv:2601.11717)。标题精确命中"非平稳 Hawkes 过程", 但读全文确认与 local independence 框架基本脱节; stability margin/separation 参数被要求固定不退化, 未讨论其趋于临界值时的行为。撞车无。
- **Graphical modeling of stochastic processes driven by correlated errors** (arXiv:2005.07568)。mu-separation 最完整技术展开; 未发表的删除草稿章节中"非平稳"仅指初始条件非平稳分布采样, 非动力学参数随时间变化。撞车无。
- **Didelez (2008), Graphical Models for Marked Point Processes based on Local Independence** (arXiv:0710.5874)。**Local independence 奠基论文**, Extensions 一节明确提出应否放松"图结构对所有 t 恒定"这一假设, 称其"deserves further investigation"——19 年后 (含本轮 2310.13796) 仍未被解决。撞车无, 是本轮最强的 novelty 证据: 领域奠基作者自己标记的空白至今未填补。
- **Dynamic Structural Causal Models** (arXiv:2406.01161)。SDE 严格映射为 σ-分离图; 919 行全文确认零涉及非平稳/参数漂移/分岔/临界。撞车无。
- **Røysland, Ryalen, Nygård Evans, Didelez (2022), Graphical criteria for identification of marginal causal effects in continuous-time event-history analyses** (arXiv:2202.02311)。Didelez 本人参与的最新延伸, 附录提供唯一的正式 faithfulness 违反反例, 但是单一静态代数构造非趋近临界值式渐进坍缩; 与 2310.13796 互相点名, 三角闭环验证空白确实存在。撞车无。
- **THPs: Topological Hawkes Processes for Learning Causal Structure on Event Sequences** (arXiv:2105.10884)。把 local independence 沿空间维度扩展, 但拓扑图与强度参数仍预设时间不变; 44 篇 2024-2026 反引文全部沿拓扑/神经网络/潜混淆维度延伸, 无一涉及非平稳性。撞车无。

### AJ. 精读后判定低相关 (红线要求全读, 结论是排除而非补充)

- **Complete Causal Identification from Ancestral Graphs under Selection Bias** (arXiv:2603.26301)。纯理论 selection bias/PAG 可识别性论文, S2 中零引用零被引, 与本课题机制完全不同轴, 大概率是关键词命中的边缘论文。撞车无。

### AK. LKIF conditioner benchmarking (second-estimator reconstruction, [idea-refiner, 2026-07-11])

- **Siddique, Rong, et al. (2026), "Benchmarking Conditioners in Liang–Kleeman Information Flow: Application to Land–Atmosphere Interactions"** (submitted to Earth System Dynamics; code at https://github.com/FareehaSiddique/Conditioners-LKIF, no arXiv ID found). Problem: multivariate LKIF estimates change depending on which additional variables are conditioned on (mediators, moderators, confounders), and no prior work systematically benchmarks which conditioner choice is appropriate for a given causal role. Method: (i) a synthetic 5-variable VAR with a hidden confounding layer (`x4/x5`) feeding two observed nodes, used to compare bivariate vs. conditioned (multivariate) LKIF divergence (`Delta-IF`) as confounding strength is swept on a fixed grid; (ii) four diagnostic indices (Mediator Dominance Index, Moderation Gain, Confounding Pressure, Convergence Rate) computed on real land-atmosphere station/reanalysis data (soil moisture, LAI, VPD, temperature, shortwave radiation). Results: `Delta-IF` grows systematically with hidden confounding strength in the toy model (their Appendix A); on real data, adding conditioners changes the sign/magnitude of soil-moisture-to-GPP information flow depending on which of VPD/temperature/shortwave radiation is included. Relevance: this is the closest existing precedent for a confound-strength sweep applied specifically to LKIF conditioning, but it is a static (fixed-coefficient) benchmarking design for choosing conditioners in one time series, not a sliding-window or tipping-point/early-warning design — no time-varying coefficients, no event/control contrast, no instability-as-signal framing.
- **Rong, Liang et al., `LK_Info_Flow` official Python package** (https://github.com/YinengRong/LKIF, MIT license, `codes/python/LK_Info_Flow/`). Not a paper but the reference implementation of multivariate normalized Liang-Kleeman information flow (Liang 2014/2016/2021, already in this landscape as arXiv:2104.11360), including a documented sliding-window/time-varying-causality example (`examples/case8_time varying.py`). Relevance: confirms the closed-form multivariate LKIF estimator is a genuinely reusable, publicly available second estimator for windowed causal-instability audits; the upstream code currently breaks under NumPy >= 2.0 (`np.mat` removed), which any downstream reuse needs to work around.

## [idea-refiner, 2026-07-12]

- **Ganesh S., Beucler, Tam, Gomez, Runge, Gerhardus (2023), "Selecting Robust Features for Machine Learning Applications using Multidata Causal Discovery"** (arXiv:2304.05294; published as Environmental Data Science 2:e27, 2023). Problem: standard feature-selection methods for ML prediction models often keep causally spurious predictors, hurting generalization when domain knowledge is limited. Method: a "Multidata" causal feature-selection framework (M-PC1/M-PCMCI, built on Tigramite) that pools an ensemble of time-series datasets to find one shared set of causal drivers, then feeds those features into downstream regression/random-forest models. Results: applied to Western Pacific tropical-cyclone intensity prediction, the causal feature set (31 features, out of thousands of candidates) outperformed random selection, lagged correlation, and XAI-based feature selection, with M-PC1 slightly ahead of M-PCMCI; more stringent independence-test thresholds removed more spurious links and improved generalization to unseen storms. Relevance: this is the direct precursor to the already-logged 2510.02050 (Multidata-PC SHIPS) by an overlapping author group, and is the closest existing precedent for the general question "do causal-discovery-derived features improve downstream ML prediction skill" — but its mechanism is feature *pruning* (drop non-causal predictors before training), not feature *fusion* (combine a causal-instability score with an existing feature set via a weighted/learned combiner).

- **Flora, Varga, Potvin, Lang (2026), "Developing Machine Learning-Based Watch-to-Warning Severe Weather Guidance from the Warn-on-Forecast System"** (arXiv:2603.20250). Problem: ML post-processing of convection-allowing model output for severe-weather hazards (hail, wind, tornado) has shown promise at very short lead times (0-3h), but skill at the 2-6h "watch-to-warning" window is underexplored. Method: a grid-based ML framework (histogram gradient-boosted trees and a U-Net) trained on Warn-on-Forecast System (WoFS) ensemble forecasts (108 days, 2019-2023 NOAA Hazardous Weather Testbed Spring Forecasting Experiments) to predict probability of a severe hazard within 36 km of each grid point, 2-6 hours ahead. Results: both ML models beat a calibrated updraft-helicity baseline, especially at higher probability thresholds; HGBT gives the best sharpness metrics but caps near 60% probability, while the U-Net extends to 100% but is spatially smoother. Relevance: non-causal application-domain background confirming that grid/ensemble-based ML post-processing genuinely improves on a physical baseline for severe-convective guidance at the lead times relevant to this topic; no causal discovery or feature-fusion framing.

## [idea-reviewer, 2026-07-12]

- **Correction to the existing 2510.02050 (Multidata-PC SHIPS) entry above**: full-text re-check (main.tex, not abstract) shows the paper's central experiment is feature *augmentation*, not pruning alone. The paper explicitly constructs "SHIPS+" = the original 21 operational SHIPS predictors *plus* newly-discovered causally-selected predictors, and directly compares models trained on "original SHIPS" vs. "SHIPS+" (augmented) predictor sets across cross-validation folds and multiple forecast lead times, then validates the added predictors operationally on independent real 2022-2024 North Atlantic TC forecasts (percent MAE reduction relative to the pre-existing operational baseline, "SHIPS forecasts improved at all forecast times"). This is a real precedent, with real operational data and a real skill metric, for the general question "does augmenting an existing strong operational predictor set with causal-discovery-derived features beat the existing set alone" — closer to a fusion/augmentation framing than the earlier entry's "selects predictors" phrasing suggested. The remaining gap relative to any sliding-window causal-instability-diagnostic idea is narrower than previously stated: not the augmentation framing itself, but the specific instability/dispersion-based feature construction, the severe-convective (not tropical-cyclone) domain, and the sliding-window/regime-shift setting.

- **Yu, Liang (2026), "SpatioTemporal Causal Network Diagnostics for Geographic Tipping Point Early Warning" (ST-CND)** (arXiv:2606.17553). Problem: classical spatial early-warning indicators (Moran's I, fixed-neighborhood AR(1)/variance EWS) are diluted by spatially correlated noise and Euclidean-neighborhood assumptions when a tipping event is triggered in a small subregion. Method: reconstructs a directed information-flow topology via transfer entropy (replacing fixed spatial neighborhoods with a data-driven causal graph), estimates local recovery-rate decay via subgraph-constrained dynamic mode decomposition, and combines three signals (internal fluctuation, internal synchronization, external decoupling) into a single "ST-CND" vulnerability score per candidate subnetwork; validated on synthetic bifurcations/reaction-diffusion fields and two observational SST benchmarks (Indo-Pacific, North Atlantic AMOC 1870-2022). Results: ST-CND matches classical methods on Indo-Pacific SST and outperforms Moran's-I/recurrence-network/lambda-AR1/DL-EWS baselines on the AMOC task (AUROC 0.783, critical-subnetwork IoU 0.378). Full-text check (tex read, not abstract-only): the paper's headline comparison structure is a standalone competition (ST-CND vs. four separate classical/DL baselines in a head-to-head benchmark table), the same "does the causal-derived diagnostic win alone" framing as most prior work in this landscape; a secondary "auxiliary logistic readout" (explicitly labeled "for interpretability analysis only", not a headline confirmatory result) trains a joint classifier on an expanded library that includes both the causal-network features and conventional EWS/RQA statistics together, but this is used only to inspect feature-importance weights, not to test with a preregistered decision rule whether the fused feature set beats the conventional-EWS-alone baseline. Domain is geographic/spatial tipping points (ecosystems, ocean circulation, ice sheets), not severe convective storms, and the causal component is a spatial information-flow graph, not a sliding-window proxy-vs-sham conditioner-sensitivity score.

- **Wang, Varambally, Watson-Parris, Ma, Yu (2025), "Discovering Latent Causal Graphs from Spatiotemporal Data" (SPACY)** (arXiv:2411.05331, ICML 2025; full text read via main.tex + sections/introduction.tex + sections/experiments.tex, not abstract-only). Problem: causal discovery on high-dimensional gridded spatiotemporal data is hard because conditional-independence tests scale poorly and spatially proximate grid points are highly correlated, obscuring true long-range causal links (e.g. climate teleconnections). Prior two-stage approaches (Varimax-rotated PCA + PCMCI, i.e. "Mapped-PCMCI") do dimensionality reduction independent of the causal structure. Method: a variational-inference framework that jointly learns spatial factors (RBF-kernel-parametrized aggregation of grid points into latent time series) and a lagged/instantaneous causal graph over those latents in one end-to-end objective; proves identifiability of the latent factors (up to permutation/scaling) in a continuous-spatial-domain limit under a nonlinear invertible generative assumption. Results: outperforms PCMCI/Mapped-PCMCI/other baselines on synthetic grids up to 250x250, and recovers known real-world climate teleconnections (MJO, NAO, AAO subgraphs) on monthly-scale global precipitation/temperature/SST-type gridded data, with causal lags of 1-2 months. Relevance: this is a second (after Mapped-PCMCI) concrete precedent that "causal discovery at climate-grid scale" is an active, competitive sub-field, but it operates on a purely horizontal 2D grid (no vertical/pressure-level axis), at monthly teleconnection timescales (not hourly/daily severe-convective timescales), and its contribution is a better causal-*structure*-recovery method/identifiability theory, not an instability-as-signal diagnostic, and not a fusion-vs-raw-baseline warning-skill test.

- **Sha, Sobash, Gagne (2023), "Generative ensemble deep learning severe weather prediction from a deterministic convection-allowing model"** (arXiv:2310.06045; abstract-verified via arxiv-tools). Problem: probabilistic severe-weather (tornado/hail/wind) prediction from a single deterministic convection-allowing model (CAM) run, without the cost of a full physical ensemble. Method: a conditional GAN generates synthetic CAM ensemble members from a deterministic HRRR 1-24h forecast, then a CNN converts the synthetic ensemble into severe-weather probabilities; trained/evaluated on real HRRR forecasts against real SPC severe-weather reports (2021 test set). Results: up to 20% Brier Skill Score improvement over other neural-network reference methods; ensemble spread is meaningful (if overconfident) for distinguishing good/bad forecasts. Relevance: non-causal, but a concrete real-data precedent that HRRR-input, SPC-labeled, grid-based ML severe-weather post-processing already achieves a fairly strong, well-benchmarked operational skill level — this is the class of "strong raw-history/operational baseline" that any causal-feature-fusion claim in this topic would need to beat, not merely a naive persistence baseline.

- **Feldmann, Beucler, Gomez, Martius (2024), "Lightning-Fast Convective Outlooks: Predicting Severe Convective Environments With Global AI-Based Weather Models"** (arXiv:2406.09474; abstract-verified via arxiv-tools). Problem: assessing whether AI-based global medium-range weather models (GraphCast, Pangu-Weather, FourCastNet) can skillfully forecast the vertically-resolved dynamic/thermodynamic combinations (instability, shear) needed for severe-thunderstorm-environment prediction, not just single-level variables. Method: process-based evaluation of the three AI models' convective-parameter forecasts (up to 10-day lead time) against reanalysis and ECMWF IFS, via case studies and seasonal analysis. Results: GraphCast and Pangu-Weather match or exceed IFS skill for instability and shear parameters. Relevance: non-causal, but confirms that "vertical atmospheric structure for severe-convective environment assessment" is already an active, real-data research question in the AI-weather literature (via forecast-skill evaluation of AI emulators), independent of and prior to any causal framing; relevant background for judging how crowded the general "vertical structure matters for severe convection" observation is.

## [idea-reviewer, 2026-07-12]

<!-- v9 审查阶段补充搜索 (WebSearch + arXiv abstract 页 WebFetch 核验, 非全文精读, 因两篇均非因果发现/因果特征方法, 与本 topic 核心因果机制不构成直接撞车). -->

- **(2025), "Towards mechanistic understanding in a data-driven weather model: internal activations reveal interpretable physical features"** (arXiv:2512.24440; abstract-verified via WebFetch)。把稀疏自编码器 (sparse autoencoder, 借鉴 LLM 可解释性技术) 应用于 GraphCast 中间层的神经元空间, 发现热带气旋、大气河流、降水模态、季节行为等物理可解释特征自发涌现于内部激活中; 案例研究显示操纵气旋特征可对演化中的飓风产生物理一致的、可解释的模型输出变化。方法聚焦机制可解释性本身, 摘要未显式测试这些内部特征相对模型自身原始输出是否带来增量预测技能。撞车无 (无因果发现、无 (D,J) 式不稳定性诊断、无预警增量检验)；作为背景资料, 独立佐证"已训练的端到端天气模型内部会自发学到物理有意义的结构化表征"这一前提在气候/天气 AI 文献中已有其他证据支持。

- **Flora, Varga, Potvin, Lang (2026), "Developing Machine Learning-Based Watch-to-Warning Severe Weather Guidance from the Warn-on-Forecast System"** (arXiv:2603.20250; abstract-verified via WebFetch)。用直方图梯度提升树 (HGBT) 与 U-Net 深度学习模型, 基于 Warn-on-Forecast System (WoFS) 对流许可模式输出, 预测 2-6 小时内的冰雹/大风/龙卷等强对流灾害, 相对传统气象基线方法有改进。方法非因果, 是"直接在对流许可模式原始场输出上训练的 ML (含 U-Net 真正在网格张量上端到端训练)"用于 2-6h 强对流灾害预警的真实业务化先例。撞车无。相关性: 提供了一个真实存在的、在原始格点场 (而非人工聚合统计特征) 上端到端训练的强对流预警模型族 (U-Net) 的具体参照, 可作为该 topic 下任何"trained raw-field baseline"设计阶段的具体基线家族候选。
