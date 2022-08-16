# Chap16 JSON 응답과 요청 처리
웹 페이지에서 Ajax를 이용해서 서버 API를 호출하는 사이트가 많다. 이들 API는 웹 요청에 대한 응답으로 HTML 대신 JSON이나 XML을 사용한다. 웹 요청에도 쿼리 문자열 대신 JOSN, XML을 데이터로 보내기도 한다. GET, POST만 사용하지 않고 PUT, DELETE와 같은 다른 방식도 사용한다. 스프링 MVC를 사용하면 이를 위한 웹 컨트롤러를 쉽게 만들 수 있다. 
## Jackson 의존 설정
Jackson은 자바 객체와 JSON 형식 문자열 간 변환을 처리하는 라이브러리이다. 스프링 MVC에서 Jackson 라이브러리를 이용해서 자바 객체를 JSON으로 변환하려면 클래스 패스에 Jackson 라이브러리를 추가하면 된다. 

Jackson은 아래 그림과 같이 자바 객체와 JSON 사이의 변환을 처리한다.![](https://velog.velcdn.com/images/yh_lee/post/02e31332-01f4-4bf5-8e1b-cdaae6aeb5f8/image.png)


## @RestController로 JSON 형식 응답

스프링 MVC에서 JSON 형식으로 데이터를 응답하기 위해선 @Controller 대신 @RestController 어노테이션을 사용하면 된다.
```java
@RestController
public class RestMemberController{
	private MemberDao memberDao;
    private MemberRegisterService registerService;
    
    @GetMapping("/api/members")
    public List<Member> members(){
    	return memberDao.selectAll();
    }
    
    @GetMapping("/api/members/{id}")
    public Member member(@PathVariable Long id, HttpServletResponse response) throws IOException{
    	...
    }
    ...
}
```
@RestController 어노테이션을 붙이면 스프링 MVC는 요청 매핑 어노테이션을 붙인 메서드가 리턴한 객체를 알맞은 형식으로 변환해서 응답데이터로 전송한다. 이때 Jackson을 사용하면 JSON 형식의 문자열로 변환해서 응답한다. 

리턴 타입이 List<Member\>인 경우에 List 객체를 JSON 형식의 배열로 변환한 결과이다. ![](https://velog.velcdn.com/images/yh_lee/post/610332b0-fa1a-4930-ab35-43a46426b14b/image.png)

> @RestController 어노테이션이 추가되기 전에는 다음과 같이 @Controller 어노테이션과 @ResponseBody 어노테이션을 사용했다. 
```java
@Controller
public class RestMemberController{
	private MemberDao memberDao;
    private MemberRegisterService registerService;
    
    @RequestMapping(path="/api/members", method=RequestMethod.GET)
    @ResponseBody
    public List<Member> members(){
    	return memberDao.selectAll();
    }
}
```

### @JsonIgnore를 이용한 제외 처리
응답 결과에 password가 포함되어 있으면 안되기 때문에 응답 결과에서 제외시켜야 한다. Jackson이 제공하는 @JsonIgnore 어노테이션을 사용하면 이를 간단히 처리할 수 있다. 다음과 같이 JSON 응답에 포함시키지 않을 대상에 어노테이션을 붙인다.
```java
public class Member{
	private Long id;
    private String email;
    @JsonIgnore
    private String password;
    private String name;
    private LocalDateTime registerDateTime;
}
```

### @JsonFormat 사용
Jackson에서 날짜나 시간 값을 특정한 형식으로 표현하는 가장 쉬운 방법은 @JsonFormat을 사용하는 것이다. 
```java
public class Member{
	private Long id;
    private String email;
    private String name;
    @JsonFormat(pattern="yyyyMMddHHmmss")
    private LocalDateTime registerDateTime;
}
```
Json 응답 결과는 다음과 같다. 
```json
{
  "id":1,
  "email":"abc@gmail.com",
  "name:"spring",
  "registerDateTime":"202208150130"
}
```

### 날짜 형식 변환 처리
스프링 MVC는 자바 객체를 HTTP 응답으로 변환할 때 HttpMessageConverter라는 것을 사용한다. JSON으로 변환할 때 사용하는 컨버터를 새롭게 등록해서 날짜 형식을 원하는 형태로 변환하도록 설정하면 모든 날짜 형식에 동일한 변환 규칙을 적용할 수 있다. 
```java
@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfiguration{
	...
    @Override
    public viod extendMessageConverter{
    	List<HttpMessageConverter<?>> converters){
        	ObjectMapper objectMapper = Jackson2ObjectMapperBuilder
            .json()
            .featuresToDisable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
            .build();
            converters.add(0,new MappingJackson2HttpMessageConverter(objectMapper));
        }	
    }
}
```


### 응답 데이터의 컨텐츠 형식
응답 헤더의 Content-type이 application/json인 것을 알 수 있다. 

## @RequestBody로 JSON 요청 처리
POST, PUT 방식을 사용하면 name=이름&age=20과 같은 쿼리 문자열 형식이 아니라 다음과 같은 JSON 형식의 데이터를 요청 데이터로 전송할 수 있다. 
```json
{"name":"이름", "age":20}
```
JSON 형식으로 전송된 요청 데이터를 커맨드 객체로 전달받는 방법은 커맨드 객체에 @RequestBody 어노테이션을 붙이면 된다.
```java
@RestController{
	...
    @PostMapping("/api/members")
    public void new Member(@RequestBody @Valid RegisterRequest regReq, HttpServletResponse response){
    	...
    }
}
```

### JSON 데이터의 날짜 형식 다루기
특정 패턴을 가진 문자열을 LocalDateTime이나 Date 타입으로 변환하고 싶다면 @JsonFormat 어노테이션의 pattern 속성을 이용해서 패턴을 지정한다.
```java
@JsonFormat(pattern="yyyyMMddHHmmss")
private LocalDateTime birthDateTime;

@JsonFormat(pattern="yyyyMMdd HHmmss")
private Date birthDate;
```

### 요청 객체 검증하기 
newMember()의 regReq 파라미터에 @Valid 어노테이션이 붙어있다. JSON 형식으로 전송한 데이터를 변환한 객체도 동일한 방식으로 @Valid 어노테이션이나 별도 Validator를 이용해서 검증할 수 있다. Validator를 사용할 경우 직접 상태 코드를 처리해야 한다.

## ResponseEntity로 객체 리턴하고 응답 코드 지정하기
지금까지는 상태 코드를 지정하기 위해 HttpServletResponse의 setStatus(), sendError()를 사용했다. 문제는 HttpServletResponse를 이용해서 404 응답을 하면 JSON 형식이 아닌 서버가 기본으로 제공하는 HTML을 응답 결과로 제공한다는 점이다. 

API를 호출하는 입장에서 JSON과 HTML을 모두 처리하는 것은 부담스럽다. 처리에 실패한 경우 HTML 응답 데이터 대신에 JSON 형식의 응답 데이터를 전송해야 API 호출 프로그램이 일관된 방법으로 응답을 처리할 수 있다. 

### ResponseEntity를 이용한 응답 데이터 처리
정상인 경우와 비정상인 경우 모두 JSON 응답을 전송하는 방법은 ResponseEntity를 사용하는 것이다. 

에러 상황일 때 응답으로 사용할 ErrorResponse 클래스이다. 
```java
public class ErrorResponse{
	private String message;
    
    public ErrorResponse(String message){
    	this.message = message;
    }
    
    public String getMessage(){
    	return message;
    }
}
```
ResponseEntity를 사용하면 member() 메서드를 아래와 같이 구현할 수 있다.
```java
@RestController
public class RestMemberController{
	...
    @GetMapping("/api/members/{id}")
    public ResponseEntity<Object> member(@PathVariable Long id){
    	Member member = memberDao.selectById(id);
        if(member==null){
        	//ErrorResponse를 body로 지정해서,
            //ErrorResponse를 JSON으로 변환한다.
        	return ResponseEntity
            	.status(HttpStatus.NOT_FOUND)
               	.body(new ErrorResponse("no member"));
        }
        //member를 body로 지정해서, member 객체를 JSON으로 변환한다.
        return ResponseEntity.status(HttpStatus.OK).body(member);
    }
}
```
스프링 MVC는 리턴 타입이 ResponseEntity이면 body로 지정한 객체를 사용해서 변환을 처리한다. ResponseEntity의 status로 지정한 값을 응답 상태 코드로 사용한다.

존재하지 않는 ID를 이용해서 실행한 결과 404 상태 코드와 함께 JSON 형식으로 응답 데이터를 전송한 것을 확인할 수 있다.![](https://velog.velcdn.com/images/yh_lee/post/46cfa06d-3639-4184-81c8-8f8bbab3a22a/image.png)

ResponseEntity를 생성하는 기본 방법은 status와 body를 이용해서 상태 코드와 JSON으로 변환할 객체를 지정하는 것이다.

	ResponseEntity.status(상태코드).body(객체)

200(OK) 응답 코드와 몸체 데이터를 생성할 경우 다음과 같이 ok() 메서드를 이용할 수 있다.

	ResponseEntity.ok(member)
    
만약 몸체 내용이 없다면 다음과 같이 body를 지정하지 않고 build()로 바로 생성한다.

	ResponseEntity.status(HttpStatus.NOT_FOUND).build()

몸체 내용이 없는 경우 status() 메서드 대신에 다음과 같이 관련 메서드를 사용해도 된다. 

	ResponseEntity.notFound().build()

- noContent(): 204
- badRequest(): 400
- notFound(): 404

newMember()에서 201(Created) 상태 코드와 Location 헤더를 함께 전송하는 방법
```java
//1
response.setHeader("Location", "/api/members/"+newMemberId);
response.setStatus(HttpServletResponse.SC_CREATED);

//2
URI uri = URI.create("/api/members/"+newMemberId);
return ResponseEntity.created(uri).build();
```

### @ExceptionHandler 적용 메서드에 ResponseEntity로 응답하기 
앞선 코드는 member가 존재하지 않을 때 기본 HTML 에러 응답 대신에 JSON 응답을 제공하기 위해 ResponseEntity를 사용했다. 그런데 회원이 존재하지 않을 때 404 상태 코드를 응답해야 하는 기능이 많으면 에러 응답을 위해 ResponseEntity를 생성하는 코드 중복이 발생한다. 이때 @ExceptionHandler 어노테이션을 적용한 메서드에서 에러 응답을 처리하도록 구현하면 중복을 없앨 수 있다.
```java
@GetMapping("/api/members/{id}")
public ResponseEntity<Object> member(@PathVariable Long id){
  	Member member = memberDao.selectById(id);
    if(member==null){
    	throw new MemberNotFoundException();    
    }
    return member;
}

@ExceptionHandler(MemberNotFoundException.class)
public ResponseEntity<ErrorResponse> handleNoData(){
	return ResponseEntity
    			.status(HttpStatus.NOT_FOUND)
                .body(new ErrorResponse("no member");
}
```
위 코드의 member() 메서드는 Member 자체를 리턴한다. 회원 데이터가 존재하면 Member 객체를 리턴하므로 JSON으로 변환한 결과를 응답한다. 회원 데이터가 존재하지 않으면 MemberNotFoundException을 발생한다. 이 익셉션이 발생하면 @ExceptionHandler 어노테이션을 사용한 handleNoData() 메서드가 에러를 처리한다. 404 상태 코드와 ErrorResponse 객체를 몸체로 갖는 ResponseEntity를 반환한다. 즉, MemberNotFoundException가 발생하면 상태코드가 404이고 몸체가 JSON 형식인 응답을 전송한다. 

@RestControllerAdvice 어노테이션을 이용해서 에러 처리 코드를 별도 클래스로 분리할 수도 있다. @RestControllerAdvice는 @ControllerAdvice와 동일하다. 차이는 @RestController와 동일하게 응답을 JSON, XML 형식으로 변환한다는 것이다.
```java
@RestControllerAdvice("controller")
public class ApiExceptionAdvice{
	@ExceptionHandler(MemberNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNoData(){
		return ResponseEntity
    			.status(HttpStatus.NOT_FOUND)
                .body(new ErrorResponse("no member");
    }
}	
```
@RestControllerAdvice를 사용하면 에러 처리 코드가 한 곳에 모여 효과적으로 에러 응답을 관리할 수 있다. 

### @Valid 에러 결과를 JSON으로 응답하기 
@Valid 어노테이션을 붙인 커맨드 객체가 값 검증에 실패하면 400 상태 코드를 응답한다. 
```java
@PostMapping("/api/members")
public ResponseEntity<Object> newMember(@RequestBody @Valid RegisterRequest regReq){
	...
}
```
문제는 HttpServletResponse를 이용해서 상태 코드를 응답했을 때와 마찬가지로 HTML 응답을 전송한다. @Valid 어노테이션을 이용한 검증을 실패했을 때 HTML 응답 대신 JSON 형식의 응답을 제공하고 싶으면 Errors 타입 파라미터를 추가해서 직접 에러 응답을 생성하면 된다. 
```java
@PostMapping("/api/members")
public ResponseEntity<Object> newMember(@RequestBody @Valid RegisterRequest regReq, Errors errors){
	if(errors.hasErrors()){
    	String errorCodes=errors.getAllErrors()//List<ObjectError>
        		.stream()
                .map(error->error.getCodes()[0])//error는 ObjectError
                .collect(Collectors.joining(","));
                
        return ResponseEntity
        		.status(HttpStatus.BAD_REQUEST)
                .body(new ErrorResponse("errorCodes="+errorCodes); 
    }
    ...
}
```
hasErrors() 메서드를 이용해서 검증 에러가 있는지 확인한다. 검증에러가 존재하면 모든 에러 정보를 구하고 각 에러 코드 값을 연결한 문자열을 생성해서 errorCodes 변수에 할당한다. 이처럼 코드를 수정한 후 검증에 실패하는 데이터를 전송하면 JSON 응답이 오는 것을 확인할 수 있다. 

@RequestBody를 붙인 경우 @Valid를 붙인 객체의 검증에 실패했을 때 Errors 타입 파라미터가 존재하지 않으면 MethodArgumentNotValidException이 발생한다. 따라서 다음과 같이 @ExceptionHandler 어노테이션을 이용해서 검증 실패시 에러 응답을 생성해도 된다. 

```java
@RestControllerAdvice("controller")
public class ApiExceptionAdvice{
	@ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleBindException(MethodArgumentNotValidException ex){
    	String errorCodes=ex.getBindingResult().getAllErrors()
        				.stream()
                        .map(error->error.getCodes()[0])
                        .collect(Collectors.joining(","));
                        
        return ResponseEntity
        		.status(HttpStatus.BAD_REQUEST)
                .body(new ErrorResponse("errorCodes="+errorCodes);
    }
}
```