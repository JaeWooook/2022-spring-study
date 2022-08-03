# <spring:message> 태그로 메시지 출력
```
<label>이메일</label>
<input type="text" name="email">
```
'이메일'을 '이메일 주소'로 변경하게 되면 각 폼을 출력하는 모든 JSP를 찾아서 모두 변경해야 한다. 그리고 하드 코딩 되어 있을 경우에 다국어 지원에 유연성이 떨어진다. '이메일'을 'E-mail'로 각 언어에 맞게 따로 뷰 코드를 만들어야 하는 상황이 발생한다.

이런 문제를 해결할 수 있는 방법은 뷰 코드에서 사용할 문자열을 언어별로 파일에 보관하고 뷰 코드는 언어에 따라 파일을 읽어와 출력하는 것이다. 스프링이 자체적으로 이 기능을 제공하고 있다. 

#### label.properties
```
member.register = 회원가입

term = 약관
term.agree = 약관동의
next.btn = 다음단계

member.info = 회원정보
email = 이메일
name = 이름
password = 비밀번호
pqssword.confirm = 비밀번호 확인
register.btn = 가입 완료

register.done = {0}님, 회원 가입을 완료했습니다. 

go.main = 메인으로 이동
```
설정파일에서 MessageSource 타입의 빈을 추가한다.
```java
@Configuration
@EnalbeWebMvc
public class MvcVonfig implements WebMvcConfigurer{
	...
    @Bean
    public MessageSource messageSource(){
    	ResourceBundleMessageSource ms = new ResourceBundleMessageSource();
        ms.setBasenames("message.label");
        ms.setDefaultEncoding("UTF-8");
        return ms;
    }
}
```
위 코드의 ms.setBasename("message.label")에서 프로퍼티 값으로 "message.label"을 주었다. message 패키지에 속한 label 프로퍼티 파일로부터 메시지를 읽어온다고 설정한 것이다. 따라서 label.properties 파일로부터 메시지를 읽어온다. 

JSP 코드는 아래와 같이 수정한다.![](https://velog.velcdn.com/images/yh_lee/post/207e5634-4d27-43ae-a352-946a847814fa/image.png)

<spring:message> 태그는 MessageSource로부터 코드에 해당하는 메시지를 읽어온다. 

**다국어 지원을 위한 메세지 파일**
각 프로퍼티 파일 이름에 언언에 해당하는 로케일 문자를 추가하면 된다. 
- lable_ko.properties
- lable_en.properties

## 메세지 처리를 위한 MessageSource와 <spring:message> 태그
스프링은 로케일(지역) 에 상관없이 일관된 방법으로 문자열(메시지)을 관리할 수 있는 MessageSource 인터페이스를 정의하고 있다. 
## <spring:message> 태그의 인자처리
lable.properties를 보면 다음과 같은 프로퍼티를 포함한다. 이 프로퍼티의 {0} 부분은 인덱스 기반 변수 중 0번 인덱스(첫번째 인덱스) 값으로 대치된다.
```
register.done = {0}님, 회원 가입을 완료했습니다. 
```
Object 배열을 생성해서 인덱스 기반 변수값을 전달할 수 있다. 
 
```java
Object[] args=new Object[1];
args[0]="Java";
messageSource.getMessage("register.done",args,Locale.KOREA);
```
<spring:message> 태그를 사용할 때는 arguments 속성을 사용해서 인덱스 기반 변수값을 전달한다. 
```
<spring:message code="registe.done" arguments="${registerRequest.name}"/>
```

# 커맨드 객체의 값 검증과 에러 메시지 처리
앞서 작성한 회원 가입 처리 코드는 동작은 하지만 비정상 값을 입력해도 동작하는 문제가 있다. 이를 해결하기 위해 입력한 값에 대한 검증 처리를 해야한다. 또 다른 문제는 중복된 이메일 주소를 입력해서 다시 폼을 보여줄 때 왜 가입에 실패했는지 이유를 알려주지 않는다. 그럼 사용자는 혼란을 겪게 된다.

폼 값 검증과 에러 메시지 처리는 어플리케이션을 개발할 때 놓쳐서는 안된다. 스프링은 이 두가지를 해결하기 위해 아래 방법을 제공한다.

- 커맨드 객체를 검증하고 결과를 에러 코드로 저장
- JSP에서 에러 코드로부터 메시지를 출력

## 커맨드 객체 검증과 에러 코드 지정
스프링 MVC에서 커맨드 객체의 값이 올바른지 검사하려면 다음의 두 인터페이스를 사용한다.
- org.springframework.validation.Validator
- org.springframework.validation.Errors
```java
public interface Validator{
	boolean supports(Class<?> clazz);
    void validate(Object target, Errors errors);
}
```
supports()는 Validator가 검증할 수 있는 타입인지 검사한다.
validate() 첫번째 파라미터로 전달받은 객체를 검증하고 오류 결과를 Errors에 담는 기능을 한다.

- 검사 대상 객체가 올바른지 검사
- 올바르지 않으면 Errors의 rejectValue() 메서드를 이용해서 에러 코드 저장

Errors의 첫번째 파라미터로 프로퍼티 이름을 전달받고, 두번째 파라미터로 에러코드를 전달받는다. "email" 프로퍼티 값이 존재하지 않으면 에러 코드로 "required"를 추가한다.
```java
public class RegisterRequestValidator implements Validator{
	@Override
    public void validate(Object target, Errors errors){
    	RegisterRequest regReq = (RegisterRequest) target;
        if(regReq.getEmail()==null || regReq.getEmail().trim().isEmpty()){
        	errors.rejectValue("email","required");
        }
        ...
       
		//errors 객체의 getFieldValue("password") 메서드를 실행해서
		//커맨드 객체의 password 프로퍼티 값을 구함
        //커맨드 객체를 직접 전달하지 않아도 값 검증 가능
        VaildationUtils.rejectIfEmpty(erros,"password","required")
    }
}
```
validate()를 실행하는 과정에서 유효하지 않은 값이 존재하면 Errors의 rejectValue()를 실행한다. 이 메서드가 한번이라도 불리면 Errors.hasErrors()는 true를 반환한다.

커맨드 객체 자체에 문제가 있을 때는 reject()를 사용한다. 아이디, 비밀번호가 잘못 입력한 경우 불일치한다는 메시지를 보여줘야 한다. 이경우 에러를 추가하기 보다 커맨드 객체 자체에 에러를 추가한다. 
_**reject() 메서드는 개별 프로퍼티가 아닌 객체 자체에 에러 코드를 추가하므로 이 에러를 글로벌 에러라고 한다.**_
```java
try{
	...
}catch(WrongIdPasswordException ex){
	errors.reject("notMatchingIdPassword");
    return "login/loginForm";
}
```

Errors 타입 파라미터는 반드시 커맨드 객체를 위한 파라미터 다음에 위치해야 한다. 그렇지 않으면 익셉션이 발생한다.
```java
@PostMapping              //익셉션 발생
public String handleStep3(Errors errors, RegisterRequest regReq){
	...
}
```

## 커맨드 객체의 에러 메시지 출력
에러코드에 해당하는 메시지 코드를 찾을 때는 다음 규칙을 따른다.
1) 에러코드.커맨트객체이름.필드명[인덱스\].필드명
2) 에러코드.필드명
3) 에러코드.필드타입
4) 에러코드

