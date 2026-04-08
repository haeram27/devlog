# kotlin DSL 설정

## settings.gradle.kts

```gradle
pluginManagement {
    val privateMavenRepositoryUrl = providers.gradleProperty("privateMavenRepositoryUrl").orNull
    if (privateMavenRepositoryUrl.isNullOrBlank()) {
        println("[WARN] gradle property 'privateMavenRepositoryUrl' is missing.")
    }
    repositories {
        if (!privateMavenRepositoryUrl.isNullOrBlank()) {
            maven {
                url = uri(privateMavenRepositoryUrl)
            }
        }
        gradlePluginPortal()
        mavenCentral()
    }
}

dependencyResolutionManagement {
    val privateMavenRepositoryUrl = providers.gradleProperty("privateMavenRepositoryUrl").orNull
    if (privateMavenRepositoryUrl.isNullOrBlank()) {
        println("[WARN] gradle property 'privateMavenRepositoryUrl' is missing.")
    }
    repositories {
        if (!privateMavenRepositoryUrl.isNullOrBlank()) {
            maven {
                url = uri(privateMavenRepositoryUrl)
            }
        }
        mavenCentral()
    }
}

gradle.startParameter.isOffline = false
if (gradle.startParameter.isOffline) {
    println("======================")
    println("gradle in offline mode")
    println("======================")
}

rootProject.name = "springwebex-kotlin"
```

## build.gradle.kts

```gradle
import org.jetbrains.kotlin.gradle.tasks.KotlinCompile

plugins {
    java
    id("org.jetbrains.kotlin.jvm") version "2.1.21"
    id("org.jetbrains.kotlin.plugin.spring") version "2.1.21"
    id("org.springframework.boot") version "3.5.10"
}

val mybatisVersion = "3.0.3"
val springBootVersion = "3.5.10"

group = "com.example"
version = "0.0.1-SNAPSHOT"

java {
    sourceCompatibility = JavaVersion.VERSION_21
}

configurations {
    all {
        exclude(group = "org.springframework.boot", module = "spring-boot-starter-logging")
    }
}

dependencies {
    implementation(platform("org.springframework.boot:spring-boot-dependencies:$springBootVersion"))

    implementation("io.github.oshai:kotlin-logging-jvm:7.0.3")
    runtimeOnly("org.postgresql:postgresql")

    implementation("org.jetbrains.kotlin:kotlin-reflect")
    implementation("com.google.code.gson:gson")
    implementation("org.mybatis.spring.boot:mybatis-spring-boot-starter:$mybatisVersion")

    implementation("org.springframework.boot:spring-boot-starter-log4j2")
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.springframework.boot:spring-boot-starter-aop")
    implementation("org.springframework.boot:spring-boot-starter-validation")
    implementation("org.springframework.boot:spring-boot-starter-quartz")
    implementation("org.springframework.boot:spring-boot-starter-data-mongodb")

    testImplementation("org.jetbrains.kotlin:kotlin-test-junit5")
    testImplementation("com.google.code.gson:gson")
    testImplementation("org.springframework.boot:spring-boot-starter-log4j2")
    testImplementation("org.springframework.boot:spring-boot-starter-aop")
    testImplementation("org.springframework.boot:spring-boot-starter-test")
}

kotlin {
    jvmToolchain(21)
}

tasks.withType<KotlinCompile>().configureEach {
    kotlinOptions {
        freeCompilerArgs += "-Xjsr305=strict"
        jvmTarget = "21"
    }
}

tasks.jar {
    enabled = false
}

tasks.test {
    useJUnitPlatform()
    testLogging {
        showStandardStreams = true
        showCauses = true
        showExceptions = true
        showStackTraces = true
        exceptionFormat = org.gradle.api.tasks.testing.logging.TestExceptionFormat.FULL
        events(
            "passed",
            "skipped",
            "failed",
            "standardOut",
            "standardError"
        )
    }
}

tasks.named<org.springframework.boot.gradle.tasks.run.BootRun>("bootRun") {
    environment("spring.output.ansi.console-available", "true")
}
```