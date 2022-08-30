## 프로필과 프로퍼티 파일

### 프로필
>  개발을 진행하는 동안에는 실제 서비스 목적으로 운영중인 DB를 사용할 수는 없다.
> 
> 개발하는 동안에는 개발용 DB를 따로 사용하거나 개발 PC에 직접 DB를 설치해서 사용한다.

즉 개발을 완료한 어플레키이션을 실제 서버에 배포하려면 실 서비스 환경에 맞는 JDBC 연결 정보를 사용해야 한다.

__개발 목적 설정과 실 서비스 목적의 설정을 구분해서 작성하는 것을 위한 스프링 기능이 프로필(profile)이다.__

#### @Configuration 설정에서 프로필 사용하기
```java
package config;

import org.apache.tomcat.jdbc.pool.DataSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;

@Configuration
@Profile("dev")
public class DsDevConfig {

	@Bean(destroyMethod = "close")
	public DataSource dataSource() {
		DataSource ds = new DataSource();
		ds.setDriverClassName("com.mysql.jdbc.Driver");
		ds.setUrl("jdbc:mysql://localhost/spring5fs?characterEncoding=utf8");
		ds.setUsername("spring5");
		ds.setPassword("spring5");
		ds.setInitialSize(2);
		ds.setMaxActive(10);
		ds.setTestWhileIdle(true);
		ds.setMinEvictableIdleTimeMillis(60000 * 3);
		ds.setTimeBetweenEvictionRunsMillis(10 * 1000);
		return ds;
	}
}
```
 - @Profile은 "dev"를 값으로 갖는다. 스프링 컨테이너를 초기화할 때 "dev"프로필을 활성화하면 DsDevConfig 클래스를 설정으로 사용한다.

```java
package config;

import org.apache.tomcat.jdbc.pool.DataSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;

@Configuration
@Profile("real")
public class DsRealConfig {

	@Bean(destroyMethod = "close")
	public DataSource dataSource() {
		DataSource ds = new DataSource();
		ds.setDriverClassName("com.mysql.jdbc.Driver");
		ds.setUrl("jdbc:mysql://realdb/spring5fs?characterEncoding=utf8");
		ds.setUsername("spring5");
		ds.setPassword("spring5");
		ds.setInitialSize(2);
		ds.setMaxActive(10);
		ds.setTestWhileIdle(true);
		ds.setMinEvictableIdleTimeMillis(60000 * 3);
		ds.setTimeBetweenEvictionRunsMillis(10 * 1000);
		return ds;
	}
}
```
 - @Profile 어노테이션 값을 real로 설정해서 dev와 real의 구분을 나눈다.

> 두 Config파일 모두 dataSource 빈 중에서 어떤 빈을 사용할지 활성화한 프로필에 따라 달라진다.
> "dev"프로필을 활성화하면 @Profile("dev") 어노테이션을 붙인 설정 클래스의 dataSource 빈을 사용하고,
> 
> "real" 프로필을 활성화하면 @Profile("real") 어노테이션을 붙인 설정 클래스의 dataSource 빈을 사용한다.

__특정 프로필을 선택하려면 컨테이너를 초기화하기 전에 setActiveProfiles() 메서드를 사용해서 프로필을 선택해야 한다.__

```java
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
context.getEnvironment().setActiveProfiles("dev");
context.register(MemberConfig.class, DsDevConfig.class, DsRealConfig.class);
//context.register(MemberConfigWithProfile.class);
context.refresh();
```
 - getEnvironment() 메서드는 스프링 실행 환경을 설정하는데 사용되는 Environment를 리턴한다.
 - 이 Environment의 setActiveProfiles() 메서드를 사용해서 사용할 프로필을 선택할 수 있다.
 - 따라서 DsDevConfig 클래스와 DsRealConfig 클래스에 정의되어 있는 "dataSource"중에서 "dev" 프로필에 속하는 DsDevConfig에 정의된 "dataSource"빈을 사용한다.

> 프로필을 사용할 때 주의할 점은 설정 정보를 전달하기 전에 어떤 프로필을 사용할지 지정해야 한다.

 - setActiveProfiles() 메서드로 "dev" 프로필을 사용한다고 설정한 뒤에 register() 메서드로 설정 파일 목록을 지정했다.
 - 그런 뒤 refresh() 메서드를 실행해서 컨테이너를 초기화했다.
 - 이 순서를 지키지 않고 프로필을 선택하기 전에 설정 정보를 먼저 전달하면 프로필을 지정한 설정이 사용되지 않기 때문에 __설정을 읽어오는 과정에서 빈을 찾지 못해 익셉션이 발생한다.__

