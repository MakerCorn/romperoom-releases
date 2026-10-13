# Security

This page is for reviewers. It covers what Romperoom protects, how the Electron host is locked
down, and what is still open. Each claim names the code that enforces it and the test that pins
it. Where something has not been measured, the page says so.

The short version:

- Scans are read-only.
- The window cannot reach the network.
- Romperoom itself goes online only when you press Download for me, Check game database updates
  or Get cover art (GitHub), Look up N games (ScreenScraper's API host), or Check for a new version
  and Install in Settings › Updates (GitHub's API, `github.com` and GitHub's two asset hosts), or,
  if you turn it on, once as it opens to check for a new version.
- The page has no Node.js and no file access.
- Every call from the page to the host goes through one checked, typed list of channels.
- Only Fix up moves games and artwork in a library: into its set-aside folder after a preview, or,
  for Standardise and re-link, to new names after a review, keeping each edited game list's
  original. Deleting set-aside files forever needs typed words, and is the only deletion of the
  player's files. Cover art adds pictures only under `.romperoom/media`, never replacing a file.
  Sync a card adds the games and saves the player approves (console folders, `.romperoom/saves`),
  replacing a save only after a backup, and writes on a card only its id file and saves.

## Contents

- [Threat model](#threat-model)
- [Electron hardening](#electron-hardening)
- [Launch switches](#launch-switches)
- [Content Security Policy](#content-security-policy)
- [The app and media schemes](#the-app-and-media-schemes)
- [IPC channels](#ipc-channels)
- [Network isolation](#network-isolation)
- [Data at rest](#data-at-rest)
- [Filesystem safety](#filesystem-safety)
- [Emptying the quarantine](#emptying-the-quarantine)
- [Deploy containment](#deploy-containment)
- [Tidying up](#tidying-up)
- [Junk files](#junk-files)
- [What a compromised renderer can and cannot do](#what-a-compromised-renderer-can-and-cannot-do)
- [Gaps](#gaps)
- [Packaging checklist](#packaging-checklist)

## Threat model

**What is protected.** First, the player's ROM library, which is often years of curation on a
NAS: it must never be changed, overwritten or lost. Second, the player's other files and their
network.

**Where untrusted input comes from:**

- File and folder names in the library. They can hold any byte a filesystem allows: control
  characters, bidi overrides, `..`-looking names, reserved Windows names, very long names.
- File contents: damaged or hostile zips, zip bombs, files that change while they are read.
- Cover art and other media files. They are shown in the page.
- Device profiles and `systems.json`. These are shipped data, but they are validated as if
  untrusted.
- Game database (DAT) files the player imports (see [decisions.md](decisions.md), ADR 38). In
  later milestones, scraper responses too (see [roadmap.md](roadmap.md)).

**Who the attackers are.** Someone who can put files into the library: a shared NAS, or a
downloaded ROM set. That person may also aim for code running in the renderer, through a
rendering bug or an injected script. The design assumes the renderer can be compromised and
limits what that buys (see
[What a compromised renderer can and cannot do](#what-a-compromised-renderer-can-and-cannot-do)).

**Out of scope:**

- Anyone who controls the app's environment variables or its data folder. They already control
  the process, which is why `ROMPEROOM_DATA_DIR` is honoured in packaged builds (see
  [configuration.md](configuration.md)).
- A malicious operating system or Electron binary.
- Physical access.

## Electron hardening

| Control                | Setting                                                                                              | Where                                          |
| ---------------------- | ---------------------------------------------------------------------------------------------------- | ---------------------------------------------- |
| Context isolation      | `contextIsolation: true`                                                                             | `windowOptions()`, `src/main/security.ts`      |
| Node in the page       | `nodeIntegration: false`, `sandbox: true`                                                            | `windowOptions()`                              |
| Webviews               | `webviewTag: false`, and attachment prevented                                                        | `windowOptions()`, `hardenWebContents()`       |
| Web security           | default (`webSecurity` on)                                                                           | `windowOptions()`                              |
| DevTools               | only when unpackaged                                                                                 | `windowOptions()`                              |
| Menu                   | packaged menu has no reload, force-reload or DevTools items                                          | `appMenuTemplate()`, `src/main/lifecycle.ts`   |
| New windows            | `window.open` denied                                                                                 | `hardenWebContents()`                          |
| Navigation             | `will-navigate`, `will-redirect` and `will-frame-navigate` blocked except the app page (`file:` too) | `hardenWebContents()`, `isAllowedNavigation()` |
| Permissions            | every request and every check denied                                                                 | `hardenSession()`                              |
| Response CSP           | our policy replaces any response's own                                                               | `hardenSession()`, `withCsp()`                 |
| WebRTC                 | UDP only through a proxy (`disable_non_proxied_udp`) plus a dead proxy for TCP                       | `hardenWebContents()`, `blockWebRtcTcp()`      |
| DNS                    | `host-resolver-rules` fails every lookup except loopback                                             | `hostResolverRules()`, before `ready`          |
| Every session          | `guardSession()` on the default session, then on each new one via `session-created`                  | `src/main/index.ts`                            |
| Preload                | one bundled CommonJS file that requires only `electron`; exposes a frozen `window.romperoom`         | `src/preload/index.ts`                         |
| Drag and drop          | `dragover` and `drop` refused in the page; navigation blocked as a backstop                          | `lib/page-guards.ts`, `hardenWebContents()`    |
| Renderer crash or hang | reload up to 3 times a minute, then ask; the engine keeps running in main                            | `src/main/recovery.ts`                         |
| Quit                   | held until the engine closes, at most 5 seconds                                                      | `createQuitCoordinator()`                      |

The table's rows are checked by `apps/desktop/test/security.test.ts`, which mocks the
`electron` module, and in the real app by `e2e/security.spec.ts` and `e2e/resilience.spec.ts`.
Packaged builds also flip Electron's fuses ([decisions.md](decisions.md#17-electron-fuses)):

| Fuse                                    | Set to | So that                                                                                  |
| --------------------------------------- | ------ | ---------------------------------------------------------------------------------------- |
| `RunAsNode`                             | off    | `ELECTRON_RUN_AS_NODE` cannot turn the app into a plain Node runtime                     |
| `EnableNodeOptionsEnvironmentVariable`  | off    | `NODE_OPTIONS` is ignored                                                                |
| `EnableNodeCliInspectArguments`         | off    | `--inspect` and friends cannot attach a debugger to the main process                     |
| `EnableEmbeddedAsarIntegrityValidation` | on     | a modified `app.asar` is refused at load                                                 |
| `OnlyLoadAppFromAsar`                   | on     | the app loads only from `app.asar`, never a loose `app/` folder                          |
| `EnableCookieEncryption`                | off    | no first-launch Keychain prompt: the app keeps no cookies or secrets in Chromium (below) |
| `GrantFileProtocolExtraPrivileges`      | off    | `file://` loses the extra privileges Electron grants it by default                       |
| `LoadBrowserProcessSpecificV8Snapshot`  | off    | Electron's default, one V8 snapshot                                                      |
| `WasmTrapHandlers`                      | on     | Electron's default                                                                       |

The policy is `src/main/fuse-policy.json`. `scripts/after-pack.mjs` writes it with
`@electron/fuses` (`strictlyRequireAllFuses`, so a fuse this Electron has but the policy does
not name fails the build), `verify:package` reads it back from the binary, and the running app
reads it again from its own executable in `--self-check` (`src/main/fuse-wire.ts`).
`--remote-debugging-port` is a Chromium switch, not a Node one, so no fuse governs it: the
packaged app refuses it itself ([launch switches](#launch-switches)).

Cookie encryption is off on purpose. The app stores no cookies and no secrets in the Chromium
session (its settings are in localStorage, see [data at rest](#data-at-rest)), and on macOS the
fuse can make the first launch ask for Keychain access. Revisit it when the app stores
credentials or cookies, and prefer Electron's
[`safeStorage`](https://www.electronjs.org/docs/latest/api/safe-storage) for secrets.

## Launch switches

A packaged build will not start with any of these switches on its command line. It prints one
line to standard error that names the switches (never their values) and exits with code 2. It
does this first, before it registers its schemes, appends switches of its own or becomes ready,
which is when Chromium would open a remote-debugging port. `forbiddenLaunchSwitches()` in
`src/main/security.ts` decides; `src/main/index.ts` calls it before anything else
([decisions.md](decisions.md#20-a-packaged-build-refuses-debugging-and-network-switches)).

| Switch                                                   | Refused because it                                                                                   |
| -------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| `--remote-debugging-*` (`-port`, `-pipe`, `-address`)    | opens the DevTools protocol: full control of the page from another process                           |
| `--remote-allow-origins`                                 | lets web pages reach that protocol                                                                   |
| `--inspect*` (`-brk`, `-port`, `-wait`), `--debug-port`  | attaches a debugger to the main process (the `EnableNodeCliInspectArguments` fuse also ignores them) |
| `--js-flags`                                             | changes how V8 runs the app                                                                          |
| `--no-sandbox`                                           | turns Chromium's renderer sandbox off                                                                |
| `--disable-web-security`                                 | turns the same-origin policy off                                                                     |
| `--host-rules`, `--host-resolver-rules`                  | replaces the app's resolver rules ([network isolation](#network-isolation))                          |
| `--proxy-server`, `--proxy-pac-url`, `--no-proxy-server` | sends traffic around the dead proxy that blocks WebRTC's TCP                                         |
| `--user-data-dir`                                        | moves Chromium's profile out of the data folder                                                      |

- **Every spelling matches.** Chromium reads `--name`, `-name` and, on Windows, `/name`, and
  ignores case on Windows. All of those match on every platform, with or without `=value`. The
  first argument (the executable's path) is not read. `app.commandLine.hasSwitch()` is asked
  about the same names as a backstop.
- **Unpackaged builds refuse nothing.** Playwright drives the development build through
  `--inspect` and `--remote-debugging-port` (`e2e/support.ts`).
- **What it does not cover.** Chromium has hundreds of switches; the list holds the ones that
  open a control channel, weaken the sandbox or web security, or reroute the network. Chromium
  creates a `--user-data-dir` folder (empty) before any app code runs, so a refused launch can
  still leave that folder behind. A local process that can launch the app with switches can
  already run code as the player; what the refusal removes is a control port left open while
  the player uses the app.
- **Evidence.** `test/security.test.ts` checks every spelling, ordinary arguments that must
  pass (`--self-check`, `--sandbox`, `--proxy-bypass-list`, macOS's `-psn_…`), the backstop,
  and that `index.ts` calls the function before `registerSchemesAsPrivileged`, `appendSwitch`
  and `whenReady`. `e2e:packaged` launches the packaged app with `--remote-debugging-port` on a
  free port, tries to connect the whole time it runs, and requires exit code 2 with no
  connection ever accepted; a server the test opens on that port first is the positive control.
  It launches the other families too, each of which must exit 2. Measured on macOS (Apple
  silicon): exit after about 1.5 seconds, 0 of about 51,000 connection attempts accepted, where
  the development build with the same switch accepted connections in the same time. Mutations:
  eight in `forbiddenLaunchSwitches()` (unit tests), and `app.isPackaged` inverted in
  `index.ts` (both refusal tests in `e2e:packaged` failed).

## Content Security Policy

`buildCsp()` in `src/main/security.ts` builds the policy. The page gets it twice, from one
function: as a `<meta>` in `index.html` and as a response header. In `index.html` the Vite
plugin `vite/csp-html.ts` replaces the placeholder `__ROMPEROOM_CSP__` at build time. The policy
of the built (packaged) app, where `'self'` is `app://romperoom`:

<!-- generated:csp — a docs test compares this block with buildCsp(APP_LOCATION). -->

```text
default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data: romperoom-media:; font-src 'self'; connect-src 'none'; frame-src 'none'; child-src 'none'; worker-src 'none'; object-src 'none'; base-uri 'none'; form-action 'none'
```

<!-- /generated:csp -->

- `connect-src 'none'`: no `fetch`, XHR, WebSocket or EventSource to anywhere, the app's own
  origin included.
- `img-src` allows `romperoom-media:` for cover art, plus `'self'` and `data:` URIs.
- `font-src 'self'`: the bundled pixel fonts. They are never fetched from a CDN (see
  `packages/ui/THIRD_PARTY.md`).
- `style-src 'unsafe-inline'`: React's `style` attributes, such as the virtualized wall's row
  offsets. Scripts are never inline.

Only in an unpackaged app that loads the Vite dev server does the policy change. `script-src`
then also allows `'unsafe-inline'` (React refresh's preamble) and the dev origin. `connect-src`
allows `'self'`, the dev origin and its `ws://` hot-reload socket. The dev server must be
loopback `http`, and `ELECTRON_RENDERER_URL` is ignored when packaged.

Media responses carry their own policy, `default-src 'none'; sandbox` (`MEDIA_CSP`), so a
media file opened as a document can run nothing.

## The app and media schemes

**`app://romperoom/`** serves the renderer (`src/main/app-protocol.ts`). It is registered before
`ready` as `standard`, `secure` and `supportFetchAPI`, never `bypassCSP` or `corsEnabled`. The
renderer is never loaded from `file://`, where CSP `'self'` would match every local file.

`resolveAppRequest()` decodes the path once. It refuses with 403:

- NUL characters and backslashes;
- `.`, `..` and empty segments;
- drive-letter forms, other hosts and credentials in the URL;
- any file whose `realpath` is not strictly inside the renderer folder, symlinks included.

Missing files and folders get 404, and `/` serves `index.html`. Only GET and HEAD are allowed.
Responses carry the CSP and `X-Content-Type-Options: nosniff`.

**`romperoom-media://media/<id>`** serves cover art (`src/main/media-protocol.ts`). It has the
same privileges, and it is never `stream`.

1. The page names only a catalogue id, written in canonical decimal (`007` is refused).
2. The host resolves the id with `engine.resolveMedia()`. That refuses an unknown id, a missing
   file, a file outside its library, a symlink (or anything below a symlinked folder), a
   non-regular file, a file over 50 MiB (`MAX_MEDIA_BYTES`) and any type outside the MIME
   allowlist.
3. The host opens the file `O_RDONLY | O_NOFOLLOW | O_NONBLOCK` and then trusts only the open
   descriptor's `fstat`. A symlink or FIFO swapped in after step 2 is refused.
4. The name must still be that file: `lstat` of the path must not be a link and must have the
   descriptor's device and inode. Where `O_NOFOLLOW` exists this is redundant. Windows has
   neither flag, so there this check is what refuses a swapped-in link (FIFOs do not arise).

Replies are 405 for anything but GET, 400 for a malformed URL and 404 otherwise. No body
carries a path or an errno. A 200 has the catalogue MIME type, `nosniff`, the sandbox CSP above
and a private cache header. `test/media-protocol.test.ts` and `test/app-protocol.test.ts` cover
both handlers.

## IPC channels

The page reaches the host only through `window.romperoom`. That object has one function per
channel in the `IPC` table (`src/shared/ipc.ts`), plus ten push listeners (`onScanProgress`,
`onDeployProgress`, `onTidyProgress`, `onIdentifyProgress`, `onDatsProgress`,
`onArtProgress`, `onSyncProgress`, `onStandardiseProgress`, `onJunkProgress` and
`onUpdatesProgress`). Raw
`ipcRenderer` and IPC event objects never cross the bridge. `IPC_MATCHES_API` makes the type
check fail unless the table and the `RendererApi` type list the same names, both ways. A docs
test checks that this table lists exactly the channels in `IPC`.

| Channel                     | Direction   | What it does                                                                                                                                                                                            | Writes?                                                                                                                                          |
| --------------------------- | ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| `engine:addLibrary`         | page → host | Adds an absolute folder as a library (not a filesystem root, not one already added, not inside or around one)                                                                                           | catalog only                                                                                                                                     |
| `engine:removeLibrary`      | page → host | Forgets a library's catalog rows, journals and runs; never touches files; refused while a scan runs, a job holds it or something of it waits in Recovery                                                | catalog only                                                                                                                                     |
| `engine:listLibraries`      | page → host | Lists libraries; with `{ details: true }` (nothing else is accepted) also each one's name, game count, last scan and whether its folder answers (the host reads each library's own folder, at most 3 s) | no                                                                                                                                               |
| `engine:scan`               | page → host | Scans a library read-only; progress comes back on `scanProgress`                                                                                                                                        | catalog only                                                                                                                                     |
| `engine:cancelScan`         | page → host | Cancels the running scan                                                                                                                                                                                | no                                                                                                                                               |
| `engine:listSystems`        | page → host | Systems that have games, with counts                                                                                                                                                                    | no                                                                                                                                               |
| `engine:listKnownSystems`   | page → host | Every known system, for assigning a folder                                                                                                                                                              | no                                                                                                                                               |
| `engine:listGames`          | page → host | A page of games (at most 500), filtered, searched and sorted                                                                                                                                            | no                                                                                                                                               |
| `engine:gameLetters`        | page → host | How many games a filter has, and where each letter starts in title order (the games page's letter strip)                                                                                                | no                                                                                                                                               |
| `engine:listUnreadable`     | page → host | Files the scans could not read, with reasons                                                                                                                                                            | no                                                                                                                                               |
| `engine:assignFolder`       | page → host | Maps a top-level folder to a system                                                                                                                                                                     | catalog only                                                                                                                                     |
| `engine:unassignFolder`     | page → host | Undoes `assignFolder`                                                                                                                                                                                   | catalog only                                                                                                                                     |
| `engine:ignoreFolder`       | page → host | Marks a top-level folder as not a console                                                                                                                                                               | catalog only                                                                                                                                     |
| `engine:unignoreFolder`     | page → host | Includes an ignored folder again                                                                                                                                                                        | catalog only                                                                                                                                     |
| `engine:listIgnored`        | page → host | A library's ignored folders                                                                                                                                                                             | no                                                                                                                                               |
| `engine:health`             | page → host | The library's check-up, for Home and Fix up                                                                                                                                                             | no                                                                                                                                               |
| `engine:sizes`              | page → host | Space used per console                                                                                                                                                                                  | no                                                                                                                                               |
| `app:pickFolder`            | page → host | Opens the native folder dialog                                                                                                                                                                          | no                                                                                                                                               |
| `app:scanStatus`            | page → host | The scan the host is running for this window, if any                                                                                                                                                    | no                                                                                                                                               |
| `engine:scanProgress`       | host → page | Scan progress pushes                                                                                                                                                                                    | no                                                                                                                                               |
| `deploy:listProfiles`       | page → host | The device profiles, with their folders, limits and sources                                                                                                                                             | no                                                                                                                                               |
| `deploy:listVolumes`        | page → host | Connected drives by id (never a mount path), each judged by the safe-target rules                                                                                                                       | no                                                                                                                                               |
| `deploy:plan`               | page → host | Plans a package for a volume id, a card size or an export-folder token                                                                                                                                  | no                                                                                                                                               |
| `deploy:dryRun`             | page → host | What a start would do, file by file, without writing                                                                                                                                                    | no                                                                                                                                               |
| `deploy:start`              | page → host | Writes a kept plan to its card or folder, after listing and judging it again                                                                                                                            | the card                                                                                                                                         |
| `deploy:cancel`             | page → host | Stops this window's deploy after the current chunk                                                                                                                                                      | no                                                                                                                                               |
| `deploy:status`             | page → host | This window's running or last deploy (a reloaded page adopts it)                                                                                                                                        | no                                                                                                                                               |
| `deploy:pickExportFolder`   | page → host | Opens the native folder dialog; returns a token and the folder's name                                                                                                                                   | no                                                                                                                                               |
| `deploy:pickBiosFolder`     | page → host | Opens the native folder dialog and stores a library's BIOS folder                                                                                                                                       | catalog only                                                                                                                                     |
| `deploy:clearBiosFolder`    | page → host | Forgets a library's BIOS folder                                                                                                                                                                         | catalog only                                                                                                                                     |
| `deploy:listBiosFolders`    | page → host | Each library's BIOS folder, by name                                                                                                                                                                     | no                                                                                                                                               |
| `deploy:progress`           | host → page | Deploy progress, then the report, to the window that started it                                                                                                                                         | no                                                                                                                                               |
| `tidy:overview`             | page → host | Duplicate, leftover-artwork and set-aside totals per library, by name                                                                                                                                   | no                                                                                                                                               |
| `tidy:findDuplicates`       | page → host | One library's duplicate groups under a keeper policy; returns a window-bound id. With `across: true`, the files held in two or more libraries, by library name (a report, no id)                        | no                                                                                                                                               |
| `tidy:groups`               | page → host | Another page of the same groups (refused once they changed)                                                                                                                                             | no                                                                                                                                               |
| `tidy:planDuplicateCleanup` | page → host | Plans setting aside every copy but the keeper, with per-group choices                                                                                                                                   | no                                                                                                                                               |
| `tidy:detectOrphans`        | page → host | Artwork no game uses, by media id and cause (optionally of one cause), with a count per cause                                                                                                           | catalog only (checksums)                                                                                                                         |
| `tidy:planOrphanCleanup`    | page → host | Plans setting aside the chosen artwork, by media id                                                                                                                                                     | no                                                                                                                                               |
| `tidy:apply`                | page → host | Runs a plan once: moves its files into `.romperoom-quarantine`                                                                                                                                          | the library                                                                                                                                      |
| `tidy:cancel`               | page → host | Stops this window's tidy run after the current file                                                                                                                                                     | no                                                                                                                                               |
| `tidy:status`               | page → host | This window's running or last tidy job (a reloaded page adopts it)                                                                                                                                      | no                                                                                                                                               |
| `tidy:undo`                 | page → host | Puts an operation's set-aside files back                                                                                                                                                                | the library                                                                                                                                      |
| `tidy:listOperations`       | page → host | Past tidy and delete operations, with what each still has set aside                                                                                                                                     | no                                                                                                                                               |
| `tidy:listQuarantine`       | page → host | Set-aside files, library-relative, by library or operation                                                                                                                                              | no                                                                                                                                               |
| `tidy:restore`              | page → host | Puts chosen set-aside files (by item id) or an operation's files back                                                                                                                                   | the library                                                                                                                                      |
| `tidy:purgePreview`         | page → host | What deleting set-aside files forever (all, or those set aside more than 30 or 90 days ago) would delete, and the words to type                                                                         | no                                                                                                                                               |
| `tidy:purge`                | page → host | Deletes a previewed set forever; the engine checks the typed words and size                                                                                                                             | the quarantine                                                                                                                                   |
| `tidy:recoverList`          | page → host | Tidy runs and deletions a crash interrupted, by library name                                                                                                                                            | no                                                                                                                                               |
| `tidy:resolveJournal`       | page → host | Finishes or undoes one interrupted run or deletion, or discards the rest of a Tidy up run (catalog only)                                                                                                | the library                                                                                                                                      |
| `tidy:reconnectLibrary`     | page → host | After "Is this still your library?": re-points a library's runs at the folder now at its path (`{ confirm: true }`; the engine refuses an empty or different folder)                                    | no (reads the library folder)                                                                                                                    |
| `tidy:mergeArt`             | page → host | Merge picture folders: what moving each console's `Imgs` pictures into `images` would do, names only (a library id and a page; held under a window-bound id)                                            | no (reads the library)                                                                                                                           |
| `tidy:mergeArtRun`          | page → host | Runs that look once as a tidy job: the engine reads both folders again and refuses on any change, then moves and sets aside under the library's lock (Undo is `tidy:undo`)                              | the library                                                                                                                                      |
| `tidy:progress`             | host → page | Tidy progress, then the result, to the window that started it                                                                                                                                           | no                                                                                                                                               |
| `identify:importDat`        | page → host | Opens the host's file picker and imports the picked game database (DAT); no path from the page                                                                                                          | catalog only (reads the picked file)                                                                                                             |
| `identify:listDats`         | page → host | The imported game databases                                                                                                                                                                             | no                                                                                                                                               |
| `identify:removeDat`        | page → host | Removes a game database by id and reverts the matches it made                                                                                                                                           | catalog only                                                                                                                                     |
| `identify:run`              | page → host | Identifies one library, or every library, against the game databases                                                                                                                                    | catalog only (re-reads library files)                                                                                                            |
| `identify:cancel`           | page → host | Stops this window's identify run                                                                                                                                                                        | no                                                                                                                                               |
| `identify:status`           | page → host | This window's running or last identify run (a reloaded page adopts it)                                                                                                                                  | no                                                                                                                                               |
| `identify:listReview`       | page → host | Matches waiting for review, library-relative                                                                                                                                                            | no                                                                                                                                               |
| `identify:listUnidentified` | page → host | Present files with no DAT match and why, library-relative                                                                                                                                               | no                                                                                                                                               |
| `identify:decideReview`     | page → host | Accepts or rejects matches on the review list, by file id                                                                                                                                               | catalog only                                                                                                                                     |
| `identify:game`             | page → host | One game's identity: its database name, confidence, flags and reasons                                                                                                                                   | no                                                                                                                                               |
| `identify:progress`         | host → page | Identify progress, then the result, to the window that started it                                                                                                                                       | no                                                                                                                                               |
| `dats:plan`                 | page → host | What "Download for me" would fetch (files, sizes, commit, licence), computed without a request                                                                                                          | no                                                                                                                                               |
| `dats:openLink`             | page → host | Opens one of four fixed pages in the browser, by key (`no-intro`, `redump`, `licence`, `source`)                                                                                                        | no                                                                                                                                               |
| `dats:download`             | page → host | Downloads, verifies and imports the chosen consoles' DATs from the pinned listing; console ids and the reviewed commit only                                                                             | catalog and data folder                                                                                                                          |
| `dats:cancel`               | page → host | Stops this window's download job                                                                                                                                                                        | no                                                                                                                                               |
| `dats:status`               | page → host | This window's running or last download job (a reloaded page adopts it)                                                                                                                                  | no                                                                                                                                               |
| `dats:checkForUpdates`      | page → host | Asks GitHub for a newer listing and moves the pin forward                                                                                                                                               | data folder (pin)                                                                                                                                |
| `dats:networkLog`           | page → host | Every request attempt, newest first                                                                                                                                                                     | no                                                                                                                                               |
| `dats:progress`             | host → page | Download progress, then the result, to the window that started it                                                                                                                                       | no                                                                                                                                               |
| `art:review` | page → host | Lists the libretro-thumbnails pictures for consoles with gaps (GitHub listings; cached a day) and returns a review; with `{ source: 'screenscraper' }` it returns ScreenScraper's review instead, from the catalog, asking nothing | data folder (listing cache)                                                                                                                      |
| `art:download` | page → host | Downloads, verifies and saves the reviewed pictures; plan id, console ids and kinds only. For a ScreenScraper review's plan it runs the lookup: requests to `api.screenscraper.fr` with each chosen game's checksums, file name, size and console number, and saves pictures and descriptions | library (`.romperoom/media`) and catalog (and `game_description`) |
| `art:cancel`                | page → host | Stops this window's cover art run                                                                                                                                                                       | no                                                                                                                                               |
| `art:status`                | page → host | This window's running or last cover art run (a reloaded page adopts it)                                                                                                                                 | no                                                                                                                                               |
| `art:cardReview`            | page → host | Reads a listed card's art through a device profile; volume id and profile id only                                                                                                                       | no                                                                                                                                               |
| `art:importCard`            | page → host | Saves the reviewed card pictures                                                                                                                                                                        | library (`.romperoom/media`) and catalog                                                                                                         |
| `art:remove`                | page → host | Deletes the pictures cover art added that are unchanged                                                                                                                                                 | library and catalog                                                                                                                              |
| `art:scraperAccount`        | page → host | Whether ScreenScraper can be used: the account's state and whether this copy has developer details; never the name or password                                                                          | no                                                                                                                                               |
| `art:setScraperAccount`     | page → host | Saves the player's ScreenScraper account in the system keychain (`{ user, password }`), or forgets it (`{ forget: true }`); returns the state only                                                      | the account file (ciphertext)                                                                                                                    |
| `art:progress`              | host → page | Cover art progress, then the result, to the window that started it                                                                                                                                      | no                                                                                                                                               |
| `sync:review`               | page → host | Reads a listed card through a device profile against a library (hashing card games, offline) and returns a review; volume id, profile id and library id only                                            | catalog (card game hashes of a known card)                                                                                                       |
| `sync:run`                  | page → host | Imports the chosen games and syncs the chosen saves; the review's plan id, its item ids and a side per conflict only                                                                                    | library (console folders, `.romperoom/saves`, `.romperoom/saves-backup`, `.romperoom/tmp`), the card (saves, `.romperoom/card.json`) and catalog |
| `sync:cancel`               | page → host | Stops this window's card sync (the file being copied is finished or discarded)                                                                                                                          | no                                                                                                                                               |
| `sync:status`               | page → host | This window's running or last card sync (a reloaded page adopts it)                                                                                                                                     | no                                                                                                                                               |
| `sync:undo`                 | page → host | Undo this sync: what it wrote into the library goes back where unchanged; a sync id only                                                                                                                | library and catalog                                                                                                                              |
| `sync:list`                 | page → host | The latest syncs, newest first                                                                                                                                                                          | no                                                                                                                                               |
| `sync:progress`             | host → page | Card sync progress, then the result, to the window that started it                                                                                                                                      | no                                                                                                                                               |
| `standardise:review`        | page → host | Reads a library against a device profile and returns a review (nothing written but the chosen profile); library id and profile id only                                                                  | catalog (the library's chosen profile)                                                                                                           |
| `standardise:run`           | page → host | Renames and merges the chosen console folders and renames the chosen games, with their art, saves, cue sheets, playlists and game lists; the review's plan id and its item ids only                     | library (console folders, games, art, saves, game lists, `.romperoom/lists-backup`, `.romperoom/tmp`, `.romperoom-quarantine`) and catalog       |
| `standardise:cancel`        | page → host | Stops this window's standardise review or run, or its re-link run (a run stops between units, a re-link run between pictures)                                                                           | no                                                                                                                                               |
| `standardise:status`        | page → host | This window's running or last standardise run (a reloaded page adopts it)                                                                                                                               | no                                                                                                                                               |
| `standardise:undo`          | page → host | Undo this run: what it renamed and replaced goes back where unchanged; a run id only                                                                                                                    | library and catalog                                                                                                                              |
| `standardise:list`          | page → host | Each library's chosen profile and the latest standardise runs                                                                                                                                           | no                                                                                                                                               |
| `standardise:relinkReview`  | page → host | Reads a re-link review of a library: leftover pictures one game clearly matches, and game list entries whose game file is gone; a library id only                                                       | catalog only (checksums)                                                                                                                         |
| `standardise:relinkRun`     | page → host | Renames the chosen pictures after their game and updates the game lists that name them, and re-points the chosen entries; the re-link review's plan id and its item ids only                            | library (pictures, game lists, `.romperoom/lists-backup`, `.romperoom/tmp`) and catalog                                                          |
| `standardise:progress`      | host → page | Standardise progress, then the result, to the window that started it                                                                                                                                    | no                                                                                                                                               |
| `junk:look`                 | page → host | Reads a library (its id) or the card a copy of this window wrote to (the copy's plan id) for junk files (Finder and Windows metadata, each checked by content); never a path                            | no                                                                                                                                               |
| `junk:page`                 | page → host | Another page of a look's files; the look's id only (bound to the window that made it)                                                                                                                   | no                                                                                                                                               |
| `junk:remove`               | page → host | Removes a look's files for good, each checked again first; the look's id only; a library's removal holds its lock and is listed in History                                                              | library or card (junk files only), catalog (History)                                                                                             |
| `junk:cancel`               | page → host | Stops this window's look or removal (a removal finishes or puts back the file in hand first)                                                                                                            | no                                                                                                                                               |
| `junk:progress`             | host → page | How far this window's look (entries looked at) or removal (files done of all) is                                                                                                                        | no                                                                                                                                               |
| `updates:status` | page → host | The running version, how this copy installs, the last check and this window's install (a reloaded page adopts it) | no |
| `updates:check` | page → host | Asks GitHub for the newest release (one API request) and, for a newer one, verifies its `SHA256SUMS.txt` signature | no |
| `updates:install` | page → host | Downloads, verifies and installs (Windows, AppImage) or shows (macOS, .deb, Windows zip) the version the last check offered; the version string only | data folder (`updates/`); on Windows and AppImage, the installed app |
| `updates:cancel` | page → host | Stops this window's update download | no |
| `updates:showFile` | page → host | Shows the verified file in the system file manager | no |
| `updates:openReleasePage` | page → host | Opens the offered release's page, or the releases page, in the browser | no |
| `updates:progress` | host → page | Update download progress, then the result, to the window that started it | no |

`engine:gameLetters`, like `engine:listGames`, takes no path: only a console id (checked by the
engine's `systemIdArg`) and a search string of bounded length, under the same filter
(`gameFilter`).

Every handler (`src/main/handlers.ts`) applies the same rules:

- **Who may call.** The call must come from the main window's `webContents`, from its main
  frame (`senderFrame === sender.mainFrame`, so a subframe is refused even when it shows the app
  page), and that frame must show the app's own page. Anything else rejects with
  `untrusted IPC sender`, and the engine is not called.
- **Arguments.** They go to the engine unchanged. The engine validates them as untrusted:
  string lengths are capped at 4096 (`MAX_STRING`), list limits at 500 (`MAX_LIMIT`), and
  library paths must be absolute directories.
- **Replies.** A handler resolves to `{ ok: true, value }` or
  `{ ok: false, error: { name, message } }`. The message is its first line, at most 1000
  characters, and never carries a stack or extra properties. One named exception: a
  `DatNeedsSystemError` keeps its `suggestion` (the console the DAT's name suggests), and only
  when it is a system id (`^[a-z0-9-]{1,64}$`). The preload turns an error reply back into a
  thrown `Error`, with the name (and a suggestion, as `(suggested console: …)`) in its message,
  since the bridge drops an Error's own properties. An unexpected error's first line is passed on
  as it is: it can name a path inside a library (the page already lists library paths), never a
  stack.

The `deploy:` channels are handled by the deploy host (`src/main/deploy-host.ts`), not passed
to the engine; see [Deploying to a card](#deploying-to-a-card). The `tidy:` channels are
handled by the tidy host (`src/main/tidy-host.ts`); see [Tidying up](#tidying-up). The
`identify:` channels are handled by the identify host (`src/main/identify-host.ts`), which checks
every argument before the engine checks it again. `identify:importDat` takes no path from the
page: the host opens its own file picker and passes the file it returns to the engine. A run's
progress and result go only to the window that started it, and after a complete scan the host
identifies that library by itself when a game database is imported.

The `junk:` channels are handled by the junk host (`src/main/junk-host.ts`). They take a
library id, a plan id of a copy this window started (the deploy host remembers where that copy
wrote), and a look id bound to the window that made it; never a path. Every message goes back
through `pageText`, so a card or folder is named, never shown by its path. What a removal may
touch is the engine's rule: [Junk files](#junk-files).

The `updates:` channels are handled by the updates host (`src/main/updates-host.ts`). The page
sends nothing but, to `updates:install`, back the version string the last check offered (at most
32 characters, compared with the host's own record, else `UpdatesArgumentError`); never a URL, a
path, a file name or a hash. The host chose the file by name from the verified checksum list.
One check and one install run at a time (`UpdateBusyError`); an install binds to the window that
started it, its progress and result go only to that window, only it can stop it, and closing it
or quitting stops the download (the partial file is removed; a Stop that lands while the file
is being checked removes the checked file too). While a copy to a card, a Tidy or Standardise
run, a scan, a junk removal or any other job holding a library runs, an install is refused with
the reason, at the press and again after the download, before anything runs. The re-check also
sees work that was pressed and has not taken its lock yet: the deploy and tidy hosts' slots, and
every call of a channel that starts work while it runs (`WorkTracker` in
`src/main/handlers.ts`). A download refused there is removed. Show in folder hashes the kept
file again first and removes it, rather than show it, when it changed.

From the moment an automatic install (the Windows installer, the AppImage swap) is decided, just
before the busy re-check, the app is closing for it (`UpdatesHost.closing()`): every channel that
starts work (a copy or its dry run, a Tidy plan, run or undo, a Standardise run or undo, a scan,
identify, a game database download or check, a cover art download, a card sync, a junk look or
removal, a library change: the 36 in `WORK_STARTS` in `src/main/handlers.ts`) is refused with
"Romperoom is closing to install an update" before its host sees it. The pure setters (the BIOS
folder, the ScreenScraper account) and every read stay open. Then the order is fixed: every host
is stopped and the engine closed (the quit's own bounded close, `closeForInstall`), the installer
is hashed once more and started (or the AppImage's restart set up from the new file on disk,
with this process's arguments), and the process exits at once (`app.exit(0)`), so the
installer's check for a running Romperoom never races the engine's close. Once the result is
`restarting` that is final: a second Install, a check, Stop, Show in folder and the release page
are all refused (the status still answers). A failure before the close ends it, and work may
start again; after the close nothing is reopened, so work stays refused and the tab says to
close Romperoom and open it again.

Romperoom runs the installer or swaps the AppImage itself only when the `updates/` folder is
private to this user (`isPrivateFolder` in `src/main/updates/install.ts`). On macOS and Linux:
a real folder owned by this user, with no group or other access (0700). On Windows, which has no
mode bits (the folder inherits its parent's access list): only the app's own per-user data
folder, never one `ROMPEROOM_DATA_DIR` moved (a stick, or a folder on `C:\` other users can
write). Otherwise the file is downloaded and verified, then shown in its folder as on macOS
(`reason: 'not-private'`): "Romperoom is keeping its files somewhere other users could change, so
it will not run the installer for you."

The `dats:` channels are handled by the dats host (`src/main/dats-host.ts`). `dats:download`
and `dats:openLink` take no URL or path from the page. `dats:openLink` takes a key of the closed
`DAT_LINKS` map. `dats:download` takes 1 to 79 distinct console ids and the commit the page was
shown in the plan; the host accepts the commit only as exactly 40 lower-case hex characters
(anything else is refused before the service sees it) and uses it only to compare with the pin,
so a job runs only against what was reviewed: never to build a URL or a path. Which consoles
may be fetched is the service's check (mapped, in the library, not up to date). One job runs at a
time and binds to the window that started it; a second, or "Check game database updates" while one
runs, is refused with `DatDownloadBusyError`. Quitting or closing that window stops the job; each
DAT already imported stays. `identify:importDat` also takes `inDownloads` (a boolean): the picker
then opens in the Downloads folder, preselecting the newest `.dat`, `.zip` or `.xml` modified since
an official page was opened (`main/downloads.ts`: `lstat` only, no link followed, at most 2,000
entries looked at). It never passes an origin from the page to the engine.

The `art:` channels are handled by the art host (`src/main/art-host.ts`). The page sends console
ids (1 to 1,000 distinct, each `^[a-z0-9-]{1,64}$`), kind names from the closed set `box`,
`screenshot` and `title` (each once), a volume id from the host's own drive listing (at most
100 characters), a device profile id the host knows, and back the review's plan id. It never
sends a URL, a path, a repository or a file name. The plan id must be a lower-case UUID, the
shape the service hands out, and one the host gave to that same window (at most 8 are kept);
anything else is refused with `ArtsArgumentError` before the service is called, and the service
uses the id only as a key into its own reviews. A card review is bound to the card's id and
mount path: at import the host lists the drives again and refuses the import ("the card changed
since it was reviewed") when the card is gone or mounted elsewhere, and only then passes the
mount path, which the page never sees. One cover art job (a review, a run or a removal) runs at
a time, and any other is refused with `ArtBusyError`; a run binds to the window that started
it, its progress and result go only to that window, only it can stop it, and closing that window
or quitting stops it (each saved picture stays).

The `sync:` channels are handled by the sync host (`src/main/sync-host.ts`). The page sends a
volume id from the host's own drive listing (at most 100 characters), a device profile id the
host knows, a library id, back the review's plan id with item ids from that review (`g<n>`,
`s<n>`, each once) and, per conflict, the side that wins (`card` or `library`), and a sync id. It
never sends a path, a URL or a file name. The plan id must be a lower-case UUID and one the host
gave to that same window (at most 4 are kept); anything else is refused with
`SyncArgumentError` before the engine is called. A card is synced only when every deploy
safe-target rule passes for it, at the review and again before the run, which also needs the
same mount ([Filesystem safety](#filesystem-safety)); a card that fails that second check (taken
out, mounted elsewhere, refused by a rule, or not checkable) is refused with
`SyncCardRefusedError`, and the page says the card can't be synced. One sync job (a review, a
run or an undo) runs at a time, and any other is refused with `SyncBusyError`; a review or run
binds to the window that started it, its progress and result go only to that window, only it can
stop it, and closing that window or quitting stops it (what was copied stays). The results
carry names and library-relative places, never the card's mount or the library's path; why Undo
left a file is one of a closed set of reasons, and the file system's own message is only logged.

The `standardise:` channels are handled by the standardise host (`src/main/standardise-host.ts`).
The page sends a library id the host lists, a device profile id the host knows, back the review's
plan id with item ids from that review (`f<n>`, `g<n>`, `c<n>`, each once, each in its own list),
and a run id. For a re-link it sends a library id, then the re-link review's plan id with its
picture and entry ids (`p<n>`, `e<n>`, each once, each in its own list). It never sends a path or a
file name. The plan id must be a lower-case UUID the host gave to that same window (the host keeps
as many reviews of each kind as the engine does, 2); anything else is refused with
`StandardiseArgumentError` before the engine is called. One standardise job (a review, a run, a
re-link run or an undo) runs at a time, and any other is refused with `StandardiseBusyError` (a
re-link review only reads the library, so it is no job and runs beside them; a reload drops the
window's re-link reviews, one still being read is not kept, and one a refused run would put back is
not put back); a review or run binds to the window that started it, its progress and result go only
to that window, only it can stop it, and closing that window or quitting stops it between units
(what was renamed stays; Recovery, Fix up's **Finish or undo…**, finishes or undoes it). A run and
its undo hold the library lock Tidy up uses, so a run started while another job holds the library is
refused with nothing written, and while one runs every other job says the library is busy with "Tidy
up or Standardise". The engine refuses a choice the review did not offer, a review older than the
catalog's last change, and a new run while an earlier run of the library waits in Recovery; removing
that library is refused too. The results carry names and library-relative paths, never the library's
own path; why Undo left a file is one of a closed set of reasons, and the file system's own message
is only logged.

The engine members that are _not_ exposed are `close`, `lastScan`, `resolveMedia`,
`recoverJournals`, `resolveJournal` and the `tidy`, `identify`, `art`, `sync` and `standardise`
facades themselves (host-only). The operation engine (`applyPlan`, `undoJournal` and the rest) has
no channel at all: the page reaches tidy runs only through the tidy host's ids.

Adding a channel: add the `Engine` method and its `IPC` entry together, since
`IPC_MATCHES_API` fails until both exist. Then list it where the tests pin the exact surface:

- `RESULTS` and the method count in `test/ipc-contract.test.ts`;
- `API_METHODS` in `test/ipc.test.ts`;
- the count in `test/security.test.ts`;
- `BRIDGE_KEYS` in `e2e/security.spec.ts`;
- the renderer's fake API (`test/renderer/fake-api.ts`);
- the table above.

## Network isolation

Romperoom makes no network request unless the user presses Download for me or Check game database
updates under Fix up › Game databases, or Get cover art in Fix up › Artwork (and then Download in
its review), or Look up N games in a ScreenScraper review, or Check for a new version or Install in
Settings › Updates (or, when the player turned it on, one check as the window opens), and the page
cannot make one at all. The CSP
stops `fetch`, XHR and sockets. Three more layers cover what CSP does not govern.

WebRTC is outside CSP: `connect-src 'none'` does not stop a peer connection. Also, any
same-origin frame an injected script makes (about:blank, `srcdoc`, nested) has its own
`RTCPeerConnection`. These layers keep it from reaching any host, the LAN included:

- **UDP** (STUN, TURN over UDP, ICE over UDP). `hardenWebContents()` sets the IP handling
  policy `disable_non_proxied_udp` on every webContents. WebRTC may then use UDP only through a
  proxy, and no proxy here relays UDP.
- **TCP** (TURN over TCP, ICE-TCP). That policy still allows TCP, which follows the session's
  proxy settings. Before the first window loads, `blockWebRtcTcp()` gives the session a dead
  proxy: `http`, `https` and `socks` all point at `127.0.0.1:9`, where nothing listens.
  `<-loopback>` removes Chromium's implicit loopback bypass. `app:` and `romperoom-media:` are
  custom protocols and are never proxied. In dev only the dev server's `host:port` is bypassed.
- **DNS** (dns-prefetch, preconnect, STUN and TURN hostnames). The proxy and the CSP stop
  connections, not lookups. A lookup alone tells the network's resolver a name the page chose,
  and an OS resolver also retries a single-label name with each search domain appended. Before
  `ready`, the `host-resolver-rules` switch is set to
  `MAP * ~NOTFOUND, EXCLUDE localhost, EXCLUDE 127.0.0.1`. Every hostname, and every IP literal
  other than `127.0.0.1`, then fails as not found inside Chromium, before any resolver is asked.
  This covers the whole process, packaged or not. In dev, a `[::1]` dev server adds
  `EXCLUDE ::1`.
- **Workers and frames.** `worker-src 'none'` and `frame-src 'none'` stop workers and framed
  documents from other origins. As defence in depth only, the page also removes its own WebRTC
  constructors (`lib/page-guards.ts`).
- **Every session.** The proxy is a per-session setting, and a session made later with
  `session.fromPartition()` starts with none (measured: `DIRECT`). `guardSession()` (permissions,
  CSP, dead proxy) is awaited on the default session before the first window. An
  `app.on('session-created')` handler then applies it to every session created later. A future
  partition that needs its own settings must set the same ones itself, never fewer.

How it is tested: `e2e/resilience.spec.ts` runs all four WebRTC channels (STUN/UDP, TURN/UDP,
TURN/TCP and ICE-TCP), from the top window and from an injected frame. The targets are a local
UDP socket and a TCP server, on loopback and on the LAN address.

- Guarded, nothing is contacted.
- Under the test-only control `ROMPEROOM_TEST_UNGUARDED_WEBRTC=1` (ignored when packaged), every
  channel gets through. This is the positive control.

The same file also records a Chromium net log while the page names hosts through dns-prefetch,
preconnect and STUN and TURN servers. Guarded, no net log event mentions any of those names.
Under the control, they are looked up.

On Electron 44, the process-wide `--force-webrtc-ip-handling-policy` switch was measured to do
nothing on its own: with only the switch, the STUN probe still received UDP packets. It stays as
a harmless extra layer.

**Letting a future feature through.** The switch and the proxy are both allowlists, with one
entry per host. Take an update feed at `updates.example.com`. Pass it to
`hostResolverRules(['updates.example.com'])`. Then bypass the proxy for exactly that host, on a
dedicated session that makes the request from the main process with Electron's `net`:
`proxyBypassRules: '<-loopback>,updates.example.com'`. On Electron 44 that bypass was measured
to send `https://updates.example.com/` direct, while `evil.updates.example.com`, other hosts and
loopback still went to the dead proxy. Never lift the renderer session's proxy. The game
database download chose `node:https` instead, so that no host is excluded from the resolver
rules (ADR 40).

### Main-process requests

`main/dat-download/transport.ts` is the only module that opens a socket (enforced twice: the
import list and the lint rules under "Static tests" and "Lint rules" below). It requests nothing
on its own: its only callers are the game database download, "Check game database updates", Get
cover art (its listings, then its review's Download), a ScreenScraper lookup and the app updater
(Settings › Updates), each started by a press (the updater's opt-in check at start is the
page's, once per app start: a page that loads again uses the host's answer). Cover art, ScreenScraper and the updater share the game database
download's one transport, so the
allowlist, the transport rules and the request log below apply to them unchanged.

**Allowlist** (`main/dat-download/allowlist.ts`). `checkDatUrl` runs before every request; a
refused URL never reaches a socket and is logged as `refused`. A URL passes only when all of
these hold:

- the URL is exactly its canonical serialization: a spelling the parser would normalise (upper
  case, `:443`, a backslash, a tab, dot segments, full-width letters) is refused, not rewritten;
- `https:`, no username or password, no port, no fragment;
- the parsed host is exactly `raw.githubusercontent.com`, `api.github.com`,
  `api.screenscraper.fr` or `github.com` (string equality: no suffix, prefix, trailing dot or
  wildcard), or, as a release file's redirect only, `release-assets.githubusercontent.com` or
  `objects.githubusercontent.com`;
- on `raw.githubusercontent.com`: the path is
  `/libretro/libretro-database/<40 lower-case hex>/metadat/(no-intro|redump)/<name>.dat`, the
  name is a safe DAT name in the exact encoding the app builds (no `%2F`, no `..`, no
  alternative escapes), and there is no query;
- on `api.github.com`: exactly `/repos/libretro/libretro-database/branches/master` with no
  query, or `/repos/libretro/libretro-database/contents/metadat/(no-intro|redump)` or
  `/repos/libretro/libretro-database/contents/dat` with the query exactly
  `ref=<40 lower-case hex>`;
- for the BIOS list, on `raw.githubusercontent.com`: exactly
  `/libretro/libretro-database/<40 lower-case hex>/dat/System.dat`, with no query;
- on `api.screenscraper.fr`: exactly `/api2/jeuInfos.php` or `/api2/mediaJeu.php`, with the
  query's keys in exactly the order the app builds them (the developer details, `output=json`
  and the user details, then the game's checksums, console number, `romtype=rom`, name and size;
  or the console number, game id and media name), each value of its own fixed shape, MD5 and CRC the only keys that may be left out, and
  the query exactly its own canonical encoding;
- for cover art, on `api.github.com`: a thumbnail tree,
  `/repos/libretro-thumbnails/<repo>/git/trees/<branch>` with the query exactly `recursive=1`,
  or, with no query, `/repos/libretro-thumbnails/<repo>/git/trees/<branch>:Named_Boxarts` (or
  `Named_Snaps`, `Named_Titles`) for a listing GitHub cut short;
- for cover art, on `raw.githubusercontent.com`:
  `/libretro-thumbnails/<repo>/<branch>/Named_(Boxarts|Snaps|Titles)/<name>`, with no query,
  where `<name>` is one safe picture name (one segment, no `..`, no separator or control
  character, `.png`, at most 255 bytes) in exactly the percent-encoding the app builds.
- for app updates, on `api.github.com`: exactly
  `/repos/MakerCorn/romperoom-releases/releases?per_page=20` (the release list);
- for app updates, on `github.com`: exactly
  `/MakerCorn/romperoom-releases/releases/download/v<version>/<name>` with no query, where the
  version is `X.Y.Z` (no leading zero, no prerelease) and the name is `SHA256SUMS.txt`,
  `SHA256SUMS.txt.sig` or `Romperoom-<the same version>-` followed by `win-x64.exe`,
  `win-x64.zip`, `linux-x86_64.AppImage`, `linux-amd64.deb` or `mac-arm64.dmg`;
- a release file's one redirect (`checkReleaseAssetUrl`): https, one of the two asset hosts
  exactly, no username, password, port, fragment, dot segment, space, control or non-ASCII
  character, at most 8 KiB. Its query is GitHub's signature: sent as received, never logged or
  shown in a refusal. `checkDatUrl` refuses the asset hosts, so nothing else reaches them.

In both cover art shapes, `<repo>` and `<branch>` must be a pair from the shipped measured list
(`packages/profiles/data/thumbnails.json`, 131 repositories): each repository's branch is the
one that list records for it (`master`, or `main` for 12), never taken from the page or from a
listing. A repository with the right shape but not on the list, or a listed repository at the
other branch, is refused. `thumbnails.libretro.com` is never contacted.

The four web pages the app may open in the browser for game databases (DAT-o-MATIC, Redump, the
CC BY-SA 4.0 deed and the repository) are a separate closed map, `DAT_LINKS`; the updater opens
two more through the same `openExternal`: the releases page and one release's page
(`RELEASES_PAGE`, `releasePageUrl` of a version the check verified as `X.Y.Z`). The browser
requests those, never Romperoom. All are https except Redump's download page, which is plain http because Redump serves
no https; the host allows http for that one exact URL and refuses any other non-https link.
The test reads the main process's sources as code: Electron's `shell` appears in exactly one
`shell.openExternal(…)` call and one `shell.showItemInFolder(…)` call (the updater's Show in
folder), and nowhere else (no alias, no destructuring, no other member; a comment never counts).

**Transport rules.**

- HTTPS with certificate checks pinned against the environment. Node's default for
  `rejectUnauthorized` follows `NODE_TLS_REJECT_UNAUTHORIZED` at every connect (measured: with
  it set to `0`, a self-signed certificate was accepted), so the options pass
  `rejectUnauthorized: true` explicitly, and `ca` is Node's bundled roots, so
  `NODE_EXTRA_CA_CERTS` is not trusted either. They never carry `checkServerIdentity` or
  `secureContext`. Certificate failures (`CERT_*`, `UNABLE_TO_*`, `HOSTNAME_MISMATCH`, …) and
  handshake failures are reported as `tls`.
- `agent: false`: a fresh agent per request, no pooling, no global or environment-configured
  agent, so `HTTPS_PROXY` and `NODE_USE_ENV_PROXY` are not used.
- DNS goes through our own `lookup`: the name must be allowlisted, and every address in the
  answer must lie outside every IANA special-purpose range, or the whole answer is refused
  (loopback, private, link-local, CGNAT, multicast, unspecified, documentation including
  `3fff::/20`, benchmarking, 6to4, Teredo, unique-local, SRv6 `5f00::/16`, IPv4-compatible
  `::/96`, every IPv4-mapped and IPv4-translated address, local-use NAT64 `64:ff9b:1::/48`, and
  zone ids; `main/dat-download/address.ts`, a `net.BlockList`). Well-known NAT64
  (`64:ff9b::/96`) stays allowed so IPv6-only networks with DNS64 work, but only when the IPv4
  address it carries is public. The socket connects only to checked addresses, and the address
  actually connected to is logged.
- Requests carry only `Host`, `User-Agent: Romperoom` (no version), `Accept`,
  `Accept-Encoding: identity` (byte caps stay exact), `Connection: close` and, for GitHub's API,
  `X-GitHub-Api-Version: 2022-11-28`. No cookie, `Authorization`, `Referer` or token. The only
  credentials Romperoom ever sends are ScreenScraper's two pairs, Romperoom's developer id and
  password and the player's user name and password, in that request's query as ScreenScraper's
  API requires: the request log and every error message take the host and path only, and a
  refused ScreenScraper URL is logged without its query.
- No redirect is followed, with one exception: a release file's first request (`github.com`) may
  answer one `3xx` whose `Location` passes the rule above; that hop is logged on its own, and any
  further `3xx` ends the download with reason `redirect`.
- Timeouts: 20 s to the response headers, 30 s without a byte, 300 s for a whole response (an
  hour for a release's update file, `UPDATE_TOTAL_MS`, one total shared by its redirect and the
  asset request). Each
  destroys the request with reason `timeout`.
- Caps: API answers at most 1 MiB, except thumbnail tree listings, at most 32 MiB
  (`MAX_TREE_JSON_BYTES`; a whole repository's recursive tree can pass 1 MiB).
- A release's `SHA256SUMS.txt` is at most 64 KiB and its `.sig` 1 KiB; a build that trusts no
  key asks for neither. The `.sig` is one or two lines (two during a key rotation, one per
  signing key), each a 64-byte signature in canonical base64, LF only, no blank or repeated
  line; it is accepted when at least one line verifies under a trusted key
  ([release.md](release.md#signed-checksums)). An update file must be exactly the size the release lists, at most
  512 MiB; it is created new (`wx`, owner-only 0600 on POSIX) under a random name in the data
  folder's `updates/`, its SHA-256 computed while streaming and compared with the verified list,
  and removed on any failure.
- The releases repository is fixed. If it were renamed or moved, GitHub would answer its
  downloads with a redirect from `github.com` to `github.com`, which the rule above refuses:
  updates then fail closed (the release page still opens) until a build names the new place.
- A game database file must be exactly its listed size: a `content-length` over it ends the
  request before the body, and a body that passes it is aborted at that byte. The git blob
  SHA-1 is computed while streaming and must match the listing; a file that fails any check is
  removed. A file that already exists is never overwritten.
- A cover art picture is read by `getBytes` into memory, at most 16 MiB (`MAX_ART_BYTES`, or
  less when the run's remaining total is smaller): a picture listed over the cap is not
  requested ("too large"); any other length than the listed one, declared or read, ends the
  request as changed, and reading stops at the listed size, so the cap holds on bytes read. Its
  git blob SHA-1 is computed while streaming and must match the listing, and the bytes must
  start with the PNG signature before the engine sees them.
- Every request takes an `AbortSignal`; cancel destroys the socket. No telemetry, no analytics,
  no update ping: the updater asks only when pressed, or once as Romperoom starts when the player
  turned that on.
- One rate reading. The transport keeps the `x-ratelimit-*` headers of the last `api.github.com`
  answer, whichever feature asked. GitHub allows 60 unauthenticated API requests an hour per
  address, shared by "Check game database updates", Get cover art and the updater's check (one
  request, pressed or as Romperoom starts), so a cover art review lists
  only as many consoles as that reading says fit, and says when the rest can continue. Pictures come
  from `raw.githubusercontent.com` and use none of it.

**Request log** (`main/dat-download/request-log.ts`). Every attempt, refused and cancelled ones
included: time, purpose (`check`, `download`, `art-listing`, `art-download`, `scrape-lookup`,
`scrape-download`, `update-check` or `update-download`), host, path (never a query), the
address connected to, outcome, HTTP status, bytes received, duration and failure reason. Never
headers, bodies or anything about the library. The newest 500 are kept in
`<dataDir>/network-log.json`, written to a temporary file and renamed; an unreadable log is
ignored with a warning.

**Static tests** (`test/security.test.ts`). One lists every import of `node:http`, `node:https`,
`node:http2`, `node:net`, `node:tls`, `node:dns` (with or without the `node:` prefix, static,
dynamic or `require`), `undici`, Electron's `net` named in an import or a destructured `require`
of `electron` or `electron/main`, and calls to Electron's `net.request` or `net.fetch` across
`apps/desktop/src` and `packages/*/src`, and compares them with an exact list: `transport.ts`
(`node:https`, `node:dns`, `node:tls` for the bundled roots), `address.ts` (`node:net`, for
`BlockList` and `isIP` only) and `index.ts` (the `net` import from `electron` and `net.fetch`,
the self-check's `app://` request). Type-only imports are not counted. Another requires
`rejectUnauthorized: true` and the bundled roots, forbids every other TLS option, and requires
`agent: false`. A third compares `buildCsp`, `hostResolverRules`,
`webRtcDeadProxy` and `WEBRTC_IP_POLICY` with the copy recorded at 0.4.0, byte for byte. A
fourth checks that `main/index.ts` calls `createDatTransport` once, with the request log as its
only option, and never names `testPort`, `allowLoopback` or `testCa`. The end-to-end tests' test
transport (`dat-download/fixture-transport.ts`, behind `ROMPEROOM_TEST_DAT_FIXTURES`) is loaded
only through a dynamic import when that unpackaged-only seam is set, and the package never ships
its chunk (`verify:package`). The
transport itself is tested over real loopback sockets with an injected resolver, including real
TLS handshakes against a self-signed certificate generated in the test: no test contacts
GitHub.

**Lint rules** (`eslint.config.js`, proven by `test/network-lint.test.ts`). Some APIs open a
socket without any import, so the import list cannot see them. In `apps/desktop/src/main`,
`apps/desktop/src/preload`, `packages/engine/src` and `packages/profiles/src`, ESLint refuses the
globals `fetch`, `WebSocket`, `EventSource` and `XMLHttpRequest` (bare or through `globalThis`,
`window` or `self`), every `.fetch()` call (`session.fetch`, `net.fetch`, …), every
`downloadURL`, and Electron's `net` when it is imported from `electron` or `electron/main` under
any local name, destructured from any object (`const { net: n } = require('electron')`), or read
as a `.net` property. A computed read (`e['net']`) is not caught. The one existing use,
the self-check's `app://` fetch in `main/index.ts`, carries a reviewed `eslint-disable` on the
import and on the call. The renderer is outside this list: the CSP's `connect-src 'none'`
guards it, and its probes run there.

**Why not Electron's `net`.** Chromium's stack would bring the OS certificate store and make
the resolver rules and the dead proxy a second allowlist. But each host would have to be
excluded from the process-wide `--host-resolver-rules` switch, which also lets the page's
resolver look those names up, and the switch is set before `ready`, so the exclusion would
hold for every launch, used or not. `net` also needs Electron, so the transport could be
tested only end to end. With `node:https` the renderer's guards stay byte-identical, Node
never follows a redirect on its own, and the custom `lookup` checks the host and every
address before connecting. The cost: no system proxy and no private certificate authority
for the download (the official-site path still works).

## Data at rest

- **The catalog** is `catalog.sqlite` in the app data folder (see
  [configuration.md](configuration.md#data-folder)). It is never kept on the NAS. It holds
  library paths, relative file paths, sizes, modification times, CRC32/MD5/SHA-1 hashes, parsed
  titles, folder mappings and scan state, and descriptions a ScreenScraper lookup saved. It is
  not encrypted. It holds no credentials.
- **The ScreenScraper account** the player saves is encrypted with Electron's `safeStorage` (the
  macOS keychain, Windows DPAPI, or a Linux secret service) and written as ciphertext to
  `screenscraper-account.json` in the data folder (mode 0600). Where `safeStorage` has no real
  keychain (on Linux, any backend but a named keyring, such as `basic_text`) nothing is saved. The
  page sends it once and only ever learns whether one is saved; it is decrypted in the main process
  only to build a lookup's request. Whether one is saved is read from the file without decrypting
  it, so opening Fix up or Settings never asks the keychain; an account the keychain can no longer
  read (a reset, or a data folder from another computer) stops a lookup before any request, asking
  for it again. Reading the account for a lookup, saving it and forgetting it run one after another
  in the order asked (a queue), so when the keychain asks for the account to be encrypted again
  during a lookup's read, that write can never bring back an account the player forgot, or overwrite
  one saved, while it ran: the file always ends as the last Save or Forget left it. A lookup already
  running keeps the account it read when it started and goes on using it for its requests; a Forget
  or Save affects the next lookup. Whether one is saved is not queued (it never decrypts), so it
  never waits behind a keychain prompt. Romperoom's own ScreenScraper developer details are not in
  the repository or the build: an unpackaged run may read them from `ROMPEROOM_SCREENSCRAPER_DEV`
  (see [configuration.md](configuration.md#environment-variables)), and they come from the
  maintainer's cloud secret store.
- **Settings** (theme and light or dark) live in the renderer's localStorage, under the key
  `romperoom.settings.v1`, the card wizard's last choices under `romperoom.deploy.v1`, and its
  saved packages under `romperoom.deploy.packages.v1` (see
  [configuration.md](configuration.md#saved-settings)). Neither wizard key holds a path; a
  package's choices are validated again by the engine's schema when they are planned.
- **Electron's own user data** (Chromium caches and local storage) shares the data folder.
  Chromium's cookie store is not encrypted (the `EnableCookieEncryption` fuse is off): the app
  sets no cookies and keeps no secrets there.
- **Nothing is written to the library** by a scan. Romperoom's own media store
  (`.romperoom/media`) gains pictures only from cover art, when the player confirms a review (see
  [Filesystem safety](#filesystem-safety)); each picture it adds is recorded in the catalog
  (`art_added`, with its size, SHA-1, source and pin, and `art_dir` for the folders it made).
  Standardise and re-link rename pictures there, as anywhere in the library, only as their review
  offered, and keep each replaced game list, cue sheet or playlist in `.romperoom/lists-backup`
  (`standardise_run`, `standardise_backup`).
  Sync a card writes the games and saves the player approves into console folders,
  `.romperoom/saves` and `.romperoom/saves-backup`, through one journal (see
  [Filesystem safety](#filesystem-safety)); the catalog keeps its card ids, save bases and card
  game hashes (`card`, `card_save`, `card_rom_hash`, `card_sync`).
- **Logs** go to the terminal (stdout and stderr). Messages may hold library paths. They never
  hold file contents. The one log file is `network-log.json` (below).
- **Game database files** (see [configuration.md](configuration.md#data-folder)):
  `network-log.json` keeps the newest 500 request attempts (time, purpose, host, path, address,
  outcome, status, bytes, duration, reason; never headers, bodies or library data).
  `libretro-pin.json` holds the listing "Check game database updates" last stored. `downloads/`
  holds a file only while it is downloaded and verified, and is emptied at every start.
  `art-listings/` keeps each console's last thumbnail listing (names, sizes, git SHAs and the
  tree SHA), or GitHub's 404 for it in its own strict shape, for 24 hours, one file per
  repository named only from the shipped list. Transient failures are never kept, and a file in
  neither shape is ignored with a warning.
  `updates/` (owner-only, 0700, on macOS and Linux; on Windows it has the data folder's access
  list) holds an update file only while it is downloaded and verified, then the verified file
  the app showed, or a Windows installer and its `pending.json` until the next start (removed
  there once the new version runs). It is emptied before each download and at every start.
  Before anything is done in it, it must be a real folder whose real path is the data folder's
  own `updates`: a file, a symbolic link or a Windows junction in its place is removed itself
  (never followed) and a fresh folder made, and the sweep removes each entry by its own name,
  never following a link. The start-up sweep never stops Romperoom starting: an entry it cannot
  remove (a Windows installer still running) is logged by name and left. `pending.json` is
  trusted only in a folder private to this user by the install's own rule (`isPrivateFolder`),
  judged before the folder is tightened (Romperoom never writes a marker into a folder that is
  not private, so one found there is someone else's, and it is removed with everything beside
  it), and only exactly as written: three keys, the installer named for its version, a regular
  file no larger than an update may be (512 MiB, checked before it is read), whose streamed
  SHA-256 matches; anything else is ignored.
- **The installer's last moment.** The file that runs is the one this process created (`wx`,
  random name, then renamed into place); it is hashed against the signed list after the
  download and again immediately before the spawn. What remains is the instant between that
  last hash and the operating system opening the file, with no handle held across it: only a
  process of the same user could change the file then (the folder is private, else nothing
  runs), and such a process could change the installed app as easily.

## Filesystem safety

- **Scans are read-only.** The walker lists folders and stats files. The hasher opens files
  read-only in worker threads. Zips are read with `yauzl` without being extracted, with a size
  cap ("Too big to check inside the zip"). Symlinks are reported, not followed. AppleDouble
  `._*` files, `.DS_Store` and similar junk are ignored.
- **A folder that looks empty or gone is not forgotten.** That can be an unmounted share or a
  half-connected drive. The scan keeps the folder's games and asks the player first (Fix up:
  "Some game folders look empty or gone"). A whole library that looks empty is never marked
  missing. The guards are in [architecture.md](architecture.md#scanning-and-its-safety-guards).
- **Operations never overwrite.** Every rename goes
  through an exclusive `O_CREAT | O_EXCL` name reservation, with its device and inode recorded.
  Cross-volume moves copy to a temporary name, verify the hash, rename into place and only then
  remove the source. "Delete" means moving into `<library>/.romperoom-quarantine`. Every step is
  journaled before anything moves, and every journal can be undone.
- **A journal only touches the library it ran in.** It records the library folder's device,
  inode, real path and a fingerprint of its top-level folder names. An empty mountpoint or
  another folder at the same path blocks finish, rollback and undo; nothing is marked failed or
  missing (see [architecture.md](architecture.md#operations-and-the-journal)). Reconnecting a
  library that was remounted or moved (`tidy:reconnectLibrary`, or a complete scan by itself)
  never accepts an empty folder or one sharing none of the recorded top-level folders, needs
  the recorded real path or 80% of the folders the same, and is audited in `library_reconnect`
  ([decision 37](decisions.md#37-a-library-that-moved-is-reconnected-by-the-user-or-by-a-scan-that-proves-it)).
- **One job per library.** A scan, an operation and a purge never run at the same time on one
  library; two deploys may read it together
  ([lock matrix](architecture.md#one-work-lock-per-library)).
- **The last copy stays.** A duplicate or artwork cleanup step re-hashes the copy it keeps just
  before it moves the other one, and fails if that copy is gone or changed, reached through a
  link, or the same file (device and inode) as the one to move. A purge checks the kept copy
  again before deleting its extra.
- **Cover art writes only under `.romperoom/media` and `.romperoom/tmp`**
  ([decision 41](decisions.md#41-a-library-gains-one-writer-outside-tidy-up),
  `packages/engine/src/art/writer.ts`). The rules:
  - Only the engine writes; the host and the page pass no path. A picture's place is
    `.romperoom/media/<system>/<box|screenshot|title>/<name>`, where `<name>` is built by the
    engine from the game's own ROM file stem and checked: one segment, no `..`, no separator or
    control character, `.png` (or `.jpg` from a card), at most 255 bytes. A game whose name
    would not pass is never offered.
  - Never through a link: every component of `.romperoom/media` or `.romperoom/tmp`, and of the
    target's own folders, must be a real folder to `lstat` whose real path lies inside the
    library's real path (`isRealFolderChain`), or nothing is written.
  - Never over a file: the run holds the library's `art` work lock (no scan, Tidy up, identify
    or deploy runs alongside it); each picture goes to a random part file in `.romperoom/tmp`,
    is fsynced and closed, then hard-linked into place (an exclusive create), or, where the file
    system has no hard links (SMB shares, FAT, exFAT), renamed into place after `lstat` finds the
    target absent. It is never copied in place, so no half-written picture is visible at its
    name. A target that exists is counted "already present".
  - Remove downloaded art deletes only what cover art recorded (`art_added`): a file at exactly
    a shape the writer makes, whose size and SHA-1 still match, through the same real-folder
    check; it then removes only the empty folders it recorded making (`art_dir`). Anything else
    is left and counted.
  - Two windows remain (see [Gaps](#gaps)): a folder swapped for a link between the check and
    the write, and, without hard links, a file another program drops at the target between the
    absence check and the rename.
- **Import art from an SD card reads, then saves.** It reads only the card the host listed and
  the review was made from, through the device profile's art folders. Each reviewed picture is
  read again at import as a regular file (no final link, no FIFO), at most 16 MiB, below folders
  checked to be a real chain inside the reviewed card, and skipped ("couldn't read this picture
  from the card") unless its size and format still match the review. The check and the read are
  two steps (see [Gaps](#gaps)).

- **Sync a card writes only what the review offered**
  ([decision 42](decisions.md#42-card-sync-writes-into-a-library-and-onto-a-card),
  `packages/engine/src/sync/`). The rules:
  - Only the engine writes. The page passes no path, only ids from the review; the host passes
    only the mount path of the card it listed.
  - The host syncs only a card from its own listing that every deploy safe-target rule allows
    (not a system disk, not read-only, not a network drive, not a fixed disk, not a protected
    folder such as the home or temp folder, not holding or inside a library),
    checked at the review and again before the run, which also needs the same mount. A review's
    plan id works only for the window it was given to; one sync job runs at a time.
  - Into the library: card bytes go to `.romperoom/tmp/sync-<uuid>.part` (exclusive create,
    fsync, close) and are checked against the SHA-1 the review measured; then one Tidy up journal
    moves each into its console folder or `.romperoom/saves/<system>/`, never over a file. A
    library save being replaced is first moved to `.romperoom/saves-backup/<system>/`. Every
    folder on the way is a real folder chain inside the library. The run holds the library's
    `sync` lock.
  - On the card: only `.romperoom/card.json` (`{ "v": 1, "id": "<random uuid>" }`, at most
    4 KiB, read with a strict schema) and saves in the profile's save folders. Each is written
    first to a part file beside it (`<name>.romperoom-part`, exclusive create, fsync, close); a
    regular file already holding that name is Romperoom's own leftover and is removed first,
    anything else there refuses the write. A new save goes into place by an exclusive hard link,
    or, on a card without hard links (FAT, exFAT), by the no-clobber rename: neither ever
    replaces a file, so a save the player made meanwhile wins. A card save is replaced only
    after its bytes are in the library's `.romperoom/saves-backup` folder (not journaled, so no
    undo or rollback can move that copy), and only while it still holds what the review saw;
    the part file is then renamed over it. The windows these checks leave are in
    [Gaps](#gaps).
  - Reading the card: links, FIFOs, devices and sockets are refused before anything is opened;
    names over 255 bytes or with control characters and files over the caps (games 4 GiB,
    saves 32 MiB) are left and counted; a file is opened without following a final link and
    without blocking, below folders checked to be a real chain inside the card.
  - Nothing of the player's is deleted on either side. Undo this sync reverses the journal where
    files still hold what the sync wrote: it puts back the saves it replaced, and removes the
    copies of card games and the new saves it added to `.romperoom/saves` that are unchanged
    (they come back into `.romperoom/tmp` first); card writes stay, their earlier bytes in the
    backup folder.
- **Standardise writes only what the review offered, inside the library**
  ([decision 43](decisions.md#43-standardise-renames-folders-and-games-and-edits-two-kinds-of-files-other-programs-own),
  `packages/engine/src/standardise/`). The rules:
  - Only the engine writes. The page passes a library id, a profile id and ids from the review;
    the host passes no path at all.
  - The run holds the library's `op` lock, the one Tidy up takes, so no scan, Tidy up, identify,
    cover art, card sync or copy to a card runs alongside it.
  - Every rename is a step of Tidy up's journal and never lands over a file: a console folder or
    a game that is a folder moves whole (a case-only rename through `<folder>.romperoom-case`),
    and a file moves through the same name reservation as every operation. Each step checks the
    real-folder chain of its folders just before it runs, and fails rather than follow a link.
  - A game list, cue sheet or playlist changes only where it names a renamed file. Its new text
    is staged in `.romperoom/tmp`, the original is moved to `.romperoom/lists-backup/<run>/`,
    then the new text moves into place; Undo puts the original back only while the file still
    holds what the run wrote.
  - Nothing is deleted. An identical copy a merge meets is set aside in `.romperoom-quarantine`;
    a merged folder left empty stays where it is.
- **Re-link writes only what its review offered, inside the library**
  ([decision 44](decisions.md#44-re-link-renames-a-leftover-picture-after-the-one-game-that-clearly-matches-it),
  `packages/engine/src/tidy/relink.ts`, `tidy/relink-run.ts`). The page passes a library id and
  ids from the review. The run holds the library's `op` lock; each picture is renamed in its own
  folder by one step of Tidy up's journal, through the same name reservation and real-folder
  checks as Standardise, never over a file; a `gamelist.xml` changes only in the renamed pictures'
  media references and the chosen entries' `<path>`, staged in `.romperoom/tmp` with the original
  moved to `.romperoom/lists-backup/<run>/`. Nothing is deleted, and no entry is ever removed.

## Emptying the quarantine

Emptying the quarantine (`purge` on the tidy facade) is the only code in Romperoom that deletes a
user's file. Everything else moves files into quarantine and can be undone. Removing cover art
and Undo this sync delete only files Romperoom itself added, still unchanged since; Standardise
deletes only its own staged text (`std-*.part` in `.romperoom/tmp`, the rewritten game lists, cue
sheets and playlists, including those Undo takes back there), once no journal needs it.

**What it defends against**

| Threat                                                  | Defence                                                                            |
| ------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| A click or a script deletes by accident                 | A preview first; then the words `DELETE FOREVER` and the exact byte count          |
| A caller names a file to delete                         | The facade takes preview ids only, never paths; ids are random, single use         |
| The library changed after the preview                   | The id is bound to the catalog generation (`plan-stale`); every file is re-checked |
| A file was put in the quarantine folder by someone else | Only files a journal moved there are candidates                                    |
| A quarantined file was replaced or edited               | Its SHA1 must equal the one recorded when it was moved                             |
| A link or folder planted in the quarantine              | `lstat` must say a regular file; links are never followed                          |
| A parent folder swapped for a link to elsewhere         | The file's real path must be the real quarantine path plus its relative path       |
| The quarantine folder itself replaced by a link         | `lstat` must say a real folder whose real path is inside the library's real path   |
| A journal row pointing outside the quarantine folder    | Refused by path before anything else ("not inside the quarantine folder")          |
| The kept copy of a duplicate is gone, changed or linked | The extra is kept ("its kept copy is missing" or "has changed")                     |
| A restore blocked, then the file deleted by the purge   | Kept unless the user asks with `includeUndoBlocked`                                |
| The drive was unplugged, another folder is at the path  | The library's identity is checked before anything; an empty folder never passes    |
| Two jobs at once                                        | The library's work lock (`op`) is held for the whole purge                         |
| A crash mid-purge                                       | Every item is journaled `pending` first; `resolvePurge` settles it                 |

**Invariants**

- A file is deleted only if, just before its `unlink`: it is inside
  `<library>/.romperoom-quarantine` by path and by real path, with no link on the way; the
  quarantine folder is a real folder in the library; a journal recorded moving it there and it
  is still `done` (or `undo-blocked`, when asked); `lstat` says a regular file; its SHA1
  matches the journal's; and, for a duplicate, its kept copy is still there, unlinked and
  unchanged.
- A file that fails any check is kept and reported with the reason. Nothing else is deleted.
- Only empty folders are removed (`rmdir`), up to the quarantine folder itself.
- A purge is an `irreversible` operation; undo refuses it. The quarantine steps it deleted
  become `purged`, so an undo of the tidy run reports them instead of looking for them.
- A purge interrupted between a deletion and its record is settled by `resolvePurge('finish')`
  only after the library's identity is proven again: a file missing from a library that is not
  there is never recorded as deleted. Until it is settled, restore and undo of its files are
  refused; an item restored anyway is kept.

**Not defended against:** a process with the user's permissions that edits a quarantined file
and its journal row in the catalog together, and a file replaced between its final hash and its
`unlink` (the window is one system call).
- **Device profiles cannot point outside the card.** Every path is relative, and segments are
  checked for FAT32, exFAT and NTFS portability (see [profiles.md](profiles.md#path-safety)).
- **Names are shown, never interpreted.** Display text has control and invisible characters
  removed (`DisplayName`). Bidi overrides cannot reorder the rest of a row.

## Deploy containment

A device package is planned from names, and written later to a real card that may have changed
in between. The planner ([architecture.md](architecture.md#deploy-planner)) and the writer each
check it.

- **The planner cleans every destination.** Library names can hold what a card refuses or
  silently changes: Windows drops a trailing dot or space, so `Game.` and `Game` would be one
  file; FAT refuses `<>:"|?*` and `\`; a case-insensitive card merges `Game` and `GAME`; an RTL
  override can make `gpj.exe` look like `exe.jpg`. Every segment is rewritten to a form every
  card stores as written, and paths are claimed case- and Unicode-folded. A BIOS file whose name
  would change is left out, not renamed.
- **`verifyPlanPaths` checks again at write time.** Each destination must again obey the name
  rules and the profile's path length, must join to a path inside the card root, and must stay
  inside it after resolving every link that exists on the card now (the deepest existing
  ancestor is resolved with `realpath`). A folder it cannot look into is reported as "could not
  check", never as fine. The card writer calls it and writes nothing when it returns a
  problem.
- **Sources are only what the catalog confirmed.** A BIOS folder is a library setting: an
  existing absolute directory, stored as its realpath, never a filesystem root. Its listing is
  non-recursive, takes regular files only (never through a link), and is capped at 5000 entries
  and 2 GiB copied. A BIOS folder that cannot be read blocks the plan rather than reading as
  empty.

The card writer ([architecture.md](architecture.md#card-writer)) adds its own rules:

- **No format, erase or partition code exists.** The writer only creates, renames and removes
  files and folders under the card root. A test (`packages/engine/test/deploy-guards.test.ts`)
  scans the engine source and the desktop main and preload code for disk-destroying commands and
  APIs (`diskutil eraseDisk`, `mkfs`, `newfs`, `diskpart`, `Format-Volume`, `dd if=` and
  others); its positive control proves each pattern matches.
- **One file runs programs, and only listing ones** (and the app updater, below).
  `deploy/volumes.ts` is the only engine source that imports `child_process`. It runs `diskutil` (by absolute path), PowerShell with a fixed script,
  `lsblk` and `findmnt`, through `execFile` with `shell: false`, fixed arguments, a timeout and
  an output cap. `deploy/folder-network.ts` runs `/sbin/mount` with no arguments through the same
  function; a folder's path is matched against the output, never passed to a command. No argument
  comes from the page. The test pins both lists. The only other importer is the app updater's
  `src/main/updates/install.ts`: one `spawn`, with `shell: false`, which starts a downloaded
  Windows installer detached only after its signed checksum and size verified and it was hashed
  again where it runs from. The test reads every source of the app and its packages as code
  (the TypeScript parser, so comments never count): exactly these two files name
  `child_process`, each once, in one static import of `execFile` and of `spawn` respectively;
  nothing else reaches it (an import without `node:`, `import()`, `require`, or a
  `createRequire` used to load); `spawn` is used once, called directly, with an options object
  written in place saying `shell: false` and `detached: true` and no spread. Each of those
  bypasses is fed to the guard in the test and must fail it. The guard is about
  `child_process`; Electron's `utilityProcess`, `shell.openPath` and Node's `process.execve`
  also start programs, and none is used anywhere: the guard fails if the updater's code
  (`src/main/updates/`, `updates-host.ts`) names any of them, with a bypass row for each.
- **A listing that fails is not an empty list.** It returns `ok: false` with the reason, and a
  deploy to a volume is then refused. Output that cannot be parsed is refused too; the plist
  reader refuses entity declarations and caps size, depth and nodes.
- **Refused targets:** the system disk and system volumes, read-only and (unconfirmed) network
  volumes, fixed disks without the typed volume name, protected paths, too little room, and any
  volume or export folder that holds or lies inside a library root, a BIOS folder or the app's
  data folder. All are compared after `realpath`. An export folder must already exist and be
  empty or an earlier export, and one on a network drive, or whose drive cannot be told, is
  written only when the caller confirmed exactly that folder's path (`confirmNetwork`). On Linux
  a card is read-only when its mount options hold `ro`, or when its mount point is missing from
  the mount table.
- **Only its own files change.** The manifest says which files the writer owns; any other file
  is never replaced or removed, except moved aside on request (`overwriteUnmanaged`). Every
  card path goes through `resolveInside`, which refuses `..`, empty segments, backslashes,
  colons, NUL and anything that joins outside the root. A fuzz test writes 200 hostile plans
  and checks nothing landed outside the card. A planned file outside the profile's folders
  refuses the whole plan, with or without `overwriteUnmanaged`; for ROMs at the card root (the
  R4xMP) those folders are the system folders, never the root, so its kernel (`__rpg/`,
  `_DS_MENU.DAT`, `_DSMENU.DAT`) is never named by a write, a move or a removal; likewise the
  DSpico's launcher (`_picoboot.nds`, `_pico/`) beside its `roms` folder. The card root still
  gets the writer's own `romperoom-manifest.json` and `.romperoom-removed/`.
- **No step goes through a link.** Writing makes or checks each folder of a path with `lstat`
  and refuses one that is a link or a file (`ensureDirs`). Removing does the same before any
  move aside, erase, stale part file, leftover reservation or empty folder: a file whose folder
  on the card is a link or a file is left where it is, kept in the manifest, and listed as a
  conflict. On FAT32 and exFAT cards no link can exist, so this matters for export folders and
  cards formatted APFS, HFS+, NTFS or ext4: there, a folder the player swapped for a link (to
  the library, or to the card root) can no longer lead a removal outside the profile's folders
  or off the card. A link made between the check and the step (a race on a card being changed
  while Romperoom runs) is not covered.
- **No half-written file under a final name.** Copies go to `.romperoom-part` files and are
  renamed into place without clobbering, then read back. Removal is a move to
  `.romperoom-removed/` on the card; erasing needs `eraseRemoved` and a fresh matching hash.
- **A crash leaves nothing a later run must refuse.** Before a new file's empty no-clobber
  reservation exists, its name is listed in the manifest's `reserved` list, with the
  reservation's device and inode once known. The next run removes an empty regular file at
  that name only when it matches that entry (identity when recorded), the manifest does not own
  it, and it lies inside the profile's folders. An empty file of the user's at an unlisted name,
  or one that is not the recorded reservation, stays a conflict.
- **The manifest is input, not instructions.** `pending` and `dirs` entries outside the
  profile's folders are ignored: no part file is unlinked and no folder removed there. A
  temporary manifest that is blank or all NUL (a crash mid-write on FAT) counts as absent.
- **After the first change to the card, a deploy always returns its report.** Any unexpected
  error from then on (a manifest save failing with `EIO` or `EROFS`, the root vanishing) ends
  the run as `card-error`, after one last try to save the manifest, with the counts, failures
  and bytes so far. 25 card errors in a row stop the run the same way. Only checks made before
  anything is written may refuse or throw.
- **Guarded folders are compared where they really are.** An export folder is checked against
  the library roots, the BIOS folders and the app's data folder after resolving links on both
  sides (a folder that does not exist yet is resolved through its deepest existing parent). A
  guarded folder that cannot be resolved refuses the export: "could not check" is never "not
  inside".
- **Removable means the OS says removable.** On Windows only `DriveType = Removable` counts; a
  USB hard disk or SSD is external but fixed, so it needs the typed name like on macOS. Two
  volumes listed with the same id refuse the listing, so an id never names the wrong drive.
- **Another device's files wait for consent.** When the manifest was written for a different
  profile, nothing it owns is replaced, moved or erased unless the start request carries
  `confirmProfileSwitch: true`, which the wizard sends only when the user turns the switch on.
- **A frontend's game list is added to, never replaced.** An owned `gamelist.xml` the frontend
  changed keeps its text; the plan's missing `<game>` entries are added before its last
  `</gameList>`. The card's text is read with a 64 MiB cap, as strict UTF-8, in one pass. A file
  that is not a game list it can read stays a conflict.
- **Zipped games unzipped on copy are untrusted input** (`deploy/unzip.ts`, for a system with
  `extractArchives`: R4xMP, DSpico). No name inside a zip is ever a path: the card file is named
  after the zip itself (its stem, plus the extension of the one ROM chosen, which is one of the
  profile's `[a-z0-9]` extensions), then cleaned like any destination, so an entry named
  `../../x.nds`, `/etc/x.nds` or `C:\x.nds` only tells which entry is the game (zip-slip cannot
  happen). A name is decoded only to tell what an entry is (its extension, or that a computer
  added it: `__MACOSX/`, `._`, `.DS_Store`), and is never kept; yauzl's own name checks are off
  because nothing is extracted by name. The planner reads only the central directory (at most 10,000 entries and a
  16 MiB directory; zip64 fields read with bounds checks; anything else malformed is "damaged")
  and refuses an entry that is locked, not stored or deflated, stored with two different sizes,
  whose compressed bytes end past the zip, that claims more than 1,100 bytes per compressed byte
  (more than deflate can reach: a size that lies, as in a zip bomb), or more than 8 GiB, and
  the card's file limit (FAT32's 4 GiB - 1 byte) is weighed against the unzipped size. The writer
  never trusts the plan's listing either: it finds the entry again by where its local header
  starts (refusing one that is gone, listed twice, locked, or with other sizes, CRC-32 or
  method), streams it with
  yauzl's size validation and its own byte count, stops past the declared size, checks the
  CRC-32 at the end, and only then renames the part file into place; the SHA-1 is checked
  against the catalog when the catalog's hash is of that entry, and the file is read back as any
  copy is. A zip that fails fails that one file. At most one entry is ever unzipped, and only into
  the card's part file: nothing is unzipped into memory or into the library, and the library's
  zip is only read. Memory and time are bounded too: only a fixed-size verdict per zip is
  remembered (at most 50,000), never a zip's entries, so a hostile pack with huge directories
  costs one directory read at a time, and each plan reads zips for at most about 2 s on the main
  process before leaving the rest "not checked yet".
- **Pictures converted to PNG are untrusted input** (`image/convert.ts`, `image/to-png.ts`,
  `deploy/convert.ts`, decision 67). A JPEG or GIF from the library (scraped, downloaded or copied
  from anywhere) is decoded only to make the card's PNG, and only in the **picture worker**, a
  worker thread, never on the main process. What is bounded, and how:
  - **Size, before anything is allocated:** the pixel count from the header, at most 2048 × 2048
    (4,194,304 pixels, any shape); a 20000 × 20000 header is refused without jpeg-js ever being
    called (tested with a spy on it).
  - **A JPEG's structure, before decoding:** the whole file is walked (each segment by its length,
    each scan's data to its end: linear, nothing decoded). Refused: more than 100 scans or 1,024
    segments, a scan order the standard forbids (libjpeg's rule: a progressive scan refines a
    coefficient only one bit below its last scan, so no coefficient is decoded more than 14
    times; a sequential scan covers its components once), no image data, or less than one bit of
    data per block (a picture claiming 2048 × 2048 pixels in a few hundred bytes). This is what
    stops the review's "scan bomb": 100 refinement scans of 22 bytes each, 2.3 KB, which made
    jpeg-js work for 1.1 s (3,000 of them, 66 KB, froze the app for 26 s), is refused in well
    under a millisecond. The plan does the same walk, so such a picture is listed as "left out"
    with its reason before any copy. CMYK, 12-bit, lossless and arithmetic-coded JPEGs are refused
    by their headers. jpeg-js also gets its own caps (`maxResolutionInMP` 4.19,
    `maxMemoryUsageInMB` 160, tolerant decoding off).
  - **Time, by a hard limit:** each picture has 10 s in the worker; past that the worker is
    terminated and that picture fails ("took too long: not converted"; nothing written, the run
    goes on with the next picture, which starts a new worker). Cancel terminates the worker at
    once. The main process only sends bytes and waits: its event loop stays free while a picture
    converts (tested: the loop's lag stays a fraction of the conversion's time). A worker that
    dies fails its picture; three failures in a row with nothing converted (a worker that cannot
    start) and no further worker is started in that run.
  - **Cost of a legal picture at the cap:** about 1.2 s and about 390 MB of memory at its peak
    (a 2048 × 2048 progressive JPEG of noise, 12 MB: decode 0.56 s, PNG encode 0.6 s, measured on
    an M-series Mac); a runner or laptop about seven times slower takes 7 to 8 s, inside the
    limit. The memory is the worker's, in the app's process: the heap limit of the worker (256 MB)
    does not cover typed arrays, so the pixel cap and jpeg-js's memory cap are what bound it.
  - **GIF** is read by Romperoom's own reader, not omggif: fuzzing found a damaged LZW stream that
    makes omggif 1.0.10 loop for ever. Romperoom's LZW reader refuses any code that names no entry
    yet, so every string ends within 4096 steps, and every code consumes at least three bits of
    data, so the work is bounded by the file. Only the first frame is read.
  - **Containment:** every failure of either decoder, whatever it throws, is a refusal of that
    picture (`convertToPng` never throws), listed with its reason. The fuzz test (truncated,
    overwritten and garbage-filled JPEGs and GIFs, a fixed seed) checks that each mutant either
    decodes or throws an Error (never anything else, never a crash), that some of them do fail,
    and that none takes a second; many damaged JPEGs still decode (to a damaged-looking picture).
  - **The library and the card:** the source is read whole, at most 16 MiB, through a read-only
    open; the card name is the ROM's stem plus `.png` (never anything from inside the picture); the
    PNG goes through the usual part file, no-clobber rename, read-back and manifest, and is never
    larger than the plan counted. A converted PNG is never adopted, so a file at its name the
    player put there stays theirs. A PNG converted earlier is never removed because its picture
    could not be checked this time (unreadable, changed since the scan, not checked yet): it is
    kept and stays recorded.
- **`ROMPEROOM_TEST_VOLUME`** lists a folder as the only volume, for tests. A packaged build
  ignores it ([configuration.md](configuration.md#environment-variables)).

## Tidying up

Fix up's tidy tasks (`src/renderer/tidy/`, shown by `src/renderer/fixup/`) reach the engine's tidy
facade only through the tidy host (`src/main/tidy-host.ts`). The renderer is untrusted, so the host
follows the deploy host's pattern.

- **No paths in.** The page sends a library id, a file, media or set-aside item id from a list
  the host gave it, or a random id the host handed to that same window. Every field is checked
  (unknown fields, wrong types, page sizes over 200 and selections over 100,000 are refused), and
  the engine validates again with its own schemas.
- **No paths out.** Every path shown is library-relative; anything absolute is cut to its last
  name in either separator style (`shownPath`). Libraries are named by their folder's name, with
  a number when two share it (`roms`, `roms (2)`).
- **Window-bound ids.** Duplicate answers, plans and delete-forever previews are held under
  random host ids, bound to the window that asked, at most 6 per window and kind, for 30 minutes,
  and forgotten when the window navigates. The engine id behind one never reaches the page.
- **Stale plans are refused.** The engine binds each plan to the catalog generation: after any
  catalog change (a scan, another tidy run, a checksum Tidy up writes; not an art checksum the card
  writer fills in) the id is refused with
  `TidyPlanError` and the page offers **Refresh list**. Paging re-asks the engine and refuses when
  the totals changed, so one plan never mixes two answers.
- **One job at a time.** Apply, undo, restore, delete forever and recovery share one slot, taken
  before any wait. The engine's per-library work lock also refuses while a scan or a copy to a
  card runs (`LibraryBusyError`).
- **The delete-forever check is the engine's.** The page sends the typed words and the size it
  showed; the host passes both through unchanged and the engine compares them with
  `PURGE_CONFIRM_TOKEN` and the preview's bytes. See
  [Emptying the quarantine](#emptying-the-quarantine) for what is checked per file.
- **Quitting.** While a tidy job runs, quitting asks first ("Stop tidying and quit?", or "Quit
  while files are being deleted?"); confirming cancels the run between files and waits up to 10
  seconds for it to stop.
- **Merge picture folders** (decision 66). `tidy:mergeArt` reads only and returns names (a
  console folder, a file name in `images` or `Imgs`), never a path; its look is held under a
  window-bound id like a plan. `tidy:mergeArtRun` takes that id only. The engine takes the
  library's `op` lock (all or none, never waiting), reads the folders again with every content
  hash and refuses (`TidyPlanError`, nothing moved) if anything shown differs or the catalog
  changed (junk files written meanwhile left out). Which name reaches which file is asked of the
  file system (lstat, then device and inode), never decided by a string fold, so a FAT32 or exFAT
  card never has another game's picture set aside for a name it keeps apart. Every game list in
  the console folder is read and one that can't be read fails closed. Only regular files with one
  name, of the scanner's picture types, move, and only from
  `<console>/Imgs` to `<console>/images` or into the library's own `.romperoom-quarantine`; a
  linked `Imgs` or `images`, a console folder reached through a link, a link, a folder, a hard link
  or any other file is never moved or followed. Each step is a Tidy up journal step: it re-hashes
  its file against the hash read under the lock and refuses a link on the way just before it
  moves, never overwrites (`renameNoClobber`), and a set-aside of an `Imgs` copy is guarded by the
  copy kept in `images` (the last-copy guard). `Imgs` is removed with `rmdir` (which refuses a
  folder that is not empty) only after it is read again empty, and only when the run emptied it.
- **Recovery.** A crash mid-run leaves its journal `running`. At startup the page shows what was
  interrupted (`tidy:recoverList`, names only) and offers to finish it or undo what was done
  (`tidy:resolveJournal`). Neither runs until the library's identity is proven again; a library
  that is not there stays blocked. When the folder at the path no longer looks like the library,
  the page asks "Is this still your library?" and only a Yes calls `tidy:reconnectLibrary`
  (`{ confirm: true }`, a known library id; never a path). The engine still refuses an empty
  folder or another drive whatever the page says.

## Junk files

Fix up's **Junk files** and the SD card's **Clean up this card** remove Finder and Windows
metadata for good (decision 65). The code is `packages/engine/src/junk/`. What may be removed:

| Rule                                                | Checked how                                                                                                                                          |
| --------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| Only `._*`, `.DS_Store` and `Thumbs.db`             | The name (`junkKindOfName`; the last two in any case, as case-insensitive volumes see them); the manifest and every kernel or launcher file fail it  |
| Only metadata, by its contents                      | AppleDouble: magic, version, a sane entry list, no data fork, a resource fork no longer than macOS's empty one (286 bytes), no other large entry     |
|                                                     | `.DS_Store`: `00 00 00 01 Bud1` and a root block whose two offsets agree and lie inside the file; `Thumbs.db`: the OLE signature, version, byte order and sector size |
| Never larger than that kind of metadata             | 1 MiB (AppleDouble), 16 MiB (`.DS_Store`), 64 MiB (`Thumbs.db`), before anything is read                                                            |
| Only a regular file with one name                   | `lstat`: a link, a folder or a special file stays; `nlink > 1` (a hard link) stays (see below for SMB)                                               |
| Never through a link, never outside the place       | Each folder on the way `lstat`ed as a real folder; the folder's real path inside the root's real path; then rename-and-verify (below)               |
| Never Romperoom's own folders or a system folder    | No segment `.romperoom*`, macOS's (`.Spotlight-V100`, `.Trashes`, `.fseventsd`…), a Linux `.Trash-<uid>`, a NAS's or Windows' (`@eaDir`, `$RECYCLE.BIN`…) |
| Never another drive mounted inside                  | A folder on another device than the root is listed and never walked                                                                                  |
| Still the file the list showed                      | Device, inode, size and time from the list, read again right before; the opened file (`O_NOFOLLOW`) is that one                                      |
| A card: only the device's folders                   | `cardAllows`: inside a folder the profile writes into, or the `._` beside one of those folders; no `..` or empty segment                            |
| The copy's own clean-up: only what this copy caused | `causedAppleDoubles` (beside a file it wrote or a folder it made, in the profile's folders) and modified at or after the run's start on the card's clock |
| A library's removal never overlaps other work on it | `acquireLibraryLock(path, 'junk')`, all or none, never waiting; a look is refused while another job holds it                                         |
| A card's removal never overlaps a copy              | The deploy facade's card slot: no copy starts while it runs, and it refuses while one runs                                                          |

**Platforms.** `O_NOFOLLOW` and `O_NONBLOCK` are used only where the system defines them: on
Windows the open follows a final link and opens a folder, so the `fstat` of the opened file,
compared with the `lstat` made just before (a regular file, the same device and inode), is the
guard there, as everywhere. Real paths come from the native `realpath` on both sides of every
comparison (no 8.3 short names; a junction is a link to `lstat`). Paths are joined and compared
with `path`, never by '/' string tests; stored paths are '/'-separated and split before joining.

**Rename and verify.** Node has no `unlinkat`, so a path checked and then unlinked could be led
elsewhere by a folder on the way swapped for a link in between. So each file is first renamed to
`.romperoom-junk-<random>` in its own folder (onto an exclusive empty reservation: nothing is
replaced); the entry under that name must then be the very file checked (regular, one name, same
device, inode, size and time) in a real folder chain inside the root; only then is that unique
name unlinked. Anything else is renamed back (or, when its name was taken meanwhile, left under
the unique name) and reported, never removed. What this guarantees, per file system:

- **FAT and exFAT** (cards) cannot hold links, so the swap cannot happen there at all.
- **APFS, HFS+, ext4, NTFS and SMB shares with links:** the swap is caught for the check-to-rename
  window. What remains is the instant between the last look at the unique name and its unlink: a
  swap there would have to bring a file of that same random name, which nothing can guess. A crash
  in that instant leaves the verified junk file under its unique name (the scanner ignores it).
- **SMB shares:** `nlink` is often reported as 1 whatever the truth, so a hard link there cannot
  be told apart. Removing one name of a hard-linked file only removes that name: the data stays
  under its other name.

The root must still be the folder the list read (real path, device and inode) or nothing is
removed. A read-only volume (`EROFS`), or one that refuses the first removal (`EACCES`, `EPERM`),
stops the run with nothing changed, and is never tried again. A busy file (`EBUSY`, and `EPERM`
on Windows, where a virus scanner or an indexer holding a file open shows as that) is renamed or
unlinked up to three times with a short wait, then reported as not removed. A file that went away meanwhile is
skipped. **Stop** (`junk:cancel`) takes effect between two files: the one in hand is removed or
put back first, the library lock is released, and the result says `cancelled` with what was
removed. The page names a library by id and a card by the plan id of a copy this window started;
the host turns those into places and passes every message back through `pageText`.

## What a compromised renderer can and cannot do

Suppose an attacker runs script in the page, through a rendering bug or a hostile file name that
somehow became markup.

**It can:**

- Call any channel in the [IPC table](#ipc-channels) with any arguments.
- Add any absolute folder the user's account can read as a library, and scan it. That reads file
  names, sizes and hashes into the catalog, and the page can then list them.
- Show any image the catalog has for such a library, through `romperoom-media:`. Images only;
  at most 50 MiB each; no symlinks.
- Change the catalog: forget libraries, assign or ignore folders, confirm removals. These are
  catalog rows, not files. A rescan or the player can redo them.
- Open the native folder dialog.
- Set aside duplicate copies and unused artwork of a library it has added, and put them back.
- Merge a library's picture folders: move pictures from a console's `Imgs` into its `images` and
  set aside the smaller (or an identical) one of two with one name, and put them back. Only
  pictures inside that console's two folders move, never over anything.
  Only catalogued files of that library move, only into its `.romperoom-quarantine` folder. The
  same holds across libraries: each extra copy moves into its own library's set-aside folder,
  and only while the copy kept in the other library is proven.
- Delete set-aside files forever: it can type `DELETE FOREVER` as well as a person can. The
  words stop accidents, not script. Only files a tidy run set aside, unchanged since (same
  SHA1, a regular file inside the quarantine folder by real path), can be deleted.
- Make the app list cover art pictures on GitHub (spending GitHub's hourly budget of 60 API
  requests, shared with "Check game database updates"), and download the pictures a review found for
  the library's own gaps into `.romperoom/media`. It can import pictures from a card the host lists,
  and remove the pictures cover art added that are unchanged.
- Standardise a library it has added to a device profile it names: rename and merge the
  console folders and rename the identified games its review offered, with their art, saves, cue
  sheets, playlists and game lists (each list's original moved to `.romperoom/lists-backup`
  first), set aside identical copies a merge meets, and undo a run. Nothing is deleted, and
  nothing outside that library is touched.
- Re-link the leftover pictures and game list entries a re-link review of a library it has added
  offered: rename each picture after its game in its own folder, update the game lists that name
  it and re-point the chosen entries (each list's original moved to `.romperoom/lists-backup`
  first), and undo a run. Nothing is deleted, and nothing outside that library is touched.
- Save a ScreenScraper account it types (replacing the player's) or forget the saved one.
- Where this copy has Romperoom's developer details (no release build does today), run a
  ScreenScraper lookup of the games still missing art: the host sends each one's file name, size,
  checksums (SHA-1, MD5, CRC) and console number to `api.screenscraper.fr` under the saved
  account, at most 1,000 games a run, spending that account's daily quota, and saves pictures and
  descriptions only where a game has none.
- Sync a card the host lists, if every deploy safe-target rule allows it: import the games its
  review offered into the library's console folders, copy the saves it offered both ways (each
  replaced save backed up first, a conflict's side as the page picks), give the card its id
  file, and undo a sync.

**It cannot:**

- Run Node.js, require a module, or reach `ipcRenderer` directly.
- Create or change any file, or move or delete one outside the tidy, cover art, card sync and
  standardise rules. Outside [Tidying up](#tidying-up), cover art, card sync and standardise no
  channel writes anything to a library, and the operation engine has no channel. Cover art
  writes only new pictures under `.romperoom/media` (and part files in `.romperoom/tmp`), never
  over a file. Card sync and standardise write only what their review offered
  ([Filesystem safety](#filesystem-safety)), and standardise changes a game list, cue sheet or
  playlist only where it names a renamed file, after moving the original to
  `.romperoom/lists-backup`. Re-link changes a game list only in the renamed pictures' references
  and the chosen entries' paths, after moving the original to `.romperoom/lists-backup`.
- Read a file's contents. It sees hashes and sizes, never bytes, except images through the
  media scheme.
- Load `file://` or another origin, or navigate the window away.
- Reach any host, the LAN included, over HTTP, WebSocket, WebRTC or DNS.
- Open new windows, webviews, workers or frames.
- Get any permission (camera, microphone, notifications, clipboard-read, and so on).
- Choose a host, a path, a query, a header or a commit for a game database download; send any
  data to GitHub beyond the fixed URLs; make the host follow a redirect; or reach any other
  host. The commit it sends back is only compared with the pin, and a different one refuses
  the job.
- Name a URL, a path, a repository, a branch or a file name for cover art, or make the app write
  anywhere but the store. It sends console ids, kind names, a listed volume id, a profile id and
  plan ids the host gave to its own window; the pictures, their names and their places come
  from the review the host made.
- Name a path, a file name or a folder for card sync. It sends a listed volume id, a profile id,
  a library id, a plan id the host gave to its own window with item ids from that review and a
  side per conflict, and a sync id; the files, their names and their places come from the review.

It can also make the host download mapped DATs from `raw.githubusercontent.com` at the current
pin and import them (which can replace a same-named DAT and so revert its matches, as an import
from the picker can today), run "Check game database updates" (one API request, or four when there
is a newer listing), open one of four fixed web pages in the browser, and read the network log.

The app's requests go only to the four allowlisted hosts (`raw.githubusercontent.com`,
`api.github.com`, `api.screenscraper.fr`, `github.com`), and to GitHub's two asset hosts only as a
release file's one redirect, at fixed paths, with fixed headers.
GitHub's requests carry no data from the library. Those requests are not invisible, though.
GitHub (and anyone who can see the connection's metadata) learns which consoles' DATs and
pictures were fetched, from which address, and when, as it would for any download. Picture names
are game titles, so GitHub can see which games a library lacks art for.

ScreenScraper's requests do carry library data. A page that can run a lookup (above) can send,
for each game still missing art, its file name, size, checksums and console number to
ScreenScraper, under the player's saved account or one it saved itself. It still cannot choose
the host, the path, a key or any value's shape (the allowlist checks every one), cannot pick
games outside the host's review, and learns nothing back but the counts the run reports. If a
future feature opens another network path, that feature must be reviewed against this list.

The app updater is such a path, reviewed here. A page can make the host list the public
releases (one API request), fetch a newer release's `SHA256SUMS.txt` and `.sig` from
`github.com`, and, when they verify against a compiled-in key, download that release's file for
this computer and install or show it. It cannot choose the repository, the release, the file,
the URL or the hash, cannot install an older or a prerelease version, an unsigned release or one
whose file does not match the signed list, and cannot install while work runs. The requests carry
no library data. On Windows and AppImage an install quits and restarts the app: a page can force
that restart, once, to the newest signed release only; from its start no other work can begin.

## Deploying to a card

Writing to a card is one of the few things Romperoom does outside its own data folder (the others
write into a library: Fix up's tidying with Standardise and re-link, cover art and card sync), so
the page is treated as hostile here too. The rule: **the page never names a path to write to.**

**Threat model.** Assume the page is compromised (a bug in rendering a game title, a malicious
gamelist). It can call every `deploy:` channel with any arguments, in any order, as often as it
likes. It must not be able to make the host write, move or remove anything outside the card the
user chose, nor write to a drive the safe-target rules refuse.

- **Cards are ids.** The host lists the volumes itself and keeps them by id. The page sees a
  label, sizes, the file system and why a drive is refused, never its mount path. A plan for a
  volume id measures the host's own listing.
- **Folders are tokens.** An export folder is chosen in the host's native dialog. The page gets
  a random token (`randomUUID`) and the folder's name. A token works only for the window it was
  given to, for 30 minutes, and a window holds at most 8 (choosing a ninth drops the oldest). A
  reload forgets them. BIOS folders are chosen the same way and shown by name. Whether a folder
  is on a network drive is checked by the host once per token, when it is first planned (at most
  5 seconds, asynchronous, one check per folder in flight), and the page sees only `network` or
  `unknown`. A start request's `confirmNetwork: true` becomes the plan's own folder path, and only
  when the plan asked; on a card's plan it is refused. The writer refuses the folder if its
  device changed while it was checked (a share mounted over it).
- **Plans are ids too.** `deploy:plan` returns an opaque random plan id bound to the window. The
  host keeps at most 2 per window (the engine at most 4 in all), each for 30 minutes. Any
  change to the catalog (a library added or removed, a folder mapped or ignored, a scan, a BIOS
  folder) makes every earlier plan stale: start answers `plan-stale` and the wizard plans
  again. A card size (a preset) can be planned, never written (`not-writable`).
- **Checked again at start.** `deploy:start` lists the volumes again and runs the safe-target
  rules on the fresh entry: a card swapped, renamed, gone read-only or now holding the library
  is refused. A fixed (internal) disk is written only when the typed label matches the fresh
  listing's label exactly (`label-mismatch` otherwise). The writer then checks the same rules
  once more itself (see [the card writer](architecture.md)).
- **Arguments.** Every argument is validated in the host before the engine sees it: objects
  must have exactly the expected keys (a `path`, `toRel` or `mountPath` key is refused), ids
  are strings of at most 100 characters, a typed label at most 256, and a card size must be one
  of the presets. The package definition is then validated by the engine's strict schema.
  Extra arguments are refused (`arity`).
- **One at a time.** One deploy runs at a time in the whole app (`busy`), and none starts
  while a library scan runs (`scan-running`). The slot is taken before the first `await`, so two
  starts cannot both pass. Progress and the report go only to the window that started it; only
  that window can cancel it, and cancelling always works.
- **Quitting.** Quitting while a card is being written asks "Cancel the copy and quit?". No
  keeps the app and the copy running. Yes cancels, waits up to 10 seconds for the deploy to
  stop, then closes as usual (`createQuitCoordinator`).
- **Reports.** The report the page gets has the card's absolute paths removed (`root`,
  `manifestPath`); file names in it are relative to the card. Every sentence the host sends
  (the report's refusals, warnings, reasons, stop reason and fatal message, a refused start's
  detail, a volume's reasons) goes through `pageText`: the places the host knows by name, and a
  line still naming a folder replaced whole, so the engine's full messages never reach the page
  in those sentences. A path glued to a word (`word/opt/x`) is not recognised; no engine message
  produces one. Errors the deploy channels throw (shown on the wizard through `friendlyError`)
  do not pass through `pageText`; none of them names a path today.
- **Make it fit.** `deploy:plan`'s options take `fit: { pinned }` (at most 1,000 positive game
  ids, checked in the host and by the engine's schema); a suggestion changes nothing until the
  page plans with `selection.excludeGameIds`, which the engine's schema caps at 1,000,000.

Test seams (`ROMPEROOM_TEST_VOLUME`, `ROMPEROOM_TEST_EXPORT_FOLDER`,
`ROMPEROOM_TEST_FOLDER_NETWORK`, `ROMPEROOM_TEST_DEPLOY_DELAY_MS`, `ROMPEROOM_TEST_VOLUME_BYTES`)
are read only when the app is not packaged ([configuration.md](configuration.md)).
`test/deploy-host.test.ts` pins each rule above; `e2e/deploy.spec.ts` probes the real channels
from the page.

## Gaps

These are known and tracked in [roadmap.md](roadmap.md#must-fix-before-later-milestones).
Limits that do not bear on safety (what Standardise, re-link or Across libraries leave out, for
example) are in [development.md](development.md#known-limitations).

- **The media protocol checks folders and then reads.** Between `resolveMedia()` and the open,
  an intermediate folder can be swapped for a symlink. `O_NOFOLLOW` covers only the last path
  component. The impact is limited to reading another image-typed file. Not yet fixed.
- **The media protocol reads a whole file into memory** (up to 50 MiB) rather than streaming it.
- **The media protocol serves `.mp4`, `.webm` and `.pdf`** with their real MIME types, under the
  sandbox CSP. Nothing in Milestone 1 displays them. A later screen that does must keep the
  sandbox.
- **`addLibrary` accepts any absolute directory from the page,** not only one the folder dialog
  returned. A compromised renderer can therefore scan any readable folder (above).
- **Picture conversion's memory is the app's.** The picture worker is a thread of the app's
  process: a picture at the cap holds about 390 MB at its peak while it converts (bounded by the
  pixel cap and jpeg-js's memory cap, not by the worker's heap limit, which typed arrays escape).
  The packaged app's self-check starts the picture worker and converts a tiny embedded JPEG
  (verify-package checks that it and its imports are unpacked). A worker file left inside
  `app.asar` still starts on Electron 44 (measured), so only `app-location` catches that.
- **The builds are not code-signed or notarized**
  ([decisions.md](decisions.md#16-the-beta-ships-unsigned)). Players check downloads against
  `SHA256SUMS.txt`, which only helps if they fetch it from the releases page. Fuses and asar
  integrity are checked on every build (below).
- **The packaged app refuses a list of switches, not every switch** (see
  [launch switches](#launch-switches)), and Chromium creates a `--user-data-dir` folder before
  the app can refuse it.
- **`e2e:packaged` cannot look inside the packaged window.** Playwright attaches only through
  `--inspect` or `--remote-debugging-port`, which the packaged app refuses. Its page URL and
  CSP are covered by `--self-check` (which serves `index.html` through `app://` with the CSP)
  and by the unit tests, not by watching the window.
- **Windows has never been run by hand.** Its tests pass on CI runners only (see
  [testing.md](testing.md#platforms)).
- **The writer's own-file re-check is not atomic with the rename.** Replacing its own copy, the
  writer checks the card file is still the one it wrote, then renames over it. A file a user
  writes in the instant between is overwritten. A file not the writer's is never replaced this
  way: new names use a no-clobber rename.
- **Cover art checks its folders, then writes.** Node has no no-follow open for folders, so a
  folder in `.romperoom/media` or `.romperoom/tmp` swapped for a link between the real-folder
  check and the write would be written through. Where hard links are unavailable (SMB shares,
  FAT, exFAT), a file another program drops at the target between the `lstat` absence check
  and the rename, while the lock is held, would be replaced. Both need another program changing
  the store at that instant
  ([decision 41](decisions.md#41-a-library-gains-one-writer-outside-tidy-up)).
- **Importing art from a card checks, then reads.** A card folder swapped for a link between
  the real-chain check and the open would be read through, and a picture replaced since the
  review by another of the same size and format is imported as found. The checks narrow both to
  that instant; the bytes saved are still only a picture of at most 16 MiB in the store.
- **Card sync checks a card's folders, then writes.** `O_NOFOLLOW` covers only a file's last name,
  so a card folder swapped for a link between the real-folder check and the open, the part file's
  exclusive create, its link or its rename (`.romperoom/card.json` or a save) would be followed, and
  one write could land outside the card. It needs another program changing the card's folders at
  that instant, not a hostile card's own contents. A card save rewritten between the sync's last
  check of its bytes and the rename that replaces it is replaced, and those newest bytes are in no
  backup: no file system offers a compare-and-swap rename, so the check narrows the window to that
  instant. A new card save goes into place by an exclusive hard link; on a card without hard links
  (FAT, exFAT), by the no-clobber rename, where a write into Romperoom's own reserved empty file in
  the instant before the rename would be replaced ([decision
  42](decisions.md#42-card-sync-writes-into-a-library-and-onto-a-card)).
- **Emptying the quarantine checks a file, then deletes it.** A file replaced between its final
  hash and its `unlink` is deleted. The window is one system call, inside a folder only the app
  writes to ([emptying the quarantine](#emptying-the-quarantine)).
- **The work lock is per process.** A second program using the engine on the same library is
  not locked out ([decisions.md](decisions.md#28-one-work-lock-per-library-in-the-process)).
- **The read-back can come from the OS cache.** It proves the bytes the OS holds for the file,
  not that the card's flash stored them. Each file is `fsync`ed before the rename.
- **Network detection for export folders is incomplete.** A Linux network file system outside
  `LINUX_NETWORK_FS` (`fuse.rclone`, `davfs`, `fuse.glusterfs`, for example) and a Windows mapped
  drive whose real path stays a drive letter read as local, and are written without asking.
  Linux is tested from hand-written recorded output only, and the Windows rule against expected
  paths only.
- **A Linux card can be refused as read-only while writable.** Any mount-table entry for exactly
  its mount point that holds `ro` counts, so an old read-only mount under a newer writable one
  refuses the card. It fails safe: nothing is written to a read-only card.
- **A writer refusal names the folder's full path on the Done screen.** See
  [development.md](development.md#the-desktop-app).
- **A folder on the card can be swapped for a symlink during a run.** The writer checks the
  path's real location before it writes (containment), but a local attacker who can change the
  card's folders at the same moment can turn a checked folder into a link and make one write
  land outside the card. It needs a concurrent local attacker with write access to the card.
- **A FIFO at the manifest path hangs the deploy.** Opening a named pipe named
  `romperoom-manifest.json` blocks the read. FAT and exFAT cards cannot hold a FIFO, so it needs
  a card (or a folder target) on a file system that can.
- **The dry run can call a link folder writable while the real run reports partial.** The
  preview tests folders by looking, not by writing, so a link whose target refuses writes passes
  the preview and the real run reports the files it could not place as partial.
- **An edited game list that hit the write-time conflict is never updated again.** When the
  list on the card changes again between the merge and the rename, the writer leaves it and
  drops it from the manifest. Later runs then see a file Romperoom does not own (a conflict), so
  new games are never added to it. Delete it to have a fresh one written.
- **Gaps in disc numbering without an `of N` tag are not warned about.** Discs 1 and 3 of a set
  that never says how many discs it has look complete; only a disc some region of the title has
  and the chosen one lacks is named.
- **An unexpected error's message reaches the page.** A handler passes on the first line of an
  error it did not expect (see Replies under [IPC channels](#ipc-channels)), which can name a path
  inside a library. Wording every such error in the host instead is a project-wide follow-up.
- **No proxy or private-CA support for Download for me.** The game database transport uses Node's
  bundled roots and ignores system and environment proxies, so a network that requires a proxy
  or inspects TLS with its own certificate authority cannot use it; the official-site path (the
  browser) still works.
- **Redump's download page, and any DAT the user downloads from it, come over plain http.**
  Redump serves no https, so the browser opens `http://redump.org/downloads/` and fetches the
  file the user picks there over http too. Someone on the network path could alter either, and
  unlike Download for me nothing checks the file's integrity before it is imported. The DAT
  reader fails closed, so the realistic harm is wrong identification, not code execution.
- **Drag and drop is tested synthetically, not by hand.** Before each release, drag a `.html`
  file and a ROM from Finder (or Explorer) onto the wizard and a games page. The window should stay
  on the app, show the not-allowed cursor and log nothing.
- **Standardise and re-link check, then rename.** Each journal step checks the folders it renames
  into are reached through no link, when it is checked and again right before its rename; a folder
  swapped for a link in between is followed at that instant. On POSIX a folder rename can replace an
  empty folder that appeared at its destination after the check; it holds nothing
  ([decision 43](decisions.md#43-standardise-renames-folders-and-games-and-edits-two-kinds-of-files-other-programs-own)).
  A frontend's own database is not updated, so a renamed game may need a rescan there.
- **A share that hangs can freeze the main process.** Add a library… reads every existing
  library's folder synchronously, and Across libraries reads each candidate copy's device and
  inode synchronously after its folder probe, so a network share that stops answering stalls the
  app until it answers or the system gives up on it. The Libraries tab's probe gives up after 3
  seconds but keeps one file-system thread per hung library until the share answers.
- **Card sync checks, then reads and writes.** The windows of the two cover art entries above
  apply to its library folders and card reads too. The hash pool reads a card game by path, so a
  game swapped for a link after its check is hashed through it; only a hash is learned, and the
  copy reads the card itself, no-follow, and must match that hash
  ([decision 42](decisions.md#42-card-sync-writes-into-a-library-and-onto-a-card)).
- **The update signing keys.** A release is trusted when its `SHA256SUMS.txt` carries a signature
  by a key compiled into the app. `ci-2026-10` is the release key (the `UPDATE_SIGNING_KEY` secret
  of the `release-signing` environment, of which no other copy exists); a cold backup key
  (`cold-2026-10`, generated by the owner offline, never stored in GitHub) is added as the second
  trusted key in a later release; the first updater release trusts only `ci-2026-10`.
  <!-- second-key-pending -->
  Whoever holds a trusted private key can sign a release that every Windows and Linux copy
  installs when its player presses Install. There is no code
  signing behind it, and no revocation but a release that drops the key
  ([release.md](release.md#signed-checksums)).
- **Anyone who can push a `v*` tag can sign.** The `release-signing` environment admits tag
  refs matching `v*` only, so branch runs and pull requests never see the key, but a pushed tag
  runs the workflow at that commit, a changed `release.yml` included. GitHub's tag protection
  rulesets are not available on the repository's plan; the repository is private, with a single
  owner ([release.md](release.md#signed-checksums)).
- **Signing runs on a shared runner.** GitHub-hosted runners are not available on this account,
  so the `sign` job runs on the self-hosted Linux runner, which pull-request CI also uses (never
  on the Mac). Anything that could run code on that runner before a `v*` tag job could capture
  the key from the sign step's environment. Mitigated by `.npmrc` `ignore-scripts=true`, no
  dependency install in the job, the tag-only environment, a single-owner repository, and Node
  run by its absolute path under `env -i` with only the keys; a runner used for signing only is
  the stronger fix ([release.md](release.md#signed-checksums)).
- **Updates carry no download mark.** A file Romperoom writes has no "from the internet" mark
  (Windows' Zone.Identifier, macOS quarantine), so SmartScreen and Gatekeeper do not prompt for
  the installer or the dragged app: the signature and checksum checks stand in for them.
- **The installer is hashed, then started.** Between the last hash and the operating system
  opening the installer no handle is held, so a process of the same user that changed it in that
  instant would have it run; such a process could change the installed app as easily
  ([the installer's last moment](#data-at-rest)).
- **No proxy for updates.** Like the game database download, the updater uses no system proxy or
  private certificate authority; behind one, the check fails as offline or TLS, and the release
  page still works in the browser.

## Packaging checklist

What each release build must satisfy, with the evidence ([release.md](release.md)). "Mutation"
means the check was broken on purpose and its test (or the packaged run) failed, then restored.
Checked items are run by CI's `package` job and by every release build; on macOS (Apple
silicon) they have also passed by hand, on Windows on CI only.

- [x] **Fuses** as in the table above. Evidence: `verify:package` reads every fuse back from the
      binary and compares it with the policy; `--self-check` reads them again from inside the
      running app; `e2e:packaged` starts the app with `ELECTRON_RUN_AS_NODE=1` and a malformed
      `NODE_OPTIONS`, and the app still creates its catalogue and keeps running. Mutations: the
      comparison in `verify-package.mjs`, `strictlyRequireAllFuses` off, the cookie fuse turned
      back on (the policy test), and a package built with `RunAsNode` left on (failed
      `e2e:packaged`).
- [x] **Launch switches refused:** debugging, sandbox, network and profile switches make the
      packaged app exit 2 before any port opens. Evidence and mutations:
      [launch switches](#launch-switches).
- [x] **asar integrity:** the hash electron-builder records (macOS `Info.plist`, Windows the
      executable's resources) equals the hash of the shipped `app.asar` header. Mutation: the
      `Info.plist` hash comparison.
- [x] **Contents:** `app.asar` holds `out/` and production dependencies only (no source maps,
      TypeScript, tests, the smoke chunk, the game database fixture-transport chunk or other
      platforms' native binaries); the unpacked files are exactly the hash worker, its
      dependencies and this platform's SQLite binary. Mutations: the smoke-chunk pattern, and
      the worker's dependency walk (reading `devDependencies` instead).
- [x] **Network boundary:** only `dat-download/transport.ts` opens sockets (static test), the
      production composition passes it no test-only parameter (static test over
      `main/index.ts`), and the renderer guards are byte-identical to 0.4.0
      ([main-process requests](#main-process-requests)).
- [x] **The packaged layout works:** `--self-check` opens a fresh catalogue with the shipped
      SQLite, hashes a file through the unpacked worker, converts an embedded JPEG to PNG
      through the unpacked picture worker, serves `index.html` through `app://`
      from inside `app.asar` with its CSP, and refuses a `..` escape with 403. Mutations: the
      403 check and the DevTools check in `self-check.ts`; the picture worker's path (a wrong
      folder fails `picture-worker`, a path inside `app.asar` fails `app-location`).
- [x] **Development seams ignored when packaged:** `ROMPEROOM_TEST_PICK_FOLDER`,
      `ROMPEROOM_TEST_SCAN_ONLY`, `ROMPEROOM_SMOKE_EXIT_MS`, `ROMPEROOM_TEST_UNGUARDED_WEBRTC`,
      `ROMPEROOM_TEST_LARGER_THAN_SCREEN`, the deploy seams (`ROMPEROOM_TEST_VOLUME`,
      `ROMPEROOM_TEST_EXPORT_FOLDER`, `ROMPEROOM_TEST_FOLDER_NETWORK`,
      `ROMPEROOM_TEST_DEPLOY_DELAY_MS`, `ROMPEROOM_TEST_VOLUME_BYTES`), the game database seams
      (`ROMPEROOM_TEST_DAT_FIXTURES`, `ROMPEROOM_TEST_OPEN_EXTERNAL`) and `ELECTRON_RENDERER_URL`.
      Evidence: each resolver takes `isPackaged` and is unit-tested both ways
      (`test/security.test.ts`); `e2e:packaged` sets them all on a plain packaged launch and checks
      the app creates its catalogue in `ROMPEROOM_DATA_DIR` and is still running long after the
      smoke timer. It gives the test volume, export folder, DAT fixture folder and open-link log
      relative paths, and the folder-network seam an unknown answer, which an unpackaged build
      refuses at startup. The packaged window cannot be inspected (see gaps), so
      `ELECTRON_RENDERER_URL` is covered by the unit tests only. Mutation: the smoke timer honoured
      when packaged (failed `e2e:packaged`: the package does not ship the smoke chunk, so the app
      logs a startup failure).
- [x] **No DevTools, no reload:** `--self-check` reads the window options and menu roles this
      build installs. Mutation: the `windowDevTools` check.
- [x] **Licences:** `THIRD_PARTY_NOTICES.txt` covers every shipped package; an unknown, missing
      or `UNLICENSED` licence, or a package with no licence file, fails the build. Mutations:
      the bad-licence list and the no-licence-file guard.
- [x] **Checksums:** `SHA256SUMS.txt` lists every artifact, and the release merges each build's
      sums only after checking them against the files that arrived. Mutations: the checksum
      comparison, the unlisted-artifact check, and in the merge the changed-file check, the
      listed-twice check, the unlisted check and writing the file despite problems.
- [x] **Unsigned means unsigned:** with no certificate, identity discovery is off, so a Mac with a
      Developer ID in its keychain still builds unsigned. Mutation:
      `CSC_IDENTITY_AUTO_DISCOVERY`.
- [x] **Entitlements:** one, `com.apple.security.cs.allow-jit`, used only when signing with the
      hardened runtime (`build/entitlements.mac.plist`).
- [ ] **Code signing and notarization** (macOS hardened runtime and notarization, Windows
      signing). Wired, not provisioned ([release.md](release.md#unsigned-and-signed-builds)).
- [ ] **The manual drag-and-drop check** above, on each platform, before each release.
- [ ] **A first install by hand on Windows and on a Linux desktop.** CI runs the packaged apps on
      CI runners only (the self-hosted Windows PC and Linux container, else GitHub's).
- [ ] Review the `npm audit` findings from CI (reported, not blocking) before each release.
