# git 자주 사용하는 명령어

## 로컬에서 리모트 브랜치 생성/삭제 하기

```bash
# local 브랜치(feature01) 생성
git checkout -b feature01

# local 브랜치로 부터 remote 브랜치(feature01) 생성
# local과 remote 브랜치는 자동으로 upstream 관계로 연결 됨
git push -u origin feature01

# upstream 관계 설정 - remote 브랜치를 local 브랜치의 remote-tracking-branch로 설정하기
# push 할 때 -u 옵션을 사용했으면 이 단계(upstream 설정)는 건너뛰기 가능
# .git/config 파일에 heads/feature01과 remotes/origin/feature01의 merge관계 명시됨
# --set-upstream-to 와 -u 는 같은 옵션
git branch -u origin/feature01
git branch --set-upstream-to origin/feature01

# 브랜치 삭제 전 삭제 대상이 아닌 브랜치로 이동
git checkout develop

# local feature01 브랜치 삭제
git branch --delete feature01

# local 브랜치 강제 삭제(-D) - 저장되지않은 작업내용이 남아 브랜치가 삭제되지 않는경우
git branch -D feature01

# remote branch
git push origin :feature01
```