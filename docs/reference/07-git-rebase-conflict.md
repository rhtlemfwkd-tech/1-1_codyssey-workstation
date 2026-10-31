# 트러블슈팅 3 — GitHub 웹 수정과 로컬 커밋 충돌 해결 (Git Rebase)

## 문제

**환경:** 로컬에서 7번 실습 기록과 README를 수정해 커밋한 뒤 `git push`를 실행했지만, GitHub의 `main`에 로컬에 없는 커밋이 먼저 반영되어 Push가 거절됐습니다.

**실제 실행 기록 — 첫 번째 오류**

```bash
$ git add README.md docs/07-git-github.md
$ git commit -m "Correct Git practice records"
[main 072eb75] Correct Git practice records
 2 files changed, 54 insertions(+), 14 deletions(-)

$ git push
To https://github.com/rhtlemfwkd-tech/1-1_codyssey-workstation.git
 ! [rejected]        main -> main (fetch first)
error: 레퍼런스를 'https://github.com/rhtlemfwkd-tech/1-1_codyssey-workstation.git'에 푸시하는데 실패했습니다
hint: Updates were rejected because the remote contains work that you do not
hint: have locally.
```

## 원인 가설과 확인

로컬 커밋 `072eb75`를 만들기 전에 GitHub 웹에서 직접 수정한 파일들이 원격에 반영되었지만, 로컬은 그 커밋들을 가져오지 않은 상태였습니다.

원격의 새로운 커밋 3개를 확인했습니다.

| 원격 커밋 | GitHub에서 변경한 내용 |
| --- | --- |
| `e19f8d1` | `README.md`의 중복 문서 링크 삭제 |
| `ab6bdaa` | `docs/04-dockerfile.md`에 nano 저장 방법 추가 |
| `fc127a3` | `docs/07-git-github.md` 문장 및 재현 안내 보완 |

로컬 커밋도 `README.md`와 `docs/07-git-github.md`를 변경했으므로, 두 작업의 이력을 합치는 과정에서 충돌할 가능성이 있었습니다.

**원인:** 원격 `main`과 로컬 `main`의 이력이 갈라져서 일반적인 `git push`로는 fast-forward 갱신을 할 수 없었습니다.

## 해결 과정과 검증

### 1. 원격 커밋을 먼저 반영하도록 리베이스

```bash
$ git pull --rebase origin main
https://github.com/rhtlemfwkd-tech/1-1_codyssey-workstation URL에서
 * branch            main       -> FETCH_HEAD
   6cfc4b4..fc127a3  main       -> origin/main
자동 병합: README.md
충돌 (내용): README.md에 병합 충돌
자동 병합: docs/07-git-github.md
error: 다음을 적용할(apply) 수 없습니다: 072eb75... Correct Git practice records
```

이 명령은 원격의 새 커밋들을 가져와 적용한 뒤 로컬 커밋 `072eb75`를 그 위에 다시 올리려고 시도했습니다.

이때 **`README.md`에서 내용 충돌**이 발생해 리베이스가 일시 중단됐습니다.

### 2. README 충돌 부분 수동 해결

`README.md`의 충돌 표시(`<<<<<<<`, `=======`, `>>>>>>>`)를 확인하고 아래 사항을 반영했습니다.

- **원격에서 삭제한 중복 링크 2개는 삭제된 상태로 유지**
- **로컬에서 고친 7번 실습 설명과 Public 전환 내용은 유지**
- **충돌 표시를 모두 제거**

`docs/07-git-github.md`는 자동 병합돼 추가 수동 충돌 처리가 필요하지 않았습니다. 기존 Dockerfile 문서의 nano 저장 안내도 보존했습니다.

### 3. 리베이스 계속 진행 및 Push 성공

**실제 실행 기록 — 최종 성공**

```bash
$ git add README.md
$ GIT_EDITOR=true git rebase --continue
[HEAD 분리됨 1c8ca35] Correct Git practice records
 2 files changed, 54 insertions(+), 14 deletions(-)
Successfully rebased and updated refs/heads/main.

$ git push
오브젝트 나열하는 중: 9, 완료.
오브젝트 개수 세는 중: 100% (9/9), 완료.
Delta compression using up to 32 threads
오브젝트 압축하는 중: 100% (5/5), 완료.
오브젝트 쓰는 중: 100% (5/5), 2.33 KiB | 2.33 MiB/s, 완료.
Total 5 (delta 3), reused 0 (delta 0), pack-reused 0 (from 0)
remote: Resolving deltas: 100% (3/3), completed with 3 local objects.
To https://github.com/rhtlemfwkd-tech/1-1_codyssey-workstation.git
   fc127a3..1c8ca35  main -> main

$ git status -sb
## main...origin/main
```

## 결과 및 배운 점

**해결 완료:** 로컬과 원격 `main`이 동기화돼었습니다.

- 충돌이 생겼을 때는 관련 파일을 열어 두 수정사항을 비교하고, **필요한 내용을 모두 보존한 뒤** `git add`와 `git rebase --continue`를 실행해야 합니다.
