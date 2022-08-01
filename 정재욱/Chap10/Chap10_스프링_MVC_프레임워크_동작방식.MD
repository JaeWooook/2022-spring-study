## Chapter10 스프링 MVC 프레임워크 동작방식
### 스프링 MVC 핵심 구성 요소
![image](https://user-images.githubusercontent.com/55322459/180656680-74553413-5cc2-419c-8ae5-d43135a603f6.png)
 - 직접 개발을 하는 구성요소는 JSP, Controller를 의미한다.
 - 컨트롤러 구성 요소는 개발자가 직접 구현해야 하고 스프링 빈으로 등록해야 한다.
 - 중앙에 위치한 DispatcherServlet은 모든 연결을 담당한다.
 - 웹 브라우저로부터 요청이 들어오면 DispatcherServlet은 그 요청을 처리하기 위한 컨트롤러 객체를 검색한다.
 - 이때 DispatcherServlet은 직접 컨트롤러를 검색하지 않고 HandlerMapping이라는 빈 객체에게 컨트롤러 검색을 요청한다.
 - HandlerMapping은 클라이언트의 요청 경로를 이용해서 이를 처리할 컨트롤러 빈 객체를 DispatcherServlet에 전달한다.
 - ex) 웹 요청 경로가 '/hello'라면 등록된 컨트롤러 빈 중에서 '/hello'요청 경로를 처리할 컨트롤러를 리턴한다.
 - 컨트롤러 객체를 DispatcherServlet이 전달받았다고 해서 바로 컨트롤러 객체의 메서드를 실행할 수 있는 것은 아니다.
 - DispatcherServlet은 @Controller 어노테이션을 이용해서 구현한 컨트롤러뿐만 아니라 Controller 인터페이스를 구현한 컨트롤러, HttpRequestHanlder 인터페이스를 구현한 클래스를 동일한 방식으로 실행할 수 있다.
 - __@Controller, Controller 인터페이스, HttpRequestHandler 인터페이스를 동일한 방식으로 처리하기 위해 중간에 사용되는 것이 HandlerAdapter 빈이다.__
 - DispatcherServlet은  HandlerMapping이 찾아준 컨트롤러 객체를 처리할 수 있는 HandlerAdapter 빈에게 요청 처리를 위임한다.
 - HandlerAdapter는 컨트롤러의 알맞은 메서드를 호출해서 요청을 처리하고 그 결과를 DispatcherServlet에 리턴한다.
 - HandlerAdapter는 컨트롤러의 처리 결과를 ModelAndView라는 객체로 변환해서 DispatcherServlet에 리턴한다.
 - ViewResolver는 DispatcherServlet의 결과를 보여줄 뷰를 찾기 위해 빈 객체를 사용한다.
 - ViewResolver는 ModelAndView에 컨트롤러가 리턴한 담고있는 뷰 이름을 해당하는 View 객체를 찾거나 생성해 리턴해준다.
 - 응답을 생성하기 위해 JSP를 사용하는 ViewResolver는 매번 새로운 View 객체를 생성해서 DispatcherServlet에 리턴한다.
 - DispatcherServlet은 ViewResolver가 리턴한 View 객체에게 응답 결과 생성을 요청한다.

#### 컨트롤러와 핸들러
 - 클라이언트의 요청을 실제로 처리하는 것은 컨트롤러이고, DispatcherServlet은 클라이언트의 요청을 전달받는 창구 역할을 한다.
 - DispatcherServlet은 클라이언트의 요청을 처리할 컨트롤러를 찾기위해 HandlerMapping을 사용한다.
 - DispatcherServlet 입장에서는 클라이언트 요청을 처리하는 객체의 타입이 반드시 @Controller를 적용한 클래스일 필요는 없다.
 - __MVC는 웹 요청을 실제로 처리하는 객체를 핸들러라고 표현한다.__
 - @Controller 적용 객체나 Controller 인터페이스를 구현한 객체는 모두 스프링 MVC입장에서는 핸들러가 된다. 특정 요청 경로를 처리해주는 핸들러를 찾아주는 객체를 HandlerMapping이라고 부른다.
 - __DispatcherServlet은 핸들러 객체의 실제 타입에 상관없이 실행 결과를 ModelandView라는 타입으로만 받을 수 있으면 된다.__
 - __그런데,__ 핸들러의 실제 구현 타입에 따라 ModelAndView를 리턴하는 객체도(Controller 인터페이스를 구현한 클래스의 객체)있고, 그렇지 않은 객체도 있다.
 - 따라서 핸들러의 처리 결과를 ModelAndView로 변환해주는 객체가 필요하며 HandlerAdapter가 이 변환을 처리해준다.
 - __핸들러 객체의 실제 타입마다 그에 알맞은 HandlerMapping과 HandlerAdapter가 존재하기 때문에, 사용할 핸들러의 종류에 따라 해당 HandlerMapping과 HandlerAdapter를 스프링 빈으로 등록해야 한다.__
 - __스프링이 제공하는 설정 기능을 사용하면 이 두종류의 빈을 직접 등록하지 않아도 된다.__

