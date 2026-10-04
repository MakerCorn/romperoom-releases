# Systems

<!-- Generated from packages/profiles/data/systems.json by `npm run systems-doc`.
     Do not edit by hand: a test fails when this file and the catalog disagree. -->

Romperoom knows 307 systems (console 84, handheld 31, computer 74, arcade 39, engine 13, port 51, other 15). A top-level folder of a library resolves to a system when its name, compared without case, accents, spaces or punctuation, equals the id, the name or an alias of exactly one system.

## Sources

- ES-DE `resources/systems/unix/es_systems.xml` (its FreeBSD configuration: the same systems as its Linux file, without the Linux launcher formats such as `.appimage` and `.desktop`) and the user guide "Supported game systems" table: the ids, names and extension lists. Where sources disagree, ES-DE wins.
- Batocera `es_systems.yml`: more systems, their kind (console, portable, ...) and folder names; extensions from its `*.emulator.yml` and `*.libretro.core.yml` files.
- Onion OS emulator packages (`static/packages/Emu/*/Emu/<FOLDER>/config.json`): the upper-case folder names handheld firmwares use (FC, SFC, MD, POKE, ...).
- The reference library: a few systems no source lists (their extensions are conservative).

Each system from a source lists its URLs in `systems.json`; the 17 from the reference library have no upstream source URL. Extensions are lower-cased and dotted. Media extensions (`.png`, `.jpg`, `.mp4`, `.pdf`, ...) are dropped from every list, because the scanner treats them as media: PICO-8 and TIC-80 `.png` cartridges are not catalogued yet. Every file of a disc image (`.cue` and its `.bin`s) is catalogued on its own. Two formats the reference library stores are added to ES-DE lists: `.wsc` to `wonderswan` (Onion OS's WS folder holds both) and `.pak` to `openbor` (Batocera's OpenBOR list).

Batocera entries left out (not game systems): `flatpak` `imageviewer` `library` `moonlight` `recordings` `vgmplay` `windows_installers`.

## Judgement calls

| Folder | System | Why |
| --- | --- | --- |
| `megadrive` | `genesis` | ES-DE lists genesis and megadrive (and megadrivejp) as regional twins of one machine; Romperoom keeps its existing id genesis, because catalogs store it. |
| `famicom` | `nes` | Regional twin in ES-DE; one system here. Also fc (Onion OS). |
| `sfc` | `snes` | Regional twins sfc and snesna in ES-DE. |
| `mdcd` | `segacd` | Handheld-firmware name for the Mega-CD; ES-DE megacd and megacdjp fold in too. |
| `sega32xjp` | `sega32x` | Regional twins sega32xjp and sega32xna in ES-DE. |
| `tg16` | `pcengine` | ES-DE tg16 is the PC Engine outside Japan. |
| `tg-cd` | `pcenginecd` | ES-DE tg-cd is the PC Engine CD. |
| `saturnjp` | `saturn` | Regional twin in ES-DE. |
| `neogeocdjp` | `neogeocd` | Regional twin in ES-DE. |
| `mark3` | `mastersystem` | The Sega Mark III is the Japanese Master System (ES-DE lists both). |
| `videopac` | `odyssey2` | The Philips Videopac G7000 is the Magnavox Odyssey 2; o2em is its emulator. |
| `msx1` | `msx` | ES-DE lists msx and msx1 with one platform. |
| `msx2+` | `msx2` | Folder names are compared without punctuation, so msx2+ (Batocera) cannot be told from msx2 and resolves to it. |
| `ATARI` | `atari2600` | Onion OS keeps Atari 2600 games in ATARI; no other source uses the bare name. |
| `NGC` | `gc` | Nintendo GameCube (ES-DE id gc; Batocera gamecube). .ngc is also a Neo Geo Pocket Color extension, but no source names a Pocket folder NGC. |
| `pico` | `pico` | Batocera pico is the Sega Pico. Onion OS stores PICO-8 in PICO; that is not an alias: use pico8. |
| `cave` | `cave3rd` | Cave CV1000 arcade boards (Batocera cave3rd). |
| `VARCADE` | `arcade` | A collection of vertical-screen arcade romsets (MAME zips) in the reference library. |
| `MAME2010` | `mame` | A MAME romset version; the version is not tracked separately yet. |
| `HBMAME` | `hbmame` | Homebrew and hacked MAME sets: its own system, since its romsets are not MAME. |
| `PSPMINIS` | `psp` | PSP minis are PSP software. |
| `gzdoom` | `doom` | ES-DE's doom covers every Doom engine; prboom and uzdoom (Batocera) fold in too. |
| `TYRQUAKE` | `quake` | A Quake engine. |
| `vitaquake2` | `quake2` | A Quake II engine. |
| `boom3` | `doom3` | A Doom 3 engine (dhewm3 "boom3"). |
| `c20` | `vic20` | Batocera c20 is the Commodore VIC-20. |
| `xegs` | `atarixe` | The XE Game System is an Atari XE. |
| `THOMSON` | `moto` | ES-DE moto is the Thomson MO/TO series. |
| `oricatmos` | `oric` | The Oric Atmos is an Oric. |
| `cdi` | `cdimono1` | ES-DE id for the Philips CD-i. |
| `casloopy` | `loopy` | The Casio Loopy (Batocera loopy). |
| `segastv` | `stv` | Sega Titan Video (ES-DE stv). |
| `VMAC` | `macintosh` | Mini vMac, a Macintosh emulator. |
| `VIDEOTON` | `tvc` | The Videoton TVC (Batocera tvc). |
| `gb-msu` | `sgb-msu1` | Batocera sgb-msu1, Game Boy with MSU-1. |
| `snes-msu` | `snes-msu1` | SNES MSU-1 games; also SFCMSU. |
| `sonic3air` | `sonic3-air` | Batocera sonic3-air. |
| `sonicmania` | `sonic-mania` | Batocera sonic-mania. |
| `ONS` | `onscripter` | ONScripter visual novels. |
| `COMMODORE` | `c64` | Onion OS keeps Commodore 64 games here. |
| `FLASHBACK` | unmapped | Could be the Atari Flashback or the game Flashback (see reminiscence): unmapped. |
| `PANASONIC` | unmapped | A maker, not a system (3DO, JR-200, ...): unmapped. |
| `SFX` | unmapped | No source names a system SFX: unmapped. |
| `pegasus` | unmapped | A frontend (Pegasus) or a Famicom clone; empty in the reference library: unmapped. |
| `nes3d_launchers` | unmapped | A support folder of 3dSen launcher files, not a system: unmapped (ignore it). |

## Conservative extension lists

No source lists extensions for these systems (most run on MAME, whose software lists are archives), so they catalogue archives, or, for a variant of another system (such as `snes-msu1`), that system's list:

