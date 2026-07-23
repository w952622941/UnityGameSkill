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

## Follow-up: Local SVN Art Repository

After the Git baseline and project restructure, a separate local SVN repository was created for art and large source assets.

Final SVN state:

- SVN repository path: `E:\SVN\Repositories\TF_2D_Art`
- SVN working copy: `E:\SVNWork\TF_2D_Art`
- SVN URL: `file:///E:/SVN/Repositories/TF_2D_Art`
- Initial committed revision: `1`
- Initial hotcopy backup: `E:\SVN\Backups\TF_2D_Art-r1-hotcopy`

Tools installed:

- VisualSVN Server
- Slik Subversion
- TortoiseSVN

Important decision:

- No business folders were created in SVN.
- No `trunk/branches/tags` structure was created.
- The user will define the art repository structure later.
- The SVN working copy stays outside the Unity Git project.

Problem encountered:

VisualSVN Server was installed and running, but changing its service-level repository root required elevated permission and failed with:

```text
Cannot open registry key HKEY_LOCAL_MACHINE\SOFTWARE\VisualSVN\VisualSVN Server: Access is denied.
```

Solution:

For the current single-machine use case, use a local `file:///` repository created by `svnadmin`:

```powershell
& 'C:\Program Files\SlikSvn\bin\svnadmin.exe' create 'E:\SVN\Repositories\TF_2D_Art'
& 'C:\Program Files\SlikSvn\bin\svn.exe' checkout 'file:///E:/SVN/Repositories/TF_2D_Art' 'E:\SVNWork\TF_2D_Art'
```

Then set root-level generic rules only:

- `svn:auto-props` for common large binary files with `svn:needs-lock`.
- `svn:global-ignores` for temporary files.

The Unity Git project was updated with:

- `Docs/SVNArtWorkflow.md`
- `/.svn/` in `.gitignore`

This records the Git/SVN boundary without putting SVN content into the Unity project.

## Reusable Lessons

- Establish baseline first; restructure second.
- Use branches to make structural work reviewable.
- A root `.gitignore` is a safety device, not a cleanup afterthought.
- Do not treat all art files the same. Runtime assets, editable source assets, references, and archives have different storage policies.
- If a GUI Git client fails, reproduce the operation with CLI and inspect proxy/auth settings.
- Prefer repo-local fixes when solving machine-specific network problems.
- Final verification must include both Git state and Unity project health.
- When SVN is used for art, keep it independent from the Unity Git project and avoid inventing business folders before the user needs them.

## Follow-up: Verified Mobile Online Battle Architecture

The same project later exposed a second class of reusable problems on a Huawei Android 10 device: high-latency movement, monsters snapping, stuck joystick input, delayed settlement, second-battle state leakage, and client/server replay divergence.

### What Did Not Work

- Driving a realtime battle through repeated HTTPS requests over a temporary tunnel. Around 600 ms RTT made the player and monsters visibly stall and jump.
- Queueing every joystick sample. Network delay turned old directions into future movement.
- Treating cloud-save balance as the authoritative economy.
- Calling Unity path APIs from a background save task.
- Measuring replay performance with a Debug server build.
- Assuming equal source configuration meant equal runtime configuration; one serialized direction table arrived as an empty object and caused deterministic divergence.

### Methods That Were Verified

1. Ordinary single-player PvE moved to Verified Local: the client runs the fixed-tick battle immediately, uploads bounded input evidence and checkpoints, and the server replays from Tick 0 before granting rewards.
2. The client/server deterministic core uses versioned rules, deterministic RNG, stable serialization, and exact cross-runtime golden hashes.
3. Joystick movement uses a latest-state mailbox and writes zero on pointer release/cancel, focus loss, pause, disable, and destroy.
4. Battle terminal state, asset ledger, and Outbox events are idempotent and atomic on the server.
5. DIAG exposes the first divergent Tick, input/ack cursors, versions, hashes, evidence batches, verification state, entity counts, displayed/earned currency, and stage timings.
6. Each new battle resets HUD, level, skill modal, timers, generators, input cursors, evidence, terminal, and continuation state.
7. Terminal UI is shown as soon as the terminal fact is known; the later asset refresh does not block it.

