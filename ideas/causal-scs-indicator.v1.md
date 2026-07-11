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

- Score: 4/10
- Closest prior work:
  1. Bian, Wang, Leng, Lin, Shi (2025), "Utilizing Causal Network Markers to Identify Tipping Points ahead of Critical Transition" (arXiv:2412.16235, *Advanced Science*) — 用 Granger causality / transfer entropy 构造的因果网络 marker (CNM) 作为分岔早期预警信号, 理论证明 (线性化+特征值分解) 系统趋近分岔点时 dominant-group→non-dominant-group 方向的因果强度确定性趋于 0, marker 相应发散. 在基因/生态/Turing 三类 benchmark 及癫痫 iEEG 真实数据上验证, 优于传统 DNB (方差/自相关) 指标.
  2. NPG (2026), "Quantitative Comparison of Causal Inference Methods for Climate Tipping Points" (原预印本 EGUsphere-2025-6258) — 直接在合成分岔动力学数据 (AMOC-北极海冰) 上比较 PCMCI / LKIF / GCSS 三种因果方法, 核心发现之一是"当分岔事件落在分析窗口内时, 所有方法都可靠地失效" —— 即跨方法失效/分歧现象在气候临界点场景下已被观测到, 但被当作方法局限性 (需规避), 而非可用信号.
  3. Ruiz et al. (2026), "Causal-Audit" (arXiv:2604.02488) — 把假设违背 (含 confounding proxies) 量化为校准风险分数, 用于决定是否信任/弃权因果推断结果, 与本 idea"利用违背程度做正向信号"方向相反但机制邻近.
- Key differentiator: (a) 应用场景 (中尺度强对流风暴, 小时级快变) 相对于三篇前作 (基因/生态/癫痫慢变系统; AMOC/海冰十年际尺度) 确实是新的, 目前未查到严格意义上的先例; (b) 概念上, 本 idea 假设的机制是"因果估计过程本身在假设违背 (confounder/non-stationarity) 下的统计不稳定性携带信号"(认识论/统计机制), 区别于 CNM 的"真实因果强度确定性趋于 0/发散"(动力学机制) —— 这是可以立住的机制差异, 但当前 idea 稿完全没有引用/讨论这两篇论文, 也没有明确论证两种机制的可分性, 使得"新颖"目前只是潜在的, 尚未被 idea 自身证成. Stage 1 (Lorenz+参数漂移+PCMCI+ 滑动窗口) 的 pilot 设计与 NPG 论文已做的合成分岔实验高度相似, 后者已经报告了方法失效/分歧现象, 削弱了 Stage 1 阳性结果的信息增量.

## Quality

评估视角: topic (`topics/0710-causal-scs.md`) 未声明 `target-venue` 或 `preferred-contribution-types`, 也无 `## Target venues` / `## Review standards` 小节, 按大气科学 + 因果推断交叉的应用型研究标准推断 (如 AIES / npj Climate and Atmospheric Science / AI-for-science workshop 级别).

