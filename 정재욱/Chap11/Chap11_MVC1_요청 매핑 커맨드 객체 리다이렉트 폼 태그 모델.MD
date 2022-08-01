## MVC1 : 요청 매핑, 커맨드 객체, 리다이렉트, 폼 태그, 모델
### 요청 매핑 어노테이션을 이용한 경로 매핑
 - 특정 요청 URL을 처리할 코드
 - 처리 결과를 HTML과 같은 형식으로 응답하는 코드

#### 요청 매핑 어노테이션 종류
 - @RequestMapping, @GetMapping, @PostMapping 등이 있다.

```java
@Controller
public class RegisterController {

    @RequestMapping("/register/step1")
    public String handleStep1() {
        ...
    }

    @RequestMapping("/register/step2")
    public String handleStep2(
            ...
}

    @RequestMapping("/register/step3")
    public String handleStep3() {
        ...
    }

}
```
 - 위 코드에서 /register 라는 경로가 공통되는 경로라는것을 알 수 있다.

```java
@Controller
@RequestMapping("/register")
public class RegisterController {

    @RequestMapping("/step1")
    public String handleStep1() {
        ...
    }

    @RequestMapping("/step2")
    public String handleStep2(
        ...
    }

    @RequestMapping("/step3")
    public String handleStep3() {
        ...
    }

}
```
 - 컨트롤러 클래스 자체에 @RequestMapping 어노테이션을 이용해서 공통 부분의 경로를 값을 매핑한다.
 - 이후의 각각의 메서드에 나머지 경로를 값으로 갖는 요청 매핑 어노테이션을 적용할 수 있다.
 - 스프링 MVC는 클래스에 적용한 요청 매핑 어노테이션의 경로와 메서드에 적용한 매핑 어노테이션의 경로를 합쳐서 찾기 때문에 위 메서드의 경로는 "/step1"이 아닌, "/register/step1"이 된다.

```java
package config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import controller.RegisterController;
import spring.MemberRegisterService;
import survey.SurveyController;

@Configuration
public class ControllerConfig {

	@Bean
	public RegisterController registerController() {
		return RegisterController();
	}

}
```
 - 생성한 컨트롤러의 빈을 생성하기 위해 컨트롤러Config파일을 작성하고, 정리한다.

### GET과 POST구분 : @GetMapping, @PostMapping
주로 폼을 전송할 때 POST방식을 사용하는데 스프링 MVC는 별도 설정이 없으면 GET과 POST 방식에 상관없이 @RequestMapping에 지정한 경로와 일치하는 요청을 처리한다.

```java
	@PostMapping("/register/step2")
	public String handleStep2(
			@RequestParam(value = "agree", defaultValue = "false") Boolean agree,
			Model model) {
		if (!agree) {
			return "register/step1";
		}
		model.addAttribute("registerRequest", new RegisterRequest());
		return "register/step2";
	}
```
 - 위와 같이 포스트매핑을 사용하면 GET방식의 "/register/step2" 요청 경로는 __처리하지 않는다.__
 - POST방식의 "/register/step2" 요청 __경로만 처리한다.__

> @RequestMapping 어노테이션으로도 method속성을 정해줄수 있다.

```java
@Controller
public class RegisterController {

    @RequestMapping(value = "/register/step1", method = RequestMethod.GET)//get방식
    public String handleStep1() {
        ...
    }

    @RequestMapping(value = "/register/step2", method = RequestMethod.POST)//post방식
    public String handleStep2(
        ...
    }
}
```
 - @PutMapping, @DeleteMapping, @PatchMapping을 이용해서 HTTP의 GET, POST, PUT, DELETE, PATCH에 대한 매핑을 제한할 수 있다.

### 요청 파라미터 접근
```java
<form action="step2" method="post">
<label>
    <input type="checkbox" name="agree" value="true"> 약관 동의
</label>
<input type="submit" value="다음 단계" />
</form>
```
 - 약관의 동의 할경우 agree 요청 파라미터의 값을 POST방식으로 전송한다.
 - 따라서 폼에서 지정한 agree 요청 파라미터의 값을 이용해서 약관 동의 여부를 확인할 수 있다.

#### 컨트롤러 메서드에서 요청 파라미러틀 사용하는 방법 1
> HttpServletRequest를 직접 이용하는 것
> 
> HttpServletRequest 타입을 사용하고 HttpServletRequest의 getParameter() 메서드를 이용해서 파라미터의 값을 구하면 된다.

```java
package controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;

import spring.RegisterRequest;

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

#### 컨트롤러 메서드에서 요청 파라미러틀 사용하는 방법 2
```java
package controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

import spring.RegisterRequest;

@Controller
public class RegisterController {

