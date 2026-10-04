# Roadmap

Romperoom is built in four milestones. **Milestone 1, Foundation, is complete,** and so are the
DAT part of Milestone 2: identifying games against DAT files you import or have Romperoom
download when you ask, the first part of Milestone 3: Tidy up, which sets aside duplicates and
leftover artwork, and the first part of Milestone 4: copying games to an SD card. Everything
marked planned is not built yet, and plans change as each milestone starts. Next come a first
release, then the rest of Organize. The detailed plans, with their tests, are in the design
history (foundation,
identify,
game database downloads).

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
- Folder-to-console mapping for every system in the [catalog](systems.md), with a way to sort the folders it cannot
  place.
- Cover art found in common layouts, and served to the page through a locked-down protocol.
- The desktop app: a first-run wizard, a cover-art wall, a game drawer and a Health screen. It
  has three themes, light and dark, and works with keyboard and gamepad.
- An engine for journaled, undoable file operations (quarantine and moves). Tidy up
  (Milestone 3) runs on it; the card writer keeps its own manifest (see Milestone 4).
- Four device profiles as data (Batocera, ES-DE, muOS and Onion).
- Packaging and a release pipeline: unsigned beta installers for macOS (Apple silicon) and
  Windows (x64), checked on every CI run, drafted by a release workflow
  ([release.md](release.md)). No release has been published yet.

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

**Deferred:** scrapers and artwork; see [Open questions](#open-questions). The
design spec keeps the research.

## Milestone 3: Organize (Tidy up done)

Tidy the library, always as a plan you review first, and always undoable.

**Done:** the Tidy up screen ([user guide](user-guide.md#tidy-up-your-library)) over the tidy
engine ([architecture.md](architecture.md#tidying-the-library)), behind one lock per library and
a journal that knows its library.

- **Duplicates:** byte-identical copies, with a keeper chosen by your region order and revision,
  shown with the reason, and any copy can be kept instead. The last copy of a game is never
  removed. Removed copies are set aside in the library, not put in the bin.
- **Leftover artwork:** pictures for games you don't have or that were removed, and extra
  copies of one picture (never one a frontend reads where it is). Art in a folder Romperoom
  doesn't recognise is left alone. When the last scan was incomplete, it says it could not check
  rather than reporting nothing.
- **History and Set aside:** every run listed with Undo all, set-aside files put back one at a
  time or together, and Delete forever behind a preview and typed words.
- **Recovery:** a run that stopped (a crash, or Cancel) is offered to finish or undo, at the next
  start and from its own result.

Proven end to end on the fixture library on macOS, and with the fixture's Windows shape
simulated. Not run against a real Windows drive or a NAS.

**Still planned:**

- **Folder standardisation:** rename and merge console folders (`GBA` and `Game Boy Advance`)
  to one scheme, and rename files to their DAT names. Their art is renamed with them.
- **Re-linking artwork** to a renamed game, game-list entries without a game, duplicates across
  libraries, and emptying only files older than a date.
- **Filtering leftover artwork by cause,** and thumbnails beside each duplicate copy.
- **A cancelled run that is never settled stays interrupted,** even after a fresh run tidied
  the same files ([decision 33](decisions.md#33-a-stopped-tidy-is-finished-or-undone-not-undone-in-part)).

## Milestone 4: Deploy (SD cards done)

Put a playable selection on a handheld's SD card.

**Done:**

- **The card wizard** ("Put games on a card"): pick a device, the consoles and the card. A size
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
- **Code signing, notarization, automatic updates, Intel Macs and Linux** (see
  [release.md](release.md#not-in-this-release)).

## Next

1. **A first release:** publish the unsigned beta the release workflow drafts, then signing
   and updates (see [release.md](release.md)).
2. **[Milestone 3, Organize](#milestone-3-organize-tidy-up-done):** folder standardisation.
3. **[Milestone 2, Identify](#milestone-2-identify-offline-dat-matching-done):** scrapers and
   artwork, once the owner decisions in [Open questions](#open-questions) are made.

## Must-fix before later milestones

Known gaps in Milestone 1 that are harmless today, because nothing triggers them, but must be
fixed before the feature that would:

| Item                                                                                                                                                                                                                                        | Fix before                             |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------- |
| **The media protocol checks folders, then opens the file.** A folder swapped for a link in between is not caught. It also reads each file whole (up to 50 MiB), and serves catalogued `.mp4`, `.webm` and `.pdf` files under a sandbox CSP. | Any screen that shows video or manuals |
| **Windows has never been run by hand.** CI passes on GitHub-hosted runners only.                                                                                                                                                            | The first Windows release              |

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

- Which DAT sources and scraping services to use, and their terms and rate limits.
- The owner decisions the online part needs (services, where art lives, what a lookup sends, the
  network stack).
- A source for BIOS hashes that can be verified.
- Signing identities, and who provisions them.
