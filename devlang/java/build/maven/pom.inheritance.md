# pom의 inheritance

---

refs:
- [maven pominheritance](https://maven.apache.org/pom.html#Inheritance)
---

project.xxxManagement tag는 모듈화 maven 프로젝트에서 모듈간 element의 공통 속성을 정의하는 사용된다.

예를 들면 project.dependencyManagement 내에 정의된 dependency element는
dependencyManagement가 정의된 pom.xml과 이를 parent로 하는 자식 pom.xml에 추가된 project.dependencies.dependency element의 "미정의" 속성을 보강한다.
project.dependencies.dependency element내 미정의된 속성은 project.dependencyManagement.dependencies.dependency 정의된 것을 사용하는 것이다.
이를 통해서 모듈화된 child pom.xml에서는 모두 동일한 속성을 갖는 package를 참조할 수 있다
또한 project.dependencies.dependency에 직접 정의된 속성은 dependencyManagement에 명시된 내용을 무시하고 overwrite된다.

## dependency 공통 속성 정의 tag

공통 선언 tag: project.dependencyManagement.dependencies.dependency
참조하는 tag:  project.dependencies.dependency

## plugin 공통 속성 정의 tag

공통 선언 tag: project.build.pluginManagement.plugins.plugin
참조하는 tag:  project.build.plugins.plugin
