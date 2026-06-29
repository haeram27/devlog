# Git Configuration

## Global git config

~/.gitconfig

```plain
[user]
    name = foo
    email = foo@email.com
[diff]
    tool = bc3
[difftool "bc3"]
    trustExitCode = true
[merge]
    tool = bc3
[mergetool "bc3"]
    trustExitCode = true
[branch]
    autosetuprebase = always
[alias]
    br = branch
    cf = config
    cfl = config -l
    cl = clone --recurse-submodules
    cln = clean -ffdxn
    clnx = clean -ffdx
    clear = !git restore :/ && git clnx && git submodule update
    cm = commit
    cma = commit --amend
    cmm = commit --allow-empty-message -m
    cmmd = !git commit -m \"$(date --rfc-3339=seconds)\"
    co = checkout
    cp = cherry-pick
    df = diff --color=always --word-diff=plain
    dfh = diff HEAD --color=always --word-diff=plain
    dfs = diff --staged --color=always --word-diff=plain
    dt = difftool -y
    lo = "!sh -c 'clear; git log -20 --graph --pretty=format:\" %C(auto)%h%d (%ar : %an) %s\"'"
    lon = log --name-only
    mt = mergetool -y
    sh = stash
    shl = stash list
    shm = stash -m
    sho = stash pop
    sha = stash apply
    shl = stash list
    st = status
    sub = submodule
    subu = !git submodule sync && git submodule update --remote --recursive
    subui = git submodule update --init --remote --recursive
    pl = pull --recurse-submodules=yes --rebase=true --autostash
    purge=!git reset --hard && git clnx
    ps = push
    psf = push -f
    reb = rebase
    reba = rebase --autostash
    rebc = rebase --continue
    rebi = rebase -i
    rebia = rebase -i --autostash
    res = restore
    rst = reset
    rsts = reset --soft
    rsth = reset --hard
[core]
    editor = vim


# using https instead ssh-git protocol
# git config --global url."https://github.com/".insteadOf git@github.com:
# git config --global url."https://".insteadOf git://

# using ssh-git instead https protocol
# git config --global url."git@github.com:".insteadOf https://github.com/
# git config --global url."git://".insteadOf https://

# global proxy settings
# socks5 proxy
#[http]
#   proxy = socks5h://localhost:1080
# domain specific proxy settings
#[http "http://my-remote-domain.com"]
#   proxy = socks5h://localhost:1080

# http proxy
#[http]
#   proxy = http://1.2.3.4:3128
# domain specific proxy settings
#[http "http://my-remote-domain.com"]
#   proxy = http://1.2.3.4:3128

# ssl(https) check disable
#[http]
#      sslverify = false
```

- 옵션 참고
  - https://git-scm.com/docs/git-config

## 동일 git remote repository에 git 프로젝트 별로 다른 ssh auth 사용하기

동일 git remote repository는 같은 url (예-github.com)을 사용하기 때문에 개발용 host의 ssh config 설정시 같은 url에 대해 하나의 ssh만 사용할 경우 프로젝트 별로 구분된 ssh 공개키 적용이 어려움

각 프로젝트별 ssh auth 사용을 위한 설정은 다음 두 가지 방법이 있음

- 방법 1: ssh config의 remote Host 이름에 suffix 추가 하여 프로젝트별 remote Host 구분하기
- 방법 2: project별 gitconfig에 sshCommand 사용하여 ssh 공개키 직접 지정하기

### 방법 1: ssh config의 remote Host 이름에 suffix 추가 하여 프로젝트별 remote Host 구분하기

다음 두 파일에서 ssh url에 suffix를 붙임으로써 local 환경에서 ssh url을 각각 구분하고 구분된 url별로 ssh key file을 지정하는 방법임

- ~/.ssh/config
- {project_root_dir}/.git/config

### Example

#### ~/.ssh/config

```text
Host github.com-projA
  User projA@gmail.com
  HostName github.com
  IdentityFile ~/.ssh/id_rsa_projA
  IdentitiesOnly yes
  StrictHostKeyChecking no

Host github.com-projB
  User projB@gmail.com
  HostName github.com
  IdentityFile ~/.ssh/id_rsa_projB
  IdentitiesOnly yes
  StrictHostKeyChecking no
```

#### {prjectA_root_dir}/.git/config

```text
[user]
        name = projA
        email = projA@gmail.com
[remote "origin"]
        url = git@github.com-projA:projectA/myproject.git
        fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
        remote = origin
        merge = refs/heads/master
```

#### {prjectB_root_dir}/.git/config

```text
[user]
        name = projB
        email = projB@gmail.com
[remote "origin"]
        url = git@github.com-projB:projectB/myproject.git
        fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
        remote = origin
        merge = refs/heads/master
```

### 방법 2, project별 gitconfig에 sshCommand 사용하여 ssh 공개키 직접 지정하기

### Example - aws codecommit git project dir

`<AWS SSH key ID>`:  APKAEIBAERJR2EXAMPLE 형식의 ssh-key를 이용하는 aws codecommit 접속시 login ID

#### {project_root_dir}/.git/config

```text
[core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
        sshCommand = ssh -l <AWS SSH key ID> -i ~/.ssh/id_rsa.h27g -F /dev/null
[user]
        name = foo
        email = foo@example.com
[remote "origin"]
        url = ssh://git-codecommit.<region>.amazonaws.com/v1/repos/<project>
        fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
        remote = origin
        merge = refs/heads/master
        rebase = true

```

## 프로젝트별 git config에 별도 sshCommand 설정하기

### Example - github git project dir

#### {project_root_dir}/.git/config

```text
[core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
        sshCommand = ssh -i ~/.ssh/id_ed25519 -F /dev/null
[user]
        name = bar
        email = bar@example.com
[remote "origin"]
        url = git@github.com:bar/myproject.git
        fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
        remote = origin
        merge = refs/heads/master
```

### Example - aws codecommit git project dir

- `<AWS SSH key ID>`:  APKAEIBAERJR2EXAMPLE 형식의 ssh-key를 이용하는 aws codecommit 접속시 login ID

#### {project_root_dir}/.git/config

```text
[core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
        sshCommand = ssh -l <SSH key ID> -i ~/.ssh/id_rsa -F /dev/null
[user]
        name = foo
        email = foo@example.com
[remote "origin"]
        url = ssh://git-codecommit.<region>.amazonaws.com/v1/repos/<project>
        fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
        remote = origin
        merge = refs/heads/master
        rebase = true
```

or

#### ~/.ssh/config

```text
Host git-codecommit.*.amazonaws.com
  User <AWS SSH key ID>
  IdentityFile ~/.ssh/id_rsa
```
