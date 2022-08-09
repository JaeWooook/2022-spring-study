# 세션(Session) & 쿠키(Cookie)

## HTTP 프로토콜

- 무상태 프로토콜(Stateless) : 통신이 끝나면 상태를 유지하지 않음

    클라이언트의 요청과 그에 따른 서버의 응답이 끝나면 통신이 끝나며 해당 상태 정보는 유지되지 않음


- 비연결성(Connetionless) : 클라이언트가 요청을 한 후 응답을 받으면 그 연결을 끊어버림

    비연결성을 가지는 HTTP에선 요청을 주고받을때만 연결을 유지하고 응답을 주고나면 TCP/IP 연결을 끊음


➡️ HTTP의 기본 특성인 ```비연결성```과 ```무상태성```때문에, 서버는 클라이언트를 기억하지 못함

BUT 서버가 클라이언트를 기억(연결 유지)하고 싶다면?

<Br>

## 쿠키

: 클라이언트가 웹 사이트에 방문할 경우, 그 사이트가 사용하는 서버에서 **사용자의 컴퓨터**에 저장하는 클라이언트 기록 정보 파일.

(클라이언트 로컬에 저장했다가 필요시 정보를 참조/재사용)

<br>

### 🔎 SPRING 쿠키 사용

쿠키 값을 사용해야하는 메소드 내부에 @CookieValue("key") 어노테이션을 붙인 변수를 파라미터로 넘기면 됨

    @GetMapping
    public String form(LoginCommand loginCommand, @CookieValue(value="cookie_key",required = false) Cookie rCookie){
        if(rCookie!=null){
            loginCommand.setEmail(rCookie.getValue());
            loginCommand.setRememberEmail(true);
        }
        return "login/loginForm";
    }

: 스프링이 알아서 해당 키 값을 가진 쿠키 값을 변수에 할당함

<bR>

### 🔎 쿠키 특징

- 클라이언트의 상태 정보를 로컬에 저장했다가 참조
- 쿠키는 사용자가 따로 요청하지 않아도 브라우저가 request하면 request header를 넣어 자동으로 서버에 전송
- 클라이언트에 총 300개의 쿠키 저장 가능
- 이름, 값, 만료일(저장기간), 경로 정보로 구성됨
    - 쿠키는 파일로 저장되므로 브라우저를 종료해도 정보가 유지됨. (세션은 삭제)


<br><BR>

## 세션

: 방문자가 웹 서버에 접속해 있는 상태를 하나의 단위로 보고 이를 세션이라 함

쿠키를 기반으로 하지만, 사용자 정보를 브라우저에 저장하는 쿠키와 달리 세션은 서버측에서 관리

<Br>

### 🔎 SPRING 세션 사용

1. 요청 매핑 메소드에 파라미터로 HttpSession 전달 

2. HttpServletRequest의 getSession() 메소드 이용

3. @Autowired로 HttpSession 주입

    ```
    @Controller
    public class Controller {
        @Autowired
        private HttpSession session;
        
        ...
    }
    ```

<br>


```
// 생성 (위 2번 방법)
HttpSession session = request.getSession();
HttpSession session = request.getSession(false);

// 값 저장
session.setAttribute(String name, Object value);
➡️ session.setAttribute("authInfo", authInfo); // HttpSession의 "authInfo"속성에 인증정보 객체 저장

// 값 얻기
Object obj = session.getAttribute(String name);

// 값 제거
session.removeAttribute(String name);// 특정 이름의 속성 제거
session.invalidate(); // binding 되어 있는 모든 속성 제거

```

<br>

### 🔎 세션의 특징

- 웹 서버에 웹 컨테이너의 상태를 유지하기 위한 정보를 저장
- 각 클라이언트에게 고유 ID 부여(웹 브라우저가 서버에 접속해서 브라우저를 종료할때까지 인증상태 유지)

    - 세션 ID : 클라이언트가 요청를 보내면, 해당 서버의 엔진이 클라이언트에게 부여하는 유일한 ID

- 세션 ID로 클라이언트를 구분해 클라이언트의 요구에 맞는 서비스 제공
- 보안면에서 쿠키보다 우수 (브라우저를 닫거나, 서버에서 세션을 삭제했을 때만 삭제가 됨)
    - 쿠키는 로컬에 저장되므로 변질되거나 request에서 스니핑 당할 우려o
    - 세션은 쿠키를 이용해 session ID만 저장하고 그것으로 부분하여 서버에서 처리하므로 쿠키보다 보안 굿

- 사용자가 많아질수록 서버 메모리를 많이 차지함
- 세션도 만료기간 정할 수 있지만, 브라우저가 종료되면 만료기간에 상관없이 삭제됨

<br><Br>

## 쿠키와 세션의 차이

![image](https://user-images.githubusercontent.com/81572478/183613598-fda47733-4480-4684-b168-65a6cec8ca51.png)


📌 쿠키를 사용하는 이유

세션이 쿠키에 비해 보안이 좋지만, 세션은 서버에 저장되고 서버의 자원을 사용함. 서버 자원엔 한계가 있고 속도가 느려질 수 있으니 쿠키와 세션을 적절하게 병행 사용!