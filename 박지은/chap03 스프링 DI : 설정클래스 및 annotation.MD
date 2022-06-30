# 스프링 DI

<br>

## 설정클래스 및 annotation

1. **@Configuration** : 설정파일을 만들기 위한/Bean을 등록하기 위한 애노테이션, 스프링의 설정 클래스임을 의미 (설정클래스 = bean)<br><Br>
2. **@Bean** : 스프링 빈 = 해당 메서드가 생성한 객체, Bean 애노테이션 이용해 메소드 이름으로 스프링에 빈 객체 등록함. <Br><Br>
3. **@Autowired** : 스프링 빈에 의존하는 다른 빈을 **자동으로** 주입, @Autowired를 사용하면 @Bean 메소드에서 의존 주입 코드 작성x  <br> <Br>
4. **@Import** : 두개 이상의 설정 파일일때 사용. 다른 설정 파일을 import하면 스프링 컨테이너에 **최상위 설정 클래스** 한 개만 사용

💡 스프링 컨테이너가 생성한 빈은 **싱글톤** 객체
   : @Bean이 붙은 메소드에 대해 **한 개**의 객체만 생성 = 항상 **같은** 객체 리턴<br>

    ➡️ 한 번 생성한 객체를 보관했다가 이후에 동일한 객체를 리턴


<br>

### 두 개 이상의 설정파일 

- AppConfig1
```
@Configuration
public class AppConf1 {

    @Bean
    public MemberDao memberDao(){
        return new MemberDao();
    }

    @Bean
    public MemberPrinter memberPrinter(){
        return new MemberPrinter();
    }
}
```
- AppConfig2
```
@Configuration
public class AppConf2 {
    @Autowired //스프링의 자동 주입기능
    private MemberDao memberDao;

    @Autowired // AppConfig1에 설정된 빈을 자동으로 찾아 할당
    private MemberPrinter memberPrinter;

    @Bean
    public MemberRegisterService memberRegSvc(){
        return new MemberRegisterService(memberDao);
    }

```

📌 @Autowired 이용해서 다른 설정 파일에 정의한 빈을 필드에 할당

``` 
@Bean
    public MemberListPrinter listPrinter(){
        return new MemberListPrinter(memberDao, memberPrinter);
    } //필요한 빈 주입

```

### 스프링 컨테이너 생성 <br>

1️⃣ 여러 설정 파일: 파라미터로 설정 클래스 추가 전달!

```
ctx = new AnnotationConfigApplicationContext(AppConfig1.class, AppConfig2.class); 
```

2️⃣ 다중 Import : 최상위 설정 클래스만 사용 = 스프링 컨테이너 코드 변경할 필요 없음!!

```
@Configuration // 최상위 설정 클래스
@Import(AppConf2.class)
public class AppConfImport {

    @Bean
    public MemberDao memberDao(){
        return new MemberDao();
    }

    @Bean
    public MemberPrinter memberPrinter(){
        return new MemberPrinter();
    }

}
 ```

```
... // 스프링 컨테이너 선언
ctx = new AnnotationConfigApplicationContext(AppConfImport.class);
...
 ```

 ### getBean() 메소드 사용

 - getBean("빈 이름", 빈 타입)
1. 해당 빈 이름이 잘못된 경우
2. 해당 빈 타입이 기대한 타입과 다른 경우
3. 해당 빈 타입이 없는 경우

📌 @Bean 애노테이션 사용하지 않고 일반 객체로 선언 : 스프링 컨테이너에서 관리 x = 자동 주입, 라이프사이클 관리 등 객체 관리 기능 사용 x
