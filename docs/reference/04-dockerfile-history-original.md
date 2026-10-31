# Dockerfile 상세 실습 및 트러블슈팅 기록 (참고)

> 기존 HTTP 403 문제 분석과 단계별 출력 로그를 보존한 재현자료입니다.
>  403 오류는 환경에 따라 재현되지 않을 수 있습니다.

# 4. Dockerfile 기반 웹 서버 컨테이너

## 실습 시작

프로젝트 루트에서 터미널을 열고 실행합니다.

```bash
$ cd practice/04-dockerfile
$ pwd
/home/사용자명/projects/codyssey/1-1_codyssey-workstation/practice/04-dockerfile
```

※ 아래 빌드·실행 출력은 **기존 환경에서 기록한 트러블슈팅 결과**입니다.
다른환경에서는 v1·v2의 `403 Forbidden`이 재현되지 않을 수 있습니다.
재현에 필요한 Dockerfile과 HTML은 아래 예제를 참고하여 작성하며, v3의 HTTP `200 OK`를 정상 동작 기준으로 확인하기 전의 히스토리입니다.

## 4-1. Ubuntu 이미지 실행 및 컨테이너 내부 진입

**① Ubuntu 24.04 이미지 다운로드**

```bash
$ docker pull ubuntu:24.04
24.04: Pulling from library/ubuntu
7a48b8f0e5c7: Pull complete
98144013299c: Download complete
Digest: sha256:fe81560990fae86dc4743202b2cb4d3df98d0bcf949173cecef15ce1127ef852
Status: Downloaded newer image for ubuntu:24.04
docker.io/library/ubuntu:24.04
```

**② Ubuntu 컨테이너 생성 및 대화형 Bash 진입**

```bash
$ docker run -it \
  --name codyssey-ubuntu \
  --network none \
  ubuntu:24.04 bash
root@c7550374bc2b:/# pwd
/
root@c7550374bc2b:/# whoami
root
root@c7550374bc2b:/# ls /
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@c7550374bc2b:/# cat /etc/os-release
PRETTY_NAME="Ubuntu 24.04.5 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.5 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo
root@c7550374bc2b:/# echo "Hello from Ubuntu Container!"
Hello from Ubuntu Container!
root@c7550374bc2b:/# exit
exit
```

**③ 컨테이너 종료 및 호스트에서 상태 확인**

```bash
$ docker ps -a --filter name=codyssey-ubuntu
CONTAINER ID   IMAGE          COMMAND   CREATED              STATUS                     PORTS     NAMES
c7550374bc2b   ubuntu:24.04   "bash"    About a minute ago   Exited (0) 6 seconds ago             codyssey-ubuntu
```

## 4-2. 컨테이너 유지·종료 및 attach/exec 차이

**① 종료된 Ubuntu 컨테이너 재시작 후 터미널 분리**

```bash
$ docker start -ai codyssey-ubuntu
root@c7550374bc2b:/#
```

`Ctrl+P` → `Ctrl+Q`를 눌러 호스트 터미널로 돌아간다.

```bash
$ docker ps --filter name=codyssey-ubuntu
CONTAINER ID   IMAGE          COMMAND   CREATED         STATUS         PORTS     NAMES
c7550374bc2b   ubuntu:24.04   "bash"    7 minutes ago   Up 8 seconds             codyssey-ubuntu
```

**② exec로 별도의 Bash 프로세스 실행**

```bash
$ docker exec -it codyssey-ubuntu bash
root@c7550374bc2b:/# echo "Inside exec session"
Inside exec session
root@c7550374bc2b:/# ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 06:44 pts/0    00:00:00 bash
root           9       0  0 06:44 pts/1    00:00:00 bash
root          17       9  0 06:44 pts/1    00:00:00 ps -ef
root@c7550374bc2b:/# exit
exit
$ docker ps --filter name=codyssey-ubuntu
CONTAINER ID   IMAGE          COMMAND   CREATED         STATUS          PORTS     NAMES
c7550374bc2b   ubuntu:24.04   "bash"    7 minutes ago   Up 38 seconds             codyssey-ubuntu
```

**③ attach로 기존 Bash에 재연결한 뒤 종료**

```bash
$ docker attach codyssey-ubuntu
root@c7550374bc2b:/# exit
exit
$ docker ps -a --filter name=codyssey-ubuntu
CONTAINER ID   IMAGE          COMMAND   CREATED         STATUS                     PORTS     NAMES
c7550374bc2b   ubuntu:24.04   "bash"    8 minutes ago   Exited (0) 2 seconds ago             codyssey-ubuntu
```

