---
unprocessed: 0                        # 未消费条目数, deep-lit 写入后 +N, scientist 清空后置 0
---

# Lit Feed: fex-synchro-prior

<!--
lit-feed.md: 文献 inbox (收件箱), 不是知识库。
reviewer 后的 experiment-scope deep-lit 往这里投递"针对当前急需问题、刚搜来的解决方案/工具/垫脚石"。
scientist 开工第一步消费: 逐条 promote 进 STATE (§5/§6/A1) 或 LESSONS, 用不上的写一句丢弃理由, 处理完删条目 + 置 unprocessed=0。

与同目录文档的分工:
- wiki (`$ARXIV_WIKI_DIR/`): 每篇论文全文精读笔记, 长期沉淀。wiki 池位置由 `$ARXIV_WIKI_DIR` 配置。
- workspace 内 idea.md: deep-lit 搜出的全部新文献总账 (一篇不漏, 可追溯), 长期归档, 允许膨胀。
- 本文件 (inbox): 只装命中当前急需问题的那几条, 流动, 处理完即清空, 不沉淀。

inbox 永远是空或近空。膨胀的东西在 idea.md 总账和 wiki, 不在这里。
不要删除本注释, 一直保留作为本文件的填写指引。
-->

## Inbox

<!-- deep-lit 每条按下面格式追加。scientist 消费后整条删除。

### [arxiv_id] 一句话标题
- 解决哪个急需问题: [对应 STATE.md 的卡点 / 待对比 baseline / 正在实现的 method]
- 解决方案/工具/垫脚石: [它提供了什么可直接用的东西]
- 来源: arxiv_id + wiki 路径 (`$ARXIV_WIKI_DIR/<id>.md`, 可打开看全文细节)
-->

<!-- 2026-06-22 iter5 消费记录:
Promoted:
- 2505.22617 → STATE §4.4 + A1 retention experiment design (Cov(log π, π·A) 机制)
- 2509.04259 → A1 retention experiment (KL_from_init 作为 retention 量化指标)
- 2510.18874 → STATE §4.2 理论支撑 (mode-seeking 保护失效解释)
- 2510.18927 → STATE §4.4 (top-ε 过滤机制解释 standard retention failure)
Discarded:
- 2510.10150 (STEER): 4-factor 分解 overkill, 我们用更直接的 target_freq_prob+KL 足够
- 2603.11682 (Entropy-Preserving RL): BF16 不相关(我们用fp32); REPO 作为 future work 记入 LESSONS
-->

<!-- 2026-06-25 iter12 消费记录:
Promoted:
- 2512.21319 / 2606.12050 / 2507.02227 → STATE A1/A2 true_sol-free residual validation design.
- 2601.21669 / 2605.28860 → STATE A1 retention-cd2d-trajectory and LESSONS action-level TFP note.
-->
