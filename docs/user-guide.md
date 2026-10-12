# Using Romperoom

Romperoom shows your ROM collection on a Home page of consoles, each a list or a wall of covers,
copies the games you pick to an SD card, syncs a handheld's card back, and tidies up duplicates and
leftover artwork. Only Fix up changes your games, and only after you have seen what it will do;
cover art only adds pictures to Romperoom's own folder, when you ask; Sync a card only adds the
games and saves you approve, and backs up any save it replaces. This guide walks through every
screen.

The pictures come from a small made-up test library, so the sizes are tiny. Its cover art is
generated for the screenshots (made-up pictures, no real box art or logo), and games without art
show a placeholder.

## Contents

- [Before you start](#before-you-start)
- [Set up your library](#set-up-your-library)
- [Folders that need a console](#folders-that-need-a-console)
- [Home](#home)
- [A console's games](#a-consoles-games)
- [Game details](#game-details)
- [Your libraries](#your-libraries)
- [Fix up](#fix-up)
- [Cover art](#cover-art)
- [Why a game shows no picture](#why-a-game-shows-no-picture)
- [Getting game databases](#getting-game-databases)
- [Identify your games](#identify-your-games)
- [What the labels mean](#what-the-labels-mean)
- [Check name matches](#check-name-matches)
- [When folders look empty or gone](#when-folders-look-empty-or-gone)
- [Files that couldn't be read](#files-that-couldnt-be-read)
- [Settings and themes](#settings-and-themes)
- [Put games on an SD card](#put-games-on-an-sd-card)
- [Sync a card](#sync-a-card)
- [Duplicates, leftover artwork and History](#duplicates-leftover-artwork-and-history)
- [Merge picture folders](#merge-picture-folders)
- [Junk files](#junk-files)
- [Standardise your library](#standardise-your-library)
- [Keyboard](#keyboard)
- [Gamepad](#gamepad)
- [Questions](#questions)

## Before you start

- Your games can be on this computer, an external drive or a network share (NAS). Connect the
  drive first.
- Romperoom works best when each console has its own folder, such as `snes` or
  `Game Boy Advance`. Folder names are matched to [known systems](systems.md) without minding
  case, accents, spaces or punctuation.
- Zipped games (`.zip`, `.7z`) are fine.

## Set up your library

The first time you open Romperoom, it asks for your ROM folder. Press **Choose your ROM
folder** and pick the folder that holds your console folders.

Romperoom then counts your games in two steps. First it finds them: every game is in your
library within minutes, even on a NAS. Then it reads every file once to fingerprint it (a
hash), which is how it spots exact copies; on a NAS that can take hours for a large collection.
While it checks, press **Go to my library** to start browsing: checking carries on in the
background, and **Fix up** says how many files are still waiting. You can press **Cancel scan** at
any time. A later scan skips the files it has already read, so it picks up close to where it
stopped.

| Light                                                           | Dark                                                                 |
| --------------------------------------------------------------- | -------------------------------------------------------------------- |
| ![The setup screen](screenshots/wizard-console-shelf-light.png) | ![The setup screen, dark](screenshots/wizard-console-shelf-dark.png) |

When the scan finishes, you see how many games it found. Press **Go to my library**. If the
folder had no games, choose a different folder.

## Folders that need a console

If a folder's name doesn't match a console, Romperoom asks you which console it is. Pick one
from the list and press **Assign**. Nothing is assigned until you press it, so you can't change
a folder by mistake while moving through the list. If a folder isn't games at all, choose
**Not a console — ignore this folder** and press **Ignore folder**. Ignored folders are listed in
**Fix up** › **Names**, where **Include again** brings one back.

When Romperoom can guess a folder's console from its name, it says so ("Looks like Game Boy
Advance"): **Use suggestion** fills the picker in, and you press **Assign** as usual. To take
several suggestions at once, tick **Use Game Boy Advance** (and so on) on each folder you want, or
**Select all** above the list (it reaches every folder, also those past **Show all**), then press
**Accept selected (N)**. N is the number of ticked folders it will assign: a folder that leaves
the list, or whose picker you set to something else, drops out of it. A folder with no guess has
no tick box. Nothing is assigned until you press the button; the folders are then assigned one
after another, with "Assigning 2 of 5" while it works, and Romperoom says how many it assigned.
While it works, the tick boxes, **Select all** and the pickers wait ("Accepting…"), so what is
assigned is exactly what was ticked when you pressed it. If one can't be assigned, it stays in
the list, still ticked, and the message names it; the others are still assigned. Space ticks a
box. Shift-click (or Shift+Space) on a box does the same to every folder between it and the last
box you changed: ticks them if it ticks, clears them if it clears.

There is no Undo for the whole lot at once: each folder it assigned is under **Recently assigned**
with its own **Undo**.

Each assignment can be undone under **Recently assigned**. Press the scan button below the list
afterwards (**Scan** and your library's name, or **Scan all libraries**) to add the games in the
folders you gave a console.

| Light                                                                          | Dark                                                                                |
| ------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------- |
| ![A folder that needs a console](screenshots/unmapped-console-shelf-light.png) | ![A folder that needs a console, dark](screenshots/unmapped-console-shelf-dark.png) |

## Home

**Home** is where Romperoom opens. From the top:

- **Search games** looks through every game as you type; select a match to open its details.
  When more match than fit, **See every match in All games** carries the search on.
- **Needs attention** lists, in a few short lines, what is waiting: duplicates, games not
  identified, folders without a console, files that couldn't be read, games gone from their
  folders, a library not scanned yet or a scan that didn't finish, a library that isn't connected,
  work that stopped part way. Each line opens where it is done: most in **Fix up**; a library that
  isn't connected, in Home's libraries just below. With nothing to do, the list isn't shown.
- **Recently added**: the newest games, one row.
- **Your libraries**: your libraries, each with its own **Scan** (named by the library) and
  **Remove**, and **Add a library…** (see [Your libraries](#your-libraries)).
- **Your consoles**: **All games** and a card per console, with its games and size. **Filter
  consoles** narrows the cards and **Sort consoles by** orders them by name, games or size. With
  many consoles Home shows the first 24; **Show all N consoles** shows the rest, and so does
  typing the name of a console past the first 24 while a card has focus.

| Light                                             | Dark                                                   |
| ------------------------------------------------- | ------------------------------------------------------ |
| ![Home](screenshots/home-console-shelf-light.png) | ![Home, dark](screenshots/home-console-shelf-dark.png) |

## A console's games

Choose a console (or **All games**) to open its games. **Back to Home** returns to its card. A
console no longer in your libraries (or an old bookmark to one) says "This console isn't in your
libraries (any more)." with the way back.

- **Search games** filters by title as you type, and **Sort by** orders the games by **Title** or
  by **Size**.
- **View**: **List** shows a dense list (the title, then on All games the console, the region and
  the size), two to three times as many games a screen as the covers; **Grid** shows the covers.
  Games without cover art get a placeholder with their initials. Romperoom starts in List when
  most of a console's games have no cover art, else in Grid, and keeps that view while the page is
  open (a scan or new cover art doesn't switch it under you); it remembers the one you choose.
- **Jump to letter**: `#` and A to Z go to the first game starting so (titles only: it is hidden
  when sorting by size). A letter no game starts with is dimmed text, not a button, and says so to a
  screen reader. Typing a letter or a digit in the list or grid does the same. Titles that start
  after Z (an accented letter such as É, or another alphabet) have no letter: reach them with End
  or by scrolling.
- The same game in two regions, such as USA and Europe, is two games.

| List                                                             | Grid                                                             |
| ---------------------------------------------------------------- | ---------------------------------------------------------------- |
| ![The list](screenshots/games-list-console-shelf-light.png)      | ![The grid](screenshots/games-grid-console-shelf-light.png)      |
| ![The list, dark](screenshots/games-list-console-shelf-dark.png) | ![The grid, dark](screenshots/games-grid-console-shelf-dark.png) |

## Game details

Select a game to open its details: the console, the region, whether the game is identified,
how many files it has, its size on disk, its size unzipped, and its cover art's size (only when
its picture shows; with only a video or a manual it says so). When the game has
a screenshot or a title screen, **Screens** shows them side by side. Close the panel with its
close button or Escape.

**Identified** is one of the three labels in [What the labels mean](#what-the-labels-mean).
**Identified by** names the game database and version that matched it, and that database's own
title for the game (which can differ from the file name). Some databases, such as FinalBurn
Neo's, file games under a short code; that code is shown too, as "DAT entry". It says "Not
identified yet" until you import a database and identify (see
[Identify your games](#identify-your-games)).

**About this game** shows a game's description when a ScreenScraper lookup found one (see
[Look up on ScreenScraper](#look-up-on-screenscraper)), with "From ScreenScraper" under it.

| Light                                                           | Dark                                                                 |
| --------------------------------------------------------------- | -------------------------------------------------------------------- |
| ![A game's details](screenshots/drawer-console-shelf-light.png) | ![A game's details, dark](screenshots/drawer-console-shelf-dark.png) |

## Your libraries

**Home** › **Your libraries** lists every folder Romperoom keeps track of: its name, where it is,
how many games it holds and when it was last scanned. Two folders with the same name are told
apart by a number (`roms`, `roms (2)`). Fix up, card sync and the card wizard use the same
names. A library whose drive isn't connected stays in the list, marked "Can't reach its folder.
Is the drive connected?" (or "Its folder is empty. Is the drive connected?" when the drive left an
empty folder behind).

- **Add a library…** opens the folder picker. Choose the folder that holds your console folders.
  Romperoom doesn't scan it until you press its **Scan** button. It refuses a folder that is
  already one of your libraries, or one inside or around another library (its games would be
  counted twice).
- **Scan** and the library's name (for example **Scan roms (2)**) reads that library's folder for
  new, moved and removed games, and counts them. Fix up's scan button reads every library.
- **Remove** asks first. It only makes Romperoom forget the library: its games, its Fix up
  history and what it set aside, its Standardise and re-link runs and its card syncs. Nothing in
  its folder is moved, changed or deleted. Files set aside in Fix up stay in its
  `.romperoom-quarantine` folder, but Romperoom no longer offers to put them back. Removing your
  only library takes you back to setup.

Romperoom won't remove a library while a scan runs, while it is busy with that library (a tidy,
Standardise, re-link, cover art, identifying games, a game database import, or a card), or while
something it was doing there stopped part way: finish or undo that in **Fix up** › **History** first
(see [troubleshooting](troubleshooting.md#a-library-cant-be-removed)).

| Light                                                            | Dark                                                                  |
| ---------------------------------------------------------------- | --------------------------------------------------------------------- |
| ![Your libraries](screenshots/libraries-console-shelf-light.png) | ![Your libraries, dark](screenshots/libraries-console-shelf-dark.png) |

Good to know:

- Adding a removed folder again starts afresh: it needs a scan, and its history and card syncs
  are not brought back.
- A game you keep in two libraries counts in each library's games.
- "Can't reach its folder" is decided by reading the folder for up to 3 seconds; a slow network
  drive can read as not reachable and be fine a moment later. The list reads every folder again
  each time you open Home.
- A library whose folder was moved and replaced by a symbolic link reads "Can't reach its
  folder", yet its **Scan** button still works and counts its games (see
  [troubleshooting](troubleshooting.md#a-library-says-it-cant-reach-its-folder-but-it-scans)).
- If a network drive holding one of your libraries stops answering, **Add a library…** makes
  Romperoom stop responding until the drive answers or the system gives up on it (see
  [troubleshooting](troubleshooting.md#a-folder-cant-be-added-as-a-library)).
- Removing a library while **Identify games** works through all your libraries can show an error
  for the removed one; the others are identified as usual.
- Two connections to the same network share added as two libraries look like two libraries.

## Fix up

**Fix up** gathers everything that looks after your library. It opens on **Needs attention**: the
same short list as Home; the cleanups' overview (what each could free, what is set aside, the device
Standardise uses); the space used by console and the largest games; and any folders that look empty
or gone (see [When folders look empty or gone](#when-folders-look-empty-or-gone)). Fix up's one scan
button and the last scan's outcome sit above the tabs, whatever tab is open: **Scan all libraries**
with more than one library, or **Scan** and the library's name with one. It reads your drives for
new, moved and removed games, which takes a while for a big library; press it after you change your
files. It waits, saying why, while a tidy runs. A tab that needs a scan says why and points at this
button rather than showing its own; only **Duplicates** keeps a scan of the one library it shows,
when you have several. **Refresh list**, where you see it, only asks Romperoom again for what it
already knows: it never reads your drives. With more than one library, **Library to tidy** in Fix
up's header says which library Duplicates, Standardise (in Names), Merge picture folders, leftover
artwork and Re-link, Junk files and Set aside work on; it is not shown on Needs attention and Game databases, which cover every library.
The tabs:

- **Names**: identify your games and check name matches, the unidentified games, the folders
  without a console (and the ones you ignored), the files that couldn't be read, and
  [Standardise](#standardise-your-library).
- **Duplicates**: copies of the same game, **In one library** or **Across libraries** (see
  [Duplicates, leftover artwork and History](#duplicates-leftover-artwork-and-history)).
- **Artwork**: [Cover art](#cover-art), [why a game shows no picture](#why-a-game-shows-no-picture)
  and leftover artwork.
- **Junk files**: the `._` files, `.DS_Store` and `Thumbs.db` that Finder and Windows leave
  beside your games, found and removed after you have seen them (see [Junk files](#junk-files)).
- **Artwork**: [Cover art](#cover-art), [why a game shows no picture](#why-a-game-shows-no-picture),
  [Merge picture folders](#merge-picture-folders) and leftover artwork.
- **Game databases**: see [Getting game databases](#getting-game-databases).
- **History**: what Fix up did, with Undo, what is set aside, and work that stopped part way.

| Light                                                | Dark                                                      |
| ---------------------------------------------------- | --------------------------------------------------------- |
| ![Fix up](screenshots/fixup-console-shelf-light.png) | ![Fix up, dark](screenshots/fixup-console-shelf-dark.png) |

## Cover art

The **Games without cover art** card in **Fix up** › **Artwork** counts the games missing a picture,
and how many miss box art, screenshots and title screens. It fills gaps only: Romperoom never
replaces a picture you already have, and goes online only when you press **Get cover art** (to
GitHub) or look up games on ScreenScraper (see [Look up on
ScreenScraper](#look-up-on-screenscraper)).

| Light                                                 | Dark                                                       |
| ----------------------------------------------------- | ---------------------------------------------------------- |
| ![Cover art](screenshots/art-console-shelf-light.png) | ![Cover art, dark](screenshots/art-console-shelf-dark.png) |

**Get cover art** first asks GitHub for the list of pictures for each console with gaps (one request
per console, usually), then shows a review before anything is downloaded: each console with how many
pictures it found, for how many games, how many of those pictures were matched by the game's file
name, their size, and how many pictures are not available (for example "Game Boy Advance: 2 pictures
for 2 games (2 matched by file name) · 135.3 KB · 3 not available"). Each picture not available is
one game and one kind this download can't fill: the collection has no picture by that game's name,
or the game can't take one (see [Troubleshooting](troubleshooting.md#a-picture-didnt-appear)). Then
come the kinds (box art, screenshots, title screens); the total; the source and its terms; and how
many of GitHub's hourly requests the review used and how many are left. Untick any console or kind
you don't want. The terms read: "Pictures from libretro-thumbnails, a community collection used by
RetroArch. The pictures are box and screen images of commercial games; the collection declares no
licence." A console Romperoom can't fetch for (no collection for it, or GitHub's hourly limit
reached) is shown with how many of its games go without ("Super Nintendo: not available for 1
game"), but can't be chosen. Above 2 GB the review warns that the download may take a while; it
never refuses. While a scan runs, **Download** waits for it to finish. **Cancel** backs out without
downloading anything.

| Light                                                                  | Dark                                                                        |
| ---------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| ![Get these pictures?](screenshots/art-review-console-shelf-light.png) | ![Get these pictures?, dark](screenshots/art-review-console-shelf-dark.png) |

**Download** shows its progress picture by picture, with **Stop**: pictures saved before you
press it are kept. The results start with what changed ("Saved 5 of 5 pictures", and how many
failed, if any), then list, for each console you chose, how many pictures were saved, already
present, not available, and any that failed with the reason ("Game Boy Advance: 2 saved, 0
already present, 3 not available"). Not available includes the pictures the review already
counted so, in the kinds you chose; "Saved 5 of 5" counts only the pictures Romperoom tried. The
covers show in the library at once, without a scan.

| Light                                                                 | Dark                                                                       |
| --------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| ![Cover art results](screenshots/art-results-console-shelf-light.png) | ![Cover art results, dark](screenshots/art-results-console-shelf-dark.png) |

**Import art from an SD card** takes pictures from a card you used with a device (choose the
card and the device, as when you put games on a card). Romperoom only reads the card: it reviews
the pictures that fill a gap, per console and kind, and imports the ones you keep. It never adds
a picture for a game that already has that kind.

### Look up on ScreenScraper

**Look up on ScreenScraper** fills the gaps libretro-thumbnails left, and can save a short
description of each game, from [ScreenScraper](https://www.screenscraper.fr/), a community
database. It needs two things: your own ScreenScraper account (free), saved in **Settings** ›
**ScreenScraper**, and Romperoom's own registration with ScreenScraper, which this copy of
Romperoom does not have yet. Until both are there the button is unavailable and says why: "This
copy of Romperoom can't use ScreenScraper yet.", "Save your ScreenScraper account in Settings ›
ScreenScraper first.", or, on a computer with no system keychain, that Romperoom won't save a
password there.

The button first shows a review, without asking ScreenScraper anything: each console with how
many of its games are missing something ("Game Boy Advance: 3 games"), **What to fill** (box
art, screenshots, title screens, descriptions), and exactly what each lookup sends. For each
game: the game file's checksums (SHA-1, and MD5 and CRC when Romperoom has them), its file name
and size, and its console, with your account (user name and password) and Romperoom's app name,
developer id and developer password. For each picture it then fetches, a second request sends
the game's ScreenScraper number, its console and the kind of picture, with the same account and
app details. **Look up N games** asks about one game at a time, at most 1,000 a run,
and stops when your account's daily limit is reached. A game is filled only when ScreenScraper
matched it by its checksums; one it found only by name gets nothing. The results say how many
pictures and descriptions were saved, and for how many games ScreenScraper had nothing. Like
Get cover art, it only fills gaps: get cover art from libretro-thumbnails first. A game Romperoom
can't save a picture for (its file name is too long for a picture's name, for example) is counted
and looked up only for a description, so ticking only picture kinds spends no lookup on it.
A lookup can take an hour or more; scanning, copying to a card, tidying and identifying still work
meanwhile, because a library is held only while a picture is being saved into it. That has a
cost: while one of those holds a library, each picture for its games is skipped and counted ("The
library is busy with a scan. Try again when it finishes."). A skipped picture is not downloaded,
so it spends none of your account's daily lookups, but the lookup does not wait for it either:
look up again once that work is done to fill it. If a scan or Identify changes a game's file
during a lookup, Romperoom saves nothing from the answer for that game, since the answer was for
the file it asked about.

**Remove downloaded art** deletes the pictures Romperoom added, and only those still exactly as
it saved them; it asks first, in place, saying how many there are and their size. A picture you
changed, or one you added yourself, stays.

Pictures go in your library's own `.romperoom/media` folder, one folder per console and kind.
Each is written first to a temporary folder beside it, `.romperoom/tmp`, and moved into place
once complete. Romperoom never changes your games, and writes nowhere else in your library for
this.

## Why a game shows no picture

A game can show a placeholder though its art is on your drive. **Fix up** › **Artwork** says how
many pictures Romperoom found in your libraries, how many it linked to a game, and why the rest
weren't: no game of that exact name in that library, case ignored (a picture is linked when its
name is the game's file name,
like `Tetris (World).png` for `Tetris (World).zip`), or the picture is in a folder Romperoom
couldn't match to a console. A game's cover is its box art, else a screenshot, else a title
screen, else any other picture of it (a marquee, a wheel, an ES-DE mix image or 3D box). Mix
images and 3D boxes are used only as a last-resort cover: they are not box art, so Get cover art
still offers box art for those games. A picture added since the last scan shows after the next
scan.

Romperoom reads pictures from these folders in each library:

- `.romperoom/media/<console>/box`, `screenshot` or `title` (where Get cover art saves), or any
  other folder there
- `downloaded_media/<console>/covers`, `screenshots`, `titlescreens`, `marquees` (ES-DE);
  `miximages`, `3dboxes` or any other folder there, used as a last-resort cover
- `<console>/images`, `image`, `covers`, `boxart`, `screenshots`, `snaps`, `marquees`, `media`,
  `imgs`, `downloaded_images`, `snap`, `wheel`

Not read yet: a picture named with a suffix (`Tetris-thumb.png`), art listed only in a
`gamelist.xml`, muOS or LaunchBox folders, an art folder of its own per library, and folders that
are shortcuts to another place (see the [roadmap](roadmap.md)).

## Getting game databases

Romperoom ships with no game databases (DAT files) of its own. Open **Fix up** and its **Game
databases** tab: it offers two ways to get them, above the list of what you
have already imported.

### Get a database from the official site

Pick a console under **Get a database from the official site** (consoles in your library come
first) and press **Open download page**. Romperoom opens the group's own site in your browser —
[No-Intro](https://datomatic.no-intro.org/) for cartridge-based consoles and handhelds,
[Redump](http://redump.org/) for disc-based ones — and tells you the file name to look for.
Romperoom itself makes no request for this: your browser downloads the file under the site's own
terms.

Once you have it, use **Import a DAT file** in the same section, just below: its own **Console
for the file you import** says which console the file is for (leave it at **Work it out from the
file** and Romperoom reads it from the file itself); once you've just opened a download page, the
picker opens straight to your Downloads folder with the file you got already chosen, where your
system supports that. If Romperoom can't tell the console from the file, it asks you to choose one
before it imports anything — when it has a guess, that guess is already chosen, for you to
confirm. Once an import is done, this console choice goes back to **Work it out from the file**,
so the next file is never tied to the console you chose for the last one.

| Light                                                            | Dark                                                                  |
| ---------------------------------------------------------------- | --------------------------------------------------------------------- |
| ![Game databases](screenshots/databases-console-shelf-light.png) | ![Game databases, dark](screenshots/databases-console-shelf-dark.png) |

### Download for me

Under **Download for me**, press **Review downloads** to see what Romperoom would fetch for the
consoles already in your library, from [libretro-database][libretro-database] on GitHub (a copy
of No-Intro's and Redump's data, converted and shared under the Creative Commons
Attribution-ShareAlike 4.0 licence). Nothing is downloaded until you review the list:
every file, its size, the total, where it comes from and the licence, before you press
**Download**. A console already up to date is shown but can't be chosen again; one with a newer
version is picked for you; one you already have a database for from elsewhere is not picked, and
downloading it replaces that database. **Cancel** backs out without downloading anything.

Above the consoles, **BIOS checksums** offers libretro's list of BIOS files (`System.dat`, about
75 KB), so Romperoom can recognise a BIOS file by its contents whatever its name. It is picked
the first time and when a newer version is listed, and can't be chosen again once up to date.
Once stored, the section says which version and how many files it lists, and the results say
"BIOS checksums: N files listed". Then a card copy renames each BIOS file it recognises to the
name the device looks for (see [Put games on an SD card](#put-games-on-an-sd-card)).

| Light                                                                     | Dark                                                                           |
| ------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| ![Review downloads](screenshots/databases-review-console-shelf-light.png) | ![Review downloads, dark](screenshots/databases-review-console-shelf-dark.png) |

While it downloads, Romperoom shows its progress file by file; **Stop downloading** keeps whatever
has already been imported. The results list what happened to each file — imported, replaced,
already up to date, or why one failed — with **Identify now** offered when something changed.

| Light                                                                      | Dark                                                                            |
| -------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| ![Download results](screenshots/databases-results-console-shelf-light.png) | ![Download results, dark](screenshots/databases-results-console-shelf-dark.png) |

**Check for updates** looks for a newer version of libretro-database's listing and says so next
to the button, or says why it couldn't check. **Network activity**, below both sections, lists
every request Romperoom has made for this feature — when, where to, what happened and how many
bytes — so you can see for yourself that it only ever goes to GitHub, or to ScreenScraper for a
lookup, and only when you pressed Download, Check for updates, Get cover art or Look up N
games. A ScreenScraper line shows its host and path only, never your account or a checksum. It is
also in **Settings** › **Network activity**.

### Managing what you have imported

Each imported database is listed below both sections, with its console, its name, where it came
from, its version, the day you imported it, its number of games and its size. A database Romperoom
downloaded for you also has a **Details** disclosure, named with that database (several downloaded
databases are never all just "Details"): the source repository, the commit, the file's address and
a **Licence** link. **Remove** asks you to confirm in place before it takes a database off the
list. Games it had named go back to their file names until you identify again with the remaining
databases.

## Identify your games

Once you have imported a game database, **Fix up** › **Names** shows an **Identify your games**
card. Press
**Identify games** to compare every file against the databases you imported for its console. It
shows the phase it is in (reading your games, matching them, grouping discs and versions, then
linking cover art) and how far it has gotten; **Stop** stops it there. Your files are never
changed, but labels it had already worked out before you stopped it stay, with their cover art;
the next run picks up where this one left off.

A scan that finds a game database already imported identifies your library on its own once it
finishes, so you rarely need to press the button yourself after the first time. When you scan
several libraries at once, each one is identified in turn as the one before it finishes.

Identifying never changes the files in your library: it only reads them. When it cannot finish
(the library drive was disconnected, or many files in a row could not be read), it changes
nothing it could not check and says so, rather than showing a count of zero; the next run carries
on from where this one stopped. The card also flags when your databases changed since the last
identify, so you know the labels may be out of date until you identify again.

| Light                                                          | Dark                                                                |
| --------------------------------------------------------------- | -------------------------------------------------------------------- |
| ![Identify your games](screenshots/identify-console-shelf-light.png) | ![Identify your games, dark](screenshots/identify-console-shelf-dark.png) |

## What the labels mean

Identifying says how sure Romperoom is about each game, the same words wherever they appear (the
card above, and a game's own details):

- **Verified:** the file matches a known good copy of the game.
- **Name match:** its name matches a game in your databases, but its contents don't match any
  known copy (it was read and checked, and differs: a hack, a translation, a bad copy or a
  different version).
- **Unidentified:** Romperoom doesn't recognize this game yet. It still plays as usual.

An unidentified file says why:

- **Not read yet:** Romperoom couldn't read this file. Scan its library, then identify.
- **In another console's database:** the file matches a game in another console's database. Move
  it to that console's folder to identify it.
- **No database for this console:** you have not imported a database for its console.
- **Not in your databases:** the console's databases don't list this exact file, or you rejected
  every match they offered for it. It may be a hack, a translation or a bad copy.

Files a scan hasn't finished reading yet are skipped; they are identified once a later scan has read
them. Files a scan has read but no identify has looked at since (for example after you removed a
database) are counted as not identified yet: press **Identify games**. **Why each game is
unidentified** lists every one of them by name, with its reason (for "In another console's
database", the console it matches).

A verified file the database itself flags as a bad dump still plays, but says so under
**Identified by**: it may not be a clean copy.

## Check name matches

A name match, or a file that matches several games at once, is offered for review on the same
card, named by the database's title and the file it matches. **Accept** it to keep it. **Reject**
asks you to confirm, since Romperoom never offers that pairing again; once confirmed, it tries the
next match, or puts the file back under its own file name once you have rejected every match it
had (the file then leaves the list). **Show more** loads further matches when there are many.

To keep many matches at once, tick the ones you want (Space ticks one; Shift-click or
Shift+Space does the same to every match between it and the last one you changed), or tick
**Select all** above the list, which reaches every match, also those not loaded yet: it reads
them first ("Reading every match: 200 of 600"), then ticks them all, keeping any you ticked
meanwhile. If that read fails, Romperoom says so and ticks nothing more. A tick is for a match,
the file and the game together: when a match is rejected (on its own or with **Reject all on this
page**) and Romperoom offers the file's next match, that new match is not ticked.

Then press **Accept selected (N)**: N is the number of ticked matches on the list as Romperoom
last read it, counted once each, and a match that leaves the list drops out of it. Nothing is
kept until you press it. While it works, the tick boxes and **Select all** wait ("Accepting…"),
and N stays as it was when you pressed it. Romperoom keeps only matches still waiting on a
decision, and then says how many it kept (its own count, which can be fewer if the list changed
meanwhile, for example after identifying games in another window). If some couldn't be kept, it
says how many, reads the list again so it shows what was kept, and the rest stay ticked. Kept
matches have no Undo. **Reject all on this page** rejects everything currently loaded at once,
after asking.

| Light                                                    | Dark                                                          |
| --------------------------------------------------------- | ---------------------------------------------------------------- |
| ![Check name matches](screenshots/review-console-shelf-light.png) | ![Check name matches, dark](screenshots/review-console-shelf-dark.png) |

## When folders look empty or gone

If a console's folder, or its cover art, seems to have vanished since the last scan, Romperoom
does **not** take those games off your list straight away. A drive that is only partly
connected looks just like deleted games. Instead it shows **Some game folders look empty or
gone** or **Some cover art folders look empty or gone**, with the folders named.

- If the drive was disconnected, reconnect it and scan (Fix up's **Scan all libraries**, or
  **Scan** and the library's name).
- If you removed the games on purpose, press **It's OK — they were removed**. Romperoom takes
  them off its list. Nothing on your drive is deleted.

| Light                                                                           | Dark                                                                                 |
| ------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| ![Folders that look empty or gone](screenshots/removal-console-shelf-light.png) | ![Folders that look empty or gone, dark](screenshots/removal-console-shelf-dark.png) |

## Files that couldn't be read

A damaged zip, a password-protected archive or a drive that stopped answering shows up in **Fix up**
› **Names** with the reason. Damaged files usually need to be downloaded again: Romperoom never
changes your files. Press **Try again** to read every one of them again now.

| Light                                                                          | Dark                                                                                |
| ------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------- |
| ![Files that couldn't be read](screenshots/unreadable-console-shelf-light.png) | ![Files that couldn't be read, dark](screenshots/unreadable-console-shelf-dark.png) |

## Settings and themes

**Settings** (top right) has three tabs: **Appearance**, **ScreenScraper**, where you save your
own ScreenScraper account for [Look up on ScreenScraper](#look-up-on-screenscraper) (your user
name and password go into your computer's keychain, and the tab then only says that an account
is saved, with **Forget the account**), and **Network activity**: every request Romperoom has
made, newest first. Appearance changes how Romperoom looks:

- **Theme:** Console shelf (warm and cosy), CRT neon (glowing arcade colours) or Clean modern
  (quiet, so the cover art stands out).
- **Light or dark:** light, dark, or **Match my computer**.

Your choice is remembered.

| Light                                                     | Dark                                                           |
| --------------------------------------------------------- | -------------------------------------------------------------- |
| ![Settings](screenshots/settings-console-shelf-light.png) | ![Settings, dark](screenshots/settings-console-shelf-dark.png) |

| Light                                                                          | Dark                                                                                |
| ------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------- |
| ![Settings › ScreenScraper](screenshots/screenscraper-console-shelf-light.png) | ![Settings › ScreenScraper, dark](screenshots/screenscraper-console-shelf-dark.png) |
| ![Settings › Network activity](screenshots/network-console-shelf-light.png)    | ![Settings › Network activity, dark](screenshots/network-console-shelf-dark.png)    |

| CRT neon, dark                                              | Clean modern, dark                                                  |
| ----------------------------------------------------------- | ------------------------------------------------------------------- |
| ![Home, CRT neon theme](screenshots/home-crt-neon-dark.png) | ![Home, Clean modern theme](screenshots/home-clean-modern-dark.png) |

## Put games on an SD card

**SD card** in the header copies games to an SD card for a handheld, or for a frontend such as
ES-DE. It opens on three choices: **Copy games to a card**, **Sync a card** (see
[Sync a card](#sync-a-card)) and **Saved packages** (the wizard at What to copy, for the device you
used last). A copy that is still running, or its report, opens straight away instead. The copy
wizard has six steps, shown along the top throughout, and until you press **Start copying**
nothing is written. Put the card in your computer first. Cover art from a card is in **Fix up** ›
**Artwork**: see [Cover art](#cover-art).

| Light                                                                    | Dark                                                                          |
| ------------------------------------------------------------------------ | ----------------------------------------------------------------------------- |
| ![SD card's choices](screenshots/deploy-landing-console-shelf-light.png) | ![SD card's choices, dark](screenshots/deploy-landing-console-shelf-dark.png) |

1. **Device.** Pick the device or frontend the card is for: Batocera, ES-DE, muOS, Onion (Miyoo
   Mini), or the DSpico or R4xMP flashcart. Each one says how many consoles it plays. All six are marked
   **Community**: they were set up from the projects' own documentation and what other players
   shared, and not yet tested on a real device by us. A console in your library that the device
   can't play is named under the choices, and its games stay off the card.

   **R4xMP** (a Nintendo DS flashcart) gets your DS, Game Boy Advance, Game Boy Color and NES games
   in its own folders at the card's root: `NDS ROMS`, `GBA ROMS`, `GBC ROMS` and `NES ROMS`. Only
   `.nds`, `.gba`, `.gb`, `.gbc` and `.nes` files go; no box art, BIOS, saves or game lists, and no
   Game Boy games (it has no folder for them). Before you copy, the card must be formatted FAT32
   (another file system is refused, with a warning that reformatting erases the kernel too), be
   32 GB or smaller (Romperoom warns about a larger one, with its size), and hold the R4xMP's kernel, which you
   install yourself: the `__rpg` folder, `_DS_MENU.DAT` and `_DSMENU.DAT` at the card's root.
   Romperoom never writes, moves or removes those. Like every card it copies to, the root also
   gets Romperoom's own `romperoom-manifest.json` (what it copied) and, once a later copy leaves
   a game out, a `.romperoom-removed` folder holding it. The R4xMP's kernel needs every game as a
   plain file, so Romperoom **unzips zipped games as it copies them** (see
   [Zipped games on a flashcart](#zipped-games-on-a-flashcart)). The layout comes from one seller's
   page and is **unconfirmed** on a real card, as are these: how long a file name may be
   (Romperoom uses FAT32's 255 characters), and any other console or file type.

   **DSpico** (a Nintendo DS flashcart that boots Pico Launcher) gets your DS games in `roms/nds`
   on the card. Game Boy Advance, Game Boy Color and NES are offered too, in `roms/gba`, `roms/gbc`
   and `roms/nes`, but start unticked: each needs an emulator program you install on the card
   yourself, and which files it reads is **unconfirmed**. Tick one under **What to copy** to add
   it; until you do, none of its games is copied (and a package that names no consoles says so).
   FAT32 is assumed, not confirmed: on another file system the Check step warns and still copies.
   If the card already starts Pico Launcher, copy as it is; reformatting it erases everything on
   it, Pico Launcher too. The card must hold Pico Launcher, which you install: `_picoboot.nds` and
   the `_pico` folder at the card's root. Romperoom never writes, moves or removes those (its own
   `romperoom-manifest.json` and `.romperoom-removed` go at the root too). Only `.nds` for DS games is confirmed,
   from third-party guides rather than the official manual; file name limits, saves and covers
   are unconfirmed, so no saves, art or game lists are copied. As for the R4xMP, zipped games
   are **unzipped as they are copied**, for every console it takes (see
   [Zipped games on a flashcart](#zipped-games-on-a-flashcart)).

   For either card, choosing it says so under the choices: zipped games are unzipped as they are
   copied, and your zip files stay as they are.

   | Light                                            | Dark                                           |
   | ------------------------------------------------ | ---------------------------------------------- |
   | ![Choosing the device, light][deploy-device-lt] | ![Choosing the device, dark][deploy-device-dk] |

2. **What to copy.** Tick the consoles you want. A console a device plays only with extra
   software (DSpico's emulator programs) starts unticked, with a note saying why. **One version of each game** keeps the best
   region in the order you set and leaves the rest out; a region that has every disc of a
   multi-disc game wins over one missing a disc, and the review step names any disc the library
   doesn't have. **Skip identical copies** leaves out a
   second copy of the same file. You can also choose the kinds of artwork, and for devices with
   a BIOS folder, a folder of BIOS files to copy. With BIOS checksums downloaded (see
   [Download for me](#download-for-me)), a BIOS file is recognised by its contents and copied
   under each name the device looks for, whatever it is called in your folder; the check step
   names any console whose BIOS files were not found. For a device that shows box art at a fixed
   width (Onion), Romperoom copies a smaller box art picture, never wider than the device shows
   it; your own picture is not changed. Only PNG pictures are made smaller. A device that shows
   only PNG pictures (muOS, Onion) gets a PNG of each JPEG or GIF picture, and ES-DE and Batocera
   get one of each GIF: see [Pictures converted to PNG](#pictures-converted-to-png). A size estimate
   updates as you choose.
   Romperoom remembers these choices for next time. To keep more than one set for a device, save
   it under a name: see [Saved packages](#saved-packages).
3. **Where.** Pick the card. Each card shows its size and free space. A card Romperoom can't use
   is greyed out with the reason, for example the disk your computer runs from, or a card that is
   locked. For a disk inside your computer, you type its name to show you mean it. You can also
   **Export to a folder** and move it to the card yourself, or **Just check a card size** before
   you buy one. If the folder is on a network drive, or Romperoom can't tell whether it is, it
   asks first: tick **Copy to** (the folder's name) **anyway** to go on. That tick is for this
   choice of folder only; choosing a folder again asks again. The meter shows the games, the
   artwork, the space lost to the card's format and what is left. If it doesn't fit, Romperoom
   offers what to leave out: the artwork, the other versions of each game, or a large console;
   **Suggest games to leave out** lists the games themselves (see [Make it fit](#make-it-fit)).

   ![Choosing the card, with its space][deploy-where-lt]

4. **Check.** A summary of the games, consoles and space, what was left out and why, and
   anything that stops the copy. For a card whose games are unzipped (R4xMP, DSpico), it also says
   how many zipped games are unzipped and the space they take on the card, and how much more
   that is than their zips; the space shown everywhere is the unzipped size. When a game file is in two of your libraries with different
   contents (the same name, other bytes), one copy goes to the card and **Copies that differ**
   names the library it comes from: the copy that matches your game database, else the one in
   the shorter folder name, else the one Romperoom found first (to send the other copy, see
   [The card got a game from the other library](troubleshooting.md#the-card-got-a-game-from-the-other-library)).
   **Preview the changes** works out what would happen without writing anything. **Write … to
   …** asks once more, with what Romperoom promises: it only writes to this card, it keeps files
   it didn't put there, and anything it removes goes to the `.romperoom-removed` folder on the
   card. On a Mac copying to a FAT or exFAT card, **Remove the ._ files macOS adds** is ticked:
   macOS writes a small `._` file beside every file and folder it copies to such a card, and the
   copy removes the ones it made (only those, and only in the device's folders). Untick it to keep
   them; Romperoom remembers your choice.

   | Light                                    | Dark                                   |
   | ---------------------------------------- | -------------------------------------- |
   | ![The check step, light][deploy-check-lt] | ![The check step, dark][deploy-check-dk] |

5. **Copy.** A progress bar with the files and bytes so far, the speed and the time left.
   **Cancel the copy** stops after the file being written; what was already copied stays, and
   the next copy picks up from there. If you try to quit during a copy, Romperoom asks first.
6. **Done.** What was copied, what was already there, and anything that couldn't be copied (open
   **Details** for the reasons). Details name your card, libraries and folders by their names,
   never their full paths; a detail that would name any other folder on your computer reads "A
   detail is left out here because it names a folder on this computer." Eject the card before
   you take it out. After a copy from a Mac to a FAT or exFAT card it also says how many macOS
   metadata files it removed and the free space left afterwards. **Clean up this card** finds the
   other junk files in the device's folders on the card (a `.DS_Store` Finder left, `._` files
   from earlier copies) and shows them before anything is removed, as in
   [Junk files](#junk-files); it never touches the card's other files. **Copy more** goes back to
   the consoles with the same device; **Start over** forgets the choices.

   ![The report after a copy][deploy-done-lt]

Copying the same selection again only writes what changed. If you leave a console out next time,
its games are moved to `.romperoom-removed` on the card, not deleted: delete that folder yourself
once you are sure. Files you put on the card yourself are never replaced or moved.

- **A card set up for another device.** If you last copied to this card for a different device,
  the Check step says so after **Preview the changes**, with a switch: **Replace what Romperoom
  put here for** that device. Leave it off and the games copied for the other device stay as
  they are; only new files are added. Turn it on and they are replaced, or moved to
  `.romperoom-removed` if the new device does not need them. Your own files are never touched
  either way.
- **Play counts and favourites.** ES-DE and Batocera save them in the game list Romperoom put
  on the card. Romperoom keeps that list as the device left it and only adds the games it is
  missing. If the list on the card is not valid UTF-8 text, or is not a plain `<gameList>` file
  (another root element, for example), Romperoom leaves it untouched and reports it as a
  conflict with that reason; the games are still copied, but they are not added to that list.
- **A card from a newer Romperoom.** If a newer version of Romperoom last copied to this card,
  the Check step asks you to update the app first, and nothing on the card is changed.
- **Two games with the same name on the card.** When two games would end up with one file
  name, each gets a short code from its own file, such as `Game (3fa9c1).nes`. Adding or
  removing one of them does not rename the other.
- **Identified games.** Once a game is identified, its name on the card comes from the game
  database, not the file name; two revisions of one identified game still count as one game for
  **One version of each game**.

### Zipped games on a flashcart

The R4xMP's and the DSpico's kernels read only plain game files, so for those cards Romperoom
unzips zipped games as it copies them. Your zip files are only read, never changed.

- **What is unzipped.** A `.zip` holding exactly one game for its console (a `.nds` for DS, a
  `.gba` for Game Boy Advance, and so on) is copied as that game. It is named after the zip, not
  the file inside it: `Mario Kart DS (USA).zip` becomes `Mario Kart DS (USA).nds`, then the card's
  usual name rules apply (characters a card refuses, the length limit). Anything else in the zip
  (a readme, a save, a picture) is left behind. So is what a computer adds when it makes a zip:
  the `__MACOSX` folder and `._` files that Finder's **Compress** puts in (a hidden copy of each
  file's details, which would otherwise look like a second game), and `.DS_Store`, `Thumbs.db`
  and `desktop.ini`.
- **What is left out, and why.** The **Check** step lists each under **Left out**: a zip with more
  than one game inside, a zip with no game for its console, a zip Romperoom can't unzip (locked
  with a password, damaged or unreadable, packed with bzip2, LZMA, Deflate64 or zstd as 7-Zip and
  WinZip can, claiming sizes that can't be true, or over 8 GiB unzipped), and `.7z` and `.rar`
  files (not unzipped yet). A game too big for a FAT32 card once unzipped (4 GB or more) is left
  out as too large, like any other. A zip that changed since your library was last scanned is
  left out too, and a note names the button to press: **Scan** and the library's name (or **Scan
  all libraries**), then copy.
- **Looking inside zips takes a moment.** Romperoom reads each zip's list of files once, then
  remembers it while the zip is unchanged. So the app never stalls on a big library on a slow
  drive, each measurement spends at most about two seconds reading zips it hasn't seen yet. Zips
  it didn't reach are listed under **Left out** as "Not checked yet: plan again to continue":
  every time the wizard measures again (going back a step and on, or changing a choice), it
  checks more, until none are left.
- **Space.** Every size the wizard shows (the estimate, the meter, **Make it fit**, the Check
  step) is the unzipped size, since that is what the card needs. The Check step says how many
  games are unzipped, the room they take on the card (in the card's whole blocks) and how much
  more that is than their zips.
- **Checked as it copies.** Each game is unzipped into a temporary file on the card, checked
  against the size and checksum the zip records (and against your game database when it knows
  the game), and only then given its name. A zip that fails the check fails that one game, never
  the whole copy; the report lists it.
- **Copying again.** A game already on the card is left alone, so copying again only adds what
  changed. A zip that changed in your library is unzipped again. A game you leave out of a later
  copy has its unzipped file moved to `.romperoom-removed` on the card, as any other. When the
  card already holds a file of the same size that Romperoom didn't record, it unzips the game to
  compare the two before anything is written; that shows as checking the card, and **Cancel**
  stops it.
- **Sync.** **Sync a card** matches an unzipped game on the card with its zip in your library by
  its contents, so it isn't offered for import again. It may offer an unzipped game again when its
  game is not the largest file in its zip and Romperoom has no record of the zip's other files
  (zips of more than 64 files, or ones scanned before Romperoom kept that record).

### Make it fit

When the games don't fit the card you chose, **Suggest games to leave out** on the **Where** step
lists the games Romperoom would leave out to make room: "Leave out 3 games to make it fit", what
that saves, how many per console, and each game with its console, region and size. It starts
with the biggest games; of games the same size, one whose other version (another region) is
still going comes first. Nothing is left out yet:

- **Keep this one** keeps a game on the card, and Romperoom chooses others in its place. While it
  chooses, the list you were reading stays on screen, but its buttons wait ("Choosing games to
  leave out") until the new list is there; a screen reader then hears what you did and the new
  heading, such as "Keeping Zelda (NES, USA). Leave out 2 games to make it fit." **Start the list
  again** forgets the games you kept.
- Every game on the list is ticked, as suggested. Untick any you want to keep (Space unticks one;
  Shift-click or Shift+Space does the same to every game between it and the last one you
  changed). **Select all** ticks or clears every game, including the ones counted under "And 2
  more games". The heading counts the ticked games, and the bar says how much they free of what
  the card needs ("Frees 1.2 GB of 1.5 GB needed"). When that is not enough, a line under it says
  what is still needed and what to do.
- **Leave out selected (N)** applies the list: it leaves out the N ticked games only, and keeps
  the ones you unticked, as **Keep this one** does, so Romperoom never suggests them again. A note
  says how many are left out, the meter shows the new size, and the **Check** step lists them
  under **Left out**. If the card still doesn't fit, **Suggest games to leave out** chooses others
  in place of the games you kept. **Put them back** forgets the list.
- If even leaving out every game you didn't keep can't make room, Romperoom says so: keep fewer
  games, or choose a bigger card.

The list is for this card and these choices only: changing any choice (or loading a saved
package), choosing another card, **Copy more** and **Start over** forget it, and it is never
saved. Because it starts with the biggest games, it may leave out a big game when only a little
is over; **Keep this one** asks for another choice.


### Pictures converted to PNG

Some devices show only PNG pictures. When you copy to one of them, Romperoom converts your JPEG and
GIF pictures to PNG **on the card only**: your library keeps its own `.jpg`, `.jpeg` and `.gif`
files exactly as they are, and the card gets `<game name>.png`.

- **What is converted:** JPEG pictures (ordinary and progressive) and the first picture of a GIF
  (an animated GIF gives its first frame) for muOS and Onion; GIF pictures for ES-DE and Batocera,
  which show JPEG as it is. A GIF's see-through parts stay see-through. Onion's box art is also made
  at most 250 pixels wide. The pixels are copied as they are: no quality is lost beyond what the
  JPEG had already lost. The way a phone or camera turned a photo (its EXIF rotation) is ignored,
  as most devices do.
- **A PNG wins:** when a game has both a PNG and a JPEG of the same picture, the PNG is copied and
  the JPEG isn't converted. The check step says how many.
- **What isn't converted, and why:** the check step lists each reason with a count, and those
  pictures are left off the card:
  - **WebP** pictures: Romperoom doesn't convert WebP yet;
  - JPEGs in **CMYK** (print) colours, and rare kinds of JPEG Romperoom can't read (12-bit,
    lossless or arithmetic-coded);
  - JPEGs built so that reading them would take too long (more than 100 passes over the picture,
    or passes in an order the JPEG standard doesn't allow);
  - pictures of more than about 4.2 million pixels (2048 × 2048, or any other shape with as many
    pixels), or over 16 MB;
  - damaged pictures, or files whose contents aren't the JPEG or GIF their names say.
- **Space:** a PNG is usually larger than the JPEG it comes from, and its exact size is only known
  once it is made. The space shown counts each converted picture at the largest it can be, so the
  card keeps more room than it shows, and **Make it fit** counts it that way too. The progress bar
  ends at what was really written.
- **Time:** while it measures, Romperoom checks how each picture is built without drawing it (a
  few seconds at most each time; if there are many, the check step asks you to go back a step and
  on again to check more). It converts each picture while it copies, one at a time, away from the
  window, so the app keeps answering and **Cancel** stops at once. Box art takes a fraction of a
  second; the largest picture it converts takes about a second on a recent Mac (several seconds
  on a slower computer). A picture that takes longer than 10 seconds is stopped, listed as "took
  too long", and the copy goes on.
- **Art converted earlier stays:** if a picture can't be checked this time (its file can't be
  read, it changed since Romperoom last scanned, or it wasn't checked yet), the PNG Romperoom put on
  the card on an earlier copy is kept as it is, and the check step says so. Copying removes a
  converted picture only when you no longer copy it (you leave out the game or that kind of art,
  or the picture is gone from your library).
- **Your own files on the card:** a PNG already on the card that Romperoom didn't write is never
  replaced; it is listed as left as it is. A picture that turns out to be damaged while it is
  copied is listed with its reason, and nothing is written for it.

### Saved packages

A saved package is a set of choices for one device under a name you give it, such as `Kids` or
`Road trip`: the consoles, the regions, **One version of each game**, **Skip identical copies**,
the artwork, the BIOS files, what to do with games too big for the card, and the card size you
check. It never holds a card or a folder: you choose where to copy each time. Packages are under
**Saved packages**, at the top of **What to copy**.

- **Save.** Type a name in **Package name** and press **Save as a package** (or Enter). A name
  has up to 40 characters, and each of a device's packages needs its own name (capitals don't
  count). A device keeps up to 20; at 20, delete one to save another.
- **Load.** **Load** sets every choice on the page to the package's; the card or folder you
  chose stays chosen. The package then reads **Loaded**, or **Loaded, then changed** once you
  change something. **Load** again puts its choices back. The mark is only for this visit to the
  wizard: it is gone when you open the wizard again or pick another device.
- **Replace, Rename, Delete.** **Replace** saves the choices on the page into that package.
  **Rename** gives it a new name (Enter saves, Escape cancels). **Delete** forgets it; your games
  and cards are not touched. Replace and Delete ask first, in place of the package's buttons.
- **A console you don't have now.** If a package lists a console that isn't in your library (or
  that the device no longer plays), loading it says so, and a note says so while the package is
  loaded. That console stays off the card. It stays in the package: add its games back and it is
  copied again.
- **Every console.** A package saved after **Select all** also takes consoles you add to your
  library later; one with some consoles ticked keeps to those.
- **A device Romperoom no longer knows.** If an update drops a device, its packages are kept but
  not listed, and a note says how many. **Remove them** forgets them all, after asking.

Packages are kept on this computer only, with Romperoom's other saved settings, so deleting
Romperoom's data folder ([Starting over](troubleshooting.md#starting-over)) forgets them.
Romperoom's memory of your last choices is separate: loading a package sets them, and **Start
over** forgets them but keeps your packages.

[libretro-database]: https://github.com/libretro/libretro-database
[deploy-device-lt]: screenshots/deploy-device-console-shelf-light.png
[deploy-device-dk]: screenshots/deploy-device-console-shelf-dark.png
[deploy-where-lt]: screenshots/deploy-where-console-shelf-light.png
[deploy-check-lt]: screenshots/deploy-check-console-shelf-light.png
[deploy-check-dk]: screenshots/deploy-check-console-shelf-dark.png
[deploy-done-lt]: screenshots/deploy-done-console-shelf-light.png

## Sync a card

**Sync a card** brings back what a handheld picked up away from your library: games copied
straight onto its card, and the saves you made while playing. Newer saves from the library go
back to the card, so you can carry on with a game on any of your devices. Nothing is deleted on
either side, save states and BIOS files are never touched, and Romperoom stays offline.

1. **Choose.** Open **SD card** and press **Sync a card**. Pick the card, the device
   it is from and, with more than one library, the library. Only cards Romperoom can write to are
   listed (the same ones the copy wizard would use). **Read the card** looks at it without
   changing anything; checking games the library may already have under another name can take a
   while for large ones, and **Stop** goes back.
2. **Review.** One screen, and nothing has been written yet:
   - **New games on the card**, per console: each with its size and the library folder it goes
     to (a new folder is marked as such). Games the library already has, by name or under
     another name, are counted, not listed. A file that looks like a BIOS, or is part of a game
     made of several files (a cue sheet and its tracks), starts unticked. A game whose name is
     already used in the library by a different file is shown greyed out, with the reason; it is
     never renamed or copied over. So is a game whose console folder can't be used: something
     else in your library already has that folder's name, or the folder can't be used right
     now. Files in the card's game folders that aren't games are skipped, and counted.
   - **Saves**: **To the library**, **To the card**, and **Changed on both sides**, where you
     pick which copy to keep (each with its size and when it last changed); the other is kept
     as a backup. **Decide later** changes nothing. Saves deleted on one side are listed; the
     other side keeps its copy. A device whose save folders Romperoom doesn't know only has its
     games imported: "Romperoom doesn't know where this device keeps saves."
   - **Saves left alone**, under their console: a save Romperoom can't use (it can't be read, is
     empty or larger than a save should be, isn't a normal file, has another copy in a second folder
     or a twin whose name differs only in capitals, could belong to games on two consoles, or its
     game is no longer on the card), with the reason. Nothing is copied onto or over it, and it is
     never reported deleted. When a console's save folder can't be used, the review says that
     console's saves weren't checked.
   - The totals, and a warning when the library or the card lacks room.

   | Light                                     | Dark                                     |
   | ----------------------------------------- | ---------------------------------------- |
   | ![The sync review, light][sync-review-lt] | ![The sync review, dark][sync-review-dk] |

3. **Sync.** A progress bar with the item and bytes so far, and **Stop**: the file being copied
   is finished or left out, and everything copied before stays. Every save that is replaced, on
   either side, is first copied to the library's `.romperoom/saves-backup` folder, and backups
   are never removed by Romperoom.
4. **Results.** What was imported and synced, anything that could not be (with the reason), and how
   much the backup folder holds. **Scan these consoles** adds the new games to your library (they
   appear after that scan). **Undo this sync** takes back what it put in the library: imported games
   and new saves leave their folders and replaced library saves come back from their backups, but
   only where nothing changed since; each file it left is listed with the reason. Saves it wrote to
   the card stay there; their earlier copies are in the backup folder. **Recent syncs** under **Sync
   a card** lists the latest ones, each with its own Undo while it can still be undone. When the
   library was busy (a scan, say) and nothing was written, **Back to the review** lets you sync the
   same choice once it is free.

   | Light                                            | Dark                                            |
   | ------------------------------------------------ | ----------------------------------------------- |
   | ![The results of a sync, light][sync-results-lt] | ![The results of a sync, dark][sync-results-dk] |

The first sync gives the card a small file, `.romperoom/card.json`, holding only a random id, so
Romperoom knows the card next time. Saves are matched to games by name: `Tetris (World).srm`
belongs to `Tetris (World).gb`. Which devices keep their saves where is in
[profiles.md](profiles.md#saves).

A known limit: a copy of a card made with a disk tool carries the same id, so Romperoom takes it
for the card it was copied from. Syncing both may then show more saves as changed on both sides
than you expect. Nothing is ever deleted either way: pick the copy to keep, or **Decide later**.

[sync-review-lt]: screenshots/sync-review-console-shelf-light.png
[sync-review-dk]: screenshots/sync-review-console-shelf-dark.png
[sync-results-lt]: screenshots/sync-results-console-shelf-light.png
[sync-results-dk]: screenshots/sync-results-console-shelf-dark.png

## Duplicates, leftover artwork and History

**Fix up**'s **Duplicates**, **Artwork** and **History** tabs find copies of the same game and
pictures no game uses, and put back what was set aside. Nothing moves until you have
seen a preview and pressed its button. Nothing is deleted unless you choose Delete forever: files
are **set aside** in a folder named `.romperoom-quarantine` inside your library, and you can put
them back at any time.

**Needs attention** shows how much each cleanup could free, and what is set aside now.

| Light                                      | Dark                                             |
| ------------------------------------------ | ------------------------------------------------ |
| ![Duplicates to tidy][fixup-duplicates-lt] | ![Duplicates to tidy, dark][fixup-duplicates-dk] |

**Duplicates** lists each set of identical copies: files with exactly the same bytes, so two zips of
one game with different saves or patches inside are not copies. Each set shows its game's cover (box
art, else a screenshot, a title screen or any other picture) beside its name, the picture a games
page shows for that game; a set whose game has none shows a plain square. A copy that belongs to
another game (Romperoom matched it by its name) shows that game's cover beside it. One copy is kept,
and the reason is shown, such as "Keeping the USA copy". Romperoom never sets aside the last copy of
a game. Copies that are one file under two names (hard links), tracks of a disc set and the same
file in two consoles' folders are left alone. Duplicates and **Needs attention** say how many files
are skipped that way ("N files are skipped because they're part of a multi-file game or shared
between consoles"), and how many archives from an older version of Romperoom haven't been checked
yet: press **Scan** (on Duplicates it names the library, such as **Scan roms**) to include them.

- **Keep a different copy** lets you pick which one stays. The set's picture doesn't change: it
  is still the one of the copy Romperoom suggested keeping.
- **Which copy to keep** changes the rules for every set: the order of regions (move one up or
  down), and **Prefer unzipped copies**.
- **Set aside all extras** opens the preview.

The **Across libraries** choice on the Duplicates tab lists the files you hold in two or more of
your libraries: files with exactly the same bytes, such as a game on your NAS and the same game on a
USB drive. Each set shows the game's picture, its console, how many libraries hold it and its size,
then each copy with its library's name and where it is inside that library. The line above the list
says how many sets there are and how much space the extra copies take. Each copy is marked **Keep**
or **Set aside**: Romperoom suggests keeping the copies in one library (the one with the copy it
would keep on Duplicates) and setting aside the copies in the others. **Keep the copy in a different
library** chooses another library for that set, or **Keep every copy** leaves the set alone. **Set
aside the extra copies** opens the preview: each copy goes to the set-aside folder of its own
library, never to another drive, and you can put it back from **History** as with any tidy. It is
unavailable, saying why, while a scan runs or while **Refresh list** is fetching the list. The
libraries are tidied one after another; if one stops part way (or you press **Cancel**), the result
names the library it stopped at, counts the files of the libraries it never reached ("3 files in the
other libraries were not started. Refresh the list to set them aside."), and names the kept library
to connect when that is the reason. Just before anything moves Romperoom checks each kept copy again
(it must still be there, with the same contents), and both libraries must be connected and scanned.
Copies inside the kept library are never touched here: **Duplicates** handles those. With one
library there is nothing to compare: add another in **Home** › **Your libraries** (see [Your
libraries](#your-libraries)). **Library to tidy** at the top of Fix up doesn't apply to this choice,
which compares every library: "Every library is compared here." stands in its place.

A library Romperoom can't compare is listed under **Not checked**, with the reason: it is being
scanned, it hasn't been scanned completely, its last scan didn't see every file, its folder isn't
available (is the drive connected?), or part of its folder is also another library. The other
libraries are still compared. When a scan would fix it, the tab says so and points at **Scan all
libraries** at the top of Fix up; **Refresh list** asks Romperoom once more for what it already
knows (it never reads your drives), for example after you connect a drive. As on Duplicates, tracks
of a disc set and the same file in two consoles' folders are left out and counted, and a file
hard-linked into two libraries is one file, so it is not listed. Files that couldn't be read aren't
compared (**Fix up** › **Names** lists them), and a copy moved or renamed since the last scan is
counted as moved or gone until the next scan. The list is as each library's last scan saw it: a
file replaced since with a different one of the same name is still listed. Scan both libraries again
before you remove a copy by hand. After **Refresh list**, a screen reader hears the result once (not
again if it is the same as the last thing read out). See
[troubleshooting](troubleshooting.md#a-library-says-not-checked-under-across-libraries) for each
reason, and for the copies the list [leaves out or shows
twice](troubleshooting.md#across-libraries-leaves-out-a-copy-or-shows-one-twice).

| Light                                              | Dark                                                     |
| -------------------------------------------------- | -------------------------------------------------------- |
| ![Copies in more than one library][fixup-across-lt] | ![Copies in more than one library, dark][fixup-across-dk] |

The **Artwork** tab, under the cover art cards, lists pictures no game in your library uses: art for
games you don't have or that were removed, and extra copies of the same picture. A picture your
frontend reads where it is (ES-DE's `downloaded_media`, or a console folder's `images` or `Imgs`) is
never an extra copy, and art in a folder Romperoom doesn't recognise is left alone. All of them
start ticked: untick any you want to keep (or **Select none**), then press **Set aside**, which
names the count and size. Romperoom can only tell which pictures are left over after a full scan, so
after a scan that didn't finish it says so rather than showing nothing.

When the leftover pictures have more than one cause, **Show** above the list picks one: **All**,
**No game in your library**, **Game removed** or **Extra copies**, each with how many pictures
and how much space. **Select all**, **Select none** and **Set aside** then work on the pictures
shown, and a picture you unticked stays unticked when you show another cause. The pictures the
re-link section above offers are not counted. With nothing ticked, **Set aside** is unavailable
and says why: "Tick at least one picture to set aside." After a scan, pictures of a game whose
files you removed are listed under **No game in your library**; **Game removed** rarely appears.

| Light                                         | Dark                                                |
| --------------------------------------------- | --------------------------------------------------- |
| ![Leftover artwork by cause][fixup-artwork-lt] | ![Leftover artwork by cause, dark][fixup-artwork-dk] |

The preview says how many files, how much space and where they will go (naming the library they are
set aside in), and lists the first of them. **Cancel** has the focus, so pressing Enter by accident
changes nothing. The other button names the count, such as **Set aside 3 files**.

![The set-aside preview][fixup-confirm-lt]

While it runs you see the progress, and **Cancel** stops between files. When it ends it says
what moved and how much was freed, with **Undo all** to put everything back. A file that
couldn't be moved is listed with the reason and stays where it was.

**If you stopped it part way,** the files left stay where they were, and the result offers
**Finish or undo…**: **Finish** moves the rest, **Undo what was done** puts back what already
moved, and **Discard the rest** keeps what already moved set aside and leaves the other files
where they are, so the run no longer waits. Each run says when it started, so two runs of the
same kind are told apart. Discard the rest changes nothing and says so when a
file of the run isn't where the run left it (for example after Romperoom quit in the middle of
moving it): finish it or undo it instead. If Romperoom quit in the middle, the same choice opens
by itself the next time it starts. **Later** leaves it, and **Needs attention** offers it again.

While a run waits like this, **Needs attention** and **History** say "Something Romperoom was
tidying stopped before it finished. Finish it or undo what was done. If it was setting files aside,
you can also discard the rest." It means a tidy, a standardise run, a re-link or a card sync stopped
part way, and nothing more happens to it until you choose. Only a run that was setting files aside
offers **Discard the rest**. While the drive your library is on isn't connected, none of the three
works: Romperoom says so and changes nothing, so connect it first. See
[troubleshooting](troubleshooting.md#romperoom-was-interrupted-while-tidying).

The **History** tab lists every tidy with what is still set aside, and **Undo** for each. A run
whose rest you discarded says **Stopped: the rest was discarded**. A run from Across libraries also
names the library that kept the copies ("· more · kept in roms"). **Finish** of such a run waits
until the kept library is connected when a file it has left to move is kept there; **Undo** does not
need it. When Finish can't reach the kept library, the **Finish tidying up** drawer asks "Is roms
still the library that keeps the other copies?": answer **Yes, reconnect** if the drive was
remounted or the folder moved back, as for the run's own library, or **Not now**, then connect it
and press **Finish** again.

Under it, **Set aside in** (the library's name) lists the files waiting, grouped by when they were
set aside. **Put back** returns one file, and **Put all back** a whole group. Romperoom never
overwrites: if another file now sits where one came from, that one stays set aside and the result
says so.

![History, with the files set aside][fixup-history-lt]

**Delete forever…** is the only way Romperoom deletes a file it did not add itself. Beside it,
choose what to delete: **Everything**, **Older than 30 days** or **Older than 90 days**, meaning set
aside more than 30 or more than 90 days ago; each choice says how many files it holds, and a file's
age is the time the run that set it aside started. With nothing that old, **Delete forever…** is
unavailable and says so. The preview names the age in full ("… set aside in roms more than 30 days
ago will be deleted forever."), shows how much space, and you type `DELETE FOREVER` to confirm. A
file that changed or went missing since it was set aside is kept, and the preview says how many.
So is a copy set aside from Across libraries while the library that kept its other copy is
unplugged or removed, or that copy changed ("the copy kept for it can't be checked"): put it back,
or connect that library first. This
cannot be undone.

**Re-link to renamed games** sits above the leftover artwork. When you renamed a game outside
Romperoom, its pictures keep the old name and look left over. If exactly one game of the same
console has the same title once tags like (USA) or (Rev 1) are ignored, the picture is listed here
with the name it would get, such as `Tetris (USA).png → Tetris (World).png`, and is ticked.
Anything less clear stays in the leftover list with no suggestion, and a picture that matches but
can't be renamed (another picture wants the same game or the same new name, or the name is taken
or too long) stays there with the reason. When every leftover picture is offered here, the
leftover list says **No other leftover artwork**. When the first 200 it loads are all offered here
and there are more, it says so: re-link them to see the rest.

A picture whose game already has a picture of that kind starts unticked here: ticking it adds this
one too, and nothing is ever replaced. It is also in the leftover list below, ticked like every
leftover, so **Set aside** would move it to the set-aside folder with the rest: untick it there if
you want to keep it.

The part **Game list entries without a game** lists each entry of a console folder's
`gamelist.xml` whose game file isn't there. One that one game clearly matches is offered with its
new path; the rest are only listed. Romperoom never deletes an entry.

| Light                                      | Dark                                             |
| ------------------------------------------ | ------------------------------------------------ |
| ![Re-link to renamed games][relink-rev-lt] | ![Re-link to renamed games, dark][relink-rev-dk] |

Press **Re-link N items**. Each picture is renamed in its own folder, never overwriting anything,
and the game list entries that name it follow it, with the original list kept in
`.romperoom/lists-backup`. A game list Romperoom can't read safely is left as it is, and the
picture is still renamed. **Undo this re-link** (or **Undo** under History, in **Recent
re-links**) puts the pictures and the lists back, as long as they haven't changed since.

| Light                       | Dark                              |
| --------------------------- | --------------------------------- |
| ![Re-linked][relink-res-lt] | ![Re-linked, dark][relink-res-dk] |

While it runs you see the progress, and **Stop** stops between pictures: everything done before you
pressed it is kept. A run that stopped part way offers **Finish or undo…**, to finish the rest or
put back what already changed. If your library changed since the list was made, or Romperoom is
busy with another job, the run is refused before anything changes, and **Refresh list** makes a
fresh list. While a stopped or interrupted re-link waits to be finished or undone, Standardise
won't run on that library (and the other way round). After you reload the window, Fix up
shows only the latest result, of a re-link or a standardise run; the other is under **History**.
A game on several discs that Romperoom recognises disc by disc is never offered; see
[troubleshooting](troubleshooting.md#a-picture-wasnt-offered-for-re-linking) for what else isn't.

If Fix up says Romperoom is busy, the library folder isn't available, or your library changed
since you looked, see [troubleshooting](troubleshooting.md#fix-up-says-romperoom-is-busy).

## Merge picture folders

Frontends and tools keep box art in a folder inside each console folder, under different names:
tools that write `gamelist.xml` use `images`, and Onion OS uses `Imgs`. A console folder can end up
with both, each holding some of its pictures. Romperoom itself never makes either folder (the art
it downloads lives in `.romperoom/media`), and it reads the pictures in both.

**Merge picture folders**, on **Fix up › Artwork** under the cover art cards, lists every console
folder that has both an `images` and an `Imgs` folder directly inside it, and what merging does:
the pictures in `Imgs` move into `images`, so each console has one picture folder. Nothing moves
until you press **Merge picture folders** and confirm. With no console folder that has both, it
says so in one line. Romperoom can only tell after a full scan, like the leftover artwork below it:
until then the section says "Romperoom can merge picture folders after a full scan." and points at
the scan button by its name ("Press Scan roms above.").

- **A picture only in `Imgs`** moves into `images` with its name.
- **A picture with the same name in both**: Romperoom asks the drive whether the name of the `Imgs`
  picture already reaches a file in `images`, so the drive's own rules decide what "the same name"
  means. On a Mac's own drive `Mario.png` and `mario.png` are one name, and so are `Straße.png`
  and `Strasse.png`; on an SD card formatted FAT32 or exFAT only letter case is ignored, so those
  two are two pictures and both end up in `images`. When the file in `images` is spelled otherwise,
  the list names it ("The one in images (mario.png) is larger…"). The drive's answer counts only
  when the two names are also the same once letter case is ignored: a drive that reaches a file by
  a short 8.3 name (`SUPERM~1.PNG` for `Super Mario Bros.png`, on Windows) never makes them one,
  and the picture stays in `Imgs` ("images has a file this name reaches, but Romperoom can't tell
  which one"). If both have exactly the same bytes, the `Imgs` copy is set aside. Otherwise the larger one stays
  in `images` and the smaller is set aside; when both are the same size, the one already in
  `images` stays. Set aside means the library's own set-aside folder (`.romperoom-quarantine`), as
  everywhere in Fix up: nothing is deleted.
- **Anything in `Imgs` that isn't a picture** (`Thumbs.db`, the `._` files a Mac leaves on other
  drives, other kinds of files), a folder inside `Imgs`, a link or a file that also has another
  name (a hard link) stays where it is, listed under **Left in Imgs**. Pictures are png, jpg,
  jpeg, webp and gif: the kinds the scanner reads as art.
- **`Imgs` itself** is removed only when the merge emptied it: Romperoom looks again after the
  moves, and keeps it if anything is still in it. An `Imgs` that was empty before is left alone.
  On a FAT32 or exFAT card a Mac writes a hidden `._` file beside every picture. Those often move
  with their pictures, and then `Imgs` goes; when some stay, the result says "Imgs still holds 3
  hidden files…" and keeps it. Romperoom never deletes those files here: remove them with
  [Fix up › Junk files](#junk-files), then remove the empty folder.
- Romperoom never touches your other art folders (`covers`, `boxart`, `screenshots`, `snaps`,
  `wheel`, `marquees`, `downloaded_media`), its own `.romperoom` folders or the set-aside folder.

A console is left alone, and says why, when its `Imgs` or `images` folder is a link to another
folder (Romperoom never follows links), when it has two folders spelled `images` (or `Imgs`) in
different letter case (only possible on a drive where case counts), or when a game list in it names
pictures inside `Imgs`: moving them would break those entries. Every game list a frontend keeps in a
console folder counts, in any letter case: `gamelist.xml` (EmulationStation, ES-DE, Batocera),
`miyoogamelist.xml` (Onion OS and other Miyoo firmware), any other `…gamelist….xml` or `.json`, and
Pegasus's `metadata.pegasus.txt` or `metadata.txt`. A list Romperoom can't read as plain text
(saved as UTF-16, binary, or over 16 MB) leaves the console alone too, since it might name `Imgs`.
Some lists aren't checked: ES-DE's own lists outside the library (in its `ES-DE/gamelists`
folder), a list in another console folder or one folder up that points into this console's `Imgs`,
and Onion OS's own files (its `…_cache6.db` caches and an emulator's `config.json` `imgpath`).
Check those yourself if they read `Imgs` from this library.

On Onion OS, `Imgs` is where the box art lives: after a merge, Onion reading this folder directly
shows no box art for those games. Putting games on a card with the Onion profile writes the art into
`Imgs` on the card again, wherever it sits in the library; if you copy the folder to the card
yourself, merge after copying, or skip the console. On a drive that ignores
letter case, `Images` and `images` are one folder: the pictures go into it as it is spelled.

The section shows a line per console (how many pictures move, how many have the same name in both,
and whether `Imgs` is removed or stays), then **Pictures with the same name in both** (which one
stays and why), **Left in Imgs** and **Pictures that move**. Each list shows its first 50 and says
how many there are in all.

| Light                                   | Dark                                          |
| --------------------------------------- | --------------------------------------------- |
| ![Merge picture folders][merge-look-lt] | ![Merge picture folders, dark][merge-look-dk] |
| ![What happens to each picture][merge-prev-lt] | ![What happens to each picture, dark][merge-prev-dk] |

Press **Merge picture folders**. The drawer asks plainly, for example "Move 214 pictures from Imgs
into images in 9 consoles? 3 with the same name are set aside: the larger stays. Nothing is deleted
and you can undo this." **Cancel** has the focus. The other button names the count, such as **Move
214 pictures**.

| Light                                       | Dark                                              |
| ------------------------------------------- | ------------------------------------------------- |
| ![The merge's question][merge-confirm-lt] | ![The merge's question, dark][merge-confirm-dk] |

Just before anything moves, Romperoom reads both folders again, and refuses if anything differs from
what you saw ("Your library changed since you looked"): **Refresh list** shows what is there now. A
hidden file a system writes meanwhile (`.DS_Store`, `Thumbs.db` or `THUMBS.DB`, `desktop.ini`, a
`._` file) is never moved, so it doesn't count as a change.
Each picture is checked once more right before it moves (its bytes, and that no link is on its way),
and nothing is ever overwritten. When a larger picture in `Imgs` replaces a smaller one, the smaller
is set aside first; if the larger then can't move (it changed, or another program held it open),
the smaller one is put straight back in `images`, and the result says so; if even that can't
happen (another file took its place, or the drive went away), the result says the smaller picture
is still set aside and **Undo all** puts it back. The two steps are always in one part of the run,
so stopping between parts never separates them. While it runs you see the progress, and **Cancel** stops between
files. The result says how many pictures moved into `images`, how many were set aside, and which
`Imgs` folders were removed, with **Undo all** to put every file back where it was (an `Imgs` folder
that was removed comes back with its pictures). Its run is in **History**, where **Undo** does the
same, and a run that stopped part way offers **Finish or undo…** like any other tidy.

| Light                                  | Dark                                         |
| -------------------------------------- | -------------------------------------------- |
| ![The merge's result][merge-res-lt] | ![The merge's result, dark][merge-res-dk] |

The library's game list for each picture follows it: your games show the same art straight away,
with no scan needed. **Undo** links each picture it puts back to its game at once, also when you
scanned in between (on a big library this takes a moment: about a tenth of a second for 50,000
pictures on a Mac). If that link can't be made, the files are still back and Undo says "The
pictures put back will link to their games at the next scan.", pointing at the scan button.
Undoing a set-aside game (Duplicates) doesn't link that game's pictures again: a scan does. A frontend that reads `Imgs` in this very folder (Onion OS reading the library
directly) no longer finds the pictures there; putting games on a card writes the art where the
card's frontend expects it, whatever folder it came from.

[merge-look-lt]: screenshots/merge-look-console-shelf-light.png
[merge-look-dk]: screenshots/merge-look-console-shelf-dark.png
[merge-prev-lt]: screenshots/merge-preview-console-shelf-light.png
[merge-prev-dk]: screenshots/merge-preview-console-shelf-dark.png
[merge-confirm-lt]: screenshots/merge-confirm-console-shelf-light.png
[merge-confirm-dk]: screenshots/merge-confirm-console-shelf-dark.png
[merge-res-lt]: screenshots/merge-result-console-shelf-light.png
[merge-res-dk]: screenshots/merge-result-console-shelf-dark.png
[fixup-duplicates-lt]: screenshots/fixup-duplicates-console-shelf-light.png
[fixup-duplicates-dk]: screenshots/fixup-duplicates-console-shelf-dark.png
[fixup-across-lt]: screenshots/fixup-across-console-shelf-light.png
[fixup-across-dk]: screenshots/fixup-across-console-shelf-dark.png
[fixup-confirm-lt]: screenshots/fixup-confirm-console-shelf-light.png
[fixup-history-lt]: screenshots/fixup-history-console-shelf-light.png
[fixup-artwork-lt]: screenshots/fixup-artwork-console-shelf-light.png
[fixup-artwork-dk]: screenshots/fixup-artwork-console-shelf-dark.png
[relink-rev-lt]: screenshots/relink-review-console-shelf-light.png
[relink-rev-dk]: screenshots/relink-review-console-shelf-dark.png
[relink-res-lt]: screenshots/relink-results-console-shelf-light.png
[relink-res-dk]: screenshots/relink-results-console-shelf-dark.png

## Junk files

Finder (on a Mac) and Windows leave small files beside your games: `._` files (AppleDouble: a
Mac's icons and labels on drives that can't hold them, such as a NAS share, FAT and exFAT), a
`.DS_Store` in folders Finder opened, and `Thumbs.db` where Windows made picture thumbnails. They
only hold metadata, never a game. Romperoom never counts them as games and never copies them from
your library to a card.

To clean them out of a library, open **Fix up › Junk files** (with several libraries, pick the
library under **Library to tidy** first) and press **Find junk files in** and the library's name.
It reads the library's folders and changes nothing; it shows how many files it has looked at
so far, and **Stop** ends it (nothing is removed). Then it shows how many it found and their
size, by kind, a list of them (100 at a time), and what it leaves:

- **Not removed:** a file with a junk file's name whose contents are not metadata (a game called
  `._something` stays, and so does a `._` file holding a classic Mac file's resource fork), a
  link, a file with a second name elsewhere (a hard link), or one that could not be read, each with
  the reason.
- **System folders left alone:** folders macOS, a NAS or Windows keep for themselves, such as
  `.Spotlight-V100`, `.Trashes` and `.fseventsd`. Romperoom never looks inside them or removes
  them. Romperoom's own folders (its art store, files set aside) are never touched either, nor is
  another drive mounted inside the library (listed as **Other drives inside, left alone**).

**Remove** (with the count and size) asks once: "They are only Finder and Windows metadata. This
can't be undone." Each file is checked again just before it goes; one that changed since the list
was made stays. While it removes, it shows how many of the files are done; **Stop** ends it after
the file in hand, and the result says how many were removed. The result says how many were removed, and lists any that were left and why.
History lists the removal as deleted forever. If the drive is read-only (a NAS share mounted
read-only, for example), nothing is removed and the result says so.

| Light                                                                  | Dark                                                                        |
| ---------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| ![Junk files found](screenshots/fixup-junk-console-shelf-light.png)     | ![Junk files found, dark](screenshots/fixup-junk-console-shelf-dark.png)     |
| ![Remove junk files?](screenshots/fixup-junk-confirm-console-shelf-light.png) | ![Remove junk files?, dark](screenshots/fixup-junk-confirm-console-shelf-dark.png) |
| ![Junk files removed](screenshots/fixup-junk-result-console-shelf-light.png) | ![Junk files removed, dark](screenshots/fixup-junk-result-console-shelf-dark.png) |

On an SD card, see steps 4 and 6 of [Put games on an SD card](#put-games-on-an-sd-card): a copy
from a Mac to a FAT or exFAT card removes the `._` files macOS made for it, and **Clean up this
card** on the Done screen finds the rest in the device's folders.

| Light                                                                     | Dark                                                                           |
| ------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| ![Clean up this card](screenshots/deploy-cleanup-console-shelf-light.png) | ![Clean up this card, dark](screenshots/deploy-cleanup-console-shelf-dark.png) |

## Standardise your library

**Standardise** (in **Fix up** › **Names**) names your console folders the way one device does,
and your identified games by their official names from your game databases. A library built up
over the years often has `GBA`, `gba` and `Game Boy Advance` side by side, and games named
`tetris.gb`: this makes them one scheme. You review every change first, nothing is deleted, and
the whole run can be undone. **Needs attention** shows the device you chose last.

1. Choose the device whose folder names you want: ES-DE, muOS, Onion (Miyoo Mini), Batocera,
   DSpico or R4xMP (`NDS ROMS`, `GBA ROMS`, `GBC ROMS` and `NES ROMS`).
   Romperoom remembers it for this library, and you can pick another later. **Review the
   changes** checks the library (**Stop** goes back to choosing) and shows the review.
2. In the review, tick what to change. Anything left unticked stays as it is. It has these parts:
   - **Folders:** each console folder whose name differs, such as `Game Boy Advance → gba`, with
     how many games it holds. When two folders are one console, the second merges into the one
     that takes the name, so it can only be ticked while that one is. A folder Romperoom can't
     place, one this device has no folder for, one whose name is taken by something else, and a
     folder of a device's saves are listed with the reason and left alone.
   - **Games**, under the name of their console: each identified game whose file name differs
     from its official name, with what it brings along: its pictures, its saves (in
     `.romperoom/saves` and any device's save folder inside your library) and the game list
     entries that name it. Two copies of one game are each listed, by where each one is. A game
     of several files (a disc's `.cue` and `.bin` files, a playlist) is renamed whole, its `.cue`
     or `.m3u` rewritten to the new names, or not at all. A game whose picture or save can't come
     along (its new name is taken) starts unticked and says which file stays. A game Romperoom
     can't rename says why. Games that aren't identified are never touched, and a game whose file
     matches more than one game waits until you choose under **Check name matches**.
   - **Files with the same name:** when a merge meets a file already there, it stays in its own
     folder. An identical copy can be set aside instead (tick it), and a different one always
     stays where it is.
   - **Game lists** that will change, and any left alone with the reason.
3. **Standardise.** Progress shows each change; **Stop** stops between games, keeping what is done.
   The results say what was renamed and anything left as it was, with **Undo this run** and a line
   pointing at Fix up's scan button, which checks the renamed files on your drive. They stay until
   you press **Done**, even if you visit another screen and come back. If the library was busy with
   another job, nothing changes, the result names the job, and **Back to the review** returns to it.

| Light                                              | Dark                                                     |
| -------------------------------------------------- | -------------------------------------------------------- |
| ![The standardise review][standardise-review-lt]   | ![The standardise review, dark][standardise-review-dk]   |
| ![The standardise results][standardise-results-lt] | ![The standardise results, dark][standardise-results-dk] |

Game lists (`gamelist.xml` in a console folder, which Batocera and ES-DE read) change only where
they name a renamed game. Before a game list, cue sheet or playlist changes, its original is
moved to `.romperoom/lists-backup` in your library, and it stays there. A game list Romperoom
can't read with certainty is left as it is and named in the review, and so is the game list of a
folder that merges into another (it stays in that folder). A frontend's own database isn't
changed, so a renamed game may need a rescan there. Saves on a card keep their old names.

**Recent standardise runs**, here and under **History**, offer **Undo** while a run can still be
undone. Undo puts back only what is unchanged since; anything else stays where it is and is listed.
A run stopped part way, cut short by a crash or a disconnected drive, or one that couldn't put a
failed item back, waits to be finished or undone: the results, **Needs attention** and **History**
offer **Finish or undo…**. While it waits, Romperoom won't start another run of that library ("An
earlier run of this library was interrupted") or remove the library ("This library has an
interrupted standardise run"): finish it or undo it first, then try again.

If your library changed since the review, or the review is too old, Romperoom says so and
**Review again** checks the library afresh; nothing was changed.

[standardise-review-lt]: screenshots/standardise-review-console-shelf-light.png
[standardise-review-dk]: screenshots/standardise-review-console-shelf-dark.png
[standardise-results-lt]: screenshots/standardise-results-console-shelf-light.png
[standardise-results-dk]: screenshots/standardise-results-console-shelf-dark.png

## Keyboard

Everything works from the keyboard. Tab moves between controls, and a visible ring shows
where you are.

| Where                    | Key                                  | Does                                          |
| ------------------------ | ------------------------------------ | --------------------------------------------- |
| Anywhere                 | Tab, Shift+Tab                       | Next or previous control                      |
| Top of any screen        | Tab, then Enter on "Skip to content" | Skips past the header to the screen's content |
| Your consoles (Home)     | Arrow keys, Home, End                | Move between console cards                    |
| Your consoles (Home)     | Type a name                          | Go to the console whose name starts so        |
| Your consoles (Home)     | Enter                                | Open that console's games                     |
| Games (list or grid)     | Arrow keys                           | Move between games                            |
| Games (list or grid)     | Home, End                            | First game, last game                         |
| Games (list or grid)     | Page Up, Page Down                   | Up or down a screenful                        |
| Games (list or grid)     | A letter or digit                    | The first game starting so (title order)      |
| Games (list or grid)     | Enter or Space                       | Open the game's details                       |
| Jump to letter           | Left, Right, Home, End               | Move along the letters                        |
| Fix up and Settings tabs | Left, Right, Home, End               | Move between tabs                             |
| Fix up and Settings tabs | Enter or Space                       | Show that tab                                 |
| Details or Settings      | Escape                               | Close the panel                               |

## Gamepad

Connect a controller and Romperoom shows its buttons at the bottom of the window. Xbox-style
names are used. On other pads the buttons in the same positions do the same thing.

| Button              | Does                                                                                                                                              |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| D-pad or left stick | Move                                                                                                                                              |
| A                   | Open or press what is selected                                                                                                                    |
| B                   | Back: closes the open panel, else goes up one level (a console's games, a Fix up task or an SD card choice to its screen, another screen to Home) |
| LB, RB              | Previous or next console on a games page, task in Fix up                                                                                          |

Holding a direction repeats it.

## Questions

**Does Romperoom change, move or delete my games?**
Only when you ask it to in Fix up, after a preview. Duplicates and leftover artwork are only set
aside inside your library, where you can put them back; deleting them forever needs you to type
`DELETE FOREVER`.
[Standardise](#standardise-your-library) renames folders and games, and changes the game
lists, cue sheets and playlists that name them (keeping each original), only after you approve its
review, and Undo puts them back. Scanning and copying to a card only read your library. Confirming
that games were removed only changes Romperoom's own list. On a card, it only replaces or moves
files it put there itself, except that [Sync a card](#sync-a-card) replaces a save you approved
after copying the card's own to the library's `.romperoom/saves-backup`. In the library, Sync a card
adds games but never changes or deletes one, moves a save it replaces to the same backup folder
first, and its Undo removes only what it added.
Cover art never changes a game file: it only adds pictures to the library's own
`.romperoom/media` folder when you ask, and removes only the ones it added.

**Does it go online?**
Only if you ask. Under Fix up › Game databases, Download for me and Check for updates contact
GitHub, and so do Get cover art and its Download in Fix up › Artwork; Network activity lists every
request. Everything else, importing art from an SD card included, works offline. See
[security.md](security.md).

**Where does it keep its data?**
In a folder of its own, not in your ROM folder (see
[configuration.md](configuration.md#data-folder)). To start over, quit Romperoom and delete
that folder. Your games are not touched.

**Why does a game say "Unidentified"?**
You have not imported a game database for its console, or have not pressed **Identify games**
yet, or the file does not match what is in the databases you imported. See
[What the labels mean](#what-the-labels-mean) for every reason and what to do about it.

**Why are some games missing?**
Their folder may not match a console: look under **Folders without a console** in **Fix up** ›
**Names**. A
file that couldn't be read is listed there too. Two revisions of one game show as one until it is
identified, and a game stored as a whole folder (some DOS and PC games) shows as several games
until a game database identifies its files as one game. See [troubleshooting](troubleshooting.md).

**Can it put games on my handheld's SD card?**
Yes: see [Put games on an SD card](#put-games-on-an-sd-card). If a card doesn't show up, or
copying is slow, see [troubleshooting](troubleshooting.md#a-card-doesnt-show-up).

**My drive is slow. Can I stop a scan?**
Yes: **Cancel scan**. The next scan continues close to where it stopped.

**Is there a Windows or Linux version?**
Yes: Windows 10 or 11 (64-bit), and Linux x64 (a `.deb` for Ubuntu and Debian, and an AppImage).
Both are new: they pass automated tests on a Windows PC and in a Linux container, but have had
little use by people yet.
