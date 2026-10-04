# Testing

How Romperoom is tested, how to run each layer, and what the tests can and cannot show.

## Contents

- [Strategy](#strategy)
- [Commands](#commands)
- [The fixture library](#the-fixture-library)
- [End-to-end tests](#end-to-end-tests)
- [Coverage gates](#coverage-gates)
- [Mutation testing](#mutation-testing)
- [Flake policy](#flake-policy)
- [Test seams](#test-seams)
- [CI jobs](#ci-jobs)
- [Platforms](#platforms)
- [Running against a real library safely](#running-against-a-real-library-safely)
- [Live identification run](#live-identification-run)
- [Live game database download run](#live-game-database-download-run)
- [Live cover art run](#live-cover-art-run)
- [Screenshots](#screenshots)
- [Fresh-clone gate](#fresh-clone-gate)

## Strategy

| Layer            | Tool                        | Where                                 | What it proves                                                             |
| ---------------- | --------------------------- | ------------------------------------- | -------------------------------------------------------------------------- |
| Engine           | Vitest, Node, real SQLite   | `packages/engine/test`                | Scanning, guards, hashing, journaled operations, deploy plans, the repository scripts, on real temp folders |
| Profiles         | Vitest                      | `packages/profiles/test`              | Schema rules, shipped profiles, `systems.json`, the generated `systems.md` |
| UI primitives    | Vitest, jsdom, axe          | `packages/ui/test`                    | Primitive behaviour, the token contract, no raw values in feature styles   |
| Desktop main     | Vitest, Electron mocked     | `apps/desktop/test`                   | Security policy, protocols, IPC contract, lifecycle, recovery              |
| Desktop renderer | Vitest, jsdom, a fake API   | `apps/desktop/test/renderer`          | Screens, focus, keyboard and gamepad navigation, friendly errors           |
| Docs             | Vitest                      | `apps/desktop/test/docs*.test.ts`     | Links, diagrams, screenshots, and docs that must agree with the code       |
| Workflows        | Vitest                      | `apps/desktop/test/workflows.test.ts` | Every third-party action is pinned to a commit SHA                         |
| End to end       | Playwright for Electron     | `apps/desktop/e2e`                    | The built app: real windows, real IPC, real files, security probes, axe    |
| Packaged app     | Playwright, plain processes | `apps/desktop/e2e/packaged.spec.ts`   | `--self-check`, refused launch switches, development seams ignored         |

The engine tests use real files and a real catalog. Only the hash pool's failure modes use a
fake pool. The desktop unit tests never load the real `electron` module. The renderer tests
fake layout per CSS selector (`test/renderer/layout.ts`), so virtualization and column maths
run for real. The axe checks in jsdom skip colour contrast, which `e2e/a11y.spec.ts` checks in
the real app.

The docs tests cover `README.md`, `docs/*.md` and the package READMEs. `docs.test.ts` checks
Mermaid syntax, links and anchors, the screenshot manifest, and that no doc states a test count.
`docs-guards.test.ts` checks the docs against the code: every doc is linked from an index, the
environment variable and IPC tables match the source both ways, the CSP block is the one the app
sends, the fixture table is the tree the fixture builds, the README's layout paths exist, every
`npm run` script named exists, no system count is typed by hand, no doc carries a private
path, email address or plan task number, and `decisions.md`'s ADRs are numbered without gaps,
listed in its Contents and never superseded by an earlier one. `packages/profiles/test` checks
the generated `systems.md`, the README counts and the profile field table. Each guard has a
control, and each was mutation-checked when it was written.

## Commands

| What                         | Command                                                   |
| ---------------------------- | --------------------------------------------------------- |
| Every unit test              | `npm test`                                                |
| With coverage (what CI runs) | `npm run test:coverage`                                   |
| One workspace                | `npm test -w @romperoom/engine`                           |
| One file, by name            | `npm test -w @romperoom/engine -- title`                  |
| One test, by name            | `npm test -w @romperoom/engine -- title -t "region tags"` |
| End-to-end (build first)     | `npm run build && npm run e2e -w @romperoom/desktop`      |
| End-to-end, one spec         | `npm run e2e -w @romperoom/desktop -- security`           |
| Smoke the built app          | `npm run smoke -w @romperoom/desktop`                     |
| Regenerate `docs/systems.md` | `npm run systems-doc`                                     |

Anything after `--` goes to Vitest or Playwright. A bare word filters test files by name, and
`-t` filters tests by title. From the repository root, the end-to-end command CI runs is
`npx playwright test -c apps/desktop/e2e/playwright.config.ts`.

The identify performance test (`packages/engine/test/identify-perf.test.ts`) has two sizes. By
default it identifies 10,200 files against 20,000 DAT roms and removes the DAT again, inside
`test:coverage` on every OS. Both sizes check the paging without timing: no query that loads
files for matching returns more than one page (1,000 rows), and the match pass takes exactly one
page per 1,000 files. The default size prints the durations and asserts no time or heap (raw
heap there is mostly garbage and depends on GC timing). The full size (100,000 matching files
plus 2,000 unmatched, 200,000 roms) also asserts the budgets, identify under 180 s and removal
under 60 s (three times that on Windows), and a retained heap under 256 MiB, sampled after a
forced GC (`vitest.config.ts` passes `--expose-gc` only when `ROMPEROOM_PERF_FULL=1`). It does
not fit CI's time limit, so it runs by hand on the self-hosted Mac:
`ROMPEROOM_PERF_FULL=1 npm run test:perf -w @romperoom/engine`.

The desktop tests import the engine's built `dist/`. After changing engine code, run
`npm run build -w @romperoom/engine` first. Tests remove every temp folder they create, so a
full `npm test` leaves no new `rr-*` entry in the OS temp folder.

## The fixture library

`packages/engine/test/fixture.ts` builds a small library in a temp folder. Every engine suite
that needs a library uses it, and so does the end-to-end `rr.sandbox('fixture')`. It is
synthetic: none of its names come from a real collection. A docs test checks that this table
lists exactly the paths the fixture creates.

| Path                                                 | What it exercises                                                  |
| ---------------------------------------------------- | ------------------------------------------------------------------ |
| `GBA/Advance Wars (USA).gba`                         | A plain ROM                                                        |
| `Game Boy Advance/Advance Wars (USA).gba`            | A byte-identical duplicate in a second folder for the same console |
| `Game Boy Advance/Advance Wars (Europe).gba`         | A regional variant: not a duplicate                                |
| `SNES/Zelda (USA).zip`                               | A zipped ROM (unpacked size differs from size on disk)             |
| `SNES/._Zelda (USA).zip`                             | AppleDouble junk next to a ROM: ignored                            |
| `SNES/.DS_Store`                                     | Finder junk: ignored                                               |
| `SNES/._Zelda (USA).png`                             | AppleDouble junk next to art: ignored                              |
| `SNES/images/Zelda (USA).png`                        | Cover art in a system's `images` folder, linked to the zip         |
| `Mystery Box/thing.bin`                              | A folder no system name matches: reported as unmapped              |
| `Mystery Box/cover.png`                              | Art in an unmapped folder: never walked                            |
| `psx/Broken (USA).zip`                               | A damaged zip: unreadable                                          |
| `psx/Empty (USA).zip`                                | An empty zip file: unreadable                                      |
| `nes/Tetris (World).nes`                             | A plain ROM                                                        |
| `GBA/images/Advance Wars (USA).png`                  | Cover art in a system folder                                       |
| `downloaded_media/gba/covers/Advance Wars (USA).png` | ES-DE's scraper layout                                             |
| `.romperoom/media/snes/box/Ghost (USA).png`          | Romperoom's own media store; no matching ROM, so it stays unlinked |
| `nes/locked/Hidden (USA).nes`                        | POSIX only: a folder with no permissions (`chmod 000`)             |
| `nes/loop`                                           | POSIX only: a symlink loop back to `nes`                           |

Windows has no chmod-style lock, and symlinks there need a privilege, so on Windows the
fixture leaves out the last two rows and the tests about them are skipped.
`ROMPEROOM_FIXTURE_SIMULATE_WIN32=1` builds that Windows shape on any OS (see
[Platforms](#platforms)). Call `unlockFixture()` before removing a fixture, or the locked folder
cannot be deleted.

## End-to-end tests

`apps/desktop/e2e` drives the **built** app (`out/main/index.js`) with Playwright's Electron
support, so run `npm run build` first. There is no browser to install.

| Spec                   | What it covers                                                                                                                                                         |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `foundation.spec.ts`   | First run, persistence across a relaunch, an unreachable folder, an empty folder, the removal guard, unreadable files, keyboard use                                    |
| `security.spec.ts`     | No `require` or `process` in the page, the exact `window.romperoom` keys, `file://` blocked for `fetch` and XHR                                                        |
| `deploy.spec.ts`       | The card wizard on a test volume: files, game lists, art and hashes on disk, a no-op re-run, a console moved aside, cancel, a full card, hostile IPC, zoom, gamepad    |
| `a11y.spec.ts`         | axe with zero violations on the wizard, library, drawer and health, light and dark; focus and the live region while sorting folders; real keys in a picker; "Show all" |
| `layout.spec.ts`       | Small windows at high zoom (up to 400%), forced colours, hostile file names                                                                                            |
| `resilience.spec.ts`   | WebRTC and DNS probes with positive controls, dropped files, a crashed renderer reloading mid-scan                                                                     |
| `tidy.spec.ts`         | Tidy up on the fixture: set aside and undo byte for byte, a chosen keeper, a stop part way, a crash and recovery, delete forever, busy, keyboard only, 320 px          |
| `dat-download.spec.ts` | Game database downloads over the fixture transport (no GitHub): nothing requested without a press, two DATs with their labels, stop, exact URLs for the official pages |
| `capture/`             | Not a test: the screenshot capture (see [Screenshots](#screenshots))                                                                                                   |

How the harness (`e2e/support.ts`) works:

- Each test gets its own temp folder, with a fresh `ROMPEROOM_DATA_DIR` and a copy of the
  fixture library. The fixture is imported from the engine's test folder, not copied.
  `ROMPEROOM_TEST_PICK_FOLDER` stands in for the folder dialog.
- Every console error from the main process or the page, and every uncaught page error, fails
  the test. The one scoped exception is the security test's own probe. Chromium logs each
  blocked `file:///etc/hosts` load, and that test consumes exactly those messages and asserts
  that they happened.
- On failure, a screenshot and a Playwright trace per launched app are kept in
  `apps/desktop/test-results/e2e/`. The HTML report goes to `apps/desktop/playwright-report/`.
  Open a trace with `npx playwright show-trace <trace.zip>`.
- The windows open on screen while the suite runs. The CI runners have a desktop session, so no
  virtual framebuffer is set up.
- The CI runners (macOS and Windows) show classic scrollbars, which take about 15 px of the
  window; a Mac with a trackpad shows overlay scrollbars, which take none. At 300% zoom that is
  5 CSS px, so a layout can fit locally and scroll sideways in CI. To see what CI sees on such a
  Mac, run the zoom test with a window 16 px narrower.

## Coverage gates

Each workspace's `vitest.config.ts` sets coverage `thresholds`. They sit at the measured floor
minus one point, so `npm run test:coverage` fails when a change lowers coverage. Raise them when
coverage rises. Coverage uses istanbul. Some folders and files (the engine's scanner, ops, tidy
and lock, the desktop main process and renderer `lib/`) have their own, stricter thresholds. Read
the current numbers in the config files rather than here.

The engine's hash worker (`src/hash/hash-worker.mjs`) is excluded from coverage. It runs in a
worker thread that istanbul does not instrument, and the hash tests drive it end to end. No
workspace sets `passWithNoTests`, so a workspace whose test glob matches nothing fails.

## Mutation testing

Coverage shows that a line ran, not that a test would notice it breaking. Every guard added in
Milestone 1 was mutation-checked by hand: break the code on purpose, watch the test fail, then
restore the code. From Milestone 2 onward that hand process is driven by `scripts/mutate.mjs`,
one in-place mutation, one command, one verdict. Its inputs travel only through the environment
(`MUT_FILE`, `MUT_OLD`, `MUT_NEW`, `MUT_CMD`), never argv, so a mutation can hold arbitrary code
without any shell-quoting risk. The file is read into memory, the single occurrence of `MUT_OLD`
is replaced with `MUT_NEW`, `MUT_CMD` runs, and the file is restored from that in-memory copy in a
`finally` (never `git checkout`, which would destroy unstaged work). The script prints exactly one
of three verdicts and exits accordingly:

- `KILLED` (exit 0): the command's test output reported a failure. The guard works.
- `SURVIVED` (exit 0): the command still reported success. Either the test does not guard what
  its name says, or the code is dead.
- `NOT-EVALUATED` (exit 2): the inputs were missing, `MUT_OLD` did not match the file exactly
  once, or no test summary line was found in the command's output (for example `No test files
  found`). This is a third outcome, never folded into `SURVIVED`: a probe that could not run is
  not a probe that passed.

```sh
MUT_FILE=packages/engine/src/dat/values.ts MUT_OLD='return v.toLowerCase();' MUT_NEW='return v;' \
  MUT_CMD='npm test -w @romperoom/engine -- dat-' node scripts/mutate.mjs
```

The rules that make a hand mutation mean something:

- Change one value or condition in place, keeping the signature. A mutation that drops a
  parameter fails every test for the wrong reason.
- Back up the file in the script that mutates it, and restore it in a `finally`. Never restore
  with `git checkout`: it destroys unstaged work.
- A mutation that survives is a finding. Either the test does not guard what its name says, or
  the code is dead. Explain every survivor before moving on.
- Fixtures with round values let mutations survive. Prefer values that a wrong formula cannot
  produce by accident.

Commit messages for a new guard say which mutation it was checked against.

The tidy engine (duplicates, unused artwork, undo and the purge) was checked with a scripted
run of more than 50 mutants, one guard each, over the full engine suite. Its tests follow the
same patterns as the deploy tests:

- **Fault injection at every step:** a simulated crash after step k of an apply, an undo and a
  purge, then finish or roll back; each ends with the catalog matching the disk.
- **Fuzz invariants:** seeded random libraries, applied and undone; no file is lost, the last
  copy always stays, and the catalog matches the disk.
- **The library is not the library:** an empty mountpoint, a swapped folder and a copied one,
  after a crash.
- **A 5,000-file plan** under 5 s (15 s on Windows runners).
- **The review's probes, kept:** `tidy-review.test.ts` holds the phase C1 review's probes (zips
  with different extras, hard links, a folder or the quarantine replaced by a link, blocked
  restores, a restore against an unfinished purge, a run that stops on an error or a cancel) and
  a test per guard the review's 22 scripted mutants got past. Each failed on the build before
  the fix. It also covers the fix's own guards that a second set of mutants got past: a kept
  copy that is in quarantine, a folder, a hard link or behind a link; a quarantine folder that
  is a file or resolves elsewhere; and an undo of a file reached through a link. A link on the
  way is simulated by injecting `realpath`, so these tests run on every platform.

Some guards are tests of the repository rather than the code. `fixtures-privacy.test.ts` in the
engine fails when a recorded volume fixture carries a volume name or a real UUID from the
machine it was captured on; recapture fixtures, then scrub them to the synthetic UUIDs
`00000000-0000-4000-8000-0000000000NN`. The game-list merge tests include a hostile list of
200,000 unclosed `<game ` tags that must be refused quickly, which catches a scan that turns
quadratic.

## Flake policy

- No retries, in unit tests or end to end. One worker for end to end.
- No fixed sleeps. Every wait is on a visible state or an event.
- A flaky test is a bug. Fix it or delete it; never mark it to retry. The fix goes in its own
  commit, and the message names the race (for example `22eaa09`, which waits for a pool to break
  rather than counting spawns).
- Known timing-sensitive area: the hash pool tests, which depend on worker start-up under load.
  They are tracked in [roadmap.md](roadmap.md#deferred-minors).
- Slow tests: CI runners are slower than a laptop, and Vitest's default timeout is 5 s. A unit
  test should take well under 1.5 s here. One that is slow by design (real respawn back-off in
  `hash.test.ts`, the macOS disk-image test) says why and sets its own timeout. In renderer tests
  over many controls, find them with `getByLabelText` or a container query: `getByRole` with a
  `name` computes the accessible name of every candidate, which made one test take 4.5 s.
- Speed budgets: two tests time the deploy planner at scale (10,000 games in `deploy.test.ts`,
  20,000 files planned and dry-run in `deploy-guards.test.ts`). Each also counts the work it
  should do (name claims; lookups on the card), so a slow algorithm fails on any runner, and the
  time limit is only a backstop. The limit is 3 times longer on Windows (`SLOW`). The Windows
  runner is several times slower at the pure-CPU plan (CI run 36901733052: 2.3 s, against 0.3 s
  on a Mac) and much slower at each file lookup: its dry run took 16 s, and keeping four lookups
  in flight (`lookAhead` in `writer.ts`) only brought that to 14.8 s (run 36903601542). So a
  fresh card is now checked per folder: a file in a folder the card lacks needs no lookup of its
  own, either for its real path (`verify.ts`) or for what is on the card (`writer.ts`). The
  20,000-file test runs on a fresh card and allows fewer than 100 real-path lookups; a second
  test runs on a card that has the folders, where every file is looked up once (not once per
  folder level), the lookups overlap at most four at a time, and they start in plan order. The
  perf line in the test output gives the time spent on real paths and on each side's lstat.

## Test seams

The app reads a few `ROMPEROOM_*` variables that exist only for tests and scripted runs. All of
them are listed, with whether a packaged build honours them, in
[configuration.md](configuration.md#environment-variables). In short:

- `ROMPEROOM_DATA_DIR` isolates each run's catalog. It is the only one honoured when packaged.
- `ROMPEROOM_TEST_PICK_FOLDER` replaces the folder dialog.
- `ROMPEROOM_TEST_SCAN_ONLY` limits scans to some folders.
- `ROMPEROOM_SMOKE_EXIT_MS` runs the self-check smoke.
- `ROMPEROOM_TEST_UNGUARDED_WEBRTC` is the positive control for the network probes.
- `ROMPEROOM_TEST_VOLUME` lists one folder as the only volume, for deploys without a card.
- `ROMPEROOM_TEST_EXPORT_FOLDER` replaces the export-folder dialog.
- `ROMPEROOM_TEST_DEPLOY_DELAY_MS` slows each chunk written, so a test can cancel mid-copy.
- `ROMPEROOM_TEST_VOLUME_BYTES` makes the test volume small, so a test can fill it.
- `ROMPEROOM_TEST_TIDY_DELAY_MS` slows each file a tidy job moves, so a test can cancel or
  crash mid-run.
- `ROMPEROOM_TEST_DAT_FIXTURES` swaps the game database transport for one that serves synthetic
  DATs from a folder, so the end-to-end suite never contacts GitHub. It also serves cover art:
  thumbnail listings from `art/trees/<repo>.json`, pictures from `art/raw/<repo>/<folder>/` and
  GitHub's rate reading from `api/rate-limit.json`, through the real allowlist, size and git SHA
  checks and request log (the stand-in GitHub of `e2e/cover-art.spec.ts`).
- `ROMPEROOM_TEST_OPEN_EXTERNAL` records each URL the host would open in the browser instead of
  opening it.

### The tidy end-to-end tests

`e2e/tidy.spec.ts` tidies the test's own copy of the fixture library and then reads the disk:
every set-aside file is in `.romperoom-quarantine` with its SHA-1, and undo puts each back
with the same SHA-1 at its old path. The harness's `kill()` ends the app with `SIGKILL` (no
quit handlers, nothing flushed) part way through a slowed run, and waits until the app's own
pid is gone. On Windows Playwright starts the app through cmd.exe, so there it ends the whole
tree with `taskkill /T /F`: killing only the shell left the app running with the
single-instance lock, and the next launch quit at once. The next launch over the same
data folder must offer recovery, and undoing must restore every file. The host is called
directly too (`window.romperoom.tidyPurge` with the wrong words) to prove the engine, not only
the screen, refuses. The busy test waits for the deploy facade to report `done`, never for a
file on the card: the manifest is written before the copy ends. The fixture's unreadable files
mean it never evaluates leftover artwork, so that test builds a small library of its own.

### The deploy end-to-end tests

`e2e/deploy.spec.ts` points `ROMPEROOM_TEST_VOLUME` at a folder in the test's own temp folder,
so the wizard lists it as the only card (removable, file system "other") and the writer copies
into it. The tests then read the folder: the profile's layout, SHA-1 against the library, the
game list, the manifest, `.romperoom-removed` after a console is left out, and no
`.romperoom-part` file left behind. `ROMPEROOM_TEST_VOLUME_BYTES` caps the folder's reported
size, and `ROMPEROOM_TEST_DEPLOY_DELAY_MS` makes a copy slow enough to cancel. These seams are
ignored in a packaged build. No test volume is a real removable disk, so the volume listing's
safety rules are covered by unit tests and by the [real-card run](#real-card-run), not here.
The fixture's `Advance Wars (USA)` is left out by the planner (its identical copy in a second
GBA folder clashes on the card's file name), so the copy assertions use Tetris and Zelda.

To run the same wizard against a FAT32 disk image by hand, attach an image as in the
[real-card run](#real-card-run) and start the built app with
`ROMPEROOM_TEST_VOLUME=/tmp/rr-card-mnt/RRCARD`; detach only that image's device afterwards.

## CI jobs

`.github/workflows/ci.yml` runs on pull requests and on pushes to `main`. A newer push to the
same ref cancels the run in progress. Every job uses Node 22.

| Job          | Runs on          | What it does                                                         |
| ------------ | ---------------- | -------------------------------------------------------------------- |
| `test`       | macOS, Windows   | `npm ci`, lint, type-check, `test:coverage`                          |
| `e2e`        | macOS, Windows   | After `test`: build, the Playwright suite, traces on failure         |
| `package`    | macOS, Windows   | After `test`: build, `package:dir`, `verify:package`, `e2e:packaged` |
| `security`   | Ubuntu           | gitleaks over the full history; `npm audit`, reported only           |
| `commitlint` | Ubuntu, PRs only | Each commit of the pull request, with `commitlint.config.js`         |

CI first ran on 2026-09-30 (commit 5a751e4) and failed on both runners. Windows stopped in
`npm ci`, which tried to compile better-sqlite3 (fixed by turning install scripts off, see
[development.md](development.md#install-scripts-are-off)). macOS timed out in one renderer test
that cost about 4.5 s locally (fixed; see [Flake policy](#flake-policy)). On pull request #1
the `test` job has since passed on both runners (Windows first in run 36819450456), and the
Windows-only failures it then showed were test timing, not product behaviour: the runner is far
slower at file-backed SQLite and at starting worker threads, so `testTimeout` is 20 s on Windows
and the hash-pool tests wait on events instead of fixed sleeps. Run 36823687024 was the first
to pass every job, `e2e` included, on both runners. As of 2026-10-01 `main` is green: run
36825558092 (commit fdb0d05) passed `test` and `e2e` on macOS and Windows, with every test
passing, plus `security` and, for pull requests, `commitlint`.

`npm ci` runs no install scripts (`.npmrc`), so every job that launches Electron runs
`node node_modules/electron/install.js` itself.

- `security` downloads the gitleaks release at `GITLEAKS_VERSION` and checks it against
  `GITLEAKS_SHA256` before running it. Bump both together.
- The audit step does not block, on purpose. An advisory in a runtime dependency often has no
  fixed release yet, and a red build nobody can act on teaches everyone to ignore CI. Findings
  go to the run's summary and a warning annotation. Review them before each release.
- The `e2e` job caches the Electron binary. `electron_config_cache` points at
  `$RUNNER_TEMP/electron-cache`, and the cache key is the runner OS, the architecture and the
  locked Electron version. After `npm ci` the job runs `node node_modules/electron/install.js`
  explicitly. With a warm cache it unzips from the cache; with a cold one it downloads.
- The `package` job packages the unpacked app (no installers) and checks it as a release build
  is checked: fuses, asar integrity and contents, `--self-check`, the refused launch switches
  and a launch of the packaged app ([release.md](release.md#what-packaging-guarantees)). The
  packaged app refuses the switches Playwright attaches through, so these tests run it as a
  plain process and read its exit code, output, files and ports. It caches Electron as `e2e`
  does. It has not run on GitHub yet: the packaged checks have passed on macOS only, by hand.

`.github/workflows/release.yml` builds the installers on a `v*` tag or **Run workflow** and drafts
a release; [release.md](release.md#the-release-workflow) describes it.

## Platforms

- **macOS** is where Romperoom is built and checked: unit tests, the end-to-end suite, the smoke
  and manual runs.
- **Windows is checked by CI only.** The `test` job runs the unit suites on `windows-latest` and the
  `e2e` job runs the built app there; both have passed (see [CI jobs](#ci-jobs)). Nobody has run the
  app on Windows by hand. NTFS refuses control characters in names, so the hostile-name e2e cases
  that contain them run on macOS only. Symlink cases run only where the `canSymlink` probe
  (`test/fs-caps.ts` in the engine and the desktop app) can make links, which includes the elevated
  CI runner, and the survey script's POSIX cases are replaced by one that expects it to refuse.
  Without the symlink cases the engine falls below its coverage thresholds, so the full gate on
  Windows needs Administrator or Developer Mode. `ROMPEROOM_TEST_NO_SYMLINKS=1` skips them on any
  OS. Junctions need no privilege on Windows, so the deploy tests for a junction on the card
  that leads outside it, and for a guarded folder given through one, run on every Windows
  runner (elsewhere the same tests make a folder symlink). `ROMPEROOM_FIXTURE_SIMULATE_WIN32=1` builds the fixture's Windows shape on macOS (no locked
  folder, no symlink loop). That shows the suites pass without those parts. It does not show that
  anything works on Windows. The CI runners do load the native SQLite binding and launch the
  built app there. Never checked on a real Windows PC:
  - drive letters and backslashes in real Electron (the pure path checks refuse them);
  - line endings;
  - the data folder location.
- **Linux** runs the whole suite (unit, end-to-end under `xvfb-run`, packaged) in CI, in an
  Ubuntu 24.04 x64 container ([ci-runners.md](ci-runners.md#a-linux-runner-in-docker)). The
  container runs as an ordinary user: as root, the fixture's unreadable folder is readable and
  the scan tests fail. Never checked on a real Linux desktop: the AppImage's sandbox where user
  namespaces are restricted (Ubuntu 23.10+), SD card readers, and desktop integration.

## Running against a real library safely

Run the survey script before a first scan of a big share. `scripts/nas-survey.mjs` reports what
each top-level folder holds, as the scanner sees it:

- files, and ROM files and bytes by extension;
- zips and disc images;
- what is ignored and why;
- unmapped folders, media folders and odd names.

```sh
npm run build
node scripts/nas-survey.mjs "/path/to/your/roms" /tmp/rr-nas-survey.json --max-minutes 20
```

It is safe to point at a library you must not change:

- It uses the engine's own walker and only lists folders and stats files. It never opens a file:
  no hashing, no zip reading.
- It writes nothing to the library. The report must go under the system temp folder or `/tmp`
  (every link resolved first), or the script refuses with exit code 2. A link, a second hard
  link, a non-file or another user's file at the report's name is refused too.
- The report is rewritten after each folder, so a run stopped by `--max-minutes` (default 20)
  keeps what it surveyed. A folder too big for the time left is reported with what was walked of
  it and `stoppedEarly: true`.
- A metadata walk over a network share is slow: expect minutes, not seconds, for a large
  library. Run it in the background and read the report.

To scan a live library from the app without walking all of it, set `ROMPEROOM_TEST_SCAN_ONLY`
to a few folders and use a scratch `ROMPEROOM_DATA_DIR`. To prove a live run left a share
untouched, record a metadata fingerprint of the folders involved before and after (paths,
sizes and mtimes, from a bounded `find`), and compare the two. The live identification run
below does this itself.

## Live identification run

`scripts/live-identify.mjs` runs the whole identify pipeline against a real library, read-only:
a scan of the named folders, a DAT import for each file in `--dats`, then one identify run, all in
a scratch catalogue under the system temp folder that is removed at the end.

```sh
npm run build
node scripts/live-identify.mjs "/path/to/your/roms" --folders gb,gbc,gba --dats /tmp/rr-live-dats \
  --out /tmp/rr-live-identify.json --max-minutes 60
```

- **DATs come from the maintainer**, placed under `/tmp` by hand. The script never downloads one,
  and none is ever copied into the repository. A DAT whose console the app cannot tell from its
  header is refused, as in Settings; name the console with
  `--bind "<file name in --dats>=<system id>"` (one per DAT).
- **The fingerprint rule:** before anything else it lists each named folder and every entry under
  it as its path, size and mtime, read with `lstat` only (a link is listed, never followed, and no
  file is opened), and lists them again at the end. Any difference is printed and recorded as
  `libraryChanged`, and is a read-only violation until explained. A fingerprint that cannot be
  taken (a share that dropped) is never "unchanged": the report says `fingerprintError` (the
  error's code) with `libraryChanged: null`, and the run exits 1.
- **The report holds counts only:** per system (`exact`, `nameOnly`, `unidentified`), per run
  (`byReason`, `notChecked`, `notEvaluated`), the DATs (name, version, games, bytes), timings and
  both digests. No ROM path or name: an error is written as its code or kind only (`ENOENT`,
  `DatFormatError`), and its message, which may name a path, is printed instead.
  The report file follows the survey's rules (under the temp folder, never through a link).
- `--max-minutes` (default 60) cancels the scan or the identify run, and the report says
  `cancelled`. `--keep-catalog` keeps the scratch catalogue and prints its path, for a by-hand
  check of the names it chose; delete it afterwards.
- **Exit codes:** 0 when the scan and every run completed and the library is verified unchanged;
  1 when either is not true (the report is still written); 2 for a usage error or a refused
  report, before anything is touched.
- The test seam `ROMPEROOM_TEST_LIVE_TOUCH` (append a byte to one file) is honoured only for a
  library under the system temp folder, so a variable left in a shell never writes to a share.

What the last live run measured (macOS, a NAS over SMB, three 8-bit console folders, FinalBurn
Neo console DATs, which list only FinalBurn Neo's sets rather than a full No-Intro catalogue):

- about 1,850 ROM files (0.5 GB) scanned in about a minute; three DATs of 300 to 800 games each
  imported in well under a second, and identify itself took under a second;
- per system, 76% to over 99% of games identified by hash and at most 1% by name only; the
  rest are not in those DATs (`not-in-database`), mostly titles the FinalBurn Neo sets leave out;
- every identified game took its title from the DAT description ("Jurassic Park", not FinalBurn
  Neo's set id `jpark`), with the region read from tags such as `(Euro)`; the PC-Engine
  descriptions carry no region tag, so those games have none;
- the fingerprint of about 8,000 entries was identical before and after, and so was an
  independent `find`/`stat` listing of the same folders.

## Live game database download run

CI never talks to GitHub: the end-to-end suite swaps the transport for the fixture transport
(`ROMPEROOM_TEST_DAT_FIXTURES`). What only a live run can show is GitHub's real TLS and answers,
real DNS, real libretro DATs passing the DAT reader, and the operating system's own behaviour.
Run it before a release that changes anything under `apps/desktop/src/main/dat-download/`, and
after re-recording the listing.

The procedure, on macOS, with the built app (`npm run build`) and no test transport:

1. **Library and data folder.** A scratch `ROMPEROOM_DATA_DIR` under `/tmp`, the real library
   through `ROMPEROOM_TEST_PICK_FOLDER`, and `ROMPEROOM_TEST_SCAN_ONLY` naming two no-intro
   consoles and one redump console (for example `GB,gamegear,pcenginecd`). Fingerprint those
   folders before and after (paths, sizes and mtimes, see
   [Running against a real library safely](#running-against-a-real-library-safely)).
2. **Capture.** With root, the `tcpdump` command in the
   design spec (Live verification section).
   Without root, poll `lsof -nP -i -a -p <pids>` every 100 ms over the app's main process and
   all its helpers, and resolve the hosts with `dig +short`. Run the positive control first
   (`curl -sI https://example.com` under the same poller): a control the capture does not see
   makes the capture "not evaluated", never "clean". DNS queries are not visible without root.
3. **Idle.** 60 s on Settings › Game databases with Network activity open: the log stays empty,
   `network-log.json` is not created, and the capture shows no connection but loopback.
4. **Official site.** Open the No-Intro and Redump pages: the browser opens the exact URLs and
   the log stays empty. Download a DAT there and press Import a DAT file to see whether the
   picker preselects it (a person has to do this step).
5. **Download for me.** Review and download the three files. Each imports with
   `No-Intro via libretro (CC BY-SA 4.0)` or `Redump via libretro (CC BY-SA 4.0)`; Details
   shows the commit, the file URL, the licence and the date; each DAT's header name equals its
   file base name; `downloads/` is empty afterwards; every connection goes to an address
   `raw.githubusercontent.com` resolved to, and Network activity lists exactly those requests.
   Then Identify now, and record the identification rate per console.
6. **Check for updates.** It adds only `api.github.com`, with 1 request (3 if the head moved).
7. **Offline.** Without touching the machine's network, run the app alone without a network:
   `sandbox-exec` with a profile that denies outbound IP except loopback, on a separate scratch
   data folder. Chromium's own sandbox cannot start inside another one, so that launch (only)
   needs `--no-sandbox`. Names still resolve (the system resolver is outside the profile); the
   connection is refused. The job stops with the offline message, no DAT is added and
   `downloads/` is empty. Turning Wi-Fi off is the by-hand equivalent.

What it cannot observe: macOS's picker preselection and any OS dialog (a person must press
them); DNS queries without root; system proxies, PAC files and private certificate authorities
(not supported); Windows and Linux; GitHub's rate limiting (never provoked on purpose); and
connections shorter than the polling interval, which `lsof` sampling can miss (the request log
is the complete record; the capture only corroborates it).

**Last run: 2026-10-03,** macOS, Electron 44, a NAS over SMB (Game Boy, Game Gear and PC Engine
CD folders, about 1,450 ROM files), pinned commit `d5bae90` of 2026-09-27. Everything above
passed except the picker step in step 4 (not observable: the run could not press the OS dialog)
and one finding: Redump does not serve HTTPS (`redump.org` refuses port 443; port 80 answers),
so the Redump page did not load. Fixed after this run, in the same release: the Redump link
is now `http://redump.org/downloads/`. The fixed link was not part
of the live run; checked separately, that URL answered 200 over http and https was refused.
What it measured:

- the `lsof` control caught 4 of 20 short (about 60 ms) `curl` connections and 3 of 3 held for
  a second; the idle minute (576 polls) and the official pages showed loopback only;
- three DATs (885 KB) downloaded and imported in 1.4 s, all from one `raw.githubusercontent.com`
  address that name resolved to; the request log and the capture agreed; Details, labels and
  header names as expected; `downloads/` empty. The main process resolved the name with
  Chromium's resolver rules in force, so those rules do not reach its client;
- Identify now named 96% of Game Boy and Game Gear games by hash (1,582 of 1,649 and 488 of
  511); the three PC Engine CD games (CHD images, while Redump lists BIN/CUE tracks) matched by
  name only;
- Check for updates made 1 request to `api.github.com` and reported "Up to date";
- offline (sandboxed) the job stopped with the offline message in 0.25 s, nothing imported,
  `downloads/` empty;
- the fingerprint of about 15,000 entries was identical before and after.

## Live cover art run

CI never talks to GitHub for cover art either: the end-to-end suite serves listings and
pictures from the fixture transport (`ROMPEROOM_TEST_DAT_FIXTURES`) and poses a folder as the
card (`ROMPEROOM_TEST_VOLUME`). Only a live run shows GitHub's real tree listings, rate headers
and TLS, real libretro pictures matching real No-Intro file names, and the `main`-branch
repositories. Run it before a release that changes anything under
`apps/desktop/src/main/art-download/` or `packages/engine/src/art/`, and after re-recording
`thumbnails.json`. It is driven by hand with a throwaway Playwright script over the built app
(`npm run build`), in the manner of `e2e/support.ts`, with no test transport:

1. **Library.** Copy two or three console folders from your library's share into a scratch
   library outside the share (never writing to the share), or make a scratch library of small
   files named with exact No-Intro titles that libretro-thumbnails has pictures for, plus a few
   it has none for. Include a console whose repository is on `main` (NAOMI, CD-i). Fingerprint
   the share's folders before and after (they must be identical), and the scratch library
   outside `.romperoom` (paths, sizes and content hashes).
2. **Capture.** Poll `lsof -nP -i -a -p <pids>` about every 100 ms over the app's main process
   and all its helpers (`pgrep -P`, recursively). Run the positive control first: a `curl`
   download held open for a second or more (`--limit-rate`) under the same poller; a control the
   poller does not see makes the capture "not evaluated", never "clean". Resolve
   `api.github.com` and `raw.githubusercontent.com` during the run; only their addresses and
   loopback may appear.
3. **Idle.** Set up the library (a fresh `ROMPEROOM_DATA_DIR`, the scratch library through
   `ROMPEROOM_TEST_PICK_FOLDER`) and stay 60 s on Health without pressing anything: no connection
   but loopback, `network-log.json` absent.
4. **Get cover art.** Only `api.github.com`, one `art-listing` request per console; the review's
   "used N GitHub requests" equals the log's count. Cancel and press it again: 0 requests.
5. **Download.** Only `raw.githubusercontent.com`; every `art_added` row's file has its recorded
   size and SHA-1, its git blob SHA equals the listing's (checked against a listing fetched
   independently) and it starts with the PNG signature. The library shows the covers without a
   rescan; a drawer shows a screenshot and a title screen. **Scan again**: every picture stays
   linked and `health().art` does not change.
6. **Remove downloaded art.** Every recorded file and every folder Romperoom made is gone, and
   the scratch library's fingerprint equals the one taken before.
7. **Card.** Relaunch with `ROMPEROOM_TEST_VOLUME` naming a fixture card folder in a device
   profile's layout (ES-DE: `ES-DE/downloaded_media/<console>/covers` and `titlescreens`), holding
   two pictures named after two library ROMs; import them through Import art from an SD card.
   Both are saved with the source `sd:es-de` and the card is unchanged. The card's id is
   `test:<real path>`, and ids over 100 characters are refused, so keep the folder's path short.

What it cannot observe: DNS queries without root; GitHub's rate limiting (never provoked on
purpose); system proxies, PAC files and private certificate authorities (not supported);
Windows and Linux; other devices' real cards (the card is a folder); and connections shorter
than the polling interval (the request log is the complete record; the capture corroborates it).

**Last run: 2026-10-04,** macOS, the built app, a scratch library of 22 small files named with
real No-Intro titles across Game Boy, Game Gear and NAOMI (`main`), three of them with no
picture upstream. Your library's share was not mounted, so the share run (copying real folders
and fingerprinting the share) was **not evaluated**. What it measured:

- the `lsof` control caught a held `curl` download (28 samples); boot, the scan and the idle
  minute on Health showed loopback only, the request log was empty and `network-log.json` absent;
- Get cover art made 3 requests (Game Boy 2.0 MB, Game Gear 0.9 MB, NAOMI 0.1 MB, 1.8 s in
  all), all to one `api.github.com` address; the review read "used 3 GitHub requests, 57 of 60
  left", and GitHub's own rate endpoint agreed (3 used); a second review made none;
- Download saved 57 of 57 pictures (13.6 MB, 10.8 s), all from one `raw.githubusercontent.com`
  address; every file matched its SHA-1, its listed git blob SHA and the PNG signature; covers
  showed at once and the Tetris drawer showed its screenshot and title screen; after Scan again
  all 57 stayed linked;
- Remove downloaded art removed 57 pictures and `.romperoom` with them; the library's
  fingerprint was identical to the one before;
- the ES-DE fixture card imported its 2 pictures (`sd:es-de`), with no connection but loopback,
  and the card was unchanged.

## Real-card run

`packages/engine/test/real-card.test.ts` runs the card writer against a real FAT32 file system:
a disk image, never a real disk. It is skipped unless `ROMPEROOM_REAL_CARD` is set, and runs on
macOS only. It covers listing and `statfs` against `df`, a first write of about 320 files (with
hostile names and a 50 MiB file) verified by independent hashes, a no-op second run, the FAT32
4 GiB check (a sparse source, never written), `EBUSY`/`EPERM`, a real `ENOSPC`, moving files
aside, cancel mid-file, and a forced detach mid-write followed by recovery.

```sh
# Refuse to run if a volume with this name exists already.
diskutil list | grep -q RRCARD && { echo "RRCARD exists"; exit 1; }
mkdir -p /tmp/rr-card /tmp/rr-card-mnt
hdiutil create -size 512m -fs "MS-DOS FAT32" -volname RRCARD -type SPARSE /tmp/rr-card/card
hdiutil attach -plist -nobrowse -mountroot /tmp/rr-card-mnt /tmp/rr-card/card.sparseimage \
  > /tmp/rr-card/attach.plist
# Note the whole-disk /dev/diskN in attach.plist (the dev-entry without an "s").
cd packages/engine && ROMPEROOM_REAL_CARD=/tmp/rr-card-mnt/RRCARD \
  ROMPEROOM_REAL_CARD_DEVICE=/dev/diskN \
  ROMPEROOM_REAL_CARD_IMAGE=/tmp/rr-card/card.sparseimage \
  ROMPEROOM_REAL_CARD_RECORD=/tmp/rr-card/reattached.txt \
  ROMPEROOM_REAL_CARD_EVIDENCE=/tmp/rr-card/evidence.json \
  npx vitest run test/real-card.test.ts
# The forced-detach test attaches the image again: detach the device it recorded, only that.
hdiutil detach "$(cat /tmp/rr-card/reattached.txt)"
```

Before it touches anything, `test/real-card-guard.ts` must prove all of this from
`hdiutil info -plist`, or the suite is skipped with the reason in its title:
`ROMPEROOM_REAL_CARD_IMAGE` is a regular `.sparseimage`/`.dmg` file, attached exactly once;
`ROMPEROOM_REAL_CARD_DEVICE` is a `/dev/diskN` of that image; `ROMPEROOM_REAL_CARD` is mounted from
one of its slices; and the mount is either empty or carries the test's sentinel file
(`.romperoom-real-card-test`, naming the image) and nothing but the test's own output. On an empty
image the test writes the sentinel first. It then removes only its own output, never the sentinel,
and asks the guard again right before the forced detach and after attaching the image again. A real
card is never empty and never carries the sentinel, so it is refused.

What the last run measured on macOS (Darwin 27, a 512 MiB image, 4 KiB clusters):

- 318 files and 52.5 MB in 4.1 s (12.2 MB/s), each one `fsync`ed and read back. A single
  50 MiB copy with one `fsync` took 53 ms on the same image; that is a cached sequential copy,
  so the two numbers are not like for like. The per-file cost is the overhead.
- The estimate was 55,414,784 bytes and the card lost 55,300,096: never below.
- A second run wrote nothing in 64 ms. A cancelled run and a detached card both recovered on
  the next run.
- macOS writes a 4096-byte `._` AppleDouble file beside every file and folder on FAT, and moves
  them with their files. Space freed on FAT shows up in `statfs` a little later.

## Screenshots

`docs/screenshots/` is generated, never edited by hand:

```sh
npm run build && npm run screenshots -w @romperoom/desktop
```

The script (`apps/desktop/scripts/capture-screenshots.mjs`) runs `e2e/capture` against the
built app and the fixture library. The window is 1280 x 800 at a device scale factor of 1. It
captures in Console Shelf, light and dark:

- Settings, the wizard and an unmapped folder;
- the library and the game drawer;
- Health, unreadable files and the removal review;
- the card wizard's device and check steps (where to and done in light only);
- Tidy up's overview and duplicates (the set-aside preview and the Set aside tab in light
  only).

It also captures the library in CRT Neon and Clean Modern (dark). It writes `manifest.json`,
which records for each file the surface, theme, mode, viewport, command and app commit. The
commit gets `-dirty` when `apps/` or `packages/` had uncommitted changes, so commit code changes
before capturing. Files the manifest no longer lists are deleted. The script fails if a listed
file is missing or larger than 400 KB.

Refresh the images in the same change as any UI change they show. Docs tests check three
things: the manifest and the files agree, the expected set of shots exists, and every shot is
used by a doc.

## Fresh-clone gate

Before calling a change done, run the full gate on a fresh clone of the committed branch. It
catches anything that only works because of files left in your working tree, such as a prebuilt
`dist/` or an untracked file:

```sh
git clone --branch <branch> <repo> /tmp/rr-verify && cd /tmp/rr-verify && npm ci && \
  npm run lint && npm run typecheck && npm run test:coverage && npm run build && \
  node node_modules/electron/install.js && \
  npx playwright test -c apps/desktop/e2e/playwright.config.ts
```
