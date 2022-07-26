### 스프링 MVC를 위한 설정
MvcConfig

@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer {
    
    //DispatcherServlet의 매핑 경로를 '/'로 주었을때, JSP/HTML/CSS 등을 올바르게 처리하기 위한 설정
    @Override
    public void configureDefaultServletHandling(
            DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    //JSP를 이용해서 컨트롤러의 실행 결과를 보여주기 위한ㅅ ㅓㄹ정
    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.jsp("/WEB-INF/view", ".jsp");
    }
}
@EnableWebMvc 애노테이션은 스프링 MVC 설정을 활성화한다.

WebMvcConfigurer 인터페이스는 스프링 MVC의 개별 설정을 조정할 때 사용한다.

## web.xml 파일에 DispatcherServlet 설정

DispatcherServlet은 초기화 과정에서 contextConfiguration 초기화 파라미터에 지정한 설정 파일을 이용해서 스프링 컨테이너를 초기화한다.

즉 여기서는 MvcConfig 클래스와 ControllerConfig 클래스를 이용해서 스프링 컨테이너를 생성한다.
```
HelloController

@Controller
public class HelloController {

    @GetMapping("/hello")
    public String hello(Model model, @RequestParam(value = "name", required = false) String name) {
        model.addAttribute("greeting", "안녕하세요, " + name);
        return "hello";
    }
}
```
### 컨트롤러
스프링 MVC 프레임워크에서 컨트롤러란 간단히 설명하면 웹 요청을 처리하고 그 결과를 뷰에 전달하는 스프링 빈 객체이다.

@GetMapping 애노테이션은 메서드가 처리할 요청 경로를 지정한다.

Model 파라미터는 컨트롤러의 처리 결과를 뷰에 전달할 때 사용한다.

@RequestParam 애노테이션은 HTTP 요청 파라미터의 값을 메서드의 파라미터로 전달할 때 사용한다

"hello"를 리턴하는데 이것은 뷰 이름이다. 뷰 일므은 논리적인 이름이며 실제 뷰 이름에 해당하는 뷰 구현을 찾아주는 것은 ViewResolver가 처리한다.
```
ControllerConfig

@Configuration
public class Controller\Config {

    @Bean
    public HelloController helloController() {
        return new HelloController();
    }
}
hello.jsp

<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    인사말: ${greeting}
</body>
</html>
profile
June
```
