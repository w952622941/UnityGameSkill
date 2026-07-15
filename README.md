# Unity Project Bootstrap Knowledge

这是一个给 AI 和团队成员复用的 Unity 新项目启动知识库。它来自 `TF_2D` 项目执行过程中的真实经验：建立 Git 基线、设置 Unity `.gitignore` 和 LFS 策略、保留原始版本、完成目录重构、处理 UGit/GitHub 网络连接问题，并把这些操作沉淀成可重复执行的流程。

## 这个仓库解决什么问题

当一个新的 Unity 游戏项目要进入长期开发时，先做三件事：

1. 建立干净的 Git 基线，确保任何重构都可回退。
2. 明确哪些内容进 Git，哪些留在本地，尤其是大型美术源文件。
3. 把 Unity 项目目录从“能运行”整理成“团队可以长期维护”。

## AI 应该先读什么

- `AGENTS.md`：给 AI 的总规则，适合作为新项目任务前置上下文。
- `docs/new-unity-project-runbook.md`：从零开始执行 Git 基线和目录重构的步骤。
- `docs/tf2d-case-study.md`：TF_2D 这次操作的正确动作、问题和解决方法。
- `checklists/preflight.md`：执行前检查。
- `checklists/verification.md`：执行后验证。
- `templates/`：可复制的 `.gitignore`、`.gitattributes` 和目录规范模板。

## 快速使用

把 `AGENTS.md` 的内容交给 AI，然后让 AI 按 `docs/new-unity-project-runbook.md` 执行。每一步都必须先验证当前状态，再做变更；涉及 GitHub 远端、分支、标签、目录移动时，要留下清晰提交记录。

## 核心原则

- 先保存原始基线，再做目录重构。
- Unity 资源移动优先通过 Unity Editor 或保留 `.meta` 的方式完成。
- 不提交 `Library/`、`Temp/`、`Logs/`、`UserSettings/` 和 IDE 生成文件。
- 游戏实际使用的导入后资源可以进仓库；庞大的美术源文件、素材库和历史导出默认留在本地或独立资产库。
- 大型二进制资源如果必须进入 Git，使用 Git LFS，并提前确认仓库容量和团队下载成本。
- 遇到 GitHub 连接问题，先确认代理、认证和 Git/UGit 使用的是同一套网络配置。
