# Chap14 MVC4
## @DateTimeFormat
회원이 가입한 일시를 기준으로 회원을 검색하기 위해 시작 시간과 끝 시간 기준을 파라미터로 전달받는다고 하자. 검색 기준 시간을 표현하기 위한 커맨드 클래스이다.
```java
public class ListCommand{
	private LocalDateTime from;
    private LocalDateTime to;
    ...
}	
```
검색을 위한 입력 폼은 다음과 같다.
```
<input type="text" name="from"/>
<input type="text name="to"/>
```
여기서 문제는 <input\>에 입력한 문자열을 LocalDateTime 타입으로 변환해야 한다는 것이다. 이때 @DateTimeFormat을 적용하면 된다. 
```java
public class ListCommand{
	@DateTimeFormat(pattern="yyyyMMddHH")
    private LocalDateTime from;
    @DateTimeFormat(pattern="yyyyMMddHH")
    private LocalDateTime to;
    ...
}
```
예로 "2022081011"라는 문자열을 "2022년 8월 10일 11시" 값을 갖는 LocalDateTiem 객체로 변환해준다. 

<br>
컨트롤러에서는 별도 설정 없이 ListCommand 클래스를 커맨드 객체로 사용하면 된다.

```java
@Controller
public class MemberListController{
	private MemberDao memberDao;
    
    public void setMemberDao(MemberDao memberDao){
    	this.memberDao=memberDao;
    }		
    
    @RequestMapping("/members")
    public String list(@ModelAttribute("cmd") ListCommand listCommand,Model model){
    	if(listCommand.getFrom()!=null&&listCommand.getTo()!=null){
        	List<Member> members = memberDao.selectByRegdate(listCommand.getFrom(),listCommand.getTo());
            model.addAttribute("members",members);
        }
        return "member/memberList";
    }
}
```
## 변환 에러 처리
형식에 맞지 않는 "20220810"을 입력하면 지정한 형식 "yyyyMMddHH"에 맞지 않기 때문에 400 에러가 발생한다. 이때 400 에러 대신 폼에 알맞는 에러 메시지를 보여주고 싶으면 Errors 타입 파라미터를 RequestMapping 적용 메서드에 추가하면 된다. 

```java
@Controller
public class MemberListController{
	@RequestMapping("/members")
    public String list(@ModelAttribute("cmd") ListCommand listCommand,
    					Errors errors, 
                        Model model
                        ){
    	if(errors.hasError()){
        	return "member/memberList";
        }	
        ...
    }
}
```
Errors 타입 파라미터를 가질 경우 @DateTimeFormat에 지정한 형식이 맞지 않으면 Errors 객체에 "typeMismatch" 에러 코드를 추가한다. 

## 변환 처리 이해
@DataTimeFormat을 사용하면 지정한 형식의 문자열을 LocalDateTime 타입으로 변환해준 것을 확인했다. 이때 WebDataBinder가 값 변환에 관여한다. 

스프링 MVC는 메서드와 DispatcherServlet 사이를 연결하기 위해 RequestMappingHandlerAdapter 객체를 사용한다. 이 핸들러 어댑터 객체는 요청 파라미터와 커맨드 객체 사이 변환 처리를 위해 WebDataBinder를 이용한다.

