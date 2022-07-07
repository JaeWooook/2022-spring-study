# @Component 스캔 대상 지정
컴포넌트 스캔은 **스프링이 직접 클래스를 검색해서 빈으로 등록해주는 기능**이다. 설정 클래스에서 빈으로 등록하지 않아도 원하는 클래스를 빈으로 등록할 수 있으므로 컴포넌트 스캔 기능을 사용하면 설정 코드 양이 줄어든다.

```java
@Component
public class MemberDao{
	...
}
```
@Component("infoPrinter")로 속성값을 지정하면 빈으로 등록될 때 이름을 지정할 수 있다. 
```java
@Component("infoPrinter")
public class MemberInfoPrinter{
	...
}
```

## @ComponentScan으로 스캔 설정
@Component 어노테이션을 붙인 클래스를 스캔해서 스프링 빈으로 등록하려면 설정 클래스에서 @ComponentScan 어노테이션을 적용해야 한다. 
basePackages 속성은 스캔을 시작할 대상 패키지 목록을 지정한다. "spring"으로 지정하면 spring 패키지와 그 하위 패미지에 속한 클래스를 스캔 대상으로 설정한다. 스캔 대상에 해당하는 클래스 중에서 @Component 어노테이션이 붙은 클래스의 객체를 생성해서 빈으로 등록한다.

```java
@Component
@ComponentScan(basePackages={"spring"})
public class AppCtx{
	...
}
```

## 스캔 대상에서 제외하거나 포함하기
excludeFilters 속성을 사용하면 스캔할 때 특정 대상을 자동 등록 대상에서 제외할 수 있다. 

REGEX 정규표현식을 사용해서 제외 대상을 지정하도록 했다. spring으로 시작하고 Dao로 끝나는 정규표현식을 지정했으므로 spring.MemberDao 클래스를 컴포넌트 스캔 대상에서 제외한다. 
```java
@Configuration
@ComponentScan(excludeFilters=@Filter(type=FilterType.REGEX,pattern="spring\\..*Dao"))
public class AppCtxWithExclude{...}
```
ASPECTJ 필터타입으로 설정할 수도 있다. 
```java
@Configuration
@ComponentScan(excludeFilters=@Filter(type=FilterType.ASPECTJ,pattern="spring.*Dao"))
public class AppCtxWithExclude{...}
```

특정 어노테이션을 붙인 타입을 컴포넌트 대상에서 제외할 수도 있다. 예를 들어 @NoProduct나 @ManualBean 어노테이션을 붙인 클래스는 컴포넌트 스캔 대상에서 제외하고 싶다고 하자.
```java
@Retention(RUNTIME)
@Target(TYPE)
public @interface NoProduct{}

@Retention(RUNTIME)
@Target(TYPE)
public @interface ManualBean{}
```
두 어노테이션을 컴포넌트 스캔 대상에서 제외하려면 다음과 같이 excludeFilter 속성을 설정한다. type 속성 값으로 FilterTYPE.ANNOTATION을 사용하면 classes 속성에 필터로 사용할 어노테이션 타입을 값으로 준다. 
```java
@Configuration
@ComponentScan(basePackages={"spring","spring2"},
	excludeFilters=@Filter(type=FilterTYPE.ANNOTATION, classes={NoProduct.class,ManualBean.class}))
public class AppCtxWithExclude{...}
```
아래 코드는 @ManualBean 어노테이션을 붙였기 때문에 컴포넌트 스캔 대상에서 제외된다. 
```java
@ManualBean
@Component
public class MemberDao{...}
```

특정 타입이나 그 하위 타입을 컴포넌트 스캔 대상에서 제외하려면 ASSINABLE_TYPE을 필터 타입으로 사용한다. classes 속성에는 제외할 타입 목록을 지정한다. 
```java
@Configuration
@ComponentScan(basePackages={"spring"},
	excludeFilters=@Filter(type=FilterTYPE.ASSIGNABLE_TYPE, classes=MemberDao.class))
public class AppCtxWithExclude{...}
```

설정할 필더가 두 개 이상이면 @ComponentScan의 excludeFilters 속성에 배열을 사용해서 @Filter 목록을 전달하면 된다.
```java
@Configuration
@ComponentScan(basePackages={"spring"},
	excludeFilters={
    	@Filter(type=FilterTYPE.REGEX, classes=MemberDao.class, pattern="spring2\\..*"),
    	@Filter(type=FilterTYPE.REGEX, classes=MemberDao.class, classees=ManualBean.class)
    })
public class AppCtxWithExclude{...}
```

### 기본 스캔 대상
@Component를 붙인 클래스만 컴포넌트 스캔 대상에 포함되는 것은 아니다. 다음 어노테이션을 붙인 클래스도 컴포넌트 스캔 대상에 포함된다.
- @Component
- @Controller
- @Service
- @Repository
- @Aspect
- @Configuration

## 컴포넌트 스캔에 따른 충돌 처리
### 빈 이름 충돌
패키지 spring, spring2 모두에 MemberRegisterService 클래스가 존재하고 두 클래스 모두 @Component 어노테이션을 붙였을 때 @ComponentScan을 사용하면 어떻게 될까? 
아래와 같은 익셉션이 발생한다. 이런 문제는 컴포넌트 스캔 과정에서 쉽게 발생할 수 있다. 둘 중 하나에 명시적으로 빈 이름을 지정해서 이름 충돌을 피해야 한다.
```java
@Configuration
@ComponentScan(basePackages={"spring","spring2"}
public class AppCtx{...}
```
	...
    Annotation-specified bean name 'memberRegisterService' 
    for bean class [spring2.MemberRegisterService] 
    confilicts with existing, 
    non-compatible bean definition of same name and 
    class [spring.MemberRegisterService]
    
### 수동 등록한 빈과 충돌
MemberDao 클래스에 @Component 어노테이션을 붙였다. MemberDao는 컴포넌트 스캔 대상이다. 자동 등록된 빈의 이름은 "memberDao"이다. 그런데 설정 클래스에서 직접 MemberDao 클래스를 "memberDao"라는 이름의 빈으로 등록하면 어떻게 될까?
```java
@Component
public class MemberDao{...}
```
```java
@Configuration
@ComponentScan
public class AppCtx{
	@Bean
    public MemberDao memberDao(){
    	MemberDao memberDao = new MemberDao();
        return memberDao;
    }	
}
```
스캔할 때 사용하는 빈 이름과 수동 등록한 빈 이름이 같은 경우 **수동 등록한 빈이 우선이다.** 즉 MemberDao 타입 빈은 AppCtx에서 정의한 한 개만 존재한다.

다음과 같이 다른 이름을 사용하면 어떻게 될까?
이 경우 스캔을 통해 등록한 "memberDao" 빈과 수동 등록한 "memberDao2" 빈이 모두 존재한다. MemberDao 타입의 빈이 두 개가 생성되므로 자동주입하는 코드는 @Qualifier, @Primary, @Autowired 필드 변경 등을 사용해서 알맞은 빈을 선택해야 한다.
```java
@Configuration
@ComponentScan
public class AppCtx{
	@Bean
    public MemberDao memberDao2(){
    	MemberDao memberDao = new MemberDao();
        return memberDao;
    }	
}
```