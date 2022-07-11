# 컴포넌트 스캔

<br>

## @Component 애노테이션으로 스캔 대상 지정

- 컴포넌트 스캔 : 스프링이 직접 클래스를 검색해서 빈으로 등록해주는 기능,
     설정 클래스에서 빈으로 등록하지 않아도 원하는 클래스를 빈으로 등록할 수 있음

- @Component : 빈으로 등록하길 원하는 클래스에 @Component 애노테이션을 붙이면 해당 클래스를 스캔 대상으로 표시함
    
    - 애노테이션에 값을 주지 않은 경우 : 빈 이름 = 클래스 이름의 첫 글자를 소문자로 바꾼 이름
    - 값을 준 경우 : 빈 이름 = 해당 값

```
@Component("infoPrinter") // 빈 이름 = infoPrinter
public class MemberInfoPrinter {

    private MemberDao memberDao;
    private MemberPrinter printer;

    ...
}
```

<br>


## @ComponentScan 애노테이션으로 스캔 설정

<br>

📌 @Component 애노테이션을 붙인 클래스를 스캔해서 스프링 빈으로 등록하려면 설정클래스에 @ComponentScan 애노테이션 적용!

```

@Configuration
@ComponentScan(basePackages={"spring"}) //@Component가 붙은 클래스를 빈으로 스캔함
public class AppCtx {

    @Bean
    @Qualifier("printer")
    public MemberPrinter memberPrinter1(){
        return new MemberPrinter();
    }

    @Bean
    @Qualifier("summaryPrinter")
    public MemberSummaryPrinter memberPrinter2(){
        return new MemberSummaryPrinter();
    }

    @Bean
    public VersionPrinter versionPrinter(){
        VersionPrinter versionPrinter = new VersionPrinter();
        versionPrinter.setMajorVersion(5);
        versionPrinter.setMinorVersion(0);
        return versionPrinter;
    }
}


```

- basePackages : 스캔 대상 패키지 목록 지정, 속성값에 적용된 패키지와 그 하위 패키지에 속한 클래스를 대상으로 설정
    
    

### 🔎 스캔 대상에서 제외하거나 포함시키기

- excludeFilter : 스캔할 때 특정 대상을 자동 등록 대상에서 제외시킴

```
@Configuration
@ComponentScan(basePackages = {"spring"},
       excludeFilters =@ComponentScan.Filter(type = FilterType.REGEX,pattern = "spring\\.*Dao"))
public class AppCtx {

    ..

}
```
: 정규표현식(FilterTyep.REGEX)을 사용해서 제외 대상 지정
    <BR> → "spring."으로 시작하고 Dao로 끝나는 클래스를 컴포넌트 스캔 대상에서 제외 (spring.MemberDao) <Br><Br>

```
@Configuration
@ComponentScan(basePackages = {"spring","spring2"},
       excludeFilters =@ComponentScan.Filter(type = FilterType.ANNOTATION,classes = {NoProduct.class, ManualBean.class}))
public class AppCtx {

    ..

}
```
: 특정 애노테이션을 붙인 타입을 컴포넌트 대상에서 제외
 <BR> → @NoProduct 혹은 @ManualBean 애노테이션을 붙인 클래스를 컴포넌트 스캔 대상에서 제외 <Br><Br>

 ➕ 설정할 필터가 두개 이상이면 excludeFilter 속성에 배열 사용

 ➕ classes 속성에 제외할 타입이 두개 이상이면 배열 사용 



 <br>

 ### 🔎 기본 스캔 대상
 
 - @Component
 - @Controller
 - @Service
 - @Repository
 - @Aspect
 - @Configuration

 : @Component 애노테이션을 붙이 클래스 뿐만이 아닌 위  애노테이션을 붙인 클래스도 컴포넌트 스캔 대상에 포함!



 <br>

 ## 컴포넌트 스캔에 따른 충돌 처리

### 1. 빈 이름 충돌

: 서로 다른 타입인데 같은 빈 이름을 사용한다면 둘 중 하나에 명시적으로 빈 이름 지정해서 이름 충돌 피하기!

➕ 패키지가 다르다면 타입도 다른 것으로 생각
<br>spring.MemberRegisterService와 spring2.MemberRegisterService는 다른 것!


<Br>

### 2. 수동 등록에 따른 충돌

: 스캔할 때 사용하는 빈 이름과 수동 등록한 빈 이름이 같은 경우, **수동 등록**한 빈이 우선!

∴ 해당 타입 빈은 설정 클래스에 수동 등록한 빈 한 개만 존재하는 것 

if 같은 타입을 다른 이름을 쓴다면?
    
    스캔 통해 등록한 빈과 수동 등록한 빈 모두 생성
    ➡️ @Qualifier 사용해 알맞은 빈 선택! 
