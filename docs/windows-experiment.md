# Windows experiment (Paul-Osborn fork)

This branch evaluates upstream cubefarm 0.3.2, commit `11237cf554f21312a2aecd9758d611f2817fc71e`. It adds documentation, not a new controller or UI. The upstream fork relationship remains intact. Use native Windows 11, PowerShell, Git for Windows, Node 24, GitHub CLI and Google Chrome. WSL is not required.

## Install this fork

In PowerShell, from the folder where you keep projects:

```powershell
git clone https://github.com/Paul-Osborn/cubefarm.git
Set-Location cubefarm
git switch experiment/windows-first
npm.cmd ci
npm.cmd run build
```

Stop if an install/build command fails. `npm.cmd` avoids PowerShell execution-policy restrictions on npm.ps1; do not change your machine's execution policy. A source build uses the committed lockfile. `npx cubefarm` downloads the published upstream package, not this fork.

## Start the free demo

Use a separate state folder and port for the experiment. Set these variables again in each new PowerShell window; they affect only that window and its child processes.

```powershell
$env:SWARM_HOME = Join-Path $env:LOCALAPPDATA 'CubeFarmExperiment'
$env:SWARM_PORT = '4497'
node .\bin\cubefarm.js --demo --port 4497
```

Open http://127.0.0.1:4497 if the browser does not open. The launcher serves the built app, without the source launcher's automatic git update/restart loop. Demo agents and GitHub are simulated: no subscriptions or repository writes. Demo writes its own local state. Use `WASD` to walk, `E` to interact, `P` for the phone, `H` for help and `Esc` to close a panel/release the mouse.

If 4497 is occupied, choose another free port and change both places. Do not use 4317/5317 for this experiment. App previews and QA can use additional ports; do not run a second office against the same state folder.

## Connect real Claude Code and Codex

The demo cannot prove agent authentication or hooks. For a real trial, use a disposable GitHub project with a small issue and test suite. Keep production projects out of the first trial.

```powershell
gh auth login
node .\bin\cubefarm.js login
codex --version
codex login
node .\bin\cubefarm.js doctor
```

The bundled Claude Code binary comes from the Claude Agent SDK dependency. The `login` command signs in that binary using your Claude subscription; an existing separately installed Claude binary is not the executable cubefarm necessarily launches. Claude API-key/session environment variables are filtered by the launcher. **The CEO always runs Claude**, even if every developer uses Codex.

Codex must already be installed and signed in on Windows, and visible on PATH. Cubefarm launches the Codex CLI in a pseudo-terminal, not through the Codex app-server protocol. It injects task instructions, a Playwright MCP server, activity hooks and a completion notification. If the office asks you to trust its hooks, open that worker's terminal, type `/hooks`, and press `t` as directed. Until trusted, task/terminal visibility can work while detailed tool activity is incomplete. CLI versions and first-run screens must be checked on the actual laptop; `doctor` does not fully validate Codex compatibility.

Stop the demo with Ctrl+C. With the same environment variables set, start real mode by omitting `--demo`:

```powershell
node .\bin\cubefarm.js --port 4497
```

Complete setup, set the manager's **session limit to 1 or 2**, keep hiring approval enabled, and turn auto-merge off for the test project before letting workers handle issues. The default session limit is unlimited. Connect one disposable GitHub repository; hire a developer and a QA worker. Choose Claude and then Codex per worker to test both. Read the resulting PR, its QA report and checks before merging manually.

Cubefarm starts and controls its own agent sessions. It does not automatically observe unrelated Claude/Codex sessions you start elsewhere. Codex runs with approvals/sandbox bypassed; Claude's hook approves tools. Project/user skills and MCP settings can be loaded. Treat this like running trusted local coding agents with access to your Windows account. Task restrictions are instructions, not enforced isolation. Keep the office on loopback; do not publish its port through a tunnel.

## State, stop and reset

Default upstream home is `%USERPROFILE%\.cubefarm`. These instructions instead use `%LOCALAPPDATA%\CubeFarmExperiment`:

