<!-- 书写报告使用中文 -->
---
topic: topics/0710-causal-scs.md
landscape: topics/0710-causal-scs-landscape.md
workspace: workspace/causal-scs-indicator/
---

- One-sentence summary: 因果效应估计在假设违背 (confounder/non-stationarity)、方法差异、滑动窗口下产生的不稳定性 (方差/突变/DAG 结构变化) 本身可作为强对流事件的预警指标, 而非追求更准确的因果效应估计.
- Hypothesis: 系统接近状态转换/分岔点时, 滑动窗口因果效应估计的不稳定性会系统性增强; 离散型指标 (滚动方差/标准差/置信区间宽度, 对应临界慢化 CSD 理论) 可能与经典早期预警信号 (EWS) 部分重叠, 而突变型指标 (归一化一阶差分、变点检测 PELT/BOCPD/CUSUM、效应符号翻转、DAG 结构相邻窗口 edge 变化) 可能提供经典 EWS 无法捕捉的独立信息.
- Expected outcome: 成功: 模拟系统 (耦合 Lorenz + 隐藏驱动项 + 参数缓慢漂移) 中, 分岔点附近窗口的不稳定性指标显著高于无分岔的平稳对照段, 且该效应在真实强对流个例中复现, 跨方法 (PCMCI+/J-PCMCI+ vs LKIF) 分歧显著超出二者在合成数据上建立的方法学固有分歧基线; 失败: 分岔点前后指标无统计显著差异, 或效应无法被 surrogate data (IAAFT) 检验排除为统计伪影. 最便宜的证伪信号: 阶段一模拟实验中, 分岔点窗口与平稳对照窗口的不稳定性指标 (如滚动方差) 效应量 (Cohen's d) 未达到预设阈值.
- Contribution type: empirical-finding+diagnostic
- Risk: HIGH
- Estimated effort:
  - Compute: 阶段一 (合成 Lorenz 数据 + PCMCI+ 滑动窗口) CPU-only, 数小时量级; 阶段二 (真实 3km 环境数据, 事件前 15 天至后 5 天, 多方法比较) 预计数十到上百 CPU-hours, 无需 GPU.
  - Data: 阶段一 available (自行合成); 阶段二 needs collection (强对流个例的 3km 分辨率环境变量场 + 事件记录, 需筛选出有文献记录、环境演变清楚的个例)
  - Implementation: 阶段一 1-2 周, 阶段二 3-4 周
- Novelty quick-check: 尚未做正式检索 (本 idea 由用户提供的课题笔记直接转写, 跳过了 idea-creator 的 landscape survey/生成阶段). 已知相邻领域: 生态学/气候科学中临界慢化 (critical slowing down) 与早期预警信号 (EWS) 文献大量存在, 但多基于状态变量本身的统计矩 (方差/自相关), 而非"因果效应估计的跨方法/跨窗口不稳定性"; PCMCI+/J-PCMCI+/LKIF 在强对流环境预警上的应用现状待查. idea-reviewer 需要运行完整 novelty check.
- Strongest objection: 因果发现方法在 confounder 和 non-stationarity 假设违背下本身估计方差就很大, 观测到的"不稳定性升高"很可能只是估计噪声、有限样本效应或参数漂移的伪影, 而非系统真实状态的信号; 若无法用 surrogate data 等方法排除这一可能, 整个指标体系不成立.
- Why we should do this: 现有强对流环境分析以静态 diagnostics 为主, AI 概率预测又缺乏显式因果结构和时序演化信息; 若因果效应不稳定性能提供独立于经典 EWS 的预警信号, 将为强对流预警开辟一个新的诊断范式, 且阶段一的低成本模拟实验能在投入真实数据前快速证伪核心假设.
- Pilot:
  - Setup: 耦合 Lorenz 系统, 隐藏一个驱动项 (不纳入观测), 引入缓慢参数漂移使系统穿越已知分岔点; 用 PCMCI+ 在滑动窗口上计算因果效应, 同时设置无分岔的平稳对照段.
  - Metric: 分岔点前窗口 vs 平稳对照段窗口的不稳定性指标 (滚动方差 + 归一化一阶差分/变点检测) 效应量 Cohen's d; 若 d > 0.8 且平稳对照段无此效应, 信号为正.
  - Result: pending
  - Signal: SKIPPED (跳过 idea-creator 阶段, 未运行 pilot; 建议作为 refine 阶段或阶段一实验的第一步执行)

<review date="2026-07-10">

## Novelty

- Score: 3/10 (Claude 初评 4/10, codex second opinion 3/10, 合并取更严格的 3 — codex 补充发现的第四篇前作进一步压缩了新颖空间)
- Closest prior work:
  1. Bian, Wang, Leng, Lin, Shi (2025), "Utilizing Causal Network Markers to Identify Tipping Points ahead of Critical Transition" (arXiv:2412.16235, *Advanced Science*) — 用 Granger causality / transfer entropy 构造的因果网络 marker (CNM) 作为分岔早期预警信号, 理论证明 (线性化+特征值分解) 系统趋近分岔点时 dominant-group→non-dominant-group 方向的因果强度确定性趋于 0, marker 相应发散. 在基因/生态/Turing 三类 benchmark 及癫痫 iEEG 真实数据上验证, 优于传统 DNB (方差/自相关) 指标.
  2. NPG (2026), "Quantitative Comparison of Causal Inference Methods for Climate Tipping Points" (原预印本 EGUsphere-2025-6258) — 直接在合成分岔动力学数据 (AMOC-北极海冰) 上比较 PCMCI / LKIF / GCSS 三种因果方法, 核心发现之一是"当分岔事件落在分析窗口内时, 所有方法都可靠地失效" —— 即跨方法失效/分歧现象在气候临界点场景下已被观测到, 但被当作方法局限性 (需规避), 而非可用信号.
  3. Ruiz et al. (2026), "Causal-Audit" (arXiv:2604.02488) — 把假设违背 (含 confounding proxies) 量化为校准风险分数, 用于决定是否信任/弃权因果推断结果, 与本 idea"利用违背程度做正向信号"方向相反但机制邻近.
  4. Yu, Liang (2026), "SpatioTemporal Causal Network Diagnostics for Geographic Tipping Point Early Warning" (arXiv:2606.17553, 由 codex second opinion 发现, 已通过 arxiv-tools 核验存在) — ST-CND 框架用 transfer entropy 推断空间因果网络拓扑 + dynamic mode decomposition 估计局部恢复率, 结合"高内部波动/高内部同步/低外部耦合"三 signal 识别最脆弱子网络, 在合成分岔数据及 Indo-Pacific SST / AMOC 真实观测数据上验证 (AMOC AUROC 0.783). 进一步说明"因果网络诊断类早期预警"是一个 2025-2026 年正活跃的研究谱系, 而非空白领域.
- Key differentiator: 目前仍未被上述四篇前作直接覆盖的点, 是把"因果估计器本身对窗口/方法/假设违背的敏感性"(而非因果强度、网络拓扑或恢复率) 当作快变中尺度强对流的预事件信号. 但这一差异目前只是潜在的 empirical finding, 尚未被 idea 自身证成 (未引用/讨论任何一篇前作, 未论证机制可分性); "把已有因果预警框架应用到强对流"本身不构成方法新颖性, 只有证明该脆弱性信号提供稳健、独立于经典 diagnostics 且有实际 lead time 的增量预警信息, 才可能形成新贡献.

## Quality

评估视角: topic (`topics/0710-causal-scs.md`) 未声明 `target-venue` 或 `preferred-contribution-types`, 也无 `## Target venues` / `## Review standards` 小节, 按顶级大气科学 / Earth-system methods / AI-for-science 视角推断评估 (Claude 与 codex 一致).

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 4/10 (Claude 5, codex 3, 合并偏严) | 核心断言"估计不稳定性系统性增强"缺少解析推导支撑其机制, 且未证明强对流启动满足慢参数漂移/局部平衡/临界慢化这组标准假设 (Lorenz pilot 建立不了这一桥梁, "strongest objection"字段完全没讨论这一尺度不匹配问题). codex 进一步指出: PCMCI+ 的条件依赖统计量与 LKIF 信息流并非同一量纲, "跨方法分歧"当前没有定义良好的距离/校准; 真实实验也未定义统计单位、窗口长度、严格预事件截断及事件间独立性, 重叠窗口直接算 Cohen's d 会产生伪重复和过窄不确定区间. |
| Missing evidence signals | 2/10 (Claude 3, codex 2) | 除未讨论三篇 (现为四篇) 高度相关前作、未讨论 15 天窗口下滑动因果发现的样本量/功效外, codex 补充: 缺少匹配的"非平稳但不发生强对流"负对照、经典 EWS 与 CAPE/shear 等业务基线对比、原始变量漂移和数据质量基线、跨事件/季节/区域外部验证, 以及事件级 AUROC/校准/lead-time skill; 还需要区分 estimator fragility、真实网络变化、样本量下降、资料同化/观测变化四种可能解释, 目前均未涉及. |
| Narrative | 4/10 (Claude 5, codex 4) | "开辟新诊断范式"表述过度自信, 且把 CSD、真实因果结构变化、算法失效三种机制混成一个故事, 低估了 CNM / ST-CND / NPG / Causal-Audit 已占据的叙事空间. |
| Venue contribution | 4/10 (Claude 5, codex 3) | 现版本是已有因果方法与简单不稳定性统计量在新领域的应用, 单个合成系统加若干真实个例不足以支撑顶级 venue; 若能清晰做出机制区分并在大规模独立事件上取得令人意外的增量预警 finding, 才可能够格, 当前差异化论证不足. |
| Testability | 5/10 (Claude 7, codex 5) | Stage 1 pilot 形式上具体、便宜、可执行, 但 codex 指出"相对平稳段 Cohen's d 未达 0.8"不是有力证伪门槛: 阈值任意, 平稳对照过弱, 且人为加入参数漂移+隐藏驱动后阳性结果几乎是设计产物 (design-guaranteed positive). 更严格的最低成本门槛应要求在相同样本量/漂移幅度/噪声谱下区分 tipping 与 matched non-tipping nonstationarity, 并使用严格预事件窗口. |
| Outcome realism | 5/10 (Claude 6, codex 4) | Stage 1 阳性结果现实但信息增量低 (现象已被 NPG 部分观察到); Stage 2 同时要求真实事件复现、排除非特异非平稳性/数据质量混淆、获得独立于经典 diagnostics 的 skill、并跨不可直接比较的方法建立稳定分歧基线, 当前估计过于乐观. IAAFT 只能检验特定线性平稳 surrogate null, 不能单独排除非平稳混杂、资料质量变化或窗口泄漏. |
| Contribution type compliance | n.a. | idea types = {empirical-finding, diagnostic}; topic 未声明 `preferred-contribution-types`, 跳过, 不触发 hard cap (Claude 与 codex 一致). |
| Overall Quality | 4/10 (Claude 5, codex 4, 合并取平均后归入 4) | 核心直觉值得低成本检验, 但机制桥梁 (快变对流 vs 慢变 CSD 假设)、跨方法指标定义/量纲统一、负对照设计与真实数据混淆排除方案均未达到 top-venue proposal 的最低严谨度. |

## Contribution Drift (n >= 2 only; n=1 写 N/A)

N/A (v1, 无上一版可比较; hard cap: no)

## Alternative Framing

Claude 与 codex 的框架建议方向一致, 合并如下: 放弃笼统的"因果效应不稳定性是强对流预警指标"和未经证明的通用"分岔/CSD"叙事, 明确改写为"因果推断脆弱性 (estimator fragility) 作为强对流前 latent-process activation / regime-change diagnostic, 检验其是否能从 CNM/ST-CND/NPG 已覆盖的慢变生态/气候尺度迁移到快变中尺度对流系统". 具体做法: 预先定义经样本量校正和方法内标准化的 fragility score, 在严格预事件窗口中检验其相对于 CAPE/shear、经典 EWS、原始非平稳性和数据质量指标的增量预测价值, 并明确将对比对象设为上述四篇最近前作而非笼统的"经典 EWS". 该 framing 仍属于 empirical-finding+diagnostic, 把贡献焦点从已被前作覆盖的"因果网络也能预警"转向尚未成立的"估计失败具有事件特异信息"这一更窄但更可能立住的问题.

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 在所评估的数据源、事件类型和 lead-time 范围内, 经事件级独立验证和匹配负对照后, 校准的因果推断脆弱性指标 (区别于 CNM 的因果强度确定性变化机制) 对经典环境 diagnostics/EWS 提供增量预警信息. 不能据此声称识别了真实因果结构、隐藏过程或物理分岔机制. |
| NULL | 在预注册的方法、变量、分辨率和事件样本中, 未发现超越经典 diagnostics、原始非平稳性或数据质量指标的增量信息; 不能外推为所有因果方法或强对流类型均无此信号. |
| NEGATIVE | 若信号仅在窗口包含事件后出现, 或可被匹配非平稳对照、样本量变化、资料质量指标或 surrogate null 解释, 则应结论为该 pipeline 中的不稳定性是非特异估计伪影, 不能作为独立预警指标; 不应扩张为"因果诊断普遍无用". |

三档声明经 codex second opinion 校正后更加 bounded (明确排除"识别真实因果结构/隐藏过程/物理分岔机制"等过度声称), 采用 codex 版本作为最终表述.

## Likelihood-Impact Matrix

- Priority: Medium = Likelihood: Low x Impact: Medium
- Numeric score for ideas.xml: 5
- Rationale:
  - Likelihood: Claude 初评 Medium, codex second opinion Low, **分歧 1 level**, 合并后取 Low — codex 指出的跨方法量纲不可比、Stage 1 falsifier 设计上近乎保证阳性 (design-guaranteed positive)、以及需要同时满足严格预事件窗口/匹配负对照/跨事件泛化/estimator-failure 事件特异性排除等多重条件, 这些是比 Claude 初评时更具体的额外风险点, 足以把 Likelihood 下修到 Low: 达成 top venue 结果依赖一个反直觉且高风险的真实数据结果, 而非常规工程补强.
  - Impact: Claude 初评 Medium, codex second opinion High, **分歧 1 level**, 合并后维持 Medium (未采纳 codex 的 High) — 若最乐观条件成立, 确能为"failure-aware causal diagnostics"路线在强对流预警这一细分领域提供清楚贡献, 但鉴于 CNM/ST-CND/NPG 已经建立"因果网络类早期预警"这条研究线, 本 idea 即便成功也更接近"在已成形的研究线内做严格差异化 + 新领域迁移"而非"影响一条明确研究线"本身, 故保留 Medium 判断, 在此明确记录与 codex 的分歧.
- 查表: Low x Medium = Medium (5).

## Overall

- Priority: Medium
- Score: 5
- Comments: 核心 pilot 设计 (Stage 1 Lorenz) 务实且便宜, 但完整检索 (含 codex second opinion 补充发现的 ST-CND, arXiv:2606.17553, 已通过 arxiv-tools 核验) 显示, idea 声称的核心现象已被至少四篇 2024-2026 年前作从不同角度覆盖 (CNM: 确定性因果强度机制; NPG: 跨方法失效于分岔; Causal-Audit: 假设违背风险评分; ST-CND: 空间因果网络+恢复率诊断), 且 idea 完全未引用/讨论任何一篇, 也未处理强对流"快变"与经典 CSD"慢变"假设之间的尺度错配问题. codex 进一步指出 Stage 1 的 Cohen's d 门槛是弱证伪设计 (design-guaranteed positive), 且缺少匹配负对照、业务基线对比和事件级验证, 这些补强了 Claude 初评的判断并把 Quality 各维度评分进一步下修. Claude 与 codex 在 Likelihood-Impact Matrix 的两个轴上各有 1-level 分歧 (Likelihood: Medium vs Low; Impact: Medium vs High), 已在上节分别说明合并依据; 最终查表结果恰好与仅用 Claude 初评时的 numeric score 一致 (均为 5), ideas.xml 分数不变. 建议下一版明确定位为"causal estimator fragility 作为 latent-process-activation 诊断 + 新领域迁移", 并正面处理尺度不匹配、量纲统一、负对照设计三个问题, 才能达到与已发表前作真正差异化的顶会/顶刊贡献水准.

</review>

## Deep-lit findings (deep-lit-tick --scope topic, 2026-07-10)

<!-- 本轮 topic deep-lit-tick 精读 24 篇 (B4 饱和) 后, 对本 idea 直接相关的发现. 全部条目见 topics/0710-causal-scs-landscape.md 的 [deep-lit-tick] 段; 每篇有全文 wiki. 下面只提"改变本 idea 该怎么写/该怎么做"的部分. -->

### 1. 撞车与必引 (逐 claim)

| idea claim | 覆盖情况 | 最接近的已有工作 | 结论 |
|---|---|---|---|
| 因果量接近转换时变化=预警信号 | 现象已被占 (确定性机制) | CNM 2412.16235, ST-CND 2606.17553, NPG 2026 | 必须机制区分: 它们=真实因果强度确定性变化; 本 idea=估计过程统计脆弱性 (认识论). idea 未论证可分性. |
| 突变型指标超出经典 EWS | 工具已被占, 取向相反 | 2407.07290, 2605.05809, 2505.12023, 2403.12677 | 这些是"满足假设后干净检测"的 robust detector, 会过滤掉 fragility. 必须作 baseline 超越. |
| 统计脆弱性≠确定性因果强度变化 | 区分真实但无人经验分离 | CNM vs 本 idea | 开放, 但须给可经验区分判据 (当前缺失). |
| 迁移到快变中尺度强对流 | niche 无人占, 但迁移受威胁 | 2506.01981, 2605.16128 | 二者定量证经典 CSD 在慢驱动快/快强迫 regime AUC≈0.5 失效. 须证 fragility 在此仍有信号. |
| 跨方法分歧作信号 | 分歧已被观测(当噪声), 度量工具已有 | NPG, 2505.16620, 2605.18633 | 开放为信号; DAGgr s_ij 给跨方法统一 [0,1] 度量. |

**结论: 文献不 kill 本 idea (确切 niche = "estimator fragility 作快变 SCS 预警" 无人占), 但四面被围**——相同现象被当噪声去除 (2604.02488/2605.05809/2605.18633/2510.19138), 快变迁移被证困难 (2506.01981). 不建议暂停, 建议按下方 reframing 收窄后推进.

### 2. idea 必须新增的 baseline / 负对照 (review 已点名, 现成素材)

- 稳健因果变点 baseline (须超越): 2605.05809 (kernel/copula, 刻意对 confounder/漂移不变)、2403.12677 (Causal-CCP, invariance)、2407.07290 (Causal-RuLSIF).
- 弃权/风险基线 (须超越): 2604.02488 Causal-Audit 四维校准风险分 (AUROC>0.95 证违背是"可校准噪声"). 须证 fragility 对 SCS 预测力超出其滑窗风险分.
- shift-aware 对照 (回应 strongest objection): 2510.19138 InvarGC. 须证信号在 shift-aware 去混杂后仍存活.
- 匹配非平稳但不 tipping 负对照 = 现成模板: **2605.05809 的 25 个 null 分类学 (NCF/NMD/NIV/NPO/NNS/NCL)** 可直接采用.
- 有限样本伪影校正: 2306.12896 Nickell FPR 膨胀 (固定 T 增 M) + 2306.08946 40-60% 置信度正偏, 是"不稳定=有限样本伪影"最强反驳的具体机制, 须显式排除.

### 3. 可直接复用的不稳定性度量装置 (回应 reviewer "量纲不可比/需方法内标准化")

- 跨方法分歧度量: 2605.18633 DAGgr 边重要性分数 **s_ij∈[0,1]** 把 PCMCI+ (CI 统计量) 与 LKIF (信息流) 压到同一尺度; 相邻窗口 s_ij 变化=结构不稳定; τ/δ 分类="edge 方向翻转".
- 方法内标量 fragility: 2007.01884 LPCMCI **effect size=min|ParCorr|** (within-method, 无量纲问题).
- PCMCI+ 不稳定量化现成品: 2306.08946 Bagged-PCMCI+ **per-link bootstrap 频率** (tigramite 直出); 2510.02050 **pc_alpha 扫描×7 折折间计频 abacus** (本文阈值化求稳, 本 idea 读取波动本身).
- LKIF 腿: 2409.06797 (稳健 LKIF+LIM; 白 vs 有色噪声给不同因果=假设敏感证据) + 2104.11360 (LKIF 基础) + 2402.15184 Colored-LIM (追踪 A/τ/特征值漂移; A 对 lag ρ 敏感=现成差异化支点).

### 4. Stage-1 / Stage-2 现成素材

- Stage-1 合成引擎: 2505.16620 CausalDynamics create_scm/simulate_system (耦合混沌+隐混杂+噪声+time-lag+真值图; **需自加参数漂移/分岔穿越**) + 10 方法分歧作固有分歧基线. 更干净 testbed: 2506.01981 耦合一维 bistable 上下游模型 (有已知答案, 直击快/慢错配); 2402.15184 SIS+colored+分岔.
- 校准/null 检验 recipe: 2606.01214 surrogate-null bootstrap (order-p Markov null + moving-block → p̂_global/BH-FDR, 本文描述未执行, "跑通它"即超越点); 2505.12023 MEND model-X CRT; 2606.17553 IAAFT+BH-FDR.
- Stage-2 强对流数据/变量/协议: 2512.08974 FuXi-Nowcast 候选因果变量池 (CAPE/CIN/TPW/wind divergence/500-1000hPa bulk shear/dewpoint 梯度/θe) + CSI+neighborhood maxpool 协议 + dryline CI/squall-line 个例 (数据 license 受限, 建议改用公开 ERA5+雷达); 2510.02050 多数据范式 (多 SCS 个例当同一过程多次实现).
- 方法基础必引 (idea v1 全未引): PCMCI+ 2003.03685、J-PCMCI+ 2306.12896、LPCMCI 2007.01884、LKIF 2104.11360.

### 5. 建议 reframing (与 review Alternative Framing 一致, 现有证据已支撑)

把 idea 从"因果效应不稳定性是强对流预警指标"收窄为三支柱: **(a) 机制区分**——estimator fragility (认识论/统计脆弱性) vs CNM/ST-CND 确定性因果强度变化, 给可经验区分判据; **(b) 新领域迁移+正面回应尺度错配**——在 2506.01981 证明经典 CSD 失效的 downstream-within-upstream 快级联 regime 里同跑 CSD 与 fragility 对比 (把 Stage-1 判决实验改成此设置); **(c) 超越 robust baseline**——证 fragility/跨方法分歧提供超出 2605.05809/2403.12677/2604.02488/2510.19138 的增量事件级 lead-time skill. 三支柱都有本轮精读的现成工具与基线支撑.
