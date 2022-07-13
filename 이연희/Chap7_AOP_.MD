# Chap7 AOP 프로그래밍 - 프록시
> AOP(Aspect Oriented Programming)는 여러 객체에 공통으로 적용할 수 있는 기능을 분리해서 재사용성을 높여주는 프로그래밍 기법이다. **AOP는 핵심 기능과 공통 기능의 구현을 분리**함으로써 핵심 기능을 구현한 코드의 수정 없이 공통 기능을 적용할 수 있게 만들어 준다. 


팩토리얼을 구하기 위한 인터페이스를 정의한다.
```java
public interface Calculator{
	public long factorial(long num);
}
```
for문을 이용해서 팩토리얼을 구하는 클래스와, 재귀호출을 히용해서 팩토리얼을 구하는 클래스를 정의한다.
```java
public class ImpeCalculator implements Calculator{
	@Override
    public long factorial(long num){
    	long result=1;
        for(long i=1; i<=num; i++{
        	result*=i;
        }
        return result;
    }	
}

public class RecCaculator implements Calculator{
	@Override
    public long factorial(long num){
    	if(num==0) return 1;
        else return num*factorial(num-1);
    }
}
```

위에서 구현한 팩토리얼 클래스의 실행 시간을 출력하려면 메서드의 시작과 끝에서 시간을 구하고 차이를 출력하면 된다.
```java
public class ImpeCalculator implements Calculator{
	@Override
    public long factorial(long num){
    	//시작시간 start
    	long start=System.currentTimeMillis();
    	
        long result=1;
        for(long i=1; i<=num; i++{
        	result*=i;
        }
        
        //끝나는 시간 end
        long end=System.currentTimeMillis();
        System.out.printf("실행시간=%d\n",end-start);
        
        return result;
    }	
}
```
RecCalculator 클래스는 재귀호출로 인해서 factorial() 메서드의 시작과끝에 시간을 구해서 시간을 출력하면 여러 번 출력되는 문제가 있다. 그래서 기존 코드를 변경하는 것보다 다음 코드처럼 메서드 실행 전후에 값을 구하는게 나을 수도 있다.
```java
ImpeCalculator impeCal=new ImpeCalculator();
long start1=System.currentTimeMillis();
long fourFactorial1=impeCal.factorial(4);
long end1=System.currentTimeMillis();
System.out.printf("impeCal 실행시간=%d",end1-start1);

RecCalculator recCal=new RecCalculator();
long start2=System.currentTimeMillis();
long fourFactorial2=recCal.factorial(4);
long end2=System.currentTimeMillis();
System.out.printf("recCal 실행시간=%d",end2-start2);
```
그런데 위 방식에서 실행시간을 밀리초 단위가 아닌 나노초 단위로 구해야 하면 어떻게 될까? 중복되어 있는 시간을 구하는 부분을 모두 변경해야 한다.

기존 코드를 수정하지 않고 코드 중복을 피할 수 있는 방법은 없을까?

## 프록시
프록시 객체를 사용해서 중복을 처리할 수 있다. 
ExeTimeCalculator 클래스는 생성자를 통해 다른 Calculator 객체를 전달받아 delegate 필드에 할당하고 delegate.factorial() 메서드를 실행한다. 
```java
public class ExeTimeCalculator implements Calculator{
	private Calculator delegate;
    public ExeTimeCaculator(Calculator delegate){
    	this.delegate = delegate;
    }
    
    @Override
    public long factorial(long num){
    	long start=System.nanoTime();
        long result=delegate.factorial(num);
        long end=System.nanoTime();
        
        System.out.printf("%s 실행시간=%d\n", delegate.getCalss().getSimpleName(), (end-start));
        return result;
    }
```

ExeTimeCalculator 클래스를 사용하면 다음과 같은 방법으로 ImpeCalculator 실행 시간을 측정할 수 있다.
```java
ImpeCalculator implCal=new ImpeCalculator();
ExeTimeCalculator cal=new ExeTimeCalculator(impeCal);
long result=cal.factorial(4);
```
실행흐름을 보면 ExeTimeCalculator 클래스의 factorial() 메서드는 결과적으로 ImpeCalculator의 메서드 실행시간을 구해서 콘솔에 출력한다. 
![](https://velog.velcdn.com/images/yh_lee/post/5470751f-942e-4973-9ed0-75e4d4f7ad03/image.png)

```java
public class MainProxy{
	public static void main(String[] args){
    	ExeTimeCalculator ttCal1=new ExeTimeCalculator(new ImpeCalculatore());
        System.out.println(ttCal1.factorial(20));
        
        ExeTimeCalculator ttCal2=new ExeTimeCalculator(new RecCalculatore());
        System.out.println(ttCal2.factorial(20));
    }
}
```
	ImplCalculator 실행시간=3123
    RecCalculator 실행시간=3570
    
- 기존 코드를 변경하지 않고 실행시간을 출력할 수 있다. 
- 실행시간을 구하는 코드의 중복을 제거했다. 나노초 대신 밀리초를 사용하고 싶으면 ExeTimeCalculator 클래스만 변경하면 된다.
- factorial() 기능 자체를 직접 구현하기보다 _**다른 객체에 factorial()의 실행을 위임한다.**_
- 계산 기능 외의 다른 부가적인 기능을 실행한다. 여기서 부가적인 기능은 실행 시간 측정이다.

### Summary
프록시의 특징은 핵심 기능은 구현하지 않는다는 점이다. ExeTimeCalculator 클래스는 팩토리얼 연산 자체를 구현하고 있지 않다. 프록시는 핵심 기능을 구현하지 않는 대신 _**여러 객체에 공통으로 적용할 수 있는 기능을 구현한다.**_ 이렇게 공통 기능 구현과 핵심 기능 구현을 분리하는 것이 AOP의 핵심이다. 