# Preflight Checklist

Run this before creating a baseline or restructuring a Unity project.

## Project Identity

- [ ] Confirm the Unity project root contains `Assets/`, `Packages/`, and `ProjectSettings/`.
- [ ] Confirm the intended project name, for example `TF_2D`.
- [ ] Confirm the target GitHub owner and private repository name.
- [ ] Confirm whether the repository already exists.

## Safety

- [ ] Check whether the directory is already a Git repository: `git status --short --branch`.
- [ ] If Git exists, inspect remote URLs: `git remote -v`.
- [ ] Check current branches and tags: `git branch -vv` and `git tag --list`.
- [ ] Do not delete existing files unless the user explicitly authorizes it.
- [ ] Do not rewrite history unless the user explicitly asks.

## Unity Ignore Rules

- [ ] Root `.gitignore` exists or will be created.
- [ ] `.gitignore` excludes Unity generated folders.
- [ ] `.gitignore` excludes IDE generated projects.
- [ ] `.gitignore` does not exclude required Unity source folders by mistake.

## Art And Binary Files

- [ ] Team has decided whether game-ready art goes into Git.
- [ ] Team has decided where raw source art and reference libraries live.
- [ ] `.gitattributes` exists or will be created for LFS patterns.
- [ ] `git lfs install` has been run if LFS is used.

## GitHub Access

- [ ] GitHub CLI login works: `gh auth status` or explicit `E:\GitHub\gh.exe auth status`.
- [ ] If GitHub access requires a proxy, identify the current proxy address.
- [ ] Prefer repo-local Git proxy config over global config.
- [ ] Verify remote access with `git ls-remote origin refs/heads/main` after remote setup.

## Unity Health

- [ ] Project opens in Unity.
- [ ] Console has no blocking compile errors.
- [ ] Build settings scenes are known.
- [ ] Existing scenes and prefabs are not already broken before restructuring.
