## MVC2 : 커맨드 객체 검증
### 커맨드 객체의 값 검증과 에러 메시지 처리
> 폼 값 검증과 에러 메시지 처리는 어플리케이션 개발에서 중요하다.

폼 값 검증과 에러 메시지 처리를 위한 2가지 기능이다.

- 커맨드 객체를 검증하고 결과를 에러 코드로 저장
- JSP에서 에러 코드로부터 메시지를 출력

#### 커맨드 객체 검증과 에러 코드 지정하기
 > 스프링 MVC에서 커맨드 객체의 값이 올바른지 검사하기 위해 두가지 인터페이스를 사용한다.

 - org.springframework.validation.Validator
 - org.springframework.validation.Errors

객체를 검증할 때 사용하는 Validator 인터페이스
```java
package  org.springframework.validation;

public interface Validator {
    boolean supports(Class<?> clazz);
    void validate(Object target, Error erros);
}
```
 - supports() 메서드는 Validator가 검증할 수 있는 타입인지 검사한다.
 - validate() 메서드는 첫 번째 파라미터로 전달받은 객체를 검증하고, 오류 결과를 Errors에 담는 기능을 정의한다.

#### RegisterRequest 객체를 검증하기 위한 Validator 구현 클래스
```java
package controller;

import java.util.regex.Matcher;
import java.util.regex.Pattern;

import org.springframework.validation.Errors;
import org.springframework.validation.ValidationUtils;
import org.springframework.validation.Validator;

import spring.RegisterRequest;

public class RegisterRequestValidator implements Validator {
	private static final String emailRegExp = 
			"^[_A-Za-z0-9-\\+]+(\\.[_A-Za-z0-9-]+)*@" +
			"[A-Za-z0-9-]+(\\.[A-Za-z0-9]+)*(\\.[A-Za-z]{2,})$";
	private Pattern pattern;

	public RegisterRequestValidator() {
		pattern = Pattern.compile(emailRegExp);
		System.out.println("RegisterRequestValidator#new(): " + this);
	}

	@Override
	public boolean supports(Class<?> clazz) {
		return RegisterRequest.class.isAssignableFrom(clazz);
	}

	@Override
	public void validate(Object target, Errors errors) {
		System.out.println("RegisterRequestValidator#validate(): " + this);
		RegisterRequest regReq = (RegisterRequest) target;
		if (regReq.getEmail() == null || regReq.getEmail().trim().isEmpty()) {
			errors.rejectValue("email", "required");
		} else {
			Matcher matcher = pattern.matcher(regReq.getEmail());
			if (!matcher.matches()) {
				errors.rejectValue("email", "bad");
			}
		}
		ValidationUtils.rejectIfEmptyOrWhitespace(errors, "name", "required");
		ValidationUtils.rejectIfEmpty(errors, "password", "required");
		ValidationUtils.rejectIfEmpty(errors, "confirmPassword", "required");
		if (!regReq.getPassword().isEmpty()) {
			if (!regReq.isPasswordEqualToConfirmPassword()) {
				errors.rejectValue("confirmPassword", "nomatch");
			}
		}
	}

}
```
 - support() 메서드는 파라미터로 전달받은 clazz 객체가 RegisterRequest클래스로 타입 변환이 가능한지 확인한다.
 - validate() 메서드는 target 파라미터는 검사 대상 객체이고, errors 파라미터는 검사 결과 에러 코드를 설정하기 위한 객체이다.
 - validate() 메서드 구현하는 방법
   - 검사 대상 객체의 특정 프로퍼티나 상태가 올바른지 검사
   - 올바르지 않다면 Errors의 rejectValue() 메서드를 이용해서 에러 코드 저장
 - 검사 대상의 값을 구하기 위해 전달받은 target을 실제 타입으로 변환한 뒤에 검사를 실행한다.
 - 유효성 검사를 위해 email 프로퍼티 값이 존재하지 않거나, 빈문자열인 경우 errors.rejectValue를 이용해서 에러 코드로 "required"를 추가한다.
 - 정규 표현식을 이용해서 이메일이 올바른지 확인한다. 정규 표현식이 일치하지 않으면 errors.rejectValue를 이용해서 "email"프로퍼티의 에러 코드로 "bad"를 추가한다.
 
