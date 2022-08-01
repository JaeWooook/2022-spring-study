# MVC : 요청 매핑, 커맨드 객체, 리다이렉트, 폼 태그, 모델

<BR>

## 1. 요청 매핑
<bR>

- 요청 매핑 어노테이션
    <br>

    - @RequestMapping

        : value는 요청받을 url을 설정, 
        method는 어떤 요청으로 받을지 정의함(GET, POST, PUT, DELETE, PATCH)
    - @GetMapping
    - @PostMapping

<br>

### 🔎 요청 파라미터 접근
: jsp에서 요청 파라미터 값을 전송하면 컨트롤러 메소드에서 요청 파라미터에 접근해야함!

<br>

#### 1. HttpServletRequest

: 컨트롤러 처리 메소드의 파라미터로 HttpServletRequest 타입을 사용하고 getParameter() 메소드 이용해 파라미터 값 구함 

    ```
    @PostMapping("/register/step2")
        public String handleStep2(HttpServletRequest request){
            String agreeParam = request.getParameter("agree");
            if(agreeParam ==  null || !agreeParam.equals("true")){
                return "register/step1";
            }
            return "register/step2";
        }
    ```

    : jsp에서 보낸 form의 파라미터 값 request로 얻을 수 있음

<br>

#### 2. @RequestParam

: 요청 파라미터 개수가 몇개 안되면 간단하게 구할 수 있음

```
@PostMapping("/register/step2")
    public String handleStep2(@RequestParam(value="agree",defaultValue = "false") Boolean agree){
        if(!agree){
            return "register/step1";
        }
        return "register/step2";
    }
```
- value : HTTP 요청 파라미터의 이름 지정
- required : 필수여부 지정
- defaultValue : 요청 파라미터가 값이 없을 시 사용할 문자열 값

<br>

### 🔎 컨트롤러 구현 없는 경로로 매핑

: 특별한 로직이 없고 단순히 뷰 이름만 리턴하는 컨트롤러 클래스 생성은 귀찮음


➡️ WebMvcConfigurer 인터페이스의 addViewController() 메소드 이용

```
@Override
public void addViewControllers(ViewControllerRegistry registry){
    registry.addViewController("/main").setViewName("main");
}
```
→ 그냥 뷰로 연결만 해줌


<br><Br>

## 2. 리다이렉트

: @RequestMapping, @GetMapping 등 요청 관련 어노테이션을 적용한 메소드가 **redirect:**로 시작하는 경로를 리턴하면 나머지 경로를 이용해 리다이렉트 경로 구성

- "/"로 시작하는 경로 : 웹 어플리케이션 경로를 기준으로 함 <Br>

    ex) redirect:/register/step1 <Br>
    웹 어플리케이션 경로인 "/sp-chap11"에 /register/step1연결 = /sp5-chap11/register/step1

- "/"로 시작하지 않는 경로 : 현재 경로를 기준으로 상대경로 사용
- 완전한 url 사용


<br><Br>

## 3. 커맨드 객체

: 클라이언트가 전달해주는 파라미터 데이터를 주입받기 위해 사용되는 객체 (getter, setter가 반드시 있어야 함)

ex) VO, DTO

    @PostMapping("/register/step3")
        public String handleStep3(RegisterRequest regReq){
            try {
                memberRegisterService.regist(regReq);
                return "register/step3";
            }catch (DuplicateMemberException e){
                return "register/step2";
            }
        }

- 커맨드 객체 : RegisterRequest regReq

<br><Br>

### 🔎 커맨드 객체의 역할

1. 컨트롤러에서 view로 바인딩 : view에서 form:form 태그를 사용하는 경우

<br>

2. view에서 컨트롤럴로 바인딩 : view에서 input type="text" 혹은 input type="hidden" 등으로 값을 컨트롤러로 전송하는 경우

    - view의 폼에서 입력한 값을 파라미터(DTO)를 통해 알아서 setter를 불러와 바인딩 시킴(같은 이름이어야 함)

<br>

3. 컨트롤러에서 Mapper.xml로 바인딩 : 커맨드 객체를 통해 #{변수명}과 커맨드 객체의 필드명을 통해 바인딩하는 경우


<br>

### 🔎 뷰 JSP 코드에서 커맨드 객체 사용하기

```
... 
    <p><strong>${registerRequest.name}님</strong> 
        회원 가입을 완료했습니다.</p>
    <p><a href="<c:url value='/main'/>">[첫 화면 이동]</a></p>  
... 
```

  - $registerRequest.name}
    - registerRequest : 커맨드 객체
    - .name : 커맨드객체의 name 필드

<br>
➕ 커맨드 객체 접근할때 이름을 변경하고 싶다면?
    
: @ModelAttribute 어노테이션 사용

```
@PostMapping("/register/step3")
        public String handleStep3(@ModelAttribut("commend") RegisterRequest regReq){
            ...
        }

```

<br>
➕ 커맨드 객체와 스프링 폼 연동

