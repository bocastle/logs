# 🧾 Git 레포지토리 정보 확인 및 SSH 변경 (GitLab 이전 대비)

## 📌 현재 로컬 Git 레포지토리 정보 확인

### 🔹 원격 저장소(Remote) 확인

```bash
git remote -v
```

```bash
origin https://github.com/username/repo-name.git (fetch)
origin https://github.com/username/repo-name.git (push)
```

### 🔹 현재 브랜치 확인

```bash
git branch --show-current
```

### 🔹 커밋 로그 확인

```bash
git log --oneline
```

# 🔧 GitHub ➡ GitLab SSH 변경 가이드

## 1. GitLab에서 SSH 키 등록

### 🔹 로컬 SSH 키 확인 또는 새로 생성

```bash
ls ~/.ssh
# 또는 새로 생성
ssh-keygen -t ed25519 -C "your-email@example.com"
```

# 🔧 GitHub ➡ GitLab SSH 설정 변경 요약

## 🔹 공개 키 복사 및 등록

```bash
cat ~/.ssh/id_ed25519.pub
```

# 🔧 GitLab 원격 주소 및 SSH 설정 변경

## 2. 기존 원격 주소 변경

```bash
# 기존 주소 확인
git remote -v

# 기존 origin 삭제
git remote remove origin

# GitLab SSH 주소로 등록
git remote add origin git@gitlab.com:your-username/your-repo.git

# 변경된 주소 확인
git remote -v
```

# ✅ 설정 확인 및 기타 정보

## 🔹 Git 설정 정보

```bash
git config --list

```

## 🔹 원격 브랜치 목록

```bash
git branch -r
```

## 🧪 SSH 연결 테스트

```bash
ssh -T git@gitlab.com
```

### 🔹 응답 예시

```text
Welcome to GitLab, @your-username!
```

## 🗂️ 기타 정보

- **GitLab 레포 주소:**  
  `git@gitlab.com:your-username/your-repo.git`

- **SSH 키 위치:**  
  `~/.ssh/id_ed25519`  
  (또는 `~/.ssh/id_rsa` 등)
