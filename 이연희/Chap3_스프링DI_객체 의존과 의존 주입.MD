Chap3 Dependency Injection
==========================
## ❗ Dependency(의존) 상태
------
#### 이메일로 회원가입을 처리하는 로직
```java
public class MemberRegisterService{
	//의존 객체를 직접 생성
	private MemberDao memberDao = new MemberDao();
    
    public void regist(RegisterRequest req){
    	Member member = memberDao.selectByEmail(req.getEmail());
        //이메일이 DB에 이미 존재할 때 예외 처리
        if (member!=null){
        	throw new DuplicateMemberException("duplicated email "+req.getEmail());
		}
        //신규 가입이 가능할 때 DB에 insert()
        Member newMember=new Member(
        	req.getEmail(), req.getPassword(), req.getName(), LocalDateTime.now()
        );
        memberDao.insert(newMember);
    }
}
```
한 클래스(MemberRegisterService)가 다른 클래스(MemberDao)의 메서드(selectByEmail(), insert())를 실행할 때 이를 **Dependency(의존)** 한다고 표현한다.

__"MemberRegisterService 클래스가 MemberDao 클래스에 의존한다."__


~~~java
//의존하는 MemberDao 객체도 함께 생성
MemberRegisterService svc = new MemberRegisterService();
~~~
의존 객체를 직접 생성(new MemberDao())했기 때문에 MemberRegisterService 객체를 생성하는 순간 MemberDao 객체도 함께 생성된다.

  

## ❗ Dependency Injection(의존 주입)
---------
💡 **Dependency Injection은 의존하는 객체를 직접 생성하는 대신 의존 객체를 전달받는 방식을 사용한다.**

#### MemberRegisterService 클래스에 DI 방식을 적용
```java
public class MemberRegisterService{

	private MemberDao memberDao; 
 
    public MemberRegisterService(MemberDao memberDao){
		this.memeberDao=memberDao;
	}
}
```

**생성자를 통해 MemberRegisterService가 의존(Dependency)하고 있는 MemberDao 객체를 주입(Injection) 받았다.**

```java
MemberDao dao = new MemberDao();
//의존 객체를 생성자를 통해 주입한다.
MemberRegisterServcie svc = new MemberRegisterService(dao);
```
MemberRegisterService 객체를 생성할 때 생성자에 MemberDao 객체를 전달해야 한다.


## ❗ DI와 의존 객체 변경의 유연함
--------
회원 데이터의 빠른 조회를 위해 캐시를 적용해야 하는 상황이 발생했다고 가정하자. MemberDao 클래스를 상속받은 CachedMemberDao 클래스를 생성한다.
```java
public class CachedMemberDao extends MemberDao{
	...
}
``` 
#### 객체 직접 생성시 변경 사항
```java 
//변경전
public class MemberRegisterService{
	private MemberDao memberDao = new MemberDao();
}

//변경후
public class MemberRegisterService{
	private MemberDao memberDao = new ChachedMemberDao();
}
```

#### DI 사용시 변경 사항
```java 
//변경전
MemberDao memberDao = new MemberDao();
MemberRegisterService regSvc = new MeberRegisterService(MemberDao);

//변경후
MemberDao memberDao = new CachedMemberDao();
MemberRegisterService regSvc = new MeberRegisterService(MemberDao);
```
💡 **DI를 사용하면 MemberDao 객체를 사용하는 클래스가 여러 개여도 변경할 곳은 의존 주입 대상이 되는 객체를 생성하는 코드 한 곳 뿐이다.**