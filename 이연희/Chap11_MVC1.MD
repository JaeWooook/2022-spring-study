> 스프링 MVC를 사용해서 웹 어플리케이션을 개발하는 것은 **컨트롤러와 뷰 코드를 구현한다는 것을 뜻한다.** 컨트롤러를 통해 어떤 요청 경로를 처리할지 결정하고, 웹 브라우저가 전송한 요청에서 필요한 값을 구하고, 처리 결과를 화면에 뿌려주면 된다. 

# 요청 매핑 어노테이션을 이용한 경로 매핑

- 특정 URL을 처리할 코드
- 처리 결과를 HTML과 같은 형식으로 응답하는 코드
```java

@Controller
public class HelloController {
    @GetMapping("/hello")
    public String hello(Model model, @RequestParam(value = "name", required = false) String name) {
        model.addAttribute("greeting", "안녕하세요 " + name);
        return "hello";
    }
}
```

@RequestMapping 어노테이션을 사용하면 동일한 경로에 대해 전체 메서드에 적용시켜줄 수 있다. 
```java
@Controller
@RequestMapping("/register")//각 메서드에 공통되는 경로
public class RegistController {

    @RequestMapping("/step1")//하위 세부 경로
    public String handleStep1() {
        return "reister/step1";
    }

    public String handleStep2() {
        return "reister/step2";
    }
	...
} 
```
남은 작업은 ControllerConfig.java에 RegisterController 클래스를 빈으로 등록하는 것이다. 

# @GetMapping, @PostMapping
스프링 MVC는 별도의 설정이 없으면 GET과 POST 방식에 상관없이 @requestMapping에 지정한 경로와 일치하는 요청을 처리한다. 만약 @GetMapping이나 @PostMapping을 사용하면 요청 방식을 제한할 수 있다. 
```java
@Controller
public class LoginController
	@GetMapping("/member/login")
    public String form(){
    	...
    }
    
    @PostMapping("member/login")
    public String login(){
    	...
    }
} 
```

# 요청 파라미터 접근
컨트롤러 메서드에서 요청 파라미터를 사용하는 첫번째 방법은 HttpServletRequest를 직접 이용하는 것이다. HttpServletRequest의 getParameter() 메서드를 이용해서 파라미터의 값을 구하면 된다.
```
<form action="step2" method="post">
<label>
  <input type="checkbox" name="agree" value="true"> 약관 동의
</label>
</form>
```
```java
@Controller
public class RegisterController {
    @RequestMapping("/register/step1")
    public String handleStep1() {
        return "register/step1";
    }

    @PostMapping("/register/step2")
    public String handleStep2(HttpServletRequest request) {
        String agreeParam = request.getParameter("agree");
        if (agreeParam == null || !agreeParam.equals("true")) {
            return "register/step1";
        }
        return "register/step2";
    }
}
```
요청 파라미터에 접근하는 두번째 방법능 @RequestParam 어노테이션을 사용하는 것이다. 아래 코드는 agree 파라미터의 값을 읽어와 agreeVal 파라미터에 할당한다. 요청 파라미터의 값이 없으면 "false" 문자열을 값으로 사용한다. 스프링 MVC는 파라미터 타입에 맞게 String 값을 변환해준다. 
```java
@Controller
public class RegisterController {
 
    @PostMapping("/register/step2")
    public String handleStep2(@RequestParam(value = "agree", defaultValue = "false") Boolean agree) {
        if (!agree) {
            return "register/step1";
        }
        return "register/step2";
    }
}
```
# 리다이렉트 처리
웹 브라우저에 http://localhost:8080/sp5-chap11/register/step2 주소를 직접 입력하면 에러 화면이 출력된다. 
RegisterController 클래스의 handleStep2()는 POST 방식만 처리하기 때문에 웹 브라우저에 직접 주소를 입력할 때 사용되는 GET 방식 요청은 처리하지 않는다. 그래서 405 상태 코드를 응답한다. 

잘못된 전송 방식으로 요청이 왔을 때 에러 화면보다 알맞은 경로로 리다이렉트할 수 있다. 

