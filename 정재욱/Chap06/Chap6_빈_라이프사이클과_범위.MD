## Chapter6 빈 라이프사이클과 범위
### 컨테이너 초기화와 종료
```java
//1. 컨테이너 초기화
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(AppContext.class);

//2. 컨테이너에서 빈 객체를 구해서 사용
Greeter g = ctx.getBean("greeter", Greeter.class);
String msg = g.greet("스프링");
System.out.println(msg);

//3. 컨테이너 종료
ctx.close();
```
#### 1. 컨테이너 초기화
 - AnnotationConfigApplicationContext 생성자를 이용해서 스프링 컨테이너를 초기화 한다.
 - 스프링 컨테이너는 설정 클래스에서 정보를 읽어와 알맞은 빈 객체를 생성하고, 각 빈을 연결(의존주입)하는 작업을 수행한다.
#### 2. 컨테이너 사용
 - 컨테이너는 getBean()과 같은 메서드를 이용해서 컨테이너에 보관된 빈 객체를 구하는 것이다.

#### 3. 컨테이너 종료
 - 컨테이너의 사용이 끝나면 종료한다.
 - close()메서드는 AbstractApplicationContext 클래스에 정의되어 있다.
 - 자바 설정을 사용하는 AnnotationConfigApplicationContext 클래스나 XML 설정을 사용하는 GenenricXmlApplicationContext 클래스 모두 AbstractApplicationContext클래스를 상속 받는다.
 - 위의 클래스들에서 close메서드의 클래스를 상속받고 있기 때문에 close()메서드를 이용해서 컨테이너를 종료할 수 있다.

### 스프링 빈 객체의 라이프사이클
> 객체 생성 -> 의존 설정 -> 초기화 -> 소멸

 - 스프링 컨테이너를 초기화할 때 스프링 컨테이너는 가장 먼저 빈 객체를 생성하고 의존을 설정한다.
 - 의존 자동 주입을 통한 의존 설정이 이 시점에 수행된다.
 - 모든 의존 설정이 완료되면 빈 객체의 초기화를 수행한다.
 - 빈 객체를 초기화하기 위해 스프링은 빈 객체의 지정된 메서드를 호출한다.
 - 스프링 컨테이너를 종료하면 스프링 컨테이너는 빈 객체의 소멸을 처리한다. 이때에도 지정한 메서드를 호출한다.

#### 빈 객체의 초기화와 소멸 : 스프링 인터페이스
> 빈객체를 초기화 하고 소멸하기 위해 빈 객체의 지정한 메서드를 호출한다.

#### InitializingBean
> org.springframework.beans.factory.InitializingBean
```java
public interface InitializingBean {
    void afterPropertiesSet() throws Exception;
}
```
 - 빈 객체가 InitializingBean 인터페이스를 구현한다.
 - 스프링 컨테이너는 초기화 과정에서 빈 객체의 afterPropertiesSet() 메서드를 실행한다.
 - 빈 객체를 생성한 뒤에 초기화 과정이 필요하면 InitializingBean 인터페이스를 상속하고 afterPropertiesSet()메서드를 알맞게 구현하면 된다.

#### DisposableBean
> org.springframework.beans.factory.DisposableBean
```java
public interface DisposableBean {
    void destroy() throws Exception;
}
```
 - 스프링 컨테이너는 빈 객체가 DisposableBean 인터페이스를 구현한 경우 소멸 과정에서 빈 객체의 destroy() 메서드를 실행한다.
 - 빈 객체의 소멸 과정이 필요하면 DisposableBean 인터페이스를 상속하고 destroy()메서드를 알맞게 구현하면 된다.

#### 빈 객체 초기화와 소멸 과정의 예시
> 데이터베이스 커넥션풀
 - 커넥션 풀을 위한 빈 객체는 초기화 과정에서 데이터베이스를 연결을 생성한다.
 - 컨테이너를 사용하는 동안 연결을 유지하고,
 - 빈 객체를 소멸할 때 사용중인 데이터베이스 연결을 끊어야 한다.

> 채팅 클라이언트
 - 채팅클라이언트와 서버의 연결을 시작하면 빈 객체를 초기화 하고,
 - 채팅을 종료할 때 빈 객체를 소멸하여 채팅클라이언트와 서버의 연결을 종료한다.

```java
package config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import spring.Client;

@Configuration
public class AppCtx {

    @Bean
    public Client client() {
        Client client = new Client();
        client.setHost("host");
        return client;
    }
}
```

```java
package spring;

import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;

public class Client implements InitializingBean, DisposableBean {

	private String host;

	public void setHost(String host) {
		this.host = host;
	}
//채팅 클라이언트 빈 객체 초기화로 서버와 연결 시작
	@Override
	public void afterPropertiesSet() throws Exception {
		System.out.println("Client.afterPropertiesSet() 실행");
	}

	public void send() {
		System.out.println("Client.send() to " + host);
	}
//채팅 클라이언트 빈 객체 소멸로 서버와 연결 종료
	@Override
	public void destroy() throws Exception {
		System.out.println("Client.destroy() 실행");
	}

}
```
 
