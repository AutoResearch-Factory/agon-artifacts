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

## Deep-lit findings (deep-lit-tick --scope idea, 2026-07-11)

<!-- 本轮 idea-scope deep-lit-tick 针对上面三支柱深挖, 4 轮循环精读 26 篇 (B4 饱和: 第 4 轮 3 篇的 top_related 除已读集外, 只剩通用 CI-test 基础设施论文, 与 SCS/tipping 核心机制无直接关联). 已读记录以 topic_slug=0710-causal-scs 复用, 与 topic-level tick 的 24 篇共享同一 wiki 池, 全部有全文 wiki: $ARXIV_WIKI_DIR/<id>.md 含 "Read by: 0710-causal-scs"。B7 反向扩展 104 次调用 (26 篇 × 4), 达标。下按对三支柱的推进程度分组, 全部为 landscape 24 篇之外的新发现。 -->

### 支柱 (a) 机制区分: estimator fragility vs 确定性因果强度变化——理论基础已显著加固

- **Rabel & Runge (2025), HCCD** (arXiv:2511.21537, R-Score 高). 两条不可能性定理: 弱 regime (变化幅度小) 下任何有限样本检验都无法同时控制两类错误 (Impossibility B)。为"统计脆弱性≠确定性因果强度变化"提供目前最硬的形式化支撑——弱信号区域内二者原理上不可靠区分, 这既是 idea 机制区分论证急需的理论锚点, 也划出了 fragility 指标注定失效的参数区间, 须在 proposal 中显式承认。mCIT acceptance-interval 宽度可作新的方法内不稳定性度量。
- **Shah & Peters (2018), GCM/no-free-lunch** (arXiv:1804.07203, R-Score S). Theorem NFL: 连续 Z 下不存在对任意备择假设都有功效的一致有效 CI 检验。这不是经验观察而是定理——CI 检验/因果效应估计在假设违背下"理应"不稳定有了第一性原理依据, 而非仅靠 CNM/NPG 等前作的经验现象。GCM 统计量把回归偏置项与真实效应项 ρ_P=E[cov(X,Y|Z)] 显式分开, 直接对应 idea 需要区分的两个来源。
- **He, Pogodin, Li, Deka, Gretton, Sutherland (2025), Hardness of CI Testing in Practice** (arXiv:2512.14000, NeurIPS 2025). 证明 Shah&Peters 不可能定理刻画的是信息论极限, 若条件均值嵌入精确已知则可构造有限样本有效检验; 实践失效根源是"回归误差", 给出 Type-I error 膨胀定理, UTKFace 真实数据证明仅换回归器质量、同一 CI 检验 p 值结论会系统翻转。是 idea"排除有限样本伪影"竞争解释的最精确数学工具, 与 1804.07203、2402.13196 构成一条文献线 (必须按此顺序引用)。
- **Jahn & Janzing (2026), Mutual Compatibility** (arXiv:2606.00278, ICML 2026)。**解除本 idea 目前已知最大 novelty 风险**: 前作 2307.09552 (Faller et al. Self-Compatibility) 的兼容性打分是否已被同作者群扩展到时间/动态场景——本文核实为否, 兼容性检验轴仍是"变量对" (固定集合单一快照), 全文无滑动窗口/非平稳性讨论。confounding postulate 打分公式可作静态合理性快检工具, 但需"跨变量对→跨时间窗口"的方法论改造才能用, 这一改造本身即是差异化空间。
- **Schkoda, Faller, Blöbaum, Janzing (2024), LOVO** (arXiv:2411.05625)。独立确认同一 novelty 风险已排除 (扰动轴是"变量子集"而非时间)。额外产出 LOVO cross-validation error 作新候选 per-window 自评指标, 补充 DAGgr s_ij / LPCMCI effect size / Bagged-PCMCI+ 频率工具箱。
- **Faller, Vankadara, Mastakouri, Locatello, Janzing (2023), Self-Compatibility** (arXiv:2307.09552)。核心直觉与本 idea 共享同一元方法论 ("假设违背/扰动导致的不一致性可作信号而非纯噪声"), 但变化轴正交 (变量子集 vs 时间窗口)。observational falsifiability 定义为 idea 的 "estimator fragility" 概念提供形式化语言。

### 支柱 (b) 快慢尺度错配: downstream-within-upstream 级联——威胁与工具同时增强

