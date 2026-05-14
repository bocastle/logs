# Windows에서 Codex에 codex-peon 적용하기

Codex를 쓰다 보면 작업이 끝났는지, 권한 승인이 필요한지 계속 터미널을 들여다보게 된다. 이 문제를 해결하려고 `codex-peon`을 적용해봤다.

`codex-peon`은 Codex CLI의 `notify` 훅을 이용해서 작업 완료 시 Warcraft/StarCraft 스타일의 사운드를 재생해주는 도구다. 원래 저장소는 다음이다.

- https://github.com/mrdavey/codex-peon

README만 보면 설치는 간단하다.

```bash
curl -fsSL https://raw.githubusercontent.com/mrdavey/codex-peon/main/install.sh | bash
```

하지만 Windows + PowerShell + Codex 환경에서는 그대로 동작하지 않았다. 이번 글은 그 원인과 해결 과정을 정리한 기록이다.

## 증상

설치는 된 것처럼 보였지만 Codex에서 소리가 나지 않았다.

먼저 Codex 설정을 확인했다.

```toml
notify = ["python3", "C:\\Users\\D-\\.codex\\hooks\\codex-peon\\codex-peon.py"]
```

`notify` 설정 자체는 들어가 있었다. 문제는 Codex가 실제로 이 명령을 실행할 수 있느냐였다.

PowerShell에서 확인해보면 `python3`는 Microsoft Store용 WindowsApps 스텁을 가리키고 있었다.

```powershell
Get-Command python3
```

결과는 대략 이런 형태였다.

```text
C:\Users\D-\AppData\Local\Microsoft\WindowsApps\python3.exe
```

실행하면 다음과 같은 오류가 발생했다.

```text
'python3.exe' 프로그램을 실행하지 못했습니다.
지정한 로그온 세션이 없습니다. 이미 종료되었을 수도 있습니다.
```

즉 Codex의 `notify` 훅은 등록되어 있었지만, 실제 Python 런타임을 실행하지 못하고 있었다.

## 두 번째 문제: bash 스크립트

`codex-peon` 명령도 확인했다.

```powershell
Get-Command codex-peon
```

설치된 위치는 다음이었다.

```text
C:\Users\D-\.local\bin\codex-peon
```

파일 내용을 보면 bash 스크립트였다.

```bash
#!/usr/bin/env bash
set -euo pipefail

PEON_DIR="${CODEX_PEON_DIR:-$HOME/.codex/hooks/codex-peon}"
exec python3 "$PEON_DIR/codex-peon.py" "$@"
```

Windows PowerShell에서는 이 파일을 직접 실행할 수 없다. 그래서 다음과 같은 오류가 났다.

```text
'codex-peon' 프로그램을 실행하지 못했습니다. 액세스가 거부되었습니다.
```

정리하면 문제는 두 가지였다.

1. `python3`가 실제 Python이 아니라 WindowsApps 스텁이었다.
2. `codex-peon` 실행 파일이 PowerShell에서 실행할 수 없는 bash 스크립트였다.

## Python 설치

먼저 실제 Python 런타임을 설치했다.

```powershell
winget install -e --id Python.Python.3.12 --accept-package-agreements --accept-source-agreements
```

설치 후 실제 Python 경로를 확인했다.

```powershell
Get-ChildItem -Path $HOME\AppData\Local\Programs\Python -Recurse -Filter python.exe
```

사용한 경로는 다음이다.

```text
C:\Users\D-\AppData\Local\Programs\Python\Python312\python.exe
```

정상 동작도 확인했다.

```powershell
& 'C:\Users\D-\AppData\Local\Programs\Python\Python312\python.exe' --version
```

```text
Python 3.12.10
```

## Codex notify 설정 수정

`~/.codex/config.toml`의 `notify` 값을 실제 Python 경로로 바꿨다.

```toml
notify = [
  "C:\\Users\\D-\\AppData\\Local\\Programs\\Python\\Python312\\python.exe",
  "C:\\Users\\D-\\.codex\\hooks\\codex-peon\\codex-peon.py"
]
```

한 줄로 쓰면 다음과 같다.

