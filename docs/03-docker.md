# 3. Docker 기본 실습

Ubuntu에서 실행한 Docker 설치 점검과 컨테이너 조작 기록입니다. 명령과 주요 출력만 남겼으며, 다운로드 진행 표시 등 반복되는 부분은 발췌했습니다.

## 3-1. Docker 상태 확인

```bash
$ docker --version
Docker version 29.9.0, build f415da8

$ sudo docker info --format 'Server={{.ServerVersion}} Driver={{.Driver}} Running={{.ContainersRunning}}'
[sudo: authenticate] 비밀번호:
Server=29.9.0 Driver=overlayfs Running=11
```

**배운 내용:**
- `docker --version`: 클라이언트 버전 확인
- `docker info`: Docker 서버와 실행 상태 확인
- `--format`: 필요한 정보만 골라 출력하는 옵션

서버 버전 `29.9.0`, 드라이버 `overlayfs`, `Running=11`은 **조회 시점 제 시스템 전체** 컨테이너 수였습니다.

이번 PC에는 Docker가 **이미 설치**되어 있었습니다. 미설치 Ubuntu의 참고 설치 절차는 아래와 같으며 **실제로 실행한 로그가 아닙니다**.

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
rm get-docker.sh
```

관리자 권한으로 외부 스크립트를 실행하므로 설치 전 내용을 검토하고, 공유 서버에서는 공식 패키지 설치 절차를 우선합니다.

## 3-2. 이미지 다운로드 및 확인

**이미지 다운로드 (출력 발췌)**

```bash
$ sudo docker pull hello-world:latest
Status: Downloaded newer image for hello-world:latest
docker.io/library/hello-world:latest

$ sudo docker pull python:3.12-alpine
Status: Downloaded newer image for python:3.12-alpine
docker.io/library/python:3.12-alpine

$ sudo docker images hello-world
IMAGE                ID             DISK USAGE   CONTENT SIZE
hello-world:latest   c567b231f656       25.9kB         9.49kB
$ sudo docker images python
IMAGE                ID             DISK USAGE   CONTENT SIZE
python:3.12-alpine   5789aa194f75       74.7MB         18.7MB
```

**배운 내용:**
- `docker pull`: 이미지 다운로드
- `docker images`: 로컬 이미지 목록 확인
- **이미지는 실행용 설계도입니다.** 이미지를 받아 두면 같은 환경의 컨테이너를 여러 번 만들 수 있습니다.

이미지 ID와 크기는 버전에 따라 달라질 수 있습니다.

## 3-3. 컨테이너 실행 및 관리

**hello-world 실행과 종료 확인**

```bash
$ sudo docker run --name codyssey-hello hello-world:latest
Hello from Docker!
This message shows that your installation appears to be working correctly.

$ sudo docker ps -a --filter name=codyssey-hello
CONTAINER ID   IMAGE                COMMAND    CREATED          STATUS                      PORTS     NAMES
8bae412b1b7d   hello-world:latest   "/hello"   17 seconds ago   Exited (0) 16 seconds ago             codyssey-hello

$ sudo docker logs codyssey-hello
Hello from Docker!
This message shows that your installation appears to be working correctly.
이하생략
```

**배운 내용:**
- `docker run`: 컨테이너 실행
- `docker ps -a`: 종료된 컨테이너까지 확인
- `docker logs`: 컨테이너 로그 확인


**백그라운드 실행·로그·리소스·중지/재시작**

```bash
$ sudo docker run -d --name codyssey-ops python:3.12-alpine sh -c 'echo "Codyssey started"; sleep 3600'
f592e266e64a50393f968a543ea4cc5dab802c0541f363ec4b01bf5cd86b0fbe
$ sudo docker ps --filter name=codyssey-ops
CONTAINER ID   IMAGE                COMMAND                    CREATED          STATUS          PORTS     NAMES
f592e266e64a   python:3.12-alpine   "sh -c 'echo \"Codyss…"   26 seconds ago   Up 25 seconds             codyssey-ops
$ sudo docker logs codyssey-ops
Codyssey started

$ sudo docker stats --no-stream codyssey-ops
CONTAINER ID   NAME           CPU %     MEM USAGE / LIMIT   MEM %     NET I/O         BLOCK I/O   PIDS
f592e266e64a   codyssey-ops   0.00%     372KiB / 121.1GiB   0.00%     2.32kB / 126B   0B / 0B     1
$ sudo docker stop codyssey-ops
codyssey-ops
$ sudo docker start codyssey-ops
codyssey-ops
$ sudo docker stop codyssey-ops
codyssey-ops
```

**배운 내용:**
- `-d`: 백그라운드 실행
- `docker stats`: 리소스 사용량 확인
- `docker stop`·`start`: 중지·재시작

**Ubuntu 컨테이너 내부 진입**

```bash
$ sudo docker run --rm -it --network none ubuntu:24.04 bash
root@7a9c25f4e5b9:/# pwd
/
root@7a9c25f4e5b9:/# ls /
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr
root@7a9c25f4e5b9:/# echo "Hello from Ubuntu"
Hello from Ubuntu
root@7a9c25f4e5b9:/# exit
exit
```

**배운 내용:**
- `-it`: 컨테이너 내부 셸과 상호작용
- `--rm`: 종료 후 자동 삭제
- `--network none`: 컨테이너 네트워크 비활성화

**기존 실행 컨테이너에 `docker exec`**

```bash
$ sudo docker run -d --name codyssey-ubuntu-check ubuntu:24.04 sleep 300
46ca020bb1fd2e4a71ec4527e01b04787dc280e7b1c810e00ec4e8151cfb768b
$ sudo docker exec codyssey-ubuntu-check sh -c 'pwd; echo "Docker exec OK"'
/
Docker exec OK
$ sudo docker stop codyssey-ubuntu-check
codyssey-ubuntu-check
$ sudo docker rm codyssey-ubuntu-check
codyssey-ubuntu-check
```

**배운 내용:**
- `docker exec`: 실행 중인 컨테이너에서 새 명령 실행
- `docker attach`: 기존 프로세스 입출력에 연결하는 별도 기능(이번 실습에서 실행하지 않음)
- **차이:** `run`은 새 컨테이너를 만들고, `exec`는 실행 중인 컨테이너에서 명령을 추가로 실행합니다. `attach`는 이미 실행 중인 프로세스의 입출력에 연결합니다.

## 3-4. 실습 리소스 정리

```bash
$ sudo docker rm codyssey-hello codyssey-ops
codyssey-hello
codyssey-ops
$ sudo docker ps -a --filter name=codyssey
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

$ sudo docker image rm hello-world:latest python:3.12-alpine
Untagged: hello-world:latest
Deleted: sha256:c567b231f65614688711a75f31518a064258a51d442117a3e771a3a8fdba78ef
Untagged: python:3.12-alpine
Deleted: sha256:5789aa194f757888f41fa3d48e14b6feaa1b57c0af0b56fcb1372d6474d3bff0
```

**배운 내용:**
- `docker rm`: 컨테이너 삭제
- `docker image rm`: 이미지 삭제
- **왜 따로 지울까?** 컨테이너는 실제 실행 단위이고 이미지는 그 기반이므로, 컨테이너를 삭제해도 이미지는 남습니다.
