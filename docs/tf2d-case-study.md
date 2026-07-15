# TF_2D Case Study

This is the reusable record of the TF_2D Git baseline and directory restructure work.

## Final Confirmed State

- Local project path: `F:\AI_Game\TF_2D`
- GitHub repository: `w952622941/My-private-Unity-game-project`
- Default branch: `main`
- Baseline tag: `baseline-2026-07-16`
- Baseline commit: `e0b499f chore: establish Unity project baseline`
- Restructure branch: `chore/project-structure`
- Final merge commit on `main`: `654572d Merge Unity project structure refactor`
- Final local status: `main` was synchronized with `origin/main`
- Repo-local Git proxy: `http://127.0.0.1:33210`

## Correct Operations That Worked

1. Created a Git baseline before changing structure.
2. Put `.gitignore` in the Unity project root, not inside `Assets/`.
3. Added `.gitattributes` to normalize Unity text assets and route common binary files through Git LFS.
4. Created a permanent baseline tag: `baseline-2026-07-16`.
5. Pushed `main` and the baseline tag to the private GitHub repository.
6. Created a separate branch for structure work: `chore/project-structure`.
7. Reorganized project-specific content from broad roots such as `Assets/Common` into `Assets/TF_2D`.
8. Kept Unity-generated directories local through `.gitignore`.
9. Split code into Runtime, Editor, and EditMode test assemblies.
10. Added documentation for architecture, art workflow, and Git workflow.
11. Added project integrity tests to catch missing scenes, missing scripts, legacy folders, and stage unlock data problems.
12. Merged the restructure branch back into `main` after verification.

## Target Structure Established

```text
Assets/
  TF_2D/
    Art/
    Audio/
    Code/
      Runtime/
      Editor/
      Tests/
    Data/
    Prefabs/
    Scenes/
      Dev/
  Plugins/
  TextMesh Pro/
Docs/
Packages/
ProjectSettings/
```

The important idea is not the literal project name `TF_2D`; the reusable pattern is to put business assets under `Assets/<ProjectName>/` and keep Unity system folders in their normal places.

## Git Ignore Decisions

Ignored:

- `Library/`
- `Temp/`
- `Obj/`
- `Build/`
- `Builds/`
- `Logs/`
- `UserSettings/`
- `.vs/`, `.vscode/`, `.idea/`
- Generated `.csproj`, `.sln`, `.user`, `.opendb`, and IDE files
- Build artifacts and crash dumps

Tracked:

- `Assets/`
- `Packages/`
- `ProjectSettings/`
- `Docs/`
- `.gitignore`
- `.gitattributes`
- project documentation

## Art Asset Decision

The user said: art assets should stay local. The practical policy derived from that is:

- Do not put raw art source libraries, unused packs, references, and historical exports into the code repository.
- Track only assets that the Unity project actually needs to open and run, and use Git LFS for large binary runtime assets when they must be shared through Git.
- If a team later wants all source art in Git, create an explicit policy first: LFS quota, locking, folder ownership, and expected clone size.

## Problems Encountered And Solutions

### Problem: GitHub CLI Web Login Failed

Symptom:

```text
failed to authenticate via web browser
wsarecv: A connection attempt failed because the connected party did not properly respond
```

Cause:

GitHub CLI was trying to reach GitHub without using the local proxy that the machine needed for GitHub access.

Solution:

```powershell
$env:HTTPS_PROXY='http://127.0.0.1:33210'
$env:HTTP_PROXY='http://127.0.0.1:33210'
& 'E:\GitHub\gh.exe' auth login --hostname github.com --git-protocol https --web
```

Then complete the device-code browser login and verify:

```powershell
& 'E:\GitHub\gh.exe' auth status
```

### Problem: UGit Pull Failed With Connection Reset

Symptom:

```text
fatal: unable to access 'https://github.com/w952622941/My-private-Unity-game-project.git/':
Recv failure: Connection was reset
```

Cause:

UGit/Git was not using the local proxy.

Solution:

Set proxy only for this repository:

```powershell
git config --local http.proxy http://127.0.0.1:33210
git ls-remote origin refs/heads/main
```

After this, retrying UGit `更新` succeeded and returned:

```text
Already up to date.
```

Why repo-local proxy is preferred:

- It fixes this project.
- It does not affect other Git repositories.
- It does not become a tracked file or enter commits.

### Problem: Branch Screen Looked Confusing

Observed branches:

- `main`
- `chore/project-structure`
- `origin/main`
- `origin/chore/project-structure`

Meaning:

- `main` is the default stable branch.
- `chore/project-structure` is the local restructure work branch.
- `origin/*` branches are remote tracking references from GitHub.
- The branch being one commit behind `main` after merge is normal when feature work has already been merged into `main`.

### Problem: Generated Unity Files Can Pollute Git

Risk:

Unity creates large local folders and IDE files that should not be versioned.

Solution:

Create `.gitignore` before the first broad `git add`, then verify:

```powershell
git status --ignored --short
git ls-files Library Temp Logs UserSettings
```

The second command should output nothing.

## Environment Built During This Work

- Git repository initialized and linked to a private GitHub repository.
- GitHub CLI installed at `E:\GitHub\gh.exe`.
- GitHub CLI authenticated as `w952622941` using HTTPS.
- UGit was used as the visual Git client.
- Local proxy used for GitHub access: `127.0.0.1:33210`.
- Git repo-local proxy configured in `F:\AI_Game\TF_2D\.git\config`.
- Git LFS patterns configured through `.gitattributes`.

## Reusable Lessons

- Establish baseline first; restructure second.
- Use branches to make structural work reviewable.
- A root `.gitignore` is a safety device, not a cleanup afterthought.
- Do not treat all art files the same. Runtime assets, editable source assets, references, and archives have different storage policies.
- If a GUI Git client fails, reproduce the operation with CLI and inspect proxy/auth settings.
- Prefer repo-local fixes when solving machine-specific network problems.
- Final verification must include both Git state and Unity project health.
