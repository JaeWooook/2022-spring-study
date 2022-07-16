## Chapter7 AOP 프로그래밍
> AOP기능은 Spring-aop 모듈이 제공한다.
> Spring-context모듈을 의존 대상에 추가하면 spring-aop 모듈도 함께 의존대상에 포함된다.
> build.gradle , pom.xml에 의존대상 추가하는 것을 말한다.

### 프록시와 AOP
> 기존 코드를 수정하지 않고 코드의 중복도 피할 수 있는 방법을 위한것이 프록시 객체이다.
```java
package chap07;

public interface Calculator {

    public long factorial(long num);

}
```
```java
package chap07;

public class ImpeCalculator implements Calculator {

    @Override
    public long factorial(long num) {
        long result = 1;
        for (long i = 1; i <= num; i++) {
            result *= i;
        }
        return result;
    }

}
```
```java
package chap07;

public class RecCalculator implements Calculator {

    @Override
    public long factorial(long num) {
        if (num == 0)
            return 1;
        else
            return num * factorial(num - 1);
    }

}
```
```java
package chap07;

public class ExeTimeCalculator implements Calculator {

    private Calculator delegate;

    public ExeTimeCalculator(Calculator delegate) {
        this.delegate = delegate;
    }

    @Override
    public long factorial(long num) {
        long start = System.nanoTime();
        long result = delegate.factorial(num);
        long end = System.nanoTime();
        System.out.printf("%s.factorial(%d) 실행 시간 = %d\n",
                delegate.getClass().getSimpleName(),
                num, (end - start));
        return result;
    }

}

```
- ExeTimeCalculator클래스는 Calculator 인터페이스를 구현하고 있다.
- 이 클래스는 생성자를 통해 다른 Calculator 객체를 전달 받아 delegate 필드에 할당한다.
- delegate필드를 이용해서 주입 받은 객체의 factorial 메서드를 사용한다.
- 해당 메서드를 실행하기 이전에 시간과 종료된 시간을 계산하여 총 실행하는데 걸린시간을 계산한다.

```java
package main;

import chap07.ImpeCalculator;
import chap07.RecCalculator;
import chap07.ExeTimeCalculator;

public class MainProxy {

    public static void main(String[] args) {
        ExeTimeCalculator ttCal1 = new ExeTimeCalculator(new ImpeCalculator());
        System.out.println(ttCal1.factorial(20));

        ExeTimeCalculator ttCal2 = new ExeTimeCalculator(new RecCalculator());
        System.out.println(ttCal2.factorial(20));
    }
}
```
- 출력 해보면 각각의 객체를 주입해서 각자 객체의 메서드를 실행하여 해당 메서드의 실행 시간을 구할 수 있다.

> 위 결과를 통한 방법으로 알 수 있는점
- 기존 코드를 변경하지 않고 실행 시간을 출력할 수 있다. ImpeCalculator 클래스나 RecCalculator 클래스의 코드 변경 없이 이 두 클래스의 factorial() 메서드 실행 시간을 출력할 수 있게 되었다.
- RecCalculator의 경우 실행 시간을 구하는 코드의 중복을 제거했다.
- 나노초 대신 밀리초를 이용해서 시간을 구하고싶다면 ExeTimeCalculator클래스만 변경하면 된다.

> 이렇게 할 수 있는 이유
- ExeTimeCalculator클래스를 factorial() 기능 자체를 직접 구현하기 보다는 다른 객체에 factorial()의 실행을 위임했기 때문이다.
- 계산 기능 외에 시간을 측정하는 기능만을 해당 클래스에서 실행한다.

> __핵심 기능의 실행은 다른 객체에 위임하고 부가적인 기능을 제공하는 객체를__ "프록시"라고 한다.
> 실제 핵심 기능 실행하는 객체는 "대상 객체"라고 한다.
> ex) ExeTimeCalculator가 프록시이고, ImpeCalculator객체가 프록시의 대상 객체가 된다.

