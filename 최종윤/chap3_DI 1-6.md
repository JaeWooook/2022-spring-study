- 1.의존이란?
- 2.DI를 통한 의존처리
- 3.DI와 의존 객체 변경의 유연함
- 4예제 프로젝트
- 5.객체 조립기
- 6.스프링 DI설정

## 의존이란?
 한 클래스가 다른 클래스의 메서드를 실행할 때 의존한다고 표현한다. 

## DI를 통한 의존처리 

객체를 클래스 내부에서 직접 생성하는 대신 DI는 생성자의 전달인자로 객체를 전달받는 방식을 사용한다 . 

## DI와 의존 객체 변경의 유연함

MD클래스는 db에 데이터를 저장하는데 , 빠른 조회를 위해 캐시를 적용해야 하는 상황에
MD클래스를 상속받은 CachedMD 클래스를 사용하기로 하면 
MRS클래스 내부의 코드와 CPS클래스 내부의 코드를 수정해야 한다. 
클래스 외부에서 생성후 인자로 전달하는 경우  외부에서만 바꾸어주면 수정이 완료된다. 

```
MD md = new MD(); 에서 다음과 같이 바꾸면된다. 
MD md = new CMD(); - CMD는 MD를 상속받았으므로 MD타입 참조변수에 넣을 수 있다.

public class MRS{
	private MD md = new MD();    - 이렇게 클래스 내부에서 생성한 경우 의존 객체가 변경되는 경우 내부도 수정해야함. 
	... }
public class CPS{
	private MD md = new MD();
	...}
```

## 4.예제 프로젝트 만들기
public class Member{
id , email password , name, LocalDateTime이 필드로 들어간다. 
LocalDateTime은 등록시간을 기록하고 
id는 뭐지?

getter 와 setter가 메서드로 있고,
LocalDateTime은 getter만 있고, 
ChangePassword라는 메서드가 있다 .  - 현재 패스워드와  입력받은 전 패스워드가 다르면 exception을 발생시키고 
맞으면  새로운 패스워드로 변경한다. 
인자의 순서를 제대로 입력했는 지 확인하는건가? 

```
public class MemberDao{
	nextId 와  member객체를 email 과 쌍으로 저장하는 Map이 있다. 
member 객체를 키값인 email로 꺼내는 public Member selectByEmail(String email) 이 있다. 
public Member selectByEmail(String email) {
return map.get(email);
}
```

member의 등록된 순서를 나타내며 식별자인 id를 1씩증가시키면서 등록시킨다. 
프로그램이 종료될때까지 값이 유지되도록 static 변수로 지정한다. 

```
public void insert(Member member){
	member.setId(++nextId);
	map.put(member.getEmail().member);   -멤버 저장하는 map은 member의 email을 키값으로 갖는다 . 
}     	
public void update(Member member){
	map.put(member.getEmail(),member); -  member의 키값을 업데이트한다.? 뭐하는거
}
```

회원가입 처리 클래스 
멤버가 중복될때 익셉션을 나타내는 클래스,  
등록할 멤버의 필드값들을 모아서 인자로 전달할 RegisterRequest - 왜 근데 Member로 그냥 전달받지 않고? 
MemberRegisterService -  멤버 등록하는 로직이 담긴 클래스가 필요하다.  


- 컨트롤러 - 웹 MVC의 컨트롤러 역할
- 서비스 - 도메인을 가지고 핵심 비즈니스 로직 구현
- 도메인 - 회원 , 주문,  쿠폰,등 db에 저장되는 비즈니스 도메인 객체 
- 리포지터리 - db에 접근, 도메인 객체를 db에 저장하고 관리 
- db저장소가 선정되지 않은 경우 우선 인터페이스로 구현 클래스를 변경할 수 있도록 설계한다.  

member는 회원 데이터를 나타내고 memberDao는 member를 email과 쌍으로 저장해서
email로 member객체를 찾는 기능을 하네 

중복될떄 발생하는 익셉션은 RuntimeException을 상속받는다.  super(message)를 통해 메시지를 전달한다. 
RegisterRequest 클래스는 Member클래스에서 id 와 LocalDateTime 필드가 빠지고 confirmpassword가 추가됬다 ,,
비슷하게 getter와 setter가 있고  비밀번호가 확인비밀번호와 같은지 boolean값을 리턴하는 메서드가 있다. 

```
MemberRegisterService 는 memberDao의 selectByEmail()를 이용해 동일 회원이 존재하는지 확인하고 존재하면DuplicateMemberException을 발생시킨다. 
public class MemberRegisterService{
private MD md;	
public MRS(MemberDao md){
		this.md = md;
	}
	```
	
