# 실제 Android 기기에서 테스트하기

개발 중인 앱을 USB(또는 Wi-Fi)로 연결한 실제 폰에 설치·실행·삭제하는 순서.
에뮬레이터도 명령은 같다 (`adb devices` 에 `emulator-5554` 로 보일 뿐이다).

- 개발 PC: Ubuntu 24.04 기준. Windows / macOS 는 차이가 있는 단계에만 따로 적었다
- 이 앱 값: 패키지 `com.example.myapplication`, 시작 Activity `.ui.MainActivity`
- `adb` 위치: `$ANDROID_HOME/platform-tools/adb` (이 PC 는 `~/dev/android/sdk`, PATH 에 있음)

---

## 1. 폰 준비 — 개발자 옵션과 USB 디버깅

1. **개발자 옵션 켜기** — 설정 → 휴대전화 정보 → 소프트웨어 정보 → **빌드 번호를 7번 탭**
   - 삼성: 설정 → 휴대전화 정보 → 소프트웨어 정보 → 빌드번호
   - 픽셀: 설정 → 휴대전화 정보 → 빌드 번호
   - "개발자 모드를 사용하도록 설정했습니다" 메시지가 나오면 된다 (잠금 화면 PIN 을 물을 수 있다)
2. **USB 디버깅 켜기** — 설정 → 개발자 옵션 → **USB 디버깅** ON
3. (선택) **USB를 통해 설치** / **USB 디버깅(보안 설정)** — 샤오미 등 일부 제조사는 이것까지 켜야 설치된다
4. (선택) **화면 켜짐 상태 유지** — 충전 중 화면이 꺼지지 않아 테스트가 편하다

> 테스트가 끝나면 USB 디버깅을 꺼 두는 것이 안전하다. 켜 둔 채 공용 충전기에 꽂지 않는다.

## 2. PC 준비 — USB 접근 권한 (드라이버)

### Linux (Ubuntu)

Linux 는 **별도 드라이버가 필요 없다.** 대신 일반 사용자가 USB 장치에 접근하도록 **udev 규칙**이 필요하다.
규칙이 없으면 `adb devices` 에 `no permissions` 가 뜬다.

udev 규칙은 **아래 방법 1 과 방법 2 중 하나만 택해서** 준비하면 된다. 둘 다 할 필요는 없다.
Android SDK 는 `adb` 실행 파일만 제공하고 udev 규칙은 설치하지 않으므로, SDK 가 있어도 이 단계는 따로 필요하다.

```bash
# ── 방법 1 과 방법 2 중 택 1 ──

# 방법 1 — 패키지로 설치 (주요 제조사 벤더 ID 가 미리 들어 있는 규칙 파일)
#   이 패키지의 실질적인 내용은 udev 규칙(51-android.rules)이다. adb 는 들어 있지 않다
sudo apt install android-sdk-platform-tools-common

# 방법 2 — 직접 작성 (패키지를 쓰기 싫거나, 방법 1 에 없는 제조사의 기기)
lsusb                                   # 폰의 "ID 04e8:6860" 같은 값에서 앞 4자리가 벤더 ID
echo 'SUBSYSTEM=="usb", ATTR{idVendor}=="04e8", MODE="0660", GROUP="plugdev"' \
    | sudo tee /etc/udev/rules.d/51-android.rules    # 04e8 = 삼성, 18d1 = 구글

# 공통 — 규칙 다시 읽기, 그룹 확인
sudo udevadm control --reload-rules && sudo udevadm trigger
id -nG | grep -w plugdev                # 없으면: sudo usermod -aG plugdev $USER 후 재로그인
```

### Windows

- **Google USB Driver** — Android Studio → SDK Manager → SDK Tools → "Google USB Driver" 설치 (픽셀·레퍼런스 기기)
- **제조사 드라이버** — 삼성은 "Samsung Android USB Driver", 그 외 제조사 사이트에서 OEM USB 드라이버
- 장치 관리자에 "Android ADB Interface" 로 보이면 된다

### macOS

- 드라이버·설정 필요 없음

## 3. 연결과 인증

1. **데이터 전송이 되는 USB 케이블**로 연결한다 (충전 전용 케이블은 목록에 안 뜬다)
2. 폰의 USB 연결 알림에서 모드를 **파일 전송(MTP)** 으로 바꾸면 인식이 잘 되는 기기가 있다
3. 연결 확인:
   ```bash
   adb devices -l
   ```
