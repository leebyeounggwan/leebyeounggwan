# 캐시, 조건부 요청

#### 기본 동작
1. 클라이언트 요청 -> 서버
2. 서버 응답 -> 클라이언트 -> 캐시저장소에 저장
3. 재요청시 캐시저장소 확인하여 서버를 거치지 않고 응답

#### 캐시 유효기간 초과시
1. 서버에서 기존 데이터 변경한 경우 
   -> 서버 요청
2. 기존데이터 변경하지 않은 경우
   -> 캐시된 데이터와 서버의 데이터가 동일하다면 새로 다운받아 갱신하지 않고 재사용하는 것이 효율적
      (서버 데이터와 캐시된 데이터가 동일한지 확인하기 위해 *검증헤더* 사용)

	1) 첫번째 요청에 대한 응답시 *검증헤더*(Last-Modified) 
	```
	cache-control: max-age=60
	Last-Modified: 2023년 11월 01일 00:00:00
	```

	2) 캐시 유효기간 만료 후 두번째 요청 시 *조건부 요청*
	```
	GET /~~
	if-modified-since: 2023년 11얼 01일 00:00:00
	```

	3) 두번째 요청에 대한 응답
	```
	HTTP/1.1 304 Not Modified	
	Content-Type: ~~
	cache-control: ~~
	Last-Modified: 2023년 11얼 01일 00:00:00
	Content-Length: ~~
	## HTTP Body 없이 헤더만 전송
	```

 	4) 클라이언트는 서버의 응답 헤더정보로 캐시의 메타정보 갱신 후 재활용

> 캐시의 유효기간이 만료되더라도 데이터 갱신이 없는 경우 네트워크검증헤더와 조건부 요청을 통해 컨텐츠를 다운로드 하지 않고,
   용량이 적은 헤더정보만 활용하여 네트워크 부하를 최소화 할 수 있다.
 

---

### 검증헤더와 조건부 요청
: 데이터가 수정되어 Last-Modified는 변경되었지만, 데이터는 바뀌지 않는 경우가 있음

#### ETag(Entity Tag)
: 캐시용 데이터에 임의의 고유한 버전 이름을 달아둠
: 데이터 변경시 이 이름을 변경하여 데이터 수정여부 판단가능

	```
	GET /~~~
	if-None-Match: "aaaa" <-ETag
	```	

> 캐시 제어 로직을 서버에서 관리
	 ex) 애플리케이션 배포주기에 맞춰 Etag 갱신


### 캐시와 관련된 헤더
1. 캐시제어 헤더
		- Cache-Control :
											max-age = 캐시유효시간(초)
											no-cache = 데이터 캐시 O But 항상 Origin서버에 검증 후 사용
						 				no-store = 데이터에 민감한 정보가 있으므로 저장 x

		- Pragma : 캐시제어(하위 호환)
  - Expires : 날짜로 캐시 유효기간 지정(하위 호환)
				(더 유연한 max-age 권장, max-age와 함께 사용시 Expires 무시)

2. 검증 헤더(Validator)
		- ETag
		- Last-Modified

3. 조건부 요청 헤더
		- If-Match, If-None-Match (ETag 값 사용)
		- If-Modified-Since, If-Unmodified-Since( Last-Modified 값 사용)


### 프록시 캐시
: 클라이언트와 서버 사이에 프록시캐시 서버를 두고 클라이언트는 프록시 서버로 접근

Cache0-control:
									public = 응답이 public 캐시(프록시)에 저장되어도 됨
									private = 응답이 해당 사용자 만을 위한 것(Default)
									s-maxage = 프록시 캐시에만 적용되는 max-age
									Age: 60 = 오리진 서버에서 응답 후 프록시 캐시 내에 머문 시간(초)


### 캐시 무효화
: 캐시를 적용하지 앟ㄴ아도 웹 브라우져에서 캐시를 하는 경우가 있기 때문에 캐시가 되지 않길 바라는 경우 캐시 무효화 해주어야 한다.

Cache-Control:
							no-cache
							no-store
							must-revalidate = 캐시 만료 후 최초 조회시 원서버에 검증
																									,원서버 접근 실패시 반드시 오류 발생(504)
																									,유효시간 내 캐시 사용

Pragma:
				no-cache(HTTP 1.0 하위호환)


*no-cache vs must-revalidate*
> no-cache는 origin 서버와 네트워크가 안될 경우 UX측면에서 이전 캐시를 사용하는 옵션도 있다.
  must-revalidate는 origin서버 접근 불가시 오류를 발생시키기 때문에 상황에 따라서 두가지를 같이 사용하면 좋음































