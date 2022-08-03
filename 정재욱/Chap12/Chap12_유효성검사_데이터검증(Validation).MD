## 유효성 검사 / 데이터 검증 (Validation)

### 유효성 검사란?
 > 서비스의 비즈니스 로직이 올바르게 동작하기 위해 사용되는 데이터에 대한 사전 검증하는 작업이 필요함
 > 
> 유효성 검사 혹은 데이터 검증이라고 부르고, 흔히 Validation이라고 부름
> 
> Validation은 들어오는 데이터에 대해 의도한 형식의 값이 제대로 들어오는지 체크하는 과정

### 일반적인 Validation의 문제점
 - 어플리케이션 전체적으로 분산되어 존재
 - 코드의 중복이 심함
 - 비즈니스 로직에 섞여 있어 검사 로직 추적이 어려움
![](https://velog.velcdn.com/images/mystyle730/post/069ba73d-aa97-40fb-acbc-dfb8d8ae9deb/image.png)

### 해결하기 위한 방법
#### Bean Validation / Hibernate Validator

 - Bean Validation은 어노테이션을 통해 다양한 데이터를 검증할 수 있게 기능을 제공
 - Hibernate Validator는 Bean Validation 명세에 대한 구현체
 - Spring Boot의 유효성 검사 표준은 Hibernate Validator
 - 스프링 부트 2.3이전 버전에는 starter-web에 validation이 포함되어 있지만, 2.3버전 이후 부터는 starter-validation을 추가해야 한다.

## Entity , DAO , DTO , VO 사용하는 이유
### Entity란?
> Entity 클래스는 DB의 테이블에 존재하는 Column들을 필드로 가지는 객체를 말한다.
> __Entity는 DB의 테이블과 1:1로 대응되며,__ 테이블이 가지지 않는 컬럼을 필드로 가져서는 안된다.
> Entity 클래스는 다른 클래스를 상속받거나 인터페이스의 구현체여서는 안된다.

### Entity 예시 코드
```java
@Entity
public class Post {
    private String title;
    private String content;
    private String author;
}
```
 - Post란 테이블에 title, content, author라는 컬럼을 가지고 있다면, 이것에 대응하는 Entity 클래스 Post이다.
 - Post테이블의 컬럼들을 필드로 가진다.
 - Entity는 외부에서 최대한 Entity의 Getter를 사용하지 않도록 내부에서 로직을 구현한다.
 - 이 때 Domain 로직만 구현하고, Presentation 로직은 구현하지 않는다.
 - __Entity는 Setter의 사용을 피해야 한다.__ Entity의 일관성을 해칠 수 있기 때문이다.
 - Setter를 무분별하게 사용하게 되면, Entity의 인스턴스 값들이 언제 어디서 변하는지 명확히 알 수없다.

> Setter 대신 다른 방법으로 필드에 값을 넣어 주는 것이 좋다.
> 
> 일반적으로 인스턴스의 생성 시점에 생성자로 필드에 값을 넣어주는 방법은 좋지못하다.
> 
> __생성자에 현재 넣는 값이 어떤 필드인지 명확히 알 수 없고, 매개변수끼리의 순서가 바뀌더라도 코드가 모두 실행되기 전까지는 문제를 알 수 없다는 단점이 있기 때문이다.__

 - Builder 패턴을 사용하면 멤버 변수가 많아지더라도 어떤 값을 어떤 필드에 넣는지 코드를 통해 확인할 수 있고, 필요한 값만 집어 넣는 것이 가능하기 때문이다.

> Entity 클래스 생성 및 Builder 패턴 예시
```java
@Getter
@Entity
@NoArgsConstructor
public class Membmer member {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;
    private String name;
    private String email;
    @Column(length = 13, nullable = false)
    private String phoneNumber;
 
    @Builder
    public Member(long id, String name, String email, String phoneNumber) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.phoneNumber = phoneNumber;
    }
}

// 사용 방법

Member member = new member.builder()
        .name("홍길동")
        .email("test@gmail.com")
        .phoneNumber("010-1234-5678")
        .build();
```

### DAO란?
> DAO는 Data Access Object로, 실제로 DB에 접근하는 객체다.
> 
> DAO는 프로젝트의 서비스 모델과 실제 데이터베이스를 연결하는 역할을 하며, DB를 사용해 데이터를 조회하거나 조작하는 기능을 전담하도록 만든 Object를 말한다.
> 

### DTO란?
![image](https://user-images.githubusercontent.com/55322459/182524904-a1b9c426-9274-412e-aa5a-b726d9dbefb6.png)

 - DTO는 Data Transfer Object로, 계층 간 데이터 교환 역할을 한다. DB에서 꺼낸 데이터를 저장하는 Entity를 가지고 만드는 일종의 Wrapper라고 볼 수 있다.
 - DTO는 그저 계층간 데이터 교환이 이루어 질 수 있도록 하는 객체이기 때문에, 특별한 로직을 가지지 않는 순수한 데이터 객체여야한다.
 - DTO에는 Setter를 만들 필요가 없고 생성자에서 값을 할당한다.(Entity처럼 Builder 패턴을 이용해서 할당하는 방법도 가능하다.)

> 유효성 검사를 사용한 DTO 코드 예시

```java
@PostMapping("/signup")
public void registerUser(@RequestBody @Valid SignupDTO signupDTO) {
    User user = signupDTO.toEntity(signupDTO);
    Long userId userService.signup(user);
}

public class SignupDTO {
    
    @NotBlank(message = "이메일은 필수값입니다.")
    @Email(message = "이메일 형식이 아닙니다.")
    private String email;
    
    @NotBlank
    @Pattern(regexp="(?=.*[0-9])(?=.*[a-zA-Z])(?=.*\\W)(?=\\S+$).{8,20}", message="비밀번호는 영문 대, 소문자와 숫자, 특수기호가 적어도 1개 이상씩 포함된 8~20자 비밀번호여야 합니다.")
    private String password;
    
    private String name;
    
    @NotBlank(message = "사용자 이름은 필수값 입니다.")
    @Pattern(regexp = "^[a-zA-Z0-9]*$", message="사용자 이름은 영어랑 숫자만 가능합니다.")
    private String nickname;
}
```
 - 위와 같이 DTO에 유효성검사를 미리 체크하면 비즈니스로직을 구현함에 있어서 비즈니스 로직만을 집중하여 구현할 수 있다.

### DTO 와 VO의 차이
VO(Value Object)도 DTO와 동일한 개념이다. 다만 DTO와의 차이는, DTO는 데이터를 계층간 교환 하는데 의미가 있고, VO는 읽기만 가능한 __read-only__ 속성을 가진 객체로서 데이터 그자체에 의미를 두고 있다는 점이다.