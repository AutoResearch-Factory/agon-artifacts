<!-- 书写报告使用中文 -->
# Deep Literature Review — causal-scs-indicator (idea scope) — 2026-07-13

<!-- 本轮由 proposal v1 review (verdict: REVISE) 触发, 聚焦其三个 CRITICAL 缺口: (1) HRRRCast/StormCast 能否作 P2 raw-only 骨干的预训练初始化; (2) n>=50 storm systems 小样本功效分析与跨 prevalence 的 AUPRC 比较陷阱; (3) 因果/物理诊断特征与预训练大模型融合的现有先例. 本轮开工前已加载 already_read_ids (189 篇, 覆盖此前 4 轮 idea-scope + 3 轮 topic-scope), 本轮 23 篇全部在此之外, 0 篇重复. reader=claude(opus-4-8; 配置的 deepseek CLI claude-ds 本机未安装, 按约定 fallback, 已告知用户). 每篇均有全文 wiki: $ARXIV_WIKI_DIR/<id>.md 含 "Read by: 0710-causal-scs"(与 topic-scope/此前 idea-scope 共用去重 key). -->

## 循环统计

| 轮次 | 触发方式 | 候选池 | 选中读全文 | Wiki 写入 | 本轮方向 |
|------|---------|--------|-----------|----------|---------|
| 1 | 用户指定2篇(HRRRCast/StormCast) + B1 axis 搜索 + B0 web search | ~30 | 9 | 9 | 缺口(1)(2)(3) 首批: HRRRCast/StormCast 架构核查 + PEFT/frozen-decoder 先例 + WoFS 校准谱系 + Williams PR-imbalance + TabPFN-CFM |
| 2 | B7 衍生(cited HRRRCast/StormCast 发现 Stormscope successor) + B0 web search | ~20 | 8 | 8 | Stormscope 三方对比 + TC强度post-processing类比 + 4篇统计学bootstrap/评测方法论 |
| 3 | B7 衍生(ERO 反引文发现 Partial VOROS) | ~10 | 3 | 3 | Partial VOROS 早期预警cost-aware指标 + MacKinnon summclust杠杆诊断 + 聚类层级检验 |
| 4 | B7 衍生(三篇独立读者一致点名 MacKinnon 2026新作) | ~5 | 2 | 2 | "何时信任cluster-robust推断"综述 + 数据驱动聚类发现 |
| 5(收尾) | B7 衍生(references列表发现, 用户未指定但判断为CRITICAL遗漏) | 1 | 1 | 1 | McDermott et al. 2024 (Tier S, NeurIPS, 155引用): 反驳"AUPRC总优于AUROC"这一广泛误引说法 |
| **合计** | | | **23** | **23** | |

B7 反向扩展总调用数 >=100 次 (references+cited+author-search+title-term-search, 覆盖全部 23 篇, 满足 >=23x4=92 discipline 下限)。S2 持续 HTTP 429 限流, 多次 fallback 到 arXiv 关键词搜索产生噪声(空间天气/粒子物理/纯数学论文), 通过重试与手动核验 arXiv ID 过滤噪声, 未影响核心发现。

B4 饱和判定依据: 第4-5轮 B7 反向扩展已明显收敛到反复出现同一小簇作者(MacKinnon/Nielsen/Webb/Cai 的聚类稳健推断谱系)或完全跑题的噪声(BESIII粒子物理、JUNO中微子探测器、Riemann zeta函数等), 且第4轮读者明确写出"本缺口收尾轮, 四篇文献已收敛, 建议直接采纳该协议"——三个独立信号同时满足饱和判据。

## 已读论文清单（23篇，按三个缺口分组）

### 缺口(1): HRRRCast/StormCast 能否作 P2 raw-only 骨干预训练初始化

