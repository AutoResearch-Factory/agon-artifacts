# Deep Literature Review — 0710-causal-scs — 2026-07-11 (iteration 2)

reader model: 配置为 `deepseek` (claude-ds), 但本机未安装 → 按 env 规则 fallback 到同接口 `claude` (opus-4-8[1m]). dispatcher=claude.

第 2 次对本 topic 运行 deep-lit-tick (第 1 次 2026-07-10 已读 24 篇 [topic scope] + 2026-07-11 idea-scope 26 篇, 共 50 篇已在 wiki 池中去重, 均加入 already_read_ids). 本轮只找新论文。

## 循环统计

| 轮次 | 搜索 | 新论文候选 | 选中读全文 | Wiki 写入 | 主要新关键词/候选来源 |
|------|------|-----------|-----------|----------|---------|
| R1 | B0: 5 web + B1: 8 arxiv (6 axes) | S2 429→arXiv fallback 噪声大, 人工过滤 ~10 强相关 | 7 | 7 | axis 覆盖 (方法/应用/数据/评估/失败模式/对抗framing) |
| B7-R1 | 28 calls (references+cited+author+title × 7) | ~15 (Boers/Ashwin/Bathiany EWS-可靠性簇 + causal-regime-detection 线索) | — | — | — |
| R2 | B7 派生候选 + 8 补充查询 (覆盖弱轴: 应用/数据集/校准) | ~12 | 8 | 8 | AMOC CSD 不确定性谱系 + SDE 可辨识性谱系起点 |
| B7-R2 | 32 calls | ~20 (Boers 组 AMOC 批判链条 + GCLM/Lyapunov 可辨识性簇) | — | — | — |
| R3 | B7 派生候选 (2604.24345/2604.20341 各独立命中 4 次) | ~15 | 8 | 8 | 红噪声 CSD、观测量选择、SDE 可辨识性延伸 |
| B7-R3 | 32 calls | ~10 (持续收敛到已读簇 + GCLM 后续) | — | — | — |
| R4 | B7 派生候选 (recurrence-based 筛选) | ~10 | 8 | 8 | intermingledness/basin stability、SDE 识别理论收尾 |
| B7-R4 | 32 calls | 2 (TRACE, saddle-node overshoot) + 1 (MOSAIC, 3 次独立命中) | — | — | — |
| R5 | B7 派生候选收尾 | 3 | 3 | 3 | — |
| B7-R5 | 12 calls | 0 新的、非已读、非切题候选 | — | — | — |
| **合计** | | | **34** | **34** | **B4 饱和** |

**B7 总调用数 = 28+32+32+32+8+4 = 136 = 34×4，discipline 达标（无降级）。**

**B4 饱和判据**: (1) 第 4 轮 2310.05587 的反引文核实显示"CSD 鲁棒性/红噪声"子方向 7 篇高相关反引文全部已入库, 该 sub-lineage 显式闭合; (2) 第 5 轮唯一剩余候选 MOSAIC (2605.05524) 的 B7 反向扩展未产生任何新的、非已读、非纯噪声的候选; (3) B7 持续把候选导向两个已充分探索的簇——(a) Boers/Morr/Bathiany/Ashwin/Lohmann 的 EWS-可靠性文献簇 (高度互引, 反引文网络已闭合), (b) SDE/GCLM 因果可辨识性理论簇 (多篇读者独立指出"更适合姊妹课题 Causal Discovery Uncertainty, 非本 topic"); (4) 6 axes 全部有覆盖 (含第 2 轮补充的应用/数据集/校准弱轴)。→ 循环终止。

## 已读论文清单 (34, 每篇有全文 wiki)

