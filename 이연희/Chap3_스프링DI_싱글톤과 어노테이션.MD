# Dependency Injection - 싱글톤과 어노테이션
## 싱글톤
#### 설정 정보 AppCtx class
```java
@Configuration
public class AppCtx{
	@Bean
    public MemberDao memberDao(){
    	return new MemberDao();
    }
    @Bean
    public MemberRegisterService memberRegSvc(){
    	return new MemberRegisterService(memberDao());
    }
    @Bean
    public ChangePasswordService changePwdSvc(){
		ChangePasswordService pwdSvc = new ChangePasswordService();
        pwdSvc.setMemberDao(memberDao());
        return pwdSvc;
	}
}
```
**Q**❓ memberDao()가 새로운 MemberDao 객체를 생성해서 리턴하므로 memberRegSvc()에서 생성한 MemberRegisterService 객체와 changePwdSvc()에서 생성한 ChangePasswordService 객체는 서로 다른 MemberDao 객체를 사용하는 것이 아닌가?

**A**💡 스프링 컨테이너는 싱글톤 객체이다. 스프링 컨테이너는 @Bean이 붙은 메서드에 대해 **한 개의 객체만 생성한다. 이는 다른 설정 메서드에 의해 memberDao()를 몇 번 호출하더라도 항상 같은 객체를 리턴함을 의미한다.**
스프링이 런타임에 생성한 설정 클래스의 memberDao() 메서드는 매번 새로운 객체를 생성하지 않는다. 대신 한 번 생성한 객체를 보관했다가 이후에는 동일한 객체를 리턴한다.
 
## @Autowired Annotation
@Autowired는 스프링의 자동 주입 기능을 위한 어노테이션이다. 이 설정은 의존 주입과 관련이 있다. 스프링 설정 클래스 필드에 @Autowired를 붙이면 해당 타입의 빈을 찾아서 필드에 할당한다. 
```java
@Autowired
private MemberDao memberDao;
```
다른 설정 파일에 정의한 빈을 필드에 할당했으면 설정 메서드에서 이 필드를 사용해서 필요한 빈을 주입하면 된다. 
스프링 컨테이너는 설정 클래스에서 사용한 @Autowired에 대해서도 자동 주입을 처리한다. 스프링은 @Configuration 어노테이션이 붙은 설정 클래스를 내부적으로 스프링 빈에 등록한다.

#### 설정 파일1 AppConf1 class
```java
@Configuration
public class AppConf1{
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
즉, 컨테이너는 AppConf2 객체를 빈으로 등록하고 @Autowired 어노테이션이 붙은 두 필드(memberDao, memberPrinter)에 해당 타입의 빈 객체를 주입한다.
#### 설정 파일2 AppConf2 class
```java
@Configuration
public class AppConf2{
	@Autowired
    private MemberDao memberDao;
    @Autowired
    private MemberPrinter memberPrinter;
    
    @Bean
    public MemberRegisterService memberRegSvc(){
    	return new MemberRegisterService(memberDao());
    }
    @Bean
    public ChangePasswordService changePwdSvc(){
		ChangePasswordService pwdSvc = new ChangePasswordService();
        pwdSvc.setMemberDao(memberDao());
        return pwdSvc;
	}
    ...
}
```
설정 클래스가 두 개 이상이어도 스프링 컨테이너를 생성하는 코드는 아래과 같이 파라미터로 설정 클래스를 추가로 전달하면 된다.
```java
ctx = new AnnotationConfigApplicationContext(AppConf1.class, AppConf2.class);
```

### @Import Annotation
두 개 이상의 설정 파일을 사용하는 또 다른 방법은 @Import 어노테이션을 사용하는 것이다. 
AppConfImport 설정 클래스를 사용하면 @Import 어노테이션으로 지정한 AppConf2 설정 클래스도 함께 사용하기 때문에 AppConf2 설정 클래스를 지정할 필요가 없다.

#### 설정 클래스 AppConfImport class
```java
@Configuration
@Import(AppConf2.class)
public class AppConfImport{
	@Bean
    public MemberDao memberDao(){
    	rerturn new MemberDao();
    }
    @Bean
    public MemberPrinter memberPrinter(){
		return new MemberPrinter();
	}
}
```
AppConfImport 클래스만 사용하면 AppConf2 클래스의 설정도 함께 사용해서 컨테이너를 초기화한다.
#### MainForImport class
```java
public class MainForSpring{
	private static ApplicationCOntext ctx = null;
    public static void main(String[] args) throws IOExceprion{
    	ctx=new AnnotationConfigApplicationContext(AppConfImport.class);
        ...
    }
}
```
@Import 어노테이션은 배열을 이용해서 두 개 이상의 설정 클래스도 지정할 수 있다.
```java
@Configuration
@Import({AppConfig1.class, AppConf2.class})
public class AppConfImport{
	...
}
```