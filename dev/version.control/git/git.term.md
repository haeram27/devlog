# git terms

## branch

* 브랜치간 tracking을 위한 업스트림 관계
  * upstream 브랜치는 local 또는 remote에 위치할 수 있음

```text
downstream(tracking) branch <-(pull/fetch)- upstream branch
downstream(tracking) branch    -(push)->    upstream branch
```

* `upstream branch`의 위치가 remote repository인 경우에 local repository에는 `remote-tracking branch`라는 참조가 생성됨
* `remote-tracking branch`는 remote repository에 위치한 `upstream branch`의 상태 정보를 가진 local reposity의 참조 포인트 이며 push/pull/fetch 등 remote repository와 연결되는 작업이 발생했을 때 해당 시점의 `upstream branch`의 마지막 상태 정보로 업데이트됨

```text
| local repository                              | remote repository |
--------------------------------------------------------------------|
| tracking branch - remote-tracking branch(ref) | upstream branch   |
| main            - orgin/main                  | main              |
```

### tracking branch

Git에서 "추적 브랜치(tracking branch)"는 원격 저장소(remote repository)의 특정 브랜치와 직접적인 연결 관계를 맺고 있는 로컬 브랜치(local branch)를 의미합니다. 이 연결을 "업스트림(upstream)" 관계라고도 합니다.

주요 특징과 장점은 다음과 같습니다.

1. **간편한 Push & Pull**: 로컬 브랜치가 원격 브랜치를 추적하고 있으면, `git push`나 `git pull` 명령을 실행할 때 추가적인 인자 없이도 Git이 자동으로 어떤 원격 브랜치와 통신해야 할지 알 수 있습니다.
    * `git pull`: 추적하고 있는 원격 브랜치의 최신 변경 사항을 가져와 현재 로컬 브랜치에 병합(merge)합니다.
    * `git push`: 현재 로컬 브랜치의 변경 사항을 추적하고 있는 원격 브랜치로 보냅니다.

2. **상태 확인 용이**: `git status`나 `git branch -vv` 명령을 사용하면, 로컬 브랜치가 원격 브랜치에 비해 몇 개의 커밋을 앞서고(ahead) 있는지 또는 뒤처져(behind) 있는지 쉽게 확인할 수 있습니다. 이를 통해 로컬과 원격 저장소의 동기화 상태를 파악하기 좋습니다.

### 추적 브랜치는 어떻게 생성되나요?

추적 브랜치는 여러 가지 방법으로 생성될 수 있습니다.

* **`git clone` 사용 시**: 원격 저장소를 복제하면, 기본적으로 `main` 또는 `master` 브랜치가 생성됩니다. 이 로컬 브랜치는 자동으로 원격 저장소의 `origin/main` 또는 `origin/master` 브랜치를 추적하게 됩니다.

* **`git checkout` 사용 시**: 로컬에 존재하지 않는 브랜치 이름을 사용하여 체크아웃하면, Git은 동일한 이름의 원격 브랜치가 있는지 찾아봅니다. 만약 있다면, 해당 원격 브랜치를 추적하는 로컬 브랜치를 자동으로 생성하고 그 브랜치로 전환합니다.

    ```bash
    # origin/feature-A 브랜치를 추적하는 'feature-A' 로컬 브랜치를 생성하고 전환
    git checkout feature-A
    ```

* **`git branch` 명령으로 직접 설정**: 이미 존재하는 로컬 브랜치가 특정 원격 브랜치를 추적하도록 설정할 수도 있습니다.

    ```bash
    # 현재 브랜치가 origin/develop 브랜치를 추적하도록 설정
    git branch -u origin/develop

    # 또는 --set-upstream-to 옵션 사용
    git branch --set-upstream-to=origin/develop
    ```

요약하자면, 추적 브랜치는 원격 저장소와의 작업을 매우 편리하게 만들어주는 Git의 핵심 기능 중 하나입니다.

### remote-tracking-branch

### 1. 추적 브랜치 (Tracking Branch)

* **이것은 여러분이 직접 작업하는 '로컬 브랜치'입니다.** (`main`, `feature/login` 등)
* 이 로컬 브랜치는 특정 원격 브랜치(업스트림 브랜치)와 연결되어(추적하도록 설정되어) 있습니다.
* 이 연결 덕분에 `git pull`, `git push` 같은 명령을 간단하게 사용할 수 있습니다.
* **여러분이 직접 커밋하고 코드를 수정하는 브랜치입니다.**

