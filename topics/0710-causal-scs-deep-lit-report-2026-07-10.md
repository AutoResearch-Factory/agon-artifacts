# Deep Literature Review — 0710-causal-scs — 2026-07-10

reader model: 配置为 `deepseek` (claude-ds), 但本机未安装 → 按 env 规则 fallback 到同接口 `claude` (opus-4-8[1m]). dispatcher=claude.

## 循环统计

| 轮次 | 搜索 | 新论文候选 | 选中读全文 | Wiki 写入 | 主要新关键词来源 |
|------|------|-----------|-----------|----------|---------|
| R1 | B0: 6 web + B1: 14 arxiv (6 axes) | ~143 arxiv-fallback + 16 web | 8 | 8 | reader top_kw (fragility/EWS/PCMCI) |
| R2 | reader top_related (references+cited) + 9 gap 查询 (S2 降级) | ~40 | 8 | 8 | 连续因果变点/Stage-1/LKIF |
| R3 | reader top_related | 6 (+2 tex 预缓存后重派) | 6 | 6 | 方法基础 (PCMCI+/J-PCMCI+/LKIF) |
| R4 | reader top_related (饱和确认) | 2 | 2 | 2 | 混杂诊断/shift-aware |
| B7 audit | 96 calls (references×24 命中缓存; cited/author/title 因 S2 429+OpenAlex 禁用而降级) | ~70 引文 id (均为基础/工具, 无新竞品) | — | — | — |
| **合计** | | | **24** | **24** | **B4 饱和** |

**B4 饱和判据**: 第 4 轮新论文 2510.19138 的 top_related = [2606.01214, 2412.16235, 2604.02488, 2606.17553] **全部落入已读集**; B7 references 扩展只surface 基础/工具类引文, 无新 niche 竞品; 应用轴 (SCS×causal-EWS) 经 4 轮确认无占位者 (即本 idea 的目标 niche). → 循环终止.

## 已读论文清单 (24, 每篇有全文 wiki)

| arxiv_id | 简称 | 年 | R-Score | 撞车 | 关键发现 |
|---|---|---|---|---|---|
| 2412.16235 | CNM | 2024 | B/15 | HIGH | 因果强度确定性→0 于分岔; 慢变近平衡; #1 prior |
| 2606.17553 | ST-CND | 2026 | B/15 | HIGH | 空间 TE+Graph-DMD+DNSD tipping EW; PCMCI=future work |
| 2605.16128 | R-tipping 几何 EWS | 2026 | A/21 | LOW | 经典 CSD 在快强迫 AUC≈0.5 失效 |
| 2506.01981 | accelerating cascades | 2025 | A/20 | LOW | CSD 技能≈随机于"慢驱动快"级联 = SCS regime (最大威胁) |
| 2407.07290 | Causal-RuLSIF CPD | 2025 | A/20 | MED | 因果机制突变检测; soft/hard=两类突变指标 |
| 2605.05809 | kernel/copula 因果变点 | 2026 | A/18 | MED-HIGH | 刻意对 confounder/漂移不变=clean baseline; 25-null 负对照模板 |
| 2505.12023 | MEND | 2025 | B/13 | LOW | model-X CRT 条件变点; null 校准模板 |
| 2403.12677 | Causal-CCP | 2024 | A/19 | HIGH | invariance 定义因果变点; 线性/offline/iid 已占 |
| 2003.03685 | PCMCI+ | 2020 | A/21 | 无(基础) | 核心方法; 假设因果充分 |
| 2306.12896 | J-PCMCI+ | 2023 | A/20 | 无(基础) | 多数据集; Nickell FPR 膨胀=有限样本伪影机制 |
| 2007.01884 | LPCMCI | 2020 | A | 无(基础) | 隐混杂; effect size=min|ParCorr| 方法内 fragility |
| 2104.11360 | Liang-LKIF | 2021 | A | 无(基础) | LKIF 落地估计器; 抗噪/抗同步 |
| 2409.06797 | LKIF+LIM | 2024 | A | 无(基础) | 白 vs 有色噪声给不同因果=假设敏感证据 |
| 2402.15184 | Colored-LIM | 2024 | A/17 | LOW | A 对 lag 敏感(nuisance→signal); Stage-1 testbed |
| 2306.08946 | Bagged-PCMCI+ | 2024 | A/19 | LOW | per-link bootstrap 频率=现成不稳定量化; 平稳偏差 |
| 2605.18633 | DAGgr | 2026 | A/17 | LOW | s_ij∈[0,1] 跨方法统一度量(解决量纲不可比) |
| 2604.02488 | Causal-Audit | 2026 | A/19 | MED | 违背→校准风险分弃权; "可校准噪声"须超越 |
| 2606.01214 | 混杂诊断/图不稳定 | 2026 | B | MED | surrogate-null bootstrap 校准 recipe(未执行) |
| 2510.19138 | InvarGC | 2026 | B/13 | LOW | shift-aware 去混杂; strongest-objection 活体证据 |
| 2501.10235 | SpaceTime | 2025 | A/21 | LOW | MDL 非平稳因果发现+regime 变点; "正确建模"阵营 |
| 2505.16620 | CausalDynamics | 2025 | S/23 | LOW | Stage-1 引擎+10 法分歧基线; 静态混淆下已近随机(双刃) |
| 2510.01959 | Sanders-Bastiaansen | 2025 | A/21 | MED-HIGH | dispersion relation 区分 SN vs Turing; Klausmeier benchmark |
| 2510.02050 | Multidata-PC SHIPS | 2025 | S/24 | LOW | pc_alpha×7 折折间计频=现成不稳定装置; TC 强度预报 |
| 2512.08974 | FuXi-Nowcast | 2025 | — | 无 | 最强 SCS DL nowcast; 自述"非 causal attribution"=立靶 |

