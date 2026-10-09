# Development

How to build, run and change Romperoom. New to the repository? Read
[architecture.md](architecture.md) for how the pieces fit, [testing.md](testing.md) for the
tests and CI, [security.md](security.md) for the desktop security model, and
[configuration.md](configuration.md) for environment variables and test seams.

Requirements: Node.js 22.12 or newer (CI uses 22; development happens on 25; Electron 44,
`@electron/fuses` and electron-builder's Electron download need 22.12) and npm. No C++
toolchain is needed: install scripts are off ([below](#install-scripts-are-off)). macOS is the
verified platform. CI passes on Windows and Linux too, but only on CI runners (see
[Platforms](#platforms)).

## Contents

- [Commands](#commands)
- [Install scripts are off](#install-scripts-are-off)
- [Electron binary](#electron-binary)
- [Lint, format and commits](#lint-format-and-commits)
- [Native modules (better-sqlite3)](#native-modules-better-sqlite3)
- [Desktop renderer](#desktop-renderer)
- [Changing the IPC surface](#changing-the-ipc-surface)
- [Platforms](#platforms)
- [Known limitations](#known-limitations)

## Commands

| What                                  | Command                                        |
| ------------------------------------- | ---------------------------------------------- |
| Install                               | `npm ci`                                       |
| Fetch the Electron binary             | `node node_modules/electron/install.js`        |
| Lint (ESLint + Prettier)              | `npm run lint`                                 |
| Type-check every workspace            | `npm run typecheck`                            |
| Tests / with coverage                 | `npm test` / `npm run test:coverage`           |
| Build everything                      | `npm run build`                                |
| Engine tests only                     | `npm test -w @romperoom/engine`                |
| Desktop app with hot reload           | `npm run dev -w @romperoom/desktop`            |
| Build the desktop app                 | `npm run build -w @romperoom/desktop`          |
| Launch the built app, self-checked    | `npm run smoke -w @romperoom/desktop`          |
| End-to-end tests (after `build`)      | `npm run e2e -w @romperoom/desktop`            |
| Regenerate the docs screenshots       | `npm run screenshots -w @romperoom/desktop`    |
| Package: unpacked app (after `build`) | `npm run package:dir -w @romperoom/desktop`    |
| Package: installers and checksums     | `npm run package -w @romperoom/desktop`        |
| Check the packaged app                | `npm run verify:package -w @romperoom/desktop` |
| Packaged-app smoke                    | `npm run e2e:packaged -w @romperoom/desktop`   |
| Regenerate the app icons              | `npm run icons -w @romperoom/desktop`          |

From the repository root, the end-to-end command CI runs is
`npx playwright test -c apps/desktop/e2e/playwright.config.ts`. Packaging, its checks and the
release workflow are in [release.md](release.md#building-locally).

`typecheck` and `build` build `@romperoom/profiles`, then `@romperoom/engine`, then the other
workspaces, because the desktop app consumes the engine's built `dist/`. After changing engine
code, rebuild it (`npm run build -w @romperoom/engine`) before `dev` picks it up. In the desktop
workspace `typecheck` runs `typecheck:node` (main, preload, shared, `vite/`, tests) and then
`typecheck:web` (renderer).

Dev catalogues from before 0.1.0 live in the old data folder (`@romperoom/desktop`, beside
`Romperoom`): the app does not migrate them. Move that folder's contents into `Romperoom`, or
add the library again and re-scan ([configuration.md](configuration.md#data-folder)).

Every package, and the root, is `"private": true` and `"license": "UNLICENSED"`: nothing is
published (`apps/desktop/test/manifests.test.ts` pins it). Tests remove every temp folder they
create, so a full `npm test` leaves no new `rr-*` entry in the OS temp folder. The engine build
ends with `scripts/copy-worker.mjs`, which copies the hash worker into `dist/` using paths from
its own location (any working directory works) and fails with `copy-worker: missing …` when the
source is gone.

If your shell has `ELECTRON_RUN_AS_NODE=1` set (some editors and agent hosts export it), Electron
runs as plain Node and `dev` fails with "does not provide an export named 'BrowserWindow'". Unset
it: `env -u ELECTRON_RUN_AS_NODE npm run dev -w @romperoom/desktop`. The smoke script unsets it
itself.

## Install scripts are off

The root `.npmrc` sets `ignore-scripts=true`
([decision 15](decisions.md#15-install-scripts-are-off)), so `npm ci` runs no install script of
any dependency, and no `pre`/`post`/`prepare` script of ours either (`npm run <name>` itself
still works). Nothing needs one:

| Package (lockfile `hasInstallScript`)          | Its script                                 | Why it is not needed                                               |
| ---------------------------------------------- | ------------------------------------------ | ------------------------------------------------------------------ |
| `esbuild` 0.28                                 | `postinstall: node install.js`             | Checks the binary that npm installs as `@esbuild/<platform>`       |
| `esbuild` 0.25 (under electron-vite)           | `postinstall: node install.js`             | The same, for the copy electron-vite pins                          |
| `fsevents` (macOS, optional)                   | none in the installed package              | Ships a prebuilt `fsevents.node`                                   |
| `better-sqlite3` (not flagged)                 | implicit `node-gyp rebuild`                | Ships Node-API prebuilds (see below)                               |
| `electron-winstaller` (under electron-builder) | `install: node ./script/select-7z-arch.js` | Picks 7-Zip for Squirrel.Windows installers; Romperoom builds NSIS |

better-sqlite3 is the one that broke Windows CI: its own `package.json` says `"gypfile": false`,
but `npm ci` reads package metadata from the lockfile, which does not carry that field, and so
ran `node-gyp rebuild` because a `binding.gyp` is present. Electron 44 has no install script
(its binary is fetched explicitly, next section).

Before adding a dependency, look up its entry in `package-lock.json`. If it has
`"hasInstallScript": true` (or a `binding.gyp`), find out what the script does and make that a
command or a test here. To list them:

```sh
node -e "for (const [k, v] of Object.entries(require('./package-lock.json').packages)) \
  if (v.hasInstallScript) console.log(k, v.version)"
```

Running one script by hand, for a one-off check, is `npm rebuild <package> --ignore-scripts=false`.

## Electron binary

Electron 44 downloads its binary lazily, the first time `require('electron')` or the `electron`
CLI runs (not during `npm ci`), into `node_modules/electron/dist`. Offline installs therefore
succeed; the first `dev`/`smoke` needs the network (or `ELECTRON_OVERRIDE_DIST_PATH`). Unit
tests never load the real `electron` module: `security.test.ts` mocks it.

The download goes through `node_modules/electron/install.js`, which takes its download cache
folder from `electron_config_cache` (the npm-config form). It ignores `ELECTRON_CACHE`. Without
the variable the zip goes to the per-user default cache. To fetch the binary ahead of time, run
`node node_modules/electron/install.js`. It does nothing when the binary is already there.

In CI the `e2e` job sets `electron_config_cache=$RUNNER_TEMP/electron-cache`. `actions/cache`
restores and saves that folder, keyed on the runner OS, the architecture and the locked Electron
version (read from `package-lock.json` before `npm ci`). After `npm ci` the job runs `install.js`
explicitly: with a warm cache it unzips from it, and with a cold one it downloads (so a cold run
needs network access to the Electron release host). A lockfile change that keeps the Electron
version reuses the cache.

## Lint, format and commits

Lint is ESLint (`--max-warnings 0`, including the plain-ESM `.mjs` scripts and the hash worker,
with Node globals) and then `prettier --check`. Prettier wraps code at 100 columns, and ESLint's
`max-len` (100) catches what Prettier cannot, such as long comments. Strings, template
literals, URLs and regular expressions are exempt.

Commit messages are conventional commits: a header of at most 100 characters, a subject that
does not start with a capital letter, and body lines of at most 100 characters. Check a branch
before opening a pull request with `npx commitlint --from main --to HEAD`.

Coverage thresholds, mutation testing and the CI jobs are described in
[testing.md](testing.md).

## Native modules (better-sqlite3)

better-sqlite3 13 ships prebuilt **Node-API** binaries (`prebuilds/<platform>-<arch>.node`, built
with `NAPI_VERSION=10`) for `darwin`, `linux`, `linuxmusl` and `win32`, each on `arm64` and `x64`.
Its loader (`lib/binding.js`) takes that file first and looks in `build/Debug` or `build/Release`
only when it is missing. `packages/engine/test/prebuilds.test.ts` fails if macOS arm64 or x64,
Windows x64 or Linux x64 lose theirs, or if a `build/` folder appears. Node-API is ABI-stable across
runtimes, so the very same file loads under Node (engine tests; Node 25 is ABI 141) and under
Electron 44 (ABI 149, Node-API 10). There is no rebuild step and no second copy:

```sh
ELECTRON_RUN_AS_NODE=1 "$(node -p "require('electron')")" -e \
  "console.log(new (require('better-sqlite3'))(':memory:').prepare('select 1 x').get())"
```

This replaced the planned `@electron/rebuild`-into-`apps/desktop/native/` design, which would
only have been needed for an ABI-specific (non-Node-API) build. The escape hatch still exists:
`createEngine({ nativeBinding })` / `openCatalog(path, { nativeBinding })` take an absolute path
to a `better_sqlite3.node` and load it instead of the default resolution; a path that cannot be
loaded fails with "could not load the SQLite native binding at <path>". Use it if a future
better-sqlite3 drops Node-API prebuilds or the packaged app needs an explicit unpacked copy.

**sharp** (for media conversion, a later milestone) also ships Node-API prebuilds
(`@img/sharp-<platform>-<arch>`), so the same reasoning should hold; verify it the same way (load
it under `ELECTRON_RUN_AS_NODE=1`), add it to `asarUnpack` in `apps/desktop/electron-builder.yml`
and to the unpacked files `verify:package` expects
([release.md](release.md#what-packaging-guarantees)).

## Desktop renderer

`apps/desktop/src/renderer` is a React app over `window.romperoom`, with hash routes `#/setup`
(first-run wizard), `#/library` (the cover-art wall), `#/health`, `#/deploy` (the card wizard)
and `#/tidy` (Tidy up). `#/` redirects to the library when one exists, else to setup.

- `App.tsx` wires the shell (`shell/`: header, skip link, `<main id="main">`, settings drawer),
  the theme and the routes. The theme and light/dark mode are saved in localStorage
  (`romperoom.settings.v1`, `lib/settings.ts`). They are read synchronously, so `<html>` has its
  `data-theme`/`data-mode` before the first paint.
- `data/` holds the react-query hooks over an injectable api (`RomperoomProvider`, `api-context`).
  Tests pass a fake; `main.tsx` passes `window.romperoom`. `lib/friendly.ts` turns every error
  into plain copy. The raw message is shown only behind ErrorState's "Show details".
  `tidy/copy.ts` adds Tidy up's words, picked by the error's name (`errorNameOf()`, which reads
  the name the preload carried across the bridge).
- `tidy/` holds the Tidy up tabs. `useTidyJob.ts` runs the one tidy job (progress, Cancel, the
  result, adopting a running job after a reload), and `Recovery.tsx` the finish-or-undo drawer,
  opened at startup only when the first health reading counts interrupted work.
- `lib/nav.tsx` is one focus router shared by the keyboard and `gamepad/useGamepadNav.ts`. Zones
  (the wall, the shelves) own their arrow keys, `back()` closes the top panel, and
  `switchTab()` changes the shelf. The gamepad is polled with rAF only while a pad is connected.
- `library/GameWall.tsx` virtualizes rows (`@tanstack/react-virtual`) and pages 200 games at a
  time. It is one focusable region with a roving tab stop.
- Styling: `screens.css` and feature code use `@romperoom/ui` primitives and tokens only. The
  raw-value scan (`packages/ui/test/no-raw-values.test.ts`) covers every `.css`, `.tsx` and
  `.html` file here.
- No Node code: the only engine import is `@romperoom/engine/format` (pure, via the package
  `exports`). `vite/renderer-bundle-guard.ts` fails `npm run build` if the renderer's module
  graph or chunks contain a Node builtin (Vite's `__vite-browser-external` stub), better-sqlite3,
  or any other engine module. The build log prints
  `renderer bundle guard: N modules, M chunks, no Node code`.
- Tests (`apps/desktop/test/renderer`, jsdom) fake layout per CSS selector in `layout.ts`, so
  virtualization and column math run for real. `setup.ts` pins an in-memory localStorage
  (Node 25's global stub shadows jsdom's). The axe checks there skip colour contrast, which is
  checked in the real app.
- Real-app check: `npm run build`, then launch Electron on `out/main/index.js` with a temporary
  `ROMPEROOM_DATA_DIR` and `ROMPEROOM_TEST_PICK_FOLDER=<fixture>`, with `ELECTRON_RUN_AS_NODE`
  unset.

## Changing the IPC surface

Every channel is listed in `src/shared/ipc.ts`, and the handlers and the preload are built from
it. Adding an engine method touches several pinned lists on purpose; the steps are in
[security.md](security.md#ipc-channels).

## Platforms

- **macOS** is where Romperoom is built and verified: unit tests, the end-to-end suite, the
  smoke and manual runs.
- **Windows is checked by CI only.** CI runs the same jobs on the self-hosted Windows PC when
  `WIN_RUNNER_LABELS` is set, else on `windows-latest`
  ([ci-runners.md](ci-runners.md#a-windows-machine)). Its first run (2026-09-30) failed in `npm ci`,
  before any test ran; that is fixed, and the unit and end-to-end jobs now pass there. Nobody has
  run the app on a real Windows PC by hand. `ROMPEROOM_FIXTURE_SIMULATE_WIN32=1` only builds the
  test fixture's Windows shape on macOS. Details are in [testing.md](testing.md#platforms).
- **Writing code and tests that hold on Windows:**
  - Paths: build them with `join`; compare with `sep`, never `'/'`. Library-relative paths in the
    catalog use `/` on every platform, and so does any path shown to people.
  - Real paths: use `realpathSync.native` (or `fs.promises.realpath`) on both sides of a
    comparison. The JS `realpathSync` keeps 8.3 short names such as `RUNNER~1` on Windows.
  - `O_NOFOLLOW` and `O_NONBLOCK` are undefined there. Code that relies on them needs another
    check (the media protocol compares the opened file with the name), and
    `scripts/nas-survey.mjs` and `scripts/live-identify.mjs` refuse to run.
  - Symlinks need Administrator or Developer Mode, and there is no chmod-style lock. Tests
    that make links run when the `canSymlink` probe (`test/fs-caps.ts` in the engine and the
    desktop app) says they can, so they run on the elevated CI runner; never gate them on
    `process.platform`. Without them the engine's coverage gate fails.
  - Error codes: never decide what is on disk from an errno alone. Windows reports a directory
    where a file was expected as `EPERM`, not `EEXIST` or `EISDIR`; look with `lstat` first.
  - Temp folders: `tmpdir()`, never `/tmp`. Clean up with `rm(…, { maxRetries })`, because an
    antivirus or search indexer can briefly hold a file open (`EBUSY`).
  - Line endings: `.gitattributes` forces LF on checkout, whatever `core.autocrlf` says.
- **Linux** (x64) is built and tested by CI in an Ubuntu 24.04 container under Xvfb
  ([ci-runners.md](ci-runners.md#a-linux-runner-in-docker)); it has never run on a real Linux
  desktop. Linux can mount case-insensitive volumes (an SMB share, an exFAT card), so the
  journal asks the library's volume how it treats case rather than assuming from the platform.
- The packaged layout (`app.asar`, fuses, asar integrity) is checked on all three platforms by CI's
  `package` job and by every release build; signing is wired but not provisioned. See
  [release.md](release.md#what-packaging-guarantees).

## Known limitations

### Scanning and the catalog

- **A library whose ROM folders are all gone always ends could-not-read.** Suppose every system
  folder of a library with catalogued files is deleted, but a media root (`downloaded_media` or
  `.romperoom/media`) is still there. Every scan then ends `could-not-read` with "The library
  folder looks empty but the catalog has N files ...". This is deliberate. The whole-root-empty
  guard cannot tell that library from an unmounted share or an empty mount point, so it never
  lets the media roots vouch for the ROMs, and it never marks the files missing.
  `confirmRemoval` does not lift this guard, because it runs before any folder is judged. To
  recover, remove the library in **Settings** › **Libraries** (which forgets its catalog rows
  only, never files) and add the folder again. There is no way to confirm "this library really is
  empty".
- **Change detection is size plus mtime.** A file rewritten with the same size and modification
  time is not re-hashed. An `unreadable` file is retried on every scan, except a corrupt or
  encrypted archive after 3 content failures in a row: that one is retried only once its size or
  mtime changes (so repairing it in place with the same mtime is not noticed). A file failing
  for infrastructure reasons 10 times in a row (a timeout: 3 times) is then tried only on every
  10th scan, so after a NAS recovers it can take up to 10 scans to be catalogued, unless the
  player presses "Try again" on Health (`retryUnreadable`), which hashes every one of them once.
- **Scans can be cancelled, not paused.** A later scan skips files that did not change, so it
  picks up close to where a cancelled one stopped.
- **Directory listing has no timeout.** A hung network mount can stall the walk. Hashing has a
  per-file timeout (10 minutes); listing folders and reading file details do not.
- **Renames look like a removal plus a new file.** The one exception is a case-only or NFC/NFD
  rename of a top-level system folder on a volume where both spellings are one directory, which is
  recognized and keeps its hashes. Renaming a file, a subfolder, or a folder to a different name is
  not tracked (re-link can rename a picture a renamed game left behind). A folder is relocated only
  when it has catalog rows, so a user mapping of a folder with none (an empty folder, or one with no
  file of its system's extensions yet) stays under the old spelling after a case or NFC rename, and
  the folder has to be mapped again.
- **A game stored as a folder is not one game, until identify joins its files.** The scanner
  catalogues files whose extension the system lists, so a folder that is one game is either
  several games or none. A DOS game's `.exe` and `.bat` files, Quake's `pak0.pak` and
  `pak1.pak`, and a Wii U title's `.rpx` and `.tmd` are each catalogued as a game; a PS3
  `Game.ps3/` folder (its files carry none of the `ps3` extensions) is never catalogued, so it
  stays 0 games even after identify. When a DAT lists each of a game's files under one DAT game
  name, matching joins them like any other multi-file game (see
  [Identification](architecture.md#identification)); without a matching DAT, each stays its own
  filename game.
- **A disc image is several files.** A `.cue` and its `.bin` tracks, or the discs of an
  `.m3u` playlist, are catalogued as separate files of one game (their bytes add up, and a
  deleted track is not tied to its cue sheet). Identify joins them: files matching different
  roms of one DAT game share it, and the group phase joins a playlist itself to the game of the
  discs it lists (see [Identification](architecture.md#identification)).
- **Revisions of one game collapse into one game, until identify tells them apart.** A filename
  game is unique by system, title and region, so `(Rev 1)` and the original share a row until a
  DAT identifies them: an identified game is keyed by its DAT game name instead (see
  [decisions.md](decisions.md), ADR 39), so two revisions a DAT lists separately become two
  games.
- **Case-insensitive search is ASCII-only.** It uses SQLite `LIKE`, so `é` does not match `É`.
- **Search and sort use the stored title, not the cleaned one.** `listGames` searches and sorts
  the stored title: the collection-clean `display_title` when the directory is numbered, else the
  title parsed from the file name. The wall then shows it through the display cleaner (control
  and invisible characters removed), which search and sort do not apply, so a title holding
  such characters sorts by them and a search must match them. Indexes are only recognized in
  directories of at least 20 names; a smaller numbered directory keeps its numbers. A leading
  four-digit run from 1900 to 2099 is taken for a year, never an index ("1941 - Counter
  Attack"), so a collection numbered into that range keeps those numbers, and a run outside it
  ("2100 ...") can count towards a numbering.
- **Symlinks are reported, not followed.** A symlinked system folder is listed in the scan
  report and not walked. A symlinked media root keeps its catalogued art `present` forever (the
  cover protocol still refuses to follow it).
- **Symlinked subfolders are skipped, and their rows go missing.** Only a top-level system
  folder that is a symlink keeps its catalogued rows (seen, not verified). A symlink deeper
  inside a system folder is ignored like any other symlink, so if a catalogued subfolder is
  replaced by a symlink to the same files, the next scan marks those files missing, and the
  symlink is not listed in the scan report.
- **A title made only of a tag can show as "Untitled game".** The title parser strips bracketed
  tags without nesting, so a file name holding an ANSI escape such as `ESC[31mANSI ESC[0m
(USA).sfc` loses everything from the first `[` to the `)`. What is left is a lone escape
  character, which the display cleaner removes, so the game is shown as "Untitled game".
- **One process at a time.** The one-scan guard and the operation lock live in the process. The
  single-instance lock is per data folder, so two copies of the app can run only with different
  `ROMPEROOM_DATA_DIR`s. Even then, a library that another process is scanning shows as
  "interrupted".
- **A catalog row for a file that is now ignored can linger.** A catalog written before ignored
  extensions existed can hold a file such as `genesis/README.md` as a ROM. If that folder has
  another ROM, the next scan marks the old row missing. If the README is the folder's only
  file, the folder has no ROMs left and is reported suspect, so its row stays `present` until
  the removal is confirmed with `confirmRemoval`.

### Libraries

- **Removing a library forgets its Tidy up history.** What it set aside stays in its
  `.romperoom-quarantine` folder, but Undo, put back and Delete forever no longer offer it; move
  the files back by hand. Adding the folder again starts afresh.
- **Add a library… can stall on a hung share.** `addLibrary` is synchronous in the main process
  and reads every existing library's folder (`placeOf`), so Romperoom stops responding until
  the drive answers or the system gives up on it.
- **One stuck probe thread per hung library.** The Libraries tab's folder probe gives up after 3
  seconds, but a read of a share that hangs keeps one thread of the main process's file-system
  pool until the share answers. Reads of one folder are shared while one is going, so that is at
  most one stuck thread per hung library.
- **A symlinked library reads "Can't reach its folder".** A library whose folder was moved and
  replaced by a symbolic link at its old path is probed with `lstat`, which sees the link, so it
  is listed as not reachable; its scans still read through the link.
- **An identify run can be overtaken by a removal.** A run over every library fixes its list
  when it starts and locks each library only when it reaches it, so a library removed in between
  is not refused; its part fails with a foreign-key error, and the other libraries' parts run as
  usual. Nothing is lost.

### Tidy up, Standardise and re-link

- **A duplicate set's picture is the suggested copy's,** even after the player chooses to keep a
  different copy.
- **A set-aside file's age is its run's start.** Delete forever's "Older than 30 days" and "Older
  than 90 days" count from when the run that set a file aside started, not from each file's move.
- **"Game removed" rarely shows.** A scan unlinks a picture whose game's files are gone, so
  Artwork lists it under "No game in your library"
  ([roadmap](roadmap.md#milestone-3-organize-tidy-up-done)).
- **Standardise renames only identified games, inside one library.** A frontend's own database
  and a game list kept outside the library are not updated, and saves on a card keep their old
  names, so the next card sync takes a renamed game's card save for a save of the old name. On a
  volume whose folders change inode on rename or remount (an SMB share without stable file IDs),
  a folder move can't be undone or finished. A playlist matched by its new name before a run was
  undone keeps that match until Identify games runs again. The full list is in
  [decision 43](decisions.md#43-standardise-renames-folders-and-games-and-edits-two-kinds-of-files-other-programs-own).
- **Re-link offers only one clear match.** A picture two games could take, a misspelling, an
  index prefix, a scraper-style name or a game on several discs recognised disc by disc is never
  offered, and an absolute media path in a game list is never changed. After Recovery rolls back
  a re-link's game lists, or an Undo keeps a list changed since the run, the list and the
  pictures' names disagree until the run is undone or the frontend rescans. A re-link of game
  list entries only that crashed before its journal was recorded stays "Interrupted" in History
  ([decision 44](decisions.md#44-re-link-renames-a-leftover-picture-after-the-one-game-that-clearly-matches-it)).
- **Standardise and re-link share one run table.** A re-link waiting in Recovery refuses a
  standardise run of that library, and the other way round, and after a reload Tidy up shows
  only the latest result of either kind (the other is in History).

### Across libraries

- **The list is as each library's last scan saw it.** A file changed in place since that scan
  (the same name, different bytes) is still listed as a copy; scan both libraries again before
  removing a copy by hand.
- **Two mounts of one network share are two libraries.** Their device numbers differ, so the
  overlap check cannot see it and every file in the share reads as held twice.
- **The look can freeze the window on a hung share.** After the folder probe it reads each
  candidate copy's device and inode synchronously in the main process, as Duplicates does.
- **A copy renamed while the tab looks reads as gone** (a Tidy up, Standardise or re-link run
  renaming files at that moment), and its set may drop out until **Look again**.
- **A pair hard-linked inside one library is one file,** so a copy of it in another library is
  not listed.
- **It only reports.** Setting copies aside across libraries is planned
  ([decision 46](decisions.md#46-copies-across-libraries-are-reported-never-tidied)).

### Copying to a card

- **Which copy goes, when a game file differs between libraries, is decided by little.** Copies
  of one name share their region, revision and tags, so a content match with a game database
  decides, then the shorter folder name, then the copy found first. Without a game database a
  rescan that finds a file anew can change which copy goes
  ([decision 48](decisions.md#48-copies-that-differ-send-tidys-keeper-and-say-so)).
- **Some copies that differ still leave the game out as a name clash:** two different copies in
  one library (two of its folders), a game holding a playlist or a `.cue`, `.gdi`, `.ccd` or
  `.toc` sheet, and a copy with no checksum or of 0 bytes. (Two different games of one name are
  not picked from either: each keeps its own name with a suffix, as before.)
- **Art checksums fill as art is copied.** The card writer keeps the checksum of a picture it
  reads whole, and Tidy up's leftover-artwork check its own; the scan never reads art, so a
  picture never copied or compared carries none and the writer reads it the first time.
- **Make it fit starts with the biggest games,** so it can leave out a big game when only a
  little is over; **Keep this one** asks for another choice. A list is for one card and one set
  of choices, and is never saved.
- **Saved packages stay in the page's localStorage**
  ([decision 49](decisions.md#49-saved-packages-live-in-the-pages-storage)), so they are lost
  with the data folder and do not travel to another computer. Packages of a device profile that
  no longer exists are kept but never listed: the card counts them and removes them only all
  together (**Remove them**), and they still count toward the 400 packages read in all
  (`MAX_STORED`). Every change reads the list again before writing, but two windows could still
  race (the app opens one). The **Loaded** mark is not stored: it lasts the wizard's visit and
  clears on a device change, on deleting that package and on **Start over**.

### Card sync

- **A card copied with a disk tool keeps the original's id,** so Romperoom takes the copy for
  the card it was copied from, and syncing both can show more saves as changed on both sides
  than expected. Nothing is deleted either way.

### The desktop app

- **A sentence a screen says when it opens is said on every visit.** Since each announcement is
  a new node of the live region, "Still copying to `<card>`" and identify's "still running" are
  read out each time their screen opens, not only the first time.
- **Pressing Cancel while the first of several libraries finishes** stops the next library from
  starting, but the announcement says "Scan finished", not "Scan cancelled".
- **End jumps to the last game loaded so far, not the last game.** The wall loads games a page at
  a time as it scrolls, so on a large library End (and PageDown near the end) stops at the last
  loaded tile. Reaching it loads the next page, so pressing End again goes further.
- **The interface is English only**, and the plural helper is English-specific.
- **A card writer refusal can name full paths.** Many of the writer's refusals start with the
  target's real path (for an export folder on a network drive: "`<path>` is on a network drive
  (...); choose the folder again to confirm it"), and the library and BIOS overlap refusals also
  name that folder's path ("... is inside `<library>`, which is never written to"). Others name
  none ("the plan: ...", "the target cannot be opened (...)"). The Done screen lists them under
  its technical details, so that one place shows the page a path. Showing folder names instead
  (as `reportView` already does for the report's root) is an open follow-up for the owner.
- **Drag and drop is tested synthetically, not by hand.** The page refuses `dragover` and `drop`
  (`lib/page-guards.ts`), and `will-navigate` blocks the navigation Chromium would otherwise start.
  `e2e/resilience.spec.ts` checks both with a synthetic `DragEvent` carrying a `File` and with a
  CDP `Input.dispatchDragEvent` of a real file. A manual check before each release is still
  advisable: drag a `.html` file and a ROM from Finder (Explorer on Windows) onto the wizard and
  the wall, and check that the window stays on the app, shows the not-allowed cursor and logs
  nothing.
- **WebRTC is blocked by the IP policy and a dead proxy, not by a switch.** On Electron 44 the
  process-wide `--force-webrtc-ip-handling-policy` switch was measured inert: with only the
  switch, the e2e STUN probe still received UDP packets. Chromium has no switch that turns
  WebRTC off. The per-webContents `setWebRTCIPHandlingPolicy` blocks UDP and the session's dead
  proxy blocks TCP (see [security.md](security.md#network-isolation)). The switch stays as
  a harmless extra layer.
- **Chromium reaches nothing on the network, the main process's `net` included.** The dead proxy is
  on every session and the host resolver rules cover the whole process, so Electron `net` requests
  from the main process fail too (measured: `ERR_PROXY_CONNECTION_FAILED`). The only network code is
  the game database download and cover art, which use `node:https` in the main process instead,
  outside Chromium's stack, behind its own exact URL allowlist, address check, pinned TLS roots,
  caps and request log
  ([ADR 40](decisions.md#40-game-databases-can-be-downloaded-from-one-pinned-source-only-when-asked),
  [security.md](security.md#main-process-requests)). Cover art followed that pattern; a later
  feature that needs the network (updates) should too: extend `main/dat-download/`'s allowlist and
  transport, add its module to the static import list, and leave the renderer's guards unchanged.
- **Page-named hostnames used to be looked up.** Before the host resolver rules, the dead proxy
  and the CSP stopped every connection, but dns-prefetch and TURN server names still reached the
  system resolver (measured in a net log, with the machine's search domain appended to a
  single-label name). The rules now fail every lookup inside Chromium; the net log check in
  `e2e/resilience.spec.ts` guards it.
- **Cover art may also be video or PDF.** The media protocol serves catalogued `.mp4`, `.webm`
  and `.pdf` files with their real MIME type, under the response's
  `default-src 'none'; sandbox` CSP. Nothing in Milestone 1 displays them (the page's CSP allows
  `romperoom-media:` only for images), but a later screen that does must keep that sandbox.

### Operations and hashing

- **A stopped tidy waits until it is settled.** Cancel leaves the run's journal `running`; until it
  is finished, undone or (a Tidy up run) its rest discarded, Health counts it and the next start
  offers it again. Discard the rest refuses while a file of the run is not where the run left it
  ([decision 33](decisions.md#33-a-stopped-tidy-is-finished-or-undone-not-undone-in-part)).
- **Artwork shows the first 200 leftover pictures** of a library, or of the cause shown; setting
  them aside and looking again shows the next ones.
- **A journal knows its library by device, inode, real path and top-level names**
  ([decision 29](decisions.md#29-a-journal-knows-its-library-without-writing-to-it)). A remount
  with renamed top-level folders blocks recovery. Journals written before this check have no
  identity; they are only refused when the library folder is empty.
- **Library health counts duplicates as duplicate cleanup does**: within one library, by whole-file
  hash, one console only. A copy in another library is listed under Across libraries, never counted
  here or moved. Hard links and disc sets are only found on disk, so cleanup may offer fewer than
  health counts.
- **A tidy preview goes stale on any catalog write**, including a deploy record or a scan of a
  different library: the user previews again.
- **A restored file counts as present but unconfirmed** until the next scan re-hashes it, so it
  is not offered as a duplicate again before then.
- **The purge checks a file, then deletes it.** A file replaced in the single system call between
  its final hash and `unlink` is deleted ([security.md](security.md#emptying-the-quarantine)).
- **Operations leave empty folders behind** and do not fsync the parent folder after a rename. A
  move across volumes copies only the file's contents and mtime, not extended attributes or
  macOS resource forks.