| arxiv_id | 一句话 |
|---|---|
| 2507.05658 | HRRRCast(NOAA GSL, Flora/Potvin 共同作者): ResHRRR(~23.5M参数, SE-ResNet+FiLM+DDIM扩散)。精读发现三处非平凡不匹配: 实际训练分辨率6km(非3km)、12个HRRR气压层命中1000/850hPa但缺500/250hPa、**最关键**——单时刻输入单时刻输出的状态仿真器, 零96h历史维度, 需另建时序聚合模块才能用(与proposal"不构建新时空骨干"non-goal冲突)。权重可得性未经证实(GitHub仓库存在但配置与论文有版本漂移)。 |
| 2408.10958 | StormCast(NVIDIA, Pathak): 两阶段回归+扩散UNet, 单体架构无可拆卸的4气压层专用编码器, 唯一形状匹配分支是粗分辨率ERA5插值场。无权重开源声明。 |
| 2601.17268 | Stormscope(Pathak 2026年StormCast后续作): 单阶段DiT+2D邻域注意力, 直接吃卫星+雷达观测。四维度对比HRRRCast/StormCast后**结论更差**: 气压层实质为零(仅Z500标量)、历史仅nowcast 6帧/1h堆叠(远不足96h)、三者均无确认checkpoint。**不能解决HRRRCast的根本性不匹配**。给出P2骨干最终排序建议：自建自监督预训练 > HRRRCast > StormCast > Stormscope(仅架构模式参考)。 |
| 2509.22020 | WeatherPEFT: Fisher信息引导选择性微调模块(SFAS)架构无关、可迁移；但另一模块TADP硬依赖patch-embedding transformer结构(Aurora/Prithvi-WxC), 明确未验证CNN/GNN架构(HRRRCast/StormCast恰在此列)。三个下游任务全是稠密空间场回归, 无标量二分类先例。 |
| 2509.03816 | Prithvi WxC(2.3B参数)冻结编码器-解码器+卷积三明治微调重力波参数化：小样本正面证据(全面超从零训练baseline)，但反向传播需穿过整个2.3B冻结栈，微调GPU小时(208h)反高于baseline(110h)——"冻结"不等于"廉价"。 |
| 2405.17455 | WeatherFormer: 预训练小样本优势方向一致(-26% RMSE)但模态错配(1D逐点非空间网格)；且自身消融显示该优势在小样本+大容量下会萎缩——直接挑战"小样本=预训练自动赢"简化逻辑。 |
| 2506.19088 | Aurora冻结骨干+轻量MLP decoder微调未见变量：省算力具体数字(50%时间/35%显存/77x FLOPS)可直接引用；但对与预训练目标弱相关的变量(土壤湿度动态、径流)近乎完全失败——强对流微物理信号可能落入同一风险类别。 |
| 2508.17903 | TC强度post-processing(Pangu/FourCastNet v2输出+线性/MLP/CNN/UNet四级head)：**a fortiori证据**——样本量大于我们的场景下CNN/UNet反而在验证/测试集过拟合、跑不赢简单MLR/ANN，作者诚实报告这一反直觉负面结果。tracking-independent固定窗口+位移分位数设计为storm-relative坐标缺口提供现成退化方案。 |

### 缺口(2): 小样本功效分析/storm-cluster bootstrap/AUPRC跨prevalence比较陷阱

