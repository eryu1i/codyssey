# 터미널 조작 로그

## 기본 조작
```bash
# 현재 위치 확인
$ pwd
/Users/ruel/codyssey_workstation

# 파일 목록 확인 (숨김 파일 포함)
$ ls -la

# 폴더 만들기
$ mkdir practice1

# 폴더 이동
$ cd practice1

# 빈 파일 만들기
$ touch hello.txt

# 파일 내용 쓰기
$ echo "hello world" > hello.txt

# 파일 내용 확인
$ cat hello.txt
hello world

# 파일 복사
$ cp hello.txt hello_copy.txt

# 파일 이름 변경
$ mv hello_copy.txt hello_renamed.txt

# 파일 삭제
$ rm hello_renamed.txt
```

## 권한 실습
```bash
# 파일 권한 확인
$ ls -la hello.txt
-rw-r--r-- 1 ruel staff 12 hello.txt

# 파일 권한 변경 (755)
$ chmod 755 hello.txt

# 변경 후 확인
$ ls -la hello.txt
-rwxr-xr-x 1 ruel staff 12 hello.txt

# 디렉토리 권한 변경 (644)
$ cd ..
$ chmod 644 practice1

# 변경 후 확인
$ ls -la
drw-r--r-- practice1
```