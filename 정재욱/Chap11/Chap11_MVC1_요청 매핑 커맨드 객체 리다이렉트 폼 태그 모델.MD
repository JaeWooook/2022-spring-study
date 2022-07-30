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
