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


#### 날짜 형식 변환 처리 : 기본 적용 설정

#### 응답 데이터의 컨텐츠 형식

### @RequestBody로 JSON 요청 처리

#### JSON 데이터의 날짜 형식 다루기

#### 요청 객체 검증하기

### ResponseEntity로 객체 리턴하고 응답 코드 지정하기

#### ResponseEntity를 이용한 응답 데이터 처리

#### @ExceptionHandler 적용 메서드에서 ResponseEntity로 응답하기

#### @Valid 에러 결과를 JSON으로 응답하기