```java
package main;

import java.io.IOException;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.support.AbstractApplicationContext;

import config.AppCtx;
import spring.Client;

public class Main {

	public static void main(String[] args) throws IOException {
		AbstractApplicationContext ctx = 
				new AnnotationConfigApplicationContext(AppCtx.class);

		Client client = ctx.getBean(Client.class);
        //출력 : Client.afterPropertiesSet() 실행
		client.send();
        //출력 : Client.send() to host
		ctx.close();
        //출력 : Client.destroy() 실행
	}

}
```
 - 빈 객체의 생성을 마무리한뒤에 초기화 메서드를 실행
 - 가장 마지막에 컨테이너를 종료하면서 destroy()메서드를 실행

#### 빈 객체의 초기화와 소멸 : 커스텀 메서드
> 직접 구현한 클래스가 아닌 외부에서 제공받은 클래스를 스프링 빈 객체로 설정하고 싶을때
> 스프링 설정에서 직접 메서드를 지정하는 방법

```java
package spring;

public class Client2 {

	private String host;

	public void setHost(String host) {
		this.host = host;
	}

	public void connect() {
		System.out.println("Client2.connect() 실행");
	}

	public void send() {
		System.out.println("Client2.send() to " + host);
	}

	public void close() {
		System.out.println("Client2.close() 실행");
	}

}
```
 - Client2 클래스를 빈으로 사용하려면 초기화 과정에서 connect() 메서드를 실행하고, 소멸과정에서 close() 메서드를 실행해야 한다.

```java
	@Bean(initMethod = "connect", destroyMethod = "close")
	@Scope("singleton")
	public Client2 client2() {
		Client2 client = new Client2();
		client.setHost("host");
		return client;
	}
```
 - Client2클래스를 빈으로 사용할때 설정클래스에서 initMethod 속성으로 초기화에 사용될 메서드 이름인 connect를 지정한다.
 - destroyMethod속성에 소멸과정에서 사용될 메서드 이름인 close를 지정해준다.

```java
package main;

import java.io.IOException;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.support.AbstractApplicationContext;

import config.AppCtxWithPrototype;
import spring.Client;

public class MainWithPrototype {

	public static void main(String[] args) throws IOException {
		AbstractApplicationContext ctx = 
				new AnnotationConfigApplicationContext(AppCtxWithPrototype.class);
        
		Client client2 = ctx.getBean(Client.class);
        //출력 : Client2.connect() 실행
        client2.send();
        //출력 : Client2.send() to host
		ctx.close();
        ////출력 : Client2.destroy() 실행
	}

}
```
 > 초기화 코드는 설정클래스에서 자바코드이기 때문에 직접 메서드를 호출할 수 있다.

```java
@Bean(destroyMethod = "close")
public Client2 client2() {
    Client2 client = new Client2();
    client.setHost("host");
    client.connect();
    return client;
        }
```
 - connect()메서드를 이용해서 초기화를 해줄 수 도 있다.

 > 초기화 관련 메서드를 빈 설정 코드에서 직접 실행 할때 주의점
```java
@Bean
public Client client() {
    Client client = new Client();
    client.setHost("host");
    client.afterPropertiesSet();
    return client;
        }
```
 - afterPropertiesSet()메서드는 빈 객체를 생성하면서 초기화를 진행하기 때문에 이렇게 하게되면 __afterPropertiesSet()를 2번 호출하게 된다. !!__

### 빈 객체의 생성과 관리 범위
> 스프링 컨테이너는 빈 객체를 한 개만 생성한다.

```java
		Client client1 = ctx.getBean("client", Client.class);
		Client client2 = ctx.getBean("client", Client.class);
        //client1 == client2 -> true
```
 - 빈 객체는 같은 Client.class타입인 경우 1개의 객체만을 생성한다.
 - client1과 client2는 동일한 객체가 된다.

> 위와 같이 한 식별자에 대해 한 개의 객체만 존재하는 빈은 싱글톤 범위를 갖는다.

#### 특정 빈을 프로토타입 범위로 지정하는 방법
```java
	@Bean
	@Scope("prototype")
	public Client client() {
		Client client = new Client();
		client.setHost("host");
		return client;
	}
```
 - 빈 설정클래스에서 @Scope 어노테이션을 이용해 "prototype"이라고 적어준다.

#### 싱글톤 스코프값을 명시적으로 지정하는 방법
```java
	@Bean
	@Scope("singleton")
	public Client2 client2() {
		Client2 client = new Client2();
		client.setHost("host");
		return client;
	}
```

 - 빈 설정클래스에서 @Scope 어노테이션을 이용해 "singleton"이라고 적어준다.

 > #### 주의!
 > - 프로토타입 범위를 갖는 빈은 완전한 빈의 라이프사이클을 따르지 않는다.
 > - 즉, __스프링 컨테이너는 프로토타입의 빈 객체를 생성하고 프로퍼티를 설정하고 초기화 작업까지는 수행하지만__
 > - __컨테이너를 종료한다고 해서 생성한 프로퍼티타입 빈 객체의 소멸 메서드를 실행하지 않는다.__
 > - 따라서, 프로토타입 범위의 빈을 사용할 때에는 빈 객체의 소멸 처리를 코드에서 직접 해야 한다.
