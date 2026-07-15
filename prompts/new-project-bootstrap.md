# Prompt: Bootstrap A New Unity Project

Use this prompt when asking an AI to prepare a new Unity project for team development.

```text
你现在是这个 Unity 游戏项目的技术总监。请先把当前项目建立 Git 版本控制并提交原始基线，然后再在单独分支中完成目录结构重构。

请按以下知识库执行：
- 先读 AGENTS.md
- 再按 docs/new-unity-project-runbook.md
- 执行前用 checklists/preflight.md
- 执行后用 checklists/verification.md

约束：
- .gitignore 和 .gitattributes 必须放在 Unity 项目根目录。
- 先提交原始基线并打 baseline-YYYY-MM-DD 标签，再做目录重构。
- 美术源文件、素材库、历史导出默认留在本地或独立资产库；只有项目运行必需的导入资源才进入仓库，需要时使用 Git LFS。
- 不提交 Library、Temp、Logs、UserSettings、IDE 生成项目文件或构建产物。
- 移动 Unity 资源时必须保留 .meta 文件和 GUID 引用。
- 遇到 GitHub 或 UGit 网络问题，先检查认证、代理和 repo-local Git 配置。

最后请给出：
- 本地路径
- GitHub 远端
- main 提交 hash
- baseline 标签
- 重构分支和合并提交
- 验证结果
- 遇到的问题及解决方式
```
