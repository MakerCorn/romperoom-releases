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
- [Live card sync run](#live-card-sync-run)
- [Live standardise run](#live-standardise-run)
- [Live re-link run](#live-re-link-run)
- [Live tidy up batch run](#live-tidy-up-batch-run)
- [Live libraries run](#live-libraries-run)
- [Live across libraries run](#live-across-libraries-run)
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

| Spec                   | What it covers                                                                                                                                                                                                                                   |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `foundation.spec.ts`   | First run, persistence across a relaunch, an unreachable folder, an empty folder, the removal guard, unreadable files, keyboard use                                                                                                              |
| `security.spec.ts`     | No `require` or `process` in the page, the exact `window.romperoom` keys, `file://` blocked for `fetch` and XHR                                                                                                                                  |
| `deploy.spec.ts`       | The card wizard on a test volume: files, game lists, art and hashes on disk, a no-op re-run, a console moved aside, cancel, a full card, hostile IPC, zoom, gamepad                                                                              |
| `a11y.spec.ts`         | axe with zero violations on the wizard, library, drawer and health, light and dark; focus and the live region while sorting folders; real keys in a picker; "Show all"                                                                           |
| `layout.spec.ts`       | Small windows at high zoom (up to 400%), forced colours, hostile file names                                                                                                                                                                      |
| `resilience.spec.ts`   | WebRTC and DNS probes with positive controls, dropped files, a crashed renderer reloading mid-scan                                                                                                                                               |
| `tidy.spec.ts`         | Tidy up on the fixture: set aside and undo byte for byte, a chosen keeper, a stop part way (finished, or the rest discarded), a crash and recovery, delete forever, busy, keyboard only, 320 px, a file held in two libraries (Across libraries) |
| `dat-download.spec.ts` | Game database downloads over the fixture transport (no GitHub): nothing requested without a press, two DATs with their labels, stop, exact URLs for the official pages                                                                           |
| `standardise.spec.ts`  | Standardise on its own scratch library: review, run and Undo through the bridge and the screens, files and the game list on disk, refusals, axe light and dark                                                                                   |
| `relink.spec.ts`       | Re-link on its own scratch library: review, run and Undo through the bridge and by keyboard, files and the game list on disk, refusals, axe light and dark, no request                                                                           |
| `identify.spec.ts`     | A game database imported through the host's picker, identify through the bridge and from Health, a scan that identifies by itself, a file that is not a DAT refused                                                                              |
| `cover-art.spec.ts`    | Cover art over the fixture transport (no GitHub): nothing requested before Get cover art, a verified download linked without a rescan, the day's listing cache, Stop, art from a card and Remove downloaded art, hostile IPC, the Health card    |
| `sync.spec.ts`         | Sync a card on a folder posing as a Batocera card: a new game and a save imported, then taken back by Undo this sync; paths and foreign ids refused; no request                                                                                  |
| `libraries.spec.ts`    | Settings › Libraries by keyboard: a second library added, scanned and removed, then the last one, with nothing in either folder changed                                                                                                          |
| `packaged.spec.ts`     | Not in this run: `npm run e2e:packaged -w @romperoom/desktop` runs it on the packaged app ([release.md](release.md#what-packaging-guarantees))                                                                                                   |
| `capture/`             | Not a test: the screenshot capture (see [Screenshots](#screenshots))                                                                                                                                                                             |

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
- The windows open on screen while the suite runs. The macOS and Windows CI runners have a
  desktop session, so no virtual framebuffer is set up; the Linux runner has none, and CI wraps
  the suite in `xvfb-run`.
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

`.github/workflows/ci.yml` runs on pull requests, on pushes to `main` and on **Run workflow**. A
newer push to the same ref cancels the run in progress. Every job uses Node 22.

| Job          | Runs on                                 | What it does                                                         |
| ------------ | --------------------------------------- | -------------------------------------------------------------------- |
| `test`       | macOS, Windows, Linux                   | `npm ci`, lint, type-check, `test:coverage`                          |
| `e2e`        | macOS, Windows, Linux                   | After `test`: build, the Playwright suite, traces on failure         |
| `package`    | macOS, Windows, Linux                   | After `test`: build, `package:dir`, `verify:package`, `e2e:packaged` |
| `security`   | The self-hosted Mac, else Ubuntu        | gitleaks over the full history; `npm audit`, reported only           |
| `commitlint` | The self-hosted Mac, else Ubuntu; PRs   | Each commit of the pull request, with `commitlint.config.js`         |

A draft pull request runs `test` and `e2e` on macOS only, and no `package`; each OS's legs run
on its self-hosted runner when its variable is set ([ci-runners.md](ci-runners.md)).

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

- `security` downloads the gitleaks release at `GITLEAKS_VERSION` and checks it against the
  pinned checksum for its runner (`GITLEAKS_SHA256_LINUX_X64` or `GITLEAKS_SHA256_DARWIN_ARM64`)
  before running it. Bump all three together.
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
  does, and runs Linux's launch under `xvfb-run`.

`.github/workflows/release.yml` builds the installers on a `v*` tag or **Run workflow** and drafts
a release; [release.md](release.md#the-release-workflow) describes it.

## Platforms

- **macOS** is where Romperoom is built and checked: unit tests, the end-to-end suite, the smoke
  and manual runs.
- **Windows is checked by CI only.** The `test` job runs the unit suites on the Windows runner (the
  self-hosted PC when `WIN_RUNNER_LABELS` is set, else `windows-latest`) and the `e2e` job runs the
  built app there; both have passed (see [CI jobs](#ci-jobs)). Nobody has run the app on Windows by
  hand. NTFS refuses control characters in names, so the hostile-name e2e cases that contain them
  run on macOS only. Symlink cases run only where the `canSymlink` probe (`test/fs-caps.ts` in the
  engine and the desktop app) can make links, which includes the elevated CI runner, and the survey
  script's POSIX cases are replaced by one that expects it to refuse. Without the symlink cases the
  engine falls below its coverage thresholds, so the full gate on Windows needs Administrator or
  Developer Mode. `ROMPEROOM_TEST_NO_SYMLINKS=1` skips them on any OS. Junctions need no privilege
  on Windows, so the deploy tests for a junction on the card that leads outside it, and for a
  guarded folder given through one, run on every Windows runner (elsewhere the same tests make a
  folder symlink). `ROMPEROOM_FIXTURE_SIMULATE_WIN32=1` builds the fixture's Windows shape on macOS
  (no locked folder, no symlink loop). That shows the suites pass without those parts. It does not
  show that anything works on Windows. The CI runners do load the native SQLite binding and launch
  the built app there. Never checked on a real Windows PC:
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

## Live card sync run

CI never syncs a real card: the end-to-end suite poses a folder as a Batocera card
(`ROMPEROOM_TEST_VOLUME`). Only a live run shows a real library's names and sizes, a real share's
file system (no hard links, case-insensitive names) and a real card's layout. Run it before a
release that changes anything under `packages/engine/src/sync/`,
`apps/desktop/src/main/sync-host.ts` or a profile's `saves` block. It is driven by hand with a
throwaway Playwright script over the built app (`npm run build`), in the manner of
`e2e/support.ts`, never committed:

1. **Library.** Copy two or three console folders from your library's share into a scratch
   library outside the share (never writing to the share), or make one of small files named with
   real No-Intro titles, with one large file (hundreds of MB) so hashing shows in the timings.
   Fingerprint the share's folders before and after (they must be identical; an error is "not
   evaluated", never "unchanged"), and the scratch library outside `.romperoom` (paths, sizes and
   content hashes).
2. **Cards.** Scratch folders in two device profiles' layouts (Batocera: `roms/<folder>` and
   `saves/<folder>`; muOS: `ROMS/<folder>` and `MUOS/save/file/<core>`), holding: a game the
   library has under the same name; a copy of a library game under another name (the large one);
   a new game; a save the library lacks; a game whose save only the library holds; and a save
   state. With a real handheld's card, use a copy of it, never the card itself. The card's id is
   `test:<real path>`, and ids over 100 characters are refused, so keep its path short.
3. **Sync.** Launch with a fresh `ROMPEROOM_DATA_DIR`, the scratch library through
   `ROMPEROOM_TEST_PICK_FOLDER` and the card through `ROMPEROOM_TEST_VOLUME`; set up the library,
   open Sync a card, read the card and record the review (present by name and by hash, the new
   game's folder, the saves and their actions). Sync. Check: the new game is in its console
   folder with the card file's SHA-1; the card's save is in `.romperoom/saves/<system>/`, the
   library's is on the card, the save state is left; the card has `.romperoom/card.json` with
   only `v` and `id`.
4. **Undo from the results.** The game and the new library saves are gone, the scratch
   library's fingerprint outside `.romperoom` equals the one before, and the next review offers
   them again. Sync again, then **Scan these consoles**: the new game appears on the wall.
5. **Again.** Read the card a second time: nothing to do, and no game is hashed again (the large
   copy's review takes milliseconds, not seconds: the card's hash cache).
6. **Each way, then both.** Change the card's save and a library save, and sync: each goes the
   other way, and the replaced copies are in `.romperoom/saves-backup/<system>/`. Change one save
   on both sides: the review shows a conflict set to **Decide later** and Sync explains it has
   nothing to do; pick the library's copy and sync: the card's save is backed up, then replaced.
7. **Undo from Recent syncs** an earlier sync whose save has changed since: it is left, and
   listed with its reason.
8. **Pull the card.** Add a few large games, sync, and rename the card's folder once the first is
   done: the run stops with "The card was removed", what was copied stays, nothing waits in Tidy
   up's Recovery, no part file is left, and **Undo this sync** removes what it imported.
9. **Throughout.** Snapshot both sides before and after every step: no file present before is
   gone after a sync, and no content is lost anywhere; the card is unchanged outside its save
   folders and `.romperoom`. The request log is empty and `network-log.json` absent; `lsof -nP
   -i -a -p <pids>` over the app's processes shows loopback only (Playwright's own `--inspect`
   and DevTools ports), with a listening socket in the script itself as the positive control.

What it cannot observe: Windows and Linux; a real handheld playing the synced save (the layouts
in [profiles.md](profiles.md#saves) are from each system's source, not checked on a device);
FAT32 and exFAT cards (the card is a folder on the Mac's disk); a card pulled out mid-write (a
renamed folder is not a removed device: a file already open is still read to its end); and
connections between `lsof` samples (the request log is the complete record).

**Last run: 2026-10-05,** macOS, the built app, a scratch library of 10 small files named with
real No-Intro titles across Game Boy, Game Boy Advance and NES, plus one 512 MiB Game Boy Advance
file, with one library save; a Batocera fixture card and a muOS fixture card. Your library's
share was not mounted, so the share run (copying real folders and fingerprinting the share) was
**not evaluated**. What it measured:

- the first review counted 6 games present (2 under another name) and offered Metroid Fusion
  for the folder Game Boy Advance, 2 saves to the library, 1 to the card and the save state left;
  it took 1.4 s, hashing the 512 MiB copy; the review after the sync took 0.06 s;
- the sync imported the game with the card file's SHA-1, both saves with the card's SHA-1s, wrote
  the library's save to the card and `card.json` with only `v` and `id`;
- Undo from the results restored the library's fingerprint outside `.romperoom` exactly, and the
  next review offered the game and both saves again; after a second sync, Scan these consoles
  put Metroid Fusion on the wall;
- one save each way went across, each replaced copy in `.romperoom/saves-backup`; a conflict
  defaulted to Decide later (Sync disabled, with its reason), and the library's pick backed up
  the card's save before replacing it;
- Undo from Recent syncs of a sync whose save had changed since was "Partly undone", listing the
  changed save and the backup that could not go back;
- renaming the card's folder during a 4-game, 1.6 GiB sync stopped it with "The card was
  removed" after 2 games; Recovery was empty, no part file remained, and Undo removed both;
- the muOS card synced per core: its game and save came in, and the library's saves went to the
  `mGBA` and `Gambatte` folders;
- across 8 sync and undo steps no file was removed and no content lost on either side; the
  library ended with exactly the 2 imported games added, the cards were unchanged outside their
  save folders and `.romperoom`, the request log was empty, `network-log.json` absent, and
  `lsof` saw loopback only (its control saw its own socket).

## Live standardise run

CI never standardises a real library: the end-to-end suite builds its own scratch library
(`e2e/standardise-fixtures.ts`). Only a live run shows real No-Intro and Redump names, disc games
of a realistic size, the built app's screens over a whole run, and a real process killed part way.
Run it before a release that changes anything under `packages/engine/src/standardise/`, the
journal's folder moves (`packages/engine/src/ops/journal.ts`) or
`apps/desktop/src/main/standardise-host.ts`. Drive it with a throwaway Playwright script over the
built app (`npm run build`), in the manner of `e2e/support.ts`, kept outside the repository and
never committed:

1. **Library.** Build a scratch library outside the repository and outside your library's share.
   Give it folders named otherwise than the profile names them, two of one console so they merge
   (`GBA` and `Game Boy Advance`; on a Mac `GBA` to `gba` is a case-only rename), and a folder of
   another console under an alias (`Mega Drive`, `PlayStation`). Name the files with real No-Intro
   and Redump titles: games under another name (`aw.gba`), games already named by their database,
   and one game no database knows. Add disc games of a `.cue` and two `.bin` tracks, one in a
   folder of its own and one beside a playlist (`.m3u`) and a note; make the tracks large
   (200 MiB each was enough) so that Stop and a kill land part way. Add a `gamelist.xml` in a
   console folder naming some games, with an `<image>` element and a comment, and one naming the
   disc games; a picture under `images/`; a save in `.romperoom/saves/<system>/` and one in a
   profile's layout (`saves/gba/`); and, in the folder that merges, an identical copy of a file of
   the other folder and a different file of the same name as one there. Write a DAT per console
   for these files (a logiqx file with the real header name; a disc's `.cue` entry carries the
   checksums of the cue sheet as it is on disk). If the share is mounted, copy real folders from it
   instead (never writing to it) and fingerprint the share's folders before and after: they must
   be identical. Fingerprint the scratch library: the path, size, modification time and SHA-1 of
   every file. A fingerprint that fails to read is "not evaluated", never "unchanged".
2. **Identify.** Launch with a fresh `ROMPEROOM_DATA_DIR`, the scratch library through
   `ROMPEROOM_TEST_PICK_FOLDER`, and one path through `ROMPEROOM_TEST_PICK_FILE` whose file you
   replace with each DAT before importing it. Set up the library (Choose your ROM folder, Go to my
   library), import each DAT (a header the app cannot place, such as
   `Sega - Mega Drive - Genesis`, asks for its console), then Identify games.
3. **Review.** Tidy up › Standardise, choose ES-DE, Review the changes. Record the
   folders (renamed, merged, left and why), the games (offered, left and why, what follows), the
   clashes and the game lists. `standardiseReview` through the bridge gives the same as data.
4. **Run.** Tick the identical clash, Standardise. Check: folders and games have their new names;
   the merged folder keeps only the different file; the disc games' `.cue` and `.m3u` name the
   new parts; the picture and both saves follow their game; each `gamelist.xml` differs, line by
   line, only in the renamed games' `<path>` and `<image>` elements; the originals of every game
   list, cue sheet and playlist the run replaced are in `.romperoom/lists-backup/<run>/`; the
   set-aside copy is in `.romperoom-quarantine`; every SHA-1 from before is somewhere after.
5. **Scan and Undo from the results.** Press Scan again on the results, then Undo this run. The
   fingerprint outside `.romperoom` equals the one before, modification times included. Review
   again: it equals the first review, the scan between the run and its Undo notwithstanding (the
   disc games are offered again, none left as not matching).
6. **Stop, then Finish or undo….** Review again, tick the clash, Standardise, and press Stop a
   few steps in. Each game is either renamed whole or untouched, the run says Stopped with the
   changes that did not run, and its journal waits in Tidy up's Recovery. Press Finish or undo…,
   then Finish: everything is renamed and both game lists are rewritten. Scan, open the library:
   the renamed game's picture shows on the wall. Undo the run from History.
7. **A kill.** Review and run again (the bridge is enough), and kill the app's process
   (`SIGKILL`) about half way, while a disc game is being renamed. Fingerprint. Start the app
   again with the same data folder: the "Finish tidying up" panel opens by itself; press Undo what
   was done. The fingerprint outside `.romperoom` equals the one before.
8. **A frontend.** If one that reads a ROM folder and `gamelist.xml` is installed (ES-DE,
   Batocera), point it at the scratch library after a run and rescan: the renamed games show with
   their names from `gamelist.xml`.
9. **Throughout.** After every step, every SHA-1 from before is somewhere in the library (a
   set-aside copy in `.romperoom-quarantine`, an original in `.romperoom/lists-backup`). At the
   end the request log is empty and `network-log.json` absent.

What it cannot observe: Windows and Linux; a case-only folder rename on a case-sensitive volume;
FAT32, exFAT and network shares (the library is on the Mac's own disk); a frontend's own database;
and saves on a card.

**Last run: 2026-10-06,** macOS, the built app, a scratch library of 34 files (2.0 GiB) named with
real No-Intro and Redump titles across Game Boy Advance, PlayStation and Mega Drive, with five disc
games of a `.cue` and two 200 MiB tracks (one beside an `.m3u`) and two `gamelist.xml` files.
Your library's share was not mounted, so the share run (copying real folders and fingerprinting
the share) was **not evaluated**. What it measured:

- the review renamed 3 folders (`GBA` to `gba` by letter case, `Mega Drive` to `genesis`,
  `PlayStation` to `psx`), merged 1 (`Game Boy Advance` into `gba`) and left `saves` (a device's
  saves); it offered 9 games and left none, listed 1 identical and 1 different clash, and 2 game
  lists of 2 entries each; the games already named and the unidentified one were not listed;
- folders renamed and merged: **observed**; the merged folder kept only the different file, the
  identical copy went to `.romperoom-quarantine`, and the folder stayed in place;
- a game renamed with its art and saves: **observed**; the picture, the library save and the
  Batocera-layout save took the new name, and after a scan the picture showed on the wall;
- the multi-file games: **observed**; the folder game's folder, `.cue` and tracks were renamed and
  its `.cue` names the new tracks; the other game's `.cue`, `.m3u` and tracks were renamed in
  place and its `.m3u` names the new `.cue`;
- `gamelist.xml`: **observed**; a line diff showed only the renamed games' `<path>` and `<image>`
  lines changed, CRLF line ends and the comment kept;
- backups: **observed**; the originals of 2 game lists, 5 cue sheets and 1 playlist were in
  `.romperoom/lists-backup/1/`;
- Undo from the results: **observed**; 49 changes put back, the fingerprint outside `.romperoom`
  (and inside it) equal to the one before, modification times included. The next review left the
  5 disc games ("A part of this game doesn't match the official data.") because Scan again had
  run between the run and its Undo; after another scan it equalled the first review, and without a
  scan in between (run, Undo, review) it was equal at once. Fixed since: Undo now gives a
  rewritten cue sheet's or playlist's catalog row back what it said before the run. The engine
  suite covers it (run, scan, Undo, review, run again, for a cue sheet and a playlist); this live
  run was not repeated;
- Stop, then Finish or undo…: **observed**; Stop at step 4 of 45 ended the run Stopped with 4
  folders and 1 game done and 8 changes not run, each game whole; its journal waited in Recovery,
  and Finish renamed the rest and rewrote both game lists; Undo from History then restored the
  fingerprint;
- a kill mid-run, then rollback: **observed**; killed at step 22 of 45 with a disc game's folder
  renamed but not yet its files; the next start opened the panel, the run read Interrupted, and
  Undo what was done restored the fingerprint outside `.romperoom` exactly;
- a frontend rescan: **not evaluated**; no frontend that reads a ROM folder or `gamelist.xml` is
  installed on this Mac (OpenEmu is, but it imports games into its own library);
- the share: **not evaluated**, the share was not mounted;
- the network: **observed**; the request log was empty and `network-log.json` absent;
- throughout, every SHA-1 from before was in the library after each run, stop, finish and kill.

## Live re-link run

CI never re-links a real library: the end-to-end suite builds its own scratch library
(`e2e/relink-fixtures.ts`). Only a live run shows real No-Intro titles, a game list as a real
frontend writes it, and the built app's screens over a whole run. Run it before a release that
changes `packages/engine/src/tidy/relink.ts`, `tidy/relink-run.ts`, the list rewriter
(`standardise/lists.ts`) or the standardise host's re-link calls. Drive it with a throwaway
Playwright script over the built app (`npm run build`), in the manner of `e2e/support.ts`, kept
outside the repository and never committed:

1. **Library.** Build a scratch library outside the repository and outside your library's share.
   If the share is mounted, copy two console folders from it with their `images/` and
   `gamelist.xml` (read only; never write to the share), and fingerprint the share's folders
   before and after: they must be identical. If it is not mounted, build the folders by hand,
   named with real No-Intro and Redump titles, with a DAT per console written for them (a logiqx
   file with the real header name), as a ROM manager would have left them after renaming:
   - `gb/`: `Tetris (World).gb` with its old pictures `images/Tetris (USA).png` and
     `snaps/Tetris (USA).png`; `Super Mario Land (World) (Rev 1).gb` (a `(Rev 1)` added) with
     `images/Super Mario Land (World).png`; two games of one title (`Donkey Kong (Japan, USA)
     (En).gb` and `Donkey Kong (World) (Rev 1) (SGB Enhanced).gb`) and a picture
     `images/Donkey Kong (World).png`; `Dr. Mario (World).gb` with its box art in `images/` and an
     old `images/Dr. Mario (Japan, USA).png` (its new name is taken); `Alleyway (World).gb` with
     box art in `covers/` and an old `images/Alleyway (USA).png` (the game already has box art);
     and `snaps/001 - Super Mario Land (World).png` (an index prefix). Its `gamelist.xml` starts
     with a byte order mark, has CRLF line ends and a comment, and names the old Tetris and Super
     Mario Land files with `<image>` and `<thumbnail>` elements, a game that is gone (Link's
     Awakening) and `./Donkey Kong (World).gb`.
   - `megadrive/`: `Sonic the Hedgehog (USA, Europe).md` (renamed from `Sonic The Hedgehog
     (USA).md`, letter case too) and `Columns (World).md`, with their old pictures, and a list
     with LF line ends naming both old files, plus a fresh entry for `./Columns (World).md` (a
     frontend that rescanned after the rename).
   - `psx/`: a two-disc game identified disc by disc (`Final Fantasy VII (USA) (Disc 1).cue` and
     `.bin`, `Disc 2`, and `Final Fantasy VII (USA).m3u`), an old picture and an entry naming
     `./Final Fantasy VII (Europe).m3u`.
   - `gbc/`: 1,200 games recognised by file name, each renamed from `(USA)` to `(World)` with its
     old picture and entry, so that Stop and a kill land part way (a whole run took 1.4 seconds).

   Fingerprint the scratch library: the path, size, modification time and SHA-1 of every file. A
   fingerprint that fails to read is "not evaluated", never "unchanged".
2. **Scan and identify.** Launch with a fresh `ROMPEROOM_DATA_DIR`, the scratch library through
   `ROMPEROOM_TEST_PICK_FOLDER`, and one path through `ROMPEROOM_TEST_PICK_FILE` whose file you
   replace with each DAT before importing it (`Sega - Mega Drive - Genesis` needs its console
   named); Choose your ROM folder, Go to my library, then identify.
3. **Review.** Tidy up › Artwork. Record the section: the pictures offered (and which
   start unticked), the entries offered and listed, and every reason; and the leftover list below
   it. `standardiseRelinkReview` through the bridge gives the same as data. Expected: Tetris (both
   pictures), Super Mario Land, Sonic, Columns and every `gbc` picture offered; Alleyway unticked
   in the section and in the leftover list with its note; Dr. Mario only in the leftover list,
   with "That name is already taken."; Donkey Kong, the index-prefixed picture and
   Final Fantasy VII in the leftover list with no suggestion; the Tetris, Super Mario Land, Sonic
   and `gbc` entries offered, the Columns entry listed "The game list already has an entry for
   this game.", and the Link's Awakening, Donkey Kong and Final Fantasy VII entries listed with no
   match.
4. **Run.** Re-link with the defaults. Check: each offered picture has its game's file name in its
   own folder; each `gamelist.xml` differs, line by line, only in the renamed pictures' media
   elements and the re-pointed entries' `<path>` (the byte order mark, line ends and comment kept;
   the `psx` list untouched); its original is in `.romperoom/lists-backup/<run>/`; every SHA-1
   from before is somewhere after; the leftover list no longer shows the renamed pictures, and
   the game's cover (`listGames`' `coverMediaId`, and the Tetris tile on the library wall) is the
   renamed picture without a scan.
5. **Undo from the results.** The fingerprint outside `.romperoom` equals the one before,
   modification times included.
6. **Stop, then Finish or undo….** Done, then Re-link again and press Stop as soon as the progress
   passes the first picture; the results say Stopped and the run waits in Recovery. While it
   waits, a standardise run of the library through the bridge (an empty choice, so nothing could
   be written) is refused. Press Finish or undo…, then Finish: every picture is renamed and the
   game lists are rewritten. Undo the run from History › Recent re-links: the fingerprint equals
   the one before.
7. **A kill.** Review and run again through the bridge, and `SIGKILL` the app part way. Start it
   again with the same data folder: the "Finish tidying up" panel opens by itself; Undo what was
   done. The fingerprint outside `.romperoom` equals the one before.
8. **A reload.** Take a re-link review through the bridge, reload the window, and run that
   review's plan id: it is refused ("this review belongs to another window, or has expired") and
   nothing changes.
9. **A frontend.** If ES-DE or Batocera is installed, point it at the scratch library after a run
   and rescan: the renamed pictures show on their games.
10. **Throughout.** At the end the request log is empty and `network-log.json` absent.

What it cannot observe: Windows and Linux; FAT32, exFAT and network shares (the library is on the
Mac's own disk); a frontend that rewrites its own list after the rename; the reverse refusal (a
standardise run waiting in Recovery refusing a re-link, covered by the engine tests); a reload
showing only the window's latest result of either kind (covered by the host tests); and the file
hashes a review caches in the catalog.

**Last run: 2026-10-07,** macOS, the built app, a scratch library of 2,429 files (133 MiB) named
with real No-Intro and Redump titles across Game Boy, Mega Drive, PlayStation and Game Boy Color,
with four `gamelist.xml` files. Your library's share was not mounted, so the share run (copying
real folders and fingerprinting the share) was **not evaluated**. What it measured:

- the review: **observed**; 1,207 pictures, 1,205 offered, 1 starting unticked (Alleyway, which
  already had box art, also in the leftover list with "You can re-link it above instead."), 1 left
  (Dr. Mario, `name-taken`, in the leftover list with its reason); Donkey Kong, the
  index-prefixed picture and the two-disc game's picture stayed in the leftover list with no
  suggestion; 1,207 entries, 1,203 offered and 4 listed (Columns with `game-has-entry`, Link's
  Awakening, Donkey Kong and Final Fantasy VII with no match); the review refused no game list;
  the button read "Re-link 2,408 items";
- the run: **observed**; "Renamed 1,205 pictures and fixed 1,203 game list entries" in 1.4 seconds;
  every offered picture had its new name in its own folder and the Alleyway and Dr. Mario pictures
  were still at their old names; the `gb`, `megadrive` and `gbc` lists equalled the expected text
  exactly (the `gb` list's byte order mark, CRLF line ends and comment kept; 5 lines changed in
  `gb`, 3 in `megadrive`, the Columns stale entry's `<image>` followed its picture while its
  `<path>` stayed), the `psx` list was untouched, and the 3 originals were in
  `.romperoom/lists-backup/1/`; no content was lost; the leftover list no longer showed the renamed
  pictures, and Tetris's cover went from none to the renamed picture, shown on its tile, with no
  scan;
- Undo from the results: **observed**; "Undid this re-link: 1,211 changes put back.", the
  fingerprint outside `.romperoom` equal to the one before, modification times included;
- Stop, then Finish or undo…: **observed**; Stop pressed at picture 13 of 1,205 ended the run
  Stopped with 44 pictures renamed, no game list touched and "2,364 items did not run."; the run
  waited in Recovery; Finish renamed the rest and wrote all three lists as the full run had; Undo
  from History › Recent re-links restored the fingerprint exactly;
- a re-link waiting in Recovery refuses a standardise run: **observed** (at the bridge, with an
  empty choice; the screen's wording not seen); "a standardise run of this library waits in
  Recovery: finish or undo it first";
- the reverse, a standardise run waiting in Recovery refusing a re-link: **not evaluated** (engine
  tests);
- a reload showing only the window's latest result of either kind: **not evaluated** (host
  tests);
- the file hashes a review caches in the catalog: **not evaluated** (the catalog was not
  inspected);
- a kill part way, then rollback: **observed**; killed at picture 40, with 41 pictures renamed
  and no content lost; the next start opened the panel with the run Interrupted, and Undo what was
  done restored the fingerprint outside `.romperoom` exactly;
- a review a reload forgot: **observed**; refused, nothing changed;
- a frontend rescan: **not evaluated**; no frontend that reads a ROM folder or `gamelist.xml` is
  installed on this Mac (OpenEmu is, but it keeps its own library);
- the share: **not evaluated**, the share was not mounted;
- the network: **observed**; the request log was empty and `network-log.json` absent.

## Live tidy up batch run

CI never runs these on a real library: the unit suites build their own temp libraries
(`packages/engine/test/tidy-batch.test.ts`) and the end-to-end suite its own sandbox
(`e2e/tidy.spec.ts`). Run it before a release that changes `tidy/orphans.ts`, `tidy/dedupe.ts`'s
`duplicateCover`, `tidy/purge.ts`'s ages or `tidy/recovery.ts`'s `discardJournal`. Drive the
built app (`npm run build`) by hand or with a throwaway Playwright script kept outside the
repository:

1. **Library.** A scratch library outside the repository and outside your library's share: copy
   two console folders with their art from the share if it is mounted (read only; fingerprint
   the share before and after: it must be identical), else build them by hand. Include leftover
   pictures of each cause (a picture with no game, one of a game whose file you removed, and an
   identical copy of a kept picture in a folder no frontend reads), and a few pairs of identical
   games, one with box art. A game removed before a scan leaves its picture unlinked, so it reads
   "No game in your library", not "Game removed". Fingerprint the scratch library (path, size,
   modification time and SHA-1 of every file; a fingerprint that fails to read is "not
   evaluated").
2. **Scan,** with a fresh `ROMPEROOM_DATA_DIR` and the library through
   `ROMPEROOM_TEST_PICK_FOLDER`.
3. **Artwork.** Record the **Show** row's counts and sizes, choose each cause, check that
   the list, **Select all**, **Select none** and **Set aside N files** speak of the pictures shown,
   and that a picture unticked in one cause stays unticked in All. Set aside one cause's pictures.
4. **Duplicates.** Each set shows its game's picture or the plain square; narrow the window to
   320 px and check nothing overflows sideways (`document.documentElement.scrollWidth`).
5. **Delete forever by age.** Quit, back-date the journals of the run from step 3 in the scratch
   catalog (`UPDATE op_journal SET created_at = …` by 40 days), start again, and check the
   select's counts (30 days: those files; 90 days: none, with the button unavailable and its
   reason), then delete the 30-day files forever and check only they are gone.
6. **Discard the rest.** Start setting aside the duplicates with a slowed run
   (`ROMPEROOM_TEST_TIDY_DELAY_MS`), press **Cancel** part way, then **Finish or undo…** ›
   **Discard the rest**: what moved stays set aside, nothing else moved (fingerprint), the
   overview no longer offers the run, and History says "Stopped: the rest was discarded". Then
   repeat with a kill part way instead of Cancel and record whether Discard the rest settles it
   or is refused.
7. **Record** each step as observed, failed or not evaluated, with the counts, in this section.

What it cannot observe: Windows and Linux; FAT32, exFAT and network shares (the library is on the
Mac's own disk); a real 30-day wait (the age comes from a back-dated catalog); and a killed run
that Discard the rest settles, since the test delay waits after a step reserves its name, so every
kill leaves that reservation behind (the engine tests cover a clean one).

**Last run: 2026-10-07,** macOS, the built app, a scratch library of 43 files (2.3 MB) built by
hand: 8 pairs of identical games (two with box art), 3 kept games with an identical copy of their
box art in `covers/`, 2 games whose files were removed after the first scan, and 12 pictures with
no game. Your library's share was not mounted, so the share run was **not evaluated**. What it
measured:

- the scan: **observed**; nothing changed in the library but the two files removed by hand.
- **Show:** **observed**; "All · 17 pictures (1.7 MB)", "No game in your library · 14 pictures
  (1.4 MB)" and "Extra copies · 3 pictures (308.1 KB)". The two removed games' pictures counted as
  "No game in your library" (see step 1), so **Game removed** never showed. Each choice listed
  only its pictures; Select none left "Set aside 0 files (0 B)", unavailable; Select all ticked
  the view again; Ghost 01 unticked under No game in your library stayed unticked under All
  ("Set aside 16 files (1.6 MB)"). Setting aside Extra copies moved exactly the 3 copies, and the
  Show row then hid (one cause left).
- **Duplicates:** **observed**; the two sets with box art showed it (`alt=""`, loaded), the six
  without showed the plain square. At 320 px Duplicates, Leftover artwork, Set aside and History
  had `scrollWidth` 320 of 320.
- **Delete forever by age:** **observed**; with the Extra copies run back-dated 40 days and the
  Gone pictures' run left as it was, the select read "Everything set aside (5 files)", "Set aside
  more than 30 days ago (3 files)" and "Set aside more than 90 days ago (0 files)" (the labels have
  since been shortened to "Everything", "Older than 30 days" and "Older than 90 days"). At 90 days
  **Delete forever…** was unavailable, described by "Nothing was set aside more than 90 days
  ago."; at 30 days the preview said "3 files (308.1 KB) set aside in library more than 30 days
  ago will be deleted forever.", and only those 3 files left the set-aside folder.
- **Discard the rest after Cancel:** **observed**; Cancel stopped a run of 8 after 3; after a
  restart the panel opened by itself with the new words, "3 files done, 5 files left" and Discard
  the rest with its line. Discarding changed nothing on disk (modification times included), the
  overview counted 0 interrupted and no longer showed the banner, and History read "Stopped: the
  rest was discarded".
- **Three kills (`SIGKILL`):** **observed**; three runs (5 duplicates, then two runs of 6
  pictures) each killed during a move, each leaving one empty reservation in the set-aside folder
  and no content lost. At the next start the panel listed all three. Discard the rest on the
  first was **refused** with "Romperoom couldn't discard the rest." and nothing changed; Undo what
  was done then put it back, and focus moved to the next run's heading; Finish moved the second
  run's 6 pictures, focus again on the next heading; Undo what was done put the third run's
  pictures back, and the panel closed with focus on the screen, not the page.
- the end: **observed**; every file's content from the start was still in the library, except the
  two games removed by hand.

## Live libraries run

CI never runs it on a real library: the unit suites build their own temp libraries
(`packages/engine/test/libraries.test.ts`) and the end-to-end suite its own sandbox
(`e2e/libraries.spec.ts`). Run it before a release that changes `packages/engine/src/libraries.ts`,
`addLibrary` or `removeLibrary` in `engine.ts`, or `renderer/settings/Libraries.tsx`. Drive the
built app (`npm run build`) by hand or with a throwaway Playwright script kept outside the
repository:

1. **Libraries.** Three scratch libraries outside the repository: two folders named `roms`, each
   with a few pairs of identical games (copies of two console folders from the share if it is
   mounted, read only, fingerprinting the share before and after: it must be identical; else
   built by hand), and a third big enough that its scan lasts a few seconds. Fingerprint all
   three (path, size, modification time and SHA-1 of every file; a fingerprint that fails to read
   is "not evaluated") before and after each add and remove.
2. **Setup** with the first, a fresh `ROMPEROOM_DATA_DIR` and `ROMPEROOM_TEST_PICK_FOLDER`. Quit
   and start again with the second as the picked folder and a slowed Tidy up
   (`ROMPEROOM_TEST_TIDY_DELAY_MS`).
3. **Add.** Settings › Libraries › **Add a library…**: the second is listed as `roms (2)`, "Not
   scanned yet", focus on its **Scan now**, and nothing scans. **Scan now**: its games and "Last
   scanned" with today's date.
4. **Refusals.** Pick, in turn, the second again, the first's `GBA` folder, the first's parent, a
   symbolic link to the second, and a folder that doesn't exist: "That folder is already one of
   your libraries", "That folder is inside one of your libraries", "That folder holds one of your
   libraries", "That folder is already one of your libraries", "Can't reach your game folder";
   nothing is added.
5. **Remove while a scan runs.** Add the third and press its **Scan now**: every **Scan now** says
   why it waits and the third reads "Scanning now". **Remove** `roms (2)` meanwhile: "A scan is
   running", the confirmation stays, nothing is forgotten.
6. **Not reachable.** Rename the second library's folder away, then switch tabs and back at once:
   its row stays, marked "Can't reach its folder. Is the drive connected?". Put it back. Then
   move it and leave a symbolic link at its old path: the row reads "Can't reach its folder" and
   **Scan now** still counts its games. Put it back.
7. **Remove while Tidy up runs.** Set aside some of `roms (2)`'s duplicates (a finished run), then
   start a slowed run of the rest and **Remove** it meanwhile: "Romperoom can't remove roms (2)
   right now", nothing forgotten.
8. **Remove while Recovery waits.** Start a slowed run of the first library's duplicates, kill the
   app (`SIGKILL`) once a file has moved, and start again: **Remove** `roms` says "Something
   Romperoom was doing in roms stopped before it finished", nothing forgotten or moved. Undo the
   run.
9. **Remove** `roms (2)`: asked first in place of the list, then gone from the list, announced,
   focus on "Your libraries"; its folder byte-identical (its `.romperoom-quarantine` too), the
   others untouched, its games gone from the wall, nothing of it in History or Set aside.
10. **The network log** (`datsNetworkLog`, and no `network-log.json`) is empty.
11. **Remove** the third, then `roms`: the confirmation says it's the only library, setup opens,
    Settings closes and focus is on `<main>`. Every folder is as it was before its remove.
12. **Record** each step as observed, failed or not evaluated, with the counts, in this section.

What it cannot observe: Windows and Linux; a share that hangs (the 3-second limit is pinned by the
engine tests' seam, and Add waiting on one is not tried); one folder reached by two real paths
(macOS has no bind mounts: the device and inode rule is pinned by the unit tests); an identify
run overtaken by a removal (the timing is not reachable by hand); a Standardise run waiting in
Recovery (pinned by the engine tests).

**Last run: 2026-10-07,** macOS, the built app, three scratch libraries built by hand: `roms`
(20 files: 8 pairs of identical Game Boy Advance games and 4 Super Nintendo games), `roms (2)` (18
files: 8 pairs and 2 NES games) and `slow` (2,403 files, 954 MB, its scan about 2 seconds). Your
library's share was not mounted, so the share run was **not evaluated**. The picked folder was
changed between adds through the same `ROMPEROOM_TEST_PICK_FOLDER` seam (the host reads it each
time the picker opens). What it measured, in 33 checks (31 observed, 1 failed and since fixed, 1
not evaluated; the 33rd is the failed cell's re-run below):

- setup, add and scan: **observed**; `roms (2)` listed as "0 games · Not scanned yet" with focus on
  **Scan now: roms (2)**, "Added roms (2). Press Scan now to count its games." announced and no
  scan running; after **Scan now** "10 games · Last scanned Oct 7, 2026". Neither folder changed.
- the five refusals: **observed**; each said the words above, focus stayed on **Add a library…**,
  and the list kept 2 libraries; no folder changed.
- remove while a scan runs: **observed**; during the third library's scan all three **Scan now**
  were `aria-disabled` and described by "A scan is running. Scan now works again when it
  finishes.", the third read "0 games · Scanning now", and **Remove from Romperoom** on `roms
  (2)` said "A scan is running" while the scan still ran; 3 libraries stayed.
- not reachable at once: **failed**; with the folder renamed away, switching to Appearance and
  back left the row without the line, though the host's own answer was already `unreachable`:
  the tab showed the list it had read less than 30 seconds before. Fixed in 356e7f3
  (`refetchOnMount: 'always'`, unit-tested).
- not reachable at once, re-run after the fix: **observed**; the built app at e254edb, two
  scratch libraries of 2 files each: with the folder present the row had no line; renamed away,
  switching to Appearance and back at once (0 seconds after the tab's last read) showed "Can't
  reach its folder. Is the drive connected?" within 0.1 seconds, the host answering
  `unreachable`; put back, the next switch cleared it. The folder was byte-identical afterwards.
- not reachable after 30 seconds: **observed**; the row stayed, marked "Can't reach its folder. Is
  the drive connected?".
- behind a symbolic link: **observed**; "Can't reach its folder", and **Scan now** counted its 10
  games ("Last scanned" moved on).
- remove while Tidy up runs: **observed**; with a slowed run of 6 copies going, "Romperoom can't
  remove roms (2) right now / Wait for Tidy up or Standardise to finish, then try again.", 3
  libraries stayed, and the run finished (6 moved).
- remove while Recovery waits: **observed**; after a `SIGKILL` during a run of 8, the next start
  opened the recovery panel (Later), Recovery listed the run, **Remove** `roms` said "Something
  Romperoom was doing in roms stopped before it finished", and the folder was identical; Undo
  then settled the run.
- the remove: **observed**; the confirmation in place with focus on its heading, then "Removed
  roms (2) from Romperoom." with focus on "Your libraries". Its folder was byte-identical,
  modification times and its 8 set-aside files included; the other two were untouched. Its games
  left the catalog (a search for one of them: 1, then 0; the NES console left the wall), its 2
  runs left History and its 8 set-aside files left Set aside (0 left), and History kept the first
  library's undone run.
- the network log: **observed**; empty, and no `network-log.json`.
- the last library: **observed**; "It's your only library, so Romperoom goes back to setup
  afterwards.", then setup, Settings closed, focus on `<main>`, no library left. Every folder was
  as before its remove, and no content was lost over the run.

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

## Live across libraries run

CI never runs this on a real library: the unit suites build their own temp libraries
(`packages/engine/test/tidy-across.test.ts`) and the end-to-end suite its own sandbox
(`e2e/tidy.spec.ts`). Run it before a release that changes `tidy/across.ts` or
`renderer/tidy/Across.tsx`. Drive the built app (`npm run build`) by hand or with a throwaway
Playwright script kept outside the repository:

1. **Libraries.** Scratch libraries outside the repository and outside your library's share:
   copy one console folder from the share into two of them if it is mounted (read only;
   fingerprint the share before and after: it must be identical), else build them by hand. Two
   folders named `roms` sharing games (more than 50, so the list pages), each also holding a
   game only it has, a pair inside the first only, the same bytes under two consoles, a disc set
   in both, and a file hard-linked into both; a third sharing three of the games; a fourth left
   unscanned; a fifth big enough that its scan lasts a few seconds; and two small ones that will
   come to overlap others. Fingerprint every library (path, size, modification time and SHA-1 of
   every file; a fingerprint that fails to read is "not evaluated") before and after each look.
2. **One library.** Set up with the first (a fresh `ROMPEROOM_DATA_DIR`, the folder through
   `ROMPEROOM_TEST_PICK_FOLDER`): **Across libraries** says "Only one library".
3. **Two and three libraries.** Quit, start again, add the others in **Settings** ›
   **Libraries** (**Add a library…**, then **Scan now**; the host reads the picker seam each time
   it opens). Record the sets, the line above them, the badges, the skipped count, the pager,
   the live region and where focus goes with **Look again** and **Scan again**, and whether the
   Library choice is hidden with its note.
4. **Not checked.** The unscanned library; a folder of the third made unreadable (`chmod 000`)
   and the library scanned again; the fifth's scan running; the second's folder moved away, then
   an empty folder at its place; a copy moved off since the scan; a parent folder of a library
   swapped for a symbolic link into another library (nested), or to another library's own
   folder (the same folder); a library's own folder swapped for a link. Put each back.
5. **During a tidy.** Start again with a slowed Tidy up (`ROMPEROOM_TEST_TIDY_DELAY_MS`), set
   aside the first library's pairs, look across while it runs, then undo the run.
6. **Nothing changed.** Every fingerprint identical; the network log empty.

What it cannot observe: Windows and Linux; a network share or a removable drive unplugged for real
(the folder is moved away); two mounts of one share; a share that hangs during the look.

**Last run: 2026-10-08,** macOS, the built app at 69431ef, seven scratch libraries built by hand
(your library's share was not mounted, so the share cells were **not evaluated**): `roms` (83
files) and `roms (2)` (76 files) sharing 8 Game Boy Advance and 60 Super Nintendo games and 4
games `roms` holds twice, `more` (5 files, three of the shared games), `fresh` (1 file, never
scanned at first), `slow` (3,007 files, 3.0 GB, its scan about 3.4 seconds), and `GBA` and
`roms (3)` (1 file each). What it measured, in 48 checks (46 observed, 0 failed, 2 not
evaluated):

- one library: **observed**; "Only one library", no **Look again**.
- two libraries: **observed**; named `roms` and `roms (2)`, both compared; 72 sets, "72 sets of
  copies in more than one library · 92.6 KB in extra copies" (the sum of the shared sizes);
  each card "Super Nintendo · In 2 libraries · 1.5 KB" with a `roms` and a `roms (2)` badge
  before each path; 6 skipped (the two-console pair and the disc set's two files in each), the
  hard-linked file neither listed nor counted, the game only one holds and the pair inside one
  library not listed; a pair one library holds twice is one set of 3 copies in 2 libraries.
- paging: **observed**; "Showing 1–50 of 72 sets", **Next** showed 22 with focus kept on it.
- live region: **observed**; nothing said on opening or paging; **Look again** pressed with
  Enter kept focus and said "72 sets of copies in more than one library · 92.6 KB in extra
  copies" once; pressed again with the same outcome, nothing new was said (the shared
  announcer's known limit).
- the Library choice: **observed**; on this tab the choice is `visibility: hidden` (Shift+Tab
  from the tab skips it) and "Every library is compared here." shows; on Duplicates the choice
  shows again.
- three libraries: **observed**; the three games read "In 3 libraries", extra twice their size.
- not scanned: **observed**; "fresh: It hasn't been scanned completely. Scan it again to include
  it.", the other three compared, and **Look again** said the outcome with " · 1 library not
  checked" (not on screen). **Scan again** by keyboard scanned it; when it left, focus moved to
  the heading, and the list updated by itself.
- partial: **observed**; with a folder unreadable the scan completed and `more` read "Its last
  scan didn't see every file"; readable again, **Scan again** on the tab made it compared.
- scanning: **observed**; **Scan again** pressed by keyboard on `slow` stayed focused,
  `aria-disabled` and described by "A scan is running. This list updates when it finishes.";
  **Look again** meanwhile read "slow: It is being scanned now."; when the scan ended the list
  updated by itself and only "Scan finished: 92 games found" was said.
- offline: **observed**; folder moved away, and an empty folder with only `.DS_Store` at its
  place, both read "roms (2): Its folder isn't available. Is the drive connected?", the other
  four compared (3 sets); back, compared again.
- gone: **observed**; one copy moved off: "1 file has moved or gone since the last scan — scan
  again to check it", **Scan again** offered, its set not listed; put back, listed again.
- overlap: **observed**; nested and the same folder through a link both read "Part of its folder
  is also another library, so its files could be counted twice." for both libraries and none of
  their files were listed; a library whose own path became a link read "Its folder isn't
  available" (the open limit); links removed, all seven compared.
- during a tidy: **observed** through the page's bridge (the window running a job shows its job
  panel in place of the tabs); sampled every 1.5 seconds while 5 copies were set aside: each
  moved copy left its set at once, no file read as gone, and the 72 sets stayed. **Not
  evaluated** during a Standardise or re-link run: the delay seam slows only Tidy up's renames.
- nothing changed: **observed**; every look left every library byte-identical; after the undo,
  `roms` had every file back with the same content and modification time, and nothing
  left set aside; the network log was empty.

## Screenshots

`docs/screenshots/` is generated, never edited by hand:

```sh
npm run build && npm run screenshots -w @romperoom/desktop
```

The script (`apps/desktop/scripts/capture-screenshots.mjs`) runs `e2e/capture` against the
built app and the fixture library. The window is 1280 x 800 at a device scale factor of 1. It
captures in Console Shelf, light and dark:

- Settings (Appearance, Libraries and Game databases, with the download review and results),
  the wizard and an unmapped folder;
- the library and the game drawer;
- Health, unreadable files and the removal review;
- Health's Identify card and the name matches to check;
- cover art: the Health card, its review and its results;
- the card wizard's device and check steps (where to and done in light only);
- Sync a card's review and results;
- Standardise's review and results, and the re-link review and results;
- Tidy up's overview, duplicates, across libraries and artwork tabs (the set-aside preview and
  the Set aside tab in light only). The artwork shot scans its own small library (one game, a
  picture no game has and an extra copy of a kept picture), so its Show row has two causes and no
  other shot changes. The across libraries shot scans two small libraries of its own ("roms" and
  "more", the second added in Settings › Libraries), sharing two games, in a window 900 pixels
  tall so both sets show whole.

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
