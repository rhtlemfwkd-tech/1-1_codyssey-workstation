# 4번 트러블슈팅 — NGINX HTTP 403 Forbidden

이 문서는 4번 Dockerfile 기본 실습 중 **실제 발생한 HTTP 403 오류의 조사와 해결 과정**만 기록합니다. 아래는 과거 환경의 실측 로그이며, 새 환경에서는 같은 오류가 재현되지 않을 수 있습니다.

명령과 출력 전체를 포함한 수정 전 자료는 [원본 실행 기록](04-dockerfile-history-original.md)에 별도로 보존했습니다.

## 문제

NGINX 기본 이미지에 HTML을 복사한 첫 이미지(`codyssey-web:v1`)는 **빌드와 컨테이너 실행에 성공**했으나, 내부 HTTP 요청이 실패했습니다.

**당시 v1 Dockerfile의 핵심:**

```dockerfile
FROM nginx:alpine
COPY web/index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

**실제 실패 로그:**

```bash
$ docker exec codyssey-web wget -qO- http://127.0.0.1/
wget: server returned error: HTTP/1.1 403 Forbidden
```

이미지가 정상적으로 빌드된 것과 파일을 NGINX가 읽을 수 있는지는 별개의 문제였습니다.

## 원인 가설

이미지는 빌드됐지만 NGINX가 HTML 파일을 읽을 권한이 없어 `403`이 발생했을 것으로 예상했습니다.

## 확인

### v1: 파일 권한과 오류 로그

컨테이너의 NGINX 사용자 및 HTML 파일 권한을 확인했습니다.

```bash
$ docker exec codyssey-web sh -c 'id nginx; ls -l /usr/share/nginx/html/index.html'
uid=101(nginx) gid=101(nginx) groups=101(nginx),101(nginx)
-rw-rw----    1 root     root           201 Oct 29 07:05 /usr/share/nginx/html/index.html
```

오류 로그도 같은 원인을 가리켰습니다.

```text
open() "/usr/share/nginx/html/index.html" failed (13: Permission denied)
```

- HTML은 `root:root` 소유, 권한은 `660`이었습니다.
- `nginx` 사용자(UID 101)에게는 해당 파일 읽기 권한이 없었습니다.
- 따라서 당시 HTTP `403 Forbidden`의 직접 원인은 파일 접근 권한 부족입니다.

### v2: 권한 지정 후 재확인

Dockerfile의 `COPY`에 `--chmod=644`를 추가했습니다.

```dockerfile
COPY --chmod=644 web/index.html /usr/share/nginx/html/index.html
```

v2 이미지는 빌드됐지만 HTTP 403 오류가 지속됐습니다.

```bash
$ docker exec codyssey-web-v2 wget -qO- http://127.0.0.1/
wget: server returned error: HTTP/1.1 403 Forbidden
```

컨테이너와 이미지 자체에서 확인한 권한도 `660`이었습니다.

```bash
$ docker run --rm --network none \
  --entrypoint sh codyssey-web:v2 \
  -c 'stat -c "%a %U:%G %n" /usr/share/nginx/html/index.html'
660 root:root /usr/share/nginx/html/index.html
```

이미지와 컨테이너 ID가 일치했고, 마운트 목록도 `[]`였습니다. 즉, 실행 중 마운트가 해당 파일을 덮어쓴 상황은 아니었습니다.

### v2 원인 조사: ACL과 BuildKit

호스트의 HTML 파일에서 확장 ACL을 확인했습니다.

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

**확인한 업스트림 결함:** [Moby #49766 — Setting an ACL with setfacl breaks COPY --chmod](https://github.com/moby/moby/issues/49766)은 ACL이 설정된 파일에서 `COPY --chmod`의 권한 지정이 제대로 적용되지 않는 문제를 **버그(`kind/bug`, `area/builder/buildkit`)**로 보고합니다. [BuildKit 개발자 설명](https://github.com/moby/moby/issues/49766#issuecomment-2837024195)에 따르면, 빌드 컨텍스트에서 복사된 `system.posix_acl_access` 확장 속성이 보존되면서 `stat.mode`에 적용한 `--chmod` 권한과 충돌할 수 있습니다. 즉, 파일에 `644`를 지정해도 ACL이 최종 접근 권한에 영향을 줄 수 있는 **이미 알려진 Docker/BuildKit 결함**입니다.

**이번 v2와의 연결:** (1) 호스트 파일에 확장 ACL이 존재, (2) `COPY --chmod=644`를 지정했지만 이미지 내부 파일 권한은 `660`, (3) 마운트 없이 이미지 자체에서도 같은 권한 확인, (4) 빌드 단계에서 `RUN chmod 644`를 추가한 v3에서는 `644`와 HTTP `200` 확인. 이 네 가지 증거가 위 결함의 발생 방식과 일치합니다. 따라서 **v2의 유력 원인을 기존 Docker/BuildKit ACL 처리 버그로 특정**했습니다. 단, ACL을 제거한 대조 빌드를 수행하지 않았으므로 해당 환경에서 이 버그가 유일한 원인이었다는 독립적인 인과 검증까지 완료한 것은 아닙니다.

## 해결/대안

### v3: 빌드 단계에서 권한 수정

v3에서는 `COPY --chmod=644`에 더해 빌드 도중 파일 권한을 다시 설정하고 검사했습니다.

```dockerfile
RUN chmod 644 /usr/share/nginx/html/index.html \
    && test "$(stat -c '%a' /usr/share/nginx/html/index.html)" = "644"
```

**기존 실습 검증 결과:**

```bash
$ docker run --rm --network none \
  --entrypoint sh codyssey-web:v3 \
  -c 'stat -c "%a %U:%G %n" /usr/share/nginx/html/index.html'
644 root:root /usr/share/nginx/html/index.html
```

컨테이너 내부 요청에서 `HTTP/1.1 200 OK`와 작성한 HTML이 반환됐습니다. HTML 원문과 일반적인 실행 방법은 [4번 기본 실습](../04-dockerfile.md)에서 확인할 수 있습니다.

## 결과와 배운 점

| 버전 | 시도 | 확인 결과 |
| --- | --- | --- |
| v1 | HTML 파일 복사 | `660` 권한, HTTP `403` |
| v2 | `COPY --chmod=644` | 빌드 성공, 파일은 여전히 `660`, HTTP `403` |
| v3 | `RUN chmod 644` 추가 | 최종 권한 `644`, HTTP `200` |

- 이미지 빌드 성공과 HTTP 접근 가능 여부는 별도 검증이 필요합니다.
- 오류가 나면 먼저 컨테이너 로그와 파일 권한을 확인합니다.
- **v2에서 찾아낸 원인:** 기존 [Docker/BuildKit ACL 처리 버그 #49766](https://github.com/moby/moby/issues/49766)와 실측 증상이 일치했습니다. 업스트림 개발자의 동작 설명까지 확인했고, `RUN chmod 644`로 해결했습니다. (ACL 제거 대조 빌드는 수행하지 않음.)
