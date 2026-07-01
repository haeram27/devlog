# Java LSP (jdtls) 설정 가이드

OpenCode에서 `.java` / `.kt` 파일에 대해 `lsp_diagnostics`, `goto_definition`,
`find_references`, `rename` 등 LSP 도구를 사용하기 위한 설정 방법입니다.

---

## 환경 요구사항

| 항목 | 요구사항 |
|------|----------|
| JDK | **17 이상** (서버 실행용; 프로젝트 타겟 버전과 무관) |
| jdtls | PATH에 등록된 `jdtls` 실행파일 |

현재 프로젝트 환경: **OpenJDK 25.0.3** ✅

---

## Step 1 — jdtls 설치 (Linux)

### 방법 A: 패키지 매니저

```bash
# Ubuntu / Debian
sudo apt-get install jdtls

# Arch Linux
sudo pacman -S jdtls
```

### 방법 B: 직접 다운로드

1. [eclipse-jdtls Releases](https://github.com/eclipse-jdtls/eclipse.jdt.ls/releases) 에서 최신 릴리스 다운로드
2. 압축 해제 후 `bin/jdtls` 를 PATH에 추가

```bash
# 예시
tar -xzf jdt-language-server-*.tar.gz -C ~/.local/jdtls
echo 'export PATH="$HOME/.local/jdtls/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### 설치 확인

```bash
command -v jdtls
# 출력 예: /usr/bin/jdtls
```

---

## Step 2 — OpenCode 프로젝트 설정

`jdtls`는 builtin 서버이므로 `.java` 파일에 **별도 설정 없이 자동 적용**됩니다.

아래 옵션이 필요한 경우에만 `.opencode/lsp.json`을 생성하세요.

### 기본 설정 (우선순위 지정)

```json
{
  "lsp": {
    "jdtls": {
      "priority": 100
    }
  }
}
```

### JAVA_HOME 수동 지정 (jdtls가 JDK를 못 찾는 경우)

```bash
# 현재 JAVA_HOME 확인
echo $JAVA_HOME
# 또는
readlink -f $(which java) | sed 's|/bin/java||'
```

```json
{
  "lsp": {
    "jdtls": {
      "env": {
        "JAVA_HOME": "/usr/lib/jvm/java-25-openjdk"
      }
    }
  }
}
```

---

## Step 3 — 동작 확인

jdtls 설치 후 아래 명령으로 LSP 연결을 검증합니다.

```bash
# 실제 .java 파일 경로로 교체
bun ~/.cache/opencode/packages/oh-my-openagent@latest/node_modules/oh-my-openagent/dist/skills/lsp-setup/scripts/verify-lsp.ts \
  server/src/main/java/com/ahnlab/one/platform/deployment/server/grpc/ConsoleS3PresignGrpcService.java
```

| 출력 | 의미 |
|------|------|
| `OK` | LSP 정상 작동 |
| `FAIL: language server not installed` | Step 1 재확인 |
| `FAIL: ...` | 오류 메시지 확인 후 Troubleshooting 참고 |

---

## Troubleshooting

| 증상 | 해결 방법 |
|------|-----------|
| `jdtls: command not found` | PATH 재확인, 셸 재시작 |
| 서버가 즉시 종료됨 | `JAVA_HOME`을 JDK 17+ 경로로 명시 |
| 첫 실행이 느림 (1분 이상) | 정상 — 최초 classpath 인덱싱 시간 |
| 심볼 해석 오류 | `build.gradle` / `pom.xml` 문법 오류 확인 |
| 인덱스 오류 (의존성 변경 후) | jdtls workspace 캐시 삭제 후 재시작 |

### jdtls workspace 캐시 삭제

```bash
rm -rf ~/.cache/jdtls
# 또는 jdtls가 생성한 데이터 디렉토리 (경로는 설치 방식에 따라 다름)
```

---

## 참고 링크

- [eclipse-jdtls GitHub](https://github.com/eclipse-jdtls/eclipse.jdt.ls)
- [OpenCode LSP 설정 문서](https://opencode.ai/docs/lsp)