> 두 개 이상의 프로필을 활성화하고 싶다면 다음과 같이 각 프로필 이름을 메서드에 파라미터로 전달한다.

```java
context.getEnvironment().setActiveProfiles("dev", "real");
```

#### @Configuration을 이용한 프로필 설정
> 중첩 클래스를 이용해서 프로필 설정을 한 곳으로 모을 수 있다.

```java
package config;

import org.apache.tomcat.jdbc.pool.DataSource;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import spring.AuthService;
import spring.ChangePasswordService;
import spring.MemberDao;
import spring.MemberRegisterService;

@Configuration
@EnableTransactionManagement
public class MemberConfigWithProfile {
	@Autowired
	private DataSource dataSource;
	
	@Bean
	public PlatformTransactionManager transactionManager() {
		DataSourceTransactionManager tm = new DataSourceTransactionManager();
		tm.setDataSource(dataSource);
		return tm;
	}

	@Bean
	public MemberDao memberDao() {
		return new MemberDao(dataSource);
	}

    ...
    
	@Configuration
	@Profile("dev")
	public static class DsDevConfig {

		@Bean(destroyMethod = "close")
		public DataSource dataSource() {
			DataSource ds = new DataSource();
			ds.setDriverClassName("com.mysql.jdbc.Driver");
			ds.setUrl("jdbc:mysql://localhost/spring5fs?characterEncoding=utf8");
			ds.setUsername("spring5");
			ds.setPassword("spring5");
			ds.setInitialSize(2);
			ds.setMaxActive(10);
			ds.setTestWhileIdle(true);
			ds.setMinEvictableIdleTimeMillis(60000 * 3);
			ds.setTimeBetweenEvictionRunsMillis(10 * 1000);
			return ds;
		}
	}

	@Configuration
	@Profile("real")
	public static class DsRealConfig {

		@Bean(destroyMethod = "close")
		public DataSource dataSource() {
			DataSource ds = new DataSource();
			ds.setDriverClassName("com.mysql.jdbc.Driver");
			ds.setUrl("jdbc:mysql://realdb/spring5fs?characterEncoding=utf8");
			ds.setUsername("spring5");
			ds.setPassword("spring5");
			ds.setInitialSize(2);
			ds.setMaxActive(10);
			ds.setTestWhileIdle(true);
			ds.setMinEvictableIdleTimeMillis(60000 * 3);
			ds.setTimeBetweenEvictionRunsMillis(10 * 1000);
			return ds;
		}
	}

}
```
 - 중첩된 @Configuration 설정을 사용할 때 주의할 점은 __중첩 클래스는 static이어야 한다.__

#### 다수 프로필 설정
> 스프링 설정은 두 개 이상의 프로필 이름을 가질 수 있다.

```java
@Configuration
@Profile("real,test")
public class DataSourceJndiConfig {
    ...
}
```
 - real프로필을 사용할 때와 test 프로필을 사용할 때 모두 해당 설정을 사용한다.
 - 하지만 이렇게 되어 같은 설정을 사용한다면 굳이 2개를 할필요가 있는지 의문이든다.

```java
@Configuration
@Profile("!real")
public class DsDevConfig {
    @Bean(destroyMethod = "close")
    public DataSource dataSource() {
        DataSource ds =  new DataSource();
        ds.setDriverClassName("com.mysql.jdbc.Driver");
        ...
    }
}
```
 - "!real" 값은 "real"프로필이 활성화 되지 않을 때 사용한다는 것을 의미한다.
 - 보통 "!프로필" 형식은 특정 프로필이 사용되지 않을 때 기본으로 사용할 설정을 지정하는 용도이다.

#### 어플리케이션에서 프로필 설정하기

> 웹 어플리케이션의 경우에도 spring.profiles.active 시스템 프로퍼티나 환경 변수를 사용해서 사용할 프로필을 선택할 수 있다.

```xml
<init-param>
    <param-name>spring.profiles.active</param-name>
    <param-value>dev</param-value>
</init-param>
```
 - profile.active 초기화 파라미터를 이용해서 프로필을 선택할 수 있다.
 - 다소 불편한 방법처럼 보인다.

