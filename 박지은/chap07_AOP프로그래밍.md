# AOP 프로그래밍

<BR>

## 프록시와 AOP

<BR>

### 1. 프록시

: 핵심 기능의 실행은 다른 객체에 위임하고 부가적인 기능을 제공하는 객체
➡️ 여러 객체에 공통으로 적용할 수 있는 기능 구현

- 대상 객체 : 실제 핵심 기능을 실행하는 객체

<bR><bR>

### 2. AOP

: 여러 객체에 공통으로 적용할 수 있는 기능을 분리해 재사용성을 높이는 프로그래밍 기법

<BR>

➡️ 핵심 기능을 구현한 코드의 수정없이 공통 기능을 적용!

- 핵심 기능에 공통 기능 삽입

    - 컴파일 시점에 코드에 공통 기능 삽입
    - 클래스 로딩 시점에 바이트 코드에 공통 기능 삽입
    - 런타임에 프록시 객체를 생성해 공통 기능 삽입
    
    <BR>
    
    🌱 스프링 AOP 방식은 프록시를 이용함!

    →  프록시 객체를 생성해 실제 객체의 기능을 실행하기 전/후에 공통기능 호출


<BR>
➕ 스프링 AOP는 프록시 객체를 자동으로 만들어주기 때문에 프록시 클래스를 직접 구현할 필요없고, 공통 기능을 구현한 클래스만 구현!!

![image](https://user-images.githubusercontent.com/81572478/179706802-ab7d8dee-cbb4-47ba-bf21-59c75de1811a.png)

<br>

#### 🔎 AOP 주요 용어

- Advice : 언제 공통 관심 기능을 핵심 로직에 적용할지 정의

- Joinpoint : Advice를 적용 가능한 지점 의미

- Pointcut : 실제 Advice가 Jointpoint 적용되는 Joinpoint 나타냄

- Weaving : Advice를 핵심 로직 코드에 적용하는 것

- Aspect : 여러 객체에 공통으로 적용되는 기능

<br>

#### 🔎 Around Advice

: 대상 객체의 메소드 실행 전, 후 또는 익셉션 발생 시점에 공통 기능을 실행하는데 사용

→ 다양한 시점에 원하는 기능 삽입 가능

<BR><bR>

### 스프링 AOP 구현

1. Aspect로 사용할 클래스에 @Aspect 어노테이션 붙임
2. @Pointcut 어노테이션으로 공통 기능을 적용할 Pointcut을 정의
3. 공통 기능을 구현한 메소드에 @Around 어노테이션 적용


```

@Aspect // 공통기능을 제공하는 클래스
@Order(2)
public class ExeTimeAspect {
    @Pointcut("execution(public * chap07..*(..))") // aspect를 적용할 대상
    private void publicTarget(){
    }

    @Around("publicTarget()") //공통 기능을 구현한 메소드
    public Object measure(ProceedingJoinPoint joinPoint) throws Throwable{
        long start = System.nanoTime(); // 기능 수행
        try{
           Object result = joinPoint.proceed(); // 대상 객체의 메소드 호출
            return result;
        }finally {
            long finish = System.nanoTime(); // 기능 수행
            Signature sig = joinPoint.getSignature();
            System.out.printf("%s.%s(%s) 실행시간 : %d ns\n",joinPoint.getTarget().getClass().getSimpleName(),
                    sig.getName(), Arrays.toString(joinPoint.getArgs()),(finish-start));
        }
    }
}

```

- ProceedingJoinPoing  = 프록시 대상 객체의 메소드 호출시 사용 <br>
    - proceed() : 메소드 사용해 대상 객체의 메소드 호출
    - getSignature() : 대상 객체의 메소드 이름 + 파라미터


<br>

➕ @Aspect 어노테이션 붙인 클래스를 공통 기능으로 사용하기 위해선 @EnableAspectJAutoProxy 를 설정클래스에 붙여야 함



<br>

```
  Calculator cal = ctx.getBean("calculator", Calculator.class);
        long fiveFact = cal.factorial(5);
        System.out.println("cal.factorial(5) ="+fiveFact);
        System.out.println(cal.getClass().getName());
        ctx.close();

```

1️⃣ AOP 적용

```
RecCalculator.factorial([5]) 실행시간 : 49900 ns
cal.factorial(5) =120
com.sun.proxy.$Proxy20  // 스프링이 생성한 프록시 타입 객체가 리턴
```


2️⃣ AOP 적용 X
```
cal.factorial(5) =120
chap07.RecCalculator 
```
<BR><bR>

### 프록시 생성 방식

: 스프링 AOP는 프록시 객체를 생성할때 실제 생성할 빈 객체가 인터페이스를 상속하면 **프록시 역시 인터페이스를 이용해서 생성**함

<Br>

```
// 설정 클래스: AOP 적용시 RecCalculator가 상속받은 Calculator 인터페이스를 이용해 프록시 생성

 @Bean
    public Calculator calculator(){
        return new RecCalculator();
    }

// 자바 코드 : "calculator" 빈의 실제 타입은 Calculator를 상속하 프록시 타입
// RecCalculator로 타입 변환 불가능 

RecCalculator cal = ctx.getBean("calculator",RecCalculator.class);

```


→ 빈의 실제 타입이 RecCalculator라도 "calculator" 이름에 해당하는 빈 객체의 타입은 Calculator 인터페이스르 상속받은 프록시 타입


<br>

📌 빈 객체가 인터페이스를 상속하는데, 클래스를 이용해서 프록시를 생성하고 싶다면?
    
    @Configuration
    @EnableAspectJAutoProxy(proxyTargetClass = true)
    public class AppCtx {
        ...
    }
    → 자바 클래스를 상속받아 프록시 생성함

    RecCalculator cal = ctx.getBean("calculator",RecCalculator.class);
    →  가능


<br>

#### 🔎 execution 명시자 표현식

: Advice 적용할 메소드 지정할 때 사용

![image](https://user-images.githubusercontent.com/81572478/179714280-9493dc46-f664-41e4-8968-56605e110be4.png)

<br>

#### 🔎 Advice 적용 순서

: 한 Pointcut에 여러 Advice 적용 가능, 어떤 Aspect가 먼저 적용될지는 스프링 프레임워크/자바 버전에 따라 달라짐

    ➡️ 적용 순서가 중요할때 직접 순서 지정 : Aspect 클래스에 @Order 어노테이션 적용

    @Aspect 
    @Order(2) // 두번째로 실행 되는 aspect
    public class ExeTimeAspect {
        @Pointcut("execution(public * chap07..*(..))") 
        private void publicTarget(){
        }
        ...

    }


<br>

#### 🔎 @Around의 Pointcut 설정 및 @Pointcut 재사용

- @Around 어노테이션에 execution 명시자를 직접 지정 가능 

    = 공통기능 메소드를 적용할 클래스 지정


- 같은 Pointcut을 여러 Advice가 함께 사용하면 공통 Pointcut을 재사용 가능
```
@Around("publicTarget()") // Pointcut 재사용
    public Object measure(ProceedingJoinPoint joinPoint) throws Throwable{
        long start = System.nanoTime(); // 기능 수행
        try{
           Object result = joinPoint.proceed(); // 대상 객체의 메소드 호출
            return result;
        }finally {
```

→ 다른 패키지/클래스에 있다면 패키지/클래스명 입력

<br>




