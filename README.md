# Kitbash-Mods

The official mod catalog of the [Kitbash](https://github.com/ImKirit/Kitbash) launcher.
Every installed launcher reads `catalog.json` from this repository on start and every
30 minutes, so a mod that is added here shows up everywhere by itself.

Anyone can run a catalog of their own in the same format. Users add it in the launcher
under `Sources`. Such a source is shown as **unofficial**, with a warning, because a mod
is program code that runs inside the game with the rights of the player.

## Layout

```
catalog.json                          the list the launcher reads
mods/<game>/<mod>/mod.json            description of a mod, input for the publish script
mods/<game>/<mod>/<mod>-<version>.zip the package the launcher downloads
images/mods/<mod>/                    pictures shown on the mod page
```

## catalog.json

```json
{
  "schemaVersion": 1,
  "updated": "2026-10-11T18:00:00.000Z",
  "games": [
    {
      "id": "repo",
      "name": "R.E.P.O.",
      "status": "available",
      "tagline": { "en": "Co-op horror with physics.", "de": "Koop-Horror mit Physik." },
      "platform": "steam",
      "steam": { "appId": 3241660 },
      "executable": "REPO.exe",
      "processName": "REPO.exe",
      "engine": "unity-mono",
      "antiCheat": false
    }
  ],
  "mods": [
    {
      "id": "repoportalgun",
      "game": "repo",
      "name": { "en": "Portal Gun", "de": "Portalkanone" },
      "version": "0.1.0",
      "author": "ImKirit",
      "status": "experimental",
      "released": "2026-10-11",
      "summary": { "en": "Two linked portals.", "de": "Zwei verbundene Portale." },
      "description": { "en": "Longer text. Blank lines make paragraphs.", "de": "..." },
      "images": ["images/mods/repoportalgun/shot.png"],
      "loader": "bepinex5-x64",
      "minLauncherVersion": "0.1.0",
      "requirements": [
        { "type": "minecraft-java", "minVersion": "1.20.2", "level": "required", "text": { "en": "..." } },
        { "type": "steam-game", "appId": 3241660, "name": "R.E.P.O.", "level": "recommended", "text": { "en": "..." } }
      ],
      "notes": [
        { "kind": "multiplayer", "text": { "en": "Everyone in the lobby needs this mod in the same version." } }
      ],
      "controls": [ { "key": "G", "text": { "en": "Draw or put away the gun." } } ],
      "conflicts": [],
      "thirdParty": [ { "name": "Some library 1.0", "license": "MIT", "url": "https://example.org" } ],
      "persist": ["BepInEx/config/dev.imkirit.repoportalgun.cfg"],
      "package": { "url": "mods/repo/repoportalgun/repoportalgun-0.1.0.zip", "size": 29025, "sha256": "..." },
      "files": [
        { "path": "BepInEx/plugins/RepoPortalGun/RepoPortalGun.dll", "size": 60928, "sha256": "..." }
      ]
    }
  ]
}
```

### Games

| Field | Meaning |
|---|---|
| `id` | lowercase letters, digits and `-`, 2 to 40 characters |
| `status` | `available` or `coming-later` (shown, cannot be launched, needs only `id`, `name`, `status`) |
| `platform`, `steam.appId` | only `steam` is supported, the launcher finds the install through the app manifest |
| `executable`, `processName` | the exe in the game folder and the process the launcher waits for |
| `engine` | `unity-mono` is the only engine with a loader today |
| `antiCheat` | must be `false`. Games with anti-cheat are not accepted, and the launcher also refuses a game folder that contains anti-cheat files |

### Mods

| Field | Meaning |
|---|---|
| `name`, `summary`, `description`, texts | a string or `{ "en": "...", "de": "..." }`, English is required |
| `status` | `stable`, `beta` or `experimental` |
| `loader` | a loader the launcher knows. Today: `bepinex5-x64` (BepInEx 5.4.23.5). The launcher downloads it from the official BepInEx release with a fixed hash. A `loaders` section in an unofficial catalog is ignored |
| `requirements` | checked on the player's PC. `steam-game` (installed), `minecraft-java` (optional `minVersion`). `level` is `required` (blocks the launch) or `recommended` |
| `notes` | free text shown on the mod page, `kind` is `multiplayer`, `privacy`, `testing` or `info` |
| `conflicts` | ids of mods of the same catalog that cannot be combined with this one |
| `persist` | files or folders, relative to the game folder, that hold the mod's settings and saved data. Kept by the launcher between sessions. For BepInEx they must be below `BepInEx/config/` or `BepInEx/plugins/` and must not overlap `files` |
| `package` | the zip, its size in bytes and its SHA256 |
| `files` | every file in the zip with path, size and SHA256. For BepInEx all paths must be below `BepInEx/plugins/` or `BepInEx/patchers/`. A zip with a file that is not listed is refused |
| `minLauncherVersion` | older launchers show the mod but do not launch it |

### Rules for package addresses

- A path relative to the catalog (as above), or
- an `https` URL on the same host as the catalog, or
- if the catalog is served from `raw.githubusercontent.com/<owner>/<repo>/...`: an `https` URL
  on `github.com` or `raw.githubusercontent.com` inside that same repository, for example a
  release asset `https://github.com/<owner>/<repo>/releases/download/v1/mod.zip`.

Anything else makes the launcher reject the whole catalog. Images are always relative paths.

### Ids across catalogs

Mods of an unofficial catalog are kept in their own id space, so they can never replace a
mod of this catalog. A game defined under an id this catalog already uses is ignored, the
official definition stays, and the mods attach to it. An unofficial catalog may therefore
list mods for `repo` or `peak` without repeating the game.

## Publishing a mod

The Kitbash repository contains the script. It zips the files, works out sizes and hashes,
writes the entry into `catalog.json`, validates the whole catalog and makes one local commit.
It never pushes.

```
node scripts/publish-mod.js ../Kitbash-Mods/mods/repo/repoportalgun --files "D:\path\to\built\plugin"
```

`mod.json` in the mod folder holds everything except `package`, `files` and `released`, plus:

| Field | Meaning |
|---|---|
| `installTo` | folder inside the game folder the files go to, for example `BepInEx/plugins/RepoPortalGun` |
| `images` | picture files next to `mod.json`, copied to `images/mods/<id>/` |
| `include` | extra files next to `mod.json` that go into the package, for example a license note |

Options: `--repo <folder>` if the mod folder is not inside the catalog repository, `--no-commit`,
`--dry-run`. Files ending in `.pdb`, `.mdb` and `.deps.json` are left out.

To test a catalog before it is public, start the launcher with `--catalog <folder>`.

## The mods here

All of them are untested in multiplayer and marked experimental. Each mod page in the launcher
says what has been checked and what has not.

| Game | Mod | Needs |
|---|---|---|
| R.E.P.O. | Mimics 0.2.0 | everyone in the lobby with the same version, voice recording after consent |
| R.E.P.O. | Portal Gun 0.1.0 | everyone in the lobby with the same version |
| R.E.P.O. | Minecraft in R.E.P.O. 0.1.0 | Minecraft Java 1.20.2 or newer on every player's PC |
| PEAK | Minecraft in PEAK 0.1.0 | Minecraft Java 1.20.2 or newer on every player's PC |
| PEAK | R.E.P.O. Monsters 0.2.0 | R.E.P.O. installed for the real models (optional) |
| PEAK | Among Us in PEAK 0.1.0 | everyone in the lobby with the same version |

R.E.P.O. Monsters ships AssetsTools.NET 3.0.0 and `classdata.tpk` from UABEA, both MIT licensed
by nesrak1. The license text is inside the package.
