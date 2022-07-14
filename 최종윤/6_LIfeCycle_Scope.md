
스프링 컨테이너는 빈 객체의 라이프사이클을 관리

컨테이너 초기화 → 빈 객체의 생성, 의존주입, 초기화

객체 생성 -> 의존 설정 -> 초기화 -> 소멸

1) 빈 객체의 초기화와 소멸 : 스프링 인터페이스

다음 인터페이스에 빈객체의 초기화와 소멸 메서드를 정의함
```
public interface InitializingBean{
void afterpropertiesSet() throws Exception;
}

public interface DisposableBean{
void destroys() throws Exception;
}
```
빈 객체가 InitializingBean 인터페이스를 구현하면 스프링 컨테이너는 초기화 과정에서 빈 객체의 afterpropertiesSet() 메서드를 실행함
```
package spring;

import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;

public class Client implements InitializingBean, DisposableBean{
	private String host;
	
	public void setHost(String host) {
		this.host = host;
	}

	@Override
	public void afterPropertiesSet() throws Exception{ //빈객체 생성을 마무리 한뒤에 초기화 메서드를 실행
		System.out.println("Client.afterPropertiesSet() 실행");
	}
	
	public void send() {
		System.out.println("Client.send() to "+host);
	}
	@Override
	public void destroy() throws Exception {  //스프링컨테이너를 종료하면 호출
		System.out.println("Client.destroy() 실행");
		
	}
	
}

package config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import spring.Client;
import spring.Client2;

@Configuration
public class AppCtx {

	@Bean
	public Client client() {
		Client client = new Client();
		client.setHost("host");
		return client;
	}

}
package main;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.support.AbstractApplicationContext;

import java.io.IOException;
import config.AppCtx;
import spring.Client;

public class Main {
	public static void main(String[] args)throws IOException{
		AbstractApplicationContext ctx = 
				new AnnotationConfigApplicationContext(AppCtx.class);
		
		Client client = ctx.getBean(Client.class);
		client.send();
		
		ctx.close();
	}
}	
```
2) 빈 객체의 초기화와 소멸 : 커스텀 메서드

직접 구현한 클래스가 아닌 외부에서 제공받은 클래스를 스프링 빈 객체로 설정하고 싶은 경우 소스 코드를 받지 않았다면 두 인터페이스를 구현하도록 수정할 수 없다. 이 경우 스프링 설정에서 직접 메서드를 지정할 수 있다.
```
package spring;


public class Client2{
	private String host;
	
	public void setHost(String host) {
		this.host = host;
	}

	public void connect(){
		System.out.println("Client2.connect() 실행");
	}
	
	public void send() {
		System.out.println("Client2.send() to "+host);
	}

	public void close(){
		System.out.println("Client2.close() 실행");
		
	}

}
```
그리고 초기화와 소멸과정에서 사용할 메서드 이름인 connect/close를 지정해준다.

```	
	@Bean(initMethod="connect", destroyMethod="close")
	public Client2 client2() {
		Client2 client = new Client2();
		client.setHost("host");
		return client;
	}
*initMethod 속성과 destroyMethod 속성에 지정한 메서드는 파라미터가 없어야한다.
```
< 빈 객체의 생성과 관리범위>

빈은 싱글톤 범위를 갖는다.

Client client1 = ctx.getBean("client",Client.class);
Client client2 = ctx.getBean("client",Client.class);
//client1 ==client2 -> true

3. 빈 객체의 생성과 관리범위
2장에서 우리는 빈 객체는 기본적으로 싱글톤 범위(scope)를 가진다고 배웠다.

- 자바 싱글톤 : 클래스 식별자에 대해서 클래스당 하나의 객체만 생성하도록 보장
- 스프링 싱글톤 : 컨테이너에서 빈 id별로 하나의 빈 객체만 생성하도록 보장
- 싱글톤이 아닌 프로토타입의 scope를 가지도록 빈을 설정할 수도 있다.

⇒ 프로토타입 : 빈 객체를 구할때마다 매번 새로운 객체를 생성.

특정 빈을 프로토타입 범위로 지정하려면 @Scope 어노테이션의 값으로 "prototype"을 @Bean 어노테이션과 함께 사용하면 됨.

↓설정파일

@Bean @Scope("prototype") public Client client(){ // ~~ 빈 설정 구성 }

싱글톤 범위를 명시적으로 지정하고 싶다면 @Scope 어노테이션 값으로 "singleton"을 주면 됨.

※ 프로토 타입 범위를 갖는 빈은 완전한 라이프사이클을 따르지 않는것에 주의해야함.
-  프로토 타입의 빈 객체를 설정하고 초기화 작업까지는 수행하지만, 컨테이너를 종료한다고 해서 생성한 프로토타입 빈 객체의 소멸 메서드를 실행하지는 않음.

-  즉 프로토타입 Scope의 빈 객체를 사용할 때에는 빈 객체의 소멸 처리를 개발자가 코드로 직접 해줘야함.

- 프로토타입 범위의 빈 : 컨테이너를 종료한다고 해서 생성한 프로토타입 빈 객체의 소멸메서드를 실행하지 않는다. -> 잘 사용하지 않는 이유
