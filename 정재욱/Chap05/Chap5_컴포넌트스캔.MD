## Chapter5 컴포넌트 스캔
> 컴포넌트 스캔이란?

> 스프링이 직접 클래스를 검색해서 빈으로 등록해주는 기능이다.
> 설정 클래스(AppCtx클래스)에 빈으로 등록하지 않아도 원하는 클래스를 빈으로 등록할 수 있으므로 컴포넌트 스캔 기능을 사용하면 설정 코드가 크게 줄어든다.

### @Component 애노테이션으로 스캔 대상 지정
- 스프링이 검색해서 빈으로 등록할 수 있으려면 클래스에 @Component 애노테이션을 붙여야한다.
- @Component 애노테이션은 해당 클래스를 스캔 대상으로 표시한다.

#### Componet 애노테이션을 생성할 빈 클래스에 붙인 예시
```java
package spring;

import java.util.Collection;
import java.util.HashMap;
import java.util.Map;

import org.springframework.stereotype.Component;

@Component
public class MemberDao {

	private static long nextId = 0;

	private Map<String, Member> map = new HashMap<>();

	public Member selectByEmail(String email) {
		return map.get(email);
	}

	public void insert(Member member) {
		member.setId(++nextId);
		map.put(member.getEmail(), member);
	}

	public void update(Member member) {
		map.put(member.getEmail(), member);
	}

	public Collection<Member> selectAll() {
		return map.values();
	}
}
```
- ChangePasswordService 클래스와 MemberRegisterService 클래스에도 동일하게 @Component 애노테이션을 붙인다.

```java
package spring;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Component;

@Component("infoPrinter")
public class MemberInfoPrinter {

	private MemberDao memDao;
	private MemberPrinter printer;

    ...

}
```
- @Component 애노테이션 값을 주지않으면 클래스 이름이 MemberDao 이면 -> __빈(빈 객체)__ 이름은 memberDao가 된다.
- 위의 코드처럼 @Component 애노테이션에 값을 주면 그 값을 __빈(빈 객체)__ 이름으로 사용한다. MemberInfoPrinter -> infoPrinter가 __빈(빈 객체)__ 이름이 된다.
- 나머지 빈을 생성할 클래스도 동일하게 설정할 수 있다. ex) MemberListPrinter, ... 등

### @ComponentScan 애노테이션으로 스캔 설정
 - @Component 애노테이션을 붙인 클래스를 스캔해서 스프링 빈으로 등록하려면 설정 클래스에 @ComponentScan 애노테이션을 적용해야 한다.

#### ✅ 컴포넌트 스캔 사용 예시 설정코드
```java
package config;

import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

import spring.MemberPrinter;
import spring.MemberSummaryPrinter;
import spring.VersionPrinter;

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
 - @ComponentScan 애노테이션의 basePackages 속성값은 {"spring"}이다.
 - 이 속성은 스캔 대상 패키지 목록을 지정한다.
 - spring값 한개만 존재하는 이유는 spring 패키지(ex)trimory에서는 com또는 com.backend로 하면 그 하위 패키지들을 검색한다.)와 그 하위 패키지에 속한 클래스를 스캔 대상으로 설정하기 때문이다.
 - 스캔 대상에 해당하는 클래스 중에서 @Component 애노테이션이 붙은 클래스의 객체를 생성해서 빈으로 등록한다.

#### ❌ 컴포넌트 스캔을 사용하지 않은 예시 설정코드
```java
package config;

import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import spring.ChangePasswordService;
import spring.MemberDao;
import spring.MemberInfoPrinter;
import spring.MemberListPrinter;
import spring.MemberPrinter;
import spring.MemberRegisterService;
import spring.MemberSummaryPrinter;
import spring.VersionPrinter;

@Configuration
public class AppCtx {

	@Bean
	public MemberDao memberDao() {
		return new MemberDao();
	}
	
	@Bean
	public MemberRegisterService memberRegSvc() {
		return new MemberRegisterService();
	}
	
	@Bean
	public ChangePasswordService changePwdSvc() {
		return new ChangePasswordService();
	}
	
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
	public MemberListPrinter listPrinter() {
		return new MemberListPrinter();
	}
	
