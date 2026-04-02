# Docker Compose 로그

## 개념
`docker run` 명령어를 파일로 문서화해두는 것.
누구든 `docker compose up -d` 한 줄로 똑같은 환경을 실행할 수 있음.

## 기본 구조
```yaml
services:
  서비스이름:
    image: 이미지이름   # 어떤 이미지로 실행할지
    ports:
      - "호스트포트:컨테이너포트"  # 포트 매핑
    volumes:
      - ./폴더:컨테이너폴더        # 바인드 마운트
    environment:
      - 변수명=값                  # 컨테이너 내부 환경변수
```

## 멀티 컨테이너 (web + redis)
```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "${APP_PORT}:80"
    volumes:
      - ./app:/usr/share/nginx/html
    environment:
      - APP_ENV=production

  redis:
    image: redis:alpine
```

## 환경 변수 (.env 파일)
```
# .env 파일
APP_PORT=8082
```

- `.env` 파일 → Compose 레벨에서 사용 (포트 번호 등 `${변수명}` 형식)
- `environment:` → 컨테이너 내부에서 사용하는 변수

## 운영 명령어
```bash
# 컨테이너 실행 (백그라운드)
docker compose up -d

# 컨테이너 종료
docker compose down

# 실행 중인 컨테이너 목록
docker compose ps

# 로그 확인
docker compose logs
```

## 네트워크
Docker Compose는 실행할 때 자동으로 네트워크를 생성함.
같은 compose 파일 안의 컨테이너들은 서비스 이름으로 서로 통신 가능.
```bash
# web 컨테이너에서 redis 컨테이너로 통신 확인
docker exec -it codyssey_workstation-web-1 sh
ping redis  # redis 라는 이름으로 접근 가능
```