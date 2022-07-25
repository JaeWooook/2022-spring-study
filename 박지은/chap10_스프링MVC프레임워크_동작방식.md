# 스프링 MVC 프레임워크 동작방식

## 스프링 MVC 핵심 구성 요소


![image](https://user-images.githubusercontent.com/81572478/180781072-92e2f4ea-28f7-4d89-9b68-e654098883e9.png)


- 컨트롤러, JSP : 개발자가 직접 구현해야 하는 요소
- Spring Bean : 스프링 빈으로 등록해야 하는 요소

### 1. DispatcherServlet (모든 연결 담당)
<br>
: 클라이언트의 요청을 전달받는 창구 역할<br>

DispatcherServlet은 웹 브라우저로 요청이 들어오면 요청을 처리할 컨트롤러를 찾기 위해   HandlerMapping에게 컨트롤러 객체를 검색 요청

➡️ DispatcherServlet은 HanderMapping이 찾아준 컨트롤러 객체를 처리할 수 있는 Handler 빈에게 요청 처리 위임

<br><BR>

### 2. HandlerMapping (컨트롤러 찾기 담당)
<br>
: DispatcherServlet은 HandlerMapping에게 핸들러(컨트롤러)를 찾게 요청

➡️ HandlerMapping은 DispatcherServlet에게 컨트롤러 객체 반환(이때 ModelAndView 타입으로 반환받아야 함)

- HandlerMapping 
: Handler = 웹 요청을 실제로 처리하는 객체 <br>(@Controller 적용 객체/@Controller인터페이스를 구현한 객체 등)

<BR><bR>

### 3. HandlerAdapter (컨트롤러 메소드 처리 담당)
<br>
: DispatcherServlet은 HandlerAdapter에 요청 처리 위임,

컨트롤러의 알맞은 메소드를 호출해 요청을 처리하고 그 결과를 DispatcherServlet에게 리턴

➡️ 컨트롤러 처리결과를 **ModelAndView 객체로 반환**해 리턴함


<bR><bR>

### 4. ViewResolver (뷰 탐색 담당)
<br>
: 컨트롤러의 실행 결과를 받은 DispatcherServlet은 ViewResolver에게 뷰 이름에 해당하는 View 객체 요청

➡️ 컨트롤러에서 지정한 model 속성은 request 객체 속성으로 JSP에 전달


<br><Br>

![image](https://user-images.githubusercontent.com/81572478/180785112-2e7b6f2e-d155-45a8-994b-8b571f92de5c.png)




➕ HandlerMapping, HandlerAdapter, 컨트롤러 빈, ViewResolver 등의 빈은 DispatcherServlet이 생성한 스프링 컨테이너에서 구함


<br><BR>
🔎 @EnableWebMvc : @Controller 어노테이션 붙은 컨트롤러 위한 설정 생성 

🔎 WebMvcConfigurer 인터페이스 : MVC 추가 설정

: 재정의가 필요한 메소드만 구현하면 됨!
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

    // 뷰 설정
}


```

<br><Br>

### 🔎 디폴트 핸들러와 HandlerMapping 우선순위

: 매핑경로가 "/"인 경우, .jsp로 끝나는 요청을 제외한 모든 요청은 DispatcherServlet이 처리(.html/.css 등)

**but**, HandlerMapping은 @Controller 적용한 빈 객체가 처리할 수 있는 요청 경로만 대응가능(@GetMapping("/hello")) <br>

= /index.html or .css 같은 요청을 처리할 수 있는 컨트롤러 객체 찾지 못함 
<br>
= 404 응답

```
@Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer){
        configurer.enable();

        ...
    }
```
➡️ DispatcherServletHandlerConfigurer#enable

: 클라이언트의 모든 요청을 WAS가 제공하는 디폴트 서블릿에 전달

1. RequestMappingHandlerMapping을 사용해 요청할 핸들러(컨트롤러) 검색 

    → 존재하면 해당 컨트롤러 이용해 요청 처리

2. 존재하지 않으면 SimpleUrlHandlerMapping 사용해 요청 처리할 핸들러 검색

    → DispatcherServletHandlerConfigurer#enable() 메소드가 등록한 SimpleUrlHandlerMapping은 "/**" 경로 (모든 경로)에 대해  cDefaultServletHttpRequestHandler 리턴

    → DispatcherServlet은 DefaultServletHttpRequestHandler에게 처리 요청

    → DefaultServletHttpRequestHandler은 디폴트 서블릿에 처리 위임

    ➡️ 별도 설정이 없는 모든 요청 경로:
    SimpleUrlHandlerMapping → DefaultServletHttpRequestHandler → **디폴트 서블릿**
