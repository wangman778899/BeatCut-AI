# BeatCut AI

BeatCut AI 是一个面向轻度内容创作者，尤其是 vlog 博主的 AI 自动剪辑工具项目。目标是让用户通过上传多个视频素材和一段 BGM，快速生成一条具备节奏感、卡点感、可直接导出的短视频初稿。

当前仓库以产品设计、技术选型、实施计划和项目记忆文件为主，适合作为后续 AI 开发执行的起点。

## 当前目标

- 验证内部 Demo 的可行性
- 打通 `上传 -> 参数设置 -> 自动生成 -> 预览 -> 导出 -> 重生成` 主流程
- 建立一套简单但健壮的技术方案和开发顺序

## 仓库结构

```text
BeatCut-AI/
  README.md
  Agents.md
  memory-bank/
    design_document.md
    TechStack.md
    implementation-plan.md
    progress.md
    architecture.md
```

## 关键文档

- `memory-bank/design_document.md`
  产品设计文档，定义了产品目标、功能边界、页面流程、自动剪辑策略和异常处理规则。

- `memory-bank/TechStack.md`
  技术栈建议文档，定义了内部 Demo 最适合的前后端、任务队列、媒体处理和部署方案。

- `memory-bank/implementation-plan.md`
  实施计划文档，包含面向 AI 开发者的分步执行指令，每一步都带有测试要求。

- `memory-bank/progress.md`
  用于记录已完成步骤。

- `memory-bank/architecture.md`
  用于记录项目中各文件和模块的作用。

- `Agents.md`
  项目的通用规则文件。

## 开发规则

在写任何代码之前，必须先完整阅读：

- `memory-bank/design_document.md`

这是当前项目最核心的约束文档，后续实现必须以它为准。

## 推荐开发顺序

建议严格按照以下文档顺序推进：

1. 阅读 `memory-bank/design_document.md`
2. 阅读 `memory-bank/TechStack.md`
3. 阅读 `memory-bank/implementation-plan.md`
4. 按实施计划逐步开发并在 `memory-bank/progress.md` 记录进展
5. 在 `memory-bank/architecture.md` 持续维护文件职责说明

## 当前状态

- 已完成产品设计文档整理
- 已完成技术栈建议整理
- 已完成实施计划拆分
- 已建立 `memory-bank` 目录
- 已初始化 Git 并推送到 GitHub

## 下一步建议

- 初始化项目基础目录结构
- 开始执行 `implementation-plan.md` 中的步骤 1
- 同步维护 `progress.md` 和 `architecture.md`

## 仓库地址

- GitHub: [wangman778899/BeatCut-AI](https://github.com/wangman778899/BeatCut-AI)
