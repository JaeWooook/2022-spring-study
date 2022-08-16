# JSON 응답과 요청 처리

<BR><BR>

## JSON 개요

- JSON : 간단한 형식을 갖는 문자열로 데이터 교환에 주로 사용
    - 중괄호를 사용해 객체 표현
    - "이름" : "값"
    - 배열은 대괄호로 표현

```
{
    "employees" : [
        { "firstName" : "지은" , "lastName" : "박" },
        { "firstName" : "지은2" , "lastName" : "박" }
    ]
}
```

<BR>

## @RestController로 JSON 형식 응답

: 스프링 MVC에서 JSON 형식으로 데이터를 응답하는 방법

```
@RestController
public class RestMemberController {

    ...

    @GetMapping("/api/members")
    public List<Member> members(){
        ...
    }
```

@RestController 어노테이션을 붙인 경우 스프링 MVC는 요청 매핑 어노테이션을 붙인 메소드가 리턴한 객체를 알맞은 형식으로 변환해 응답 데이터로 전송

➡️ 클래스 패스에 Jackson이 존재하면 JSON 형식의 문자열로 변환해서 응답

📌 Jackson : 자바 객체와 JSON 형식 문자열 간 변환을 처리하는 라이브러리

<BR>

```
public List<Member> members(){
```
: 리턴타입이 ```List<Member>```인데 이 경우 해당 List 객체를 JSON 형식의 배열로 변환해 응답

<BR>

➕ @RestController = @Controller + @ResponseBody


<br><br>

### 🔎 @JsonIgnore를 이용한 제외 처리

: JSON 응답에 포함시키지 않을 대상에 @JsonIgnore 어노테이션을 붙임

```
public class Member {

    private Long id;
    private String email;
    @JsonIgnore
    private String password;
```


<Br>

### 🔎 @JsonFormat을 이용한 날짜 형식 변환 처리

- LocalDateTime타입 이라면 JSON 값은 ```배열```로 바뀜
- java.util.Date 타입이면 유닉스 타임 스탬프로 날짜 값 표현

➡️ Jackson에서 날짜/시값 값을 특정 형식으로 표현하고 싶다면 ```@JsonFormat``` 이용

```
public class Member {

    ...
    @JsonIgnore
    private String password;
    private String name;

    @JsonFormat(shape= JsonFormat.Shape.STRING) // ISO-8601 형식으로 변환
    private LocalDateTime registerDateTime;
```

➕ ISO-8601 형식이 아닌 원하는 형식으로 변환하고 싶다면 @JsonFormat에서 pattern 속성 이용
```
@JsonFormat(pattern="yyyyMMddHHmmss") 
    private LocalDateTime registerDateTime;
```


<br>

### ✅ 날짜 타입에 해당하는 모든 대상에 동일한 변환 규칙을 적용하고 싶다면?

<br>

: 스프링 MVC 설정 변경! JSON으로 변환할때 사용하는 MappingJackson2HttpMessageConverter를 새롭게 등록해 날짜 형식을 원하는 형식으로 변환하도록 설정

➡️ 모든 날짜 형식에 동일한 변환 규칙 적용

```
@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer {

    ...

    @Override
        public void extendMessageConverters(
                List<HttpMessageConverter<?>> converters
        ){
            ObjectMapper objectMapper = Jackson2ObjectMapperBuilder
                    .json()
                    .featuresToDisable(
                            SerializationFeature.WRITE_DATE_KEYS_AS_TIMESTAMPS
                    ) // 날짜 타입의 값을 ISO-8601 형식으로 출력
                    .build();
            converters.add(0, new MappingJackson2HttpMessageConverter(objectMapper));
        }

    ...
}
```
<Br>

- 모든 java.util.Date 타입의 값을 원하는 형식으로 출력하고 싶다면
```
ObjectMapper objectMapper = Jackson2ObjectMapperBuilder
                    .json()
                    .simpleDateFormat("yyyyMMddHHmmss")
                    .build();
```

<br>

- 모든 LocalDateTime 타입에 대해 ISO-8601 형식 대신 원하는 패턴을 설정하고 싶다면
```
ObjectMapper objectMapper = Jackson2ObjectMapperBuilder
                    .json()
                    .serializerByType(LocalDateTime.class, new LocalDateTimeSerializer(formatter))
                    .build();
```

<Br><Br>

## @RequestBody로 JSON 요청 처리

: JSON 형식의 요청 데이터를 자바 객체로 변환하는 기능

➡️ 커맨드 객체에 @RequestBody어노테이션을 붙이면 JSON 형식으로 전송된 요청 데이터를 커맨드 객체로 

```
 @PostMapping("/api/members")
    public void newMember(@RequestBody @Valid RegisterRequest regReq, HttpServletResponse response) throws  IOException{
        ...
    }
```

