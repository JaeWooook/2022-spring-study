## Chapter3 스프링DI

### 의존이란?
> 한 클래스가 다른 클래스의 메서드를 실행할 때 이를 '의존'한다고 표현한다.

```java
package spring;

import java.time.LocalDateTime;

public class MemberRegisterService {
    private MemberDao memberDao = new MemberDao();

    public Long regist(RegisterRequest req) {
        Member member = memberDao.selectByEmail(req.getEmail());
        if (member != null) {
            throw new DuplicateMemberException("dup email " + req.getEmail());
        }
        Member newMember = new Member(
                req.getEmail(), req.getPassword(), req.getName(),
                LocalDateTime.now());
        memberDao.insert(newMember);
        return newMember.getId();
    }
}
```

 - 위와 같은 MemberRegisterService클래스에서 MemberDao클래스의 메서드인 selectByEmail등을 사용한다.
 - MemberRegisterService클래스가 MemberDao클래스에 의존한다고 할 수 있다.
 - 의존은 변경에 의해 영향을 받는 관계이다.
 - MemberDao클래스의 메서드가 변경되면 MemberRegisterService클래스에도 영향을 줄 수있다.
 - MemberRegisterService클래스에 의존하는 MemberDao객체를 생성하면, 이후에 MemberRegisterService객체를 생성할때 의존하는 MemberDao객체도 함께 생성될 수 있다.
 - 유지보수할 때 의존성에 대한 문제가 발생할 수 있다.

### DI를 통한 의존 처리
> DI = Dependency Injection = 의존 주입 : 의존하는 객체를 직접 생성하는 대신 의존 객체를 전달 받는 방식이다.

