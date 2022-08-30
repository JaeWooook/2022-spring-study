# 프로필과 프로퍼티 파일

<br>

## 프로필

<br>
개발 목적 설정과 실 서비스 목적의 설정을 구분해서 작성하는  등 환경에 따라 알맞은 설정 정보를 사용하기 위한 기능

<br>

➡️ 스프링 컨테이너는 설정 집합 중에서 지정한 이름을 사용하는 프로필 선택 및 해당 프로필에 속한 설정을 이용해 컨테이너 초기화 할 수 있음

### 1️⃣ @Configuration 설정에서 프로필 사용

```

@Configuration
@Profile("dev")
public class DsDevConfig {

    @Bean(destroyMethod = "close")
    public DataSource dataSource(){
        DataSource ds = new DataSource();
        ds.setDriverClassName("com.mysql.jdbc.Driver");
        ...
        return ds;

    }

}

```

→ 스프링 컨테이너를 초기화할 때 dev 프로필을 활성화하면 DsDevConfig 클래스를 설정으로 이용!


<br>

### 🔎 특정 프로필 선택

컨테이너를 초기화하기 전에 setActiveProfiles() 메소드 사용해 프로필 선택!

```
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
context.getEnvironment().setActiveProfiles("dev");
context.register(MemberConfig.class, DsDevConfig.class, DsRealConfig.class);
context.refresh();
```

- getEnviroment() : 스프링 실행 환경을 설정하는데 사용되는 Enviroment 리턴
    - setActiveProfiles() 메소드를 이용해 사용할 프로필 선택 가능! 2개 이상 활성화 한다면 각 프로필 이름 , 로 구분해 전달

- register() : 설정 파일 목록 지정
- refresh() : 컨테이너 초기화

📌 프로필 선택하기 전에 설정 정보를 먼저 전달하면 프로필을 지정한 설정이 사용되지 않음! 설정 읽어오는 과정에서 빈 찾지 못해 익셉션 발생 ∴ **설정 정보 목록 불러오기 → 프로필 설정**



### 중첩 클래스를 이용한 프로필 설정

```
@Configuration
@EnableTransactionManagement
public class MemberConfig {

    @Configuration
    @Profile("dev")
    public static class DsDevConfig {

        @Bean(destroyMethod = "close")
        public DataSource dataSource(){
            DataSource ds = new DataSource();
            ds.setDriverClassName("com.mysql.jdbc.Driver");
            ...
            return ds;

        }

    @Configuration
    @Profile("real")
    public static class DsRealConfig {

        @Bean(destroyMethod = "close")
        public DataSource dataSource(){
            DataSource ds = new DataSource();
            ds.setDriverClassName("com.mysql.jdbc.Driver");
            ...
            return ds;

        }

}
```
중첩된 @Configuration 설정을 사용할땐 중첩클래스를 static으로 해야함



<br>

### 2️⃣ spring.profiles.active 시스템 프로퍼티에 사용할 프로필 값 지정

```
명령 행에 -Dspring.profiles.active=dev 옵션을 주거나

System.setProperty("spring.profiles.active","dev") 코드로 
spring.profiles.active 시스템 프로퍼티 값을 "dev"로 설정하면 dev 프로필 활성

AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(
    MemberConfig.class, DsDevConfig.class, DsRealConfig.class
);
```

➕ 스프링 설정은 두 개 이상의 프로필 이름을 가질 수 있음
```
@Configuration
@Profile("real,test")
public class DataSourceJndiConfig{
    ...
}
```
real 프로필을 사용할때와 test 프로필 사용할때 모두 해당 설정 사용!

➕ 프로필 값 앞에 ! 있다면 해당 프로필이 활성화 되지 않을 때 사용함

<br><br>

## 프로퍼티 파일을 이용한 프로퍼티 설정

스프링은 외부의 프로퍼티 파일을 이용해 스프링 빈을 설정함.

 프로퍼티 값을 자바 설정에서 사용할 수 있으며 이를 통해 설정 일부를 외부 프로퍼티 파일을 사용해 변경할 수 있음!


 <br>

 ### 1️⃣ @Configuration 어노테이션 이용 자바 설정에서의 프로퍼티 사용

- PropertySourcesPlaceholderConfigurer 빈 설정
- @Value 어노테이션으로 프로퍼티 값 사용


```
@Configuration
public class PropertyConfig {

    @Bean
    public static PropertySourcesPlaceholderConfigurer properties(){
        PropertySourcesPlaceholderConfigurer configurer = new PropertySourcesPlaceholderConfigurer();
        configurer.setLocations(new ClassPathResource("db.properties"), new ClassPathResource("info.properties"));

        return configurer;
    }
}
```
- setLocations() : 프로퍼티 파일 목록을 인자로 전달받음
- PropertySourcesPlaceholderConfigurer는 static 메소드 : 특수한 목적의 빈이기 때문에 정적 메소드로 지정하지 않으면 원하는 방식으로 동작 x

- PropertySourcesPlaceholderConfigurer 타입 빈은 setLocation() 메소드로 전달받은 프로퍼티 파일 목록 정보를 읽어와 필요할때 사용

```
@Configuration
public class DsConfigWithProp {

    @Value("${db.driver}") //${} 형식의 플레이스 홀더
    private String driver;

    @Value("${db.url}")
    private String jdbcUrl;

    @Value("${db.user}")
    private String user;

    @Value("${db.password}")
    private String password;

    @Bean(destroyMethod = "close")
    public DataSource dataSource(){
        DataSource ds = new DataSource();
        ds.setDriverClassName(driver);
        // @Value가 붙은 필드 이용해 실제 프로퍼티 값 이용
        ...
    }
}
```

- PropertySourcesPlaceholderConfigurer는 플레이스 홀더의 값을 일치하는 프로퍼티 값으로 치환
- 실제 빈을 형성하는 메소드는 @Value 어노테이션이 붙은 필드 통해 해당 프로퍼티 값 사용 가능