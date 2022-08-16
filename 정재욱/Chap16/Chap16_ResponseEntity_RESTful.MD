## ResponseEntity

### ResponseEntity란?

> Spring Framework에서 제공하는 클래스 중 HttpEntity라는 클래스가 존재한다.
>
> 이것은 HTTP 요청(Request) 또는 응답(Response)에 해당하는 HttpHeader와 HttpBody를 포함하는 클래스다.
> 그러므로 ResponseEntity에서는 HTTP header, body, status를 함께 반환할 수 있다.

### ResponseEntity 사용 이유

> ResponseEntity를 사용하지 않고 그냥 객체를 반환하면 되지 않을까?
>
> @RequestBody가 붙은 곳에서 객체를 HttpMessage로 자동 변환 해준다.
> 또한, JSON형태로 변환이 필요하다면 MappingJacksonHttpMessageConverter가 자바 Object와 JSON문서 사이에서 자동으로 변화나해준다.
> @ResponseBody를 붙이면 알맞은 HttpMessageConter가 동작해서 header, body, status를 만들어준다.
> @ResponseStatus(HttpStatus.OK)를 사용해서 직접 상태를 만들어 줄수 있다.
>
> ResponseEntity를 사용하면, __객체와 status를 함께 보내줄 수 있다.__ 때문에 @ResponseStatus를 사용하지 않아도 되고 메서드 별로 다른 status를 리턴할 수 있기 때문에 좀 더 세밀하게 상태 전송이 가능하다.

```java
public class HttpEntity<T> {
    
    private  final HttpHeaders headers;
    
    @Nullable
    private final T body;
}
```

```java
public class ResquestEntity<T> extends HttpEntity<T>
```

- HttpEntity 클래스를 상속받아 구현한 클래스가 RequestEntity, ResponseEntity 클래스이다.
- ResponseEntity는 사용자의 HttpRequest에 대한 응답 데이터를 포함하는 클래스 이다.
- 따라서 HttpStatus, HttpHeaders, HttpBody를 포함한다.

> ResponseEntity의 생성자

```java
public class ResponseEntity<T> extends HttpEntity<T> {

   private final Object status;

   /**
    * Create a {@code ResponseEntity} with a status code only.
    * @param status the status code
    */
   public ResponseEntity(HttpStatus status) {
      this(null, null, status);
   }

   /**
    * Create a {@code ResponseEntity} with a body and status code.
    * @param body the entity body
    * @param status the status code
    */
   public ResponseEntity(@Nullable T body, HttpStatus status) {
      this(body, null, status);
   }

   /**
    * Create a {@code ResponseEntity} with headers and a status code.
    * @param headers the entity headers
    * @param status the status code
    */
   public ResponseEntity(MultiValueMap<String, String> headers, HttpStatus status) {
      this(null, headers, status);
   }

   /**
    * Create a {@code ResponseEntity} with a body, headers, and a status code.
    * @param body the entity body
    * @param headers the entity headers
    * @param status the status code
    */
   public ResponseEntity(@Nullable T body, @Nullable MultiValueMap<String, String> headers, HttpStatus status) {
      this(body, headers, (Object) status);
   }

   /**
    * Create a {@code ResponseEntity} with a body, headers, and a raw status code.
    * @param body the entity body
    * @param headers the entity headers
    * @param rawStatus the status code value
    * @since 5.3.2
    */
   public ResponseEntity(@Nullable T body, @Nullable MultiValueMap<String, String> headers, int rawStatus) {
      this(body, headers, (Object) rawStatus);
   }

   /**
    * Private constructor.
    */
   private ResponseEntity(@Nullable T body, @Nullable MultiValueMap<String, String> headers, Object status) {
      super(body, headers);
      Assert.notNull(status, "HttpStatus must not be null");
      this.status = status;
   }
 }
```
- 다양한 생성자가 존재하기 때문에 body, status, header값을 null로 해도 무방하다.

#### 간단한 예시

