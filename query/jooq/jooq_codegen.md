Jooq Codegem
------------

jooq를 효율적으로 사용하려하니, [jooq codegen](https://www.jooq.org/doc/3.11/manual/code-generation/)을 활용하는 것이 좋겠다고 생각이 되었다. [홈페이지에](https://www.jooq.org/doc/3.11/manual/getting-started/tutorials/jooq-in-7-steps/) 자료가 많으니 금방 해치울수 있겠다 했는데... 영어가 짧아서 ㅠ

방법은 세가지가 있다.

1) [maven plugin](https://www.jooq.org/doc/3.11/manual/code-generation/codegen-maven/) 을 사용하는 방법

2) [gradle plugin](https://www.jooq.org/doc/3.11/manual/code-generation/codegen-gradle/) 을 사용하는 방법

3) [java code](https://www.jooq.org/doc/latest/manual/code-generation/codegen-programmatic/) 로 하는 방법

먼저 maven 을 사용하는 방법으로 진행.

### maven

-	java: 8
-	plugin: jooq-codegen-maven
-	db: h2
-	spring-boot

> pom.xml

의존성

```
...
<dependencies>
  .
  .
  .
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jooq</artifactId>
  </dependency>
  <!-- spring boot 를 사용한 환경이 아니라면,
  <dependency>
    <groupId>org.jooq</groupId>
    <artifactId>jooq</artifactId>
    <version>3.10.8</version>
    <scope>compile</scope>
  </dependency>
  이어야 하겠지만, 예제에서는 spring-boot-starter-jooq 가 대신했다-->

  <dependency>
    <groupId>org.jooq</groupId>
    <artifactId>jooq-meta</artifactId>
  </dependency>

  <dependency>
    <groupId>org.jooq</groupId>
    <artifactId>jooq-codegen</artifactId>
  </dependency>
  .
  .
  .
</dependencies>

```

플러그인

```
.
.
.
<build>
  <plugins>
  ...
  <plugin>
        <groupId>org.jooq</groupId>
        <artifactId>jooq-codegen-maven</artifactId>
        <version>3.10.8</version>

        <!-- 자동생성하기 위한 골은 'generate' -->
        <executions>
          <execution>
            <goals>
              <goal>generate</goal>
            </goals>
          </execution>
        </executions>

        <dependencies/>

        <configuration>
          <jdbc>
            <driver>org.h2.Driver</driver>
            <url>jdbc:h2:tcp://localhost/~/{database}</url>
            <user>sa</user>
            <password></password>
          </jdbc>

          <generator>
            <database>
              <name>org.jooq.util.h2.H2Database</name>
              <includes>.*</includes>
              <excludes></excludes>
              <inputCatalog/>
              <inputSchema>PUBLIC</inputSchema><!-- 비워두면 모든 스키마를 포함시킨다. -->
            </database>
            <generate>
              <deprecated>false</deprecated>
              <records>false</records>
              <fluentSetters>true</fluentSetters>
              <relations>true</relations>
              <daos>true</daos><!-- dao 생성여부-->
              <fluentSetters>true</fluentSetters>
            </generate>
            <target>
              <packageName>com.xxx.demo.jooq.model</packageName>
              <directory>src/main/java</directory>
            </target>
          </generator>
        </configuration>
      </plugin>
      ...
  </plugins>
</build>

```

위와 같은 상태로 코드를 생성한 결과는 아래와 같다.

![generated code](/images/2018/08/genereted.png)

일단 생성은 되었는데, plugin 옵션에 이해하지 못한 내용들이 많이 있다. plugin의 <database>, <generator> 지정할수 있는 옵션들이 더 많이 있다.

```
<database>
  <includes>.*</includes>
  <excludes>schema_version</excludes>
  <inputSchema>public</inputSchema>
  <includeTables>true</includeTables>
  <includeRoutines>true</includeRoutines>
  <includePackages>false</includePackages>
  <includeUDTs>true</includeUDTs>
  <includeSequences>true</includeSequences>
  <includePrimaryKeys>false</includePrimaryKeys>
  <includeUniqueKeys>false</includeUniqueKeys>
  <includeForeignKeys>false</includeForeignKeys>
</database>
<generate>
  <deprecated>false</deprecated>
  <records>false</records>
  <fluentSetters>true</fluentSetters>
</generate>
```

좀더 코드를 더 해가면서, 위 옵션들을 테스트 해 볼 생각 이다.

github - https://github.com/junbae2/spring-challenge/blob/master/mvc-jooq/pom.xml
