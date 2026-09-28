# Android Jetpack Compose 입문 개발 가이드

## Jetpack Compose란?

**Jetpack Compose는 Android의 최신 네이티브 UI 툴킷**입니다.  
기존 XML 기반 View 시스템처럼 레이아웃 파일과 코드가 분리되는 방식이 아니라, Kotlin 코드 안에서 UI를 선언적으로 작성합니다.

쉽게 말해, Compose에서는 "버튼 텍스트를 직접 바꾼다"보다 **"현재 상태를 바꾼다"**에 집중합니다.  
상태가 변경되면 Compose가 필요한 UI만 다시 그려 주기 때문에, 화면 로직을 더 일관되고 예측 가능하게 관리할 수 있습니다.

참고로 공식 문서는 아래 링크가 가장 정확하고 업데이트가 빠릅니다.

- Android Developers Compose 개요: https://developer.android.com/jetpack/compose
- Compose 학습 경로(Pathway): https://developer.android.com/courses/pathways/compose
- Compose API 레퍼런스: https://developer.android.com/reference/kotlin/androidx/compose/package-summary

---

Jetpack Compose를 처음 사용할 때 핵심은 **UI를 "수정"하는 방식이 아니라, 상태(State)를 "선언"하는 방식**으로 사고를 바꾸는 것입니다.  
이 문서는 초심자가 실무에서 바로 적용할 수 있도록 **필수 선행 지식**, **권장 개발 방식**, **기능 구현 순서**를 한 번에 정리합니다.

---

## 1) 시작 전에 꼭 알아야 할 지식

### 1. Kotlin 기초
- `data class`, `null safety`, `when`, 고차 함수/람다
- `StateFlow`/`Flow`의 기본 개념(값 스트림, 수집)
- 코루틴(`viewModelScope.launch`) 기본 사용

### 2. Android 기본 구조
- `Activity` 생명주기
- `ViewModel` 역할(화면 데이터 보관/비즈니스 로직)
- 리소스 관리(strings/colors/dimens)

### 3. Compose 핵심 개념
- **Composable 함수**: UI를 그리는 함수 (`@Composable`)
- **State & Recomposition**: 상태가 바뀌면 UI가 자동 갱신됨
- **Unidirectional Data Flow(UDF)**:  
  `State 내려보내기` + `Event 올려보내기`
- **Modifier**: 크기/간격/클릭/배경 등 UI 속성 연결

---

## 2) 초심자에게 권장하는 개발 방식

Compose에서는 다음 원칙을 지키면 유지보수가 쉬워집니다.

1. **UI와 상태 관리를 분리**한다.  
   - Composable은 가능한 한 "그리기"에 집중
   - 상태/로직은 ViewModel에 집중

2. **화면 상태를 하나의 UiState로 관리**한다.  
   - 여러 `mutableStateOf`를 흩뿌리기보다 `UiState` 데이터 클래스로 묶기

3. **이벤트를 함수로 위임**한다.  
   - 버튼 클릭, 입력 변경 등은 `onXxx()` 콜백으로 ViewModel에 전달

4. **한 화면을 작은 Composable로 분해**한다.  
   - `Screen`(컨테이너) + `Content`(표현) + `Item`(재사용 단위)

5. **미리보기(Preview)와 상태별 UI를 함께 만든다.**  
   - loading / success / empty / error 상태를 초기부터 분리

---

## 3) Composable은 어디서 호출해야 하나?

핵심 규칙은 **Composable 호출은 Compose 런타임 안에서만** 해야 한다는 점입니다.

- 시작점은 보통 `Activity`의 `setContent { ... }`
- 그 안에서 최상위 `Route` 또는 `Screen` Composable을 호출
- Composable 내부에서 다른 Composable을 계층적으로 호출
- `onCreate()` 일반 코드 블록, `ViewModel`, `Repository` 같은 비-Compose 계층에서는 Composable을 직접 호출하지 않음

예시 구조:

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            AppTheme {
                SampleRoute() // Composable 호출 시작점
            }
        }
    }
}
```

주의할 점:
- `@Composable` 함수는 일반 함수처럼 아무 데서나 호출할 수 없습니다.
- 비즈니스 로직을 Composable 안에서 처리하지 말고, 이벤트를 ViewModel로 전달해 상태만 바꾸는 방식으로 유지합니다.

---

## 4) Composable, Activity, ViewModel의 관계

세 요소의 관계를 한 줄로 요약하면 다음과 같습니다.

**Activity는 진입점, ViewModel은 상태/로직 관리자, Composable은 상태를 그리는 UI**

흐름은 보통 아래와 같습니다.

1. `Activity`가 `setContent`로 Compose 트리를 시작
2. `Composable(Route)`가 `ViewModel`의 `uiState`를 구독
3. `Composable(Screen)`이 `uiState`를 렌더링
4. 사용자 입력(클릭/입력)은 이벤트로 `ViewModel`에 전달
5. `ViewModel`이 상태를 갱신하면, Compose가 자동 재구성

```text
Activity(setContent)
   -> Route Composable
      -> ViewModel.uiState collect
      -> Screen Composable(state)
