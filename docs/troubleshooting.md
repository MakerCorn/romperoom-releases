# Troubleshooting

Problems you might meet, and what to do. Only Tidy up changes your games: it sets files aside,
where you can put them back, and Standardise renames folders and games, which Undo puts
back; cover art only adds pictures to Romperoom's own `.romperoom/media` folder, when you ask;
Sync a card only adds the games and saves you approve, and backs up any save it replaces. None of
these problems can harm your collection.

## Contents

- [Romperoom won't start from a terminal](#romperoom-wont-start-from-a-terminal)
- [Can't reach your game folder](#cant-reach-your-game-folder)
- [We didn't find any games in that folder](#we-didnt-find-any-games-in-that-folder)
- [Some games are missing](#some-games-are-missing)
- [Some games stay unidentified](#some-games-stay-unidentified)
- [A game database won't download](#a-game-database-wont-download)
- [A console says "not available at this version"](#a-console-says-not-available-at-this-version)
- [Cover art won't download](#cover-art-wont-download)
- [A picture didn't appear](#a-picture-didnt-appear)
- [The card's art wasn't imported](#the-cards-art-wasnt-imported)
- [ScreenScraper can't be used yet](#screenscraper-cant-be-used-yet)
- [A ScreenScraper lookup stopped or found nothing](#a-screenscraper-lookup-stopped-or-found-nothing)
- [Some game folders look empty or gone](#some-game-folders-look-empty-or-gone)
- [Files that couldn't be read](#files-that-couldnt-be-read)
- [The scan is slow](#the-scan-is-slow)
- [The scan didn't finish](#the-scan-didnt-finish)
- [Starting over](#starting-over)
- [A folder can't be added as a library](#a-folder-cant-be-added-as-a-library)
- [A library can't be removed](#a-library-cant-be-removed)
- [A card doesn't show up](#a-card-doesnt-show-up)
- [A card is greyed out](#a-card-is-greyed-out)
- [Romperoom asks before exporting to a folder](#romperoom-asks-before-exporting-to-a-folder)
- [The games don't fit on the card](#the-games-dont-fit-on-the-card)
- [The card got a game from the other library](#the-card-got-a-game-from-the-other-library)
- [A detail is left out on the Done screen](#a-detail-is-left-out-on-the-done-screen)
- [A picture on the card wasn't made smaller](#a-picture-on-the-card-wasnt-made-smaller)
- [A BIOS file wasn't recognised](#a-bios-file-wasnt-recognised)
- [A saved package is missing or can't be saved](#a-saved-package-is-missing-or-cant-be-saved)
- [Copying to the card is slow](#copying-to-the-card-is-slow)
- [A card can't be synced](#a-card-cant-be-synced)
- [A game from the card wasn't offered](#a-game-from-the-card-wasnt-offered)
- [A save wasn't synced](#a-save-wasnt-synced)
- [Undo this sync left something](#undo-this-sync-left-something)
- [Tidy up says Romperoom is busy](#tidy-up-says-romperoom-is-busy)
- [Your library folder isn't available](#your-library-folder-isnt-available)
- [A library says it can't reach its folder, but it scans](#a-library-says-it-cant-reach-its-folder-but-it-scans)
- [Your library changed since you looked](#your-library-changed-since-you-looked)
- [A library says Not checked under Across libraries](#a-library-says-not-checked-under-across-libraries)
- [Across libraries leaves out a copy, or shows one twice](#across-libraries-leaves-out-a-copy-or-shows-one-twice)
- [A copy set aside from Across libraries wasn't deleted](#a-copy-set-aside-from-across-libraries-wasnt-deleted)
- [A file couldn't be put back](#a-file-couldnt-be-put-back)
- [Romperoom was interrupted while tidying](#romperoom-was-interrupted-while-tidying)
- [Standardise left a folder or game as it was](#standardise-left-a-folder-or-game-as-it-was)
- [A game list wasn't changed](#a-game-list-wasnt-changed)
- [A frontend still shows the old names](#a-frontend-still-shows-the-old-names)
- [Undo this run left something](#undo-this-run-left-something)
- [A picture wasn't offered for re-linking](#a-picture-wasnt-offered-for-re-linking)
- [A game list entry wasn't re-pointed](#a-game-list-entry-wasnt-re-pointed)
- [A frontend can't find a picture after a re-link was undone](#a-frontend-cant-find-a-picture-after-a-re-link-was-undone)
- [Windows and Linux](#windows-and-linux)
- [For developers](#for-developers)

## Romperoom won't start from a terminal

Some editors and tools set `ELECTRON_RUN_AS_NODE`. With it set, Electron runs as plain Node
and the app fails to start. Unset it for the run:

```sh
env -u ELECTRON_RUN_AS_NODE npm run dev -w @romperoom/desktop
```

## Can't reach your game folder

"Can't reach your game folder — Is your NAS turned on and connected?" means the folder could
not be opened: the drive is unplugged, the network share is not mounted, or the network
dropped.

1. Open the folder in Finder. If Finder can't open it, reconnect the drive or share first.
2. Back in Romperoom, press **Scan again**.

"Romperoom isn't allowed to open that folder" means the share or the folder's permissions
refuse your account. Fix the sharing permissions, then try again.

## We didn't find any games in that folder

Romperoom expects the folder that **holds** your console folders, such as a folder containing
`snes`, `gba` and `megadrive`. Choosing a single console's folder, or a folder one level too
high, finds nothing. Press **Choose a different folder** and pick the right one. The list of
folder names it knows is in [systems.md](systems.md).

## Some games are missing

- **Their folder needs a console.** Health lists it under **Folders without a console**. Choose
  a console for it in setup, then press **Scan again**.
- **You ignored the folder.** Health lists it under **Ignored folders**. Press **Include
  again**, then **Scan again**.
- **The file couldn't be read.** See [below](#files-that-couldnt-be-read).
- **The file's extension isn't one that console uses.** Such files are skipped.
- **Two revisions of one game** (such as `(Rev 1)` and the original) show as one game until it
  is identified (see [Identify your games](user-guide.md#identify-your-games)).
- **A game stored as a folder of files** (some PC and DOS games) shows as several games (until
  a game database identifies its files as one game), or as none.

## Some games stay unidentified

The **Identify your games** card on Health says why, next to each reason:

| Reason                        | What to do                                                       |
| ----------------------------- | ---------------------------------------------------------------- |
| Not read yet                  | Scan again, then identify.                                       |
| In another console's database | Move it to that console's folder, then identify again.           |
| No database for this console  | Add a game database for this console, then identify.             |
| Not in your databases         | May be a hack, a translation or a bad copy; it still plays fine. |

If you only just imported a database, press **Identify games** on Health — importing one does not
identify your library by itself (a scan that finishes after a database exists does, from then on).
A file a scan hasn't read yet shows as **Not checked yet**, never as a zero: identify it again
once a later scan has read it. Files counted as **not identified yet** were read, but no identify
has looked at them since (after removing a database, say): press **Identify games**.

## A game database won't download

Download for me and Check for updates (Settings › Game databases), and Get cover art on Health, are
the only things that take Romperoom online to GitHub (a ScreenScraper lookup goes only to
ScreenScraper). Each request is listed under
**Network activity**, with its outcome. A file that fails leaves nothing behind: no partial file and
no change to your databases. The message says what happened:

| The message starts with                     | What to do                                    |
| ------------------------------------------- | --------------------------------------------- |
| Romperoom couldn't reach GitHub             | You're offline, or GitHub can't be reached.   |
| GitHub took too long to answer              | Try again later.                              |
| Romperoom couldn't make a secure connection | See "Proxies and filtered networks" below.    |
| GitHub has had too many requests            | Wait until the time it gives, then retry.     |
| This file isn't at that version any more    | Press **Check for updates**, then review.     |
| The file didn't match what GitHub listed    | Nothing was kept. Try later, or use the site. |
| Romperoom couldn't find libretro-database's | Try later, or use the official site.          |
| GitHub's answer was bigger than expected    | Try again later.                              |

**Rate limits.** Downloading a file never counts against GitHub's limit for unsigned-in
requests (60 an hour per address). Check for updates does: it uses 1 request when nothing has
changed and 4 when there is a newer version. Others on the same network (an office, a school)
share that limit.

**Proxies and filtered networks.** Download for me connects to GitHub directly. It does not use
a system proxy, a PAC file or a proxy named in the environment, and it trusts only the
certificate authorities built into Romperoom, not ones your organisation installed. A network
that requires a proxy, or that inspects secure connections with its own certificate, cannot use
it. Use **Get a database from the official site** instead: your browser does the downloading.

## A console says "not available at this version"

The version of libretro-database Romperoom is using has no file for that console (it may have
been renamed or removed upstream). Press **Check for updates**; if the console still says so,
the next Romperoom release will update the mapping. Meanwhile, get its database from the
official site.

## Cover art won't download

**Get cover art** (Health › Games without cover art) and its review's **Download** take cover
art online only to GitHub; **Look up on ScreenScraper** is the other way, and goes only to
ScreenScraper (see
[A ScreenScraper lookup stopped or found nothing](#a-screenscraper-lookup-stopped-or-found-nothing)).
Each request is listed under **Network activity**. A picture that fails is never kept half-written, and one failure never undoes the
pictures already saved. The review or the results say what happened:

| The message starts with                      | What to do                                    |
| -------------------------------------------- | --------------------------------------------- |
| Romperoom couldn't reach GitHub              | You're offline, or GitHub can't be reached.   |
| GitHub is limiting requests right now        | Wait until the time it gives, then retry.     |
| This picture changed on GitHub               | Review again: the collection was just edited. |
| Not enough space in the library              | Free space on the library's drive.            |
| Romperoom can't write to the library folder  | Check that the folder or share allows writes. |

**A console says "not available".** Either Romperoom has no libretro system for that console
(it can't know which picture collection is the right one, so it never guesses), or
libretro-thumbnails has no repository for it. Nothing is downloaded for that console; the next
Romperoom release may add it. Consoles GitHub's limit didn't leave room for also show "not
available", with the time to try again.

**A console's row ends "· 3 not available".** That many pictures (one game and one kind each)
are missing and won't come from this download: the collection has no picture with that game's
name, or the game can't take one (see [A picture didn't appear](#a-picture-didnt-appear)). The
results count them under "not available" too, but not in "Saved 5 of 5 pictures", which counts
only the pictures Romperoom tried.

**Rate limits.** Each console's picture list costs one GitHub request (a very large one, a few
more), out of the 60 an hour GitHub allows per address without signing in, shared with Check for
updates and with others on the same network. A second review within a day reuses the lists and
asks nothing. Downloading the pictures themselves never counts.

**Proxies and filtered networks.** As for game databases, cover art connects to GitHub directly
and trusts only the certificate authorities built into Romperoom (see
[A game database won't download](#a-game-database-wont-download)).

## A picture didn't appear

- **The ROM was renamed after its picture was saved.** A picture is named after the ROM file
  (without its extension), and a scan links pictures to games by that name. Rename the picture
  in `.romperoom/media/<console>/<kind>/` to match, or remove it and get cover art again.
- **Two games share a file name.** When two games of one console in one library have the same
  file name in different folders, Romperoom can't tell which one a picture belongs to, so
  neither is offered a picture. Rename one of them.
- **The file name starts with a dot, or is very long.** A picture named after it would be hidden
  (a scan skips names starting with a dot), or longer than file systems allow, so the game is
  shown as not available. Rename the ROM.
- **There is no picture for that game.** Games without a DAT match are matched by their exact
  file name, never a similar one. A file named differently from the collection's No-Intro name
  gets nothing; identifying the game first (Settings › Game databases) matches it by its DAT name.

## The card's art wasn't imported

- **The device's folders.** Romperoom reads the art folders the chosen device keeps (for
  example ES-DE's `ES-DE/downloaded_media/<console>/covers`). Choose the device the card was
  made for; pictures in other folders aren't seen. A device that keeps no box art, screenshots
  or title screens has nothing to import.
- **Unmatched names.** A picture is matched to a game first by the ROM's SHA-1, when the card
  was made by Romperoom (its manifest records each ROM's), then by the game's title, then by the
  ROM's file name. The review counts the pictures it couldn't match for each console.
- **The game already has that kind.** Import never replaces a picture you have, from any source.

## ScreenScraper can't be used yet

**Look up on ScreenScraper** needs Romperoom's own registration with ScreenScraper, and this copy
of Romperoom doesn't have one yet: "This copy of Romperoom can't use ScreenScraper yet." There is
nothing to fix on your side; you can still save your account in **Settings** › **ScreenScraper**
for later. "Save your ScreenScraper account in Settings › ScreenScraper first." asks for your own
account (free at screenscraper.fr). On a computer without a system keychain (some Linux desktops)
Romperoom won't save a password, so ScreenScraper can't be used there.

## A ScreenScraper lookup stopped or found nothing

- **The daily limit.** "You've used today's ScreenScraper lookups for your account. Try again
  tomorrow." Pictures and descriptions saved before it stopped are kept; the next lookup asks only
  about what is still missing.
- **Your account.** "ScreenScraper didn't accept your account. Check it in Settings ›
  ScreenScraper." Forget the account there and save it again. "Save your ScreenScraper account in
  Settings › ScreenScraper first." at the start of a lookup means this computer's keychain can no
  longer read the saved account (it was reset, or the data folder came from another computer):
  nothing was sent; forget it and save it again.
- **ScreenScraper is busy or closed.** "ScreenScraper is busy with your account's other lookups"
  or "ScreenScraper isn't taking lookups right now". Try again later. Pictures and descriptions
  saved before it stopped are kept.
- **Romperoom's app details.** "ScreenScraper didn't accept Romperoom's app details." means
  ScreenScraper refused Romperoom's own registration, and "ScreenScraper no longer accepts this
  version of Romperoom. Update Romperoom." means this version is too old for it. Neither is
  about your account; what was saved before it stopped is kept.
- **A game skipped.** "Romperoom couldn't ask ScreenScraper about this game, so it was skipped."
  The game's file name or size can't be sent as it is (for example a name with a backslash); the
  lookup goes on with the next game.
- **Nothing for a game.** ScreenScraper fills a game only when it matched the game file's
  checksums. A game it found only by name gets nothing (the results count those), and so does a
  game it doesn't know. Get cover art from libretro-thumbnails, or identify the game first.

## Some game folders look empty or gone

Romperoom never takes games off its list on its own when whole folders seem to vanish, because a
drive that is only partly connected looks the same. If the drive was disconnected, reconnect it
and press **Scan again**. If you removed those games on purpose, press **It's OK — they were
removed**. Only Romperoom's list changes. The same goes for **Some cover art folders look empty
or gone**.

## Files that couldn't be read

Health lists each file with a reason:

| Reason                      | What to do                                                  |
| --------------------------- | ----------------------------------------------------------- |
| This zip file looks damaged | Get a fresh copy of the game. Romperoom will not repair it. |
| Password-protected archive  | Unzip it yourself, or replace it with an unprotected copy.  |
| The drive stopped answering | Check the drive or network, then press **Try again**.       |

**Try again** reads every listed file once more. It does nothing while a scan is running.
Without it, a file that failed many times in a row is only retried on some scans.

## The scan is slow

The first scan reads every file once to fingerprint it, so it has to read the whole collection
off the drive. On a network share that is limited by the network. Measured on one NAS library
of 22,331 games (419 GB): finding every game took under 5 minutes, and reading the files ran at
about 17 MB/s, so fingerprinting all of them would take about 6.6 hours on that connection.

- You do not have to wait: once every game is found, **Go to my library** opens the library
  while checking carries on. Duplicates among files not checked yet show up later.
- A wired connection to the NAS helps most: 17 MB/s is what Wi-Fi or a 100 Mbit link gives;
  wired gigabit is usually around 110 MB/s.
- You can press **Cancel scan** and continue later. Files already read are not read again.
- Later scans only read new or changed files.

## The scan didn't finish

"The last scan didn't finish" appears if Romperoom closed during a scan. Press **Scan again**.
It continues from what it already found.

## Starting over

Romperoom keeps its list in a data folder of its own, never in your ROM folder. To start from
scratch (for example, to choose another library):

1. Quit Romperoom.
2. Delete the data folder. On macOS that is `~/Library/Application Support/Romperoom`, on
   Windows `%APPDATA%\Romperoom` (see [configuration.md](configuration.md#data-folder)).
3. Open Romperoom again. Your theme choice and the card wizard's saved packages are reset too.

Files you set aside in Tidy up stay in the `.romperoom-quarantine` folder inside your library,
but a fresh start no longer knows about them. Put back what you want first, or move them back
by hand afterwards: each one sits under a dated folder, at its old path.

To use another folder instead, add it in **Settings** › **Libraries** and remove the old one
(see [Your libraries](user-guide.md#your-libraries)): nothing in either folder changes.

If your library is gone after updating from a build before 0.1.0: those builds kept their
catalogue in a folder named `@romperoom/desktop`, and nothing moves it. Quit Romperoom and move
it to the folder above, or add your library again and re-scan.

## A folder can't be added as a library

**Add a library…** in **Settings** › **Libraries** refuses a folder and says why:

- "That folder is already one of your libraries": it's in the list (perhaps under another name,
  or reached through a shortcut).
- "That folder is inside one of your libraries" or "That folder holds one of your libraries":
  one library inside another would count its games twice. Choose a folder that isn't inside
  another library, or remove the library inside it first.
- "Romperoom can't use a whole drive as a library": choose the folder on the drive that holds
  your console folders.
- "That isn't a folder": choose the folder that holds your console folders.
- "Can't reach your game folder" or "Romperoom isn't allowed to open that folder": check the
  drive is connected and the folder opens in Finder.

Nothing is added when it refuses.

Romperoom looks at each of your libraries' folders when you add one. If a network drive that
holds one of them has stopped answering, Romperoom stops responding until the drive answers or
the system gives up on it. Reconnect the drive.

## A library can't be removed

**Remove** in **Settings** › **Libraries** waits for whatever is using the library, and says so:

- "A scan is running": wait for the scan to finish (or cancel it), then try again.
- "Romperoom can't remove roms right now": Tidy up, Standardise, re-link, cover art, identifying
  games, a game database import, or a card is busy with that library. Wait for it to finish.
- "Something Romperoom was doing in roms stopped before it finished", or "This library has an
  interrupted standardise run": open Tidy up, press **Finish or undo…** on the Overview, settle
  the run, then remove the library.

Nothing is forgotten when it refuses.

Once a library is removed, Romperoom forgets its Tidy up history: History and Set aside no
longer list its runs, and Undo, put back and Delete forever no longer offer what it set aside.
Those files stay in the `.romperoom-quarantine` folder inside that library's folder, each under a
dated folder at its old path; move back what you want by hand. Adding the folder again later
starts afresh and doesn't bring the history back.

If you remove a library while **Identify games** is working through all your libraries, it may
show an error for the library you removed when it gets to it. Nothing is lost, and your other
libraries are identified as usual.

## A card doesn't show up

On **SD card**, the **Where** step lists the drives Romperoom could use.

- Check that the card is in, and that your computer shows it (in Finder, or File Explorer on
  Windows). A card in a reader that isn't mounted can't be listed. Then press **Refresh**.
- A card that needs formatting doesn't show up: Romperoom never formats cards. Format it with
  your device, or with your computer's disk tool, then press **Refresh**.
- No card at hand? **Export to a folder** copies the same layout into a folder; move that
  folder's contents to the card's top level yourself.

## A card is greyed out

A greyed-out card says why under its name:

- **Locked (read-only):** slide the lock switch on the side of the SD card (or its adapter) up,
  then put it back in and press **Refresh**. On Linux the same words mean the card's file system
  is mounted read-only, even with the switch off (your computer may do that after it finds an
  error on the card): take it out, put it back in, and press **Refresh**. If it stays locked,
  check the card on another computer or in your device.
- **Your computer's own system drive**, or **a part of your computer Romperoom never writes to:**
  Romperoom never writes there, on purpose. Choose the SD card.
- **A network drive:** copy to a card plugged into this computer instead.
- **Your game library or BIOS files are on it,** or **it's inside your game library:** Romperoom
  never copies games into your library. Use a separate card.
- A disk inside your computer isn't greyed out, but you have to type its name to use it. That
  check is there so a wrong click can't fill the wrong disk.

## Romperoom asks before exporting to a folder

When the folder you chose for **Export to a folder** is on a network drive, the **Where** step
asks before Romperoom copies there: tick **Copy to** (the folder's name) **anyway**, then
**Next**. Copying over a network is slower, and if the connection drops the copy stops part-way;
copy again to finish. It asks the same when it can't tell (the folder didn't answer within 5
seconds, or your computer's list of drives couldn't be read).

- Choosing a folder again, even the same one, asks again.
- If you connected a network drive over the folder after choosing it, **Preview the changes** or
  **Write** stops with "Romperoom didn't copy anything" and **Details** says the folder "is on a
  network drive" (or, if the drive arrived while Romperoom was checking, that the folder "changed
  while it was being checked"). Choose the folder again: the Where step then asks.
- How it tells: on a Mac, a drive your Mac doesn't call local (a shared folder from another
  computer or a NAS); on Windows, a network path (`\\server\share`), and usually a mapped network
  drive too, which Windows normally leads to its network path (not yet checked on a Windows
  computer); on Linux, an NFS, SMB/CIFS, SSHFS, 9p, AFS or Ceph mount. Other network
  file systems on Linux (an `rclone` or `davfs` mount, for example) are not recognised.

## The games don't fit on the card

The **Where** step shows the space the games, artwork and the card's own format take. If it doesn't
fit, it says by how much and offers what to leave out: the artwork, the other versions of each game,
or a big console. **Suggest games to leave out** lists the games it would leave out, biggest first;
keep any you want, then **Leave these games out** (see [Make it fit](user-guide.md#make-it-fit)).
You can also go **Back** and untick consoles. Copying stays off until it fits. The card's format
loses some space to every file, so a card full of small games fits fewer than their sizes suggest;
Romperoom counts that.

If a card fills up during a copy (something else wrote to it), the report says **The card
filled up**. Free some space, or leave something out, and copy again: what was already copied
stays.

## The card got a game from the other library

When a game file is in two of your libraries and the copies are different (two dumps of one
game with the same name), only one can go to the card. The **Check** step lists them under
**Copies that differ**, each with the library its copy comes from. Romperoom keeps the copy
that matches your game database (Settings › Game databases), else the one in the shorter folder
name (`GBA` before `Game Boy Advance`), else the one it found first, so without a game database
a new scan can change which one goes. It never asks per game.

- To send the copy that matches a game database, add one for that console and identify your
  games (see [Identify your games](user-guide.md#identify-your-games)), then check again: the
  copy that matches it goes.
- To send the other copy without a game database, take the one you don't want out of its
  library folder, scan that library again, and check again.
- A game made of several files (a CD game with a cue sheet and tracks, or discs and a playlist)
  whose files differ between libraries is still left out, under **Left out**: "Another game would
  use the same file name". Keep one complete copy of it, then scan again.
- Two different copies inside **one** library (in two of its folders, such as `GBA` and
  `Game Boy Advance`) are not chosen between either: the game is left out the same way. Keep one
  of them in that library, then scan again.

## A detail is left out on the Done screen

**Details** on the Done screen (and under a refusal on **Where** or **Check**) name your card,
your libraries, BIOS folders and Romperoom's data folder by their names, never by their full
path. A detail that would name any other folder on your computer is replaced by "A detail is left
out here because it names a folder on this computer." The short reason above the details still
says what went wrong.

## A saved package is missing or can't be saved

[Saved packages](user-guide.md#saved-packages) are kept on this computer, in Romperoom's data
folder, not in your library or on a card.

- **"Some saved packages couldn't be read, so they aren't listed."** The saved list was damaged
  (or edited by hand). The packages Romperoom could read are listed; the next package you save,
  replace, rename or delete writes a fresh list without the damaged ones.
- **"Romperoom can't open saved packages on this computer just now."** Romperoom couldn't reach
  its own storage. Quit and open Romperoom again. Nothing is written while the list can't be
  read, so trying again never loses a package.
- **"There's no more room on this computer for saved packages."** Delete a package you no longer
  use, or remove the packages for devices Romperoom no longer knows, from the note on the same
  card. Then save again.
- **Save as a package is greyed out.** That device has 20 packages, the most Romperoom keeps:
  delete one first.
- **A device's packages aren't listed.** Packages belong to the device they were saved for: pick
  that device on the first step. If a Romperoom update drops a device, its packages stay saved
  but hidden until it comes back; a note on the card counts them and can remove them.
- **The packages are gone.** They stay on the computer they were saved on, and are forgotten
  when Romperoom's data folder is deleted ([Starting over](#starting-over)).

## A picture on the card wasn't made smaller

Romperoom makes box art smaller only for a device that shows it at a fixed width (Onion), and
only PNG pictures: "N pictures copied at full size: Romperoom makes only PNG pictures smaller."
A picture already no wider than the device shows it is copied as it is, and so is one whose
smaller copy would take more bytes (some simple pictures compress better at full size). "N
pictures could not be read to make them smaller and were copied as they are." means the picture
is damaged or not really a PNG (or, rarely, the computer ran short of memory while making it);
open it in an image viewer. A PNG over 2048 × 2048 pixels, or over 16 MiB, is larger than Romperoom
makes smaller and is copied at full size, and the copy says so. Either way the card records the
picture as handled, so the next copy leaves it alone until the picture changes. Your own pictures
are never changed.

## A BIOS file wasn't recognised

The check step says how Romperoom treated the BIOS folder:

- "BIOS files are copied by name. Download the BIOS checksums under Settings › Game databases to
  check them." Without libretro's list Romperoom copies BIOS files under their own names.
- "N BIOS files are not in libretro's list; copied by their own names." The file's contents
  match nothing in the list (another region or revision, or a file that is not a BIOS).
- "N BIOS files have a name libretro lists but other contents; copied as they are." A file with
  the right name but other bytes: an emulator may refuse it.
- "No BIOS file for Sony PlayStation was found; some of its games may need one." libretro's list
  names BIOS files for that console and none was found directly in the BIOS folder (files in its
  subfolders aren't read). Not every game needs one.

## Copying to the card is slow

- Romperoom checks every file after copying it, so a copy of many small files is slower than a
  plain drag and drop. A second copy only writes what changed, so it is quick.
- Cheap or old cards, and USB 2 card readers, are slow at small files. A faster card or a
  reader in a USB 3 port helps most.
- **Windows:** antivirus software (Windows Security included) scans every new file on the card,
  which slows a copy of many files. That is normal; let it finish. Nobody has used Romperoom on
  Windows by hand yet, so tell us how it went.
- You can **Cancel the copy** at any time. The next copy picks up from there.

## A card can't be synced

Sync a card lists only cards Romperoom could also copy games to: not the disk your computer runs
from, not a read-only or locked card, not a network drive, and not a card that holds (or is
inside) your library. Unlock the card or plug it in directly, then press **Look again**. If the
card was synced from another window or taken out since the review, read it again.

## A game from the card wasn't offered

The review offers only games the library lacks. A game is already there when a library file of
the same console has the same file name (letter case and accents aside), or the same contents
under another name; the review counts those. A game is shown greyed out when a different file
already has its name in the library folder it would go to: rename one of them yourself. Files
the device's profile does not list for that console (`.txt`, pictures, folders inside a console
folder) are not games here, and links, special files, names longer than 255 bytes and files over
4 GiB are left alone.

## A save wasn't synced

A save belongs to the game whose name it carries (`Tetris (World).srm` for
`Tetris (World).gb`). Save states are never synced. On muOS and Onion, saves sit in one folder
per emulator core: a save whose name matches games of two consoles on the card is left alone,
and a library save goes to a console's core folder on the card only when the card uses just one
for that console (or none yet). A save that changed on both sides waits for you to pick which to
keep; **Decide later** leaves both as they are. Devices whose save folders Romperoom doesn't know
(ES-DE) have only their games imported ([profiles.md](profiles.md#saves)).

## Undo this sync left something

Undo puts back only what still holds exactly what the sync wrote: a game or save that changed
since is left, and listed. Saves written to the card are never undone; their earlier copies are
in the library's `.romperoom/saves-backup` folder, named `<game>.<date and time>.<ext>`. A sync
cut short by a crash or by the library going away waits in Tidy up's Recovery, like a tidy, to
be finished or rolled back.

## Tidy up says Romperoom is busy

"Romperoom is busy with a scan/copy. Try again when it finishes." means a scan, a copy to a
card, another tidy or another job (Standardise, re-link, identifying games, cover art or a card
sync) is using that library. Nothing was changed. Wait for it to finish (or
cancel it), then press the button again.

## Your library folder isn't available

"Your library folder isn't available right now — is the drive connected?" means the folder at
the library's path is missing, or is not the library that was tidied (a drive that mounted
empty, or another drive at the same path). Nothing was changed. Reconnect the drive, check it
opens in Finder, then try again.

A run from **Across libraries** needs both libraries: the one its copies were set aside in, and
the one that kept the other copy. Setting aside and **Finish** say the library that holds the kept
copies is not reachable until it is connected; **Undo** works without it.

## A library says it can't reach its folder, but it scans

Settings › Libraries says "Can't reach its folder" when the library's folder was moved and a
symbolic link (a shortcut made in Terminal with `ln -s`) was left at its old place. Romperoom
looks at the folder itself and doesn't follow the link to say it's there, though **Scan now**
still reads through it and counts its games. Nothing is wrong with your games. To make it read
normally, remove the library in Settings › Libraries and add the folder where it now is, then
press **Scan now**. Removing it forgets its Tidy up history, so finish or undo anything waiting
in Recovery first.

## Your library changed since you looked

A preview is only good for the library as it was when you looked. A scan, a copy to a card or
another tidy in between makes it out of date, and Romperoom refuses it rather than move the
wrong file. Press **Look again** to see what is there now.

## A library says Not checked under Across libraries

**Across libraries** only compares libraries it can vouch for, and lists the others under **Not
checked** with the reason. The libraries it can check are still compared with each other.

- **It is being scanned now:** wait for the scan to finish; the list updates by itself.
- **It hasn't been scanned completely** or **its last scan didn't see every file:** press **Scan
  again** on the tab.
- **Its folder isn't available:** connect the drive or the network share, then press **Look
  again**. An unplugged drive's empty folder counts as not available, as in **Settings** ›
  **Libraries**. A library whose folder was moved and replaced by a symbolic link also reads this
  way (see [A library says it can't reach its folder, but it
  scans](#a-library-says-it-cant-reach-its-folder-but-it-scans)).
- **Part of its folder is also another library:** one library's folder is inside the other's, or
  both lead to the same folder, so every file would look like a copy of itself. Romperoom refuses
  to compare them, as **Add a library…** refuses to add such a folder. This usually comes from a
  drive mounted somewhere else or a moved folder: put the folders back as they were, then press
  **Look again**. If two libraries really are one folder, remove one in **Settings** ›
  **Libraries** (removing a library forgets its history; see
  [Your libraries](user-guide.md#your-libraries)).

## Across libraries leaves out a copy, or shows one twice

**Across libraries** only lists files with exactly the same bytes, so two zips of one game with
different contents are not copies. It leaves some files out on purpose:

- **A file "has moved or gone since the last scan":** a copy was moved, renamed or deleted
  after the last scan, so Romperoom can't check it, and its set may be missing from the list.
  Press **Scan again**, then **Look again**. This can also happen for a moment while Tidy up,
  Standardise or re-link is renaming files in one of your libraries; press **Look again** when it
  ends.
- **A file replaced since the last scan** with a different one of the same name is still listed:
  the list is as each library's last scan saw it. Scan both libraries again before you remove a
  copy by hand.
- **Files that couldn't be read** have nothing to compare, so they are neither listed nor counted.
  **Health** lists them (see [Files that couldn't be read](#files-that-couldnt-be-read)).
- **Tracks of a multi-file game** and **the same file in two consoles' folders** are left out
  and counted, as on **Duplicates**.
- **A file hard-linked in two places** (one file with two names, made in Terminal with `ln`) is
  one file, not two copies, so it is not listed. If such a pair sits inside one library, a copy
  of it in another library is not listed either.

The same network share connected twice (at two different places in Finder) and added as two
libraries looks like two libraries: every file in it then shows as held twice. Connect the share
at one place again if you can, as it was when you added it. If you no longer need one of the two
libraries, remove it in **Settings** › **Libraries** (removing a library forgets its history; see
[Your libraries](user-guide.md#your-libraries)).

The look checks each possible copy on disk. On a slow network share that stops answering
part way, the window can stop responding until the share answers again.

With a screen reader: nothing is read out when you open the tab or turn a page (the line above
the list says where you are). After **Look again** the result is read out once, with how many
libraries were not checked. A look that fails reads nothing out (the message on the tab says
what went wrong), and a result the same as the last one read out is not read again.

## A copy set aside from Across libraries wasn't deleted

**Delete forever** deletes a copy set aside from **Across libraries** only while the copy it was
set aside for is still provably there: the library that kept it is connected and still in
Settings › Libraries, and its copy has the same contents. Otherwise the copy stays set aside and
the preview counts it as kept. Connect the library and try again, or put the copy back.

After **Standardise** renamed folders or games in the library that kept the other copy, Delete
forever finds the kept copy under its new name by its contents and checks it the same way. If you
renamed or moved the kept copy yourself, scan that library first: until then Romperoom can't find
it, and the set-aside copy stays. Nothing is deleted without a proven kept copy; put the copy back
if you want it.

## A file couldn't be put back

After an undo or a put-back, the result lists any file that stayed set aside:

- **A file is already in that spot:** a file now sits where the set-aside one came from.
  Romperoom never overwrites it. Move or rename the file in the way, then put the other back
  again from **Set aside**.
- **It couldn't be put back:** the file changed or went missing from the set-aside folder, or
  the drive refused the move. Check the drive is connected and the file is still in
  `.romperoom-quarantine`, then try again.

## Romperoom was interrupted while tidying

If Romperoom quit, or you pressed **Cancel**, part way through a tidy, it says "Something
Romperoom was tidying stopped before it finished. Finish it or undo what was done. If it was
setting files aside, you can also discard the rest." At the next start a panel opens by itself;
after a Cancel, press **Finish or undo…** on the result. For each run:

- **Finish** moves the files that were left, with the same checks as a new run.
- **Undo what was done** puts back the files that already moved.
- **Discard the rest** (a Tidy up run only) keeps what already moved set aside and leaves the
  other files where they are; nothing moves. If it says "Romperoom couldn't discard the rest.", a
  file of the run isn't where the run left it, so part of it may already have moved: choose
  **Finish** or **Undo what was done** instead. A Standardise, re-link or card-sync run has no
  Discard the rest: it is finished or undone as a whole.
- **Later** leaves it for now. Tidy up's overview keeps offering it until you choose.

If it says your library folder isn't available, reconnect the drive first: Finish, Undo what
was done and Discard the rest all need it, and change nothing without it. An interrupted
delete forever offers **Finish deleting** or **Keep the rest**.

## Standardise left a folder or game as it was

The review says why, in one line:

- **"This profile has no folder name for this console."** The device you chose has no folder for
  it: choose another device, or leave it.
- **"Romperoom doesn't know which console this folder is for."** Assign the folder a console (on
  Health), then review again.
- **"This folder could be more than one console."** Its games were catalogued as more than one
  console, or as another console than its name says. If the folder really holds two consoles,
  move one console's games to its own folder; otherwise scan again. Then review again.
- **"A different file with this name is already there."** Something else has the new name.
  Romperoom never overwrites: move or rename it yourself, then review again.
- **"The official name can't be used as a file name."** The game database's name has a character
  or a length no file system takes.
- **"A part of this game is missing"** or **"doesn't match the official data."** Every file of a
  game must be there and match the database by its contents, so it is never half renamed. Scan
  again, and identify, then review again.
- **"This game's file matches more than one game in your databases."** Its contents are the same
  as several games in the database, so Romperoom won't pick one: choose the right one under
  **Check name matches**, then review again.
- **"Romperoom couldn't read this item, so it was left alone."** A file or folder couldn't be
  opened (it may be locked, or the drive refused it). Check the drive, then review again.
- **"A picture or save of this game is an identical copy you chose to set aside, so the game
  keeps its name."** A clash you ticked sets aside this game's picture or save (an identical copy
  is already in the folder it merges into), so the game keeps its name rather than be renamed
  without it. Untick that clash to rename the game.
- **"Romperoom couldn't put everything back after an item failed."** An item failed part way and
  something it had already moved couldn't go back (its old name was taken meanwhile). The run
  waits under Tidy up's **Finish or undo…**: free the name and finish it, or undo it.

A run as a whole can end early, with one of these lines:

- **"Not enough space in the library."** A rewritten game list, cue sheet or playlist needs room
  on the drive. Free some space, then review again; anything already renamed stays.
- **"The library is busy with …"** Another job (Tidy up, a scan, identify, cover art, a card sync
  or a deploy) is using the library. Try again when it finishes.
- **"Stopped. Everything done before you pressed Stop is kept and can be undone."** You pressed
  **Stop**. What was left waits under Tidy up's **Finish or undo…**.
- **"Your library folder isn't available."** The drive went away during the run. Reconnect it;
  Tidy up's **Finish or undo…** finishes or rolls back the rest.

While a run waits under **Finish or undo…**, Romperoom won't start another run of that library
(it says **"An earlier run of this library was interrupted"**) or remove the library in Settings ›
Libraries (it says **"This library has an interrupted standardise run"**). Finish it or undo it
first, then try again.

Games that aren't identified are never renamed. When two folders merge, a file whose name is
already taken stays in its folder unless it is an identical copy you ticked to set aside, so the
folder may not end up empty; Romperoom never removes folders.

## A game list wasn't changed

A game list (`gamelist.xml`), cue sheet or playlist is changed only where it names a renamed game,
and only when Romperoom can read it with certainty. One it can't (not UTF-8, damaged, or naming a
game in a way that could mean two files) is left exactly as it was and named in the review or the
results: "Romperoom couldn't read this game list, so it was left alone." A game whose own cue
sheet or playlist can't be read is not renamed. A game list a frontend keeps outside your library
(ES-DE's own `gamelists` folder) is not touched.

## A frontend still shows the old names

A frontend's own database (as opposed to a game list in your library) cannot be updated: run its
scan or "update game lists" after standardising. On a card, copy the games again with **SD card**.
Saves on a card keep their old names, and the next card sync treats the renamed game's card save as
a save of the old name.

## Undo this run left something

Undo puts back only what still holds exactly what the run wrote: a file or folder that changed or
moved since, or whose old name is now taken, is left and listed. Every game list, cue sheet and
playlist the run replaced keeps its original in `.romperoom/lists-backup/<run>/` in your library.
A run cut short by a crash or by the library going away waits in Tidy up's **Finish or undo…**,
like a tidy.

## A picture wasn't offered for re-linking

Re-link offers a picture only when exactly one game of the same console has the same title once
tags like (USA) or (Rev 1) are ignored. Two games of that title (a USA and a Japanese copy), a
title that differs by more than tags (a misspelling, a collection number such as `001 - `), a
picture named the way some scrapers name them (`Tetris-image.png`), a copy Romperoom recognises
beside one it doesn't, a game on several discs that Romperoom recognises disc by disc (each disc
counts as a game of its own), or a picture in a folder Romperoom doesn't link to a console, all
leave it in the leftover list with no suggestion. One that matches but can't be renamed says why
there: another picture wants the same game, or the new name is taken (often because the game
already has a picture of that name in the same folder, or because a picture of another game in
that folder would get the same name) or too long. Re-link needs a full scan, as the leftover list
does.

## A game list entry wasn't re-pointed

An entry is re-pointed only when one game clearly matches it, no other entry of that list
already names that game ("The game list already has an entry for this game": your frontend may
have added a fresh entry after the rename), and no other entry without a game in that list looks
like the same game ("Another entry without a game looks like the same game, so Romperoom can't
tell which one to change."). An entry whose path is absolute (such as `/roms/gb/Tetris.gb`) is
never listed, because Romperoom can't check it inside your library. A game list Romperoom can't
read with certainty is never changed. Romperoom never deletes an entry.

A picture's reference in a game list is updated only when it is written relative to the console
folder (`./images/Tetris (USA).png`). One written as an absolute path is never changed, so a
frontend reading it still looks for the old name.

## A frontend can't find a picture after a re-link was undone

A re-link changes two things, the pictures' names and the game lists, and each can be put back on
its own:

- If a re-link was interrupted while it was writing the game lists and you chose **Undo what was
  done** in **Finish or undo…**, the game lists get their old text back but the pictures keep their
  new names, so the lists name pictures that aren't there. Undo the whole re-link (from the results
  or **History**, in **Recent re-links**) to give the pictures their old names back too, or let
  your frontend rescan or re-scrape its game list.
- If **Undo** kept a game list because it changed after the run, the pictures get their old names
  back but that list keeps the text it has now, which names the new ones. Let your frontend rescan
  or re-scrape its game list; the list as it was before the re-link is in
  `.romperoom/lists-backup/<run>/` in your library.

## Windows and Linux

Romperoom has only been run by hand on macOS. Its unit and end-to-end tests pass in CI on a real
Windows PC and in an Ubuntu container, but nobody has used the app by hand on either. See
[development.md](development.md#platforms).

On Linux, prefer the `.deb`: it installs Electron's sandbox helper so the app starts on Ubuntu
23.10 and later. If the AppImage exits at once with a message about the sandbox, that is the
reason; use the `.deb` instead (never run it with `--no-sandbox`).

If you try it on Windows from source:

- `npm ci` needs no Visual Studio or Python: install scripts are off, and the SQLite binary
  ships with the package.
- Run `node node_modules/electron/install.js` once if the first launch can't download Electron.
- `scripts/nas-survey.mjs` and `scripts/live-identify.mjs` refuse to run on Windows ("this
  platform cannot refuse a link at the name"). That is deliberate.
- Some tests are skipped without Administrator or Developer Mode, because they need symlinks,
  and the engine's coverage gate then fails. Run the gate from an elevated shell or turn on
  Developer Mode.

## For developers

- Tests, the e2e suite and the fresh-clone gate: [testing.md](testing.md).
- Native module and Electron binary problems: [development.md](development.md).
- **`npm ci` tries to compile something** (`gyp`, `MSBuild`, `cl.exe` in the log): the root
  `.npmrc` is missing or was overridden (`npm_config_ignore_scripts=false`). Install scripts stay
  off ([development.md](development.md#install-scripts-are-off)).
- **"Electron failed to install correctly"** or no `node_modules/electron/dist`: install scripts
  are off, so fetch the binary with `node node_modules/electron/install.js`.
- **"Could not locate the bindings file"** from better-sqlite3: the package has no prebuild for
  this platform and architecture. `packages/engine/test/prebuilds.test.ts` lists the ones we
  rely on; see [development.md](development.md#native-modules-better-sqlite3).
- Git prints `non-monotonic index` errors in a clone on an exFAT drive: harmless macOS `._` files
  inside `.git`. Keep clones on APFS or another native volume.
