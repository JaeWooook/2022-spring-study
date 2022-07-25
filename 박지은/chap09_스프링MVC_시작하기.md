# 스프링 MVC 시작하기

## 스프링 MVC 설정


```
@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer {


    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer){
        configurer.enable();
    }

    // 매핑 경로를 '/'로 주었을때, JSP/HTML/CSS 등을 올바르게 처리하기 위한 설정

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry){
        registry.jsp("/WEB-INF/view",".jsp");
    }

    // 컨트롤러의 실행 결과를 보여주기 위한 설정
}


```


#### @EnableWebMvc 

: 스프링 MVC 설정 활성화, 스프링 MVC를 사용하는데 필요한 기본적인 구성 설정
 
<BR>

#### 1. HandlerMapping 
#### 2. ViewResolver 
#### 3. DispatcherServlet
: 웹 요청을 처리하려면 DispatcherServlet을 통해 웹 요청을 받음


<br>

### 컨트롤러 구현

```
@Controller 
public class HelloController {

    @GetMapping("/hello")
    public String hello(Model model, @RequestParam(value = "name", required = false) String name){
        model.addAttribute("greeting","안녕하세요"+name);
        return "hello";
    }
}
```

- @Controller : 컨트롤러 선언
- @GetMapping("/hello") : "hello" 경로로 들어온 요청을 hello 메소드를 이용해 처리(GET 메소드에 대한 매핑)
- Model 파라미터 : 컨트롤러의 처리 결과를 뷰에 전달

```
model.addAttribute("greeting","안녕하세요"+name);

"greeting" : 모델 키
"안녕하세요"+name : 모델 값
```

- @RequestParam : http 요청 파라미터 값을 메소드의 파라미터로 전달

<br>

![image](https://user-images.githubusercontent.com/81572478/180778119-656a7cd9-1a67-415a-a7c4-d17dfb00d6ae.png)


<br>
<br>

### JSP 구현

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry){
        registry.jsp("/WEB-INF/view",".jsp");
    }


✅ MvcConfig 클래스에서 jsp를 뷰 구현으로 사용할 수 있도록 설정(registry.jsp)
- 파일 경로 : "/WEB-INF/view"
- 파일 타입 : .jsp

<Br>

```
<%@ page contentType="text/html; charset=utf-8" %>
<!DOCTYPE html>
<html>
  <head>
    <title>Hello</title>
  </head>
  <body>
    인사말: ${greeting}
  </body>
</html>

➡️ controller에서 받은 키(greeting)에 값(안녕하세요+name) 출력
```