errors.rejectValue("email","required") 코드로 "email" 프로퍼티에 "required" 에러 코드를 추가했고 커맨드 객체 이름이 "registerRequest"이면 다음 순서대로 메세지 코드를 검색한다.
1) required.registerRequest.email
2) required.email
3) required.String
4) required

# 글로벌 범위 Validator와 컨트롤러 범위 Validator
## 글로벌 범위 Validator 설정과 @Valid 어노테이션
글로벌 범위 Validator는 모든 컨트롤러에 적용할 수 있는 Validator이다.

설정 클래스에서 WebMvcConfigurer의 getValidator()가 Validator 구현 객체를 리턴하도록 구현
```java
@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcCofigurer{
	@Override
    public Validator getValidator(){
    	return new RegisterRequestValidator();
    }
}
```
스프링 MVC는 WebMvcConfigurer 인터페이스의 getValidator() 메서드가 리턴한 객체를 글로벌 범위 Validator로 사용한다. 글로벌 범위로 지정하면 @Valid 어노테이션을 사용해서 Validator를 적용할 수 있다. 
```java
@Controller
public class RegisterController{
	@PostMapping
    public String handleStep3(@Valid RegisterRequest regReq, Errors error){
    	...
    }	
}
```

## @InitBinder 어노테이션을 이용한 컨트롤러 범위 Validator
```java
@Controller
public class RegisterController{
	@PostMapping
    public String handleStep3(@Valid RegisterRequest regReq, Errors error){
    	...
    }
    @InitBinder
    protected void initBinder(WebDataBinder binder){
    	binder.setValidator(new RegisterRequestValidator());
    }
}
```

# Bean Validation을 이용한 값 검증 처리 
@Valid 어노테이션은 Bean Validation 스펙에 정의되어 있다. 이 스펙은 @NotNull, @Digits, @Size 등의 어노테이션을 정의하고 있다. 

Bean Validation이 제공하는 어노테이션을 이용해서 커맨드 객체 값을 검증하는 방법은 다음과 같다.
- Bean Validatoin과 관련된 의존을 설정에 추가한다.
- 커맨드 객체에 @NotNull, @Digits 등의 어노테이션을 이용해서 검증 규칙을 설정한다.

```java
public class RegisterRequest{
	@NotNull
    @Email
    private String email;
    
    @Size(min=6)
    private String password;
    
    @NotEmpty
    private String confirmPassword;
    
    @NotEmpty
    private String name;
}
```
 