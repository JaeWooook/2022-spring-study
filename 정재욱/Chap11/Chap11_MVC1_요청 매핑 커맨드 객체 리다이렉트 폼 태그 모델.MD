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
