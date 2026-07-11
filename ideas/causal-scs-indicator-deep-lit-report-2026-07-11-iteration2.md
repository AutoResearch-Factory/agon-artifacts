# Deep Literature Review — causal-scs-indicator (idea scope, topic=0710-causal-scs) — 2026-07-11 iteration 2

reader model: 配置为 `deepseek` (claude-ds), 但本机未安装 → 按 env 规则 fallback 到同接口 `claude` (opus-4-8, --effort max)。dispatcher=claude。这是本 idea 第 2 次运行 idea-scope deep-lit-tick (第 1 次 2026-07-11 早些时候已读 26 篇, 判定饱和); 已读记录与 topic-level iteration 1 (24 篇, 2026-07-10) + topic-level iteration 2 (34 篇, 2026-07-11) + 本 idea iteration 1 (26 篇) 共享同一 wiki 池 (`$ARXIV_WIKI_DIR`, 以 `topic_slug=0710-causal-scs` 去重), 开工前 grep 得 84 篇已读 arxiv_id, 全部跳过。本轮聚焦用户指定的 v2 idea 四个具体设计点: 双族 D/J profile、synthetic stop gate、matched negative controls、PCMCI+/LKIF。

## 循环统计

| 轮次 | 搜索 | 候选来源 | 选中读全文 | Wiki 写入 | 主要新关键词/信号来源 |
|------|------|---------|-----------|----------|---------|
| R1 | B0: 3 web + 2 corner-case (openreview/github) + B1: 8 arxiv (6 axes 锚定四个设计点) | ~140 arxiv 结果 + openreview 命中 TCD-Arena (经 info 核验为真) | 9 | 9 | TCD-Arena (33 类假设违背基准) 为本轮最高价值发现 |
| B7-R1 | round1 9篇 × 4 (references+cited+author+title) = 36 调用 | ~150 条引文/反引文/作者/标题结果 | — | — | VCDF 姊妹篇线索 (RCV-PCMCI 作者)、TCD-Arena 反引文命中 Causal-Audit (已入库, 印证子领域正在 idea 周边形成) |
| R2 | 4 条 TCD-Arena 具名引用追猎 (Faltenbacher/Schkoda/Machlanski/Poinsot) + 2 条 keyword 搜索 | Schkoda→已读LOVO确认, Ferdous→已读TimeGraph确认, Machlanski→新论文 2310.18212 确认, Poinsot/Faltenbacher 本轮未解析 (见 Errors) | 5 | 5 | Butler"因果Nyquist"窗口敏感性实证、Stein/Denzler 因果基础模型立场论文 |
| B7-R2 | round2 5篇 × 4 = 20 调用 | ~100 条引文/反引文/作者/标题结果 | — | — | VCDF (Gene Yu 团队 2026-02 更新版) 浮现——同一作者簇后续作 |
| R3 | round2 B7 派生 2 高信号候选 (Wahl&Runge 图距离度量 + DOTS) | 2 候选核验后有效 | 2 | 2 | Wahl&Runge 论文内提及 2502.14719 (Faltenbacher 本人 2025 新作, 解开 R2 未解析的具名引用) |
| B7-R3 | round3 2篇 × 4 = 8 调用 | ~40 条结果, 均落入已读集或明确领域外 | — | — | 无新增 |
| R4 | 追猎 2502.14719 (Faltenbacher/Wahl/Herman/Runge, PCMCI+团队本人 coherency score) | 1 篇, 高价值 (PCMCI+ 内部自洽性诊断) | 1 | 1 | 反引文/正文提及 Miersch(PCMCI+真实流域评测)、Gamella(Causal Chambers)两篇高价值但无 arXiv ID 文献 |
| B7-R4 | 1篇 × 4 = 4 调用 | ~20 条结果 | — | — | 无新增 (Miersch/Gamella 确认均为 DOI-only, 见 Errors) |
| R5 | 追猎 2412.10039 (Petersen, 随机猜测负对照评估因果发现) | 1 篇, 高价值 (matched negative controls 直接方法论) | 1 | 1 | — |
| B7-R5 | 1篇 × 4 = 4 调用 | ~55 条结果, 全部落入已读集或已评估-低优先级候选 (2406.19503/2510.26485/2305.09565/2502.06232) | 0 新增 | 0 | B4 饱和确认 |
| **合计** | | | **18** | **18** | **B4 饱和** |