4. 폰에 **"USB 디버깅을 허용하시겠습니까?"** 창이 뜨면 → **"이 컴퓨터에서 항상 허용"** 체크 후 허용
5. 다시 `adb devices -l` 로 상태 확인:

| 상태 | 의미 / 조치 |
|---|---|
| `device` | 정상 |
| `unauthorized` | 폰에서 허용 창을 누르지 않았다. 폰 화면 확인. 안 뜨면 개발자 옵션 → "USB 디버깅 권한 승인 취소" 후 재연결 |
| `no permissions` | (Linux) udev 규칙 없음 → 2절 |
| 목록에 없음 | 케이블·USB 모드·USB 디버깅 확인. `adb kill-server && adb start-server` |
| `offline` | 재연결, 또는 `adb kill-server` |

### (선택) Wi-Fi 로 연결 — Android 11+

케이블 없이 같은 Wi-Fi 에서 연결한다.

1. 개발자 옵션 → **무선 디버깅** ON → "페어링 코드로 기기 페어링"
2. 폰에 표시된 **IP:포트와 6자리 코드**로 페어링:
   ```bash
   adb pair 192.168.0.23:37251          # 페어링용 포트 — 코드를 입력하라고 한다
   adb connect 192.168.0.23:41603       # 무선 디버깅 화면의 "IP 주소 및 포트" (페어링 포트와 다르다)
   ```
3. 끊기: `adb disconnect`

## 4. 빌드와 설치

빌드는 저장소의 Gradle wrapper(`./gradlew`)로 한다. 시스템 `gradle` 은 쓰지 않는다 (`CLAUDE.md`).

```bash
# 방법 1 — Gradle 이 빌드 + 설치 (연결된 기기 전부에 설치)
./gradlew installDebug

# 방법 2 — 빌드 후 adb 로 직접 설치
./gradlew assembleDebug
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

| `adb install` 옵션 | 의미 |
|---|---|
| `-r` | 이미 설치돼 있으면 **데이터를 유지한 채** 교체 |
| `-d` | 낮은 `versionCode` 로 되돌리기 허용 (디버그 빌드만) |
| `-t` | `testOnly` APK 설치 허용 |
| `-g` | 매니페스트의 런타임 권한을 모두 미리 허용 |

- 폰에 "출처를 알 수 없는 앱" 경고나 Play 프로텍트 검사 창이 뜨면 허용한다 (디버그 서명이라서)
- **서명이 다른 빌드로 덮어쓸 수 없다** — 디버그 ↔ 릴리즈, 다른 PC 의 디버그 키로 만든 APK 는
  `INSTALL_FAILED_UPDATE_INCOMPATIBLE` 이 난다 → 먼저 삭제(7절)하고 설치. **앱 데이터가 지워진다**

## 5. 실행과 종료

```bash
# 실행 — 런처 아이콘을 누른 것과 같다
adb shell monkey -p com.example.myapplication -c android.intent.category.LAUNCHER 1

# 실행 — Activity 를 직접 지정 (-W: 실행 완료까지 기다리고 시작 시간을 출력)
adb shell am start -W -n com.example.myapplication/.ui.MainActivity

# 강제 종료 — 프로세스까지 죽인다 (재실행 시 "마지막 위치 복원" 같은 동작 확인용)
adb shell am force-stop com.example.myapplication