### DispatcherServlet과 스프링 컨테이너
![image](https://user-images.githubusercontent.com/55322459/180784364-e793c0de-7b18-4585-8022-5a24e87f1c8d.png)

 - DispatcherServlet은 전달받은 설정 파일을 이용해서 스프링 컨테이너를 생성한다.
 - HandlerMapping, HandlerAdapter, 컨트롤러, ViewResolver 등의 빈은 DispatcherServlet이 생성한 스프링 컨테이너에서 구한다.
 - 따라서 DispatcherServlet이 사용하는 설정 파일에 이들 빈에 대한 정의가 포함되어 있어야 한다.

### Controller를 위한 HandlerMapping과 HandlerAdapter
 - DispatcherServlet은 웹 브라우저의 요청을 처리할 핸들러 객체를 찾기 위해 HanlderMapping을 사용하고, 핸들러를 실행하기 위해 HandlerAdapter를 사용한다.
 - DispatcherServlet은 스프링 컨테이너에서 HandlerMapping과 HandlerAdapter타입의 빈을 사용하므로 핸들러에 알맞은 HandlerMapping빈과 HandlerAdapter빈이 스프링 설정에 등록되어 있어야 한다.

#### RequestMappingHandlerMapping
 - RequestMappingHandlerMapping은 @Controller 어노테이션에 적용된 객체의 요청 매핑 어노테이션(@GetMapping)값을 이용해서 웹 브라우저의 요청을 처리할 컨트롤러를 찾는다.

#### RequestMappingHandlerAdapter
 - RequestMappingHandlerAdapter는 컨트롤러의 메서드를 알맞게 실행하고 그 결과를 ModelAndView 객체로 변환해서 DispatcherServlet에 리턴한다.
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
 - RequestMappingHandlerAdapter 클래스는 "/hello" 요청 경로에 대해 hello() 메서드를 호출한다.
 - 이때 Model 객체를 생성해서 첫 번째 파라미터로 전달한다. 비슷하게 이름이 "name"인 HTTP 요청 파라미터의 값을 두 번째 파라미터로 전달한다.
 - RequestMappingHandlerAdapter는 컨트롤러 메서드 결과 값이 String 타입이면 해당 값을 뷰 이름으로 갖는 ModelAndView 객체를 생성해서 DispatcherServlet에 리턴한다.
 - 첫 번째 파라미터로 전달한 Model 객체에 보관된 값도 ModelAndView에 함께 전달한다.

### WebMvcConfigurer 인터페이스와 설정
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
 - @EnableWebMvc 어노테이션을 사용하면 @Controller 어노테이션을 붙인 컨트롤러를 위한 설정을 생성한다.
 - @EnableWebMvc 어노테이션을 사용하면 WebMvcConfigurer 타입의 빈을 이용해서 MVC 설정을 추가로 생성한다.
 - WebMvcConfigurer 인터페이스를 상속하고 있으므로 MvcConfig클래스는 WebMvcConfigurer 타입의 빈이 된다.
 - @EnableWebMvc 어노테이션을 사용하면 WebMvcConfigurer 타입인 빈 객체의 메서드를 호출해서 MVC 설정을 추가한다.
 - 예를 들어, ViewResolver 설정을 추가하기 위해 WebMvcConfigurer 타입의 빈 객체의 configureViewResolvers()메서드를 호출한다.

 > WebMvcConfigurer 인터페이스에 일부 구현되어있는 메서드들이 있기 때문에
> 해당 메서드들을 가져다가 구현해서 사용하면 된다.

### JSP를 위한 ViewResolver
```java
	@Override
	public void configureViewResolvers(ViewResolverRegistry registry) {
		registry.jsp("/WEB-INF/view/", ".jsp");
	}
```
 - WebMvcConfigurer 인터페이스에 정의된 configureViewResolvers() 메서드는 ViewResolverRegistry 타입의 registry 파라미터를 갖는다.
 - ViewResolverRegistry#jsp() 메서드를 사용하면 JSP를 위한 ViewResolver를 설정할 수 있다.

```java
import o.s.w.servlet.view.InternalResourceViewResolver;

	@Bean
	public ViewResolver viewResolver() {
		InternalResourceViewResolver vr = new InternalResourceViewResolver();
		vr.setPrefix("/WEB-INF/view/");
		vr.setSuffix(".jsp");
		return vr;
	}
```
 - 컨트롤러의 실행 결과를 받은 DispatcherServlet은 ViewResolver에게 뷰 이름에 해당하는 View 객체를 요청한다.
 - 이때 InterenalResourceViewResolver는 "prefix+뷰이름+suffix"에 해당하는 경로를 뷰 코드로 사용하는 InternalResourceView 타입의 View객체를 리턴한다.
 - DispatcherServlet이 InternalResourceView 객체에 응답 생성을 요청하면 InternalResourceView 객체는 경로에 지정한 JSP 코드를 실행해서 응답 결과를 생성한다.
 - DispatcherServlet은 컨트롤러의 실행 결과를 HandlerAdapter를 통해서 ModelAndView 형태로 받는다.

```java
package chap10;

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
 - 이 경우 DispatcherServlet은 View 객체에 응답 생성을 요청할 때 greeting 키를 갖는 Map 객체를 View 객체에 전달한다.
 - View 객체는 전달받은 Map 객체에 담긴 값을 이용해서 알맞은 응답 결과를 출력한다.
 - InternalResourceView는 Map 객체에 담겨 있는 키 값을 request.setAttribute()를 이용해서 request의 속성에 저장한다.

### 디폴트 핸들러와 HandlerMapping의 우선순위
```xml
<servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```
 - 매핑 경로가 '/'인 경우 .jsp로 끝나는 요청을 제외한 모든 요청을 DispatcherServlet이 처리한다.
 - 즉, /index.html이나, /css/bootstrap.css와 같이 확장자가 .jsp가 아닌 모든 요청을 DispatcherServlet이 처리하게 된다.
 - __그런데__ @EnableWebMvc 어노테이션이 등록하는 HandlerMapping은 @Controller 어노테이션을 적용한 빈 객체가 처리할 수 있는 요청 경로만 대응할 수 있다.
 - /index.html, /css/bootstrap.css와 같은 요청은 처리할 수 있는 컨트롤러 객체를 찾지 못해 DispatcherServlet은 404응답을 전송한다.

```java
	@Override
	public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
		configurer.enable();
	}