```toml
notify = ["C:\\Users\\D-\\AppData\\Local\\Programs\\Python\\Python312\\python.exe", "C:\\Users\\D-\\.codex\\hooks\\codex-peon\\codex-peon.py"]
```

이제 Codex가 작업 완료 시 `python3`가 아니라 실제 Python 3.12 실행 파일로 `codex-peon.py`를 호출하게 된다.

## Windows용 codex-peon 래퍼 만들기

PowerShell에서 `codex-peon` 명령을 쓸 수 있도록 `.cmd` 래퍼를 만들었다.

생성한 파일:

```text
C:\Users\D-\.local\bin\codex-peon.cmd
```

내용은 다음과 같다.

```bat
@echo off
"C:\Users\D-\AppData\Local\Programs\Python\Python312\python.exe" "C:\Users\D-\.codex\hooks\codex-peon\codex-peon.py" %*
exit /b %ERRORLEVEL%
```

기존 bash 스크립트는 PowerShell이 잘못 잡지 않도록 이름을 바꿨다.

```text
codex-peon      -> codex-peon.bash
codex-peon.ps1 -> codex-peon.ps1.disabled
```

PowerShell의 실행 정책 때문에 `.ps1` 래퍼는 막힐 수 있다. 그래서 여기서는 `.cmd` 래퍼를 쓰는 쪽이 가장 단순했다.

정리 후 `Get-Command` 결과는 다음처럼 나왔다.

```text
Path        : C:\Users\D-\.local\bin\codex-peon.cmd
CommandType : Application
```

## 권한 문제

Codex 샌드박스 안에서 테스트할 때는 다음 오류가 보였다.

```text
PermissionError: [Errno 13] Permission denied:
'C:\\Users\\D-\\.codex\\hooks\\codex-peon\\config.json'
```

`codex-peon`은 실행 중 `config.json`과 `.state.json`을 읽고 쓴다. 이 파일들은 워크스페이스 밖의 `~/.codex/hooks/codex-peon` 아래에 있기 때문에, Codex 샌드박스에서는 쓰기 권한이 제한될 수 있다.

일반 터미널에서 실행할 Codex CLI는 샌드박스 밖에서 동작하므로, 실제 사용 경로 기준으로 권한을 확인했다.

```powershell
codex-peon status
```

결과:

```text
codex-peon: active, pack=peon, enabled=True
```

## 네이티브 Windows 오디오 지원 추가

여기까지 하면 `codex-peon`은 실행되지만, Windows에서는 실제 음성이 아니라 터미널 벨로 fallback될 수 있었다.

원인은 `codex-peon.py`의 플랫폼 감지와 오디오 재생 코드가 macOS, WSL, Linux만 처리하고 있었기 때문이다.

기존 흐름은 대략 이랬다.

```python
def detect_platform() -> str:
    if sys.platform == "darwin":
        return "mac"
    if sys.platform.startswith("linux"):
        rel = platform.release().lower()
        if "microsoft" in rel or "wsl" in rel:
            return "wsl"
        return "linux"
    return "unknown"
```

Windows에서는 `unknown`이 되고, 결국 `print("\a")`로 터미널 벨만 울렸다.

그래서 Windows 감지를 추가했다.

```python
def detect_platform() -> str:
    if sys.platform == "darwin":
        return "mac"
    if sys.platform.startswith("linux"):
        rel = platform.release().lower()
        if "microsoft" in rel or "wsl" in rel:
            return "wsl"
        return "linux"
    if sys.platform.startswith("win"):
        return "windows"
    return "unknown"
```

그리고 Windows용 재생 함수를 추가했다. PowerShell의 `PresentationCore`와 `System.Windows.Media.MediaPlayer`를 이용하는 방식이다.

