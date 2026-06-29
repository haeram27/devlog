# Folded Scalar

YAML에서 `>` 기호는 “folded scalar”라고 불리는 구문으로, 여러 줄에 걸쳐 작성한 문자열을 읽을 때 줄바꿈을 공백으로 바꾸어 하나의 문자열로 인식하게 만듭니다. 따라서 다음 예제의의 `command: >` 아래에 있는 여러 줄짜리 스크립트 전체가 공백을 사이에 두고 연결된 단일 명령어로 실행되게 됩니다.

```yaml
services:
  setup:
    image: ...
    user: "0"
    command: >
      bash -c '
        if [ x${APP_USER} == x ]; then
          echo "Set the APP_USER env variable in the .env file";
          exit 1;
        elif [ x${APP_PASSWORD} == x ]; then
          echo "Set the APP_PASSWORD env variable in the .env file";
          exit 1;
        fi;
```