## 4-3. Dockerfile 작성 및 커스텀 이미지 빌드

실습용 `Dockerfile`과 `web/index.html`은 Git에 포함되지 않습니다. 다음 코드 예제를 참고하여 직접 생성하세요. 처음 클론했을 때 두 파일이 없는 것이 정상입니다.

**① 웹 페이지와 Dockerfile 직접 작성**

```bash
$ mkdir -p web
$ nano web/index.html
$ nano Dockerfile
$ cat web/index.html

<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>Codyssey Web Server</title>
</head>
<body>
  <h1>Hello, Codyssey!</h1>
  <p>My first custom Docker image.</p>
</body>
</html>
$ cat Dockerfile
FROM nginx:alpine

COPY web/index.html /usr/share/nginx/html/index.html

EXPOSE 80
```

**② Dockerfile 기반 커스텀 이미지 빌드**

위에서 만든 v1 Dockerfile을 저장한 후 빌드합니다.

```bash
$ docker build -t codyssey-web:v1 .
[+] Building 3.5s (7/7) FINISHED                                                                                                          docker:default
 => [internal] load build definition from Dockerfile                                                                                                0.0s
 => => transferring dockerfile: 201B                                                                                                                0.0s
 => [internal] load metadata for docker.io/library/nginx:alpine                                                                                     2.0s
 => [internal] load .dockerignore                                                                                                                   0.0s
 => => transferring context: 2B                                                                                                                     0.0s
 => [internal] load build context                                                                                                                   0.0s
 => => transferring context: 516B                                                                                                                   0.0s
 => [1/2] FROM docker.io/library/nginx:alpine@sha256:785835ea65db4bba6f021acc0f3be91b0fef76a53fe41f8588c6698d2aed0606                               1.1s
 => [2/2] COPY web/index.html /usr/share/nginx/html/index.html                                                                                      0.0s
 => exporting to image                                                                                                                              0.3s
 => => naming to docker.io/library/codyssey-web:v1                                                                                                  0.0s
```

**③ 생성된 커스텀 이미지 확인**

```bash
$ docker images codyssey-web
                                                                                                                                    i Info →   U  In Use
IMAGE             ID             DISK USAGE   CONTENT SIZE   EXTRA
codyssey-web:v1   b94be29bac6d       93.6MB         26.3MB
```

**배운 내용:**

- `docker images`: 빌드한 이미지 확인

## 4-4. 커스텀 웹 서버 컨테이너 실행

**① 커스텀 이미지로 웹 서버 실행**

```bash
$ docker run -d \
  --name codyssey-web \
  codyssey-web:v1
a20e97cd7587310c2e6aa4ec77949d512122aa5fd89c07af831b2ab25ada302d
```

**② 컨테이너 실행 상태 및 로그 확인**

```bash
$ docker ps --filter name=codyssey-web
$ docker logs codyssey-web
CONTAINER ID   IMAGE             COMMAND                   CREATED          STATUS          PORTS     NAMES
a20e97cd7587   codyssey-web:v1   "/docker-entrypoint.…"   10 seconds ago   Up 10 seconds   80/tcp    codyssey-web
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2026/10/29 07:12:01 [notice] 1#1: using the "epoll" event method
2026/10/29 07:12:01 [notice] 1#1: nginx/1.31.8
2026/10/29 07:12:01 [notice] 1#1: built by gcc 15.2.0 (Alpine 15.2.0)
2026/10/29 07:12:01 [notice] 1#1: OS: Linux 7.0.0-38-generic
2026/10/29 07:12:01 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1024:524288
2026/10/29 07:12:01 [notice] 1#1: start worker processes
2026/10/29 07:12:01 [notice] 1#1: start worker process 30
```

**③ 컨테이너 내부에서 HTTP 요청 검증 — 실패 기록**

```bash
$ docker exec codyssey-web wget -qO- http://127.0.0.1/
wget: server returned error: HTTP/1.1 403 Forbidden
```


**④ 권한 및 NGINX 오류 로그 검증 (v1)**

```bash
$ docker exec codyssey-web sh -c 'id nginx; ls -l /usr/share/nginx/html/index.html'
uid=101(nginx) gid=101(nginx) groups=101(nginx),101(nginx)
-rw-rw----    1 root     root           201 Oct 29 07:05 /usr/share/nginx/html/index.html

$ docker logs --tail 20 codyssey-web
2026/10/29 07:12:29 [error] 30#30: *1 open() "/usr/share/nginx/html/index.html" failed (13: Permission denied), client: 127.0.0.1, server: localhost, request: "GET / HTTP/1.1", host: "127.0.0.1"
127.0.0.1 - - [29/Oct/2026:07:12:29 +0000] "GET / HTTP/1.1" 403 153 "-" "Wget" "-"
```

