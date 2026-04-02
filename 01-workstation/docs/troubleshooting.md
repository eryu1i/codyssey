# 트러블슈팅

## 1. 바인드 마운트 후 403 Forbidden

### 문제
바인드 마운트로 컨테이너 실행 후 `localhost:8081` 접속시 403 Forbidden 발생

### 원인 가설
이미지 다운로드 중 마운트가 제대로 연결되지 않음

### 확인
```bash
$ docker logs my-web-bind
[error] directory index of "/usr/share/nginx/html/" is forbidden
```

### 해결
```bash
# 컨테이너 삭제 후 재실행
$ docker rm -f my-web-bind
$ docker run -d -p 8081:80 --name my-web-bind \
  -v $(pwd)/app:/usr/share/nginx/html nginx:alpine
```

---

## 2. 환경변수 ${APP_PORT} 랜덤 포트로 열림

### 문제
`docker-compose.yml` 에서 `${APP_PORT}` 로 포트를 설정했는데
`docker compose up` 후 8082가 아닌 랜덤 포트로 열림

### 원인 가설
`environment` 에 설정한 `APP_PORT=8082` 는 컨테이너 내부 환경변수라
Compose가 `ports` 의 `${APP_PORT}` 를 읽을 수 없음

### 확인
```bash
$ docker compose ps
# 포트가 8082가 아닌 랜덤 포트로 열려있음
```

### 해결
```bash
# .env 파일 생성
$ touch .env
```
```
# .env 파일 내용
APP_PORT=8082
```
```bash
# 재실행
$ docker compose down
$ docker compose up -d
# 8082 포트로 정상 열림
```