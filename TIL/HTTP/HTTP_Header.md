# HTTP 헤더

1. 용도 
: HTTP 전송에 필요한 모든 부가정보(크기, 압축, 인증, 클라이언트, 서버 정보, 캐시관리 등)

2. HTTP BODY
: 메시지 본문(payload)를 통해 표현(Representaion) 데이터 전달 / 표현헤더 + payload

2.1 표현 헤더
: 표현 데이터를 해석할 수 있는 정보 제공
- Content-Type : 표현 데이터 형식(text/html, application/json)
- Content-Encoding : 표현 데이터의 압축 방식 (인코딩 정보를 바탕으로 압축해제)
- Content-Language : 자연 언어
- Content-Length : 표현 데이터의 길이/바이트단위 (정확하게는 payload의 header)
* 전송, 응답 둘다 사용

3. 협상(컨텐츠 네고시에이션)
: 클라이언트가 선호하는 표현요청
- Accept : 클라이언트가 선호하는 미디어 타입
- Accept-Charset : 클라이언트가 선호하는 문자 인코딩
- Accept-Encoding : 클라이언트가 선호하는 압축 인코딩
- Accept-Language :  클라이언트가 선호하는 자연 언어
* 협상헤더는 요청시에만 사용

>> 협상헤더에 담아서 보내면 서버에서 요청에 맞춰 응답(가능할 경우)
ex) 독일어 사이트에 한글 요청 -> 영어만 있음 -> 한글이 없기 때문에 독일어로 응답

3.1 협상과 우선순위
: Quality Values 값을 사용하여 우선순위 지정 가능 (0~1 / 높을수록 우선순위 높음 / 생략시 1)
`Accept-Language: ko-KR,ko;q=0.9;en-US;q=0.8;en;q=0.7`

4. 전송방식
- 단순전송
- 압축전송
- 분할전송
- 범위전송

5. 일반정보
- From : 유저 에이전트의 이메일 정보(주로 검색엔진에서 사용)
- Referer : 이전 웹 페이지의 주소(A->B 이동시 B로 요청할 때 Referer:A를 포함하여 요청), 유입경로 분석
- User-Agent : 유저 에이전트 애플리케이션 정보(웹브라우져 등) => 어떤 브라우저에서 장애 발생하는지 파악가능
- Server : 요청을 처리하는 origin 서버의 소프트웨어 정보
- Date : 메시지 생성 날짜

6. 특별정보

6.1 Host 
: 요청한 호스트 정보(도메인) *필수*
: 하나의 서버가 여러 도메인을 처리해야 할 때, 하나의 IP에 여러 도메인이 적용되어 있을 때 
```
GET /welcome HTTP/1.1
Host:aaa.com
```

6.2 Location
: 3xx 응답 결과에 Location 헤더가 있으면 해당 위치로 자동 리다이렉트

6.3 Allow 
: 허용 가능한 HTTP메서드 (405)

6.4 Retry-After
: 서비스가 언제까지 불능인지 알려 줄 수 있음 503(Service Unavailable) 


7. 인증
Authorization : 클라이언트 인증 정보를 서버에 전달
WWW-Authenticate : 리소스 접근 시 필요한 인증 방법 정의(401 Unauthorized)

8. 쿠키
- Set-Cookie : 서버에서 클라이언트로 쿠키 전달(응답)
- Cookie : 서버로부터 받은 쿠키 저장, HTTP 요청시 서버로 쿠키 전달

쿠키 필요성 : HTTP는 Stateless이기 때문에 필요시(인증, 사용자정보 등) 쿠키에 담아 응답을 보내고 
      		   클라이언트는 쿠키저장소에 해당 쿠키를 저장 후 이후 요청 시 쿠키저장소를 확인하고 요청애 포함하여 전송
`set-cookie: sessionId=xxxxx; expires=Sat, 26-Dec-2023 00:00:00 GMT; path=/; domain=google.com; Secure`

8.1 사용처 : 사용자 로그인 세션관리, 광고정보 트래킹 등

# 쿠키 정보는 항상 서버에 전송됨
: 추가 트래픽을 유발하기 때문에 최소한의 정보(세션id, 토큰)만 사용, 민감한 데이터는 X
(서버에 전송하지 않고 저장할 때는 웹 스토리지(localStorage/sessionStorage) 참고)

8.2 쿠키 생명주기
- Set-Cookie: expires= ~~~~   -> 만료일이 되면 쿠키 삭제
- Set-Cookie: max-age=3600(초) -> 시간지정

8.3 쿠키 종류
- 세션 쿠키 : 만료 날짜 생략시 브라우저 종료시까지 유지
- 영속 쿠키 : 만료 날짜 입력하면 해당 날짜까지 유지

8.4 쿠키 도메인
명시 : 명시한 문서 기준 도메인 + 서브 도메인 포함
생략 : 현재 문서 기준 도메인만 적용

8.5 쿠키 경로
: 해당 경로를 포함한 하위 경로 페이지만 쿠키 접근(일반적으로 path=/ 루트로 지정)

8.6 쿠키 보안
- Secure : https 일 경우에만 전송
- HttpOnly : XSS 공격 방지 (자바스크립트 접근 불가), HTTP전송에만 사용
- SameSite : XSRF 공격 ㅂ아지 (요청 도메인과 쿠키에 설정된 도메인이 같은 경우에만 쿠키 전송)

