	@Bean
	public MemberInfoPrinter infoPrinter() {
		MemberInfoPrinter infoPrinter = new MemberInfoPrinter();
		return infoPrinter;
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
 - Chapter4의 비해서 @Component 애노테이션을 붙인 클래스를 검색해서 빈으로 등록해주기 때문에 설정 코드가 줄었다.

### 스캔 대상에서 제외하거나 포함하기
> __excludeFilters 속성을 사용하면__ 스캔할 때 특정 대상을 자동 등록 대상에서 제외할 수 있다.

#### 🔽 FilterType.REGEX : 정규표현식으로 제외 대상 지정하기
```java
package config;

import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;
import org.springframework.context.annotation.ComponentScan.Filter;

import spring.MemberDao;
import spring.MemberPrinter;
import spring.MemberSummaryPrinter;
import spring.VersionPrinter;

@Configuration
@ComponentScan(basePackages = {"spring"}, 
	excludeFilters = @Filter(type = FilterType.REGEX, patter = "spring\\..*Dao" ))
public class AppCtxWithExclude {
	@Bean
	public MemberDao memberDao() {
		return new MemberDao();
	}
	
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
 - 위 코드느 @Filter 애노테이션의 type 속성값으로 FilterType.REGEX를 주었다.
 - 이는 정규표현식을 사용해서 제외 대상을 지정한다는 것을 의미한다.
 - "spring."으로 시작하고 Dao로 끝나는 정규표현식을 지정했으므로 spring.MemberDao 클래스는 컴포넌트 스캔 대상에서 제외한다.

#### 🔽 FilterType.ASPECTJ : AspectJ패턴을 사용해서 대상 제외하기
````java
@Configuration
@ComponentScan(basePackage = {"spring"},
 excludeFilters = @Filter(type = FilterType.ASPECTJ, pattern = "spring.*Dao"))
public class AppCtxWithExclude {
    @Bean
    public MemberDao memberDao() {
        return new MemberDao();
    }
}
````
 - AspectJ패턴은 spring 패키지의 Dao로 끝나는 타입을 지정한다.
 - spring 패키지에서 이름이 Dao로 끝나는 타입을 컴포넌트 스캔 대상에서 제외한다.
 - AspectJ패턴 사용하려면 pom.xml 또는 build.gradle에 aspectjweaver 모듈을 추가해야한다.

#### 🔽 FilterType.ANNOTATION : 애노테이션을 붙여서 대상 제외하기
```java
@Configuration
@ComponentScan(basePackage = {"spring", "spring2"},
 excludeFilters = @Filter(type = FilterType.ANNOTATION, classes = {NoProduct.class, ManualBean.class}))
public class AppCtxWithExclude {
    @Bean
    public MemberDao memberDao() {
        return new MemberDao();
    }
}
```
 - @NoProduct나 @ManualBean애노테이션을 붙인 클래스는 컴포넌트 스캔 대상에서 제외된다.
 - classes속성에 필터로 사용할 애노테이션의 타입값을 준다.

#### 🔽 FilterType.ASSIGNABLE_TYPE : 특정 타입이나 그 하위 타입을 대상 제외하기
```java
@Configuration
@ComponentScan(basePackage = {"spring"},
 excludeFilters = @Filter(type = FilterType.ASSIGNABLE_TYPE, classes = MemberDao.class))
public class AppCtxWithExclude {
    @Bean
    public MemberDao memberDao() {
        return new MemberDao();
    }
}
```
 - classes 속성에 제외할 타입 목록을 지정한다.
 - 제외할 타입이 1개이므로 배열에 표기하지 않았다. ex) 배열표기 : {arr, arr2} 이런식이다.

#### 두개 이상의 설정 필터 인경우 배열형태로 넣어준다.
```java
@Configuration
@ComponentScan(basePackages = {"spring"}, 
	excludeFilters = { 
			@Filter(type = FilterType.ANNOTATION, classes = ManualBean.class ),
            @Filter(type = FilterType.REGEX, pattern = "spring2.\\..*")         
})
public class AppCtxWithExclude {
```

#### 기본 스캔 대상
 > @Component 애노테이션을 붙인 클래스만 컴포넌트 스캔 대상에 포함되는 것은 아니다.

 > 기본이 되는 대상
 - @Component
 - @Controller
 - @Service
 - @Repository
 - @Aspect
 - @Configuration

### 컴포넌트 스캔에 따른 충돌 처리
 - 컴포넌트 스캔 기능을 사용해서 자동으로 빈을 등록할 때에는 충돌에 주의해야 한다.
 - 빈 이름 충돌과 수동 등록에 따른 충돌이 발생할 수 있다.

#### 빈 이름 충돌
 > spring패키지와 spring2패키지에 MemberRegisterService 클래스가 모두 존재하고 @Component 애노테이션을 모두 붙였다면?
```java
@Configuration
@ComponentScan(basePackages = {"spring", "spring2"})
public class AppCtx {
    ...
}
```
 - spring2 패키지의 MemberRegisterService클래스를 빈으로 등록할 때 기존의 spring패키지의 동일한 클래스 빈이름 과 충돌이난다.
 - 이런 경우 둘중 하나를 수동으로 @ComponentScan("registerService") 이런식으로 변경해주어서 빈(빈객체)이름 충돌을 피해준다.
 
#### 수동 등록한 빈과 충돌

#### 스캔할  때 사용하는 빈 이름 과 수동 등록한 빈 이름이 __같은 경우!__

> 자동으로 빈 등록 하는 방법
```java
@Component
public class MemberDao {//명시적으로 변경하지 않았기 때문에 빈(빈객체) 이름은 memberDao로 생성된다.
        
}
```
 > 수동으로 빈을 등록하는 방법
```java

@Configuration
@ComponentScan(basePackages = {"spring", "spring2" })
public class AppCtxWithExclude {
    @Bean
    public MemberDao memberDao() {
        return new MemberDao();
    }
}
```
 - 스캔할 때 사용하는 빈 이름과 수동 등록한 빈 이름이 같은 경우
 - 수동 등록한 빈을 우선으로 한다.
 - __즉__ MemberDao 타입 빈은 AppCtx에서 정의한 1개만 존재한다.
 
#### 스캔할  때 사용하는 빈 이름 과 수동 등록한 빈 이름이 __다른 경우!__

> 자동으로 빈 등록 하는 방법
```java
@Component
public class MemberDao {//명시적으로 변경하지 않았기 때문에 빈(빈객체) 이름은 memberDao로 생성된다.
        
}
```
> 수동으로 빈 등록 하는 방법

 ```java

@Configuration
@ComponentScan(basePackages = {"spring"})
public class AppCtxWithExclude {
    @Bean
    public MemberDao memberDao2() {
        MemberDao memberDao = new MemberDao();
        return memberDao;
    }
}
```
 - 2개다 수동으로 등록한 빈이지만 같은 타입의 다른 이름의 빈인 경우
 - 스캔을 통해 등록한 memberDao와 수동으로 등록한 memberDao2 빈이 모두 존재한다.
 - MemberDao 타입의 빈이 2개가 생성되므로 자동 주입하는 코드는 @Qualifier(한정자)를 이용해서 알맞은 빈을 선택 해야한다.
