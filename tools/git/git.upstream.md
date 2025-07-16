# 정의: “upstream”이란?

현재 로컬 브랜치가 연결되어 있는 원격 브랜치
일반적으로 git pull, git push의 기본 대상이 되는 원격 브랜치를 말합니다.

## 용도

```text
git pull # upstream 브랜치에서 가져옴
git push # upstream 브랜치로 푸시
git status # upstream과의 차이 보여줌 (ahead, behind)
```

## upstream 설정 관련 명령어

```text
git switch --track <remote-repository>/<remote-branch>  
git push --set-upstream(-u) <remote-repository> <remote-branch>
git branch --set-upstream-to=<remote-repository>/<remote-branch>  
```

| 명령어 | 설명 |
| --- | --- |
| `git branch -vv`                               | 모든 로컬 브랜치의 upstream 브랜치와 상태 표시 |
| `git rev-parse --abbrev-ref @{u}`              | 현재 브랜치의 upstream 조회 |
| `git checkout -b feature origin/feature`       | 신규 로컬 브랜치를 생성하면서 upstream 브랜치 설정 |
| `git switch --track origin/feature`            | 신규 로컬 브랜치를 생성하면서 upstream 브랜치 설정 |
| `git branch --set-upstream-to=origin/feature`  | 수동으로 upstream 설정 |
| `git push -u origin feature`                   | 현재 로킬 브랜치를 push하면서 upstream 브랜치 설정 |
| `git push -u origin feature`                   | 현재 로킬 브랜치를 push하면서 upstream 브랜치 설정 |

## upstream 설정시 내부 동작

upstream을 설정하면 현재 브랜치(예: mybranch)에 대해서 다음의 설정이 추가 된다.

```text
[branch "mybranch"]
        remote = origin
        merge = refs/heads/mybranch
        rebase = true
```