### 2. 원격 추적 브랜치 (Remote-Tracking Branch)

* **이것은 일반적인 작업용 브랜치가 아닌 원격 저장소의 브랜치 상태를 '복사'해 놓은 '로컬의 읽기 전용 브랜치'(참조)입니다.**
* 이름은 보통 `[원격저장소이름]/[브랜치이름]` 형식을 가집니다 (예: `origin/main`, `origin/develop`).
* **여러분이 이 브랜치에 직접 커밋하거나 수정할 수 없습니다.**
* 이 브랜치의 역할은 마지막으로 원격 저장소와 통신했을 때(예: `git fetch`, `git pull`)의 상태를 로컬에 기록해두는 '책갈피'와 같습니다.
* `git fetch` 명령을 실행하면, Git은 원격 저장소에서 최신 정보를 가져와서 이 **원격 추적 브랜치**(`origin/main`)를 업데이트합니다. 여러분의 로컬 작업 브랜치(`main`)는 이때 변경되지 않습니다.

### 관계 정리

일반적인 워크플로우에서 이 둘의 관계는 다음과 같습니다.

```text
[원격 저장소의 'main' 브랜치] <-(fetch/pull)- [로컬의 'origin/main' 브랜치] <-(merge/rebase)- [로컬의 'main' 브랜치]
                                (push)                                                     (commit)
```

* **`main`**: **추적 브랜치** (로컬 작업 브랜치)
* **`origin/main`**: **원격 추적 브랜치** (원격 `main` 브랜치의 로컬 복사본/책갈피)

`git status`를 실행했을 때 "Your branch is ahead of 'origin/main' by 2 commits." 와 같은 메시지가 나오는 것을 본 적이 있을 것입니다.

이것은 여러분의 **추적 브랜치**(`main`)가 **원격 추적 브랜치**(`origin/main`)보다 2개의 커밋이 앞서 있다는 의미입니다. 즉, 로컬에서 2번 커밋한 후 아직 `push`하지 않은 상태를 말합니다.

**결론:**

* **추적 브랜치 (Tracking Branch)**: 내가 작업하는 로컬 브랜치.
* **원격 추적 브랜치 (Remote-Tracking Branch)**: 원격 브랜치의 상태를 보여주는 읽기 전용 로컬 참조. `origin/main`과 같은 형태.

### 3. Upstream branch

GitHub Copilot: "업스트림 브랜치(Upstream Branch)"는 로컬 브랜치가 추적하고 있는 원격 저장소의 브랜치를 의미합니다. 간단히 말해, 내 로컬 브랜치의 '기준점'이 되는 원격 브랜치입니다.

이전 질문의 "추적 브랜치(Tracking Branch)"와 같은 개념을 다른 관점에서 부르는 용어입니다.

* **로컬 브랜치**는 원격 브랜치를 **추적(track)**합니다. -> 이 로컬 브랜치를 **추적 브랜치**라고 부릅니다.
* **원격 브랜치**는 로컬 브랜치의 **업스트림(upstream)**이 됩니다. -> 이 원격 브랜치를 **업스트림 브랜치**라고 부릅니다.

#### 비유로 이해하기

강물을 생각하면 쉽습니다.

* **업스트림(Upstream)**: 강물의 상류. 물이 흘러내려오는 근원지입니다.
* **다운스트림(Downstream)**: 강물의 하류. 물이 흘러가는 곳입니다.

Git에서는 원격 저장소(예: `origin`)가 보통 '상류' 역할을 합니다.

* `git pull` 또는 `git fetch`: **업스트림 브랜치**(상류)에서 최신 변경 사항(물)을 내 로컬 브랜치(하류)로 가져옵니다.
* `git push`: 내 로컬 브랜치(하류)의 변경 사항(물)을 **업스트림 브랜치**(상류)로 올려보냅니다.

#### 업스트림 브랜치 설정

업스트림 브랜치는 주로 로컬에서 새로 만든 브랜치를 처음으로 원격 저장소에 푸시(push)할 때 설정합니다.

```bash
# my-feature 라는 새 브랜치를 만들고 작업했다고 가정
git push -u origin my-feature
# 또는 --set-upstream 옵션을 사용
git push --set-upstream origin my-feature
```

여기서 `-u` 또는 `--set-upstream` 옵션이 바로 로컬 `my-feature` 브랜치의 업스트림 브랜치를 원격 저장소 `origin`의 `my-feature` 브랜치로 지정하는 역할을 합니다.