**배운 내용:**

- NGINX 사용자: UID 101로 HTML 파일에 접근
- 파일 권한 `660`: `root:root` 소유 파일을 NGINX 사용자가 읽지 못함

**⑤ 권한 변경 후 v2 재빌드 및 HTTP 요청 (아직 실패)**

`nano Dockerfile`로 다음 v2 내용을 작성해 저장합니다.

```dockerfile
FROM nginx:alpine

COPY --chmod=644 web/index.html /usr/share/nginx/html/index.html

EXPOSE 80
```

```bash
$ nano Dockerfile
$ docker build -t codyssey-web:v2 .
[+] Building 1.3s (7/7) FINISHED
 => CACHED [1/2] FROM docker.io/library/nginx:alpine
 => [2/2] COPY --chmod=644 web/index.html /usr/share/nginx/html/index.html
 => exporting to image
 => => naming to docker.io/library/codyssey-web:v2

$ docker run -d --name codyssey-web-v2 codyssey-web:v2
0f8f57b5f5c107c049f4e5c0be4873593c16eb0bfebccd570df1d6957fdae707
$ docker exec codyssey-web-v2 wget -qO- http://127.0.0.1/
wget: server returned error: HTTP/1.1 403 Forbidden
```

**배운 내용:**

- `COPY --chmod=644`: v2 이미지 빌드는 성공
- HTTP `403`: 빌드 성공 후에도 웹 서버 접근 실패

**⑥ v2 컨테이너 파일 권한과 NGINX 오류 재확인**

```bash
$ docker exec codyssey-web-v2 sh -c '
id nginx
ls -ld /usr /usr/share /usr/share/nginx /usr/share/nginx/html
ls -l /usr/share/nginx/html/index.html
'
uid=101(nginx) gid=101(nginx) groups=101(nginx),101(nginx)
drwxr-xr-x    1 root     root          4096 Oct 21 21:52 /usr
drwxr-xr-x    1 root     root          4096 Oct 21 23:23 /usr/share
drwxr-xr-x    1 root     root          4096 Oct 21 22:47 /usr/share/nginx
drwxr-xr-x    1 root     root          4096 Oct 29 07:17 /usr/share/nginx/html
-rw-rw----    1 root     root           201 Oct 29 07:05 /usr/share/nginx/html/index.html

$ docker logs --tail 15 codyssey-web-v2
2026/10/29 07:17:58 [notice] 1#1: start worker process 49
2026/10/29 07:17:58 [error] 30#30: *1 open() "/usr/share/nginx/html/index.html" failed (13: Permission denied), client: 127.0.0.1, server: localhost, request: "GET / HTTP/1.1", host: "127.0.0.1"
127.0.0.1 - - [29/Oct/2026:07:17:58 +0000] "GET / HTTP/1.1" 403 153 "-" "Wget" "-"
```

**배운 내용:**

- 상위 디렉터리 `755`: 경로 탐색 가능
- v2 HTML 파일 `660`: 읽기 권한 문제 지속

**⑦ v2 이미지 자체와 실행 컨테이너 비교**

```bash
$ docker run --rm --network none \
  --entrypoint sh codyssey-web:v2 \
  -c 'stat -c "%a %U:%G %n" /usr/share/nginx/html/index.html'
660 root:root /usr/share/nginx/html/index.html

$ docker image inspect codyssey-web:v2 \
  --format 'Image: {{.Id}}'
Image: sha256:1301951fd72ac5614fbfbf989d2d046a61ff79739832b51c86b90d60b5b2a888

$ docker inspect codyssey-web-v2 \
  --format 'Container image: {{.Image}}'
Container image: sha256:1301951fd72ac5614fbfbf989d2d046a61ff79739832b51c86b90d60b5b2a888

$ docker inspect codyssey-web-v2 \
  --format 'Mounts: {{json .Mounts}}'
Mounts: []
```

**배운 내용:**

- 이미지·컨테이너 ID: 일치 확인
- 마운트: 없음
- v2 이미지 내부 HTML 권한: `660`

**추가 조사: 호스트 파일 ACL과 BuildKit**

```bash
$ getfacl -p web/index.html
# file: web/index.html
# owner: devspace-agent
# group: devspace-agent
user::rw-
user:devspace-agent:rwx    #effective:rw-
user:사용자명:rwx             #effective:rw-
group::r-x                 #effective:r--
mask::rw-
other::---
```

