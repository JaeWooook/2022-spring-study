## 간단한 웹 어플리케이션의 구조

### 간단한 웹 어플리케이션의 구성 요소
> 간단한 웹 어플리케이션을 개발할 때 사용하는 전형적인 구조는 다음 요소를 포함한다.

 - 프론트 서블릿
   - 프론트 서블릿은 웹 브라우저의ㅏ 모든 요청을 받는 창구 역할을 한다.
   - 프론트 서블릿은 요청을 분석해서 알맞은 컨트롤러에 전달한다.
   - 스프링 MVC에서는 DispatcherServlet이 프론트 서블릿의 역할을 수행한다.
 - 컨트롤러 + 뷰
   - 컨트롤러는 실제 웹 브라우저의 요청을 처리한다.
   - 컨트롤러는 클라이언트(브라우저)의 요청을 처리하기 위해 알맞은 기능을 실행하고 그 결과를 뷰에 전달한다.
     - 클라이언트가 요구한 기능을 실행
     - 응답 결과를 생성하는데 필요한 모델 생성
     - 응답 결과를 생성할 뷰 선택
   - 컨트롤러는 어플리케이션이 제공하는 기능과 사용자 요청을 연결하는 매개체로서 기능 제공을 위한 로직을 직접 수행하지 않는다.
   - 해당 로직을 제공하는 서비스에 그 처리를 위임한다.
 - 서비스
   - 서비스는 기능의 로직을 구현한다.
   - ex) 사용자에게 비밀번호 변경 기능을 제공하려면 수정폼을 제공하고, 로그인 여부를 확인하고, 실제로 비밀번호를 변경해야 한다.
   - 서비스는 DB 연동이 필요하면 DAO를 사용한다.
 - DAO
   - DAO는 DB와 웹 어플리케이션 간에 데이터를 이동시켜 주는 역할을 맡는다. 어플리케이션은 DAO를 통해서 DB에 데이터를 추가하거나 DB에서 데이터를 읽어온다.
   - 목록이나 상세 화면과 같이 데이터를 조회하는 기능만 있고 부가적인 로직이 없는 경우에는 컨트롤러에서 직접 DAO를 사용하기도 한다.

### 서비스의 구현

> 서비스는 핵심이 되는 기능의 로직을 제공한다고 했다. 예를 들어 비밀번호 변경 기능은 다음 로직을 서비스에서 수행한다.

 - DB에서 비밀번호를 변경할 회원의 데이터를 구한다.
 - 존재하지 않으면 익셉션을 발생시킨다.
 - 회원 데이터의 비밀번호를 변경한다.
 - 변경 내역을 DB에 반영한다.

위와 같은 과정을 중간 과정에서 실패가 나면 이전 까지 했던 것을 취소해야 하고, 모든 과정을 성공적으로
진행했을 때 완료해야 한다. 이런 이유로 서비스 메서드를 트랜잭션 범위에서 실행해야 한다.

각 서비스 클래스는 기능 제공을 위해 한 개의 public 메서드를 제공하고 있다.
같은 데이터를 사용하는 기능들을 한 개의 서비스 클래스에 모아서 구현할 수도 있다.

필요한 데이터를 전달받기 위해 별도 타입을 만들면 스프링 MVC의 커맨드 객체로 해당 타입을 사용할 수 있어 편하다.
회원 가입 요청을 처리하는 컨트롤러 클래스의 코드는 다음과 같이 서비스 메서드의 입력 파라미터로 사용되는 타입을 커맨드 객체로 사용했다.

```java
@PostMapping("/register/step3")
public String handleStep3(RegisterRequest regReq, Errors errors) {
        ...
        memberRegisterService.regist(regReq);
        }
```

비밀번호 변경의 changePassword() 메서드처럼 웹 요청 파라미터를 커맨드 객체로 받고 커맨드 객체의 프로퍼티를 서비스 메서드에 인자로 전달할 수도 있다.

```java
@RequestMapping(method = RequestMethod.POST)
public String submit(@ModelAttribute("command") ChangePwdCommand pwdCmd, Errors errors, HttpSession session) {
        ...
        changePasswordService.changePassword(
                authInfo.getEmail(),
                pwdCmd.getCurrentPassword(),
                pwdCmd.getNewPassword());
        ...
        }
```
커맨드 클래스를 작성한 이유는 스프링 MVC가 제공하는 폼 값 바인딩과 검증, 스프링 폼 태그와의 연동 기능을 사용하기 위함이다.

#### 서비스 메서드는!
> 기능을 실행한 후에 결과를 알려주어야 한다. 결과는 크게 두가지 방식으로 알려준다.

 - 리턴 값을 이용한 정상 결과
 - 익셉션을 이용한 비정상 결과

위 두가지 예시를 보여주는 예제 코드 AuthService 클래스이다.

```java
public class AuthService {
   ...
   
   public AuthInfo authenticate(String email, String password) {
       Member member = memberDao.selectByEmail(email);
       if(member == null) {
           throw new WrongIdPasswordException();
       }
       if(!member.matchPassword(password)) {
           throw new WrongIdPasswordException();
       }
       return new AuthInfo(member.getId(), member.getEmail(), member.getName());
   }
}
```
 - authenticate() 메서드는 리턴 타입으로 AuthInfo를 사용하고 있다.
 - authenticate() 메서드는 인증에 성공할 경우 인증 정보를 담고 있는 AuthInfo 객체를 리턴해 정상 실행을 알려준다.
 - 하지만, 비밀 번호 변경 처럼 리턴 타입이 void인 경우에는 익셉션이 발생하지 않는 것이 정상적으로 실행됨을 의미한다.
 - 로그인 같은 경우 비밀번호가 다르다는 WrongIdPasswordException을 발생시켜서 인증 실패로 인지한다.
 - 로그인 같은 경우 익셉션이 발생하지 않는다면 로그인을 성공한것으로 인지할 수 있다.

### 컨트롤러에서의 DAO 접근
