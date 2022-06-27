# 스프링의 DI 설정
## ❗스프링을 이용한 객체 조립

스프링은 Assembler 클래스의 생성자 코드처럼 필요한 객체를 생성하고 생성한 객체에 의존을 주입한다. 스프링은 Assembler#getMemberRegisterService() 메서드처럼 객체를 제공하는 기능을 정의하고 있다. 차이점은 Assembler는 MemberRegisterService나 MemberDao와 같이 특정 타입의 클래스만 생성한 반면 **스프링은 범용 조립기**라는 점이다. 

스프링을 사용하려면 스프링이 어떤 객체를 생성하고 의존을 어떻게 주입할지 정의한 설정 정보를 작성해야 한다. 
#### 설정 정보
```java
package config;
@Configuration
public class AppCtx{
	@Bean
    public MemberDao memberDao(){
    	return new MemberDao();
    }
    @Bean
    public MemberRegisterService memberRegSvc(){
    	//memberDao()가 생성한 객체를 MemberRegisterService 생성자를 통해 주입한다.
    	return new MebmerReigsterService(memberDao());
    }
    @Bean
    public ChangePasswordService changePwdSvc(){
    	ChangePasswordService pwdSvc = new ChangePasswordService();
        //setMemberDao() 메서드를 통해 의존 객체를 주입한다.
        pwdSvc.setMemberDao(memberDao());
        return pwdSvc;
    }
}
```
- @Configuration: 스프링 설정 클래스를 의미한다. 
- @Bean: 해당 메서드가 생성한 객체를 스프링 빈이라고 설정한다. 각각의 메서드마다 한 개의 빈 객체를 생성한다. 이때 메서드 이름은 빈 객체의 이름으로 사용한다. memberDao() 메서드를 이용해서 생성한 빈 객체는 "memberDao"라는 이름으로 스프링에 등록된다. 


**객체를 생성하고 의존 객체를 주입하는 것은 스프링 컨테이너이다.** 따라서 설정 클래스를 이요해서 컨테이너를 생성해야 한다.
#### 스프링 컨테이너 생성
```java
ApplicationContext ctx = new AnnotationConfigApplicationContext(AppCtx.class);
```
컨테이너를 생성하면 **getBean()**메서드를 이용해서 사용할 객체를 구할 수 있다. 아래 코드는 스프링 컨테이너 ctx로 부터 이름이 "memberRegSvc"인 빈 객체를 구한다.
```java
MemberRegisterService regScv = ctx.getBean("memberRegSvc",MemberRegisterService.class);
```

#### MainForSpring
```java
package main;
public class MainForSpring{
	private static ApplicationContest ctx = null;
    public static void main(String[] args) throws IOException{
    	ctx = new AnnotationConfigApplicationContext(AppCtx.class);
        ...
    }
    private static void processNewConmmand(String[] arg){
    	...
        MemberRegisterSerivce regSvc = ctx.getBean("mebmerRegSvc",MemberRegisterService.class);
        ...
    }
    private static void processChangeCommand(String[] arg){
    	ChangePasswordService changePwdSvc = ctx.getBean("changePwdSvc",ChangePasswordService.class);
    }
}
```
MainForSpring 클래서가 MainForAssembler 클래스와 다른 점은 Assembler 클래스 대신 **스프링 컨테이너인 ApplicationContext를 사용했다는 것이다.**

## ❗ DI 방식
### 1. 생성자 방식

```java
public class MemberRegisterServcie{
	private MemberDao memberDao;
    public MemberRegisterServcie(MemberDao memberDao){
    	this.memberDao = memberDao;
    }
}
```
생성자를 이용해서 의존 객체를 주입하기 위해 해당 설정을 담은 메서드를 호출한다.
```java
@Bean
public MemberDao memberDao(){
	return new MemberDao();
}
@Bean
public MemberRegisterServcie memberRegSvc(){
	return new MemberRegisterServcie(memberDao());
}
```
생성자에 전달한 의존 객체가 두 개 이상이어도 동일한 방식으로 주입하면 된다. MemberDao 클래스의 selectAll() 메서드를 추가한다.
```java
public class MemberDao{
	public Collection<Member> selectAll(){
    	return map.values();
    }
}
```

### 2. setter 메서드 방식
지정한 이메일을 갖는 Member를 찾아서 정보를 콘솔에 출력하는 클래스이다.
```java
public class MemberInfoPrinter{
	private MemberDao memberDao;
    private MemberPrinter printer;
    public void printMemberInfoPrinter(String email){
    	Member member = memberDao.selectByEmail(email);
        if (member==null){
        	System.out.print("No Data\n");
            return;
        }
        printer.print(member);
        System.out.println();
    }
    //memberDao 타입의 객체 의존 주입
    public void setMemberDao(MemberDao memberDao){
    	this.memberDao = memberDao;
    }		
    //MemberPrinter 타입의 객체 의존 주입
    public void setPrinter(MemberPrinter printer){
    	this.printer = printer;
    }
}	
```

setter 메서드를 이용해서 의존을 주입하는 설정 코드를 AppCtx 클래스에 추가한다.
```java
@Configuration
public class AppCtx{
	...
    @Bean
    public MemberInfoPrinter infoPrinter(){
    	MemberInfoPrinter infoPrinter = new MemberInfoPrinter();
        infoPrinter.setMebmerDao(memberDao());
        infoPrinter.setPrinter(mebmerPrinter());
        return infoPrinter;
    }
}	
```

### 🔻 생성자 vs setter 메서드
1. **생성자 방식**: 빈 객체를 생성하는 시점에 모든 의존 객체가 주입된다.
	🔹 **장점**
    빈 객체를 생성하는 시점에 필요한 모든 의존 객체를 주입받기 때문에 객체를 사용할 때 완전한 상태로 사용할 수 있다. 
    🔸 **단점**
    생성자의 파라미터가 많을 경우 각 인자가 어떤 의존 객체를 설정하는지 알아내려면 생성자의 코드를 확인해야 한다.
    
2. **setter 메서드 방식**: setter 메서드 이름을 통해 어떤 의존 객체가 주입되는지 알 수 있다. 
	🔹 **장점**
    메서드 이름만으로도 어떤 의존 객체를 설정하는지 쉽게 유추할 수 있다. 
    🔸 **단점**
    setter 메서드를 사용해서 필요한 의존 객체를 전달하지 않아도 빈 객체가 생성되기 때문에 객체를 사용하는 시점에 NullPointerException이 발생할 수 있다. 
    
    
### 3. 기본 데이터 타입 값 설정
```java
public class VersionPrinter{
	private int majorVersion;
    private int minorVersion;
   	public void print(){
    	System.out.printf("This program version is %d.%d\n",majorVersion,minorVersion);
    }
    public void setMajorVersion(int majorVersion){
    	this.majorVersion = majorVersion;
    }
    public void setMinorVersion(int minorVersion){
    	this.minorVersion = minorVersion;
    }
}
```
```java
@Configuration AppCtx{
	...
    @Bean
    public VersionPrinter versionPrinter(){
    	VersionPrinter versionPrinter = new VersionPrinter();
        versionPrinter.setMajorVersion(5);
        versionPrinter.setMinorVersion(0);
        return versionPrinter;
    }
}
```
int,long과 같은 기본 데이터 타입과 String 타입의 값은 일반 코드처럼 값을 설정하면 된다. 