# Spring Framework
> 스프링 프레임워크를 사용하며 개발하는 동안 겪은 문제에 대한 해결 과정을 기록하는 공간


## Contents
- [Mybatis를 사용하는 멀티 모듈 환경에서 Mapper XML을 찾지 못하는 경우](#mybatis를-사용하는-멀티-모듈-환경에서-mapper-xml을-찾지-못하는-경우)
- [Spring 환경에서 sleuth, zipkin를 사용할 때 컨트롤러 Mock 유닛 테스트하는 방법](#spring-환경에서-sleuth-zipkin를-사용할-때-컨트롤러-mock-유닛-테스트하는-방법)


## Mybatis를 사용하는 멀티 모듈 환경에서 Mapper XML을 찾지 못하는 경우
### 환경
- Spring Boot
- Spring Mybatis
- IntelliJ

### 프로젝트 구조

```
├── Parent_Project
├── build
│   └── ...
├── Child_Project_1
│   └── build
│       ├── ...
├── Child_Project_@
│   └── build
│       ├── ...
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