# 1-1 Codyssey 개발 워크스테이션 구축

터미널, 파일 권한, Docker, Git의 기본 동작을 직접 실습하고 명령어와 출력 결과를 기록하는 코디세이 과제입니다.

## 시작하기

GitHub에서 저장소를 클론한 뒤 **프로젝트 루트**에서 터미널을 열어 실습합니다.

```bash
git clone https://github.com/rhtlemfwkd-tech/1-1_codyssey-workstation.git
cd 1-1_codyssey-workstation
```

각 실습 문서는 프로젝트 루트에서 시작합니다.

**1~6번**은 각 문서의 「실습 시작」에 나온 `mkdir -p practice/...`로 실습 폴더를 만든 다음 `cd practice/...`로 이동합니다.

**7번**은 Git 저장소 생성·원격 연결 과정이므로 별도 프로젝트 폴더에서 진행한 기록입니다.

## 1. 프로젝트 개요

### 1-1. 실행 환경

| 항목 | 실습 환경 |
| --- | --- |
| OS | Ubuntu 26.04.1 LTS |
| Shell | bash |
| Docker Engine | 29.9.0 |
| Git | 2.53.0 |

### 1-2. 디렉터리 구조

```text
1-1_codyssey-workstation/
├── .gitignore
├── README.md
├── app/
│   ├── Dockerfile
│   └── web/
│       └── index.html
└── docs/
    ├── 01-terminal.md
    ├── 02-permissions.md
    ├── 03-docker.md
    ├── 04-dockerfile.md
    ├── 05-port-mapping.md
    ├── 06-volumes.md
    ├── 07-git-github.md
    ├── evidence/   (VS Code 연동 증거 이미지 3장)
    └── reference/  (원본 과제 지침·트러블슈팅 4건·실행 로그 원본)

# practice/는 새 실습 시 생성하고 Git에서는 제외
```

## 2. 필수 수행 체크리스트

- [x] **1. 터미널 조작 로그** — [실습 기록](docs/01-terminal.md)
  - 현재 위치 및 목록 확인(숨김 파일 포함)
  - 디렉터리 이동·생성, 파일 복사·이름 변경·삭제
  - 파일 내용 확인 및 빈 파일 생성
  - 명령어와 실제 출력 결과를 문서에 기록
  - `practice/01-terminal/memo/drafts/archive` 하위 폴더와 실습 파일을 직접 만든 뒤 상대·절대경로 재현

- [x] **2. 파일 및 디렉터리 권한 실습** — [실습 기록](docs/02-permissions.md)
  - 파일과 디렉터리 각각 1개 이상 권한 변경
  - `chmod`, `ls -l`로 변경 전후의 권한 비교
  - 소유자·그룹·기타 사용자 권한 확인

- [x] **3. Docker 운영 및 검증** — [실습 기록](docs/03-docker.md)
  - `docker --version`으로 설치 확인 (미설치 시 간편 설치)
  - 이미지 다운로드 및 목록 조회: `docker pull`, `docker images`
  - 컨테이너 실행·중지·목록: `docker run`, `docker stop`, `docker ps -a`
  - 로그·리소스 확인: `docker logs`, `docker stats`
  - 실습용 컨테이너 삭제 및 공개 이미지 선택적 정리: `docker rm`, `docker image rm`
  - Docker 미설치 Ubuntu용 공식 간편 설치 절차 안내

- [x] **4. Dockerfile 기반 웹 서버 컨테이너** — [실습 기록](docs/04-dockerfile.md)
  - 3번에서 공개 이미지로 컨테이너 실행, 4번에서 웹 서버 이미지 제작
  - `docker exec`를 이용한 컨테이너 내부 웹 요청 확인
  - 기존 `nginx:alpine` 이미지 기반으로 HTML 콘텐츠를 교체한 커스텀 이미지 제작
  - `FROM`, `COPY`, `RUN`, `EXPOSE`의 목적 및 빌드·실행 로그 기록
  - 과거 403 오류 분석 및 해결 기록은 별도 참고 문서로 보존

- [x] **5. 포트 매핑 및 접속 증거** — [실습 기록](docs/05-port-mapping.md)
  - `docker run -p`로 호스트 포트와 컨테이너 포트 연결
  - `curl -i http://127.0.0.1:18080/`의 HTTP 200 및 HTML 전체 응답 기록

- [x] **6. 바인드 마운트 및 Docker 볼륨 영속성** — [실습 기록](docs/06-volumes.md)
  - 바인드 마운트로 호스트 파일의 변경 반영 확인
  - Docker 볼륨 생성 및 컨테이너 연결
  - 컨테이너 삭제 전후 데이터 유지 여부를 명령어와 출력으로 검증

- [x] **7. Git 설정 및 VS Code–GitHub 연동 (실습 수행 확인)** — [실습 기록](docs/07-git-github.md)
  - Ubuntu: GitHub CLI 설치·로그인, 작성자 설정, `main` 최초 커밋·Push·공개 전환
  - Windows VS Code: GitHub `rhtlemfwkd-tech` 로그인, HTTPS Clone, Source Control 커밋 그래프·`main` 브랜치, Trust Folder 승인 확인
  - **제출 화면 증거 완료:** [Clone URL](docs/evidence/07-vscode-clone-url.png), [Source Control](docs/evidence/07-vscode-source-control.png), [GitHub 계정](docs/evidence/07-vscode-github-account.png) 3장 원격 저장소 업로드 확인
  - **Push 인증 검증:** Windows Git Credential Manager에 이전 계정이 선택되어 발생한 403 해결 후 `8c8e375`까지 Push 성공
  - Git 미설치 PC 안내는 미실습 재현 절차로 구분. Remote-SSH 및 GitHub SSH 키 인증은 **선택·미실습**

## 3. 트러블슈팅 참고 기록

- **트러블슈팅 1:** [Docker/BuildKit의 ACL 처리로 인한 HTTP 403 오류](docs/reference/04-dockerfile-history.md)
- **트러블슈팅 2:** [헤드리스 SSH 환경에서 별도 기기로 GitHub CLI 인증](docs/reference/07-headless-github-auth.md)
- **트러블슈팅 3:** [GitHub 웹 수정과 로컬 커밋 충돌 — Git Rebase로 해결](docs/reference/07-git-rebase-conflict.md)
- **트러블슈팅 4:** [Windows Git 자격 증명 계정 불일치로 발생한 403 오류 해결](docs/reference/07-windows-git-auth-403.md)

## 4. 보너스 과제 (선택)

- **Docker Compose 기본:** `compose.yaml`로 단일 서비스를 정의하고 `up`·`down` 사용
- **Compose 멀티 컨테이너:** 웹 서버와 보조 서비스 간 연결 및 네트워크 통신 확인
- **Compose 운영:** `ps`·`logs`로 서비스 상태 점검
- **환경 변수:** Dockerfile 또는 Compose를 통해 서버 설정과 코드를 분리
- **GitHub SSH 인증:** HTTPS 대신 SSH 키로 인증하는 방법 실습
- **VS Code Remote-SSH (자율 추가 실습):** Windows VS Code에서 Ubuntu 서버에 접속해 프로젝트를 여는 방법 실습. GitHub SSH 키 인증과는 별도 기능