> ValidationUtils.rejectIfEmptyOrWhitespace(errors, "name", "required");

 - ValidationUtils 클래스는 객체의 값 검증 코드를 간결하게 작성할 수 있도록 도와준다.
 - 검사 대상 객체의 "name"프로퍼티가 null이거나 공백문자로만 되어 있는 경우 "name" 프로퍼티의 에러 코드로 "required"를 추가한다.

```java
if (regReq.getEmail() == null || regReq.getEmail().trim().isEmpty()) {
			errors.rejectValue("email", "required");
		}

ValidationUtils.rejectIfEmptyOrWhitespace(errors, "name", "required");  
```
 > 위 코드에서 if조건문 내부의 코드와 ValidationUtils클래스의 rejectIfEmptyOrWhitespace메서드를 이용한 코드는 동일한 기능을 제공한다.

 > ValidationUtils.rejectIfEmptyOrWhitespace메서드를 실행할 때 검사 대상 객체인 target을 파라미터로 전달하지 않았는데 어떻게 target 객체의 "name" 프로퍼티의 값을 검사할까?

 - 스프링 MVC에서 Validator를 사용하는 코드는 요청 매핑 어노테이션 적용 메서드에 Errors 타입 파라미터를 전달받고, 이 Errors 객체를 Validator의 validate() 메서드에 두 번째 파라미터로 전달한다.

```java
@PostMapping("/register/step3")
	public String handleStep3(RegisterRequest regReq, Errors errors) {
		new RegisterRequestValidator().validate(regReq, errors);
		if (errors.hasErrors())
			return "register/step2";

		try {
			memberRegisterService.regist(regReq);
			return "register/step3";
		} catch (DuplicateMemberException ex) {
			errors.rejectValue("email", "duplicate");
			return "register/step2";
		}
	}
```
 - public String handleStep3(RegisterRequest regReq, Errors errors) 객체 파라미터 뒤에 Errors 타입 파라미터가 위치하면, 스프링 MVC는 handleStep3() 메서드를 호출할 때 커맨드 객체와 연결된 Errors 객체를 생성해서 파라미터로 전달한다.
 - 이 Errors 객체는 커맨드 객체의 특정 프로퍼티 값을 구할 수 있는 getFieldValue() 메서들ㄹ 제공한다.
 - 따라서 ValidationUtils.rejectIfEmptyOrWhitespace() 메서드는 커맨드 객체를 전달받지 않아도 Errors 객체를 이용해서 지정한 값을 구할 수 있다.
 - new RegisterRequestValidator().validate(regReq, errors); --> __RegisterRequestValidator 객체를 생성하고, validate() 메서드를 실행 한다.__
 - 이를 통해서 RegisterRequest 커맨드 객체의 값이 __올바른지 검사하고, 그 결과를 Errors 객체에 담는다.__
 - validate()를 실행하는 과정에서 유효하지 않은 값이 존재하면 Errors의 rejectValue() 메서드를 실행한다. __이 메서드가 한 번이라도 불리면 Errors의 hasErrors() 메서드는 true를 리턴한다.__
 - memberRegisterService.regist(regReq); email에 대한 중복이 발생하면, errors.rejectValue("email", "duplicate");를 이용해서 이메일 중복 에러를 추가하기 위해 "email" 프로퍼티의 에러 코드로 "duplicate"를 추가한다.

```java
try{
        ...아이디와 비밀번호 인증 코드
        } catch(WrongIdPasswordException ex) {
    errors.reject("notMatchingIdPassword");
    return "login/loginForm";
        }
```
 - 아이디와 비밀번호 불일치의 경우 특정 프로퍼티에 에러 추가보다는 커맨드 객체 자체에 에러를 추가하는게 맞다.
 - 커맨드 객체의 특정 프로퍼티가 아닌 커맨드 객체 자체가 잘못되는 경우는 __rejectValue() 메서드 대신 reject() 메서드를 사용한다.__
 - __reject() 메서드는 개별 프로퍼티가 아닌 객체 자체에 에러 코드를 추가하므로 이 에러를 글로벌 에러라고 부른다.__

> 요청 매핑 어노테이션을 붙인 메서드에 Errors 타입의 파라미터를 추가할 때 주의할 점은 Errors 타입 파라미터는 반드시 커맨드 객체를 위한 파라미터 다음에 위치해야한다.
> 그렇지 않고, Errors 타입 파라미터가 커맨드 객체 앞에 위치하면 익셉션이 발생한다.

