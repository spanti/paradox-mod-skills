# CK3 environment and source lookup

Resolve each needed path once per session. Game sources require `<game>`, static validation requires `<tiger>`, and runtime evidence requires `<logs>`; unrelated paths need not block the task. Reuse verified project-local paths across sessions, but keep them out of this portable skill.

| Placeholder | Resolved target | Verification |
|---|---|---|
| `<game>` | `<steam library>/steamapps/common/Crusader Kings III/game` | `<game>/common/` and `<install>/launcher/launcher-settings.json` exist, where `<install>` is the parent of `<game>` |
| `<user>` | CK3's per-user data directory | The directory exists; game-created subdirectories can be checked separately |
| `<logs>` | `<user>/logs` | The directory exists when log evidence is required |
| `<mods>` | `<user>/mod` | The directory exists when launcher descriptors are required; follow a descriptor's `path=` to its source folder |
| `<workshop>` | `<steam library>/steamapps/workshop/content/1158310` | The directory exists when Workshop sources are required |
| `<tiger>` | `ck3-tiger` executable for the agent's actual operating system | Running `<tiger> --version` succeeds |

## Bounded discovery

Make one pass in this order and stop at the first candidate that passes verification. Confine the search to the configured keys and exact locations below. The game and tiger are external tool installations; this skill supplies instructions, not those binaries.

1. Read existing project instructions and session notes. In workspace `.vscode/settings.json` and the editor's user `settings.json`, read only `px.gamePath`, `px.logsPath`, `px.modPath`, `px.tigerPath`, legacy `ck3.gamePath`/`ck3.logsPath`/`ck3.tigerPath`, and `cwtools.cache.ck3`. On Windows the user settings are normally `%APPDATA%/Code/User/settings.json`; on Linux, `~/.config/Code/User/settings.json`. Verify every configured path before using it.
2. Run the applicable operating-system probes below.
3. If anything required remains unresolved, ask once for only those absolute paths: "Please provide the CK3 `game` directory (the one containing `common/`) and/or the `ck3-tiger` executable. If tiger is not installed, say so; I can continue with source checks." Continue work that does not depend on the missing path. Record tiger validation as `NOT RUN` until its executable is available.

### Windows probes

Discover Steam from its registry entry and the default installation, then read only the library roots named by `libraryfolders.vdf`. This PowerShell block prints verified `<game>` candidates:

```powershell
$roots = @(
  (Get-ItemProperty 'HKCU:\Software\Valve\Steam' -ErrorAction SilentlyContinue).SteamPath
  "${env:ProgramFiles(x86)}\Steam"
) | Where-Object { $_ -and (Test-Path $_) } | Select-Object -Unique

$libraries = @($roots)
foreach ($root in $roots) {
  $vdf = Join-Path $root 'steamapps\libraryfolders.vdf'
  if (Test-Path $vdf) {
    $libraries += Select-String -Path $vdf -Pattern '^\s*"path"\s+"(.+)"\s*$' |
      ForEach-Object { $_.Matches[0].Groups[1].Value -replace '\\\\', '\' }
  }
}

$libraries | ForEach-Object { [IO.Path]::GetFullPath($_) } | Sort-Object -Unique | ForEach-Object {
  $candidate = Join-Path $_ 'steamapps\common\Crusader Kings III\game'
  $launcher = Join-Path (Split-Path $candidate -Parent) 'launcher\launcher-settings.json'
  if ((Test-Path (Join-Path $candidate 'common') -PathType Container) -and
      (Test-Path $launcher -PathType Leaf)) {
    $candidate
  }
}
```

Resolve `<user>` through the actual Documents folder, which handles redirection and OneDrive:

```powershell
Join-Path ([Environment]::GetFolderPath('MyDocuments')) 'Paradox Interactive\Crusader Kings III'
```

Probe tiger on `PATH` and in Paradox Toolkit's own download storage. The second command covers tiger installations that are not on `PATH`:

```powershell
Get-Command ck3-tiger -All -ErrorAction SilentlyContinue | Select-Object -ExpandProperty Source
Get-ChildItem "$env:APPDATA\Code\User\globalStorage\jdeffner.px-toolkit\tiger" `
  -Filter ck3-tiger.exe -File -Recurse -ErrorAction SilentlyContinue |
  Select-Object -ExpandProperty FullName
```

### Linux probes

Check Steam roots `~/.local/share/Steam`, `~/.steam/steam`, and `~/.var/app/com.valvesoftware.Steam/.local/share/Steam`. For each existing root, read `steamapps/libraryfolders.vdf` and probe the same `<library>/steamapps/common/Crusader Kings III/game` candidate as above. The usual `<user>` is `~/.local/share/Paradox Interactive/Crusader Kings III`.

Use `command -v ck3-tiger`, then check `~/.config/Code/User/globalStorage/jdeffner.px-toolkit/tiger/*/ck3-tiger`. Verify the selected binary with `--version` before using it.

## Build and schema

Read `launcher/launcher-settings.json` beside the game directory and inspect `rawVersion`. On Windows PowerShell, specify UTF-8:

```powershell
(Get-Content -Raw -Encoding UTF8 '<install>/launcher/launcher-settings.json' | ConvertFrom-Json).rawVersion
```

Read the relevant `_*.info` file's notes and structure, then a working definition in the same database. Angle-bracket fields such as `<key>` are placeholders. Comments often state types, defaults, and scopes. A missing documented default alone does not prove a field is required; check native use.

For exact identifiers, read fresh `effects.log`, `triggers.log`, `event_targets.log`, and `event_scopes.log` from `script_docs`. The game's `tests/` directory supplies additional examples. A dump older than a game update cannot establish the current signature.

Use `rg` to search the relevant schema, vanilla database, or dump. Resolve an effect's scope and find a real caller before adding it. Keep installed game and Workshop files read-only.

## Runtime evidence

| Question | Evidence |
|---|---|
| Did the engine accept the change? | Fresh `error.log` after loading and exercising the changed system |
| Which override won? | `database_conflicts.log` from the current launch |
| What GUI functions and types exist? | `dump_data_types` output under `logs/data_types/` |
| Why does the screen fail? | Rendered screen, `gui_warnings.log`, and `error.log` |
| Did the effect run correctly? | Before/after state through the normal entry point, supported by current logs |

Check timestamps before reading a dump as evidence. Record a reproduction baseline to separate old errors from new ones. If a required dump is missing, use available authorized controls to produce it. Otherwise ask the user for the exact launch or console step and read the files yourself afterwards. Missing GUI controls do not prevent static work or log analysis.

## Workshop patterns

Princes of Darkness (`2216659254`) and A Game of Thrones (`2962333032`) supply advanced patterns when installed. Read the distilled notes in [pod.md](../mods/pod.md) and [agot.md](../mods/agot.md). Their variables, effects, widget types, and textures may be mod-specific. Trace dependencies before adapting a sample into the user's mod.
