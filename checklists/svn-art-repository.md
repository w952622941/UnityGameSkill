# SVN Art Repository Checklist

用于新 Unity 项目建立本机 SVN 美术大资源库。默认不创建业务目录结构。

## 执行前

- [ ] 用户明确希望 SVN 只管理美术/大资源，不管理 Unity 工程本体。
- [ ] 用户明确是否允许 AI 创建业务目录。若未明确，默认不创建。
- [ ] 已确认 Unity Git 工程路径。
- [ ] 已确认 SVN 仓库路径，例如 `E:\SVN\Repositories\<ProjectName>_Art`。
- [ ] 已确认 SVN 工作副本路径，例如 `E:\SVNWork\<ProjectName>_Art`。
- [ ] SVN 工作副本不在 Unity Git 工程内部。

## 环境

- [ ] 检查 `svn`、`svnadmin`、`svnserve`、`winget`。
- [ ] 检查 VisualSVN Server 是否安装。
- [ ] 检查 Slik Subversion 是否安装。
- [ ] 检查 TortoiseSVN 是否安装。
- [ ] 若命令不在 PATH 中，使用完整路径调用。

## 创建仓库

- [ ] 创建 `E:\SVN\Repositories`。
- [ ] 创建 `E:\SVN\Backups`。
- [ ] 创建 `E:\SVNWork`。
- [ ] 使用 `svnadmin create` 创建仓库。
- [ ] 使用 `svn checkout file:///...` 创建工作副本。
- [ ] 不创建 `trunk/branches/tags`，除非用户明确要求。
- [ ] 不创建 Source、References、Exports 等业务目录，除非用户明确要求。

## 通用规则

- [ ] 设置 `svn:auto-props`。
- [ ] 大型二进制文件默认 `svn:needs-lock`。
- [ ] 设置 `svn:global-ignores`。
- [ ] 提交初始规则 revision。

## 验证

- [ ] `svnadmin verify` 通过。
- [ ] `svn status` 工作副本干净。
- [ ] `svn info` 显示正确 URL 和 revision。
- [ ] 完成本机 `svnadmin hotcopy` 热备份。

## Git 项目记录

- [ ] Unity Git 项目 `.gitignore` 加入 `/.svn/`。
- [ ] Unity Git 项目新增或更新 `Docs/SVNArtWorkflow.md`。
- [ ] 文档说明 Git 和 SVN 的边界。
- [ ] 文档说明用户只使用工作副本，不手动修改仓库目录。
- [ ] Git 文档变更已提交并推送。

## 交付说明

- [ ] 用通俗语言告诉用户两个文件夹分别是什么。
- [ ] 告诉用户日常只打开工作副本。
- [ ] 告诉用户右键 `SVN Commit...` 保存版本。
- [ ] 告诉用户以后云服务器迁移可用 `svnadmin dump/load`。
