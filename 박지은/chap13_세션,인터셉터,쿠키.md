# MVC3 : 세션, 인터셉터, 쿠키

<BR>

## 🔎 HttpSession

: 클라이언트와 웹 브라우저와 웹 서버의 연결 상태를 관리하는 객체 

- 웹 서버에 상태를 유지하기 위한 정보 저장
- 웹 서버에 저장되는 쿠키(세션 쿠키)
- 서버에서 세션을 삭제했을때만 삭제가 됨(쿠키보다 비교적 보안 좋음)
- 각 클라이언트에게 고유 session id 부여

→ HttpSession이용해 로그인 상태를 유지함

<br><Br>

### HttpSession 사용방법

1. 요청 매핑 메소드에 파라미터로 HttpSession 전달

: 요청 매핑 어노테이션 적용 메소드에 HttpSession 파라미터가 존재할 경우, 스프링 MVC는 컨트롤러의 메소드 호출시 HttpSession 객체를 파라미터로 전달함

```
@PostMapping
public String form(LoginCommand loginCommand, Errors errors, HttpSession session){
    ... // session 사용 코드
}
```

: 항상 HttpSession 생성

<br>

2. HttpServletRequest의 getSession() 메소드 이용

```
@PostMapping
public String form(LoginCommand loginCommand, Errors errors, HttpServletRequest req){
    HttpSession session = req.getSession();
    ...
}
```

: 필요한 시점에만 HttpSession을 생성할 수 있음

<br>

➕ 서버를 재시작하면 세션정보가 유지되지 않으므로 세션에 보관된 "authInfo" 객체 정보 사라짐 = 로그인부터 다시 해야함

<br><Br>

### 세션 사용

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

<br><Br>

## 인터셉터

:컨트롤러 코드마다 세션 확인 코드를 삽입 = 많은 중복 코드 생성

➡️ **HandlerInterceptor**: 컨트롤러에 대해 동일한 기능을 적용해야 할때 사용 

<br>

### 🔎 인터셉터 메소드

- 컨트롤러(핸들러) 실행 전 : preHandle()

    preHandle() 메소드가 false를 리턴하면 컨트롤러를 실행하지 않음.


- 컨트롤러(핸들러) 실행 후, 아직 뷰를 실행하기 전 : postHandle()

    컨트롤러가 정상적으로 실행된 후 추가기능 구현시 사용.
    컨트롤러가 익셉션을 발생하면 postHandle() 메소드는 실행 x

- 뷰 실행 후 : afterCompletion()

    컨트롤러 실행과정에서 익셉션이 발생하면 afterCompletion()의 네번째 파라미터로 전달됨. 이 익셉션을 로그로 남기거나 실행시간을 기록하는 등 후처리에 적합한 메소드


<br>

➕ HandlerInterceptor 인터페이스를 상속받고 필요한 메소드만 재정의!

```
public class AuthCheckInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        HttpSession session = request.getSession(false);
        if(session!=null){
            Object authInfo = session.getAttribute("authInfo");
            if(authInfo!=null){
                return true;
            }
        }

        response.sendRedirect(request.getContextPath()+"/login");
        return false;
    }
}
```
: 로그인 여부에 따라 로그인 폼으로 보내거나 컨트롤러 실행하도록 구현 

= HttpSession에 "authoInfo" 속성이 존재하지 않으면 지정한 경로로 리다이렉트!

### 🔎 HandlerInterceptor 설정

    @Configuration
    @EnableWebMvc
    public class MvcConfig implements WebMvcConfigurer {

        ... 

        @Override // 인터셉터 설정 메소드
        public void addInterceptors(InterceptorRegistry registry) {
            registry.addInterceptor(authCheckInterceptor()) // authCheckInterceptor를 인터셉터로 설정
                    .addPathPatterns("/edit/**"); //이 경로 들어가면 메소드 실행 됨
        }



<br><Br>


## 쿠키

: 브라우저를 통해 클라이언트에 저장되는 사용자 정보

### 🔎 스프링에서 쿠키를 사용하는 방법 : @CookieValue

: 요청 매핑 어노테이션 적용 메소드의 Cookie 타입 파라미터에 @CookieValue 적용

= 쿠키를 Cookie 파라미터로 전달 받음

```
@GetMapping
    public String form(LoginCommand loginCommand, @CookieValue(value="REMEMBER",required = false) Cookie rCookie){
        if(rCookie!=null){ // 쿠키 존재하면
            loginCommand.setEmail(rCookie.getValue()); // 쿠키 값을 email로 셋팅
            loginCommand.setRememberEmail(true);
        }
        return "login/loginForm";
    }
```

: REMEMBER 쿠키가 존재하면 쿠키의 값을 읽어와 커맨드 객체의 email 프로퍼티 값을 설정함

커맨드 객체를 사용해 폼을 출력하므로 REMEMBER 쿠키 존재하면 입력 폼의 email 프로퍼티에 쿠키값이 채워져 출력됨

<br>

➕ 실제 쿠키를 생성하는 부분 : 로그인 처리하는 메소드

쿠키를 생성하려면 HttpServletResponse 객체 필요함

```
@PostMapping
    public String submit(LoginCommand loginCommand, Errors errors, HttpSession session, HttpServletResponse response){
        ...

            session.setAttribute("authInfo", authInfo);

            Cookie rememberCookie= new Cookie("REMEMBER",loginCommand.getEmail());
            rememberCookie.setPath("/");
            if(loginCommand.isRememberEmail()){ //이메일 기억하기 선택하면?
                rememberCookie.setMaxAge(60*60*24*30); // 30일동안 쿠키 유지
            }else{
                rememberCookie.setMaxAge(0); // 기억 클릭x면 바로 삭제
            }
            response.addCookie(rememberCookie);

            return "login/loginSuccess";

        ...
    }
```