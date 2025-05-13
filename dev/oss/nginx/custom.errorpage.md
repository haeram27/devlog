
# nginx custom error page


## add error.html
path: `${NGINX_HOME}\html\error.html`

```html
<!DOCTYPE html>
<html>

<head>
    <title>Error</title>
    <style>
        html {
            color-scheme: light dark;
        }

        body {
            height: 100%;
            width: 35em;
            margin: 0 auto;
            font-family: Tahoma, Verdana, Arial, sans-serif;
        }
    </style>
</head>

<body>
    <h1>An error occurred.</h1>
    <p>Sorry, the page you are looking for is currently unavailable.<br /> Please try again later.</p>
    <p>If you are the system administrator of this resource then you should check the error log for details.</p>
</body>

</html>
```

## configure error_page in nginx.conf

refs:
https://nginx.org/en/docs/http/ngx_http_core_module.html#error_page

```
error_page <http-error-code> <uri>
```
error_page 설정은 http 또는 server 블럭에 작성할 수 있다.
http 블럭에 error_page 설정을 정의한 경우 error_page 설정의 uri에 해당하는 실제 html파일을 root 설정으로 지정해야하며, server 블럭에 error_page 설정을 정의된 경우 error_page 설정의 uri에 해당하는 실제 html파일을 location 설정으로 좀 더 상세히 지정할 수 있다.

http 블럭 보다 server 블럭 당 작성하는 것이 location 설정을 사용할 수 있기 때문에 좀 더더 안전한 설정을 할 수 있다.
http 블럭에 error_page 설정을 하게 되면 다음과 같은 단점이 있을수 있다.
-  error_page 설정의 마지막 uri에 지정되는 html 파일을 FS의 html과 매칭할 때 root (html등의 file resource root 경로) 설정만 사용할 수 있다. root 설정은 상속이 가능한 설정이기 때문에 server 블럭 등 http 블럭의 하위 블럭에서 root를 재정의 하면 http 블럭의 root 설정이 무시 되어 html을 찾을 수 없는 경우가 생길수 있다. 다시말하면 uri의 html 파일을 server 블럭 별로 접근할 수 있도록 설계를 고려해야 한다.
- http 블럭에는 location 설정을 지정 할 수 없기 alias와 internal 설정을 할 수 없어서 상세 설정이 불가능하다. alias 설정은 uri별로 직접 html을 지정할 수 있게 해주며, internal 설정은 location으로 지정하는 경로의 uri를 외부에서 접근할 수 없도록 하여 보안에 도움이 된다. 
 

### http 블럭에 작성할 경우 
```yml
http {
	root /permanent/etc/nginx/html;
	error_page
		400 401 402 403 404 405 406 408 409 410 411 412 413 414 415 416 421 429 497
		500 501 502 503 504 505 507
		/error.html;
}
```
`root /permanent/etc/nginx/html;`
- **문서 루트 설정**  
    HTTP 레벨(`http { … }`)에 `root`를 선언하면,  
    아래에 위치한 모든 `server`·`location` 블록에서 기본으로 사용할 파일 시스템 경로가 `/permanent/etc/nginx/html`로 통일됩니다.
    
- **스코프 우선순위**
    - 만약 개별 `server` 또는 `location` 블록에서 별도 `root`를 선언하지 않았다면,  
        HTTP 레벨 `root`가 그대로 적용됩니다.
    - 반대로, 더 좁은 컨텍스트(`server`나 `location`)에 `root`가 있으면 그 값이 우선합니다.


`error_page … /common-error.html;`
- **역할**  
    지정한 **4xx, 5xx 상태 코드**가 발생할 때마다,  
    클라이언트에 그 상태 코드를 직접 내보내지 않고  
    내부적으로 URI `/common-error.html`로 **재요청**(internal redirect)합니다.
    
- **매핑 과정**
    
    1. 클라이언트 요청 처리 중 (`try_files`나 다른 로직에서) 404 같은 에러가 발생
        
    2. Nginx가 “/common-error.html”로 내부 리다이렉트
        
    3. `/common-error.html` URI를 다시 처리하는데,
        
        - HTTP 레벨 `root`가 적용되어
            
        - `/permanent/etc/nginx/html/common-error.html` 파일을 찾습니다
            
    4. 해당 파일을 그대로 응답 바디로 돌려줍니다.
        
- **장점**
    
    - 여러 에러 코드를 **한 줄**로 묶어서 관리 가능
        
    - 서버 블록마다 따로 설정할 필요 없이, 전역 한 번만 선언으로 모든 가상호스트에 적용


### server 블럭에 작성할 경우 (recommended)
```
http {
    server {
        listen           8080;
        listen      [::]:8080;
        server_name http_server;

        error_page
            400 401 402 403 404 405 406 408 409 410 411 412 413 414 415 416 421 429 497
            500 501 502 503 504 505 507
            /common-error.html;
			
		location = /common-error.html {
			internal;
			alias /permanent/etc/nginx/html/error.html;
		}
    }
}
```

error_page :
- **역할**  
    이 서버 블록에서 발생하는 위 리스트의 상태 코드가 응답될 때,  
    클라이언트에게 해당 코드 자체를 내보내지 않고  
    내부적으로 URI `/common-error.html` 로 재요청(internal redirect)하게 만듭니다.
    
- **리스트된 코드**
    - **4xx** 클라이언트 오류 (잘못된 요청·권한 문제 등)
    - **5xx** 서버 오류 (내부 에러·과부하 등)
    - 특수 코드 `497` (nginx가 SSL 포트에 평문 HTTP가 들어왔을 때 사용하는 내부 코드)
        
- **이점**
    - 각각의 에러마다 별도 페이지를 만들 필요 없이,
    - 공통 페이지 하나로 모든 에러 화면을 관리할 수 있어 유지보수가 쉽습니다.

location :
- **`location =` (정확히 일치)**
       - URI가 정확히 `/common-error.html` 일 때만 이 블록이 적용됩니다.
    - 다른 경로(예: `/common-error.html/foo`)는 매칭되지 않아요.
        
- **`internal;`**
    - 클라이언트가 직접 `/common-error.html` 으로 요청했을 때 **접근을 차단**합니다.
    - 오직 `error_page` 등에 의한 내부 리다이렉션 때만 이 블록을 통해 응답됩니다.
        
- **`alias /permanent/etc/nginx/html/common-error.html;`**
    - 이 위치 블록이 처리할 실제 파일 경로를 **절대 경로**로 지정합니다.
    - `alias` 는 `root` 와 달리, location 매칭 부분을 잘라내고 남은 경로 대신 이 경로를 그대로 사용합니다.
    - 결과적으로 파일 시스템 상의 `/permanent/etc/nginx/html/common-error.html` 파일이 반환됩니다.