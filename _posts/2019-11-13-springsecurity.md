---
layout: post
date : 20191113
title : "Spring Security"
---


1. rest api시로 인증시 필요한 부분 (비동기)

1.1  api key(key만 알게될경우 뚫려버림), api token 

1.2 api권한 인가 
- api gateway
: 요청한 사용자의 아이디가 token과 매핑되는 아이디와 일치하는경우는 접근권한 통제 쉽지만 
  admin권한이나 필요정보가 http body나 header에 들어가있는 경우 해당 메세지 전체를 파싱해서 처리해야 됨  
- api 서버 
: 비지니스 로직단에 권한 처리 -> 각각의 메세지에 파싱하기에 권한인가 로직을 구현하기 쉬움 
: api gateway가 accesstoken을 권한인가에 필요한 id, role로 변환해서 api에 전달해주면 필요한 사용자정보등을 별도로 데이터베이스를 뒤지지 않고 header내용만을 api권한 인가로 처리 가능 

1.3 restapi사용할때 same origin policy도 처리해야 됨 
proxy -> 각각의 사이트에 reverse proxy를 넣고 주소 세팅 -> 서로 호출하는 사이트가 자사서비스용일 경우 가능 
cors -> 모든소스에서 들어오는 api호출 허용 

1.4 순서 
api token request with userid, password 
-> token서버에서 accesstoken 발급하고 token store(redis, memcached)에 저장  
-> token은 secure cookie와 http only 옵션을 통해 브라우저로 전달 
-> api 호출할때 자동적으로 전송됨

offline_type : access_token, refresh_token

사용자가 로그인하면 client server가 resource server에 승인가능한지 호출
code return해주면 code, client id/password 로 다시 resource server호출
resource server에서 access_token 발급 
만료됐을때 

1.4 token type 
토큰자체만
JWT : 토큰에 권한까지

------------------------------------

2. Spring security 

2.1 회원가입시 고려할 부분  
Spring security가 지원해주는 password encoder만 사용해서 password저장 

2.2 로그인/로그아웃 커스터마이징

Principal < Authentication < SecurityContext < SecurityContextHolder(ThreadLocal사용)
ThreadLocal : 파라미터를 전달하지 않아도 한 thread내에서 공유가능 
Principal을 Authentication에 담아서 관리, thread가 달라지면 authentication이 달라짐

Authenication : Principal(UserDetailService에서 return한 객체), GrantAuthority
UserDetailService는 user정보를 제공하는 역할, 인증은 AuthenticationManager에서 인증 

AuthenticationManager -> 인증을 담당
SecurityContextHolder -> 인증정보 담고 있음 

http header나 인증정보를 가져와서 context에 넣는 filter 필요 !!

요청을 servlet container가 받음 
servlet container에는 fileter가 존재 

DelegatingFilterProxy
서블릿필터 
서블릿 필터처리를 스프링에 들어있는 빈으로 위임하고싶을때 사용하는 서블릿 필터

AuthenticationManager가 인증을 하고 SpringSecutiryContextHoler가 인증 넣어주고 
넣어줄때 SecurityContextPersistenceFilter, UsernamePasswordAuthenticationFilter 사용 
이 부분을 FilterChainProxy가 호출 FilterChainProxy를 DelegatingFilterProxy가 호출
SpringSecutiryContextHoler가 인증을 가지고있고

권한 
User > 
Role에 추가
Hierachy 설정 가능 

accessDecisionManager를 따로 지정하지 않는다면
AffirmativeBases()기본으로 사용 

FilterSecurityInterceptor : 마지막 필터 

ExceptionTranslationFilter에서 처리하는 exception 
AuthenticationException
- AccessDeniedException
anonymous : entrypoint로 이동
not anonymous : AccessDeniedHandler에 위임

servlet container에 요청이 들어오면
servlet filter중에 deligatingfilterproxy가 filterchainproxy로 위임
filter들은 websecurity, httpsecurity를 통해 설정됨(설정에 따라 사용filter달라짐)



● 동적 리소스는 http.authorizeRequests()에서 처리 권장
● 정적 리소스는 WebSecurity.ignore()를 권장
  예외적인 정적 자원(인증이 필요한 정적자원이 있는 경우)는 http.authorizeRequests()를 사용

동일 thread내에서만 security동일 

1. WebAsyncManagerIntegrationFilter : 스프링 MVC의 Async 기능(핸들러에서 'Callable'을 리턴할 수 있는 기능)을 사용할 때에도 SecurityContext를 공유하도록 도와주는 필터

Async : Thread를 다르게 사용
Thread를 다르게 사용하면 security도 달라지나 
SecurityContextHolder.setStrategyName(SecurityContextHolder.MODE_INHERITABLETHREADLOCAL); 사용하면 Async 다른 thread에서도 security 상속됨

2. SecurityContextPersistenceFilter
SecurityContextRepository를 사용해서 Http session에서 기존의 SecurityContext를 읽어오거나 초기화 시킴
● Spring-Session과 연동하여 세션 클러스터를 구현가능

3 . HeaderWriterFilter
1) XContentTypeOptionsHeaderWriter : 마임타입 스니핑방지, Content-Type으로만 rendering해줌
2) XXssProtectionHeaderWriter: 브라우저에 내장된 XSS 필터 적용.
3) CacheControlHeadersWriter: 캐시 히스토리 취약점 방어.
4) HstsHeaderWriter: HTTPS로만 소통하도록 강제.
5) XFrameOptionsHeaderWriter: clickjacking 방어.

4. CsrfFilter (Cross-site request forgery)
: 도메인이 다를경우 호출 허용하지 않음
: csrf토큰을 사용하여 방지 
: form tag사용할경우 csrf 자동으로 넣 

cross-origin resource sharing : 도메인이 다르더라도 호출 허용



* HTTP Basic인증 (formlogin말고 header에 전송)

* 뷰에서 인증정보 사용하는 방법 

* OAuth2, 메소드 시큐리티 등 .. 

------------------------------------

3. Restapi

Web
JPA
HATEOAS(헤이티오스)
REST Docs
H2
PostgreSQL
Lombok

restlet(툴 : postman)

3.1 lombok관련 
- @Entity에 @Data를 쓰는 경우 자동 EqualsAndHashCode가 모든 프로퍼티에 걸리기 때문에 스택오버플로우 걸릴 수 있음 
- @EqualsAndHashCode(of="id")로도 가능 
- @Lombok 관련 annotation -> customizing해서 쓸수 없음 

3.2 설정
spring.jackson.deserialization.fail-on-unknown-properties=true -> json에서 object할때 없는 필드값 들어오면 에러 

3.3 결과처리
- ResponseEntity처리
- serialize : object->json
  deserialize : json -> object
- response type -> 201, bad request 처리 

------------------------------------

4. git 반영 

1) cd 닉네임.github.io
2) git status
 - 빨간색 : 커밋전 
3) git add *
 - 커밋가능하게 
4) git status
- 초록 : 커밋가능
5) git commit -m "커밋메시지"
- 커밋 및 메세지 입력
6) git status
7) git remote -v
- 현재 저장소 확인 
8) git push
- 깃허브 저장소에 반영 