WebDataBinder는 커맨드 객체를 생성한다. 그리고 커맨드 객체의 프로퍼티와 같은 이름을 갖는 요청 파라미터를 이용해서 프로퍼티 값을 생성한다.![](https://velog.velcdn.com/images/yh_lee/post/3ce59162-0dcb-4a05-9b0b-a0a991497993/image.png)
WebDataBinder는 직접 타입을 변환하지 않고 ConversionService에 역할을 위임한다. 스프링 MVC를 위한 설정인 @EnableWebMvc를 사용하면 DefaultFormattingConversionService를 ConversionService로 사용한다.

## @PathVariable 경로 변수 처리
다음은 ID가 10인 회원의 정보를 조회하기 위한 URL이다.

	http://localhost:8080/sp5-chap14/members/10
    
이 형식의 URL을 사용하면 각 회원마다 경로의 마지막 부분이 달라진다. 이때 @PathVariable을 사용하면 가변 경로를 처리할 수 있다.

```java
@Controller
public class MemberDetailController{
	pirvate MemberDao memberDao;
    public void setMemberDao(MemberDao memberDao){
    	this.memberDao=memberDao;
    }	
    
    @GetMapping("/members/{id}")
    public String detail(@PathVariable("id") Long memId, Model model){
    	Member member = memberDao.selectById(memberId);
        if(member=null){
        	throw new MemberNotFoundException();
        }
        model.addAttribute("member",member);
        return "member/memberDetail";
    }
}	
```
매핑 경로에 {id}와 같이 중괄호로 둘러 쌓인 부분을 경로 변수라고 한다. {id}에 들어오는 변수는 @PathVariable 파라미터에 전달된다. 

## 컨트롤러 익셉션 처리
없는 ID를 경로변수로 사용하면 MemberNotFoundException이 발생한다. 숫자가 들어가야 하는데 'a'같은 문자가 변수로 들어가면 400 에러가 발생한다. 에러 발생 화면이 출력 되는데 알맞게 익셉션을 처리해서 사용자에게 더 적합한 안내를 해주는 것이 좋다. 그런데 타입 변환 실패에 따른 익셉션은 어떻게 에러 화면을 보여줄 수 있을까? 이때 @ExceptionHandler를 사용할 수 있다.

같은 컨트롤러에 ExceptionHandler가 붙은 메서드가 있다면 이 메서드가 익셉션을 처리한다. 따라서 컨트롤러에서 발생한 익셉션을 직접 처리하고 싶으면 ExceptionHandler를 붙인 메서드를 구현하면 된다.

```java
@Controller
public class MemberDetailController{
	private MemberDao memberDao;
    ...
    
    @ExceptionHandler(TypeMismatchException.class)
    public String handleTypeMismatchException(){
    	return "member/invalidId";
    }
    
    @ExceptionHandler(MemberNotFoundException.class)
    public String handleTypeMismatchException(){
    	return "member/noMember";
    }
}
```

### @ControllerAdvice를 이용한 공통 익셉션 처리
여러 다른 컨트롤러에서 동일 타입의 익셉션 처리 코드를 적용하고 싶을 때 @ControllerAdvice를 사용해서 중복을 없앨 수 있다.

```java
@ControllerAdvice("spring")
public class CommonExceptionHandler{
	
    @ExceptionHandler(RuntimeException.class)
    public String handleRuntimeException(){
    	return "error/commonException";
    }	
}
```
@ControllerAdvice 어노테이션이 적용된 클래스는 지정한 범위의 컨트롤러에 공통으로 사용될 설정을 지정할 수 있다. 위의 코드는 spring 패키지와 그 하위 패키지에 속한 컨트롤러에서 RuntimeException이 발생하면 익셉션을 처리한다.

### @ExceptionHandler 적용 메서드의 우선 순위
@ControllerAdvice 클래스에 있는 @ExceptionHandler 메서드보다 컨트롤러 클래스에 있는 @ExceptionHandler 메서드가 우선순위가 높기 때문에 먼저 작동한다.

1. 같은 컨트롤러에 위치한 @ExceptionHandler 메서드 중 익셉션을 처리할 수 있는 메서드 검색
2. 같은 클래스에 위치한 메서드가 익셉션을 처리할 수 없는 경우 @ControllerAdvice 클래스에 위치한 @ExceptionHandler 메서드를 검색
3. 

### @ExceptionHandler 적용 메서드의 파라미터와 리턴 타입
@ExceptionHandler를 붙인 메서드는 다음 파라미터를 가질 수 있다.
- HttpServletRequest, HttpServletResponse, HttpSession
- Model
- 익셉션
리턴 타입은 다음과 같다.
- ModelAndView
- String(뷰 이름)
- (@ReponseBody를 붙인 경우) 임의 객체
- ResponseEntity