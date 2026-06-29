# git cheatsheet

- [git cheatsheet](#git-cheatsheet)
  - [tag](#tag)
    - [local tag list 보기](#local-tag-list-보기)
    - [remote tag ref list 보기](#remote-tag-ref-list-보기)
    - [remote 전체 ref list 보기](#remote-전체-ref-list-보기)
    - [remote로 부터 전체 tag 다운로드](#remote로-부터-전체-tag-다운로드)
    - [tag ref를 기준으로 local branch 생성](#tag-ref를-기준으로-local-branch-생성)
  - [stash](#stash)
    - [push](#push)
      - [push options](#push-options)
    - [list](#list)
    - [apply](#apply)
    - [drop](#drop)
    - [pop](#pop)
    - [stash 반영 되돌리기](#stash-반영-되돌리기)

---

## tag

### local tag list 보기

```bash
git tag -l
```

### remote tag ref list 보기

```bash
git ls-remote --tags origin
```

### remote 전체 ref list 보기

```bash
git ls-remote --refs origin
```

### remote로 부터 전체 tag 다운로드

```bash
git fetch origin --tags
```

### tag ref를 기준으로 local branch 생성

```bash
git checkout tags/<tag> -b <branch>
```

---

## stash

stash는 local repository에 기록되는 변경사항을 commit으로 만들어 임시 저장하는 ref 이다.
stash는 commit object를 생성하며 해당 object는 `.git/obejcts/`아래에 보관된다.
생성된 object는 `.git/refs/stash`에서 참조 된다.

### push

- stash의 기본 명령어, sub command 없이 stash 명령을 실행하면 push가 실행된다.
- push 기본적으로 working tree와 index의 `tracked 변경 사항`을 스택에 저장하고, 워킹트리/인덱스를 HEAD로 되돌린다.
- `untracked/ignored 파일은 기본적으로 대상이 되지 않으나` 옵션을 통해서 추가할 수 있다.
- untracked 파일/디렉터리 포함 필요 시 -u/--include-untracked
- ignored 파일 포함 필요 시 -a/--all (untracked + ignored)

```bash
git stash push -m "msg" -- <pathspec>
```

#### push options

```bash
git stash push -u : untracked 까지 포함
git stash push -a : ignored 파일까지 모두 포함
git stash push --keep-index : working tree 변경만 stash(인덱스는 유지)
git stash push --staged : staged 된 변경만 stash
git stash push -- <pathspec> : 특정 경로만 대상으로(기본은 tracked만, -u와 함께 쓰면 해당 경로의 untracked도 포함)
```

### list

```bash
git stash list
```

### apply

- stash commit을 working tree에 반영하고 stash stack에는 그대로 보존한다.

```bash
git stash apply <stash-oid>
```

### drop

- stash coomit를 삭제한다.

```bash
git stash drop <stash-oid>
```

### pop

- stash apply + drop
- stash commit을 working tree에 반영하고 stash stack에서 삭제한다.

```bash
git stash pop <stash-oid>
```

### stash 반영 되돌리기

- stash commit을 working tree에 반영하고 stash stack에는 그대로 보존한다.

```bash
// 가장 최근의 stash를 사용하여 패치를 만들고 해당 변경사항을 working tree로 부터 되돌린다.
$ git stash show -p | git apply -R

// stash 이름(ex. stash@{2} or 2)에 해당하는 stash를 이용하여 해당 변경사항을 working tree로 부터 되돌린다.
$ git stash show -p <statsh-oid> | git apply -R
```

되돌리기 명령을 분해하여 살펴보자.

- `git stash show -p <stash-oid>`는 stash object를 기반으로 file diff patch를 생성한다.
- 생성되는 `diff patch`는 linux에서 사용하는 diff patch 형석과 동일하다.
- `git apply -R` 명령은 diff patch 파일을 git working tree에 적용한다.

ex) diiff patch

```bash
$ git stash show -p
diff --git a/src/test/resources/log4j2.xml b/src/test/resources/log4j2.xml
index 10b1178075..fb8d7f66d1 100644
--- a/src/test/resources/log4j2.xml
+++ b/src/test/resources/log4j2.xml
@@ -25,11 +25,11 @@
        </Properties>
        <appenders>
                <Console name="console_log" target="SYSTEM_OUT">
-                       <EncodedPatternLayout pattern="%msg" />
+                       <PatternLayout pattern="%msg" />
                </Console>
                <RollingFile name="eppfile_log" fileName="${logFileName}"
                        filePattern="${logFilePattern}">
-                       <EncodedPatternLayout pattern="%msg" />
+                       <PatternLayout pattern="%msg" />
                        <Policies>
```
