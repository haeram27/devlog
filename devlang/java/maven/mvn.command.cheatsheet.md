# mvn cheatsheet

## shell 별 옵션 입력 포맷

```text
sh$ mvn -e -Dcheckstyle.skip -Dpmd.skip=true -Dcpd.skip=true -Dmaven.test.skip=true clean install
cmd> mvn -e install -Dcheckstyle.skip -Dpmd.skip=true -Dcpd.skip=true -Dmaven.test.skip=true clean install
PS> mvn -e install '-Dcheckstyle.skip' '-Dpmd.skip=true' '-Dcpd.skip=true' '-Dmaven.test.skip=true' clean install
```

## build 및 local repo에 package 설치(install)

```bash
mvn -Dcheckstyle.skip -Dpmd.skip=true -Dcpd.skip=true -Dmaven.test.skip=true install
```

## 현재 모듈의 최종 pom 보기

* 상속된 상위 설정 머지된 최종 pom 완성본
* mvn 명령 실행 시 참조되는 최종 pom 내용 확인시 사용

```bash
mvn help:effective-pom
```

## 현재 모듈의 dependency tree 보기

```bash
mvn dependency:tree > dep.txt
```

## remote repository로 부터 특정 artifact(jar) 만 local repository로 다운로드 하기

```bash
mvn -U -X dependency:get -Dartifact=com.github.wvengen:proguard-maven-plugin:2.7.0
mvn -U -X dependency:get -Dartifact=com.guardsquare:proguard-base:7.7.0
mvn -U -X dependency:get -Dartifact=com.guardsquare:proguard-core:7.7.0
mvn -U -X dependency:get -Dartifact=com.guardsquare:proguard-core:9.1.11
```
