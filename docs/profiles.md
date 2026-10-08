# Device profiles and the systems catalog

`packages/profiles` holds two kinds of data:

- **The systems catalog** (`data/systems.json`): every console, computer and engine Romperoom
  knows, with the folder names and file extensions that identify it. The scanner uses it to
  work out which folder holds which console's games. [systems.md](systems.md) is generated from
  it.
- **Device profiles** (`data/<id>.json`): how one device or frontend lays out its SD card. That
  means where ROMs, BIOS files and media go, and which folder and extensions each system uses.

It also ships `data/libretro.json`, the recorded libretro-database listing and system mapping
used by the "get game databases" feature, and `data/thumbnails.json`, the recorded
libretro-thumbnails repository list used by the cover art feature. Neither is a systems catalog
entry or a device profile: `loadShippedProfiles()` skips both by name, the same way it skips
`systems.json` (see [libretro.json](#libretrojson) and [thumbnails.json](#thumbnailsjson) below).

**Status:** four profiles ship: `batocera`, `es-de`, `muos` and `onion`. All are `community`.
The [deploy planner](architecture.md#deploy-planner) reads them to lay
out an SD card, and the card wizard ([user guide](user-guide.md#put-games-on-an-sd-card)) copies
to that layout. Sync a card, cover art from a card and Standardise (its folder names) read them
too. The wizard marks a `community` profile with a **Community** badge and says it
was set up from what other players shared and has not been tested by us yet.

## Contents

- [Profile fields](#profile-fields)
- [Path safety](#path-safety)
- [Status and sources](#status-and-sources)
- [Shipped profiles and their sources](#shipped-profiles-and-their-sources)
- [Saves](#saves)
- [A worked example](#a-worked-example)
- [Adding a device profile](#adding-a-device-profile)
- [The systems catalog](#the-systems-catalog)
- [Adding a system](#adding-a-system)
- [libretro.json](#libretrojson)
- [thumbnails.json](#thumbnailsjson)

## Profile fields

`profileSchema` (`packages/profiles/src/schema.ts`) validates every profile.
`loadShippedProfiles()` loads every `.json` in `data/` except `systems.json`, `libretro.json` and
`thumbnails.json`, sorted by name, and names the file in any error. A docs test checks that this
table lists every key of the schema.

| Key                            | Type                                                                        | Meaning                                                                                                               |
| ------------------------------ | --------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `schemaVersion`                | `1`                                                                         | The profile format version.                                                                                           |
| `id`                           | `[a-z0-9-]+`                                                                | The profile's id. Use the file name (nothing enforces it).                                                            |
| `name`                         | text                                                                        | The display name, such as `ES-DE`.                                                                                    |
| `family`                       | text                                                                        | A grouping, such as `frontend` or `linux-handheld`.                                                                   |
| `status`                       | `verified` or `community`                                                   | How far the layout is confirmed (see [Status and sources](#status-and-sources)).                                      |
| `sources`                      | URLs, at least one                                                          | The documentation the layout was taken from.                                                                          |
| `root.roms`                    | relative path                                                               | The folder that holds one folder per system.                                                                          |
| `root.bios`                    | relative path, optional                                                     | The BIOS folder. Leave it out when the firmware has none.                                                             |
| `root.media`                   | relative path, optional                                                     | The media folder, including any parent (`ES-DE/downloaded_media`). Without it, media folders are inside `root.roms`. |
| `systems.<id>.folder`          | one path segment                                                            | The device's folder name for the system. `<id>` is an id from the systems catalog.                                    |
| `systems.<id>.extensions`      | `.ext` in lower case, at least one, optional                                | The extensions the device accepts. Leave it out when the device takes any file in the folder (muOS).                  |
| `systems.<id>.acceptsArchives` | true or false, optional                                                     | Whether the device runs the system's games from `.zip` or `.7z`. Left out means not documented, treated as no.        |
| `systems.<id>.mediaFolder`     | one path segment, optional                                                  | The system's name in media folders, for `{mediaFolder}` (muOS's catalogue name).                                      |
| `media.<kind>.folder`          | relative path template                                                      | Where that kind of media goes (see [Templates](#templates)).                                                         |
| `media.<kind>.format`          | `png`, `jpg`, `mp4` or `pdf`, or a list of them                             | The file formats the device reads. `jpg` also covers `.jpeg`.                                                         |
| `media.<kind>.suffix`          | text, optional                                                              | Added to the ROM's stem to name the media file (Batocera's `-image`).                                                 |
| `media.<kind>.maxWidth`        | whole number, optional                                                      | The widest image the device shows well.                                                                               |
| `saves.root`                   | relative path                                                               | Where the device keeps in-game saves. Leave `saves` out when no source says (see [Saves](#saves)).                    |
| `saves.folder`                 | relative path template                                                      | The folder below `saves.root`: `{system}`, `{folder}`, or `{core}` as the whole last segment.                         |
| `saves.extensions.<id>`        | `.ext` in lower case, at least one                                          | The save files of that system. A system left out has no saves synced.                                                 |
| `saves.cores.<id>`             | one path segment, optional                                                  | The core folder for a console with no saves on the card yet (`{core}` layouts only).                                  |
| `gamelist.format`              | `none`, `es-de-xml`, `emulationstation-xml`, `batocera-xml` or `onion-json` | The game list file the device reads.                                                                                  |
| `gamelist.path`                | relative path template                                                      | Where one system's game list goes. Required with a format, refused with `none`.                                       |
| `limits.fileSystem`            | `fat32`, `exfat`, `ntfs`, `ext4` or `any`                                   | The card file system the device needs.                                                                                |
| `limits.maxFileBytes`          | whole number, optional                                                      | The largest single file the device can use.                                                                           |
| `naming.stripRegionTags`       | true or false                                                               | Whether file names should drop tags such as `(USA)`.                                                                  |
| `naming.maxPathLength`         | whole number                                                                | The longest path the device handles.                                                                                  |

`<kind>` is one of `box`, `screenshot`, `marquee`, `video`, `manual`, `wheel` or `title` (title
screens). Each is optional. Leave a system out rather than guess a folder name the documentation
does not state.

### Templates

A media folder and a game list path may use three placeholders, expanded per system:

| Placeholder     | Expands to                                       | Example (muOS, NES)      |
| --------------- | ------------------------------------------------ | ------------------------ |
| `{system}`      | the system's id                                  | `nes`                    |
| `{folder}`      | `systems.<id>.folder`                            | `nes`                    |
| `{mediaFolder}` | `systems.<id>.mediaFolder` (media folders only) | `Nintendo NES - Famicom` |

Any other `{...}` is refused. A media file is named `<ROM stem><suffix>.<ext>`, where the stem
is the ROM's file name on the card without its extension.

## Path safety

A profile describes paths on someone's card, so every path is checked before any later code
can join it to a real folder:

- Every path is relative to the card root. A leading `/`, `\` or `~` is refused, and so are
  `..`, drive letters (`C:`), empty or `.` segments and control characters.
- A system `folder` is a single segment: no `/` or `\`.
- Every segment must also be portable to FAT32, exFAT and NTFS cards. That rules out:
  - Windows device names (`CON`, `PRN`, `AUX`, `NUL`, `COM1`-`COM9`, `LPT1`-`LPT9`), with or
    without an extension, in any case;
  - a trailing dot or space (Windows drops it, so `nes.` and `nes` would be one folder);
  - any of `<>:"|?*`.
- Two systems may not share a folder. Folders are compared the way a case-insensitive file
  system compares them (`foldName`: case and Unicode normalization), so `NES` and `nes` collide.
- `root.media` must not be inside `root.roms`, or the other way round. Otherwise media would be
  catalogued as games.
- `root.bios` must not overlap `root.roms` or `root.media` either.
- System ids are `[a-z0-9-]+`, and `folder` and `mediaFolder` obey the segment rules, so a
  template expands to a safe path. A `suffix` holds no separator, control character or
  `<>:"|?*`, and does not end in a dot or space.

The schema is the first check, not the last. The deploy planner cleans every file name it
writes and checks every planned path again, and the card writer must check each final, resolved
path before it writes (see [security.md](security.md#deploy-containment)).

## Status and sources

- `verified` means the layout was checked on real hardware, or against official documentation
  that states it.
- `community` means anything less: documentation that implies a layout, forum posts, or a
  layout seen once.

All four shipped profiles are `community`: they are taken from each project's own source code
and documentation, but none has been checked on a real device.

**A note for the community.** If you have one of these devices, the most useful thing you can do
is copy a few games with Romperoom and say whether they show up and start, with their box art
and names, and which firmware version you run. A profile becomes `verified` once a real device
confirms its layout, with that report listed in `sources`. List every page you used in
`sources`. Work from the device's or firmware's own documentation, never from memory alone.

## Shipped profiles and their sources

Each profile covers the same twelve consoles (Onion has no N64 emulator, so eleven). Where a source
lists an extension the catalog does not, the profile leaves it out, because the planner only
ever sees files the catalog classified.

| Profile    | Layout                                                                                                                                                                                                                 | Taken from                                                                                                                                                                                                                                         |
| ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `muos`     | `ROMS/<folder>`, `MUOS/bios`, art in `MUOS/info/catalogue/<catalogue name>/box` and `/preview`, named like the ROM. No game list. Folders are muOS's own short names (`nes`, `genesis`), the catalogue names its own. | MustardOS `internal`: `share/info/manifest/libretro.json` (folder keys and catalogue names), `script/device/storage.sh` (`ROMS`), `script/device/bind.sh` (`MUOS/bios`, `MUOS/info/catalogue`), `script/system/catalogue.sh`; muos.dev artwork. |
| `batocera` | `roms/<folder>`, `bios`, art in `roms/<folder>/images/<stem>-image`, `-thumb` and `-marquee`, `videos/<stem>-video.mp4`, `manuals/<stem>-manual.pdf`, and `roms/<folder>/gamelist.xml`. Genesis is `megadrive`.       | `es_systems.yml` (folder = system key, `FORMATS.md`), the bundled `roms/` tree, batocera-emulationstation `Scraper.cpp` (`getSaveAsPath`), `MetaData.cpp` (tags), `Settings.cpp` (defaults), and wiki.batocera.org system pages (formats).       |
| `es-de`    | `ROMs/<folder>`, art in `ES-DE/downloaded_media/<folder>/covers`, `screenshots`, `marquees`, `videos`, `manuals` and `titlescreens`, and `ES-DE/gamelists/<folder>/gamelist.xml`. No BIOS folder of its own.          | The ES-DE user guide (media folders, game list location, the `./` path rule) and `es_systems.xml` (folders and extensions).                                                                                                                       |
| `onion`    | `Roms/<FOLDER>` (upper case, such as `FC` and `SFC`), `BIOS`, box art in `Roms/<FOLDER>/Imgs/<stem>.png` at most 250 wide. FAT32 only. No game list written.                                                          | Onion's docs (rom folders, BIOS in the root `/BIOS`, the scraping guide, FAT32 install) and each emulator's `config.json` (`rompath`, `imgpath`, `extlist`).                                                                                      |

Known judgement calls, all reasons the profiles stay `community`:

- muOS does not filter by extension (it skips names listed in `share/info/skip.ini`), so its
  systems leave `extensions` out. Its archive support is per core and not documented, so
  `acceptsArchives` is left out (treated as no).
- muOS's artwork page shows `Nintendo SNES-SFC`; the manifest the firmware ships says
  `Nintendo SNES - SFC`. The profile follows the manifest.
- Batocera's `image` tag defaults to a screenshot and `thumbnail` to a 2D box (`Settings.cpp`),
  so screenshots map to `-image` and box art to `-thumb`. A user can change those scraper
  settings on the device.
- Onion reads a `miyoogamelist.xml` in each ROM folder. Its FAQ says it has the format of
  `gamelist.xml`, and its own generator (`miyoogamelist_gen.sh`) writes `./<rom>` paths and
  `./Imgs/<stem>.png` images, but the profile names no game list, so none is written and
  Standardise does not change one.
- **Standardise** and **re-link** read the game list of the formats the shipped profiles name
  (`gamelist.xml`: Batocera's, and ES-DE's when it reads lists from the ROM folders) in each
  console folder of a library: its `<game>` and `<folder>` entries' `<path>`, relative to the
  folder (ES-DE `Gamelist.cpp`/`GamelistFileParser.cpp`, batocera-emulationstation
  `Gamelist.cpp`). muOS names no game list.

## Saves

The optional `saves` block says where a device keeps in-game saves, so **Sync a card** can keep
them in step with the library ([user guide](user-guide.md#sync-a-card)). A profile without it
can still have its card's games imported; the review then says "Romperoom doesn't know where
this device keeps saves." A save belongs to the game whose ROM stem it carries
(`Tetris (World).srm` for `Tetris (World).gb`), compared in Unicode NFC, lower-cased. Save
states (`.state`, `.state1`, `.state.auto`, `.ss0`) are never synced. `{core}` stands for one
folder per emulator core, as RetroArch's "Sort saves into folders by core name" makes them; a
save there belongs to the console whose ROM on the card carries its stem, and one whose stem
names games of two consoles is left alone.

| Profile    | Saves                                          | Consoles                        | Taken from                                                                                                                                                                                                                          |
| ---------- | ----------------------------------------------- | -------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `muos`     | `MUOS/save/file/<core>/<stem>.srm`             | all but DS (DraStic by default) | MustardOS `internal`: `retroarch.default.cfg` (`savefile_directory`, `sort_savefiles_enable`), `bind.sh` (`MUOS/save` on the card) and the manifest's default cores; core names from `libretro-core-info` (`corename`).             |
| `onion`    | `Saves/CurrentProfile/saves/<core>/<stem>.srm` | all but DS (DraStic)            | Onion's `retroarch.cfg` (`savefile_directory`, `sort_savefiles_enable`), each emulator's `launch.sh` (its core; gpSP's `saves/gpSP`) and Onion's FAQ (`.srm`, named like the ROM); core names from `libretro-core-info`.            |
| `batocera` | `saves/<folder>/<stem>.srm`                    | all but DS                      | `batocera-launch-libretro`'s `emulator.py` (saves go to the system's saves folder, sorting off), `batocera_launch`'s `emulator.py` (`saves_dir` is `SAVES / system`) and `batocera_common/paths.py` (`SAVES` is `/userdata/saves`). |
| `es-de`    | none                                           |                                 | ES-DE leaves saves to each emulator, outside its own folders; its user guide names no save location, so nothing on a card says where they are.                                                                                      |

Judgement calls, recorded so a check on a real device can overturn them:

- Only `.srm` is synced: RetroArch's save name for every core these profiles use by default
  (PCSX ReARMed keeps memory card 1 as `.srm` too). Saves of standalone emulators (DraStic on
  muOS and Onion, DuckStation if chosen on Batocera) are never read.
- `cores` holds each default core's `corename` from `libretro-core-info`, the folder name
  RetroArch sorts saves into. A card that already holds saves of a console uses that folder
  instead; when it holds them in more than one, the library's saves are not offered to it.
- No layout has been checked against a real card yet, so the profiles stay `community`.

## A worked example

A cut-down muOS profile (the shipped `packages/profiles/data/muos.json` has twelve consoles and
more sources):

```json
{
  "schemaVersion": 1,
  "id": "muos",
  "name": "muOS",
  "family": "linux-handheld",
  "status": "community",
  "sources": ["https://muos.dev/installation/artwork"],
  "root": { "roms": "ROMS", "bios": "MUOS/bios", "media": "MUOS/info/catalogue" },
  "systems": {
    "nes": { "folder": "nes", "mediaFolder": "Nintendo NES - Famicom" }
  },
  "media": {
    "box": { "folder": "{mediaFolder}/box", "format": "png" },
    "screenshot": { "folder": "{mediaFolder}/preview", "format": "png" }
  },
  "gamelist": { "format": "none" },
  "limits": { "fileSystem": "any" },
  "naming": { "stripRegionTags": false, "maxPathLength": 255 }
}
```

An NES game `ROMS/nes/Tetris (World).nes` gets its box art at
`MUOS/info/catalogue/Nintendo NES - Famicom/box/Tetris (World).png` on the card.

## Adding a device profile

1. Read the device's or firmware's own documentation for its folder layout.
2. Create `packages/profiles/data/<id>.json` with `schemaVersion: 1`, an `id`, a `name` and a
   `family`.
3. List your sources and set `status` (see [Status and sources](#status-and-sources)).
4. Fill in `root`, then `systems`, keyed by ids from `data/systems.json`. Then fill in `media`,
   `gamelist`, `limits` and `naming`.
5. Update `packages/profiles/test/shipped.test.ts`: add the id to the expected set and pin the
   profile's key contents. Other tests there check that every system key is a known id, that each
   profile names a source, and that every extension a profile lists for a system is in the
   catalog's list for it.
6. Run `npm test -w @romperoom/profiles`.

## The systems catalog

`packages/profiles/data/systems.json` is `{ "schemaVersion": 1, "systems": [...] }`. Each system
has these fields:

| Field             | Meaning                                                                    |
| ----------------- | -------------------------------------------------------------------------- |
| `id`              | The canonical id (`[a-z0-9-]+`), such as `snes`. Unique.                   |
| `name`            | The display name.                                                          |
| `kind`            | `console`, `handheld`, `computer`, `arcade`, `engine`, `port` or `other`.  |
| `aliases`         | Other folder names that mean this system, such as `Super Nintendo`.        |
| `extensions`      | The ROM extensions, lower case with the dot, no duplicates.                |
| `extensionSource` | Where the extension list came from: `es-de`, `batocera` or `conservative`. |
| `sources`         | Optional URLs the entry was taken from.                                    |

A top-level library folder maps to a system when its name matches the system's id, name or an
alias, after folding case, Unicode form and punctuation (`foldAlias`). The README's counts and
[systems.md](systems.md) are generated from this file, and tests fail when either is stale.

## Adding a system

1. Take the extensions and folder names from a real frontend's files: ES-DE first, then
   Batocera. Never work from memory. A system no source lists extensions for gets a short,
   hand-written list with `extensionSource: "conservative"`.
2. Add the entry to `data/systems.json`, with its sources.
3. If a folder name's system is a judgement call, add a row to `MAPPING_NOTES` in
   `packages/profiles/src/systems-doc.ts`. The tests resolve each row.
4. Run `npm run systems-doc` to regenerate [systems.md](systems.md), and update the counts in
   the README the same way (the failing test prints the expected block).
5. Run `npm test -w @romperoom/profiles`.

## libretro.json

`packages/profiles/data/libretro.json` (`packages/profiles/src/libretro.ts`, re-exporting
`packages/profiles/src/libretro-listing.ts`) holds what the "get game databases" feature needs to
offer a download before making any request:

- **The pin**: the commit SHA of libretro-database's `master` branch the listing was recorded
  at, its commit date, and the date it was recorded.
- **The listing**: every file's name, byte size and git blob SHA-1 in that commit's
  `metadat/no-intro` and `metadat/redump` folders.
- **`systems`**: the mapping from a Romperoom system id to `{ dir, name }`, the file that console
  downloads.

It is read once, synchronously, into `LIBRETRO` when the module loads, validated by
`libretroFileSchema`. It is **not a device profile**: `loadShippedProfiles()` skips it by name
(`LIBRETRO_FILE`), the same way it skips `systems.json`, so it never has to pass `profileSchema`
and never appears in a profile list.

Rules, each pinned by a test in `packages/profiles/test/libretro.test.ts`:

- Every mapped system id exists in `SYSTEMS`; every mapped name exists in the recorded listing,
  in the mapped folder; no file is mapped to two systems; every mapped size is at most
  `MAX_DAT_DOWNLOAD_BYTES`.
- Mapped names match a strict safe-name pattern (letters, digits, space and `.,+()'&!-`, ending
  `.dat`, no `/`, `\` or `..`), because a URL is built from one.
- The listing's commit is 40 lower-case hex characters and its dates parse.

**Who records it:** a maintainer runs `npm run record:libretro` before each release (never CI,
never a test). It asks `api.github.com` for the branch head and both folder listings with the
maintainer's own network, through `listingFromApi` (the one place that shape is derived; the
desktop app's own "Check for updates" calls the same function), replaces `listing` and keeps
`systems`, then reports the old and new commit, the counts per folder, and any mapping problems
against the mapping it just kept.

## thumbnails.json

`packages/profiles/data/thumbnails.json` (`packages/profiles/src/thumbnails.ts`) holds the
measured list of `libretro-thumbnails` repositories the cover art feature downloads from: 131
repository names, each with its default branch (`master` or `main`). It is read once,
synchronously, into `THUMBNAILS` when the module loads, validated by `thumbnailsFileSchema`. It is
**not a device profile**: `loadShippedProfiles()` skips it by name (`THUMBNAILS_FILE`), the same
way it skips `systems.json` and `libretro.json`, so it never has to pass `profileSchema` and never
appears in a profile list.

A console's thumbnail repository is its mapped libretro DAT name (`data/libretro.json`) with
spaces replaced by underscores (`repoNameFor`); a console with no DAT mapping, or whose repository
name is absent from this list, is "not available" (`thumbnailRepoFor`). A repository is only ever
listed or fetched at the branch this file records for it, never at a branch read from a GitHub
answer or a listing: `allowedRepos()` is the allowlist the transport checks requests against, and
it comes from this shipped file alone.

Rules, each pinned by a test in `packages/profiles/test/thumbnails.test.ts`:

- Every repository name matches a strict safe-name pattern (letters, digits, `_`, `.` and `-`, no
  `..`), because it becomes a URL path segment; no name is listed twice; the branch is `master` or
  `main`, nothing else.
- `recordedAt` is a parseable date-time.

**Who records it:** a maintainer runs `npm run record:thumbnails` before each release (never CI,
never a test). It asks `api.github.com` for the organisation's repository pages with the
maintainer's own network, through `thumbnailsFromApi` (the one place that shape is derived),
and replaces the whole file, sorted by name, requesting only at each repository's own recorded
branch thereafter.
