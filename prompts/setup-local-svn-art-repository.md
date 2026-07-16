# Prompt: Setup Local SVN Art Repository

把这段提示交给 AI，可以让它给一个 Unity 项目建立本机 SVN 美术大资源库。

```text
你现在是这个 Unity 游戏项目的技术总监。请给当前项目建立一个独立的本机 SVN 美术/大资源库。

请先读取：
- AGENTS.md
- docs/svn-art-repository-runbook.md
- checklists/svn-art-repository.md

约束：
- Git 继续管理 Unity 工程本体。
- SVN 只管理美术源文件、大资源、参考图、素材包、历史交付。
- 不要把 SVN 工作副本放进 Unity Git 工程。
- 不要让 SVN 管理 Assets、Packages、ProjectSettings。
- 不要预设业务目录结构。不要创建 trunk/branches/tags、Source、References、Exports，除非我明确要求。
- 用户日常只使用 E:\SVNWork\<ProjectName>_Art。
- E:\SVN\Repositories\<ProjectName>_Art 是 SVN 内部仓库，不让用户手动改。

请执行：
1. 检查 svn、svnadmin、svnserve、winget、VisualSVN Server、Slik Subversion、TortoiseSVN。
2. 缺少工具时优先用 winget 安装 VisualSVN Server、Slik Subversion、TortoiseSVN。
3. 创建 SVN 仓库目录、备份目录、工作副本目录。
4. 使用 svnadmin create 创建空仓库。
5. 使用 svn checkout file:///... 创建工作副本。
6. 只设置通用 svn:auto-props 和 svn:global-ignores，不创建业务目录。
7. 提交初始 SVN 规则。
8. 用 svnadmin verify、svn status、svn info 验证。
9. 做一次 svnadmin hotcopy 本机热备份。
10. 在 Unity Git 项目中新增或更新 Docs/SVNArtWorkflow.md，并在 .gitignore 加入 /.svn/。
11. 提交并推送 Git 文档变更。

最后用通俗语言告诉我：
- 哪个文件夹是保险柜，不要动。
- 哪个文件夹是工作桌面，日常使用。
- 我怎么右键提交 SVN。
- 当前 SVN revision。
- 当前 Git 提交 hash。
```
