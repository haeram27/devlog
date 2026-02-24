# kotlin 개발 환경

- [jdk](https://jdk.java.net/archive/)
- [kotlinc](https://github.com/JetBrains/kotlin/releases)

## kotlin path

- bashrc or zshrc

```bash
## JAVA PATH
temp="/opt/jdk/jdk-21.0.2"
if [[ -d $temp ]]; then
    export JDK_HOME=$temp
    export JAVA_HOME=${JDK_HOME}
    export CLASSPATH=.:${JAVA_HOME}/lib
    export PATH=${PATH}:${JAVA_HOME}/bin
fi

## KOTLIN PATH
temp="/opt/kotlin/2.3.10/kotlinc"
if [[ -d $temp ]]; then
    export KOTLIN_HOME=$temp
    export CLASSPATH=${CLASSPATH}:${KOTLIN_HOME}/lib
    export PATH=${PATH}:${KOTLIN_HOME}/bin
fi
```