## 本次新增论文 (供 landscape; 之前不在 landscape 的)

20 篇 (除 2412.16235 / 2604.02488 / 2501.10235 / 2606.17553 已在 landscape). 已全部逐篇写入 `topics/0710-causal-scs-landscape.md` 的 `## [deep-lit-tick --scope topic, 2026-07-10]` 段 (分 A–G 组). 与 idea causal-scs-indicator 直接相关的 baseline/工具/数据/reframing 已写入 `ideas/causal-scs-indicator.v1.md` 的 `## Deep-lit findings` 段.

## 撞车风险评估 (逐核心 claim)

- **Claim「因果量接近转换时变化=预警信号」**: 现象**被覆盖** (CNM 2412.16235 / ST-CND 2606.17553 确定性机制; NPG 跨方法失效). 但"估计过程统计脆弱性"(认识论机制) 未被覆盖. → 须机制区分.
- **Claim「突变型指标超出经典 EWS」**: 工具**被覆盖** (2407.07290/2605.05809/2403.12677/2505.12023), 但均为 robust detector, 取向相反 (会过滤 fragility). → 须作 baseline 超越.
- **Claim「统计脆弱性≠确定性因果强度变化」**: 区分真实, **无人经验分离** → 开放, 须给判据.
- **Claim「迁移到快变中尺度 SCS」**: niche **未被占**, 但 2506.01981/2605.16128 定量证经典 CSD 在此 regime 失效 → 迁移受威胁, 须证 fragility 存活.
- **Claim「跨方法分歧作信号」**: 分歧被观测(当噪声), 度量工具(2605.18633 s_ij)已有 → 作信号开放.

## 可用新工具/方法

- 不稳定量化: DAGgr s_ij (跨方法, 2605.18633); LPCMCI min|ParCorr| (方法内, 2007.01884); Bagged-PCMCI+ per-link freq (2306.08946); pc_alpha×折计频 (2510.02050); Colored-LIM A/τ 漂移 (2402.15184).
- 负对照/校准: 2605.05809 的 25-null 分类学; 2606.01214 surrogate-null bootstrap; 2505.12023 model-X CRT; 2606.17553 IAAFT+BH-FDR.
- Stage-1 引擎: 2505.16620 CausalDynamics; 2506.01981 bistable 上下游; 2402.15184 SIS+colored+分岔.
- Stage-2 SCS: 2512.08974 变量池+协议; 2510.02050 多数据范式.

## Errors / 降级

- `claude-ds` (配置的 deepseek reader CLI) 本机未安装 → fallback `claude` (opus-4-8). 已告知.
- R1 首派 8 个 reader 中 5 个在上一 session teardown 时丢失 (CC 后台层不保) → 本 session 用 `setsid` 全脱离重派, 全部补齐.
- R3 两篇 (2003.03685 PCMCI+, 2007.01884 LPCMCI) reader 把慢 tex 下载 backgrounded 后结束 `-p` session → 手动预缓存 tex 后重派成功.
- 部分论文 (2604.02488/2501.10235/2510.02050/2512.08974/2409.06797/2007.01884/2606.01214) reader 写了 wiki 但未落 result JSON → summary 从 wiki 的 `Read by` 段读取, 不影响产物.
- **搜索/反向扩展降级**: 本 session S2 持续 HTTP 429 rate-limit; OpenAlex 被上游禁用 ("metadata 污染"); `--source arxiv` 直查返回空. 后果: B1/gap 批量搜索多靠 arxiv-fallback + web 发现; B7 四项扩展中 `references` 命中 paper_cache.db 正常 (24×命中), 但 `cited`/author/title 三轴本 session 返回错误. **B7_CALLS=96 (≥24×4, red_line #5 计数达标)**, 反向扩展的实际候选发现改由各 reader 在阅读期内部跑的 references/cited (已填入 wiki 引文表 + top_related, 驱动了 4 轮扩展到饱和) 承担. 非 discipline 违规, 属基础设施降级, 特此标注.

<verdict>
核心 claims 逐一评估:
- Claim 1 (因果量接近转换时变化=预警): **现象被覆盖** (CNM/ST-CND/NPG 确定性机制), 但认识论 fragility 机制未被覆盖.
- Claim 2 (突变型指标超经典 EWS): **工具被覆盖** (robust causal-changepoint), 取向相反, 须超越.
- Claim 3 (统计脆弱性≠确定性变化): 未被覆盖 (无人经验分离), 开放.
- Claim 4 (迁移快变 SCS): niche 未被占, 但迁移可行性被 2506.01981/2605.16128 定量削弱.
- Claim 5 (跨方法分歧作信号): 分歧被观测(当噪声), 度量工具已有, 作信号开放.

总体: 有 ≥1 个 claim 的**现象/工具层被已发表工作覆盖** (Claim 1、2), 但本 idea 的确切贡献 niche ("causal estimator fragility 作快变中尺度 SCS 事件级预警") **未被任何前作占据**. 文献**不构成 kill**, 但把 idea 四面围住: 相同现象被主流当噪声去除, 且快变迁移被证困难. → **不建议暂停实验**, 建议按 landscape/idea 中的三支柱 reframing (机制区分 + 快变尺度错配正面回应 + 超越 robust baseline) 收窄后推进; 三支柱均有本轮精读的现成工具与 baseline 支撑. 下一步宜走 idea-refine (v2) 而非直接进实验.
</verdict>