- **Ritchie, Bastiaansen, von der Heydt et al. (2025)** (arXiv:2509.03996, 2506.01981 确定性姊妹篇)。Discussion 明确指出: 已知单向耦合系统中仅凭 tipping 时序做因果方向推断 (点名 Runge 2018/PCMCI) 会被 UaDB regime 系统性误导——下游先 tip 不代表下游导致上游 tip。**这是独立于统计噪声/confounder/样本量的第五种混淆渠道**, 须补入 idea 现有四种混淆解释 (estimator fragility/真实网络变化/样本量下降/资料同化) 之外, 且是由耦合几何和时间尺度决定的结构性混淆, 无法靠更大样本量消除。同时提供带 regime 标签的 Stage-1 耦合双稳 testbed (线性/局域两种耦合), 比自建耦合 Lorenz 系统更精确定位 DwUB 参数区。
- **Li, Zhu, Zhao et al. (2026), RCDyM** (arXiv:2603.14944)。非因果动力学测度类 EWS (经典 EWS→DEV→RCDyM 谱系) 目前最全面方法, 应作阶段二最强非因果 baseline。局限 2 (速率诱导转变未验证) 印证快慢错配问题在 2026 年最新非因果文献中仍未解决——不仅因果方法, 连非因果 EWS 前沿也回避了这一 regime, 反过来说明 idea 若能在此 regime 给出信号将是稀缺贡献。
- **Masuda (2026), TIPMOC** (arXiv:2602.10817)。仅用样本方差的分岔检测方法, 补第五个稳健经典 EWS baseline (原列表只有 DEV/CNM)。两个无分岔负对照假阳性率均为 0%, 提供可复用负对照模板; 证伪"τ 大即 EWS 好"的业界惯例, 对 idea 的"趋势强度≠信号真实性"论证有旁证价值。

### 支柱 (c) 超越 robust baseline——须超越名单扩至 5 篇+新增反面证据

- **Tusoni, Masi, Coletta et al. (2025), PLaCy** (arXiv:2507.12257, ICML 2026)。功率谱幂律拟合 + 谱趋势序列 Granger 检验, 非平稳/非高斯噪声下更稳健。第五个"稳健化因果发现"须超越 baseline (续 2605.05809/2403.12677/2407.07290/2604.02488)。Appendix 八张表给出 PCMCI/PCMCI_Ω 在受控非平稳噪声下的 F1/TNR fragility 数据, 可直接用于 Stage 2 排除方法常规敏感性混淆。
- **Bergen, Sejdinovic, Didelez (2026), GKCM** (arXiv:2604.03721)。回归模型无关的核 CI 检验, 补上"有限样本伪影"证据链第四种机制——联合嵌入设计导致的 Type-I 膨胀 (区别于 SplitKCI 针对的核岭回归偏差)。
- **Montagna, Mastakouri, Eulig et al. (2023), score-matching 鲁棒性 benchmark** (arXiv:2310.13387, NeurIPS 2023)。**双刃剑发现**: score-matching 方法在多数假设违背场景下依然显著优于随机 baseline (证明"假设违背不必然导致估计不稳定, 因方法而异"), 但 **non-iid/自回归数据是唯一让所有方法 (含 score-matching) 一起崩溃的场景**——这提示 topic 真实数据 (强对流前 15 天非平稳环境场) 上观测到的不稳定性可能只是通用 non-iid 噪声而非转换点邻近信号, 是 review 现有"四种可能解释"之外的**第五种新混淆源**, 必须在实验设计中排除 (例如用非事件期的 non-iid 段做负对照)。
- **Nanavati, Vreeken, Kaltenpoth (2026), StruBI** (arXiv:2606.18834)。用 mutual information + 祖先覆盖率区分隐藏混杂 (覆盖率低) 与选择偏差 (覆盖率≈1)。为 idea"机制区分"支柱提供另一候选经验判据: 可尝试同一套规则判断因果效应估计失稳的变量集合在 PCMCI+/J-PCMCI+ 图上是否祖先闭合。迁移限制: 定理仅对单潜变量、稀疏独立漂移、无环图证明, 强对流场景可能违反 (临近分岔多变量同步漂移、中尺度反馈成环), 须作待验证前提。
- **Huang, Zhang, Zhang et al. (2019), CD-NOD** (arXiv:1903.01672)。异质/非平稳因果发现奠基性工作, 认识论立场与本 idea 相反 (机制变化当真实信号, 而非估计伪影)。KNV (kernel PCA 驱动力可视化) 可作新不稳定性度量候选; 2008 金融危机案例是概念最近的先例但缺 lead-time 评估, 可作差异化反例。**Novelty check 应补引这支更早的奠基谱系** (idea v1 novelty check 仅引用 2024 年后四篇前作, 遗漏了 CD-NOD 这条 2017-2019 年起的更早谱系)。
- **Ferdous, Hasan, Gani (2023), CDANs** (arXiv:2302.03246)。PCMCI 式 CI 检验找 lagged parents + KCI 联合检测 changing modules, 是 PCMCI+ 在非平稳场景的直接扩展竞品, 须区分"changing modules 检测"(CDANs 的目标, 是信号) 与"estimator fragility"(本 idea 的目标, 是噪声也是信号)。

### 数据集/领域证据 (Stage-2 支撑)

