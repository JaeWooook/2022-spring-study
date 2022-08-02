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

#### Errors와 ValidationUtils 클래스의 주요 메서드

#### 커맨드 객체의 에러 메시지 출력하기

### 글로벌 범위 Validator와 컨트롤러 범위 Validator

#### 글로벌 범위 Validator 설정과 @Valid 어노테이션

#### @InitBinder 어노테이션을 이용한 컨트롤러 범위 Validator

### Bean Validation을 이용한 값 검증 처리

#### Bean Validation의 주요 어노테이션

