# ubuntu 에서 폰트 설정 하기

## google noto font 사용

```bash
sudo apt update
sudo apt install -y fonts-noto-cjk fonts-noto-color-emoji fonts-noto-mono
sudo fc-cache -fv
```

## 1. GNONE 터미널 폰트 설정 (코딩용)
우분투 기본 터미널(GNOME Terminal) 기준 설정 방법입니다.

   1. 터미널 실행 후 상단 메뉴(≡) → 기본 설정(Preferences)을 클릭합니다.
   2. 왼쪽 사이드바에서 현재 사용 중인 프로필(기본값은 '이름 없음' 또는 'Default')을 선택합니다.
   3. 텍스트 탭에서 사용자 지정 글꼴(Custom font) 체크박스를 활성화합니다.
   4. 글꼴 선택 창에서 Noto Sans Mono Regular를 검색하여 선택합니다.
   * 참고: 한글과 이모지는 시스템이 자동으로 Noto CJK와 Color Emoji로 연결(Fallback)하므로, 메인 폰트만 Mono로 설정하면 됩니다.

## TERMINATOR 터미널 폰트 설정

  1. Terminator 실행
  2. Context Menu(`Shift+F10`) > Preferences
  3. Profile > default > General
     1. disable `Use the system fixed width font
     2. Font : `Noto Sans Mono Regular`, size = 12

## 2. 우분투 전체 UI 폰트 설정
시스템 전체(메뉴 바, 창 제목 등) 폰트를 변경하려면 '기능 개선(GNOME Tweaks)' 도구가 필요합니다.

   1. 도구 설치: `sudo apt install gnome-tweaks`
   2. 프로그램 실행: 앱 목록에서 '기능 개선(Tweaks)'을 찾아 엽니다.
   3. 글꼴(Fonts) 탭으로 이동하여 다음 항목들을 변경합니다.
      - 인터페이스 텍스트: `Noto Sans CJK KR Regular` (또는 UI 최적화를 원하면 Noto Sans Arabic UI 등이나 일반 Sans 사용)
      - 문서 텍스트: `Noto Sans`
      - 고정 폭 텍스트: `Noto Sans Mono Regular`
   
## 3. VS Code 등 코드 편집기 설정
코딩 용도라면 편집기 설정도 중요합니다. settings.json이나 설정 UI에서 다음 값을 우선순위로 두세요.

- Editor: Font Family: 'Noto Sans Mono', 'Noto Sans CJK KR', 'Noto Color Emoji', monospace
- 이렇게 적어주면 영문/숫자는 Mono, 한글은 CJK, 이모지는 Emoji 폰트로 순서대로 불러옵니다.

## 4. 폰트 캐시 갱신 (필요 시)
설치 직후 리스트에 폰트가 보이지 않는다면 터미널에서 다음 명령어로 캐시를 수동 갱신해 주세요.

```bash
fc-cache -fv
```
- -f (force): 캐시 파일이 있더라도 강제로 다시 스캔합니다.
- -v (verbose): 처리되는 과정을 화면에 자세히 보여줍니다. 




