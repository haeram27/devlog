# launcher

VS Code launcher는 프로젝트 디버깅 및 실행 환경을 설정하는 데 사용되는 설정 파일입니다. launch.json이라는 파일로 구성되며, 다양한 언어와 환경에 대한 디버깅 설정을 정의합니다. 주로 Python, C/C++, Node.js 등과 같은 언어의 디버깅 환경을 설정하고, 프로그램 실행 시 필요한 파라미터나 환경 변수를 지정할 수 있습니다.

launch.json 파일 구성 및 사용법

1. .vscode 폴더 생성: 프로젝트 폴더 내에 .vscode 폴더를 생성합니다.
1. launch.json 파일 생성: .vscode 폴더 내에 launch.json 파일을 생성합니다.

1. 디버깅 설정: launch.json 파일에 다음과 같은 정보를 정의합니다.
    * version: 설정 파일의 버전을 지정합니다 (보통 "0.2.0" 또는 "0.2.1"을 사용합니다).
    * configurations: 실제 디버깅 설정을 정의하는 배열입니다.
      * name: 디버깅 설정 이름을 지정합니다 (사용자가 인식하기 쉬운 이름을 사용합니다).
      * type: 사용할 디버깅 엔진을 지정합니다 (예: python, node, cppdbg).
      * request: 디버깅 동작을 지정합니다 (launch는 새로운 프로세스를 시작, attach는 실행 중인 프로세스를 연결).
      * program: 디버깅할 프로그램의 경로를 지정합니다.
      * args: 프로그램에 전달할 파라미터를 지정합니다 (배열 형태로 입력합니다).
      * cwd: 프로그램의 작업 디렉토리를 지정합니다.
      * env: 프로그램 실행 시 필요한 환경 변수를 지정합니다.
      * console: 디버깅 콘솔을 어디에 표시할지 지정합니다 (예: integratedTerminal은 VS Code 내부 터미널, externalTerminal은 외부 터미널).
      * breakPoints: 디버깅 중 중단점을 지정합니다.
      * preLaunchTask: 디버깅 전에 실행할 작업을 지정합니다 (예: build 작업).

1. VS Code에서 디버깅 시작: VS Code에서 "실행 및 디버그" 아이콘 (Ctrl + Shift + D)을 눌러 디버깅을 시작합니다.
1. 설정 선택: launch.json 파일에 정의된 디버깅 설정 목록이 나타나며, 원하는 설정을 선택합니다.
1. 디버깅: VS Code에서 코드를 디버깅할 수 있습니다.

### 예시 (Python)

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Python: 현재 파일",
      "type": "python",
      "request": "launch",
      "program": "${file}",
      "console": "integratedTerminal",
      "stopOnEntry": false,
      "args": ["-v"],
      "env": {
        "MY_ENV_VAR": "value"
      }
    }
  ]
}
```

이 예시에서는 현재 파일 (${file})을 실행하고, 파라미터 -v를 전달하며, MY_ENV_VAR 환경 변수를 설정합니다.

참고: launch.json 파일은 언어별로 특수한 옵션을 지원할 수 있습니다. 각 언어의 디버깅 설정을 확인하여 필요한 옵션을 추가해야 합니다.