**B4 饱和判据**: (1) 候选池逐轮系统性收窄 (9→5→2→1→1), 且从 R4 起每轮 B7 反向扩展只产出 1-2 个新候选, 全部落入同一高度已覆盖的作者簇 (Runge/Wahl/Faltenbacher/Machlanski/Butler/Gideon-Stein/Petersen — 事实上是 causal discovery robustness/fragility 这个具体子领域的几乎全部活跃作者); (2) R5 的 B7 反向扩展 (4 调用) 产出的 4 个候选经 info 核验后均为同簇内已充分代理覆盖的次要变体 (温和相关但不构成独立新发现), 无一构成 must-read; (3) B3 筛选标准 (claim 直接交集/survey优先/2026新作/高引用/已知引用列表) 应用于 R5 之后的候选池, 无论文能清过门槛。→ 循环终止。

**红线执行记录**: B7 反向扩展 (references+cited+author-search+title-term-search) 每篇 wiki_written 论文均在 dispatcher 层独立执行, 总计 36+20+8+4+4=72 次调用, 对应 18 篇 wiki_written × 4 = 72, 达标 (≥72, 无缺口)。

## 已读论文清单 (18, 每篇有全文 wiki, 均为已有 84 篇之外的新发现)

| arxiv_id | 简称 | 年 | 撞车 | 相关设计点 | 关键发现 |
|---|---|---|---|---|---|
| 2410.19412 | RCV-PCMCI/VarLiNGAM | 2024 | LOW | D/J profile | Consistency/Variability 与 (D,J) 数学同源, 用途相反 (滤噪声 vs 当信号) |
| 2602.21381 | VCDF | 2026 | LOW | D/J profile | 同上作者后续版; **本人承认未验证跨折不一致是否反映真实结构变化** |
| 2502.14719 | PC 系方法 coherency score | 2025 | LOW | D/J profile + PCMCI+/LKIF | PCMCI+团队本人提出零额外成本内部自洽性诊断, 候选第三诊断通道 |
| 2402.04952 | 分离陈述图距离族 | 2024 | LOW | D/J profile | Wahl&Runge, Tigramite 兼容, 可严格化 J 通道定义 |
| 2605.03045 | TCD-Arena | 2026 | LOW | synthetic stop gate | 33 类假设违背×16 regime×10方法基准, R-Score S/22, 现成正对照+混淆生成器库 |
| 2602.19903 | 因果 Nyquist (窗口/采样率敏感性) | 2026 | LOW | synthetic stop gate | GC/PCMCI/DYNOTEARS 对 (Q,k) 高度敏感实证, 候选第七类混淆源 |
| 2310.18212 | 超参数误设 vs 因果结构学习鲁棒性 | 2023 | LOW | synthetic stop gate | 鲁棒性剖面与 oracle 最优表现近乎正交, 建议 gate 设计固定超参数控制 |
| 2412.10039 | 随机猜测负对照评估因果发现 | 2024 | LOW | matched negative controls | 超几何精确检验+模拟负对照四步管线, 呼应 idea 已有 paired-trajectory 设计 |
| 2210.00528 | DANCE (数据驱动负对照验证) | 2022 | LOW | matched negative controls | vanishing tetrad test 验证横截面负对照, 仅方法论姿态可类比 |
| 2507.14528 | PU-learning 控制组构建 | 2025 | LOW | matched negative controls | 无对照组时构造高置信度对照, 真实数据阶段候选工具(非当前目标) |
| 2510.03959 | 雷暴驱动停电两阶段早期预警 | 2025 | LOW | PCMCI+/LKIF (应用背景) | "causal"=无泄漏特征工程非结构因果; 应用域高度相关但方法不撞车; 自身也是 NEGATIVE 结果 |
| 2508.01848 | TD2C (监督式因果发现) | 2025 | LOW | PCMCI+/LKIF (镜像对照) | 主动消除假设违背求更准图, 方向正相反 |
| 2606.23880 | GRACE (L0 门控精炼骨架) | 2026 | LOW | D/J profile (镜像对照) | bootstrap 多数投票丢弃窗口不稳定性, 坐实 Bagged-PCMCI+=2306.08946 |
| 2402.10300 | 高维分岔嵌入 EWS | 2024 | LOW | 背景类比 | 神经网络 EWS, 经典慢变 regime, "silent catastrophe"概念可类比 |
| 2509.01601 | 无序辅助早期预警 | 2025 | 无 | 背景类比 | 凝聚态物理; 无序拉长预警窗口直觉可类比异质性对指标可靠性影响 |
| 2402.09305 | Causal Pretraining | 2024 | LOW | PCMCI+/LKIF (镜像对照) | 监督端到端映射因果图, 方向正交, 报告诚实(含负结果) |
| 2303.01412 | 超参数/指标选择对 CATE 影响 | 2023 | LOW | 背景(低优先级) | 横截面 CATE 而非时序因果发现, 方法论警示 |
| 2510.24639 | DOTS (扩散排序时序因果发现) | 2025 | LOW | PCMCI+/LKIF (镜像对照) | 多噪声尺度排序软投票求更准结构, 方向正交 |

