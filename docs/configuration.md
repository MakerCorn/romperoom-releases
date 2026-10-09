# Configuration

Romperoom has no settings file. A player changes the theme and light or dark mode in the
Settings drawer (which also lists the libraries and the game databases), and everything else
works without configuration. This page lists what an
engineer or a scripted install can set.

## Contents

- [Data folder](#data-folder)
- [Environment variables](#environment-variables)
- [Other names that start with ROMPEROOM\_](#other-names-that-start-with-romperoom_)
- [Command-line switches](#command-line-switches)
- [Electron and toolchain variables](#electron-and-toolchain-variables)
- [Packaging and release variables](#packaging-and-release-variables)
- [Chromium switches](#chromium-switches)
- [Saved settings](#saved-settings)

## Data folder

The catalog (`catalog.sqlite`) and Electron's own user data (Chromium caches and local storage)
share one folder. By default that is Electron's `userData` path for the app, named after the
`productName` in `apps/desktop/package.json`, `Romperoom` (Electron prefers `productName` over
`name`). Installed and unpackaged builds use the same folder:

| OS      | Default data folder                           | Verified                                 |
| ------- | --------------------------------------------- | ---------------------------------------- |
| macOS   | `~/Library/Application Support/Romperoom`     | Yes, from the packaged and dev builds    |
| Windows | `%APPDATA%\Romperoom`                         | No: Electron's default (CI runners only) |
| Linux   | `$XDG_CONFIG_HOME/Romperoom` or `~/.config/…` | No: Electron's default                   |

Builds before 0.1.0 had no `productName` and used `@romperoom/desktop` instead; to keep a
catalogue from one of them, quit the app and rename that folder to `Romperoom`. Uninstalling on
Windows keeps the folder. To start over, quit the app and delete the folder. Your ROM library is
not stored there and is not touched.

Game databases add three entries to the same folder, never to a library:

| Name                | What it is                                                                                      |
| ------------------- | ----------------------------------------------------------------------------------------------- |
| `libretro-pin.json` | The listing "Check for updates" last stored. The newer of it and the shipped listing is used.   |
| `network-log.json`  | The newest 500 request attempts (Network activity). Never headers, bodies or library data.      |
| `downloads/`        | Temporary: each file is downloaded and verified here, then imported. Emptied at every start.    |

Saving a ScreenScraper account (Settings › ScreenScraper) adds `screenscraper-account.json`: the
account encrypted by the system keychain (Electron's `safeStorage`), mode 0600, removed by
**Forget the account**. Nothing is saved on a computer without a keychain.

Get cover art adds `art-listings/`: each console's last libretro-thumbnails listing (names, sizes,
git SHAs and its tree SHA), or GitHub's "not found" for it, one file per repository, reused for 24
hours so a second review asks GitHub nothing. Failures that may pass (offline, rate limiting, a
server error, a malformed answer) are never kept. The pictures themselves go into the library's
own `.romperoom/media` folder.

## Environment variables

Every variable whose name starts with `ROMPEROOM_` and that the code reads is listed here. A
docs test (`apps/desktop/test/docs-guards.test.ts`) fails if the code reads one this page does
not list, or if this page lists one the code never mentions.

| Variable                                 | Read by                                      | Packaged build | What it does                                                                                                                                                                                                 |
| ---------------------------------------- | -------------------------------------------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `ROMPEROOM_DATA_DIR`                     | `src/main/lifecycle.ts`                      | Honoured       | Absolute folder for the catalog and Electron user data. It also scopes the single-instance lock.                                                                                                             |
| `ROMPEROOM_TEST_PICK_FOLDER`             | `src/main/security.ts`                       | Ignored        | The folder dialog returns this path instead of opening.                                                                                                                                                      |
| `ROMPEROOM_TEST_SCAN_ONLY`               | `src/main/security.ts`                       | Ignored        | Comma-separated top-level folders. Every scan judges only these.                                                                                                                                             |
| `ROMPEROOM_SMOKE_EXIT_MS`                | `src/main/security.ts`                       | Ignored        | Runs the self-check smoke this many ms after load, then quits.                                                                                                                                               |
| `ROMPEROOM_TEST_UNGUARDED_WEBRTC`        | `src/main/security.ts`                       | Ignored        | `1` turns off the WebRTC and DNS guards. Only for the e2e positive controls.                                                                                                                                 |
| `ROMPEROOM_TEST_LARGER_THAN_SCREEN`      | `src/main/security.ts`                       | Ignored        | `1` lets macOS size the window past the screen. Set by the e2e harness.                                                                                                                                      |
| `ROMPEROOM_TEST_VOLUME`                  | `src/main/security.ts`                       | Ignored        | An absolute folder the engine lists as the only volume, so a deploy can run without a card.                                                                                                                  |
| `ROMPEROOM_TEST_EXPORT_FOLDER`           | `src/main/security.ts`                       | Ignored        | The "Export to a folder" dialog returns this absolute folder instead of opening.                                                                                                                             |
| `ROMPEROOM_TEST_FOLDER_NETWORK`          | `src/main/security.ts`                       | Ignored        | `network` or `unknown`: every export folder's network check gives that answer, so the e2e suite can drive the confirmation.                                                                                  |
| `ROMPEROOM_TEST_PICK_FILE`               | `src/main/security.ts`                       | Ignored        | The game database (DAT) file dialog returns this absolute file instead of opening.                                                                                                                           |
| `ROMPEROOM_TEST_DEPLOY_DELAY_MS`         | `src/main/security.ts`                       | Ignored        | Each chunk a deploy writes waits this long (1 to 5000 ms), so a test can cancel mid-copy.                                                                                                                    |
| `ROMPEROOM_TEST_VOLUME_BYTES`            | `src/main/security.ts`                       | Ignored        | The test volume reports at most this many bytes (from 4096), to fill a small card.                                                                                                                           |
| `ROMPEROOM_TEST_TIDY_DELAY_MS`           | `src/main/security.ts`                       | Ignored        | Each file a tidy job moves waits this long first (1 to 5000 ms), so a test can cancel or crash mid-run.                                                                                                      |
| `ROMPEROOM_TEST_DAT_FIXTURES`            | `src/main/security.ts`                       | Ignored        | An absolute folder the game database download and cover art serve from instead of GitHub: `listing.json`, `metadat/`, `api/`, `art/trees/`, `art/raw/` and an optional `hold.json`.                          |
| `ROMPEROOM_TEST_OPEN_EXTERNAL`           | `src/main/security.ts`                       | Ignored        | An absolute file the host appends each URL it would open in the browser to, instead of opening it.                                                                                                           |
| `ROMPEROOM_TEST_FAKE_KEYCHAIN`           | `src/main/security.ts`                       | Ignored        | `1` keeps the ScreenScraper account with a stand-in for the system keychain, so the e2e suite never asks the real one.                                                                                       |
| `ROMPEROOM_SCREENSCRAPER_DEV`            | `src/main/scraper/client.ts`                 | Ignored        | ScreenScraper's developer details for Romperoom (JSON: `devid`, `devpassword`, `softname`). Secrets issued to the maintainer; never committed or logged. Without them ScreenScraper reads "can't use yet".   |
| `ROMPEROOM_REAL_CARD`                    | `packages/engine/test/real-card.test.ts`     | Not in the app | The mount of a FAT32 disk image. Turns on the real-card tests (macOS).                                                                                                                                       |
| `ROMPEROOM_REAL_CARD_DEVICE`             | `packages/engine/test/real-card.test.ts`     | Not in the app | The image's whole-disk device (`/dev/diskN`): the only one the test detaches.                                                                                                                                |
| `ROMPEROOM_REAL_CARD_IMAGE`              | `packages/engine/test/real-card.test.ts`     | Not in the app | The image file, attached again after the forced-detach test.                                                                                                                                                 |
| `ROMPEROOM_REAL_CARD_RECORD`             | `packages/engine/test/real-card.test.ts`     | Not in the app | A file the test writes the re-attached device to, so the caller detaches the right one.                                                                                                                      |
| `ROMPEROOM_REAL_CARD_EVIDENCE`           | `packages/engine/test/real-card.test.ts`     | Not in the app | A JSON file for the run's measurements.                                                                                                                                                                      |
| `ROMPEROOM_FIXTURE_SIMULATE_WIN32`       | `packages/engine/test/fixture.ts`            | Not in the app | `1` builds the test fixture's Windows shape on any OS.                                                                                                                                                       |
| `ROMPEROOM_TEST_NO_SYMLINKS`             | `packages/engine/test/fs-caps.ts`            | Not in the app | `1` makes the engine tests act as if symlinks cannot be made (Windows without the privilege).                                                                                                                |
| `ROMPEROOM_PERF_FULL`                    | `packages/engine/test/identify-perf.test.ts` | Not in the app | `1` runs the full-size identify perf test; `npm run test:perf -w @romperoom/engine`                                                                                                                          |
| `ROMPEROOM_TEST_SURVEY_STOP_AFTER_FILES` | `scripts/nas-survey.mjs`                     | Not in the app | Makes the survey run out of time after this many files.                                                                                                                                                      |
| `ROMPEROOM_TEST_LIVE_TOUCH`              | `scripts/live-identify.mjs`                  | Not in the app | Test only: a library-relative file the live identify run appends one byte to before its second fingerprint, so a test can see a change reported. Ignored unless the library is under the system temp folder. |
| `ROMPEROOM_CAPTURE_OUT`                  | `e2e/capture/capture.screens.ts`             | Not in the app | Where the screenshot capture writes. Set by `scripts/capture-screenshots.mjs`.                                                                                                                               |
| `ROMPEROOM_CAPTURE_COMMIT`               | `e2e/capture/capture.screens.ts`             | Not in the app | The app commit the capture records in the manifest. Set by the same script.                                                                                                                                  |

Details:

- **`ROMPEROOM_DATA_DIR`** is honoured in packaged builds on purpose. It allows portable or
  scripted installs and separate profiles, and whoever controls the app's environment already
  controls the process. An empty value means unset. A relative path stops startup with
  "ROMPEROOM_DATA_DIR must be an absolute path". Two copies of the app can run at once only with
  different data folders.
- **`ROMPEROOM_TEST_VOLUME`** replaces the volume listing with one volume: that folder,
  removable, file system "other". A relative path stops startup with "ROMPEROOM_TEST_VOLUME must
  be an absolute path". An empty value means unset. The real-card variables are described in
  [testing.md](testing.md#real-card-run).
- **`ROMPEROOM_TEST_EXPORT_FOLDER`** stands in for the export dialog. The page still receives
  only a token and the folder's name ([security.md](security.md#deploying-to-a-card)). A
  relative path stops startup with "ROMPEROOM_TEST_EXPORT_FOLDER must be an absolute path".
- **`ROMPEROOM_TEST_FOLDER_NETWORK`** replaces the export folder's network check for the plan and
  the writer alike. Any value other than `network` or `unknown` stops startup with
  "ROMPEROOM_TEST_FOLDER_NETWORK must be network or unknown".
- **`ROMPEROOM_TEST_PICK_FILE`** stands in for the game database (DAT) file dialog. The page
  never names the file either way. A relative path makes the import fail with
  "ROMPEROOM_TEST_PICK_FILE must be an absolute path".
- **`ROMPEROOM_TEST_DAT_FIXTURES`** swaps the game database transport for a test transport
  (`src/main/dat-download/fixture-transport.ts`, loaded only when this is set and the app is not
  packaged). It keeps the real URL allowlist, the size and git SHA checks and the real request
  log, and makes no request. The folder holds `listing.json` (the listing the plan uses, in the
  shipped listing's shape), `metadat/<no-intro|redump>/<file name>` (the files raw URLs answer
  with), `api/branch.json`, `api/no-intro.json` and `api/redump.json` (the answers "Check for
  updates" gets) and, optionally, `hold.json`: a JSON list of file names held, writing nothing,
  until the job is stopped (so a test can stop mid-job without a delay). A missing file answers
  like a 404. It also stands in for GitHub's cover art: `art/trees/<repo>.json` answers a
  repository's whole-tree listing and `art/trees/<repo>/<Named_X>.json` one kind's (a folder per
  repository, so no file name holds the URL's `:`), `art/raw/<repo>/<Named_X>/<name>` holds the
  pictures (names decoded, also held by `hold.json`), and an optional `api/rate-limit.json`
  (`{ "limit": 60, "remaining": 41, "reset": <epoch seconds> }`) is the budget every API answer
  reports. Repositories and branches pass the same allowlist as real requests. A relative path
  stops startup with "ROMPEROOM_TEST_DAT_FIXTURES must be an absolute path".
- **`ROMPEROOM_TEST_OPEN_EXTERNAL`** stands in for the browser: "Open download page", "Licence"
  and "Source" append the exact URL and a newline to that file. A relative path stops startup
  with "ROMPEROOM_TEST_OPEN_EXTERNAL must be an absolute path".
- **`ROMPEROOM_TEST_DEPLOY_DELAY_MS`**, **`ROMPEROOM_TEST_TIDY_DELAY_MS`** and
  **`ROMPEROOM_TEST_VOLUME_BYTES`** are ignored unless they are whole numbers in range. The byte cap
  applies only together with `ROMPEROOM_TEST_VOLUME`.
- **`ROMPEROOM_TEST_SCAN_ONLY`** is meant for live runs against a large share: scan a few
  consoles instead of all of them. It replaces any `onlyFolders` the page passed. Other scan
  options, such as `confirmRemoval`, are kept. Blank entries are dropped, and an empty list means
  unset.
- **`ROMPEROOM_SMOKE_EXIT_MS`** drives the real preload API from the page: add a temp library,
  scan it with progress, list games and make a rejected call. It also checks the page's single
  CSP meta. Then it runs a local-file probe, and each of these must report `blocked`:
  - `fetch` and XHR of a temp secret file and of `file:///etc/hosts`;
  - a `<script src=file://…>`;
  - a `..%2f` traversal through `app://`;
  - an `<iframe>` of a local file.

  It prints `ROMPEROOM_SMOKE {json}` and quits. The exit code is 1 if a check failed, a probe
  got through, or the window logged a console error before the probe. `npm run smoke -w
@romperoom/desktop` wraps it. The value must be a positive whole number of ms, at most 600000.

- **`ROMPEROOM_TEST_UNGUARDED_WEBRTC=1`** drops the per-webContents IP policy, the dead proxy,
  the WebRTC switch and the host resolver rules. The e2e suite uses it to prove its probes can
  see a leak. Any other value leaves the guards on.

The end-to-end harness (`e2e/support.ts`) removes every test seam and `ELECTRON_RUN_AS_NODE`
from the app's environment before each launch. It then sets only the seams that test asked for.

## Other names that start with ROMPEROOM\_

These look like variables but are not:

| Name                | What it is                                                                                |
| ------------------- | ----------------------------------------------------------------------------------------- |
| `ROMPEROOM_SMOKE`   | The prefix of the smoke report line on stdout (`ROMPEROOM_SMOKE {json}`), not a variable. |
| `__ROMPEROOM_CSP__` | The placeholder in `src/renderer/index.html` that the build replaces with the CSP.        |

## Command-line switches

| Switch         | What it does                                                                                                                                                                                                               |
| -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `--self-check` | Runs a health check of what packaging can break, without a window, prints one JSON line `{ "ok": …, "packaged": …, "checks": [...] }` and exits 0 if every check passed or was skipped, 1 otherwise. Works in every build. |

`--self-check` runs, in order:

- `catalog`: creates a fresh catalogue with the shipped SQLite in a temporary sub-folder of the
  data folder, then removes it. The player's catalogue is never opened.
- `hash-worker`: hashes the bundled `self-check.bin` (1 KB) through the worker pool, so the
  unpacked worker and its dependencies load.
- `identify`: writes a one-game DAT for `self-check.bin`, imports it, scans a temporary library
  holding that file (in a temporary sub-folder of the data folder, removed after) and identifies
  it: the check passes only when the run completes with exactly one game identified by hash.
- `app-protocol`: loads `index.html` through `app://` from inside `app.asar`, checks the CSP it
  carries, and checks that a `..` escape is refused with 403.
- `app-location`: the page is served from `app.asar`, not from a dev server.
- `fuses`: the fuse wire read from the running executable matches `src/main/fuse-policy.json`.
- `dev-tools`: the window would be created with DevTools off and no DevTools menu entry.

Checks that only mean something packaged (`app-location`, `fuses`) are skipped in a dev build.
Chromium's own state goes to a throwaway profile in the system temp folder, removed on exit. On
macOS run it as `Romperoom.app/Contents/MacOS/Romperoom --self-check`; on Windows,
`Romperoom.exe --self-check` (a GUI program: its output shows when you capture or redirect it,
as `e2e:packaged` does, not in a console window).

### Switches a packaged build refuses

A packaged build exits with code 2, before it opens anything, when its command line carries
any of these. It prints `Romperoom will not start with these command-line switches:` and the
switch names to standard error, never their values. Development builds accept them all (the
end-to-end tests need `--inspect` and `--remote-debugging-port`).

- `--remote-debugging-port`, `--remote-debugging-pipe`, `--remote-debugging-address` (any
  `--remote-debugging-*`) and `--remote-allow-origins`
- `--inspect`, `--inspect-brk`, `--inspect-port`, `--inspect-wait` (any `--inspect*`) and
  `--debug-port`
- `--js-flags`, `--no-sandbox`, `--disable-web-security`
- `--host-rules`, `--host-resolver-rules`, `--proxy-server`, `--proxy-pac-url`,
  `--no-proxy-server`
- `--user-data-dir`

The `-name` and (Windows) `/name` spellings and any letter case count too. Why each is on the
list, and what the list does not cover: [security.md](security.md#launch-switches).

## Electron and toolchain variables

| Variable                      | Effect                                                                                                                                                                              |
| ----------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ELECTRON_RUN_AS_NODE`        | If set (some editors and agent hosts export it), Electron runs as plain Node and the app fails to start. Unset it: `env -u ELECTRON_RUN_AS_NODE npm run dev -w @romperoom/desktop`. |
| `ELECTRON_RENDERER_URL`       | Set by `electron-vite dev`: the dev server URL. It must be loopback `http`. Ignored when packaged.                                                                                  |
| `electron_config_cache`       | Where Electron's installer caches the binary download. CI points it at a cached folder (see [testing.md](testing.md#ci-jobs)). `ELECTRON_CACHE` is ignored.                         |
| `ELECTRON_OVERRIDE_DIST_PATH` | Use an Electron binary already on disk instead of downloading one.                                                                                                                  |

## Packaging and release variables

Read by the packaging scripts and the release workflow, never by the app
([release.md](release.md#secrets-and-variables)):

| Variable                                                   | Read by                                 | What it does                                                                                               |
| ---------------------------------------------------------- | --------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| `PACKAGED_APP_PATH`                                        | `e2e/packaged.spec.ts`                  | The app `e2e:packaged` runs (`Romperoom.app`, or the folder holding `Romperoom.exe`). Default: `release/`. |
| `CSC_LINK`, `CSC_KEY_PASSWORD`                             | `scripts/package.mjs`, electron-builder | macOS signing certificate. Set: a signed build that fails rather than ship unsigned.                       |
| `WIN_CSC_LINK`, `WIN_CSC_KEY_PASSWORD`                     | the same                                | Windows signing certificate (`CSC_LINK` is the fallback).                                                  |
| `APPLE_ID`, `APPLE_APP_SPECIFIC_PASSWORD`, `APPLE_TEAM_ID` | electron-builder                        | Notarization of a signed macOS build.                                                                      |
| `CSC_IDENTITY_AUTO_DISCOVERY`                              | electron-builder                        | `scripts/package.mjs` sets it to `false` for unsigned builds, so no keychain identity is used.             |

## Chromium switches

The app sets these itself, before `ready`:

| Switch                            | Value                                                   | Why                                                                                                                                                                                                                                                                                                 |
| --------------------------------- | ------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `host-resolver-rules`             | `MAP * ~NOTFOUND, EXCLUDE localhost, EXCLUDE 127.0.0.1` | No hostname is ever looked up inside Chromium. In dev, a `[::1]` dev server adds `EXCLUDE ::1`. The main process's game database requests resolve their two GitHub names through Node instead ([security.md](security.md#main-process-requests)). See [security.md](security.md#network-isolation). |
| `force-webrtc-ip-handling-policy` | `disable_non_proxied_udp`                               | Measured to have no effect on Electron 44. It stays as an extra layer; the per-webContents policy is the real block.                                                                                                                                                                                |

Tests pass two more on the command line: `--force-device-scale-factor=1` (screenshot capture) and
`--log-net-log=<file>` (the DNS check in `e2e/resilience.spec.ts`).

## Saved settings

The page saves three keys in localStorage:

| Key                            | Value                                                                                 | Default                                   |
| ------------------------------ | ------------------------------------------------------------------------------------- | ----------------------------------------- |
| `romperoom.settings.v1`        | JSON `{ "theme": …, "mode": … }`                                                      | `console-shelf`, `system`                 |
| `romperoom.deploy.v1`          | JSON: the card wizard's last device, consoles, region order and options; never a card | Every console of the device, box art only |
| `romperoom.deploy.packages.v1` | JSON: the card wizard's saved packages, each a name, a device and its choices         | None saved                                |

- `theme` is one of `console-shelf`, `crt-neon` or `clean-modern`.
- `mode` is one of `light`, `dark` or `system` ("Match my computer").
- A missing or unreadable value falls back to the defaults.
- The page reads the value synchronously, so `<html>` has its `data-theme` and `data-mode`
  before the first paint (`src/renderer/lib/settings.ts`).
- `romperoom.deploy.v1` (`src/renderer/deploy/choices.ts`) is read when the wizard opens; a
  missing or unreadable value, or a field out of range, falls back to the defaults.
- `romperoom.deploy.packages.v1` (`src/renderer/deploy/packages.ts`) is
  `{ "version": 1, "packages": [ … ] }`, each package its `name` beside the fields of
  `romperoom.deploy.v1`; never a card. It is read when the wizard's What to copy step opens, and
  again before each change. A value that is not a version-1 list, or is over 1,000,000
  characters, lists nothing (the step says so) and is replaced by the next change; one bad
  package is left out alone, and one bad field falls back alone. Each device keeps at most 20
  packages, named in up to 40 characters, unique ignoring case; at most 400 are read in all,
  counting the hidden packages of a device profile that no longer exists (the step counts those
  and can remove them together). It is independent of `romperoom.deploy.v1`.
- Three sessionStorage keys only remember, for the window's life, which Tidy up, standardise and
  re-link result was already shown or closed (`romperoom.tidy.seen`,
  `romperoom.standardise.dismissed`, `romperoom.relink.dismissed`).
- Choices made per library are kept in the catalog, not the page: the BIOS folder
  (`source_root.bios_folder`) and the device Standardise names folders by
  (`standardise_profile`).
