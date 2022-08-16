## JSON 응답과 요청 처리

### JSON 개요
> JSON은 간단한 형식을 갖는 문자열로 데이터 교환에 주로 사용한다.

```json
{
  "name": "유관순",
  "birthday": "1902-12-16",
  "edu": [
    {
      "title": "이화학당보통과",
      "year": 1916
    },
    ...
  ]
}
```
 - JSON은 중괄호를 사용해서 객체를 표현한다.
 - 객체는 (이름, 값) 쌍을 갖는다.
 - 이때 이름과 값은 콜론(:)으로 구분한다.

### Jackson 의존 설정
> Jackson은 자바 객체와 JSON 형식 문자열 간 변환을 처리하는 라이브러리이다.
> 
> 스프링 MVC에서 Jackson 라이브러리를 이용해서 자바 객체를 JSON으로 변환하려면 클래스 Path에 Jackson 라이브러리를 추가한다.

Jackson은 자바 객체와 JSON 사이의 변환을 처리한다.

![image](https://user-images.githubusercontent.com/55322459/184500340-eacba6cf-7930-4d18-a191-b0eface7d1c1.png)

 - 프로퍼티 타입이 배열이나 List인 경우 JSON 배열로 변환된다.

### @RestController로 JSON 형식 응답
> @Controller 어노테이션 대신 @RestController 어노테이션을 사용하면 스프링 MVC에서는 JSON형식으로 데이터를 응답하게 된다.

```java
package controller;

import java.io.IOException;
import java.net.URI;
import java.util.List;
import java.util.stream.Collectors;

import javax.servlet.http.HttpServletResponse;
import javax.validation.Valid;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.Errors;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

import spring.DuplicateMemberException;
import spring.Member;
import spring.MemberDao;
import spring.MemberNotFoundException;
import spring.MemberRegisterService;
import spring.RegisterRequest;

@RestController
public class RestMemberController {
	private MemberDao memberDao;
	private MemberRegisterService registerService;

	@GetMapping("/api/members")
	public List<Member> members() {
		return memberDao.selectAll();
	}
	
	@GetMapping("/api/members/{id}")
	public ResponseEntity<Object> member(@PathVariable Long id) {
		Member member = memberDao.selectById(id);
		if (member == null) {
			return ResponseEntity
					.status(HttpStatus.NOT_FOUND)
					.body(new ErrorResponse("no member"));
			// return ResponseEntity.notFound().build();
		}
		return ResponseEntity.ok(member);
	}

	@GetMapping("/api/members2/{id}")
	public Member member2(@PathVariable Long id, HttpServletResponse response) throws IOException {
		Member member = memberDao.selectById(id);
		if (member == null) {
			response.sendError(HttpServletResponse.SC_NOT_FOUND);
			return null;
		}
		return member;
	}

	@GetMapping("/api/members3/{id}")
	public Member member3(@PathVariable Long id) {
		Member member = memberDao.selectById(id);
		if (member == null) {
			throw new MemberNotFoundException();
		}
		return member;
	}

	@PostMapping("/api/members")
	public ResponseEntity<Object> newMember(
			@RequestBody @Valid RegisterRequest regReq /*,
			Errors errors */) {
		/*
		if (errors.hasErrors()) {
			String errorCodes = errors.getAllErrors()
					.stream()
					.map(error -> error.getCodes()[0])
					.collect(Collectors.joining(","));
			return ResponseEntity
					.status(HttpStatus.BAD_REQUEST)
					.body(new ErrorResponse("errorCodes = " + errorCodes));
		}
		*/
		try {
			Long newMemberId = registerService.regist(regReq);
			URI uri = URI.create("/api/members/" + newMemberId);
			return ResponseEntity.created(uri).build();
		} catch (DuplicateMemberException dupEx) {
			return ResponseEntity.status(HttpStatus.CONFLICT).build();
		}
	}

	@PostMapping("/api/members2")
	public void newMember2(
			@RequestBody RegisterRequest regReq, 
			Errors errors, 
			HttpServletResponse response) throws IOException {
		try {
			new RegisterRequestValidator().validate(regReq, errors);
			if (errors.hasErrors()) {
				response.sendError(HttpServletResponse.SC_BAD_REQUEST);
				return;
			}
			Long newMemberId = registerService.regist(regReq);
			response.setHeader("Location", "/api/members/" + newMemberId);
			response.setStatus(HttpServletResponse.SC_CREATED);
		} catch (DuplicateMemberException dupEx) {
			response.sendError(HttpServletResponse.SC_CONFLICT);
		}
	}

	public void setMemberDao(MemberDao memberDao) {
		this.memberDao = memberDao;
	}

	public void setRegisterService(MemberRegisterService registerService) {
		this.registerService = registerService;
	}
}
```
 - @RestController 어노테이션을 붙인 경우 스프링MVC는 요청 매핑 어노테이션을 붙인 메서드가 리턴한 객체를 알맞은 형식으로 변환해서 응답 데이터로 전송한다.
 - 예를 들어 members() 메서드는 리턴 타입이 List<Member>인데 이 경우 해당 List 객체를 JSON형식의 배열로 변환해서 응답한다.

#### @JsonIgnore를 이용한 제외 처리
응답 결과에 password가 포함되어 있다. 보통 암호와 같이 민감한 데이터는 응답 결과에 포함시키면 안되므로 password 데이터를 응답 결과에서 제외 시킨다.
@JsonIgnore 어노테이션을 사용해 JSON 응답에 포함시키지 않을 대상에 어노테이션을 붙인다.

```java
package spring;

import java.time.LocalDateTime;

import com.fasterxml.jackson.annotation.JsonIgnore;

public class Member {

    private Long id;
    private String email;
    @JsonIgnore
    private String password;
    private String name;
}
```
 - @JsonIgnore 어노테이션을 붙이게되면 붙인 대상이 JSON결과에서 제외됨을 알 수 있다.

#### 날짜 형식 변환 처리: @JsonFormat 사용
```java
@JsonFormat(shape = Shape.STRING) // ISO-8601 형식으로 변환
private LocalDateTime registerDateTime;

//ISO-8601 형식이 아닌 원하는 형식으로 변환
@JsonFormat(pattern = "yyyyMMddHHmmss")
private LocalDateTime registerDateTime;
```
 - pattern 속성은 java.time.format.DateTimeFormatter 클래스나 java.text.SimpleDateFormat 클래스의 API 문서에 정의된 패턴을 따른다.

#### 날짜 형식 변환 처리 : 기본 적용 설정

> @JsonFormat 어노테이션을 지속적으로 사용하지 않고 Jackson의 변환 규칙을 모든 날짜 타입에 적용하는 방법

스프링 MVC는 자바 객체를 HTTP 응답으로 변환할 때 HttpMessageConverter라는 것을 사용한다.
JSON으로 변환할 때 사용하는 MappingJackson2HttpMessageConverter를 새롭게 등록해서 날짜 형식을 원하는 형식으로 변환하도록 설정한다.
그러면 모든 날짜 형식에 동일한 규칙ㅇ을 적용할 수 있다.

> MvcConfig 클래스 MappingJackson2HttpMessageConverter를 설정하도록 수정

```java
@Override
	public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
		//DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
		ObjectMapper objectMapper = Jackson2ObjectMapperBuilder
				.json()
				.featuresToDisable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
				//.serializerByType(LocalDateTime.class, new LocalDateTimeSerializer(formatter)) // 원하는 데이터 패턴으로 변형하는 방법
				//.deserializerByType(LocalDateTime.class, new LocalDateTimeDeserializer(formatter)) // 특정 속성이 아니라 해당 타입을 갖는 모든 속성에 적용하는 방법
				//.simpleDateFormat("yyyy-MM-dd HH:mm:ss") // Date를 위한 변환 패턴
				.build();
		converters.add(0, new MappingJackson2HttpMessageConverter(objectMapper));
	}
```
 - extendMessageConverters() 메서드를 이용해서 WebMvcConfigurer 인터페이스에 정의된 메서드로서 HttpMessageConverter를 추가로 설정한다.
 - @EnableWebMvc 어노테이션을 사용하면 스프링 MVC는 여러 형식으로 변환할 수 있는 HttpMessageConverter를 미리 등록한다.
 - 미리 등록된 컨버터에는 Jackson을 이용하는 것도 포함되어 있기 때문에 새로 생성한 HttpMessageConverter는 목록의 제일 앞에 위치 시켜야 한다.
 - 가장 앞에 위치 시키기 위해 HttpMessageConverter를 0번 인덱스에 추가한다.

### @RequestBody로 JSON 요청 처리

> JSON형식의 요청 데이터를 자바 객체로 변환하는 기능 (JSON 형식으로 전송된 요청 데이터를 커맨드 객체로 전달 받는 방법)-> @RequestBody 어노테이션

```java
@PostMapping("/api/members")
	public ResponseEntity<Object> newMember(
			@RequestBody @Valid RegisterRequest regReq) {
		try {
			Long newMemberId = registerService.regist(regReq);
			URI uri = URI.create("/api/members/" + newMemberId);
			return ResponseEntity.created(uri).build();
		} catch (DuplicateMemberException dupEx) {
			return ResponseEntity.status(HttpStatus.CONFLICT).build();
		}
	}
```
 - @RequestBody 어노테이션을 커맨드 객체에 붙이면 JSON 형식의 문자열을 해당 자바 객체로 변환한다.
 - JSON 데이터를 RegisterRequest 타입 객체로 변환할 수 있다.
 - 스프링 MVC가 JSON 형식으로 전송된 데이터를 올바르게 처리하려면 요청 컨텐츠 타입이 __application/json이어야 한다.__
 - 보통 POST방식의 폼 데이터는 쿼리 문자열인 application/x-www-form-urlencoded이다.

#### 요청 객체 검증하기

```java
@PostMapping("/api/members")
	public ResponseEntity<Object> newMember(
			@RequestBody @Valid RegisterRequest regReq) {
		try {
			Long newMemberId = registerService.regist(regReq);
			URI uri = URI.create("/api/members/" + newMemberId);
			return ResponseEntity.created(uri).build();
		} catch (DuplicateMemberException dupEx) {
			return ResponseEntity.status(HttpStatus.CONFLICT).build();
		}
	}
```
 - JSON 형식으로 전송한 데이터를 변환한 객체도 동일한 방식으로 @Valid 어노테이션이나 별도 Validator를 이용해 검증한다.
 - @Valid 어노테이션을 사용한 경우 검증에 실패하면 400(Bad Request) 상태 코드를 응답한다.

### ResponseEntity로 객체 리턴하고 응답 코드 지정하기

> 지금까지 예제 코드는 상태 코드를 지정하기 위해 HttpServletResponse의 setStatus() 메서드와 sendError() 메서드를 사용했다.
> API를 호출하는 입장에서는 404 또는 500과 같이 HTML 응답 데이터 대신에 JSON 형식의 응답 데이터를 전송해야 API 호출 프로그램이 일관된 방법으로 응답을 처리할 수 있다.

#### ResponseEntity를 이용한 응답 데이터 처리

> 정상인 경우와 비정상인 경우 모두 JSON 응답을 전송하는 방법은 ResponseEntity를 사용하는 것이다.

 - 에러 상황일 때 응답으로 사용할 ErrorResponse 클래스 예시
```java
package controller;

public class ErrorResponse {
	private String message;

	public ErrorResponse(String message) {
		this.message = message;
	}

	public String getMessage() {
		return message;
	}

}
```

 - ResponseEntity를 이용한 응답 데이터 처리 예시
```java
@GetMapping("/api/members/{id}")
	public ResponseEntity<Object> member(@PathVariable Long id) {
		Member member = memberDao.selectById(id);
		if (member == null) {
			return ResponseEntity
					.status(HttpStatus.NOT_FOUND)
					.body(new ErrorResponse("no member"));
			// return ResponseEntity.notFound().build();
		}
		return ResponseEntity.ok(member);
	}
```
 - 스프링 MVC는 리턴 타입이 ResponseEntity이면 ResponseEntity의 body로 지정한 객체를 사용해서 반환 처리를 한다.
 - member가 null이면 ErrorResponse 객체를 body로 지정하고, null이 아니면 member 객체를 body로 지정한다.
 - ID가 존재하지 않는다면, 404 상태코드로 아이디가 존재한다면 200(OK)상태 코드로 응답한다.

> ResponseEntity를 생성하는 기본 방법
```java
ResponseEntity.status(상태코드).body(객체)
```
 - 상태 코드는 HttpStatus 열거 타입에 정의된 값을 이용해서 정의한다.
 - 200(OK) 응답 코드와 body 데이터를 생성할 경우
 - ResponseEntity.ok(body) 이런식으로 생성할 수도 있다.
 - 만일 body의 내용이 없다면 ResponseEntity.status(HttpsStatus.NOT_FOUND).build() 이런식으로 생성할 수도 있다.
 - ResponseEntity.notFound().build() 이렇게 사용할 수 있다.

> body가 없을 때 status() 대신 사용할 수 있는 메서드

 - notContent() : 204
 - badRequest() : 400
 - notFound() : 404

> 201(Created) 상태 코드와 Location 헤더를 함께 전송할 때

 - 기존의 코드
```java
response.setHeader("Location", "/api/members" + newMemberId);
response.setStatus(HttpServletResponse.SC_CREATED);
```

 - ResponseEntity로 구현한 코드
```java
@PostMapping("/api/members")
	public ResponseEntity<Object> newMember(
			@RequestBody @Valid RegisterRequest regReq) {
		try {
			Long newMemberId = registerService.regist(regReq);
			URI uri = URI.create("/api/members/" + newMemberId);
			return ResponseEntity.created(uri).build();
		} catch (DuplicateMemberException dupEx) {
			return ResponseEntity.status(HttpStatus.CONFLICT).build();
		}
	}
```
 - ResponseEntity.created() 메서드를 이용해서 Location 헤더로 전달할 URI를 전달한다.

#### @ExceptionHandler 적용 메서드에서 ResponseEntity로 응답하기

> member가 존재하지 않을 때 기본 HTML 에러 응답 대신에 JSON 응답을 제공하기 위해 ResponseEntity를 사용한다.
> 그러나, __회원이 존재하지 않을 때 404 상태 코드를 응답해야 하는 기능이 많다면__ 에러 응답을 위해 ResponseEntity를 생성하는 코드가 여러 곳에 중복된다.

> 해결 방법으로 @ExceptionHandler 어노테이션을 적용한 메서드에서 에러 응답을 처리하도록 한다.

```java
@GettMapping("/api/members/{id}")
public Member member(@PathVariable Long id) {
    Member member = memberDao.selectById(id);
    if(member == null) {
        throw new MemberNotFoundException();
    
    }
    return member;
}
@ExceptionHandler(MemberNotFoundException.class)
public ResponseEntity<ErrorResponse> handleNoData(){
        return ResponseEntity
        .status(HttpStatus.NOT_FOUND)
        .body(new ErrorResponse("no member"));
    }
```
 - 위 코드에서 member() 메서드는 Member 자체를 리턴한다. 회원 데이터가 존재하면 Member 객체를 리턴하므로 JSON 변환한 결과를 응답한다.
 - 회원데이터가 존재하지 않으면 MemberNotFoundException를 발생한다.
 - MemberNotFoundException 익셉션이 발생하면 @ExceptionHandler 어노테이션을 사용한 handleNoData() 메서드가 에러를 처리한다.
 - handleNoData()는 404 상태 코드와 ErrorResponse 객체를 body로 갖는 ResponseEntity를 리턴한다.
 - 따라서 MemberNotFoundException이 발생하면 상태 코드가 404이고, body가 JSON 형식인 응답을 전송한다.

> @RestControllerAdvice 어노테이션을 이용해서 에러 처리 코드를 별도 클래스로 분리할 수도 있다.
> @ControllerAdvice 와 차이는 응답을 JSON이나 XML과 같은 형식으로 변환한다는 것이다.

 - @RestControllerAdvice 예시 코드
```java
package controller;

import java.util.stream.Collectors;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import spring.MemberNotFoundException;

@RestControllerAdvice("controller")
public class ApiExceptionAdvice {

    @ExceptionHandler(MemberNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNoData() {
        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(new ErrorResponse("no member"));
    }
}
```
 - @RestControllerAdvice 어노테이션을 사용하면 에러 처리 코드가 한 곳에 모여 효과적으로 에러 응답을 관리할 수 있다.

#### @Valid 에러 결과를 JSON으로 응답하기

> @Valid 어노테이션을 붙인 커맨드 객체가 값 검증에 실패하면 400 상태 코드를 응답한다.

> __문제는 HttpServeltResponse를 이용해서 상태 코드를 응답했을 때와 마찬가지로 HTML 응답을 전송한다는 것이다.__

 - @Valid 어노테이션의 검증 실패 했을 때 JSON 형식 응답을 제공하는 방법
 - Errors 타입 파라미터를 추가해서 직접 에러 응답을 생성한다.

```java
@PostMapping("/api/members")
	public ResponseEntity<Object> newMember(
			@RequestBody @Valid RegisterRequest regReq ,
			Errors errors) {
		if (errors.hasErrors()) {
			String errorCodes = errors.getAllErrors()
					.stream()
					.map(error -> error.getCodes()[0])
					.collect(Collectors.joining(","));
			return ResponseEntity
					.status(HttpStatus.BAD_REQUEST)
					.body(new ErrorResponse("errorCodes = " + errorCodes));
		}
		try {
			Long newMemberId = registerService.regist(regReq);
			URI uri = URI.create("/api/members/" + newMemberId);
			return ResponseEntity.created(uri).build();
		} catch (DuplicateMemberException dupEx) {
			return ResponseEntity.status(HttpStatus.CONFLICT).build();
		}
	}
```
 - hasError() 메서드를 이용해서 검증 에러가 존재하는지 확인
 - 검증 에러가 존재하면 getAllErrors() 메서드로 모든 에러 정보를 구하고 각 에러의 코드 값을 연결한 문자열을 생성해서 errorCodes 변수에 할당한다.
 - @Valid형태로만 두는게 아니라 Errors를 이용해서 미리 검증하고 검증되어 걸러진 부분이 존재할 경우 알맞는 ResponseEntity를 리턴한다.
 - HTML대신 JSON 응답이 온다.
 
> @ExceptionHandler 어노테이션을 적용한 방법

```java
package controller;

import java.util.stream.Collectors;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import spring.MemberNotFoundException;

@RestControllerAdvice("controller")
public class ApiExceptionAdvice {
	@ExceptionHandler(MethodArgumentNotValidException.class)
	public ResponseEntity<ErrorResponse> handleBindException(MethodArgumentNotValidException ex) {
		String errorCodes = ex.getBindingResult().getAllErrors()
				.stream()
				.map(error -> error.getCodes()[0])
				.collect(Collectors.joining(","));
		return ResponseEntity
				.status(HttpStatus.BAD_REQUEST)
				.body(new ErrorResponse("errorCodes = " + errorCodes));
	}

}
```
 - @Valid 어노테이션을 붙인 객체의 검증에 실패했을 때 Errors 타입 파라미터가 존재하지 않으면 MethodArgumentNotValidException이 발생한다.