	@RequestMapping("/register/step1")
	public String handleStep1() {
		return "register/step1";
	}

	@PostMapping("/register/step2")
	public String handleStep2(
			@RequestParam(value = "agree", defaultValue = "false") Boolean agree) {
		if (!agree) {
			return "register/step1";
		}
		return "register/step2";
	}
}

```
 - @RequestParam 어노테이션을 사용하는 것이다. 요청 파라미터 개수가 몇개 안되면 이 어노테이션을 사용해서 간단하게 요청 파라미터의 값을 구할 수 있다.
 - 위 코드는 agree 요청 파라미터의 값을 읽어와 agreeVal 파라미터에 할당한다.
 - 요청 파라미터의 값이 없으면 "false" 문자열을 값으로 사용한다.

> @RequestParam 어노테이션의 속성

 - value , String타입 : HTTP 요청 파라미터의 이름을 지정한다.
 - required , boolean타입 : 필수 여부를 지정한다. 이 값이 true이면서 해당 요청 파라미터에 값이 없으면 익셉션이 발생한다. 기본값은 true이다.
 - defaultValue , String타입 : 요청 파라미터가 값이 없을 때 사용할 문자열 값을 지정한다. 기본값은 없다.

> @RequestParam 어노테이션을 사용한 코드를 보면 다음과 같이 agreeVal 파라미터의 타입이 Boolean이다.

```java
@RequestParam(value = "agree", defaultValue = "false") Boolean agreeVal
```
 - 스프링 MVC는 파라미터 타입에 맞게 String 값을 변환해준다. 위 코드는 agree 요청 파라미터의 값을 읽어와 Boolean 타입으로 변환해서 agreeVal 파라미터에 전달한다.
 - Boolean 타입 외에 int, long, Integer, Long 등 기본 데이터 타입과 래퍼 타입에 대한 변환을 지원한다.
 
> 
```java
	@PostMapping("/register/step2")
	public String handleStep2(
			@RequestParam(value = "agree", defaultValue = "false") Boolean agree) {
		if (!agree) {
			return "register/step1";
		}
		return "register/step2";
	}
```
 - 위 코드를 통해 agree에 true값이라면 "/step2"을 리턴해 회원 정보 입력 폼을 보여준다.
 - fasle값이라면 "/step1"을 리턴해 다시 약관 동의 폼을 보여준다.

### 리다이렉트 처리
```java
	@GetMapping("/register/step2")
	public String handleStep2Get() {
		return "redirect:/register/step1";
	}
```
 - 기존의 코드는 @PostMapping이기 때문에 url을 입력하는 방식으로는 접근할 수 없다.
 - HTTP Status 405 에러를 띄워주는 방식 보다는 리다이렉트를 이용한 페이지 이동을 돕기위해
 - @GetMapping으로 접근하였을 때 "redirect:/register/step1" 페이지로 이동 시켜준다.
 - 리다이렉트를 실행 시켜주는 방법으로는 redirect:경로를 뷰 이름으로 리턴하면 된다.
 - redirect:/register/step1을 사용하면 이동 경로가 "/"로 시작하므로 실제 리다이렉트할 경로는 웹어플리케이션 경로인 "/sp5-chap11"과 "/register/step1"을 연결한 /sp5-chap11/register/step1가 된다.
 - "/"로 시작하지 않으면 현재 경로를 기준으로 상대 경로를 사용한다. 예를 들어 "redirect:step1"을 리턴 한다면 요청 경로인 "http://localhost:8080/sp5-chap11/register/step2"를 기준으로 상대 경로인 "http://localhost:8080/sp5-chap11/register/step1"을 리다이렉트 경로로 사용한다.
 - redirect:http://localhost:8080/sp5-chap11/register/step1과 같이 완전한 URL을 사용하면 해당 경로로 리다이렉트한다.

### 커맨드 객체를 이용해서 요청 파라미터 사용하기
```java
@PostMapping("/register/step3")
	public String handleStep3(HttpServletRequest request) {
		String email = request.getParameter("email");
        String name = request.getParameter("name");
        String password = request.getParameter("password");
        String confirmPassword = request.getParameter("confirmPassword");
        
        RegisterRequest regReq = new RegisterRequest();
        regReq.setEmail(email);
        regReq.setName(name);
        ...
	}
```
 - 위 코드가 올바르게 동작하지만, 요청 파라미터 개수가 증가할 때마다 handleStep3()메서드의 코드 길이도 함께 길어지는 단점이 있다.

> 스프링은 이러한 불편함을 줄이기 위해 요청 파라미터의 값을 커맨드(command) 객체에 담아주는 기능을 제공한다.
> 
> 예를 들어 이름이 name인 요청 파라미터의 값을 커맨드 객체의 setName() 메서드를 사용해서 커맨드 객체에 전달하는 기능을 제공한다.
> 요청 파라미터의 값을 전달받을 수 있는 세터 메서드를 포함하는 객체를 커맨드 객체로 사용하면 된다.

```java
@PostMapping("/register/step3")
	public String handleStep3(RegisterRequest regReq) {
        ...
	}