![image](https://user-images.githubusercontent.com/55322459/176185225-f7cfacc3-bb9a-4e93-9fa6-219447f479d4.png)

```java
package spring;

import java.time.LocalDateTime;

public class MemberRegisterService {
	private MemberDao memberDao;

	public MemberRegisterService(MemberDao memberDao) {
		this.memberDao = memberDao;
	}

	public Long regist(RegisterRequest req) {
		Member member = memberDao.selectByEmail(req.getEmail());
		if (member != null) {
			throw new DuplicateMemberException("dup email " + req.getEmail());
		}
		Member newMember = new Member(
				req.getEmail(), req.getPassword(), req.getName(), 
				LocalDateTime.now());
		memberDao.insert(newMember);
		return newMember.getId();
	}
}
```
 - 위의 코드와 같이 의존 객체를 직접 생성하지 않고, 생성자를 통해 의존 객체를 전달받았기 때문에, DI패턴을 사용한다고 할 수 있다.
 - DI를 사용하면 유지보수의 용이성을 가질 수 있다.

### DI와 의존 객체 변경의 유연함
> DI를 사용하면 A객체를 사용하는 클래스가 세 개여도 변경할 곳은 의존 주입 대상이 되는 객체를 생성하는 코드 한 곳 뿐이다.
> 
> 의존 객체를 직접 생성했던 방식을 사용하면 변경할 코드가 의존하는 객체를 생성하는 곳마다 변경해주어야 하는 불편함이 있다.

#### DI의 변경의 유연함 예시
회원정보에 캐시를 사용하기 위해, MemberDao클래스에 상속받은 CachedMemberDao클래스를 만들었다.
기존의 MemberDao객체를 CachedMemberDao로 변경해야 하는 상황

#### ❌ 의존 주입 하지않은 경우, 객체를 직접 생성하는 경우의 변경

❔ 변경 전
```java
public class MemberRegisterService {
    private MemberDao memberDao = new MemberDao();
    ...
}

public class ChangePasswordService {
    prviate MemberDao memberDao = new MemberDao();
    ...
}
```
❕ 변경 후
```java
public class MemberRegisterService {
    private MemberDao memberDao = new CachedMemberDao();
    ...
}

public class ChangePasswordService {
    prviate MemberDao memberDao = new CachedMemberDao();
    ...
}
```
 - 생성하는 객체마다 전부 새롭게 추가된 클래스명을 변경해줘야한다.
 - 매우 번거롭다..

#### ⭕ 의존주입을 한 경우의 변경, 객체를 다른 클래스의 생성자를 통해서 주입하는 경우의 변경

❔ 변경 전
```java
MemberDao memberDao = new MemberDao();
MemberRegisterService rgSc = new MemberRegisterService(memberDao);
ChangePasswordService pwSc = new ChangePasswordService(memberDao);
```
❕ 변경 후
```java
MemberDao memberDao = new CachedMemberDao();
MemberRegisterService rgSc = new MemberRegisterService(memberDao);
ChangePasswordService pwSc = new ChangePasswordService(memberDao);
```
- 의존 주입의 대상이되는 객체를 생성하는 곳에서만 변경 해주면된다.
- 이전의 직접 객체 생성하는 방식이라면, MemberRegisterService클래스, ...등 모든 사용하는 클래스에서 생성할때마다 변경해주어야한다.
- 생성자를 통해 객체를 주입하기 때문에 의존하는 객체를 사용하는 상황에서만 생성자를 생성하면서 주입하면된다!

### 객체 조립기(assembler)
> 의존 객체를 주입한다는 것은 서로 다른 두 객체를 조립한다고 생각할 수 있는데, 이런 의미에서 이 클래스를
> 조립기라고도 표현한다.

```java
package assembler;

import spring.ChangePasswordService;
import spring.MemberDao;
import spring.MemberRegisterService;

public class Assembler {

	private MemberDao memberDao;
	private MemberRegisterService regSvc;
	private ChangePasswordService pwdSvc;

	public Assembler() {
		memberDao = new MemberDao();
		regSvc = new MemberRegisterService(memberDao);
		pwdSvc = new ChangePasswordService();
		pwdSvc.setMemberDao(memberDao);
	}

	public MemberDao getMemberDao() {
		return memberDao;
	}

	public MemberRegisterService getMemberRegisterService() {
		return regSvc;
	}

	public ChangePasswordService getChangePasswordService() {
		return pwdSvc;
	}

}
```
 - Assembler 클래스에서 Assembler객체를 생성한다.
 - MemberRegisterService 클래스는 생성자를 통해, ChangePasswordService 클래스는 setter를 통해서 객체를 주입한다.

이전에 학습한 내용처럼 MemberDao클래스가 아니라 MemberDao 클래스를 상속받는 CachedMemberDao클래스를
사용한다면 Assembler클래스에서 객체를 초기화하는 코드만 변경하면 된다.

### 스프링의 DI 설정
> 스프링은 Assembler 클래스의 생성자 코드 처럼 필요한 객체를 생성하고,
> 생성한 객체에 의존을 주입한다.
> 또한 스프링은 Assembler#getMemberRegisterService() 메서드처럼 객체를 제공하는 기능을 정의하고있다.
> 차이점이라면 Assembler는 MemberRegisterService나 MemberDao와 같이 특정 타입의 클래스만 생성한 반면 스프링은 범용 조립기라는 점이다.

#### 스프링을 이용한 객체 조립과 사용
```java
package config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import spring.ChangePasswordService;
import spring.MemberDao;
import spring.MemberRegisterService;

@Configuration
public class AppCtx {
    public AppCtx() {
    }

    @Bean
    public MemberDao memberDao() {
        return new MemberDao();
    }

    @Bean
    public MemberRegisterService memberRegSvc() {
        return new MemberRegisterService(this.memberDao());
    }

    @Bean
    public ChangePasswordService changePwdSvc() {
        ChangePasswordService pwdSvc = new ChangePasswordService();
        pwdSvc.setMemberDao(this.memberDao());
        return pwdSvc;
    }
}
```
- @Configuration 애노테이션은 스프링 설정 클래스를 의미한다.
- 이 애노테이션을 붙여야 스프링 설정 클래스로 사용할 수 있다.
- @Bean 애노테이션은 해당 메서드가 생성한 객체를 스프링빈이라고 설정한다.
- @Bean이 붙어있는 각각의 메서드마다 한 개의 빈 객체를 생성한다.
- 메서드의 이름을 빈 객체의 이름으로 사용한다.

> 객체를 생성하고 의존 객체를 주입하는 것은 스프링 컨테이너 이므로 설정 클래스를 이용해서 컨테이너를 생성해야 한다.

```java
AbstractApplicationContext ctx = new AnnotationConfigApplicationContext(AppCtx.class);
```

AnnotationConfigApplicationContext 클래스를 이용해서 스프링 컨테이너를 생성한다.
컨테이너를 생성하면 getBean()메서드를 이용해서 사용할 객체를 구할 수 있다.

```java
MemberRegisterService regSvc = ctx.getBean("memberRegSvc", MemberRegisterService.class);
```
스프링 컨테이너(ctx)로 부터 이름이 "memberRegSvc"인 빈 객체를 구한다.
```java
package main;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

import config.AppCtx;
import spring.ChangePasswordService;
import spring.DuplicateMemberException;
import spring.MemberNotFoundException;
import spring.MemberRegisterService;
import spring.RegisterRequest;
import spring.WrongIdPasswordException;

public class MainForSpring {

    private static ApplicationContext ctx = null;

    public static void main(String[] args) throws IOException {
        ctx = new AnnotationConfigApplicationContext(AppCtx.class);

        BufferedReader reader =
                new BufferedReader(new InputStreamReader(System.in));
        while (true) {
            System.out.println("명령어를 입력하세요:");
            String command = reader.readLine();
            if (command.equalsIgnoreCase("exit")) {
                System.out.println("종료합니다.");
                break;
            }
            if (command.startsWith("new ")) {
                processNewCommand(command.split(" "));
                continue;
            } else if (command.startsWith("change ")) {
                processChangeCommand(command.split(" "));
                continue;
            } else if (command.equals("list")) {
                processListCommand();
                continue;
            } else if (command.startsWith("info ")) {
                processInfoCommand(command.split(" "));
                continue;
            } else if (command.equals("version")) {
                processVersionCommand();
                continue;
            }
            printHelp();
        }
    }

    private static void processNewCommand(String[] arg) {
        if (arg.length != 5) {
            printHelp();
            return;
        }
        MemberRegisterService regSvc =
                ctx.getBean("memberRegSvc", MemberRegisterService.class);
        RegisterRequest req = new RegisterRequest();
        req.setEmail(arg[1]);
        req.setName(arg[2]);
        req.setPassword(arg[3]);
        req.setConfirmPassword(arg[4]);

        if (!req.isPasswordEqualToConfirmPassword()) {
            System.out.println("암호와 확인이 일치하지 않습니다.\n");
            return;
        }
        try {
            regSvc.regist(req);
            System.out.println("등록했습니다.\n");
        } catch (DuplicateMemberException e) {
            System.out.println("이미 존재하는 이메일입니다.\n");
        }
    }

    private static void processChangeCommand(String[] arg) {
        if (arg.length != 4) {
            printHelp();
            return;
        }
        ChangePasswordService changePwdSvc =
                ctx.getBean("changePwdSvc", ChangePasswordService.class);
        try {
            changePwdSvc.changePassword(arg[1], arg[2], arg[3]);
            System.out.println("암호를 변경했습니다.\n");
        } catch (MemberNotFoundException e) {
            System.out.println("존재하지 않는 이메일입니다.\n");
        } catch (WrongIdPasswordException e) {
            System.out.println("이메일과 암호가 일치하지 않습니다.\n");
        }
    }
    
    ... printHelp 생략
}
```
- import org.springframework.context.ApplicationContext;
  import org.springframework.context.annotation.AnnotationConfigApplicationContext; 부분이 임포트 되었다.
- 기존의 Assembler 클래스를 이용해서 객체를 한번에 조립해서 주입하는 방식에서 스프링 컨테이너를 사용하는 방식으로 변경
#### 변경된 부분
- ctx = new AnnotationConfigApplicationContext(AppCtx.class);
  - 스프링 컨테이너를 생성 = Assembler와 동일하게 객체를 생성하고 의존 객체를 주입
- MemberRegisterService regSvc =
   ctx.getBean("memberRegSvc", MemberRegisterService.class);
  - 스프링컨테이너로 부터 "memberRegSvc"의 이름을 가진 빈 객체 생성
- MemberRegisterService regSvc =
  ctx.getBean("memberRegSvc", MemberRegisterService.class);
  - 스프링컨테이너로 부터 "changePwdSvc"의 이름을 가진 빈 객체 생성
  

#### ❌ 에러대응
 - NoSuchBeanDefinitionException : @Bean 설정을 해주지 않았을 때 발생하는 에러이다.

#### DI 방식 : 생성자 방식
 - 의존객체를 주입받아 필드에 할당해준다. ex) this.memberDao
 - 전달할 의존 객체가 두개 이상도 가능하다.

