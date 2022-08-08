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