## 撞车风险评估 (针对用户指定的四个设计点)

| 设计点 | 本轮是否出现直接竞品占位 | 结论 |
|---|---|---|
| 双族 (D,J) profile | 否——2410.19412/2602.21381 的 C/V 统计量数学同源但方向相反(过滤而非利用), VCDF 自陈未验证"真实结构变化"情形, 恰是 idea 要填补的具体空白 | niche 未被占, 且找到迄今最强的"原作者亲口承认未解决"式 novelty 佐证 |
| synthetic stop gate | 否——TCD-Arena/因果Nyquist/超参数鲁棒性三篇均是评测/诊断基准, 未提出"把脆弱性当预警信号"框架 | niche 未被占, 且 TCD-Arena 提供可直接复用的现成 gate 基础设施 |
| matched negative controls | 否——Petersen/DANCE/PU-learning 三篇负对照方法均服务于"因果图/效应估计器评估"而非"事件早期预警", 问题设定不同 | niche 未被占, Petersen 论文的超几何精确检验方法论可直接借鉴 |
| PCMCI+/LKIF | 否——本轮 0 篇 LKIF 相关新发现 (LKIF 支线在 topic-level iteration 1/2 已充分覆盖); PCMCI+ 相关新发现 (TCD-Arena/coherency score/因果Nyquist/Miersch/Gamella) 全部是"PCMCI+ 本身脆弱"的证据而非竞品, 反而加固 idea 的动机 | niche 未被占, 5 篇独立来源现在共同证实 PCMCI+ 在假设违背/真实数据下确实脆弱 |

**总体**: 18 篇中无一篇声称"因果估计不稳定性可作强对流(或任何领域)事件级早期预警指标"这一 idea 核心命题。本轮最大价值不在于排除撞车 (idea 的 niche 此前已被 2 轮共 84 篇充分确认未被占), 而在于**补齐 v2 review 遗留的具体方法论缺口**: (a) VCDF 提供了迄今最直接的"同源统计量、原作者承认未验证"式 novelty 佐证; (b) TCD-Arena 提供了 Stage-1 gate 急需的现成正对照 regime 库; (c) coherency score 提供了零成本候选第三诊断通道; (d) Petersen 的负对照方法论直接呼应 idea 已有设计。

## 可用新工具/方法 (汇总)

