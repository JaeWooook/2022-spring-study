컴포넌트 스캔은 스프링이 직접 클래스를 검색해서 빈으로 등록해주는 기능이다.
- 설정 클래스에 빈으로 등록하지 않아도 원하는 클래스를 빈으로 등록할 수 있으므로 컴포넌트 스캔 기능을 사용하면 설정 코드가 크게 줄어든다.

@Component 애노테이션으로 스캔 대상 지정
애노테이션 값을 주지 않으면 클래스 이름을 첫 근자를 소문자로 바꾼 다음 빈 이름으로 사용한다.
```
MemberInfoPrinter

@Component("infoPrinter")
public class MemberListPrinter {

    private MemberDao memberDao;
    private MemberPrinter printer;


}
```
@Component 애노테이션에 속성 값을 주었다.

@CompononetScan 애노테이션으로 스캔 설정
@Component 애노테이션을 붙인 클래스를 스캔해서 스프링 빈으로 등록하려면 설정 클래스에 @ComponentScan 애노테이션을 적용해야 한다.

AppCtx
```
@Configuration
@ComponentScan(basePackages = {"spring"})
public class AppCtx {

    @Bean
    @Qualifier("printer")
    public MemberPrinter memberPrinter1() {
        return new MemberPrinter();
    }

    @Bean
    @Qualifier("summaryPrinter")
    public MemberSummaryPrinter memberPrinter2() {
        return new MemberSummaryPrinter();
    }

    @Bean
    public VersionPrinter versionPrinter() {
        VersionPrinter versionPrinter = new VersionPrinter();
        versionPrinter.setMajorVersion(5);
        versionPrinter.setMinorVersion(0);
        return versionPrinter;
    }
}
```
스프링 컨테이너가 @Component 애노테이션을 붙인 클래스를 검색해서 빈으로 등록해주기 때문에 설정코드가 줄어든 것을 알 수 있다.

@ComponentScan 애노테이션의 basePackages 속성값은 {"spring"}이다. 이 속성은 스캔 대상 패키지 목록을 지정한다. 여기서는 "spring" 값 하나만 존재하는데 이는 spring 패키지와 그 하위 패키지에 속한 클래스를 스캔 대상으로 설정한다. 스캔 대상에 해당하는 클래스 중에서 @Component 애노태이션이 붙은 클래스의 객체를 생성해서 빈으로 등록한다.

스캔 대상에서 제외하거나 포함하기
정규식
```
@Configuration
@ComponentScan(basePackages = {"spring"},
    excludeFilters = @Filter(type = FilterType.REGEX, pattern = "spring\\..*Dao"))
public class AppCtx {

    ...
}
```
이 코드는 @Filter 애노테이션의 type 속성값으로 FilterType.REGEX를 주었다. 이는 정규표현식을 사용해서 제외 대상을 지정한다는 것을 의미한다. 위 설정에서는 "spring."으로 시작하고 Dao로 끝나느 정규표현식을 지정했으므로 spring.MemberDao 클래스를 컴포넌트 스캔 대상에서 제외한다.

Aspectj
```
@Configuration
@ComponentScan(basePackages = {"spring"},
    excludeFilters = @Filter(type = FilterType.ASPECTJ, pattern = "spring.*Dao"))
public class AppCtx {

    ...
}
```

일단 지금은 "spring.*Dao" AspectJ 패턴은 spring 패키지의 Dao로 끝나는 타입을 지정한다는 정도로만 알고 넘어가자. AspectJ 패턴이 동작하려면 의존대상에 aspectjweaver 모듈을 추가해야 한다.

애노테이션
특정 애노테이션을 붙인 타입을 컴포넌트 대상에서 제외할 수도 있다. 예를 들어 @NoProduct나 @ManualBean 애노테이션을 붙인 클래스는 컴포넌트 스캔 대상에서 제외하고 싶다고 하자.
```
@Configuration
@ComponentScan(basePackages = {"spring"},
    excludeFilters = @Filter(type = FilterType.ANNOTATION, classes = {NoProduct.class, ManualBean.class}))
public class AppCtx {

    ...
}
```
타입
특정 타입이나 그 하위 타입을 컴포넌트 스캔에서 제외하려면 ASSIGNABLE_TYPE을 FilterType으로 사용한다.
```
@Configuration
@ComponentScan(basePackages = {"spring"},
    excludeFilters = @Filter(type = FilterType.ASSIGNABLE_TYPE, classes = MemberDao.class))
public class AppCtx {

    ...
}
```
기본 스캔 대상
@Component 애노테이션을 붙인 클래스만 컴포넌트 스캔 대상에 포함되는 것은 아니다. 다음 애노테이션을 붙인 클래스가 컴포넌트 스캔 대상에 포함된다.

@Component
@Controller
@Service
@Repository
@Aspect
@Configuration
@Aspect 애노테이션을 제외한 나머지 애노테이션은 실제로는 @Component 애노테이션에 대한 특수 애노테이션이다.

컴포넌트 스캔에 따른 충돌 처리
컴포넌트 스캔 기능을 사용해서 자동으로 빈을 등록할 때는 충돌에 주의해야 한다. 크게 빈 이름 충돌과 수동 등록에 따른 충돌이 발생할 수 있다.

빈 이름 충돌
spring 패키지와 spring2 패키지에 MemberRegisterService 클래스가 존재하고 두 클래스 모두 @Component 애노테이션을 붙였다고 하자. 이 상태에서 spring과 spring2 모두 컴포넌트 스캔을 하면 익셉션이 발생하낟.

이렇게 컴포넌트 스캔 과정에서 서로 다른 타입인데 같은 빈 이름을 사용하는 경우가 있다면 둘 중 하나에 명시적으로 빈 이름을 지정해서 이름 충돌을 피해야 한다.

수동 등록한 빈과 충돌
스캔할 때 사용하는 빈 이름과 수동 등록한 빈 이름이 같은 경우 수동 등록한 빈이 우선한다.

