# Recommended Unity Project Layout

Replace `<ProjectName>` with the actual project namespace.

```text
Assets/
  <ProjectName>/
    Art/
      Animations/
      Materials/
      Shaders/
      Sprites/
      Textures/
      VFX/
    Audio/
      Music/
      SFX/
    Code/
      Runtime/
      Editor/
      Tests/
        EditMode/
        PlayMode/
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

## Folder Responsibilities

- `Assets/<ProjectName>/Art`: visual assets used by the project.
- `Assets/<ProjectName>/Audio`: music and sound effects used by the project.
- `Assets/<ProjectName>/Code/Runtime`: code that can be included in player builds.
- `Assets/<ProjectName>/Code/Editor`: Unity Editor tooling, inspectors, migrations, build helpers.
- `Assets/<ProjectName>/Code/Tests`: automated Unity tests.
- `Assets/<ProjectName>/Data`: ScriptableObjects, input actions, configuration assets.
- `Assets/<ProjectName>/Prefabs`: reusable game objects.
- `Assets/<ProjectName>/Scenes`: production scenes.
- `Assets/<ProjectName>/Scenes/Dev`: development and experiment scenes.
- `Packages`: Unity package manifest and embedded packages.
- `ProjectSettings`: Unity project settings.
- `Docs`: team documentation.

## Naming Guidance

- Keep one project namespace under `Assets/`.
- Do not recreate vague roots such as `Assets/Common` unless the project truly has shared packages across multiple games.
- Avoid moving files without their `.meta` files.
- Avoid renaming public serialized types unless you plan a migration.
