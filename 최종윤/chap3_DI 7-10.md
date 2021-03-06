
7. @Configuration 설정 클래스와 @Bean 설정과 싱글톤
8. 두 개 이상의 설정 파일 사용하기
9. 1getBean()메서드 사용 
10. 주입 대상 객체를 모두 빈 객체로 설정해야 하나?

### 7.@Configuration 설정 클래스와 @Bean 설정과 싱글톤
```
memberDao() 메서드는 실행될 때 마다 다른 객체를 생성해서 리턴하지만
@Configuration  설정 파일내에서 같은 객체를 생성한다.  
스프링 컨테이너는 @Bean이 붙은 메서드에 대해 한 개의 빈 객체만 생성한다. 
설정 클래스를 그대로 사용하는 것이 아닌 설정클래스를 상속한 새로운 설정 클래스를 만들어 사용하는데,
public class AppCtxExt extends Appctx{
	pricate Map<String,Object>beans =...;
@Override
public MemberDao memberDao(){
if(!beans.containsKey("memberDao"))     memberDao 라는 이름의 빈 객체가 처음 생성된 경우 beans에 보관한다. 
beans.put("memberDao",super.memberDao());  한번 생성한 객체는 beans에 보관한다. 
return (MemberDao)beans.get("memberDao");  보관되어있는 빈객체를 리턴한다. 
}
-PwdSvc()와 RegSvc() 에서 호출한 memberDao()는 beans에 저장되어있는 memberDao를 꺼낸다. - 동일 객체 사용
```

### 8. 두 개 이상의 설정 파일 사용하기
```
설정하는 빈의 개수가 증가하면 두개 이상의 설정 파일을 만들어 나누어 관리하면 편리해진다. 
Appctx1{
@Bean
public MemberDao memberDao(){
return new MemberDao();
}

AppCtx2{
@Autowired - 해당 타입의 빈을 찾아서 필드에 할당한다. 
private MemverDao memberDao;
설정 클래스 2개 컨테이너 
ctx = new AnnotationConfigApplicationContext(AppConf1.class,AppConf2.class);
```

- 8.1configuration
```
@autowired -설정 클래스에서 사용한다 . 
- 설정클래스로 스프링 컨테이너를 만든다.
- 스프링 컨테이너는 빈을 객체 생성 관리한다.
-설정 클래스에서 @Autowired를 이용하여 의존 객체를 세터 메서드를 이용하지 않아도 자동으로 주입한다. 
스프링 빈에 의존하는 다른 빈을 자동으로 주입하고 싶을 때 사용
class MemberPrinterP
@Autowired
private MemberDao memDao;
@Bean
public MemberInfoPrinter infoPrinter(){
MemberInfoPrinter infoPrinter = new MemberInfoPrinter();
-setter injection 안해도 자동으로 의존 주입 
return infoprinter;
```
```
8.2
Import를 사용해 함께 사용할 클래스를 지정하면 Autowired등 을 하지 않아도 지정할 필요없다
@Configuration
@Import(AppConf2.class)
public class AppConfImport{
	...
}
```

### 9.1getBean()메서드 사용 
```
VersionPrinter versionPrinter = ctx.getBean("versionPrinter",VersionPrinter.class);
getBean 메서드의 첫번째 인자는 이름이고 두번째는 빈객체의 클래스 타입이다.
다른 이름을 인자로주면 NoSuchBeanDefinitionException 이 발생한다. 타입을 다르게 주면 notOfRequiredType이 발생하고
실제 타입을 알려준다. 이 름을 전달하지 않고 타입만으로 빈을 구할 수도 있다.
이때 빈 객체가 존재하지 않으면  no qualifying bean of typ MemberPrinter  라며 익셉션이 발생한다. 
클래스 타입으로만 getBean() 실행할 때 빈객체가 두개 이상인 경우 NoUniqueBean Exception이 발생한다. 
```

### 10.주입 대상 객체를 모두 빈 객체로 설정해야 하나?
```
설정 클래스 파일에 빈으로 등록하지 않은 객체를 생성하여 주입할 수 있다. 
단 일반 객체는 스프링 컨테이너를 통해 구할 수 없다. 
ctx.getBean(MemberPrinter.class); -> Exception 

스프링 컨테이너는 자동 주입, 라이프사이클 관리,등 단순 객체 생성 외에 객체 관리를 다양한 기능을 제공한다. 
스프링 컨테이너의 기능이 필요없고 getBean()메서드로 구할 필요가 없는경우 꼭 빈으로 등록할 필요는 없지만
자동주입 기능을 사용하는 추세이기 때문에  의존주입 대상은 스프링 빈으로 등록하는 것이 보통이다 . 
```
