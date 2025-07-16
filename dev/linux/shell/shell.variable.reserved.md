# Reserved variables

## refs
- [Bash: Shell Variables](https://www.gnu.org/software/bash/manual/html_node/Shell-Variables.html)
- [ZSH: Parameters-Set-By-The-Shell](https://zsh.sourceforge.io/Doc/Release/Parameters.html#Parameters-Set-By-The-Shell)
- [ZSH: Parameters-Used-By-The-Shell](https://zsh.sourceforge.io/Doc/Release/Parameters.html#Parameters-Used-By-The-Shell)

| 문자 | 설명 |
| --- | --- |
| HOME | 사용자의 홈 디렉토리 |
| PATH | 실행 파일을 찾을 경로 |
| LANG | 프로그램 사용시 기본 지원되는 국가 언어 |
| PWD | 사용자의 현재 작업중인 디렉토리 |
| FUNCNAME | 현재 함수 이름 |
| SECONDS | 스크립트가 실행된 초 단위 시간 |
| SHLVL | 쉘 레벨(중첩된 깊이를 나타냄) |
| SHELL | 현재 로그인해서 사용하는 쉘 |
| PPID | 부모 프로세스의 PID |
| BASH | BASH 실행 파일 경로 |
| BASH_ENV | 스크립트 실행시 BASH 시작 파일을 읽을 위치 변수 |
| BASH_VERSION | 설치된 BASH 버전 |
| BASH_VERSINFO | BASH_VERSINFO[0]~BASH_VERSINFO[5] <br>배열로 상세정보 제공 |
| MAIL | 메일 보관 경로 |
| MAILCHECK | 메일 확인 시간 |
| OSTYPE | 운영체제 종류 |
| TERM | 로긴 터미널 타입 |
| HOSTNAME | 호스트 이름 |
| HOSTTYPE | 시스템 하드웨어 종류 |
| MACHTYPE | 머신 종류(HOSTTYPE과 같은 정보지만 조금더 상세하게 표시됨) |
| LOGNAME | 로그인 이름 |
| UID | Real User ID <br>사용자 구분 목적의 ID <br>uid=$(id -ru) |
| EUID | Effective User ID <br>사용자가 실행한 프로세스의 권한 평가 목적의 ID <br>사용자가 어떤 프로세스를 실행했을 때 실행된 프로세스는 자신을 실행한 유저의 EUID를 기반으로 시스템 리소스(파일 등)에 대한 접근 권한(rwx)을 평가 받는다. <br>EUID 확인 명령어 <br>`echo ${EUID}` <br>`euid=$(id -u)` |
| USER | 사용자의 이름 |
| USERNAME | 사용자 이름 |
| GROUPS | 사용자 그룹(/etc/passwd 값을 출력) |
| HISTFILE | history 파일 경로 |
| HISTFILESIZE | history 파일 크기 |
| HISTSIZE | history 저장되는 개수 |
| HISTCONTROL | 중복되는 명령에 대한 기록 유무 |
| DISPLAY | X 디스플레이 이름 |
| IFS | 입력 필드 구분자(기본값: - 빈칸) |
| VISUAL | VISUAL 편집기 이름 |
| EDITOR | 기본 편집기 이름 |
| COLUMNS | 현재 터미널이나 윈도우 터미널의 컬럼 수 <br> `printf "%*s" ${COLUMNS:-$(tput cols)} '' \| tr ' ' '-'` |
| LINES | 터미널의 라인 수 |
| LS_COLORS | ls 명령의 색상 관련 옵션 |
| PS1 | 기본 프롬프트 변수(기본값: bash\\$) |
| PS2 | 보조 프롬프트 변수(기본값: >), 명령을 "\\"를 사용하여 명령 행을 연장시 사용됨 |
| PS3 | 쉘 스크립트에서 select 사용시 프롬프트 변수(기본값: #?) |
| PS4 | 쉘 스크립트 디버깅 모드의 프롬프트 변수(기본값: +) |
| TMOUT | 0이면 제한이 없으며 time시간 지정시 지정한 시간 이후 로그아웃 |