뷰 값으로 "redirect:/register/step"을 사용했는데 이동 경로가 "/"로 시작하므로 실제 리다이렉트할 경로는 웹 어플리케이션 경로인 "/sp5-chap11"과 "/register/step"을 연결한 "/sp5-chap11/register/step1"이 된다. 
```java
@Controller
public class RegisterController {
 
    @GetMapping("/register/step2")
    public String handleStep2Get(){
    	return "redirect:/register/step1;
    }
}
```


# 커맨드 객체를 이용해서 요청 파라미터 사용
step2.jsp가 생성하는 폼은 다음 파라미터를 이용해서 정보를 서버에 전송한다. 
- email
- name
- password
- confirmPassword
폼 전송 요청을 처리하는 컨트롤러를 아래와 같이 생성했다. 하지만 이 코드는 올바르게 동작하지만 요청 파라미터 개수가 증가할 때마다 코드의 길이도 함께 길어지는 단점이 있다. 
```java
@PostMapping("register/step3")
public String handleStep3(HttpServletRequest request) {
    String email = request.getParameter("email");
    String name = request.getParameter("name");
    String password = request.getParameter("password");
    String confirmPassword = request.getParameter("confirmPassword");

    RegisterRequest regReq = new RegisterRequest;
    regReq.setEmail(email);
    regReq.setName(name);
}
```
스프링은 이런 불편함을 줄이기 위해 요청 파라미터 값을 커맨드(command) 객체에 담아주는 기능을 제공한다. 커맨드 객체는 요청 파라미터의 값을 전달받을 수 있는 세터 메서드를 포함하는 객체이다. 

helloStep3()는 MemberRegisterService를 이용해서 회원 가입을 처리한다. 
```java
@Controller
public class ReigsterController {
    private MemberRegisterService memberRegisterService;

    public void setMemberRegisterService(MemberRegisterService memberRegisterService) {
        this.memberRegisterService = memberRegisterService;
    }
    ...

    @PostMapping("/register/step3")
    public String handelStep3(RegisterRequest regReq) {
        try {
            memberRegisterService.regist(regReq);
            return "register/step3";
        } catch (DuplicateMemberException exception) {
            return "register/step2";
        }
    }
}
```
# 뷰 JSP 코드에서 커맨드 객체 사용
JSP의 ${registerRequest.name} 코드가 있다. 여기서 registerRequest가 커맨드 객체에 접근할 때 사용한 속성 이름이다. 스프링 MVC는 커맨드 객체의 (첫 글자를 소문자로 바꾼) 클래스 이름과 동일한 속성 이름을 사용해서 커맨드 객체를 뷰에 전달한다. 커맨드 객체의 클래스 이름이 RegisterRequest인 경우 JSP 코드는 registerRequest 이름을 사용해서 커맨드 객체에 접근할 수 있다. 

```jsp
<p>${registerRequest.name}님 회원가입을 완료했습니다.</p>
```
```java
@PostMapping("register/step3")
public String handleStep3(RegisterRequest regReq){
	...
}
```

# @ModelAttribute 어노테이션으로 커맨드 객체 속성 이름 변경
커맨드 객체에 접근할 때 사용할 속성 이름을 변경하고 싶으면 커맨드 객체로 사용할 파라미터에 @ModelAttribute 어노테이션을 적용하면 된다. 
```java
@PostMapping("/register/step3")
public String handleStep3(@ModelAttribute("formData") RegisterRequest regReq){
	...
}
```
@ModelAttribute는 모델에서 사용할 속성 이름을 값으로 설정한다. 위 설정을 이용하면 뷰 코드에서 "formData"라는 이름으로 커맨드 객체에 접근할 수 있다.

