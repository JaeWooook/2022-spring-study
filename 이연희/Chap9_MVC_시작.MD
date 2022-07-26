# Chap9 스프링 MVC 시작하기
## 컨트롤러 구현
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
- **@Controller** 어노테이션을 적용한 클래스는 스프링 MVC에서 컨트롤러로 사용한다.
- **@GetMapping** 어노테이션은 메서드가 처리할 요청 경로를 지정한다. "/hello" 경로로 들어온 요청을 처리한다. 
- **Model** 파라미터는 컨트롤러의 처리 결과를 뷰에 전달할 때 사용한다.
- **@RequestParam** 어노테이션은 HTTP 요청 파라미터의 값을 메서드의 파라미터로 전달할 때 사용된다. name 요청 파라미터 값을 name 파라미터에 전달한다.
- **"greeting" 이라는 모델 속성**에 값을 설정한다. 값으로는 "HI, {name}"의 문자열을 사용한다. 뒤에 작성할 JSP 코드에서 이 속성을 이용해서 값을 출력한다.
- 컨트롤러 처리 결과를 보유줄 뷰 이름으로 **"hello"**를 사용한다.

> 스프링 MVC 프레임워크에서 컨트롤러란 웹 요청을 처리하고 그 결과를 뷰에 전달하는 스프링 빈 객체이다. 스프링 컨트롤러로 사용될 클래스는 @Controller 어노테이션을 붙여야 하고, @GetMapping이나 @PostMapping 같은 요청 매핑 어노테이션을 이용해서 처리할 경로를 지정해줘야 한다.

![](https://velog.velcdn.com/images/yh_lee/post/13720885-e286-4f98-8091-50b982c7a70a/image.png)

컨트롤러를 구현했으므로 컨트롤러를 스프링 빈으로 등록할 단계이다. 컨트롤러 클래스를 빈으로 등록할 때 사용할 설정 파일은 다음과 같다.
```java
@Configuration
public class ControllerConfig{
	@Bean
    public HelloController helloCotroller(){
    	return new HelloController();
    }
}
```

## JSP 구현
컨트롤러가 생성한 결과를 보여줄 뷰 코드를 만들어보자. 
**WEB-INF/view/hello.jsp**
```jsp
...
<body>
 인사말: ${greeting}
</body>
...
```
뷰 이름과 JSP 파일과의 연결은 MvcConfig 클래스의 다음 설정을 통해서 이루어진다.
```java
@Override
public void configureViewResolvers(ViewResolverRegistry registry){
	registry.jsp("/WEB-INF/view/",".jsp");
   
}
```
registry.jsp() 코드는 JSP를 뷰 구현으로 사용할 수 있게 한다. ![](https://velog.velcdn.com/images/yh_lee/post/20272fd9-9ba4-4c0a-a787-f2fa7f56314e/image.png)
"greeting"은 컨트롤러에서 설정한 속성을 뷰 JSP 코드에서 접근할 수 있는 이유는 스프링 MVC 프레임워크가 아래 그림처럼 모델에 추가한 속성을 JSP 코드에서 접근할 수 있게 HttpServletRequest에 옮겨주기 때문이다.![](https://velog.velcdn.com/images/yh_lee/post/04d7f806-4dc5-46a9-9321-27fea02effd5/image.png)