```
 - RegisterRequest 클래스에는 setEmail(), setName(), setPassword(), setConfirmPassword() 메서드가 있다.
 - 스프링은 메서드를 사용해서 email, name, password, confirmPassword 요청 파라미터의 값을 커맨드 객체에 복사한 뒤 regReq 파라미터로 전달한다.
 - 스프링 MVC가 handleStep3() 메서드에 전달할 RegisterRequest 객체를 생성하고 그 객체의 세터 메서드를 이용해서 일치하는 요청 파라미터의 값을 전달한다.

```java
    private MemberRegisterService memberRegisterService;
    
    public void setMemberRegisterService(
            MemberRegisterService memberRegisterService) {
            this.memberRegisterService = memberRegisterService;
            }

	@PostMapping("/register/step3")
	public String handleStep3(RegisterRequest regReq) {
		try {
			memberRegisterService.regist(regReq);
			return "register/step3";
		} catch (DuplicateMemberException ex) {
			return "register/step2";
		}
	}
```
 - MemberRegisterService를 이용해서 회원 가입을 처리한다.
 - 회원가입에 성공하면 뷰 이름으로 "register/step3"를 리턴하고, 이미 동일한 이메일 주소를 가진 회원 데이터가 존재하면 뷰 이름으로 "register/step2"를 리턴한다.

```java
	@Autowired
	private MemberRegisterService memberRegSvc;

	@Bean
	public RegisterController registerController() {
		RegisterController controller = new RegisterController();
		controller.setMemberRegisterService(memberRegSvc);
		return controller;
	}
```
 - RegisterController 클래스는 MemberRegisterService 타입의 빈을 의존하므로 ControllerConfig.java 파일에 의존 주입을 설정한다.

### 뷰 JSP 코드에서 커맨드 객체 사용하기
스프링 MVC는 커맨드 객체의 (첫 글자를 소문자로 바꾼) 클래스 이름과 동일한 속성 이름을 사용해서 커맨드 객체를 뷰에 전달한다.

```java
@PostMapping("/register/step3")
	public String handleStep3(RegisterRequest regReq) {
        ...
	}
```

```html
<p><strong>${registerRequest.name}님</strong> 
        회원 가입을 완료했습니다.</p>
```
 - 커맨드 객체의 클래스 이름이 RegisterRequest인 경우 JSP 코드는 registerRequest라는 이름을 사용해서 커맨드 객체에 접근할 수 있다.

### @ModelAttribute 어노테이션으로 커맨드 객체 속성 이름 변경
```java
@PostMapping("/register/step3")
	public String handleStep3(@ModelAttribute("formData") RegisterRequest regReq) {
        ...
	}
```
 - 커맨드 객체에 접근할 때 사용할 속성 이름을 변경하고 싶다면 커맨드 객체로 사용할 파라미터에 @ModelAttribute 어노테이션을 적용한다.
 - 위와 같은 코드로 작성하면 모델에서 사용할 속성 이름을 값으로 설정해준다.
 - 뷰 코드에서 "formData"라는 이름으로 커맨드 객체에 접근할 수 있다.

### 커맨드 객체와 스프링 폼 연동
```html
<input type="text" name="email" id="email" value="${registerRequest.email}">
```
 - 회원 정보에 대한 입력 폼에서 다시 페이지를 띄울 때 여태 입력했던 값이 날라가지 않고, 커맨드 객체의 값을 폼에 채워주면 다시 입력하는 불편함을 해소한다.
 
### 컨트롤러 구현 없는 경로 매핑
 > 첫 화면은 단순히 환영 문구와 회원 가입을 위한 링크만 제공한다면, 이를 위한 컨트롤러 클래스는 특별히 처리할 것이 없기 때문에 단순히 뷰 이름만 리턴하도록 구현된다.
> 
> 이러한 특별한 로직이 없는 컨트롤러 클래스를 만드는것은 별로 바람직하지 못하다.
> 

```java
@Override
public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/main").setViewName("main");
        }
