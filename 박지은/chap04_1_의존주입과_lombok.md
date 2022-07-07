# 의존 주입과 lombok


## 1️⃣ 의존 주입 방법

<br>

### 1. 생성자 주입

- 생성자 호출 시점에 딱 1번만 호출하는 것이 보장


- 불변, 필수 의존 관계에 사용
    
    - 불변 : getter/setter가 필요없음 
    - 필수 : final이 붙으면 반드시 값이 필요 


- 생성자가 1개일땐 **Autowired 생략** 가능

```
public class MemberRegisterService {

    private final MemberDao memberDao; 

    @Autowired //생성자 주입 : 생성자가 1개이므로 생략가능
    public MemberRegisterService(MemberDao memberDao) {
        this.memberDao = memberDao;
    }

   ...
}

```

### 2. 수정자 주입


- 선택, 변경 의존 관계에서 사용
    
    - 선택: required = false로 주입할 대상이 없어도 동작 가능
    - 변경: setter를 호출해 의존관계를 변경 가능


### 3. 필드/ 일반 메소드 주입

: 잘 사용 x, 테스트 코드에서나 사용하는 것이 좋음

<br><Br>

## 2️⃣ 생성자 주입의 장점

1. 불변 : 대부분의 의존 관계는 애플리케이션 종료전까지 변하지 않음
2. 누락 : 생성자 주입을 쓰면 final 키워드를 넣을 수 있는데, final을 통해 생성자에서 값이 설정되지 않는 오류를 컴파일 시점에 막아줌

<br><Br>

## 3️⃣ lombok 라이브러리 
: Java 라이브러리로 반복되는 getter, setter, toString 등의 메서드 작성 코드를 줄여주는 코드 다이어트 라이브러리


### 기존 방식의 단점
1. 멤버변수를 제어하기 위해 모델 객체마다 반복적으로 메소드들을 생성

2. 변수명이 바뀌면 다시 만들어야 함


### → 롬복 사용 : Annotation을 사용해서 자동으로 작성

- 코딩 과정에선 lombok 관련 annotation만 보이고 getter, setter, toString 등 메소드는 보이지 않음 

    but 실제 컴파일된 결과물(.class)에는 코드가 생성되어 있음!

<br>

- annotation 종류

    - @RequiredArgsContructor 

        : 객체에 final, @NotNull이 붙은 필드(필수 값)를 파라미터로 받는 생성자 생성

    - @EqualsAndHashCode
        
        : 해당 객체의 equals(), hashcode() 메소드 생성

    - @Data
        : @Getter, @Setter, @ToString, @EqualsAndHashCode, @RequiredArgsConstructor를 모두 포함


<br><Br>

### 🔎 lombok 사용 예시



```
import org.springframework.beans.factory.annotation.Autowired;

import java.time.LocalDateTime;

public class MemberRegisterService {
     
    private final MemberDao memberDao;

    @Autowired  // 생략가능
    public MemberRegisterService(MemberDao memberDao) {
        this.memberDao = memberDao;
    } 

    ...
}

```


➡️ lombok 사용

```
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Autowired;

import java.time.LocalDateTime;

@RequiredArgsConstructor 
public class MemberRegisterService {

    private final MemberDao memberDao;


//    @Autowired : 생성자 1개이므로 생략
//    public MemberRegisterService(MemberDao memberDao) {
//        this.memberDao = memberDao;
//    } 
//  lombok 사용으로 인해 생성자 생략


  ...
}

```

### 📌 lombok 사용시 주의점
<Br>

#### 1. @AllArgsConstructor, @RequiredArgsContstructor 사용 금지

 - @AllArgsConstructor : 객체 내부의 인스턴스멤버들을 모두 가지고 있는 생성자를 생성하는 Lombok 어노테이션

 - @RequiredArgsConstructor : 객체 내부의 final, @Notnull 이 붙은 인스턴스멤버들을 가지고 있는 생성자를 생성하는 Lombok 어노테이션

 : 두 annotation은 인스턴스 멤버의 순서가 중요함. 멤버 순서가 생성자의 **파라미터 순서**가 되므로!

<br>

 ```
@RequiredArgsConstructor
public class Person {
 
    private final String lastName;
    private final String firstName;
 
}

→ 객체 생성시 Person person = new Person("성", "이름");으로 생성됨
 ```

이때, 인스턴스 멤버의 순서가 바뀐다면?

    인스턴스 멤버는 동일한 타입이기 때문에 컴파일 시, 오류는 발생하지 않음 
    but 성과 이름이 바뀌게 됨.

∴ 생성자를 (IDE generate등으로) 직접 만들고 필요할 경우에 직접 만든 생성자에 @Builder 애노테이션을 붙이는 것을 권장.

➡️ 파라미터 순서가 아닌 이름으로 값을 설정하기 때문에 리팩토링에 유연하게 대응 가능

<br>

- @Builder : 빌더 패턴을 lombok이 어노테이션으로 만듦
```
Person person= Person.builder()
             .lastName("성")
             .firstName("이름").build();
```


<br>

- @RequiredArgsConstructor 를 @Builder로 변경

```
public class MemberRegisterService {

    private final MemberDao memberDao;

    @Builder
    public MemberRegisterService(MemberDao memberDao) {
        this.memberDao = memberDao;
    }

    ...
}

```
<br>

#### 2. @EqualsAndHashCode의 무분별한 사용  


<br>

#### 3. @Data 사용 금지
→  @EqualsAndHashCode와 @RequiredArgsConstructor를 포함하기 때문에 사용 금지

<Br>

#### 4. @Value 사용 금지
<br>

#### 5. @Builder를 클래스보다는 직접 만든 생성자, static 객체 생성 메소드에 붙이는 것을 권장
<Br>

#### 6. Log는 가급적 @Slf4j를 사용하자.







