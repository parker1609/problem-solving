# Spring Framework
> 스프링 프레임워크를 사용하며 개발하는 동안 겪은 문제에 대한 해결 과정을 기록하는 공간


## Contents
- [Mybatis를 사용하는 멀티 모듈 환경에서 Mapper XML을 찾지 못하는 경우](#mybatis를-사용하는-멀티-모듈-환경에서-mapper-xml을-찾지-못하는-경우)
- [Spring 환경에서 sleuth, zipkin를 사용할 때 컨트롤러 Mock 유닛 테스트하는 방법](#spring-환경에서-sleuth-zipkin를-사용할-때-컨트롤러-mock-유닛-테스트하는-방법)


## 실제 데이터베이스에 적용되지 않는 테스트하기
### 상황
실제 데이터베이스와 연동하는 통합 테스트에서 데이터를 수정하는 `INSERT`, `UPDATE`, `DELETE` 등을 테스트하고 싶은 경우, 실제 데이터베이스에 적용되지 않도록 하고 싶었다.

그리고 한 테스트 클래스에서 여러 테스트 메서드가 동작하는데, 그 중 하나의 메서드가 데이터를 수정할 때 다른 메서드에 영향을 미칠 수도 있다.

### 해결방법
해결방법은 매우 간단하다. 위 상황을 해결하고 싶은 테스트 클래스에 `@Transactional` 어노테이션을 추가해준다. `@Transactional` 어노테이션은 스프링 프레임워크에서 제공해주는 것으로 `@Test` 어노테이션과 함께 사용하면 해당 **메서드가 끝날 때 자동으로 롤백을 수행한다.**

주의할 점은 테스트 클래스 위에 `@Transactional`이 있어야 한다는 점이다. 테스트 내부에서 사용하는 메서드에 `@Transactional`이 선언되어있는지는 상관없다.

#### Reference
- <http://credemol.blogspot.com/2011/01/spring-junit-transaction.html>
- <https://docs.spring.io/spring/docs/current/spring-framework-reference/testing.html>

만약 테스트 메서드에서 트랜잭션이 동작중이고, 해당 트랜잭션이 롤백을 하는지 알고 싶다면 다음과 같은 `assert` 문을 사용한다. 이를 메서드 처음에 선언해두면 만약 실수로 `@Transactional`을 선언해두지 않으면 `assert` 문에서 에러가 발생하므로 그 아래의 데이터베이스의 데이터를 변경하는 쿼리가 동작하는 것을 막을 수 있다.

```java
// 트랜잭션이 동작하고 있는지 확인
assertTrue(TestTransaction.isActive());

// 해당 트랜잭션의 롤백이 활성화되어 있는지 확인
assertTrue(TestTransaction.isFlaggedForRollback());
```

#### Reference
- <https://www.baeldung.com/spring-test-programmatic-transactions>


## Mybatis를 사용하는 멀티 모듈 환경에서 Mapper XML을 찾지 못하는 경우
### 환경
- Spring Boot
- Spring Mybatis
- IntelliJ

### 프로젝트 구조

```
├── Parent_Project
│   ├── Child_Project_1
│   │    └── build
│   │        └──  ...
│   └── Child_Project_2
│        └── build
│            └──  ...
└── build
    └── ...
```

### 해결방법
만약 위와 같은 프로젝트 구조에서 Child_Project에 있는 Mapper를 사용한다고 하자. 여기서 Mapper XML 파일을 찾지 못했다는 에러가 발생한다면 Mapper 경로를 설정하고 있는 곳이 `classpath:...`인지 확인하자. 만약 `classpath:...`로 되어 있다면 `classpath*:...`로 변경해야 한다.

위와 같은 프로젝트 구조에서 `classpath:...`로 경로를 설정하면 Parent_Project의 build 파일 내부를 찾는다. 하지만 Child_Project_1의 Mapper가 있다면 이는 Child_Project_1 하위 build 파일에 Mapper XML이 위치한다. 따라서 찾을 수 없다.

#### `classpath` VS `classpath*`
- `classpath`: 현재 프로젝트가 실행 되었을 때 현재 classloader에 해당하는 경로의 리소스만 선택
- `classpath*`: 현재 프로젝트가 실행 되었을 때 현재 classloader의 경로 뿐만 아니라 상위 classloader를 모두 검색하여 해당 경로의 리소스를 선택

### Reference
- <https://okky.kr/article/442860>

## Spring 환경에서 sleuth, zipkin를 사용할 때 컨트롤러 Mock 유닛 테스트하는 방법
### 환경
- Spring Boot
- spring-cloud-starter-sleuth
- spring-cloud-starter-zipkin

### 해결방법
span, tracer 관련 에러가 발생하면, 수행하는 테스트 코드(Junit4)에 다음과 같은 코드를 추가한다.

```java
@MockBean
private Tracer tracer;

@MockBean
private Span span;

private Random random = new Random();

@Before
public void setup() {
    long id = createId();
    span = Span.builder().name("mock").traceId(id).spanId(id).build();
    doReturn(span).when(tracer).getCurrentSpan();
    doReturn(span).when(tracer).createSpan(anyString());
}

private long createId() {
    return random.nextLong();
}
```

### Reference
- <https://stackoverflow.com/questions/45941614/spring-cloud-is-failing-when-i-try-to-mock-sleuth-in-unit-tests>