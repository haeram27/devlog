# Postgresql 연산자

- PostgreSQL은 jsonb 타입을 처리하기 위한 다양하고 강력한 내장 함수와 연산자를 제공합니다. 주요 jsonb 관련 함수 및 연산자는 다음과 같습니다. 

- 다음의 연산자와 함수들은 데이터베이스에 저장된 비정형 JSON 데이터를 효과적으로 쿼리하고 조작하는 데 사용

## 주요 JSONB 연산자

함수 외에도 편리한 연산자들을 자주 사용합니다.

- `->` (integer/text): JSON 배열의 요소를 인덱스로 추출하거나, JSON 객체의 필드를 키로 추출합니다 (결과: jsonb).
- `->>` (integer/text): JSON 배열 요소를 텍스트로, 또는 객체 필드를 텍스트로 추출합니다 (결과: text).
- `#>` (text[]): 주어진 경로의 JSON 객체를 추출합니다. (중요)
- `#>>` (text[]): 주어진 경로의 JSON 객체를 텍스트로 추출합니다.(중요)
- `@>` (jsonb): 왼쪽 jsonb가 오른쪽 jsonb를 포함하는지 확인합니다.
- `<@` (jsonb): 왼쪽 jsonb가 오른쪽 jsonb에 포함되는지 확인합니다.
- `?` (text): 문자열이 JSON 객체의 키 또는 배열의 요소로 존재하는지 확인합니다.
- `||` (jsonb): 두 jsonb 객체 또는 배열을 병합합니다.
- `-` (text/integer): 키나 인덱스로 요소를 제거합니다.

## JSONB 생성 및 변환 함수

- to_jsonb(anyelement): 임의의 값을 jsonb로 변환합니다.
- jsonb_build_array(variadic "any"): 인자 목록으로부터 jsonb 배열을 생성합니다.
- jsonb_build_object(variadic "any"): 키-값 쌍의 목록으로부터 jsonb 객체를 생성합니다.
- jsonb_object(text[]), jsonb_object(text[], text[]): 텍스트 배열로부터 jsonb 객체를 생성합니다.
- jsonb_object_agg(key, value): 여러 행의 데이터를 하나의 jsonb 객체로 병합하여 집계합니다.
- jsonb_array_elements(jsonb): jsonb 배열을 jsonb 값의 집합으로 확장합니다.
- jsonb_array_elements_text(jsonb): jsonb 배열을 텍스트 값의 집합으로 확장합니다.

## JSONB 조회 및 추출 함수

- jsonb_each(jsonb): jsonb 객체의 최상위 키-값 쌍을 집합으로 반환합니다.
- jsonb_each_text(jsonb): jsonb 객체의 최상위 키-값 쌍을 텍스트 집합으로 반환합니다.
- jsonb_extract_path(from_json jsonb, variadic path_elems text[]): 주어진 경로의 jsonb 값을 추출합니다 (#>).
- jsonb_extract_path_text(from_json jsonb, variadic path_elems text[]): 주어진 경로의 jsonb 값을 텍스트로 추출합니다 (#>>).

## JSONB 수정 및 조작 함수

- jsonb_set(target jsonb, path text[], new_value jsonb, [create_missing boolean]): 주어진 경로의 값을 new_value로 설정/수정합니다.
- jsonb_insert(target jsonb, path text[], new_value jsonb, [insert_after boolean]): jsonb 구조에 값을 삽입합니다.
- jsonb_strip_nulls(from_json jsonb): jsonb 객체에서 null 값을 가진 필드를 모두 제거합니다.

## JSONB 정보 함수

- jsonb_array_length(jsonb): jsonb 배열의 길이를 반환합니다.
- jsonb_typeof(jsonb): jsonb 최상위 값의 형식을 텍스트(object, array, string, number, boolean, null)로 반환합니다.
- jsonb_pretty(jsonb): jsonb 데이터를 들여쓰기된 텍스트로 변환합니다.