# Roadmap

Romperoom is built in four milestones. **Milestone 1, Foundation, is complete,** and so are the DAT
part of Milestone 2: identifying games against DAT files you import or have Romperoom download when
you ask, and cover art from libretro-thumbnails and SD cards, the first part of Milestone 3: Tidy
up, which sets aside duplicates and leftover artwork, standardises folder and game names, re-links
artwork and lists copies across libraries, with libraries added and removed in Settings, and the
first part of Milestone 4: copying games to an SD card. Everything marked planned is not built yet,
and plans change as each milestone starts. Next come signing and automatic updates, then the rest of
Organize. The detailed plans, with their tests, are in the design history
(foundation,
identify,
game database downloads,
cover art,
card sync,
Standardise,
re-link,
the Tidy up batch,
libraries,
copies across libraries).

## Contents

- [Milestone 1: Foundation (done)](#milestone-1-foundation-done)
- [Milestone 2: Identify (offline DAT matching done)](#milestone-2-identify-offline-dat-matching-done)
- [Milestone 3: Organize (Tidy up done)](#milestone-3-organize-tidy-up-done)
- [Milestone 4: Deploy (SD cards done)](#milestone-4-deploy-sd-cards-done)
- [Next](#next)
- [Must-fix before later milestones](#must-fix-before-later-milestones)
- [Deferred minors](#deferred-minors)
- [Open questions](#open-questions)

## Milestone 1: Foundation (done)

- A catalog of your ROM library in SQLite, built by a read-only scan. Scans are hashed,
  resumable and cancellable, and guarded so that an unmounted or half-mounted share never makes
  games look deleted.
- Folder-to-console mapping for every system in the [catalog](systems.md), with a way to sort the
  folders it cannot place.
- Cover art found in common layouts, and served to the page through a locked-down protocol.
- The desktop app: a first-run wizard, a cover-art wall, a game drawer and a Health screen. It
  has three themes, light and dark, and works with keyboard and gamepad.
- An engine for journaled, undoable file operations (quarantine and moves). Tidy up
  (Milestone 3) runs on it; the card writer keeps its own manifest (see Milestone 4).
- Four device profiles as data (Batocera, ES-DE, muOS and Onion).
- Packaging and a release pipeline: unsigned beta installers for macOS (Apple silicon) and
  Windows (x64) and Linux (x64), checked on every CI run of main and of ready pull requests,
  drafted by a release workflow ([release.md](release.md)). Every release is published on the
  public releases page (0.1.0 to 0.4.0 as pre-releases).

## Milestone 2: Identify (offline DAT matching done)

Tell you which game each file really is.

- **DAT import:** game databases (Logiqx XML and clrmamepro DAT files, plain or zipped) that you
  import in Settings › Game databases. None is bundled.
- **Getting game databases:** Settings › Game databases opens the official No-Intro or Redump
  page in your browser and then the file picker in Downloads, or downloads the DATs for the
  consoles in your library from libretro-database on GitHub (CC BY-SA 4.0), pinned to one
  commit, after you have reviewed the list. Check for updates moves the pin; Network activity
  lists every request. Nothing goes online until you press a button
  ([ADR 40](decisions.md#40-game-databases-can-be-downloaded-from-one-pinned-source-only-when-asked)).
- **Matching within one system:** exact hashes first (SHA-1, MD5, then CRC32 with the size),
  then headerless hashes for the systems with copier headers, then every entry of a zip against
  one game, then names. A name-only match is labelled as such and kept for you to review.
  Files that stay unidentified say why.
- **Stable games:** a game keeps its identity across rescans and DAT changes; replacing or
  removing a DAT reverts only its own matches.
- **Cards named by DAT titles:** an identified game shows the DAT's title (its description
  when it has one, as FinalBurn Neo's set-id names need, else its name), on the wall and on a
  card. Revisions are separate games in the library; a card gets the newest of them.
- **Cover art from libretro-thumbnails and SD cards (done):** Health › Games without cover art
  lists the pictures libretro-thumbnails on GitHub has for the games missing box art,
  screenshots or title screens, matched by DAT name or exact file name, and downloads them into
  the library's `.romperoom/media` folder after a review, never replacing a picture; or imports
  the art a device already keeps on an SD card. Remove downloaded art takes them out again.
  Nothing goes online until you press Get cover art
  ([ADR 41](decisions.md#41-a-library-gains-one-writer-outside-tidy-up)).

**Deferred:** scrapers (ScreenScraper and other account-based services); see
[Open questions](#open-questions). The
design spec keeps the research.

## Milestone 3: Organize (Tidy up done)

Tidy the library, always as a plan you review first, and always undoable.

**Done:** the Tidy up screen ([user guide](user-guide.md#tidy-up-your-library)) over the tidy
engine ([architecture.md](architecture.md#tidying-the-library)), behind one lock per library and
a journal that knows its library.

- **Duplicates:** byte-identical copies, with a keeper chosen by your region order and revision,
  shown with the reason, and any copy can be kept instead. The last copy of a game is never
  removed. Removed copies are set aside in the library, not put in the bin.
- **Artwork:** leftover pictures for games you don't have or that were removed, and extra
  copies of one picture (never one a frontend reads where it is). Art in a folder Romperoom
  doesn't recognise is left alone. When the last scan was incomplete, it says it could not check
  rather than reporting nothing.
- **History and Set aside:** every run listed with Undo all, set-aside files put back one at a
  time or together, and Delete forever behind a preview and typed words.
- **Recovery:** a run that stopped (a crash, or Cancel) is offered to finish or undo, at the next
  start and from its own result; a Tidy up run can also discard the rest.
- **Tidy up, smaller things:** leftover artwork shown one cause at a time, a picture beside each
  set of duplicates, and Delete forever limited to what was set aside more than 30 or 90 days ago
  ([user guide](user-guide.md#tidy-up-your-library)).
- **Standardise:** console folders renamed and merged to one device profile's names, and
  identified games renamed to their DAT names with their art, saves, cue sheets, playlists and
  game list entries ([user guide](user-guide.md#standardise-your-library),
  [architecture.md](architecture.md#standardise-the-library)), all or nothing per game, undoable.
- **Libraries:** Settings › Libraries lists every library (its games, its last scan, whether its
  folder answers), adds a second one through the folder picker and removes one, forgetting it
  without touching its files ([user guide](user-guide.md#your-libraries),
  [decision 45](decisions.md#45-removing-a-library-forgets-it-whole-and-touches-nothing-on-disk)).
- **Copies across libraries:** the files held in two or more libraries, listed as a report;
  libraries that can't be compared say why ([user guide](user-guide.md#tidy-up-your-library),
  [decision 46](decisions.md#46-copies-across-libraries-are-reported-never-tidied)).
- **Re-link artwork:** a leftover picture renamed after the one game that clearly matches it,
  with its game list entries, and game list entries without a game listed or re-pointed
  ([user guide](user-guide.md#tidy-up-your-library),
  [architecture.md](architecture.md#re-link-artwork)), undoable.

Proven end to end on the fixture library on macOS, and with the fixture's Windows shape
simulated; Standardise and Re-link artwork were also run live on scratch libraries on
macOS ([testing.md](testing.md#live-standardise-run),
[the re-link run](testing.md#live-re-link-run)), and so were the leftover artwork filter, the
duplicate pictures, Delete forever by age and Discard the rest
([the tidy up batch run](testing.md#live-tidy-up-batch-run)), and so was Settings › Libraries
([the libraries run](testing.md#live-libraries-run)), and so were copies across libraries
([the across libraries run](testing.md#live-across-libraries-run)). Not run against a real Windows
drive or a NAS.

**Still planned:**

- **Setting aside copies across libraries.** Across libraries only lists them today.
- **A renamed playlist's name match after Standardise is undone.** A playlist (`.m3u`) a run renamed
  without rewriting it, matched by identify under its new name before the run was undone or rolled
  back, keeps that match, and the review leaves its game as not matching until Identify games runs
  again (a scan does not clear it). The likely fix is to keep every renamed playlist's catalog row
  as rewritten ones are kept
  ([decision 43](decisions.md#43-standardise-renames-folders-and-games-and-edits-two-kinds-of-files-other-programs-own)).
- **A re-link of game list entries only that crashed before its journal was recorded** stays
  "Interrupted" in History and can't be undone. Nothing was written, so there is nothing to undo.
- **Pictures of a removed game read "Game removed" after a scan.** A scan unlinks a picture whose
  game's files are gone, so the Artwork tab lists it under "No game in your library" and the Game
  removed choice rarely shows
  ([decision 36](decisions.md#36-art-a-frontend-reads-is-never-a-spare-copy)).

## Milestone 4: Deploy (SD cards done)

Put a playable selection on a handheld's SD card.

**Done:**

- **The card wizard** (**SD card** in the menu): pick a device, the consoles and the card. A size
  meter counts the card's file system overhead, and when the selection is too big it offers what
  to leave out. A check step and a confirmation come before any write
  ([user guide](user-guide.md#put-games-on-an-sd-card)).
- **The deploy planner** lays out a package for a profile, sizes it in bytes on the card, and
  writes ES-DE and Batocera game lists ([architecture.md](architecture.md#deploy-planner)).
- **The card writer:** volume listing on macOS, Windows and Linux, the safe-target rules, and a
  verified, resumable, incremental writer with a manifest, moving dropped files aside, never
  formatting ([architecture.md](architecture.md#card-writer)). The page never names a path
  ([decisions.md](decisions.md#23-the-page-never-names-a-path-to-write-to)).
- **BIOS folders:** a BIOS folder chosen per library is copied to a device that has one.
- **Sync a card:** bring a handheld's new games and in-game saves into the library and newer
  saves back to the card, in one reviewed, undoable step that deletes nothing on either side
  ([user guide](user-guide.md#sync-a-card),
  [ADR 42](decisions.md#42-card-sync-writes-into-a-library-and-onto-a-card)).
  Save layouts are measured from each system's source for muOS, Onion and Batocera; ES-DE syncs
  games only.

Proven on a FAT32 disk image on macOS and, end to end, on a test folder posing as a card.
Windows and Linux listing is tested from recorded output only, the wizard has not been run
against a real card on Windows, and exFAT is not tested on a real file system.

**Still planned:**

- **Saved packages:** named selections per device. Only the last choices are remembered now.
- **A fuller "make it fit"** that picks games for you. Today it offers artwork, other versions
  and the biggest consoles.
- **Device profiles** checked on real devices (all four are `community` today), and more of them.
- **Art resized for the device,** and BIOS files found by their hash, not their name.
- **Read-only cards on Linux.** `lsblk` does not report mount options, so a card mounted
  read-only is not refused up front, and the run fails at its writes instead.
- **Export folders on a network share** are written without asking first.
- **Cover art carries no SHA-1 from the catalog,** so the writer hashes the art file itself
  whenever it must compare it with the card.
- **A game whose files differ between two libraries** (same folder and name, different bytes)
  is left off the card as a name clash. It should pick one copy instead.
- **Code signing, notarization, automatic updates and Intel Macs** (see
  [release.md](release.md#not-in-this-release)).

## Next

1. **Signing and automatic updates** for the beta builds (see [release.md](release.md)).
2. **[Milestone 3, Organize](#milestone-3-organize-tidy-up-done):** setting aside copies
   across libraries.
3. **[Milestone 4, Deploy](#milestone-4-deploy-sd-cards-done):** saved packages, profiles
   checked on real devices and a fuller "make it fit".
4. **[Milestone 2, Identify](#milestone-2-identify-offline-dat-matching-done):** scrapers
   (ScreenScraper, its own spec), once the owner decisions in [Open questions](#open-questions)
   are made.

## Must-fix before later milestones

Known gaps in Milestone 1 that are harmless today, because nothing triggers them, but must be
fixed before the feature that would (the Windows and Linux row excepted: those builds already
ship, as a beta):

| Item                                                                                                                                                                                                                                                        | Fix before                             |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------- |
| **The media protocol checks folders, then opens the file.** A folder swapped for a link in between is not caught. It also reads each file whole (up to 50 MiB), and serves catalogued `.mp4`, `.webm` and `.pdf` files under a sandbox CSP.                 | Any screen that shows video or manuals |
| **Windows and Linux have never been run by hand.** Their builds have been published as an untried beta since 0.1.0 (Windows) and 0.2.0 (Linux); their tests pass only on CI machines (a self-hosted Windows PC and a Linux container, or GitHub's runners). | Calling Windows or Linux supported     |

## Deferred minors

Small, known issues, accepted for Milestone 1:

- The Assign buttons on the unmapped-folders list do not name their folder in their accessible
  name.
- After a rescan, the Undo message does not say that games already added stay added.
- The session's list of recent folder assignments (for Undo) is not pruned when a library is
  removed.
- A few hash pool tests are timing-sensitive under heavy load.
- Pressing Cancel just as the first of several libraries finishes announces "Scan finished",
  not "Scan cancelled".
- End jumps to the last game loaded so far, not the last game.
- The interface is English only.
- The title parser has edge cases: dotted names without an extension, and mismatched brackets.
- A rename between NFC and NFD spellings looks like a removal plus a new file (except for a
  top-level system folder).
- `journal.ts` has grown long and should be split before more is added to it.
- Case-insensitive search only folds ASCII letters.
- A hash worker that dies right after starting is respawned without limit.

The full list of current behaviour limits is in
[development.md](development.md#known-limitations).

## Open questions

- Which scraping services to use, and their terms and rate limits.
- The owner decisions a scraper needs (accounts, what a lookup sends). Where art lives and the
  network stack were decided for cover art (ADR 41).
- A source for BIOS hashes that can be verified.
- Signing identities, and who provisions them.
