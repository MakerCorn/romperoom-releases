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

Superseded by ADR 40 for downloading game databases and by ADR 41 for cover art; every other part
stands.

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
  details, relative to the card.

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
  reading counts interrupted work.
- **Why:** the engine's undo refuses an operation with a running journal, and should: half a
  journal has no settled "before". Recovery already re-checks every file the way a fresh run
  does, so one path covers both a crash and a cancel. A prompt that opened mid-session would
  cover the result the player had just asked for.
- **Cost:** a stopped run that is neither finished nor undone stays counted as interrupted, and
  is offered again at the next start, even after a fresh run tidied the same files.

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
  the not-recognised list yet.

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
  most 3 requests) only when pressed. Main process only, `node:https`, exact URL allowlist, no
  redirects, timeouts and byte caps, every file checked against its listed size and git SHA,
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
