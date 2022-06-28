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



