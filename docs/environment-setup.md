# Environment Setup Notes

These notes capture the Windows environment setup used during the TF_2D work and the reusable approach for new projects.

## Tools

Required:

- Git for Windows
- GitHub CLI
- Unity Editor matching the project version
- A Git GUI if the user prefers one, such as UGit

Optional but recommended:

- Git LFS
- Visual Studio or Rider for C# editing
- A project-specific local proxy configuration when GitHub access requires it
- VisualSVN Server, Slik Subversion, and TortoiseSVN when the project uses a local SVN art repository

## GitHub CLI Location

In the TF_2D environment, GitHub CLI was installed here:

```powershell
E:\GitHub\gh.exe
```

Use the explicit path when `gh` is not in `PATH`:

```powershell
& 'E:\GitHub\gh.exe' auth status
```

## Login Behind A Local Proxy

If GitHub CLI browser login times out, set proxy environment variables for the current PowerShell session:

```powershell
$env:HTTPS_PROXY='http://127.0.0.1:33210'
$env:HTTP_PROXY='http://127.0.0.1:33210'
& 'E:\GitHub\gh.exe' auth login --hostname github.com --git-protocol https --web
```

This only affects the current shell session.

## Git Or UGit Behind A Local Proxy

If `git pull`, `git push`, or UGit fails with connection reset, configure the proxy at repository scope:

```powershell
git config --local http.proxy http://127.0.0.1:33210
git config --local --get http.proxy
git ls-remote origin refs/heads/main
```

Do not set a global proxy unless the user explicitly wants all repositories to use it:

```powershell
git config --global http.proxy http://127.0.0.1:33210
```

The global command above is shown as a warning example. Prefer local config for project work.

## When `git push` Fails But GitHub API Still Works

A reset or port-443 timeout from `git push` does not always mean the token is invalid. Git HTTPS transport and GitHub's REST API can take different local network paths. Diagnose them separately:

```powershell
git ls-remote origin refs/heads/main
gh auth status
gh api user --jq .login
```

Use this order:

1. Confirm the repository, branch, staged scope and remote URL. Do not recommit or rewrite history just because transport failed.
2. Retry only after checking the active repo-local proxy. `git -c http.version=HTTP/1.1 push ...` is a bounded compatibility test, not a permanent cure.
3. If Git transport remains unavailable but `gh api` is authenticated, the user has explicitly authorized publication, and a new non-default branch is being created, the Git Data API can be used as an advanced fallback: upload blobs, create a tree, create a commit, then create the branch ref.
4. Compare every uploaded blob SHA with `git rev-parse HEAD:<path>`, the remote tree SHA with `git show -s --format=%T HEAD`, and the final PR head/file list with the intended scope. Never update `main` directly through this fallback.
5. GitHub may serialize commit metadata or the final message newline differently, so the commit SHA can differ even when the tree is identical. Treat verified blob/tree equality as content evidence, then fetch and align the local branch when normal Git transport is available.
6. Remove temporary request files and verify that the PR is open, mergeable, targets the expected base and contains only intended files.

This fallback solved a real publication where three HTTPS pushes failed with connection reset/timeouts while `gh api user` stayed healthy. The published branch was accepted only after all 11 blob hashes, the full tree hash, branch ref and PR file list matched. Prefer ordinary `git push`; use the API path only as a controlled recovery procedure.

## Create A Private GitHub Repository

Use:

```powershell
& 'E:\GitHub\gh.exe' repo create <repo-name> --private --description "<description>" --clone=false
```

Then connect a local repository:

```powershell
git remote add origin https://github.com/<owner>/<repo-name>.git
git push -u origin main
git push origin --tags
```

## Create This Knowledge Repository Pattern

For a reusable knowledge repository:

```powershell
New-Item -ItemType Directory -Path 'E:\GitHub\<repo-name>' -Force
Set-Location 'E:\GitHub\<repo-name>'
git init -b main
```

Then add Markdown docs, commit, create the GitHub private repository, and push.

## Local SVN Art Repository Tools

For a local SVN art repository, install:

```powershell
winget install --id VisualSVNSoftwareLtd.VisualSVNServer --silent --accept-package-agreements --accept-source-agreements --disable-interactivity
winget install --id Slik.Subversion --silent --accept-package-agreements --accept-source-agreements --disable-interactivity
winget install --id TortoiseSVN.TortoiseSVN --silent --accept-package-agreements --accept-source-agreements --disable-interactivity
```

If the current PowerShell session does not find `svn` after installation, call the tools by full path:

```powershell
& 'C:\Program Files\SlikSvn\bin\svn.exe' --version --quiet
& 'C:\Program Files\SlikSvn\bin\svnadmin.exe' help
```

Use `docs/svn-art-repository-runbook.md` for the full setup process.
