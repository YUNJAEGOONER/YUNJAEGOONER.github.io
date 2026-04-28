---
title: 와이제이 쇼핑몰 리뉴얼1
description: >-
  카테캠 2단계를 통해 구현한 쇼핑몰을 리뉴얼 해보자!!!
author: yunjae
date: 2026-01-26 21:35:00 +0800
categories: [backend, jwt]
tags: [getting started]
pin: true
---

카테캠 2단계 미션은 쇼핑몰 클론 코딩을 수행했다. 당시 심신미약의 상태 ~~CSR과 SSR의 차이도 이해하지 못했으며 어디까지가 프론트엔드의 역할인지 백엔드 역할인지도 몰랐음~~로 진행했기에 코드가 매우 엉망이다. 지금도 완벽하게 모든걸 이해한 상태는 아니지만 예전보다는 많은 지식을 쌓았음으로 시간이 나는대로 와이제이 쇼핑몰을 개선해 보려 한다.

### 토큰 유효시간 설정하기 및 RefreshToken 사용하기
미친 수준의 보안을 자랑하고 있었다. 엑세스 토큰만을 발급해주고 있었으며, 엑세스 토큰의 유효 시간 만료 시, 엑세스 토큰의 재발급을 위해 사용되는 RefreshToken은 사용하고 있지도 않았다. JWT가 무엇인지, AccessToken과 RefreshToken은 무슨 역할을 하는지 살펴보도록 하겠다.

**JWT**
- JSON Web Token (JavaScript Object Notation)
- 헤더.페이로드.시그니처 구조의 문자열
- Header(알고리즘 정보) + Payload(서버-클라이언트가 주고 받을 정보) + Signature(유효성 검증)
- 토큰은 클라이언트에 저장( <-> 세션 방식의 경우 세션 정보를 모두 서버가 관리)되므로 *서버가 클라이언트의 토큰을 제어할 수 없다*

**Access Token**
- 서버는 로그인 시, 사용자의 인증 정보(아이디, pw)가 확인되면 Access Token을 발급해준다. (+ Refresh Token도 함께)
- 클라이언트를 요청을 보낼때 마다 사용자 인증을 위해 토큰을 함께 보낸다.
- 서버는 클라이언트에게 받은 토큰으로 사용자의 식별 정보를 추출한다.
- Access Token의 경우, *탈취될 위험*이 있으므로 만료시간을 짧게 설정한다.

**Refresh Token**
- Refresh을 위한 토큰 뭐를? AccessToken을...(즉, Access Token을 재발급하기 위한 토큰)
- AccessToken이 만료된 경우, RefreshToken을 기반으로 AccessToken을 재발급한다.
- Refresh Token은 Access Token보다 긴 유효기간을 갖는다.
- RefreshToken을 통해 클라이언트를 확인(클라이언트가 보낸 refresh token과 DB에 저장되어 있는 Refresh Token의 일치하는지)해야함으로 RefreshToken은 데이터베이스에 저장되어 있어야 한다.
- 서버의 응답을 통해 Access Token이 만료되었음을 알게 되면 프론트엔드에서 Refresh Token과 Access Token을 함께 서버로 전달해 Access Token 재발급을 요청한다.

### 필터 개선하기

필터에 대한 단단한 오해가 있었던거 같다.필터는 Spring 외부에서 동작하는것으로(서블릿이 지원하는 기능으로 Dispatcher Servlet에 요청이 전달 되기 filter의 `dofilter` 메서드가 실행)로그인 여부를 확인하는 역할을 수행해야하는데, 현재 내 필터는 로그인 로직까지 담당하고 있었다. ~~고생 많았다. 너의 책임을 덜어주마~~

**~~나의 미친 필터~~**
```java
//로그인 요청에 대해서만 동작하는 필터임...
@Override
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
        throws IOException, ServletException {

    HttpServletRequest httpServletRequest = (HttpServletRequest) request;
    HttpServletResponse httpServletResponse = (HttpServletResponse) response;

    String url = httpServletRequest.getRequestURI();
    String Method = httpServletRequest.getMethod();

    //view로 로그인 요청이 들어오는 경우
    if(url.equals("/view/login") && Method.equals("POST")){

        log.info("로그인 필터 is working,,,,");

        String email = request.getParameter("email");
        String password = request.getParameter("password");

        //인증실패예외를 반환하고 이를 Http Response로 렌더링하는 작업이 필요할것 같아요.
        if(email.isBlank()|| password.isBlank()){
            throw new LoginError(ErrorCode.EMAIL_PASSWORD_REQUIRED);
        }

        Member member = memberService.checkMember(email, password);
        String token = jwtAuthService.createJwt(email, member.getMemberId(), member.getRole());

        //쿠키 발행 (쿠키에 토큰을 저장)
        Cookie tcookie = new Cookie("token", token);
        tcookie.setPath("/");
        tcookie.setHttpOnly(true);
        httpServletResponse.addCookie(tcookie);

        request.setAttribute("token", token);
        request.setAttribute("role", member.getRole());
        log.info("토큰 생성 완료");
    }

    //다음 필터가 있다면 동작해라;
    chain.doFilter(request, response);
}
```

**(서블릿) 필터란?**
- 서블릿이 지원하는 기능이다.
- 서블릿(Dispatcher Servlet)이 호출 되기 이전에 호출된다.
  - HTTP 요청이 들어오면 `doFilter()`가 호출된다.
  - 여러개의 filter가 연속적으로 동작하도록 체이닝 할 수 있다.
  - 모든 요청에 대한 로그를 남길 수 있다. (Servlet이 호출 되기 전에 항상 호출됨 + 필터간의 우선 순위 설정 가능)
- **서블릿 컨테이너**는 필터를 싱글턴으로 생성하고 관리한다.
- 로그인 여부는 여러 컨트롤러에서 갖는 공통 관심사이기 때문에 (서블릿)필터 또는 인터셉터를 통해 처리할 수 있다.

**필터에서는 로그인 여부 확인만 수행**

```
@Override
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {

    HttpServletRequest httpServletRequest = (HttpServletRequest) request;
    HttpServletResponse httpServletResponse = (HttpServletResponse) response;

    String requestURI = httpServletRequest.getRequestURI();

    try{
        log.info("인증 체크 필터 시작 {}", requestURI);
        if(!isWhiteList(requestURI)){
            log.info("인증 체크 로직 실행 {}", requestURI);
            String token = authUtil.extractToken(httpServletRequest);
            if(token == null){
                log.info("[로그인 페이지로 이동]미인증 사용자 요청");
                httpServletResponse.sendRedirect("/view/login/form?redirectURL=" + requestURI);
                return;
            }
            log.info("토큰의 유효성 검사");
            jwtUtil.checkValidation(token);
        }
        chain.doFilter(request, response);  // 다음 필터가 있다면 동작해라;

    }catch (Exception e){
        throw e; //예외를 톰캣까지 ~~
    }
    finally {
        log.info("loginCheckFilter 종료");
    }
}
```

**로그인은 컨트롤러에서 처리**

```java
@PostMapping("/login")
public ResponseEntity<Void> login(
        @RequestBody @Valid MemberRequestDto memberRequestDto
){
    MemberResponseDto member = memberService.login(memberRequestDto.email(), memberRequestDto.password());
    jwtUtil.createAccessToken(member.id(), member.role()); // 엑세스 토큰 발급 
    refreshTokenService.issueRefreshToken(member.id()); // 리프레시 토큰 발급
    return ResponseEntity.ok().build();
}
```