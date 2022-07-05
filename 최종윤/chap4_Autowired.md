1. @Autowired를 통한 자동 의존 주입
2. @Qualifier 애노테이션을 이용한 의존 객체 선택 
3. 빈 이름과 기본 한정자 
4. 상위/하위 타입 관계와 자동 주입
5. @Autowired 의 필수 여부
6. 자동 주입과 명시적 의존 주입 간의 관계

## @Autowired를 통한 자동 의존 주입
setter 메서드에 @Autowired를 붙이면 setter의 인자에 해당하는 빈 객체를 자동 주입한다. 
인자가 없는 기본 생성자를 추가한다. 필드 또는 세터 메서드에 @Autowired 를 붙인다. 
- 설정클래스의 필드에 붙인 경우 의존 주입이 필요한 빈 객체에 자동으로 주입된다. 
- 일반 클래스의 필드에 붙인 경우 해당 타입의 빈 객체를 찾아서 필드에 할당한다. (설정 클래스에서)
- 일반 클래스의 세터메서드에 붙인 경우 세터메서드의 파라미터로 오는 객체를 설정 클래스에서 자동으로 의존 주입해준다. 
```
public MemberInfoPrinter infoPrinter(){
MemberInfoPrinter infoPrinter = new MemberInfoPrinter();
 return infoPrinter;
```
- 설정 클래스에서 해당 필드를 의존 주입하는 코드를 삭제하면 된다.  (삭제 안해도 됨? -> 안하면 코드 무시하고 , 자동 의존 주입이 실행된다. ->목차 6번  )

모든 클래스에 @Autowired를 적용하고 설정 클래스에서 의존을 주입하는 코드를 제거한다. 

- 2.1일치하는 빈이 없는 경우
의존을 충족하지 않는다는 내용  , 빈이 존재하지 않는다는 내용의 exception이 발생하면서 실행이 멈춘다. 

한 타입의 빈 객체가 두개 이상인 경우 
이름이 a,b인 두개의 빈을 발견했다는 에러메시지와 익셉션이 발생한다. 

### @Qualifier 애노테이션을 이용한 의존 객체 선택 
자동 주입할 빈을 지정할 방법 - @Qualifier는 두 위치에서 사용 가능하다.
1.@Bean 붙인 빈설정 메서드  
```
@Bean
@Qualifier("printer")
public MemberPrinter memberPrinter(){
return new MemberPrinter();
}

2.@Autowired               - 다음 위치에 붙이면 앞서 설정 클래스에서 @Qualifier 로 "printer"를 준 MemberPrinter 타입의 빈을
@Qualifier("printer")	자동 주입 대상으로 사용한다. 
public void setMemberPrinter(MemberPrinter printer){
}
```
## 3.빈 이름과 기본 한정자 
빈 설정에 @Qualifier 이 없으면 빈의 이름을 한정자로 지정한다. 
```
@Bean
public MemberPrinter printer(){
return new MemberPrinter();
}

@Bean
@Qualifier("mprinter")
public MemberPrinter printer2(){
return new MemberPrinter();
}
```
@Autowired 도 @Qualifier가 없으면 필드나 파라미터 이름을 한정자로 사용한다.
빈이 두개 이상 존재하면 필드도 한정자가 없으면 필드이름을 한정자로 사용한다. 
@Autowired
private MemberPrinter printer;

## 4.상위/하위 타입 관계와 자동 주입
```
public class MSP extneds MP{
}
AppCtx 클래스 설정에서 mP2() 메서드가 MSP 메서드의 빈 객체를 설정하도록 변경하자.
그리고 @Qualifier 도 삭제한다. 
@Bean
public MP mp1(){
return new MP();
}
@Bean
public MSP mP2(){
return new MSP();
}
```
MLP 와 MIP는 MP를 주입받는다.  MSP는 MP를 상속받는다. 
- MP 대신 MSP를 주입받을 수 있다. - > 한정자가 정해지지 않은 빈이 두개 일때 나타나는 익셉션 발생
```
class MLP{
  void setMP(MSP p){  -> MLP가 MP가 아닌 MSP를 주입받도록 수정하면 MLP는 익셉션 발생을 피할 수 있다. 
    this.p = p;
    }
class MIP{
  @Autowired
  @Qualifier("mP2")     -> 또는 MIP 처럼 @Qualifier("mP2")를 사용하면 익셉션 발생을 피할 수 있다. (빈에서 Qualifier없으니 메서드 이름이 한정자로 사용됬다. )
  private MP p;
  }
  ```
