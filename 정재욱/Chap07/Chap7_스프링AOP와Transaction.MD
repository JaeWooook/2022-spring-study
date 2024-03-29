## 스프링 AOP와 @Transcational 소개
### AOP란?
> AOP는 어플리케이션의 핵심적인 기능에서 부가적인 기능을 분리해서 애스펙트라는 독특한 모듈로 만들어서 설계하고 개발하는 방법이다.

![image](https://user-images.githubusercontent.com/55322459/179388392-c67dd06a-c3bd-41e3-aef0-57dfc48c0744.png)
- 핵심 기능의 공통 메서드를 추출해서 새로운 부가기능을 만드는 것이 AOP이다.

### Spring AOP 용어 정리
> 타겟 (Target)
- 부가 기능을 부여할 대상을 얘기한다.
- 여기선 핵심 기능을 담당하는 getBoards, getUsers, Service등을 얘기한다.

> 애스펙트 (Aspect)
- 객체지향 모듈을 오브젝트라 부르는것과 비슷하게 부가기능 모듈을 애스펙트라고 부른다.
- __핵심기능에 부가되어 의미를 갖는__ 특별한 모듈이라 생각하면된다.
- 애스펙트는 부가될 기능을 정의한 __어드바이스__ 와 어드바이스를 어디에 적용할지를 결정하는 __포인트컷__ 을 함께 갖고 있다.

> 포인트컷 (PointCut)
- 부가기능이 적용될 대상(메서드)를 선정하는 방법을 얘기한다.
- 즉, 어드바이스를 적용할 조인포인트를 선별하는 기능을 정의하는 모듈을 얘기한다.

> 조인포인트 (JoinPoint)
- 어드바이스가 적용될 수 있는 위치를 얘기한다.
- Spring에서는 메소드 조인포인트만 제공하고 있다.
- 따라서, Spring프레임워크 내에서 조인포인트라 하면 메소드를 가리킨다고 생각해도 된다.

> 프록시 (Proxy)
- 타겟을 감싸서 타겟의 요청을 대신 받아주는 랩핑(Wrapping)오브젝트 이다.
- 호출자(클라이언트)에서 타겟을 호출하게 되면 타겟이 아닌 타겟을 감싸고 있는 프록시가 호출되어, 타겟 메서드 실행전에 선처리, 타겟 메서드 실행 후, 후처리를 실행시키도록 구성되어있다.

> 인트로덕션 (Introduction)
- 타겟 클래스에 코드 변경없이 신규 메서드나 멤버변수를 추가하는 기능을 얘기한다.

> 위빙 (Weaving)
- 지정된 객체에 애스펙트를 적용해서 새로운 프록시 객체를 생성하는 과정을 얘기한다.
- ex) A라는 객체에 트랜잭션 애스팩트가 지정되어 있다면, A라는 객체가 실행되기전 커넥션을 오픈하고 실행이 끝나면 커넥션을 종료하는 기능이 추가된 프록시 객체가 생성되고, 이 프록시 객체가 앞으로 A객체가 호출되는 시점에서 사용된다.
- 이때의 프록시 객체가 생성되는 과정을 위빙이라고 생각하면 된다.
- Aspect가 target에 적용되는 전체적인 과정을 말한다. 즉, Pointcut으로 지정된 JoinPoint에 Advice가 적용되어 Target을 호출 시 AOP Proxy가 만들어지는 과정이다.

#### AOP를 이용한 트랜잭션 분리

> 트랜잭션을 적용하지 않은 코드
```java
public class FakeService {
    private final PlatformTransactionManager transactionManager;
    private final CrewDao crewDao;
    
    public FakeService(PlatformTransactionManager transactionManager, CrewDao crewDao) {
        this.transactionManager =  transactionManager;
        this.crewDao = crewDao;
    }
    
    public void addCrew(String str) {
        final TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());
        
        try{
            List<Crew> crews = crewDao.findAll();
            for (Crew crew : crews) {
                if(crew.isNotDup()) {
                    crewDao.addCrew(crew, str);
                }
            }
            
            transactionManager.commit(status);
        } catch (RuntimeException e) {
            transactionManager.rollback(status);
            throw e;
        }
    }
}
```
- 위의 코드는 비즈니스 로직을 수행하고 완료하면 트랜잭션을 커밋하고, 예외가 발생하면 롤백되는 코드이다.
- 트랜잭션 관련 코드가 매번 작성되기 때문에 코드의 중복이 일어난다.

