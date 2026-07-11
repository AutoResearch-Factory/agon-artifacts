# Deep Literature Review — causal-scs-indicator (idea scope, topic=0710-causal-scs) — 2026-07-11

reader model: 配置为 `deepseek` (claude-ds), 但本机未安装 → 按 env 规则 fallback 到同接口 `claude` (opus-4-8). dispatcher=claude。已读记录与 topic-level tick (2026-07-10, 24 篇) 共享同一 wiki 池 (`$ARXIV_WIKI_DIR`), 以 `topic_slug=0710-causal-scs` 去重。

## 循环统计

| 轮次 | 搜索 | 候选来源 | 选中读全文 | Wiki 写入 | 主要新关键词/信号来源 |
|------|------|---------|-----------|----------|---------|
| R1 | B0: 4 web + B1: 12 arxiv (6 axes，锚定 idea 三支柱而非泛 topic 方向) | ~180 arxiv 结果 + 4 web 候选 (均经 info 核验) | 8 | 8 | 三支柱直接命中 (快慢级联/机制区分理论/robust baseline) |
| R2 | B7 references+cited+author+title (round1 8篇 × 4 = 32 调用) | ~120 条引文/反引文/作者/标题结果 | 8 | 8 | GCM/KCI 谱系、非平稳基准竞品、novelty 风险线索浮现 |
| R3 | reader top_related_papers (round2 8篇内部信号) + info 核验 | 8 候选, 核验后 8 篇全部有效 | 7 (1 因已读被去重) | 7 | 两条 "novelty 最大风险" 显式标记 (2606.00278/2512.14000), KCI 基础论文 (1202.3775) |
| R4 | reader top_related_papers (round3 内部信号) | 3 高信号候选 (novelty 风险核查 + GKCM + LOVO) | 3 | 3 | — |
| B7 补做 | round2+round3 共 15 篇的 dispatcher 级 references+cited+author+title 补齐 (60 调用) + round4 3 篇 B7 (12 调用) | ~200 条引文/搜索结果 | 0 新增 (全部落入已读集或明确不相关领域) | 0 | B4 饱和确认 |
| **合计** | | | **26** | **26** | **B4 饱和** |

**B4 饱和判据**: (1) round4 3 篇 (2606.00278/2604.03721/2411.05625) 的 B7 反向扩展 (60 次调用) 产出的新 arxiv_id 系统性收窄为——已读论文重复出现 (强收敛信号), 或与 SCS/tipping/因果脆弱性核心机制无直接关联的通用 CI-test 方法学延伸 (Itô 过程 CI 检验、量子因果兼容性、图像修复等无关领域同名作者碰撞); (2) 两个独立 reader 分别标记的"novelty 最大风险点"均已通过后续精读排除, 无新风险浮现; (3) B3 按标准 (claim 直接交集/survey优先/2026新作/高引用/已知引用列表) 筛选，round4 后无论文能清过门槛。→ 循环终止。

**红线执行记录**: B7 反向扩展总计 104 次调用 (references+cited+author-search+title-term-search × 26 篇 wiki_written = 26×4=104)，达标 (≥104)。round2/round3 曾一度仅用 reader 内部 top_related_papers 信号推进下一轮 (未在 dispatcher 层单独执行 references/cited/author/title)，已在 B4 判定前补齐全部缺口 (60 次追加调用)，确认无遗漏。

## 已读论文清单 (26, 每篇有全文 wiki, 均为 topic-level 24 篇之外的新发现)

