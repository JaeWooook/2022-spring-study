# MVC4 : 날짜값 변환, @PathVariable, 익셉션 처리

<br>

## 커맨드 객체 Date 타입 프로퍼티 변환 처리 : @DateTimeFormat

<BR>

- "2022080915" 문자열을 LocalDateTime 타입으로 변환하고 싶다면?

    스프링은 Long이나 int 같은 기본 데이터 타입으로의 변환은 기본적으로 처리하지만 LocalDateTime 타입으로 변환은 추가 설정 필요함!

    ➡️ @DateTimeFormat 어노테이션 적용!
    ```
    🏷️ ListCommand.java

    @DateTimeFormat(pattern = "yyyyMMddHH")
    private LocalDateTime from;
    @DateTimeFormat(pattern = "yyyyMMddHH")
    private LocalDateTime to;
    ```

    ```
    🏷️ formatDateTime.tag

    <% if (pattern==null) pattern="yyyy-MM-dd"; %>
    <%= DateTimeFormatter.ofPattern(pattern).format(value) %> 
    ```
    : jstl이 제공하는 날짜 형식 태그는 자바 8의  LocalDateTime 타입 지원x, 태그 파일 이용해 LocalDateTime 값을 지정한 형식으로 출력!


    <br>

    ```
    🏷️ memberList.jsp

    <label>from:<form:input path="from" /></label>
    ...
    <label>to:<form:input path="to" /></label>
    ```
    : 스프링 폼 태그는 커맨드 객체의 프로퍼티 값을 출력할때 @DateTimeFormat 어노테이션에 설정한 패턴을 사용해 값 출력

    ∴ from,to 프로퍼티 모두 @DateTimeFormat(pattern = "yyyyMMddHH") 설정되어 있으므로 ```<input>``` 태그에 사용할 값을 생성할때 이 형식으로 값 출력함!


➕ 웹 브라우저에서 주소를 입력해 들어오면 프로퍼티 값이 존재하지 않음!

따라서 해당 프로퍼티들이 null이 아닐때만 데이터를 읽어오게 해야 함

```
@RequestMapping("/members")
    public String list(@ModelAttribute("cmd") ListCommand listCommand, Errors errors , Model model){
        if(errors.hasErrors()){
            return "member/memberList";
        }

        if(listCommand.getFrom()!=null && listCommand.getTo()!=null){
            List<Member> members = memberDao.selectByRegdate(
                    listCommand.getFrom(), listCommand.getTo()
            );
            model.addAttribute("members",members);
        }
        return "member/memberList";
    }
```

➕ 변환 에러 처리

해당 형식에 대해 잘못된 형식으로 입력하면 400 에러 발생

400 에러 대신 알맞은 에러메세지를 보여주고 싶다면 Errors 타입 파라미터를 요청 매핑 어노테이션 적용 메소드에 추가
```
@RequestMapping("/members")
    public String list(@ModelAttribute("cmd") ListCommand listCommand, Errors errors , Model model){
        if(errors.hasErrors()){
            return "member/memberList";
        }
```

jsp 파일에 ```<form:errors>``` 적용

```
<label>from:<form:input path="from" /></label>
<form:errors path="from"/>
```


<br><br>

### 🔎 변환 처리

* WebDataBinder : 요청 파라미터와 커맨드 객체 사이의 변환 처리

![image](https://user-images.githubusercontent.com/81572478/183580971-4bc8444c-a088-4f85-bf1f-f044536d834d.png)


1. DispatchrServlet은 컨트롤러의 요청 매핑 어노테이션 적용 메소드를 실행하기 위해 RequestMappingHandlerAdapter 객체를 사용함

2. 핸들러 어댑터 객체는 요청 파리미터와 커맨드 객체 사이의 변환처리 위해 WebDataBinder 이용

3. WebDataBinder는 커맨드 객체 생성, 커맨드 객체의 프로퍼티와 같은 이름 갖는 요청 파라미터 이용해 프로퍼티 값 생성

4. WebDataBinder는 직접 타입을 변환하지 않고 **ConversionService**에게 역할 위임 

    (@EnableWebMvc 어노테이션 사용하면 DefaultFormattingConversionService를 ConversionService로 사용 가능)



<br><Br>


## @PathVariable을 이용한 경로 변수 처리

<br>

@PathVariable : 경로의 일부가 고정되어 있지 않고 달라질 때 (가변경로) 사용할 수 있음

```
@GetMapping("/members/{id}")
    public String detail(@PathVariable("id") Long memId, Model model){
        Member member = memberDao.selectById(memId);
        if(member==null){
            throw new MemberNotFoundException();
        }
        model.addAttribute("member",member);
        return "member/memberDetail";
    }
```

- 경로변수 : {경로변수}와 같이 중괄호로 둘러쌓인 부분으로, @PathVariable 파라미터에 전달됨

∴ "/member/{id}"에서 {id}에 해당하는 부분의 경로 값을 @PathVariable("id") 어노테이션이 적용된 memId에 전달됨(String을 알아서 Long 으로 타입변환해줌)

<br><br>

## 컨트롤러 익셉션 처리

<br>

### 🔎 @ExceptionHandler 어노테이션

컨트롤러에 @ExceptionHandler 어노테이션을 적용한 메소드가 존재하면 그 메소드가 익셉션을 처리함

➡️ 컨트롤러에서 발생한 익셉션을 직접 처리하고 싶다면 @ExceptionHandler 어노테이션을 적용한 메소드를 구현

```

@Controller
public class MemberDetailController {

    ...

    @GetMapping("/members/{id}")
    public String detail(@PathVariable("id") Long memId, Model model){
        ...
        return "member/memberDetail";
    }


    // 변수 값의 타입이 올바르지 않을 때 발생
    @ExceptionHandler(TypeMismatchDataAccessException.class)
    public String handleTypeMismatchException(){
        return "member/invalidId";
    }

    // 없는 회원 조회시 발생
    @ExceptionHandler(MemberNotFoundException.class)
    public String handleNotFoundException(){
        return "member/noMember";
    }
}

```
: 해당 컨트롤러에서 발생한 익셉션만 처리 가능

### 🔎 @ControllerAdvice 어노테이션

다수의 컨트롤러에 동일 타입의 익셉션은 @ControllerAdvice 어노테이션 이용 (빈으로 등록해야함!)

```
@ControllerAdvice("spring") // 지정 범위
public class CommonExceptionHandler{ 

    @ExceptionHandler(RuntimeException.class)
    public String handleRuntimeException(){
        return "error/commonException";
    }
}
```

: spring 패키지와 하위 패키지에 속한 컨트롤러 클래스를 위한 공통 기능 정의


<br>

#### 📌 @ExceptionHandler 적용 메소드의 우선순위

1. 같은 컨트롤러에 위치한 @ExceptionHandler 메소드 중 해당 익셉션을 처리할 수 있는 메소드 검색

2. 같은 클래스에 위치한 메소드가 익셉션을 처리할 수 없을 경우 @ControllerAdvice 클래스에 위치한 @ExceptionHandler 메소드 검색

<br>

#### 📌 @ExceptionHandler 어노테이션 적용 메소드의 파라미터 & 리턴 타입

- 파라미터 
    - HttpServletRequest, HttpServletResponse, HttpSession
    - Model
    - 익셉션


- 리턴 타입
    - ModelAndView
    - String (뷰이름)
    - (@ResponseBody 어노테이션 붙인 경우) 임의 객체
    - ResponseEntity

    
     


