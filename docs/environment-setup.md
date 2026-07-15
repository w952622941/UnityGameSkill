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