```python
def _play_windows(sound: Path, volume: float) -> int | None:
    powershell = shutil.which("powershell.exe") or shutil.which("powershell")
    if not powershell:
        return None
    try:
        ps_volume = max(0.0, min(1.0, float(volume)))
    except (TypeError, ValueError):
        ps_volume = 1.0

    sound_uri = json.dumps(sound.resolve().as_uri())
    ps_script = (
        "Add-Type -AssemblyName PresentationCore; "
        "$p = New-Object System.Windows.Media.MediaPlayer; "
        f"$p.Open([Uri]::new({sound_uri})); "
        f"$p.Volume = {ps_volume}; "
        "Start-Sleep -Milliseconds 150; "
        "$p.Play(); "
        "Start-Sleep -Seconds 3; "
        "$p.Close()"
    )
    proc = subprocess.Popen(
        [powershell, "-NoProfile", "-NonInteractive", "-Command", ps_script],
        stdout=subprocess.DEVNULL,
        stderr=subprocess.DEVNULL,
    )
    return proc.pid
```

`play_sound`에도 Windows 분기를 추가했다.

```python
def play_sound(sound: Path, volume: float) -> int | None:
    plat = detect_platform()
    playback_pid: int | None = None
    if plat == "mac":
        playback_pid = _play_mac(sound, volume)
    elif plat == "windows":
        playback_pid = _play_windows(sound, volume)
    elif plat == "wsl":
        playback_pid = _play_wsl(sound, volume)
    elif plat == "linux":
        playback_pid = _play_linux(sound)

    if playback_pid is None:
        print("\a", end="", flush=True)
    return playback_pid
```

수정 후 문법 검사를 했다.

```powershell
& 'C:\Users\D-\AppData\Local\Programs\Python\Python312\python.exe' -m py_compile 'C:\Users\D-\.codex\hooks\codex-peon\codex-peon.py'
```

## 동작 확인

상태 확인:

```powershell
codex-peon status
```

결과:

```text
codex-peon: active, pack=peon, enabled=True
```

미리보기 재생:

```powershell
codex-peon preview acknowledge
```

결과:

```text
codex-peon: played acknowledge -> PeonYesAttack2.wav
```

Codex의 실제 `notify` payload 형태도 직접 테스트했다.

```powershell
& 'C:\Users\D-\AppData\Local\Programs\Python\Python312\python.exe' `
  'C:\Users\D-\.codex\hooks\codex-peon\codex-peon.py' `
  '{"type":"agent-turn-complete","thread-id":"codex-peon-test","last-assistant-message":"done"}'
```

정상적으로 종료 코드 `0`으로 끝났다.

## Sarah Kerrigan 사운드팩으로 변경

기본 `peon` 팩 대신 Sarah Kerrigan 사운드팩을 사용하도록 바꿨다.

```powershell
codex-peon pack sc_kerrigan
```

결과:

```text
codex-peon: switched to sc_kerrigan (Sarah Kerrigan (StarCraft))
```

상태 확인:

```powershell
codex-peon status
```

```text
codex-peon: active, pack=sc_kerrigan, enabled=True
```

미리보기:

```powershell
codex-peon preview acknowledge
```

```text
codex-peon: played acknowledge -> IReadYou.mp3
```

## 최종 상태

최종적으로 Windows에서 Codex + codex-peon 조합이 동작하도록 만든 변경점은 다음과 같다.

- 실제 Python 3.12 설치
- Codex `notify` 설정을 실제 `python.exe` 경로로 변경
- PowerShell에서 실행 가능한 `codex-peon.cmd` 래퍼 추가
- bash/ps1 래퍼 충돌 제거
- `codex-peon.py`에 네이티브 Windows 오디오 재생 분기 추가
- 사운드팩을 `sc_kerrigan`으로 변경

현재 설정 기준으로 Codex 작업이 끝나면 `Sarah Kerrigan (StarCraft)` 사운드팩이 재생된다.

## 마무리

`codex-peon` 자체는 Codex의 `notify` 훅을 잘 활용한 작은 도구다. 다만 설치 스크립트가 Unix 계열 셸을 기준으로 되어 있어서 Windows PowerShell 환경에서는 몇 가지 부분을 직접 맞춰줘야 했다.

핵심은 간단하다.

1. Codex가 실행할 수 있는 실제 Python 경로를 `notify`에 넣는다.
2. PowerShell에서 실행 가능한 래퍼를 만든다.
3. 네이티브 Windows 오디오 백엔드를 추가한다.

이 세 가지를 정리하면 Windows에서도 Codex 작업 완료 알림을 꽤 그럴듯하게 쓸 수 있다.
