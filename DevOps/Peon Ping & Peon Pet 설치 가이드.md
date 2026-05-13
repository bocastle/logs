# Peon Ping / Peon Pet 설치 가이드

이 문서는 **Windows / macOS / Linux(WSL2 포함)** 환경에서 **Peon Ping**을 설치하고, **Cursor Agent가 멈추거나 완료되거나 입력 대기 상태일 때 알림을 받는 방법**을 정리한 매뉴얼입니다.

---

## 1. 목적

Peon Ping은 다음 상태를 **소리 / 데스크톱 알림**으로 알려주는 도구입니다.

- Cursor Agent 작업 완료
- Cursor Agent 멈춤 또는 입력 대기
- 승인 필요 상태
- 에러 상태

추가로 **Peon Pet**은 오크 캐릭터를 보여주는 시각화 도구입니다.

- **Peon Ping**: 알림/사운드 핵심 도구
- **Peon Pet**: 오크 캐릭터 시각화 도구

---

## 2. 공통 사전 준비

### 2.1 Cursor 설정

Cursor에서 아래 설정이 켜져 있어야 합니다.

- `Cursor Settings → Features → Third-party skills`

### 2.2 설치 후 기본 확인 명령어

아래 명령으로 설치 상태를 확인합니다.

```bash
peon status
```

자세한 상태를 보려면:

```bash
peon status --verbose
```

볼륨 조정:

```bash
peon volume 1
```

데스크톱 알림 켜기:

```bash
peon notifications on
```

---

## 3. Windows 설치 방법

### 3.1 PowerShell 설치

PowerShell에서 아래 명령을 실행합니다.

```powershell
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/PeonPing/peon-ping/main/install.ps1" -UseBasicParsing | Invoke-Expression
```

또는 파일로 받아 실행하려면:

```powershell
cd $env:USERPROFILE\Downloads
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/PeonPing/peon-ping/main/install.ps1" -OutFile ".\install.ps1" -UseBasicParsing
powershell -ExecutionPolicy Bypass -File .\install.ps1
```

### 3.2 설치 확인

```powershell
peon status
peon status --verbose
```

정상 예시:

```text
peon-ping: ENABLED
peon-ping: 10 pack(s) installed
```

### 3.3 Cursor 연동 확인

1. Cursor를 완전히 종료합니다.
2. Cursor를 다시 실행합니다.
3. `Settings → Features → Third-party skills`가 켜져 있는지 확인합니다.
4. Cursor의 Skills 목록에 아래 항목들이 보이면 정상에 가깝습니다.

- `peon-ping-config`
- `peon-ping-log`
- `peon-ping-toggle`
- `peon-ping-use`

### 3.4 Cursor에서 Peon Ping 사용

Cursor Agent 채팅창에서 아래처럼 사용할 수 있습니다.

```text
/peon-ping-toggle on
```

필요 시:

```text
/peon-ping-config
/peon-ping-use
/peon-ping-log
```

### 3.5 Windows 주의사항

- Windows에서는 **`peon-ping-setup` 명령을 쓰지 않는 흐름**으로 보는 것이 안전합니다.
- Windows에서는 주로 `install.ps1` 실행 후 `peon ...` 명령으로 확인합니다.
- 소리가 안 나면 먼저 Windows 기본 소리 자체가 정상인지 확인해야 합니다.

Windows 기본 사운드 테스트:

```powershell
(New-Object Media.SoundPlayer "C:\Windows\Media\Windows Notify System Generic.wav").PlaySync()
```

---

## 4. macOS 설치 방법

### 4.1 Homebrew 설치

터미널에서 아래 명령을 실행합니다.

```bash
brew install PeonPing/tap/peon-ping
```

### 4.2 초기 설정

설치 후 아래 명령을 실행합니다.

```bash
peon-ping-setup
```

이 단계에서 훅 등록과 관련 설정이 잡히는 흐름으로 사용합니다.

### 4.3 설치 확인

```bash
peon status
peon status --verbose
```

### 4.4 Cursor 연동

1. Cursor를 완전히 종료합니다.
2. Cursor를 다시 실행합니다.
3. `Cursor Settings → Features → Third-party skills`를 켭니다.
4. Cursor Agent를 실행해 작업 완료 / 입력 대기 시 알림이 오는지 확인합니다.

### 4.5 macOS 팁

볼륨 조정:

```bash
peon volume 1
```

데스크톱 알림 켜기:

```bash
peon notifications on
```

---

## 5. Linux / WSL2 설치 방법

### 5.1 Bash 설치

