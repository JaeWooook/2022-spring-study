# 빈 라이프사이클과 범위

<br>

## 컨테이너 초기화와 종료

<br>

```
1. 컨테이너 초기화

AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(AppCtx.class);

2. 컨테이너에서 빈 객체 구해서 사용

MemberRegisterService regSvc = ctx.getBean(MemberRegisterService.class);

3. 컨테이너 종료

ctx.close();

```

<br>

### 1. 스프링 컨테이너 초기화

: AnnotationConfigApplicationContext(컨텍스트) 객체를 생성하는 시점에서 스프링 컨테니어 초기화가 일어남

    ✍️ 컨테이너 초기화가 완료되면 컨테이너 사용가능  = getBean()과 같은 메서드를 이용해 컨테이너에 보관된 빈 객체 구할 수 있음  
<br>

### 2. 빈 객체 생성 및 의존주입

: 스프링 컨테이너는 설정클래스에서 정보를 읽어와 알맞은 빈 객체 생성 및 각 빈을 연결(의존 주입) 작업 수행

<br>

### 3. 컨테이너 종료

: close() 메소드 이용해 컨테이너 종료

📌 컨테이너가 종료되면 컨테이너 내의 빈 객체도 소멸!



<br><br>

## 스프링 빈 객체의 라이프 사이클

* 스프링 컨테이너 : 스프링에서 자바 객체들을 관리하는 공간으로 빈 객체의 라이프 사이클을 관리함

![image](https://user-images.githubusercontent.com/81572478/179510133-2f2e516a-4397-4a88-9765-5ed3079b4075.png)


<br>

#### 1. 객체 생성 및 의존 설정 
: 스프링 컨테이너를 초기화 할때 컨테이너는 가장 먼저 빈 객체를 생성하고 의존을 설정함

<Br>

#### 2. 초기화 
: 모든 의존 설정이 완료되면 빈 객체의 초기화 수행

→ 초기화 위해 스프링은 빈 객체의 **지정된 메소드** 호출

<br>

#### 3. 소멸 
: 스프링 컨테이너 종료하면 컨테이너는 빈 객체의 소멸을 처리함

→ 소멸 위해 스프링은 빈 객체의 **지정된 메소드** 호출


<br>
<br>

###  🔎 빈 객체의 초기화와 소멸

<br>

#### 1️⃣ 스프링 인터페이스

<br>

- InitializingBean 인터페이스의 afterPropertiesSet() 메소드

: 빈 객체가 InitializingBean 인터페이스를 구현하면 스프링 컨테이너는 초기화 과정에서 빈 객체의 afterPropertiesSet() 메소드를 실행

∴ 빈 객체를 생성한 뒤 초기화 과정이 필요하면 afterPropertiesSet() 메소드를 알맞게 구현하면 됨


<Br>

- DisposableBean 인터페이스의 destroy() 메소드

: 빈 객체가 DisposableBean 인터페이스를 구현하면 스프링 컨테이너는 소멸 과정에서 빈 객체의 destroy() 메소드를 실행

∴ 빈 객체를 생성한 뒤 소멸 과정이 필요하면 destroy() 메소드를 알맞게 구현하면 됨

<br>


    ✅ 초기화와 소멸과정이 필요한 경우?

    - 데이터베이서 커넥션 풀
    : 커넥션 풀을 위한 빈 객체는 초기화 과정에서 데이터 베이스 연결을 생성해야 함! 또한, 컨테이너를 사용하는 동안 연결을 유지하고 빈 객체를 소멸할때 연결을 끊어야 함

    → 초기화 : DB와 연결 설정
    → 소멸 : DB와 연결 해제


<BR>

#### 2️⃣ 커스텀 메소드

<BR>

: 외부에서 제공받은 클래스는 스프링 인터페이스를 구현하도록 수정할 수 없음. 혹은, 스프링 인터페이스를 사용하고 싶지 않은 경우 직접 메소드 지정!


- @Bean(initMethod="초기화 메소드", destroyMethod="소멸 메소드")

```
@Bean(initMethod = "connect", destroyMethod = "close")
    public Client client(){
        Client client = new Client();
        client.setHost("host");
        return client;
    }
```
<br>


    ✅ initMethod 속성을 사용하지 않고 직접 초기화?

    @Bean(destroyMethod = "close")
    public Client client(){
        Client client = new Client();
        client.afterPropertiesSet(); // 초기화 메소드
        client.setHost("host");
        return client;
    }

    : 그냥 코드 작성하면 됨!

    이때, 해당 클래스가 InitializeingBean 인터페이스를 구현했다면 afterPropertiesSet() 메소드를 실행하므로 초기화 메소드가 두번 호출됨 = 에러


➕ initMethod, destroyMethod 속성에 지정한 메소드는 파라미터가 없어야 함!

<br><Br>

### 빈 객체의 생성과 관리 범위

<br>

: 스프링 컨테이너는 기본적으로 빈 객체를 한 개만 생성 = 싱글톤


<Br>

- 싱글톤 범위 빈: 한 식별자에 대해 한 개의 객체만 존재하는 빈

```
Client client1 = ctx.getBean("client",Client.class);
Client client2 = ctx.getBean("client",Client.class);
// client1 == client2  -> true
```
<Br><br>

- 프로토타입 범위 빈: 빈 객체를 구할때마다 매번 새로운 객체 생성

ex) "clien" 이름을 갖는 빈을 프로토타입 범위의 빈으로 설정하면 getBean() 메소드는 매번 새로운 객체 리턴

```
Client client1 = ctx.getBean("client",Client.class);
Client client2 = ctx.getBean("client",Client.class);
// client1 != client2  -> true
```

➕ 프로토타입 범위 빈은 완전한 라이프 사이클 따르지 않음 

: 생성한 프로토타입 빈 객체의 소멸 메소드 실행x
➡️ 소멸 처리 코드에서 직접 해야함

<Br><br>

📌 특정 빈을 프로토타입 범위로 지정: @Scope("prototype")
```
    @Bean
    @Scope("prototype")
    public Client client(){
        Client client = new Client();
        client.setHost("host");
        return client;
    }

```