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