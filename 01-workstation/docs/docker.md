# Docker 로그

## 설치 및 점검
```bash
# Docker 버전 확인
$ docker --version
Docker version 20.3.1

# Docker 데몬 동작 확인 (정상이면 Docker 정보가 쭉 출력됨)
$ docker info
```

## 기본 운영 명령어
```bash
# 이미지 목록 확인
$ docker images

# 실행 중인 컨테이너 목록
$ docker ps

# 전체 컨테이너 목록 (종료된 것 포함)
$ docker ps -a

# 컨테이너 로그 확인
$ docker logs $(docker ps -a -q --filter ancestor=hello-world)

# 리소스 확인 (실행 중인 컨테이너 없으면 빈 표 출력)
$ docker stats --no-stream
```

## 컨테이너 실행 실습
```bash
# hello-world 실행
$ docker run hello-world
Hello from Docker!

# ubuntu 컨테이너 실행 및 내부 진입
$ docker run -it ubuntu bash
root@컨테이너ID:/# ls
root@컨테이너ID:/# echo "hello from ubuntu"
hello from ubuntu
root@컨테이너ID:/# exit
```

## 컨테이너 종료/유지 차이
```bash
# docker run -it → exit하면 컨테이너도 종료됨
$ docker run -it ubuntu bash
root@컨테이너ID:/# exit
$ docker ps  # 컨테이너 없음

# docker exec -it → exit해도 컨테이너는 계속 실행 중
$ docker run -d --name vol-test1 ubuntu sleep infinity
$ docker exec -it vol-test1 bash
root@컨테이너ID:/# exit
$ docker ps  # vol-test1 여전히 실행 중
```