**배운 내용:**

- `getfacl`: 확장 ACL 확인
- 확장 ACL: 파일의 최종 접근 권한에 영향

**원인 추정:** Moby [#49766](https://github.com/moby/moby/issues/49766)에도 ACL이 있는 파일의 `COPY --chmod` 권한 불일치가 보고됐다. [BuildKit 개발자 설명](https://github.com/moby/moby/issues/49766#issuecomment-2837024195)에 따르면 `COPY --chmod`는 `stat.mode`를 설정하지만 
복사된 `system.posix_acl_access` 확장 속성이 최종 권한에 영향을 줄 수 있다.
이번 v2의 `660` 결과와 부합하지만 **ACL 없는 원본으로 대조 빌드하지 않았으므로 추정**
즉, 다른사용자에게 허용하기 위해 ACL 확장속성을 설정한게 docker 결함과 겹쳐 환경 구성에 이슈로 발생한 것

**⑧ v3: 빌드 중 권한 재설정 및 검사**

```bash
$ nano Dockerfile

$ cat Dockerfile
FROM nginx:alpine

COPY --chmod=644 web/index.html /usr/share/nginx/html/index.html

RUN chmod 644 /usr/share/nginx/html/index.html \
    && test "$(stat -c '%a' /usr/share/nginx/html/index.html)" = "644"

EXPOSE 80

$ docker build --no-cache -t codyssey-web:v3 .
[+] Building 1.2s (8/8) FINISHED                                                                                                          docker:default
 => [internal] load build definition from Dockerfile                                                                                                0.0s
 => => transferring dockerfile: 336B                                                                                                                0.0s
 => [internal] load metadata for docker.io/library/nginx:alpine                                                                                     1.0s
 => [internal] load .dockerignore                                                                                                                   0.0s
 => => transferring context: 2B                                                                                                                     0.0s
 => [internal] load build context                                                                                                                   0.0s
 => => transferring context: 304B                                                                                                                   0.0s
 => CACHED [1/3] FROM docker.io/library/nginx:alpine@sha256:785835ea65db4bba6f021acc0f3be91b0fef76a53fe41f8588c6698d2aed0606                        0.0s
 => [2/3] COPY --chmod=644 web/index.html /usr/share/nginx/html/index.html                                                                          0.0s
 => [3/3] RUN chmod 644 /usr/share/nginx/html/index.html     && test "$(stat -c '%a' /usr/share/nginx/html/index.html)" = "644"                     0.1s
 => exporting to image                                                                                                                              0.1s
 => => naming to docker.io/library/codyssey-web:v3                                                                                                  0.0s
```

**배운 내용:**

- `RUN chmod 644`: 이미지 빌드 중 권한 재설정
- `test`: 권한 값 확인
- v2 원인 추정: ACL·BuildKit 상호작용(대조 빌드 미실시)

**⑨ v3 이미지 내부 권한 검증**

```bash
$ docker run --rm --network none \
  --entrypoint sh codyssey-web:v3 \
  -c 'stat -c "%a %U:%G %n" /usr/share/nginx/html/index.html'
644 root:root /usr/share/nginx/html/index.html
```

**배운 내용:**

- `stat`: v3 이미지 내부 `index.html` 권한이 `644`임을 확인

**⑩ v3 컨테이너 웹 서버 HTTP 응답 검증 — 성공**

```bash
$ docker run -d --name codyssey-web-v3 codyssey-web:v3

$ docker exec codyssey-web-v3 \
  wget -S -O- http://127.0.0.1/
4137eb20813329e9233896cd58b1d220cf7c936b8cd2af6fe5fb5c1a35a0ffdf
Connecting to 127.0.0.1 (127.0.0.1:80)
  HTTP/1.1 200 OK
  Server: nginx/1.31.8
  Date: Thu, 29 Oct 2026 07:28:33 GMT
  Content-Type: text/html
  Content-Length: 201
  Last-Modified: Thu, 29 Oct 2026 07:05:15 GMT
  Connection: close
  ETag: "6ae2f02b-c9"
  Accept-Ranges: bytes

writing to stdout
-                    100% |********************************|   201  0:00:00 ETA
written to stdout

<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>Codyssey Web Server</title>
</head>
<body>
  <h1>Hello, Codyssey!</h1>
  <p>My first custom Docker image.</p>
</body>
</html>
```

**배운 내용:**

- HTTP `200 OK`: 작성한 HTML이 정상 반환됨
