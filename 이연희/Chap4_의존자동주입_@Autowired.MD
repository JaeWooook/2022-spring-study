# 의존 자동 주입
## @Autowired
### 필드

아래 코드는 의존 대상을 설정 코드에서 직접 주입한다. 
```java
public class AppCtx{
	@Bean
    public MemberDao memberDao(){
    	return new MemberDao();
    }
    @Bean
    public ChangePasswordService changePwdSvc(){
    	ChangePasswordService pwdSvc = new ChangePasswordServcie();
        pwdSvc.setMemberDao(memberDao());// 의존 주입
        return pwdSvc;
    }
}
```
의존 대상을 설정 코드에서 직접 주입하지 않고 스프링이 자동으로 의존하는 빈 객체를 주입해주는 기능이 있다. 이를 자동 주입이라 한다.
@Autowired 어노테이션을 이용하면 스프링이 알아서 의존 객체를 찾아서 주입한다. 위 코드처럼 의존 객체를 명시하지 않아도 된다. 아래와 같이 코드가 간결해진다.
```java
@Bean
public MemberDao memberDao(){
	return new MemberDao();
}
@Bean
public CHangePasswordService changePwdSvc(){
	ChangePasswordService pwdSvc = new ChangePasswordService();
    return pwdSvc;
}	
```
자동 주입을 사용하기 위해서 의존을 주입할 대상에 @Autowired 어노테이션을 붙이면 된다. 아래 코드와 같이 memberDao 필드에 @Autowired을 붙이면 설정 클래스에서 의존을 주입하지 않아도 된다. 스프링이 해당 타입의 빈 객체를 찾아서 필드에 할당한다. 
```java
public class ChangePasswordServic{
	@Autowired
    private MemberDao memberDao;
    ..
}
```
### 메서드
@Autowired는 메서드에도 붙일 수 있다. 
#### MemberInfoPrinter
```java
public class MemberInfoPrint{
	private MemberDao memDao;
    private MemberPrinter printer;
    ...
    
    @Autowired
    public void setMemberDao(MemberDao memberDao){
    	this.memDao = memberDao;
    }
    @Autowired
    public void setPrinter(MemberPrinter printer){
    	this.printer=printer;
    }
}
```
#### AppCtx
```java
@Configuration
public class AppCtx{
	@Bean
    public MemberDao memberDao(){
    	return new MemberDao();
    }	
    ...
    @Bean
    public MemberPrinter memberPrinter(){
    	return new MemberPrinter();
    }
    @Bean
    public MemberInfoPrinter infoPrinter(){
    	MemberInfoPrinter infoPrinter = new MemberInfoPrinter();
        return infoPrinter;
    }
}
```
아래 그림은 @Autowired를 적용했을 때 주입이 어떻게 연결되는지 보여준다. @Autowired를 필드나 세터 메서드에 붙이면 스프링은 타입이 일치하는 빈 객체를 찾아서 주입한다.
![](https://velog.velcdn.com/images/yh_lee/post/e95a23a0-37aa-4a5f-85fc-ddfd50ab64d3/image.png)

### 일치하는 빈이 없는 경우
❓ @Autowired 어노테이션을 적용한 대상에 일치하는 빈이 없으면 어떻게 될까?
```java
@Configuration
public class AppCtx{
	@Bean
    public MemberDao memberDao(){
    	return new MemberDao();
    }
    @Bean
    public MemberRegisterService memberRegSvc(){
		return new MemberRegisterService();
	}	
}
```

@Autowired를 붙인 MemberRegisterService의 memberDao 필드에 주입할 MemberDao 빈이 존재하지 않아 에러가 발생했다는 사실을 알려준다. 

#### 에러메세지

	1.Error creating bean with name 'memberRegSvc'
	2.Unsatisfied dependency expressed through field 'memberDao'
    3.No qualifying bean of type 'spring.MemberDao' available
    
    1.'memberRegSvc'빈을 생성하는데 에러가 발생했다. 
	2.'memberDao'에 대한 의존을 충족하지 않는다.
	3. 적용할 수 있는 MemberDao 타입의 빈이 없다. 
    
    
❓ 반대로 @Autowired 어노테이션을 붙인 주입 대상에 일치하는 빈이 두 개 이상이면 어떻게 될까?

```java
//@Bean
//public MemberPrinter memberPrinter(){
//	return new MemberPrinter();
//}
@Bean
public MemberPrinter memberPrinter1(){
	return new MemberPrinter();
}
@Bean
public MemberPrinter memberPrinter2(){
	return new MemberPrinter();
}
```

MemberPrinter 타입의 빈을 한정할 수 없는데 해당 타입 빈이 한 개가 아니라 두 개가 발견했다는 사실을 알려준다.
#### 에러메세지

	No qualifying bean of type 'spring.MemberPrinter' available: expected single matching bean but found 2: memberPrinter1, memberPrinter2

	