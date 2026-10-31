# 4. Dockerfile로 웹 서버 만들기

## 개요

기존 `nginx:alpine` 이미지를 바탕으로 HTML을 교체한 나만의 웹 서버 이미지를 만듭니다. Dockerfile의 설정이 어떤 역할을 하는지 보고, 빌드·실행 후 웹페이지 응답으로 결과를 확인합니다.

프로젝트 루트에서 시작합니다. 새 클론에는 빈 `practice/` 폴더가 없으므로 먼저 실습 폴더를 만듭니다.

```bash
mkdir -p practice/04-dockerfile
```

다음 `cd`를 한 번 실행하고 순서대로 진행합니다.

```bash
$ cd practice/04-dockerfile
```

3번 실습에서 Docker 설치를 완료했다고 가정합니다.
제출·검증용으로 동일 내용을 추적하는 [Dockerfile 원본](../app/Dockerfile)과 [HTML 원본](../app/web/index.html)을 별도로 보존했습니다.

## 4-1. 웹 페이지 만들기

**① HTML 파일 작성**

```bash
$ mkdir -p web
$ nano web/index.html
```

아래 내용을 입력하고 저장합니다. nano 저장명령은 CTRL+O -> ENTER -> CTRL+X 

```html
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

- `mkdir -p`: 폴더 만들기
- `nano`: 파일 내용 편집

## 4-2. Dockerfile 작성 및 이미지 빌드

**① Dockerfile 작성**

```bash
$ nano Dockerfile
```

다음 내용을 입력하고 저장합니다.

```dockerfile
FROM nginx:alpine

COPY --chmod=644 web/index.html /usr/share/nginx/html/index.html

RUN chmod 644 /usr/share/nginx/html/index.html

EXPOSE 80
```

**배운 내용:**

- `FROM`: 사용할 기본 이미지
- `COPY`: 웹 페이지를 컨테이너에 복사
- `RUN`: 빌드 중 명령 실행
- `EXPOSE`: 컨테이너에서 사용하는 포트 표시(호스트에 자동 공개되지는 않음)
- **왜 NGINX를 쓸까?** 웹 서버를 처음부터 설치하지 않고 HTML만 교체해 같은 실행 환경을 쉽게 만들 수 있기 때문입니다.

**② 이미지 빌드**

```bash
$ sudo docker build -t codyssey-web:v3 .
[sudo: authenticate] 비밀번호:
[+] Building 1.7s (8/8) FINISHED                           docker:default
 => [internal] load build definition from Dockerfile                 0.0s
 => => transferring dockerfile: 263B                                 0.0s
 => [internal] load metadata for docker.io/library/nginx:alpine      1.1s
 => [internal] load .dockerignore                                    0.0s
 => => transferring context: 2B                                      0.0s
 => [internal] load build context                                    0.0s
 => => transferring context: 515B                                    0.0s
 => CACHED [1/3] FROM docker.io/library/nginx:alpine@sha256:785835e  0.0s
 => => resolve docker.io/library/nginx:alpine@sha256:785835ea65db4b  0.0s
 => [2/3] COPY --chmod=644 web/index.html /usr/share/nginx/html/ind  0.0s
 => [3/3] RUN chmod 644 /usr/share/nginx/html/index.html             0.1s
 => exporting to image                                               0.4s
 => => exporting layers                                              0.0s
 => => exporting manifest sha256:53ee72a29d4731f63a075e0acf9a4cc163  0.0s
 => => exporting config sha256:8715070cc701450f80ad349dad191c84067c  0.0s
 => => exporting attestation manifest sha256:d1b22dde89dd52be811d5d  0.0s
 => => exporting manifest list sha256:43be4a8518b27f6cac7975bcf08e6  0.0s
 => => naming to docker.io/library/codyssey-web:v3                   0.0s
 => => unpacking to docker.io/library/codyssey-web:v3                0.3s

$ sudo docker images codyssey-web
                                                     i Info →   U  In Use
IMAGE             ID             DISK USAGE   CONTENT SIZE   EXTRA
codyssey-web:v3   43be4a8518b2       93.6MB         26.3MB
```

**배운 내용:**

- `docker build -t`: 이미지를 빌드하고 이름 붙이기
- `docker images`: 만든 이미지 확인
- `.`: 현재 폴더를 빌드에 사용
- **이미지와 컨테이너는 별개입니다.** HTML을 바꿔 이미지를 다시 빌드해도 이미 실행 중인 컨테이너는 자동으로 바뀌지 않습니다. 새 이미지로 컨테이너를 다시 만들어야 합니다.

## 4-3. 웹 서버 실행 및 확인

**① 컨테이너 실행**

```bash
$ sudo docker run -d --name codyssey-web-v3 codyssey-web:v3
17f937c8c1baffb84a5821b8ac2592401a41c9d89e75b418e927948815e8a5d4

$ sudo docker ps --filter name=codyssey-web-v3
CONTAINER ID   IMAGE             COMMAND                   CREATED         STATUS         PORTS     NAMES
17f937c8c1ba   codyssey-web:v3   "/docker-entrypoint.…"   4 seconds ago   Up 4 seconds   80/tcp    codyssey-web-v3

```

**배운 내용:**

- `docker run -d`: 백그라운드로 컨테이너 실행
- `docker ps`: 실행 상태 확인

**② 웹 페이지 확인**

```bash
$ sudo docker exec codyssey-web-v3 wget -qO- http://127.0.0.1/
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

정상 실행되면 위에서 작성한 HTML이 출력됩니다.

**배운 내용:**

- `docker exec`: 실행 중인 컨테이너 안에서 명령 실행
- `wget`: 웹 서버에 요청해 응답 확인

**③ 컨테이너 종료 및 삭제**

※주의※ 5번 포트 매핑 실습에서 같은 `codyssey-web:v3` 이미지를 다시 사용하므로,

이미지 삭제방법은 알아만 두시고 삭제는 하지 말아주시길 바랍니다.

```bash
$ sudo docker stop codyssey-web-v3
codyssey-web-v3

$ sudo docker rm codyssey-web-v3
codyssey-web-v3

```

**배운 내용:**

- `docker stop`: 컨테이너 중지
- `docker rm`: 컨테이너 삭제

---

기존 환경에서 v1·v2의 HTTP 403 오류를 추적하고 v3에서 해결한 상세 로그는 [Dockerfile 트러블슈팅 기록(참고)](reference/04-dockerfile-history.md)에 보존했습니다. 새 클론에서는 파일 권한이 달라 403 오류가 재현되지 않을 수 있습니다.
