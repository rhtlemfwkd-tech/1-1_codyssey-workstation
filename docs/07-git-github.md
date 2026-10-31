# 7. Git 설정, GitHub CLI 및 VS Code 연동

## 개요

Git에서 변경 이력을 관리하고 GitHub 저장소에 업로드하는 과정을 익힙니다. 계정 인증과 원격 저장소 연결을 진행하고 VS Code에서도 연동 상태를 확인합니다.

## 7-1. Git 설치 확인과 새 PC 설치 방법

**Git 설치 확인 및 설치**

```bash
$git --version
bash: git: 명령을 찾을 수 없음
$sudo apt install git
$git --version
git version 2.53.0
```

**배운 내용:**
- **Git**은 로컬 버전 관리 도구입니다
- `sudo apt install git`: git 설치 명령어
- `git --version`: git 설치 버전 확인 명령어 

## 7-2. GitHub CLI 설치 — 실제 실행 기록

**① 미설치 상태 확인**

```bash
$ gh --version
bash: gh: 명령을 찾을 수 없음
```

**② Ubuntu 패키지로 설치**

```bash
$ sudo apt install gh
설치할 패키지:
  gh

요약:
  업그레이드: 0, 설치: 1, 삭제: 0, 업그레이드 안 함: 83
  다운로드 크기: 8,609 kB
  필요한 공간: 37.5 MB / 921 GB 사용 가능

받기:1 https://esm.ubuntu.com/apps/ubuntu resolute-apps-security/main amd64 gh amd64 2.46.0-4ubuntu0.26.04.1~esm1 [8,609 kB]
내려받기 8,609 k바이트, 소요시간 4초 (2,213 k바이트/초)
Selecting previously unselected package gh.
Preparing to unpack .../gh_2.46.0-4ubuntu0.26.04.1~esm1_amd64.deb ...
Unpacking gh (2.46.0-4ubuntu0.26.04.1~esm1) ...
gh (2.46.0-4ubuntu0.26.04.1~esm1) 설정하는 중입니다 ...
Processing triggers for man-db (2.13.1-1build1) ...

$ gh --version
gh version 2.46.0 (2026-06-18 Ubuntu 2.46.0-4ubuntu0.26.04.1~esm1)
https://github.com/cli/cli/releases/tag/v2.46.0
```

**배운 내용:**
- `gh`는 터미널에서 GitHub 인증, 저장소 생성·조회 등을 수행하는 GitHub CLI입니다, Git과는 별개의 프로그램입니다.
- `sudo apt install gh`: gh 설치 명령어
- `gh --version`: gh 설치 버전 확인 명령어 

## 7-3. GitHub 로그인

**① 기존 인증 상태 확인 및 로그아웃**

```bash
$ gh auth status
github.com
  ✓ Logged in to github.com account ****1749
  - Active account: true
  - Git operations protocol: https
  [토큰 정보 생략]

$ gh auth logout
✓ Logged out of github.com account ****1749

$ gh auth status
You are not logged into any GitHub hosts. To log in, run: gh auth login
```
실습 이전의 로그인 인증정보가 남아있어 별도로 인증정보를 확인 후 로그아웃 하였습니다.

**배운 내용:**
- `gh auth status`: gh 인증 상태확인
- `gh auth logout -h github.com -u 계정`: 특정 계정 로그아웃

**② 헤드리스 원격 서버에서 기기 인증**

```bash
$ gh auth login
? What account do you want to log into? GitHub.com
? What is your preferred protocol for Git operations on this host? HTTPS
? How would you like to authenticate GitHub CLI? Login with a web browser
! First copy your one-time code: [일회용 코드 비공개]
Press Enter to open github.com in your browser...
✓ Authentication complete.
- gh config set -h github.com git_protocol https
✓ Configured git protocol
! Authentication credentials saved in plain text
✓ Logged in as rhtlemfwkd-tech
```

Ubuntu 서버는 헤드리스이므로, **별도 화면이 있는 기기의 브라우저**에서 `https://github.com/login/device`를 열어 서버 터미널의 일회용 코드를 입력했습니다.
로그인 계정은 `rhtlemfwkd-tech`였고, 추가 이메일 인증을 마친 뒤 서버 터미널에서 성공 메시지를 확인했습니다.

이 과정의 문제·원인·해결을 [트러블슈팅 2: 헤드리스 GitHub CLI 인증](reference/07-headless-github-auth.md)에 정리했습니다.

## 7-4. 새 로컬 Git 저장소와 첫 커밋 — 실제 기록

