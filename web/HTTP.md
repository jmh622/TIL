# URL

- URI (Uniform Resource Identifier)
  - URI는 locator(URL), name(URN) 또는 둘 다 추가로 분류될 수 있다
- URL (Uniform Resource Locator)

  - 리소스가 있는 위치를 지정

## 구조

- scheme://[userinfo@]host[:port][/path][?query][#fragment]
- https://www.google.com:443/search?q=hello&hl=ko

# HTTP 메시지 구조

- start-line
- header
- empty line
- message body

## 예시

- Request
  - GET /search?q=hello&hl=ko HTTP/1.1
  - Host: www.goole.com
  -
- Response

  - HTTP/1.1 200 OK
  - Content-Type: text/html;charset=UTF-8

    Content-Length: 3423

  -
  - `<html></html>`

# Method

- GET
  - 리소스를 가져올 때 사용
  - Idempotent: Yes
- POST
  - 리소스를 생성할 때 사용
  - Idempotent: No
- PUT
  - 리소스를 덮어씌우거나 생성할 때 사용
  - Idempotent: Yes
- DELETE
  - 리소스를 삭제할 때 사용
  - Idempotent: Yes
- PATCH
  - 리소스를 부분 적으로 수정할 때 사용
  - Idempotent: No

## Idempotent (멱등성)

동일한 요청을 한 번 보내는 것과 여러 번 연속으로 보내는 것의 결과가 같을 때 'Idempotent하다'라고 말한다.

# Response Status Code

- 2XX Successful
  - 200 OK
  - 201 Created
  - 204 No Content
- 3XX Redirection
  - 301 Moved Permanently
  - 302 Found
- 4XX Client error
  - 400 Bad Request
  - 401 Unauthorized (미인증)
  - 403 Forbidden (미인가, 미승인)
  - 404 Not Found
  - 405 Method Not Allowed
- 5XX Server error
  - 500 Internal Server Error
  - 502 Bad Gateway
  - 503 Service Unavailable

# Headers

- Authentication
  - Authorization: 인증 정보
- Cookies
  - Cookies: 모든 쿠키 정보
  - Set-Cookie: 서버에서 클라이언트로 쿠키를 전송
    - Expires
      - date
    - Max-Age
      - number (second)
      - Expires와 Max-Age가 둘 다 설정되어 있으면 Max-Age를 우선시한다.
    - Secure
      - HTTPS인 경우에만 전송
    - HttpOnly
      - XSS 공격 방지
      - 자바스크립트로 접근 불가
    - SameSite
      - XSRF 공격 방지
      - 요청 도메인과 쿠키에 설정된 도메인이 같은 경우에만 전송
- Message body information
  - Content-Type: 리소스의 미디어 타입 표시
  - Content-Length
  - Content-Encoding
  - Content-Language
- Content negotiation
  - Accept: 클라이언트 선호 미디어 타입
  - Accept-Charset
  - Accept-Encoding
  - Accept-Language
- Request context
  - Host
  - Referer: 이전 웹 페이지 주소
  - User-Agent: 클라이언트의 애플리케이션 타입, 운영 체제, 소프트웨어 벤더/버전
- Response context
  - Allow: 허용 가능한 HTTP 메서드 나열
  - Server: 오리진 서버 정보
- Redirects
  - Location: 페이지를 리다이렉트할 URL 표시

# HTTP version

- v1
  - text-based
  - uncompressed headers
  - one file at a time
- v2
  - binary based
  - header compression
  - multiplexing
  - stream prioritization
  - secure
- v3
  - only allow secure
  - UDP
