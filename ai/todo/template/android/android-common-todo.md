# Android 앱 공통 todo

앱 고유 기능과 무관하게 **모든 Android 앱이 출시까지 한 번씩은 거쳐야 하는 작업** 목록.
새 프로젝트를 시작할 때 이 파일을 복사해 `docs/todo.md` 옆에 두고, 해당 없는 항목은
지우지 말고 `- 해당 없음 — 이유` 로 남긴다 (빠뜨린 것과 구분하기 위해).

- 기준: Kotlin + Jetpack Compose + Material 3, 단일 Activity, Google Play 배포
- `(결정)` 표시는 앱마다 선택이 필요한 항목이다. 선택과 근거를 옆에 적는다
- Play 정책·API 요구사항은 매년 바뀐다. 날짜·수치가 적힌 항목은 착수 시점에 공식 문서로 다시 확인한다

---

## 1. 프로젝트 기본 설정

- [ ] **`applicationId` / `namespace` 확정** — 템플릿 기본값(`com.example.*`)은 Play 에 올릴 수 없다.
      출시 후에는 바꿀 수 없으므로(다른 앱이 된다) **첫 출시 전에** 정한다.
      패키지 경로·Room DB 파일·SharedPreferences 이름이 모두 영향을 받으니 코드가 커지기 전에 할수록 싸다
- [ ] 앱 이름(`app_name`) 확정 + 지원 언어별 번역
- [ ] `versionCode` / `versionName` 규칙 `(결정)` — 수동 / 커밋 수 / 날짜 기반. `versionCode` 는 올리기만 가능
- [ ] `minSdk` `(결정)` — 지원 기기 비율과 쓰고 싶은 API 사이의 절충. 근거를 기록
- [ ] `targetSdk` — Play 요구 수준 이상 (매년 상향. 신규 앱·업데이트 모두 적용)
- [ ] 빌드 환경 고정 — Gradle wrapper 버전, JDK toolchain, version catalog(`libs.versions.toml`)
- [ ] 코드 스타일 — `.editorconfig`, 린터(ktlint/detekt) 도입 여부 `(결정)`
- [ ] 빌드 타입 — `debug` 에 `applicationIdSuffix ".debug"` 를 붙여 릴리즈판과 한 기기에 같이 설치할지 `(결정)`
- [ ] `CLAUDE.md` / README — 빌드 명령, 구조, 코드 규칙, 주의사항

## 2. 브랜딩 — 아이콘과 시작 화면

- [ ] **런처 아이콘** — adaptive icon (foreground / background 분리, 108dp 캔버스 안 72dp 안전 영역)
  - [ ] **monochrome 레이어** — Android 13+ 테마 아이콘. 없으면 테마 아이콘을 켠 사용자에게 원본이 그대로 나온다
  - [ ] 원형 마스크·다양한 런처 모양에서 잘리지 않는지 확인
  - [ ] Play 스토어용 512×512 아이콘 (런처 아이콘과 별도 제출)
- [ ] **시작 화면(로고 화면)** — Android 12+ 는 시스템이 **항상** 스플래시를 띄운다
  - [ ] `androidx.core:core-splashscreen` 의 SplashScreen API 로 통일 (12 미만도 같은 모양)
  - [ ] **별도 스플래시 Activity 를 만들지 않는다** — 12+ 에서 스플래시가 두 번 뜬다
  - [ ] 아이콘 · 배경색 지정, 다크 모드 배경색
  - [ ] 초기 데이터 로딩이 필요하면 `setKeepOnScreenCondition` 으로 짧게 유지 (길게 붙잡지 않는다)
- [ ] 브랜드 색 → Material 3 색 체계. 동적 색(Android 12+ 배경화면 색) 사용 여부 `(결정)`

## 3. 권한

- [ ] **최소 권한 원칙** — 필요한 권한 목록과 이유를 표로 기록. 대체 API 가 있으면 권한을 쓰지 않는다
  - 파일: SAF(`OpenDocument` / `OpenDocumentTree`) — 권한 불필요
  - 사진·영상: Photo Picker — 권한 불필요
  - 연락처 하나 고르기: `PickContact` — 권한 불필요