| arxiv_id | 简称 | 年 | R-Score | 撞车 | 关键发现 |
|---|---|---|---|---|---|
| 2605.28260 | VAR multiscale EWS | 2026 | — | 无 | non-leading eigenvalue 携带正确失稳趋势于奇异 Hopf |
| 2603.26537 | 周期强迫相位 EWS | 2026 | A/18 | 无 | Floquet 乘子慢强迫下失效; 相位漂移指标替代 |
| 2605.08111 | TTCD | 2026 | B/15 | 无(镜像对立) | 主流"消除非平稳性"立场, 训练方差可能被误认结构脆弱性 |
| 2511.03168 | UnCLe | 2025 | — | 无 | TVSEM/ND8 = 缺失的 positive control regime |
| 2512.12381 | Entropy Collapse | 2025 | — | 无 | 一阶相变熵坍缩, CSD 结构性无预警 (可信度瑕疵需谨慎引) |
| 2602.10060 | 网络渗流 susceptibility EWS | 2026 | A/19 | 无 | 观察者节点度量, 缓变准静态范式 |
| 2603.11090 | CausalTimePrior | 2026 | A/19 | 无 | regime-switching TSCM 生成器 = 第二 positive control |
| 2511.04361 | Causal Regime Detection Energy Markets | 2025 | C/7 | 无(空壳) | **降级**: 无实验/内部矛盾/空 workshop 摘要, 非真实竞品 |
| 2303.06448 | AMOC CSD 不确定性传播 | 2023 | — | 无 | 数据感知 null model 必要性活案例 |
| 2602.02830 | SC3D | 2026 | A/19 | 无 | 独立确认 TVSEM/ND8 可用; 明列"窗口不稳定性即信号"为 future work |
| 2309.08521 | tipping time 外推不确定性 | 2023 | — | 无 | 崩溃时间外推可从2050跨无穷, "是否失稳"探测相对稳健 |
| 2503.22111 | Reconciled AMOC warning | 2025 | — | 无 | 观测(1990s)-模式(2040s) CSD 50年错位 |
| 2506.11735 | Fokker-Planck 观测量选择 | 2025 | A/18 | 无 | 准平稳假设对快速强迫失效, 独立佐证路线选择 |
| 2308.16773 | Kramers-Moyal 多维 CSD | 2024 | — | 无 | 漂移雅可比特征值泛化 CSD 到状态相关噪声 |
| 2402.18477 | Signature Kernel CI | 2024 | — | 无 | 路径空间 CI 检验; 环存在时完整图发现不可能 |
| 2604.24345 | Langevin 回归非平稳韧性 (更正: Smith/Morr/Schötz/Boers) | 2026 | — | 无 | NGRIP 反例证伪同作者 2018 正结论; novelty check 需剔除 |
| 2604.20341 | AMOC 崩溃时间预测批判 | 2026 | — | 无 | bootstrap CI 实际覆盖率仅0-45%; 2代码bug致显著性反转 |
| 2505.19034 | 数据缺口扭曲 CSD 指标 | 2025 | S/22 | 无 | λ_AC1/λ_Var 一致性由首末数据点决定; pilot 外部效度警示 |
| 2311.18597 | 交叉耦合噪声欺骗 CSD | 2023 | S/23 | 无 | 6种EWS方法全部可给出方向相反趋势 (结构性定理) |
| 2603.08311 | OU 因果符号可辨识性 | 2026 | S/24 | 无 | 隐变量→第六类候选混淆源 |
| 2410.22729 | APPEX SDE 边际识别 | 2024 | A/21 | 无 | 假设 SDE 参数恒定, 与 idea 捕捉漂移本身对立 |
| 2512.17142 | AMOC 观测量选择偏差 | 2025 | A/20 | 无 | 自然观测量测不到信号, 正交变量才对齐 |
| 2509.19609 | 统一韧性指标框架 | 2025 | — | 无 | 7种韧性概念在不同转变机制下矛盾 |
| 2310.05587 | 红噪声 CSD ACS/PSD-fit | 2024 | — | 无 | **反引文核实该子方向搜索饱和** (7篇均已入库) |
| 2603.17142 | 图连续 Lyapunov 可辨识 | 2026 | — | 无 | 高阶累积量→可辨识到比例因子 |
| 2505.15987 | 干预 SDE 可辨识性 | 2025 | — | 无 | 证伪 APPEX 反引文线索 (未推广到临界转变) |
| 2606.28228 | 潜在 SDE 可辨识 (扩散偏移) | 2026 | — | 无 | Hardanger 悬索桥真实数据验证模板 |
| 2510.04985 | GCLM 结构可辨识性 | 2025 | — | 无 | 可辨识率96.9%远超贝叶斯网络8.1%; 更贴姊妹课题 |
| 2604.09661 | intermingledness 高维替代状态 | 2026 | — | 无 | 候选第六类混淆源, 解释 pilot 负结果假说 |
| 2603.08861 | committor 几何 EWS | 2026 | — | 无 | 强噪声/短窗口下仍可计算, "确定性几何路径"子类 |
| 2304.12786 | RAFM/Attractors.jl | 2023 | A/20 | 无 | Stage-1 吸引域分数 ground truth 工具 |
| 2601.21135 | TRACE | 2026 | S/22 | 无 | W(t)/α(t) 双轨消融 = 缺失基线的诊断模板 |
| 2401.07712 | 鞍结 overshoot 安全边界 | 2024 | A/21 | 无 | 解析边界公式, 直接回应 v2 review logical gap #4 |
| 2605.05524 | MOSAIC 模块发现 | 2026 | — | 无 | Stage-1-only VAE 可作非因果脆弱性基线 |

