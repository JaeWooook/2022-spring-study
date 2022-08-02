# MVC2 : 메시지, 커맨드 객체 검증

<BR>

## 1. 메시지 처리

<br>

### 🔎 ```<spring:message>``` 태그로 메시지 출력

<Br>

1️⃣ 문자열 담은 메시지 파일 작성 : label.properties

```
...
member.register=회원가입
term=약관
term.agree=약관 동의
next.bin=다음단계
...
```
<br>

2️⃣ 메시지 파일에서 값을 읽어오는 MessageSource 빈 설정
```
<MvcConfig.java>

...
@Bean // 빈 이름 반드시 MessageSource여야 함
    public MessageSource messageSource(){
        ResourceBundleMessageSource ms = new ResourceBundleMessageSource();

        ms.setBasenames("message.label"); // 사용할 메시지 프로퍼티 목록 전달
        ms.setDefaultEncoding("UTF-8");
        return ms;
    }

* setBasenames → basenames 프로퍼티 값으로 message 패키지에 속한 label 프로퍼티 파일로부터 메시지 읽어온다고 설정
```
<br>

3️⃣ jsp 코드에서 ```<spring:message>``` 태그 이용해 메시지 출력

    <head>
    <title><spring:message code="member.register" /></title>
    </head>
    <body>
        <h2><spring:message code="member.info" /></h2>
        <form:form action="step3" modelAttribute="registerRequest">
        <p>
            <label><spring:message code="email" />:<br>
            <form:input path="email" />
            <form:errors path="email"/>
            </label>
        </p>
        <p>`
        ...

- ```<spring:message>```의 code 값 = label.properties의 프로퍼티 이름과 동일

<br>

📌 ```<spring:message>``` 태그는 MessageSource로부터 코드에 해당하는 메세지 읽어옴 ➡️ MessageSource는 label.properties 파일로부터 메시지 읽어옴


<br><Br>

### 🔎 MessageSource

: 스프링은 지역에 상관없이 일관된 방법으로 문자열을 처리할 수 있는 MessageSource 인터페이스 정의 

<br>

```
public interface MessageSource {

	@Nullable
	String getMessage(String code, @Nullable Object[] args, @Nullable String defaultMessage, Locale locale);

```
- code : 메시지를 구분하기 위한 파라미터
- locale : 지역을 구분하기 위한 파라미터
    ∴ 같은 코드라 하더라도 지역에 따른 다른 메시지 제공 가능

- Object[] args : 인덱스 기반 변수 전달하기 위한 배열 타입 파라미터 
    
    ```
    🏷️ properties

    msg.text ={0}님, {1}되었습니다.

    🏷️ jsp

    <spring:message code="msg.text" arguments="개발자, 처리" var="msg"/>
    ${msg}
    <c:out value="${msg}">

    → 개발자님, 처리되었습니다.
    ```

 <Br>

 📌 MessageSource인터페이스의 구현체 : ResourceBundleMessageSource (자바의 프로퍼티 파일로부터 메시지 읽어옴)

 → 메시지 코드와 일치하는 이름을 가진 프로퍼티의 값을 메시지로 제공 (해당 프로퍼티 파일은 클래스 패스에 위치해야함)

 ➕ ```<spring:message>``` 태그를 실행하면 내부적으로 MessageSource이 getMessage() 메소드 실행해 필요한 메시지 구함

(code 속성에 지정한 메시지 없으면 500 익셉션 발생)

<br><Br>

## 2. 커맨드 객체 값 검증

<br>

### 🔎 커맨드 객체 값 검증과 에러메시지 처리

- 커맨드 객체를 검증하고 결과를 에러코드로 저장
- jsp에서 에러코드로부터 메시지 출력

📌 Validator : 객체를 검증할때 사용하는 인터페이스 
```
public interface Validator {

	boolean supports(Class<?> clazz);
    // 검증할 수 있는 타입인지 검사

	void validate(@Nullable Object target, Errors errors);
    // target 검증하고 오류 결과 errors에 저장
}
```

```
public class RegisterRequestValidator implements Validator {
    
    ...