Linux 또는 WSL2, Git Bash 환경에서는 아래 명령을 사용합니다.

```bash
curl -fsSL https://raw.githubusercontent.com/PeonPing/peon-ping/main/install.sh | bash
```

### 5.2 설치 확인

```bash
peon status
peon status --verbose
```

### 5.3 Cursor 연동

1. Cursor를 완전히 종료합니다.
2. Cursor를 다시 실행합니다.
3. `Cursor Settings → Features → Third-party skills`를 켭니다.
4. Agent 작업을 실행해서 완료/입력 대기 시 알림이 오는지 확인합니다.

### 5.4 Linux/WSL2 팁

볼륨 조정:

```bash
peon volume 1
```

데스크톱 알림 켜기:

```bash
peon notifications on
```

---

## 6. Peon Pet 설치

Peon Pet은 오크 캐릭터를 보여주는 보조 도구입니다.
실제 핵심은 Peon Ping이고, Peon Pet은 시각화용입니다.

### 6.1 Cursor에서 설치

Cursor는 VS Code 기반이므로 확장을 설치할 수 있습니다.

명령 팔레트에서:

```text
Extensions: Install Extensions
```

검색:

```text
Peon Pet
```

또는 직접 설치:

```text
ext install smcqueen.vscode-peon-pet
```

---

## 7. 동작 테스트 방법

### 7.1 기본 확인

```bash
peon status --verbose
```

확인 포인트:

- `ENABLED`
- `pack(s) installed`
- `desktop notifications on`
- `volume` 값 확인

### 7.2 Cursor에서 실제 테스트

Cursor Agent에게 아래처럼 간단한 작업을 시킵니다.

```text
현재 프로젝트 구조를 훑고 요약해줘.
```

또는 입력 대기를 유도하려면:

```text
프로젝트 파일을 확인하고, 수정이 필요하면 수정 전에 나한테 확인받아줘.
```

기대 결과:

- 작업 완료 시 알림
- 입력 대기 시 알림
- 승인 필요 시 알림

---

## 8. 문제 해결

### 8.1 `peon` 명령이 인식되지 않음

- 터미널/PowerShell을 완전히 종료 후 다시 엽니다.
- 다시 `peon status`를 실행합니다.

### 8.2 Windows에서 `peon-ping-setup` 인식 안 됨

- Windows에서는 일반적으로 `peon-ping-setup`을 쓰지 않습니다.
- `install.ps1` 실행 후 `peon status`로 확인합니다.

### 8.3 소리가 안 남

1. 볼륨을 올립니다.

```bash
peon volume 1
```

2. 데스크톱 알림을 켭니다.

```bash
peon notifications on
```

3. Windows라면 기본 사운드가 나는지 먼저 확인합니다.

```powershell
(New-Object Media.SoundPlayer "C:\Windows\Media\Windows Notify System Generic.wav").PlaySync()
```

4. OS 볼륨 믹서에서 PowerShell / Windows Terminal / Cursor 음소거 여부를 확인합니다.

### 8.4 Cursor에서 스킬이 안 보임

- `Settings → Features → Third-party skills`가 켜져 있는지 확인합니다.
- Cursor를 완전히 종료 후 재실행합니다.
- 필요 시 Peon Ping 재설치를 진행합니다.

---

## 9. 권장 사용 순서

### Windows

1. `install.ps1` 설치
2. `peon status` 확인
3. `peon status --verbose` 확인
4. `peon volume 1`
5. `peon notifications on`
6. Cursor 재시작
7. Third-party skills 확인
8. Agent 작업 테스트

### macOS

1. `brew install PeonPing/tap/peon-ping`
2. `peon-ping-setup`
3. `peon status`
4. `peon volume 1`
5. `peon notifications on`
6. Cursor 재시작
7. Third-party skills 확인
8. Agent 작업 테스트

### Linux / WSL2

1. `curl ... install.sh | bash`
2. `peon status`
3. `peon volume 1`
4. `peon notifications on`
5. Cursor 재시작
6. Third-party skills 확인
7. Agent 작업 테스트

---

## 10. 핵심 요약

- **알림/사운드 핵심은 Peon Ping**입니다.
- **오크 캐릭터 시각화는 Peon Pet**입니다.
- Cursor에서는 **Third-party skills 활성화**가 중요합니다.
- Windows는 **PowerShell install.ps1**, macOS는 **Homebrew**, Linux/WSL2는 **install.sh**가 기본 흐름입니다.
- 설치 후에는 반드시 `peon status`, `peon status --verbose`, `peon notifications on`, `peon volume 1`로 상태를 확인합니다.