#### 생성자 방식 사용하는 예시
> 🔽 0. spring/MemberDao.java 여러개의 객체가 주입 되기 때문에 한번에 찾아내는 메서드를 생성
```java
pulbic class MemberDao {
   
    public Collection<Member> selectAll() {
        return map.value();
    }
    ... 추가 기능
}
```

> 🔽 1. spring/testA.java 클래스를 추가(2개의 객체를 받는 생성자)
```java
pulbic class teatA {
    private MemberDao memberDao;
    private MemberPrinter printer;
    
    public testA(MemberDao memberDao, MemberPrinter printer) {
        this.memberDao = memberDao;
        this.printer = printer;
    }
    ... 추가 기능
}
```

> 🔽 2. config/AppCtx.java(설정파일) 두 개 이상의 인자를 받는 생성자를 사용하는 설정 추가
```java
import spring.testA;

@Bean
public testA listPrinter() {
    return new testA(memberDao(), memberPrinter());//memberDao, memberPrinter 빈 주입
        }
```
> 🔽 3. spring/MainForSpring.java(메인) testA관련 코드 추가
```java
import spring.testA;

private static void testB() {
    testA listPrinter = ctx.getBean("listPringer", testA.class);
        ...기능 사용
        }
```

#### DI 방식 : setter 메서드 방식
 - 전달할 의존 객체가 1개이다.
 - 리턴 타입이 void 이다.

