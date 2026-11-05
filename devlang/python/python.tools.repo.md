# Python 패키지 저장소(Index/Repository) 설정: uv와 pip

- [Python 패키지 저장소(Index/Repository) 설정: uv와 pip](#python-패키지-저장소indexrepository-설정-uv와-pip)
  - [개요](#개요)
  - [핵심 개념](#핵심-개념)
    - [용어 정리](#용어-정리)
    - [uv와 pip에서의 용어 매핑](#uv와-pip에서의-용어-매핑)
  - [사용 방법](#사용-방법)
    - [pip 설정](#pip-설정)
    - [uv 설정](#uv-설정)
  - [주의사항 / 트러블슈팅](#주의사항--트러블슈팅)
  - [검증 방법](#검증-방법)
  - [참고 자료](#참고-자료)

## 개요

- 무엇인지: Python 패키지 설치 도구(`pip`, `uv`)에서 패키지를 조회/다운로드할 **패키지 인덱스 URL**을 설정하는 방법이다.
- 언제 쓰는지: 기본 PyPI 대신 사내 저장소(Artifactory, Nexus, Azure Artifacts 등)를 사용하거나, 공용/사설 인덱스를 함께 사용할 때 필요하다.
- 한계/주의: URL 형식, 우선순위, 인증 처리 방식이 도구마다 다르므로 같은 개념이라도 설정 키를 정확히 구분해야 한다.

## 핵심 개념

### 용어 정리

- **Index(인덱스)**: 패키지 목록과 버전 메타데이터를 제공하는 엔드포인트
- **Repository / Registry(저장소/레지스트리)**: 실무에서 인덱스와 거의 같은 의미로 사용하는 경우가 많다.
- **Simple API**: Python 패키지 도구가 주로 사용하는 표준 인덱스 인터페이스(PEP 503). 일반적으로 URL이 `.../simple` 형태다.

예:

- 사람이 보는 프로젝트 페이지: `https://pypi.org/project/requests/`
- 도구가 조회하는 인덱스: `https://pypi.org/simple/requests/`


- PIPY
  - pypi.org는 전 세계 파이썬(Python) 개발자들이 만든 오픈소스 라이브러리와 패키지를 모아둔 공식 중앙 저장소(Python Package Index, PyPI)
  - pypi의 default URL은 `https://pypi.org/simple`

### uv와 pip에서의 용어 매핑

| 도구 | 대표 설정 키 | 의미 |
| --- | --- | --- |
| pip | `index-url`, `extra-index-url` | 기본 인덱스 / 추가 인덱스 |
| uv | `[[tool.uv.index]]`, `--index`, `--default-index` | 인덱스 목록 및 기본 인덱스 |

## 사용 방법

### pip 설정

- 1회성 설치 시:

```bash
python -m pip install requests --index-url https://pypi.org/simple
```

- 영구 설정(권장: `pip config` 사용):

```bash
python -m pip config set global.index-url https://pypi.org/simple
python -m pip config set global.extra-index-url https://repo.company.com/pypi/simple
python -m pip config set global.trusted-host repo.company.com
```

- 설정 파일 직접 작성 시(INI):

```ini
[global]
index-url = https://pypi.org/simple
extra-index-url =
    https://repo.company.com/pypi/simple
```

- Windows 사용자 설정 파일 경로 예:
  - `%APPDATA%\pip\pip.ini`

- 환경 변수 사용 시:
  - `PIP_INDEX_URL`
  - `PIP_EXTRA_INDEX_URL`

### uv 설정

- 프로젝트(`pyproject.toml`)에 인덱스 정의:

```toml
[[tool.uv.index]]
name = "company"
url = "https://repo.company.com/pypi/simple"

[[tool.uv.index]]
name = "pypi"
url = "https://pypi.org/simple"
default = true
```

- 해석 포인트:
  - `[[tool.uv.index]]`는 인덱스 엔트리를 여러 개 선언하는 방식이다.
  - `default = true`가 지정된 인덱스가 기본 인덱스 역할을 한다.
  - `--index`, `--default-index`, `UV_INDEX`, `UV_DEFAULT_INDEX`로 CLI/환경 변수에서 오버라이드할 수 있다.

- 1회성 실행 예:

```bash
uv lock --index company=https://repo.company.com/pypi/simple
uv lock --default-index https://pypi.org/simple
```

## 주의사항 / 트러블슈팅

- 인덱스 URL은 보통 `.../simple` 엔드포인트를 사용해야 한다.
- 사설 저장소 인증 토큰/비밀번호는 설정 파일에 평문으로 저장하지 말고 환경 변수 또는 자격 증명 저장소 사용을 우선한다.
- `pip`과 `uv`는 다중 인덱스 해석 전략이 다를 수 있으므로, 사내 패키지 사용 정책이 있으면 도구별 우선순위 동작을 확인한다.
- 네트워크 오류가 반복되면 프록시, TLS 인증서, 저장소 접근 권한(401/403), 인덱스 경로 오타를 먼저 점검한다.

## 검증 방법

- pip 설정 확인:

```bash
python -m pip config debug
python -m pip config list
```

- uv 설정 반영 확인:

```bash
uv lock -v
```

- 설치 대상 인덱스 확인(예시 패키지):

```bash
python -m pip install -v requests
uv pip install -v requests
```

로그에서 조회한 인덱스 URL이 기대값과 일치하면 설정이 정상 반영된 상태다.

## 참고 자료

- pip Configuration: https://pip.pypa.io/en/stable/topics/configuration/
- uv Package indexes: https://docs.astral.sh/uv/concepts/indexes/
- PEP 503 (Simple Repository API): https://peps.python.org/pep-0503/