> 트랜잭션을 사용하지 않은 Spring AOP적용 코드 1
```java
@Component
@Aspect
public class TransactionAspect {
    
    @Autowired
    private PlatformTransactionManager transactionManager;
    
    @Pointcut("execution(public void add*(..))")
    private void publicTarget() {
        
    }
    
    @Around("publicTarget()")
    public void transaction(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        final TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());
        
        try{
            proceedingJoinPoint.proceed();
            transactionManager.commit(status);
        } catch (RuntimeException e) {
            transactionManager.rollback(status);
            throw e;
        }
    }
}
```
- @Around 어노테이션을 이용해서 실제 대상이 되는 객체를 실행한 후 성공 여부에 따라 트랜잭션을 commit 또는 rollback해준다.
- @Pointcut을 이용해서 실제 대상이 되는 타겟을 지정해준다.

> 트랜잭션을 사용하지 않은 Spring AOP적용 코드 2(어노테이션화하기)
```java
@Component
@Aspect
public class TransactionAspect {
    
    @Autowired
    private PlatformTransactionManager transactionManager;
    
//    @Pointcut("execution(public void add*(..))")
//    private void publicTarget() {
//        
//    }
    
    @Around(@annotation(FakeTransactional))
    public void transaction(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        final TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());
        
        try{
            proceedingJoinPoint.proceed();
            transactionManager.commit(status);
        } catch (RuntimeException e) {
            transactionManager.rollback(status);
            throw e;
        }
    }
}
```

```java
@Target({ElementType.METHOD})
@Rentetion(RetentionPolicy.RUNTIME)
public @interface FakeTransactional {
}
```
```java
@Component
public class FakeServiceImpl implements FakeService {
    private final CrewDao crewDao;
    
    public FakeServiceImpl(CrewDao crewDao) {
        this.crewDao = crewDao;
    }
    
    @Override
    @FakeTransactional
    public void addCrew(String str) {
        List<Crew> crews = crewDao.findAll();
        for(Crew crew : crews) {
            if(crew.isNotDup()) {
                crewDao.addCrew(crew, str);
            }
        }
    }
}
```

> 트랜잭션을 적용하지 않은 코드
```java
public void addUsers(List<User> userList) {
	TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
	
	try {
		for (User user: userList) {
			if(isEmailNotDuplicated(user.getEmail())){
				userRepository.save(user);
			}
		}

		this.transactionManager.commit(status);
	} catch (Exception e) {
		this.transactionManager.rollback(status);
		throw e
	}
}
```
- 위의 코드는 여러 책임을 가질 뿐만 아니라 서로 성격도 다르고 주고받는 것도 없으므로 분리하는 것이 적합하다.
- Spring에서는 마치 트랜잭션 코드와 같은 부가 기능 코드가 존재하지 않는 것 처럼 보이기 위해 __해당 로직을 클래스 밖으로 빼내서 별도의 모듈로 만드는 AOP__ 를 고안 및 적용하게 되었다.
- 이를 적용한 트랜잭션 어노테이션(@Transactional)을 지원하게 되었다.

> 트랜잭션을 적용한 AOP방법의 코드
```java
@Service
@RequiredArgsConstructor
@Transactional
public class UserService {

    private final UserRepository userRepository;

    public void addUsers(List<User> userList) {
        for (User user : userList) {
            if (isEmailNotDuplicated(user.getEmail())) {
                userRepository.save(user);
            }
        }
    }
}
```
- 특별하게 트랜잭션을 코드로 commit하고 rollback하거나 하는 코드가 보이지 않아 코드가 간결하다.
- UserService라는 도메인에 맞게 유저를 추가하는 내용의 코드만 작성되어 있어 도메인의 기능에 충실하고 있다.

