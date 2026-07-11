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
