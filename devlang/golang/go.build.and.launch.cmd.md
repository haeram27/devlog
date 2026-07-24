# Go 프로젝트 시작부터 배포까지 사용하는 핵심 명령어

## 1. 프로젝트 초기화 및 의존성 관리

프로젝트를 처음 만들거나 외부 라이브러리를 추가할 때 가장 먼저 사용합니다.

- `go mod init <모듈_이름>`
  - 목적: 현재 디렉터리에 새로운 Go 모듈을 초기화합니다.
    - 역할: 의존성을 추적하고 관리할 수 있는 go.mod 파일을 생성합니다.
    - 주의: 모듈 이름에는 `https://` 같은 URL 스킴을 붙이지 않습니다. GitHub에 올릴 계획이면 `github.com/사용자명/저장소명` 형태의 경로를 사용합니다.
    - 예시: go mod init github.com/username/reponame
- `go mod tidy`
  - 목적: 프로젝트의 외부 라이브러리 의존성을 정리합니다.
    - 역할: 소스 코드에서 쓰이는 라이브러리를 추가하고, 안 쓰는 라이브러리는 go.mod와 go.sum에서 지웁니다.
    - 예시: go mod tidy

### go.mod / go.sum 파일

- `go.mod`
  - 용도: 모듈 이름, 사용하는 Go 버전, 직접/간접 의존 패키지와 그 버전을 정의하는 모듈 정의 파일입니다.
  - `go mod init`, `go mod tidy`, `go get` 명령을 실행하면 자동으로 생성/수정됩니다.
  - 예시:
    ```
    module github.com/username/reponame

    go 1.22

    require (
        github.com/gin-gonic/gin v1.9.1
        github.com/stretchr/testify v1.9.0 // indirect
    )
    ```
    - `module`: 이 프로젝트의 모듈 경로(임포트 경로의 루트)
    - `go`: 이 모듈이 요구하는 최소 Go 언어 버전
    - `require`: 의존하는 패키지 이름과 버전 목록 (`// indirect`는 직접 코드에서 쓰지 않고 다른 의존성이 간접적으로 필요로 하는 패키지)

- `go.sum`
  - 용도: `go.mod`에 명시된 각 의존 패키지 버전의 암호화 해시(체크섬)를 기록해, 다운로드한 패키지가 변조되지 않았는지 검증합니다.
  - 직접 수정하지 않으며, `go mod tidy`/`go get` 실행 시 자동으로 갱신됩니다. 재현 가능한 빌드를 보장하기 위해 반드시 버전 관리(git)에 포함시켜야 합니다.
  - 예시:
    ```
    github.com/gin-gonic/gin v1.9.1 h1:4idEAncQnU5cB7BeOkPtxjfCSye0AAm1R0RVIqJ+Jmg=
    github.com/gin-gonic/gin v1.9.1/go.mod h1:hPrL7YrpYKXt5YId3A/Tnip5kqbEAP+KLuI3SUcPTeU=
    ```
    - 각 줄은 `모듈경로 버전 해시` 형태이며, `/go.mod` 접미사가 붙은 줄은 해당 패키지의 go.mod 파일 자체에 대한 해시입니다.

## 2. 개발 및 테스트 단계

코드를 수정하면서 빠르게 결과를 확인할 때 사용합니다.

- `go run <파일명>`
  - 목적: 코드를 즉시 컴파일하고 실행합니다.
    - 역할: 임시 실행 파일을 만들어 실행한 뒤, 종료되면 해당 파일을 자동으로 삭제합니다.
    - 예시: go run main.go

## 3. 빌드 및 배포 단계

코드 작성이 끝나고 실제 실행 가능한 파일로 내보낼 때 사용합니다.

- `go build -o <출력_파일명>`
  - 목적: 소스 코드를 단일 실행 파일(바이너리)로 컴파일합니다.
    - 역할: 현재 디렉터리에 실행 파일(.exe 등)을 생성하며, 이 파일만 있으면 다른 환경에서도 프로그램이 돌아갑니다.
    - 예시: go build -o myapp main.go
- `go install`
  - 목적: 컴파일된 실행 파일을 시스템 전역에 설치합니다.
    - 역할: 패키지를 컴파일한 뒤 생성된 바이너리를 자동으로 $GOPATH/bin 디렉터리로 이동시켜, 터미널 어디서나 명령어로 실행할 수 있게 만듭니다.
    - 예시: go install

------------------------------
## 한 눈에 보는 추천 작업 순서

   1. 최초 설정: go mod init 프로젝트명
   2. 코드 작성 후 의존성 정리: go mod tidy
   3. 개발 중 실시간 테스트: go run main.go
   4. 최종 배포용 파일 생성: go build -o 결과물명
