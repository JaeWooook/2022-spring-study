# Dependency Injection - getBean()과 빈 객체

## getBean() 메서드
getBean은 사용할 빈 객체를 구하는 메서드이다. 첫번째 인자는 빈의 이름, 두번째 인자는 빈의 타입이다. 

```java
VersionPrinter versionPrinter = ctx.getBean("versionPrinter",VersionPrinter.class);
```

아래와 같이 이름이 listPrinter인 빈의 타입이 MemberListPrinter인데, VersionPrinter 클래스를 전달했다고 하자. 빈의 실제 타입과 getBean() 메서드에 지정한 타입이 다르면 **"BeanNotOfRequiredTypeException"** 익셉션이 발생한다.
```java
VersionPrinter versionPrinter = ctx.getBean("listPrinter",VersionPrinter.class);
```

다음과 같이 빈 이름을 지정하지 않고 타입만으로 빈을 구할 수도 있다. 이때 해당 타입의 빈 객체가 **한 개만 존재하면 해당 빈을 구해서 리턴한다.** 해당 타입의 빈 객체가 존재하지 않으면 **"NoSuchBeanDefinitionException"** 익셉션이 발생한다.
```java
VersionPrinter versionPrinter = ctx.getBean(MemberPrinter.class);
```

같은 타입의 빈 객체가 두 개 이상 존재할 수도 있다. 설정 클래스에서 VersionPrinter 타입 빈을 두 개 설정했다고 하자. 이 경우 ctx.getBean(VersionPrinter.class) 메서드를 실행하면 **"NoUniqueBeanDefinitionException"** 익셉션이 발생한다.

```java
@Configuration
public class AppCtx{
	@Bean
    public VersionPrinter versionPrinter(){
    	...
    }
    @Bean
    public VersionPrinter versionPrinter(){
    	...
    }
}
```
## 빈 객체
### ❓ 주입 대상 객체를 모두 빈 객체로 설정해야 할까?
주입할 객체가 꼭 스프링 빈이어야 할 필요는 없다. 예시로 MemberPrinter를 빈으로 등록하지 않고 일반 객체로 생성해서 주입할 수 있다. 
이 설정 코드는 MemberPrinter를 빈으로 등록하지 않았다. pinter를 필드에 할당하고 이 필드를 사용해서 listPrinter와 infoPrinter인 빈을 생성했다. 

```java
@Configuration
public class AppCtxNoMemberPrinterBean{
	private MemberPrinter printer = new MemberPrinter();// 빈이 아님
    ...
    @Bean
    public MemberListPrinter listPrinter(){
    	return new MebmerListpRinter(memberDao(),printer);
    }
	@Bean
    public MemberInfoPrinter infoPrinter(){
    	...
        infoPrinter.setPrinter(printer);
        return infoPrinter;
    }
}
```

💡 객체를 스프링 빈으로 등록할 때와 하지 않을 때의 차이는 **스프링 컨테이너가 객체를 관리하는지 여부이다.** 위 코드처럼 설정하면 MemberPrinter를 빈으로 등록하지 않으므로 스프링 컨테이너에서 사용할 수 없다.
```java
//스프링 컨테이너에 등록되어 있지 않으므로 익셉션 발생
MemberPrinter printer = ctx.getBean(MemberPrinter.class);
```
> 스프링 컨테이너는 자동 주입, 리사이클 관리 등 단순 객체 생성 이외에 객체 관리를 위한 다양한 기능을 제공한다. 이는 빈으로 등록한 객체에만 기능을 적용한다.
따라서 스프링 컨테이너가 제공하는 관리 기능이 필요없고 getBean() 메서드로 구할 필요학 엇ㅂ다면 빈 객체로 꼭 등록해야 하는 것은 아니다.
최근엔 의존 자동 주입 기능을 프로젝트 전반에 걸쳐 사용하는 추세이기 때문에 의존 주입 대상은 스프링 빈으로 등록하는 것이 일반적이다.