#### setter 메서드 방식 사용하는 예시
> 🔽 1. spring/testB.java setter메서드를 이용해서 의존 객체를 주입 받을 메서드 추가
```java
pulbic class teatB {
    private MemberDao memberDao;
    private MemberPrinter printer;
    
    public void setMemberDao(MemberDao memberDao) {
        this.memberDao = memberDao;
    }
    public void setPrinter(MemberPrinter printer) {
        this.printer = printer;
    }
    ... 추가 기능
}
```

> 🔽 2. config/AppCtx.java(설정파일) setter 메서드 방식을 사용하는 설정 추가
```java
import spring.testB;

@Bean
public testB infoPrinter() {
    testB infoPrinter = new testB();
    infoPrinter.setMemberDao(memberDao());//memberDao 빈 주입
    infoPrinter.setPrinter(memberPrinter());//memberPrinter 빈 주입
    return infoPrinter;
        }
```
> 🔽 3. spring/MainForSpring.java(메인) testA관련 코드 추가
```java
import spring.testB;

private static void testB() {
    testB infoPrinter = ctx.getBean("infoPrinter", testB.class);
        ...기능 사용
        }
```

#### 생성자 vs setter 메서드 방식의 객체 주입 차이점
 - 생성자 방식 : 빈 객체를 생성하는 시점에 모든 의존 객체가 주입된다.
 - setter 메서드 방식 : setter 메서드 이름을 통해 어떤 의존 객체가 주입되는지 알 수 있다.

> 생성자 방식은 어떤 의존 객체를 설정하는지 알아내려면 생성자의 코드를 확인해야 한다. 하지만 setter 메서드 방식은
메서드의 이름으로 유추할 수 있다.
반면에 생성자 방식은 빈 객체를 생성하는 시점에 필요한 모든 의존 객체를 주입 받기 때문에 객체를 사용할 때 완전한 상태로
사용할 수 있다. setter메서드 방식은 setter메서드를 사용해서 필요한 의존 객체를 전달 하지 않아도 빈 객체가 생성되기 때문에 객체를 사용하는 시점에서 
NullPointerException이 발생할 수 있다.

