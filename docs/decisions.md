# Decisions

Short architecture decision records for the choices a newcomer would otherwise question. Each
says what was decided, why, and what it costs. The longer reasoning is in the
design history.

## Contents

1. [One repository, four workspaces](#1-one-repository-four-workspaces)
2. [Electron, electron-vite and React](#2-electron-electron-vite-and-react)
3. [SQLite through better-sqlite3, with no rebuild](#3-sqlite-through-better-sqlite3-with-no-rebuild)
4. [Scans never touch the library](#4-scans-never-touch-the-library)
5. [A suspicious scan does not mark games missing](#5-a-suspicious-scan-does-not-mark-games-missing)
6. [Hashing in a worker pool](#6-hashing-in-a-worker-pool)
7. [Every file operation is a journaled plan](#7-every-file-operation-is-a-journaled-plan)
8. [The page loads from app://, never file://](#8-the-page-loads-from-app-never-file)
9. [Cover art by catalogue id](#9-cover-art-by-catalogue-id)
10. [No network at all](#10-no-network-at-all)
11. [One typed IPC table](#11-one-typed-ipc-table)
12. [Primitives and tokens only](#12-primitives-and-tokens-only)
13. [Systems and profiles are data from real sources](#13-systems-and-profiles-are-data-from-real-sources)
14. [Private source, public releases](#14-private-source-public-releases)
15. [Install scripts are off](#15-install-scripts-are-off)
16. [The beta ships unsigned](#16-the-beta-ships-unsigned)
17. [Electron fuses](#17-electron-fuses)
18. [A packaged self-check](#18-a-packaged-self-check)
19. [Releases are drafts, in two repositories](#19-releases-are-drafts-in-two-repositories)
20. [A packaged build refuses debugging and network switches](#20-a-packaged-build-refuses-debugging-and-network-switches)
21. [A deploy plan is pure, and sized in bytes on the card](#21-a-deploy-plan-is-pure-and-sized-in-bytes-on-the-card)
22. [The card writer owns only what its manifest lists, and never formats](#22-the-card-writer-owns-only-what-its-manifest-lists-and-never-formats)
23. [The page never names a path to write to](#23-the-page-never-names-a-path-to-write-to)
24. [A crash leaves only what the manifest names](#24-a-crash-leaves-only-what-the-manifest-names)
25. [One clean release, under a name that stays put](#25-one-clean-release-under-a-name-that-stays-put)
26. [The card's own changes win](#26-the-cards-own-changes-win)
27. [Could not check means no](#27-could-not-check-means-no)
28. [One work lock per library, in the process](#28-one-work-lock-per-library-in-the-process)
29. [A journal knows its library without writing to it](#29-a-journal-knows-its-library-without-writing-to-it)
30. [Quarantined files stay in the catalog](#30-quarantined-files-stay-in-the-catalog)
31. [Emptying the quarantine is the only deletion](#31-emptying-the-quarantine-is-the-only-deletion)
32. [A tidy plan is an opaque id, bound to the catalog](#32-a-tidy-plan-is-an-opaque-id-bound-to-the-catalog)
33. [A stopped tidy is finished or undone, not undone in part](#33-a-stopped-tidy-is-finished-or-undone-not-undone-in-part)
34. [Duplicates are files with the same bytes](#34-duplicates-are-files-with-the-same-bytes)
35. [A tidy run that stops part way reports, never rejects](#35-a-tidy-run-that-stops-part-way-reports-never-rejects)
36. [Art a frontend reads is never a spare copy](#36-art-a-frontend-reads-is-never-a-spare-copy)
37. [A library that moved is reconnected by the user, or by a scan that proves it](#37-a-library-that-moved-is-reconnected-by-the-user-or-by-a-scan-that-proves-it)
38. [DATs are parsed by streaming, with refusals rather than guesses](#38-dats-are-parsed-by-streaming-with-refusals-rather-than-guesses)
39. [A game's identity is its DAT name, and a match never leaves its system](#39-a-games-identity-is-its-dat-name-and-a-match-never-leaves-its-system)
40. [Game databases can be downloaded from one pinned source, only when asked](#40-game-databases-can-be-downloaded-from-one-pinned-source-only-when-asked)
41. [A library gains one writer outside Tidy up](#41-a-library-gains-one-writer-outside-tidy-up)
42. [Card sync writes into a library and onto a card](#42-card-sync-writes-into-a-library-and-onto-a-card)
43. [Standardise renames folders and games, and edits two kinds of files other programs own](#43-standardise-renames-folders-and-games-and-edits-two-kinds-of-files-other-programs-own)
44. [Re-link renames a leftover picture after the one game that clearly matches it](#44-re-link-renames-a-leftover-picture-after-the-one-game-that-clearly-matches-it)
45. [Removing a library forgets it whole, and touches nothing on disk](#45-removing-a-library-forgets-it-whole-and-touches-nothing-on-disk)
46. [Copies across libraries are reported, never tidied](#46-copies-across-libraries-are-reported-never-tidied)
47. [An export folder on a network drive asks first](#47-an-export-folder-on-a-network-drive-asks-first)
48. [Copies that differ send Tidy's keeper, and say so](#48-copies-that-differ-send-tidys-keeper-and-say-so)
49. [Saved packages live in the page's storage](#49-saved-packages-live-in-the-pages-storage)
50. [Make it fit suggests, and the player applies](#50-make-it-fit-suggests-and-the-player-applies)
51. [Copies across libraries are set aside in their own library](#51-copies-across-libraries-are-set-aside-in-their-own-library)
52. [Card art is made smaller by Romperoom's own PNG code](#52-card-art-is-made-smaller-by-romperooms-own-png-code)
53. [BIOS files are recognised by libretro's list](#53-bios-files-are-recognised-by-libretros-list)
54. [ScreenScraper uses the player's account and fills gaps by checksum](#54-screenscraper-uses-the-players-account-and-fills-gaps-by-checksum)
55. [The screenshots show generated covers](#55-the-screenshots-show-generated-covers)
56. [One rule for a game's cover](#56-one-rule-for-a-games-cover)
57. [Home and Fix up replace Library, Health and Tidy up](#57-home-and-fix-up-replace-library-health-and-tidy-up)
58. [The games page reads by offset](#58-the-games-page-reads-by-offset)
59. [Tab names stay two words; Settings keeps what is about the app](#59-tab-names-stay-two-words-settings-keeps-what-is-about-the-app)
60. [A profile's ROMs may sit at the card root, and the writer stays in its folders](#60-a-profiles-roms-may-sit-at-the-card-root-and-the-writer-stays-in-its-folders)
61. [A profile may offer consoles unticked](#61-a-profile-may-offer-consoles-unticked)
62. [Scan reads drives and names its scope; Refresh list re-reads the catalog](#62-scan-reads-drives-and-names-its-scope-refresh-list-re-reads-the-catalog)
63. [Suggestions are accepted with one shared select-and-accept bar](#63-suggestions-are-accepted-with-one-shared-select-and-accept-bar)
64. [Zipped games are unzipped on copy for devices that need plain ROMs](#64-zipped-games-are-unzipped-on-copy-for-devices-that-need-plain-roms)
65. [Junk files are removed by content, after a preview](#65-junk-files-are-removed-by-content-after-a-preview)
66. [Merge picture folders: Imgs into images, the larger stays](#66-merge-picture-folders-imgs-into-images-the-larger-stays)
67. [JPEG and GIF art is converted to PNG on the card only](#67-jpeg-and-gif-art-is-converted-to-png-on-the-card-only)
68. [The app updates itself from signed checksums, without code signing](#68-the-app-updates-itself-from-signed-checksums-without-code-signing)

## 1. One repository, four workspaces

- **Decision:** npm workspaces: `packages/profiles` (data), `packages/engine` (catalog, scanner,
  operations), `packages/ui` (design system) and `apps/desktop` (Electron).
- **Why:** the engine has no Electron dependency, so it is tested in plain Node with real files.
  A later command-line tool or second app can reuse it.
- **Cost:** a build order (profiles, then engine, then the rest), because the desktop app uses
  the engine's built `dist/`. The root `typecheck` and `build` scripts encode it.

## 2. Electron, electron-vite and React

- **Decision:** Electron 44, built with electron-vite, with a React 19 renderer.
- **Why:** one codebase for macOS and Windows, a real file system API in the main process, and
  native modules (SQLite).
- **Cost:** a Chromium-sized app, and a renderer that must be locked down
  ([security.md](security.md)).

## 3. SQLite through better-sqlite3, with no rebuild

- **Decision:** the catalog is one SQLite file (`catalog.sqlite`), opened with better-sqlite3.
- **Why:** a library of many thousands of files needs indexed queries, and better-sqlite3 13 ships
  Node-API prebuilds that load under both Node and Electron. That removed the planned
  `@electron/rebuild` step, and install scripts are off ([15](#15-install-scripts-are-off)).
- **Cost:** a synchronous API in the main process, so queries are kept small and paged.
  `createEngine({ nativeBinding })` is the escape hatch if a future release drops Node-API.

## 4. Scans never touch the library

- **Decision:** a scan only lists folders, reads file details and reads file contents to hash
  them. Nothing is written inside a library. The catalog lives in the app's data folder.
- **Why:** players point Romperoom at a collection built over years, often on a NAS.
- **Cost:** no sidecar files, so a moved or renamed file is only recognised by its hash.

## 5. A suspicious scan does not mark games missing

- **Decision:** when a scan sees a folder emptied, a whole library empty, or a burst of errors,
  it ends `could-not-read` or holds those folders as suspect. Only the player's confirmation
  (`confirmRemoval`) lets files go missing.
- **Why:** an unmounted share, an empty mount point and a dropped network look exactly like a
  deleted library.
- **Cost:** a library whose ROM folders really are all gone needs removing and adding again.

## 6. Hashing in a worker pool

- **Decision:** files are hashed in `worker_threads`, with a per-file timeout and a pool that
  recovers from crashed workers.
- **Why:** a large zip or a stalled network read must not freeze the app or hang a scan.
- **Cost:** the worker is plain ESM outside coverage, and pool tests depend on timing.

## 7. Every file operation is a journaled plan

- **Decision:** a change to files is first a plan you can preview. Applying it journals each
  step before and after it runs. Undo replays the steps backwards and refuses any step whose file
  changed. Removing means moving to a quarantine folder inside the library.
- **Why:** the player must be able to see, and take back, everything Romperoom does.
- **Cost:** slower than plain moves. Only the Tidy up screens apply one (the card writer keeps
  its own manifest).

## 8. The page loads from app://, never file://

- **Decision:** the renderer is served from a custom `app://romperoom/` scheme, with a strict CSP.
- **Why:** under `file://`, CSP `'self'` matches every local file, so an injected script could
  read any of them.
- **Cost:** a protocol handler with its own path checks and tests.

## 9. Cover art by catalogue id

- **Decision:** the page asks for art as `romperoom-media://media/<id>`. The main process looks
  the id up and opens the file without following links. Windows cannot refuse a link at the
  open, so there the opened file must also still be the file the name points at.
- **Why:** the page never names a path, so it cannot ask for one it should not see.
- **Cost:** art the scan has not catalogued cannot be shown.

## 10. No network at all

- **Decision:** a dead proxy on every session, a host resolver rule that fails every lookup,
  and a WebRTC IP policy. Milestone 1 needs no network.
- **Why:** a ROM library is private. CSP alone does not stop WebRTC or DNS lookups.
- **Cost:** each later network feature must allowlist its one host, from the main process.

Superseded by ADR 40 for downloading game databases, by ADR 41 for cover art, by ADR 54 for
ScreenScraper lookups and by ADR 68 for app updates; every other part stands.

## 11. One typed IPC table

- **Decision:** `src/shared/ipc.ts` lists every channel. The handlers and the preload are
  built by walking it, and the type check fails if it and the engine disagree. Every handler
  checks its sender. Errors cross the bridge as a name and a short message only.
- **Why:** the bridge is the app's attack surface. One table keeps it small and reviewable.
- **Cost:** adding a method touches a few pinned lists in the tests, on purpose.

## 12. Primitives and tokens only

- **Decision:** every screen is built from `@romperoom/ui` primitives, styled with semantic
  tokens. A test fails on raw colours or sizes in feature code.
- **Why:** three themes, light and dark, and a re-skin from one place.
- **Cost:** a new kind of control starts as a primitive, not as one-off markup.

## 13. Systems and profiles are data from real sources

- **Decision:** systems and device profiles are JSON, validated by Zod. Extensions and folder
  names come from ES-DE first, then Batocera. A profile is `verified` only when checked against
  hardware or official documentation.
- **Why:** a wrong folder name puts games where a device never looks.
- **Cost:** a system no source documents gets a short, hand-written list.

## 14. Private source, public releases

- **Decision:** the source repository is private. Installers are published to a separate public
  repository that holds no source, created from `releases-repo-template/`; a future updater
  would read it too.
- **Why:** players and a future updater must reach the downloads without a token, and the
  source stays private.
- **Cost:** a second repository to keep in step, release notes that must not leak the private
  history, and, for the automated copy, a token scoped to that one repository
  ([19](#19-releases-are-drafts-in-two-repositories), [release.md](release.md)).

## 15. Install scripts are off

- **Decision:** the root `.npmrc` sets `ignore-scripts=true`, so `npm ci` runs no install script
  of any dependency, and none of ours. Anything an install should do is an explicit command, such
  as `node node_modules/electron/install.js` for the Electron binary.
- **Why:** CI's first Windows run failed in `npm ci`, compiling better-sqlite3. The package says
  `"gypfile": false`, but `npm ci` takes package metadata from the lockfile, which does not record
  that field. It saw `binding.gyp` and ran an implicit `node-gyp rebuild`, which needs a C++
  toolchain. The rebuild was never wanted: the package ships the Node-API binary for every
  platform we build ([3](#3-sqlite-through-better-sqlite3-with-no-rebuild)). The other scripts
  in the lockfile only check a binary that is already there (esbuild, twice) or are a flag with
  nothing behind it (fsevents). An install that runs no third-party code is also one less way in
  for a compromised package.
- **Cost:** a dependency that really needs its install script fails at first use rather than at
  install. `packages/engine/test/prebuilds.test.ts` checks the shipped SQLite binaries, and a
  manifests test fails on any `pre`/`post`/`prepare` script of our own, which npm would skip.
  Before adding a dependency, check its `hasInstallScript` in `package-lock.json`
  ([development.md](development.md#install-scripts-are-off)).

## 16. The beta ships unsigned

- **Decision:** the first beta builds are not code-signed or notarized. The packaging is ready
  for it: setting the certificate secrets signs (and, on macOS, notarizes) the same build, and a
  build with a certificate set fails rather than fall back to unsigned.
- **Why:** signing needs an Apple Developer ID and a Windows code-signing certificate, and
  neither exists yet. Waiting for them would hold back a beta whose testers can open an unsigned
  app.
- **Cost:** Gatekeeper and SmartScreen warn on first open, and players must take an extra step
  (the public README explains it). A download's authenticity rests on `SHA256SUMS.txt` from the
  releases page rather than a signature. No automatic updates until builds are signed.

Superseded by ADR 68 for updates: the app installs new versions itself, trusting an Ed25519
signature over `SHA256SUMS.txt` rather than code signing.

## 17. Electron fuses

- **Decision:** packaged builds flip Electron's fuses: no running as Node, no `NODE_OPTIONS`, no
  `--inspect`, asar integrity validation on, load only from `app.asar`, no extra `file://`
  privileges. Cookie encryption stays off. One policy file (`src/main/fuse-policy.json`) is
  written at `afterPack`, before signing, and read back by `verify:package` and by the app
  itself ([security.md](security.md#electron-hardening)).
- **Why:** each fuse closes a way to run code inside a trusted, installed app without changing
  its files: an environment variable or a switch is enough otherwise.
- **Why cookie encryption is off:** the app stores no cookies and no secrets in the Chromium
  session (settings are in localStorage), and the fuse can make the first launch on macOS ask
  for Keychain access. Revisit it when the app stores credentials or cookies, and prefer
  Electron's `safeStorage` for those secrets.
- **Cost:** the packaged app cannot be driven by Playwright's Electron launcher (it needs
  `--inspect`), and it refuses `--remote-debugging-port` too
  ([20](#20-a-packaged-build-refuses-debugging-and-network-switches)), so `e2e:packaged` runs
  the app as a plain process and reads its exit code, output and files.

## 18. A packaged self-check

- **Decision:** `--self-check` runs, inside the real packaged app, the checks packaging can
  break: SQLite, the unpacked hash worker, `app://` from inside `app.asar`, the fuses, DevTools.
  It prints one JSON line and exits ([configuration.md](configuration.md#command-line-switches)).
- **Why:** unit tests run unpacked, so they cannot see an `asarUnpack` mistake or a missing
  native binary. A check the shipped binary runs on itself can, on every platform CI builds,
  without a display or a test driver.
- **Cost:** a small amount of test-only code ships in the app. It opens no window, touches only a
  temporary catalogue and profile, and reports nothing anywhere but standard output.

## 19. Releases are drafts, in two repositories

- **Decision:** a pushed `v*` tag rebuilds and re-checks everything and creates a **draft**
  release in the private repository. Copying it to the public repository is a separate, opt-in
  run that also creates a draft. A person publishes both; the workflow never edits a published
  release.
- **Why:** an unsigned build is only as trustworthy as the person who checked it, and a public
  release cannot be taken back once players have it.
- **Cost:** two manual publish steps per release, and the public README's download table is
  updated by hand ([release.md](release.md#publishing-to-the-public-repository)).

## 20. A packaged build refuses debugging and network switches

- **Decision:** a packaged build exits with code 2, before it is ready, when its command line
  carries a remote-debugging, inspector, sandbox, web-security, proxy, host-resolver or
  user-data-dir switch ([security.md](security.md#launch-switches)). Unpackaged builds accept
  them all.
- **Why:** `--remote-debugging-port` gives any local process full control of the page, and
  the proxy and resolver switches would undo the app's network isolation. No fuse covers
  Chromium switches, so the app checks them itself.
- **Cost:** a player cannot debug the shipped app, and the end-to-end tests of the packaged
  app cannot see inside its window. The list has to grow if Chromium adds a switch like these.

## 21. A deploy plan is pure, and sized in bytes on the card

- **Decision:** `planPackage` reads the catalog through a small reader interface and touches no
  file. It returns every destination with its size in whole clusters, the directory overhead,
  every skipped game with a reason, warnings, and blocking problems. Writing is a separate step
  that re-checks each path against the real card (`verifyPlanPaths`).
- **Why:** a size meter that counts source bytes says a package fits when the card is full:
  a library of small files can lose a third of a card to cluster rounding. A pure plan is
  tested exactly and deterministically, with synthetic catalogs and no card.
- **Cost:** the plan can go stale between planning and writing, so the writer must check again
  and the engine keeps a plan for 30 minutes only. Unknown cluster sizes are assumed to be the
  largest Windows default, which overstates a card formatted with smaller clusters.

## 22. The card writer owns only what its manifest lists, and never formats

- **Decision:** the writer keeps `romperoom-manifest.json` at the card root: every file it
  wrote, with size, SHA-1 and the card's mtime. A file is the writer's only when it matches
  that record. Copies go to `.romperoom-part` files, are renamed into place without clobbering
  and read back; dropped files are moved to `.romperoom-removed/`. Volumes are listed with the
  OS's own commands through `execFile`, not a native module, and there is no format, erase or
  partition code at all (a test greps for it). FAT32 and the test volume are proven; exFAT is
  not tested on a real file system yet.
- **Why:** a card is shared with the user and with the device. Writing over or deleting a save,
  a theme or a hand-placed file is the worst thing this app could do, so anything not provably
  its own is left alone. A manifest makes runs incremental and resumable without guessing.
  Install scripts are off (decision 15), so `drivelist`-style native modules are out.
- **Cost:** a card written by hand or by another tool is all conflicts until the user chooses
  `overwriteUnmanaged` (which still moves, never deletes). Moved files keep using space until
  the user empties `.romperoom-removed`. A damaged manifest disowns everything. The OS command
  output must be parsed per platform, and Windows and Linux are tested from recorded output only.

## 23. The page never names a path to write to

- **Decision:** the card wizard's page refers to a card by the volume id from the latest
  listing, to an export folder by an opaque token the host minted when the user picked it, and
  to a plan by a random `planId`. Tokens and plans belong to the window that made them. At
  **Start copying**, the deploy host lists the volumes again, checks the safety rules and the
  typed name of a fixed disk against that fresh listing, and refuses a plan made before the
  library changed (`plan-stale`). One copy runs at a time, and quitting during a copy asks first.
  Every request is checked at run time, and an unknown field is an error.
- **Why:** a compromised or buggy page that could name a path could write anywhere the user
  can. With ids and tokens, the worst it can do is pick among destinations the host already
  judged safe and the user chose. Listing again at start closes the gap between planning and
  writing: a card swapped, unmounted or made read-only in between is caught before any write.
- **Cost:** more state in the host (tokens, plans, a generation per library change), and a plan
  can go stale while the user reads the Check step, so the page has to measure again. The page
  shows names, never a full destination path; paths appear only in a report's collapsed
  details, relative to the card. Every sentence the deploy host sends in a report, a refused
  start or the volume list names the card, libraries and folders by name (`pageText`), and a
  sentence naming any other folder is replaced whole, so such a detail can be lost rather than
  show a path. One gap: a path glued to a word (`word/opt/x`) is not recognised as a path; no
  engine message produces one. An error the deploy channels throw (shown by the wizard as it
  comes) does not pass through `pageText`; none names a path today.

## 24. A crash leaves only what the manifest names

- **Decision:** before a new file's empty no-clobber reservation is made, the manifest's
  `reserved` list names it (with its device and inode once known), and the entry is cleared
  when the file is in place. The next run removes an empty file at a final name only when that
  entry names it, its identity matches, the manifest does not own it, and it lies inside the
  profile's folders. `pending` and `dirs` entries outside the profile's folders are ignored, and
  a blank or all-NUL temporary manifest counts as absent. After the first change to the card,
  any unexpected error, or 25 card errors in a row, ends the run as `card-error`: the manifest
  is saved once more and the report comes back with everything done so far.
- **Why:** a crash between reserving a name and writing the file used to leave an empty file
  the next run took for the user's, a conflict forever. The manifest is read from a card anyone
  can edit, so it may only point at the writer's own folders. A deploy that throws halfway hides
  what it changed; a report that says what happened lets the user see the state of the card.
- **Cost:** one more manifest save per batch of new files. A reservation made by an older app
  version is not in any list, so it stays a conflict until the user deletes it.

## 25. One clean release, under a name that stays put

- **Decision:** the planner copies one release per game: a game tagged with several regions
  ranks by the best of them, a finished release beats a bad dump or a Proto, Beta, Demo, Sample,
  Unl or Pirate one, the highest revision wins, and discs of one release stay together: each
  disc comes from its newest revision, the region with every disc beats a better region missing
  one, and a disc no region has is named in a warning, never dropped silently. A game
  that only exists unfinished is copied with a warning. When one-file games would share a card
  name, each gets a suffix from its own file (the first six hex digits of its SHA-1, else of a
  hash of where it lives), not a number in planning order.
- **Why:** a frontend shows every file it finds, so three revisions of one game are three menu
  entries. Numbered suffixes depended on order: removing one twin renamed the others, and the
  next deploy moved files and orphaned their play counts and screenshots.
- **Cost:** a release-name convention the planner does not know falls back to the file name.
  A set mixing revisions has no playlist on the card (the library's names the older discs), so
  the frontend opens Disc 1 and changing discs is up to the emulator's own menu.
  Suffixed names are less readable than `(2)`, and every long name is cut 12 characters
  earlier to leave room for a suffix.

## 26. The card's own changes win

- **Decision:** when the manifest was written for a different profile, nothing it owns is
  replaced, moved or erased unless the user turns on **Replace what Romperoom put here for**
  that device; the start request carries `confirmProfileSwitch: true` only then. An owned
  `gamelist.xml` the frontend changed is kept: the plan's `<game>` entries it lacks (matched by
  `<path>`) are added before its last `</gameList>`, and the manifest marks it `frontendEdits`
  so later runs add to it instead of replacing it. A list that cannot be read as UTF-8 XML in one
  pass, or is over 64 MiB, stays a conflict.
- **Why:** ES-DE and Batocera write play counts, favourites and last-played dates into the game
  list on the card. Replacing it threw them away; refusing it forever stopped new games from
  appearing. Two devices sharing one card is plausible, and switching profiles silently would
  move one device's games aside for the other.
- **Cost:** a game the user removed from the selection stays in an edited list (only entries are
  added, never removed). The manifest schema is strict, so a manifest a later version wrote
  (a higher `schemaVersion`, or fields this version does not know) cannot be read in full; the
  older app refuses that card and asks for an update rather than rewriting the manifest without
  what it does not know. No version without that refusal was released.

## 27. Could not check means no

- **Decision:** an export folder is compared with the library roots, BIOS folders and the app's
  data folder by real path on both sides; a guarded folder that cannot be resolved refuses the
  export. Two volumes listed with one id refuse the listing. On Windows, only
  `DriveType = Removable` counts as removable: a USB hard disk or SSD needs the typed name like a
  fixed disk on macOS.
- **Why:** a link, a junction or a different spelling of one folder used to pass a text
  comparison. An id that names two volumes might name the wrong one at **Start copying**. USB
  disks are often a second internal drive in an enclosure, holding the user's own data.
- **Cost:** a missing BIOS folder or a flaky network library blocks exporting until it is back
  or removed from the settings. USB card readers that report themselves as fixed need the typed
  name.

## 28. One work lock per library, in the process

- **Decision:** scans, deploys, tidy operations, purges, library removal and identify runs take
  one lock per library (by resolved path) in the engine. Only two deploys may share it; every
  other pair refuses with `LibraryBusyError`. The lock lives in memory: the app already refuses a
  second instance (`requestSingleInstanceLock`).
- **Why:** a scan marking files missing while an operation moves them, or a deploy reading a
  file a purge deletes, gives wrong answers or loses data. Separate guards per job could not see
  each other.
- **Cost:** a second process (a script using the engine) is not locked out. A lock file would
  be, but it writes to the library, which scans never do
  ([decision 4](#4-scans-never-touch-the-library)), and a stale one after a crash needs its own
  recovery.

## 29. A journal knows its library without writing to it

- **Decision:** a journal records the library folder's device and inode, its real path and a
  fingerprint of its top-level folder names. Finish, roll back, undo and purge need the real path
  to match and either the device and inode or the fingerprint; an empty folder never matches.
- **Why:** after a crash with the NAS not yet mounted, the mount point is an empty folder.
  Recovery used to see neither copy of a moved file and close the journal as if nothing had
  moved. A sentinel file would identify the volume exactly, but it writes to the library.
- **Cost:** a library copied to a new disk (new device, same names) is accepted by its
  fingerprint, which is what a remount needs; a different folder with the same top-level names
  would be too. Renaming top-level folders and remounting at once blocks recovery until the
  user picks a resolution.

## 30. Quarantined files stay in the catalog

- **Decision:** a file moved to quarantine keeps its catalog row, with the status
  `quarantined` and its quarantine path. Undo sets it back to `present`; emptying the quarantine
  deletes the row. Scans skip the quarantine folder and never mark a quarantined row missing.
- **Why:** sizes, health and history must agree with the disk. A row dropped at the move
  would lose its hash, so the purge could not prove it deletes the bytes that were moved.
- **Cost:** every query that counts present files must exclude `quarantined` (they filter on
  status already). A restored file is `present` but unconfirmed until the next scan re-hashes it.

## 31. Emptying the quarantine is the only deletion

- **Decision:** tidy operations only move files into `<library>/.romperoom-quarantine`. Only
  the purge deletes, after a preview, the words `DELETE FOREVER` and the exact byte count,
  and only files a journal moved there whose content still matches
  ([security.md](security.md#emptying-the-quarantine)).
- **Why:** a wrong keeper, a wrong DAT or a bug in a later cleanup must be undoable. Keeping
  deletion in one small, separately checked path keeps the dangerous code reviewable.
- **Cost:** removed copies use disk space until the user empties the quarantine. Moving across
  volumes (a quarantine on another disk) is never done: the quarantine is inside the library.

## 32. A tidy plan is an opaque id, bound to the catalog

- **Decision:** the tidy facade returns a random id for each preview. Apply and purge take
  only that id (zod-validated); the plan itself stays in the engine, in a store with a time
  limit and a count limit that drops the least recently used. Every id is bound to the catalog's
  generation, so any write in between answers `plan-stale`, and an id is used once.
- **Why:** a page that could send a plan could send any path
  ([decision 23](#23-the-page-never-names-a-path-to-write-to)). A plan computed before a scan may
  keep a copy that is gone now.
- **Cost:** any catalog write, even an unrelated deploy record, makes the user preview again.

## 33. A stopped tidy is finished or undone, not undone in part

- **Decision:** Cancel stops a tidy between files and leaves its journal `running`, exactly as
  a crash would. Its result says what moved and what stayed, and offers **Finish or undo…**,
  which settles that journal through recovery (finish the rest, or put back what moved). "Undo
  all" is offered only for a run that ended. The startup prompt opens only when the first health
  reading counts interrupted work. A Tidy up run (never a Standardise, re-link or card-sync one)
  can also **Discard the rest**: what moved stays set aside and undoable, and each step left is
  recorded as not run, once every one of them is proven not started (its file still where it was
  and its destination free, or a later run already set that file aside).
- **Why:** the engine's undo refuses an operation with a running journal, and should: half a
  journal has no settled "before". Recovery already re-checks every file the way a fresh run
  does, so one path covers both a crash and a cancel. A prompt that opened mid-session would
  cover the result the player had just asked for.
- **Cost:** a stopped run that is neither finished, undone nor discarded stays counted as
  interrupted and is offered again at the next start. Discard the rest is refused while a file of
  the run is not where the run left it, so a run cut short in the middle of a move is still only
  finished or undone.

## 34. Duplicates are files with the same bytes

- **Decision:** duplicates group on the hash of the whole file as stored (`sha1_whole`,
  migration 8), never on the hash of a zip's largest entry. Hard links of one another, disc-set
  members and the same bytes under two consoles are left out and listed with the reason. A
  folder that cannot be read leaves its files out as part of a disc set (fails closed).
  Archives catalogued before migration 8 are counted as not checked until the next scan.
- **Why:** two zips with the same game and a different save or patch inside matched on the
  inner hash, and setting one aside lost the only copy of that save. A hard-linked "copy" frees
  nothing, a shared audio track is needed by both releases, and each console folder needs its
  own file.
- **Cost:** a zipped and a plain copy of one game are no longer offered as duplicates. An extra
  set aside before migration 8 reads "its kept copy has changed" at Delete forever (its recorded
  hash is the inner one) and stays in quarantine until restored or deleted by hand.

## 35. A tidy run that stops part way reports, never rejects

- **Decision:** once a step has moved, `applyOperation` returns a result. A library that goes
  away mid-run ends it `interrupted`, with `fatal: { reason, message }` stored on the operation;
  a cancel ends it `cancelled`, later chunks never start. Both stay resumable through recovery
  ([decision 33](#33-a-stopped-tidy-is-finished-or-undone-not-undone-in-part)).
- **Why:** a rejection after files moved told the page nothing about what had moved, and the
  operation read `running` forever. The desktop already settles a stopped run by finishing or
  rolling back its journal.
- **Cost:** two more operation statuses. The desktop shows both as a run to finish or undo.

## 36. Art a frontend reads is never a spare copy

- **Decision:** a picture under `downloaded_media/<system>/` or a console folder's `images/` or
  `Imgs/` is never listed as a duplicate, and art in a folder no console is mapped to is listed
  apart ("folder not recognised", `notRecognised`) and never cleaned.
- **Why:** frontends read those copies in place; keeping Romperoom's store copy instead removed
  the picture from the frontend. What reads an unrecognised folder is unknown.
- **Cost:** duplicate art in frontend folders stays on disk. The Tidy up screen does not show
  the not-recognised list yet (since the reorganisation, ADR 57, that screen is Fix up's Artwork,
  which does not show it either).

## 37. A library that moved is reconnected by the user, or by a scan that proves it

- **Decision:** a journal blocked because its library folder changed (a remount, a move, new
  top-level folders) can be re-pointed at the folder now at its path:
  `tidy.reconnectLibrary(rootId, { confirm: true })` after the user answers "Is this still your
  library?", or by itself after a complete, unscoped scan. Never an empty folder, never one
  sharing none of the recorded top-level folders. The user may reconnect at the recorded real
  path, or anywhere with at least 80% of the folders the same (Jaccard overlap of the names the
  journal recorded); a scan needs both the recorded real path and that overlap, and leaves a
  library alone while any of its operations runs, is blocked, or a purge of it stopped part
  way. Every change writes a `library_reconnect` row with the identities it replaced.
- **Why:** a network share gets a new device number on every mount, and adding a console folder
  changes the fingerprint; with both, undo was blocked for good with "the drive isn't
  available", which was not true. An unplugged drive's empty mountpoint, or another drive at
  the path, must still never pass.
- **Cost:** journals keep up to 2,000 folder names (migration 9). A scan is stricter than the
  user (it needs the same real path), so a moved library always asks once.

## 38. DATs are parsed by streaming, with refusals rather than guesses

- **Decision:** Logiqx XML is parsed with saxes 6.0.0 (ISC, non-validating, never fetches or
  expands DTD entities), clrmamepro text with our own tokenizer. A DOCTYPE with an internal
  subset, an unknown entity, a non-UTF-8 declaration or anything over the documented limits is
  refused; a bad entry is skipped with a warning that names it.
- **Why:** a DAT is a file from the internet; the parser must fail closed and never block.
- **Cost:** some DATs a lenient tool would read are refused; the message says why.

## 39. A game's identity is its DAT name, and a match never leaves its system

- **Decision:** an identified game is keyed by system and DAT game name; a filename game by
  system, title and region. A file matches only DATs of its own system; a hash in another
  system's DAT is reported (`other-system`), never applied. Files not hashed yet are never
  matched, not even by name.
- **Why:** two revisions of one title are two games; a Game Boy hash in a Game Boy Color
  folder is the user's to move, not ours to rename.
- **Cost:** a misfiled game stays unidentified until it is moved or its folder remapped.

## 40. Game databases can be downloaded from one pinned source, only when asked

- **Decision:** Game databases offers "Get from the official site" (the browser opens
  DAT-o-MATIC or Redump; Romperoom requests nothing) and "Download for me": libretro-database
  DATs (CC BY-SA 4.0) from `raw.githubusercontent.com` at a pinned commit, after the user
  reviews the files, sizes, source and licence. "Check for updates" asks `api.github.com` (at
  most 4 requests since batch 3 added the `dat` listing) only when pressed. Main process only,
  `node:https`, exact URL allowlist, no redirects, timeouts and byte caps, every file checked against its listed size and git SHA,
  imported through the two-phase import. The renderer's CSP, dead proxy, resolver rules and
  WebRTC policy do not change.
- **Why:** finding the right DAT is the hardest step after Milestone 2. DAT-o-MATIC and Redump
  give no permission for automated downloads; libretro-database's licence does.
- **Cost:** the first network code. No system proxy or private certificate authority for
  "Download for me" (the official-site path still works). libretro's copies can lag behind the
  official sites.

## 41. A library gains one writer outside Tidy up

- **Decision:** Cover art writes pictures into a library, only into Romperoom's own
  `.romperoom/media/<system>/<box|screenshot|title>/` (and its part files in `.romperoom/tmp`),
  only when the user presses Get cover art or Import art from an SD card and confirms a review.
  The engine's writer holds the library's `art` work lock, writes a part file in
  `.romperoom/tmp`, fsyncs and closes it, then hard-links it into place (an exclusive create), or,
  where the file system has no hard links (SMB shares, FAT, exFAT), checks that the target is
  absent and renames it into place. A picture is never written at its final name, so no
  half-written file is ever visible there. It never overwrites a file it has seen, and it
  writes only below folders it has checked are real folders inside the library, never links.
  Every file and folder it adds is recorded (`art_added`, `art_dir`), so Remove downloaded art
  deletes exactly those whose size and SHA-1 still match.
- **Why:** pictures are what a library without art lacks most, and the scanner already reads and
  prefers `.romperoom/media`; a separate store would need a second scanner and would not travel
  with the library to another computer.
- **Cost:** the library is no longer read-only outside Tidy up; the promise narrows to "never
  changes your games". A picture is named after the game's ROM file, so renaming the ROM leaves
  the picture unlinked until it is renamed too (the same rule the scanner applies to all art).
  Two windows cannot be closed from user space (Node has no `openat` or no-follow for folders):
  a folder swapped for a link between the check and the write would be written through, and,
  without hard links, a file dropped at the target between the absence check and the rename,
  while the lock is held, would be replaced. The checks narrow both windows to the moment of the
  write; another program changing the store at that instant is outside what the lock can
  exclude. Import art from an SD card has the same window on its reading side: each reviewed
  picture is read again at import as a regular file, never through a final link, below folders
  checked to be a real chain inside the reviewed card, and skipped unless its size and format
  still match the review; but a card folder swapped for a link between that check and the open
  would be read through, and a picture replaced since the review by another of the same size and
  format is imported as found.
- **Network:** Get cover art lists and downloads pictures from `api.github.com` and
  `raw.githubusercontent.com` through the game database transport (ADR 40's rules), only when
  pressed; nothing else changes in ADR 10.

## 42. Card sync writes into a library and onto a card

- **Decision:** **Sync a card** brings a handheld's new games and in-game saves into the library
  and sends newer saves back, only when the user confirms a review, and never deletes anything
  on either side. Into the library it writes only into console folders (imported games, never
  over a file) and under `.romperoom/saves`, `.romperoom/saves-backup` and `.romperoom/tmp`, with
  cover art's guarantees (ADR 41): every folder on the way is a real folder inside the library,
  never a link, and the work holds the library's `sync` lock. Card bytes reach the library only
  through part files in `.romperoom/tmp` (exclusive create, fsync, close) checked against the
  SHA-1 the review measured, then go into place through one Tidy up journal (ADR 7): `move`
  steps that never replace a file, so Undo this sync reverses them, only where a file still
  holds what the sync wrote. A library save that is replaced is first moved to
  `.romperoom/saves-backup`. On the card it writes `.romperoom/card.json` (a random id) and saves:
  a card save is replaced only after its bytes are saved in the library's backup folder, through
  a part file beside it, fsynced, then renamed over it once it still holds what the review saw.
- **Why:** "game saves should be portable" (owner decision 2): a save made on a handheld is
  unique until it is copied, and the library is the hub. The journal already proves never
  overwriting, crash recovery and undo; reusing it keeps one mechanism for moving files.
- **Cost:** the library has two writers outside Tidy up, and the promise narrows again: "never
  changes your games" still holds (games are only added), but saves under `.romperoom/saves` are
  replaced, always after a backup. The deploy writer's rule (ADR 22) still governs what it writes;
  card sync is a second writer to a card, which replaces only saves, after a backup. Card writes are
  not undone automatically (their earlier bytes are in the backup folder). The windows ADR 41
  records apply here too: a folder swapped for a link between the check and the write, or a card
  file swapped between the check and the read, at that instant; and the hash pool reads a card game
  by path, so a game swapped for a link after its check is hashed through it (only a hash is
  learned; the copy reads the card itself, no-follow, and is checked against that hash). On the
  card, no-follow covers only a file's last name: a card folder swapped for a link between its
  real-folder check and the open, the part file's exclusive create, its link or its rename (the id
  file or a save) is followed, so one write could land outside the card. It needs another program
  changing the card's folders at that instant, not a card's own contents. A new card save goes into
  place by an exclusive hard link, which never replaces a file; a card without hard links (FAT,
  exFAT, on any system) gets the no-clobber rename instead, where only a write into Romperoom's own
  reserved empty file in the instant before the rename would be replaced. A card save the player
  rewrites between the run's last check of its bytes and the rename over it is replaced, and those
  newest bytes are in no backup (no file system offers a compare-and-swap rename; the check narrows
  it to that instant). In the library, each journal step checks its folders are real folders inside
  the library right before it makes a folder or reserves its destination's name, so a folder swapped
  for a link during the run refuses the step. A sync interrupted by a crash waits, like any journal,
  in Tidy up's Recovery, which finishes or rolls it back with the same per-step check.

## 43. Standardise renames folders and games, and edits two kinds of files other programs own

- **Decision:** **Standardise** (a Tidy up tab) renames and merges a library's console folders to
  one device profile's folder names, and renames identified games to their official DAT names
  with the art, saves, disc descriptions (`.cue`), playlists (`.m3u`) and frontend game lists
  (`gamelist.xml`) that name them, only when the user approves a review, and never deletes
  anything. It writes only inside the library, under the library's `op` lock (Tidy up's), through
  one Tidy up journal (ADR 7) and a second one for the game lists, written once the games are
  settled. Every step checks its folders are real folders inside the library right before it
  makes one or reserves a name (cover art's and card sync's operations, ADR 41 and ADR 42). A
  console folder or a game that is a folder moves in one rename (a `move-dir` journal step whose
  folder's device and inode are recorded, so undo and Recovery move back only that folder). A
  duplicate met while merging moves to the set-aside folder, never deleted. A game list, cue
  sheet or playlist is changed only where it names a renamed file: the original moves to
  `.romperoom/lists-backup/<run>/` and the new file, written to a part file and fsynced first,
  takes its place; Undo puts the original back while the new one still holds what the run wrote.
  A list or sheet that cannot be read with certainty is left as it is and reported. A folder, a
  game and a list are each all or nothing: once a step fails, the rest of its unit and every unit
  that depends on it (a merge on the rename it goes into, a game on the folder move or merge that
  moves it) does not run, and its done steps are put back before the journal closes; a step that
  cannot be put back keeps the journal in Recovery, and the run stops saying "Romperoom couldn't
  put everything back after an item failed. Tidy up's Recovery finishes it or undoes it." When a
  run, its Undo or its Recovery ends, the library's other journals are recorded again with the
  folder as it is now (an audit row, as a reconnect writes), so a remount after the top-level
  folders were renamed does not refuse them; this happens only while the folder is provably the
  one the run's journal recorded (the same device and inode, or the same folder names but for the
  ones that journal renamed) and each of those journals recorded that same library. A library at
  the path with other top-level folder names is refused; one with the same names is taken for the
  same library, as the fingerprint rule already takes it, and is then recorded by its device and
  inode. A rename that landed just before a crash counts as done in Recovery even when something
  took its old name since, so its unit's put-back reports the name taken (the run waits in
  Recovery, or the rollback ends partial) rather than leaving a game half renamed. While a run
  waits in Recovery, a new run of that library and removing the library are refused, so the
  waiting run is never settled against what a later run renamed, nor without its units. When Undo,
  a rollback or a put-back brings back a rewritten cue sheet or playlist that a scan saw
  meanwhile, its catalog row gets back what it said before the run while the file there still
  holds the original bytes, so the next review offers the game again.
- **Why:** frontends and handhelds want one naming scheme, and a library built over years has
  several (owner decisions 1 and 2). Renaming a game without its art, saves and the lists that
  name it would leave them pointing at nothing (owner decision 4); editing those files minimally,
  with the original kept, is the least surprising way to keep them working (owner decisions 5
  and 6).
- **Cost:** two kinds of files Romperoom does not own are now changed (a frontend's game list,
  and a disc's description or playlist), and the library's write boundary grows to console
  folder names and game names. A frontend's own database cannot be updated, so a renamed game may
  need a rescan there. Saves on a card keep their old names: the next card sync pairs saves by
  name, so a renamed game's card save is seen as a new save of the old name. The windows ADR 41
  records apply: a folder swapped for a link between a step's check and its rename, at that
  instant; and on POSIX a folder rename can replace an empty folder that appeared at the
  destination after the check (it holds nothing). The catalog follows each step in its own
  transaction (file and media rows, a folder's mappings), and a moved picture keeps its link to
  its game; a scan afterwards confirms the rest. Known limits, each on the safe side:
  - a game list's media path that is absolute or contains `..` is left unmatched, so it keeps
    naming the old file (frontends write relative paths);
  - a clean relative subfolder path that names a different file of a renamed file's name
    (`./Hacks/tetris.gb` beside a renamed `tetris.gb`) is left alone, not taken for the game;
  - art and saves that two copies of a game would both take along stay where they are, listed as
    taken for both copies;
  - a game whose picture or save is an identical copy that a ticked merge clash sets aside keeps
    its name (`copy-set-aside`: "A picture or save of this game is an identical copy you chose to
    set aside, so the game keeps its name."), rather than be renamed without it;
  - a game split across folders (its `.cue` in the console folder, its tracks in a subfolder) is
    never offered;
  - undoing or rolling back a journal from before a console folder was renamed puts its files back
    under the folder's old name, so that folder exists again beside the renamed one (nothing is
    lost; a later run merges it);
  - a merging folder's own `gamelist.xml` stays in the source folder, so the games moved out of it
    keep their list entries only there, where no frontend reads them for the target;
  - a folder made at a moved folder's old name can pass as the moved folder when the file system
    hands it the same inode number (Linux reuses them at once); undo then renames that folder back,
    and its catalog rows follow it;
  - on a volume where a folder's inode changes on rename or remount (an SMB share without stable
    file IDs), the moved folder is no longer recognised: Undo keeps it where it is ("the folder
    there is not the one that was moved"), and Recovery's finish marks its step failed with its
    catalog rows left under the old name. Nothing moves that should not, but folder moves cannot
    be undone or finished there;
  - a playlist (`.m3u`) that a run renamed without rewriting it, and that identify matched by its
    new name before the run was undone or rolled back, keeps that name match afterwards, so the
    review leaves its game as not matching; a scan does not clear it (the file's size and time
    are unchanged, so it is not read again), but running Identify games again does.

## 44. Re-link renames a leftover picture after the one game that clearly matches it

- **Decision:** a leftover picture (Tidy up's `no-rom` or `rom-gone`) is offered for a rename
  only when exactly one game of its console has its title once every tag is ignored (its DAT name
  when identified, else its file name); the picture takes that game's keeper file name, in its own
  folder, never overwriting anything. The game lists that name it follow it, and a game list entry
  whose game file is gone is listed, and re-pointed (its `<path>` only) on one clear match. It runs
  as a standardise run of kind `relink` (migration 19): the Tidy up journal under the `op` lock
  through the real-folder operations, a second journal for the game lists with the originals in
  `.romperoom/lists-backup/<run>/`, Standardise's Undo and Recovery. Nothing is deleted.
- **Why:** a game renamed outside Romperoom leaves its art behind under the old name, and Tidy up
  would offer to set it aside (owner decision 1: rename the picture, so every frontend finds it).
  One clear match only (owner decision 2) keeps it from guessing; listing, never deleting, the
  entries without a game (owner decision 3) leaves the frontend's list the user's.
- **Cost:** a picture two games could take, or that matches by a misspelling or an index prefix, is
  never offered, and neither is one of a game on several discs recognised disc by disc (each disc is
  its own DAT game). A frontend that wrote a fresh entry for the new name keeps its stale entry
  listed (`game-has-entry`). An absolute media path in a game list is never changed, so that
  frontend still finds the old name gone. When Recovery rolls back the game lists' journal, the
  pictures stay renamed and the lists keep their old text, naming pictures that are gone until the
  run is undone or the frontend rescans; an Undo that keeps a list changed since the run puts the
  pictures back while that list keeps naming the new names; a re-link of entries only whose lists'
  journal is rolled back stays recorded `complete`, with nothing left to undo. A picture whose game
  already has art of that kind shows twice, unticked in the re-link section and ticked in the
  leftover list, so setting everything aside still sets it aside. Re-link shares Standardise's run
  table, so its Undo and History are Standardise's, a re-link waiting in Recovery refuses a
  standardise run of that library and the other way round, and a reload shows only the window's
  latest result of either kind (the other is in History). The review is no job: a review a reload
  forgot is not kept, and like the leftover list it may cache file hashes in the catalog.

## 45. Removing a library forgets it whole, and touches nothing on disk

- **Decision:** Settings › Libraries (since the reorganisation, ADR 57, Home's Libraries) removes a
  library by deleting every catalog row of it: its
  files, pictures, folder lists, identify, cover art, card sync and Standardise records, and (by
  its path) its journals, steps, operations, Delete forever records and reconnect audit rows.
  Nothing on disk is touched. It is refused while any scan runs, while a job holds the library's
  lock, and while Recovery lists anything of it. The last library may be removed: the app goes
  back to setup. Adding refuses a folder already added (it used to return the existing library)
  and one inside or around a library, by real path and by device and inode.
- **Why:** a journal kept for a library Romperoom no longer knows would list its runs in History and
  Set aside, count its set-aside bytes in Health (now Fix up), and let Undo or Finish move files in
  a folder the user asked Romperoom to forget. Refusing while something waits in Recovery keeps the
  journal a stopped run needs. Setup is already the app with no library, and refusing to remove the
  last one would block "I chose the wrong folder".
- **Cost:** a removed library's history is gone: what it set aside stays in its
  `.romperoom-quarantine` folder with nothing offering it back, and adding the folder again starts
  afresh (a scan, identify, card syncs without their agreed saves). The page still passes the
  picked folder's path to `addLibrary` (a known gap, [security.md](security.md#gaps)).

## 46. Copies across libraries are reported, never tidied

- **Decision:** Tidy up's **Across libraries** lists the files (the same whole-file hash) held in
  two or more libraries and does nothing else: no set aside, no catalog write, no lock. A library
  the catalog cannot vouch for, whose folder does not answer (the Libraries tab's probe), or whose
  folder overlaps another library's (the rule that refuses adding it, decision 45) is listed as
  not checked with its reason; the rest are compared. Copies that are one file (the same device
  and inode) are left out.
- **Why:** owner decision (2026-10-07): report only first. Setting a copy aside in one library
  because another library keeps one needs the planner's one-library keeper rules relaxed and a
  keeper that can go offline; a report needs neither. An unplugged drive or a nested folder must
  never read as "no copies", nor a file as a copy of itself.
- **Cost:** the user removes a copy across libraries by hand, or by moving it into one library and
  using Duplicates. The list is as each library's last scan saw it: a file replaced since with a
  different one of the same name is still listed, so the guide says to scan both libraries again
  before removing a copy by hand (a size and modified-time check against the catalog is left for
  setting copies aside across libraries). Two mounts of one network share added as two libraries
  are not told apart, so every file reads as held twice. The look reads each candidate copy's
  device and inode on disk in the main process, as Duplicates does, so a share that stops
  answering after the folder probe can freeze the window until it answers.

Superseded by ADR 51 for setting copies aside; the report itself is unchanged.

## 47. An export folder on a network drive asks first

- **Decision:** an export folder on a network drive, or one whose drive cannot be told, is
  written only after the user ticks **Copy to** (the folder) **anyway** for that folder choice.
  The host checks once per choice when it plans and shows the page `network` or `unknown`; for a
  plan that asked, the start request's `confirmNetwork: true` becomes the plan's own folder path,
  and the writer checks again and refuses unless that path matches exactly. On Linux a card
  mounted read-only (the `ro` mount option) is refused as read-only, like the lock switch.
- **Why:** owner decisions (2026-10-08). A network folder is slower and a dropped connection
  stops the copy part-way, so the player should choose it knowingly, as a network card is refused
  outright; not knowing must ask, never write. The read-only mount was offered and failed at the
  first write.
- **Cost:** detection is per OS and incomplete: Linux network file systems outside the list
  (`fuse.rclone`, `davfs`, `fuse.glusterfs`, for example) and a Windows mapped drive whose real
  path stays a letter read as local; macOS autofs mounts ask though local. On Linux an old
  read-only mount under a newer writable one on the same mount point refuses the card (it fails
  safe). Linux is tested from hand-written recorded output only, and the Windows rule against
  expected paths only. The question appears in the Where step only for what the plan saw; a
  share mounted over the folder later is refused at Preview or Write with "choose the folder
  again".

## 48. Copies that differ send Tidy's keeper, and say so

- **Decision:** when one game holds a file in two or more libraries with different bytes under
  one card name, the card gets the copy Tidy up's `rankCopies` ranks first, with the package's
  region order, and the Check step names that copy's library (**Copies that differ**). The
  keeper's `identified` input is the file's own content match with a game database, not the
  game's confidence. A game holding a playlist or sheet (`.m3u`, `.cue`, `.gdi`, `.ccd`, `.toc`)
  or discs of one release, a copy with no SHA-1 or 0 bytes, or two different copies in one
  library (two of its folders) still leave the game out as a name clash.
- **Why:** owner decision (2026-10-08): pick one and say so, rather than leave the game out or
  ask per game; the decision is about one game in two libraries, so a library is never picked
  from against itself. Every copy linked to one game shares its confidence, so only the file's
  own match can tell a good dump from another. Picking a disc or a track per file would mix two
  libraries' copies in one set.
- **Cost:** region and revision never decide (copies of one name share their tags): without a
  game database the copy found first wins, and a rescan that finds a file anew can change which.
  Two different copies in two folders of one library still leave the game out. Most art carries
  no checksum and is still hashed. The note does not say why a copy won.

## 49. Saved packages live in the page's storage

- **Decision:** the card wizard's saved packages (named choices per device) are kept in the
  renderer's localStorage under `romperoom.deploy.packages.v1`, a versioned list beside the
  remembered last choices (`romperoom.deploy.v1`) and independent of them. A package holds the
  device and every choice of the What step, never a destination. Loading is explicit, replaces
  every choice and keeps the card or folder already chosen.
- **Why:** owner decision (2026-10-08): named selections per device, on this computer, without
  the card. The choices already live in the page, hold no path, and are validated again by the
  engine's schema when planned; a catalog table would add a migration, channels and bridge
  methods for data only the page reads. A second key means a damaged list never costs the last
  choices, nor the reverse.
- **Cost:** packages are lost with the app data folder and do not travel to another computer.
  Every change reads the list again, but two windows could still race (the app opens one).
  Packages of a device that is gone are kept but never listed: the card counts them and removes
  them all together when the player asks, never one by one, and they still count toward the 400
  read in all (else a new package past them would be lost on the next read).

## 50. Make it fit suggests, and the player applies

- **Decision:** when the games do not fit, the wizard asks the engine for a suggestion (the
  largest games first, then a game with another version going, never a game the player kept) and
  shows it for review; nothing is left out until the player presses **Leave out selected**
  (decision 63: every suggested game ticked, any can be unticked).
  The applied list travels as `selection.excludeGameIds` (the excluded games, not an allow-list
  and not a held id), is left out after regions are picked, and is never stored: any change of
  choices or destination forgets it.
- **Why:** leaving out a player's games without their review is the one thing this feature must
  never do, and a suggestion that changes while it is read could. The excluded list is as long as
  the suggestion (an allow-list is as long as the library; a held id expires with the plan and
  needs a channel), and left out after regions so a left-out USA game never brings in its
  European version.
- **Cost:** largest first can leave out a big game when only a little is over (one press of
  **Keep this one** asks again); asking plans the package again up to three times; a list belongs
  to one plan, so changing anything means asking again.

## 51. Copies across libraries are set aside in their own library

- **Decision:** **Set aside the extra copies** on Across libraries moves each extra copy into the
  set-aside folder of its own library, never onto another drive, and never deletes. The library
  that keeps a set is chosen per set (suggested by Duplicates' keeper rules). A run locks its
  library and every kept library together, checks each kept library answers, is not empty and
  does not overlap the others by real path, and re-hashes each kept copy just before its step moves. The journal records
  its kept libraries (`op_keeper_root`), so Finish and Delete forever check them again (Delete
  forever also checks the kept library is still in Romperoom); Undo and Roll back need only the
  library the copies sit in. "The kept copy" means a proven copy with the same bytes in that
  library: the recorded file, or, when nothing is at its path any more (Standardise renamed it),
  a present copy the library's catalog holds with that hash, proven by the same checks.
- **Why:** owner decision (2026-10-09). A copy set aside because another library keeps one is
  safe only while that other copy provably exists, so every later step that could lose the last
  copy checks it again; a set-aside copy staying in its own library keeps Put back a rename.
- **Cost:** a copy whose kept library is unplugged, removed or changed stays set aside (Delete
  forever keeps it; Put back always works). Finish waits for the kept library when a file it
  has left to move is kept there (a step that already landed needs nothing of it), and Recovery
  offers to reconnect that library. Copies inside the
  kept library are left for Duplicates. The look is kept only while it is the newest, so a plan
  made from an older look is refused and the player looks again.

## 52. Card art is made smaller by Romperoom's own PNG code

- **Decision:** a device profile may give a media kind a `maxWidth` (Onion's box art: 250
  pixels). The card writer then writes a smaller copy of a PNG picture, made by Romperoom's own
  PNG reader, resizer and writer on `node:zlib` (`packages/engine/src/image/png.ts`), at write
  time, in the main process. The source is copied as it is when it is no wider, cannot be read,
  or when the smaller copy would take more bytes. The manifest records the width and a version,
  and a later copy compares those instead of the bytes.
- **Why:** no image dependency (a native module needs install scripts, which the repository turns
  off) and no JPEG code to own; the resize is deterministic and tested byte for byte. A smaller
  picture saves card space and is what the device shows anyway.
- **Cost:** JPEG art is copied at full size. The plan counts art at full size, so a card keeps a
  little more room than shown. About 15 ms a picture in the main process: a large copy makes the
  window answer a little slower. Raising `RESIZE_VERSION` makes every card's smaller pictures
  again.

## 53. BIOS files are recognised by libretro's list

- **Decision:** Download for me also offers libretro's `dat/System.dat`, from the same pinned
  commit, stored whole in the catalog. A card copy reads each BIOS file's SHA-1 and copies a
  recognised file under every name the list gives it for the chosen consoles; an unrecognised
  file keeps its own name. Which consoles need a BIOS is an explicit table (`BIOS_SECTIONS`), not a
  match of names.
- **Why:** players keep BIOS files under many names, and a device finds them only by the exact
  name its emulator looks for; the checksum is the only reliable identity. The list comes from the
  source the app already trusts for DATs.
- **Cost:** a file listed under several names takes card space once per name. Only files directly
  in a BIOS folder are read. A "missing" line means the list names files for a console and none
  was found, not that every game needs one. The table is maintained by hand.

## 54. ScreenScraper uses the player's account and fills gaps by checksum

- **Decision:** Look up on ScreenScraper uses the player's own account, kept by the system
  keychain through Electron's `safeStorage` (never on a computer without one), and Romperoom's
  developer details, which are not in the repository or the build (an unpackaged run may read them
  from the environment; release builds wait for the maintainer to register Romperoom). It asks
  one game at a time, at most 1,000 a run, stops at the daily limit, and fills a gap only from an
  answer that matched the file's checksums. The review asks nothing and says exactly what each
  lookup sends: the file's checksums, name and size, its console, the player's user name and
  password and Romperoom's app name, developer id and developer password (plus `output=json` and
  `romtype=rom`), then one request per picture with the game's number, console and media name.
- **Why:** ScreenScraper's terms tie lookups to an account and a registered app; a name-only match
  would attach the wrong picture with confidence. Credentials in a file or a build would leak.
- **Cost:** it cannot be used until the maintainer registers Romperoom. Its statuses and answer
  shapes are written from its documentation and not yet confirmed live. The account and
  checksums travel in the query, as its API requires (the log keeps host and path only).

## 55. The screenshots show generated covers

- **Decision:** the published screenshots show cover art drawn by a committed generator
  (`packages/engine/test/fake-cover.ts`): pure TypeScript into an RGBA buffer, written with the
  engine's PNG writer, seeded by the title, kind and console. No third-party picture, font or logo
  is used; the only words drawn are the fixture's titles and a closed list of the generator's own.
- **Why:** real box art cannot be published in the repository; plain colour boxes made the
  screenshots unconvincing. Drawing in Node (not SVG in Chromium) keeps the pixels the same on
  every machine, with no dependency.
- **Cost:** the pictures are simple; a change to the generator changes every screenshot with art
  (its pixel hashes are pinned, so the change is deliberate).

## 56. One rule for a game's cover

- **Decision:** a game's cover is its first present linked picture by kind: box art, then a
  screenshot, then a title screen, then any other picture (marquee, wheel, unsorted), lowest id
  first within a kind; never a video or manual. One SQL fragment (`art/cover.ts`) builds both the
  games list's query (`listGames`) and Tidy's duplicate-set picture, and a test runs both over
  one catalog. ES-DE's `miximages` and `3dboxes` keep kind `unknown`: a last-resort cover, never
  box art (owner ruling: Get cover art and the card copy must not take them for a box).
- **Why:** players with art on disk saw placeholders because only box art and screenshots could be
  covers, and two copies of one rule can drift apart unseen.
- **Cost:** a marquee or wheel can now be a game's cover until better art exists; a mix image
  shows only when a game has no box art, screenshot or title screen.

## 57. Home and Fix up replace Library, Health and Tidy up

- **Decision:** the menu is Home, Fix up, SD card and Settings. Home is a dashboard (search, what
  needs attention, recent games, libraries, a bounded console grid) and each console has its own
  page; Fix up holds Health's and Tidy up's screens as Needs attention and five tasks. Routes take
  a part (`#/home/<console>`, `#/fixup/<task>`, `#/deploy/<choice>`); the 0.11 hashes redirect.
  One deriver (`home/attention.ts`) lists what needs attention on both screens. Fix up's header
  holds the scan (**Scan all libraries** with more than one) and **Library to tidy**, which says
  which library Duplicates, Standardise, leftover artwork, Re-link and Set aside work on; each of
  those panels is keyed by that library, so nothing made for one acts on another.
- **Why:** with dozens of consoles a row of console tabs and seven Tidy tabs became unusable,
  and jobs were found by where they lived, not what they do.
- **Cost:** every screenshot and much of the guide changed at once; the data-safety screens moved
  without changing, which leaves some of their wording ("set aside") as it was. The messages that
  named Tidy up (the busy word, an interrupted sync's undo, Discard the rest) now name Fix up.

## 58. The games page reads by offset

- **Decision:** a games page asks the engine for the total and each letter's offset in title
  order (`gameLetters`, read-only, the same filter as `listGames`) and reads pages of 200 by
  offset where the window is, never the pages in between.
- **Why:** a letter jump or End in a 50,000-game console would otherwise read every page before
  it, each a full grouping query.
- **Detail:** the page chosen by the aggregates alone (an inner query), the per-game pictures and
  sizes worked out only for its 200 rows; a scroll reads the pages where it settles, not every page
  it passed, while a jump (a letter, End) reads at once.
- **Cost:** one more IPC channel; a letter is the first character only (no multi-letter jump
  inside a list that is not all read), and titles starting past Z (accented) have no letter.

## 59. Tab names stay two words; Settings keeps what is about the app

- **Decision:** Fix up's "History and undo" is labelled **History** (the naming rule allows two
  words); Libraries moved to Home and Game databases to Fix up as each got its new home, and
  Settings keeps Appearance, ScreenScraper and Network activity.
- **Why:** short nouns are what the owner chose for every menu and tab; settings are about the app,
  not the library.
- **Cost:** "History" also holds Set aside and Recovery, which its name alone doesn't say (the
  panel's own headings do).

## 60. A profile's ROMs may sit at the card root, and the writer stays in its folders

- **Decision:** `root.roms` may be `.` (`CARD_ROOT`), the card root itself, for the R4xMP
  flashcart, whose kernel reads `NDS ROMS` and the other folders there. Only `root.roms` may be
  it; with it, no BIOS, media or saves folder may be set. Paths are joined by one helper
  (`cardPath`, `romsFolderOf`), the writer manages each system folder rather than the root, and
  it refuses any plan naming a file outside the profile's folders.
- **Why:** an empty `root.roms` would have made every joined path start with `/`, and managing
  the root would have put the player's kernel (`__rpg/`, `_DS_MENU.DAT`, `_DSMENU.DAT`) inside
  Romperoom's reach. An explicit `.` reads as meant, and the writer's check holds whatever plan
  it is given.
- **Cost:** a profile can no longer write a file straight into the card root (none did), and a
  profile at the card root has nowhere for BIOS, art or saves until a source says where.
  Profiles gained two optional fields on the way, `notes` (what the sources leave unconfirmed)
  and `limits.maxCardBytes` (a warning only).

## 61. A profile may offer consoles unticked

- **Decision:** a system may be `optIn` (with the profile's `optInNote` saying why). The wizard
  lists it unticked with that note as its description, "Select all" ticks it, and a package that
  names no consoles leaves it out with a warning naming it (the planner and the wizard both read
  `optInSystemIds`; `defaultSystemIds` is the complement, for callers listing what goes by
  default). Choosing any other device starts from that device's defaults, never from consoles
  ticked for the last one.
- **Why:** the DSpico plays DS games itself but GBA, GBC and NES only through emulator programs
  the player installs, with file types nobody confirmed; copying those by default would fill the
  card with games it may not run.
- **Cost:** one more field in the profile summary; "Select all" on such a device stores the list
  it ticked, so a console added to the library later is not ticked by itself.
## 62. Scan reads drives and names its scope; Refresh list re-reads the catalog

- **Decision:** a button that says **Scan** always reads the drives and always says what it reads:
  **Scan all libraries** (two or more), **Scan** and the library's name (one library, also when it
  is the only one), or **Scan these consoles** (after a card sync). A button that asks Romperoom
  again for what it already knows says **Refresh list**, with "Checks what Romperoom already
  knows. It does not read your drives." under it (the card lists say "Checks which cards are
  plugged in. It does not read them."). Fix up has one scan button, in its header, with "Reads
  your drives for new, moved and removed games. Takes a while for big libraries." under it; a task
  that needs a scan says why and points at it ("Press Scan all libraries above."). A task keeps a
  scan of its own only where it reads fewer libraries than the header: Duplicates' **Scan** of the
  library shown, when there are several. Home's Libraries keeps each row's **Scan** of that
  library. The words live in `renderer/scan/scan-copy.ts`.
- **Why:** the owner could not tell which button did what: **Scan again**, **Scan now**, **Scan
  all libraries** and **Look again** sat on the same screens, some reading the drives for minutes
  and some only asking the catalog again.
- **Cost:** Across libraries and Standardise's results no longer have a scan of their own: they
  now point at Fix up's header scan, which scans every library, so the scan they lead to is slower
  on a big collection. Across libraries used to scan only the libraries a scan would fix.
- **Known trade-off:** Standardise's results used to scan only the one library just renamed, a
  smaller set than the header's. By the rule above that would keep a button of its own, as
  Duplicates does; the owner's approved list removes it anyway, to keep one scan button per
  screen, accepting a full scan after a rename when there are several libraries. The engine's own refusal messages ("look again") are unchanged; they only show
  behind **Show details**.

## 63. Suggestions are accepted with one shared select-and-accept bar

- **Decision:** the three lists of suggestions (folders with a suggested console in setup, name
  matches in Fix up › Names, and the games Make it fit suggests leaving out) share one bar,
  `SelectionBar` with `useSelection` in `packages/ui`: a checkbox on every suggested row, and above
  the list "Select all N", the count, and one button that names it ("Accept selected (12)",
  "Leave out selected (3)"). "Select all" reaches every page, not just the rows shown. Nothing
  happens until the button is pressed. It replaces "Accept all on this page"; each row's own
  buttons stay.
- **Why:** the owner asked for a way to accept every suggestion, and for multi-select wherever
  there is an accept. One component keeps the three screens alike (Space, Shift-click for a range,
  a mixed "Select all", the count in the button) and styled from one place.
- **Detail:** the selection is held as the items that differ from the default (none ticked, or
  every suggested game ticked on Make it fit), so it survives paging, and it is counted over the
  items on the list as last read, without repeats: one that leaves (decided here or in another
  window, a kept game) drops out of the count and is forgotten. While the action runs every tick,
  "Select all" and (for folders) the pickers are frozen (`aria-disabled`, "Accepting…"), so what
  is sent is exactly what was ticked at the press, and the count shown stays the one pressed.
  Each press does what the per-row action does: one `assignFolder` per folder in turn (with
  progress, a failed folder named and left ticked), `identifyDecideReview` in batches of
  `MAX_REVIEW_BATCH` (a failed batch stops the rest, is reported, and the list is read again),
  and the same left-out list as before, with every unticked game kept (pinned) as Keep this one
  keeps it.
- **Name matches:** a tick is for a match, the file and its game (`fileId:gameId`), so a match
  rejected and matched again (one by one or by "Reject all on this page") comes back unticked.
  "Select all" with pages not loaded reads every match once with the existing
  `identifyListReview` calls, a page at a time, showing its progress; a failed read is shown and
  announced and never applies later, and the matches read are added to the player's ticks, never
  in place of them. The matches read are kept current in place: this screen's own decisions take
  their files out of it (a batch whose answer was lost too, so it is never counted again), and
  anything else that changes the list (an identify run, a scan) reads it again. No new IPC
  channel or engine function.
- **What N promises:** the number of ticked items on the list as Romperoom last read it. The host
  only changes name matches still waiting, and decides by file: if the catalog changes between
  that read and the press (an identify run elsewhere matching a file to another game), what is
  kept can differ, and the announcement uses the host's own count. Guarding that fully needs the
  engine to accept a file only as the game shown, an engine and IPC change left for later.
- **Cost:** "Select all" on a very large review reads every page once, and again only when
  something else changes the list; each batch still refreshes the loaded pages, health and the
  games lists as a single Accept does. The Make it fit games not listed can only be ticked or
  unticked together. There is no bulk Undo: each folder is under Recently assigned with its own
  Undo, and accepted name matches have none (as before).

## 64. Zipped games are unzipped on copy for devices that need plain ROMs

- **Decision:** a profile's system may set `extractArchives`. For such a system a `.zip` holding
  exactly one of the system's ROMs (by `extensions`) is copied as that ROM, named after the zip
  (`<zip stem>.<ROM extension>`, then the card's name rules); its other entries stay behind. A zip
  of several games or none, one that is locked, packed other than stored or deflated, or whose
  sizes cannot be true, and `.7z` and `.rar`, are left out with their own skip reasons. The
  R4xMP and the DSpico set it on every system; every other device copies zips as before. The
  room, the meter, the file-size limit and Make it fit count the unzipped size. The writer
  streams the one entry into the usual part file, checks its size and CRC-32 (and the catalog's
  SHA-1 when that is of the entry), and fails that one file when the zip lies.
- **Why:** the owner: the R4 card needs every game unzipped on the SD card, and the R4 and the
  DSpico should copy games unzipped. Most libraries keep cartridge games zipped, so leaving zips
  out (as both profiles did) left most of a library off these cards.
- **Detail:** the planner stays pure: the catalog reader decides each zip from its central
  directory (`zipVerdict`, synchronous on the main process: the last 22 bytes, up to 64 KiB more
  for a zip with a comment, then the directory, a few hundred bytes for a one-game zip), because
  the catalog keeps entry names only for zips of 2 to 64 files. Only the verdict is remembered
  (a few numbers per zip, never its entries), and each plan spends at most about 2 s reading zips
  it has not seen; the rest are "not checked yet" and the next plan carries on. Entries a computer
  adds (Finder's `__MACOSX/` and `._` files, `.DS_Store`, `Thumbs.db`) are never a game. The name never comes from inside the
  zip, so no entry can choose where its bytes land. The manifest needs no new field: it records
  the card file's size and SHA-1 and the zip's time, so an unchanged zip is skipped and a changed
  one copied again. Sync matches an unzipped card game to its zip by the zip's game hash, or, for
  these devices only, by its entries when the game is not the zip's largest file.
- **Cost:** a zip of several games (a multi-game pack) is not split up, and 7z and rar are not
  unzipped (roadmap). On a large zipped library on a slow drive, the first few plans for such a
  device leave some zips "not checked yet" until the remembered verdicts cover them. Whether the R4 kernels and
  Pico Launcher accept the unzipped files is as unconfirmed on a real card as the rest of those
  profiles.

## 65. Junk files are removed by content, after a preview

- **Decision:** Romperoom removes three kinds of junk file, and only these: `._` AppleDouble
  files, `.DS_Store` and `Thumbs.db` (the last two in any case: on Windows, FAT and macOS
  `thumbs.db` is the same name). A name is never enough: each file's contents must be
  metadata of its kind. An AppleDouble file needs its magic `0x00051607`, version 2 or 1 and a
  sane entry list, and must hold only metadata: no data fork, a resource fork no longer than the
  286-byte empty one macOS writes, and no other large entry. A `._` file with a real resource fork
  is someone's data (on FAT, exFAT and SMB it is the only copy of a classic Mac program's or
  font's resources) and is listed as "it holds a resource fork". `.DS_Store` needs `00 00 00 01
  Bud1` and a root block whose two offsets agree and lie inside the file. `Thumbs.db` needs the OLE
  signature with a known version, byte order and sector size. Each must be no larger than that
  kind of metadata is (1 MiB, 16 MiB, 64 MiB). Only a regular file with one name is removed,
  through real folders inside the place being cleaned (checked with `lstat` folder by folder and by
  real path), never inside Romperoom's own `.romperoom*` folders, a system folder, or another drive
  mounted inside. Removal is permanent (they are only metadata: no set-aside copy), after a list
  and one confirmation. Each file is checked again (its device, inode, size and time from the
  list, and its contents) and then removed by rename and verify: renamed to a unique name in its
  folder, confirmed to be that very file, and only then unlinked; anything else goes back.
- **What that guarantees:** on FAT and exFAT (cards), which cannot hold links, no removal can be
  led outside the place. Elsewhere (APFS, a NAS) the rename and verify step catches a folder
  swapped for a link between the check and the removal; what remains is the instant between the
  last look at the unique name and its unlink, which a swap could only use with a file of that
  same random name. On SMB, `nlink` is not reliable, so a hard link may not be recognised; removing
  it removes only that one name.
- **Where:** Fix up › **Junk files** works on the library chosen above the tabs (it holds the
  library's lock, all or none, and records one History row, shown as Deleted forever). The SD
  card's Done screen offers **Clean up this card**, which reads only the folders the device uses
  (and the `._` beside each of them), never the card's other files. After a copy from a Mac to a
  FAT or exFAT card, the copy itself removes the `._` files macOS made for it: `._X` beside a file
  X the copy wrote or replaced, and `._D` beside a folder D the copy made, inside the device's
  folders, and only when the `._` file was made or changed during the copy (modified at or after
  the copy's start, read on the card's own clock). An older `._X` beside a file the copy replaced
  is left; on a Mac's FAT card the system itself replaces it with the new file's on the rename,
  measured. That is on by default and can be turned off on the Check step (**Remove the ._ files
  macOS adds**, remembered for next time). Anything else needs the manual list.
- **Stop and progress:** a look says how many entries it has looked at, a removal how many files of
  how many it has done; **Stop** ends either. A stopped look removes nothing; a stopped removal
  keeps what it removed (the file in hand is removed or put back first), releases the library and
  says so.
- **Words:** the button that reads the folders names them (**Find junk files in** the library, **on**
  the card), as decision 62 asks; it is never **Refresh list**, which promises not to read a
  drive.
- **Left alone, and why:** macOS's system folders (`.Spotlight-V100`, `.Trashes`, `.fseventsd`
  and the like, in any case), a Linux `.Trash-<uid>`, and a NAS's or Windows' housekeeping folders
  (`@eaDir`, `#recycle`, `$RECYCLE.BIN`, `System Volume Information`, `lost+found`) are listed as
  left alone, never opened: they belong to the system, and removing them can upset it. So is a
  drive mounted inside a library. The card manifest's own `._` file sits at the card root, outside
  the device's folders, and stays.
- **Why:** the owner found these files in the game folders and asked for them to go, and for the
  cards not to get them. The scanner already ignored them (they never reach the catalog, so a
  copy never carries one from the library); what was missing was a way to remove them, and
  macOS writing its own beside every file it copies to a FAT card. Checking the contents keeps a
  game that happens to be called `._x.nes`.
- **Cost:** a look reads every folder of a library (like a scan, but no hashing), so it takes a
  while on a big NAS (it can be stopped). One look keeps at most 50,000 files; past that it says
  so, and the player removes those and looks again. History records the count, not each file, and
  only once the removal has ended: a crash part way leaves no row. A read-only drive (a NAS mounted
  read-only) refuses the first removal: nothing changes and the result says so, with no retries.
  A busy file (a virus scanner or an indexer holding it, common on Windows) is tried up to three
  times, then reported.
  The SD card option applies only to a Mac copying to a FAT or exFAT card; an export folder on such
  a stick is cleaned with **Clean up this card**. Sync a card's own writes to a card (a save, the
  card's id file) are not cleaned after the sync yet (roadmap); their `._` files are left on the
  card.
## 66. Merge picture folders: Imgs into images, the larger stays

- **Decision:** Fix up › Artwork has a **Merge picture folders** section. In each console folder
  with both an `images` and an `Imgs` folder directly inside it (folder names compared without
  letter case), the pictures in `Imgs` move into `images`. Two pictures have one name when the drive
  says so: the `Imgs` picture's name, looked up in `images` (lstat), reaches a file there, the entry
  with the same device and inode, and only when the two names are also one once folded
  (`foldName`): a lookup that reaches a file by another name (a Windows 8.3 alias) is never taken.
  A string fold alone never decides it: FAT32, exFAT and NTFS fold less
  than full Unicode case folding (`Straße` and `Strasse`, `ﬁle` and `file` are two files there).
  Of two pictures with one name, the same bytes keep the one in `images` and set the `Imgs` copy
  aside; otherwise the larger stays in `images` and the smaller is set aside, and on a tie of size
  the one in `images` stays. Set aside is the library's `.romperoom-quarantine`, never a deletion.
  Only pictures move: regular files with one name (no hard link) of the scanner's picture types
  (png, jpg, jpeg, webp, gif), not junk or hidden files; anything else, a folder inside `Imgs` and
  a link stay where they are and are listed. `Imgs` is removed only when the run emptied it of
  pictures and holds nothing else, after reading it again (and `rmdir` refuses a folder that is not
  empty).
- **Why:** the owner's library has both folders in many consoles, written by different tools
  (Onion OS uses `Imgs`; gamelist.xml tools use `images`); Romperoom never makes either (its own
  art is in `.romperoom/media`) and already reads both, so one folder per console is the owner's
  wish, not the app's need. The larger picture is usually the better scrape; keeping it (and
  setting the other aside, not deleting it) loses nothing.
- **How it runs:** the look reads only and is held for the window that asked (`tidy:mergeArt`).
  The run (`tidy:mergeArtRun`) takes the library's `op` lock, reads both folders again with every
  content hash and refuses if anything shown differs or the catalog changed (junk files written
  meanwhile, `.DS_Store`, `Thumbs.db` or `._` files, are left out of that comparison: they never
  move), then runs one Tidy up operation: a journal of `move` and `quarantine` steps, each
  re-hashing its file against that read and refusing a link on its way just before it moves, never
  overwriting. A set-aside `Imgs` copy is guarded by the copy kept in `images`. When a smaller
  picture was set aside to make room and the larger then fails to move, the set-aside is undone at
  once under the same lock (a put-back that is blocked, or fails because the drive went, is counted
  and said, and Undo retries it). Both steps of such a pair are always in one journal (`together`:
  a journal boundary never falls between them), so a cancel or a lost drive between journals never
  splits them; a run stopped inside a journal leaves the pair to Recovery. The catalog's media row follows each picture in the step's own
  transaction, so a game shows the same art with no scan. History, Recovery and Undo are Tidy up's
  own; Undo puts every file back, makes a removed `Imgs` again on the way, and links the pictures
  it put back by the scanner's own rule (any Tidy up undo that moved a picture), so a scan in
  between no longer leaves them unlinked. That link is the whole library's (`linkMediaForRoot`, as
  a scan's link phase): measured on a synthetic 50,000-picture catalog on a Mac, about 0.4 s when
  every row changes and 0.07 s when none does. If it fails, Undo still reports the files put back
  and says the links follow at the next scan. Undoing a set-aside ROM does not relink its game's
  pictures (a scan does).
- **Left alone (and listed):** a console whose `Imgs` or `images` is a link (or reached through
  one), one with two folders spelled `images` (or `Imgs`) in different letter case (possible only
  where case counts), one where both names are one folder, and one where a game list names a path
  inside `Imgs` (moving those pictures would break its entries; Romperoom does not rewrite game
  lists for this). Every list a frontend keeps in a console folder counts, in any letter case
  (`gamelist.xml`, `miyoogamelist.xml`, any `*gamelist*.xml` or `.json`, Pegasus's metadata
  files); one that can't be read as UTF-8 text (UTF-16, binary, over 16 MB) fails closed and the
  console is left alone. On a volume that ignores letter case, `Images` and `images` are one
  folder: the pictures go into it as it is spelled, and nothing is merged into itself.
- **Cost:** an `Imgs` that was empty before is never removed, nor one a stopped run emptied (the
  next merge leaves it too), nor one that still holds junk (`._` files on a Mac-written FAT card):
  the result says how many hidden files stay, and the owner deletes those by hand. ES-DE's own
  lists outside the library, a list in another console folder or one folder up, and Onion OS's
  caches and `config.json` `imgpath` aren't checked. On Onion OS `Imgs` is the box-art folder: the
  owner chose `images`, so Onion reading the library directly loses that art (a card written with
  the Onion profile gets it in `Imgs` again). A frontend reading `Imgs` straight from the library
  folder (not a card Romperoom wrote) no longer finds the pictures. The run reads every picture it
  moves twice (the run's own read hashes it, and each step hashes it again just before it moves),
  and the copy kept in `images` once more for a set-aside, so a big `Imgs` on a NAS takes as long
  as reading it two or three times. A hard link is never moved: the journal's per-step check reads
  no link count, so the run's read under the lock is the last point a hard link is caught.

## 67. JPEG and GIF art is converted to PNG on the card only

- **Decision:** a device profile whose media format list has `png` but not a picture's own format
  gets a PNG of that picture: JPEG and GIF for muOS and Onion, GIF for ES-DE and Batocera. Only the
  card gets it, as `<rom stem>.png`; the library's file is only ever read (the owner: "one extension
  for all images", on devices only). A game's PNG of the same kind wins over its JPEG or GIF, and
  the plan says so. JPEG is read by **jpeg-js 0.4.4** (BSD-3-Clause, pure JavaScript, no install
  script), a dependency of the engine only; a GIF's first frame by Romperoom's own reader. The plan
  checks each picture's structure (a JPEG's every segment and scan, never its pixels), with a time
  budget (as zips), and counts the PNG at the largest it can be (`pngBound`); the writer has each
  picture converted in a worker thread, one at a time under a time limit, made at most `maxWidth`
  wide with the existing resizer, and never writes more bytes than the plan counted. The manifest
  records `converted: { from, maxWidth?, v }`.
- **A PNG made earlier is kept when its picture cannot be checked:** a picture that is unreadable,
  changed since the last scan, or not checked yet this time does not remove the PNG Romperoom made
  of it on an earlier copy: the plan holds its card name (`DeployPlan.keep`), and the writer keeps
  the file and its record, says so, and decides about it on a later copy. Only art the player no
  longer copies (the game or the kind left out, the picture gone from the catalog) is moved aside,
  as any file. Verdicts are remembered in memory only, so after a restart pictures past the time
  budget are "not checked yet" again; with the PNG kept that costs a step back and on, never art.
- **Left out, said why:** WebP (no decoder chosen yet), CMYK JPEGs (jpeg-js reads Adobe CMYK, but
  without the picture's colour profile its colours are a guess; refusing is plainer than showing
  wrong colours), 12-bit, lossless and arithmetic-coded JPEGs (jpeg-js cannot read them), JPEGs
  whose decoding cannot be bounded (more than 100 scans, a scan order the standard forbids, less
  than a bit of data per block), pictures of more than 2048 × 2048 pixels (4,194,304, any shape) or
  16 MiB, and damaged ones. EXIF orientation is ignored, as most handheld
  frontends ignore it; a GIF's background colour is not painted (transparent, as browsers show it).
- **Why not omggif:** the owner approved omggif 1.0.10 for GIF. A seeded fuzz of its reader found a
  damaged LZW stream that makes it loop for ever (its code table gets a cycle), on the main process.
  Guarding it would mean running an LZW decoder first anyway, so Romperoom reads GIF itself (about
  120 lines, strict, bounded by the file's bytes) and keeps omggif as a devDependency that makes the
  tests' GIFs and gives a second opinion on them.
- **Why a worker thread, with a time limit and a scan check:** the first version decoded on the
  main process, trusting the pixel cap to bound the work. The review showed it does not: jpeg-js
  decodes every scan over every block, so a 66 KB progressive JPEG of 3,000 tiny refinement scans
  froze the whole app for 26 s and ignored Cancel, and a legal 2048 × 2048 picture still blocks for
  about 1.2 s (7 to 8 s on a machine seven times slower). Now the JPEG's structure is checked first
  (more than 100 scans, an illegal scan order or too little data is refused before decoding, so
  the bomb costs well under a millisecond), and decoding, resizing and PNG encoding run in
  `image/picture-worker.ts` (a worker thread of the app's process) with a 10 s limit per picture:
  past it the worker is terminated and only that picture fails; Cancel terminates it at once.
  The worker ships unpacked with its imports (the engine's `dist/src/image` and jpeg-js), as the
  hash worker does, and `verify:package` checks that closure. `--self-check` starts it for real
  at the path the app computes and converts an embedded 13 × 7 JPEG (`picture-worker`), so the
  packaged smoke on every platform fails if the worker cannot start there. The limit is the backstop for what
  the checks cannot foresee; the pixel cap stays at 2048 × 2048 because box art needs far less
  and the worst legal picture fits the limit even on a slow machine.
- **Cost:** a PNG of a JPEG is several times larger, and the plan counts each at its largest (every
  row raw), so a PNG-only card keeps more room than it shows: about 1.1 MB counted for a 512 × 720
  box art that ends at a few hundred KB. A picture that only fails at write time (damaged image data
  behind a good header, or past the time limit) is a failure of that file, said with its reason,
  and the run is `partial`. A picture at the cap holds about 390 MB while it converts (typed
  arrays, outside the worker's heap limit). The plan reads each JPEG whole to check its scans,
  which costs more of its time budget on a slow share than reading a header did.
  An older Romperoom reading a card with a converted picture says the card was prepared by a newer
  version (the manifest field it does not know), as any new manifest field does. Raising
  `CONVERT_VERSION` converts every card's pictures again.

## 68. The app updates itself from signed checksums, without code signing

- **Decision:** Settings › Updates checks the public releases repository when the player presses
  **Check for a new version** (or once as Romperoom starts, when they turned that on), and on
  **Install version X** downloads this computer's file through the game database transport. It
  then installs it (Windows runs the per-user NSIS installer silently and restarts; a Linux
  AppImage is swapped in place and restarted) or shows it (macOS, a `.deb`, the Windows zip).
  Before anything runs, `SHA256SUMS.txt` must carry an Ed25519 signature by one of at most two
  public keys compiled into the app (`src/main/updates/trusted-keys.json`), the file is chosen by
  its exact name from that signed list, and its size and SHA-256 must match, checked while it streams and
  again just before it runs. The installer runs, or the AppImage is replaced, only from an
  updates folder private to this user (on macOS and Linux owned by them with no other access; on
  Windows only the app's own per-user data folder, never one `ROMPEROOM_DATA_DIR` moved);
  otherwise the verified file is shown, with the reason. Never a prerelease, never an older
  version, never while a copy, Tidy up, Standardise, a scan, a junk removal or any other library
  job runs (asked at the press and again after the download). From the moment an automatic
  install begins, every channel that starts work refuses ("Romperoom is closing to install an
  update"); once it is restarting, so does every update channel but the status. The release
  workflow's `sign` job signs with the `UPDATE_SIGNING_KEY` secret of the `release-signing`
  environment (tags `v*` only, on the self-hosted Linux runner) and proves that a changed byte and
  a stranger's key are refused. The database button became **Check game database updates**.
- **Keys:** `ci-2026-10` is the release key; a cold backup key (`cold-2026-10`, generated by the
  owner offline) is to be added as a second trusted key in a later release, signed by `ci-2026-10`
  while it still works. The first updater release (0.15.0) trusts only `ci-2026-10` (owner
  decision, 2026-10-12). <!-- second-key-pending -->
- **Why not electron-updater:** it trusts the release metadata, and for safe installs leans on
  code-signed builds, which do not exist yet (ADR 16); its macOS path needs a signed app; and it
  would be a second network stack beside the allowlisted transport, with its own redirects and
  proxy rules. Our own updater is small, runs on the client that exists, and checks one signature
  we control.
- **Why one signature over the checksum list:** the list already names and hashes every file, the
  release workflow already makes and checks it, and one signature covers every file. The version
  is bound too: the name the app looks up carries it. The `.sig` may hold two lines, one per key,
  so a release can be signed by an old and a new key while copies that trust either update.
- **Why one redirect is allowed:** `github.com` serves a release file by redirecting to GitHub's
  asset host with a signed URL. The transport follows exactly that hop, to two exact host names,
  logs it, and refuses a second redirect. Chromium's resolver rules and the dead proxy stay as
  they were (ADR 40). A renamed releases repository answers with a redirect to `github.com`
  itself, so updates fail closed until a build names the new place.
- **Why a cold backup key:** the release key was made in memory and stored straight into the
  secret, so no copy of it exists. With only it trusted, a lost or overwritten secret means every
  player installs by hand once; a second trusted key, kept by the owner outside GitHub, turns that
  into one secret change. It is not trusted yet: 0.15.0 ships with the release key alone, so
  until a later release adds the backup, a lost secret means one release installed by hand.
  <!-- second-key-pending -->
- **Why the self-hosted Linux runner signs:** GitHub-hosted runners are not available on this
  account (the 0.15.0 tag's `sign` job never started: failed payments or a spending limit), and
  every other job already runs on the self-hosted runners. Signing runs on the Linux one, never
  the Mac. It is shared with pull-request CI, so the residual risk is plain: anything that can run
  code on that runner before a `v*` tag job could capture the key from the sign step's
  environment. Mitigations: `.npmrc` `ignore-scripts=true`, no dependency install in the job,
  Node under `env -i` with only the keys, the tag-only environment, a single-owner repository. A
  runner used for signing only is the stronger fix.
- **Cost:** the private keys are now as sensitive as a code-signing key: whoever holds one can
  ship an update that every Windows and Linux copy installs when its player presses Install. The
  environment keeps the key from branch runs and pull requests, but not from the shared Linux
  runner (above), and anyone who can push a `v*` tag runs the workflow at that commit, and tag
  protection is not available on the repository's plan. Without the secret a release fails, unless it is run with
  `allow_unsigned` (the app then offers only the release page). Losing every trusted key (both,
  once the cold backup is trusted; until then, the release key) means one release installed by
  hand. macOS still needs a drag into Applications until the app is signed.
  The first release carrying the updater is installed by hand (0.14.0 has none), and a real
  upgrade is proven only by the next two releases that carry it.
