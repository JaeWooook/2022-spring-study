# Chap17 프로필과 프로퍼티 파일
## 프로필
개발을 진행하는 동안에는 실제 서비스 목적으로 운영중인 DB를 이용할 수는 없다. 실제 서비스 장비에 배포하기 전에 설정 정보를 변경하고 배포하는 방법을 사용할 수도 있지만 너무 원시적이고 실수하기 쉽다. 
에러를 방지하기 위해 처음부터 개발 목적 설정과 실 서비스의 목적 설정을 구분해서 작성하는 것이다. 이를 위한 스프링 기능을 프로필(profile)이라 한다.

프로필은 논리적인 이름으로서 설정 집합에 프로필을 지정할 수 있다. 스프링 컨테이너는 설정 집합 중에서 지정한 이름을 사용하는 프로필을 선택하고 해당 프로필에 속한 설정을 이용해서 컨테이너를 초기화할 수 있다. 

예를 들어 로컬 개발 환경을 위한 DataSource 설정을 "dev" 프로필로 지정하고 실 서비스를 위한 설정을 "real" 프로필로 지정한 뒤, "dev" 프로필을 이용해서 스프링 컨테이너를 초기화할 수 있다. 그럼 "dev" 프로필에 정의된 빈을 사용하게 된다. ![](https://velog.velcdn.com/images/yh_lee/post/86160b7b-6cc1-467b-be85-26f24fa43c55/image.png)

### @Configuration 설정에서 프로필 사용하기
@Configuration 어노테이션을 이용한 설정에서 프로필을 지정하려면 @Profile 어노테이션을 이용한다. 
```java
@Configuration
@Profile("dev")
public class DsDevConfig{
	@Bean(destroyMethod="close")
    public DataSource dataSource(){
    	...
    }
}

@Configuration
@Profile("real")
public class DsRealConfig{
	@Bean(destroyMethod="close")
    public DataSource dataSource(){
    	...
    }
}
```
스프링 컨테이너를 초기화할 때 "dev" 프로필을 활성화하면 DsDevConfig 클래스를 설정으로 사용한다. 
두 dataSource 빈 중에 어떤 빈을 사용할지는 활성화한 프로필에 따라 달라진다. 특정 프로필을 선택하려면 컨테이너를 초기화하기 전에 setActiveProfiles() 메서드를 사용해서 프로필을 설정해야 한다.

	context.getEnviroment().setActiveProfiles("dev");
    context.register(MemberConfig.class, DsDevConfig.class, DsRealConfig.class);
    context.refresh();

프로필을 사용할 때 주의할 점은 설정 정보를 전달하기 전에 어떤 프로필을 사용할지 지정해야 한다는 점이다. dev로 설정 후 register() 메서드로 설정 파일 목록을 지정한다. 이후 refresh() 메서드를 실행해서 컨테이너를 초기화했다. 

시스템 프로퍼티를 이용하여 설정할 수도 있다. 

	java -Dspring.profiles.active=dev main.Main
    
자바의 시스템 프로퍼티뿐만 아니라 OS의 "spring.profiles.active" 환경 변수에 값을 설정해도 된다. 프로필의 우선순위는 다음과 같다. 
- setActiveProfiles()
- 자바 시스템 프로퍼티 
- OS 환경 변수

### @Configuration을 이용한 프로필 설정
중첩 클래스를 이용해서 프로필 설정을 한 곳으로 모을 수 있다.
```java
@Configuration
public class MemberConfigWithProfile{
	@Autowired
    private DataSource dataSource;
    
    @Bean
    public MemberDao memberDao(){
    	return new MemberDao(dataSource);
    }
    
    @Profile("dev")
	public class DsDevConfig{
		@Bean(destroyMethod="close")
   	 	public DataSource dataSource(){
    	...
    	}
	}

    @Profile("real")
    public class DsRealConfig{
        @Bean(destroyMethod="close")
        public DataSource dataSource(){
            ...
        }
    }    
}
```
### 다수 프로필 설정
스프링 설정은 두 개 이상의 프로필 이름을 가질 수 있다. 
```java
@Configuration
@Profile("real,test")
public class DataSourceJndiConfig{
	...
}
```
프로필을 설정할 때 느낌표를 사용하면 해당 프로필이 활성화되지 않았을 때 사용한다는 것을 의미한다. 보통 "!프로필" 형식은 특정 프로필이 사용되지 않을 때 기본으로 사용할 설정을 지정하는 용도로 사용된다.
```java
@Configuration
@Profile("!real")
public class DataSourceJndiConfig{
	...
}
```
## 프로퍼티 파일을 이용한 프로퍼티 설정
스프링은 외부의 프로퍼티 파일을 이용해서 스프링 빈을 설정하는 방법을 제공하고 있다. 다음과 같은 db.properties 파일이 있다고 하자. 이 파일의 프로퍼티 값을 자바 설정에서 사용할 수 있으며 이를 통해 설정 일부를 외부 프로퍼티 파일을 사용해서 변경할 수 있다. 

	db.driver=com.mysql.jdbc.Driver
    db.url=jdbc:mysql://localhost/spring5fs?characterEncoding=utf8
    db.user=spring5
    db.password=spring5
    
## @Configuration 어노테이션 이용 자바 설정에서의 프로퍼티 사용
자바 설정에서 프로퍼티 파일을 이용하려면 다음 두 가지를 설정한다.
- PropertySourcesPlaceholderConfigurer 빈 설정
- @Value 어노테이션으로 프로퍼티 값 사용

```java
@Configuration
public class PropertyConfig{
	@Bean
    public static PropertySourcesPlaceholderConfigurer properties(){
    	PropertySourcesPlaceholderConfigurer configurer = new PropertySourcesPlaceholderConfigurer();
        configurer.setLocations(
        	new ClassPathResource("db.properties"),
            new ClassPathResource("info.properties"));
            
        return configurer;
    }
}
```
setLocations()는 프로퍼티 파일 목록을 인자로 전달받는다. 이때 스프링은 Resource 파입을 이용해서 파일 경로를 전달한다. 

위 코드에서 주의할 점은 PropertySourcesPlaceholderConfigurer 타입 빈을 설정하는 메서드가 정적(static)이라는 것이다. 이는 PropertySourcesPlaceholderConfigurer 클래스가 특수한 목적의 빈이기 때문이며 정적 메서드로 지정하지 않으면 원하는 방식으로 동작하지 않는다. 

PropertySourcesPlaceholderConfigurer 타입 빈은 setLocation() 메서드로 전달받은 프로퍼티 파일 목록 정보를 읽어와 필요할 때 사용한다. 이를 위한 것이 @Value 어노테이션이다. 
```java
@Configuration
public class DsConfigWithProp{
	@Value("${db.driver}")
    private String driver;
    ...
}
```
@Value 어노테이션이 ${구분자} 형식의 플레이스홀더 값으로 갖고 있다. ${db.driver} 플레이스홀더 값을 db.properties에 "db.driver" 프로퍼티 값으로 치환한다. 따라서 실제 빈을 생성하는 메서드는 @Value 어노테이션이 붙은 필드를 통해서 해당 프로퍼티의 값을 사용할 수 있다. 

### 빈 클래스에서 사용하기 
빈으로 사용할 클래스에도 @Value 어노테이션을 붙일 수 있다. @Value 어노테이션을 필드에 붙이면 플레이스홀더에 해당하는 프로퍼티를 필드에 할당한다. 
```java
public class Info{
	@Value("${info.version}")
    private String version
    ...
}
```
다음과 같이 @Value 어노테이션을 set 메서드에 적용할 수도 있다. 
```java
public class Info{
	private String version;
   	...
    @Value("${info.version}")
    public void setVersion(String version){
    	this.version=version;
    }
}
```