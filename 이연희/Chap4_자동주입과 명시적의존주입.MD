# 자동 주입과 명시적 의존 주입의 관계
설정 클래스에서 의존을 주입했는데 자동 주입 대상이면 어떻게 될까?
infoPrinter() 메서드는 setPrinter() 메서드를 호출해서 memberPrinter2 빈을 주입하고 있다. 
```java
@Configuration
public class AppCtx{
	...
    @Bean
    @Qualifier("printer")
    public MemberPrinter memberPrinter1(){
    	return new MemberPrinter;
    }
    
    @Bean
    public MemberInfoPrinter infoPrinter(){
    	MemberInfoPrinter infoPrinter = new MemberInfoPrinter();
        //의존 주입
        infoPrinter.setPrinter(memberPrinter2());
        return infoPrinter;
    }
}
```

```java
public class MemberInfoPrinter{
	...
    @Autowired
    @Qualifier("printer")
    public void setPrinter(MemberPrinter printer){
    	this.printer = printer;
    }
}
```
결과적으로 memberPrinter2가 아닌 memberPrinter1 빈을 사용한다. 즉 설정 클래스에서 setter 메서드를 통해 의존을 주입해도 해당 메서드에 @Autowired가 붙어있으면 자동 주입을 통해 일치하는 빈을 찾아준다.