- [ ] **병합 매니페스트 점검** — 라이브러리가 몰래 넣은 권한 확인 (`app/build/intermediates/merged_manifest`),
      필요 없으면 `tools:node="remove"`
- [ ] **런타임 권한 요청 흐름** (위험 권한을 쓰는 경우)
  - [ ] 기능을 쓰는 시점에 요청 (앱 시작 직후 한꺼번에 요청하지 않는다)
  - [ ] 거부 시 이유 설명(rationale) → 재요청
  - [ ] 영구 거부("다시 묻지 않음") 시 설정 화면으로 안내
  - [ ] 권한 없이도 앱이 죽지 않고 해당 기능만 비활성
  - [ ] 앱 실행 중 설정에서 권한을 끄는 경우 (프로세스가 재시작된다)
- [ ] **알림 권한** `POST_NOTIFICATIONS` — Android 13+ 런타임 권한. 알림을 쓰면 필수
- [ ] **SAF 영구 URI 권한**(파일 앱이라면) — `takePersistableUriPermission` 실패 처리, 복원 전 권한 확인,
      `releasePersistableUriPermission` 으로 정리 (상한: Android 10 이하 128개, 11+ 512개)
- [ ] 정확한 알람·전체 화면 인텐트·백그라운드 위치 등 **Play 심사 대상 권한**을 쓰는지 확인 — 쓰면 선언서 제출

## 4. 빌드·서명·배포 산출물

- [ ] **업로드 키 생성** (`keytool` / Android Studio) — 유효기간 25년 이상
- [ ] **키 보관** — keystore 파일과 비밀번호를 저장소 밖에 두고 **백업 2곳 이상**.
      잃어버리면 Play App Signing 없이는 업데이트를 영원히 못 올린다
- [ ] 서명 설정은 저장소에 비밀번호를 넣지 않는다 — `keystore.properties`(gitignore) 또는 환경 변수 / CI 시크릿
- [ ] **Play App Signing** 등록 — 앱 서명 키는 Google 이 보관, 개발자는 업로드 키만. 업로드 키 분실 시 재설정 가능
- [ ] 릴리즈는 **AAB**(`bundleRelease`) 로 빌드. APK 는 직접 배포용일 때만
- [ ] **R8 난독화·축소** — `isMinifyEnabled = true`, `isShrinkResources = true`
  - [ ] keep 규칙: 리플렉션·직렬화·Room·JNI 를 쓰는 라이브러리 확인
  - [ ] **릴리즈 빌드로 실제 기능을 한 바퀴 돌려본다** — 디버그에서만 테스트하면 R8 문제를 놓친다
  - [ ] `mapping.txt` 보관 (크래시 스택 해독용, Play Console 에 업로드)
- [ ] 릴리즈 빌드에서 디버그 전용 코드 제거 — `Log.d`, 디버그 메뉴, 테스트용 URL
- [ ] CI — PR 마다 `assembleDebug` + `test` + `lint`, 태그 시 서명된 AAB 생성 `(결정)`

## 5. 데이터와 저장

- [ ] 저장소 선택 `(결정)` — 설정값: DataStore / SharedPreferences, 구조화 데이터: Room
- [ ] **Room 스키마 export**(`exportSchema = true`) + 버전 올릴 때마다 **마이그레이션 테스트**
  - `room-testing` 의 `MigrationTestHelper` 는 kotlinx-serialization 버전이 앱과 맞아야 한다
    (안 맞으면 `AbstractMethodError`) — 필요하면 `constraints` 로 정렬
  - `fallbackToDestructiveMigration` 은 사용자 데이터를 지운다. 쓰지 않거나 명시적 결정으로만
- [ ] **백업 규칙** — `allowBackup`, `dataExtractionRules`(12+), `fullBackupContent`(11 이하)
  - [ ] 백업하면 안 되는 것 제외: 기기 고유 토큰, 캐시, 다른 기기에서 무의미한 URI 등
  - [ ] 복원 후 앱이 정상 동작하는지 (`adb shell bmgr` 로 확인)
