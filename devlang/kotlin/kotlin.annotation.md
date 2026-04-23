# Kotlin

실무에서 가장 많이 쓰는 기준으로, 코틀린 표준(언어/플랫폼) 애너테이션을 카테고리별로 정리하면 아래와 같습니다.

- [Kotlin Annotations Syntax](https://kotlinlang.org/docs/annotations.html)

## 공통(언어 레벨, kotlin 패키지)

1. Target
어디에 붙일 수 있는지(클래스, 함수, 프로퍼티 등) 지정합니다.

2. Retention
컴파일 후 어느 단계까지 애너테이션을 유지할지 지정합니다.

3. Repeatable
같은 애너테이션을 한 요소에 여러 번 붙일 수 있게 합니다.

4. MustBeDocumented
문서 생성 시 해당 애너테이션을 문서에 포함시킵니다.

5. Deprecated
사용 중단 API를 표시합니다.

6. DeprecatedSinceKotlin
코틀린 버전별 deprecate 경고/에러 레벨을 세분화합니다.

7. ReplaceWith
Deprecated 대체 코드를 IDE에 힌트로 제공합니다.

8. Suppress
컴파일 경고를 억제합니다.

9. SinceKotlin
API가 도입된 최소 코틀린 버전을 표시합니다.

10. RequireKotlin
이 API를 쓰려면 필요한 최소 코틀린 버전을 강제합니다.

11. RequiresOptIn
실험적 API 사용 시 명시적 동의를 요구하는 마커를 만듭니다.

12. OptIn
실험적 API 사용 동의를 선언합니다.

13. SubclassOptInRequired
특정 타입을 상속할 때 opt-in을 요구합니다.

14. WasExperimental
기존 실험 API 마이그레이션용 메타데이터입니다.

15. DslMarker
DSL 스코프 혼동을 막는 마커입니다.

16. BuilderInference
빌더 문맥에서 타입 추론을 확장합니다.

17. OverloadResolutionByLambdaReturnType
람다 반환 타입 기반 오버로드 해석을 허용합니다.

18. RestrictsSuspension
특정 리시버에서 suspend 호출을 제한합니다.

19. UnsafeVariance
제네릭 변성 검사 우회를 명시합니다(주의 필요).

20. PublishedApi
internal 선언을 inline public API에서 참조 가능하게 노출합니다.

21. ParameterName
함수 타입 파라미터 이름 정보를 제공합니다.

22. ExtensionFunctionType
함수 타입을 확장 함수 타입으로 취급하도록 표시합니다.

## JVM 전용(주로 kotlin.jvm 패키지)

1. JvmName
JVM 바이트코드의 이름을 바꿉니다.

2. JvmField
프로퍼티를 getter/setter 없이 필드로 노출합니다.

3. JvmStatic
동반 객체/객체 멤버를 static처럼 노출합니다.

4. JvmOverloads
디폴트 파라미터 기준 오버로드 메서드를 생성합니다.

5. JvmSynthetic
Java에서 숨기고 Kotlin에서만 보이게 합니다.

6. JvmMultifileClass
여러 파일의 top-level 선언을 하나의 클래스처럼 묶습니다.

7. JvmPackageName
파일의 JVM 패키지 이름을 조정합니다.

8. JvmInline
값 클래스를 선언합니다.

9. JvmRecord
Java record 형태로 노출되게 합니다.

10. JvmWildcard
제네릭 위치에 와일드카드 생성을 유도합니다.

11. JvmSuppressWildcards
와일드카드 생성을 억제합니다.

12. Throws
Java 호출자를 위해 checked exception 시그니처를 노출합니다.

13. Volatile
필드를 volatile로 선언합니다.

14. Transient
직렬화 제외 필드로 표시합니다.

15. Synchronized
메서드 수준 동기화를 적용합니다.

16. Strictfp
부동소수점 연산 일관성 제약을 명시합니다.

## JS 전용(주로 kotlin.js 패키지)

1. JsName
JS로 변환될 이름을 지정합니다.

2. JsExport
Kotlin 선언을 JS 외부로 export합니다.

3. JsModule
외부 JS 모듈과 매핑합니다.

4. JsNonModule
모듈이 아닌 환경에서도 사용 가능하게 표시합니다.

5. JsQualifier
외부 JS 네임스페이스 경로를 지정합니다.

6. JsFun
함수 바디를 JS 코드로 직접 지정합니다.

7. ExperimentalJsExport
JsExport 관련 실험 기능 opt-in용입니다(버전에 따라 차이).

## Native 전용(주로 kotlin.native 계열, 버전 영향 큼)

1. ThreadLocal
객체를 스레드 로컬로 취급합니다.

2. SharedImmutable
전역 불변 공유 의도를 표시합니다.

3. CName
네이티브 심볼 이름을 지정합니다.

4. HiddenFromObjC / ObjCName 계열
ObjC/Swift 노출 이름 또는 노출 정책을 제어합니다(버전별 변동).

중요 포인트

1. ETag처럼 “환경 따라 의미가 달라지는 필드”가 있듯, 코틀린 애너테이션도 타깃/버전별 차이가 큽니다.
2. 그래서 팀 기준 문서에는 공통 + JVM만 고정 목록으로 두고, JS/Native는 별도 부록으로 분리하는 것이 유지보수에 좋습니다.
3. 가장 정확한 최신 목록은 Kotlin API 레퍼런스에서 패키지별로 확인하는 방식이 안전합니다.

## JVM 백엔드에서 자주 사용되는 애너테이션 15개

| 애너테이션 | 언제 써야 하나 | 언제 피해야 하나 |
|---|---|---|
| `@Deprecated` | API를 단계적으로 교체할 때 | 대체 경로 없이 남발할 때 |
| `@Suppress` | 경고의 원인을 이해했고 의도적으로 허용할 때 | 경고를 “숨기기만” 할 때 |
| `@Target` | 커스텀 애너테이션 설계 시 적용 위치를 제한할 때 | 기본값에 기대어 모호하게 둘 때 |
| `@Retention` | 런타임 리플렉션 필요 여부를 명확히 할 때 | 런타임 필요 없는데 `RUNTIME`로 두는 경우 |
| `@RequiresOptIn` | 실험 API를 팀 내에서 안전하게 공개할 때 | 안정 API에 과하게 붙여 사용성 떨어뜨릴 때 |
| `@OptIn` | 실험 API를 사용하되 위험을 명시할 때 | 파일/모듈 전체에 광범위하게 걸어버릴 때 |
| `@PublishedApi` | `public inline` 함수가 `internal` 선언을 참조해야 할 때 | 캡슐화 의도 없이 내부 구현을 노출할 때 |
| `@JvmStatic` | Java에서 정적 메서드처럼 호출되길 원할 때 | Kotlin 전용 코드에 습관적으로 붙일 때 |
| `@JvmField` | Java에서 getter 없이 필드 직접 접근이 필요할 때 | 검증/로깅 등 접근 제어가 필요한 프로퍼티 |
| `@JvmOverloads` | Java 호출자에게 디폴트 파라미터 오버로드를 제공할 때 | 파라미터가 많아 메서드 폭증할 때 |
| `@JvmName` | 시그니처 충돌 해결, Java 친화 이름 제공 시 | Kotlin/Java 이름 불일치로 가독성 해칠 때 |
| `@Throws` | Java 호출자가 체크 예외 처리를 해야 할 때 | Kotlin 내부 전용 API에 불필요하게 붙일 때 |
| `@JvmInline` | 작은 값 객체로 타입 안정성 + 성능을 얻고 싶을 때 | JPA 엔티티 필드 등 프레임워크 호환성 불확실할 때 |
| `@Transient` | 직렬화 제외 필드(캐시, 민감정보 등) 표시할 때 | 영속/복구에 꼭 필요한 데이터를 제외할 때 |
| `@Volatile` | 다중 스레드에서 최신 값 가시성이 필요할 때 | 원자적 복합 연산까지 해결된다고 오해할 때 |

운영 기준:

1. Java interop가 요구될 때만 `@Jvm*` 계열을 적극 사용
2. 동시성은 `@Volatile` 하나로 끝내지 말고 원자 타입/락/코루틴 구조와 함께 설계
3. 실험 API는 `@RequiresOptIn` + `@OptIn`으로 명시적으로 경계
4. 커스텀 애너테이션은 항상 `@Target`, `@Retention`부터 먼저 고정

## 애너테이션 사용 예제

Spring + Kotlin 기준으로 Controller, DTO, Service, Config에 15개 애너테이션의 배치 예시

### 1) DTO: 타입 안정성, 직렬화 제어

```kotlin
package com.example.api.dto

import kotlin.jvm.JvmInline
import kotlin.jvm.Transient

@JvmInline
value class UserId(val value: Long)

data class UserResponse(
    val id: UserId,
    val name: String,
    @Transient
    val internalTraceId: String? = null
)
```

언제 이 배치가 좋은가

- @JvmInline: ID, Code 같은 작은 값 객체에 적합
- @Transient: API 응답/직렬화에서 제외할 내부 필드에 적합

피해야 할 경우

- @JvmInline을 JPA 엔티티 필드에 무분별하게 적용
- @Transient를 실제 복구/감사에 필요한 데이터에 적용

---

### 2) Controller: Java 연동 시그니처, Deprecated 마이그레이션

```kotlin
package com.example.api

import com.example.api.dto.UserResponse
import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.PathVariable
import org.springframework.web.bind.annotation.RestController
import kotlin.Deprecated
import kotlin.ReplaceWith
import kotlin.jvm.Throws

@RestController
class UserController(
    private val userService: UserService
) {
    @GetMapping("/v2/users/{id}")
    @Throws(java.io.IOException::class) // Java 호출자 고려가 필요할 때만
    fun getUser(@PathVariable id: Long): UserResponse =
        userService.getUser(id)

    @Deprecated(
        message = "Use /v2/users/{id}",
        replaceWith = ReplaceWith("getUser(id)")
    )
    @GetMapping("/v1/users/{id}")
    fun getUserV1(@PathVariable id: Long): UserResponse =
        userService.getUser(id)
}
```

언제 이 배치가 좋은가

- @Throws: Java 팀/라이브러리와 함께 쓰는 공개 API
- @Deprecated + ReplaceWith: API 버전 전환 시

피해야 할 경우

- Kotlin 내부 전용 컨트롤러에 @Throws 남발
- 대체 경로 없는 @Deprecated

---

### 3) Service: 실험 API 경계, 동시성 가시성

```kotlin
package com.example.service

import org.springframework.stereotype.Service
import java.util.concurrent.atomic.AtomicLong
import kotlin.OptIn
import kotlin.RequiresOptIn
import kotlin.jvm.Volatile

@RequiresOptIn(level = RequiresOptIn.Level.WARNING)
annotation class ExperimentalDiscountApi

@Service
class DiscountService {
    @Volatile
    private var lastRefreshEpochMs: Long = 0L

    private val counter = AtomicLong(0)

    @ExperimentalDiscountApi
    fun calculateDiscount(amount: Long): Long {
        counter.incrementAndGet()
        return amount / 10
    }

    fun refresh() {
        // 가시성 보장 목적
        lastRefreshEpochMs = System.currentTimeMillis()
    }
}

@Service
class CheckoutService(
    private val discountService: DiscountService
) {
    @OptIn(ExperimentalDiscountApi::class)
    fun checkout(amount: Long): Long {
        return amount - discountService.calculateDiscount(amount)
    }
}
```

언제 이 배치가 좋은가

- @RequiresOptIn/@OptIn: 실험 기능 사용 범위를 명시적으로 통제
- @Volatile: 단순 상태 플래그/타임스탬프 가시성

피해야 할 경우

- @OptIn을 패키지 전체에 광범위 적용
- @Volatile로 복합 연산 안전성까지 해결됐다고 가정

---

### 4) Config: Java 친화 팩토리, 오버로드 노출

```kotlin
package com.example.config

import org.springframework.context.annotation.Bean
import org.springframework.context.annotation.Configuration
import kotlin.jvm.JvmField
import kotlin.jvm.JvmName
import kotlin.jvm.JvmOverloads
import kotlin.jvm.JvmStatic

@Configuration
class ClientConfig {

    companion object {
        @JvmField
        val DEFAULT_TIMEOUT_MS: Long = 3000

        @JvmStatic
        fun defaultBaseUrl(): String = "https://api.example.com"
    }

    @Bean
    @JvmName("buildApiClient")
    @JvmOverloads
    fun apiClient(
        baseUrl: String = defaultBaseUrl(),
        timeoutMs: Long = DEFAULT_TIMEOUT_MS
    ): ApiClient = ApiClient(baseUrl, timeoutMs)
}

data class ApiClient(
    val baseUrl: String,
    val timeoutMs: Long
)
```

언제 이 배치가 좋은가

- @JvmStatic/@JvmField: Java 코드에서 Config 값을 자주 참조
- @JvmOverloads: Java에서 디폴트 인자 사용 편의 제공
- @JvmName: 시그니처 충돌/가독성 개선

피해야 할 경우

- Kotlin only 프로젝트인데 interop용 `@Jvm*` 과다 사용
- @JvmOverloads로 오버로드 수가 과도하게 늘어나는 경우

---

### 5) 커스텀 애너테이션 설계: Target, Retention 기본값 명확화

```kotlin
package com.example.support

import kotlin.annotation.AnnotationRetention.RUNTIME
import kotlin.annotation.AnnotationTarget.FUNCTION
import kotlin.annotation.MustBeDocumented
import kotlin.annotation.Retention
import kotlin.annotation.Target

@MustBeDocumented
@Target(FUNCTION)
@Retention(RUNTIME)
annotation class AuditLog(
    val action: String
)
```

언제 이 배치가 좋은가

- AOP/리플렉션 기반 기능(감사로그, 권한검사) 구현 시

피해야 할 경우

- 런타임 사용이 없는데 RUNTIME 유지 정책 사용

---

### 6) 라이브러리/공용 유틸: PublishedApi, Suppress

```kotlin
package com.example.util

import kotlin.PublishedApi
import kotlin.Suppress

@PublishedApi
internal fun normalize(input: String): String = input.trim().lowercase()

@Suppress("NOTHING_TO_INLINE")
inline fun canonical(input: String): String = normalize(input)
```

언제 이 배치가 좋은가

- public inline 함수가 internal 구현을 호출해야 할 때
- Suppress는 경고 원인을 이해한 최소 범위에서만

피해야 할 경우

- 캡슐화 의도 없이 @PublishedApi 남용
- 파일 전체 Suppress로 경고 은폐

---

실무 배치 요약

1. Controller: @Deprecated, @ReplaceWith, 필요 시 @Throws
2. DTO: @JvmInline, @Transient
3. Service: @RequiresOptIn, @OptIn, @Volatile
4. Config: @JvmStatic, @JvmField, @JvmOverloads, @JvmName
5. 공통 인프라: @Target, @Retention, @MustBeDocumented, @PublishedApi, @Suppress

## Kotlin 애너테이션 코딩 컨벤션 (JVM 백엔드)

**Kotlin 애너테이션 코딩 컨벤션 (JVM 백엔드)**

**적용 범위**

1. Kotlin + Spring 기반 서버 코드 전반
2. Controller, DTO, Service, Config, 공용 유틸/라이브러리 코드

**허용 규칙**

1. Deprecated는 반드시 대체 경로와 교체 시점을 함께 명시한다.
2. Suppress는 최소 범위(식, 함수, 프로퍼티)에서만 사용한다.
3. 커스텀 애너테이션은 Target, Retention을 반드시 명시한다.
4. RequiresOptIn은 실험 API 경계가 필요한 경우에만 사용한다.
5. OptIn은 호출 지점에 국소적으로 선언한다.
6. Volatile은 단순 상태 가시성 보장 용도로만 사용한다.
7. JvmInline은 ID, Code 등 작은 값 객체에만 사용한다.
8. JvmStatic, JvmField, JvmOverloads, JvmName은 Java 연동 요구가 있을 때만 사용한다.
9. Throws는 Java 호출자가 checked exception 처리를 반드시 해야 할 때만 사용한다.
10. Transient는 직렬화에서 제외해야 하는 내부/민감 필드에만 사용한다.
11. PublishedApi는 public inline 함수가 internal 선언을 참조할 때만 사용한다.

**금지 규칙**

1. 대체안 없는 Deprecated 사용 금지.
2. 파일 전체 또는 클래스 전체 Suppress 남용 금지.
3. 런타임 리플렉션이 필요 없는데 Retention을 RUNTIME으로 두는 행위 금지.
4. OptIn을 패키지/모듈 단위로 광범위 적용하는 행위 금지.
5. Volatile만으로 복합 연산의 스레드 안전성이 보장된다고 가정하는 행위 금지.
6. Kotlin 전용 코드에 Jvm 계열 애너테이션을 습관적으로 붙이는 행위 금지.
7. JvmOverloads로 과도한 오버로드를 생성하는 설계 금지.
8. JvmInline을 JPA 엔티티 핵심 필드에 사전 검증 없이 적용하는 행위 금지.
9. PublishedApi로 내부 구현을 불필요하게 노출하는 행위 금지.
10. Transient를 복구, 감사, 영속에 필요한 필드에 적용하는 행위 금지.

**리뷰 체크리스트**

1. 이 애너테이션이 진짜 필요한가?
2. 범위를 더 줄일 수 없는가?
3. Java 연동 요구가 실제로 존재하는가?
4. 동시성/직렬화/리플렉션 영향이 검토되었는가?
5. Deprecated, OptIn, Suppress에 근거 주석 또는 문서 링크가 있는가?

**예외 처리 규칙**

1. 금지 규칙 예외는 PR 본문에 사유, 영향, 제거 계획을 반드시 기록한다.
2. 예외는 팀 리뷰어 1명 이상 승인 시에만 허용한다.
3. 임시 예외는 종료 조건과 만료 시점을 반드시 적는다.

원하면 이 규칙을 더 짧게 압축해서 PR 템플릿용 10줄 버전으로도 만들어드릴게요.