### Evidence

- DIAG identified the first replay divergence at Tick 20 and traced it to a 3600-entry direction table serialized as an empty object. Fixing the transport property and adding a cross-runtime test allowed the Android battle to verify and settle 50 gold correctly.
- A 2607-Tick replay originally took about 10.47 seconds in Debug. Release publication plus sparse replay snapshots reduced core replay to about 1.83 seconds while preserving all 130 checkpoints and the final hash.
- The backend suite passed 100 tests, the Unity suite passed 115 tests, and the authority validation suite passed after the fixes.

These results are evidence for the methods, not universal latency promises. New games must repeat the deterministic, weak-network, device, security, and settlement checks in `checklists/online-game-acceptance.md`.


## Follow-up: Production-like ECS, Capacity, and Motion Incident Evidence

The project then moved its integration backend from a temporary Quick Tunnel path to a Beijing ECS direct HTTPS/WSS path. Quick Tunnel remained useful only for short-lived development acceptance; the fixed ECS endpoint removed an avoidable relay and made load tests reproducible. The server-side deployment used containers, loopback-only application/database ports, a public reverse proxy on 80/443, TLS, health checks, image-based rollback, and environment isolation.

### Define a Battle Before Quoting Capacity

A statement such as “this server supports N battles” is meaningless unless the workload is fixed. The TF_2D capacity run recorded at least:

- one active player and one authoritative/verified battle session;
- target Tick rate and snapshot/input frequency;
- battle duration and warm-up;
- active enemies, projectiles, skills and spawn curve;
- verification/checkpoint and database write frequency;
- connection activity, request sizes and weak-network behavior;
- p50/p95/p99 latency, error rate, CPU, RSS, GC, database and network saturation.

The first tested 4-vCPU/8-GiB integration state used **15 concurrent battles as a soft operating ceiling and 20 as a temporary hard ceiling**; the original 30/50/100 breakpoint runs exceeded acceptable behavior. Those values are historical, not the final R10 result.

A later R10 optimization cycle corrected a hidden workload error: early “180-second” tests stopped being fully loaded when battles ended after roughly 60–86 seconds. The load generator was changed to replace terminal battles until the common deadline. Under that sustained model, the Stage33 integration image produced:

| Load | Result | p95 | API CPU average / peak | Decision |
| --- | ---: | ---: | ---: | --- |
| 30 × 180 seconds | 30/30 | 150.15 ms | 51.83% / 84.11% | Green target |
| 35 × 120 seconds, qualifier + three repeats | all 35/35 | 149.90–150.32 ms | peak 78.12–84.63% | Green, thin margin |
| 40 × 120 seconds, first round | 40/40 | 150.48 ms | 57.73% / 98.86% | Red; stop |

The 40-battle sample is the useful lesson: business success and a healthy p95 did not override a failed resource gate. The team stopped the remaining 40-battle rounds and did not run 45/50. Thirty battles became the operating-planning target; 35 remained an experimental edge, not comfortable capacity.

At the time this knowledge was recorded, the Stage33 image was built from an isolated, uncommitted TF_2D R10 worktree and had not been merged into the TF_2D `main` branch. This preserves the distinction between deployed evidence and repository state.

### Zero-Rule-Change Methods That Moved the Boundary

- replace 25-ms full database/checkpoint scans with startup recovery, an in-memory active registry and low-frequency ID reconciliation;
- keep one deterministic simulation resident per battle instead of decoding/rebuilding it every Tick;
- separate private 20-Hz state from 10-Hz presentation encoding while preserving urgent events;
- use stable logical lanes with bounded global simulation/checkpoint slots and single-writer ownership per battle;
- atomically merge heartbeat/control updates so an older snapshot cannot overwrite a newer Tick;
- narrow movement SQL to required fields, reuse resident authoritative state for reliable commands and enable bounded prepared statements;
- decouple database heartbeat persistence from WSS snapshot publication;
- pool canonical JSON, envelope and gzip buffers; reuse proven delta/checksum work without changing wire bytes;
- replace repeated candidate scans with result-equivalent direction hulls and resident projectile collections.

