<!-- 书写报告使用中文 -->
# Deep Literature Review — scs-env-benchmark (idea-scope) — 2026-07-14

> Scope: idea (`scs-env-benchmark`), topic `0713-scs-benchmark`。本次是该 idea-scope tick 的第 2 次 resume, 定向追读 2026-07-13 review 六个硬缺口中被 topic-scope deep-lit（2026-07-14 完成的 54 篇）标记为"仍完全开放"的三个：(i) NWS warning/CAP 档案可得性、(iii) -15 天前置窗口物理论证、(vi) 事件聚类区域算法选型依据。

## 循环统计

| 轮次 | 搜索数 | 新论文 | 选中读全文 | Wiki 写入 | 新关键词 |
|------|--------|--------|-----------|----------|---------|
| resume-1（B1-B6, 上次会话） | 若干（gap iii 种子） | 2 | 2 | 2（2102.08523, 2412.03049） | soil moisture memory timescale 等 5 项 |
| resume-2 / 本次: B7 backfill | 11（references×2, cited×2, author×2, title×2, + gap-iii 显式追问×3） | 0（backfill, 非新搜索） | 0 | 0 | Rahmati 综述、Findell 分层等引用级命中 |
| resume-2 / 本次: Round A (B1+B0) | 12（arxiv-tool×8, WebSearch×3, 校验×1） | gap(i): IEM/NCEI（非论文）; gap(vi): tobac/PyFLEXTRKR | 0（此阶段仅搜索） | 0 | tobac, PyFLEXTRKR, MCSMIP 等 |
| resume-2 / 本次: B3 选定 + B5 精读 | — | — | 2（2506.13939, 2505.10850） | 2 | multiday soil moisture persistence 等 |
| resume-2 / 本次: B7 on Round A | 9+3 重跑 | 数篇引用级 DOI-only（Rahmati/Findell/Maruf/Tavakoli/Zan/Muñoz/MCSMIP） | 0 | 0 | Lakshmanan 追踪评价方法论（未命中, arXiv 无覆盖） |

本 tick 累计（2 次 resume 合计）：4 篇全文精读 wiki_written，约 9 篇 DOI-only 引用级采纳，B4 判定为**已饱和**（三个目标缺口分别达成：饱和闭合 / 非文献解决 / 候选与方法论充分）。

## 已读论文清单（本 idea-scope tick 全部 4 篇全文精读）

| arxiv_id | 标题（简） | 撞车风险 | 关键发现 |
|---|---|---|---|
| 2102.08523 | 南非 CI 高分辨率土壤湿度个例 | 无（个例分析 vs 基准数据集） | 土壤湿度初始场效力在 6-32h 提前量已衰减；15 天窗口无支撑 |
| 2412.03049 | GLACE 耦合强度方差分解（正负反馈热点） | 无（诊断理论 vs 基准构建） | 仅季节内/年际尺度分析；大平原为 SCS 走廊正反馈热点 |
| 2506.13939 | Great Plains 土壤湿度-降水耦合 HDMR 分解 | 无（方法论文非基准数据集） | 同日尺度晨→午后耦合达 40% 方差；作者明确剔除天气尺度混杂, 承认长期记忆未验证 |
| 2505.10850 | 拓扑（merge tree+OT）低层云追踪 | 无（云追踪方法非 SCS 基准） | 提供无 ground-truth 追踪器对比的代理指标方法论；确认 tobac/PyFLEXTRKR 为主流基线 |

## 本次新增论文（供上层并入 topics/0713-scs-benchmark-landscape.md）

**全文精读（有 wiki, arXiv 可读）**：
- 2506.13939 — Functional data decomposition reveals unexpectedly strong soil moisture-precipitation coupling over the Great Plains（Gao, Li, Foufoula-Georgiou, Vrugt, 2025）
- 2505.10850 — Tracking Low-Level Cloud Systems with Topology（Li, Chatterjee, Glassmeier, Senf, Wang, 2025）

**引用级 / DOI-only（经 arxiv-tool search/S2 核验元数据, 无 arXiv 镜像, 未获全文, 供上层判断是否值得机构访问获取全文）**：
- Rahmati et al. 2024, *Reviews of Geophysics*, "Soil Moisture Memory: State-of-the-Art and the Way Forward"（DOI:10.1029/2023RG000828, 被引92）
- Findell et al. 2024, *GMD*, "Accurate assessment of land–atmosphere coupling in climate models requires high-frequency data output"（DOI:10.5194/gmd-17-1869-2024）
- Maruf & Kumar 2026, *JHM*, "Soil moisture decorrelation timescales are sensitive to precipitation variability and land-atmosphere coupling"（DOI:10.1175/jhm-d-25-0062.1）
- Tavakoli & Dirmeyer 2026, *Nature Sci. Data*（DOI:10.1038/s41597-026-07519-2）
- tobac v1.5（Sokolowsky, Freeman, Jones et al. 2024, *GMD*）
- PyFLEXTRKR（Feng, Hardin, Barnes et al. 2023, *GMD*, DOI:10.5194/gmd-16-2753-2023, 被引90）
- MCSMIP（Feng, Prein, Kukulies et al. 2025, *JGR*, DOI:10.1029/2024JD042204, 被引26）— 10-tracker 互比基准
- Zan et al. 2019, *Atmos. Res.*, "Solving the storm split-merge problem"（DOI:10.1016/j.atmosres.2018.12.007, 被引20）
- Muñoz et al. 2018, *Atmos. Res.*, "Enhanced object-based tracking algorithm for convective rain storms and cells"（DOI:10.1016/j.atmosres.2017.10.027, 被引50）

