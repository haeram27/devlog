# proguard

## 기능

shrink, optimize, obfuscate 처리를 통해서 jar 파일의 경량화와 난독화를 제공

| 단계 | 기본 옵션 | 비활성화 옵션 (`-dont...`) | 핵심 목적 | 내부에서 수행되는 일 |
|:----:|-----------|----------------------------|-----------|-----------------------|
| **1. Shrink** | `-shrink` (기본 활성) | `-dontshrink` | **쓸모없는 코드 제거**<br>JAR·클래스·메서드·필드·리소스 중에서 실제로 한 번도 참조되지 않는 항목을 “트리 쉐이킹(tree‑shaking)” 방식으로 삭제하여 파일 크기를 최소화 | * 사용하지 않는 클래스/멤버 삭제<br>* 상수 전파·사라진 코드 접합<br>* 리소스(이미지, properties 등) trim<br>* **주의:** 리플렉션·JNI·직렬화처럼 “코드 밖”에서 호출되는 요소는 `-keep` 규칙으로 보호해야 함 |
| **2. Optimize** | `-optimizationpasses` 1 (암묵적으로 활성) | `-dontoptimize` | **바이트코드 최적화**<br>속도와 추가적인 크기 절감을 위해 바이트코드를 다시 작성 | * 메서드 인라이닝·루프 전개·상수 접합<br>* 데드‑코드 제거(더 세밀하게)<br>* 필드/메서드의 접근 제어자 조정<br>* 스택 깊이·로컬 변수 재사용으로 스택프레임 축소<br>* **주의:** 디버깅/프로파일링 툴, 일부 트레이스 코드와 호환성 이슈가 있을 수 있음 |
| **3. Obfuscate** | `-obfuscate` (기본 활성) | `-dontobfuscate` | **난독화 및 심볼 최소화**<br>코드를 이해하기 어렵게 만들고 식별자 길이를 줄여 추가적인 크기 절감 | * 클래스·메서드·필드·패키지를 짧은 이름(예: `a`, `b`)으로 전면 변경<br>* Inner‑class, generic, line‑number, source‑file 등의 메타데이터 정리/삭제<br>* 문자열·리소스 난독화는 별도 플러그인 필요<br>* **주의:** 리플렉션·직렬화·JMX 등 이름 기반 접근은 `-keep` 혹은 `-adaptresource...` 로 예외 처리 |

실행 순서는 Shrink → Optimize → Obfuscate → Preverify(선택) 이며, 앞 단계 결과가 뒷 단계의 입력이 됩니다.
따라서 -dontshrink를 넣어도 -dontobfuscate를 넣지 않는 한 난독화는 그대로 진행됩니다.

## proguard 설정 예

- jdk version = 17
- proguard-maven-plugin version = 2.7.0 (nested proguard version 7.4.1)

```xml
<project>
    <properties>
        <version.plugin.proguard>2.7.0</version.plugin.proguard>
    </properties>
    <build>
        <plugins>
            <plugin>
                <groupId>com.github.wvengen</groupId>
                <artifactId>proguard-maven-plugin</artifactId>
                <version>${version.plugin.proguard}</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>proguard</goal>
                        </goals>
                        <configuration>
                            <skip>false</skip>
                            <libs>
                                <!-- <lib>${java.home}/jmods/</lib> -->
                                <lib>${java.home}/jmods/java.base.jmod</lib>
                                <lib>${java.home}/jmods/java.datatransfer.jmod</lib>
                                <lib>${java.home}/jmods/java.desktop.jmod</lib>
                                <lib>${java.home}/jmods/java.instrument.jmod</lib>
                                <lib>${java.home}/jmods/java.logging.jmod</lib>
                                <lib>${java.home}/jmods/java.management.jmod</lib>
                                <lib>${java.home}/jmods/java.management.rmi.jmod</lib>
                                <lib>${java.home}/jmods/java.naming.jmod</lib>
                                <lib>${java.home}/jmods/java.net.http.jmod</lib>
                                <lib>${java.home}/jmods/java.prefs.jmod</lib>
                                <lib>${java.home}/jmods/java.rmi.jmod</lib>
                                <lib>${java.home}/jmods/java.scripting.jmod</lib>
                                <lib>${java.home}/jmods/java.se.jmod</lib>
                                <lib>${java.home}/jmods/java.security.jgss.jmod</lib>
                                <lib>${java.home}/jmods/java.security.sasl.jmod</lib>
                                <lib>${java.home}/jmods/java.sql.jmod</lib>
                                <lib>${java.home}/jmods/java.sql.rowset.jmod</lib>
                                <lib>${java.home}/jmods/java.transaction.xa.jmod</lib>
                                <lib>${java.home}/jmods/java.xml.crypto.jmod</lib>
                                <lib>${java.home}/jmods/java.xml.jmod</lib>
                            </libs>
                            <includeDependency>true</includeDependency>
                            <includeDependencyInjar>true</includeDependencyInjar>
                            <outjar>${project.build.finalName}-obf.jar</outjar>
                            <options>
                                <!-- ### GENERAL OPTIONS -->
                                <option>-dontshrink</option>
                                <option>-dontoptimize</option>
                                <!-- <option>-dontobfuscate</option> -->
                                <!-- !!! do NOT ignore warnings when it needs to debug -->
                                <option>-ignorewarnings</option>
                                <!-- ### KEEP OPTIONS -->
                                <!-- !!! keep public/protected members only for making library jar -->
                                <!-- <option>-keep public class * { public protected *; }</option> -->
                                <!-- !!! keep main() only for making runnable jar -->
                                <option>-keepclassmembers public class * { public static void main(java.lang.String[]); }</option>
                                <option>-keepattributes *</option>
                                <!-- ### DEBUG OPTIONS -->
                                <option>-verbose</option>
                                <!-- <option>-dontwarn</option> -->
                            </options>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

## 참고

### proguard-maven-plugin의 pom.xml 설정

https://wvengen.github.io/proguard-maven-plugin/
https://wvengen.github.io/proguard-maven-plugin/proguard-mojo.html

### proguard 옵션 설정

https://www.guardsquare.com/manual/configuration/usage

### jar 파일 디컴파일러

https://java-decompiler.github.io/