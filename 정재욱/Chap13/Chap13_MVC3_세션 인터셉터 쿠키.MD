## MVC3 : 세션, 인터셉터, 쿠키
### 컨트롤러에서 HttpSession 사용하기
로그인 상태를 유지하는 방법은 크게 HttpSession을 이용하는 방법과 쿠키를 이용하는 방법이 있다.

> 컨트롤러에서 HttpSession을 사용하려면 다음의 2가지 방법 중 한 가지를 사용하면 된다.

 - 요청 매핑 어노테이션 적용 메서드에 HttpSession 파라미터를 추가한다.
 - 요청 매핑 어노테이션 적용 메서드에 HttpServletRequest 파라미터를 추가하고 HttpServletRequest를 이용해서 HttpSession을 구한다.

> 첫번째 방법을 사용한 예시 코드 1
```java
@PostMapping
public String form(LoginCommand loginCommand, Errors errors, HttpSession session) {
    //session을 사용하는 코드
        ...
        }
```
 - 요청 매핑 어노테이션 적용 메서드에 HttpSession 파라미터가 존재할 경우 스프링 MVC는 컨트롤러의 메서드를 호출할 때 HttpSession 객체를 파라미터로 전달한다.
 - HttpSession을 생성하기 전이면 새로운 HttpSession을 생성하고 그렇지 않으면 기존에 존재하는 HttpSession을 전달한다.

> 두번째 방법을 사용한 예시 코드 2
```java
@PostMapping
public String submit(LoginCommand loginCommand, Errors errors, HttpServletRequest req) {
    HttpSession session = req.getSession();
        ...
        }
```
 - 첫 번째 방법은 항상 HttpSession을 생성하지만 두 번째 방법은 필요한 시점에만 HttpSession을 생성할 수 있다.

```java
package controller;

import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.springframework.stereotype.Controller;
import org.springframework.validation.Errors;
import org.springframework.web.bind.annotation.CookieValue;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;

import spring.AuthInfo;
import spring.AuthService;
import spring.WrongIdPasswordException;

@Controller
@RequestMapping("/login")
public class LoginController {
    
    ...

    @PostMapping
    public String submit(
    		LoginCommand loginCommand, Errors errors, HttpSession session,
    		HttpServletResponse response) {
        new LoginCommandValidator().validate(loginCommand, errors);
        if (errors.hasErrors()) {
            return "login/loginForm";
        }
        try {
            AuthInfo authInfo = authService.authenticate(
                    loginCommand.getEmail(),
                    loginCommand.getPassword());
            
            session.setAttribute("authInfo", authInfo);//인증 정보 객체 저장

            return "login/loginSuccess";
        } catch (IdPasswordNotMatchingException e) {
            errors.reject("idPasswordNotMatching");
            return "login/loginForm";
        }
    }
}
```
 - HttpSession의 "autoInfo" 속성에 인증 정보 객체를 저장하도록 한다.

```java
package controller;

import javax.servlet.http.HttpSession;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class LogoutController {

	@RequestMapping("/logout")
	public String logout(HttpSession session) {
		session.invalidate();
		return "redirect:/main";
	}

}
```
 - HttpSession을 제거한다. session.invalidate(); 명령어를 이용해서 세션 제거하면서 로그아웃 처리

```html
<%@ page contentType="text/html; charset=utf-8" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<!DOCTYPE html>
<html>
<head>
    <title>메인</title>
</head>
<body>
    <c:if test="${empty authInfo}">
    <p>환영합니다.</p>
    <p>
        <a href="<c:url value="/register/step1" />">[회원 가입하기]</a>
        <a href="<c:url value="/login" />">[로그인]</a>
    </p>
    </c:if>
    
    <c:if test="${! empty authInfo}">
    <p>${authInfo.name}님, 환영합니다.</p>
    <p>
        <a href="<c:url value="/edit/changePassword" />">[비밀번호 변경]</a>
        <a href="<c:url value="/logout" />">[로그아웃]</a>
    </p>
    </c:if>
</body>
</html>
```
 - 세션에서 저장한 authInfo 속성에 인증 정보 객체를 가져와서 확인한다.