#### Errors 타입 파라미터가 커맨드 객체 앞에 위치하면 실행 시점에 에러 발생
```java
//Errors 타입 파라미터가 커맨드 객체 앞에 위치하면 실행 시점에 에러 발생
@PostMapping("register/step3")
public String handleStep3(Errors errors, RegisterRequest regReq) {
        ...
        }
```

#### Errors 대신에 BindingResult 타입 사용 가능
```java
@PostMapping("register/step3")
public String handleStep3(RegisterRequest regReq, BindingResult errors) {
        new RegisterRequestValidator().validate(regReq, errors);
        ...
        }
```
 - BindingResult 인터페이스는 Errors 인터페이스를 상속하고 있다.

#### Errors와 ValidationUtils 클래스의 주요 메서드

> Errors 인터페이스가 제공하는 에러 코드 추가 메서드 종류

 - reject(String errorCode)
 - reject(String errorCode, String defaultMessage)
 - reject(String errorCode, Object[] errorArgs, String defaultMessage)
 - rejectValue(String field, String errorCode)
 - rejectValue(String field, String errorCode, String defaultMessage)
 - rejectValue(String field, String errorCode, Object[] errorArgs, String defaultMessage)
 
> defaultMessage 파라미터를 가진 메서드를 사용하면, 에러 코드에 해당하는 메시지가 존재하지 않을 때 익셉션을 발생시키는 대신 메시지를 출력한다.

> ValidationUtils 클래스가 제공하는 메서드

 - rejectIfEmpty(Errors errors, String field, String errorCode)
 - rejectIfEmpty(Errors errors, String field, String errorCode, Object[] errorArgs)
 - rejectIfEmptyOrWhitespace(Error errors, String field, String errorCode)
 - rejectIfEmptyOrWhitespace(Errors errors, String field, String errorCode, Object[] errorArgs)
 
> rejectIfEmpty()메서드는 field에 해당하는 프로퍼티 값이 null이거나 빈 문자열("")인 경우 에러 코드로 errorCode를 추가한다.
> 
> rejectIfEmptyOrWhitespace() 메서드는 null이거나 빈 문자열인 경우, 공백 문자(스페이스, 탭)로만 값이 구성된 경우 에러코드를 추가한다.

### 글로벌 범위 Validator와 컨트롤러 범위 Validator

> 스프링 MVC는 @Valid 어노테이션을 사용해서 커맨드 객체에 검증 기능을 적용할 수 있다.

#### 글로벌 범위 Validator 설정과 @Valid 어노테이션
> 글로벌 범위 Validator를 적용하기 위한 두가지 설정

 - 설정 클래스에서 WebMvcConfigurer의 getValidator() 메서드가 Validator 구현 객체를 리턴하도록 구현
 - 글로벌 범위 Validator가 검증할 커맨드 객체에 @Valid 어노테이션 적용

> 글로벌 범위 Validator 설정 방법

```java
@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer {
    @Override
    public Validator getValidator() {
        return new RegisterRequestValidator();
    }
}
```

 - 스프링 MVC는 WebMvcConfigurer 인터페이스의 getValidator() 메서드가 리턴한 객체를 글로벌 범위 Validator로 사용한다.
 - 글로벌 범위 Validator를 지정하면 @Valid 어노테이션을 사용해서 Validator를 적용할 수 있다.
 - 설정한 글로벌 범위 Validator인 RegisterRequestValidator는 RegisterRequest 타입에 대한 검증을 지원한다.(RegisterRequest 클래스의 supports()메서드 참고)
 
> RegisterController 에 @Valid 어노테이션 사용 예시

```java
@PostMapping("/register/step3")
	public String handleStep3(@Valid RegisterRequest regReq, Errors errors) {
		if (errors.hasErrors())
			return "register/step2";

		try {
			memberRegisterService.regist(regReq);
			return "register/step3";
		} catch (DuplicateMemberException ex) {
			errors.rejectValue("email", "duplicate");
			return "register/step2";
		}
	}
```

 - @Valid 어노테이션을 붙이면 글로벌 범위 Validator가 해당 타입을 검증할 수 있는지 확인
 - 검증 가능하면 실제 검증을 수행하고 그 결과를 Errors에 저장한다.
 - 이는 요청 처리 메서드(handleStep3) 실행 전에 적용된다.
 - 현재 메서드에서는 RegisterRequest 타입을 지원하므로 regReq 파라미터로 전달되는 커맨드 객체에 대한 검증을 수행한다.
 - 검증 수행 결과는 Errors 타입 파라미터로 받는다.
 - 메서드 실행되기 전에 이뤄지므로 메서드는 RegisterRequest 객체를 검증하는 코드를 작성할 필요없다.__( ex) new RegisterRequestValidator().validate(regReq, errors); )__ <--검증 하는 코드이다.
 - 파라미터로 전달받은 Errors를 이용해서 검증 에러 존재 여부만 확인하면 된다.

