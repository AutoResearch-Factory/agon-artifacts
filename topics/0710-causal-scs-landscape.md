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
