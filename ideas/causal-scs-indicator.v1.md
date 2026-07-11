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
