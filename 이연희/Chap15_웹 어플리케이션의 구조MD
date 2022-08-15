# Chap15 간단한 웹 어플리케이션 구조
## 웹 어플리케이션의 구성 요성
웹 어플리케이션을 개발할 때 사용하는 전형적인 구조는 다음 요소를 포함한다.
- 프론트 서블릿
- 컨트롤러 + 뷰
- 서비스
- DAO

프론트 서블릿은 웹 브라우저의 모든 요청을 받는 창구 역할을 한다. 프론트 서블릿은 요청을 분석해서 알맞은 컨트롤러에 전달한다. 스프링 MVC에서는 DispatcherServlet이 프론트 서블릿의 역할을 수행한다.![](https://velog.velcdn.com/images/yh_lee/post/1b79cbf2-11fa-44b0-a4fa-d322e0ff7be3/image.png)컨트롤러는 실제 웹 브라우저의 요청을 처리한다. 지금까지 구현해본 스프링 컨트롤러가 이에 해당한다. 컨트롤러는 클라이언트(브라우저)의 요청을 처리하기 위해 알맞은 기능을 실행하고 그 결과를 뷰에 전달한다. 
#### 컨트롤러의 주요 역할
- 클라이언트가 요구한 기능을 실행
- 응답 결과를 생성하는데 필요한 모델 생성
- 응답 결과를 생성할 뷰 선택

컨트롤러는 로직 실행을 서비스에 위임한다. 컨트롤러는 어플리케이션이 제공하는 기능과 사용자 요청을 연결하는 매개체이고, 기능 제공을 위한 로직을 직접 수행하지는 않는다. 앞에서 작성했던 ChangePasswordController의 경우 아래 코드처럼 ChangePasswordService에 비밀번호 변경 처리를 위임했다.
```java
@PostMapping
public String submit(
		@ModelAttribute("command") ChangePwdCommand pwdCmd, 
        Errors errors, 
        HttpSession session){
		
	new ChangePwdCommandValidator().validate(pwdCmd, errors);
    if(errors.hasErrors()){
    	return "edit/changePwdForm";
    }	
    AuthInfo authInfo = (AuthInfo) session.getAttribute("authInfo");
    try{
    	//컨트롤러는 로직 실행을 서비스에 위임
        changePasswordService.changePassword(
        		authInfo.getEmail(),
                pwdCmd.getCurrendPassword(),
                pwdCmd.getNewPassword());
                
        return "edit/changePwd";
    }catch(IdPasswordNotMatchingException e){
    	errors.rejectValue("currentPassword", "notMatching");
        return "edit/changePwdForm";
    }
}
```
서비스는 기능의 로직을 구현한다. 서비스는 DB 연동이 필요하면 DAO(Data Access Object)를 사용한다. DB와 웹 어플리케이션 간에 데이터를 이동시켜 주는 역할을 한다. 

## 서비의 구현
서비스는 핵심이 되는 기능의 로직을 제공한다. 예를 들어 비밀번호 변경 기능은 아래와 같은 로직을 서비스에서 수행한다.
- DB에서 비밀번호를 변경할 회원의 데이터를 구한다.
- 존재하지 않으면 익셉션을 발생시킨다.
- 회원 데이터의 비밀번호를 변경한다.
- 변경 내역을 DB에 반영한다.

웹 어플리케이션을 사용하든 명령행에서 실행하든 비밀번호 변경 기능을 위해서 서비스는 동일한 로직을 수행한다. 중간에 실패하면 이전까지 했던 것을 취소해야 하고, 모든 과정을 성공적으로 진행했을 때 완료해야 한다. 이런 이유로 서비스 메서드를 트랜잭션 범위에서 실행한다. 
```java
@Transactional
public void changePassword(String email, String oldPwd, String newPwd){
	Member member = memberDao.selectByEmail(email);
    if(member==null)
    	throw new MemberNotFoundException();
    
    member.changePassword(oldPwd, newPwd);
    
    memberDao.update(member);
}
```

같은 데이터를 사용하는 기능들을 한 개의 서비스 클래스에 모아서 구현할 수 있다. 예를 들어 회원 가입 기능과 비밀번호 변경 기능은 모두 회원에 대한 기능이므로 다음과 같이 MemberService 클래스에 기능을 구현할 수 있다.
```java
public class MemberService{
	@Transactional
    public void regist(RegisterRequest req){...}
    
    @Transactional 
    public void changePassword(String email, String lodPwd, String newPwd){...}
}
```

회원가입 기능은 RegisterRequest 클래스를 파라미터로 사용했다. 필요한 데이터를 전달받기 위해 별도 타입을 만들면 스프링 MVC의 커맨드 객체로 해당 타입을 사용할 수 있어 편하다. 

회원 가입 요청을 처리하는 컨트롤러 클래스의 코드는 다음과 같이 서비스 메서드의 입력 파라미터로 사용되는 타입을 커맨드 객체로 사용했다. 
```java
@PostMapping("/register/step3")
public String handleSteo3(RegisterRequest regReq, Errors errors){
	...
    memberRegisterService.regist(regReq);
    ...
}
```
비밀번호 변경의 changePassword() 메서드처럼 웹 요청 파라미터를 커맨드 객체로 받고 커맨드 객체의 프로퍼티를 서비스 메서드에 인자로 전달할 수도 있다.
```java
@RequestMapping(method=RequestMethod.POST)
public String submit(@ModelAttribute("command") ChangePwdCommand pwdCmd, Errors errors, HttpSession session){
	...
    changePasswordService.changePassword(
    	authInfo.getEmail(),
        pwdCmd.getCurrentPassword(),
        pwdCmd.getNewPassword()
    );
    ...
}
```

**커맨드 클래스를 작성한 이유는 스프링 MVC가 제공하는 폼 값 바인딩과 검증, 스프링 폼 태그와의 연동 기능을 사용하기 위함이다.**

## 컨트롤러에서 DAO 접근

서비스 메서드에서는 어떤 로직도 수행하지 않고 단순히 DAO의 메서드만 호출하고 끝나는 경우도 있다. 예를 들어 회원 데이터 조회를 위한 서비스 메서드를 다음과 같이 구현하곤 한다.
```java
public class MemberSerivce{
	...
    public Member getMember(Long id){
    	return memberDao.selectById(id);
    }
}
```
여기서 MemberService 클래스의 getMember() 메서드는 MemberDao의 selectByID()만 실행할 뿐 추가 로직은 없다. 컨트롤러 클래스는 이 서비스 메서드를 통해 회원 정보를 구한다.
```java
@RequestMapping("/member/detail/{id}")
public String detail(@PathVariable("id") Long id, Model model){
	//사실상 DAO를 직접 호출하는 것과 동일
    Member member = memberService.getMember(id);
  //Member member = memberDao.selectByEmail(id);
    if(member==null)
    	return "member/noFound";
    model.addAttribute("member",member);
    return "member/memberDatail";
}
```


## 패키지 구성
패키지의 구성을 조금 더 정확하게 분리하면 아래 그림처럼 '웹 요청을 처리하기 위한 것'과 '기능을 제공하기 위한 것'으로 구분할 수 있다.
![](https://velog.velcdn.com/images/yh_lee/post/815c9e52-bcb6-4ca2-84eb-e23afca29ad3/image.png)웹 요청을 처리하기 위한 영역에는 컨트롤러 클래스와 관련 클래스들이 위치한다. 커맨드 객체의 값을 검증하기 위한 Validator도 웹 요청 처리 영역에 위치할 수 있는데 관점에 따라 기능 제공 영역에 위치시킬 수 있다. 웹 영역의 패키지는 web.member와 같이 사용한다.

기능 제공 영역에는 기능 제공에 필요한 서비스, DAO, 그리고 Member와 같은 모델 클래스가 위치한다. 실제 어플리케이션에서는 domain.member와 같이 기능을 잘 표현하는 패키지 이름을 사용한다. 

기능 영역은 다음과 같이 service, dao, model 같은 세부 패키지로 구분하기도 한다. 
![](https://velog.velcdn.com/images/yh_lee/post/db829fb8-491c-4213-aa5c-d27ba197549b/image.png)

>컨트롤러-서비스-DAO 구조는 간단한 웹 어플리케이션을 개발하기에는 무리가 없다. 문제는 로직이 복잡해지면서 구조의 코드도 복잡해지는 경향이 있다. 이를 완화하는 방법 중 하나는 도메인 주도 설계를 적용하는 것이다. 
도메인 주도 설계는 UI-서비스-도메인-인프라의 네 영역으로 어플리케이션을 구성한다. 여기서 UI는 컨트롤러 영역에 대응하고 인프라는 DAO 영역에 대응한다. 중요한 점은 주요한 도메인 모델과 업무 로직이 서비스 영역이 아닌 도메인 영역에 존재하는 것이다. 또한 도메인 영역은 정해진 패턴에 따라 모델을 구현하다. 이를 통해 업무가 복잡해져도 일정 수준의 복잡도로 코드를 유지할 수 있게 한다.
