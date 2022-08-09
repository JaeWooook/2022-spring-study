# Chap13 MVC3
## 로그인 기능 구현
로그인 성공 후 인증 상태 정보를 세션에 보관할 때 사용할 AuthInfo 클래스이다. 
```java
public class AuthInfo{
	private Long id;
    private String email;
    private String name;
   	...
}
```
암호 일치 여부를 확인하기 위한 matchPassword() 메서드를 Member 클래스에 추가한다.
```java
public class Member{
	private Long id;
    private String email;
    private String password;
    private String name;
    private LocalDateTime registerDateTime;
    ...
    public boolean matchPassword(String password){
    	return this.password.equals(password);
    }
}
```
이메일과 비밀번호가 일치하는지 확인해서 AuthInfo 객체를 생성하는 AuthService 클래스이다.
```java
public class AuthService{
	private MemberDao memberDao;
   	...
    public AuthInfo authenticate(String email, String password){
  		Member member = memberDao.selectbyEmail(email);
        if(member = null){
        	throw new WrongIdPasswordException();
        }
        if(!member.matchPassword(password)){
        	throw new WrongIdPasswordException();
        }
        return new AuthInfo(member.getId(), member.getEmail(),member.getName());
    }	
}
```
AuthService를 이용해서 로그인을 처리하는 LoginController와 폼에 입력한 값을 전달받기 위한 LoginCommand클래스와 입력된 값을 검증하는 LoginCommandValidator가 있다.
```java
public class LoginCommand{
	private String email;
    private String password;
    private boolean rememberEmail;
    ...
}
```
```java
public class LoginCommandValidator implements Validator{
	@Override
    public boolean supports(Class<?> clazz){
    	return LoginCommand.class.isAssignableFrom(clazz);
    }
    
    @Override
    public void validate(Object target, Errors errors){
    	Validation Utils.rejectIfEmptyOrWhitespace(errors, "email", "required");
        ValidationUtils.rejectIfEmpty(errors, "password", "required");
    }
}
```
```java
@Controller
@RequestMapping("/login")
public class LonginController{
	private AuthService authService;
    
    public void setAuthService(AuthService authService){
    	this.authService=authService;
    }
    
    @GetMapping
    public String form(LoginCommand loginCommand){
    	new LoginCommandValidator().validate(loginCommand,errors);
        if(errors.hasErrors()){
        	return "login/loginForm";
        }
        try{
        	AuthInfo authInfo = authService.authenticate(
            	loginCommand.getEmail();
                loginCommand.getPassword());
           	//TODO 세션에 authInfo 저장해야 함
            return "login/loginSuccess";
        }catch(WrongIdPasswordException e){
        	errors.reject("idPasswordNotMatching");
            return "login/loginForm";
        }
       
    }
}
```
## 컨트롤러에서 HttpSession 사용하기
로그인 구현에서 바로 로그인 상태를 유지하는 것이 빠졌다. 로그인 상태를 유지하는 방법은 크게 HttpSession을 이용하는 방법과, 쿠키를 이용하는 방법이 있다. 

컨트롤러에서 HttpSession을 사용하면 다음의 두 가지 방법 중 하나를 사용하면 된다. 

1. 요청 매핑 어노테이션 적용 메서드에 HttpSession 파라미터 추가하기.
2. 요청 매핑 어노테이션 적용 메서드에 HttpServletRequest 파라미터를 추가하고 HttpServletRequest를 이용해서 HttpSession을 구하기.

### 첫번째 방법 
```java
@PostMapping
public String form(LoginCommand loginCommand, Errors errors, HttpSession session){
	//session을 사용하는 코드
}
```
@PostMapping을 붙인 메서드에 HttpSession 파라미터가 존재할 경우 스프링 MVC는 컨트롤러의 메서드를 호출할 때 HttpSession 객체를 파라미터로 전달한다. HttpSession을 생성하기 전이면 새로운 HttpSession를 생성하고 그렇지 않으면 기존에 존재하는 HttpSession을 전달한다.