**非文献数据基础设施发现（供上层记入 landscape 的"候选原始数据源"章节）**：
- Iowa Environmental Mesonet (IEM, mesonet.agron.iastate.edu) — NWS VTEC warning/watch/advisory 处理版下载接口, 覆盖 1986(Tornado/SVR)–至今, 公共领域许可
- NCEI Service Records Retention System (SRRS) — NWS 官方存档层, 国会强制最少保留 5 年

## 撞车风险评估

三个追读缺口均为"数据集设计参数论证"与"工程选型"性质, 不涉及本 idea 的核心贡献声明（环境场时空演变一等公民 + 事件中心封装）, 故本轮 0 撞车。四篇精读论文与 idea 分别是：个例分析 vs 基准构建（2102.08523）、诊断理论 vs 基准构建（2412.03049、2506.13939）、云追踪方法 vs 风暴基准（2505.10850）——均为工具/证据关系, 非竞品关系。

## 可用新工具/方法

- MODE 对象化空间验证、多尺度梯度敏感性设计（2102.08523）
- HDMR/D-MORPH 开源分解框架、"前 24 小时无降水"事件筛选设计（2506.13939）
- 无 ground-truth 追踪器对比代理指标：轨迹时长分布、沿轨迹物理量标准差、轨迹线性度损失、锚点匹配距离诊断（2505.10850）
- PyFLEXTRKR（对流云追踪, 推荐作为事件聚类区域算法的默认候选）、tobac（通用追踪社区标准）
- IEM VTEC 下载接口（shapefile/CSV/KML/Excel）可直接对接数据管线

## Errors / 已知限制

- Semantic Scholar 本轮持续 HTTP 429 限流, 多组查询 fallback 到 arXiv API, 对 "storm object tracking"、"NWS CAP archive" 等强领域词的 arXiv fallback 几乎全部返回不相关噪声（医学影像/天文/空间天气/生物追踪算法）——已在 idea 文件"元发现"一节记录为结构性模式, 非本次操作失误。
- `arxiv_tool.py info` 不支持以 DOI 作为标识符查询（S2 404 后 fallback arXiv 报 400）,故 DOI-only 论文的"核验"止于 `search` 子命令返回的 S2 元数据（标题/作者/年份/摘要/引用数）, 未能像 arXiv 论文一样跑 `info`/`tex`。这是 arxiv-tool 本身的能力边界, 已按既往 session 先例（如 Leinonen et al. 2022）处理：记为引用级证据, 不强求全文。
- Taylor/Klein/Harris 2024《Multiday Soil Moisture Persistence...Sahelian MCS》与 2308.15196（印度季风核心区）均只做了引用级确认（前者经 2506.13939 references 命中, 后者经独立 search 命中）, 未派 reader 精读全文——因二者均非美国 SCS 环境, 且 gap (iii) 已通过 4 篇全文精读 + 2 篇权威综述达到饱和判定, 继续精读边际收益低, 决定不再追加。

## Deep-lit 综合结论（呼应 skill D 段）

<verdict>
- 缺口 (iii)（-15 天窗口物理论证）：**已覆盖**。4 篇独立全文精读 + 2 篇权威综述交叉验证, 时间尺度覆盖小时/同日/季节/年际, 无一支持 15 天, 该区间在文献中是空白而非争议。证据：南非(2102.08523)、美国大平原(2412.03049, 2506.13939)、权威综述(Rahmati 2024, Findell 2024)。
- 缺口 (i)（NWS warning/CAP 档案可得性）：**已覆盖**（非文献路径）。IEM + NCEI SRRS 提供可执行的工程答案, 含许可结论与关键时间断点（2007-10-01 polygon 官方化）。
- 缺口 (vi)（事件聚类区域算法选型依据）：**部分覆盖**。候选从 2 个扩展到 6+ 个, 且拿到评测方法论（2505.10850 代理指标）与选型实证依据（MCSMIP 10-tracker 差异达 2-3 倍）, 但具体选哪个仍是 refine 阶段的工程决策, 非文献自动给出。

总体：三个目标缺口均有实质进展（2 个完全覆盖, 1 个部分覆盖且不再是文献调研问题）, 建议**本 idea-scope 追读任务结束**, 交回上层 dispatcher：(1) 将"本次新增论文"清单并入 topics/0713-scs-benchmark-landscape.md；(2) 建议下一步是 refine（把 -15 天改为可辩护的 -72h 等窗口设计、接入 IEM warning 源、在 PyFLEXTRKR/tobac/watershed/PPF 间做选型决策), 而非继续开新的 deep-lit round。
</verdict>
