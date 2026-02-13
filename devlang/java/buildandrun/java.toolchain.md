# java tool chain

## java

- java 프로그램 실행

### to execute a class

- `mainclass` 은 `.class` 확장자를 가지며 `public main()` 메소드를 가져야 한다.
- `args`는 `dash(-)` 보유 여부와 상관 없이 해당 위치의 공백으로 구분된 모든 word가 argument로 전달된다.

```bash
java [options] <mainclass> [args...]
```

### to execute a jar file

- `jarfile` 은 `.jar` 확장자를 가지며 `META-INF/MANIFEST.MF` 파일 내에 `mainclass`가 명시 되어야 하며, 이 `mainclass`는  `public main()` 메소드를 가져야 한다.
- `args`는 `dash(-)` 보유 여부와 상관 없이 해당 위치의 공백으로 구분된 모든 word가 argument로 전달된다.

```bash
java [options] -jar <jarfile> [args...]
```

### to execute the main class in a module

- `mainclass` 은 `.class` 확장자를 가지며 `public main()` 메소드를 가져야 한다.
- `args`는 `dash(-)` 보유 여부와 상관 없이 해당 위치의 공백으로 구분된 모든 word가 argument로 전달된다.

```bash
java [options] -m <module>[/<mainclass>] [args...]
java [options] --module <module>[/<mainclass>] [args...]
```

### to execute a single source-file program

- `sourcefile` 은 `.java` 확장자를 가지며 `public main()` 메소드를 가져야 한다.
- `args`는 dash(-) 보유 여부와 상관 없이 해당 위치의 공백으로 구분된 모든 word가 argument로 전달된다.

```bash
java [options] <sourcefile> [args]
```

### java command sample

```bash
java -Xms4096m -Xmx4096m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/permanent/log/agent -Dmy.system.prop=hello MyTest.java -s --opt-arg=value --non-opt-arg1 non-opt-arg2
java -jar executable.jar --opt-arg1=value1 --opt-arg2=value2 --non-opt-arg1 0123 non-opt-arg2 -s
java -cp non-executable.jar <mainclass> --opt-arg1=value1 --opt-arg2=value2 --non-opt-arg1 0123 non-opt-arg2 -s
gradle bootRun --args='--opt-arg1=value1 --opt-arg2=value2 --non-opt-arg1 0123 non-opt-arg2 -s'
```

## jar

- jar 파일 생성/수정/추출

### `.jar` 파일에 일반 파일 추가

```bash
jar -u -f build/libs/my-0.0.1-SNAPSHOT.jar test/hello.txt
jar uf build/libs/my-0.0.1-SNAPSHOT.jar test/hello.txt 
```

### MANIFEST.MF 파일 수정

- `-m, --manifest` 옵션은 `META-INF/MANIFEST.MF` 파일 자체를 교환하는 것이 아니라 파일의 내용(key=value)을 머지하는 방식으로 수정한다.
- 주의: MANIFEST.MF 파일을 일반 파일과 같이 `update` 명령을 통해 교체 하려고 하면, 명령 실행시 `jar`는 jar 파일 내부의 MANIFEST.MF 파일을 삭제하여 수정이 불가능하게 만든다.

```bash
jar -u -m NEWMANIFEST.MF -f build/libs/my-0.0.1-SNAPSHOT.jar
jar umf NEWMANIFEST.MF build/libs/my-0.0.1-SNAPSHOT.jar
```
