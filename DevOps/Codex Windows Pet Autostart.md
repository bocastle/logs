# Codex Windows Pet Autostart

Codex Desktop pet overlay can be closed during app shutdown. When that happens, Codex saves:

```json
"electron-avatar-overlay-open": false
```

This guide installs a tiny Windows background helper that keeps the value armed as `true`, so the pet opens again the next time Codex starts.

## Install

Open PowerShell and paste the whole block below.

```powershell
$ErrorActionPreference = "Stop"

$codexDir = Join-Path $env:USERPROFILE ".codex"
$statePath = Join-Path $codexDir ".codex-global-state.json"
$helperDir = Join-Path $codexDir "hooks\pet-autostart"
$helperPath = Join-Path $helperDir "pet-autostart.py"

if (!(Test-Path $statePath)) {
    throw "Codex state file not found: $statePath. Start Codex once first, then run this again."
}

New-Item -ItemType Directory -Force -Path $helperDir | Out-Null

$helper = @'
import json
import time
from pathlib import Path

STATE_PATH = Path.home() / ".codex" / ".codex-global-state.json"
LOG_PATH = Path.home() / ".codex" / "hooks" / "pet-autostart" / "pet-autostart.log"
KEY = "electron-avatar-overlay-open"


def log(message: str) -> None:
    try:
        LOG_PATH.parent.mkdir(parents=True, exist_ok=True)
        with LOG_PATH.open("a", encoding="utf-8") as fh:
            fh.write(time.strftime("%Y-%m-%d %H:%M:%S ") + message + "\n")
    except OSError:
        pass


def force_pet_open() -> None:
    try:
        raw = STATE_PATH.read_text(encoding="utf-8-sig")
        data = json.loads(raw)
    except Exception as exc:
        log(f"skip: cannot read state: {exc}")
        return

    if data.get(KEY) is True:
        return

    data[KEY] = True
    tmp = STATE_PATH.with_suffix(STATE_PATH.suffix + ".pet-autostart.tmp")
    try:
        tmp.write_text(
            json.dumps(data, ensure_ascii=False, separators=(",", ":")),
            encoding="utf-8",
        )
        tmp.replace(STATE_PATH)
        log("set electron-avatar-overlay-open=true")
    except Exception as exc:
        log(f"skip: cannot write state: {exc}")
        try:
            tmp.unlink(missing_ok=True)
        except OSError:
            pass


def main() -> None:
    while True:
        force_pet_open()
        time.sleep(2)


if __name__ == "__main__":
    main()
'@

Set-Content -LiteralPath $helperPath -Value $helper -Encoding UTF8

$python = $null
$pyLauncher = Get-Command py.exe -ErrorAction SilentlyContinue
$pythonCommand = Get-Command python.exe -ErrorAction SilentlyContinue

if ($pyLauncher) {
    $python = $pyLauncher.Source
    $pythonArgs = "-3 `"$helperPath`""
    & $python -3 -m py_compile $helperPath
} elseif ($pythonCommand) {
    $python = $pythonCommand.Source
    $pythonArgs = "`"$helperPath`""
    & $python -m py_compile $helperPath
} else {
    throw "Python was not found. Install Python 3 first, then run this again."
}

$startup = [Environment]::GetFolderPath("Startup")
$vbsPath = Join-Path $startup "Codex Pet Autostart.vbs"
$vbs = @"
Set WshShell = CreateObject("WScript.Shell")
WshShell.Run """$python"" $pythonArgs", 0, False
"@

Set-Content -LiteralPath $vbsPath -Value $vbs -Encoding ASCII

$raw = Get-Content -LiteralPath $statePath -Raw -Encoding UTF8
$data = $raw | ConvertFrom-Json
$data | Add-Member -NotePropertyName "electron-avatar-overlay-open" -NotePropertyValue $true -Force
$data | ConvertTo-Json -Depth 100 -Compress | Set-Content -LiteralPath $statePath -Encoding UTF8

$alreadyRunning = Get-Process python -ErrorAction SilentlyContinue | Where-Object {
    try { $_.Path -and $_.Path -eq $python } catch { $false }
}

Start-Process -FilePath $python -ArgumentList $pythonArgs -WindowStyle Hidden

Write-Host "Codex pet autostart installed."
Write-Host "Startup script: $vbsPath"
Write-Host "Helper script:  $helperPath"
Write-Host "Restart Codex to test it."
```

## How To Test

1. Fully quit Codex.
2. Wait 2-3 seconds.
3. Start Codex again.
4. The pet should appear shortly after the main window opens.

The helper checks every 2 seconds, so a small delay is normal.

## What It Does

The helper watches this file:

```text
%USERPROFILE%\.codex\.codex-global-state.json
```

It keeps this key enabled:

```json
"electron-avatar-overlay-open": true
```

Codex may write `false` when the app closes, because the pet overlay window is closed during shutdown. The helper turns it back to `true` before the next launch.

## Uninstall

Open PowerShell and paste this:

```powershell
$startup = [Environment]::GetFolderPath("Startup")
$vbsPath = Join-Path $startup "Codex Pet Autostart.vbs"
$helperDir = Join-Path $env:USERPROFILE ".codex\hooks\pet-autostart"

Remove-Item -LiteralPath $vbsPath -Force -ErrorAction SilentlyContinue
Remove-Item -LiteralPath $helperDir -Recurse -Force -ErrorAction SilentlyContinue

Write-Host "Codex pet autostart removed. Restart Windows or stop the running python helper from Task Manager."
```

## Notes

- This does not modify the Codex app package.
- This works per Windows user account.
- Python 3 is required.
- If Codex changes the state file format in a future update, check:

```text
%USERPROFILE%\.codex\hooks\pet-autostart\pet-autostart.log
```