@Qualifier를 지정해준다.
```
@Bean
@Qualifier("msp")
public MSP msp(){
return new MPS();
}
public class MIP{

@Autowired  - 설정 클래스의 필드 , 일반 클래스 세터 메서드 , 또 어디에 ? 
@Qualifier("msp")
public void setMSP(MP mp){
this.mp = mp;
}
}
```
2개이상 빈 객체가 있는 MLP에 주입받는 방법 두가지 
1. @Qualifier 이용
2. MSP를 주입받으면 MSP 는 빈 객체가 한 개이므로 , 익셉션 발생x


### 5.@Autowired 의 필수 여부

- 주입이 되지 않아도 null 인 경우 정상동작 한다면 -> 자동 주입 대상이 필수가 아니다.
- 자동 주입할 대상이 필수가 아닌 경우에 @Autowired의 required 속성을 false로 지정한다. 
```
@Autowired(required = false)
public void setDFM(DFM dfm){
this. dfm = dfm;
}
required를 false로 지정하면 매칭되는 빈이 없어도 익셉션이 발생하지 않으며 자동주입을 수행하지 않는다.
스프링 5부터 의존 주입 대상에 자바8의 Optional을 사용해도 된다. 
@Autowired
public void setDFM(Optional<DTFM> fmOPt){
	it(fmOpt.isPresent()){
	this.dTFM = fmOpt.get();
}else{
this.dTFM = null;
}
}
```
주입 대상이 Optional 인 경우 일치하는 빈이 없으면 값이 없는 Optional을 인자로 전달하고 (익셉션 발생x)
- (required 를 false로 하지 않아도 익셉션 발생x)
일치 빈이 존재하면 필드에 할당하고 없으면 null을 할당한다. 
```
세번쨰 필수여부 지정 방법은 @Nullable 을 사용하는 것이다.    
required속성을 false로 할 떄와 차이점은  @Nullable을 사용하면 빈이 존재하지 않아도
메서드가 호출된다는 점이다.
@Autowired 의 경우 requried =false 인데 빈이 존재하지 않으면 세터메서드를 호출하지 않는다. 
세가지 방식은 필드에도 그대로 적용된다. 
생성자 초기화와 필수 여부 지정 방식 동작 이해
세터메서드에서 @Autowired( required= false)로 하였다 . ->>빈이 존재하지 않아도 익셉션발생x 메서드 실행x
자동 주입 대상 필드를 생성자에서 초기화 시키는 경우  - 빈이 없는 경우 -> 필드에  생성자에서 초기화 시킨값이 할당됨.

@Nullable을 사용한 경우
기본 생성자가 먼저 실행된후 세터메서드에서 필드에 null이 할당된다.
일치하는 빈이 없으면 할당을 하지않는 @Autowired(required=false)와 달리 
@Nullable을 사용하면  일치하는 빈이 없을 떄 null을 할당한다. 
Optional은 없으면 Optional을 할당한다. 기본생성자에서 필드를 초기화할 떄 이점을 유의해야한다. 
```
## 6.자동 주입과 명시적 의존 주입 간의 관계

  설정 클래스에서 의존을 주입했는데 자동 주입 대상이면 어떻게 될까? 
```
AppCtx 설정 클래스의 infoPrinter()메서드를 변경해보자 
@Bean
public MIP infoPrinter(){
MIP mip =  new MIP();
mip.setMP(mP2());
return infoprinter;
}

setPrinter()는 다음 과같이 "printer"를 한정자로 지정한다.
@Autowired
@Qualifier("printer")
public void setMP(MP mp){
this.mp = mp;
}
그리고 "printer" 는 mp2가 아닌 mp1의 한정자 값이다. 
이 상황에서 infoPrinter는 mp2를 주입받는 것이 아닌 @Qualifier("printer")에 의한 mp1을 주입받는다. 
세터메서드에 @Autowired가 붙으면  세터메서드로 명시적으로 값을 전달해도 , 자동 주입에 의해서 주입이 이루어진다. 

수동으로 주입하는 코드와 자동 주입하는코드가 섞여있으면 주입을 제대로 하지 않아 NullPointerException이 발생했을 떄
원인을 찾는데 시간이 걸릴 수있으므로 의존 자동주입 기능을 일관되게 사용해야 이런 문제를 줄일 수 있다. 
```
일부 자동주입이 어려운 코드를 제외한 나머지 코드는 의존 자동주입을 사용한다  . - 어려운 코드가 뭐지? 
