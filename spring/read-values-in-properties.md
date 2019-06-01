# Read values in application.properties

프로젝트 설정을 편리하게 하기 위해 고정된 값을 외부로 빼내어 사용하는 경우가 많다.

Spring 에서는 `application.properties`을 이용하여 가능한데, 여러가지 자료구조형태를 파싱할 수 있기 때문에 편리하다.

`test=100`이라는 값을 Project 내부에서 사용하기 위해서 여러 방법이 있는데 가장 간단한 방법은 아래와 같다.

## @Value(...)
커스텀 value를 사용하기 위해서는 Bean 생성을 먼저 해야한다.
```java
@Configuration
public class EnvironmentConfig {
  @Bean
  public static PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer() {
    return new PropertySourcesPlaceholderConfigurer();
  }
}
```

값 연결을 위해 아래와 같이 작성한다.
```java
class ... {
  @Value("${test}")
  private int test;
}
```