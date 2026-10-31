# 트러블슈팅 2 — 헤드리스 Ubuntu 서버에서 GitHub CLI 로그인

## 문제

**환경:** GUI 브라우저가 없는 Ubuntu 서버에 SSH로 원격 접속해 GitHub CLI(`gh`) 인증을 진행했습니다.

일반적인 `gh auth login` 명령어는 브라우저 열기를 기대하지만, **서버 자체에는 인증 화면을 표시할 브라우저가 없었습니다.** 

그렇지만 GitHub가 계정 인증과 기기 승인 과정을 요구하였으므로 방법을 생각해야했습니다.

**목표:** 헤드리스환경에 원격으로 접속한상태로 웹브라우저를 통한 GitHub 계정 인증하기.

## 원인 및 판단

GitHub CLI의 **기기 인증(Device Authorization)**은 CLI가 실행되는 장치와 웹 인증을 수행하는 장치가 같을 필요가 없습니다.

따라서 헤드리스 서버가 직접 브라우저를 열 수 없는 것이 인증 불가능을 뜻하지 않습니다.

원격 터미널에서 코드를 받고, 별도 화면이 있는 PC 또는 스마트폰의 브라우저에서 승인하면 되었습니다.

## 해결 과정 및 실제 검증 로그

### 1. 기존 인증 로그아웃 확인

실제 실습 환경에는 이전 계정의 GitHub CLI 인증이 남아 있어서 먼저 로그아웃했습니다. 처음부터 GitHub CLI에 로그인한 적 없는 환경이라면 이 단계는 필요 없습니다.

```bash
$ gh auth logout
✓ Logged out of github.com account ****1749

$ gh auth status
You are not logged into any GitHub hosts. To log in, run: gh auth login
```

### 2. 헤드리스 모드로 인증 요청

```bash
$ gh auth login
? What account do you want to log into? GitHub.com
? What is your preferred protocol for Git operations on this host? HTTPS
? How would you like to authenticate GitHub CLI? Login with a web browser
! First copy your one-time code: [일회용 코드 비공개]
Press Enter to open github.com in your browser...
```

`BROWSER=true`는 서버에서 브라우저 실행을 시도하지 않도록 하는 설정입니다.
인증 코드가 서버의 터미널에 표시되었습니다.

### 3. 별도 브라우저에서 승인

별도 화면이 있는 기기에서 다음 과정을 진행했습니다.

1. `https://github.com/login/device` 접속
2. 계정 `rhtlemfwkd-tech`로 로그인돼 있는지 확인
3. 서버 터미널에 표시된 일회용 코드 입력
4. **GitHub CLI의 요청 권한 확인 후 승인**
5. 추가 이메일 인증 완료

서버 터미널에 표시된 **코드와 브라우저 인증에 사용한 계정이 일치하는지** 확인한 뒤 승인했습니다.

### 4. 터미널에서 인증 성공 확인

```bash
✓ Authentication complete.
- gh config set -h github.com git_protocol https
✓ Configured git protocol
! Authentication credentials saved in plain text
✓ Logged in as rhtlemfwkd-tech
```

**결과:** 서버에 브라우저를 설치하지 않고도 새 계정의 GitHub CLI 인증을 완료했습니다.
이후 해당 계정에서 원격 GitHub 저장소에 최초 Push까지 성공했습니다.

## 배운 내용

- 헤드리스 서버의 CLI 작업 중에서도 브라우저 인증을 서로 다른 장치에서 진행할 수 있었습니다.
