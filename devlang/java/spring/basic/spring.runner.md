# CommandLineRunner 와 ApplicationRunner

- CommandLineRunner와 ApplicationRunner는 main method에서 `SpringApplication.run()`으로 전달된 arguments들을 실제로 처리하기 위한 인터페이스
- 각 Runner는 다른 방식으로 argument들을 전달 받을 수 있음
- CommandLineRunner가 단순히 공백으로 구분된 문자열(String)으로 argument로 전달 받는다면 ApplicationRunner는 argument를 option/non-option argument로 나누어 전달 받을수 있음
- ApplicationRunner의 주 목적은 --name=value 형식의 option argument를 key와 value로 자동 파싱된 형태로 전달 받는 용도로 사용됨
- ApplicationRunner 사용을 권장. ApplicationRunner는 CommandLineRunner의 상위 호환 방식이며, CommandLineRunner에서 전달 받는 argument 형식도 포함하여 전달 받을 수 있음

## CommandLineRunner

- 각 argument를 어떤한 변형 없이 String 형식으로 전달 받을 수 있음

```java
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;
import lombok.extern.slf4j.Slf4j;

@Slf4j
@Component
public class DefaultCommandLineRunner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        log.info("## CommandLineRunner");
        for (var arg : args) {
            log.info("arg: {}", arg);
        }
    }
}
```

## ApplicationRunner

- option argument(double-dash(`--`) prefix)를 key와 value로 구분된 형태로 전달 받을 수 있음

```java
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;
import lombok.extern.slf4j.Slf4j;

@Slf4j
@Component
public class DefaultApplicationRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        log.info("## ApplicationRunner");

        var optNames = args.getOptionNames();
        for (var name : optNames) {
            log.info("option: name={}, value={}", name, args.getOptionValues(name));
        }

        var nonOptArgs = args.getNonOptionArgs();
        for (var arg : nonOptArgs) {
            log.info("non option args: {}", arg);
        }

        var srcArgs = args.getSourceArgs();
        for (var arg : srcArgs) {
            log.info("source arg: {}", arg);
        }
    }
}
```

- option arguement
  - `--key=value` 형식으로 전달된 argument, double dash(`--`)를 prefix로 하는 argument 만을 `option`이라고 함
- non-option arguement
  - double dash(`--`)를 prefix로 하지 않는 모든 argument, single-dash(`-`)나 dash가 없는 모든 argument를 의미한다.
- source argument
  - option/non-option 구분없이 어플리케이션 실행시 argument 위치에 전달된 모든 입력에서 공백으로 구분된 각 문자열을 의미하며, 다시말해 가공되지 않은 개별 argument 전체를 의미

## java 실행시 argument 지정하기

```bash
gradle bootRun --args='--opt-arg1=value1 --opt-arg2=value2 --opt-arg3 0123 non-opt-arg2 -s'
java -jar executable.jar --opt-arg1=value1 --opt-arg2=value2 --opt-arg3 0123 non-opt-arg2 -s
java -cp non-executable.jar <main-class-name> --opt-arg1=value1 --opt-arg2=value2 --opt-arg3 0123 non-opt-arg2 -s
```

Result:

```text
INFO  main DefaultCommandLineRunner:15 -- ## CommandLineRunner
INFO  main DefaultCommandLineRunner:17 -- arg: --opt-arg1=value1
INFO  main DefaultCommandLineRunner:17 -- arg: --opt-arg2=value2
INFO  main DefaultCommandLineRunner:17 -- arg: --opt-arg3
INFO  main DefaultCommandLineRunner:17 -- arg: 0123
INFO  main DefaultCommandLineRunner:17 -- arg: non-opt-arg2
INFO  main DefaultCommandLineRunner:17 -- arg: -s
INFO  main DefaultApplicationRunner:16 -- ## ApplicationRunner
INFO  main DefaultApplicationRunner:19 -- option: name=opt-arg1, value=[value1]
INFO  main DefaultApplicationRunner:19 -- option: name=opt-arg2, value=[value2]
INFO  main DefaultApplicationRunner:19 -- option: name=opt-arg3, value=[]
INFO  main DefaultApplicationRunner:24 -- non option args: 0123
INFO  main DefaultApplicationRunner:24 -- non option args: non-opt-arg2
INFO  main DefaultApplicationRunner:24 -- non option args: -s
INFO  main DefaultApplicationRunner:29 -- source arg: --opt-arg1=value1
INFO  main DefaultApplicationRunner:29 -- source arg: --opt-arg2=value2
INFO  main DefaultApplicationRunner:29 -- source arg: --opt-arg3
INFO  main DefaultApplicationRunner:29 -- source arg: 0123
INFO  main DefaultApplicationRunner:29 -- source arg: non-opt-arg2
INFO  main DefaultApplicationRunner:29 -- source arg: -s
```

## argument(CommandLineRunner)

- `argument`는 인자 라인에서 공백으로 구분된 word들을 의미

## option argument(ApplicationRunner)

- double dash(`--`)를 prefix로 하는 argument
- ApplicaitonRunner에서는 argument중 `--name=value` 형식을 가진 word를 option argument라고 하며, 특별히 name과 value로 parsing 처리
- argument name으로는 double dash(`--`)와 "equal(`=`) 연산자 또는 공백" 사이의 word가 파싱 됨
- arguemnt의 value에 대입 연산자(`=`)를 이용한 실제 값이 지정되지 않으면 코드 상 value에는 empty string(`""`)이 할당됨

## non-option argument(ApplicationRunner)

double dash prefix가 없는 모든 argument word
일반 단어 뿐 아니라 '-o' 등의 일반적인 short form options 표현도 non option argument로 파싱된다.

## source argument(ApplicationRunner)

option/non-option argment 구분 처리를 하지 않은 공백으로 구분된 argument 리스트
ApplicationRunner의 getSourceArgs()는 CommandLineRunner의 "run(String... args)"로 전달 되는 argument와 동일한 공백구분 argument 문자열 리스트를 얻을 수 있다.