| arxiv_id | 一句话 |
|---|---|
| 2012.00679 | Flora/Potvin(2020) WoFS storm-track ML校准旗舰论文：bootstrap单位是"track×30min-lead"example(8万-35万)非唯一风暴，非独立性仅用非正式抽查搪塞——**证实整条谱系(→WoFSCast→2603.20250)从未解决storm-cluster推断问题**，我们的方案是真差异化点但不能借此证明n>=50充分。 |
| 2007.01905 | Williams(2020): 严格推导Precision Gain对prevalence精确不变而AUPRC不是；给出AUPrecG/ROC重投影两种可执行修补；诚实标注：该不变性证明假设两总体间仅prevalence不同、ROC本身不变，而regime-迁移holdout存在的意义正是怀疑ROC本身漂移。 |
| 2401.06091 | **McDermott et al.(2024, NeurIPS, Tier S/155引用)**: 反驳"AUPRC类别不平衡下总优于AUROC"这一广泛误引说法。Theorem 1证明AUPRC是逐阈值加权(非标量offset)；Theorem 3证明完美校准下AUPRC最大化几乎必然只优化高流行率子群体——比Williams的修补更根本，说明PR-Gain修补必要但不充分。**具体建议**：同一holdout内raw vs causal比较仍用AUPRC(不受影响)；跨IID/regime-迁移holdout比较应把Delta AUROC提升为并列主指标(AUROC流行率不变性由定理直接证明)。 |
| 0804.4361 | Lee & Lai(2008) 双层嵌套block bootstrap覆盖率校准：block长度调优机制不可移植(storm无可调分辨率参数)，但"嵌套重采样反解覆盖率偏差"算法骨架可原样改写为storm-level iid双层校准；量级旁证——约62个block时未修正覆盖率低6个百分点，校准后腰斩误差。 |
| 2004.04339 | DTA meta-analysis场景bootstrap给AUC(SROC)构造CI：N=10-44(与我们同量级)下CI宽度0.09-0.29，佐证n=8 seed-CI宽度±0.08-0.09并非过虑；但resample机制是参数化重抽新研究，与我们需要的非参数化case-resampling storm-cluster bootstrap算法结构错配。 |
| 2406.00650 | MacKinnon/Nielsen/Webb logit聚类稳健jackknife/bootstrap：线性化技巧为避免非线性MLE重估算而设计，我们两个CNN已冻结、删一storm重算AUPRC天然更廉价，无需照搬；但确认可靠性主因是**阳性cluster数量而非总数**——直接指导功效分析应聚焦风暴阳性标签数。 |
| 2205.03288 | MacKinnon/Nielsen/Webb summclust杠杆/影响力诊断(OLS hat矩阵)：公式依赖回归系数, 无法字面套用到CNN+AUPRC；但发现两个有代数依据的朴素杠杆代理(storm正例占比、样本占比)可预注册用作P2真实数据前的诊断检查；仿真证明杠杆异质性预测wild bootstrap失效。 |
| 2301.04522 | MacKinnon/Nielsen/Webb聚类层级score-variance检验：绑定回归得分框架且仿真从未在粗聚类数<3时验证(我们只有>=2季节，低于门槛)——不能直接验证"storm是否为正确聚类单元"，建议改用低成本day-block/season-block bootstrap稳健性检查并诚实标注该假设未经检验。 |
| 2604.02000 | MacKinnon(2026)"何时信任cluster-robust推断"综述：给出异质性诊断/targeted Monte Carlo/placebo regression三类可操作程序。**综合此前三篇(2406.00650/2205.03288/2301.04522)，可直接组装为storm-cluster bootstrap可信度协议**（详见下方集成建议）。 |
| 2106.05503 | Cai(2021)数据驱动面板聚类发现(阈值化long-run协方差矩阵)：渐近依赖长共享时间轴, 与storm一次性96h窗口结构不匹配；建议以整季日历时间为共享轴重新表述，作附录级robustness check而非替换现有pipeline。 |
| 2510.04979 | 联邦学习ROC/PR曲线计算：统计目标错配(within-pool确定性聚合 vs between-cluster方差估计)，不含任何bootstrap/CI机制，**确认对storm-cluster bootstrap无可复用算法**。 |
| 2510.18520 | Partial VOROS(AISTATS2026, 医院预警precision+capacity约束cost-aware指标)：多边形可行域几何依赖连续阈值曲线不可移植；但3条设计原则可用——穷尽可行域分类(→判据分桶穷尽互斥)、单一边界线(→收敛CI下界0.01与点估计0.02两阈值)、filter-then-rank结构(→BH校正与CI阈值判据排序)。 |
| 2503.14321 | COPA多目标模型排名聚合：需几十至上千候选者(我们只有3-4臂)且主动丢弃幅度信息(正是Delta AUPRC>=0.02依赖的核心数字)——**不适用**，改建议检索非劣效性/优效性联合检验、固定顺序gatekeeping文献。 |
| 2507.15240 | ERO精确重构二分类不平衡直接指标优化：训练时约束优化与评估时prevalence不变性不在同一层面——**不适用**；独立代数验证排除了"固定recall比较precision"这一看似可行的捷径；反引文发现Partial VOROS(本轮已读)。 |

### 缺口(3): 因果/物理诊断特征与预训练大模型融合先例

| arxiv_id | 一句话 |
|---|---|
| 2606.26467 | TabPFN-CFM(ICML2026)：34个tex文件+全部24条参考文献逐条核验，**确认不构成先例**——严格局限于i.i.d.表格截面数据、单模型内部联合学习结构与效应(非"独立因果发现方法算特征→注入另一预训练骨干"两阶段融合)、零引用天气/气候/地球科学基础模型、零引用时空因果发现方法。proposal"该角度目前无人做过"的novelty论证完整存活，且是针对最可能的候选对象(通用因果基础模型)核验后的结论，证据强度高于此前仅凭关键词搜索。 |

## 撞车风险评估

| Proposal核心缺口 | 是否被填补/覆盖 | 证据 |
|---|---|---|
| 缺口(1) HRRRCast/StormCast作P2骨干 | **部分负面**：两者均非"拿来即用"，各有具体架构级不匹配(尤其HRRRCast无96h历史维度这一根本问题)；但获得清晰的骨干选型排序与至少3个"冻结/微调"设计原则(浅探针、成本预期、失败模式) | 23篇中9篇直接针对此缺口 |
| 缺口(2) 小样本功效/AUPRC-prevalence陷阱 | **真实、未被现成方案解决，但已组装出可执行协议** | 23篇中13篇直接针对此缺口，含1篇Tier S级更根本性发现(McDermott) |
| 缺口(3) 因果特征+预训练模型融合先例 | **确认无人做过**(novelty论证增强) | 1篇针对性精读+反向引文网络核验 |