#### 상태 코드만 리턴할 때
```java
@PostMapping("/post/like")
public ResponseEntity<SetLikeDto.Response> updateLike(@RequestBody SetLikeDto.Request postLikeDto, @AuthenticationPrincipal UserDetailsImpl userDetails){
        Post post = postService.setPostLike(postLikeDto,userDetails.getUser());

        SetLikeDto.Response response = modelMapper.map(post, SetLikeDto.Response.class);

        return ResponseEntity.ok().build();
        }
```
- ResponseEntity에 build 패턴을 사용할 수 있도록 코드가 되어있기 때문에 build를 사용해도 무방하다.

#### Body까지 리턴할 때
```java
@PostMapping("/post/like")
public ResponseEntity<SetLikeDto.Response> updateLike(@RequestBody SetLikeDto.Request postLikeDto, @AuthenticationPrincipal UserDetailsImpl userDetails){
    Post post = postService.setPostLike(postLikeDto,userDetails.getUser());

    SetLikeDto.Response response = modelMapper.map(post, SetLikeDto.Response.class);

    return ResponseEntity.ok(response);
}
```

#### 와일드 카드란?
> 제네릭 타입으로 데이터타입을 명시하지않고, 런타임까지 유연하게 가져가겠는 의미이다.
> 특별하게 타입을 지정하지 않고, 되는데로 다 받아준다는 의미로 생각할 수 있다.

#### 와일드 카드를 사용한 예시
```java
@PostMapping("/post/like")
public ResponseEntity<?> updateLike(@RequestBody SetLikeDto.Request postLikeDto, @AuthenticationPrincipal UserDetailsImpl userDetails){
    Post post = postService.setPostLike(postLikeDto,userDetails.getUser());

    SetLikeDto.Response response = modelMapper.map(post, SetLikeDto.Response.class);

    return ResponseEntity.ok(response);
}
```
- 하지만, 기타 여러 부분에서는 와일드카드를 사용하는 것은 가독성, 객체지향적으로 볼때 좋지 못하다는 의견이 많다.
- 그래서 타입의 형식을 맞춰주기 위해 와일드 카드를 사용하기 보다는 Object로 데이터 타입을 명시해주는 것이 가독성 측면과 혹시 모를 에러 측면, 유지보수 측면에서 낫다.

#### Object 형식으로 처리한 예시
```java
@PostMapping("/post/like")
public ResponseEntity<Objects> updateLike(@RequestBody SetLikeDto.Request postLikeDto, @AuthenticationPrincipal UserDetailsImpl userDetails){
    Post post = postService.setPostLike(postLikeDto,userDetails.getUser());

    SetLikeDto.Response response = modelMapper.map(post, SetLikeDto.Response.class);

    return ResponseEntity.ok(response);
}
```

#### 응답 상태코드 (참고)
- 1xx : 전송 프로토콜 수준의 정보 교환
- 2xx : 클라이언트 요청이 성공적으로 수행됨
- 3xx : 클라이언트는 요청을 완료하기 위해 추가적인 행동을 취해야 함
- 4xx : 클라이언트의 잘못된 요청
- 5xx : 서버쪽 오류로 인한 상태 코드

### REST API 중심 규칙

#### URI VS URL

>URI

- URI는 특정 리소스를 식별하는 통합 자원 식별자(Uniform Resource Identifier)를 의미한다. 웹 기술에서 사용하는 논리적 또는 물리적 리소스를 식별하는 고유한 문자열 시퀀스다.

> URL

- URL은 흔히 웹 주소라고도 하며, (Uniform Resource Locator)의 약자로 컴퓨터 네트워크 상에서 리소스가 어디 있는지 알려주기 위한 규약이다. URI의 서브셋이다.


