
# refname

참고: [gitrevisions](https://git-scm.com/docs/gitrevisions)

예를 들어 `master`는 일반적으로 `refs/heads/master`가 참조하는 커밋 객체를 의미합니다. `heads/master`와 `tags/master`를 모두 존재한다면 git 명령 사용시 `heads/master`를 직접 명시하여 사용할 수 있습니다. 모호한 경우, `<refname>`은 다음 규칙에서 첫 번째 매칭을 선택합니다.

1. If `$GIT_DIR/<refname>` exists, that is what you mean (this is usually useful only for HEAD, FETCH_HEAD, ORIG_HEAD, MERGE_HEAD, REBASE_HEAD, REVERT_HEAD, CHERRY_PICK_HEAD, BISECT_HEAD and AUTO_MERGE);

2. otherwise, `refs/<refname>` if it exists;

3. otherwise, `refs/tags/<refname>` if it exists;

4. otherwise, `refs/heads/<refname>` if it exists;

5. otherwise, `refs/remotes/<refname>` if it exists;

6. otherwise, `refs/remotes/<refname>/HEAD` if it exists.
    - HEAD:
      - names the commit on which you based the changes in the working tree.
      - 작업 트리의 변경 사항을 기반으로 한 커밋의 이름을 지정합니다.

    - FETCH_HEAD:
      - records the branch which you fetched from a remote repository with your last git fetch invocation.
      - 마지막 git 페치 호출과 함께 원격 저장소에서 가져온 지점을 기록합니다.

    - ORIG_HEAD:
      - is created by commands that move your HEAD in a drastic way (`git am`, `git merge`, `git rebase`, `git reset`), to record the position of the HEAD before their operation, so that you can easily change the tip of the branch back to the state before you ran them.
      - HEAD를 과감하게 움직이는 명령(`git am`, `git merge`, `git rebase`, `git reset`)에 의해 만들어지며 작업 전에 헤드 위치를 기록하여 분기 끝을 쉽게 변경하기 전에 상태로 다시 변경할 수 있습니다.

    - MERGE_HEAD
      - records the commit(s) which you are merging into your branch when you run git merge.
      - `git merge`을 실행할 때 지점에 병합하는 커밋을 기록합니다.

    - REBASE_HEAD:
      - during a rebase, records the commit at which the operation is currently stopped, either because of conflicts or an edit command in an interactive rebase.
      - `git rebase`를를 진행하는 동안 대화식 rebase에서 충돌 또는 편집 명령으로 인해 작업이 중지되는 커밋을 기록합니다.

    - REVERT_HEAD:
      - records the commit which you are reverting when you run git revert.
      - `git revert`를 실행할 때 되돌아가는 커밋을 기록합니다.

    - CHERRY_PICK_HEAD:
      - records the commit which you are cherry-picking when you run git cherry-pick.
      - `git cherry-pick`을 실행할 때 체리 피킹 커밋을 기록하십시오.

    - BISECT_HEAD:
      - records the current commit to be tested when you run git bisect --no-checkout.
      - `git bisect-no-checkout`을 실행할 때 현재 테스트를 거치기 위해 현재 커밋을 기록합니다.

    - AUTO_MERGE:
      - records a tree object corresponding to the state the ort merge strategy wrote to the working tree when a merge operation resulted in conflicts.
      - 병합 작업이 충돌을 일으킬 때 ORT 병합 전략이 작업 트리에 쓴 상태에 해당하는 트리 객체를 기록합니다.

    Note that any of the refs/* cases above may come either from the $GIT_DIR/refs directory or from the $GIT_DIR/packed-refs file. While the ref name encoding is unspecified, UTF-8 is preferred as some output processing may assume ref names in UTF-8.

@
    @ alone is a shortcut for HEAD.
