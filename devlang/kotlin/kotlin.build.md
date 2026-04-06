# kotlin 빌드

## kt와 kts 확장자에 의한 차이

## kts

- kotlin script 파일
- script로 실행 가능 (확장자로 구분)
- 최상위 영역에서 클래스나 함수의 선언뿐 아니라 함수의 호출 가능
- 최상위 영역을 main 함수의 바디 처럼 사용

#### CLI 빌드 및 실행

```bash
kotlinc -script foo.kts
```

- foo.kts: 실행할 스크립트 파일
- -script: `.kts` 파일을 인자로 받아서 코틀린 스크립트로 실행

## kt

- kotlin 파일
- script로 실행 불가능 (확장자로 구분)
- 최상위 영역에 클래스와 함수 선언만 가능
- main() 함수가 선언 되어 있다면 프로그램 실행 시작 함수가 된다.

#### CLI 빌드 및 실행

일반 코틀린 파일(`.kt`)을 실행하려면 반드시 실행가능 jar 파일로 빌드한 후 실행 해야한다.

```bash
kotlinc file1.kt file2.kt -include-runtime -d out.jar
java -jar out.jar
```

- file1.kt file2.kt: 빌드할 모든 파일명을 나열 (또는 *.kt로 현재 폴더의 모든 파일을 지정 가능)
- -include-runtime: 코틀린 실행 환경(라이브러리)을 JAR 안에 포함하여 실행가능 jar를 생성
- -d output.jar: 생성될 JAR 파일의 이름을 지정

#### 빌드도구 이용

kotlin으로 개발된 프로젝트가 있다면 gradle이나 maven 빌드 도구를 사용한다.