> 엄밀히 따지면 위 코드는 프록시보다 데코레이터 객체에 가깝다.
> 프록시는 접근 제어 관점에 초점이 맞춰져있다면, 데코레이터는 기능 추가와 확장에 초점이 맞춰져 있기 때문이다.

> ImepeCalculator와 RecCalculator는 팩토리얼을 구한다는 핵심 기능 구현에 집중하고
> 프록시인 ExeTimeCalculator는 실행 시간 측정이라는 공통 기능 구현에 집중한다.
> __이렇게 공통 기능 구현과 핵심 기능 구현을 분리하는 것이 AOP의 핵심이다.__

#### AOP
> Aspect Oriendted Programming의 약자로, 여러 객체에 공통을 적용할 수 있는 기능을 분리해서
> __재사용성을 높여주는 프로그래밍 기법이다.__ AOP는 핵심 기능과 공통 기능의 구현을 분리함으로써 핵심 기능을 구현한 코드의 수정 없이 공통 기능을 적용할 수 있게 만들어준다.

> AOP의 기본 개념은 핵심 기능에 공통 기능을 삽입하는 것이다.
> 핵심 기능에 공통 기능을 삽입하는 방법 3가지
- 컴파일 시점에 코드에 공통 기능을 삽입하는 방법
    - AOP 개발도구가 소스 코드를 컴파일 하기 전에 공통 구현 코드를 소스에 삽입하는 방식으로 동작한다.
- 클래스 로딩 시점에 바이트 코드에 공통 기능을 삽입하는 방법
    - 클래스를 로딩할 때 바이트 코드에 공통 기능을 클래스에 삽입하는 방식으로 동작한다.
- __런타임에 프록시 객체를 생성해서 공통 기능을 삽입하는 방법__
    - 스프링에서 제공하는 AOP방식 중간에 프록시  객체를 생성한다. 실제 객체의 기능을 실행하기 전 후에 공통 기능을 호출한다.

