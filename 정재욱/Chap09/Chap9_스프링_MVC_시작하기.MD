## Chapter9 스프링 MVC 시작하기
- src/main/webapp 폴더에는 HTML, CSS, JS, JSP등 웹어플리케이션을 구현하는데 필요한 코드가 위치한다.
- src/main/webapp/WEB-INF 에는 web.xml 파일이 위치한다.

> 서블릿 스펙에 따르면 WEB-INF 폴더의 하위 폴더로 lib 폴더와 classes 폴더를 생성하고 각각의 폴더에 필요한 jar파일과
> 컴파일 된 클래스 파일이 위치해야 한다. 하지만 메이븐이나 그레이들 프로젝트의 경우 필요한 jar 파일은 pom.xml/build.gradle파일의 의존을 통해
> 지정하고 컴파일된 결과는 target 폴더나 build 폴더에 위치한다. __때문에 WEB-INF 폴더 밑에 lib폴더나 classes 폴더를 생성할 필요가 없다.__ 

### 스프링 MVC를 위한 설정
 - 스프링 MVC의 주요 설정(HandlerMapping, ViewResolver 등)
 - 스프링의 DispatcherServlet 설정

#### 스프링 MVC 설정
```java
package config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.DefaultServletHandlerConfigurer;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.ViewResolverRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer {

	@Override
	public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
		configurer.enable();
	}

	@Override
	public void configureViewResolvers(ViewResolverRegistry registry) {
		registry.jsp("/WEB-INF/view/", ".jsp");
	}

}
```
 - @EnableWebMvc 어노테이션은 스프링 MVC 설정을 활성화한다. 스프링 MVC를 사용하는데 필요한 다양한 설정을 생성한다.
 - DispatcherServlet의 매핑 경로를 '/'로 주었을 때, JSP/HTML/CSS 등을 올바르게 처리하기 위한 설정을 추가한다. <- configureDefaultServletHandling의 내용이다.
 - JSP를 이용해서 컨트롤러의 실행 결과를 보여주기 위한 설정을 추가한다. <-configureViewResolvers의 내용이다.

#### @EnableWebMvc 어노테이션 설명
 - 내부적으로 다양한 빈 설정을 추가해준다. 이설정을 직접하려면 수십 줄에 가까운 코드를 작성해야 한다.
 - @EnableWebMvc 어노테이션이 스프링 MVC를 사용하는데 필요한 기본적인 구성을 설정해준다면, WebMvcConfigurer 인터페이스는 스프링 MVC의 개별 설정을 조정할 때 사용한다.

### 컨트롤러 구현
```java
package chap09;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;

@Controller
public class HelloController {

	@GetMapping("/hello")
	public String hello(Model model,
			@RequestParam(value = "name", required = false) String name) {
		model.addAttribute("greeting", "안녕하세요, " + name);
		return "hello";
	}
}

```

 - @Controller 어노테이션을 적용한 클래스는 스프링 MVC에서 컨트롤러로 사용한다.
 - @GetMapping 어노테이션은 메서드가 처리할 요청 경로를 지정한다.
 - Model 파라미터는 컨트롤러의 처리 결과를 뷰에 전달할 때 사용한다.
 - @RequestParam 어노테이션은 HTTP 요청 파라미터의 값을 메서드의 파라미터로 전달할 때 사용한다. 위의 경우 name 요청 파라미터의 값을 name 파라미터에 전달한다.
 - "greeting"이라는 모델 속성에 값을 설정한다.
 - 컨트롤러의 처리 결과를 보여줄 뷰 이름으로 "hello"를 사용한다.

> 컨트롤러란?
>
> 웹 요청을 처리하고 그 결과를 뷰에 전달하는 스프링 빈 객체이다.
> 
> 스프링 컨트롤러로 사용될 클래스는 @Controller 어노테이션을 붙여야하고, @GetMapping 어노테이션이나 @PostMapping 어노테이션과 같은 요청 매핑 어노테이션을 이용해서 처리할 경로를 지정해줘야한다.

#### @GetMapping 어노테이션과 요청 URL간의 관계 & @RequestParam 어노테이션과 요청 파라미터와의 관계
아래와 같은 url이 있다고 가정한다면,
> http://host:port/sp5-chap09/hello?name=bk

```java
@GetMapping("/hello")
public String hello(Model model, @RequestParam(value = "name", required = false) String name) {
    model.addAttribute("greeting", "안녕하세요," + name);
    return "hello";
        }
```

 - @GetMapping 어노테이션의 값은 서블릿 컨텍스트 경로를 기준으로한다.
 - @RequestParam 어노테이션은 HTTP 요청 파라미터를 메서드의 파라미터로 전달받을 수 있게 해준다.
 - @RequestParam 어노테이션의 value 속성은 HTTP 요청 파라미터의 이름을 지정하고 required 속성은 필수 여부를 지정한다.
 - 뷰 코드는 이 속성 이름을 사용해서 컨트롤러가 전달한 데이터에 접근하게 된다.
 - 실제로 뷰 이름에 해당하는 뷰 구현을 찾아주는 것은 ViewResolver가 처리한다.

#### 컨트롤러를 스프링 빈으로 등록
```java
package config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import chap09.HelloController;

@Configuration
public class ControllerConfig {

	@Bean
	public HelloController helloController() {
		return new HelloController();
	}

}
```

####뷰 이름과 JSP파일과의 연결 MvcConfig클래스
```java
	@Override
	public void configureViewResolvers(ViewResolverRegistry registry) {
		registry.jsp("/WEB-INF/view/", ".jsp");
	}
```
 - jsp() 메서드의 첫 번째 인자는 JSP 파일 경로를 찾을 때 사용할 접두어이다.
 - 두 번째 인자는 접미사이다.
 - 뷰 이름의 앞과 뒤에 각각 접두어와 접미사를 붙여서 최종적으로 사용할 JSP 파일의 경로를 결정한다.