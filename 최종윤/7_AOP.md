서비스에서 필요한 내용은 비즈니스 로직이라고 불리는 핵심기능만 수행하면 된다.

그이외에 시간을 재거나 권한을 체크한다거나 transaction을 거는 것은 모두 인프라 로직이라 불린다.

 인프라 로직 : 성능검사, 로깅, 성능검사, 권한체크

인프라 로직은 애플리케이션 전 영역에서 나타날 수 있다.

그러다보니 중복코드를 만들어낼 가능성 때문에 유지 보수가 힘들어진다.
비지니스 로직과 함께 있으면 비지니스 로직을 이해하기 어려워진다.

횡단으로 나타나기에 횡단 관심사라고도 부른다.
 

AOP

횡단 관심에 따라 프로그래밍한다.

각 언어마다 AOP의 구현체가 있다. 자바는 AspectJ를 사용한다.
AOP용어
부가기능은 횡단에 관심을 갖고 있기 때문에 이러한 질문을 만나게 된다.
- Target
- 어떤 대상에 부가기능을 부여할 것인가
- Advice
- 어떤 부가기능? before, afterRunning, afterThrowing, After, Around
- Join point
- 어디에 적용할 것인가? 메서드, 필드, 객체, 생성자 등 여러 상황에서 부가기능을 사용할 수 있으나 Spring AOP에는 메서드가 실행될 때만으로 한정하고 있다.
- Point cut
- 실제 advice가 적용될 지점, Spring AOP에서는 Advice가 적용될 메서드를 선정.
-  Weaving - Advice를 핵심 로직 코드에 끼워 넣는 것을 weaving 이라고 한다.

Aspect - 여러 객체에 공통으로 적용되는 기능을 Aspect라고 한다. 트랜잭션이나 보안 등이 Aspect의 예라고 할 수 있다.

- AOP를 스프링 빈으로 등록하기 위해서는 클래스에 @Component 애노테이션을 붙이는 방법(컴포넌트 스캔)과, SpringConfig에 빈으로 등록하는 방법이 있다. 
- 서비스, 리포지토리보다는 AOP가 특별하니까 SpringConfig에 직접 등록하는 것이 좋다.
- 그래야 SpringConfig를 보고 AOP가 등록되어서 쓰인다는 것을 바로 알 수 있기 때문이다.

AOP의 구현방법
*Spring AOP가 아니라 AOP의 전체적인 구현방법을 말함*

1. 컴파일

java를 calss로 컴파일 할때 해당하는 Asepct를 끼워넣는다.
2. 클래스 로드시

컴파일 완료되고 메모리상에 올릴때 그때 AOP를 적용하는 방식이다.
3. 프록시 패턴

스프링 AOP에서 사용하는 방식, 특정 타겟 Class를 부가기능을 제공하는 프록시로 감싸서 실행하는 방식이다.

AspectJ란 AOP를 자바에서 사용하기 위한 구현체이며 완전한 AOP 솔루션을 제공하는 것을 목표로 한다. 여러 구현체가 있는데 사실상 자바 표준이라고 한다.

Dynamic Proxy나 CGLib처럼 런타임 위빙 방식의 프록시 패턴으로 동작해 간접적인 방법으로 AOP를 구현하는 것이 아니라 

Compile Time Weaving, Load Time Weaving, Post-compile weaving(컴파일 후 위빙)을 지원하여 컴파일 시점이나 클래스파일이 로드되는 시점에 바이트코드를 조작하여 위빙을 해버린다.

package hello.hellospring.aop;
```
/*@Component*/
@Aspect
public class TimeTraceApp {
  @Around("execution(* hello.hellospring..*(..))")  - @Around에 적용된 메서드는 실행될 때 실제 객체를 생성하여 실행하는것이 아닌 프록시 객체를 생성후 , 
                        실제 실행을  실제 대상 객체에 위임한다.
  public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {   -  위빙을 한다는 것이 ProceedingJoingPoint joinPoint에  
                                                @Around에서 적용시킨 메서드의 객체를 전달받나? 
    long start = System.currentTimeMillis();

    System.out.println("START: " + joinPoint.toString());

    try {
      return joinPoint.proceed();
    } finally {
      long finish = System.currentTimeMillis();
      long timeMs = finish - start;

      System.out.println("END: " + joinPoint.toString() + " " + timeMs + "ms");
    }
  }
}
```
Spring의 AOP도 포인트컷 표현식 사용시 AspectJ의 AspectHExpressionPointcut를 차용해서 사용할만큼 매우성숙하고 발전한 AOP 기술이다.
