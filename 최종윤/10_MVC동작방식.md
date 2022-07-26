### 동작방식
그림에서 <<spring bean>>이라고 표시한 것은 스프링 빈으로 등록해야 하는 것을 의미한다.회색 배경을 가진 구성 요소는 개발자가 직접 구현해야 하는 요소이다.

DispatcherServlet은 모든 연결을 담당한다. 웹 브라우저로부터 요청이 들어오면 DispatcherServlet은 그 요청을 처리하기 위한 컨트롤러 객체를 검색한다.

이 때 DispatcherServlet은 직접 컨트롤러를 검색하지 않고 HandlerMapping이라는 빈 객체에게 컨트롤러 검색을 요청한다.

HandlerMapping은 클라이언트의 요청 경로를 이용해서 이를 처리할 컨트롤러 빈 객체를 DispatcherServlet에 전달한다.

예를 들어 웹 요청 경로가 '/hello'라면 등록된 컨트롤러 빈 중에서 '/hello' 요청 경로를 처리할 컨트롤러를 리턴한다.

컨트롤러 객체를 DispatcherServlet이 전달받았다고 해서 바로 컨트롤러 객체의 메서드를 실행할 수 있는 것은 아니다. 

DispatcherServlet은 @Controller, Contoller 인터페이스, HttpRequestHandler 인터페이스를 동일한 방식으로 처리하기 위해 중간에 사용되는 것이 바로 HandlerAdapter 빈이다.

DispatcherServlet은 HandlerMapping이 찾아준 컨트롤러 객체를 처리할 수 있는 HandlerAdapter 빈에게 요청 처리를 위임한다.

HandlerAdapter는 컨트롤러의 알맞은 메서드를 호출해서 요청을 처리하고 그 결과를 DispatcherServlet에 리턴한다. 

이때 HandlerAdapter는 컨트롤러의 처리 결과를 ModelAndView라는 객체로 변환해서 DispatcherServlet에 리턴한다.

HandlerAdapter로부터 컨트롤러의 요청 처리 결과를 ModelAndView로 받으면 DispatcherServlet은 결과를 보여줄 뷰를 찾기 위해 ViewResolver 객체를 사용한다. 

ModelAndView는 컨트롤러가 리턴한 뷰 이름을 담고 있는데 ViewResolver는 이 뷰 이름에 해당하는 View 객체를 찾거나 생성해서 리턴한다.

응답을 생성하기 위해 JSP를 사용하는 ViewResolver는 매번 새로운 View 객체를 생성해서 DispatcherServlet에 리턴한다.

DispatcherServlet은 ViewResolver가 리턴한 View 객체에게 으답 결과 생성을 요청한다. 

JSP를 사용하는 경우 View 객체는 JSP를 실행함으로써 웹 브라우저에 전송할 응답 결과를 생성하고 이로써 모든 과정이 끝이 난다.

### 컨트롤러와 핸들러
클라이언트의 요청을 실제로 처리하는 것은 컨트롤러이고 DispatcherServlet은 클라이언트의 요청을 전달받는 창구 역할을 한다. 

컨트롤러를 찾아주는 객체는 ControllerMapiing 타입이어야 할 것 같은데 왜 HandlerMapping일까?

스프링 MVC는 웹 요청을 처리할 수 있는 범용 프레임워크이다. 일반적으로 @Controller 애노테이션을 붙인 클래스를 이용해서 클라이언트의 요청을 처리하지만
 
 원한다면 자신이 직접 만든 클래스를 이용해서 클라이언트의 요청을 처리할 수도 있다. 즉 DispatcherServlet 입장에서는 클라이언트 요청을 처리하는 객체의 
 
 타입이 반드시 @Controller를 적용한 클래스일 필요는 없다. 실제로 스프링이 클라이언트의 요청을 처리하기 위해 제공하는 타입 중에 HttpRequestHandler도 존재한다.

@Controller 적용 객체나 Controller 인터페이스를 구현한 객체는 모두 스프링 MVC 입장에서는 핸들러가 된다.

DispatcherServlet은 핸들러 객체의 실제 타입에 상관없이 실행 결과를 ModelAndView라는 타입으로만 받을 수 있으면 된다. 핸들러의 구현 타입에 따라 

ModelAndView를 리턴하는 객체도, 그렇지 않은 객체도 있다. 따라서 핸들러의 처리 결과를 ModelAndView로 변환해주는 객체가 필요하며, HandlerAdapter가 이 변환을 처리해 준다.


DispatcherServlet은 전달받은 설정 파일을 이용해서 스프링 컨테이너를 생성하는데 앞에서 언급한 

HandlerMapping, HandlerAdapter, 컨트롤러, ViewResovler  빈은 DispatcherServlet이 생성한 스프링 컨테이너에서 구한다.


Controller를 위한 HandlerMapping과 HandlerAdapter

RequestMappingHandlerMapping은 @Controller 애노테이션이 적용된 객체의 요청 매핑 애노테이션(@GetMapping) 값을 이용해서 웹 브라우저의 요청을 처리할 컨트롤러 빈을 찾는다.

RequestMappingHandlerAdapter는 컨트롤러 메서드 결과 값이 String 타입이면 해당 값을 뷰 이름으로 갖는 ModelAndView 객체를 생성해서 DispatcherServlet에 리턴한다.