    public RegisterRequestValidator(){
        pattern = Pattern.compile(emailRegExp);
    }

    @Override
    public boolean supports(Class<?> clazz) {
        return RegisterRequest.class.isAssignableFrom(clazz);
    }

    @Override
    public void validate(Object target, Errors errors) {
        RegisterRequest regReq = (RegisterRequest) target; // 실제 타입으로 변환
        if(regReq.getEmail()==null || regReq.getEmail().trim().isEmpty()){
            errors.rejectValue("email","required"); //if문 통해 검사 후 email 비어있으면 errors에 required 저장(에러코드)
        }else{
            Matcher matcher = pattern.matcher(regReq.getEmail());
            if(!matcher.matches()){
                errors.rejectValue("email","bad"); // if문 통해 검사 후 정규표현식 만족x면 errors에 bad 저장
            }
        }

        ValidationUtils.rejectIfEmptyOrWhitespace(errors,"name","required");
        ValidationUtils.rejectIfEmpty(errors,"password","required");
        ValidationUtils.rejectIfEmpty(errors,"confirmPassword","required");
        if(!regReq.getPassword().isEmpty()){
            if(!regReq.isPasswordEqualToConfirmPassword()){
                errors.rejectValue("confirmPassword","nomatch");
            }
        }
    }
}

```
<br>

- supports(Class<?> clazz) : 파라미터로 받은 clazz 객체가 RegisterRequest 클래스로 타입 변환 가능한지 확인
- validate(Object target, Errors errors) 
    - target(검사 대상 객체)의 특정 프로퍼티나 상태가 올바른지 검사
    - 올바르지 않다면 Errors의 rejectValue() 메소드 이용해 에러코드 errors에 저장

    <Br>

📌 rejctValue() : 프로퍼티 이름을 첫번째 인자로 전달받아, 두번째 인자인 에러코드를 전달 받음. jsp 에서 에러코드 이용해 에러메시지 출력

<br>

📌 ValidationUtils.rejectIfEmptyOrWhitespace(errors, "name", "required");

: 객체의 값 검증 코드 간결하게 작성 = name프로퍼티가 null이거나 공백이면 에러코드 required 저장
(rejectIfEmpty는 null만 검사) 

➕ 이때, 검사 대상 객체인 target을 파라미터로 전달하지 않는데 어떻게 target의 name 프로퍼티의 값을 검사??

: Errors 객체때문! 
```
    @PostMapping("/register/step3")
    public String handleStep3(@Valid RegisterRequest regReq, Errors errors){
        if(errors.hasErrors()) {
            return "register/step2";
        }
        try{
            memberRegisterService.regist(regReq);
            return "register/step3";
        }catch(DuplicateMemberException e){
            errors.rejectValue("email","duplicate");
            return "register/step2";
        }
    }
```
handleStep3 메소드를 호출할때, 커맨드 객체와 연결된 Errors 객체를 생성해 파라미터로 전달함, Errors 객체는 커맨드 객체의 특정 프로퍼티 값을 구할 수 있는 getFieldValue() 메소드 제공

📌 errrors.hasErrors() : validate()를 실행하는 과정에서 유효하지 않는 값이 존재하면 rejectValue() 메소드 실행, 이 메소드가 한번이라도 불리면 hasErrors()는 true 리턴


<br>
➕ 커맨드 객체의 특정 프로퍼티가 아닌 커맨드 객체 자체가 잘못됐다면?

: rejectValue() 대신 reject() 메소드 사용
```
try{
    ...
}catch(WrongPasswordException e){
    //특정 프로퍼티가 아닌 커맨드 객체 자체에 에러코드 추가
    errors.reject("notMatchingIdPassword");
    return "login/loginForm";
}
```
= **글로벌 에러** : 커맨드 객체 자체에 에러 추가하므로 value(프로퍼티) 작성x, 에러코드만 추가

➕ 요청 매피 어노테이션을 붙인 메소드에 Errors 타입 파라미터 추가할때!! Errors 타입 파라미터는 반드시 **커맨드 객체 다음에** 위치!!!


<br>
<Br>

### 🔎 커맨드 객체의 에러메시지 출력 ```<form:errors>```

```<form:errors path="email">```

: email프로퍼티에 에러 코드가 존재하면 ```<form:errors>```태그는 에러코드에 해당하는 메시지 출력

- 속성
    - element : 에러메시지 출력할때 사용할 html 태그(기본값:```<span>```)
    - delimiter : 에러메시지 구분할때 사용할 html 태그(기본값:```<br/>```)
    - path : 지정하지 않으면 글로벌에러에 대한 메시지 출력

<Br><Br>

### 🔎 글로벌 범위 Validator와 컨트롤러 범위 Validator

- 글로벌 범위 Validator : 모든 컨트롤러에 적용할 수 있음

- 컨트롤러 범위 Validator : 단일 컨트롤러에 적용할 수 있음

1️⃣ 글로벌 범위 Validator & @Valid 어노테이션
    - 설정클래스에서 WebMvcConfigurer의 getValidator() 메소드가 Validator 구현 객체 리턴하게 구현
```
@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer {
    @Override
    public Validator getValidator(){
        return new RegisterRequestValidator();
    }

```

- 글로벌 범위 Validator를 지정하면 @Valid 어노테이션 사용해 Validator 적용가능 

```
@PostMapping("/register/step3")
public String handleStep3(@Valid RegisterRequest regReq, Errors errors){
    if(errors.hasErrors()) {
        return "register/step2";
    }
    try{
        memberRegisterService.regist(regReq);
        return "register/step3";
    }catch(DuplicateMemberException e){
        errors.rejectValue("email","duplicate");
        return "register/step2";
    }
}
```
1. handleStep3() 메소드를 실행 전 @Valid 어노테이션이 붙은 regReq 파라미터를 글로벌 범위 Validator로 검증

2. 그 결과 errors에 저장

3. handleStep3()는 검증하는 코드를 작성할 필요 없음. 검증 에러가 존재하는지만 확인하면 됨(hasErrors())

➕ @Valid 사용시 Errors 타입 파라미터가 없다면 검증 실패시 400 에러 응답

<br>

2️⃣ @IniBinder 어노테이션 이용한 컨트롤러 범위 Validator

```
🏷️ RegisterController

@PostMapping("/register/step3")
public String handleStep3(@Valid RegisterRequest regReq, Errors errors){
    if(errors.hasErrors()) {
        return "register/step2";
    }
    try{
        memberRegisterService.regist(regReq);
        return "register/step3";
    }catch(DuplicateMemberException e){
        errors.rejectValue("email","duplicate");
        return "register/step2";
    }
}
...

@InitBinder
protected void initBinder(WebDataBinder binder){
    binder.setValidator(new RegisterRequestValidator()); // 컨트롤러 범위에 적용할 validator 설정 가능
}
```
: @InitBinder가 붙은 메소드는 컨트롤러의 요청 처리 메소드를 실행하기 전에 매번 실행됨, 매번 호출되어 WebDataBinder 초기화함

➕ validator 우선순위 : 글로벌 > 컨트롤러 범위

📌 @Valid를 사용하면 validator() 메소드를 호출하는 코드 없음

<br><Br>

### 🔎 Bean Validation을 이용한 값 검증 처리
 = 어노테이션 이용해 커맨드 값 검증

 : Validatoer 작성없이 어노테이션만으로 커맨드 객체의 값 검증 

- 커맨드 객체에 @NotNull, @Digits 등의 어노테이션 이용해 검증규칙 설정

```
public class RegisterRequest {

    @NotBlank //Bean Validation
    @Email
    private String email;
    @Size(min=6)
    private String password;
    @NotEmpty
    private String confirmPassword;
    @NotEmpty
    private String name;
```

- @EnableWebMvc 어노테이션을 설정클래스에 적용

    : OptionalValidatorFactoryBean은 Bean Validation을 적용한 커맨드 객체를 검증함<br>
    @EnableWebMvc를 사용하면 OptionalValidatorFactoryBean을 글로벌 범위 Validator로 등록<br>

    ➕ 이때, 글로벌 범위 Validator 따로 설정한 코드 있다면 삭제!!

- @Valid를 커맨드 객체 앞에 적용
