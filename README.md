# Unity Project Bootstrap Knowledge

这是一个给 AI 和团队成员复用的 Unity 新项目启动知识库。它沉淀了 `TF_2D` 项目的真实执行经验：Git 基线、Unity 目录重构、GitHub/UGit 网络问题处理，以及本机 SVN 美术大资源库搭建。

## 知识地图

```text
unity-project-bootstrap-knowledge/
  AGENTS.md                         # AI 总入口规则
  docs/
    index.md                        # 分类导航和执行顺序
    new-unity-project-runbook.md    # Unity Git 基线和目录重构
    svn-art-repository-runbook.md   # 本机 SVN 美术大资源库
    art-and-lfs-policy.md           # 美术资源与 Git LFS 策略
    environment-setup.md            # Windows/GitHub/SVN 环境
    tf2d-case-study.md              # TF_2D 真实案例复盘
  checklists/
    preflight.md                    # Git 基线执行前检查
    verification.md                 # Git 基线执行后验证
    svn-art-repository.md           # SVN 美术库检查清单
  templates/
    unity.gitignore                 # Unity Git 忽略模板
    unity.gitattributes             # Unity Git LFS 模板
    project-layout.md               # Unity 目录模板
  prompts/
    new-project-bootstrap.md        # Git 基线/重构提示词
    setup-local-svn-art-repository.md # SVN 美术库提示词
```

## AI 先读什么

| 场景 | 先读 |
| --- | --- |
| 新 Unity 项目要进 GitHub | `AGENTS.md` + `docs/new-unity-project-runbook.md` |
| 要给项目加本机 SVN 美术资源库 | `docs/svn-art-repository-runbook.md` |
| 不确定美术资源该进 Git 还是留本地 | `docs/art-and-lfs-policy.md` |
| 要复盘 TF_2D 当时怎么做的 | `docs/tf2d-case-study.md` |
| 要直接给 AI 一段可执行提示词 | `prompts/` |

## 核心原则

- 先提交 Unity 原始 Git 基线，再做目录重构。
- `.gitignore` 和 `.gitattributes` 放在 Unity 项目根目录。
- 不提交 `Library/`、`Temp/`、`Logs/`、`UserSettings/`、IDE 生成文件或构建产物。
- Git 管 Unity 工程本体；SVN 可以独立管理美术源文件、大资源、参考资料和历史交付。
- SVN 工作副本不要放进 Unity Git 工程。
- 美术库目录结构不要替用户过早设计，除非用户明确要求。
- 遇到 GitHub、UGit、SVN 网络或权限问题，先做局部修复和清晰记录，不要随意改全局配置。

## 快速入口

让 AI 执行新项目 Git 基线：

```text
请读取 AGENTS.md，然后按 docs/new-unity-project-runbook.md 执行。
```

让 AI 给项目建立本机 SVN 美术库：

```text
请按 docs/svn-art-repository-runbook.md 建立本机 SVN 美术大资源库。
不要预设业务目录结构，目录由我以后自己定义。
```