### 두번째 방법
HttpServletRequest의 getSession() 메서드를 이용하는 것이다. 첫번째 방법은 항상 HttpSession을 생성하지만 두번째 방법은 필요한 시점에만 HttpSession을 생성할 수 있다. 
```java
@PostMapping
public String submit(
				LoginCommand loginCommand, 
				Errors errors,
                HttpServletRequest req
                		){
    HttpSession session = req.getSession();
	//session을 사용하는 코드
}
```
<br>

LoginController 코드에서 인증 후에 인증 정보를 세션에 담도록 submit() 메서드의 코드를 수정하자. 
```java
@Controller
@RequestMapping("/login")
public class LoginController{
	...
    @PostMapping
    public String submit(
    					LoginCommand loginCommand,
                        Errors errors,
                        HttpSession session
    						){
    	new LoginCommandValidator().validate(loginCommand,errors);
        if(errors.hasErrors()){
        	return "login/loginForm";
        }
        
        try{
        	AuthInfo authInfo=authService.authenticate(
            				loginCommand.getEamil(),
            				loginCommand.getPassword());
                    
            session.setAttribute("authInfo",authInfo);
                    
            return "login/loginSuccess";
        }
        catch(IdPasswordNotMatchingException e){
        	errors.reject("idPasswordNotMatching");
            return "login/loginForm";
        }
    }	
}	
```
로그인에 성공하면 "session.setAttribute("authInfo",authInfo)" 처럼 HttpSession의 "authInfo" 속성에 인증 정보 객체(autoInfo)를 저장하도록 코드를 추가했다. 

<br>

