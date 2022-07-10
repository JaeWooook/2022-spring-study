## Chapter4 의존 자동 주입
### @Autowired 애노테이션을 이용한 의존 자동 주입
- 자동 주입 기능을 사용하면 스프링이 알아서 의존 객체를 찾아서 주입한다.
- 스프링 컨테이너 설정에 의존 객체를 명시하지 않아도 스프링 필요한 의존 빈 객체를 찾아서 주입해준다.

#### @Autowired 사용 전(@Bean생성해주는 클래스 파일)
```java
@Bean
public MemberDao memberDao() {//생성자 주입 방법
    return new MemberDao();
        }
@Bean
public ChangePasswordService changePwdSvc() {//setter 메소드 주입 방법
    ChangePasswordSerivce pwdSvc = new ChangePasswordSerivce();
    pwdSvc.setMemberDao(memberDao());
    return pwdSvc;
        }
```
- pwdSvc.setMemberDao(memberDao()); 를 이용하여 빈객체 주입
- ChangePasswordService 객체의 setMemberDao로 MemberDao 빈 객체를 주입하고있다.

#### @Autowired 사용 후(@Bean생성해주는 클래스 파일)
```java
@Bean
public MemberDao memberDao() {//생성자 주입 방법
    return new MemberDao();
        }
@Bean
public ChangePasswordService changePwdSvc() {//setter 메소드 주입 방법
    ChangePasswordSerivce pwdSvc = new ChangePasswordSerivce();
    return pwdSvc;
        }
```
- @Autowired를 이용해서 자동으로 필요한 객체를 주입해주기 때문에 빈 객체를 주입하지 않는다.

#### 의존을 주입할 대상에 @Autowired 애노테이션을 붙여준다.
```java
package spring;

import org.springframework.beans.factory.annotation.Autowired;

public class ChangePasswordService {

	@Autowired
	private MemberDao memberDao;

	public void changePassword(String email, String oldPwd, String newPwd) {
		Member member = memberDao.selectByEmail(email);
		if (member == null)
			throw new MemberNotFoundException();

		member.changePassword(oldPwd, newPwd);

		memberDao.update(member);
	}

	public void setMemberDao(MemberDao memberDao) {
		this.memberDao = memberDao;
	}

}
```
- ChangePasswordService 클래스에서 MemberDao 객체에 @Autowired 애노테이션을 사용했다.
- setMemberDao() 메서드를 __사용하지 않고도__ 스프링 컨테이너가 @Autowired가 붙은 필드에 해당 타입의 빈객체를 찾아서 주입한다.
- @Autowired 애노테이션을 필드나 세터 메서드에 붙이면 스프링은 타입이 일치하는 빈 객체를 찾아 주입한다.

#### 일치하는 빈이 없는 경우
> 스프링 컨테이너가 자동 주입을 하려면 해당 타입을 가진 빈이 어떤 빈인지 정확하게 한정할 수 있어야한다.
> 주입할 타입의 빈이 2개이거나 없는 경우 스프링은 자동 주입에 실패하고 익셉션을 발생 시킨다.

### @Qualifier 애노테이션을 이용한 의존 객체 선택
> 자동 주입 가능한 빈이 2개 이상이면 @Qualifier 애노테이션을 사용해서 자동 주입 대상 빈을 한정한다.

#### @Qualifier 애노테이션 사용 방법 1번째 위치
````java
	@Bean
	@Qualifier("printer")
	public MemberPrinter memberPrinter1() {
		return new MemberPrinter();
	}
	
	@Bean
	@Qualifier("summaryPrinter")
	public MemberSummaryPrinter memberPrinter2() {
		return new MemberSummaryPrinter();
	}
````
- @Bean애노테이션을 붙인 빈 설정 메서드에서 사용한 예시이다.
- 가장 위의 printer값을 갖는 @Qualifier 애노테이션을 붙여 해당 빈의 한정 값을 "printer"로 지정한다.
- 이렇게 지정한 한정 값은 @Autowired 애노테이션에서 자동 주입할 빈을 한정할 때 사용한다.

#### @Qualifier 애노테이션 사용 방법 2번째 위치
```java
@Autowired
@Qualifier("printer")
public void setMemberPrinter(MemberPrinter printer) {
    this.printer = printer;
        }
```
- setMemberPrinter() 메서드에 @Autowired 애노테이션을 붙였으므로 MemberPrinter 타입의 빈을 자동 주입한다.
- 이제 해당 빈이 자동 주입할때 @Bean 객체중 "printer"라고 한정된 MemberPrinter 타입의 (memberPrinter1)을 자동 주입 대상으로 사용한다.


####  빈 이름과 기본 한정자
```java
	@Bean
	public MemberPrinter printer1() {
		return new MemberPrinter();
	}
	
	@Bean
	@Qualifier("mprinter")
	public MemberPrinter printer2() {
		return new MemberPrinter();
	}

    @Bean
    public MemberInfoPrinter2 infoPrinter() {
            MemberInfoPrinter2 infoPrinter = new MemberInfoPrinter2();
            return infoPrinter;
            }
```
- printer()메서드로 정의한 빈의 한정자는 빈 이름인 "printer"가 된다.
- printer2()메서드는 "mprinter"가 한정자가 된다.
- @Autowired 애노테이션도 @Qualifier 애노테이션이 없으면 필드나 파라미터 이름을 한정자로를 사용한다.
- 예를 들면 printer 필드에 일치하는 빈이 2개 이상 존재하면 한정자로 필드 이름인 "printer"를 사용한다.