- `actionmax`: `.7z` `.zip`
- `advision`: `.7z` `.zip`
- `amiga4000`: `.7z` `.adf` `.adz` `.ccd` `.chd` `.cue` `.dms` `.fdi` `.hdf` `.hdz` `.ipf` `.iso` `.lha` `.m3u` `.mds` `.nrg` `.rp9` `.uae` `.zip`
- `apfm1000`: `.7z` `.zip`
- `atom`: `.7z` `.zip`
- `beena`: `.7z` `.zip`
- `camplynx`: `.7z` `.zip`
- `cave3rd`: `.7z` `.zip`
- `cgenie`: `.7z` `.zip`
- `ctvboy`: `.7z` `.zip`
- `dragon64`: `.7z` `.zip`
- `eduke32`: `.7z` `.zip`
- `enterprise`: `.7z` `.zip`
- `fury`: `.7z` `.zip`
- `gaelco`: `.7z` `.zip`
- `gamepock`: `.7z` `.zip`
- `gb2players`: `.7z` `.bs` `.cgb` `.dmg` `.gb` `.gbc` `.gbx` `.sfc` `.sgb` `.smc` `.zip`
- `gba2players`: `.7z` `.agb` `.bin` `.cgb` `.dmg` `.gb` `.gba` `.gbc` `.gbx` `.sgb` `.zip`
- `gbc2players`: `.7z` `.bs` `.cgb` `.dmg` `.gb` `.gbc` `.gbx` `.sfc` `.sgb` `.smc` `.zip`
- `gemrb`: `.7z` `.zip`
- `gong`: `.7z` `.zip`
- `hbmame`: `.7z` `.zip`
- `hikaru`: `.7z` `.zip`
- `karaoke`: `.7z` `.zip`
- `laser310`: `.7z` `.zip`
- `loopy`: `.7z` `.zip`
- `love`: `.love`
- `mc10`: `.7z` `.zip`
- `megadrive-msu`: `.32x` `.68k` `.7z` `.bin` `.bms` `.chd` `.cue` `.gen` `.gg` `.iso` `.m3u` `.md` `.mdx` `.sg` `.sgd` `.smd` `.sms` `.zip`
- `mz2000`: `.7z` `.zip`
- `mz2500`: `.7z` `.zip`
- `mz700`: `.7z` `.zip`
- `mz800`: `.7z` `.zip`
- `mz80k`: `.7z` `.zip`
- `namco22`: `.7z` `.zip`
- `neogeo64`: `.7z` `.zip`
- `nes3d`: `.3dsen` `.7z` `.nes` `.zip`
- `onscripter`: `.7z` `.zip`
- `openlara`: `.7z` `.zip`
- `pc60`: `.7z` `.zip`
- `pc80`: `.7z` `.zip`
- `pcw`: `.7z` `.zip`
- `pdp1`: `.7z` `.zip`
- `pgm`: `.7z` `.zip`
- `pgm2`: `.7z` `.zip`
- `pico`: `.7z` `.zip`
- `pv2000`: `.7z` `.zip`
- `rx78`: `.7z` `.zip`
- `sc3000`: `.7z` `.zip`
- `segaai`: `.7z` `.zip`
- `sgb-msu1`: `.7z` `.gb` `.gbc` `.sgb` `.zip`
- `singe`: `.7z` `.zip`
- `snes-msu1`: `.7z` `.bin` `.bml` `.bs` `.bsx` `.dx2` `.fig` `.gd3` `.gd7` `.mgd` `.sfc` `.smc` `.st` `.swc` `.zip`
- `socrates`: `.7z` `.zip`
- `sv8000`: `.7z` `.zip`
- `teknoparrot`: `.7z` `.zip`
- `ti83`: `.7z` `.8xp` `.zip`
- `tutor`: `.7z` `.zip`
- `tvgames`: `.7z` `.zip`
- `vc4000`: `.7z` `.zip`
- `vis`: `.7z` `.zip`
- `zaccariapinball`: `.7z` `.zip`
- `zinc`: `.7z` `.zip`

## All systems