## 本次新增论文 (已写入 landscape N/O/P/Q 四组)

全部 34 篇均为此前 50 篇已读集合之外的新论文, 已逐篇写入 `topics/0710-causal-scs-landscape.md` 的 `## [deep-lit-tick --scope topic, 2026-07-11 iteration 2]` 段：
- **N 组** (18篇): EWS 可靠性/观测量选择/不确定性谱系 (Boers/Morr/Bathiany/Ashwin/Lohmann 研究群体)
- **O 组** (5篇): 因果发现/时序因果方法新增竞品与基础设施
- **P 组** (9篇): 因果效应/SDE 可辨识性理论谱系
- **Q 组** (2篇): EWS 失败模式新增证据

与 idea causal-scs-indicator 直接相关的 novelty 判定更新、v2 review 缺口回应、pilot 外部效度警示已写入 `ideas/causal-scs-indicator.v2.md` 的 `## Deep-lit 补充发现` 段 (不改动已定稿 review 分数)。

## 撞车风险评估 (逐核心 claim, 对照 v2 review 的 5 篇 closest prior work)

- **Claim「因果估计器统计脆弱性≠确定性因果强度变化」**: 依然**未被任何前作占据** — 本轮 N/P 两组共 27 篇均在"消除/校正不稳定性"或"总体可辨识性理论"两端, 无一篇"利用有限样本估计过程不稳定性本身作预警信号"。
- **Claim「因果估计器不稳定性优于同复杂度非因果模型脆弱性」(v2 review 点名缺口)**: 本轮**提供了直接可用的诊断设计模板** (TRACE 的 W(t)/α(t) 双轨消融), 但缺口本身仍未被本课题自己的实验填补。
- **Claim「需要已知真实 DAG 会变化的 positive control regime」(v2 review 点名缺口)**: 本轮**候选资产大幅扩充** — UnCLe/SC3D 双重独立确认 TVSEM/ND8, CausalTimePrior 提供第二套 regime-switching 生成器, 缺口从"无资产"变为"待接入哪一个"。
- **Claim「control 轨迹缺少运行时未穿越验证」(v2 review Logical gaps #4)**: 本轮**提供解析解法** (2401.07712 的严格 safe/unsafe overshoot 边界公式), 可直接替代当前经验参数构造。
- **Claim「跨方法迁移到快变中尺度 SCS」**: niche 依然未被占, N 组 18 篇进一步加固经典 EWS 在此 regime 的结构性局限证据链 (2311.18597/2512.17142/2603.26537 等给出确定性理论证明, 非仅经验观察)。
- **Claim「causal regime detection」表述层面的最大威胁**: 精读排除。arXiv:2511.04361 标题高度相似, 但全文只是内部矛盾的空壳 workshop 摘要 (无实验), 不构成真实竞品。

**总体**: 本轮 34 篇中 **0 篇构成新的 novelty 撞车**。核心贡献 niche 依然未被占据, 且本轮提供的诊断模板/positive control 资产/解析验证工具直接回应了 v2 quality review 点名的至少 3 个具体缺口 (同复杂度基线、positive control regime、control 轨迹运行时验证)，具备立即可执行价值。

## 可用新工具/方法

- **Positive control regime 资产** (回应 v2 review 缺口): UnCLe/SC3D 的 TVSEM/ND8 (2511.03168/2602.02830 双重确认); CausalTimePrior 的 regime-switching TSCM 生成器 (2603.11090, 开源代码 github.com/thummd/CausalTimePrior)。
- **非因果脆弱性基线诊断模板** (回应 v2 review 缺口): TRACE 的 W(t)/α(t) 双轨消融 (2601.21135); MOSAIC 的 Stage-1-only VAE (2605.05524)。
- **Control 轨迹解析验证**: 2401.07712 的鞍结 overshoot 安全/危险边界闭式公式, 可与已入库 2304.12786 (Attractors.jl/RAFM) 组合使用。
- **Null/校准协议参考**: 2303.06448 的观测不确定性集合传播; 2604.20341 的"结构证伪"实验设计 (平淡模型合成数据检验有趣模型误报率)。
- **候选第六类混淆源**: 2604.09661 intermingledness (观测量几何对齐问题); 2603.08311 隐变量结构性不可辨识。
- **Pilot 外部效度诊断**: 2505.19034 的缺失值/异常值对 CSD 指标偏置分析范式, 可移植用于检验 raw-EWS baseline 分数是否被"干净模拟"夸大。

## Errors / 降级

- `claude-ds` (配置的 deepseek reader CLI) 本机未安装 → fallback `claude` (opus-4-8)。已在任务开始前告知用户并按指示执行。
- **S2 API 持续 HTTP 429 rate-limit** (与 2026-07-10/07-11 前两次会话一致的已知环境问题): B1 搜索约 60% 触发 S2→arXiv API fallback, arXiv API 关键词匹配质量差 (返回大量无关旧论文, 需人工过滤); B7 的 references/cited/search 三类调用中约半数首次尝试即 429, 靠内部指数退避 (2s/4s/8s, 部分成功于 30s 二次退避) 或最终 fallback 到 arXiv API 完成。`--source openalex` 上游仍处于禁用状态 ("metadata 污染")。
- 部分论文 (2308.16773/2402.18477) 因 S2 references 端点持续 429, reader 改用论文自带 bibliography (name.bib/main.bbl) 直接提取引文表, 未编造 arxiv_id, 已在各自 wiki 标注 `tex_source_caveat`/`citation_lookup_degraded`。
- 一篇论文 (2308.16773) 的 arXiv e-print 仅有 PDF 无 LaTeX 源, reader 复用 arxiv_tool.py 自带的 PyMuPDF fallback 逻辑手动提取文本 (未修改 arxiv_tool.py), 已在 wiki 标注 `tex_source_caveat`。
- **B7 discipline**: 全部 136 次调用均为 dispatcher-level 独立执行 (非依赖 reader 内部的 references/cited 调用), 与本机器此前会话记录的"B7 discipline 必须 dispatcher-level"教训一致，未重犯。
- **基础设施问题 (与本次 deep-lit-tick 工作本身无关, 但记录在案)**: `/home/weikangqian` 家目录文件系统在 D 段自检时被发现已 100% 占满 (40G/40G 空间, 10485760/10485760 inode 均耗尽), 导致 TaskUpdate/TaskCreate (写入 `~/.claude/tasks/`) 间歇性失败, 且可能影响本次会话结束时的 auto-memory 写入 (`~/.claude/projects/.../memory/` 同样在该文件系统下)。本次 deep-lit-tick 的全部产物 (wiki、arxiv cache、git 仓库) 均位于 `/blue/yixin.wen/weikangqian/...`，不受影响，已核实 34/34 wiki 文件完整。**建议用户尽快清理 /home/weikangqian 家目录**，否则未来依赖该目录写入的功能 (任务追踪、会话记忆) 可能持续静默失败。

<verdict>
核心 claims 逐一评估：
- Claim 1 (统计脆弱性≠确定性因果强度变化): **依然未被覆盖** (N/P 两组 27 篇均在两端: 消除不稳定性 或 总体可辨识性理论, 无人利用有限样本估计脆弱性本身作信号)。
- Claim 2 (因果 vs 非因果同复杂度脆弱性基线): 未被覆盖, 但获得可直接复用的诊断模板 (TRACE)。
- Claim 3 (positive control regime 可得性): 此前缺口, 本轮已充分补齐候选资产 (三个独立来源)。
- Claim 4 (control 轨迹运行时验证): 此前缺口, 本轮获得解析解法 (2401.07712)。
- Claim 5 (跨方法迁移到快变中尺度 SCS): niche 未被占, 经典 EWS 局限性证据链本轮进一步加固 (从"经验观察"升级为"多篇独立解析定理")。
- Claim 6 (「causal regime detection」题面撞车威胁): 全文精读后**排除** (2511.04361 空壳摘要非真实竞品)。

总体: 34 篇中 0 篇构成新 novelty 撞车; 核心贡献 niche 未被占据。与 2026-07-10/07-11 前两轮不同, 本轮的价值不在"发现威胁", 而在"直接回应 v2 quality review 点名的至少 3 个具体缺口 (基线/positive control/运行时验证) 并提供现成可用资产与工具"。→ **不建议暂停实验**；建议下一步在 workspace 中直接接入本轮识别的 positive control regime 资产 (UnCLe/SC3D 的 TVSEM/ND8 或 CausalTimePrior 的生成器) 与非因果基线诊断模板 (TRACE 范式), 完成 v2 review 要求的官方 PCMCI+/LKIF 双-regime Stage-1 gate, 而非停留在当前更便宜的 VAR proxy 结果上。
</verdict>