### @Configuration 설정 클래스의 @Bean 설정과 싱글톤
> 스프링 컨테이너가 생성한 빈은 싱글톤 객체이다.
> 다른 설정 메서드에서 memberDao()를 몇 번 호출하더라도 항상 같은 객체를 리턴한다는 것을 의미한다.

```java
@Configuration//중복된 memberDao()를 호출하는 예시
public class AppCtx {

  @Bean
  public MemberDao memberDao() {
    return new MemberDao();
  }

  @Bean
  public MemberRegisterService memberRegSvc() {
    return new MemberRegisterService(memberDao());
  }

  @Bean
  public ChangePasswordService changePwdSvc() {
    ChangePasswordService pwdSvc = new ChangePasswordService();
    pwdSvc.setMemberDao(memberDao());
    return pwdSvc;
  }
}
```
```java
public class AppCtxExt extends AppCtx {//스프링 런타임에 생성한 설정 클래스 예시
    
    private Map<String, Object> beans = ...;
    
    @Override
    public MemberDao memberDao() {
        if(!beans.containsKey("memberDao"))
            beans.put("memberDao", super.memberDao());
        
        return (MemberDao) beans.get("memberDao");
    }
}
```
- 스프링 런타임에 생성한 설정 클래스의 memberDao() 메서드는 매번 새로운 객체를 생성하지 않는다.
- 한 번 생성한 객체를 보관했다가 이후에는 동일한 객체를 리턴한다.
- 따라서 memberRegSvc(), changePwdSvc()메서드에서 memberDao() 메서드를 각각 실행해도 동일한 MemberDao 객체를 사용한다.

### 두 개 이상의 설정 파일 사용하기

```java
package config; //AppCtx.java의 빈 설정을 나눠서 한다.

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import spring.MemberDao;
import spring.MemberPrinter;

@Configuration
public class AppConf1 {
    //생성자 , setter 메서드 방식으로 빈객체를 주입하지 않았다.
	@Bean
	public MemberDao memberDao() {
		return new MemberDao();
	}
	
	@Bean
	public MemberPrinter memberPrinter() {
		return new MemberPrinter();
	}
	
}
```

```java
package config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import spring.ChangePasswordService;
import spring.MemberDao;
import spring.MemberInfoPrinter;
import spring.MemberListPrinter;
import spring.MemberPrinter;
import spring.MemberRegisterService;
import spring.VersionPrinter;

@Configuration
public class AppConf2 {
    //@Autowired 애노테이션을 사용해서 빈객체를 주입한다.
	@Autowired
	private MemberDao memberDao;
	@Autowired
	private MemberPrinter memberPrinter;
	
	@Bean
	public MemberRegisterService memberRegSvc() {
		return new MemberRegisterService(memberDao);
	}
	
	@Bean
	public ChangePasswordService changePwdSvc() {
		ChangePasswordService pwdSvc = new ChangePasswordService();
		pwdSvc.setMemberDao(memberDao);
		return pwdSvc;
	}
	
	@Bean
	public MemberListPrinter listPrinter() {
		return new MemberListPrinter(memberDao, memberPrinter);
	}
	
	@Bean
	public MemberInfoPrinter infoPrinter() {
		MemberInfoPrinter infoPrinter = new MemberInfoPrinter();
		infoPrinter.setMemberDao(memberDao);
		infoPrinter.setPrinter(memberPrinter);
		return infoPrinter;
	}
	
	@Bean
	public VersionPrinter versionPrinter() {
		VersionPrinter versionPrinter = new VersionPrinter();
		versionPrinter.setMajorVersion(5);
		versionPrinter.setMinorVersion(0);
		return versionPrinter;
	}
}
```

 - @Autowired 애노테이션을 이용해서 스프링의 자동 주입 기능을 사용한다.
 - @Autowired 애노테이션을 붙이면 해당 타입의 빈을 찾아서 필드에 할당한다.
 - 위 설정의 경우 스프링 컨테이너는 MemberDao 타입의 빈을 memberDao 필드에 할당한다.