# 커맨드 객체와 스프링 폼 연동
회원 정보 입력 폼에서 중복된 이메일 주소를 입력하면 텅 빈 폼을 보여준다. 폼이 비어있으면 사용자가 입력값을 다시 입력해야 하는 불편함이 있다. 이때 커맨드 객체에 값을 폼에 채워주면 이런 불편함을 해소할 수 있다.
```java
<input type="text" name="email" id="email" value="${registerRequest.email}">
...
<input type="text" name="name" id="name" value="${registerRequest.name}">
```
스프링 MVC가 제공하는 커스텀 태그를 사용할 수도 있다. <form:form> 태그와 <form:input> 태그를 지원한다. 
```jsp
...
<form:form action="step3" modelAttribute="registerRequest">
...
<form:input path="email"/>
...
<form:password path="password"/>
...
</form:form>
```

- action: <form\>태그의 action 속성과 동일한 값을 사용한다. 
- modelAttribute: 커맨드 객체의 속성 이름을 지정한다.

step1에서 step2로 넘어오는 단계에서 이름이 "registerRequest"인 객체를 모델에 넣어야 <form:form> 캐그가 정상 동작한다. 이를 위해 RegisterController에 코드를 추가했다.
```java
@Controller
public class RegisterController {
 	...
    @PostMapping("/register/step2")
    public String handleStep2(@RequestParam(value = "agree", defaultValue = "false") Boolean agree, Model model) {
        if (!agree) {
            return "register/step1";
        }
        model.addAttribute("registerRequest", new RegisterRequest());
        return "register/step2";
    }
}
```
# 컨트롤러 구현 없는 경로 매핑
```java
//변경 전
@Controller
public class MainController{
	@RequestMapping("/main")
    public String main(){
    	return "main";
    }
}
//변경 후
@Override
public void addViewControllers(ViewControllerRegistry registry){
	registry.addViewController("/main").setViewName("main");
}
```
변경 전처럼 요청 결오와 뷰 이름을 연결해주는 기능만 있는 컨트롤러를 만드는 것은 성가신 일이다. WebMvcConfigurer 인터페이스의 addViewControllers()를 사용하여 컨트롤러 구현없이 요청 경로와 뷰 이름을 간단하게 연결할 수 있다.

# 주요 에러 발생 상황
## 요청 매핑 관련 익셉션
 1. 요청 경로를 처리할 컨트롤러가 존재하지 않거나 WebMvcConfigurer를 이용한 설정이 없다면 404 에러가 발생한다. 아래 항목은 에러가 발생하면 확인해 볼 사항들이다.
 - 요청 경로가 올바른지
 - 컨트롤러에 설정한 경로가 올바른지
 - 컨트롤러 클래스를 빈으로 등록했는지
 - 컨트롤러 클래스에 @Controller 어노테이션을 적용했는지
 2. 뷰 이름에 해당하는 JSP 파일이 존재하지 않아도 404 에러가 발생한다. 차이점은 존재하지 않는 JSP 파일의 경로가 출력된다는 점이다. 
 3. 지원하지 않는 전송 방식을 사용한 경우 405 에러가 발생한다. POST 방식만 처리하는 요청 경로를 GET 방식으로 연결하면 에러가 발생한다.
 
## @RequestParam이나 커맨드 객체 관련 익셉션
1. <input\> 태그에 값을 체크하지 않은 상태로 넘어오게 되면 파라미터에 아무 값도 전송하지 않는다. 파라미터가 존재하지 않는다는 익셉션이 발생한다. 스프링 MVC는 이 익셉션이 발생하면 400 에러를 응답으로 전송한다. 
2. 요청 파라미터 값을 @RequestParam이 적용된 파라미터 타입으로 변환할 수 없는 경우에도 에러가 발생한다. 아래 코드가 400 에러가 발생하는 이유는 "true1" 값을 Boolean 타입으로 변환할 수 없기 때문이다.
```
<input type="checkbox" name="agree" value="true1">
```

# 커맨드 객체: 중첩, 콜렉션 프로퍼티
스프링 MVC는 커맨드 객체가 리스트 타입의 프로퍼티를 가졌거나 중첩 프로퍼티를 가진 경우에도 요청 파라미터의 값을 알맞게 커맨드 객체에 설정해주는 기능을 제공하고 있다. 
```java
public class Respondent{
	private int age;
    private String location;
    ...
}
public class AnsweredData{
	private List<String> responses;
    private Respondent res;
    ...
}
```
```java
@Controller
@RequestMapping("/survey")
public class SuveyController {
    @GetMapping
    public String form() {
        return "survey/surveyForm";
    }

    @PostMapping
    public String submit(@ModelAttribute("ansData") AnsweredData data) {
        return "survey/submitted";
    }
}
```
submit() 메서드는 커맨드 객체로 AnsweredData 객체를 사용한다.

