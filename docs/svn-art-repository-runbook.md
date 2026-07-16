# Local SVN Art Repository Runbook

本方法用于给 Unity 项目建立一个独立的本机 SVN 美术/大资源库。它来自 `TF_2D` 的真实搭建过程。

目标不是让 SVN 和 Git 融合成一个仓库，而是让两者分工清楚：

```text
GitHub / Git
管理 Unity 工程本体、代码、场景、Prefab、ProjectSettings、导入后的游戏可用资源

Local SVN
管理 PSD、PSB、Aseprite、Blend、参考图、素材包、历史交付和其他大资源
```

## 重要边界

- 不要把 SVN 工作副本放进 Unity Git 工程。
- 不要让 SVN 管理 `Assets/`、`Packages/`、`ProjectSettings/`。
- 不要在用户没有要求时创建 `trunk/branches/tags` 或业务目录。
- 美术库内部结构由用户后续根据项目实际工作流定义。
- Git 项目只记录 SVN 使用说明，不记录 SVN 仓库内部文件。

## 推荐路径

把 `<ProjectName>` 换成项目名。

```text
SVN 仓库:
E:\SVN\Repositories\<ProjectName>_Art

SVN 工作副本:
E:\SVNWork\<ProjectName>_Art

SVN 备份:
E:\SVN\Backups

Unity Git 工程:
F:\AI_Game\<ProjectName>
```

用户日常只使用工作副本：

```text
E:\SVNWork\<ProjectName>_Art
```

仓库目录是 SVN 的内部保险柜，用户不要手动修改：

```text
E:\SVN\Repositories\<ProjectName>_Art
```

## 环境检查

先检查本机是否已有工具：

```powershell
$cmds = 'svn','svnadmin','svnserve','winget'
foreach ($c in $cmds) {
  $found = Get-Command $c -ErrorAction SilentlyContinue
  if ($found) { "$c`t$($found.Source)" } else { "$c`tNOT_FOUND" }
}
```

检查已安装程序：

```powershell
Get-ChildItem `
  'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall',`
  'HKLM:\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall' `
  -ErrorAction SilentlyContinue |
  Get-ItemProperty |
  Where-Object { $_.DisplayName -match 'VisualSVN|TortoiseSVN|Subversion' } |
  Select-Object DisplayName,DisplayVersion,InstallLocation
```

## 安装工具

推荐安装：

- VisualSVN Server：以后需要本机 HTTPS 或迁移服务器时方便。
- Slik Subversion：提供 `svn.exe` 和 `svnadmin.exe` 命令行。
- TortoiseSVN：给不懂命令行的用户右键提交、更新、锁文件。

使用 `winget`：

```powershell
winget install --id VisualSVNSoftwareLtd.VisualSVNServer --silent --accept-package-agreements --accept-source-agreements --disable-interactivity
winget install --id Slik.Subversion --silent --accept-package-agreements --accept-source-agreements --disable-interactivity
winget install --id TortoiseSVN.TortoiseSVN --silent --accept-package-agreements --accept-source-agreements --disable-interactivity
```

如果安装后当前 PowerShell 找不到 `svn`，直接使用完整路径：

```powershell
C:\Program Files\SlikSvn\bin\svn.exe
C:\Program Files\SlikSvn\bin\svnadmin.exe
```

## VisualSVN 权限问题处理

VisualSVN Server 安装后，服务可能默认使用：

```text
C:\Repositories
```

如果 AI 想把服务级仓库根目录改到 `E:\SVN\Repositories`，可能遇到管理员权限错误：

```text
Cannot open registry key HKEY_LOCAL_MACHINE\SOFTWARE\VisualSVN\VisualSVN Server: Access is denied.
```

处理原则：

- 不要硬改注册表。
- 不要因为权限不足阻塞整个任务。
- 对单机本地存储，优先使用 `svnadmin create` 建立 `file:///` 本机仓库。
- 以后上云时再通过 `svnadmin dump/load` 或 VisualSVN 导入迁移。

## 创建本机 SVN 空仓库

这个流程不创建任何业务目录。

```powershell
$projectName = 'TF_2D'
$repoRoot = 'E:\SVN\Repositories'
$backupRoot = 'E:\SVN\Backups'
$workRoot = 'E:\SVNWork'
$repoPath = Join-Path $repoRoot "$projectName`_Art"
$workCopy = Join-Path $workRoot "$projectName`_Art"
$svnadmin = 'C:\Program Files\SlikSvn\bin\svnadmin.exe'
$svn = 'C:\Program Files\SlikSvn\bin\svn.exe'

New-Item -ItemType Directory -Path $repoRoot -Force | Out-Null
New-Item -ItemType Directory -Path $backupRoot -Force | Out-Null
New-Item -ItemType Directory -Path $workRoot -Force | Out-Null

if (-not (Test-Path -LiteralPath $repoPath)) {
  & $svnadmin create $repoPath
}

