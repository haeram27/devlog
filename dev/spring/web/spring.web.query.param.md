# path parameter와 query parameter
주로 GET 메소드


## url parameter 참조를 위한 spring 
참고:
[spring handler methods](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/requestparam.html)

다음 mapping annotation들은 HTTP METHOD와 무관하여 GET/PUT/POST 등 Method와 관계 없이 임의의 HTTP request에서 필요한 위치의 값을 참조가 가능하다

- @PathVariable : path parameter (/{val}) 참조
- @RequestParam : query parameter (?key=value) 참조
- @RequestBody : http req 메세지의 Content 헤더의 값 참조, content 값 (보통 json 등 특정 형식 문서)을 지정한 Java 객체로 맵핑

## 경로 매개변수와 쿼리 매개변수
/api/v1/users?sort=name&limit=10

### 경로 매개변수 (Path Parameter) 와 @PathVariable
규칙: 특정 자원을 식별하기 위해 경로 매개변수를 사용합니다.
설명: 경로 매개변수는 리소스를 특정하는 데 사용됩니다.
예시:

좋은 예시: /api/v1/users/{userId} (특정 사용자 조회)
잘못된 예시: /api/v1/users?userId={userId}


### 쿼리 매개변수 (Query Parameter)
규칙: 필터링, 정렬, 페이지네이션과 같은 추가적인 요청 조건을 쿼리 매개변수로 지정합니다.
설명: 쿼리 매개변수는 URL 끝에 ?를 붙여 시작하고, &로 여러 매개변수를 연결합니다.
예시:

좋은 예시: /api/v1/users?sort=name&limit=10
잘못된 예시: /api/v1/users/sort_name