> !주의 사항
> 
> @Valid 어노테이션을 사용할 때 Errors 타입 파라미터가 없으면 검증 실패시 400에러를 응답한다.

ex) 예시 코드
```java
public String handleStep3(@Valid RegisterRequest regReq)
```
> RegisterRequestValidator 클래스는 RegisterRequest 타입의 객체만 검증할 수 있으므로 모든 컨트롤러에 적용할 수 있는 글로벌 범위 Validator를 사용하는 것은 적합하지 않다.

#### @InitBinder 어노테이션을 이용한 컨트롤러 범위 Validator

> @InitBinder 어노테이션을 이용하면 컨트롤러 범위 Validator를 설정할 수 있다.

> 컨트롤러 범위 Validator 설정 RegisterController 코드
```java
@PostMapping("/register/step3")
public String handleStep3(@Valid RegisterRequest regReq, Errors errors) {
        if (errors.hasErrors()) return "register/step2";

        try {
            memberRegisterService.regist(regReq);
            return "register/step3";
        } catch (DuplicateMemberException ex) {
            errors.rejectValue("email", "duplicate");
            return "register/step2";
        }
    }

@InitBinder
protected void initBinder(WebDataBinder binder) {
    binder.setValidator(new RegisterRequestValidator());
        }
```
 - @Vaild 어노테이션을 적용하기 때문에 handleStep3() 메서드에는 Validator 객체의 validate() 메서드를 호출하는 코드가 없다.
 - @InitBinder 어노테이션을 적용한 메서드는 WebDataBinder 타입 파라미터를 갖는데 WebDataBinder#setValidator() 메서드를 이용해서 컨트롤러 범위에 적용할 Validator를 설정할 수 있다.
 - binder.setValidator(new RegisterRequestValidator()); --> __RegisterRequest 타입을 지원하는 RegisterRequestValidator를 컨트롤러 범위 Validator로 설정했으므로 @Valid 어노테이션을 붙인 RegisterRequest를 검증할 때 이 Validator를 사용한다.__ RegisterRequestValidator.java에 정의되어 있다.
 - @InitBinder가 붙은 메서드는 컨트롤러의 요청 처리 메서드를 실행하기전에 매번 실행된다.  ex) handle1(), handle2(), handle3() 이라는 메서드가 있다면 각각의 모든 메서드를 실행하기 전에 initBinder() 메서드를 매번 호출해서 WebDataBinder를 초기화 한다.

#### 글로벌 범위 Validator와 컨트롤러 범위 Validator의 우선 순위

> setVaildator() 메서드를 사용하면 글로벌 범위 Validator 대신에 컨트롤러 범위 Validator를 사용한다.
> 
> addValidator() 메서드를 실행하면 순서상 글로벌 범위 Validator() 뒤에 새로 추가한 컨트롤러 범위 Validator()가 추가된다.

### Bean Validation을 이용한 값 검증 처리

> Bean Validation이 제공하는 어노테이션을 이용해서 커맨드 객체의 값을 검증하는 방법

 - Bean Validation과 관련된 의존을 설정에 추가한다.
 - 커맨드 객체에 @NotNull, @Digits 등의 어노테이션을 이용해서 검증 규칙을 설정한다.

> pom.xml과 build.gradle을 이용해서 의존설정을 추가한다.

> 커맨드 클래스는 다음과 같이 Bean Validation과 프로바이더가 제공하는 어노테이션을 이용해서 값 검증 규칙을 설정한다.
```java
public class RegisterRequest {
    @NotBlank
    @Email
    private String email;
    @Size(min = 6)
    private String password;
    @NotEmpty
    private String confirmPassword;
    @NotEmpty
    private String name;
}
```