```
 - DefaultServletHandlerConfigurer#enable() 메서드는 두 빈 객체를 추가한다.
 - DefaultServletHttpRequestHandler, SimpleUrlHandlerMapping
 - DefaultServletHttpRequestHandler는 클라이언트의 모든 요청을 WAS(웹 어플리케이션 서버, 톰캣이나 웹로직 등)가 제공하는 디폴트 서블릿에 전달한다.
 - 예를 들어 "/index.html"에 대한 처리를 DefaultServletHttpRequestHandler에 요청하면 이 요청을 다시 디폴트 서블릿에 전달해서 처리하도록 한다.
 - 그리고 SimpleUrlHandlerMapping을 이용해서 모든 경로 ("/**")를 DefaultServletHttpRequestHandler를 이용 해서 처리하도록 설정한다.
 - @EnableWebMvc 어노테이션이 등록하는 RequestMappingHandlerMapping의 적용 우선순위가 DefaultServletHandlerConfigurer#enable() 메서드가 등록하는 SimpleUrlHandlerMapping의 우선순위보다 높다.
 - 때문에 웹 브라우저의 요청이 들어오면 DispatcherServlet은 아래 와 같은 방식으로 요청을 처리한다.
   1. RequestMappingHandlerMapping을 사용해서 요청을 처리할 핸들러를 검색한다.
      - 존재하면 해당 컨트롤러를 이용해서 요청을 처리한다.
   2. 존재하지 않으면 SimpleUrlHandlerMapping을 사용해서 요청을 처리할 핸들러를 검색한다.
      - DefaultServletHandlerConfigurer#enable() 메서드가 등록한 SimpleUrlHandlerMapping은 "/**" 경로(모든 경로)에 대해 DefaultServletHttpRequestHandler를 리턴한다.
      - DispatcherServlet은 DefaultServletHttpRequestHandler에 처리를 요청한다.
      - DefaultServletHttpRequestHandler는 디폴트 서블릿에 처리를 위임한다.
 - 예를 들어 "/index.html" 경로로 요청이 들어오면 1번 과정에서는 해당 컨트롤러를 찾지 못하므로 2번 과정을 통해 디폴트 서블릿이 /index.html 요청을 처리하게 된다.

### 정리
#### DispatcherServlet
> 웹 브라우저의 요청을 받기 위한 창구 역할을 하고, 다른 주요 구성 요소들을 이용해서 요청 흐름을 제어하는 역할을 한다.
#### HandlerMapping
> 클라이언트의 요청을 처리할 핸들러 객체를 찾아준다. 핸들러(커맨드) 객체는 클라이언트의 요청을 실제로 처리한 뒤 뷰 정보와 모델을 산정한다.
#### HandlerAdapter
> DispatcherServlet과 핸들러 객체 사이의 변환을 알맞게 처리해준다.
#### ViewResolver
> 요청 처리 결과를 생성할 View를 찾아주고 View는 최종적으로 클라이언트에 응답을 생성해서 전달한다.