✅ 스프링 MVC가 JSON형식으로 전송된 데이터를 올바르게 처리하려면 요청 컨텐츠 타입이 application/json이어야 함.

but 보통 POST 방식의 폼데이터는 쿼리문자열인 "P1=V1&P2=V2"로 전송 = application/x-www-form-urlencoded

∴ 별로 프로그램 필요, 크롬브라우저의 Advanced REST client 확장 프로그램, Postman 등


<br>

➕ JSON 형식으로 전송한 데이터를 변환한 객체도 @Valid 어노테이션 이나 별로의 Validator이용해 검증 가능

<br><Br>

## ResponseEntity로 객체 리턴하고 응답코드 지정

- 상태코드를 지정하기 위해 HttpServletResponse의 setStatus()와 sendError() 메소드 사용

    but  이를 이용하면 404 응답시 JSON 형식이 아닌 서버가 기본으로 제공하는 HTML을 응답 결과로 제공

➡️ 정상 응답과 비정상 응답 모두 JSON 응답을 전송하는 방법은 ResponseEntity를 사용하는 것!

<BR>

```
@GetMapping("/api/members/{id}")
    public ResponseEntity<Object> member(@PathVariable Long id){
        Member member = memberDao.selectById(id);
        if(member==null){
            return ResponseEntity.status(HttpStatus.NOT_FOUND)
                    .body(new ErrorResponse("no member"));
        }
        return  ResponseEntity.status(HttpStatus.OK).body(member);
    }
```
: 스프링 MVC는 리턴 타입이 ResponseEntity이므로 body로 지정한 객체를 사용해 변환 처리함!

➡️ member가 null이면 ErrorResonse를 JSON으로 변환해 리턴하고, member가 null이 아니면 member 객체를 JSON으로 변환

<br>

➕ 상태코드와 Location헤더를 함께 전송하고 싶다면
```
@PostMapping("/api/members")
public ResponseEntity<Object> newMember(@RequestBody @Valid RegisterRequest regReq) throws  IOException{
    try{
        Long newMemberId = registerService.regist(regReq);
        URI uri = URI.create("api/members/"+newMemberId);
        return ResponseEntity.created(uri).build();
    }catch (DuplicateMemberException dupEx){
        return ResponseEntity.status(HttpStatus.CONFLICT).build();
    }
}
```
: ResponseEntity.created() 메소드에 Location 헤더로 전달할 URI 전달

<br>

📌 ResponseEntity를 생성하는 기본 방법

: status와 body를 이용해 상태코드와 JSON으로 변환할 객체 지정 ```ResponseEntity.status(상태코드).body(객체)```

### 🔎 @ExceptionHandler 적용 메소드에서 ResponseEntity로 응답하기 

- 한 메소드에서 정상 응답과 에러 응답을 ResponseBody로 생성하면 코드가 중복됨

➡️ @ExceptionHandler 어노테이션을 적용한 메소드에서 에러 응답을 처리하도록 구현하면 중복 없앨 수 있음
또는, @RestControllerAdvice 어노테이션을 이용해 에러 처리 코드를 별도 클래스로 분리!

```
@RestControllerAdvice("controller")
public class ApiExceptionAdvice {

    @ExceptionHandler(MemberNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNoData(){
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
                .body(new ErrorResponse("no member"));
    }
}
```

<br>

### 🔎 @Valid 에러 결과를 JSON으로 응답하기 

@Valid 어노테이션을 붙인 커맨드 객체가 값 검증에 실패해 400 상태 코드를 응답하면 HTML로 전송함

➡️ 이를 JSON으로 응답하고 싶다면 Errors 타입 파라미터 추가해 직접 에러 응답 생성!

```
@PostMapping("/api/members")
public ResponseEntity<Object> newMember(@RequestBody @Valid RegisterRequest regReq, Errors errors) throws  IOException{
    
    if(errors.hasErrors()){
        String errorCodes = errors.getAllErrors()
            .stream()
            .map(error -> error.getCodes()[0])
            .collect(Collectors.joining(","));

        // 에러 있으면 getAllErrors() 메소드로 모든 에러 정보 구하고, 각 에러의 코드 값을 연결한 문자열 생성(stream~collect)해 errorCodes 변수에 할당

        return ResponseEntity
            .status(HttpStatus.BAD_REQUEST)
            .body(new ErrorResponse("errorCodes="+ errorCodes));
    }
    
    ...
}
```

➕ @RequestBody 어노테이션을 붙인 경우 @Valid 어노테이션을 붙인 객체의 검증에 실패했는데 Errors 타입 파라미터 없으면 MethodArgumentNotValidException 발생