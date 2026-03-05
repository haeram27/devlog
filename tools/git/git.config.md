
<> Global git config
~/.gitconfig
[diff]
        tool = bc3
[difftool "bc3"]
        trustExitCode = true
[merge]
        tool = bc3
[mergetool "bc3"]
        trustExitCode = true
[user]
        name = bar
        email = bar@example.com
[branch]
        autosetuprebase = always
[alias]
        br = branch
        cf = config
        cfl = config -l
        cl = clone --recurse-submodules
        cln = clean -dfxn
        clnx = clean -dfx
        cm = commit
        cma = commit --amend
        cmm = commit -m
        co = checkout
        df = diff --color=always --word-diff=plain
        dt = difftool -y
        lo = log -50 --graph --pretty=format:'%C(auto)%h%d (%ar : %an) %s'
        lon = log --name-only
        mt = mergetool -y
        sh = stash
        shl = stash list
        st = status
        subi = submodule init
        subu = submodule update -f
        subui = submodule update -f --init --recursive
        pl = pull --recurse-submodules=yes --rebase=true --autostash
        ps = push
        reb = rebase --autostash
      rebi = rebase -i --autostash                                                                                  
[core]                                                                                                                
      editor = vim                                                                                                  
[http]                                                                                                                
      sslverify = false

옵션 참고)
https://git-scm.com/docs/git-config


<> 동일한 vcs(github 등)에서 여러 계정에 대해 ssh auth 사용하기
각 프로젝트별 ssh auth 사용을 위한 설정은 다음 두 가지 방법이 있다.
방법 1, ssh url에 suffix 사용하기
방법 2, local gitconfig에 sshCommand 사용하기


## 방법 1, ssh url에 suffix 사용하기
다음 두 파일에서 ssh url에 suffix를 붙임으로써 local 환경에서 ssh url을 각각 구분하고 구분된 url별로 ssh key file을 지정하는 방법이다.
~/.ssh/config, {project_root_dir}/.git/config 


example) prjectA
~/.ssh/config
Host github.com-projA
  User projA@gmail.com
  HostName github.com
  IdentityFile ~/.ssh/id_rsa_projA
  IdentitiesOnly yes
  StrictHostKeyChecking no

{prjectA_root_dir}/.git/config
[user]
        name = projA
        email = projA@gmail.com
[remote "origin"]
        url = git@github.com-projA:projectA/myproject.git
        fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
        remote = origin
        merge = refs/heads/master


example) prjectB
~/.ssh/config
Host github.com-projB
  User projB@gmail.com
  HostName github.com
  IdentityFile ~/.ssh/id_rsa_projB
  IdentitiesOnly yes
  StrictHostKeyChecking no

{prjectB_root_dir}/.git/config
[user]
        name = projB
        email = projB@gmail.com
[remote "origin"]
        url = git@github.com-projB:projectB/myproject.git
        fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
        remote = origin
        merge = refs/heads/master



## 방법 2, local gitconfig에 sshCommand 사용하기
example) aws codecommit git project dir
<SSH key ID>?  APKAEIBAERJR2EXAMPLE 형식의 ssh-key를 이용한 aws codecommit 접속시 login ID
{project_root_dir}/.git/config
[core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
        sshCommand = ssh -l <SSH key ID> -i ~/.ssh/id_rsa.h27g -F /dev/null
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
or


<> 프로젝트별 git config
example) github git project dir
{project_root_dir}/.git/config
[core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
        sshCommand = ssh -i ~/.ssh/id_ed25519.h27g -F /dev/null
[user]
        name = bar
        email = bar@example.com
[remote "origin"]
        url = git@github.com:bar/myproject.git
        fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
        remote = origin
        merge = refs/heads/master



example) aws codecommit git project dir
<SSH key ID>?  APKAEIBAERJR2EXAMPLE 형식의 ssh-key를 이용한 aws codecommit 접속시 login ID
{project_root_dir}/.git/config
[core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
        sshCommand = ssh -l <SSH key ID> -i ~/.ssh/id_rsa.h27g -F /dev/null
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
or
~/.ssh/config
Host git-codecommit.*.amazonaws.com
  User <SSH key ID> 
  IdentityFile ~/.ssh/id_rsa.h27g
