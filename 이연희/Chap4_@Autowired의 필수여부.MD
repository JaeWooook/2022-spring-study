# @Autowired 어노테이션의 필수 여부
자동 주입 대상이 필수가 아닌 경우(null 값이 들어가도 되는 경우)에는 @Autowired 어노테이션을 false로 설정하면 된다. 매칭되는 빈이 없어도 익셉션이 발생하지 않으며 자동 주입을 수행하지 않는다. 아래 코드에서 DateTimeFormatter 타입의 빈이 존재하지 않으면 익셉션이 발생하지 않고 setDateFormatter() 메서드를 실행하지 않는다. 

```java
@Autowired(required=false)
public void setDateFormateer(DateTimeFormatter dateTimeFormatter){
	this.dateTimeFormatter =dateTimeFormatter;
}
```
다음과 같이 의존 주입 대상에 자바8의 Optional을 사용해도 된다. 자동 주입 대상 타입이 Optional일 경우 일치하는 빈이 존재하지 않으면 **값이 없는 Optional**을 인자로 전달한다. 찾는 빈이 있으면 **값이 있는 Optional**을 인자로 전달한다. 
```java
@Autowired
public void setDateFormateer(Optional<DateTimeFormatter> dateTimeFormatter){
	this.dateTimeFormatter =dateTimeFormatter;
}
```
@Nullable 어노테이션을 이용하는 방법도 있다. 스프링 컨테이너는 세터 메서드를 호출할 때 자동 주입할 빈이 존재하면 해당 빈을 인자로 전달하고, 존재하지 않으면 null을 인자로 전달한다.
```java
@Autowired
public void setDateFormateer(@Nullable DateTimeFormatter dateTimeFormatter){
	this.dateTimeFormatter =dateTimeFormatter;
}
```
앞서 언급한 세 가지 방식은 필드에도 그대로 적용된다. 
```java
//1
@Autowired(required=false) 
private DateTimeFormatter dateTimeFormatter;
//2
@Autowired 
private Optional<DateTimeFormatter> dateTimeFormatter;
//3
@Autowired
@Nullable
private DateTimeFormatter dateTimeFormatter;
```
