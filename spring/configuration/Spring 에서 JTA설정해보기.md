어플리케이션에서 2개이상의 DB를 사용하것이 어려운 이유는, DB들의 트랜젝션처리가 까다로워지기 때문일 것이다.(저장소 간의 원자성이 유지되어야 한다)

이 상황을 해결하는 방법은 하나만이 아닌데,

가장먼저 생각해볼수 있는것은 chaining 된 transaction manager 를 사용하는것일 것이다. '사용하는 DB들의 트랜젝션을 한꺼번에 처리한다.' 는 것으로 누구나 해결방법을 생각하다보면 나올법한 자연스럽고 직관적인 방법이기 때문이다.

spring 의경우 org.springframework.data.transaction.ChainedTransactionManager.java 를 제공해서 이러한 기능을 쉽게 사용할수 있게 해 준다.

하지만 이번에는 java > spring 환경에서 jta를 사용하여 분산저장소환경에서 트랜젝션처리를 하기위한 방법을 고민한 과정을 정리해본다.

> 1 . XADataSource

XADataSource의 [XA](https://ko.wikipedia.org/wiki/X/Open_XA)는 X/Open 이란 단체에서 분산저장소의 트랜잭션처리를 위한 표준을 정의한것이다. XADataSource 는 이런 XA표준에 맞게 구현된 DataSource 이며, 많이 사용되는 DBMS들은 구현체를 제공한다.(mysql, oracle 등등.)

XA가 꼭 DB 에만 적용되는것은 아니다. 프로토콜에 맞게 트랜젝션처리를 할수있도록 구현이되어있다면 어떤저장소든 분산트랜젝션에 참여 할수 있다. (일단은 대표적으로 DB 와 JMS가 있다.)

Java 에서는 JTA 라는 api로 XA리소스들의 트랜젝션을 처리한다.

> 2 . JtaTransactionManager

spring boot 에서는 bitronix, atomikos 두가지의 JTA 구현체에 대해서 설정기능을 제공 하는데, 이번의 대상은 bitronix 이다. (위 두가지 외에도 원하는 jta 구현체가 있다면, 큰 차이 없이 사용설정을 할수 있다.)

spring 위에서 동작하게 할 것이기 때문에, 설정의 과정은 결국 JtaTransactionManager 를 Bean 으로 등록하는 과정이다.

##### 2-1. org.springframework.transaction.jta.JtaTransactionManager 는 두가지 생성인자를 필요로 한다

-	javax.transaction.TransactionManager, javax.transaction.UserTransaction

bitronix.tm.BitronixTransactionManager 는 TransactionManager 이면서 UserTransaction 의 역할도 한다.

```
...
  @Bean
  @ConditionalOnMissingBean({TransactionManager.class})
  public BitronixTransactionManager bitronixTransactionManager(bitronix.tm.Configuration configuration) {
    return TransactionManagerServices.getTransactionManager();
  }

  @Bean
  public JtaTransactionManager transactionManager(TransactionManager transactionManager) {
    JtaTransactionManager jtaTransactionManager = new JtaTransactionManager(transactionManager);
    if (this.transactionManagerCustomizers != null) {
      this.transactionManagerCustomizers.customize(jtaTransactionManager);
    }

    return jtaTransactionManager;
  }
...
```

> 3 . bitronix.tm.Configuration

Configuration 는 일종의 옵션정보로 트랜젝션이 진행되는 과정에서 소소하게 참여 한다. https://github.com/bitronix/btm/wiki/Transaction-manager-configuration

```
...
  @Bean
  @ConditionalOnMissingBean

  public bitronix.tm.Configuration bitronixConfiguration() {
    bitronix.tm.Configuration config = TransactionManagerServices.getConfiguration();
    if (StringUtils.hasText(this.jtaProperties.getTransactionManagerId())) {
      config.setServerId(this.jtaProperties.getTransactionManagerId());
    }

    File logBaseDir = this.getLogBaseDir();
    config.setLogPart1Filename((new File(logBaseDir, "part1.btm")).getAbsolutePath());
    config.setLogPart2Filename((new File(logBaseDir, "part2.btm")).getAbsolutePath());
    config.setDisableJmx(true);
    return config;
  }
...
```

bitronix 는 [AtomicReference](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/atomic/AtomicReference.html) 에 Configuration 정보를 저장해 두는데 초기에 설정해두면 트랜젝션 진행때 마다 참조 하게 된다.

##### 끝이다

JtaTransactionManager 는 PlatformTransactionManager 의 구현체로 어노테이션을 통한 트랜젝션트리를 가능하게 하는 옵션이 적용되어있다면 (@EnableTransactionManagement 이 선언되어 있거나 xml base 인경우 <tx:annotation-driven ... transaction-manager="{transaction manager bean}" />) @Transactional 을 적용하는것 만으로 작동이된다.

###### XA 에대해서는 전문적인 지직을 단기간에 익히기가 어려워서, 간략한 설명과 링크로 대신했다.

github - https://github.com/junbae2/spring-challenge/tree/master/mvn-jooq-multi-ds
