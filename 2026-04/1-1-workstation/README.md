# 개발 워크스테이션 구축

## 1. 프로젝트 개요
터미널, Docker, Git을 직접 세팅하고 운영하는 개발 워크스테이션 구축 과제입니다.
로컬 개발 환경 세팅, 재현 가능한 실행 환경 공유, 협업 기반 소스코드 관리를 목표로 합니다.

## 2. 실행 환경
- OS: macOS 26.3.1
- Shell: zsh
- Docker: 29.3.1
- Git: 2.50.1

## 3. 수행 체크리스트
- [x] 터미널 기본 조작 (pwd, ls -la, cd, mkdir, cp, mv, rm, touch, cat)
- [x] 파일/디렉토리 권한 변경 (chmod)
- [x] Docker 설치 및 점검 (docker --version, docker info)
- [x] Docker 기본 운영 (docker images, docker ps, docker logs, docker stats)
- [x] hello-world 컨테이너 실행
- [x] ubuntu 컨테이너 실행 및 내부 진입
- [x] A번 커스텀 이미지 제작 (nginx 기반 웹서버)
- [x] B번 커스텀 이미지 제작 (ubuntu 기반 커스텀 환경)
- [x] 포트 매핑 및 브라우저 접속 확인
- [x] 바인드 마운트 반영 확인
- [x] Docker 볼륨 영속성 확인
- [x] Git 설정 (user.name, user.email, defaultBranch)
- [x] GitHub 연동 (gh CLI, SSH 키)
- [x] Docker Compose 기초
- [x] Docker Compose 멀티 컨테이너 (web + redis)
- [x] Docker Compose 운영 명령어 (up, down, ps, logs)
- [x] 환경 변수 활용 (.env 파일)

## 4. 상세 문서

- [터미널 조작 로그](docs/terminal.md)
- [Docker 로그](docs/docker.md)
- [Dockerfile 로그](docs/dockerfile.md)
- [바인드 마운트 + 볼륨 로그](docs/mount-volume.md)
- [Git 설정 및 GitHub 연동](docs/git.md)
- [트러블슈팅](docs/troubleshooting.md)

## 5. 검증 방법

| 항목 | 명령어 | 결과 위치 |
|---|---|---|
| Docker 설치 확인 | `docker --version` | [Docker 로그](docs/docker.md) |
| 웹서버 접속 확인 | `localhost:8080` 브라우저 접속 | [Dockerfile 로그](docs/dockerfile.md) |
| 바인드 마운트 확인 | 파일 수정 후 브라우저 새로고침 | [마운트/볼륨 로그](docs/mount-volume.md) |
| 볼륨 영속성 확인 | 컨테이너 삭제 후 데이터 확인 | [마운트/볼륨 로그](docs/mount-volume.md) |
| Git 설정 확인 | `git config --global --list` | [Git 로그](docs/git.md) |