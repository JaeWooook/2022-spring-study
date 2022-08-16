## MVC4 : 날짜 값 변환, @PathVariable, 익셉션 처리
### 커맨드 객체 Date 타입 프로퍼티 변환처리: @DateTimeFormat

```java
package controller;

import java.time.LocalDateTime;

import org.springframework.format.annotation.DateTimeFormat;

public class ListCommand {

	@DateTimeFormat(pattern = "yyyyMMddHH")
	private LocalDateTime from;
	@DateTimeFormat(pattern = "yyyyMMddHH")
	private LocalDateTime to;
    
}

```
 - 커맨드 객체(form태그에서 받아오는 객체 값)에 @DateTimeFormat 어노테이션이 적용되어 있으면 @DateTimeFormat에서 지정한 형식을 이용해서 문자열을 LocalDateTime 타입으로 변환한다.
 - pattern 속성값으로 "yyyyMMddHH"를 주었는데 이 경우 "2022081024"의 문자열 즉, "2022년 8월 10일 24시" 값을 갖는 LocalDateTime 객체로 변환해준다.

### 변환 처리에 대한 이해
> @DateTimeFormat 어노테이션을 사용하면 지정한 형식의 문자열을 LocalDateTime타입으로 변환해주는 것은 WebDataBinder이다.
> 

> 스프링 MVC는 요청 매핑 어노테이션 적용 메서드와 DispatcherServlet 사이를 연결하기 위해 RequestMappingHandlerAdapter 객체를 이용한다.
> 이 핸들러 어댑터 객체는 요청 파라미터와 커맨드 객체 사이의 변환 처리를 위해 WebDataBinder를 이용한다.

 - WebDataBinder는 커맨드 객체를 생성한다.
 - 커맨드 객체의 프로퍼티와 같은 이름을 갖는 요청 파라미터를 이용해서 프로퍼티 값을 생성한다.

