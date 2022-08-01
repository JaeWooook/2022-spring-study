# Chap10 스프링 MVC 프레임워크  동작 방식
## 스프링 MVC 핵심 구성 요소
![](https://velog.velcdn.com/images/yh_lee/post/32d37cb6-78ae-456b-b1c2-aeee53ac472a/image.png)
<<spring bean\>>이라고 표시한 것은 스프링 빈으로 등록해야 하는 것을 의미한다. 회색 배경을 가진 요소는 개발자가 구현해햐 하는 요소이다. 

DispatcherServlet은 모든 연결을 담당한다. 웹 브라우저로부터 요청이 들어오면 DispatcherServlet은 요청을 처리하기 위한 컨트롤러 객체를 검색한다. 이때 직접 검색하지 않고 HandlerMapping 빈 객체에게 컨트롤러 검색을 요청(2)한다. 

HandlerMapping은 클라이언트의 요청 경로를 이용해서 컨트롤러 빈 객체를 dispatcherServlet에 전달한다. 예를 들어 웹 요청 경로가 '/hello'라면 등록된 컨트롤러 빈 중에서 '/hello' 요청 경로를 처리할 컨트롤러를 리턴한다.

@Controller, Controller 인터페이스, HttpRequestHandler 인터페이스를 동일한 방식으로 처리하기 위해 중간에 사용되는 것이 HandlerApdapter이다. 

DispatcherServlet은 HandlerMapping이 찾아준 컨트롤러 객체를 처리할 수 있는 HandlerAdapter 빈에게 요청 처리를 위임(3)한다. HandlerAdapter는 컨트롤러의 알맞은 메서드를 호출해서 요청을 처리(4,5)하고 결과를 DispatcherServlet에 리턴(6)한다. 이때 HandlerAdapter는 컨트롤러의 처리 결과를 ModelAndView 객체로 변환해서 DispatcherServlet에 리턴한다. 

DispatcherServlet은 결과를 보여줄 뷰를 찾기 위해 ViewResolver 빈 객체를 사용(7)한다. 

DispatcherServlet은 ViewResolver가 리턴한 View 객체에게 응답 결과 생성을 요청(8)한다. JSP를 사용하는 경우 View 객체는 JSP를 실행함으로써 웹 브라우저에게 전송할 응답 결과를 생성하고 끝난다.

## 컨트롤러와 핸들러 
클라이언트의 요청을 실제로 처리하는 것은 컨트롤러이다. 컨트롤러를 찾아주는 객체는 ControllerMapping이 아니라 HandlerMapping인 이유는 무엇일까?

DispatcherServlet 입장에서 클라이언트 요청을 처리하는 객체의 타입이 반드시 @Controller를 적용한 클래스일 필요 없다. 스프링이 제공하는 타입 중 HttpRequestHandler 타입도 있다. 이런 이유로 스프링 MVC는 웹 요청을 처리하는 객체를 Handler라고 표현한다.

## DispatcherServlet과 스프링 컨테이너
DispatcherServlet은 전달받은 설정 파일을 이용해서 스프링 컨테이너를 생성한다. HandlerMapping, HandlerAdapter, 컨트롤러, ViewResolver 등의 빈을 DispatcherServlet이 생성한 스프링 컨테이너에서 구한다.

## @Controller를 위한 HandlerMapping과 HandlerAdapter
DispatcherServlet은 웹 브라우저의 요청을 처리할 핸들러 객체를 찾기 위해 HandlerMapping을 사용하고, 핸들러를 실행하기 위해 HandlerAdpater를 사용한다.
```java
@Controller 
public class HelloController{
	@GetMapping("/hello")
    public String hello(Model model, @RequestParam(value="name",required=false) String name){
    	model.addAttribute("greeting","HI, "+name);
        return "hello";
    }
}	
```
RequestMappingHandlerAdapter는 컨트롤러 메서드 결과 값이 String 타입이면 해당 값을 뷰 이름으로 갖는 ModelAndView 객체를 생성해서 DispatcherServlet에 리턴한다. 

## WebMvcVonfigurer 인터페이스와 설정
@EnableWebMvc 어노테이션을 사용하면 @Controller 어노테이션을 붙인 컨트롤러를 위한 설정을 생성한다. 
```java
@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer{
	...
}
```
@EnableWebMvc 어노테이션을 사용하면 WebMvcConfigurer 타입인 빈 객체의 메서드를 호출해서 MVC 설정을 추가한다. 예를 들어 ViewResolver 설정을 추가하기 위해 WebMvcConfigurer 타입인 빈 객체의 configureViewResolvers() 메서드를 호출한다. 따라서 WebMvcConfigurer 인터페이스를 구현한 설정 클래스는 configureViewResolvers() 메서드를 재정의해서 뷰 설정을 추가하면 된다. 스프링5는 자바8부터 지원하는 디폴트 메서드를 사용해서 WebMvcConfigurer 인터페이스의 메서드에 기본 구현을 제공하고 있다. 
