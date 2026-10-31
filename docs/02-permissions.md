# 2. 파일 및 디렉터리 권한

프로젝트 루트에서 실습용 파일과 폴더를 만듭니다. `practice/`는 Git에서 제외됩니다.

```bash
$ mkdir -p practice/02-permissions
$ cd practice/02-permissions
$ touch permission-file.txt
$ mkdir -p permission-dir
```

**배운 내용:**
- `touch`: 빈 파일 생성
- `mkdir -p`: 디렉터리 생성

## 2-1. 소유자만 접근하도록 설정

```bash
$ chmod 600 permission-file.txt
$ chmod 700 permission-dir
$ ls -l permission-file.txt
-rw-------+ 1 사용자명 사용자명 0 Oct 30 18:05 permission-file.txt
$ ls -ld permission-dir
drwx------+ 2 사용자명 사용자명 4096 Oct 30 18:05 permission-dir
```

**배운 내용:**

- `chmod 600`: 파일 소유자에게 읽기·쓰기 권한만 부여합니다.
- `chmod 700`: 디렉터리 소유자에게만 읽기·쓰기·탐색 권한을 부여합니다.
- `ls -l`은 파일 권한, `ls -ld`는 디렉터리 자체의 권한을 확인합니다.

## 2-2. 다른 사용자에게 읽기·탐색 허용

```bash
$ chmod 644 permission-file.txt
$ chmod 755 permission-dir
$ ls -l permission-file.txt
-rw-r--r--+ 1 사용자명 사용자명 0 Oct 30 18:05 permission-file.txt
$ ls -ld permission-dir
drwxr-xr-x+ 2 사용자명 사용자명 4096 Oct 30 18:05 permission-dir
$ stat -c '%a %A %n' permission-file.txt permission-dir
644 -rw-r--r-- permission-file.txt
755 drwxr-xr-x permission-dir
```

**배운 내용:**

- `chmod 644`: 소유자 외 사용자도 파일을 읽을 수 있습니다.
- `chmod 755`: 소유자 외 사용자도 디렉터리를 탐색할 수 있습니다.
- `stat`: 변경된 숫자 권한을 확인할 수 있습니다.

실제 출력에서 파일은 `600 → 644`, 디렉터리는 `700 → 755`로 변경됐습니다.

**참고:** `ls` 결과의 `+`는 추가 ACL이 있음을 나타냅니다. `stat`으로 기본 권한 모드를 확인했으며, ACL 규칙 전체를 검증한 것은 아닙니다. 이 출력은 실습 명령을 다시 실행해 확보한 실제 값이며 소유자·날짜는 환경마다 달라집니다.