```
 - WebMvcConfigurer 인터페이스의 addViewController() 메서드를 사용하면 이런 코드를 없앨 수 있다.
 - addViewController() 메서드를 오버라이딩해서 재정의 하면 컨트롤러 구현없이 요청 경로와 뷰 이름을 연결할 수 있다.

### 주요 에러 발생 상황
#### 요청 매핑 어노테이션과 관련된 주요 익셉션
 > 404 에러

 - 요청 경로를 처리할 컨트롤러가 존재하지 않거나 Web MvcConfigurer를 이용한 설정이 없을 때 발생한다.
 - 요청 경로가 올바른지
 - 컨트롤러 설정한 경로가 올바른지
 - 컨트롤러 클래스를 빈으로 등록했는지
 - 컨트롤러 클래스에 @Controller 어노테이션을 적용했는지
 - 뷰 이름에 해당하는 JSP파일이 없어도 404에러가 발생하지만 JSP파일 경로는 출력된다는 차이가 있다.

 > 405 에러
 - 지원하지 않는 전송 방식(method)를 사용한 경우 405에러가 발생한다.
 - EX) POST방식만 처리하는 요청 경로를 GET방식으로 연결하면 405에러가 발생한다.

#### @RequestParam이나 커맨드 객체와 관련된 주요 익셉션
 > 400 에러

```java
@PostMapping("/register/step2")
public String handleStep2(
        //필수로 존재해야하고 기본값은 없다.
        @RequestParam("agree") Boolean agree, Model model) {
        ...
        }
```
 - @RequestParam 어노테이션을 필수로 설정하고 기본값을 지정하지 않았다
 - @RequestParam 어노테이션을 처리하는 과정에서 필수인 "agree" 파라미터가 존재하지 않는다는 익셉션이 발생하게 된다.
 - 스프링 MVC는 이 익셉션이 발생하면 400에러를 응답으로 전송한다.
 - 400에러는 요청 파라미터의 값을 커맨드 객체에 복사하는 과정에서도 동일하게 발생한다.
 - 커맨드 객체의 프로퍼티가 int 타입인데 요청 파라미터의 값이 "abc"라면, "abc"를 int타입으로 변환할 수 없기 때문에 400 에러가 발생한다.
 - "true" -> "true1"로 변경하면 true1값은 boolean으로 변경할 수 없기 때문에 400 에러가 발생한다.

### 커맨드 객체: 중첩 콜렉션 프로퍼티
> Respondent.java

```java
package survey;

public class Respondent {

	private int age;
	private String location;

	public int getAge() {
		return age;
	}

	public void setAge(int age) {
		this.age = age;
	}

	public String getLocation() {
		return location;
	}

	public void setLocation(String location) {
		this.location = location;
	}

}
```
> AnsweredData.java

```java
package survey;

import java.util.List;

public class AnsweredData {

	private List<String> responses;
	private Respondent res;

	public List<String> getResponses() {
		return responses;
	}

	public void setResponses(List<String> responses) {
		this.responses = responses;
	}

	public Respondent getRes() {
		return res;
	}

	public void setRes(Respondent res) {
		this.res = res;
	}

}
```
 - Respondent 클래스는 응답자 정보를 담는다.
 - AnsweredData 클래스는 설문 항목에 대한 답변과 응답자 정보를 함께 담는다.
 - AnsweredData 클래스는 리스트 타입의 프로퍼티 존재
 - responses 프로퍼티는 String 타입의 값을 갖는 List 콜렉션이다.
 - 중첩 프로퍼티를 갖는다.
 - res 프로퍼티는 Respondent 타입이며, res 프로퍼티는 다시 age와 location 프로퍼티를 갖는다.
 - 이를 중첩된 형식으로 표시하면 res.age 프로퍼티나 res.location프로퍼티로 표현할 수 있다.

#### 스프링 MVC 커맨드 객체가 리스트타입 & 중첩 프로퍼티를 가진 경우
> 요청 파라미터의 값을 알맞게 커맨드 객체에 설정해주는 기능을 제공, 규칙

 - HTTP 요청 파라미터 이름이 "프로퍼티이름[인덱스]" 형식이면 List 타입 프로퍼티의 값 목록으로 처리한다.
 - HTTP 요청 파라미터 이름이 "프로퍼티이름.프로퍼티이름"과 같은 형식이면 중첩 프로퍼티 값을 처리한다.

ex) 이름이 responses이고 List 타입인 프로퍼티를 위한 요청 파라미터의 이름으로 "responses[0]", "responses[1]"을 사용하면 각각의 0번 인덱스와 1번 인덱스의 값으로 사용된다.
중첩 프로퍼티의 경우 파라미터 이름을 "res.name"으로 지정하면 commandObj.getRes().setName(request.getParameter("res.name"));과 같은 방식으로 커맨드 객체에 파라미터의 값을 설정한다.

### Model을 통해 컨트롤러에서 뷰에 데이터 전달하기

#### ModelAndView를 통한 뷰 선택과 모델 전달

#### GET방식과 POST방식에 동일 이름 커맨드 객체 사용하기
