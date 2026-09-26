# ViewModel

- [Android Developers - ViewModel overview](https://developer.android.com/topic/libraries/architecture/viewmodel)
- [Android Developers - Saved State module](https://developer.android.com/topic/libraries/architecture/viewmodel/viewmodel-savedstate)

## 한눈에 보는 요약

**ViewModel = "화면(UI)보다 오래 사는, 화면용 상태 보관소 + 로직 담당자"**

Android의 Activity/Fragment는 화면 회전, 다크모드 전환, 언어 변경 같은 **구성 변경(configuration change)** 이 일어나면 **파괴되고 새로 생성** 된다.
이때 Activity 안에 들고 있던 데이터(목록, 입력값, 로딩 상태 등)는 모두 사라지고, 네트워크 요청도 다시 해야 한다.

ViewModel은 이 문제를 해결하기 위해 만들어졌다.

- **목적**
  - 구성 변경이 일어나도 **UI 상태를 유지** 한다.
  - UI(Activity/Fragment/Composable)에서 **비즈니스 로직과 데이터 처리를 분리** 한다.
  - 화면이 완전히 종료될 때 **진행 중인 작업(코루틴 등)을 자동으로 정리** 한다.
- **동작 방식 (핵심 한 줄)**
  - ViewModel 객체는 Activity가 아니라 **`ViewModelStore`** 라는 저장소에 보관되고, 이 저장소는 구성 변경 중에 **새 Activity로 그대로 넘겨진다.** 그래서 새 Activity가 같은 ViewModel 인스턴스를 다시 받는다.
  - 사용자가 뒤로가기 등으로 화면을 **정말로 종료** 하면 저장소가 비워지면서 `onCleared()`가 호출되고 ViewModel이 소멸한다.
- **한계**
  - ViewModel은 **메모리** 에만 존재한다. 앱이 백그라운드에서 **시스템에 의해 프로세스가 종료(process death)** 되면 함께 사라진다.
  - 이 경우까지 대비하려면 **`SavedStateHandle`** 을 사용한다.

### 비유

> Activity는 **무대 위 배우**, ViewModel은 **무대 뒤 매니저** 다.
> 무대 조명이 바뀔 때(구성 변경)마다 배우는 교체되지만, 매니저는 그대로 남아 대본(상태)을 새 배우에게 건네준다.
> 공연이 완전히 끝나면(화면 종료) 매니저도 퇴근한다(`onCleared()`).
> 다만 극장 자체가 정전되면(process death) 매니저도 사라지므로, 중요한 메모는 금고(`SavedStateHandle`)에 넣어둔다.

### 전체 구조 그림

```text
 ┌────────────── Activity #1 ──────────────┐        ┌────────────── Activity #2 ──────────────┐
 │  UI (View / Compose)                     │  회전   │  UI (View / Compose)                     │
 │     │ 관찰(observe/collect)               │ ─────▶ │     │ 관찰(observe/collect)               │
 │     ▼                                    │ 파괴 후 │     ▼                                    │
 │  ViewModelStoreOwner ── ViewModelStore ──┼── 전달 ─┼▶ ViewModelStoreOwner (같은 Store 재사용) │
 └──────────────────────────────│───────────┘        └──────────────────────────────────────────┘
                                ▼
                     ┌──────────────────────┐
                     │  MyViewModel (1개)    │  ← 같은 인스턴스가 유지됨
                     │  - uiState           │
                     │  - viewModelScope    │
                     │  - SavedStateHandle ─┼──▶ Bundle (process death 대비)
                     └──────────────────────┘
```

### 생명주기 요약

```text
Activity:  onCreate ─ ... ─ onDestroy(회전) ─ onCreate ─ ... ─ onDestroy(finish)
ViewModel: [생성]──────────────────── 계속 살아있음 ──────────────────────[onCleared → 소멸]
```

| 상황 | Activity | ViewModel | SavedStateHandle |
| :--- | :--- | :--- | :--- |
| 화면 회전 / 다크모드 / 언어 변경 | 재생성 | **유지** | 유지 |
| 홈 버튼으로 백그라운드 이동 | 유지(stopped) | 유지 | 유지 |
| 백그라운드에서 시스템이 프로세스 종료 | 재생성 | **소멸 후 새로 생성** | **복원됨** |
| 뒤로가기 / `finish()` | 종료 | **소멸 (`onCleared`)** | 소멸 |
| 사용자가 최근 앱 목록에서 스와이프로 제거 | 종료 | 소멸 | 소멸 |

### 역할 분담: Activity / ViewModel / Repository

**"Activity는 그리고, ViewModel은 결정하고, Repository는 데이터를 책임진다."**

- **Activity (UI 계층)**: 상태를 받아서 **그리고**, 사용자 입력과 **Android 시스템 관련 일** 을 처리한다.
- **ViewModel**: 화면에 **무엇을 보여줄지(상태)** 와 입력에 **어떻게 반응할지(로직)** 를 결정한다.
- **Repository (데이터 계층)**: 데이터가 **어디서 오고 어디에 저장되는지** (네트워크, DB, 캐시)를 관리한다.

```text
┌──────────────────────────── UI 계층 ────────────────────────────┐
│  Activity / Fragment / Composable                                │
│   - UiState를 받아 화면 그리기                                     │
│   - 클릭/입력 → ViewModel 함수 호출                                │
│   - 권한 요청, 화면 이동, Toast/Dialog 등 시스템 관련 처리            │
└───────────────▲───────────────────────────────┬──────────────────┘
                │ 상태 (StateFlow<UiState>)       │ 이벤트 (onClick, onQueryChange ...)
┌───────────────┴───────────────────────────────▼──────────────────┐
│  ViewModel                                                       │
│   - UiState 보관 (구성 변경에도 유지)                                │
│   - 화면 단위 로직: 입력 검증, 로딩/에러 처리, 데이터를 화면용으로 가공     │
│   - viewModelScope로 비동기 작업 실행                                │
└───────────────▲───────────────────────────────┬──────────────────┘
                │ 데이터 (suspend 결과, Flow)      │ 데이터 요청
┌───────────────┴───────────────────────────────▼──────────────────┐
│  Repository (필요 시 UseCase 포함)                                  │
│   - 네트워크(Retrofit), DB(Room), 파일/DataStore, 메모리 캐시 관리      │
│   - 여러 데이터 원천을 조합하고 "진짜 데이터(single source of truth)" 제공 │
└──────────────────────────────────────────────────────────────────┘
```

| 구분 | Activity (UI) | ViewModel | Repository |
| :--- | :--- | :--- | :--- |
| **핵심 역할** | 그리기, 입력 전달, 시스템 연동 | 화면 상태 보관과 화면 로직 | 데이터 획득, 저장, 캐싱 |
| **하는 일** | UiState 관찰 후 렌더링<br>클릭/입력 이벤트 전달<br>권한 요청, `startActivity`<br>Toast/Dialog/Snackbar 표시<br>`repeatOnLifecycle` 로 관찰 시작/중지 | UiState 생성과 갱신<br>입력 검증, 로딩/에러 상태 관리<br>Repository 결과를 화면용으로 가공<br>"이동/메시지 표시가 필요함" 같은 상태/이벤트 발행 | API 호출, DB 쿼리<br>원격/로컬 데이터 동기화<br>캐시 정책<br>데이터 모델 변환 |
| **하지 말 것** | 비즈니스 로직, 직접 네트워크/DB 호출 | Activity/View/Context 참조<br>Retrofit/Room 직접 사용<br>Toast/화면 이동 직접 실행 | UI 상태(로딩 여부 등) 관리<br>Activity/ViewModel 참조 |
| **수명** | 구성 변경 시 재생성 | 화면이 완전히 종료될 때까지 | 보통 앱 수명 (싱글톤, DI로 주입) |
| **아는 대상** | ViewModel | Repository | 데이터 원천 (API, DB) |

**의존 방향은 한쪽으로만** 흐른다: `Activity → ViewModel → Repository`.

- 아래 계층은 위 계층을 모른다. Repository는 ViewModel을 모르고, ViewModel은 Activity를 모른다.
- 위로 전달할 것은 **반환값이나 Flow(관찰 가능한 상태)** 로만 전달한다. 그래서 각 계층을 따로 테스트하고 교체할 수 있다.

#### Activity와 ViewModel을 나누는 기준

> **📌 Activity가 새로 만들어져도 남아 있어야 하는 상태와 "무엇을 할지"에 대한 결정은 ViewModel에, 다시 만들어도 되는 View와 결정을 화면·시스템에 반영하는 실행은 Activity에 둔다.**
>
> 1. **수명 기준** (상태를 어디에 둘지): Activity가 새로 만들어져도 이 값이 남아 있어야 하는가?
> 2. **결정 vs 실행 기준** (코드를 어디에 둘지): 무엇을 할지 판단하는가, 판단 결과를 화면과 시스템에 반영하는가?

##### 첫 번째 기준: 수명

> **"지금 Activity 객체가 버려지고 새로 만들어져도, 사용자는 이 값이 그대로 남아 있기를 기대하는가?"**
>
> - **예** → ViewModel
> - **아니오** (새로 만들어도 됨) 또는 **Context가 필요한 일** → Activity

**기준의 핵심은 "보이는지 여부"가 아니라 "수명"이다.**

사용자가 느끼는 **화면** 과 코드상의 **Activity 객체** 는 수명이 다르다.
화면을 회전하거나 다크모드를 켜면, 화면이 **보이는 상태에서도** 시스템은 Activity 객체를 버리고 새로 만든다.

```text
사용자가 느끼는 화면:  [────────────── 사용자 정보 화면 ──────────────]
Activity 객체:        [Activity#1]  회전 →  [Activity#2]  회전 →  [Activity#3]
ViewModel 객체:       [──────────────── 하나의 ViewModel ─────────────────]
```

- **Activity** = 시스템이 언제든 버리고 다시 만들 수 있는 **일회용 그리기 도구**
- **ViewModel** = 사용자가 느끼는 **"화면 하나"의 수명** 만큼 사는 상태 보관소

**흔한 오해: "ViewModel은 화면이 안 보일 때(백그라운드) 데이터를 들고 있기 위한 것이다"**

이 설명은 틀렸다.

1. **홈 버튼으로 백그라운드에 가도 Activity 객체는 보통 살아 있다.** `onStop` 상태일 뿐 멤버 변수도 그대로다. 안 보이는 동안 데이터를 들고 있는 것은 Activity도 할 수 있다.
2. **ViewModel은 백그라운드 작업용이 아니다.** 뒤로가기로 화면을 나가면 ViewModel도 소멸하고 `viewModelScope` 작업도 취소된다. 프로세스가 종료되어도 사라진다.
   - 화면과 무관하게 계속 돌아야 하는 작업(업로드, 동기화, 음악 재생 등)은 `WorkManager` 나 `Service` 가 담당한다.

**기준 적용 예시**

| 예시 | Activity가 재생성되면? | 둘 곳 |
| :--- | :--- | :--- |
| 서버에서 받아온 목록 | 사라지면 다시 요청해야 함 → 남아 있어야 함 | ViewModel |
| 진행 중인 네트워크 요청 | 회전 때문에 취소되면 안 됨 | ViewModel (`viewModelScope`) |
| 로딩 중 / 에러 상태 | 남아 있어야 함 | ViewModel |
| 입력 중인 폼 값, 검증 결과 | 남아 있어야 함 | ViewModel |
| `TextView`, `RecyclerView` 같은 View 객체 | 새 Activity가 새로 만들면 됨 | Activity |
| Toast/Dialog 띄우기, 권한 요청, 화면 이동 | 시스템(Context)이 필요한 일 | Activity |

##### 두 번째 기준: "결정"인가, "실행"인가?

> **"이 코드는 무엇을 할지 판단하는가, 아니면 판단된 결과를 실제 화면과 시스템에 반영하는가?"**
>
> - **판단(결정)** → ViewModel
> - **반영(실행)** → Activity

첫 번째 기준(수명)이 **상태를 어디에 둘지** 정한다면, 두 번째 기준은 **코드(로직)를 어디에 둘지** 정한다.

| 상황 | 결정 (ViewModel) | 실행 (Activity) |
| :--- | :--- | :--- |
| 네트워크 오류 | "오류 메시지를 보여줘야 한다" → `errorMessage` 상태 설정 | Snackbar 표시 |
| 로그인 성공 | "홈 화면으로 가야 한다" → `navigateToHome` 상태 설정 | `startActivity` / `navController.navigate` |
| 입력 검증 | "이메일 형식이 틀렸다" → `emailError` 상태 설정 | 입력창 아래 에러 문구 표시 |
| 데이터 로딩 | "지금 로딩 중이다" → `isLoading = true` | ProgressBar 표시 |
| 카메라 기능 사용 | "카메라 권한이 필요하다" → `needCameraPermission` 상태 설정 | 권한 요청 다이얼로그 실행 |
| 공유 버튼 클릭 | "이 텍스트를 공유해야 한다" → 공유할 내용 결정 | 공유 Intent 실행 |

**왜 실행을 ViewModel에서 하면 안 되나?**

1. **실행에는 Context/Activity가 필요하다.** Snackbar, `startActivity`, 권한 요청은 모두 Activity(또는 View)가 있어야 한다. 그런데 ViewModel이 Activity를 참조하면, 회전 후 버려진 Activity를 계속 붙잡아 **메모리 누수** 가 생긴다.
2. **실행하는 순간 Activity가 없을 수 있다.** 네트워크 응답이 회전 도중(이전 Activity는 파괴되고 새 Activity는 아직 준비 전)에 도착하면, 실행할 대상이 없다. 결정만 상태로 남겨두면 새 Activity가 준비된 뒤 그 상태를 보고 실행한다.
3. **테스트가 쉬워진다.** 결정만 하는 ViewModel은 Android 화면 없이 "이 입력이면 이 상태가 된다"만 검증하면 된다.

**나쁜 예 vs 좋은 예**

```kotlin
// 나쁜 예: ViewModel이 직접 실행 (Context 참조 → 누수, 테스트 불가)
class LoginViewModel(private val activity: Activity) : ViewModel() {
    fun login(id: String, pw: String) = viewModelScope.launch {
        if (repo.login(id, pw)) activity.startActivity(Intent(activity, HomeActivity::class.java))
        else Toast.makeText(activity, "로그인 실패", Toast.LENGTH_SHORT).show()
    }
}
```

```kotlin
// 좋은 예: ViewModel은 결정만 상태로 표현
data class LoginUiState(
    val isLoading: Boolean = false,
    val errorMessage: String? = null,
    val navigateToHome: Boolean = false,
)

class LoginViewModel(private val repo: AuthRepository) : ViewModel() {
    private val _uiState = MutableStateFlow(LoginUiState())
    val uiState: StateFlow<LoginUiState> = _uiState.asStateFlow()

    fun login(id: String, pw: String) = viewModelScope.launch {
        _uiState.update { it.copy(isLoading = true) }
        val success = repo.login(id, pw)
        _uiState.update {
            if (success) it.copy(isLoading = false, navigateToHome = true)
            else it.copy(isLoading = false, errorMessage = "로그인 실패")
        }
    }

    // Activity가 실행을 마쳤음을 알려주면 상태를 소비(consume)
    fun errorMessageShown() = _uiState.update { it.copy(errorMessage = null) }
    fun navigationHandled() = _uiState.update { it.copy(navigateToHome = false) }
}

// Activity는 상태를 보고 실행만 담당
viewModel.uiState.collect { state ->
    progressBar.isVisible = state.isLoading
    state.errorMessage?.let {
        Snackbar.make(root, it, Snackbar.LENGTH_SHORT).show()
        viewModel.errorMessageShown()
    }
    if (state.navigateToHome) {
        startActivity(Intent(this, HomeActivity::class.java))
        viewModel.navigationHandled()
    }
}
```

- 메시지 표시나 화면 이동처럼 **한 번만 실행해야 하는 일** 은 상태로 표현한 뒤, 실행 후 ViewModel에 알려 **상태를 지운다(consume).** 그렇지 않으면 회전 후 새 Activity가 같은 상태를 보고 Snackbar를 다시 띄운다.
- `Channel` 이나 `SharedFlow` 로 일회성 이벤트를 보내는 방식도 쓰이지만, Activity가 없는 순간에 보낸 이벤트는 유실될 수 있으므로 공식 가이드는 위처럼 **상태로 표현하고 소비하는 방식** 을 권장한다.

**테스트 예시**: 결정만 하므로 Activity 없이 검증할 수 있다.

```kotlin
@Test
fun `로그인 실패 시 에러 메시지 상태가 설정된다`() = runTest {
    val viewModel = LoginViewModel(FakeAuthRepository(loginResult = false))
    viewModel.login("id", "wrong")
    advanceUntilIdle()
    assertEquals("로그인 실패", viewModel.uiState.value.errorMessage)
    assertFalse(viewModel.uiState.value.navigateToHome)
}
```

**두 기준 함께 보기**

| 기준 | 질문 | 정하는 것 |
| :--- | :--- | :--- |
| 첫 번째: 수명 | Activity가 새로 만들어져도 이 값이 남아 있어야 하는가? | **상태** 를 어디에 둘지 |
| 두 번째: 결정 vs 실행 | 무엇을 할지 판단하는가, 판단 결과를 화면과 시스템에 반영하는가? | **코드(로직)** 를 어디에 둘지 |

#### 예: "사용자 정보 불러오기" 흐름

```text
1. [Activity]   화면 진입 / 새로고침 클릭        → viewModel.loadUser(id)
2. [ViewModel]  _uiState = Loading               → repository.getUser(id) 호출 (viewModelScope)
3. [Repository] 캐시 확인 → 없으면 API 호출 → DB 저장 → User 반환
4. [ViewModel]  User를 UiState(Success(user))로 변환하여 갱신 / 실패 시 UiState(Error(msg))
5. [Activity]   uiState 관찰 중 → 화면 다시 그리기, Error면 Snackbar 표시
```

```kotlin
// Repository: 데이터가 어디서 오는지만 책임
class UserRepository(
    private val api: UserApi,
    private val dao: UserDao,
) {
    suspend fun getUser(id: String): User =
        dao.find(id) ?: api.fetchUser(id).also { dao.insert(it) }
}

// ViewModel: 무엇을 보여줄지 결정
class UserViewModel(private val repo: UserRepository) : ViewModel() {
    private val _uiState = MutableStateFlow(UserUiState())
    val uiState: StateFlow<UserUiState> = _uiState.asStateFlow()

    fun loadUser(id: String) = viewModelScope.launch {
        _uiState.update { it.copy(isLoading = true, errorMessage = null) }
        runCatching { repo.getUser(id) }
            .onSuccess { user -> _uiState.update { it.copy(isLoading = false, user = user) } }
            .onFailure { e -> _uiState.update { it.copy(isLoading = false, errorMessage = e.message) } }
    }
}

// Activity: 그리고, 시스템 관련 일 처리
class UserActivity : ComponentActivity() {
    private val viewModel: UserViewModel by viewModels { UserViewModel.Factory }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        viewModel.loadUser(intent.getStringExtra("id")!!)
        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.uiState.collect { state ->
                    render(state)                                   // 그리기
                    state.errorMessage?.let { showSnackbar(it) }    // 시스템 UI 표시는 Activity가
                }
            }
        }
    }
}
```

#### 상태는 어디에 둘까?

| 상태 종류 | 예시 | 두는 곳 |
| :--- | :--- | :--- |
| 영구 데이터 | 사용자 정보, 게시글, 설정 값 | Repository (DB, DataStore, 서버) |
| 화면 상태 (로직과 관련 있음) | 불러온 목록, 로딩 여부, 에러, 입력값 | ViewModel |
| process death 후 복원이 필요한 최소 단서 | 검색어, 선택된 ID | ViewModel의 `SavedStateHandle` |
| 순수 UI 요소 상태 | 스크롤 위치, 애니메이션, 펼침/접힘 | UI (`rememberSaveable`, View 자체 상태 저장) |

---

## 세부 항목

### 1. ViewModel이 필요한 이유

ViewModel이 없던 시절의 문제는 다음과 같았다.

1. **데이터 유실**: 회전할 때마다 Activity가 재생성되어 멤버 변수가 초기화된다.
2. **중복 작업**: 회전할 때마다 네트워크/DB 요청을 다시 한다.
3. **메모리 누수**: 비동기 작업의 콜백이 이미 파괴된 Activity를 참조하여 누수 또는 크래시가 발생한다.
4. **비대한 Activity**: UI 코드와 로직이 한 클래스에 섞여 테스트와 유지보수가 어렵다.

`onSaveInstanceState(Bundle)` 로 일부 해결은 가능하지만, Bundle은 **작고 직렬화 가능한 데이터** 만 담을 수 있어(Binder 트랜잭션 한도 약 1MB) 목록 데이터나 객체 그래프를 넣기에는 부적합하다.

| 구분 | Activity 멤버 변수 | `onSaveInstanceState` | ViewModel |
| :--- | :--- | :--- | :--- |
| 구성 변경 시 유지 | X | O | O |
| process death 시 유지 | X | O | X (`SavedStateHandle` 사용 시 O) |
| 저장 가능한 데이터 | 무엇이든 | 작고 직렬화 가능한 것 | 무엇이든 (메모리) |
| 속도 | 빠름 | 직렬화 비용 있음 | 빠름 |

> 결론: **큰/복잡한 상태는 ViewModel**, **복원에 꼭 필요한 최소 키 값(검색어, 선택된 ID 등)은 SavedStateHandle** 에 둔다.

### 2. 기본 사용법

```kotlin
class CounterViewModel : ViewModel() {
    private val _count = MutableStateFlow(0)
    val count: StateFlow<Int> = _count.asStateFlow()   // UI에는 읽기 전용으로 노출

    fun increment() {
        _count.update { it + 1 }
    }

    override fun onCleared() {
        // 화면이 완전히 종료될 때 한 번 호출됨 (리소스 정리)
    }
}
```

```kotlin
// Activity
class MainActivity : ComponentActivity() {
    private val viewModel: CounterViewModel by viewModels()   // activity-ktx

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.count.collect { /* UI 갱신 */ }
            }
        }
    }
}

// Fragment
private val viewModel: CounterViewModel by viewModels()           // Fragment 범위
private val shared: SharedViewModel by activityViewModels()       // Activity 범위 (Fragment 간 공유)

// Compose
@Composable
fun CounterScreen(viewModel: CounterViewModel = viewModel()) {   // lifecycle-viewmodel-compose
    val count by viewModel.count.collectAsStateWithLifecycle()
    Button(onClick = viewModel::increment) { Text("$count") }
}
```

- `by viewModels()` 는 **지연(lazy) 초기화** 로, 처음 접근할 때 Store에서 꺼내거나 없으면 생성한다.
- **절대 `CounterViewModel()` 처럼 직접 생성하지 않는다.** 직접 생성하면 Store에 등록되지 않아 일반 객체와 다를 바 없다.

### 3. 내부 동작 방식

#### 3.1 주요 구성 요소

| 구성 요소 | 역할 |
| :--- | :--- |
| `ViewModel` | 상태와 로직을 담는 클래스. `onCleared()` 로 정리 시점을 받는다. |
| `ViewModelStore` | `Map<String, ViewModel>` 형태의 저장소. `clear()` 시 모든 ViewModel의 `onCleared()` 호출. |
| `ViewModelStoreOwner` | `ViewModelStore` 를 소유하는 주체. `ComponentActivity`, `Fragment`, `NavBackStackEntry` 가 구현한다. |
| `ViewModelProvider` | Owner의 Store에서 ViewModel을 찾거나, 없으면 Factory로 생성해 Store에 넣는다. |
| `ViewModelProvider.Factory` | ViewModel 인스턴스를 **어떻게** 만들지 정의한다 (생성자 인자 주입). |
| `CreationExtras` | Factory에 전달되는 부가 정보 (Application, SavedState, 인텐트 인자 등). |

#### 3.2 ViewModel을 얻는 과정

```text
viewModels() 접근
   └▶ ViewModelProvider(owner.viewModelStore, factory, extras).get(MyViewModel::class)
         ├─ key = "androidx.lifecycle.ViewModelProvider.DefaultKey:" + 클래스명
         ├─ store[key] 가 있고 타입이 맞으면 → 그대로 반환   (회전 후 재사용되는 경로)
         └─ 없으면 → factory.create(MyViewModel::class, extras) → store[key] = vm → 반환
```

- 같은 Owner + 같은 클래스(key)면 **항상 같은 인스턴스** 를 돌려준다.
- 같은 클래스를 여러 개 쓰고 싶다면 `get(key, MyViewModel::class)` 로 key를 다르게 준다.

#### 3.3 구성 변경에서 살아남는 원리

핵심은 **"ViewModel을 살리는 것"이 아니라 "ViewModelStore를 새 Activity로 넘기는 것"** 이다.

1. 회전 발생 → 시스템이 기존 Activity를 파괴하기 직전에 `onRetainNonConfigurationInstance()` 를 호출한다.
2. `ComponentActivity` 는 여기서 자신의 `ViewModelStore` 를 `NonConfigurationInstances` 객체에 담아 반환한다. (이 객체는 시스템(ActivityThread)이 메모리에 잠시 들고 있음)
3. 기존 Activity의 `ON_DESTROY` 시점에 `isChangingConfigurations()` 를 확인한다.
   - `true` (회전 등) → Store를 **비우지 않는다.**
   - `false` (진짜 종료) → `viewModelStore.clear()` → 각 ViewModel의 `onCleared()` 호출.
4. 새 Activity 생성 → `getLastNonConfigurationInstance()` 로 이전 Store를 꺼내 **그대로 재사용** 한다.
5. 새 Activity에서 `by viewModels()` 에 접근하면 Store에 이미 있는 인스턴스가 반환된다.

```text
[Activity #1] onRetainNonConfigurationInstance() ─▶ NonConfigurationInstances{ viewModelStore }
[Activity #1] onDestroy  (isChangingConfigurations = true → clear 하지 않음)
[Activity #2] onCreate   getLastNonConfigurationInstance() ─▶ 같은 viewModelStore 획득
```

- **Fragment** 는 `FragmentManager` 가 내부적으로 `FragmentManagerViewModel`(이것도 ViewModel)을 부모 Activity의 Store에 두고, 그 안에 각 Fragment의 `ViewModelStore` 를 보관한다. 결국 Activity의 Store 유지 메커니즘에 편승한다.
- **Navigation** 에서는 각 `NavBackStackEntry` 가 Owner가 되어, 해당 화면이 백스택에서 빠질 때 ViewModel이 정리된다.
- **Compose** 의 `viewModel()` 은 `LocalViewModelStoreOwner` (보통 Activity, Fragment 또는 NavBackStackEntry)를 사용한다. Composable 자체가 Owner가 되는 것이 아니다.

#### 3.4 소멸과 정리 (`onCleared`, `viewModelScope`)

- Store가 `clear()` 될 때 ViewModel의 `onCleared()` 가 한 번 호출된다.
- `viewModelScope` 는 ViewModel에 붙어 있는 `CoroutineScope` 이다.
  - 구성: `SupervisorJob() + Dispatchers.Main.immediate`
  - ViewModel이 정리될 때 **자동으로 cancel** 된다. (내부적으로 `AutoCloseable` 로 등록되어 `onCleared()` 직전에 닫힘)
- 따라서 ViewModel 안의 비동기 작업은 `viewModelScope.launch { ... }` 로 시작하면 수동 취소가 필요 없다.

```kotlin
class UserViewModel(private val repo: UserRepository) : ViewModel() {
    fun load() = viewModelScope.launch {
        val user = repo.fetchUser()   // 회전해도 계속 진행, 화면 종료 시 자동 취소
        _uiState.value = UiState.Success(user)
    }
}
```

### 4. 생성자 인자 주입 (Factory)

ViewModel은 Provider가 생성하므로, 생성자에 인자가 필요하면 **Factory** 로 생성 방법을 알려줘야 한다.

```kotlin
class UserViewModel(
    private val repo: UserRepository,
    private val savedStateHandle: SavedStateHandle,
) : ViewModel() {
    companion object {
        val Factory: ViewModelProvider.Factory = viewModelFactory {
            initializer {
                val app = this[ViewModelProvider.AndroidViewModelFactory.APPLICATION_KEY] as MyApp
                UserViewModel(
                    repo = app.container.userRepository,
                    savedStateHandle = createSavedStateHandle(),
                )
            }
        }
    }
}

private val viewModel: UserViewModel by viewModels { UserViewModel.Factory }
```

실무에서는 보통 **Hilt** 를 사용해 Factory 작성을 생략한다.

```kotlin
@HiltViewModel
class UserViewModel @Inject constructor(
    private val repo: UserRepository,
    private val savedStateHandle: SavedStateHandle,
) : ViewModel()

// Activity/Fragment: by viewModels()   /   Compose: hiltViewModel()
```

### 5. SavedStateHandle: process death 대비

- ViewModel은 메모리에만 있으므로 **프로세스가 죽으면 사라진다.**
- `SavedStateHandle` 은 key-value 저장소로, 내부적으로 `SavedStateRegistry` 를 통해 **`onSaveInstanceState` 의 Bundle에 저장** 된다. 그래서 프로세스가 재시작되어도 값이 복원된다.
- Navigation 사용 시 **화면 인자(arguments)** 도 자동으로 `SavedStateHandle` 에 들어온다.

```kotlin
class SearchViewModel(private val savedStateHandle: SavedStateHandle) : ViewModel() {
    // 키 값을 StateFlow로 노출, 값 변경 시 자동 저장
    val query: StateFlow<String> = savedStateHandle.getStateFlow("query", "")

    fun onQueryChange(q: String) {
        savedStateHandle["query"] = q
    }
}
```

**무엇을 넣을까?**

- 넣을 것: 검색어, 선택된 아이템 ID, 스크롤 위치, 입력 중인 짧은 텍스트 등 **다시 로드하기 위한 최소한의 단서**
- 넣지 말 것: 서버에서 받은 목록 전체, 이미지, 큰 객체 → ID만 저장하고 복원 후 Repository에서 다시 로드

> 테스트 방법: 앱을 백그라운드로 보낸 뒤 `adb shell am kill <패키지명>` 을 실행하고 다시 앱으로 돌아오면 process death 복원을 확인할 수 있다. (개발자 옵션의 "활동 유지 안함(Don't keep activities)" 은 Activity 파괴만 흉내내며 프로세스는 유지된다.)

### 6. 범위(Scope)와 공유

ViewModel의 수명은 **어떤 Owner의 Store에 들어가느냐** 로 결정된다.

| 획득 방법 | Owner | 수명 | 용도 |
| :--- | :--- | :--- | :--- |
| Activity의 `by viewModels()` | Activity | Activity 종료까지 | 단일 화면 Activity |
| Fragment의 `by viewModels()` | Fragment | Fragment 제거까지 | Fragment 전용 상태 |
| Fragment의 `by activityViewModels()` | 부모 Activity | Activity 종료까지 | **Fragment 간 데이터 공유** |
| `by navGraphViewModels(R.id.graph)` / `hiltViewModel(parentEntry)` | NavGraph의 BackStackEntry | 해당 그래프가 백스택에서 빠질 때까지 | 여러 단계 플로우(회원가입, 결제 등) 공유 |
| Compose `viewModel()` | `LocalViewModelStoreOwner` (Nav 사용 시 해당 화면 entry) | 해당 화면이 백스택에서 빠질 때까지 | 화면 단위 상태 |

### 7. UI 상태 노출 패턴 (단방향 데이터 흐름, UDF)

```text
         이벤트 (onClick, onQueryChange ...)
   UI  ──────────────────────────────────▶  ViewModel ──▶ Repository / UseCase
   ▲                                          │
   └──────────── 상태 (StateFlow<UiState>) ◀───┘
```

- **상태는 아래로(ViewModel → UI), 이벤트는 위로(UI → ViewModel)** 흐른다.
- 내부는 `MutableStateFlow`, 외부에는 읽기 전용 `StateFlow` 로 노출해 UI가 상태를 직접 변경하지 못하게 한다.
- 화면 상태는 하나의 `data class` 로 묶으면 관리가 쉽다.

```kotlin
data class UserUiState(
    val isLoading: Boolean = false,
    val user: User? = null,
    val errorMessage: String? = null,
)
```

- `LiveData` 도 여전히 사용 가능하지만, 신규 코드(특히 Compose)에서는 `StateFlow` + `collectAsStateWithLifecycle()` 이 권장된다.

### 8. 주의사항 (안티패턴)

| 하지 말 것 | 이유 | 대안 |
| :--- | :--- | :--- |
| ViewModel에 `Activity`, `Fragment`, `View`, Activity `Context` 참조 보관 | ViewModel이 Activity보다 오래 살아 **메모리 누수** 발생 | 필요한 경우 `Application` context만 사용 (`AndroidViewModel`), 가급적 Repository로 분리 |
| ViewModel을 직접 `MyViewModel()` 로 생성 | Store에 등록되지 않아 구성 변경 시 유지되지 않음 | `by viewModels()`, `viewModel()`, `hiltViewModel()` 사용 |
| `GlobalScope` 로 작업 실행 | 화면이 종료되어도 작업이 계속 돌아 누수/낭비 | `viewModelScope` 사용 |
| ViewModel을 다른 ViewModel이나 Repository에 전달 | 수명 불일치, 결합도 증가 | 공유 데이터는 Repository 계층에 두기 |
| 큰 데이터를 `SavedStateHandle` 에 저장 | `TransactionTooLargeException` 위험 | ID 등 최소 정보만 저장 |
| 일회성 이벤트(토스트, 화면 이동)를 상태로 계속 보관 | 회전 후 다시 실행되는 문제 | 이벤트를 상태로 모델링하고 처리 후 소비(consume) 처리 |
| ViewModel을 영구 저장소처럼 사용 | 화면 종료/프로세스 종료 시 사라짐 | 영구 데이터는 Room, DataStore 등 사용 |

### 9. 정리

1. ViewModel은 **UI 상태와 로직을 UI 컨트롤러에서 분리** 하고, **구성 변경에도 살아남는** 객체다.
2. 살아남는 원리는 **`ViewModelStore` 가 `NonConfigurationInstances` 를 통해 새 Activity로 전달** 되기 때문이다.
3. 화면이 **진짜로 종료** 되면 Store가 `clear()` 되며 `onCleared()` 와 `viewModelScope` 취소가 일어난다.
4. **process death** 까지 대비하려면 `SavedStateHandle` 에 최소한의 상태를 저장한다.
5. 수명(범위)은 **어떤 `ViewModelStoreOwner` 로 얻었는지** 에 따라 결정된다.
6. ViewModel에는 **Activity/View/Context 참조를 두지 않는다.**
7. 역할 분담: **Activity는 그리고 시스템 관련 일을 처리하며, ViewModel은 상태와 화면 로직을 결정하고, Repository는 데이터를 책임진다.** 의존 방향은 `Activity → ViewModel → Repository` 한쪽으로만 흐른다.
8. Activity와 ViewModel을 나누는 기준은 "보이는지 여부"가 아니라 **수명** 이다. **"Activity 객체가 새로 만들어져도 사용자가 이 값이 남아 있기를 기대하는가?"** 에 "예"라면 ViewModel에 둔다.
9. 코드는 **"결정"은 ViewModel, "실행"은 Activity** 에 둔다. ViewModel은 판단 결과를 상태로 표현하고, Activity는 그 상태를 보고 Snackbar 표시나 화면 이동을 실행한 뒤 ViewModel에 알려 상태를 소비한다.
