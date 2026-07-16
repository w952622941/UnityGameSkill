# AI Reusable Knowledge: Unity Git Baseline And Project Restructure

Use this file as the first instruction block when an AI helps bootstrap a new Unity game project.

## Mission

Turn an existing Unity project into a maintainable team project without losing the original state:

1. Establish a clean Git baseline on `main`.
2. Tag the baseline before any structural changes.
3. Restructure the Unity project in a separate branch.
4. Preserve Unity `.meta` GUIDs and serialized references.
5. Keep heavyweight source art and asset libraries out of the normal code repository unless the team intentionally chooses Git LFS, a dedicated asset repository, or a separate SVN art repository.

## Operating Rules

- Work from the Unity project root, the directory that contains `Assets/`, `Packages/`, and `ProjectSettings/`.
- Put `.gitignore` and `.gitattributes` in the repository root, not inside `Assets/`.
- Never commit `Library/`, `Temp/`, `Logs/`, `Obj/`, `Build/`, `Builds/`, `UserSettings/`, generated IDE projects, or local caches.
- Treat existing user files as owned by the user. Do not delete or overwrite them just to make the tree look clean.
- Move Unity assets with Unity Editor when possible. If moving by Git, preserve both asset files and matching `.meta` files.
- Before refactoring, create a baseline commit and a permanent tag such as `baseline-YYYY-MM-DD`.
- Use short-lived branches for structural work, for example `chore/project-structure`.
- Verify after each major move: Git status, Unity compile, build settings scenes, missing scripts, and asset reference health.
- Do not expose tokens or secrets in logs, docs, commits, or screenshots.
- If setting up SVN for art assets, keep the SVN working copy outside the Unity Git project and do not create business folders unless the user explicitly asks.

## Standard Sequence

1. Read `docs/new-unity-project-runbook.md`.
2. Run `checklists/preflight.md`.
3. Create or repair `.gitignore` and `.gitattributes` using `templates/`.
4. Commit the original project baseline on `main`.
5. Push `main` and the baseline tag to the private GitHub repository.
6. Create a restructure branch.
7. Move assets into a clear project namespace under `Assets/<ProjectName>/`.
8. Split code into `Code/Runtime`, `Code/Editor`, and `Code/Tests` where useful.
9. Add documentation and project integrity tests.
10. Merge the restructure branch back to `main`.
11. Run `checklists/verification.md`.

## Optional Sequence: Local SVN Art Repository

Use this only when the user wants SVN for art or large assets.

1. Read `docs/svn-art-repository-runbook.md`.
2. Run `checklists/svn-art-repository.md`.
3. Install or verify VisualSVN Server, Slik Subversion, and TortoiseSVN.
4. Create a local empty SVN repository outside the Unity Git project.
5. Create a working copy outside the Unity Git project.
6. Set only general SVN rules such as binary locking and temporary-file ignores.
7. Do not create `trunk/branches/tags` or business folders unless the user asks.
8. Verify with `svnadmin verify`, `svn status`, and `svn info`.
9. Create a local hotcopy backup.
10. Document the Git/SVN boundary in the Unity Git project.

## Decision Rules For Art Assets

- Game-ready assets that Unity scenes, prefabs, materials, or ScriptableObjects actually reference can be tracked in the project repository.
- Large binary assets should use Git LFS when they must be shared through Git.
- Editable source art, raw PSD/PSB/Aseprite/Blend files, reference packs, purchased asset archives, experiments, and historical exports should default to local storage or a dedicated team asset library.
- If the user wants local SVN, put source art and large assets in the SVN working copy, not inside the Unity Git project.
- If the team chooses to track large art sources, make that a conscious policy decision and document capacity, locking, and ownership rules.

## If GitHub Or UGit Cannot Connect

Use a repo-local fix first:

```powershell
git config --local http.proxy http://127.0.0.1:33210
git ls-remote origin refs/heads/main
```

Only use the proxy address that is actually active on the machine. Do not write a global proxy unless the user explicitly wants every Git repository to use it.

For GitHub CLI login behind a local proxy:

```powershell
$env:HTTPS_PROXY='http://127.0.0.1:33210'
$env:HTTP_PROXY='http://127.0.0.1:33210'
& 'E:\GitHub\gh.exe' auth login --hostname github.com --git-protocol https --web
```

After login, verify with:

```powershell
& 'E:\GitHub\gh.exe' auth status
```

## Expected Deliverables

- A clean `main` branch.
- A baseline tag.
- A private GitHub remote.
- A reviewed restructure commit or merge commit.
- Documentation explaining structure, Git workflow, and art policy.
- Verification evidence from Git and Unity.