- **Li, Chavas, Reed et al. (2020), ERA5/CAM6 SLS 气候学** (arXiv:2005.05489)。ERA5 瞬时尺度可靠性显著低于气候统计尺度, 是滑动窗口因果发现必须处理的数据质量风险点; 环境变量标准定义可作候选因果变量池参考。响应 landscape 已指出的"改用公开 ERA5 数据"建议, 本文是该数据源可靠性的第一手证据。
- **Chavas & Dawson (2020), 理想化 SCS 探空模型** (arXiv:2004.11636)。3MAY99 龙卷个例: 仅需两种独立微小结构扰动 (低层湿度换实测值, 或自由对流层相对湿度 0.54→0.70) 即可各自独立把风暴演变结果从"1小时耗散"翻转为"长寿命超级单体"。**为 topic.md "bulk 诊断量因果不充分"的动机提供了目前最具体的实证数字对比**, 建议直接引用替换当前笼统表述。
- **Fu, Huang, Li et al. (2026), CaDRe** (arXiv:2501.12500, ICML 2026)。气候隐藏动态过程+观测间因果关系联合识别框架, 比约束型方法快 900-3000 倍。全部实验停留在月/10分钟尺度, 无 sub-daily 中尺度实验, **进一步印证快变中尺度强对流场景无人验证的空白**, 加固 idea 新领域迁移差异化支柱。不建议作方法竞品引用 (核心目标相反: 消除隐藏混杂偏差 vs 利用假设违背不稳定性), 仅作背景。
- **Stein, Shadaydeh, Blunk et al. (2025), CausalRivers** (arXiv:2503.17452, ICLR 2025)。最大真实世界时序因果发现基准, 含真实洪水分布偏移事件子集 (RiversElbeFlood)。PCMCI (非 PCMCI+) 在真实数据上仅中等表现, 可作 idea 前提假设 ("真实数据比合成数据更能暴露估计脆弱性") 的外部支撑证据。反引文中 2501.12500 (已读) 之外, 2501.12500 本身又指向另一批 CRL 论文, 但均与本 idea 核心机制 (fragility 而非表征学习) 距离较远, 未继续追。
- **Cheng, Wang, Xiao et al. (2023), CausalTime** (arXiv:2310.01753, ICLR 2024)。与 TimeGraph (已在 topic tick 读过) 互补的"合成基准"两条路线之一。地理距离先验图提取模板可直接迁移到强对流 3km 网格空间先验图构造, 作为比 Lorenz pilot 更真实的 Stage 1.5 中间难度基准。

### 统计工具 (回应 review "量纲不可比/需严格显著性检验")

- **Zhang, Y. (2025/2026), 高维非平稳独立性检验 + ANOVA** (arXiv:2606.08498, arXiv:2509.09079, 姊妹篇)。前者提供跨窗口/regime 因果效应差异的严格显著性检验候选 (双带宽去偏 + wild bootstrap), 可替代 idea 当前"重叠窗口 Cohen's d"这一被 review 点名的弱设计; 但要求组间观测独立, 而滑动窗口通常重叠依赖, 应用前需先解决独立性假设不匹配问题。因果发现部分的混杂虚假边例证 (2606.08498 附带实验) 是支持 idea 核心前提的旁证。
- **Zhang, Peters, Janzing, Schölkopf (2012), KCI 奠基论文** (arXiv:1202.3775, R-Score S, 引用 733 次)。GCM/SplitKCI/GKCM 均建立于此, 补全"方法基础必引"链条的最上游节点。

### 反向扩展饱和信号 (B4 判据)

第 4 轮 (2606.00278/2604.03721/2411.05625) 的 B7 反向扩展 (60 次调用) 产出的新 arxiv_id 已系统性收窄为: (1) 大量与已读 32+ 篇论文重复出现的条目 (强收敛信号); (2) 与 SCS/tipping/因果脆弱性核心机制无直接关联的通用 CI-test 方法学延伸 (如 Itô 过程 CI 检验、量子因果兼容性、图像修复等无关领域同名作者碰撞)。判定 B4 饱和, 终止循环。

### 建议下一版 idea/proposal 处理的新增项

1. **第五种混淆渠道 (来自 2509.03996)**: UaDB regime 下几何/时间尺度决定的因果方向系统性误判, 与统计脆弱性正交, 须在"待排除渠道"清单显式列出并设计负对照排除。
2. **第五种混淆源 (来自 2310.13387)**: non-iid/自回归本身 (非分岔临近) 即可让所有方法崩溃, 须用非事件期 non-iid 段做负对照排除。
3. **Novelty 风险正式清零**: self-compatibility 谱系 (2307.09552→2606.00278/2411.05625) 均未扩展到时序场景, novelty check 中"最大未知风险点"应更新为已排除, 可提升 novelty 分数或至少去除该项扣分依据。
4. **补全奠基文献**: CD-NOD (1903.01672, 2017-2019 谱系) 应补入 novelty check 的相关前作对比, 当前 v1 只对比了 2024 年后四篇。
5. **理论支撑升级**: Impossibility Result B (2511.21537) + Theorem NFL (1804.07203) + Type-I 膨胀定理 (2512.14000) 三者串联, 把"机制区分难题"从经验断言升级为有形式化理论依据的已知困难问题, 应在 proposal 的 Strongest Objection 章节引用而非仅停留在 v1 现有的定性表述。
6. **须超越 baseline 扩至 5 个**: 2605.05809/2403.12677/2407.07290/2604.02488/**2507.12257 (新增)**。
7. **待验证的机制区分候选判据**: HCCD mCIT acceptance-interval 宽度、StruBI 祖先覆盖率、LOVO cross-validation error——三者均需先验证在强对流数据的稀疏性/无环性假设下是否成立, 不可直接套用。