사용자 이벤트 -> ViewModel(onAction)
ViewModel 상태 변경 -> Recomposition(UI 자동 갱신)
```

역할 분리 원칙:
- **Activity**: 화면 시작, 네비게이션/수명주기 연결
- **ViewModel**: 상태 보관, 유효성 검사, 비동기 처리
- **Composable**: 상태 표현, 이벤트 전달

---

## 5) 기능 구현 표준 순서 (실무 권장)

아래 순서로 구현하면 구조가 흔들리지 않습니다.

### Step 1. 요구사항을 상태로 번역
- "화면에 필요한 데이터"와 "사용자 액션"을 먼저 정의
- 예:  
  - 상태: `email`, `password`, `isLoading`, `errorMessage`
  - 이벤트: `onEmailChanged`, `onPasswordChanged`, `onLoginClicked`

### Step 2. UiState / UiEvent 설계
- `data class LoginUiState(...)`
- 이벤트는 함수형 콜백 또는 `sealed interface`로 설계

### Step 3. ViewModel 구현
- 단일 `StateFlow<UiState>` 노출
- 사용자 액션 처리, 유효성 검사, 로딩/에러 상태 갱신

### Step 4. Screen(컨테이너) 구성
- `collectAsStateWithLifecycle()`로 상태 수집
- 상태와 이벤트를 Content에 전달

### Step 5. Content(표현 UI) 구현
- `Column`, `LazyColumn`, `Scaffold`로 배치
- UI는 전달받은 state를 렌더링만 수행

### Step 6. 상태별 UI 처리
- `when`으로 loading/empty/error/success 분기
- 스낵바/다이얼로그 같은 일회성 이벤트는 별도 처리

### Step 7. Preview + 테스트
- Preview에서 상태별 화면 확인
- ViewModel 단위 테스트(상태 전이 검증) 우선

---

## 6) 화면 구조 템플릿 (권장 형태)

```kotlin
@Immutable
data class SampleUiState(
    val items: List<String> = emptyList(),
    val isLoading: Boolean = false,
    val errorMessage: String? = null
)

class SampleViewModel : ViewModel() {
    private val _uiState = MutableStateFlow(SampleUiState())
    val uiState: StateFlow<SampleUiState> = _uiState.asStateFlow()

    fun onRetryClicked() {
        // 상태 갱신 로직
    }
}

@Composable
fun SampleRoute(viewModel: SampleViewModel = viewModel()) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    SampleScreen(
        state = uiState,
        onRetryClicked = viewModel::onRetryClicked
    )
}

@Composable
fun SampleScreen(
    state: SampleUiState,
    onRetryClicked: () -> Unit
) {
    when {
        state.isLoading -> CircularProgressIndicator()
        state.errorMessage != null -> ErrorView(
            message = state.errorMessage,
            onRetry = onRetryClicked
        )
        else -> LazyColumn {
            items(state.items) { item ->
                Text(text = item)
            }
        }
    }
}
```

---

## 7) 초심자가 자주 하는 실수와 예방

- **실수 1: Composable 안에서 네트워크/DB 호출**
  - 예방: 비동기 작업은 ViewModel/UseCase에서 수행

- **실수 2: 상태를 여러 곳에서 중복 보관**
  - 예방: 화면 상태의 단일 소스(Single Source of Truth) 유지

- **실수 3: 큰 Composable 하나에 모든 코드 작성**
  - 예방: 역할 단위로 분해(Screen / Section / Item)

- **실수 4: 재구성(Recomposition) 비용 무시**
  - 예방: 불필요한 상태 변경 줄이고, 안정적인 파라미터 전달

- **실수 5: Preview 없이 디바이스에서만 확인**
  - 예방: Preview로 빠르게 UI 상태 검증 후 기기 테스트 진행

---

## 8) 학습 우선순위 (2주 입문 루트)

1. **1~3일차**: Kotlin + Compose 기본 문법 (`Text`, `Button`, `Column`, `Modifier`)
2. **4~6일차**: 상태 관리 (`remember`, `mutableStateOf`, `StateFlow`, `collectAsStateWithLifecycle`)
3. **7~10일차**: ViewModel 연동 + 상태 기반 화면 분기
4. **11~14일차**: 리스트(`LazyColumn`), 폼 입력, 에러/로딩 처리, Preview/테스트

---

## 9) 최종 체크리스트

- [ ] 화면 상태를 `UiState`로 모델링했는가?
- [ ] 이벤트 흐름이 `UI -> ViewModel` 단방향인가?
- [ ] Composable이 비즈니스 로직 없이 표현에 집중하는가?
- [ ] loading/empty/error/success 상태를 모두 처리했는가?
- [ ] Preview와 기기 테스트를 모두 수행했는가?

이 체크리스트를 만족하면, Compose 초심자 수준에서 **확장 가능한 구조**로 개발을 시작할 수 있습니다.