기존 1~6번 과제 파일은 보존하면서, Git 이력은 복사하지 않은 **새 작업 디렉터리**에서 시작했습니다. 다음은 이 PC에서 실제 수행한 명령입니다.

**① 기존 Git 저장소에서 추적 중인 파일을 새 폴더에 복사**

프로젝트 루트폴더에서 다음 명령어를 실행합니다.

```bash
$ pwd
/home/사용자명/projects/codyssey/1-1_codyssey-workstation

$ git init -b main
/home/사용자명/projects/codyssey/1-1_codyssey-workstation 안의 빈 깃 저장소를 다시 초기화했습니다
$ git status
## 아직 커밋이 없습니다, 위치: < main
?? .gitignore
?? README.md
?? docs/
?? practice/
```

**배운 내용:**
- `git init -b main`: 현재 폴더에서 Git 저장소의 시작 브랜치를 `main`으로 지정합니다.
- `git status`: 현재 깃 상태를 확인합니다.

**② 새 계정에 맞는 작성자 설정**

```bash
$git config user.name "본인의 계정"
$git config user.email "본인의 GitHub 이메일 주소"

$git config user.name
"본인의 계정"
$git config user.email
"본인의 GitHub 이메일 주소"
```

**배운 내용:**
- `git config user.name`: 깃 저장소에 작성할 작성자 계정을 설정하거나 확인합니다.
- `git config user.email`: 깃 저장소에 작성할 작성자 이메일을 설정하거나 확인합니다.

```bash
$ git config --list
[시스템·전역 설정과 개인 식별 정보가 포함된 부분 생략]
core.repositoryformatversion=0
core.filemode=true
core.bare=false
core.logallrefupdates=true
user.name=rhtlemfwkd-tech
user.email=247567252+rhtlemfwkd-tech@users.noreply.github.com
remote.origin.url=https://github.com/rhtlemfwkd-tech/1-1_codyssey-workstation.git
remote.origin.fetch=+refs/heads/*:refs/remotes/origin/*
branch.main.remote=origin
branch.main.merge=refs/heads/main
$ git branch --show-current
main
```
**배운 내용:**
- `git config --list`: 저장소의 설정정보를 확인합니다.

**③ README의 새 저장소 주소 수정 후 첫 커밋**

```bash
$ nano README.md
$ git add .
$ git status -sb
## 아직 커밋이 없습니다, 위치: < main
A  .gitignore
A  README.md
A  docs/01-terminal.md
A  docs/02-permissions.md
A  docs/03-docker.md
A  docs/04-dockerfile.md
A  docs/05-port-mapping.md
A  docs/06-volumes.md
A  docs/07-git-github.md
A  docs/reference/04-dockerfile-history-original.md
A  docs/reference/04-dockerfile-history.md
A  docs/reference/guide-original
A  practice/01-terminal/.gitkeep
A  practice/02-permissions/.gitkeep
A  practice/03-docker/.gitkeep
A  practice/04-dockerfile/.gitkeep
A  practice/05-port-mapping/.gitkeep
A  practice/06-volumes/.gitkeep
A  practice/07-git-github/.gitkeep

$ git commit -m "Initial Codyssey workstation project"
[main (최상위-커밋) bbc3e10] Initial Codyssey workstation project
 19 files changed, 1879 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 README.md
 create mode 100644 docs/01-terminal.md
 create mode 100644 docs/02-permissions.md
 create mode 100644 docs/03-docker.md
 create mode 100644 docs/04-dockerfile.md
 create mode 100644 docs/05-port-mapping.md
 create mode 100644 docs/06-volumes.md
 create mode 100644 docs/07-git-github.md
 create mode 100644 docs/reference/04-dockerfile-history-original.md
 create mode 100644 docs/reference/04-dockerfile-history.md
 create mode 100644 docs/reference/guide-original
 create mode 100644 practice/01-terminal/.gitkeep
 create mode 100644 practice/02-permissions/.gitkeep
 create mode 100644 practice/03-docker/.gitkeep
 create mode 100644 practice/04-dockerfile/.gitkeep
 create mode 100644 practice/05-port-mapping/.gitkeep
 create mode 100644 practice/06-volumes/.gitkeep
 create mode 100644 practice/07-git-github/.gitkeep

$ git log -1 --oneline
bbc3e10 (HEAD -> main) Initial Codyssey workstation project
```

**배운 내용:**
- `git add`: 수정한 파일을 스테이징합니다.
- `git commit`: 스테이징한 해당 상태를 이력으로 남깁니다.
- `git log`: 커밋 내역을 확인합니다.