![image](https://user-images.githubusercontent.com/55322459/179344545-63b8f269-8d80-488a-a437-b9bb4a4beadf.png)

스프링 AOP는 프록시 객체를 자동으로 만들어준다. 따라서, ExeTimeCalculator클래스 처럼 상위 타입의 인터페이스를 상속받은
프록시 클래스를 직접 구현할 필요가 없다. 단지 공통 기능을 구현한 클래스만 알맞게 구현하면 된다.

#### AOP에서 공통 기능을 Aspect라고 한다. 이외의 용어 정리
> Advice : 언제 공통 관심 기능을 핵심 로직에 적용할 지를 정의하고 있다.
> 예를 들면 '메서드'를 호출하기 전(언제)에 트랜잭션 시작(공통 기능) 기능을 적용한다는 것을 정의한다.

> Joinpoint : Advice를 적용 가능한 지점을 의미한다. 메서드 호출, 필드 값 변경 등이 Joinpoint에 해당한다.
> 스프링은 프록시를 이용해서 AOP를 구현하기 때문에 메서드 호출에 대한 Joinpoint만 지원한다.

> Pointcut : Joinpoint의 부분 집합으로서 실제 Advice가 적용되는 Joinpoint를 나타낸다. 스프링에서는 정규 표현식이나 AspectJ의 문법을 이용하여
> Pointcut을 정의할 수 있다.

> Weaving : Advice를 핵심 로직 코드에 적용하는 것을 weaving이라고 한다.

> Aspect : 여러 객체에 공통으로 적용되는 기능을 Aspect라고 한다. 트랜잭션이나 보안 등이 Aspect의 좋은 예이다.

#### Advice의 종류
- 스프링은 프록시를 이용해서 메서드 호출 시점에 Aspect를 적용하기 때문에 구현 가능한 Advice의 종류는 아래와 같다.

> Before Advice : 대상 객체의 메서드 호출 전에 공통 기능을 실행한다.
>
>  After Return Advice : 대상 객체의 메서드가 익셉션 없이 실행된 이후에 공통 기능을 실행한다.
>
> After Throwing Advice : 대상 객체의 메서드를 실행하는 도중 익셉션이 발생한 경우에 공통 기능을 실행한다.
>
> After Advice : 익셉션 발생 여부에 상관없이 대상 객체의 메서드 실행 후 공통 기능을 실행한다.(try-catch-finally) finally와 비슷하다.
>
> Around Advice : 대상 객체의 메서드 실행 전, 후 또는 익셉션 발생 시점에 공통 기능을 실행하는데 사용된다.

### 스프링 AOP 구현
> 스프링 AOP를 이용해서 공통 기능을 구현하고, 적용하는 방법은 단순하다.
- Aspect로 사용할 클래스에 @Aspect 애노테이션을 붙인다.
- @Pointcut 애노테이션으로 공통 기능을 적용할 Pointcut을 정의한다.
- 공통 기능을 구현한 메서드에 @Around 애노테이션을 적용한다.

#### @Aspect, @Pointcut, @Around를 이용한 AOP 구현
```java
package aspect;

import java.util.Arrays;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.Signature;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.core.annotation.Order;

@Aspect
// @Order(1)
public class ExeTimeAspect {

    @Pointcut("execution(public * chap07..*(..))")
    private void publicTarget() {
    }

    @Around("publicTarget()")
    public Object measure(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.nanoTime();
        try {
            Object result = joinPoint.proceed();
            return result;
        } finally {
            long finish = System.nanoTime();
            Signature sig = joinPoint.getSignature();
            System.out.printf("%s.%s(%s) 실행 시간 : %d ns\n",
                    joinPoint.getTarget().getClass().getSimpleName(),
                    sig.getName(), Arrays.toString(joinPoint.getArgs()),
                    (finish - start));
        }
    }
}
```
- @Aspect 애노테이션을 적용한 클래스는 Adivce와 Pointcut을 함께 제공한다.
- @Pointcut은 공통 기능을 적용할 대상을 설정한다.
    - execution 명시자는 chap07 패키지와 그 하위 패키지에 위치한 타입의 public 메서드를 Pointcut으로 설정한다.
- @Around 애노테이션 Around Advice를 설정한다. @Around 애노테이션의 값이 "publicTarget()"인데, 이는 publicTarget() 메서드에 정의한 Pointcut에 공통 기능을 적용한다는 것을 의미한다.
    - publicTarget() 메서드는 chap07 패키지와 그 하위 패키지에 위치한 public 메서드를 Pointcut으로 설정하고 있으므로, chap07 패키지나 그 하위 패키지에 속한 빈 객체의 public 메서드에 @Around가 붙은 measure()메서드에 적용된다.
- measure() 메서드의 ProceedingJoinPoint타입 파라미터는 프록시 대상 객체의 메서드를 호출할 때 사용한다.
- proceed() 메서드를 사용해서 실제 대상 객체(주입한 타입의 클래스에서 만들어진 객체)의 메서드를 호출한다.
- jointPoint.proceed()메서드를 실행하기 이전과 이후에 공통기능을 위한 코드를 위치 시킨다.

```java
package config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

import aspect.ExeTimeAspect;
import chap07.Calculator;
import chap07.RecCalculator;

@Configuration
@EnableAspectJAutoProxy
public class AppCtx {
    @Bean
    public ExeTimeAspect exeTimeAspect() {
        return new ExeTimeAspect();
    }

    @Bean
    public Calculator calculator() {
        return new RecCalculator();
    }

}
```
- 공통 기능을 적용하는데 필요한 스프링 설정 클래스 AppCtx
- @Aspect 애노테이션을 붙인 클래스를 공통 기능으로 적용하려면 @EnableAsepctJAutoProxy 애노테이션을 설정 클래스에 붙여야 한다.
- 이 애노테이션을 추가하면 스프링은 @Aspect 애노테이션이 붙은 빈 객체를 찾아서 빈 객체의 @Pointcut 설정과 @Around 설정을 사용한다.

```java
package main;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

import chap07.Calculator;
import config.AppCtx;

public class MainAspect {

    public static void main(String[] args) {
        AnnotationConfigApplicationContext ctx =
                new AnnotationConfigApplicationContext(AppCtx.class);

        Calculator cal = ctx.getBean("calculator", Calculator.class);
        long fiveFact = cal.factorial(5);
        System.out.println("cal.factorial(5) = " + fiveFact);
        System.out.println(cal.getClass().getName());
        ctx.close();
    }

}
```
![image](https://user-images.githubusercontent.com/55322459/179345633-56442ab8-dec0-4de1-93d2-5f4b1c3200f3.png)
- Calculator.class의 빈을 생성해도 @EnableAspectJAutoProxy 애노테이션이 추가되어있어 @Aspect 애노테이션이 붙은 빈 객체를 찾아서 빈 객체의 @Pointcut설정과 @Around설정을 사용한다.
- @Around 애노테이션은 Pointcut으로 publicTarget()메서드를 설정했기 때문에 @Pointcut으로 chap07패키지나 그 하위 패키지에 속한 빈 객체의 public 메서드를 설정한다.
- Calculaotr 타입이 chap07 패키지에 속하므로 calculator 빈에 ExeTimeAspect 클래스에 정의한 공통 기능인 measure()를 적용한다.

####ProceedingJoinPoint의 메서드
Around Advice에서 사용할 공통 기능 메서드는 대부분 파라미터로 전달받은 ProceedingJoinPoint의 proceed() 메서드만 호출하면 된다.
```java
 @Around("publicTarget()")
public Object measure(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.nanoTime();
        try {
        Object result = joinPoint.proceed();
        return result;
        } finally {
        long finish = System.nanoTime();
        Signature sig = joinPoint.getSignature();
        System.out.printf("%s.%s(%s) 실행 시간 : %d ns\n",
        joinPoint.getTarget().getClass().getSimpleName(),
        sig.getName(), Arrays.toString(joinPoint.getArgs()),
        (finish - start));
        }
        }
```
- 호출되는 대상 객체에 대한 정보, 실행되는 메서드에 대한 정보, 메서드를 호출할 때 전달된 인자에 대한 정보가 필요할 때가 있다.
- ProceedingJoinPoint 인터페이스는 다음 메서드를 제공한다.
    - Signature getSignature() : 호출되는 메서드에 대한 정보를 구한다.
    - Object getTarget() : 대상 객체를 구한다.
    - Object[] getArgs() : 파라미터 목록을 구한다.
- org.aspectj.lang.Signature 인터페이스는 다음 메서드를 제공한다. 각 메서드는 호출되는 메서드의 정보를 제공한다.
    - String getName() : 호출되는 메서드의 이름을 구한다.
    - String toLongString() : 호출되는 메서드를 완전하게 표현한 문장을 구한다(메서드의 리턴 타입, 파라미터 타입이 모두 표시된다.)
    - String toShortString() : 호출되는 메서드를 축약해서 표현한 문장을 구한다(기본 구현은 메서드의 이름만을 구한다.)

### 프록시 생성 방식
```java
//빈생성 부분 수정전
Calculator cal = ctx.getBean("calculator", Calculator.class);

//빈생성 부분 수정후
        RecCalculator cal = ctx.getBean("calculator", RecCalculator.class);

//AppCtx파일 설정
@Bean
public Calculator calculator() {
        return new RecCalculator();
        }
```
- MainAspect 클래스를 실행에 문제가 없을 것이라 생각된다 어차피 @Bean생성과정에서 Recalculator클래스 타입을 사용하기 때문에
- 하지만 __getBean() 메서드에 사용한 타입이 RecCalculator인데 반해 실제 타입은 $Proxy17이라는 에러메시지 발생한다.__
- $Proxy17은 스프링이 런타임에 생성한 프록시 객체의 클래스이름이다.
- 즉, $Proxy17 클래스는 RecCalculator클래스가 상속받는 Calculator 인터페이스를 상속받게 된다.

![image](https://user-images.githubusercontent.com/55322459/179354229-970d55be-1dd4-41a0-a2af-c514ebf391bb.png)
- 그림으로 설명하자면 RealSubject타입으로 getBean메서드를 이용해 빈 객체를 초기화하고 생성한다.
- 그렇게되면 Subject인터페이스 상속을 받게된 Proxy클래스는 RealSubject클래스의 타입으로 변환할 수 없다.

```java
//설정 클래스
//AOP 적용시 RecCalculator가 상속받은 Calculator 인터페이스를 이용해서 프록시 생성
@Bean
public Calculator calculator() {
        return new RecCalculator();
        }

//자바 코드
//calculator 빈의 실제 타입은 Calculator를 상속한 프록시 타입으로, ReCalculaotr로 타입 변환을 할 수 없다.
        RecCalculator cal = ctx.getBean("calculator", RecCalculator.class);
```

> 빈 객체가 인터페이스를 상속할 때 인터페이스가 아닌 클래스를 이용해서 프록시를 생성하고싶을때는 아래와 같이하면된다.

```java
//설정 클래스
@Configuration
@EnableAspectJAutoProxy(proxyTargetClass = true)
public class AppCtx {
    ...
}
```
- @EnableAspectJAutoProxy 애노테이션의 proxyTargetClass 속성을 true로 지정하면 __인터페이스가 아닌 자바 클래스를 상속받아 프록시를 생성한다.__
- 스프링이 프록시를 이용해 생성한 빈 객체를 구할 때 다음과 같이 getBean()메서드에 실제 클래스를 이용해서 빈 객체를 구할수 있게된다.
- 위 처럼 설정을 해주면, calculator 프록시의 실제 타입은 RecCalculator를 상속받았으므로 RecCalculator로 타입 변환이 가능하다.

#### execution 명시자 표현식
> execution 명시자는 Advice를 적용할 메서드를 지정할 때 사용한다. 기본형식은 아래와 같다.

> execution(수식어패턴? 리턴타입패턴 클래스이름패턴? 메서드이름패턴(파라미터패턴))
- 수식어 패턴은 생략 가능하며, public, protected 등이 온다.
- 리턴타입패턴은 리턴타입을 명시한다.
- 각 패턴은 *를 이용해서 모든 값을 표현하고, ..을 이용해서 0개 이상이라는 의미를 나타낼 수 있다.

#### Advice 적용 순서
> 한 Pointcut에 여러 Advice를 적용할 수 있다.

```java
package aspect;

import java.util.HashMap;
import java.util.Map;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;

@Aspect
//@Order(2)
public class CacheAspect {

    private Map<Long, Object> cache = new HashMap<>();

    @Pointcut("execution(public * chap07..*(long))")
    public void cacheTarget() {
    }

    @Around("cacheTarget()")
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
        Long num = (Long) joinPoint.getArgs()[0];
        if (cache.containsKey(num)) {
            System.out.printf("CacheAspect: Cache에서 구함[%d]\n", num);
            return cache.get(num);
        }

        Object result = joinPoint.proceed();
        cache.put(num, result);
        System.out.printf("CacheAspect: Cache에 추가[%d]\n", num);
        return result;
    }

}
```
- 첫 번째 인자를 Long타입으로 구한다.
- 구한 키값을 cache에 존재하면 키에 해당하는 값을 구해서 리턴한다.
- 구한 키값이 cache에 존재하지 않으면 프록시 대상 객체를 실행한다.
- 프록시 대상 객체를 실행한 결과를 cache에 추가한다.
- 프록시 대상 객체의 실행 결과를 리턴한다.

@Around값으로 cacheTarget()메서드를 지정했다. @Pointcut설정은 첫 번째 인자가 long인 메서드를 대상으로 한다.
따라서 execute() 메서드는 앞서 작성한 Calculator의 factorial(long) 메서드에 적용된다.

새로운 Aspect를 구현했으므로 스프링 설정 클래스에서 2개의 Aspect를 추가한다.
```java
package config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

import aspect.CacheAspect;
import aspect.ExeTimeAspect;
import chap07.Calculator;
import chap07.RecCalculator;

@Configuration
@EnableAspectJAutoProxy
public class AppCtxWithCache {

    @Bean
    public CacheAspect cacheAspect() {
        return new CacheAspect();
    }

    @Bean
    public ExeTimeAspect exeTimeAspect() {
        return new ExeTimeAspect();
    }

    @Bean
    public Calculator calculator() {
        return new RecCalculator();
    }

}
```
- ExeTimeAspect는 앞서 구현한 시간 측정 Aspect이다.
- 두 Aspect에서 설정한 Pointcut은 모두 Calculator 타입의 factorial()메서드에 적용된다.

```java
package main;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

import chap07.Calculator;
import config.AppCtxWithCache;

public class MainAspectWithCache {

    public static void main(String[] args) {
        AnnotationConfigApplicationContext ctx =
                new AnnotationConfigApplicationContext(AppCtxWithCache.class);

        Calculator cal = ctx.getBean("calculator", Calculator.class);
        cal.factorial(7);
        cal.factorial(7);
        cal.factorial(5);
        cal.factorial(5);
        ctx.close();
    }

}
```
- cal.factorial(7)을 첫번째 실행할때는 ExeTimeAspect와 CacheAspect가 모두 적용되었다.
- 2번째 cal.factorial(7)에서는 CacheAspect만 적용되었다.

> Advice의 적용 순서
>
> CacheAspect프록시 -> ExeTimeAspect프록시 -> 실제 대상 객체

- ctx.getBean("calculator", Calculator.class); 를 이용해서 구한 빈은 실제로 CacheAspect 프록시 객체이다.
- CacheAspect프록시 객체의 대상 객체는 ExeTimeAspect의 프록시 객체이다.
- 그리고 ExeTimeAspect 프록시의 대상 객체가 실제 대상 객체이다.

![image](https://user-images.githubusercontent.com/55322459/179356996-57c40ee9-8f9c-4461-8f92-a0c4fd61cd50.png)
- CacheAspect는 cache맵에 데이터가 존재하지않으면 joinPoint.proceed()를 실행해서 대상을 실행한다.
- 그 대상이 ExeTimeAspect이므로 ExeTimeAspect의 measure()메서드가 실행된다.
- ExeTimeAspect는 실제 대상 객체를 실행하고 실행 시간을 출력한다. ExeTimeAspect실행이 끝나면 CacheAspect는 cache맵에 데이터를 넣고 cache추가 메시지를 출력한다.
- 2번째 cal.factorial(7)이 실행되면 CacheAspect에서 cache맵에 저장된 값을 if문에서 걸러주고, 리턴한다.

> 어떤 Aspect가 먼저 적용될지는 스프링 프레임워크나 자바 버전에 따라 달라질 수 있기 때문에 적용 순서가 중요하다면 직접 순서를 지정해야한다.

이럴때 사용하는 애노테이션이 @Order 애노테이션이다.

> @Order애노테이션 값이 작으면 먼저 적용하고, 크면 나중에 적용한다.
```java
@Aspect
@Order(1)
public class ExeTimeAspect {
    ...
}

@Aspect
@Order(2)
public class CacheAspect {
    ...
}
```
- 위와같이 Asepct에서 생성한 Order순으로 ExeTimeAspect -> CacheAspect ->  실제 대상 객체가 적용된다.

이러한 순으로 실행하면 2번째 cal.factorial(7)을 실행해도 두가지의 Asepct가 모두 실행됨을 알 수 있다.
CacheAspect에서 걸러주지 못하기 때문이다.

#### @Around의 Pointcut 설정과 @Pointcut 재사용
> Pointcut을 여러 Advice가 함께 사용한다면 공통 Pointcut을 재사용할 수 있다.
```java
@Aspect
// @Order(1)
public class ExeTimeAspect {

    @Pointcut("execution(public * chap07..*(..))")
//    private void publicTarget() {
    public void publicTarget() {
    }

    @Around("publicTarget()")
    public Object measure(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.nanoTime();
        try {
            Object result = joinPoint.proceed();
            return result;
        } finally {
            ...

            @Aspect
//@Order(2)
            public class CacheAspect {

                private Map<Long, Object> cache = new HashMap<>();

                @Pointcut("execution(public * chap07..*(long))")
                public void cacheTarget() {
                }

                //	@Around("cacheTarget()")
//    @Around("aspect.ExeTimeAspect.publicTarget()")
                @Around("ExeTimeAspect.publicTarget()")
                public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
                    Long num = (Long) joinPoint.getArgs()[0];
                    if (cache.containsKey(num)) {
                        System.out.printf("CacheAspect: Cache에서 구함[%d]\n", num);
                        return cache.get(num);
                    }
        ...
```
- 첫번째 ExeTimeAspect에서 보면, @Around("publicTarget()")라고 되어있어 publicTarget() 메서드에 설정한 Pointcut을 사용한다.
- publicTarget()메서드를 public으로 변경하면 다른 클래스에 위치한 @Around 애노테이션에서마 publicTarget()메서드의 Pointcut을 사용할 수 있다.
- @Around("aspect.ExeTimeAspect.publicTarget()")를 이용해서 ExeTimeAspect클래스에 위치한 Pointcut을 이용한다.
- 같은 패키지에 위치함으로 패키지 이름 없이 간단한 클래스 이름으로 설정이 가능하다. @Around("ExeTimeAspect.publicTarget()")

```java
package aspect2;

import org.aspectj.lang.annotation.Pointcut;

public class CommonPointcut {

    @Pointcut("execution(public * chap07..*(..))")
    public void commonTarget() {
    }

}

package aspect2;

        import java.util.HashMap;
        import java.util.Map;

        import org.aspectj.lang.ProceedingJoinPoint;
        import org.aspectj.lang.annotation.Around;
        import org.aspectj.lang.annotation.Aspect;

@Aspect
public class CacheAspect2 {

    private Map<Long, Object> cache = new HashMap<>();

    @Around("aspect2.CommonPointcut.commonTarget()")
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
        Long num = (Long) joinPoint.getArgs()[0];
        if (cache.containsKey(num)) {
            System.out.printf("CacheAspect2: Cache에서 구함[%d]\n", num);
            return cache.get(num);
        }
        ...

package aspect2;

import java.util.Arrays;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.Signature;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;

        @Aspect
        public class ExeTimeAspect2 {

            @Around("aspect2.CommonPointcut.commonTarget()")
            public Object measure(ProceedingJoinPoint joinPoint) throws Throwable {
                long start = System.nanoTime();
                try {
                    Object result = joinPoint.proceed();
                    return result;
                } finally {
                    ...
                }
                ...
```
- CommonPointcut클래스에 공통으로 사용할 Pointcut을 설정한다.
- 그러면 CacheAspect, ExeTimeAspect에서 사용할 Pointcut을 새로 설정할 필요없다.
- 하나의 Pointcut에서 전부 관리해준다.
- 또한, 포인트컷을 위한 __CommonPointcut클래스는 빈으로 등록할 필요없이__ @Around 애노테이션에서 해당 클래스에 접근 가능하면 해당 Around를 사용할 수 있다.