	```
public Long register(RegisterRequest req){
	등록하기 전 이메일로 검색하여 회원이 존재하면 동일 이메일 회원이 존재하므로 익셉션
	Member m = MD.sByEmail(req.getEmail());
	if(m!=null)
		throw new DuplicateMemberException("dup email" +req.getEamil());
	Member newM = new Member( req.getEmail(),req.getPassword(),req.getName(),LocalDateTime.now());
	memberDao.insert(newM);
return newM.getId();
}
```

##  5.객체 조립기
- 의존관계가 있는 클래스에서  DI 방식으로 의존 객체 전달할 때
객체 생성을 main 메서드에서 생성하는 것이 나쁘지 않지만 
객체 생성해서 다른 클래스 생성자에 전달해주는 클래스를 새로 정의할 수 있는데 , 이 클래스를 객체 조립기라 부른다.

- 조립기를 사용하면 어떤 클래스가 의존관계에 있는지 몰라도 사용할 수 있겠다.
assembler.getChangePasswordService()를 사용하면 다른 필요한 클래스도 자동으로 생성해서 전달된 객체를 받으니까

- ?memberDao 클래스의 insert()에서 member.setId() 를 사용하는데  member에 의존하는건가? 한 클래스가 다른 클래스의 메서드를 실행할 떄 '의존'한다고 한다는데 ? 

- 데이터 관련 클래스 - member, memberDao 
member는 회원의 데이터를 표현한다.
memberDao는 DB연동에 사용하는 클래스 ,    (DB연동 방법 배우지 않았으므로 자바의 Map을 이용 구현 )
 memberDao는 메모리에 회원 데이터를 보관하므로 프로그램이 종료되면 새로 시작할 때 빈 데이터에서 시작하게 된다. -> 데이터베이스에 회원데이터를 저장해야한다. 

- 가입 처리 관련 클래스 RegisterRequest, MemberRegisterService


- 암호변경 관련 클래스 
ChangePasswordService

## 6.스프링 DI설정 
- 스프링은  DI를 지원하는 조립기이다.   조립기와 유사한 기능을 제공하는데, 
Assembler 생성자 처럼 객체를 생성하고 , 생성한 객체에 의존을 주입한다. 
또 Assembler#getMemberRegisterService 처럼 객체를 제공하는 기능을 정의한다.

차이점은 mrs나 md와 같이 특정 타입의 클래스만 생성한 반면 스프링은 범용 조립기이다. 

- 6.1 스프링을 이용한 객체 조립과 사용
Assembler 대신 스프링 사용하는 코드 
설정 정보:   어떤 객체를 생성하고,   의존을 어떻게 주입할지 정의 

```
@Configuration   -  스프링 설정 클래스로 사용할 것을 의미              
public class AppCtx{
 Assembler는 생성자 함수에서 객체를 생성하고 getter를 만들어 객체를 리턴했다.  생성자 함수 부분이 없어졌다.  getter와 비슷한 @Bean 붙은 메서드에서 객체를 생성하고 리턴하는데 메서드 이름으로 스프링에 등로고딘다. 
	@Bean    
	public MemberDao memberDao(){
		return new MemberDao();	- 생성한 객체는 스프링에 "memberDao"라는 이름으로 등록된다  (메서드 이름으로 등록)
	}
	
	@Bean
	public MemberRegisterService memberRegisterService(){     
		return new MemberRegeisterService(memberDao());
	}
}
설정 클래스를 만들고나서 객체를 생성하고 의존 객체를 주입하는 스프링 컨테이너를 생성해야한다. 
설정 클래스를 이용해 컨테이너 생성 . 
ApplicationContext ctx = new AnnotationConfigApplicationContext(AppCtx.class);
컨테이너를 생성하면 getBean() 메서드를 이용해 사용할 객체를 구할 수 있다. 
MRS regSvc = ctx.getBean("mRS",MRS.class);   - ctx 컨테이너에서 "mRS"인 빈 객체를 구한다. 
컨테이너에 등록할 때 설정정보에서 의존주입을 했으므로 "mRS"는 내부에서 mD 객체를 사용한다.
```
```
Assembler = new Assembler();

MRS mRS = Assembler.getMRS(); 해서 받았는데 

MRS mRS = ctx.getBean("mRS",MRS.class);

private static ApplicationContext ctx = null;     private static으로 설정하고 , null로 초기화시킨다. 
main 메서드에서 입력을 받으므로 throw IOException을 붙여준다.?
Assembler는 직접 객체를 생성하는 반면 ACAC는 설정파일로 부터 생성할 객체와 의존 주입 대상을 정한다.
```
```
의존 객체가 두개 이상인 경우 
하기 위해 추가코드
public Collection<Member> selectAll(){
	return map.values();
}
}

```