### 상위/하위 타입 관계와 자동 주입
```java
package spring;

public class MemberSummaryPrinter extends MemberPrinter {

	@Override
	public void print(Member member) {
		System.out.printf(
				"회원 정보: 이메일=%s, 이름=%s\n", 
				member.getEmail(), member.getName());
	}

}
```
위 클래스는 MemberPrinter 클래스를 상속한 MemberSummaryPrinter클래스이다.

AppCtx클래스 설정에서 memberPrinter2() 메서드가 MemberSummaryPrinter 타입의 빈 객체를 설정하도록 변경하자.
```java
	@Bean
	public MemberPrinter memberPrinter1() {
		return new MemberPrinter();
	}
	
	@Bean
	public MemberSummaryPrinter memberPrinter2() {
		return new MemberSummaryPrinter();
	}
```
- memberPrinter2()메서드를 MemberSummaryPrinter가 MemberPrinter클래스를 상속하기 때문에 위처럼 변경해줄 수 있다.
- 하지만, MemberPrinter타입 빈 2개에 __@Qualifier 애노테이션을 붙이지 않았을 때와 동일한 익셉션이 발생한다.__
- memberPrinter2 빈을 MemberSummaryPrinter 타입으로 변경했음에도 중복되는 Bean을 찾지못하는 에러가 발견되는 이유는
- MemberSummaryPrinter 클래스가 MemberPrinter 클래스를 상속했기 때문이다. 
- __MemberSummaryPrinter 클래스는 MemberPrinter 타입에도 할당할 수 있으므로, 스프링 컨테이너는 MemberPrinter 타입 빈을 자동 주입해야 하는 @Autowired 애노테이션 태그를 만나면 memberPrinter1 빈과 memberPrinter2 타입의 빈 중에서 어떤 빈을 주입해야하는지 알수없다.__

#### MemberListPrinter 클래스와 MemberInfoPrinter 클래스가 MemberPrinter 타입의 빈을 자동 주입하므로 어떤 빈을 주입할지 결정하는 2가지 방법
> 우선 AppCtx 설정 파일에 @Qualifier 애노테이션을 이용해서 주입할 빈을 한정한다.
```java
package config;

import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import spring.MemberPrinter;

@Configuration
public class AppCtx {
    
    @Bean
    @Qualifier("printer")
    public MemberPrinter memberPrinter1() {
        return new MemberPrinter();
    }
}

package spring;

        import org.springframework.beans.factory.annotation.Autowired;
        import org.springframework.beans.factory.annotation.Qualifier;

public class MemberInfoPrinter {

    private MemberDao memDao;
    private MemberPrinter printer;

    public void printMemberInfo(String email) {
        Member member = memDao.selectByEmail(email);
        if (member == null) {
            System.out.println("데이터 없음\n");
            return;
        }
        printer.print(member);
        System.out.println();
    }

    @Autowired
    public void setMemberDao(MemberDao memberDao) {
        this.memDao = memberDao;
    }

    @Autowired
    @Qualifier("printer")
    public void setPrinter(MemberPrinter printer) {
        this.printer = printer;
    }
}
```
- MemberInfoPrinter 클래스에서 사용되는 MemberPrinter 빈 객체를 한정해주고, 한정자를 이용해서 자동 주입해준다.

> MemberListPrinter 클래스에 자동 주입할 MemberPrinter 타입 빈 주입하는 방법 1
```java
	@Bean
	@Qualifier("summaryPrinter")
	public MemberSummaryPrinter memberPrinter2() {
		return new MemberSummaryPrinter();
	}

public class MemberListPrinter {

    private MemberDao memberDao;
    private MemberPrinter printer;

    public MemberListPrinter() {
    }

    ...

    @Autowired
    @Qulifier("summaryPrinter")
    public void setMemberPrinter(MemberPrinter printer) {//이부분이 다르다.
        this.printer = printer;
    }
}
```
- 첫번째 위의 방법처럼 한정자를 이용해서 자동 주입을 연결 해줄 수 있다.

> MemberListPrinter 클래스에 자동 주입할 MemberPrinter 타입 빈 주입하는 방법 2
```java
public class MemberListPrinter {

    private MemberDao memberDao;
    private MemberPrinter printer;

    public MemberListPrinter() {
    }

    ...

    @Autowired
    public void setMemberPrinter(MemberSummaryPrinter printer) {//이 부분에 상속 받는 MemberSummaryPrinter 클래스 사용
        this.printer = printer;
    }
}
```
- MemberSummaryPrinter 타입 빈은 한 개만 존재하므로 MemberSummaryPrinter 빈을 자동 주입 받도록 수정한다.
- 자동 주입 대상이 두개 이상이어서 발생하는 문제 피할 수 있다.
- MemberSummaryPrinter 클래스는 MemberPrinter 클래스를 상속 받기 때문에 사용할 수 있다.

### @Autowired 애노테이션의 필수 여부

#### 생성자 초기화와 필수 여부 지정 방식 동작 이해

### 자동 주입과 명시적 의존 주입 간의 관계
