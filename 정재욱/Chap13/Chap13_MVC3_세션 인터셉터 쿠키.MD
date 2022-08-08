## MVC3 : 세션, 인터셉터, 쿠키
### 컨트롤러에서 HttpSession 사용하기
로그인 상태를 유지하는 방법은 크게 HttpSession을 이용하는 방법과 쿠키를 이용하는 방법이 있다.

> 컨트롤러에서 HttpSession을 사용하려면 다음의 2가지 방법 중 한 가지를 사용하면 된다.

 - 요청 매핑 어노테이션 적용 메서드에 HttpSession 파라미터를 추가한다.
 - 요청 매핑 어노테이션 적용 메서드에 HttpServletRequest 파라미터를 추가하고 HttpServletRequest를 이용해서 HttpSession을 구한다.

> 첫번째 방법을 사용한 예시 코드 1
```java
@PostMapping
public String form(LoginCommand loginCommand, Errors errors, HttpSession session) {
    //session을 사용하는 코드
        ...
        }
```
 - 요청 매핑 어노테이션 적용 메서드에 HttpSession 파라미터가 존재할 경우 스프링 MVC는 컨트롤러의 메서드를 호출할 때 HttpSession 객체를 파라미터로 전달한다.
 - HttpSession을 생성하기 전이면 새로운 HttpSession을 생성하고 그렇지 않으면 기존에 존재하는 HttpSession을 전달한다.

> 두번째 방법을 사용한 예시 코드 2
```java
@PostMapping
public String submit(LoginCommand loginCommand, Errors errors, HttpServletRequest req) {
    HttpSession session = req.getSession();
        ...
        }
```
 - 첫 번째 방법은 항상 HttpSession을 생성하지만 두 번째 방법은 필요한 시점에만 HttpSession을 생성할 수 있다.