surveyForm.jsp를 살펴보자. <input\> 태그의 name 속성은 다음과 같이 커맨드 객체의 프로퍼티에 매핑된다. 
![](https://velog.velcdn.com/images/yh_lee/post/24cd6ea1-2bae-4942-8998-5576df600923/image.png)

# Model을 통해 컨트롤러에서 뷰에 데이터 전달
컨트롤러는 뷰가 응답 화면을 구성하는데 필요한 데이터를 생성해서 전달해야 한다. 이때 사용하는 것이 Model이다. 
```java
@Controller
public class HelloController{
	public String hello(Model model, @RequestParam(value="name", required=false) String name){
    	model.addAttribute("greeting", "안녕하세요 "+name);
        return "hello";
  }
}
```
addAttribute()의 첫번째 파라미터는 속성 이름이다. 뷰 코드는 이 이름을 사용해서 데이터에 접근한다. JSP는 다음과 같이 표현식을 사요해서 속성값에 접근한다.
```
${greeting}
```
앞서 작성한 SurveyController는 surveyFrom.jsp에 설문 항목을 하드 코딩했다. 설문 항목을 컨트롤러에서 생성해서 뷰에 전달하는 방식으로 변경해보자. 먼저 개별 설문 항목 데이터를 담기 위한 클래스를 아래와 같이 작성한다.
```java
public class Question{
	private String title;
    private List<String> options;
}
```
```java
@Controller
@RequestMapping("/survey")
public class SurveyController {

    @GetMapping
    public String form(Model model) {
        List<Question> questions = createQuestions();
        model.addAttribute("questions", questions);
        return "survey/survryForm";
    }

    private List<Question> createQuestions() {
        Question q1 = new Question("당신의 역할은 무엇입니까?", Array.asList("서버", "프론트", "풀스택"));
    }
}
```
form()에서 Model 타입의 파라미터를 추가헀고 questions라는 이름으로 모델에 추가했다. ![](https://velog.velcdn.com/images/yh_lee/post/216cf9aa-2b65-4f43-9bad-43857cad69d3/image.png)

Question 리스트를 사용해서 폼 화면은 생성하도록 JSP 코드를 수정했다. SurveyController에서 Model을 통해 전달한 Question 리스트를 이용해서 설문 폼을 생성할 수 있다.

## ModelAndView를 통한 뷰 선택과 모델 전달
지금까지 구현한 컨트롤러는 두가지 특징이 있다. 
- Model을 이용해서 뷰에 전달할 데이터 설정
- 결과를 보여줄 뷰 이름을 리턴

ModelAndView를 사용ㅇ하면 이 두가지를 한번에 처리할 수 있다. 요청 매핑 어노테이션을 적용한 메서드는 String 타입 대신 ModelAndView를 리턴할 수 있다. 뷰에 전달할 모델 데이터는 addObject()로 추가한다. 뷰 이름은 setViewName() 메서드를 이용해서 지정한다.
```java
@Controller
@RequestMapping("/survey")
public class SurveyController{
	@GetMapping
    public ModelAndview form(){
    	List<Question> questions = createQuestions();
        ModelAndView mav = new ModelAndView();
        mav.addObject("questions", questions);
        mav.setViewName("survey/surveyForm");
        return mav;
    }
}
```

## GET 방식과 POST 방식에 동일 이름 커맨드 객체 사용
```java
@Controller
@RequestMapping("/login")
public class LoginController{
	@GetMapping
    public String form(@MoodelAttribute("login") LoginCommand loginCommand){
    	return "login/loginForm";
    }
    
    @PostMapping
    public String form(@MoodelAttribute("login") LoginCommand loginCommand){
    	...
    }
    
}
```
