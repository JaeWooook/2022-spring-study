# 의존 자동 주입

<br>

## @Autowired 애노테이션을 이용한 의존 자동 주입

📌 chap03 스프링 DI : 설정 클래스에서 의존 대상을 직접 주입

### ✅ @Autowired : 자동으로 의존하는 빈 객체 주입하는 애노테이션

(@Resource도 가능)

-  의존 객체를 명시하지 않아도 스프링이 알아서 의존 객체를 찾아 주입
    ➡️ 의존을 주입할 **필드/메소드**에 @Autowired 애노테이션 붙임

<br>

1. 필드 : @Autowired 설정한 필드에 타입이 일치하는 빈 객체 주입

2. 메소드 : @Autowired 설정한 메소드 파라미터 타입에 해당하는 빈 객체 주입

<br>

```
@Bean
    public ChangePasswordService changePwdSvc(){
        return new ChangePasswordService();
    }
    // 기존엔 설정클래스에서 setter 메소드 이용해 의존 주입
```
```
public class ChangePasswordService {

    @Autowired
    private MemberDao memberDao; //의존 주입할 필드

    public void setMemberDao(MemberDao memberDao){
        this.memberDao = memberDao;
    }
    ...
}

```
<br>

✍️ 따라서 @Autowired를 사용하면 설정클래스에서 의존주입 코드를 따로 작성하지 않아도 됨!



<br>

### ✅ 일치하는 빈이 없는 경우

<br>
🔎 @Autowired를 설정한 대상에 일치하는 빈이 없는 경우?

<br>

``` NoSuchBeanDefinitionException: No qualifying bean of type 'spring.MemberDao' available  ``` <br>
: 해당 빈을 찾을 수 없다는 Exception 발생, 적용할 수 있는 MemberDao 타입의 빈이 없음


<br>
🔎 @Autowired를 설정한 대상에 일치하는 빈이 2개 이상인 경우?
 
``` NoUniqueBeanDefinitionException: No qualifying bean of type 'spring.MemberDao' available ``` <br>
: 두개의 빈을 발견했다는 Exception 발생

➡️ 자동 주입을 하려면 **해당 타입의 빈이 어떤 빈인지 정확하게 한정**할 수 있어야 함!



<br>

### ✅ 의존 객체 선택

- @Qualifier 애노테이션

    : 사용할 의존 객체를 선택, 설정에서 bean의 한정자 값을 설정

  ➡️ @Autowired 어노테이션이 적용된 주입 대상에 @Qualifier 어노테이션을 설정 (이때 @Qualifier의 값으로 한정자를 사용)

#### @Qualifier 위치

1. @Bean 애노테이션을 붙인 빈 설정 메소드

```
@Bean
    @Qualifier("printer")
    public MemberPrinter memberPrinter1(){
        return new MemberPrinter();
    }
```

2. @Autowired 애노테이션에 자동 주입할 빈

```
public class MemberInfoPrinter {

    private MemberDao memberDao;
    private MemberPrinter printer;

    ...

    @Autowired
    @Qualifier("printer")
    public void setPrinter(MemberPrinter printer) {
        this.printer = printer;
    }
}
```

<br>



### 📌 빈 설정에 @Qualifier가 없으면 빈의 이름을 한정자로 지정

```
    @Bean
    public MemberPrinter printer(){
        return new MemberPrinter();
    } // 한정자 : printer

    @Bean
    @Qualifier("mprinter")
    public MemberPrinter printer2(){
        return new MemberPrinter();
    } // 한정자 : mprinter

```

- @Autowired도 @Qualifier가 없으면 필드/파라미터 이름을 한정자로 사용

```
public class MemberInfoPrinter {

    @Autowired
    private MemberDao memberDao;

    @Autowired
    private MemberPrinter printer; //한정자 : printer

}
```



### 📌 상위/하위 타입 관계에서의 자동 주입

```
    @Bean
    public MemberPrinter printer1(){
        return new MemberPrinter();
    }

    @Bean
    public MemberSummaryPrinter printer2(){
        return new MemberSummaryPrinter();
    }
```

: memberPrinter2 빈을 MemberSummaryPrinter 타입임에도 **동일한 타입의 빈을 2개 발견**했다는 NoUniqueBeanDefinitionException 발생

<br>

### why?
MemberSummaryPrinter 클래스가 MemberPrinter 클래스를 상속했기 때문 → MemberSummaryPrinter 클래스는 MemberPrinter 타입에도 할당 가능

∴ 스프링 컨테이너는 MemberPrinter 타입 빈을 자동 주입해야할때 printer1 빈과 printer2 빈 둘 중 어떤 빈을 주입해야 할지 알 수 없음





<br><br>

### ✅ 자동 주입할 빈 고르는 방법
1. @Qualifier 애노테이션 사용
2. 하위 타입의 빈을 주입 받도록 코드 수정
    
    - MemberPrinter타입 빈 : MemberPritner, MemberSummaryPrinter 둘 다 가능
    - MemberSummaryPrinter타입 빈 : MemberSummaryPrinter만 가능


<br>



### ✅ @Autowired 애노테이션의 필수 여부

- @Autowired 애노테이션을 붙인 타입에 해당하는 빈이 존재하지 않으면 익셉션 발생시킴

    → 자동 주입할 대상이 필수가 아닌 경우에 대한 방법 필요 (필드/메소드 둘 다 가능)

1. @Autowired(require=false)
```
@Autowired(required = false)
public void setDateTimeFormatter(DateTimeFormatter formatterOpt){
        this.dateTimeFormatter = dateTimeFormatter;
}
```
: 빈이 없으면 자동 주입을 수행하지 않음 → 필드/메소드에 null 전달하지 않음(초기화 상태 유지)

<br>
2. Optional 

```
@Autowired
public void setDateTimeFormatter(Optional<DateTimeFormatter> formatterOpt){
        if(formatterOpt.isPresent()){
            this.dateTimeFormatter = formatterOpt.get();
        }else{
            this.dateTimeFormatter = null;
        }
}
```

: 자동 주입 대상 타입이 Optional인 경우

1️⃣ 빈 존재: 해당 빈을 값으로 갖는 Optional을 인자로 전달

2️⃣ 빈 존재x : 값이 없는 Optional을 인자로 전달

<br>

3. @Nullable
```
@Autowired
public void setDateTimeFormatter(@Nullable DateTimeFormatter dateTimeFormatter){
        this.dateTimeFormatter = dateTimeFormatter;
    }
```

: 자동 주입할 빈이 존재하면 해당 빈을 인자로 전달, 존재하지 않으면 null 전달(초기화 → null)

<Br><br>
  
 ------------------------------
  
  
  
  
  
  
🔎 @Autowired(require=false) vs. @Nullable

- @Autowired(require=false): 대상 빈이 존재하지 않으면 해당 메소드 호출x
- @Nullable : 자동 주입할 빈이 존재하지 않아도 메소드 호출

<br>
🔎 자동 주입 vs. 명시적 의존 주입

  : 설정 클래스에서 의존 주입해도 해당 메소드에 @Autowired있다면 자동 주입통해 일치하는 빈 주입

➡️ 자동 주입 기능 사용하는 편이 나음



  
  
  
