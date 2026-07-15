# Verification Checklist

Run this after the baseline, after restructure commits, and after merging back to `main`.

## Git State

```powershell
git status --short --branch
git log --oneline --decorate --graph --all -n 12
git remote -v
git tag --list
```

Expected:

- Working tree is clean unless there is a known intentional change.
- `main` tracks `origin/main`.
- Baseline tag exists.
- Restructure branch commits are present or merged.

## Remote Sync

```powershell
git fetch origin
git rev-parse HEAD
git rev-parse origin/main
```

Expected:

- On final `main`, `HEAD` and `origin/main` match.

## Ignored Generated Files

```powershell
git ls-files Library Temp Logs UserSettings Obj Build Builds
git status --ignored --short
```

Expected:

- No tracked generated Unity folders.
- Generated folders appear ignored, not staged.

## LFS

```powershell
git lfs status
git lfs ls-files
```

Expected:

- Large tracked binaries are LFS objects when policy requires it.
- No unexpected large raw source art has entered Git history.

## Unity Project Health

Open Unity and verify:

- [ ] No compile errors.
- [ ] Build settings scene list is correct.
- [ ] Build scenes open.
- [ ] No missing scripts in key scenes.
- [ ] No missing references caused by folder moves.
- [ ] ScriptableObjects required by gameplay load correctly.
- [ ] Project-specific content is under `Assets/<ProjectName>/`.
- [ ] Legacy roots such as `Assets/Common` or `Assets/Scenes` are removed if the restructure intended to remove them.

## UGit Or GUI Client

If using UGit:

- [ ] `更新` completes without network error.
- [ ] GUI branch list matches CLI branches.
- [ ] No unexpected local changes are shown.
- [ ] If UGit fails but CLI works, compare proxy and credential configuration.
