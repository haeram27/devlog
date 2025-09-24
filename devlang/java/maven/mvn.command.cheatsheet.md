# mvn cheatsheet

## shell 별 옵션 입력 포맷

powershell의 경우 각 옵션에 대해서 sigle-quoting을 적용해야 한다.

```text
sh$ mvn -e -Dcheckstyle.skip -Dpmd.skip=true -Dcpd.skip=true -Dmaven.test.skip=true clean install
cmd> mvn -e install -Dcheckstyle.skip -Dpmd.skip=true -Dcpd.skip=true -Dmaven.test.skip=true clean install
PS> mvn -e install '-Dcheckstyle.skip' '-Dpmd.skip=true' '-Dcpd.skip=true' '-Dmaven.test.skip=true' clean install
```

## build 및 local repo에 package 설치(install)

```bash
mvn -e -T $(nproc) -Dcheckstyle.skip -Dpmd.skip=true -Dcpd.skip=true -Dmaven.test.skip=true install
```

* `-D,--define <arg>` Define a system property
* `-e,--errors` Produce execution error messages
* `-T,--threads <arg>` Thread count


### Jar 실행 명령

```bash
java -jar -Xms512m -Xmx2048m -Dmax.threads=512 -Dspring.profiles.active=debug  ${RUNNALBE_JAR} | tee /tmp/debug-run-${RUNNALBE_JAR}-$(date -u +%Y%m%d%H%M%S%N).log
```

* `-D<name>=<value>` set a system property
* `-Xms<size>` set initial Java heap size
* `-Xmx<size>` set maximum Java heap size


## 현재 모듈의 최종 pom 보기

* 상속된 상위 설정 머지된 최종 pom 완성본
* mvn 명령 실행 시 참조되는 최종 pom 내용 확인시 사용

```bash
mvn help:effective-pom
```

## 현재 모듈의 dependency 조회

maven-dependency-plugin의 tree 사용

```bash
# 런타임 스코프만
mvn dependency:tree -Dscope=runtime

# 특정 그룹/아티팩트만 필터
mvn -q dependency:tree -Dincludes=com.guardsquare:*

# 더 자세한(충돌/제외 내역 표시)
mvn dependency:tree -Dverbose

# 결과를 파일/다른 포맷으로
mvn dependency:tree > dep.txt
mvn dependency:tree -Doutput=dep.txt
mvn dependency:tree -DoutputType=dot -Doutput=dep.dot   # 그래프viz용

# 사용됐는데 pom에 누락/선언됐는데 미사용 분석
mvn dependency:analyze

# 평면 목록
mvn dependency:list

# 실제 해상된 버전 확인(관리/대체 결과 포함)
mvn help:effective-pom
```

## Maven remote repo로 부터 특정 artifact(jar) 만 local repo로 다운로드 하기

```bash
mvn -U -X dependency:get -Dartifact=com.github.wvengen:proguard-maven-plugin:2.7.0
mvn -U -X dependency:get -Dartifact=com.guardsquare:proguard-base:7.7.0
mvn -U -X dependency:get -Dartifact=com.guardsquare:proguard-core:7.7.0
mvn -U -X dependency:get -Dartifact=com.guardsquare:proguard-core:9.1.11
```
