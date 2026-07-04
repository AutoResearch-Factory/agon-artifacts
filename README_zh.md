# Agon Artifacts

[English](README.md) | 中文

这是 [Agon](https://github.com/AutoResearch-Factory/Agon) 的产物仓库. Agon 负责运行科研流程; 这个仓库存放生成出来的 topics、ideas、proposals、paper notes 和 experiment workspaces.

实践中, 代码 (就是提示词) 和产物分开管理会更容易.

## 项目结构

```
.
├── servers_notes.md           # 跨项目 compute notes 模板.
├── topics/                    # Topic seeds 和 landscape reports.
│   ├── 0616-fex.md
│   └── 0616-fex-landscape.md
├── ideas/                     # 按版本保存的 ideas 和 proposals.
│   ├── ideas.xml              # Idea metadata index.
│   ├── proposals.xml          # Proposal metadata index.
│   ├── <slug>.vN.md           # 第 N 版 idea draft.
│   └── <slug>-proposal.vN.md  # 第 N 版 proposal draft.
├── wiki/                      # 跨项目 paper notes, 通常每篇论文或 arXiv id 一个 Markdown 文件.
└── workspace/                 # 实验阶段 workspace, 每个 proposal 一个目录.
    ├── workspaces.xml         # Workspace metadata index.
    └── <slug>/
        ├── topic.md           # 带入 workspace 的 topic.
        ├── idea.md            # 选中的 idea.
        ├── proposal.md        # 准备进入实验的 proposal.
        ├── STATE.md           # 当前实验状态和下一步行动.
        ├── experiment-log.md  # 按时间记录的实验日志.
        ├── NOTES.md           # 临时笔记和观察.
        └── lit-feed.md        # 与 workspace 相关的文献更新.
```

预期流程是 `topic -> idea -> proposal -> experiment`.