- **候选第三诊断通道**: coherency score (2502.14719) — 复用 PCMCI+ 已执行的 CI 检验日志, 零额外计算成本, 含"increasing subsets"启发式可区分统计噪声 vs 结构性假设违反。
- **J 通道严格化候选**: separation-based 图距离族 (2402.04952, parent-SD 等), Tigramite 兼容。
- **Stage-1 synthetic gate 现成资源**: TCD-Arena (2605.03045) 的 33 类假设违背生成器 (含真实 DAG 变化正对照 V_stat/V_coef、缺失/样本量/观测噪声混淆生成器 V_mcar/mar/mnar/V_length/V_obs); 因果 Nyquist (2602.19903) 的双变量延迟滤波器低成本合成设置。
- **候选第七类混淆源**: 窗口长度/采样率错配 (因果 Nyquist, 2602.19903 的相变现象可能被误判为真实结构变化)。
- **matched negative controls 方法论**: Petersen (2412.10039) 超几何精确检验 + 模拟负对照四步管线, 可零成本复用于"PCMCI+/VAR 恢复骨架是否优于随机猜测"附加诊断。
- **HP 控制建议**: Machlanski (2310.18212) 的 BEST/WORST/DEFAULT/SIM_MEAN 四态框架, 建议移植用于隔离超参数漂移与真实窗口驱动信号。
- **待records-only 引用 (无法全文精读)**: Miersch et al. 2025 (PCMCI+ 45 流域真实鲁棒性评测) 与 Gamella et al. 2025 (Causal Chambers, PCMCI+ 近随机猜测) —— 均无 arXiv ID, 无法通过 arxiv-tool 读取全文, 但 S2 abstract 级信息显示与 idea 自身 Stage-1 NEGATIVE 结果高度平行, 建议摘要级引用。

## Errors

- 无 tex_download_failed。全部 18 篇成功读取并写入 wiki。
- S2 API 全程持续 HTTP 429 限流 (与既往会话记录一致的已知模式), search/references/cited 反复触发 arXiv API fallback 或论文自带 .bib/.bbl 提取, 未影响文献真实性 (无编造条目), 已在各 wiki 条目中如实标注。
- **2 个 TCD-Arena 具名引用未解析**: "faltenbacher_internal_2025" 与 "poinsot2025position" 经关键词搜索未能在 R2 阶段定位 (当时误判为不存在); 后续 R4 通过其论文正文交叉引用意外发现 faltenbacher_internal_2025 实为 arXiv:2502.14719 (已读, 见上表), 完整解开; poinsot2025position 全程未能解析, 可能是非 arXiv venue 或过于新未被索引, 如实记录未强行编造。
- **2 篇高价值文献因缺少 arXiv ID 无法通过 arxiv-tool 全文精读**: Miersch/Günther/Runge et al. (2025, DOI:10.1175/aies-d-24-0114.1) 与 Gamella/Peters/Bühlmann (2025, DOI:10.1038/s42256-024-00964-x)。已通过 arxiv_tool.py search 获取摘要级信息(合规操作, 非编造), 但受红线#1"全部文献操作通过 arxiv-tools"限制, 无法下载其 tex 全文, 故不计入 wiki_written 清单, 仅在本报告与 idea 文件中摘要级记录。

<verdict>
核心 claims (C1 事件特异性 / C2 互补性 / C3 边界) 逐一评估:
- C1: 未被本轮新论文直接覆盖 (无竞品声称估计脆弱性本身是预警信号)，但 TCD-Arena/因果Nyquist/Miersch/Gamella 四个独立来源进一步证实 PCMCI+ 在真实假设违背/真实数据下确实存在系统性脆弱性，为 C1 的前提假设提供了比 v2 review 阶段更扎实的外部证据基础。
- C2: 未被覆盖，VCDF/RCV-PCMCI 提供最直接的"同源但反向"参照，其"未验证真实结构变化情形"的自陈局限是 C2 待验证部分的具体空白点。
- C3: 未被覆盖，coherency score 提供了区分"统计噪声"与"结构性假设违反"的候选工具，可能有助于未来对 C3 边界做更细致的操作化。

总体: 18 篇全部未构成 novelty 撞车。**文献调研通过，建议继续推进**，本轮四点具体收获可直接指导下一版 refine：(1) 用 VCDF 的自陈局限作为最强 novelty 佐证之一；(2) 用 TCD-Arena 的正对照/混淆生成器库设计官方 PCMCI+ Stage-1 gate；(3) 评估把 coherency score 接入作为候选第三诊断通道（可能替代或补充当前二维 D/J 设计，需与 codex second opinion 讨论是否引入会削弱"双通道正是为检验 C2"的既有论证）；(4) 引用 Petersen 负对照方法论强化 matched negative controls 段落的统计严谨性；(5) 摘要级引用 Miersch/Gamella 两篇真实世界 PCMCI+ 脆弱性证据（无法全文精读，已如实记录该限制）。
</verdict>
