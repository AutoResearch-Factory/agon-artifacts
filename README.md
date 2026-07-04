# Artifacts Example for Agon

English | [中文](README_zh.md)

This repository is an example artifacts store for [Agon](https://github.com/AutoResearch-Factory/Agon). Agon runs the research workflow; this repository stores the generated topics, ideas, proposals, paper notes, and experiment workspaces.

In practice, it is easier to manage the system when artifacts are kept separate from code (i.e., prompts).

## Project Layout

```
.
├── servers_notes.md           # Cross-project compute notes template.
├── topics/                    # Topic seeds and landscape reports.
│   ├── 0616-fex.md
│   └── 0616-fex-landscape.md
├── ideas/                     # Versioned ideas and proposals.
│   ├── ideas.xml              # Idea metadata index.
│   ├── proposals.xml          # Proposal metadata index.
│   ├── <slug>.vN.md           # Idea draft version N.
│   └── <slug>-proposal.vN.md  # Proposal draft version N.
├── wiki/                      # Cross-project paper notes, usually one Markdown file per paper/arXiv id.
└── workspace/                 # Experiment-stage workspaces, one directory per proposal.
    ├── workspaces.xml         # Workspace metadata index.
    └── <slug>/
        ├── topic.md           # Topic carried into the workspace.
        ├── idea.md            # Selected idea.
        ├── proposal.md        # Experiment-ready proposal.
        ├── STATE.md           # Current experiment state and next actions.
        ├── experiment-log.md  # Chronological experiment log.
        ├── NOTES.md           # Scratch notes and observations.
        └── lit-feed.md        # Literature updates relevant to the workspace.
```

The intended workflow is `topic -> idea -> proposal -> experiment`.
