Spring boot 로 jpa 의 설정을 수정하면서(jta 사용을 위해서) 작업을 하다보니, entity 의 column name 이 지정한 전략에 맞게 변경되지 않는 상황이 발생했다.(user0_.created_ymdt 가 되어야 한다!)

```console
org.h2.jdbc.JdbcSQLException: Column "USER0_.CREATEDYMDT" not found; SQL statement:
select user0_.id as id1_2_, user0_.createdYmdt as createdY2_2_, user0_.creator as creator3_2_, user0_.modifiedYmdt as modified4_2_, user0_.modifier as modifier5_2_, user0_.email as email6_2_, user0_.nickname as nickname7_2_, user0_.password as password8_2_ from users user0_ [42122-197]
```

application.yml 에 아래 와 같이 전략정보를 설정하고

```àpplication.yml
jpa:
  show-sql: true
  database-platform: org.hibernate.dialect.H2Dialect
  open-in-view: false
  hibernate:
    naming: # 여기가 naming 전략을 지정하는 부분
      implicit-strategy: org.hibernate.boot.model.naming.ImplicitNamingStrategyLegacyJpaImpl
      physical-strategy: com.ithink.btyes.auth.configuration.jpa.CustomPhysicalNamingStrategy
    ddl-auto: update
```

CustomPhysicalNamingStrategy.java 는 camel casing 인 entity class의 멤버이름을 DB column 이름으로 전환할때 "_" 를 붙여 주도록 확장한 것이다. 어디서 주워온 것이다.

디버깅을 했는데 JpaProperties.java 에서는 분명 지장한 전략이 입력되는 것을 확인 했다.

그렇다면 지정된 전략을 덧씌우는 무언가가 있다고 판단하고 쿼리를 생성하는 과정을 유심히 보기 시작 했다.

1.	먼저, query 생성을 위한 자원을 수집할때, 지정한 전략을 통해 명칭을 바꿔 줄것이다! 라고 생각 했다.

AbstractEntityPersister.java 를 생속한 세가지 확장 class 에서

AbstractEntityPersister:: SingleTableEntityPersister, JoinedSubclassEntityPersister, UnionSubclassEntityPersister 가 있는데(hibernate5.x 기준) Entity 의 정보를 수집해서 쿼리를 만들 준비를 해 두는? 내용들이 있다.

컬럼명, 별칭 등을 만드는 부분을 확인하니, 이미 entity class 의 column 으로 지정된 멤버의 이름이 컬럼명으로 저장되어 있었다. 여기는 아니다.

1.	CustomPhysicalNamingStrategy 에서 컬럼명 교체부분이 호출되는 부분을 역으로 찾아갔다.

이름도 특이한 org.hibernate.cfg.Ejb3Column.java 의 메서드 redefineColumnName() 에서, 인자로 넘겨진 "columnName" 을 physicalNamingStrategy를 활용해 변환을 하는곳을 발견했다.

```Ejb3Column.redefineColumnName
final Identifier physicalName = physicalNamingStrategy.toPhysicalColumnName( implicitName, database.getJdbcEnvironment() );
mappingColumn.setName( physicalName.render( database.getDialect() ) );
```

오!! 그런데 신기하게도 physicalNamingStrategy 는 지정한 CustomPhysicalNamingStrategy 가 아닌 SpringPhysicalNamingStrategy 였다. ㅜㅜ SpringPhysicalNamingStrategy 는 전략을 지정하지 않은경우 default 로 사용하는 것이다.

다시, 확인을 시작 했다. 처음 예상처럼 설정이 덧씌어 졌거나, 또 다른 이유가 있거나...

다행히 직후 디버깅에서 놀라운 것을 발견했는데, JpaProperties 의 physicalNamingStrategy를 설정정보에 넣는것보다, Ejb3Column.redefineColumnName() 가 더 빨리 호출 되는것이다.

결론은 spring boot 의 auto-configuration만 사용하지 않고, 몇몇 설정을 수정했더니 설정정보 로딩 순서가 맞지 않게 되어서 문제가 된것이다.

jpa 관련 설정을 진행하는 설정클래스가 작업을 진행하기 전에 JpaProperties.java 를 먼저 진행하도록 조건을 달아줌으로 정상동작을 확인 할수 있었다.

```JpaConfiguration.java
@Configuration
@ConditionalOnBean(value = JpaProperties.class)
@EnableJpaRepositories(
    basePackages = {""...}
    , entityManagerFactoryRef = "entity manager bean"
    , transactionManagerRef = "transaction manager bean"
)
public class JpaConfiguration {
.
.
.
```
