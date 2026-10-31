# 1. 터미널 조작

## 개요

터미널에서 폴더와 파일을 만들고, 복사·이동·삭제하는 기본 명령을 익힙니다. 명령 결과를 확인하면서 절대 경로와 상대 경로의 차이도 알아봅니다.

프로젝트 루트에서 시작합니다. `practice/`는 Git에서 제외되므로 처음 실행할 때 실습 폴더를 만듭니다.

```bash
mkdir -p practice/01-terminal
cd practice/01-terminal
```

**배운 내용:**
- `mkdir -p`: 디렉터리 생성
- `cd`: 작업 디렉터리 이동

## 1-1. 파일과 디렉터리

**폴더·파일 생성**

```bash
$ mkdir -p memo/drafts/archive
$ echo "Hello Codyssey" > hello.txt
$ echo "Codyssey memo (inside repository)" > memo/notes.txt
$ echo "Nested archive memo" > memo/drafts/archive/history.txt
$ touch .hidden.txt
```

**배운 내용:**
- `echo`와 `>`: 파일에 내용 저장
- `touch`: 빈 파일 생성

**파일 내용 확인 및 편집**

```bash
$ nano hello.txt
$ cat hello.txt
Hello Codyssey
```

**배운 내용:**
- `nano`: 파일 편집
- `cat`: 파일 내용 확인

**복사·이름 변경·삭제**

```bash
$ cp hello.txt hello.txt.bak
$ mv hello.txt.bak renamed.txt
$ ls renamed.txt
renamed.txt
$ rm renamed.txt
$ ls hello.txt
hello.txt
```

**배운 내용:**
- `cp`: 복사, `mv`: 이동·이름 변경
- `rm`: 삭제, `ls`: 목록 확인

**빈 파일 생성·내용 저장**

```bash
$ touch empty.txt
$ echo "Hello" > empty.txt
$ cat empty.txt
Hello
```

**배운 내용:**
- `touch`: 빈 파일 생성
- `>`: 기존 파일 내용 덮어쓰기

**숨김 파일과 하위 디렉터리 확인**

```bash
$ ls -a
.  ..  .gitkeep  .hidden.txt  empty.txt  hello.txt  memo

$ ls -R memo
memo:
drafts
notes.txt

memo/drafts:
archive

memo/drafts/archive:
history.txt
```

**배운 내용:**
- `ls -a`: 숨김 파일 포함 목록
- `ls -R`: 하위 디렉터리까지 표시

## 1-2. 절대 경로와 상대 경로

**상대 경로로 파일 조회**

```bash
$ cat memo/notes.txt
Codyssey memo (inside repository)
$ cat memo/drafts/archive/history.txt
Nested archive memo
```

**배운 내용:**
- 상대 경로: 현재 디렉터리를 기준으로 파일 조회

**디렉터리 이동과 `pwd` 출력**

```bash
$ cd ~/projects/codyssey/1-1_codyssey-workstation
$ cd practice/01-terminal
$ pwd
/home/사용자명/projects/codyssey/1-1_codyssey-workstation/practice/01-terminal

$ cd memo/drafts/archive
$ pwd
/home/사용자명/projects/codyssey/1-1_codyssey-workstation/practice/01-terminal/memo/drafts/archive
$ cat history.txt
Nested archive memo

$ cd ../../..
$ pwd
/home/사용자명/projects/codyssey/1-1_codyssey-workstation/practice/01-terminal
```

**배운 내용:**
- `pwd`: 현재 위치의 절대 경로 출력
- `cd ../../..`: 상위 디렉터리로 세 단계 이동

※ 절대 경로는 당시 PC의 값으로, 다른 PC에서는 달라집니다.
