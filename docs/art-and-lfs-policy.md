# Art Assets And Git LFS Policy

Unity game projects often fail slowly when the repository becomes an unplanned asset dump. This policy keeps the code repository useful while still allowing the game to open and run.

## Recommended Rule

Put only the assets needed by the game project into the Git repository. Keep large raw source art, unused packs, references, and historical exports in local storage or a dedicated asset system unless the team intentionally adopts Git LFS for them.

## GitHub Constraints To Remember

As of the checked GitHub documentation:

- GitHub recommends keeping repositories small, ideally under 1 GB, and strongly recommends staying under 5 GB.
- Normal Git objects have a 100 MB enforced single-object limit.
- Git LFS has storage and bandwidth accounting. Pushing a new version of a large LFS file consumes storage for the full new version, not only the changed bytes.

Official docs:

- https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-large-files-on-github
- https://docs.github.com/en/repositories/creating-and-managing-repositories/repository-limits
- https://docs.github.com/billing/managing-billing-for-git-large-file-storage/about-billing-for-git-large-file-storage

Because limits and quotas can change, check the current GitHub documentation before committing a large asset policy for a new team.

## What Usually Belongs In The Unity Repository

- Scenes, prefabs, ScriptableObjects, materials, shaders, animation controllers, animation clips.
- Game-ready sprites, textures, fonts, sounds, music, and models that the project references.
- Matching `.meta` files.
- Project settings and package manifests.
- Source code and tests.

## What Usually Stays Outside

- Raw PSD/PSB files not required by Unity.
- Large Aseprite or Blender source files if only one artist edits them.
- Purchased asset pack archives.
- Reference image boards.
- Old exports, unused variations, and delivery zips.
- Temporary AI-generated batches and rejected art.

## When To Use Git LFS

Use Git LFS for large binary files that the team must version with the project:

- `*.png`, `*.jpg`, `*.tga`, `*.exr`
- `*.wav`, `*.mp3`, `*.ogg`, `*.flac`
- `*.fbx`, `*.obj`, `*.blend`
- `*.ttf`, `*.otf`
- `*.psd`, `*.psb`, `*.aseprite`

Use locking for files that cannot be merged safely:

```powershell
git lfs lock Assets/<ProjectName>/Art/source/example.psd
git lfs unlock Assets/<ProjectName>/Art/source/example.psd
```

## Practical Decision Table

| Asset type | Default location | Reason |
| --- | --- | --- |
| Runtime sprites/textures | Git with LFS if large | Project needs them to run |
| Runtime audio | Git with LFS if large | Project needs them to run |
| Scenes/prefabs/materials | Git | Text-serializable Unity project state |
| Raw source art | Local or asset library | Large, binary, often single-owner |
| Purchased asset zips | Local or asset library | Archives bloat history |
| Reference collections | Local or asset library | Not required to build |
| Rejected/old exports | Local archive | Not part of current project state |

## Checklist Before Adding Art

- Is the asset referenced by a scene, prefab, material, or ScriptableObject?
- Is the matching `.meta` file included?
- Is the file large enough to need LFS?
- Will multiple people edit this binary file?
- Does the repository size remain reasonable after adding it?
- Is there a better place for source material than the code repository?
