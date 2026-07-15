# New Unity Project Git Baseline Runbook

This runbook converts an existing Unity project into a clean private GitHub project with a recoverable baseline and a maintainable structure.

## 1. Confirm The Project Root

The project root must contain:

```text
Assets/
Packages/
ProjectSettings/
```

Useful checks:

```powershell
Get-ChildItem -Force
git status --short --branch
git remote -v
```

If the directory is not already a Git repository, initialize it:

```powershell
git init -b main
```

## 2. Decide The Art Asset Policy

Before the first commit, decide what goes into Git:

- Track Unity project files needed to open, compile, and run the game.
- Track imported game-ready art/audio when the project references them and the team accepts repository size cost.
- Use Git LFS for large binary runtime assets.
- Keep raw source art, large reference libraries, export history, and unused purchased packs outside the normal repository unless there is a clear team policy.

Do not postpone this decision until after the first push. Once large files enter Git history, removing them cleanly becomes more expensive.

## 3. Add Root Ignore And LFS Rules

Create `.gitignore` and `.gitattributes` in the Unity project root.

Recommended starting points:

- `templates/unity.gitignore`
- `templates/unity.gitattributes`

Install LFS if the repository will track large binary assets:

```powershell
git lfs install
git lfs track
```

Then inspect what Git would track:

```powershell
git status --short
git status --ignored --short
git lfs status
```

## 4. Commit The Original Baseline

The first meaningful commit should represent the original project before restructuring.

```powershell
git add .gitattributes .gitignore Assets Packages ProjectSettings README.md Docs
git status --short
git commit -m "chore: establish Unity project baseline"
```

If the project has no README or Docs yet, create a minimal README that identifies the Unity project and the purpose of the baseline.

Tag the baseline:

```powershell
$date = Get-Date -Format yyyy-MM-dd
git tag "baseline-$date"
```

Push `main` and tags:

```powershell
git remote add origin https://github.com/<owner>/<repo>.git
git push -u origin main
git push origin --tags
```

## 5. Create A Restructure Branch

Do structural work away from `main`:

```powershell
git switch -c chore/project-structure
```

Recommended target structure:

```text
Assets/
  <ProjectName>/
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
Packages/
ProjectSettings/
Docs/
```

Keep project-specific business content under `Assets/<ProjectName>/`. Keep Unity-managed packages and settings in their standard root folders.

## 6. Move Unity Assets Safely

Preferred method:

1. Open Unity.
2. Move folders through the Project window.
3. Let Unity update `.meta` files and serialized references.
4. Save scenes and assets.

If using Git or file-system moves, move every asset together with its `.meta` file. Never recreate `.meta` files casually.

After each batch:

```powershell
git status --short
git diff --stat
```

Then open Unity and check the Console for missing scripts, missing references, or import errors.

## 7. Split Runtime, Editor, And Tests

For medium or larger Unity projects, create assembly definitions:

- `Assets/<ProjectName>/Code/Runtime/<ProjectName>.Runtime.asmdef`
- `Assets/<ProjectName>/Code/Editor/<ProjectName>.Editor.asmdef`
- `Assets/<ProjectName>/Code/Tests/EditMode/<ProjectName>.Tests.EditMode.asmdef`

Rules:

- Runtime code must not depend on `UnityEditor`.
- Editor code may depend on Runtime and UnityEditor.
- Tests should validate project integrity, not only isolated scripts.

Good integrity tests include:

- Build settings contain only expected scenes.
- Legacy asset roots are removed.
- Required ScriptableObjects load.
- Build scenes have no missing scripts.

## 8. Commit The Restructure In Reviewable Steps

Use commits that explain intent:

```powershell
git add Assets ProjectSettings Docs README.md
git commit -m "refactor: reorganize Unity assets"

git add Assets
git commit -m "build: split runtime and editor assemblies"

git add Assets Docs README.md
git commit -m "test: add project integrity checks and documentation"
```

Push the branch:

```powershell
git push -u origin chore/project-structure
```

Merge after verification:

```powershell
git switch main
git pull --ff-only
git merge --no-ff chore/project-structure -m "Merge Unity project structure refactor"
git push origin main
```

## 9. Verify Final State

Run:

```powershell
git status --short --branch
git rev-parse HEAD
git rev-parse origin/main
git tag --list
git ls-files Library Temp Logs UserSettings
git lfs status
```

Open Unity and verify:

- No compile errors.
- No missing scripts in build scenes.
- Build settings scene order is correct.
- Core scenes open.
- Project-specific content lives under `Assets/<ProjectName>/`.

## 10. Preserve The Knowledge

For each project, keep a short record:

- Initial baseline commit hash.
- Baseline tag.
- Restructure branch name.
- Merge commit hash.
- Remote URL.
- Any proxy, authentication, LFS, or Unity import issues encountered.