> AppConf1 클래스에서 MemberDao 타입의 빈을 설정했으므로 AppConf2 클래스의 memberDao 필드에는
> AppConf1 클래스에서 설정한 빈이 할당된다.

#### 설정파일이 2개 이기 때문에 컨테이너 생성하는 코드에도 파라미터가 2개이다.
```java
ctx = new AnnotationConfigApplicationContext(AppConf1.class, AppConf2.class );
```

#### @Configuration 애노테이션, 빈, @Autowired 애노테이션
> @Autowried 애노테이션을 붙이게되면 스프링 컨테이너 설정 파일인 @Configuration이 붙어있는 @Bean객체에서
> 생성자 주입, setter 메서드 방식 주입으로 객체를 주입하지않아도 스프링 컨테이너가 알아서 객체를 주입한다.

### getBean() 메서드 사용
 - getBean() 메서드의 첫 번째 인자는 빈의 이름이고, 두 번째 인자는 빈의 타입이다. 
 - 해당 타입의 빈 객체가 한개만 존재하면 빈 객체의 이름은 생략 가능하다. 
 - 해당 타입에 빈 객체가 2개 이상이라면 "NoSuchBeanDefinitionException" 이 발생한다.
```java
VersionPrinter versionPrinter =  ctx.getBean("versionPrinter", VersionPrinter.class);
```
```java
// VersionPrinter.class 의 빈 객체가 한개만 존재하는 경우 빈 객체의 이름 생략 버전
VersionPrinter versionPrinter =  ctx.getBean(VersionPrinter.class);
```

#### ❌ 같은 타입의 2개의 빈객체 생성 예시 이런 경우 빈 객체명을 생략하면 에러발생
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import spring.VersionPrinter;

@Configuration
public class AppCtx4GetBean {
  private MemberPrinter printer = new MemberPrinter();

  @Bean//versionPrinter 빈 객체, VersionPrinter 타입
  public VersionPrinter versionPrinter() {// VersionPrinter라는 타입의 빈 객체를 2개 생성
    VersionPrinter versionPrinter = new VersionPrinter();
    versionPrinter.setMajorVersion(5);
    versionPrinter.setMinorVersion(0);
    return versionPrinter;
  }

  @Bean//oldVersionPrinter 빈 객체, VersionPrinter 타입
  public VersionPrinter oldVersionPrinter() {
    VersionPrinter versionPrinter = new VersionPrinter();
    versionPrinter.setMajorVersion(4);
    versionPrinter.setMinorVersion(3);
    return versionPrinter;
  }
}
```

### 주입 대상 객체를 모두 빈 객체로 설정해야 하나?
- 객체를 스프링 빈으로 등록할 때와 등록하지 않았을 때의 차이는 스프링 컨테이너가 객체를 관리하는지 여부이다.
- 스프링 컨테이너는 자동 주입, 라이플 사이클 관리 등 단순 객체 생성 외에 객체 관리를 위한 다양한 기능을 제공하는데 빈으로 등록한 객체에만 기능을 적용한다. 
- 즉, 반드시 주입할 객체가 빈일 필요는 없지만, 빈으로 등록하지 않는다면 생성 이후에 관리를 직접 처리해야 하는 불편함이 있다. 
- getBean()을 이용해서 객체를 찾을 수 없다.
- __주입 대상의 객체는 빈으로 등록하는것이 편리하다.__

#### 빈으로 등록하지 않은 객체를 주입하는 예시
```java
package config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import spring.MemberDao;
import spring.MemberInfoPrinter;
import spring.MemberPrinter;

@Configuration
public class AppCtxNoMemberPrinterBean {
  private MemberPrinter printer = new MemberPrinter(); // 빈으로 등록해주 읺은 객체

  @Bean
  public MemberDao memberDao() {
    return new MemberDao();
  }

  @Bean
  public MemberInfoPrinter infoPrinter() {
    MemberInfoPrinter infoPrinter = new MemberInfoPrinter();
    infoPrinter.setMemberDao(memberDao());
    infoPrinter.setPrinter(printer);
    return infoPrinter;
  }
}
```