![image](https://user-images.githubusercontent.com/55322459/183690720-e0a80e00-e38a-4829-a7ea-51bc2c354218.png)

 - WebDataBinder는 직접 타입을 변환하지 않고 Conversion Service에 그 역할을 위임한다.
 - 스프링 MVC를 위한 설정인 @EnableWebMvc 어노테이션을 사용하면 DefaultFormattingConversionService를 ConversionService로 사용한다.
 - DefaultFormattingConversionService는 int, long과 같은 기본 데이터 타입뿐만 아니라 @DateTimeFormat 어노테이션을 사용한 시간 관련 타입 변환 기능을 제공한다.
 
### @PathVariable을 이용한 경로 변수 처리
> http://localhost:8080/sp5-chap14/members/10

```java
package controller;

import org.springframework.beans.TypeMismatchException;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

import spring.Member;
import spring.MemberDao;
import spring.MemberNotFoundException;

@Controller
public class MemberDetailController {

	private MemberDao memberDao;

	public void setMemberDao(MemberDao memberDao) {
		this.memberDao = memberDao;
	}

	@GetMapping("/members/{id}")
	public String detail(@PathVariable("id") Long memId, Model model) {
		Member member = memberDao.selectById(memId);
		if (member == null) {
			throw new MemberNotFoundException();
		}
		model.addAttribute("member", member);
		return "member/memberDetail";
	}
}
```
 - 매핑 경로에 '{경로변수}'와 같이 중괄호로 둘러 쌓인 부분을 경로 변수라고 부른다.
 - "{경로변수}"에 해당하는 값은 같은 경로 변수 이름을 지정한 @PathVariable 파라미터에 전달된다.
 - "/member/{id}" 에서 {id}에 해당하는 부분의 경로 값을 @PathVariable("id") 어노테이션에 적용된 memId 파라미터에 전달한다.
 - memId 파라미터의 타입은 Long인데 이 경우 String 타입 값 "0"을 알맞게 Long타입으로 변환한다.

> ControllerConfig 설정 클래스에서 빈으로 등록한다.

```java
@Bean
	public MemberDetailController memberDetailController() {
		MemberDetailController controller = new MemberDetailController();
		controller.setMemberDao(memberDao);
		return controller;
	}
```
 - memId값이 존재 하면 해당 url에서 @PathVariable을 통해 전달받은 경로 변수값을 사용해서 회원 정보를 읽어와 뷰에 전달한다.

### 컨트롤러 익셉션 처리하기
> 타입 변환 실패에 대한 에러 화면을 노출하고 싶을때
> 
> @ExceptionHandler 어노테이션 사용 예시

```java
package controller;

import org.springframework.beans.TypeMismatchException;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

import spring.Member;
import spring.MemberDao;
import spring.MemberNotFoundException;

@Controller
public class MemberDetailController {

	private MemberDao memberDao;

	public void setMemberDao(MemberDao memberDao) {
		this.memberDao = memberDao;
	}

	@GetMapping("/members/{id}")
	public String detail(@PathVariable("id") Long memId, Model model) {
		Member member = memberDao.selectById(memId);
		if (member == null) {
			throw new MemberNotFoundException();
		}
		model.addAttribute("member", member);
		return "member/memberDetail";
	}

	@ExceptionHandler(TypeMismatchException.class)
	public String handleTypeMismatchException() {
		return "member/invalidId";
	}

	@ExceptionHandler(MemberNotFoundException.class)
	public String handleNotFoundException() {
		return "member/noMember";
	}

}

```
 - @ExceptionHandler의 값으로 TypeMismatchException.class를 주었다.
 - 이 익셉션은 경로 변수값의 타입이 올바르지 않을 때 발생한다.
 - 이 익셉션이 발생하면 에러 응답을 보내는 대신 handleTypeMismatchException() 메서드를 실행한다.
 - 동일하게 detail() 메서드를 실행하는 과정에서 MemberNotException이 발생하면 handleNotFoundException() 메서드를 이용해서 익셉션을 처리한다.

#### @ControllerAdvice를 이용한 공통 익셉션 처리
> 컨트롤러 클래스에서 @ExceptionHandler 어노테이션을 적용하면 해당 컨트롤러에서 발생한 익셉션만 처리한다.
> 
> 다수의 컨트롤러에서 동일 타입의 익셉션이 발생할 수도 있다. 이때 익셉션 처리 코드가 동일하다면 어떻게 해야 할까?
> 
> 각 컨트롤러 클래스마다 익셉션 처리 메서드를 구현하는 것은 불필요한 코드 중복을 발생 시킨다.
> 
> 여러 컨트롤러에서 동일하게 처리할 익셉션이 발생하면 @ControllerAdvice를 이용해서 중복을 제거할 수 있다.

#### @ControllerAdvice 어노테이션 사용 예시
```java
package common;

import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

@ControllerAdvice("spring")
public class CommonExceptionHandler {

	@ExceptionHandler(RuntimeException.class)
	public String handleRuntimeException() {
		return "error/commonException";
	}
}
```
 - @ControllerAdvice 어노테이션이 적용된 클래스는 지정한 범위의 컨트롤러에 공통으로 사용될 설정을 지정할 수 있다.
 - 위 코드는 "spring" 패키지와 그 하위 패키지에 속한 컨트롤러 클래스를 공통 기능을 정의했다.
 - spring패키지와 그 하위 패키지에 속한 컨트롤러에서 RuntimeException이 발생하면 handleRuntimeException() 메서드를 통해 익셉션을 처리한다.
 - @ControllerAdivce 적용 클래스가 동작하려면 해당 클래스를 스프링에 빈으로 등록해야 한다.

#### @ExceptionHandler 적용 메서드의 우선 순위
> @ControllerAdvice 클래스에 있는 @ExceptionHandler 메서드와 컨트롤러 클래스에 있는 @ExceptionHandler 메서드 중 컨트롤러 클래스에 적용된 @ExceptionHanndler메서드가 우선한다.

 - 같은 컨트롤러에 위치한 @ExceptionHandler 메서드 중에 해당 익셉션을 처리할 수 있는 메서드를 검색
 - 같은 클래스에 위치한 메서드가 익셉션을 처리할 수 없을 경우 @ControllerAdvice 클래스에 위치한 @ExceptionHandler 메서드를 검색

> @ControllerAdvice 어노테이션은 공통 설정을 적용할 컨트롤러 대상을 지정하기 위해 제공하는 속성

 - value basePackages : String[] : 공통 설정을 적용할 컨트롤러가 속하는 기준 패키지
 - annotations : Class<? extends Annotation>[] : 특정 어노테이션이 적용된 컨트롤러 대상
 - assignableTypes : Class<?>[] : 특정 타입 또는 그 하위 타입인 컨트롤러 대상

#### @ExceptionHandler 어노테이션 적용 메서드의 파라미터와 리턴 타입
> @ExceptionHandler 어노테이션을 붙인 메서드가 가지는 파라미터

 - HttpServletRequest, HttpServletResponse, HttpSession
 - Model
 - 익셉션
 
> 리턴 가능한 타입

 - ModelAndView
 - String (뷰 이름)
 - (@ResponseBody 어노테이션을 붙인 경우) 임의 객체 (Chap16참고)
 - ResponseEntity (Chap16참고)
