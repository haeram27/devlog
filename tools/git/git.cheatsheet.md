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
    - [stash list 보기](#stash-list-보기)
    - [drop](#drop)
    - [pop](#pop)
    - [apply](#apply)

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

stash는 local repository에 기록되는 변경사항을 임시 저장하는 ref 이다.

### push

- stash의 기본 명령어, sub command 없이 stash 명령을 실행하면 push가 실행된다.
- push 기본적으로 HEAD와 다른 모든 tracked 변경을 스택에 저장하고, 워킹트리/인덱스를 HEAD로 되돌린다.
- untracked/ignored 파일은 기본적으로 대상이 되지 않으나 옵션을 통해서 추가할 수 있다.
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

### stash list 보기

```bash
git stash list
```

### drop

- stash coomit를 삭제한다.

```bash
git stash drop <stash-oid>
```

### pop

- stash commit을 working tree에 반영하고 stash stack에서 삭제한다.

```bash
git stash pop <stash-oid>
```

### apply

- stash commit을 working tree에 반영하고 stash stack에는 그대로 보존한다.

```bash
git stash apply <stash-oid>
```