> Bean Validation 어노테이션을 사용했다면 OptionalValidationFactoryBean 클래스를 빈으로 등록해야 된다.
> @EnableWebMvc 어노테이션을 사용하면 OptionalValidationFactoryBean을 글로벌 범위 Validator로 등록된다.

```java
@Configuration
@EnableWebMvc //OptionalValidationFactoryBean을 글로벌 범위 Validator로 등록
public class MvcConfig implements WebMvcConfigurer {
   ...
}
```
 - 아래의 Config파일에서 따로 Validator를 설정하면 OptionalValidationFactoryBean을 글로벌 범위 Validator로 사용하지 않기 때문에 설정에서 삭제가 필요하다.
 - 스프링 MVC는 별도로 설정한 글로벌 범위 Validator가 없을 때에 OptionalValidationFactoryBean을 글로벌 범위 Validator로 사용한다.

> 사용할 컨트롤러의 메서드에 @Valid 어노테이션 사용해서 글로벌 범위 Validator로 검증한다.

```java
@PostMapping("/register/step3")
public String handleStep3(@Valid RegisterRequest regReq, Errors errors) {
        if (errors.hasErrors()) return "register/step2";

        try {
            memberRegisterService.regist(regReq);
            return "register/step3";
        } catch (DuplicateMemberException ex) {
            errors.rejectValue("email", "duplicate");
            return "register/step2";
        }
    }
```

#### Bean Validation의 주요 어노테이션
> @AssertTrue, @AssertFalse : 지원 타입 boolean, Boolean

 - 값이 true인지 또는 false인지 검사한다. null은 유효하다고 판단한다.

> @DecimalMax, @DecimalMin : 지원 타입 BigDecimal, BigInteger, CharSequence, 정수타입

- 지정한 값보다 작거나 같은지 또는 크거나 같은지 검사한다. inclusive가 false면 value로 지정한 값은 포함하지 않는다. null은 유효하다고 판단한다.

> @Max, @Min : 지원 타입 BigDecimal, BigInteger, 정수타입

- 지정한 값보다 작거나 같은지 또는 크거나 같은지 검사한다. null은 유효하다고 판단한다.

> @Digits : 지원 타입 BigDecimal, BigInteger, CharSequence, 정수타입

- 자릿수가 지정한 크기를 넘지 않는지 검사한다. null은 유효하다고 판단한다.

> @Size : 지원 타입 CharSequence, Collection, Map, 배열

- 길이나 크기가 지정한 값 범위에 있는지 검사한다. null은 유효하다고 판단한다.

> @Null, @NotNull

- 값이 null인지 또는 null이 아닌지 검사한다.

> @Pattern : 지원 타입 CharSequence

- 값이 정규표현식에 일치하는지 검사한다. null은 유효하다고 판단한다.

> @NotEmpty : 지원 타입 CharSequence, Collection, Map, 배열

- 문자열이나 배열의 경우 null이 아니고 길이가 0이 아닌지 검사한다. 컬렉션의 경우 null이 아니고 크기가 0이 아닌지 검사한다.

> @NotBlank : 지원 타입 CharSequence

- null이 아니고 최소한 한 개 이상의 공백아닌 문자를 포함하는지 검사한다.

> @Posititve, @PositiveOrZero : 지원 타입 BigDecimal, BigInteger, 정수타입

- 양수인지 검사한다. OrZero가 붙은 것은 0 또는 양수인지 검사한다. null은 유효하다고 판단한다.

> @Negative, @NegativeOrZero : 지원 타입 BigDecimal, BigInteger, 정수타입

- 음수인지 검사한다. OrZero가 붙은 것은 0 또는 음수인지 검사한다. null은 유효하다고 판단한다.

> @Email : 지원 타입 CharSequence

- 이메일 주소가 유효한지 검사한다.. null은 유효하다고 판단한다.

> @Future, @FutureOrPresent : 지원 타입 시간 관련 타입 ex) new Date(), ... 등

- 해당 시간이 미래 시간인지 검사한다 OrPresent가 붙은 것은 현재 또는 미래 시간인지 검사한다. null은 유효하다고 판단한다.

> @Past, @PastOrPresent : 지원 타입 시간 관련 타입 ex) new Date(), ... 등

- 해당 시간이 과거 시간인지 검사한다 OrPresent가 붙은 것은 현재 또는 과거 시간인지 검사한다. null은 유효하다고 판단한다.