: 폼을 다시 보여줄때 이전에 입력한 값으로 미리 채우기
    
    <input type="text" name="email" id="email" value="${registerRequest.email}">

→ 기존에 입력한 email 값으로 띄워줌

<br>

- ```<form:form>``` 태그와 ```<form:input>``` 태그를 사용한 버전 

    (커맨드 객체가 존재해야 사용 가능 = PostMapping에서 파라미터에 커맨드 객체가 있어야 함)

```
<form:form action="step3" modelAttribute="registerRequest">
    <p>
        <label>이메일:<br>
        <form:input path="email" />
        </label>
    </p>
```

- form : modelAttribute = 커맨드 객체
- input : path = 커맨드 객체의 프로퍼티 지정
- input : value = 커냄드 객체의 프로퍼티의 값
    
    ```<input id="name" name="name" type="text" value="스프링">```


<Br>

🔎 중첩/콜렉션 프로퍼티

* 프로퍼티 : 필드와 메소드간 중간 기능의 클래스 멤버 유형
    - 리스트 타입 프로퍼티 : ```List<String> response```
    - 중첨 프로퍼티 : Respondent타입 프로퍼티 res, res는 다시 age와 location 프로퍼티 가짐


- 프로퍼티이름[인덱스] : 콜렉션 타입 프로퍼티의 값 목록으로 처리
    
    ex) response[0]

- 프로퍼티이름.프로퍼티이름 : 중첩 프로퍼티 값 처리
    
    ex) res.name

<br><Br>

## 3. Model 통해 컨트롤러에서 뷰로 데이터 전달

```
 @GetMapping
    public ModelAndView form(Model model){
        List<Question> questions = createQuestions();
        model.addAttribute("questions",questions);

        return "survey/surveyForm";
    }

    → "questions"라는 이름으로 questions 객체를 모델 태워 보냄


    <form method="post">
    <c:forEach var="q" items="${questions}" varStatus="status">
        ...
    </c:forEach>

    <p>
        <label>응답자 위치:<br>
        <input type="text" name="res.location">
        </label>
    </p>

    ...
    
    <input type="submit" value="전송">
    </form>

    → ${questions}로 값 받아서 q라는 이름으로 사용
```

<Br>

🔎 ModelAndView를 통한 뷰 선택과 모델 전달

- model 이용해 뷰에 전달할 데이터 설정
- 결과를 보여줄 뷰 이름 리턴

    ➡️ ModelAndView는 한방에 처리

```
@GetMapping
public ModelAndView form(Model model){
    List<Question> questions = createQuestions();
    ModelAndView mav = new ModelAndView();
    mav.addObject("questions",questions); // 데이터 설정
    mav.setViewName("survey/surveyForm"); // 뷰 설정

    return mav;
}
```
<br><Br>

## 4. 주요 폼태그

1. ```<form:form>```
- method 기본 값: POST
- action 기본 값 : 현재 요청 URL
- id 속성값 : 커맨드 객체의 이름 설정
    
    ➕ 만약 커맨드 객체 이름값이 기본값인 "command"가 아니라면 modelAttribute로 커맨드 객체 이름 설정해야 함
       
    ``` 
    <form:form modelAttribute="loginCommand"> 
    ...
    <input type="text" name="id" value="${loginCommand.id}/>
    </form:form>
    ```

<br>

2. ```<form:input> , <form:password>, <form:hidden>```

- path 속성: 연결할 커맨드 객체의 프로퍼티 지정

    ```
    <form:input path="email">
    
    == <input id="email" name="email" type="text" value="">

    - id,name : 프로퍼티 이름
    - value : path로 지정한 커맨드 객체의 프로퍼티 값 출력 (현재는 저장된게 없음)
    ```


<BR><BR>

## 📌 주요 에러 발생

1. 요청 매핑 어노테이션 관련 주요 익셉션

- 404 에러 
    - controller 존재 x 혹은  WebMvcConfigurer 관련 설정이 없을 때
    - jsp 파일이 존재 하지 않을때

<br>

- 405 에러
    - 지원하지 않는 방식으로 연결 : post 방식만 있는데 get방식으로 요청할때

<br>

- 400 에러
    - @RequestParam 어노테이션을 처리하는데 파라미터는 존재하지 않는데 기본값도 없을때

    <br>

    ```
    public String handleStep2(@RequestParam(value="agree") Boolean agree){
       ...
    }
    : 동의하지 않아서 파라미터가 넘어오지 않음
    ```

    - @RequestParam의 값이 bool타입인데 넘어온 값의 타입이 다를때

    <br>

    ```
    <input tyep="checkbox" name="agree" value="true1">
    : 보내는 값 string

    public String handleStep2(@RequestParam(value="agree") Boolean agree){
       ...
    }
    : 받는 값 bool
    ```

    → 비슷한 400 에러: 요청 파라미터의 값을 커맨드 객체에 복사하는 과정도 발생가능



    <br><Br>