### 비밀번호 변경 기능 구현
> 서버를 재기동 하면 HttpSession에 담긴 정보는 초기화 된다.
 
### 인터셉터 사용하기
> 로그인하지 않은 상태에서 GetMapping이기 때문에 url로 접근할 수 있다.
> 
> 이러한 오류를 막아주기 위해 HttpSession에 authoInfo 객체가 존재하는지 검사하고, 존재하지 않으면 로그인 경로로 리다이렉트 처리한다.

#### HandlerInterceptor 인터페이스 구현하기
HandlerInterceptor 인터페이스를 사용해서 세가지 시점에 공통 기능을 추가할 수 있다.
 - 컨트롤러(핸들러) 실행 전
 - 컨트롤러(핸들러) 실행 후, 아직 뷰를 실행하기 전
 - 뷰를 실행한 이후

> 세 시점을 처리하기 위한 HandlerInterceptor 인터페이스는 다음의 메서드를 정의하고 있다.

> 컨트롤러(핸들러) 실행 전

```java
boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;
```
 - handler 파라미터는 웹 요청을 처리할 컨트롤러(핸들러) 객체이다.
 - 로그인하지 않은 경우 컨트롤러 실행하지 않음
 - 컨트롤러 실행하기 전에 컨트롤러에서 필요로 하는 정보를 생성
 - boolean 타입으로 리턴하기 때문에, false를 리턴하면 컨트롤러(또는 다음 HandlerInterceptor)를 실행하지 않는다.

> 컨트롤러(핸들러) 실행 후, 아직 뷰를 실행하기 전

```java
void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception;
```
 - 컨트롤러가 정상적으로 실행된 이후에 추가 기능을 구현할 때 사용한다.
 - 컨트롤러가 익셉션을 발생하면 postHandle() 메서드는 실행하지 않는다.

> 뷰를 실행한 이후

```java
void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throw Exception;
```
 - 뷰가 클라이언트에 응답을 전송한 뒤에 실행된다.
 - 컨트롤러 실행 과정에서 익셉션이 발생하면 이 메서드의 네 번째 파라미터로 전달된다.
 - 익셉션이 발생하지 않으면, 네 번째 파라미터는 null이 된다.
 - 컨트롤러 실행 이후에 예기치 않게 발생한 익셉션을 로그로 남긴다거나 실행 시간을 기록하는 등의 후처리를 하기에 적합한 메서드이다.

> HandlerInterceptor와 컨트롤러의 실행 흐름 그림

