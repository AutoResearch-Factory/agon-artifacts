# Landscape: 因果效应不稳定性作为强对流预警指标

<!-- 本文件由 idea-reviewer 在审查 causal-scs-indicator idea 时创建 (原 idea-creator 阶段被跳过, 未生成 landscape). 以下条目仅客观描述相关论文做了什么, 不涉及与具体 idea 的关联判断. Append-only, 后续贡献者请勿删改已有条目. -->

## [idea-reviewer, 2026-07-10]

- **Bian, Wang, Leng, Lin, Shi (2025), "Utilizing Causal Network Markers to Identify Tipping Points ahead of Critical Transition"** (arXiv:2412.16235, Advanced Science). 提出 causal network marker (CNM) 框架: 用 Granger causality (CNM-GC) 和 transfer entropy (CNM-TE) 量化节点间方向性因果强度, 用 K-means 将节点分为 dominant group (DG) / non-dominant group (NDG). 理论证明 (线性化系统 + 特征值分解) 当系统趋近 codimension-one 分岔点时, DG→NDG 方向的 Granger causality 强度趋于 0, 而 marker CNM = |DG||NDG| / Σcs(DG→NDG) 相应发散, 以此作为早期预警信号. 在五基因调控网络、生态互惠网络、Turing 反应扩散网络三类 benchmark 及癫痫发作 iEEG 真实数据上验证, 预测精度优于传统 DNB (dynamical network biomarker, 基于方差/自相关). 机制是"真实因果强度随分岔确定性地趋于 0/发散", 而非"因果估计过程本身在假设违背下变得不稳定".

- **NPG (2026), "Quantitative Comparison of Causal Inference Methods for Climate Tipping Points"** (npg.copernicus.org/articles/33/313/2026/, 原预印本 EGUsphere 2025-6258, 未查到独立 arXiv ID). 比较 PCMCI、LKIF (Liang-Kleeman Information Flow)、GCSS (Granger Causality for State Space Models) 三种方法在气候临界点场景 (AMOC-北极夏季海冰范围交互) 上的表现, 数据来自三次非线性随机微分方程生成的分岔动力学合成序列, 用 Matthews Correlation Coefficient 与 ground truth 比较. 核心发现: 方法表现依数据特征分化明显 (LKIF 适合小样本弱耦合, GCSS 适合大样本强信号, PCMCI 中庸但灵活), 且**当分岔事件落在被分析窗口内时, 所有方法都可靠地失效** ("all methods fail reliably when tipping events occur within analyzed data"). 该发现将跨方法分歧/失效视为需要规避的局限性, 而非可利用的预警信号本身.

- **Ruiz, Arana-Catania, Ardila, Ventura (2026), "Causal-Audit: A Framework for Risk Assessment of Assumption Violations in Time-Series Causal Discovery"** (arXiv:2604.02488). 提出 Causal-Audit 框架, 对五类假设族 (stationarity, irregularity, persistence, nonlinearity, confounding proxies) 计算 effect-size 诊断量, 聚合为四个带不确定区间的校准风险分数, 并用 abstention-aware 决策策略, 仅在证据支持可靠推断时推荐使用 PCMCI+ / VAR-based Granger causality. 在 500 个合成 DGP (10 类假设违背) 上校准 (AUROC>0.95), 21 个外部 benchmark (TimeGraph/CausalTime) 上一致. 该框架把"假设违背程度"当作决定是否信任/弃权因果推断结果的风险指标, 而非当作物理系统状态转换的预警信号.

- **Mameche, Cornanguer, Ninad, Vreeken (2025), "SpaceTime: Causal Discovery from Non-Stationary Time Series"** (arXiv:2501.10235). 统一时序因果图发现、regime changepoint 重建、跨空间/时间不变因果关系分区三个任务, 用 Minimum Description Length 构造一致性得分, 在河流径流与生物圈-大气交互真实数据上验证. 提供 regime changepoint 检测能力, 但不涉及"用因果关系不稳定性预测未来临界事件"的框架。

## [idea-reviewer via codex second opinion, 2026-07-10]

- **Yu, Liang (2026), "SpatioTemporal Causal Network Diagnostics for Geographic Tipping Point Early Warning"** (arXiv:2606.17553). 提出 ST-CND 框架: 用 transfer entropy 推断空间节点间信息流拓扑 (替代固定欧氏邻域), 用 dynamic mode decomposition 估计每个候选子网络的局部恢复率, 再结合"高内部波动 + 高内部同步 + 低外部耦合"三signal 识别最脆弱子网络, 以抑制空间相关噪声导致的假警报. 在合成分岔数据及 Indo-Pacific SST / North Atlantic AMOC 两个真实观测 benchmark 上验证, AMOC 任务 AUROC 0.783, 关键子网络 IoU 0.378, 优于 recurrence-network 和 lambda-AR1 基线. 该工作把"因果网络拓扑 + 局部恢复率"组合为空间早期预警指标, 应用于地理尺度的临界点 (生态系统/气候子系统/冰盖), 属于因果网络诊断类早期预警框架谱系中与本 topic 邻近的又一新近工作 (已通过 arxiv-tools 核验存在, 摘要与标题一致).
