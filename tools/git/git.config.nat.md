# NAT 환경에서 ssh를 이용한 외부망 git repository에 우회접속

## 프로젝트의 gitconfig 설정

- `[remote]` 항목(remote repository url) 주소가 `git@`으로 시작하면 ssh protocol을 이용해 remote git repository 통신한다

- http protocol로 clone 한 경우
  - `url = https://github.com/myproject.git`
- ssh protocol로 clone 한 경우
  - `url = git@github.com/myproject.git`

### github ssh 주소 예

#### Url

```plain
git@github.com:myid/myproject.git
```

#### File: `.git/config`

```plain
[user]
        name = myname
        email = my@email.com
[core]
        repositoryformatversion = 0
        filemode = false
        bare = false
        logallrefupdates = true
        ignorecase = true
[submodule]
        active = .
[remote "origin"]
        url = git@githubproxy:myid/myproject.git
        fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
        remote = origin
        merge = refs/heads/master
        rebase = true
```

## HOST의 ssh 접속

내부에서 발생한 ssh 접속 요청의 host의 주소가 githubproxy 인 경우 이를 정규 Network HostName인 github.com으로 변경하고 ssh ProxyJump를 이용해 외부망과 연결되는 Gateway ssh host를 사용하여 외부 github에 접속한다

$ cat ~/.ssh/config

```plain
Host githubproxy
    HostName github.com
    IdentityFile ~/.ssh/id_rsa.h27g
    ProxyJump gateway
    StrictHostKeyChecking no

Host gateway
    User foo
    HostName 1.2.3.4
    IdentityFile ~/.ssh/id_rsa.hello
    IdentitiesOnly yes
    StrictHostKeyChecking no
    ExitOnForwardFailure yes
    ServerAliveInterval 60
    ServerAliveCountMax 10
    TCPKeepAlive no
```