None of these methods reduced spawn counts, movement/skill rules, Tick, snapshot/input cadence, projectile checks, animation timing or anti-cheat validation. More aggressive concurrency, a one-round-trip movement candidate and a value-type enemy snapshot were rejected when measurements or correctness risk did not justify them.

### Database Retention Before Performance Conclusions

The integration database had accumulated about 180,000 replay checkpoints (about 494 MiB) and 220,000 input-history rows. Old load-test data can increase index/cache pressure, vacuum work, backup size and query cost, but deletion is not automatically a CPU optimization.

The safe order is:

1. make and verify a restorable backup;
2. classify production facts, active battles, recent diagnostics and disposable load-test data;
3. define retention by environment and table purpose;
4. delete in bounded batches with observability;
5. run the database's appropriate vacuum/analyze/index maintenance;
6. compare identical load tests before and after;
7. keep the cleanup job and retention policy in version control.

### AOI Decision

A cross-linked orthogonal list was reviewed but not adopted as the default. It can be useful for specialized 2D sweep workloads, but has higher mutation complexity and worse fit for frequent spawn/despawn than a simple grid.

The implemented direction was an **adaptive spatial hash/grid**:

- use a direct scan below a measured entity threshold;
- switch to nearby-cell lookup above the threshold;
- choose cell size from actual query radii and density;
- pool buckets/buffers and expose candidates, accepted results, cells visited and stage timings;
- preserve the old implementation behind a switch for A/B and rollback.

The decision is workload-driven. A different game should profile before selecting a spatial index.

### Root Cause of the Last Visible Small Displacement

Repeated tuning of interpolation and network thresholds did not solve the issue. A same-frame trace finally separated four facts: input, local deterministic state, authority state and rendered state.

Evidence from the failing build showed:

- input was already zero and the battle state was stationary;
- the remaining trace reason was `stationary_reconciliation`;
- rendered position and authoritative position still differed by roughly one world unit;
- an old deterministic presentation delta of about `0.125583336` matched the phone-observed displacement (about `0.1255836`).

This proved the visible shift was not new server movement or a missing joystick release. The presentation layer was still consuming an old reconciliation delta after input became zero or a skill-selection pause closed.

The verified fix was:

1. introduce an explicit `stationary_hold` presentation state when input is zero and no new authoritative motion exists;
2. clear residual presentation velocity and movement animation;
3. stop consuming the historical reconciliation delta while held;
4. preserve the last valid facing instead of replacing it with the zero vector or a default right-facing direction;
5. exit the hold only on new input, a newer authoritative state or an explicit recovery transition;
6. retain the server-authoritative position and all combat/economy rules unchanged.

The release trace passed all recorded movement/skill-modal scenarios, and the user confirmed that continuous direction changes followed by joystick release, plus repeated skill selections, no longer produced visible displacement. Facing preservation has its own acceptance item and should be signed off separately on both left- and right-facing cases.

### Resume and Diagnostics Lessons

- A resumable fight is identified by the same `battleId`, not by starting a visually similar new battle.
- Persistent pause time (including a pending skill offer) must survive process death; wall-clock time while stopped must not advance battle Tick.
- A pending skill offer must be restored without applying a choice the player did not make.
- Clipboard reports should be compact and current-build-specific.
- Export success must be based on write/readback verification, not merely the absence of a thrown exception.
- Unity paths such as `Application.persistentDataPath` must be obtained on the main thread and passed to background file work as plain strings.

For the complete chronological evidence, implementation artifacts, commands, failure modes and reusable instructions, read [TF_2D Engineering and Incident-Resolution Handbook](tf2d-engineering-handbook.md). For a shorter project-neutral procedure, read [Online Battle Incident Response](online-game-incident-response.md).