![image](https://user-images.githubusercontent.com/55322459/184913517-d8c629d7-1779-489d-a682-0adb943ea40d.png)

> URL 과 URI의 차이 예시

- https://www.charlezz.com/index.html == url : O  , uri : O
- https://www.charlezz.com/index == url : X , uri : O

#### 중심 규칙
REST에서 가장 중요한 규칙
- URI는 정보의 자원을 표현해야 한다.
- 자원에 대한 행위는 HTTP Method(GET, POST, PUT, DELETE 등)으로 표현한다.

#### 세부 규칙

1. 슬래시 구분자( / )는 계층 관계를 나타내는데 사용한다.
2. URI 마지막 문자로 ( / )를 포함하지 않는다.
    - 즉 URI에 포함되는 모든 글자는 리소스의 유일한 식별자로 사용되어야 하며 URI가 다르다는 것은 리소스가 다르다는 것
    - 역으로 리소스가 다르다면 URI도 달라져야 한다.
3. 하이픈 ( - )은 URI 가독성을 높이는데 사용한다.
4. 밑줄( _ )은 URI에 사용하지 않는다.
5. URI 경로에는 소문자가 적합하다.
    - URI 경로에 대문자 사용은 피하도록 한다. EX) 카멜케이스 지양
6. 파일확장자는 URI에 포함하지 않는다.
    - REST API 에서는 메시지 바디 내용의 포맷을 나타내기 위한 파일 확장자를 URI 안에 포함시키지 않는다.
    - 대신 Accept Header를 사용한다. EX) GET : http://restapi.exam.com/orders/2/Accept: image/jpg
7. 리소스 간에 연관 관계가 있는경우
    - /리소스명/리소스ID/관계가 있는 다른 리소스 명
    - EX) GET: /users/2/orders (일반적으로 소유의 관계를 표현할 때 사용)

> 예시

![image](https://user-images.githubusercontent.com/55322459/184915124-a1514473-d8d4-4f36-8670-c3510672ee7b.png)

#### RESTful 제약 6가지
1. 클라이언트 / 서버 구조
    - 클라이언트는 유저와 관련된 처리를, 서버는 REST API를 제공함으로써 각각의 역활이 확실하게 구분되고 일괄적인 인터페이스로 분리되어 작동할 수 있게 한다
    - REST Server: API를 제공하고 비지니스 로직 처리 및 저장을 책임진다.
    - Client: 사용자 인증이나 context (세션, 로그인 정보) 등을 직접 관리하고 책임진다.
    - 서로 간 의존성이 줄어든다.
2. 무상태성 (Stateless)
    - REST는 HTTP의 특성을 이용하기 떄문에 무상태성을 갖는다.
    - 즉 서버에서 어떤 작업을 하기 위해 상태정보를 기억할 필요가 없고 들어온 요청에 대해 처리만 해주면 되기 때문에 구현이 쉽고 단순해진다.
3. 캐시 처리 가능 (Cacheable)
    - HTTP라는 기존 웹표준을 사용하는 REST의 특징 덕분에 기본 웹에서 사용하는 인프라를 그대로 사용 가능하다.
    - 대량의 요청을 효율적으로 처리하기 위해 캐시가 요구된다.
    - 캐시 사용을 통해 응답시간이 빨라지고 REST Server 트랜잭션이 발생하지 않기 때문에 전체 응답시간, 성능, 서버의 자원 이용률을 향상 시킬 수 있다.
4. 자체 표현 구조 (Self - descriptiveness)
    - JSON을 이용한 메시지 포멧을 이용하여 직관적으로 이해할 수 있고 REST API 메시지만으로 그 요청이 어떤 행위를 하는지 알 수 있다.
5. 계층화 (Layered System)
    - 클라이언트와 서버가 분리되어 있기 때문에 중간에 프록시 서버, 암호화 계층 등 중간매체를 사용할 수 있어 자유도가 높다
6. 유니폼 인터페이스 (Uniform)
    - Uniform Interface는 Http 표준에만 따른다면 모든 플랫폼에서 사용이 가능하며, URI로 지정한 리소스에 대한 조작을 가능하게 하는 아키텍쳐 스타일을 말한다
    - URI로 지정한 Resource에 대한 조작을 통일되고 한정적인 인터페이스로 수행한다.
    - 즉, 특정 언어나 기술에 종속되지 않는다.
