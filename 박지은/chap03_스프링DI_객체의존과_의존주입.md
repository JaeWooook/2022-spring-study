# 스프링 DI

<br>

## 객체 의존

📑 의존 : 한 클래스가 다른 클래스의 메소드를 실행하는 것으로, 의존은 변경에 의해 영향을 받는 관계.


* 의존하는 대상을 구하는 방법

    1️⃣ 객체 직접 생성 BUT 유지보수 관점에서 문제 유발
    ```
    public class MemberRegisterService {
        priavet MemberDao memberDao = new MemberDao();
        ...
    }
    ```
    : MemberDao 클래스가 다른 클래스로 변경된다면 MemberDao를 생성한 모든 클래스의 코드를 변경해야함


    2️⃣ DI를 통한 의존 처리 = 의존 객체를 전달 받는 방식




<br><Br>
## 의존 주입

📑 DI를 통한 의존 처리

```
    public class MemberRegisterService {
        private  MemberDao memberDao;

        public MemberRegisterService(MemberDao memberDao) {
            this.memberDao = memberDao;
        }
        ...
    }
```


: 의존 객체를 직접 생성하지 않고, 생성자를 통해 의존 객체를 전달 받음 
    ➡️  DI(의존 주입) 패턴을 따름


💡 DI를 쓰는 이유는 ?  의존 객체 변경의 유연함때문!


<br>
✍️ 위의 예와 같이 MemberDao 클래스가 다른 클래스(CachedMemberDao)로 바뀐다면?<br>
    : DI를 사용하는 상황에선 MemberDao 객체를 생성하는 코드만 변경하면 됨

```
  MemberDao memberDao = new MemberDao(); 
  MemberRegiseterService regSvc = new MemberRegisterService(memberDao);

  ➡️

  MemberDao memberDao = new CachedMemberDao(); 
  MemberRegiseterService regSvc = new MemberRegisterService(memberDao);
```
  : **Service 클래스 내부의 코드는 전혀 수정하지 않음!**

   

<br>
  
## 객체 조립기 : spring

- 객체를 생성하고 의존 객체를 주입하는 클래스를 따로 작성하는 것이 좋음 
<img width="400" alt="image" src="https://user-images.githubusercontent.com/81572478/176698074-604139b7-d2d7-4fc1-b393-9b631984c153.png">

<br>
1️⃣ 조립하는 클래스 생성 : 객체를 생성하고 의존 객체를 주입하는 기능 제공, 특정 객체가 필요한 곳에 객체 제공 <br>
    ∴ 조립기 클래스의 객체를 생성하는 시점에 사용할 객체가 모두 생성됨, 조립기 클래스의 메소드를 통해 사용할 객체 구함

<br><Br>
2️⃣ 스프링 = DI를 지원하는 조립기
: 클래스를 생성하면 특정 타입의 클래스만 생성하지만 스프링은 범용적

- 설정 파일 : 설정 정보를 나타냄

    -  @Configuration : 설정파일을 만들기 위한/Bean을 등록하기 위한 애노테이션, 스프링의 설정 클래스임을 의미 <Br>
    - @Bean : 스프링 빈 = 해당 메서드가 생성한 객체, Bean 애노테이션 이용해 메소드 이름으로 스프링에 빈 객체 등록함

- 스프링 컨테이너 : 설정 클래스 이용해 객체를 생성하고 의존 객체 주입<br>
    ```
    private static ApplicationContext ctx = null;

    ...

    ctx = new AnnotationConfigApplicationContext(AppCtx.class); 
    //스프링 컨테이너 객체 생성

    ...

    MemberRegisterService regSvc = ctx.getBean("memberRegSvc", MemberRegisterService.class);
    // 메소드 이름 이용해 bean 객체 구함

    ```