| arxiv_id | 简称 | 年 | 撞车 | 关键发现 |
|---|---|---|---|---|
| 2509.03996 | Ritchie et al. 耦合时间尺度 | 2025 | LOW | UaDB regime 下几何/尺度决定的因果方向系统误判 = 第5种混淆渠道 |
| 2603.14944 | RCDyM | 2026 | LOW | 最全面非因果 EWS baseline; 局限印证快慢错配未解决 |
| 2602.10817 | TIPMOC | 2026 | LOW | 方差幂律分岔检测; 第5个稳健经典 EWS baseline; 负对照模板 |
| 2511.21537 | HCCD | 2025 | LOW-MED | Impossibility Result B 形式化支撑"脆弱性≠强度变化" |
| 2504.21647 | dGCM | 2025 | LOW | 单实现非平稳非线性 CI 检验; PCMCI+ 候选插件检验器 |
| 2506.01361 | TimeGraph (idea-scope 精读) | 2025 | LOW | 非平稳/混杂噪声地板基线, 补充 topic-level 已读条目 |
| 2005.05489 | ERA5/CAM6 SLS 气候学 | 2020 | LOW | 瞬时尺度可靠性风险; 候选变量池来源 |
| 2004.11636 | 理想化 SCS 探空模型 | 2020 | LOW | bulk 诊断量不充分的具体实证数字 (3MAY99) |
| 1804.07203 | Shah&Peters GCM/NFL | 2018 | LOW | Theorem NFL: CI 检验假设违背下不稳定的第一性原理依据 |
| 1903.01672 | CD-NOD | 2019 | LOW-MED | 异质/非平稳因果发现奠基作; 认识论立场相反; novelty check 遗漏谱系 |
| 2503.17452 | CausalRivers | 2025 | LOW | 真实世界基准+洪水分布偏移事件; PCMCI 真实数据仅中等表现 |
| 2606.08498 | Zhang Y. 高维非平稳独立性检验 | 2026 | LOW | 严格显著性检验候选; 混杂虚假边旁证 |
| 2302.03246 | CDANs | 2023 | MED | PCMCI+ 非平稳扩展竞品; changing modules 检测 vs fragility |
| 2310.01753 | CausalTime | 2023 | LOW | 地理先验图模板可迁移 3km 网格; Stage 1.5 中间基准 |
| 2507.12257 | PLaCy | 2025 | MED | 第5个须超越的稳健化因果发现 baseline |
| 2402.13196 | SplitKCI | 2024 | LOW | KCI 偏差定理; 有限样本伪影反驳的机制性支撑 |
| 2310.13387 | score-matching 鲁棒性 benchmark | 2023 | LOW-MED | non-iid 是唯一让所有方法崩溃的场景 = 第5种混淆源 |
| 2512.14000 | Hardness of CI Testing in Practice | 2025 | LOW | Type-I 膨胀定理; 排除"回归伪影"竞争解释的精确工具 |
| 2501.12500 | CaDRe | 2025 | LOW | 气候隐藏动态因果框架; 无 sub-daily 实验佐证空白仍在 |
| 2307.09552 | Self-Compatibility | 2023 | LOW-MED | fragility 概念形式化语言来源; novelty 风险线索起点 |
| 2606.18834 | StruBI | 2026 | LOW-MED | AMI+祖先覆盖率区分混杂/选择偏差; 机制区分候选判据 |
| 2509.09079 | Zhang Y. ANOVA 非平稳 | 2025 | LOW | 严格显著性检验候选 (与 2606.08498 姊妹篇) |
| 1202.3775 | KCI 奠基论文 | 2012 | 无(基础) | 方法基础必引链最上游节点 |
| 2606.00278 | Jahn&Janzing Mutual Compatibility | 2026 | LOW | **确认 novelty 最大风险点已排除** (未扩展到时序) |
| 2604.03721 | GKCM | 2026 | LOW | 有限样本伪影证据链第4种机制 (联合嵌入 Type-I 膨胀) |
| 2411.05625 | LOVO | 2024 | LOW-MED | 独立确认 novelty 风险排除; 新候选 fragility 自评指标 |

## 撞车风险评估 (针对本 idea 三支柱)

