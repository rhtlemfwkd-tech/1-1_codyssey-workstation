# 5. 포트 매핑 및 접속 확인

## 실습 시작

4번 실습기록에서 만든 `codyssey-web:v3` 도커 이미지를 사용합니다.

**① 실습 폴더 이동**

```bash
$ mkdir -p practice/05-port-mapping
$ cd practice/05-port-mapping
$ pwd
/home/사용자명/projects/codyssey/1-1_codyssey-workstation/practice/05-port-mapping
```

**배운 내용:**

- `cd`: 폴더 이동
- `pwd`: 현재 디렉터리 확인

## 5-1. 포트 매핑

**① 기존 이미지와 컨테이너 확인**

```bash
$ sudo docker images codyssey-web
[sudo: authenticate] 비밀번호:
                                                                                                                                    i Info →   U  In Use
IMAGE             ID             DISK USAGE   CONTENT SIZE   EXTRA
codyssey-web:v3   43be4a8518b2       93.6MB         26.3MB

$ sudo docker ps -a --filter name=codyssey-web-port
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

**배운 내용:**

- `docker images`: 4번 실습에서 만든 이미지 확인
- `docker ps -a`: 동일한 이름의 컨테이너가 있는지 확인

**② 호스트 18080번 포트와 컨테이너 80번 포트 연결**

```bash
$ sudo docker run -d \
  --name codyssey-web-port \
  -p 127.0.0.1:18080:80 \
  codyssey-web:v3
3b6f4c6759e403cb2ee5f8a9f4322b6eada1d54344f00b921eeaca378041c9b5

$ sudo docker ps --filter name=codyssey-web-port
CONTAINER ID   IMAGE             COMMAND                   CREATED         STATUS         PORTS                     NAMES
3b6f4c6759e4   codyssey-web:v3   "/docker-entrypoint.…"   5 seconds ago   Up 5 seconds   127.0.0.1:18080->80/tcp   codyssey-web-port
```

**배운 내용:**

- `docker run -d`: 컨테이너를 백그라운드로 생성과 동시에 실행
- `--name`: 컨테이너 이름 지정
- `-p 127.0.0.1:18080:80`: **호스트의 18080번 포트**를 **컨테이너의 80번 포트**에 연결
- `docker ps`: 실행 상태와 포트 연결 확인
- **왜 포트를 연결할까?** 컨테이너 안에서만 열려 있는 웹 서버를 호스트에서도 이용하기 위해서입니다. 이번에는 호스트의 18080번으로 들어온 요청을 컨테이너의 80번으로 보냅니다.
- **네트워크 네임스페이스란?** 기본 네트워크 모드에서 컨테이너마다 네트워크 공간을 따로 쓰는 구조입니다. 그래서 컨테이너의 80번 포트를 호스트에서 쓰려면 `-p`로 연결해야 합니다.

`127.0.0.1`로 바인딩했으므로 서버 자신의 로컬 접속만 허용합니다.
다른 PC의 브라우저에서는 별도 SSH 포트 포워딩 없이는 직접 접속할 수 없습니다.

## 5-2. HTTP 접속 확인

**① curl로 호스트 포트 접속**

```bash
$ curl -i http://127.0.0.1:18080/
HTTP/1.1 200 OK
Server: nginx/1.31.8
Date: Fri, 30 Oct 2026 06:13:34 GMT
Content-Type: text/html
Content-Length: 200
Last-Modified: Thu, 29 Oct 2026 10:54:02 GMT
Connection: keep-alive
ETag: "6ae325ca-c8"
Accept-Ranges: bytes

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

- `curl`: 웹 서버에 HTTP 요청을 보내 응답 확인
- `-i`: HTTP 응답 헤더도 함께 출력
- `HTTP/1.1 200 OK`: 웹 서버가 요청을 정상 처리함

**검증 결과:** 호스트의 `127.0.0.1:18080`에 연결하여 NGINX의 HTML을 받았습니다.
이로써 **포트 매핑과 HTTP 접속 증거 확보 완료**입니다.

**② 포트 사용 상태 확인**

```bash
$ ss -ltn '( sport = :18080 )'
State    Recv-Q   Send-Q     Local Address:Port       Peer Address:Port
LISTEN   0        4096           127.0.0.1:18080           0.0.0.0:*
```

**배운 내용:**

- `ss -ltn`: 현재 TCP 포트의 수신 대기(`LISTEN`) 상태를 확인합니다.
- `( sport = :18080 )`: 호스트의 18080번 포트만 조회합니다.
- **결과:** 기존에 생성했던 도커외의 18080번 포트의 다른 수신 대기 소켓과 점유 프로세스가 조회되지 않았습니다.

## 5-3. 실습 컨테이너 정리 (선택)

아래 명령은 **정리할 때 실행하는 안내**입니다. 현재 제출된 로그에는 실행 결과가 없으므로 정리 완료로 기록하지 않습니다.

```bash
$ sudo docker stop codyssey-web-port
$ sudo docker rm codyssey-web-port
```

**배운 내용:**

- `docker stop`: 컨테이너 중지
- `docker rm`: 중지된 컨테이너 삭제

다만 `codyssey-web:v3` 이미지는 이후 실습에서도 사용할 수 있으므로 삭제하지 않는걸 권장드립니다.
