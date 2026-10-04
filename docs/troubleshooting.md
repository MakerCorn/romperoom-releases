# Troubleshooting

Problems you might meet, and what to do. Only Tidy up changes your library, and it only sets
files aside, where you can put them back. None of these problems can harm your collection.

## Contents

- [Romperoom won't start from a terminal](#romperoom-wont-start-from-a-terminal)
- [Can't reach your game folder](#cant-reach-your-game-folder)
- [We didn't find any games in that folder](#we-didnt-find-any-games-in-that-folder)
- [Some games are missing](#some-games-are-missing)
- [Some games stay unidentified](#some-games-stay-unidentified)
- [A game database won't download](#a-game-database-wont-download)
- [A console says "Not available at this version"](#a-console-says-not-available-at-this-version)
- [Some game folders look empty or gone](#some-game-folders-look-empty-or-gone)
- [Files that couldn't be read](#files-that-couldnt-be-read)
- [The scan is slow](#the-scan-is-slow)
- [The scan didn't finish](#the-scan-didnt-finish)
- [Starting over](#starting-over)
- [A card doesn't show up](#a-card-doesnt-show-up)
- [A card is greyed out](#a-card-is-greyed-out)
- [The games don't fit on the card](#the-games-dont-fit-on-the-card)
- [Copying to the card is slow](#copying-to-the-card-is-slow)
- [Tidy up says Romperoom is busy](#tidy-up-says-romperoom-is-busy)
- [Your library folder isn't available](#your-library-folder-isnt-available)
- [Your library changed since you looked](#your-library-changed-since-you-looked)
- [A file couldn't be put back](#a-file-couldnt-be-put-back)
- [Romperoom was interrupted while tidying](#romperoom-was-interrupted-while-tidying)
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
- **Two revisions of one game** (such as `(Rev 1)` and the original) show as one game.
- **A game stored as a folder of files** (some PC and DOS games) shows as several games, or as
  none. Folder games are handled in a later version.

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

Download for me and Check for updates (Settings › Game databases) are the only things that
take Romperoom online, and only to GitHub. Each request is listed under **Network activity**,
with its outcome. A file that fails leaves nothing behind: no partial file and no change to your
databases. The message says what happened:

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
changed and 3 when there is a newer version. Others on the same network (an office, a school)
share that limit.

**Proxies and filtered networks.** Download for me connects to GitHub directly. It does not use
a system proxy, a PAC file or a proxy named in the environment, and it trusts only the
certificate authorities built into Romperoom, not ones your organisation installed. A network
that requires a proxy, or that inspects secure connections with its own certificate, cannot use
it. Use **Get a database from the official site** instead: your browser does the downloading.

## A console says "Not available at this version"

The version of libretro-database Romperoom is using has no file for that console (it may have
been renamed or removed upstream). Press **Check for updates**; if the console still says so,
the next Romperoom release will update the mapping. Meanwhile, get its database from the
official site.

## Some game folders look empty or gone

Romperoom never takes games off its list on its own when whole folders seem to vanish, because a
drive that is only partly connected looks the same. If the drive was disconnected, reconnect it
and press **Scan again**. If you removed those games on purpose, press **It's OK — they were
removed**. Only Romperoom's list changes. The same goes for **Some cover art is missing**.

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
3. Open Romperoom again. Your theme choice is reset too.

Files you set aside in Tidy up stay in the `.romperoom-quarantine` folder inside your library,
but a fresh start no longer knows about them. Put back what you want first, or move them back
by hand afterwards: each one sits under a dated folder, at its old path.

There is no button to remove a library in this version.

If your library is gone after updating from a build before 0.1.0: those builds kept their
catalogue in a folder named `@romperoom/desktop`, and nothing moves it. Quit Romperoom and move
it to the folder above, or add your library again and re-scan.

## A card doesn't show up

On **Put games on a card**, the **Where** step lists the drives Romperoom could use.

- Check that the card is in, and that your computer shows it (in Finder, or File Explorer on
  Windows). A card in a reader that isn't mounted can't be listed. Then press **Refresh**.
- A card that needs formatting doesn't show up: Romperoom never formats cards. Format it with
  your device, or with your computer's disk tool, then press **Refresh**.
- No card at hand? **Export to a folder** copies the same layout into a folder; move that
  folder's contents to the card's top level yourself.

## A card is greyed out

A greyed-out card says why under its name:

- **Locked (read-only):** slide the lock switch on the side of the SD card (or its adapter) up,
  then put it back in and press **Refresh**.
- **Your computer's own system drive**, or **a part of your computer Romperoom never writes to:**
  Romperoom never writes there, on purpose. Choose the SD card.
- **A network drive:** copy to a card plugged into this computer instead.
- **Your game library or BIOS files are on it,** or **it's inside your game library:** Romperoom
  never writes into your library. Use a separate card.
- A disk inside your computer isn't greyed out, but you have to type its name to use it. That
  check is there so a wrong click can't fill the wrong disk.

## The games don't fit on the card

The **Where** step shows the space the games, artwork and the card's own format take. If it
doesn't fit, it says by how much and offers what to leave out: the artwork, the other versions
of each game, or a big console. You can also go **Back** and untick consoles. Copying stays off
until it fits. The card's format loses some space to every file, so a card full of small games
fits fewer than their sizes suggest; Romperoom counts that.

If a card fills up during a copy (something else wrote to it), the report says **The card
filled up**. Free some space, or leave something out, and copy again: what was already copied
stays.

## Copying to the card is slow

- Romperoom checks every file after copying it, so a copy of many small files is slower than a
  plain drag and drop. A second copy only writes what changed, so it is quick.
- Cheap or old cards, and USB 2 card readers, are slow at small files. A faster card or a
  reader in a USB 3 port helps most.
- **Windows:** antivirus software (Windows Security included) scans every new file on the card,
  which slows a copy of many files. That is normal; let it finish. Romperoom has not been run on
  a real Windows PC yet, so tell us how it went.
- You can **Cancel the copy** at any time. The next copy picks up from there.

## Tidy up says Romperoom is busy

"Romperoom is busy with a scan/copy. Try again when it finishes." means a scan, a copy to a
card or another tidy is using that library. Nothing was changed. Wait for it to finish (or
cancel it), then press the button again.

## Your library folder isn't available

"Your library folder isn't available right now — is the drive connected?" means the folder at
the library's path is missing, or is not the library that was tidied (a drive that mounted
empty, or another drive at the same path). Nothing was changed. Reconnect the drive, check it
opens in Finder, then try again.

## Your library changed since you looked

A preview is only good for the library as it was when you looked. A scan, a copy to a card or
another tidy in between makes it out of date, and Romperoom refuses it rather than move the
wrong file. Press **Look again** to see what is there now.

## A file couldn't be put back

After an undo or a put-back, the result lists any file that stayed set aside:

- **A file is already in that spot:** a file now sits where the set-aside one came from.
  Romperoom never overwrites it. Move or rename the file in the way, then put the other back
  again from **Set aside**.
- **It couldn't be put back:** the file changed or went missing from the set-aside folder, or
  the drive refused the move. Check the drive is connected and the file is still in
  `.romperoom-quarantine`, then try again.

## Romperoom was interrupted while tidying

If Romperoom quit, or you pressed **Cancel**, part way through a tidy, it says "Romperoom was
interrupted while tidying. Finish it, or undo what was done." At the next start a panel opens
by itself; after a Cancel, press **Finish or undo…** on the result. For each run:

- **Finish** moves the files that were left, with the same checks as a new run.
- **Undo what was done** puts back the files that already moved.
- **Later** leaves it for now. Tidy up's overview keeps offering it until you choose.

If it says your library folder isn't available, reconnect the drive first. An interrupted
delete forever offers **Finish deleting** or **Keep the rest**.

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
