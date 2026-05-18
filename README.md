# Tutor Skills

一套 AI 驱动的个性化学习内容生成 skills，包含诊断知识边界、构建教学大纲、并行生成详细学习章节的完整流水线。

## 核心功能

三个 skill 协同工作，将"我想学 XXX"转化为结构化的、针对你当前水平的完整学习材料：

| Skill | 职责 |
|-------|------|
| `tutor` | 主编排器，连接整个流水线，按正确顺序调度子 skill |
| `tutor-outline` | 诊断你的知识边界，构建前置依赖明确、渐进式难度的教学大纲 |
| `tutor-generate` | 根据大纲并行生成各章节的详细学习内容，含一致性审查 |

### 工作流水线

```
/learning 请求
    │
    ▼
┌──────────────────────────────────────┐
│ Phase 1 — tutor-outline              │
│  诊断目标主题的前置依赖               │
│  诊断你的知识边界                     │
│  构建个性化教学大纲 → 保存为 .md 文件 │
└──────────────┬───────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│ Phase 2 — tutor-generate             │
│  解析大纲，拆分为并行章节任务         │
│  每个章节由独立 subagent 生成内容     │
│  一致性审查（术语、风格、前置引用）   │
│  有问题则修正后重新生成               │
│  生成目录索引 README.md               │
└──────────────────────────────────────┘
```

支持大纲修改：可随时对大纲提出修改意见，修改会传播到所有受影响章节。

## 项目结构

```
tutor-skills/
├── tutor/
│   └── SKILL.md          # 主编排器 skill
├── tutor-outline/
│   └── SKILL.md          # 大纲生成与修订 skill
└── tutor-generate/
    └── SKILL.md          # 内容生成与审查 skill
```

## 使用方法

### 在 opencode 中

1. **安装**：将三个目录复制到 opencode 的 skills 目录下：

   ```bash
   # 用户级安装（所有项目可用）
   cp -r tutor tutor-outline tutor-generate ~/.config/opencode/skills/

   # 或项目级安装（仅当前项目）
   mkdir -p .opencode/skills
   cp -r tutor tutor-outline tutor-generate .opencode/skills/
   ```

2. **使用**：在 opencode 中直接发起学习请求：

   ```
   /tutor 我想学 Transformer 架构
   ```

   opencode 会自动加载 `tutor` skill，然后依次调度 `tutor-outline`（诊断知识 + 构建大纲）和 `tutor-generate`（生成学习内容）。你只需跟随对话回答问题即可。

### 在 Claude Code 中

1. **安装**：将三个目录复制到 Claude Code 的 skills 目录下：

   ```bash
   # 用户级安装（所有项目可用）
   cp -r tutor tutor-outline tutor-generate ~/.claude/skills/

   # 或项目级安装（仅当前项目）
   mkdir -p .claude/skills
   cp -r tutor tutor-outline tutor-generate .claude/skills/
   ```

2. **使用**：在 Claude Code 中直接发起学习请求：

   ```
   /tutor 我想学 Transformer 架构
   ```

   Claude Code 会自动加载 skill 并执行相同的完整流水线。安装完成后无需重启，skill 目录的变更会在当前会话中即时生效。

### 模式说明

- **完整学习**：当你从零开始学一个主题时，使用 `/tutor` 走完整的"诊断 → 大纲 → 内容生成"流程。
- **大纲修订**：如果你已有一份大纲，想对其进行修改，同样使用 `/tutor`，skill 会进入修订模式——澄清反馈 → 诊断根因 → 传播修改 → 保存更新后的大纲，并可选择重新生成受影响的章节内容。
