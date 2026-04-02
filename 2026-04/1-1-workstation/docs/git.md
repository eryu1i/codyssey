# Git 설정 및 GitHub 연동 로그

## Git 설정
```bash
# 사용자 이름 설정
$ git config --global user.name "eryu1i"

# 이메일 설정
$ git config --global user.email "깃허브이메일"

# 기본 브랜치를 main으로 설정
$ git config --global init.defaultBranch main

# 설정 확인
$ git config --global --list
user.name=eryu1i
user.email=깃허브이메일
init.defaultbranch=main
```

## GitHub 연동 (gh CLI)
```bash
# GitHub CLI 설치
$ brew install gh

# GitHub 로그인
$ gh auth login

# 연동 확인
$ gh auth status
```

## GitHub SSH 키 설정
```bash
# SSH 키 생성
# -t ed25519 → 암호화 방식
# -C → 키에 이메일 라벨 추가
$ ssh-keygen -t ed25519 -C "깃허브이메일"

# 공개키 클립보드에 복사
$ pbcopy < ~/.ssh/id_ed25519.pub

# GitHub에 공개키 등록 후 연결 테스트
$ ssh -T git@github.com
Hi eryu1i! You've successfully authenticated.

# 원격 저장소를 HTTPS에서 SSH로 변경
$ git remote set-url origin git@github.com:eryu1i/codyssey_workstation.git

# 변경 확인
$ git remote -v
origin  git@github.com:eryu1i/codyssey_workstation.git (fetch)
origin  git@github.com:eryu1i/codyssey_workstation.git (push)
```

## 저장소 생성 및 push
```bash
# 저장소 생성 및 clone
$ gh repo create codyssey_workstation --public --clone

# 첫 push
$ git push --set-upstream origin main
```