## 7-5. GitHub 저장소 연결 및 최초 Push — 실제 기록

**① 새 저장소 생성 시도와 이름 충돌**

```bash
$ gh repo create rhtlemfwkd-tech/1-1_codyssey-workstation \
  --public \
  --source=. \
  --remote=origin \
  --push
GraphQL: Name already exists on this account (createRepository)

$ git remote -v
$ git status -sb
## main
```
**배운 내용:**
- `gh repo create`: GitHub에 새 저장소를 생성합니다.
- `--public`: 공개 저장소로 생성합니다.
- `--source=. --remote=origin --push`: 현재 폴더를 원본으로 원격 저장소를 등록하고 Push까지 시도합니다.
- `git remote -v`: 등록된 원격 저장소 주소를 확인합니다.
- `git status -sb`: 현재 브랜치와 파일 변경 상태를 간략하게 확인합니다.

**② 기존 이름의 저장소가 비어 있는지 확인**

```bash
$ gh repo view rhtlemfwkd-tech/1-1_codyssey-workstation \
  --json nameWithOwner,isEmpty,isPrivate,url
{
  "isEmpty": true,
  "isPrivate": true,
  "nameWithOwner": "rhtlemfwkd-tech/1-1_codyssey-workstation",
  "url": "https://github.com/rhtlemfwkd-tech/1-1_codyssey-workstation"
}
```

**배운 내용:**
- `gh repo view`: GitHub 저장소의 정보를 조회합니다.
- `--json`: 조회할 정보의 항목을 지정합니다.
  
당시 웹에서 이미 생성해서 저장소는 이미 만들어져 있었지만 비어 있는 비공개 저장소였기 때문에, 삭제하거나 새로 만들지 않고 연결하기로 했습니다.

저장소가 만들어지지 않았다면 **① 새 저장소 생성 시도와 이름 충돌** 시도만으로 생성이 완료될 겁니다.

**③ 원격 연결 및 첫 Push**

```bash
$ git remote add origin https://github.com/rhtlemfwkd-tech/1-1_codyssey-workstation.git
$ git push -u origin main
오브젝트 나열하는 중: 19, 완료.
오브젝트 개수 세는 중: 100% (19/19), 완료.
Delta compression using up to 32 threads
오브젝트 압축하는 중: 100% (17/17), 완료.
오브젝트 쓰는 중: 100% (19/19), 26.71 KiB | 26.71 MiB/s, 완료.
Total 19 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To https://github.com/rhtlemfwkd-tech/1-1_codyssey-workstation.git
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.
```
**배운 내용:**
- `git remote add origin`: 현재 로컬 저장소에 origin이라는 이름으로 원격 저장소 주소를 등록합니다.
- `git push -u origin main`: 로컬 main 브랜치를 원격 저장소에 Push하고, 이후 Push·Pull에 사용할 추적 브랜치를 설정합니다.
- **Git과 GitHub의 차이:** Git은 내 컴퓨터에서 변경 이력을 관리하고, GitHub는 그 이력을 원격에 올려 다른 사람과 공유하는 곳입니다.

**④ 공개 전환**

```bash
$ gh repo edit rhtlemfwkd-tech/1-1_codyssey-workstation --visibility public
✓ Edited repository rhtlemfwkd-tech/1-1_codyssey-workstation
$ gh repo view rhtlemfwkd-tech/1-1_codyssey-workstation --json isPrivate
{
  "isPrivate": false
}
```
**배운 내용**:
- `gh repo edit`: 기존 GitHub 저장소의 설정을 수정합니다.
- `--visibility public`: 저장소를 공개로 전환합니다.

**후속 변경사항:** GitHub 웹에서 직접 고친 커밋과 로컬 문서 수정 커밋이 충돌해 `git push`가 거절됐지만, `git pull --rebase origin main` 이후 README 충돌을 수동 해결해 정상 Push했습니다.

실제 오류·해결·검증 로그는 [트러블슈팅 3: Git Rebase 충돌 해결](reference/07-git-rebase-conflict.md)에 기록했습니다.

## 7-6. Windows VS Code–GitHub 연동 — 실제 화면으로 검증 완료

**실습 환경:** Windows 11 ARM환경의 VS Code에서 GitHub 계정 `rhtlemfwkd-tech`로 인증하고, 공개 저장소를 **HTTPS로 Clone**했습니다.

