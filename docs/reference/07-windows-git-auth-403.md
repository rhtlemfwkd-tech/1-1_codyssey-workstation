# 트러블슈팅 4 — VS Code 로그인 계정과 Windows Git Push 인증 계정 불일치

## 문제

Windows VS Code에서 GitHub 계정 `rhtlemfwkd-tech`로 로그인하고 해당 계정의 공개 저장소를 HTTPS로 Clone했습니다.
Source Control에서도 `main` 브랜치와 커밋 그래프가 보였지만, 변경사항을 Push하자 VS Code에서 본인 저장소에 쓰기 권한이 없다는 안내와 **Fork 생성 제안**이 나왔습니다.

```text
You don't have permissions to push to "rhtlemfwkd-tech/1-1_codyssey-workstation" on GitHub.
Would you like to create a fork and push to it instead?
```

## 원인 가설

VS Code에 로그인한 계정과 실제 Git Push에 사용되는 인증 계정이 서로 달라서 발생한 문제로 의심했습니다.

## 확인

### 1. Windows Git CLI 인식 실패

VS Code 통합 PowerShell에서 Git 명령을 실행했으나, 처음에는 설치된 `git.exe`가 PATH에 없어 명령을 찾지 못했습니다.

```powershell
PS> git --version
git : 'git' 용어가 cmdlet, 함수, 스크립트 파일 또는 실행할 수 있는 프로그램 이름으로 인식되지 않습니다.
```

실행 파일을 직접 호출해 실제 설치 여부를 확인했습니다.

```powershell
PS> & "C:\Program Files\Git\cmd\git.exe" --version
git version 2.56.0.windows.4
PS> $env:Path += ";C:\Program Files\Git\cmd"
PS> git --version
git version 2.56.0.windows.4
```

`$env:Path` 설정을 통해 해당 파워쉘 세션에 git 명령을 등록했습니다.

### 2. 저장소·커밋 상태 확인

```powershell
PS> git remote -v
origin  https://github.com/rhtlemfwkd-tech/1-1_codyssey-workstation.git (fetch)
origin  https://github.com/rhtlemfwkd-tech/1-1_codyssey-workstation.git (push)

PS> git status -sb
## main...origin/main [ahead 1]
```

**판단:** 원격 저장소 URL은 올바르고, 로컬에 아직 Push되지 않은 커밋이 1개 있었습니다.

### 3. 실제 인증 계정 확인

```powershell
PS> git push origin main
remote: Permission to rhtlemfwkd-tech/1-1_codyssey-workstation.git denied to ****1749.
fatal: unable to access 'https://github.com/rhtlemfwkd-tech/1-1_codyssey-workstation.git/': The requested URL returned error: 403
```

**확인된 원인:** VS Code 자체 로그인은 `rhtlemfwkd-tech`였지만, Windows Git이 HTTPS Push할 때는 **이전 계정 `****1749`의 인증을 사용**하고 있었습니다.
따라서 해당 계정으로는 새 저장소에 쓰기 권한이 없었고 HTTP 403이 발생했습니다.

## 해결/대안

이 로컬 Git 저장소에서 사용할 GitHub 인증 사용자 이름을 명시했습니다.

```powershell
PS> git config --local credential.https://github.com.username rhtlemfwkd-tech
PS> git push origin main
info: please complete authentication in your browser...
```

이후 Git Credential Manager의 브라우저 인증 화면에서 **`rhtlemfwkd-tech` 계정을 선택하여 승인**했습니다.

### Push 성공 실측 로그

```text
Enumerating objects: 9, done.
Counting objects: 100% (9/9), done.
Delta compression using up to 8 threads
Compressing objects: 100% (7/7), done.
Writing objects: 100% (7/7), 126.88 KiB | 11.53 MiB/s, done.
Total 7 (delta 2), reused 0 (delta 0), pack-reused 0 (from 0)
remote: Resolving deltas: 100% (2/2), completed with 2 local objects.
To https://github.com/rhtlemfwkd-tech/1-1_codyssey-workstation.git
   ac11f07..8c8e375  main -> main
```

**결과:** 계정 인증 문제를 해결하고 Push가 정상 완료됐습니다. 이 Push에 포함된 VS Code 증거 이미지 3개도 GitHub의 `docs/evidence/` 경로에서 별도로 확인했습니다.

## 배운 점과 주의사항

- **VS Code의 GitHub 로그인**과 **Git Credential Manager가 HTTPS Git 작업에 사용하는 계정**은 서로 다를 수 있습니다.
- `403`이 나오면 에러의 **`denied to <계정>`** 부분을 먼저 확인합니다.
- `git remote -v`로 원격 URL을, `git status -sb`로 미전송 커밋을 확인한 다음 인증 문제를 해결합니다.
- `git config --local credential.https://github.com.username ...`로 로컬 Git 저장소에서 사용할 GitHub 인증 사용자 이름을 명시하였습니다.
- 계정별 인증이 혼재한 환경에서는 다른 저장소에 영향을 주지 않는 방식입니다.
