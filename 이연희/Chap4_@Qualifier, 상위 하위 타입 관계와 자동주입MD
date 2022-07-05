# @Qualifier를 이용한 의존 객체 선택
자동 주입 가능한 빈이 두 개 이상이면 이를 지정할 수 있는 방법이 필요하다. @Qualifier를 이용하면 자동 주입 대상 빈을 한정할 수 있다. @Qualifier는 두 위치에서 사용 가능하다.
## @Bean을 붙인 설정 메서드
아래 코드에서 @Qualifier는 빈의 한정 값으로 "printer"를 지정한다.
```java
@Configuration
public class AppCtx{
	@Bean
    @Qualifier("printer")
    public MemberPrinter memberPrinter1(){
    	return new MemberPrinter();
    }
}
```

```java
public class MemberListPrinter{
	private MemberDao memberDao;
    private MemberPrinter printer;
    @Autowired
    @Qualifier("printer")
    public void setMemberPrinter(MemberPrinter printer){
    	this.printer=printer;
    }	
}
```
setMemberPrinter() 메서드에 @Autowired 어노테이션을 붙였기 때문에 MemberPrinter 타입의 빈을 자동 주입한다. 이때 @Qualifier 어노테이션 값이 "printer"이므로 한정 값이 "printer"인 빈을 의존 주입 후보로 사용한다. **위 코드 설정 클래스(AppCtx)에서 @Qualifier 값으로 "printer"를 준 MemberPrinter 타입의 빈(memberPrinter1)을 자동 주입 대상으로 사용한다. **

## 빈 이름과 기본 한정자
빈 설정에 **@Qualifier가 없으면 빈의 이름을 한정자로 설정**한다. 아래 코드에선 printer1() 메서드로 정의한 빈의 한정자는 빈 이름인 "printer1"이 된다. printer2의 빈의 한정자 이름은 "mprinter"가 된다.
```java
@Configuration
public class AppCtx2{
	@Bean
    public MemberPrinter printer1{
    	return new MemberPrinter();
    }
    
	@Bean
    @Qualifier("mprinter")
    public MemberPrinter printer2{
    	return new MemberPrinter();
    }
}
```

@Autowired 어노테이션도 @Qualifier가 없으면 필드나 파라미터 이름을 한정자로 사용한다. printer 필드에 일치하는 빈이 두 개 이상 존재하면 한정자로 필드 이름인 "printer"를 사용한다. 
```java
public class MemberInfoPrinter2{
	@Autowired
    private MemberPrinter printer;
	...
}
```

# 상위/하위 타입 관계와 자동 주입
아래 클래스는 MemberPrinter 클래스를 상속한 MemberSummaryPrinter 클래스이다.
```java
public class MemberSummaryPrinter extends MemberPrinter{
	@Override
    public void print(Member member){
    	...
    }
}
```
AppCtx 클래스 설정에서 memberPrinter2() 메서드가 MemberSummaryPrinter 타입의 빈 객체를 설정하도록 변경하자. 그리고 @Qualifier도 삭제한다. 
```java
@Configuration
public class AppCtx{
	...
    @Bean
    public MemberPrinter memberPrinter1(){
    	return new MemberPrinter();
    }
    @Bean
    public MemberSummaryPrinter memberPrinter2(){
    	return new MemberSummartPrinter();
    }
}
```
memberPrinter2 빈을 MemberSummaryPrinter로 변경하면 에러가 발생한다. 이유는 MemberSummaryPrinter 클래스가 MemberPrinter 클래스를 상속했기 때문이다. MemberSummaryPrinter 클래스는 MemberPrinter 클래스 타입에도 할당할 수 있어서, 스프링 컨테이너에는 MemberPrinter 타입 빈을 자동 주입해야 하는 @Autowired를 만나면 memberPrinter1과 memberPrinter2 중 어떤 빈을 주입해야 하는지 알 수 없어서 익셉션이 발생한다.

MemberListPrinter 클래스와 MembeInfoPrinter 클래스가 MemberPrinter 타입의 빈을 자동 주입하므로 어떤 빈을 주입할지 결정해야 한다. 먼저 MemberInfoPrinter 클래스에 @Qualifier를 사용해서 빈을 한정하자. 
```java
@Configuration
public class AppCtx{
	...
    @Bean
    @Qualifier("printer")
    public MemberPrinter memberPrinter1(){
    	...
    }
}

public class MemberInfoPrinter{
	...
    @Autowired
    @Qualifier("printer")
    public void setPrinter(MemberPrinter printer){
    	this.printer=printer;
    }
}
```
MemberListPrinter 클래스에 자동 주입할 MemberPrinter 타입 빈은 두가지 방법으로 설정할 수 있다. 
#### 1) @Qualifier 사용
```java
@Configuration
public class AppCtx{
	...
    @Bean
    @Qualifier("printer")
    public MemberPrinter memberPrinter1(){
    	...
    }
    
    @Bean
    @Qualifier("summaryPrinter")
    public MemberPrinter memberPrinter1(){
    	...
    }
}

public class MemberListPrinter{
	...
    @Autowired
    @Qualifier("summaryPrinter")
    public void setMemberPrinter(MemberPrinter printer){
    	this.printer=printer;
    }
}
```
#### 2) MemberListPrinter가 MemberSummaryPrinter를 사용하도록 수정
MemberSummaryPrinter 타입 빈은 한 개만 존재하므로 MemberSummaryPrinter 빈을 자동 주입 받도록 수정하면 가능하다.
```java
public class MemberListPrinter{
	...
    @Autowired
    public void setMemberPrinter(MemberPrinter printer){
    	this.printer=printer;
    }
}
```