- [ ] 민감 정보 저장 시 암호화 (Keystore 기반) 여부 `(결정)`
- [ ] 캐시는 `cacheDir` 에 — 시스템이 지울 수 있다는 전제로

## 6. 생명주기와 안정성

- [ ] **구성 변경** — 회전·다크 모드 전환·언어 변경·창 크기 변경 후 상태 유지 (`rememberSaveable`, ViewModel)
- [ ] **프로세스 종료 후 복원** — 개발자 옵션 "활동 유지 안 함" 또는
      `adb shell am kill` 로 확인. ViewModel 만으로는 안 되는 상태는 `SavedStateHandle`
- [ ] 백그라운드 작업 — 즉시 실행이 아니면 WorkManager. 포그라운드 서비스는 타입 선언 필수(14+)
- [ ] 메인 스레드 IO 금지 — 디버그 빌드에 StrictMode 켜서 확인
- [ ] **크래시 리포팅** `(결정)` — Firebase Crashlytics 등. 안 쓰면 Play Console Android vitals 만으로 볼지
- [ ] 메모리 — 큰 데이터를 통째로 올리지 않는지, 누수(LeakCanary 디버그 전용) `(결정)`
- [ ] 저장소 부족·네트워크 끊김·파일 삭제 등 **외부 상태 변화에서 죽지 않는지**

## 7. UI/UX 공통

- [ ] **다크 모드** — 모든 화면. 색·타이포는 `MaterialTheme` 토큰만 쓰고 하드코딩하지 않는다
- [ ] **edge-to-edge** — Android 15(targetSdk 35+)부터 강제. 상태바·내비게이션바·키보드(IME) 인셋 처리
- [ ] **예측형 뒤로 가기** — 13+ opt-in, 이후 버전 기본값. `BackHandler` / `OnBackPressedCallback` 사용,
      `onBackPressed()` 오버라이드 금지
- [ ] **다국어**
  - [ ] 사용자 노출 문자열은 전부 리소스, 지원 언어 전부 동시에 갱신
  - [ ] 앱별 언어 설정(13+) — `locales_config.xml` + `AppCompatDelegate.setApplicationLocales`
  - [ ] 숫자·날짜·복수형(`plurals`) 현지화
  - [ ] RTL 레이아웃(아랍어 등) 지원 여부 `(결정)`
- [ ] **접근성**
  - [ ] 아이콘 버튼 `contentDescription`, 장식용 이미지는 `null`
  - [ ] 터치 타겟 48dp 이상
  - [ ] 글꼴 크기 200% 에서 레이아웃이 깨지지 않는지
  - [ ] TalkBack 으로 주요 흐름 한 바퀴
  - [ ] 색 대비 (Accessibility Scanner)
- [ ] **화면 크기** — 가로 모드, 태블릿, 폴더블 펼침·접힘, 멀티 윈도우. 지원 안 하면 명시적으로 고정 `(결정)`
- [ ] **상태별 화면** — 로딩 / 빈 상태 / 오류 / 권한 없음 / 오프라인. 오류 화면에는 복구 동작(재시도·다른 선택)을 둔다
- [ ] 설정 화면 공통 항목 `(결정)` — 테마(시스템/라이트/다크), 언어, 앱 정보(버전·오픈소스 라이선스·개인정보처리방침 링크)
- [ ] **오픈소스 라이선스 고지** — 사용 라이브러리 목록 화면 (Play 필수는 아니지만 라이선스 조건상 필요)

## 8. 네트워크 (쓰는 경우)

- [ ] `INTERNET` 권한, 평문 HTTP 금지 (`network_security_config`)
- [ ] 타임아웃·재시도·오프라인 동작
- [ ] API 키를 앱에 넣지 않거나, 넣는다면 제한(패키지명·서명 SHA) 설정
- [ ] 인증서 고정(pinning) 여부 `(결정)`

## 9. 보안·개인정보

