# Dependency Injection - 객체 조립기 
## Assembler(조립기)
💡 객체를 생성하고 의존 객체를 주입해주는 조립기 클래스를 따로 작성할 수 있다. 의존 객체를 주입한다는 것을 서로 다른 두 객체를 조립한다고 생각할 수 있는데, 이런 의미에서 이 클래스를 조립기라고도 표현한다. **Assembler(조립기)는 객체를 생성하고 의존 객체를 주입하는 기능을 제공한다.**
### Assembler class
```java
public class Assembler{

	private MemberDao memberDao;
    private MemberRegisterService regSev;
    private ChangePasswordServcie pwdSvc;
    
   	public Assembler(){
    	memberDao = new MemberDao();
        regSvc = new MemberRegisterService(memberDao);
        pwdSvc = new ChangePasswordService();
        pwdSvc.setMemberDao(memberDao);
    }
    
    public MemberDao getMemberDao(){
    	return memberDao;
    }
    public MemberRegisterservice getMemberRegisterService(){
    	return regSvc;
    }
    public ChangePasswordService getChangePasswordService(){
    	return pwdSvc;
	}
}
```
MemberRegisterService 객체와 ChangePasswordService 객체에 대한 의존을 주입한다. MemberRegisterService는 생성자를 통해 MemberDao 객체를 주입 받고, ChangePasswordService는 세터를 통해 주입받는다. 


Assembler 객체를 생성하는 시점에 사용할 객체가 모두 생성된다. 
### MainForAssembler class
```java
public class MainForAssembler{
	...
    private static Assembler assembler = new Assembler();
    pivate static void processNewCommand(String[] arg){
    	...
        MemberRegitsterService regSvc = assembler.getMemberRegisterService();
        ...
    }
    private static void processChangeCommand(String[] arg){
    	...
        ChangePasswordService changePwdSvc = assembler.getChangePasswordService();
        ...
    }
}
```

Assembler는 자신이 생성하고 조립한 객체를 리턴하는 메서드를 제공한다. 제공하는 메서드를 이용해서 필요한 객체를 구하고 그 객체를 사용하는 것이 전형적인 Assembler 사용법이다.