![image](https://user-images.githubusercontent.com/55322459/183447672-81e0f826-43fa-4f7e-b660-605c4eebf63c.png)

AOP, Filter, Interceptor  적용 참고 예시
![image](https://user-images.githubusercontent.com/55322459/183447765-3599a405-9d69-4da1-82de-4059ed853151.png)

> preHandle 메서드를 이용한 로그인 하지않았지만 GetMapping으로 넘어온 경우 예외처리

```java
package interceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.springframework.web.servlet.HandlerInterceptor;

public class AuthCheckInterceptor implements HandlerInterceptor {

	@Override
	public boolean preHandle(
			HttpServletRequest request,
			HttpServletResponse response,
			Object handler) throws Exception {
		HttpSession session = request.getSession(false);
		if (session != null) {
			Object authInfo = session.getAttribute("authInfo");
			if (authInfo != null) {
				return true;
			}
		}
		response.sendRedirect(request.getContextPath() + "/login");
		return false;
	}

}
```
 - preHandle() 메서드는 HttpSession에 "authoInfo" 속성이 존재하면 true를 리턴한다.
 - 존재하지 않으면 리다리엑트 응답을 생성한뒤 false를 리턴한다.
 - request.getContextPath()는 현재 컨텍스트 경로를 리턴한다. -> 여기서 현재 컨텍스트 경로는 http://localhost:8080/sp5-chap13이라면 컨텍스트 경로는 sp5-chap13이 된다.

#### HandlerInterceptor 설정하기
HandlerInterceptor를 구현하면 HandlerInterceptor를 어디에 적용할지 설정해야 한다.
MvcConfig 설정 클래스에 추가한다.

```java
@Override
	public void addInterceptors(InterceptorRegistry registry) {
		registry.addInterceptor(authCheckInterceptor())
			.addPathPatterns("/edit/**")
            .excludePathPatterns("/edit/help/**");
	}

@Bean
public AuthCheckInterceptor authCheckInterceptor() {
        return new AuthCheckInterceptor();
        }
```
 - addInterceptors메서드는 인터셉터를 설정하는 메서드이다.
 - InterceptorRegistry라는 클래스의 객체를 이용해서 authCheckIntercepotr()를 추가한다.
 - InterceptorRegistry의 addInterceptor는 InterceptorRegistration 객체를 리턴하는데, 이 객체의 addPathPatterns() 메서드는 인터셉터를 적용할 경로 패턴을 지정한다.
 - 2개 이상의 경로를 지정하려면 ,"콤마"를 이용해서 구분해서 지정한다.
 - 지정한 경로 패턴 중 일부를 제외하고 싶다면 excludePathPatterns() 메서드를 사용하면 된다.

> Ant 경로 패턴
> 
 - \* : 0개 또는 그 이상의 글자
 - ? : 1개 글자
 - \** : 0개 또는 그 이상의 폴더 경로

### 컨트롤러에서 쿠키 사용하기
쿠키를 이용해서 이메일 정보를 기억한다.
> 스프링 MVC에서 쿠키를 사용하는 방법 중 하나는 @CookieValue 어노테이션을 사용하는 것이다.
> 
> @CookieValue 어노테이션은 요청 매핑 어노테이션 적용 메서드의 Cookie 타입 파라미터에 적용한다.
> 
> 이를 통해 쉽게 쿠키 Cookie 파라미터로 전달 받을수 있다.

```java
@GetMapping//@CookieValue를 이용해서 쿠키를 전달받도록 수정
    public String form(LoginCommand loginCommand,
    		@CookieValue(value = "REMEMBER", required = false) Cookie rCookie) {
		if (rCookie != null) {
			loginCommand.setEmail(rCookie.getValue());
			loginCommand.setRememberEmail(true);
		}
    	return "login/loginForm";
    }
```
 - value 속성은 쿠키의 이름을 지정한다.
 - REMEMBER인 쿠키를 Cookie타입으로 전달 받는다.
 - 지정한 이름을 가진 쿠키가 존재하지 않을 수도 있다면 required 속성값을 false로 지정한다.
 - 디폴트값은 required true이며, 쿠키가 존재하지 않는다면 익셉션을 발생 시킨다.

```java
@PostMapping
    public String submit(//쿠키를 생성하도록 수정
    		LoginCommand loginCommand, Errors errors, HttpSession session,
    		HttpServletResponse response) {
        new LoginCommandValidator().validate(loginCommand, errors);
        if (errors.hasErrors()) {
            return "login/loginForm";
        }
        try {
            AuthInfo authInfo = authService.authenticate(
                    loginCommand.getEmail(),
                    loginCommand.getPassword());
            
            session.setAttribute("authInfo", authInfo);

			Cookie rememberCookie = 
					new Cookie("REMEMBER", loginCommand.getEmail());
			rememberCookie.setPath("/");
			if (loginCommand.isRememberEmail()) {//이메일을 기억하기 여부에 따라 쿠키에 30일 저장
				rememberCookie.setMaxAge(60 * 60 * 24 * 30);
			} else {
				rememberCookie.setMaxAge(0);//혹은 바로 삭제 한다.
			}
			response.addCookie(rememberCookie);//쿠키 생성을 위해 response 객체가 필요하다.

            return "login/loginSuccess";
        } catch (WrongIdPasswordException e) {
            errors.reject("idPasswordNotMatching");
            return "login/loginForm";
        }
    }
```
 - 쿠키를 생성하는 부분은 로그인을 처리하는 submit() 메서드 이다.
 - 쿠키를 생성하기 위해서는 HttpServletResponse 객체가 필요하다.