### Transaction과 @Transactional 어노테이션 소개
#### Transaction이란?
> 트랜잭션이란 DB의 쿼리를 날려서 데이터베이스를 create, insert, select, update, delete 등의 작업을 하는 가장 최소 단위를 트랜잭션이라고 한다.

#### Transaction에 이해 필요성
데이터베이스의 데이터를 수정하는 도중에 예외가 발생한다면 어떻게 해야 할까? DB의 데이터들은 수정이 되기 전의 상태로 다시 되돌아가져야 하고, 다시 수정 작업이 진행되어야 한다.
이렇듯 __여러 작업을 진행하다가 문제가 생겼을 경우 이전 상태로 롤백하기 위해 사용__ 되는것이 트랜잭션이다.

#### 트랜잭션의 필요성에 대한 입출금 예시
![image](https://user-images.githubusercontent.com/55322459/179398533-5d10ab96-224f-44a0-bba7-6482a13d19da.png)
- 병아리를 A 파도를 B라고 하였을 때, A가 B에게 입금을 하는 상황에서 A의 잔액은 10000원으로 update하고, B의 잔액을 60000원으로 update하는 과정에서 서버의 다운이나 데이터베이스의 장애가 발생하는것을 방지하고자 트랜잭션이 필요하다.
- 트랜잭션은 일부만 실행하고 실패하는 이러한 과정없이 하나로 묶어서 관리할 수 있도록 해준다.
- 예를 들어 잔액을 A의 잔액을 20000원 에서 10000원으로 차감 했지만, 서버의 장애로 B의 잔액을 50000원에서 60000원으로 update를 해주지 않았다면 하나의 트랜잭션에서 A의 잔액은 성공했지만 전체적으로 실패 했다고 하여 rollback처리한다.

### 트랜잭션 (Transaction)
#### 선언적 트랜잭션을 사용하는 예시

![image](https://user-images.githubusercontent.com/55322459/179398923-4d8bdc22-7851-4d6f-bf63-1ca6987f2e88.png)
- 트랜잭션 어노테이션은 메서드, 클래스, 인터페이스 등에 적용할 수 있다.
- 클래스 상단에 적용하는 어노테이션에서는 해당 클래스에 존재하는 모든 메서드에 어노테이션이 적용된다.
- 중첩되어 존재하는 경우에는
    - 1.클래스 메서드, 2.클래스 순으로 우선순위를 갖고 적용된다.
    - 1.인터페이스 메서드, 2.인터페이스 순으로 우선순위를 갖고 적용된다.
- 메서드를 시작하는 순간 트랜잭션이 시작되고, 메서드를 성공적으로 끝마치면 트랜잭션 커밋
- 메서드를 실행하는 중간에 문제가 발생하면 롤백하는 과정으로 진행된다.
- 하나의 작업을 수행하는 서비스 계층 메서드에 붙이는 것이 통상적이다.

#### 실제 스프링 프레임워크에 트랜잭션 옵션
![image](https://user-images.githubusercontent.com/55322459/179402147-bf8a7c2d-fe0a-417d-ba3d-c545b8474c2c.png)
- 트랜잭션 매니저 또한 속성으로 등록할 수 있다. 빈으로 등록된 특수한 트랜잭션 매니저를 속성으로 등록할 수 있다.

### @Transactional - propagation
> 트랜잭션을 시작하거나 기존 트랜잭션에 참여하는 방법을 결정하는 속성
>
> 트랜잭션 경계설정 방식의 장점은 여러 트랜잭션 적용 범위를 묶어서 커다란 트랜잭션 경계를 만들 수 있다는 점이다.
>
> 트랜 잭션 경계의 시작 지점에서 트랜잭션 전파 속성을 참조해서 해당 범위의 트랜잭션을 어떤 식으로 진행시킬지 결정할 수 있다.

#### 🔶 1. @Transactional(propagation=Propagation.REQUIRED)

>메서드1 -> 트랜잭션1 생성

![image](https://user-images.githubusercontent.com/55322459/179402112-875cb16e-58f6-4e34-b84b-eca924fa015d.png)

> 메서드2 -> 트랜잭션2 생성

![image](https://user-images.githubusercontent.com/55322459/179402090-0599c8d7-3a60-46df-ae9e-cd8638dd6aae.png)
- 진행중인 트랜잭션이 없을 경우 메서드1은 트랜잭션1로 시작되고, 중간에 메서드2가 추가되서 트랜잭션2가 되면 아래의 그림 처럼 트랜잭션1에 추가되어 하나의 트랜잭션으로 실행 되게 된다.
- 하나의 트랜잭션이 실패하면 트랜잭션1,2 모두 롤백되고, 2개의 트랜잭션이 모두 성공하면 2개의 트랜잭션 모두 commit된다.

__요약__ : 디폴트값으로 미리 시작된 트랜잭션이 있으면 참여하고 없으면 새로 시작한다.

#### 🔶 2. @Transactional(propagation=Propagation.SUPPORTS)

![image](https://user-images.githubusercontent.com/55322459/179402084-1cfe03d7-fe8c-4b17-bf93-caa9efeb20cb.png)
__요약__ : 이미 시작된 트랜잭션이 있으면 참여하고, 그렇지 않으면 트랜잭션 없이 진행한다. 트랜잭션이 없긴 하지만 해당 경계 안에서 Connection이나 하이버네이트 Session등을 공유할 수 있다.

#### 🔶 3. @Transactional(propagation=Propagation.MANDATORY)

![image](https://user-images.githubusercontent.com/55322459/179402043-a429f301-30f7-4d33-912f-6ef0863dc2ce.png)
- 진행중인 트랜잭션이 있으면 참여하고, 없으면 예외가 발생한다.

__요약__ : 혼자서는 트랜잭션을 시작할 수 없고, 메서드를 실행할 수도없다.
즉, 혼자서는 독립적으로 트랜잭션을 진행할 수 없다. 독립적으로 트랜잭션을 진행하면 안되는 경우 사용

#### 🔶 4. @Transactional(propagation=Propagation.REQUIRES_NEW)

![image](https://user-images.githubusercontent.com/55322459/179402015-a45b7451-d7cf-4d04-b475-d76b4ef07a60.png)
- 새로운 메서드1이 실행되면 트랜잭션1로 실행하고, 새로운 메서드2가 실행되면 트랜잭션1을 보류하고, 트랜잭션2를 실행한다.

__요약__ : 항상 새로운 트랜잭션을 시작한다. 이미 진행 중인 트랜잭션이 있으면 트랜잭션을 잠시 보류시킨다.

#### 🔶 5. @Transactional(propagation=Propagation.NOT_SUPPORTED)

![image](https://user-images.githubusercontent.com/55322459/179401980-459a0e9d-5272-42c2-a1ec-90f8b6f93008.png)
- 트랜잭션1이 실행되고 있다면 트랜잭션1을 보류하고 메서드2를 먼저 실행한다. 메서드2는 트랜잭션2가 되지 않는다.

__요약__ : 트랜잭션을 사용하지 않게 한다. 이미 진행 중인 트랜잭션이 있으면 보류시킨다.

#### 🔶 6. @Transactional(propagation=Propagation.NEVER)

![image](https://user-images.githubusercontent.com/55322459/179401967-3bd547dd-4c2f-46d6-96a0-dd17fe338f4f.png)
- 이미 실행되는 트랜잭션이 없다면 실행하려는 메서드를 실행하지만, 기존에 실행되는 트랜잭션이 존재한다면 예외를 발생 시킨다.

__요약__ : 트랜잭션을 사용하지 않도록 강제한다. 이미 진행 중인 트랜잭션도 존재하면 안된다. 예외를 발생 시킨다.

#### 🔶 7. @Transactional(propagation=Propagation.NESTED)

![image](https://user-images.githubusercontent.com/55322459/179401950-7e3cccdc-f08d-43ea-908c-0da6192254e0.png)
- 트랜잭션1을 실행하고 트랜잭션2가 실행되는경우 Required와 다르게 트랜잭션1이 실행되다가 문제를 일으키면 트래잭션1과 2모두 롤백된다.
- 하지만, 트랜잭션2가 문제를 발생할 경우 트랜잭션1은 아무 문제없이 commit된다.

__요약__ : 이미 진행중인 트랜잭션이 있으면 중첩 트랜잭션을 시작한다. 중첩된 트랜잭션은 먼저 시작된 부모 트랜잭션의 커밋과 롤백에는 영향을 받지만 자신의 커밋과 롤백은 부모 트랜잭션에게 영향을 주지 않는다.

#### 트랜잭션 전파 정리
![image](https://user-images.githubusercontent.com/55322459/179401806-b1a16375-feb9-4b7e-8fa9-deca63619ea2.png)

### 트랜잭션 격리 수준 (Isolation)
![image](https://user-images.githubusercontent.com/55322459/179401928-c528b142-1e2d-489e-ba68-1422222252d6.png)
> 트랜잭션의 격리 수준 아래로 갈수록 격리수준은 올라가지만, 성능이 떨어진다.
> 데이터 정합성과 성능은 반비례한다.

#### 🟢 1. READ-UNCOMMITTED
> 커밋 전의 트랜잭션의 데이터 변경 내용을 다른 트랜잭션이 읽는 것을 허용

![image](https://user-images.githubusercontent.com/55322459/179401900-1a61d149-706e-44a4-9f99-30968392ee87.png)

> 더티리드(Dirty Read)라는 현상이 발생할 수 있다.
- 트랜잭션 A가 트랜잭션을 끝마치지 못하고 Rollback이 되었다면
- 트랜잭션 B는 무효가된 값을 읽고 데이터를 처리하기 때문에 문제가 생길 수 있다.

#### 🟢 2. READ-COMMITTED
> 커밋이 완료된 트랜잭션의 변경사항만 다른 트랜잭션에서 조회 가능
트랜 잭션이 이루어지는 동안 다른 사용자는 해당 데이터에 접근이 불가능하다.

![image](https://user-images.githubusercontent.com/55322459/179401851-0ddfaba5-5aaa-4464-a0a8-d41e49a306ae.png)

> 논리피터블리드(Non-Repeatable Read) 문제
- 트랜잭션 B에서 다른 시점에 조회 했을 때 데이터가 다른값이 될 경우에 대한 문제
- 데이터 불일치에 대한 문제

#### 🟢 3. REPEATABLE-READ
> 트랜잭션 범위 내에서 조회한 내용이 항상 동일함을 보장

![image](https://user-images.githubusercontent.com/55322459/179401829-b46c1b69-b04c-4f9d-b8ee-0b71380f61ca.png)
- Non-Repeatable Read문제를 방지한다.
- 하나의 트랜잭션이 읽은 로우를 다른 트랜잭션이 수정하는 것을 막아준다.
- 하지만 새로운 로우를 추가하는 것은 제한하지 않는다. 따라서 select로 조건에 맞는 로우를 전부 가져오는 경우 트랜잭션이 끝나기 전에 추가된 로우가 발견될 수 있다.

> 팬텀리드(Phantom Read)문제
- 다른 시점에 조회했을 경우 새로운 행이 추가되었거나, 삭제되는 문제

#### 🟢 4. SERIALIZABLE
> 한 트랜잭션에서 사용하는 데이터를 다른 트랜잭션에서 접근 불가
- 트랜잭션을 순차적으로 진행시켜 주기 때문에 여러 트랜잭션이 동시에 같은 테이블의 정보를 액세스 하지못한다.
- 가장 안전한 격리 수준이지만 가장 성능이 떨어지기 때문에 극단적인 안전한 작업이 아닐경우 자주 사용되지 않는다.

#### 트랜잭션 속성에 따른 문제점들 정리
![image](https://user-images.githubusercontent.com/55322459/179401764-212ca5e9-59d2-42f0-b421-1ce4e5bbb04a.png)