无一篇构成对idea/proposal核心claim（`(D,J)`能否为同历史同容量的已训练raw-4D CNN提供增量）的novelty撞车——本轮全部23篇性质是"方法论/工具/先例调研"而非竞品占位。

## 可用新工具/方法（供proposal下一版直接采纳）

**缺口(1) P2骨干选型的具体建议：**
1. 最终排序：自建自监督预训练(在训练折自己的96h/4层/3km窗口做轻量masked-reconstruction/next-step预训练) > HRRRCast(需自建时序聚合模块，risk: 违反non-goal) > StormCast(无可拆卸编码器) > Stormscope(仅架构参考)。
2. Probe head复杂度：默认锚定线性/1-2层MLP而非另一个CNN(2508.17903 a fortiori证据)。
3. Storm-relative坐标：可复用2508.17903的tracking-independent固定窗口+训练集位移分位数掩码设计，作为proposal未操作化的P1.5缺口的现成退化方案。
4. 若选择微调路径：预注册"训练折内部验证选择骨干强弱"步骤不可省略(2506.19088证明无先验判据可提前判断成败)。

**缺口(2) storm-cluster bootstrap可信度协议(综合4篇MacKinnon/Nielsen/Webb/Cai文献 + 1篇Tier S级AUPRC理论)：**
1. 训练前花名册检查：报告G(storm总数)与G1(阳性storm数)，功效分析应聚焦G1而非G。
2. 留出后廉价诊断：leave-one-storm-out杠杆检查(用storm正例占比、样本占比作代理)，成本低于常规cluster-jackknife(因两个CNN已冻结，删一storm重算只需过滤+重排序)。
3. 覆盖率校准：双层嵌套bootstrap(外层resample storm得ΔAUPRC*分布，内层子重采样估计并修正实际覆盖率偏差)，参照0804.4361量级预期(50-100storm规模下覆盖率偏差可能达6个百分点)。
4. 聚类层级诚实标注：不做形式化检验(样本量不足)，改用day-block/season-block bootstrap作稳健性检查，明确声明"storm是正确聚样单元"这一假设本轮未经证明。
5. **AUPRC主指标修正(最重要)**：同一holdout内raw vs causal比较保留AUPRC；跨IID/regime-迁移holdout比较增设Delta AUROC为并列主指标(prevalence-invariance有定理保证)；报告两个holdout实际prevalence比值；"regime-迁移Delta应大于IID Delta"判据需两个指标同向印证。
6. 判据分桶穷尽化：借鉴Partial VOROS的穷尽可行域分类思路，显式命名"CI下界>0.01但点估计<0.02"等中间情形；收敛CI-阈值(0.01)与点估计阈值(0.02)为单一边界。

**缺口(3)：** 无新工具，确认novelty空间完整。

## Errors

无tex下载失败。S2持续HTTP 429限流导致多次retry, 部分search fallback到arXiv关键词匹配产生大量噪声(空间天气/粒子物理/纯数学论文)，均通过人工核验排除，未误读入撞车候选。references/cited 少数几次即便retry仍返回空(2509.22020的references/cited、2007.01905的cited首次)，已尽力retry后接受为S2限流导致的真实局限，不影响B7 discipline下限达标（>=92次调用，实际完成>=100次）。

<verdict>

核心 claims 逐一评估（对照 idea Problem Anchor / proposal Claim 1 与 Claim 2）：

- Claim 1（`(D,J)`能否为同历史、同容量、独立冻结架构的raw-4D CNN在真实storm-level数据上提供跨季节可复现的AUPRC增量）：**未被覆盖**。本轮23篇全部是方法论/工具/先例文献，无一篇直接研究因果时序不稳定性特征对原始4D强对流预警CNN的增量价值，也无一篇提出竞争性的"causal-instability-as-warning-signal"框架应用于强对流领域。
- Claim 2（Gate 0架构盲选+特权匹配对照的必要性验证）：**未被覆盖**。无相关竞品。
- 缺口(1)(2)(3)本身（非idea核心claim，而是proposal review提出的具体待办）：如上表，缺口(1)(2)得到部分负面但可操作的答案，缺口(3)得到确认性排除，均不构成对idea/proposal的novelty威胁，而是帮助下一版proposal把三处CRITICAL弱项修补得更扎实。

总体：0 个核心 claim 被覆盖 → 文献调研通过，idea/proposal 的 novelty 空间未受本轮23篇文献冲击。但本轮发现的 McDermott et al. (2401.06091) 对 AUPRC 主指标选择的理论质疑强度较高（Tier S, NeurIPS, 155引用），建议 proposal 下一版必须正面回应，而非仅作背景引用。

</verdict>