| Location inside experiment home | Contents |
| --- | --- |
| `demo-state.json` / `state.json` | Separate demo/real settings, projects, workers, QA, CEO and phone state |
| `workspaces` | Repository clones and worker worktrees |
| `sessions` | CLI integration/session files and browser outputs |
| `terminals` | Saved ANSI terminal screens/scrollback |
| `screens` | Latest worker browser images |
| `ceo` | CEO working directory |
| `pty.secret`, `pty-host.log` | Terminal keeper authentication and diagnostics; Windows uses a named pipe |

Agent login/config/history can also live outside this folder in the normal Claude/Codex user directories. GitHub issues, PRs, comments and screenshot evidence remain on GitHub. Deleting local state does not undo repository commits or those remote records.

Ctrl+C in the launching terminal stops the office, agents and floor previews through the normal shutdown path. An office restart can preserve agent terminals through its separate keeper; a crash/hard kill is different from a graceful stop. After a hard kill, verify experiment-related Node/agent processes have stopped before resetting. Do not kill all Node processes on the laptop. The keeper has an orphan timeout, not an immediate rollback guarantee.

For a reversible full reset, first stop the office, then rename only its experiment folder:

```powershell
$experimentHome = Join-Path $env:LOCALAPPDATA 'CubeFarmExperiment'
if (Test-Path $experimentHome) {
    Rename-Item -LiteralPath $experimentHome -NewName ('CubeFarmExperiment-backup-' + (Get-Date -Format 'yyyyMMdd-HHmmss'))
}
```

Next start creates a fresh office. Preserve the backup until you have recovered any uncommitted worker work. To reset only demo settings after a graceful stop, remove `demo-state.json` instead; other demo artifacts may remain. Never reset your normal `.cubefarm` folder accidentally.

## Validation and limits (2026-10-01)

| Check | Result |
| --- | --- |
| Local build, including TypeScript | Passed on Node 24/Linux cloud runner |
| Local unit suite | 297 passed, one terminal keeper recovery failure, one todo |
| Local packaged install/demo smoke | Passed; two floors, eleven simulated workers; client and state API served |
| Local browser suite | All six passed; WebGL scene, movement, help, phone and manager panels |
| Upstream CI at the inspected commit | Windows, macOS, Linux checks and browser job all passed; [run](https://github.com/leonvanzyl/cubefarm/actions/runs/36837045979) |
| ASUS Windows 11 laptop with real subscriptions | Not run; requires access to that machine and interactive account login |

The cloud keeper failure also reproduced in isolation; cause is unresolved. Do not treat the local unit suite as green or assume this reproduces on Windows. Existing Windows CI covers build/test/package startup, not an authenticated real Claude/Codex task.

Known upstream workflow issues: [#53](https://github.com/leonvanzyl/cubefarm/issues/53) describes fixes reported as pushed without a new commit; [#54](https://github.com/leonvanzyl/cubefarm/issues/54) describes resuming a stale QA session during a fix after restart. Avoid deliberately restarting mid-fix during the first real trial.

Code review also found that updating a behind PR's branch replaces its approved SHA without another QA run (`server/swarm.ts`, `advanceMerge`). QA's overall verdict can disagree with failed checks in its own report (`parseReport`/`onQaFinished`), and evidence publication failure does not block a passed verdict. These are candidates for focused upstream fixes, not changes made in this documentation branch. Use manual PR inspection during evaluation.

## What to measure

Run a small backend change, a UI change with a browser-visible acceptance criterion, a deliberately failing test, and a task interrupted once after baseline success. Repeat a comparable task with each CLI. Record time to notice a blocker, time to understand the current task without opening logs, correctness of QA, rework count, usage and whether the 3D view helps more than the Kanban/terminal panels. Demo performance is not real agent throughput.

Promote integration only if real activity is easier to understand and outputs remain verifiable. No UI redesign, production connection or second orchestration system is part of this branch.