로그아웃을 위한 컨트롤러 클래스는 HttpSession을 제거하면 된다. 로그아웃 처리를 하는 LogoutController 코드를 보자.
```java
@Controller
public class LogoutController{
	@RequestMapping("/logout")
    public String logout(HttpSession session){
    	session.invalidate();
        return "redirect:/main";
    }
}
```
![](https://velog.velcdn.com/images/yh_lee/post/11e5ea65-c5b9-42cd-91ac-301e2c09ae56/image.png)LoginController는 로그인에 성공할 경우 HttpSession의 "authInfo" 속성에 인증 정보 객체를 저장한다. 따라서 로그인에 성공하면 17행 조건절이 true가 되어서 19~22행 내용이 출력된다. 

## 비밀번호 변경 기능 구현
비밀번호 변경 컨트롤러
```java
@Controller
@RequestMapping("/edit/changePassword")
public class ChangePwdController{
	private ChangePasswordService changePasswordService;
    ...
    @GetMapping
    public String form(
    			@ModelAttribute("command") ChangePasswordService pwdCmd
            ){
    	return "edit/changePwdForm";
    }	
    
    @PostMapping
    public String submit(
    	@ModelAttribute("command") ChangePasswordService pweCmd,
        Errors errors,
        HttpSession session
    				){
    	new ChangePwdCommandValidator().validate(pwdCmd,errors)l
        if(errors.hasError()){
        	return "edit/changePwdForm";
        }
        AuthInfo authInfo = (AuthInfo) session.getAttribute("authInfo");
        try{
        	changePasswordService.changePassword(
            		authInfo.getEmail(),
                    pwdCmd.getCurrentPassword(),
                    pwdCmd.getNewPassword());
        	return "edit/changedPwd";
        }catch(WrongIdPasswordException e){
        	errors.rejectValue("currentPassword","notMatching");
            return "edit/changePwdForm";
        }
    	
    }
}
```

## 인터셉터 사용하기 
로그인하지 않은 상태에서 http://localhost:8080/sp5-chap13/edit/changePassword를 입력하면 비밀번호 변경 폼이 출력된다. 로그인하지 않았는데 변경 폼이 출력되는 것은 이상하다. 그것보다 로그인하지 않은 상태에서 비밀번호 변경 폼을 요청하면 로그인 화면으로 이동시키는 것이 더 좋은 방법이다. 

이를 위해 HttpSession에 "authInfo" 객체가 존재하는지 검사하고 존재하지 않으면 **로그인 경로로 리다이렉트**하도록 ChangePwdController 클래스를 수정한다.
```java
@GetMapping
public String form(
					@ModelAttribute("command") ChangePwdCommand pwdCmd,
                    HttpSession session
					){
	AuthInfo authInfo = (AuthInfo) session.getAttribute("authInfo");
    if(authInfo==null){
    	return "redirect:/login";
    }
    return "edit/changePwdForm";
}
```
그런데 실제 웹 어플리케이션에서는 비밀번호 변경 기능 외에 더 많은 기능에 로그인 여부를 확인해야 한다. 각 기능을 구현한 컨트롤러 코드마다 세션 확인 코드를 삽입하는 것은 많은 중복을 일으킨다. 

이렇게 다수의 컨트롤러에 대해 동일한 기능을 적용해야 할 때 사용할 수 있는 것이 HandlerInterceptor이다.

### HandlerInterceptor 인터페이스 구현
세 시점에 공통 기능을 넣을 수 있다.

1. 컨트롤러(핸들러) 실행 전 
```java
boolean preHandle(HttpServletRequest request,HttpServletReponse response, Object handler) throws Exceptrion;
```
preHandle()은 컨트롤러(핸들러) 객체를 실행하기 전에 필요한 기능을 구현할 때 사용한다. handler 파라미터는 웹 요청을 처리할 컨트롤러(핸들러) 객체이다. preHandle()이 false를 리턴하면 컨트롤러(다음 HandlerInterceptor)를 실행하지 않는다. 이 메서드를 사용하면 다음 작업이 가능하다.
- 로그인 하지 않은 경우 컨트롤러를 실행하지 않음
- 컨트롤러를 실행하기 전에 컨트롤러에서 필요로 하는 정보 생성
2. 컨트롤러(핸들러) 실행 후, 아직 뷰를 실행하기 전 
```java
void postHandle(HttpServletRequest request,HttpServletReponse response, Object handler, ModelAndView modelAndView) throws Exceptrion;
```
postHandle()는 컨트롤러(핸들러)가 정상적으로 실행된 이후에 추가 기능을 구현할 때 사용한다. 컨트롤러가 익셉션을 발생하면 postHandle()은 실행하지 않는다. 
3. 뷰를 실행한 이후
```java
void afterCompletion(HttpServletRequest request,HttpServletReponse response, Object handler, Exception ex) throws Exceptrion;
```
afterCompletion()은 뷰가 클라이언트에 응답을 전송한 뒤에 실행된다. 컨트롤러 실행 과정에서 익셉션이 발생하면 이 메서드의 네번째 파라미터로 전달된다. 따라서 컨트롤러 실행 후 예기치 않게 발생한 익셉션을 로그로 남긴다거나 실행 시간을 기록하는 등의 후처리를 하기에 적합한 메서드다.![](https://velog.velcdn.com/images/yh_lee/post/55363a27-7145-4835-8a6c-15de6e709303/image.png)

비밀번호 변경 기능에 접근할 때 HandlerInterceptor를 사용하면 로그인 여부에 따라 로그인 폼으로 보내거나 컨트롤러를 실행하도록 구현할 수 있다. 
```java
public class AuthCheckInterceptor implements HandlerInterceptor{
	@Override
    public boolean preHandle(HttpServletRequest request,HttpServletReponse response, Object handler) throws Exceptrion){
    	HttpSession session = request.getSession(false);
        if(session!=null){
        	Object authInfo = session.getAttribute("authInfo");
            if(authInfo!=null){
            	return true;
            }
        }
        response.sendRedirect(request.getContextPath()+"/lgoin");
        return false;
    }
}
```
preHandle()이 true를 반환하면 컨트롤러를 실행하므로 로그인 상태면 컨트롤러를 실행한다. 반대로 false를 리턴하면 로그인 상태가 아니므로 지정한 경로로 리다이렉트 한다. 

### HandlerInterceptor 설정하기 
```java
@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer{
	...
    @Override
    public void addInterceptors(InterceptorRegistry registry){
    	registry.addInterceptor(authCheckInterceptor())
        .addPathPatterns("/edit/**");
    }
    
    @Bean
    public AuthCheckInterceptor authCheckInterceptor(){
    	return new AuthCheckInterceptor();
    }
}
```
![](https://velog.velcdn.com/images/yh_lee/post/ac478e14-7b97-4acc-a3c5-513e5f2936f9/image.png)

addPathPatterns()에서 지정한 경로 중 패턴 일부를 제외하고 싶으면 excludePathPatterns()를 사용한다.
```java
@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer{
	...
    @Override
    public void addInterceptors(InterceptorRegistry registry){
    	registry.addInterceptor(authCheckInterceptor())
        .addPathPatterns("/edit/**");
        .excludePathPatterns("/edit/help/**");
    }
}
```

## 컨트롤러에서 쿠키 사용하기
사용자에 편의를 위해 아이디를 기억해두었다가 다음 로그인할 때 아이디를 자동으로 넣어주는 사이트가 많다. 이 기능을 구현할 때 쿠키를 사용한다. 

이메일 기억하기 기능을 위해 수정할 코드는 네 곳이다. 
- loginForm.jsp: 이메일 기억하기 체크박스 추가
- LoginController의 form(): 쿠키가 존재할 경우 폼에 전달할 커맨드 객체의 email 프로퍼티를 쿠키의 값으로 설정한다. 
- LoginController의 submit(): 이메일 기억하기 옵션을 선택한 경우 로그인 성공 후에 이메일을 담고 있는 쿠키를 생성한다. 
- label.properties: 메시지를 추가한다.

![](https://velog.velcdn.com/images/yh_lee/post/195eb3de-a48d-4c77-bb2e-c7a3ebd8d5fc/image.png)![](https://velog.velcdn.com/images/yh_lee/post/85bc8553-a41b-4bd8-abd6-a72dcc787bea/image.png)

LoginController의 form() 메서드는 이메일 정보를 기억하고 있는 쿠키가 존재하면 해당 쿠키의 값을 이용해서 LoginCommand 객체의 email 프로퍼티 값을 설정하면 된다. 
```java
@Controller
@RequestMapping("/login")
public class LoginController{
	private AuthService authService;
    public void setAuthService(AuthService authService){
    	this.authService = authService;
    }
    
    @GetMapping
    public String form(
    		LoginCommand loginCommand, 
    		@CookieValue(value="REMEMBER",required=false) Cookie rCookie){
   		if(rCookie!=null){
        	loginCommand.setEmail(rCookie.getValue());
            loginCommand.setRememberEmail(true);
        }
        return "login/loginForm";
    }
}
```
@CookieValue 어노테이션의 value 속성은 쿠키의 이름을 지정한다. 이름이 REMEMBER인 쿠키를 Cookie 타입으로 전달받는다. 지정한 이름을 가진 쿠키가 존재하지 않을 수도 있다면 required 속성 값을 false로 둔다.

<br>
실제로 REMEMBER 쿠키를 생성하는 부분은 로그인을 처리하는 submit() 메서드이다. 쿠키를 생성하려면 HttpServletResponse 객체가 필요하므로 submit() 메서드 파라미터로 전달한다. 

```java
@Controller
@RequestMapping("/login")
public class LoginController{
	...
    @PostMapping
    public String submit(
    		LoginCommand loginCommand,
            Errors errors,
            HttpSession session,
            HttpServletResponse response
    				){
    	new LoginCommandValidator().validate(loginCommand,errors);
        if(errors.hasError()){
        	return "login/loginForm";
        }
        try{
        	AuthInfo authInfo = authService.authticate(
            	loginCommand.getEmail(),
                loginCommand.getPassword());
                
            session.setAttribute("authInfo",authInfo);
            
            Cookie rememberCookie = new Cookie("REMEMBER", loginCommand.getEmail());
            rememberCookie.setPath("/");
            if(loginComand.isRememberEmail()){
            	rememberCookie.setMaxAge(60*60*24*30);
            }else{
            	rememberCookie.setMaxAge(0);
            }
            response.addCookie(rememberCookie);
            
            return "login/loginSuccess";
        }catch(IdPasswordNotMatchingException e){
        	errors.reject("idPasswordNotMatching");
            return "login/loginForm";
        }
    }	
}
```