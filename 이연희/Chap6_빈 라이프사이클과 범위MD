# chap6 빈 라이프사이클과 범위
스프링 컨테이너는 **초기화와 종료**라는 라이프사이클을 갖는다. 
1. AnnotationConfigApplicationContext의 객체를 생성하는 시점에 **스프링 컨테이너를 초기화**한다. 스프링 컨테이너는 설정 클래스에서 정보를 읽어 빈 객체를 생성하고 의존 관계 주입 작업을 수행한다.
2. 컨테이너 초기화가 완료되었으니 컨테이너를 사용할 수 있다. getBean()을 사용하여 빈을 가져온다.
3. 컨테이너 사용이 끝나면 close()를 통해 컨테이너를 종료한다. 
- 컨테이너 초기화 -> 빈 객체 생성, 의존주입, 초기화
- 컨테이너 종료 -> 빈 객체 소멸
```java
//1.컨테이너 초기화
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(AppContext.class);

//2.컨테이너에서 빈 객체를 구해서 사용
Greeter g = ctx.getBean("greeter",Greeter.class);
String msg = g.greet("스프링");
System.out.println(msg);

//3.컨테이너 종료
ctx.close();
```

## 스프링 빈 객체 라이프사이클

	객체생성 -> 의존설정 -> 초기화 -> 소멸

- 스프링 컨테이너를 초기화할 때 스프링 컨테이너는 빈 객체를 생성하고 의존을 설정한다. **의존 자동 주입을 통해 의존 설정이 이 시점에 수행된다.** 모든 의존 설정이 완료되면 빈 객체의 초기화를 수행한다. 빈 객체를 초기화하기 위해 스프링은 빈 객체에 지정된 메서드를 호출한다. 

### InitializingBean, DisposableBean
실제 잘 사용되지 않지만 스프링은 스프링 컨테이너 객체를 초기화하고 소멸하는 인터페이스에 초기화, 소멸 메서드를 정의하고 있다. 
afterPropertiesSet() 메서드는 초기화 과정에서 호출되고, destroy() 메서드는 소멸 과정에서 호출된다. 
```java
public interface InitializingBean{
	void afterPropertiesSet() throws Exception;
}
public interface DisposableBean{
	void destroy() throws Exception;
}	
```

### 커스텀 메서드
직접 구현한 클래스가 아닌 외부에서 제공받은 클래스를 스프링 빈 객체로 설정하고 싶은 경우 커스텀 메서드를 사용한다. @Bean 태그에서 initMethod 속성과 destroyMethod 속성을 사용해서 초기화, 소멸 메서드 이름을 지정하면 된다. 
```java
public class Client{
	private String host;
    public void setHost(String host){...}
    
    public void connect(){...};
    public void clse(){...};
}
```
```java
@Bean(initMethod="connect", destroyMethod="close")
public Client client(){
	Client clt = new Client();
    clt.setHost("host");
    return clt;
}
```

## 빈 객체 생성과 관리 범위
### 싱글톤
다음 두 객체는 동일한 빈 객체를 참조한다. 스프링은 기본적으로 객체를 싱글톤으로 유지하기 때문이다. 
```java
Client clt1 = ctx.getBean("client",Client.class);
Client clt2 = ctx.getBean("client",Client.class);
```
### 프로토타입
매번 새로운 객체를 구하고 싶으면 빈을 **프로토타입 범위의 빈**으로 설정하면 된다. 그럼 getBean() 메서드는 매번 새로운 객체를 생성해서 리턴하기 때문에 clt1과 clt2는 서로 다른 객체가 된다.
```java
@Configuration
public class AppCtxWithPrototype{
	@Bean
    @Scope("prototype")
    public Client client(){
    	Client clt = new Client();
        clt.setHost("host");
        return clt;
    }
}
```
프로토타입 범위를 갖는 빈은 완전한 라이프사이클을 따르지 않는다. 스프링 컨테이너는 프로토타입 빈 객체를 생성하고 프로퍼티를 설정하고 초기화 작업까지는 수행하지만, 컨테이너를 종료한다고 해서 생성한 프로토타입 빈 객체의 소멸 메서드까지 실행하진 않는다. 따라서 프로토타입 범위 빈을 사용할 때는 빈 객체의 소멸 처리를 코드에서 직접 해야 한다. 