# 백그라운드 상태에서 프로세스만 종료 — 시스템이 메모리 부족으로 죽인 상황 재현
#   (홈 버튼으로 앱을 백그라운드에 보낸 뒤 실행)
adb shell am kill com.example.myapplication
```

## 6. 테스트 중 자주 쓰는 명령

### 로그

```bash
adb logcat -c                                                        # 기존 로그 비우기
adb logcat --pid=$(adb shell pidof -s com.example.myapplication)     # 이 앱 로그만
adb logcat -s FileReaderViewModel FileTextReader UriPermissions      # 태그로 거르기
adb logcat -b crash                                                  # 크래시만
```

### 파일 넣기 / 꺼내기

```bash
adb push notes.txt /sdcard/Download/        # 폰의 다운로드 폴더로 복사 → 앱의 파일 선택기에서 보인다
adb pull /sdcard/Download/notes.txt .
adb shell rm /sdcard/Download/notes.txt     # "파일을 찾을 수 없음" 상황 재현
```

### 화면 캡처 / 녹화

```bash
adb exec-out screencap -p > shot.png
adb shell screenrecord /sdcard/rec.mp4      # Ctrl+C 로 멈춤 (최대 3분)
adb pull /sdcard/rec.mp4 .
```

### 상태 확인

```bash
adb shell dumpsys package com.example.myapplication | grep -E 'versionCode|versionName|firstInstall|lastUpdate'
adb shell dumpsys activity permissions | grep -A3 com.example.myapplication   # SAF 영구 URI 권한
adb shell settings put system user_rotation 1     # 가로 (자동 회전 꺼진 상태에서)
adb shell cmd uimode night yes                    # 다크 모드 (no 로 해제)
```

### 계측 테스트 (DB 마이그레이션 등)

```bash
./gradlew connectedAndroidTest                 # 연결된 기기에서 app/src/androidTest 실행
# 결과: app/build/reports/androidTests/connected/debug/index.html
```

## 7. 데이터 초기화와 삭제

```bash
# 앱은 두고 데이터만 지우기 — 설정 → 앱 → 저장공간 → 데이터 삭제 와 같다
#   (Room DB, SharedPreferences, SAF 영구 권한 모두 사라진다. 첫 실행 상태 확인용)
adb shell pm clear com.example.myapplication

# 앱 삭제
adb uninstall com.example.myapplication
adb uninstall -k com.example.myapplication  # 데이터·캐시는 남기고 삭제 (재설치 시 이어서 쓴다)

# Gradle 로 삭제 (디버그 앱 + 테스트 APK)
./gradlew uninstallAll
```

## 8. 기기가 여러 대일 때

에뮬레이터와 폰을 같이 연결하면 `adb` 명령이 `more than one device/emulator` 로 실패한다.

```bash
adb devices -l                              # 시리얼 확인 (예: R3CN90XXXXX, emulator-5554)
adb -s R3CN90XXXXX install -r app-debug.apk
export ANDROID_SERIAL=R3CN90XXXXX           # 이후 모든 adb 명령의 기본 기기로
```

`./gradlew installDebug` / `./gradlew connectedAndroidTest` 는 연결된 기기 **전부**에 설치·실행한다.

## 9. 한 번에 — 자주 쓰는 순서

```bash
adb devices -l                                          # 1. 연결 확인 (device 상태)
./gradlew installDebug                                  # 2. 빌드 + 설치
adb logcat -c                                           # 3. 로그 비우기
adb shell am start -W -n com.example.myapplication/.ui.MainActivity   # 4. 실행
adb logcat --pid=$(adb shell pidof -s com.example.myapplication)      # 5. 로그 보며 테스트
adb shell am force-stop com.example.myapplication       # 6. 종료
adb uninstall com.example.myapplication                 # 7. (필요할 때) 삭제
```

## 문제 해결

| 증상 | 원인 / 조치 |
|---|---|
| `no permissions (missing udev rules?)` | Linux udev 규칙 → 2절 |
| `unauthorized` | 폰의 USB 디버깅 허용 창 → 3절 |
| `INSTALL_FAILED_UPDATE_INCOMPATIBLE` | 서명이 다른 앱이 이미 설치됨 → 삭제 후 설치 (데이터 삭제됨) |
| `INSTALL_FAILED_VERSION_DOWNGRADE` | 설치된 것보다 낮은 `versionCode` → `adb install -r -d` |
| `INSTALL_FAILED_USER_RESTRICTED` | 제조사 옵션 "USB를 통해 설치" 꺼짐 (샤오미 등) → 1절 3번 |
| `INSTALL_FAILED_INSUFFICIENT_STORAGE` | 폰 저장 공간 부족 |
| `more than one device/emulator` | 기기 여러 대 → 8절 |
| `pidof` 결과가 비어 로그가 안 나옴 | 앱이 실행 중이 아니다. 먼저 실행한 뒤 `logcat --pid` |
| 설치는 되는데 아이콘이 안 보임 | 런처가 새로고침 전. 5절 명령으로 직접 실행 |
