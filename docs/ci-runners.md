# CI runners and Actions minutes

Hosted macOS minutes bill at 10 times, Windows at 2 times. A full matrix run costs roughly 90
billed minutes, and the free plan allows 2,000 a month. Every job can therefore run on
self-hosted runners (one per OS, below), and does whenever its variable is set; with all three
variables set, CI spends no hosted minutes.

## What runs where

| Trigger | macOS legs | Windows and Linux legs | package job |
| --- | --- | --- | --- |
| Draft pull request | yes | no | no |
| Pull request marked ready | yes | yes | yes |
| Push to `main` | yes | yes | yes |
| Manual run (Run workflow) | yes | yes | yes |

| OS | Variable (a JSON array of runner labels) | Unset |
| --- | --- | --- |
| macOS | `MAC_RUNNER_LABELS` | `macos-latest` |
| Windows | `WIN_RUNNER_LABELS` | `windows-latest` |
| Linux | `LINUX_RUNNER_LABELS` | `ubuntu-latest` |

Open pull requests as drafts while iterating; mark them ready for review to get the Windows and
Linux legs and the packaging check before merging. Commitlint and the secret scan always run, on the self-hosted Mac when
`MAC_RUNNER_LABELS` is set (else on Ubuntu).

A self-hosted runner needs nothing installed beyond what its section below lists: the workflows
install Node themselves, and the release jobs that run on the Mac (or Ubuntu) fetch a pinned,
checksum-verified `gh` when the runner has none
(`.github/actions/ensure-gh`). The release builds
upload their files with Node, so the Windows runner needs no `gh`.

## Self-hosted macOS runner

The macOS legs (and the release build's mac leg) run on a runner on the maintainer's Mac when the
repository variable `MAC_RUNNER_LABELS` holds a JSON array of its labels:

```
["self-hosted","macOS","ARM64","romperoom"]
```

Unset the variable and every leg goes back to `macos-latest` (hosted minutes).

- The runner lives in `~/actions-runner-romperoom` and is started by hand with `./run.sh`
  (stop it with Ctrl-C). It is not a background service, so it runs only while the maintainer
  wants it to. While it is offline, macOS jobs wait in the queue (and time out after 24 h).
- It takes one job at a time. The e2e jobs open real Electron windows on that Mac's desktop.
- Only this private repository may use it. Never register it on a repository that accepts
  pull requests from forks: a workflow runs arbitrary code on the Mac.
- Registration tokens are short-lived and are created with `gh api -X POST
  repos/MakerCorn/romperoom/actions/runners/registration-token`; nothing is committed.

## A Windows machine

A Windows PC can serve the Windows legs, taking the remaining hosted minutes to zero. Set it up,
or set it up again on a fresh machine, with one script, `scripts/windows-runner.ps1`:

1. On any machine with the `gh` CLI, make a registration token (valid for an hour):
   `gh api -X POST repos/MakerCorn/romperoom/actions/runners/registration-token --jq .token`
2. On the PC, in PowerShell inside a **logged-in desktop session** (not a service: the e2e jobs
   open real Electron windows), copy the script over and run `.\windows-runner.ps1`. Paste the
   token at the hidden prompt. It installs Git for Windows if missing, downloads the pinned runner
   and verifies its sha256, registers it, then starts it. `-NoRun` registers only; start later with
   `C:\r\start-runner.cmd` (not `run.cmd`: it puts Git's bash ahead of the WSL `bash.exe`, which
   would otherwise fail every bash step).
3. Leave the window open. Ctrl-C stops the runner. Running again is safe (it re-registers).
4. Remove it with `.\windows-runner.ps1 -Remove`, using a token from
   `.../actions/runners/remove-token`.

Over Remote Desktop, disconnect rather than sign out; a disconnected session may stop rendering
the desktop, which can fail the Electron tests (untested: if e2e fails only there, sign in at the
PC itself or use auto-login).

The Windows legs run on it when the repository variable `WIN_RUNNER_LABELS` holds its labels, e.g.
`["self-hosted","Windows","X64","romperoom-win"]`; unset, they use `windows-latest`.

To update the runner, bump `$Version` and `$Sha256` together in the script (the hash is in the
release notes of github.com/actions/runner, "BEGIN SHA win-x64").

To start it at sign-in, put a shortcut to `C:\r\start-runner.cmd` in the Startup folder
(`shell:startup`). It then runs in the signed-in desktop session, as the e2e jobs need.

## A Linux runner in Docker

The Linux legs run in a container built from `ci/linux-runner`:
Ubuntu 24.04 x64 with Electron's libraries, Xvfb (the workflows wrap e2e in `xvfb-run`) and the
packaging tools, plus the pinned runner verified by its sha256. It needs nothing from a desktop
session, so any Docker host works (the maintainer's runs on the Windows PC's Docker Desktop).

```
docker build -t romperoom-linux-runner ci/linux-runner
docker run -d --name romperoom-linux-runner --restart unless-stopped --shm-size=1g \
  --security-opt seccomp=unconfined -e RUNNER_NAME=<name> -e RUNNER_TOKEN=<registration token> \
  -v romperoom-runner-state:/home/runner/state romperoom-linux-runner
```

- `seccomp=unconfined` is required: Docker's default profile blocks the user namespaces
  Electron's sandbox uses, and the tests run with the sandbox on. Never add `--no-sandbox`.
- The token is read on the first start only; the registration lives in the volume, so
  `--restart unless-stopped` brings the runner back whenever Docker starts, with no new token.
- Its label is `romperoom-linux`: `LINUX_RUNNER_LABELS` is
  `["self-hosted","Linux","X64","romperoom-linux"]`.
- Building over SSH on Windows: BuildKit cannot reach Docker Desktop's credential helper outside
  the desktop session. Build with `DOCKER_BUILDKIT=0` and `DOCKER_CONFIG` pointing at a folder
  whose `config.json` is `{}` (the base image is public).
- What it cannot show: a real Linux desktop. In particular the AppImage's sandbox on Ubuntu
  23.10+, where unprivileged user namespaces are restricted, and SD card readers (no USB in the
  container). Try a build on a real desktop before publishing it.

## Artifact storage

Self-hosted runners still upload artifacts to GitHub, and artifact storage has an account
quota. A full quota fails every upload ("Artifact storage quota has been hit"); it stopped the
0.7.0 release build on 2026-10-05, and refused uploads even after every stored artifact was
deleted (GitHub recalculates usage every 6 to 12 hours).

So the workflows use artifacts only for traces and screenshots of a failed run (`ci.yml`'s
`e2e` and `package` jobs, the release build's packaged-app check), kept for 3 days. Each of
those uploads runs only on failure, in its own step with `continue-on-error`, so a full quota
neither fails the job nor hides the real failure. Release files never use artifact storage:
the release builds upload their installers to the private draft release, and the later jobs
download them from there ([release.md](release.md#the-release-workflow)); release assets do
not count against the quota.

To free space, delete old artifacts (`gh api -X DELETE
repos/OWNER/REPO/actions/artifacts/ID`).

## Shared workspace on self-hosted runners

A self-hosted runner reuses one working folder per repository. A sparse checkout leaves
`core.sparseCheckout` set there, and the next job that checks out the whole repository can be
left without the `packages/` folder ("No workspaces found: --workspace=@romperoom/profiles").
The release workflow therefore checks out the full repository in every job. To repair a runner
that already has the setting, move `_work/romperoom/romperoom` aside while it is idle; the next
job recreates it.