### 프로퍼티 파일을 이용한 프로퍼티 설정
> 스프링은 외부의 프로퍼티 파일을 이용해서 스프링 빈을 설정하는 방법을 제공하고 있다.

#### @Configuration 어노테이션 이용 자바 설정에서 프로퍼티 사용
자바 설정에서 프로퍼티 파일을 사용하려면 두 가지를 설정이 필요하다.
 - PropertySourcesPlaceholderConfigurer 빈 설정
 - @Value 어노테이션으로 프로퍼티 값 사용

먼저 PropertySourcesPlaceholderConfigurer 클래스를 빈으로 등록한다.

```java
package config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.support.PropertySourcesPlaceholderConfigurer;
import org.springframework.core.io.ClassPathResource;

@Configuration
public class PropertyConfig {

	@Bean
	public static PropertySourcesPlaceholderConfigurer properties() {
		PropertySourcesPlaceholderConfigurer configurer = new PropertySourcesPlaceholderConfigurer();
		configurer.setLocations(
				new ClassPathResource("db.properties"),
				new ClassPathResource("info.properties"));
		return configurer;
	}

}
```
 - PropertySourcesPlaceholderConfigurer().setLocations 메서드는 프로퍼티 파일 목록을 인자로 전달받는다.
 - 이때 스프링의 Resource 타입을 이용해서 파일 경로를 전달한다.
 - db.properties 파일이 클래스 패스에 위치하고 있다면 ClassPathResource 클래스를 이용해서 프로퍼티 파일 정보를 전달한다.
 - PropertySourcesPlaceholderConfigurer 타입 빈을 설정하는 메서드가 static 메서드이다.
 - PropertySourcesPlaceholderConfigurer 클래스가 특수한 목적의 빈이기 때문이며, __정적 메서드로 지정하지 않으면 원하는 방식으로 동작하지 않는다.__
 - PropertySourcesPlaceholderConfigurer 타입 빈은 setLocations() 메서드로 전달받은 프로퍼티 파일 목록 정보를 읽어와 필요할 때 사용한다.
 - 이를 위한 것이 @Value 어노테이션이다.

```java
package config;

import org.apache.tomcat.jdbc.pool.DataSource;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class DsConfigWithProp {
    @Value("${db.driver}")
    private String driver;
    @Value("${db.url}")
    private String jdbcUrl;
    @Value("${db.user}")
    private String user;
    @Value("${db.password}")
    private String password;

	@Bean(destroyMethod = "close")
	public DataSource dataSource() {
		DataSource ds = new DataSource();
		ds.setDriverClassName(driver);
		ds.setUrl(jdbcUrl);
		ds.setUsername(user);
		ds.setPassword(password);
		ds.setInitialSize(2);
		ds.setMaxActive(10);
		ds.setTestWhileIdle(true);
		ds.setMinEvictableIdleTimeMillis(60000 * 3);
		ds.setTimeBetweenEvictionRunsMillis(10 * 1000);
		return ds;
	}

}
```
 - @Value 어노테이션이 ${구분자} 형식의 플레이스홀더를 값으로 갖고 있다.
 - 이 경우 PropertySourcesPlaceholderConfigurer는 플레이스홀더의 값을 일치하는 프로퍼티 값으로 치환한다.
 - 예를 들어 ${db.drvier} 플레이스홀더를 db.properties에 정의되어 있는 "db.driver" 프로퍼티 값으로 치환한다.
 - 따라서 실제 빈을 생성하는 메서드는 @Value 어노테이션이 붙은 필드를 통해서 해당 프로퍼티의 값을 사용할 수 있다.

#### 빈 클래스에서 사용하기
> 빈으로 사용할 클래스에도 @Value 어노테이션을 붙일 수 있다.

```java
package spring;

import org.springframework.beans.factory.annotation.Value;

public class Info {

    @Value("${info.version}") // 프로퍼티를 필드에 할당할 때
	private String version;

	public void printInfo() {
		System.out.println("version = " + version);
	}

	@Value("${info.version}") // set 메서드에 적용할 때
	public void setVersion(String version) {
		this.version = version;
	}

}
```
 - @Value 어노테이션을 필드에 붙이면 플레이스홀더에 해당하는 프로퍼티를 필드에 할당한다.
 - @Value 어노테이션을 set 메서드에 적용할 수도 있다.