- [ ] **`exported` 점검** — 매니페스트의 Activity/Service/Receiver/Provider 중 외부에 열 필요 없는 것은 `false`
- [ ] 외부에서 받은 Intent·URI·파일 내용은 신뢰하지 않는다 (경로 조작, 과대 입력)
- [ ] 로그에 개인정보·토큰·파일 경로를 남기지 않는다 (릴리즈 빌드)
- [ ] `WebView` 를 쓰면 JavaScript·파일 접근 설정 최소화
- [ ] **개인정보처리방침** — 웹에 공개된 URL. 데이터를 수집하지 않아도 Play 에서 요구한다
- [ ] 서드파티 SDK 가 수집하는 데이터 파악 (데이터 보안 섹션에 반영)

## 10. 테스트와 품질

- [ ] **유닛 테스트** — Android API 에 의존하지 않는 순수 로직을 분리해 JVM 에서 테스트
- [ ] **계측 테스트** — DB 마이그레이션, 실제 기기 API 가 필요한 부분
  - minSdk 26 등 낮은 minSdk 에서는 **테스트 메서드 이름에 공백을 쓰면 DEX 빌드가 실패**한다
    (백틱 공백 이름은 JVM 유닛 테스트에서만)
- [ ] UI 테스트 — 핵심 흐름 몇 개 (Compose UI Test) `(결정)`
- [ ] **`./gradlew lint`** — 경고 정리 또는 baseline 로 고정 후 신규 경고만 막기
- [ ] **기기 매트릭스** — 최소 minSdk / targetSdk / 중간 버전 에뮬레이터 + 실기기 1대 이상
- [ ] 성능 — 콜드 스타트 시간, 스크롤 버벅임. Baseline Profile 도입 여부 `(결정)`
- [ ] **릴리즈 빌드**(R8 적용)로 최종 회귀 확인

## 11. 스토어 출시 (Google Play)

- [ ] 개발자 계정 (개인/조직) `(결정)` — 개인 계정은 신원 확인과 **비공개 테스트 요건**
      (일정 수 이상 테스터 · 일정 기간 이상 — 수치는 콘솔에서 확인)이 있다
- [ ] **스토어 등록 정보** — 앱 이름, 짧은 설명(80자), 자세한 설명, 지원 언어별 번역
- [ ] **그래픽** — 아이콘 512×512, 그래픽 이미지 1024×500, 휴대전화 스크린샷(최소 2장), 태블릿 스크린샷(태블릿 지원 시)
- [ ] **앱 콘텐츠 선언**
  - [ ] 개인정보처리방침 URL
  - [ ] 데이터 보안 섹션 (수집·공유 데이터, 암호화, 삭제 요청 방법)
  - [ ] 콘텐츠 등급 설문
  - [ ] 타깃 연령층, 광고 포함 여부
  - [ ] 민감한 권한 선언 (해당 시)
- [ ] 연락처 이메일 (공개된다), 웹사이트 `(선택)`
- [ ] **트랙 순서** — 내부 테스트 → 비공개 테스트 → 프로덕션. 프로덕션은 **단계적 출시**(예: 10% → 50% → 100%)
- [ ] 출시 노트 (언어별)
- [ ] 인앱 업데이트 / 인앱 리뷰 요청 도입 여부 `(결정)`

## 12. 출시 후 유지

- [ ] **Android vitals** 확인 — 크래시율·ANR 율이 기준을 넘으면 노출이 줄어든다
- [ ] **매년 `targetSdk` 상향** — Play 마감 전에. 동작 변경 목록(behavior changes)을 읽고 대응
- [ ] 의존성 업데이트 주기 `(결정)` — AGP·Kotlin·Compose BOM·AndroidX. 한 번에 몰아서 올리지 않는다
- [ ] 보안 공지된 라이브러리 버전 대응 (Play Console 경고)
- [ ] 사용자 리뷰·문의 응답 채널
- [ ] 버전별 변경 기록 (`CHANGELOG.md` 또는 Git 태그 + 릴리즈 노트)
