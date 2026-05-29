# [go] 설치 및 빌드

**[참고 페이지]**\
<https://go.dev/doc/install>

## GO 설치 - /usr/local/go

```bash
$ wget <https://go.dev/dl/go1.18.linux-amd64.tar.gz>
or
$ curl -LOk <https://go.dev/dl/go1.18.linux-amd64.tar.gz>

$ sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.18.linux-amd64.tar.gz

$ sudo bash -c 'cat << EOF > /etc/profile.d/go-path.sh
if [[ -d /opt/go/bin ]]; then
    export PATH=$PATH:/opt/go/bin
fi

if [[ -d /usr/local/go/bin ]]; then
    export PATH=$PATH:/usr/local/go/bin
fi

if [[ -d ${HOME}/go/bin ]]; then
    export PATH=$PATH:$HOME/go/bin
fi

if [[ -d ${HOME}/go/gopath-head/bin ]]; then
    export PATH=$PATH:${HOME}/go/gopath-head/bin
fi

EOF'

$ . /etc/profile.d/go-path.sh

$ go version
```

## GO 설치 - /opt/go

```bash
$ wget https://go.dev/dl/go1.19.3.linux-amd64.tar.gz
or
$ curl -LOk https://go.dev/dl/go1.19.3.linux-amd64.tar.gz

$ sudo rm -rf /opt/go; sudo mkdir /opt/go; sudo tar -C /opt -xzf
go1.19.3.linux-amd64.tar.gz

$ sudo bash -c 'cat << EOF > /etc/profile.d/go-path.sh
if [[ -d /opt/go/bin ]]; then
    export PATH=$PATH:/opt/go/bin
fi

if [[ -d /usr/local/go/bin ]]; then
    export PATH=$PATH:/usr/local/go/bin
fi

if [[ -d ${HOME}/go/bin ]]; then
    export PATH=$PATH:$HOME/go/bin
fi

if [[ -d ${HOME}/go/gopath-head/bin ]]; then
    export PATH=$PATH:${HOME}/go/gopath-head/bin
fi
EOF'

$ . /etc/profile.d/go-path.sh
$ go version
```

## vscode용 go package 설치

vscode를 이용하여 golang을 개발할 때,

"Go for Visual Studio Code" 확장을 사용하게 되는데 이 확장은 다음의
내용이 정상 설치되어야 한다.

1, go 실행환경 (시스템 수동 설치, wsl은 linux용 설치 필요)

2, vscode용 go 확장(language server)을 위한 support pkg :

- go-outline
- dlv
- staticcheck
- gopls

여기서 2번의 support 패키지가 정상 설치되어야 vscode에서 golang의
symbol, outline등을 정상 분석하게 되는데

windows에서 vscode 사용시에는 문제가 없으나(자동 설치됨), wsl에서는
설치가 실패하는 문제가 있으므로 수동으로 설치해 주어야 vscode에서 go
확장(language server)이 정상 작동한다.

```go
~~$ go install -v github.com/ramya-rao-a/go-outline@latest~~
$ go install -v github.com/go-delve/delve/cmd/dlv@latest
$ go install -v honnef.co/go/tools/cmd/staticcheck@latest
$ go install -v golang.org/x/tools/gopls@latest
```

## vim-go 설치 방법

### ~/.vimrc의 vim-plug 설정으로 아래 내용 추가

call plug#begin('\~/.vim/plugged')

" vim-go

Plug 'fatih/vim-go', { 'do': ':GoUpdateBinaries' }

call plug#end()

### vim 열어서 plugin 설치 명령 실행

```
:PlugInstall
```

## GO 주요 환경변수 확인하기

```
$ go env
```

### GOPATH - Workspace 디렉토리

Workspace에는 bin, pkg, src 서브디렉토리 존재
default 경로는 `$HOME/go` (the default since Go 1.8)
변경하려면 "go env -w GOPATH=<path/to/go>" 명령어 사용

GOPATH는 Go 설치 경로(GOROOT)와는 반드시 그 경로가 달라야한다.
GOPATH는 여러 개 설정 될 수 있다

windows:
```
GOPATH=PATH1;PATH2;PATH3
```

linux:
```
GOPATH=PATH1:PATH2:PATH3
```

go get 명령으로 다운로드 되는 패키지들은 항상 첫번째 GOPATH(main
workspace)에 설치된다. 보통 첫번째 workspace는 tmp용도로 언제든지 삭제
될 수 있는 용도(young)로 사용하며 두번째 workspace를 영구 보존 pkg들이
존재하는 workspace(eden)로 사용하고 마지막 세번째 workspace부터 개발
프로젝트용 worksapce를 지정한다.

### GOROOT - go runtime 패키지가 설치된 디렉토리