| 支柱 | 本轮新发现是否引入新竞品占位 | 结论 |
|---|---|---|
| (a) 机制区分 | 否——2511.21537/1804.07203/2512.14000 均为理论工具而非直接竞品; 2606.00278/2411.05625 排除了"self-compatibility 已扩展到时序"这一 novelty 风险 | niche 仍未被占, 但理论上确认这是一个已知困难问题 (Impossibility B), proposal 需正面承认边界 |
| (b) 快慢尺度错配 | 部分加剧——2509.03996 揭示新混淆渠道 (UaDB), 须新增负对照排除; RCDyM (2603.14944) 局限印证连非因果前沿也未解决此 regime | niche 未被占, 威胁增加但方向明确 (需要更细致的负对照设计) |
| (c) 超越 robust baseline | 须超越名单从 4 篇扩至 5 篇 (新增 PLaCy 2507.12257); GKCM (2604.03721) 补第4种伪影机制 | 需超越目标更明确, 未被抢先 |

**总体**: 无新论文声称"因果推断脆弱性作为快变强对流预警指标"这一 idea 核心命题本身。四轮循环持续加固了理论基础 (机制区分的形式化困难程度) 与实验设计要求 (须新增两种混淆源负对照), 但未发现任何竞品直接占据该 niche。

## 可用新工具/方法 (汇总)

- **理论支撑**: Impossibility Result B (2511.21537) + Theorem NFL (1804.07203) + Type-I 膨胀定理 (2512.14000) — 三者构成"机制区分困难"的形式化证据链。
- **候选自评/不稳定性指标**: HCCD mCIT acceptance-interval 宽度、StruBI 祖先覆盖率、LOVO cross-validation error、KNV (CD-NOD kernel PCA 驱动力可视化)。
- **严格显著性检验**: 2606.08498 / 2509.09079 (ANOVA/独立性检验姊妹篇), 可替代当前"重叠窗口 Cohen's d"弱设计, 但需先解决窗口独立性假设不匹配。
- **负对照模板**: TIPMOC 无分岔负对照 (2602.10817)、score-sortability non-iid 负对照需求 (2310.13387)。
- **Stage-1/1.5 testbed**: 2509.03996 耦合双稳 regime-labeled testbed、CausalTime 地理先验图模板 (2310.01753)。
- **须超越 baseline (扩至5)**: 2605.05809 / 2403.12677 / 2407.07290 / 2604.02488 / **2507.12257 (新增)**。

## Errors

无。全部 26 篇成功读取并写入 wiki (无 tex_download_failed)。S2 API 全程持续 HTTP 429 限流 (与 2026-07-10 topic-level session 记录一致的已知模式)，search/references/cited 均触发过 fallback (arXiv API 或论文自带 .bib/.bbl 提取)，未影响文献真实性 (无编造条目)，仅偶尔延迟部分 cited-by 数据 (已在各 wiki 条目中如实标注)。

<verdict>
核心 claims 逐一评估 (基于 idea v1 五条 claim + 本轮新发现):
- Claim 1 (因果量接近转换时变化=预警信号): 未被本轮新论文直接覆盖 (2511.21537/2606.18834 等提供工具而非竞品占位)。
- Claim 2 (突变型指标超出经典EWS): 未被新覆盖, 但须超越 baseline 增至 5 篇 (新增 PLaCy)。
- Claim 3 (统计脆弱性≠确定性因果强度变化): 未被覆盖, 但理论难度已被形式化 (Impossibility B/NFL/Type-I膨胀定理) —— niche 仍开放, 需在 proposal 中正面承认该困难而非回避。
- Claim 4 (迁移到快变中尺度强对流): 未被覆盖, 威胁增加 (新增 UaDB 混淆渠道) 但niche 仍无人占。
- Claim 5 (跨方法分歧作信号): 未被新覆盖。

总体: 全部五条 claim 均未被本轮新发现"占领" (无竞品直接声称 idea 核心命题)。novelty 最大已知风险点 (self-compatibility 时序扩展) 已被两篇独立论文排除。**文献调研通过, 建议继续推进**, 但下一版 idea/proposal 必须: (1) 正面处理两个新增混淆源 (UaDB 几何误判、non-iid 伪影) 的负对照设计; (2) 显式引用机制区分难题的三篇理论支撑; (3) 补引 CD-NOD 奠基谱系到 novelty check。
</verdict>
