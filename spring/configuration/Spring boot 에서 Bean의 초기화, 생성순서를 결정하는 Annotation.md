Spring boot 가 자동설정을 진행하는 순서대로 코드를 따라가다 보면 여러가지 어노테이션(Annotation) 이 설정 Class 마다 붙어 있는것이 보인다.

필요할때마다 문서나 검색으로 확인하고 넘어 가는데, 볼때마다 너무 헷갈린다.

자주보이는 몇가지만 정리해 보자.

> @AutoConfigureAfter

```
@AutoConfigureAfter(Target.class)
class AfterClass {}
```

Target.class 다음에 어노테이션이 선언된 Class 를 진행한다.

Target.class -> AfterClass 순서이다

> @AutoConfigureBefore(Target.class)

```
@AutoConfigureBefore(Target.class)
class BeforeClass {}
```

Target.class 이전에 어노테이션이 선언된 Class 를 진행한다.

Before.class -> Target.class 순서이다

위 두 어노테이션은 auto-configuration 과정에만 적용된다. 사용자가 생성한 @Configuration 초기화 순서를 위해서는 사용할 수 없다.

사용자의 설정초기화의 순서에 영향을 주는것도 하나 알아보면

> @Import

```
@Configuration
@Import(After.class)
class Target {}

class After{}
```

이런경우 Target.class -> After.class 순서이다. 참고할만 한것은 After.class 의 경우 @Configuration 을 선언하지 않아도, 않아도 선언을 한것처럼 동작한다.

하나만 더 알아보면

> @DependOn

```
class One {}

class Two {}

class Init {

  @Bean
  public One one() {
    return new One();
  }

  @Bean
  @DependsOn(value = {"one"})
  public Two after() {
    return new Two();
  }
}

```

이것은 일반 Bean 의 생성 순서에 영향을 줄수 있는데, 위와 같은경우 Two 를 생성하려면, "one" 이름을 가진 Bean 을 생성해야한다는 의미이다. 위의 생성순서는 "one" -> "two" 이다.
