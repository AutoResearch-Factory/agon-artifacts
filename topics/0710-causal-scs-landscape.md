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