| Id | Name | Kind | Aliases | Extensions | From |
| --- | --- | --- | --- | --- | --- |
| `3do` | 3DO Interactive Multiplayer | console |  | `.7z` `.bin` `.chd` `.cue` `.iso` `.zip` | es-de |
| `abuse` | Abuse | port |  | `.game` | batocera |
| `actionmax` | Worlds of Wonder ActionMax | console |  | `.7z` `.zip` | conservative |
| `adam` | Coleco Adam | computer |  | `.1dd` `.7z` `.bin` `.col` `.cqi` `.cqm` `.d77` `.d88` `.ddp` `.dfi` `.dsk` `.hfe` `.imd` `.mfi` `.mfm` `.rom` `.td0` `.wav` `.zip` | es-de |
| `advision` | Adventure Vision | console |  | `.7z` `.zip` | conservative |
| `ags` | Adventure Game Studio Game Engine | engine |  | `.desktop` `.sh` | es-de |
| `amiga` | Commodore Amiga | computer |  | `.7z` `.adf` `.adz` `.ccd` `.chd` `.cue` `.dms` `.fdi` `.hdf` `.hdz` `.ipf` `.iso` `.lha` `.m3u` `.mds` `.nrg` `.rp9` `.uae` `.zip` | es-de |
| `amiga1200` | Commodore Amiga 1200 | computer | `Amiga AGA` | `.7z` `.adf` `.adz` `.ccd` `.chd` `.cue` `.dms` `.fdi` `.hdf` `.hdz` `.ipf` `.iso` `.lha` `.m3u` `.mds` `.nrg` `.rp9` `.uae` `.zip` | es-de |
| `amiga4000` | Commodore Amiga 4000 | computer |  | `.7z` `.adf` `.adz` `.ccd` `.chd` `.cue` `.dms` `.fdi` `.hdf` `.hdz` `.ipf` `.iso` `.lha` `.m3u` `.mds` `.nrg` `.rp9` `.uae` `.zip` | conservative |
| `amiga500` | Amiga OCS/ECS | computer |  | `.adf` `.adz` `.dms` `.dmz` `.exe` `.hdf` `.ipf` `.lha` `.m3u` `.raw` `.scp` `.uae` `.zip` | batocera |
| `amiga600` | Commodore Amiga 600 | computer |  | `.7z` `.adf` `.adz` `.ccd` `.chd` `.cue` `.dms` `.fdi` `.hdf` `.hdz` `.ipf` `.iso` `.lha` `.m3u` `.mds` `.nrg` `.rp9` `.uae` `.zip` | es-de |
| `amigacd32` | Commodore Amiga CD32 | console |  | `.7z` `.adf` `.adz` `.ccd` `.chd` `.cue` `.dms` `.fdi` `.hdf` `.hdz` `.ipf` `.iso` `.lha` `.m3u` `.mds` `.nrg` `.rp9` `.uae` `.zip` | es-de |
| `amstradcpc` | Amstrad CPC | computer | `cpc` | `.7z` `.cdt` `.cpr` `.dsk` `.kcr` `.m3u` `.sna` `.tap` `.tar` `.voc` `.zip` | es-de |
| `android` | Google Android | other |  | `.7z` `.zip` | es-de |
| `androidapps` | Android Apps | other |  | `.7z` `.zip` | es-de |
| `androidgames` | Android Games | other |  | `.7z` `.zip` | es-de |
| `apfm1000` | M-1000 | console |  | `.7z` `.zip` | conservative |
| `apple2` | Apple II | computer |  | `.7z` `.do` `.dsk` `.nib` `.po` `.zip` | es-de |
| `apple2gs` | Apple IIGS | computer |  | `.2mg` `.7z` `.zip` | es-de |
| `arcade` | Arcade | arcade | `varcade` | `.7z` `.cmd` `.desktop` `.neo` `.sh` `.zip` | es-de |
| `arcadia` | Emerson Arcadia 2001 | console | `Arcadia 2001` | `.7z` `.bin` `.tvc` `.zip` | es-de |
| `archimedes` | Acorn Archimedes | computer |  | `.1dd` `.360` `.7z` `.adf` `.adl` `.adm` `.ads` `.apd` `.bbc` `.chd` `.cqi` `.cqm` `.d77` `.d88` `.dfi` `.dsd` `.dsk` `.hfe` `.ima` `.imd` `.img` `.ipf` `.jfd` `.mfi` `.mfm` `.msa` `.ssd` `.st` `.td0` `.ufi` `.zip` | es-de |
| `arduboy` | Arduboy Miniature Game System | handheld |  | `.7z` `.arduboy` `.hex` `.zip` | es-de |
| `astrocde` | Bally Astrocade | console | `astrocade` | `.7z` `.zip` | es-de |
| `atari2600` | Atari 2600 | console | `atari` `a2600` | `.7z` `.a26` `.bin` `.zip` | es-de |
| `atari5200` | Atari 5200 | console | `fiftytwohundred` `a5200` | `.7z` `.a52` `.atr` `.atx` `.bin` `.car` `.cas` `.cdm` `.com` `.rom` `.xex` `.xfd` `.zip` | es-de |
| `atari7800` | Atari 7800 ProSystem | console | `seventyeighthundred` `a7800` | `.7z` `.a78` `.bin` `.zip` | es-de |
| `atari800` | Atari 800 | computer | `a800` | `.7z` `.a52` `.atr` `.atx` `.bin` `.car` `.cas` `.cdm` `.com` `.rom` `.xex` `.xfd` `.zip` | es-de |
| `atarijaguar` | Atari Jaguar | console | `jaguar` | `.7z` `.abs` `.bin` `.cdi` `.cof` `.cue` `.j64` `.jag` `.prg` `.rom` `.zip` | es-de |
| `atarijaguarcd` | Atari Jaguar CD | console | `jaguarcd` | `.abs` `.bin` `.cdi` `.cof` `.cue` `.j64` `.jag` `.prg` `.rom` | es-de |
| `atarilynx` | Atari Lynx | handheld | `lynx` | `.7z` `.lnx` `.lyx` `.o` `.zip` | es-de |
| `atarist` | Atari ST | computer |  | `.7z` `.dim` `.ipf` `.m3u` `.msa` `.st` `.stx` `.zip` | es-de |
| `atarixe` | Atari XE | computer | `xegs` `Atari XE Game System` | `.7z` `.a52` `.atr` `.atx` `.bin` `.car` `.cas` `.cdm` `.com` `.rom` `.xex` `.xfd` `.zip` | es-de |
| `atom` | Atom | computer |  | `.7z` `.zip` | conservative |
| `atomiswave` | Sammy Corporation Atomiswave | arcade |  | `.7z` `.bin` `.dat` `.elf` `.lst` `.zip` | es-de |
| `bbcmicro` | Acorn Computers BBC Micro | computer |  | `.7z` `.dsd` `.img` `.ssd` `.zip` | es-de |
| `beena` | Advanced Pico Beena | console |  | `.7z` `.zip` | conservative |
| `bennugd` | Bennu Game Development | computer |  | `.dat` `.dcb` | batocera |
| `bk` | Elektronika BK | computer |  | `.7z` `.bin` `.bkd` `.dsk` `.img` `.zip` | batocera |
| `bstone` | Blake Stone | port |  | `.bstone` | batocera |
| `c128` | Commodore 128 | computer |  | `.7z` `.d64` `.d81` `.lnx` `.m3u` `.prg` `.zip` | batocera |
| `c64` | Commodore 64 | computer | `commodore` | `.7z` `.bin` `.cmd` `.crt` `.d2m` `.d4m` `.d64` `.d6z` `.d71` `.d7z` `.d80` `.d81` `.d82` `.d8z` `.g41` `.g4z` `.g64` `.g6z` `.gz` `.lnx` `.m3u` `.nbz` `.nib` `.p00` `.prg` `.t64` `.tap` `.vfl` `.vsf` `.x64` `.x6z` `.zip` | es-de |
| `camplynx` | Camputers Lynx | computer |  | `.7z` `.zip` | conservative |
| `cannonball` | Cannonball | port |  | `.cannonball` | batocera |
| `cassettevision` | Cassette Vision | console |  | `.bin777` `.zip` | batocera |
| `catacomb` | CatacombGL | port |  | `.game` | batocera |
| `cave3rd` | Cave CV1000 | arcade | `cave` | `.7z` `.zip` | conservative |
| `cavestory` | Cave Story | port |  | `.exe` | batocera |
| `cdimono1` | Philips CD-i | console | `cdi` | `.chd` `.cue` `.iso` | es-de |
| `cdogs` | C-Dogs SDL | port |  | `.game` | batocera |
| `cdtv` | Commodore CDTV | console | `amigacdtv` | `.7z` `.adf` `.adz` `.ccd` `.chd` `.cue` `.dms` `.fdi` `.hdf` `.hdz` `.ipf` `.iso` `.lha` `.m3u` `.mds` `.nrg` `.rp9` `.uae` `.zip` | es-de |
| `cgenie` | Colour Genie | computer |  | `.7z` `.zip` | conservative |
| `cgenius` | Commander Genius | port |  | `.cgenius` | batocera |
| `chailove` | ChaiLove Game Engine | engine |  | `.7z` `.chai` `.chailove` `.zip` | es-de |
| `channelf` | Fairchild Channel F | console | `fairchild` | `.7z` `.bin` `.chf` `.zip` | es-de |
| `chihiro` | Sega Chihiro | arcade |  | `.bin` | batocera |
| `coco` | Tandy Color Computer | computer | `Color Computer` | `.cas` `.ccc` `.dsk` `.rom` | es-de |
| `colecovision` | Coleco ColecoVision | console | `coleco` | `.7z` `.bin` `.cas` `.col` `.cv` `.dsk` `.m3u` `.mx1` `.mx2` `.myv` `.ri` `.rom` `.sc` `.sg` `.zip` | es-de |
| `commanderx16` | Commander X16 | computer |  | `.bas` `.img` `.prg` | batocera |
| `consolearcade` | Console Arcade Systems | arcade |  | `.7z` `.arcadedef` `.desktop` `.iso` `.ps3` `.sh` `.xbe` `.zip` | es-de |
| `corsixth` | CorsixTH | port |  | `.game` | batocera |
| `cps` | Capcom Play System | arcade |  | `.7z` `.zip` | es-de |
| `cps1` | Capcom Play System I | arcade |  | `.7z` `.zip` | es-de |
| `cps2` | Capcom Play System II | arcade |  | `.7z` `.zip` | es-de |
| `cps3` | Capcom Play System III | arcade |  | `.7z` `.zip` | es-de |
| `crvision` | VTech CreatiVision | console | `CreatiVision` | `.7z` `.bin` `.rom` `.zip` | es-de |
| `ctvboy` | Compact Vision TV Boy | console |  | `.7z` `.zip` | conservative |
| `daphne` | Daphne Arcade LaserDisc Emulator | arcade |  | `.7z` `.daphne` `.dirksimple` `.mmi` `.ogv` `.singe` `.zip` | es-de |
| `desktop` | Desktop Applications | other |  | `.desktop` `.sh` | es-de |
| `devilutionx` | Diablo | port |  | `.mpq` | batocera |
| `dice` | DICE | arcade |  | `.dmy` `.zip` | batocera |
| `doom` | Doom | port | `prboom` `uzdoom` `gzdoom` | `.desktop` `.ipk3` `.iwad` `.pk3` `.pk4` `.pwad` `.sh` `.wad` | es-de |
| `doom3` | Doom 3 | port | `boom3` | `.d3` | batocera |
| `dos` | DOS (PC) | computer | `Dos (x86)` | `.7z` `.bat` `.com` `.conf` `.cue` `.dosz` `.exe` `.img` `.iso` `.zip` | es-de |
| `dragon32` | Dragon Data Dragon 32 | computer |  | `.7z` `.cas` `.ccc` `.dsk` `.rom` `.zip` | es-de |
| `dragon64` | Dragon 64 | computer |  | `.7z` `.zip` | conservative |
| `dreamcast` | Sega Dreamcast | console |  | `.7z` `.cdi` `.chd` `.cue` `.dat` `.elf` `.gdi` `.iso` `.lst` `.m3u` `.zip` | es-de |
| `dxx-rebirth` | DXX Rebirth | port |  | `.d1x` `.d2x` | batocera |
| `easyrpg` | EasyRPG Game Engine | engine |  | `.easyrpg` `.zip` | es-de |
| `ecwolf` | ECWolf | port |  | `.ecwolf` `.pk3` `.squashfs` | batocera |
| `eduke32` | EDuke32 | port |  | `.7z` `.zip` | conservative |
| `electron` | Acorn Electron | computer |  | `.1dd` `.7z` `.adf` `.adl` `.adm` `.ads` `.bbc` `.bin` `.cqi` `.cqm` `.csw` `.d77` `.d88` `.dfi` `.dsd` `.dsk` `.hfe` `.imd` `.img` `.mfi` `.mfm` `.rom` `.ssd` `.td0` `.uef` `.wav` `.zip` | es-de |
| `emulators` | Emulators | other |  | `.desktop` `.sh` | es-de |
| `enterprise` | Enterprise | computer |  | `.7z` `.zip` | conservative |
| `epic` | Epic Games Store | other |  | `.desktop` `.sh` | es-de |
| `etlegacy` | Wolfenstein - Enemy Territory | port |  | `.etl` | batocera |
| `fallout1-ce` | Fallout Community Edition | port |  | `.f1ce` | batocera |
| `fallout2-ce` | Fallout 2 Community Edition | port |  | `.f2ce` | batocera |
| `fba` | FinalBurn Alpha | arcade |  | `.7z` `.iso` `.zip` | es-de |
| `fbneo` | FinalBurn Neo | arcade |  | `.7z` `.zip` | es-de |
| `fds` | Nintendo Famicom Disk System | console | `Family Computer Disk System` | `.7z` `.fds` `.nes` `.unf` `.unif` `.zip` | es-de |
| `flash` | Adobe Flash | port | `Flash Player` | `.ruf` `.swf` | es-de |
| `fm7` | Fujitsu FM-7 | computer |  | `.1dd` `.7z` `.cqi` `.cqm` `.d77` `.d88` `.dfi` `.dsk` `.hfe` `.imd` `.mfi` `.mfm` `.t77` `.td0` `.wav` `.zip` | es-de |
| `fmtowns` | Fujitsu FM Towns | computer |  | `.cdr` `.chd` `.cue` `.gdi` `.iso` | es-de |
| `fpinball` | Future Pinball | other |  | `.fpt` | es-de |
| `fury` | Ion Fury | port |  | `.7z` `.zip` | conservative |
| `gaelco` | Gaelco | arcade |  | `.7z` `.zip` | conservative |
| `gamate` | Bit Corporation Gamate | handheld |  | `.7z` `.bin` `.zip` | es-de |
| `gameandwatch` | Nintendo Game and Watch | handheld | `gw` | `.7z` `.mgw` `.zip` | es-de |
| `gamecom` | Tiger Electronics Game.com | handheld |  | `.7z` `.tgc` `.zip` | es-de |
| `gamegear` | Sega Game Gear | handheld | `gg` | `.68k` `.7z` `.bin` `.bms` `.chd` `.col` `.cue` `.gen` `.gg` `.iso` `.m3u` `.md` `.mdx` `.rom` `.sg` `.sgd` `.smd` `.sms` `.zip` | es-de |
| `gamepock` | Game Pocket Computer | handheld |  | `.7z` `.zip` | conservative |
| `gametank` | GameTank | console |  | `.gtr` | batocera |
| `gb` | Game Boy | handheld |  | `.7z` `.bs` `.cgb` `.dmg` `.gb` `.gbc` `.gbx` `.sfc` `.sgb` `.smc` `.zip` | es-de |
| `gb2players` | Game Boy (2 players) | handheld |  | `.7z` `.bs` `.cgb` `.dmg` `.gb` `.gbc` `.gbx` `.sfc` `.sgb` `.smc` `.zip` | conservative |
| `gba` | Game Boy Advance | handheld |  | `.7z` `.agb` `.bin` `.cgb` `.dmg` `.gb` `.gba` `.gbc` `.gbx` `.sgb` `.zip` | es-de |
| `gba2players` | Game Boy Advance (2 players) | handheld |  | `.7z` `.agb` `.bin` `.cgb` `.dmg` `.gb` `.gba` `.gbc` `.gbx` `.sgb` `.zip` | conservative |
| `gbc` | Game Boy Color | handheld |  | `.7z` `.bs` `.cgb` `.dmg` `.gb` `.gbc` `.gbx` `.sfc` `.sgb` `.smc` `.zip` | es-de |
| `gbc2players` | Game Boy Color (2 players) | handheld |  | `.7z` `.bs` `.cgb` `.dmg` `.gb` `.gbc` `.gbx` `.sfc` `.sgb` `.smc` `.zip` | conservative |
| `gc` | Nintendo GameCube | console | `gamecube` `ngc` | `.7z` `.ciso` `.dff` `.dol` `.elf` `.gcm` `.gcz` `.iso` `.json` `.m3u` `.rvz` `.tgc` `.wad` `.wbfs` `.wia` `.zip` | es-de |
| `gemrb` | GemRB Game Engine | engine |  | `.7z` `.zip` | conservative |
| `genesis` | Sega Genesis / Mega Drive | console | `mega drive` `md` `Sega Mega Drive` `megadrivejp` | `.32x` `.68k` `.7z` `.bin` `.bms` `.chd` `.cue` `.gen` `.gg` `.iso` `.m3u` `.md` `.mdx` `.sg` `.sgd` `.smd` `.sms` `.zip` | es-de |
| `gmaster` | Hartung Game Master | handheld | `Game Master` | `.7z` `.bin` `.zip` | es-de |
| `gong` | Pong | console |  | `.7z` `.zip` | conservative |
| `gp32` | GP32 | handheld |  | `.7z` `.smc` `.zip` | batocera |
| `gx4000` | Amstrad GX4000 | console |  | `.7z` `.bin` `.cdt` `.cpr` `.dsk` `.kcr` `.m3u` `.sna` `.tap` `.tar` `.voc` `.zip` | es-de |
| `halflife` | Half-Life 1 | port |  | `.game` | batocera |
| `hbmame` | HBMAME (homebrew MAME) | arcade |  | `.7z` `.zip` | conservative |
| `hcl` | Hydra Castle Labyrinth | port |  | `.game` | batocera |
| `hikaru` | Hikaru | arcade |  | `.7z` `.zip` | conservative |
| `hurrican` | Hurrican | port |  | `.game` | batocera |
| `ikemen` | IKEMEN | port |  | `.ikemen` `.pc` | batocera |
| `intellivision` | Mattel Electronics Intellivision | console | `Mattel Intellivision` | `.7z` `.bin` `.int` `.rom` `.zip` | es-de |
| `j2me` | Java 2 Micro Edition (J2ME) | handheld | `Java 2 MicroEdition` | `.7z` `.jar` `.zip` | es-de |
| `jazz2` | Jazz Jackrabbit 2 | port |  | `.game` | batocera |
| `jkdf2` | Jedi Knight - Dark Forces 2 | port |  | `.jedi` | batocera |
| `jknight` | Star Wars - Jedi Academy | port |  | `.jedi` | batocera |
| `karaoke` | Karaoke | other |  | `.7z` `.zip` | conservative |
| `kodi` | Kodi Home Theatre Software | other |  | `.desktop` `.sh` | es-de |
| `laser310` | Laser 310 | computer |  | `.7z` `.zip` | conservative |
| `laserdisc` | LaserDisc Games | arcade |  | `.7z` `.daphne` `.dirksimple` `.mmi` `.ogv` `.singe` `.zip` | es-de |
| `lcdgames` | LCD Handheld Games | handheld |  | `.7z` `.mgw` `.zip` | es-de |
| `lindbergh` | Sega Lindbergh | arcade |  | `.game` | batocera |
| `loopy` | Casio Loopy | console | `casloopy` | `.7z` `.zip` | conservative |
| `love` | LÖVE Game Engine | engine |  | `.love` | conservative |
| `lowresnx` | LowRes NX Fantasy Console | console |  | `.nx` | es-de |
| `lutris` | Lutris Open Gaming Platform | other |  | `.desktop` `.sh` | es-de |
| `lutro` | Lutro Game Engine | engine |  | `.7z` `.lua` `.lutro` `.zip` | es-de |
| `macintosh` | Apple Macintosh | computer | `vmac` | `.dsk` `.game` `.img` | es-de |
| `mame` | Multiple Arcade Machine Emulator | arcade | `mame2010` | `.7z` `.cmd` `.desktop` `.neo` `.sh` `.zip` | es-de |
| `mame-advmame` | AdvanceMAME | arcade |  | `.7z` `.zip` | es-de |
| `mastersystem` | Sega Master System | console | `sms` `mark3` `Sega Mark III` `ms` | `.68k` `.7z` `.bin` `.bms` `.chd` `.col` `.cue` `.gen` `.gg` `.iso` `.m3u` `.md` `.mdx` `.rom` `.sg` `.sgd` `.smd` `.sms` `.zip` | es-de |
| `mc10` | MC-10 | computer |  | `.7z` `.zip` | conservative |
| `megadrive-msu` | MSU-MD | console |  | `.32x` `.68k` `.7z` `.bin` `.bms` `.chd` `.cue` `.gen` `.gg` `.iso` `.m3u` `.md` `.mdx` `.sg` `.sgd` `.smd` `.sms` `.zip` | conservative |
| `megaduck` | Creatronic Mega Duck | handheld | `Mega Duck / Cougar Boy` | `.7z` `.bin` `.zip` | es-de |
| `mess` | Multi Emulator Super System | other |  | `.7z` `.chd` `.zip` | es-de |
| `model2` | Sega Model 2 | arcade |  | `.7z` `.zip` | es-de |
| `model3` | Sega Model 3 | arcade |  | `.7z` `.zip` | es-de |
| `mohaa` | Medal Of Honor - Allied Assault | port |  | `.mohaa` | batocera |
| `moto` | Thomson MO/TO Series | computer | `thomson` `Thomson - MO/TO (Theodore)` | `.7z` `.fd` `.k7` `.m5` `.m7` `.rom` `.sap` `.zip` | es-de |
| `mrboom` | MrBoom | port |  | `.libretro` | batocera |
| `msx` | MSX | computer | `msx1` | `.7z` `.cas` `.col` `.di1` `.di2` `.dmk` `.dsk` `.fd1` `.fd2` `.m3u` `.mx1` `.mx2` `.ogv` `.ri` `.rom` `.sc` `.sg` `.wav` `.xsa` `.zip` | es-de |
| `msx2` | MSX2 | computer |  | `.7z` `.cas` `.col` `.di1` `.di2` `.dmk` `.dsk` `.fd1` `.fd2` `.m3u` `.mx1` `.mx2` `.ogv` `.ri` `.rom` `.sc` `.sg` `.wav` `.xsa` `.zip` | es-de |
| `msxturbor` | MSX Turbo R | computer |  | `.7z` `.cas` `.col` `.di1` `.di2` `.dmk` `.dsk` `.fd1` `.fd2` `.m3u` `.mx1` `.mx2` `.ogv` `.ri` `.rom` `.sc` `.sg` `.wav` `.xsa` `.zip` | es-de |
| `mugen` | M.U.G.E.N Game Engine | engine |  | `.mugen` | es-de |
| `multivision` | Othello Multivision | console |  | `.7z` `.bin` `.gg` `.rom` `.sg` `.sms` `.zip` | es-de |
| `mz2000` | Sharp MZ-2000 | computer |  | `.7z` `.zip` | conservative |
| `mz2500` | Sharp MZ-2500 | computer |  | `.7z` `.zip` | conservative |
| `mz700` | Sharp MZ-700 | computer |  | `.7z` `.zip` | conservative |
| `mz800` | Sharp MZ-800 | computer |  | `.7z` `.zip` | conservative |
| `mz80k` | Sharp MZ-80K | computer |  | `.7z` `.zip` | conservative |
| `n3ds` | Nintendo 3DS | handheld | `3ds` | `.3ds` `.3dsx` `.7z` `.app` `.axf` `.cci` `.cxi` `.elf` `.zip` | es-de |
| `n64` | Nintendo 64 | console |  | `.7z` `.bin` `.d64` `.n64` `.ndd` `.u1` `.v64` `.z64` `.zip` | es-de |
| `n64dd` | Nintendo 64DD | console | `Nintendo 64 Disk Drive` | `.7z` `.bin` `.d64` `.n64` `.ndd` `.u1` `.v64` `.z64` `.zip` | es-de |
| `namco22` | Namco System 22 | arcade |  | `.7z` `.zip` | conservative |
| `namco2x6` | Namco System 246/256 | arcade |  | `.acgame` | batocera |
| `namco3xx` | Namco 3xx | arcade |  | `.squashfs` | batocera |
| `naomi` | Sega NAOMI | arcade |  | `.7z` `.bin` `.dat` `.elf` `.lst` `.zip` | es-de |
| `naomi2` | Sega NAOMI 2 | arcade |  | `.7z` `.bin` `.dat` `.elf` `.lst` `.zip` | es-de |
| `naomigd` | Sega NAOMI GD-ROM | arcade |  | `.7z` `.bin` `.dat` `.elf` `.lst` `.zip` | es-de |
| `nds` | Nintendo DS | handheld | `ds` | `.7z` `.app` `.bin` `.dsi` `.ids` `.nds` `.zip` | es-de |
| `neogeo` | SNK Neo Geo | console |  | `.7z` `.neo` `.zip` | es-de |
| `neogeo64` | SNK Hyper Neo Geo 64 | arcade |  | `.7z` `.zip` | conservative |
| `neogeocd` | SNK Neo Geo CD | console | `neogeocdjp` `neocd` | `.chd` `.cue` | es-de |
| `nes` | Nintendo Entertainment System | console | `famicom` `fc` `Nintendo Family Computer` | `.7z` `.fds` `.nes` `.unf` `.unif` `.zip` | es-de |
| `nes3d` | NES in 3D (3dSen) | other |  | `.3dsen` `.7z` `.nes` `.zip` | conservative |
| `ngage` | Nokia N-Gage | handheld |  | `.ngage` `.zip` | es-de |
| `ngp` | SNK Neo Geo Pocket | handheld | `Neo-Geo Pocket` | `.7z` `.ngc` `.ngp` `.ngpc` `.npc` `.zip` | es-de |
| `ngpc` | SNK Neo Geo Pocket Color | handheld | `Neo-Geo Pocket Color` | `.7z` `.ngc` `.ngp` `.ngpc` `.npc` `.zip` | es-de |
| `odcommander` | OD-Commander | port |  | `.odc` | batocera |
| `odyssey2` | Magnavox Odyssey 2 | console | `videopac` `Philips Videopac G7000` `odyssey` `o2em` | `.7z` `.bin` `.zip` | es-de |
| `onscripter` | ONScripter Game Engine | engine | `ons` | `.7z` `.zip` | conservative |
| `openbor` | OpenBOR Game Engine | engine |  | `.7z` `.pak` `.zip` | es-de |
| `opengoal` | OpenGOAL | port |  | `.iso` `.squashfs` | batocera |
| `openjazz` | Jazz Jackrabbit | port |  | `.game` | batocera |
| `openlara` | OpenLara | port |  | `.7z` `.zip` | conservative |
| `oric` | Tangerine Computer Systems Oric | computer | `oricatmos` | `.dsk` `.ort` `.tap` `.wav` | es-de |
| `palm` | Palm OS | handheld |  | `.7z` `.img` `.pqa` `.prc` `.zip` | es-de |
| `pc` | IBM PC | computer |  | `.7z` `.bat` `.com` `.conf` `.cue` `.dosz` `.exe` `.img` `.iso` `.zip` | es-de |
| `pc60` | PC-6000 | computer |  | `.7z` `.zip` | conservative |
| `pc80` | PC-8001 | computer |  | `.7z` `.zip` | conservative |
| `pc88` | NEC PC-8800 Series | computer | `PC-8800` | `.88d` `.cmt` `.d88` `.m3u` `.t88` `.u88` | es-de |
| `pc98` | NEC PC-9800 Series | computer | `PC-9800` | `.2hd` `.7z` `.88d` `.98d` `.cmd` `.d88` `.d98` `.dup` `.fdd` `.fdi` `.hdd` `.hdi` `.hdm` `.hdn` `.m3u` `.nhd` `.tfd` `.thd` `.xdf` `.zip` | es-de |
| `pcarcade` | PC Arcade Systems | arcade |  | `.desktop` `.sh` | es-de |
| `pcengine` | NEC PC Engine / TurboGrafx-16 | console | `turbografx` `turbografx-16` `tg16` `NEC TurboGrafx-16` `pce` | `.7z` `.ccd` `.chd` `.cue` `.img` `.iso` `.m3u` `.pce` `.rom` `.sgx` `.toc` `.zip` | es-de |
| `pcenginecd` | NEC PC Engine CD | console | `tg-cd` `NEC TurboGrafx-CD` `pcecd` | `.7z` `.ccd` `.chd` `.cue` `.img` `.iso` `.m3u` `.pce` `.sgx` `.toc` `.zip` | es-de |
| `pcfx` | NEC PC-FX | console |  | `.7z` `.ccd` `.chd` `.cue` `.m3u` `.toc` `.zip` | es-de |
| `pcw` | Amstrad PCW | computer |  | `.7z` `.zip` | conservative |
| `pdp1` | PDP-1 | computer |  | `.7z` `.zip` | conservative |
| `pet` | Commodore PET | computer |  | `.7z` `.a0` `.b0` `.crt` `.d64` `.d81` `.m3u` `.prg` `.t64` `.tap` `.zip` | batocera |
| `pgm` | IGS PolyGame Master | arcade |  | `.7z` `.zip` | conservative |
| `pgm2` | IGS PolyGame Master 2 | arcade |  | `.7z` `.zip` | conservative |
| `pico` | Sega Pico | console |  | `.7z` `.zip` | conservative |
| `pico8` | PICO-8 Fantasy Console | console |  | `.p8` | es-de |
| `plus4` | Commodore Plus/4 | computer | `cplus4` | `.7z` `.bin` `.cmd` `.crt` `.d2m` `.d4m` `.d64` `.d6z` `.d71` `.d7z` `.d80` `.d81` `.d82` `.d8z` `.g41` `.g4z` `.g64` `.g6z` `.gz` `.lnx` `.m3u` `.nbz` `.nib` `.p00` `.prg` `.t64` `.tap` `.vfl` `.vsf` `.x64` `.x6z` `.zip` | es-de |
| `pokemini` | Nintendo Pokémon Mini | handheld | `Pokemon Mini` `poke` | `.7z` `.min` `.zip` | es-de |
| `ports` | Ports | port |  | `.desktop` `.exe` `.game` `.phd` `.psx` `.sh` | es-de |
| `ps2` | Sony PlayStation 2 | console | `PlayStation 2` | `.bin` `.chd` `.ciso` `.cso` `.dump` `.elf` `.gz` `.img` `.iso` `.isz` `.m3u` `.mdf` `.ngr` `.nrg` `.zso` | es-de |
| `ps3` | Sony PlayStation 3 | console | `PlayStation 3` | `.desktop` `.iso` `.ps3` `.ps3dir` | es-de |
| `ps4` | Sony PlayStation 4 | console | `PlayStation 4` | `.7z` `.zip` | es-de |
| `psp` | Sony PlayStation Portable | handheld | `PlayStation Portable` `pspminis` | `.7z` `.chd` `.cso` `.elf` `.iso` `.pbp` `.prx` `.zip` | es-de |
| `psvita` | Sony PlayStation Vita | handheld | `PlayStation Vita` | `.psvita` | es-de |
| `psx` | Sony PlayStation | console | `playstation` `ps1` `psone` `ps` | `.7z` `.bin` `.cbn` `.ccd` `.chd` `.cue` `.ecm` `.exe` `.img` `.iso` `.m3u` `.mdf` `.mds` `.minipsf` `.pbp` `.psexe` `.psf` `.toc` `.z` `.zip` `.znx` | es-de |
| `pv1000` | Casio PV-1000 | console |  | `.7z` `.bin` `.zip` | es-de |
| `pv2000` | PV-2000 | console |  | `.7z` `.zip` | conservative |
| `pygame` | Pygame | engine |  | `.pygame` | batocera |
| `pyxel` | pyxel | console |  | `.py` `.pyxapp` | batocera |
| `quake` | Quake | port | `tyrquake` | `.desktop` `.pak` `.pk3` `.sh` | es-de |
| `quake2` | Quake II | port | `vitaquake2` | `.7zip` `.quake2` `.zip` | batocera |
| `quake3` | Quake III | port |  | `.quake3` | batocera |
| `raze` | Raze | port |  | `.raze` | batocera |
| `reminiscence` | REminiscence | port |  | `.rem` | batocera |
| `rott` | Rise of the Triad | port |  | `.rott` | batocera |
| `rtcw` | Return To Castle Wolfenstein | port |  | `.rtcw` | batocera |
| `rx78` | RX-78 | computer |  | `.7z` `.zip` | conservative |
| `samcoupe` | MGT SAM Coupé | computer |  | `.7z` `.dsk` `.mgt` `.sad` `.sbt` `.zip` | es-de |
| `satellaview` | Nintendo Satellaview | console |  | `.7z` `.bml` `.bs` `.fig` `.sfc` `.smc` `.st` `.swc` `.zip` | es-de |
| `saturn` | Sega Saturn | console | `saturnjp` | `.7z` `.bin` `.ccd` `.chd` `.cue` `.iso` `.m3u` `.mds` `.toc` `.zip` | es-de |
| `sc3000` | SC-3000 | computer |  | `.7z` `.zip` | conservative |
| `scummvm` | ScummVM Game Engine | engine |  | `.scummvm` `.svm` | es-de |
| `scv` | Epoch Super Cassette Vision | console | `Super Cassette Vision` | `.0` `.7z` `.bin` `.zip` | es-de |
| `sdlpop` | SdlPop | port |  | `.sdlpop` | batocera |
| `sega32x` | Sega Mega Drive 32X | console | `sega32xjp` `Sega Super 32X` `sega32xna` `Sega Genesis 32X` `32x` `thirtytwox` | `.32x` `.68k` `.7z` `.bin` `.chd` `.cue` `.gen` `.iso` `.m3u` `.md` `.smd` `.sms` `.zip` | es-de |
| `segaai` | Sega AI Computer | computer |  | `.7z` `.zip` | conservative |
| `segacd` | Sega CD | console | `megacd` `Sega Mega-CD` `megacdjp` `mdcd` | `.68k` `.7z` `.bin` `.bms` `.chd` `.cue` `.gen` `.gg` `.iso` `.m3u` `.md` `.mdx` `.sg` `.sgd` `.smd` `.sms` `.zip` | es-de |
| `sg-1000` | Sega SG-1000 | console | `segasgone` | `.68k` `.7z` `.bin` `.bms` `.chd` `.cue` `.gen` `.gg` `.iso` `.m3u` `.md` `.mdx` `.ri` `.rom` `.sg` `.sgd` `.smd` `.sms` `.zip` | es-de |
| `sgb` | Nintendo Super Game Boy | console | `Super Game Boy` | `.7z` `.gb` `.gbc` `.sgb` `.zip` | es-de |
| `sgb-msu1` | Super Game Boy MSU1 | console | `gb-msu` | `.7z` `.gb` `.gbc` `.sgb` `.zip` | conservative |
| `singe` | Singe | arcade |  | `.7z` `.zip` | conservative |
| `snes` | Super Nintendo | console | `super famicom` `sfc` `super nes` `Nintendo SFC (Super Famicom)` `snesna` `Nintendo SNES (Super Nintendo)` `Super Nintendo Entertainment System` | `.7z` `.bin` `.bml` `.bs` `.bsx` `.dx2` `.fig` `.gd3` `.gd7` `.mgd` `.sfc` `.smc` `.st` `.swc` `.zip` | es-de |
| `snes-msu1` | Super Disc System (MSU1) | console | `snes-msu` `sfcmsu` `msu1` | `.7z` `.bin` `.bml` `.bs` `.bsx` `.dx2` `.fig` `.gd3` `.gd7` `.mgd` `.sfc` `.smc` `.st` `.swc` `.zip` | conservative |
| `socrates` | Socrates | console |  | `.7z` `.zip` | conservative |
| `solarus` | Solarus Game Engine | engine |  | `.solarus` | es-de |
| `sonic-mania` | Sonic Mania | port |  | `.sman` | batocera |
| `sonic3-air` | Sonic 3 A.I.R. | port |  | `.s3air` | batocera |
| `sonicretro` | Sonic Retro Engine | port |  | `.scd` `.son` | batocera |
| `spectravideo` | Spectravideo | computer | `Spectravideo SV-328` | `.7z` `.cas` `.col` `.dsk` `.m3u` `.mx1` `.mx2` `.ri` `.rom` `.sc` `.sg` `.zip` | es-de |
| `steam` | Valve Steam | other |  | `.desktop` `.sh` | es-de |
| `stv` | Sega Titan Video Game System | arcade | `segastv` | `.7z` `.zip` | es-de |
| `sufami` | Bandai SuFami Turbo | console | `SuFami Turbo` | `.7z` `.bml` `.bs` `.fig` `.sfc` `.smc` `.st` `.zip` | es-de |
| `superbroswar` | Super Mario War | port |  | `.game` | batocera |
| `supergrafx` | NEC SuperGrafx | console | `sgfx` | `.7z` `.ccd` `.chd` `.cue` `.pce` `.rom` `.sgx` `.zip` | es-de |
| `supervision` | Watara Supervision | handheld |  | `.7z` `.bin` `.sv` `.zip` | es-de |
| `supracan` | Funtech Super A'Can | console | `Super A'Can` | `.7z` `.bin` `.zip` | es-de |
| `sv8000` | Super Vision 8000 | console |  | `.7z` `.zip` | conservative |
| `switch` | Nintendo Switch | console |  | `.nca` `.nro` `.nso` `.nsp` `.xci` | es-de |
| `symbian` | Symbian | handheld |  | `.sis` `.sisx` `.symbian` | es-de |
| `systemsp` | Sega System SP | arcade |  | `.7z` `.bin` `.dat` `.lst` `.zip` | batocera |
| `tanodragon` | Tano Dragon | computer |  | `.7z` `.cas` `.ccc` `.dsk` `.rom` `.zip` | es-de |
| `teknoparrot` | TeknoParrot | arcade |  | `.7z` `.zip` | conservative |
| `theforceengine` | The Force Engine | port |  | `.tfe` | batocera |
| `thextech` | TheXTech | port |  | `.smbx` `.squashfs` | batocera |
| `ti83` | Texas Instruments TI-83 | computer |  | `.7z` `.8xp` `.zip` | conservative |
| `ti99` | Texas Instruments TI-99 | computer |  | `.7z` `.rpk` `.zip` | es-de |
| `tic80` | TIC-80 Fantasy Computer | console | `tic` | `.tic` | es-de |
| `to8` | Thomson TO8 | computer |  | `.7z` `.fd` `.k7` `.m5` `.m7` `.rom` `.sap` `.zip` | es-de |
| `traider` | Tomb Raider I, II & III | port |  | `.croft` | batocera |
| `triforce` | Namco-Sega-Nintendo Triforce | arcade |  | `.7z` `.ciso` `.dff` `.dol` `.elf` `.gcm` `.gcz` `.iso` `.json` `.m3u` `.rvz` `.tgc` `.wad` `.wbfs` `.wia` `.zip` | es-de |
| `trs-80` | Tandy TRS-80 | computer |  | `.cmd` `.dsk` | es-de |
| `tutor` | Tutor | computer |  | `.7z` `.zip` | conservative |
| `tvc` | Videoton TVC | computer | `videoton` | `.cas` `.dsk` `.img` `.tap` `.zip` | batocera |
| `tvgames` | Plug and Play TV Games | console |  | `.7z` `.zip` | conservative |
| `type-x` | Taito Type X | arcade |  | `.desktop` `.sh` | es-de |
| `tyrian` | Tyrian | port |  | `.game` | batocera |
| `uqm` | Ur-Quan Masters | port |  | `.game` | batocera |
| `uzebox` | Uzebox Open Source Console | console |  | `.7z` `.uze` `.zip` | es-de |
| `vc4000` | VC 4000 | console |  | `.7z` `.zip` | conservative |
| `vectrex` | GCE Vectrex | console |  | `.7z` `.bin` `.gam` `.vc` `.vec` `.zip` | es-de |
| `vemulator` | Dreamcast VMU | console |  | `.bin` `.dci` `.vms` | batocera |
| `vic20` | Commodore VIC-20 | computer | `c20` | `.7z` `.a0` `.b0` `.bin` `.cmd` `.crt` `.d2m` `.d4m` `.d64` `.d6z` `.d71` `.d7z` `.d80` `.d81` `.d82` `.d8z` `.g41` `.g4z` `.g64` `.g6z` `.gz` `.lnx` `.m3u` `.nbz` `.nib` `.p00` `.prg` `.rom` `.t64` `.tap` `.vfl` `.vsf` `.x64` `.x6z` `.zip` | es-de |
| `videopacplus` | Videopac+ G7400 | console |  | `.7z` `.bin` `.zip` | batocera |
| `vircon32` | Vircon32 Virtual Console | console |  | `.7z` `.v32` `.zip` | es-de |
| `virtualboy` | Nintendo Virtual Boy | console | `vb` | `.7z` `.bin` `.vb` `.vboy` `.zip` | es-de |
| `vis` | Tandy Video Information System | computer |  | `.7z` `.zip` | conservative |
| `vpinball` | Visual Pinball | other | `Visual Pinball X` | `.vpt` `.vpx` | es-de |
| `vsmile` | VTech V.Smile | console |  | `.7z` `.bin` `.zip` | es-de |
| `wasm4` | WASM-4 Fantasy Console | console |  | `.7z` `.wasm` `.zip` | es-de |
| `wii` | Nintendo Wii | console |  | `.7z` `.ciso` `.dff` `.dol` `.elf` `.gcm` `.gcz` `.iso` `.json` `.m3u` `.rvz` `.tgc` `.wad` `.wbfs` `.wia` `.zip` | es-de |
| `wiiu` | Nintendo Wii U | console |  | `.elf` `.rpx` `.tmd` `.wua` `.wud` `.wuhb` `.wux` | es-de |
| `windows` | Microsoft Windows | computer |  | `.desktop` `.sh` | es-de |
| `windows3x` | Microsoft Windows 3.x | computer |  | `.7z` `.bat` `.desktop` `.dosz` `.sh` `.zip` | es-de |
| `windows9x` | Microsoft Windows 9x | computer |  | `.7z` `.bat` `.desktop` `.dosz` `.sh` `.zip` | es-de |
| `wonderswan` | Bandai WonderSwan | handheld | `wswan` `ws` | `.7z` `.pc2` `.ws` `.wsc` `.zip` | es-de |
| `wonderswancolor` | Bandai WonderSwan Color | handheld | `wswanc` `wsc` | `.7z` `.pc2` `.ws` `.wsc` `.zip` | es-de |
| `x1` | Sharp X1 | computer |  | `.2d` `.2hd` `.7z` `.88d` `.cmd` `.d88` `.dup` `.dx1` `.hdm` `.tap` `.tfd` `.xdf` `.zip` | es-de |
| `x68000` | Sharp X68000 | computer |  | `.2hd` `.7z` `.88d` `.cmd` `.d88` `.dim` `.dup` `.hdf` `.hdm` `.img` `.m3u` `.xdf` `.zip` | es-de |
| `xbox` | Microsoft Xbox | console |  | `.iso` `.xiso` | es-de |
| `xbox360` | Microsoft Xbox 360 | console |  | `.7z` `.zip` | es-de |
| `xboxone` | Microsoft Xbox One | console |  | `.7z` `.zip` | es-de |
| `xrick` | xrick | port |  | `.zip` | batocera |
| `zaccariapinball` | Zaccaria Pinball | other |  | `.7z` `.zip` | conservative |
| `zc210` | Zelda Classic | console |  | `.qst` | batocera |
| `zinc` | Sony ZN (ZiNc) | arcade |  | `.7z` `.zip` | conservative |
| `zmachine` | Infocom Z-machine | engine |  | `.dat` `.z1` `.z2` `.z3` `.z4` `.z5` `.z6` `.z7` `.z8` `.zblorb` `.zlb` | es-de |
| `zx81` | Sinclair ZX81 | computer |  | `.7z` `.p` `.tzx` `.zip` | es-de |
| `zxnext` | Sinclair ZX Spectrum Next | computer |  | `.nex` `.sna` | es-de |
| `zxspectrum` | Sinclair ZX Spectrum | computer | `zxs` | `.7z` `.dsk` `.gz` `.img` `.mgt` `.rzx` `.scl` `.sh` `.sna` `.szx` `.tap` `.trd` `.tzx` `.udi` `.z80` `.zip` | es-de |