if (-not (Test-Path -LiteralPath $workCopy)) {
  $repoUrl = "file:///" + ($repoPath -replace '\\','/')
  & $svn checkout $repoUrl $workCopy
}

& $svn info $workCopy
```

期望结果：

```text
Checked out revision 0.
URL: file:///E:/SVN/Repositories/<ProjectName>_Art
Revision: 0
```

## 设置通用 SVN 根属性

只设置通用规则，不创建业务目录。

```powershell
$svn = 'C:\Program Files\SlikSvn\bin\svn.exe'
$workCopy = 'E:\SVNWork\TF_2D_Art'

$autoProps = "*.psd = svn:needs-lock=*`n*.psb = svn:needs-lock=*`n*.ase = svn:needs-lock=*`n*.aseprite = svn:needs-lock=*`n*.blend = svn:needs-lock=*`n*.fbx = svn:needs-lock=*`n*.wav = svn:needs-lock=*`n*.mp3 = svn:needs-lock=*`n*.ogg = svn:needs-lock=*`n*.mp4 = svn:needs-lock=*`n*.mov = svn:needs-lock=*`n*.zip = svn:needs-lock=*`n*.rar = svn:needs-lock=*`n*.7z = svn:needs-lock=*`n*.unitypackage = svn:needs-lock=*"
$globalIgnores = "*.tmp`n*.bak`n*.old`nThumbs.db`n.DS_Store`n__Preview`n__Temp"

& $svn propset svn:auto-props $autoProps $workCopy
& $svn propset svn:global-ignores $globalIgnores $workCopy
& $svn commit $workCopy -m "Initialize art repository rules"
```

这些规则的意义：

- PSD、PSB、Blend、FBX、音频、视频、压缩包等大二进制文件默认需要锁定后编辑。
- 临时文件不会轻易进入 SVN。
- 仓库仍然是空的，目录结构由用户以后自己建。

## 验证和备份

```powershell
$svn = 'C:\Program Files\SlikSvn\bin\svn.exe'
$svnadmin = 'C:\Program Files\SlikSvn\bin\svnadmin.exe'
$repoPath = 'E:\SVN\Repositories\TF_2D_Art'
$workCopy = 'E:\SVNWork\TF_2D_Art'
$backupPath = 'E:\SVN\Backups\TF_2D_Art-r1-hotcopy'

& $svnadmin verify $repoPath
& $svn status $workCopy
& $svn info $workCopy

if (Test-Path -LiteralPath $backupPath) {
  Remove-Item -LiteralPath $backupPath -Recurse -Force
}
& $svnadmin hotcopy $repoPath $backupPath
```

验证通过时应看到：

```text
Verified revision 0.
Verified revision 1.
```

工作副本 `svn status` 没有输出，表示干净。

## 更新 Unity Git 项目文档

在 Unity Git 项目里增加一份文档，例如：

```text
Docs/SVNArtWorkflow.md
```

内容应记录：

- SVN 仓库路径。
- SVN 工作副本路径。
- SVN 访问地址。
- Git 和 SVN 的边界。
- 用户只使用工作副本，不手动修改仓库目录。
- 后续目录由用户自定义。

同时在 Unity 项目根 `.gitignore` 中加入：

```gitignore
/.svn/
```

这只是防误入，不代表要把 SVN 放进 Unity 工程。

## 给不懂编程用户的解释模板

可以这样解释两个文件夹：

```text
E:\SVN\Repositories\<ProjectName>_Art
这是 SVN 的保险柜。不要手动打开修改。

E:\SVNWork\<ProjectName>_Art
这是你的工作桌面。以后美术源文件、大资源、参考图都放这里。
```

日常操作：

1. 打开 `E:\SVNWork\<ProjectName>_Art`。
2. 自己创建文件夹。
3. 放入美术源文件或大资源。
4. 右键选择 `SVN Commit...`。
5. 写一句说明，点击确定。

## 以后上传云服务器

本机仓库可以通过 dump/load 迁移：

```powershell
& 'C:\Program Files\SlikSvn\bin\svnadmin.exe' dump 'E:\SVN\Repositories\TF_2D_Art' > 'E:\SVN\Backups\TF_2D_Art.dump'
```

在云服务器创建仓库后再 load：

```powershell
svnadmin load <cloud-repo-path> < TF_2D_Art.dump
```

如果使用 VisualSVN Server 云端版，也可以用它的导入/恢复功能。

## AI 完成标准

AI 完成任务时必须报告：

- 安装了哪些 SVN 工具。
- SVN 仓库路径。
- SVN 工作副本路径。
- SVN URL。
- 当前 revision。
- 是否设置 `svn:auto-props` 和 `svn:global-ignores`。
- 是否完成 `svnadmin verify`。
- 是否做了本机热备份。
- 是否更新 Unity Git 项目文档。
- 是否推送 Git 文档变更。