**① VS Code의 GitHub 계정 인증**
1. VS Code 홈페이지에서 VS Code를 설치 및 실행했습니다. 
2. 시작화면에서 VS Code에서 `Continue with GitHub`를 선택했습니다.
3. 브라우저의 `Authorize Visual Studio Code` 화면에서 `rhtlemfwkd-tech` 계정을 선택하고 인증 과정을 진행했습니다.
4. VS Code **Accounts(계정)** 메뉴에 `rhtlemfwkd-tech (GitHub)`가 표시된 것을 확인했습니다.

**② Windows Git 설치 및 GitHub 저장소 Clone**

Git을 사용하기 위해 열어본 VS Code의 Source Control 영역에 Download Git for Windows가 표시되어 Git for Windows를 설치했습니다.

설치 후 Source Control에서 Clone Repository → Clone from URL을 선택하고 다음 주소를 입력했습니다.

```text
Clone from URL:
https://github.com/rhtlemfwkd-tech/1-1_codyssey-workstation.git
```

Windows의 `codyssey` 폴더를 복제 대상 상위 디렉터리로 지정하고, 복제한 폴더를 VS Code에서 열었습니다. Trust Folder도 허용했습니다.

**③ Source Control 연동 결과**

VS Code의 Source Control에서 Changes 목록이 비어 있는 것을 확인했습니다.

또한 Graph에서 main 브랜치와 기존 커밋 이력을 확인했습니다.

```text
main — Document Git rebase conflict resolution
       Clarify repository status message in documentation
       Correct Git practice records
       Update 07-git-github.md
       ...
       Initial Codyssey workstation project
```
**배운 내용**:

- `Source Control`: Git 저장소의 변경사항과 버전 관리 상태를 확인합니다.
- `Changes`: 아직 커밋하지 않은 파일 변경사항을 표시합니다.
- `Graph`: 브랜치와 커밋 이력을 시각적으로 보여줍니다.
- `main`: 현재 저장소에서 사용하는 기본 브랜치입니다.

**④ VS Code 연동 화면 증거**

실습 과정에서 캡처한 화면 3장을 GitHub 저장소의 docs/evidence/ 폴더에 업로드했습니다.

- [Clone URL 입력 화면](evidence/07-vscode-clone-url.png)
- [Source Control의 main 브랜치와 커밋 그래프](evidence/07-vscode-source-control.png)
- [Accounts 메뉴의 rhtlemfwkd-tech GitHub 로그인](evidence/07-vscode-github-account.png)

**⑤ Windows Git Push 및 403 오류 해결**

VS Code 연동 화면을 캡처한 후, 이미지 3장을 GitHub 원격 저장소에 올리기 위해 Push를 시도했습니다.

하지만 이전에 사용하던 GitHub 계정이 Windows Git 인증에 남아 있어 403 오류가 발생했습니다.

**최초 Push 실패**
```powershell
PS> git push origin main
remote: Permission to rhtlemfwkd-tech/1-1_codyssey-workstation.git denied to rhgo1749.
fatal: unable to access 'https://github.com/rhtlemfwkd-tech/1-1_codyssey-workstation.git/': The requested URL returned error: 403
```

**인증 계정 변경 후 재시도**
```powershell
PS> git config --local credential.https://github.com.username rhtlemfwkd-tech
PS> git push origin main
info: please complete authentication in your browser...
```

**Push 성공 결과**
```text
To https://github.com/rhtlemfwkd-tech/1-1_codyssey-workstation.git
   ac11f07..8c8e375  main -> main
```
GitHub 증거 이미지 3장도 원격 저장소에 올라간 것을 확인했습니다.

**배운 내용:**
- `git push origin main`: 로컬 main 브랜치의 커밋을 원격 저장소에 업로드합니다.
- `403 Permission denied`: 인증된 계정에 해당 저장소의 쓰기 권한이 없을 때 발생할 수 있는 오류입니다.
- `git config --local credential.https://github.com.username`: 현재 저장소에서 GitHub HTTPS 인증에 사용할 사용자 이름을 지정합니다.
- `Git Credential Manager`: Git의 인증 정보를 관리하고 브라우저 인증을 지원합니다.

**트러블슈팅 요약 (실제 기록):**
- **가설:** VS Code 로그인은 맞지만 Git Push에는 이전 계정의 인증이 사용됐을 수 있습니다.
- **확인:** `git remote -v`의 주소는 정상인데, `git push`에서 이전 계정으로 `403`이 발생했습니다.
- **조치 및 결과:** 저장소별 인증 사용자 설정 후 브라우저에서 올바른 계정으로 인증하자 Push가 성공했습니다.

명령어와 실제 출력은 [트러블슈팅 4 — Windows Git Credential Manager 403](reference/07-windows-git-auth-403.md)에 정리했습니다.