이렇게 한번 설정하고 나면, 이후 해당 브랜치에서는 `git push`나 `git pull` 명령만으로도 Git이 자동으로 업스트림 브랜치를 찾아 작업을 수행해줍니다.

#### 업스트림 브랜치 확인

`git branch -vv` 명령을 사용하면 각 로컬 브랜치가 어떤 업스트림 브랜치를 추적하고 있는지 확인할 수 있습니다.

```bash
$ git branch -vv
  main                a1b2c3d [origin/main] Fix: some bug
* my-feature          e4f5g6h [origin/my-feature: ahead 1] Add new feature
  another-branch      i7j8k9l [origin/another-branch] Update docs
```

위 예시에서 `[origin/my-feature: ahead 1]` 부분은 `my-feature` 브랜치의 업스트림이 `origin/my-feature`이며, 로컬 브랜치가 원격보다 1개의 커밋을 앞서 있다는(ahead) 정보를 보여줍니다.

#### 업스트림 브랜치가 원격이 아닌 로컬 브랜치가 가능한가

Git에서 업스트림 브랜치는 기본적으로 "어디에서 변경 사항을 가져올 것인가(pull/fetch)"와 "어디로 변경 사항을 보낼 것인가(push)"의 기본 대상을 지정하는 역할을 합니다. 이 대상이 반드시 원격 저장소의 브랜치일 필요는 없습니다.

#### 로컬 브랜치를 업스트림으로 설정하는 경우

한 로컬 브랜치를 다른 로컬 브랜치의 업스트림으로 설정할 수 있습니다. 예를 들어, `main` 브랜치를 기반으로 `feature` 브랜치를 만들어서 작업하는 시나리오를 생각해 보겠습니다.

1. `feature` 브랜치로 이동합니다.

    ```bash
    git checkout feature
    ```

2. `main` 브랜치를 `feature` 브랜치의 업스트림으로 설정합니다.

    ```bash
    git branch --set-upstream-to=main
    # 또는 git branch -u main
    ```

#### 이렇게 설정하면 어떤 일이 발생하나요?

* **`git status`**: `feature` 브랜치에서 `git status`를 실행하면, 이제 원격 브랜치(`origin/main`)가 아닌 로컬 `main` 브랜치와 비교하여 커밋이 몇 개 앞서거나 뒤처졌는지 보여줍니다.

* **`git pull`**: `feature` 브랜치에서 `git pull`을 실행하면, 원격 저장소에서 변경 사항을 가져오는 대신 로컬 `main` 브랜치의 최신 커밋을 `feature` 브랜치로 병합(merge)하려고 시도합니다.

* **`git push`**: 이 설정은 `push` 동작에는 영향을 주지 않습니다. `push`는 기본적으로 원격 저장소로 보내는 동작이기 때문에, 업스트림이 로컬 브랜치로 설정되어 있어도 `git push`는 여전히 원격 저장소(`origin`)로 푸시를 시도합니다. (정확히는 `push.default` 설정에 따라 동작이 달라질 수 있습니다.)

#### 언제 사용될 수 있을까요?

이 기능은 매우 드물게 사용되지만, 다음과 같은 특정 워크플로우에서 유용할 수 있습니다.

* **대규모 기능 개발**: 하나의 큰 기능을 여러 개의 작은 브랜치로 나누어 작업할 때, 작은 브랜치들이 중간 통합 브랜치(예: `develop` 또는 `integration` 역할을 하는 로컬 브랜치)를 추적하도록 설정하여 동기화를 유지하고 싶을 때 사용할 수 있습니다.
* **오프라인 작업**: 인터넷 연결 없이 로컬에서만 여러 브랜치를 오가며 작업할 때, 특정 브랜치를 '안정 버전' 또는 '메인 라인'으로 삼고 다른 브랜치들이 이를 기준으로 작업을 진행하도록 할 수 있습니다.

**결론적으로,** 기술적으로는 로컬 브랜치를 다른 로컬 브랜치의 업스트림으로 설정하는 것이 가능하며, 이는 `pull`과 `status` 명령의 기준점을 로컬의 다른 브랜치로 변경하는 효과를 가집니다. 하지만 대부분의 협업 워크플로우에서는 원격 저장소의 브랜치를 업스트림으로 사용하는 것이 일반적입니다.
