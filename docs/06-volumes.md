# 6. 바인드 마운트 및 Docker 볼륨 영속성

## 개요

호스트에서 파일을 수정하면 컨테이너에도 반영되는지 바인드 마운트로 확인합니다. 이어 Docker 볼륨을 사용해 컨테이너를 삭제한 뒤에도 데이터가 남는지 검증합니다.

## 실습 시작

4번에서 만든 `codyssey-web:v3` 이미지를 재사용합니다.
프로젝트 루트에서 터미널을 열어 시작합니다.

**① 실습 위치와 기존 리소스 확인**

```bash
$ mkdir -p practice/06-volumes
$ cd practice/06-volumes
$ pwd
/home/사용자명/projects/codyssey/1-1_codyssey-workstation/practice/06-volumes
$ sudo docker ps -a --filter name=codyssey-bind --filter name=codyssey-vol
[sudo: authenticate] 비밀번호:
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
$ sudo docker volume ls --filter name=codyssey-6-data
DRIVER    VOLUME NAME
```

**배운 내용:**

- `cd`, `pwd`: 디렉터리 이동과 현재 위치 확인
- `docker ps -a`: 동일한 이름의 실습 컨테이너가 없는지 확인
- `--filter name=...`: 지정한 이름에 맞는 컨테이너나 볼륨만 확인
- `docker volume ls`: 동일한 이름의 볼륨이 없는지 확인

## 6-1. 바인드 마운트 — 호스트 파일 수정 반영

**① 호스트 파일을 만들고 컨테이너에 연결**

```bash
$ mkdir -p bind
$ echo "Before bind mount" > bind/note.txt
$ sudo docker run -d \
  --name codyssey-bind \
  -v "$PWD/bind:/data:ro" \
  --entrypoint sh \
  codyssey-web:v3 -c 'sleep 3600'
a69d28ef27d39608eca97a53328b657c0f3222138fb48ac9426e6ff24265fd48
```

**배운 내용:**

- `mkdir -p`: 호스트에 폴더 생성
- `-v "$PWD/bind:/data:ro"`: 호스트의 `bind` 폴더를 컨테이너의 `/data`에 **읽기 전용** 연결. `$PWD`는 현재 폴더의 절대경로로 바뀝니다.
- **왜 바인드 마운트일까?** 호스트에서 파일을 고치면 이미지를 다시 빌드하지 않고도 컨테이너에서 바뀐 내용을 볼 수 있기 때문입니다.
- `--entrypoint sh`: 웹 서버 대신 파일 확인을 위한 셸 실행

**② 컨테이너에서 최초 파일 확인**

```bash
$ sudo docker exec codyssey-bind cat /data/note.txt
Before bind mount
```

**③ 호스트 파일을 바꾸고 변경 반영 확인**

```bash
$ echo "After bind mount" > bind/note.txt
$ sudo docker exec codyssey-bind cat /data/note.txt
After bind mount
```

**배운 내용:**

- `docker exec`: 실행 중인 컨테이너에서 명령 실행
- **결과:** 호스트 파일 수정 내용이 컨테이너 내부에 바로 반영됨

## 6-2. Docker 볼륨 — 컨테이너 삭제 후 데이터 유지

**① 볼륨 생성**

```bash
$ sudo docker volume create codyssey-6-data
codyssey-6-data
```

**배운 내용:**

- `docker volume create`: Docker가 관리하는 볼륨 생성
- **왜 볼륨을 따로 만들까?** 컨테이너를 지워도 데이터는 남길 수 있도록 실행 환경과 저장 공간을 분리하기 위해서입니다.
- **볼륨은 백업일까?** 아닙니다. 볼륨을 삭제하거나 디스크에 문제가 생기면 데이터도 잃을 수 있습니다. 중요한 데이터는 볼륨 내용을 호스트의 별도 위치나 다른 저장 장치에 주기적으로 백업해야 합니다.

**② 첫 번째 컨테이너에서 파일 저장**

```bash
$ sudo docker run \
  --name codyssey-vol1 \
  -v codyssey-6-data:/data \
  --entrypoint sh \
  codyssey-web:v3 \
  -c 'echo "Hello Docker Volume" > /data/note.txt && cat /data/note.txt'
Hello Docker Volume
```

**배운 내용:**

- `-v codyssey-6-data:/data`: 볼륨을 컨테이너에 연결
- `sh -c`: 셸에 전달한 문자열을 명령으로 실행
- `&&`: 앞 명령이 성공한 경우에만 다음 명령 실행
- `echo`, `cat`: 파일에 데이터 저장하고 내용 확인

**③ 첫 번째 컨테이너 삭제**

```bash
$ sudo docker rm codyssey-vol1
codyssey-vol1
```

**④ 두 번째 컨테이너에서 같은 데이터 읽기**

```bash
$ sudo docker run \
  --name codyssey-vol2 \
  -v codyssey-6-data:/data \
  --entrypoint cat \
  codyssey-web:v3 /data/note.txt
Hello Docker Volume
```
새로운 컨테이너도 같은 볼륨에서 `Hello Docker Volume`을 읽습니다. 컨테이너를 삭제해도 이름 있는 Docker 볼륨의 데이터는 유지됩니다.

**배운 내용:**

- `--entrypoint cat`: 기본 웹 서버 실행 명령 대신 `cat`으로 `/data/note.txt` 내용을 출력하고 종료합니다.
- `docker rm`: 컨테이너 삭제

## 6-3. 실습 후 리소스 정리

**① 컨테이너 중지 및 삭제 (실제 로그)**

```bash
$ sudo docker stop codyssey-bind
codyssey-bind
$ sudo docker rm codyssey-bind codyssey-vol2
codyssey-bind
codyssey-vol2
```

**배운 내용:**

- `docker stop`: 실행 중인 컨테이너 중지
- `docker rm`: 실습용 컨테이너 삭제

**② 볼륨 삭제**

```bash
$ sudo docker volume rm codyssey-6-data
codyssey-6-data
```
바인드 마운트 변경 반영과 볼륨 영속성 검증에 성공했고, 실습용 컨테이너·볼륨 정리도 완료했습니다.

**배운 내용:**

- `docker volume rm`: 필요 없는 Docker 볼륨 삭제