| Dimension | Score | Notes |
|-----------|-------|-------|
| Logical gaps | 5/10 | 核心断言"估计不稳定性系统性增强"目前只是类比经典 CSD 理论断言, 缺少类似 CNM 论文的解析推导支撑其机制; 更关键的是, 强对流启动是小时级快速非平衡过程, 而 CSD/EWS 理论的标准假设是慢速参数漂移接近平衡分岔 (Lorenz pilot 和癫痫、生态例子都是这一类), idea 的"strongest objection"字段只讨论了统计伪影问题, 完全没讨论这一尺度/机制不匹配问题——这是最容易被 hostile reviewer 抓住的漏洞. |
| Missing evidence signals | 3/10 | idea 自身"Novelty quick-check"字段坦承跳过了检索 (诚实, 但证据缺口是真实的); 完整检索后发现三篇高度相关论文均未被讨论; 另外完全没有讨论 3km 分辨率数据在 15 天窗口下做滑动因果发现的样本量/统计功效可行性. |
| Narrative | 5/10 | "开辟新诊断范式"的表述在已知上述前作后显得过度自信, 需要收窄为"验证/迁移已知现象到新领域, 并检验一个新机制假设". |
| Venue contribution | 5/10 | 若能清晰做出与 CNM/NPG 的机制区分并在真实个例上复现, 仍具备发表价值 (应用型强对流预警诊断期刊/workshop 水准), 但当前差异化论证不足以支撑更高定位的贡献声明. |
| Testability | 7/10 | Stage 1 pilot (Lorenz+隐藏驱动+参数漂移, Cohen's d>0.8 阈值, 平稳对照段) 设计具体、便宜、可执行, 是合格的最便宜证伪信号, 且有明确的失败判据. |
| Outcome realism | 6/10 | Stage 1 结果基本可预期为阳性 (NPG 论文已在类似合成分岔实验中观察到方法失效现象), 这既降低了 Stage 1 风险也降低了其信息增量; Stage 2 真实数据复现的难度被低估, 尤其是排除"跨方法分歧只是反映已知环境场质量下降/观测噪声, 而非物理分岔临近"这一混淆的难度, idea 未给出具体的混淆排除方案 (仅提到 surrogate data IAAFT, 但未说明如何应对真实数据中的环境场质量变化)。 |
| Contribution type compliance | n.a. | topic 未声明 `preferred-contribution-types`, 跳过. |
| Overall Quality | 5/10 | 概念健全、pilot 设计务实, 但因未做检索导致的证据缺口、尺度/机制不匹配问题和真实数据混淆排除方案缺失, 综合评分中等. |

## Contribution Drift (n >= 2 only; n=1 写 N/A)

N/A (v1, 无上一版可比较)

## Alternative Framing

更锐利的框架: 不要笼统声称"因果效应不稳定性是强对流预警指标", 而应明确定位为"检验 CNM (2412.16235) 与 NPG (2026) 已经在慢变生态/气候系统中观察到的'因果推断在分岔附近失效/发散'现象, 能否迁移到快变中尺度对流系统, 以及'估计过程假设违背导致的统计脆弱性'(本 idea 机制) 与'真实因果强度确定性变化'(CNM 机制) 这两种解释能否被经验区分" —— 把已知"失败模式"显式反转为"信号", 并将对比对象明确设为这两篇最近前作而非笼统的"经典 EWS", 这样即使 Stage 1 结果预期为阳性, 论文的贡献焦点也落在"机制区分 + 新领域迁移"上, 而不是"发现现象本身".

## Claims Discipline

| Outcome | Supportable claim |
|---------|------------------|
| POSITIVE | 因果效应估计的统计不稳定性 (源自假设违背, 区别于 CNM 的因果强度确定性变化机制) 在强对流环境中是独立于经典状态变量 EWS 的预警信号, 且能从已知的慢变系统迁移到快变中尺度对流系统 |
| NULL | 分岔点/事件前窗口与平稳对照窗口的不稳定性指标无统计显著差异, 现有分析未发现该信号在此新领域中的独立信息量 |
| NEGATIVE | 观测到的不稳定性升高在 surrogate data 检验下无法与统计伪影/估计噪声区分, 或无法与 CNM 型确定性因果强度变化机制区分, 不能支持将其用作独立预警指标 |

三档声明整体 bounded 合理, 但均依赖于 idea 能否清晰讲出与三篇前作 (尤其 CNM 与 NPG) 的机制/领域差异, 目前该论证在 idea 稿中缺失.

## Likelihood-Impact Matrix

- Priority: Medium = Likelihood: Medium x Impact: Medium
- Numeric score for ideas.xml: 5
- Rationale:
  - Likelihood (Medium): Stage 1 pilot 大概率成功 (现象已被 NPG 在类似合成分岔实验中部分观察到, 技术路径清晰、成本低), 但 Stage 2 真实强对流数据复现依赖多个条件同时成立: 排除环境场质量混淆、在真实数据上获得干净的"事件前窗口", 以及说服审稿人这与 CNM/NPG 的机制真正不同而非重新包装——这些不是纯工程风险, 而是有实质不确定性的条件依赖, 故不到 High.
  - Impact (Medium): 若真实数据成功复现且机制区分成立, 对"强对流预警"这一细分领域有清楚的发表价值 (新领域应用 + 机制澄清), 但因核心现象已被 CNM 与 NPG 分别从两个角度报告过 (因果强度确定性变化; 方法学失效于分岔), 很难达到"显著改变领域判断/开辟全新路线"的 Exceptional 量级, 更贴近"局部推进+诊断"的 Medium 描述.
- 注: codex second opinion 未能获取 (见下方 Overall Comments), 以上 Likelihood/Impact 判断仅基于 Claude 单方评估, 无法核实是否存在跨模型分歧.

## Overall

- Priority: Medium
- Score: 5
- Comments: 核心 pilot 设计 (Stage 1 Lorenz) 务实且便宜, 但完整检索后发现 idea 声称的核心现象 (因果推断/网络 marker 在分岔附近失衡) 已被 arXiv:2412.16235 (确定性因果强度机制) 和 NPG 2026 (跨方法失效于分岔) 分别报告, 且 idea 完全未引用/讨论这两篇高度相关工作, 也未处理强对流"快变"与经典 CSD"慢变"理论假设之间的尺度错配问题; 建议下一版明确将自身定位为"机制区分 (统计脆弱性 vs 确定性因果强度变化) + 新领域迁移 (中尺度对流)", 并正面讨论尺度不匹配问题, 才能达到与已发表前作真正差异化的顶会/顶刊贡献水准. **Codex second opinion 未能完成**: 首次调用进程在 web-search 阶段中途异常终止 (无 output-last-message 产出, ps 确认进程已消失, 无报错回显); 按 dispatch_manual.md 规范重新调用后第二次明确报错 "workspace is out of credits", 判定为账户额度耗尽的基础设施故障而非可重试的瞬时问题, 故本轮 review 仅基于 Claude 单方评估, 未获得跨模型交叉验证.

</review>
