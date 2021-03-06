

## Intellij
### gradle(4.10.2)
#### build.gradle

의존성
```
...
dependencies {
  ...
  compile 'com.querydsl:querydsl-core:4.2.1'
  compile 'com.querydsl:querydsl-jpa:4.2.1'
  compile 'com.querydsl:querydsl-apt:4.2.1'
}
```

QClass 생성
```
...
def queryDslOutput = file("src/main/generated")

sourceSets {
    main {
        java {
            srcDir "src/main/generated"
        }
    }
}

task generateQueryDSL(type: JavaCompile, group: 'build') {
    doFirst {
        if (!queryDslOutput.exists()) {
            file(new File(projectDir, "/src/main/generated")).deleteDir();
            if (!queryDslOutput.mkdirs()) {
                throw new InvalidUserDataException("Unable to create `$queryDslOutput` directory");
            }
        }
    }

    source = sourceSets.main.java
    classpath = configurations.compile + configurations.compileOnly
    destinationDir = queryDslOutput
    options.compilerArgs = [
            "-proc:only",
            "-processor", 'com.querydsl.apt.jpa.JPAAnnotationProcessor,lombok.launch.AnnotationProcessorHider$AnnotationProcessor'
    ]
}
...
```

## 사용하는 이유
JPQL 보다 더 쉽다.

## 사이트
http://www.querydsl.com/


### JpaRepository 와 QueryDsl구현 Repository(CustomRepository)

https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.custom-implementations

### 예제

1) Fetch Lasy 기능을 명시적으로 분리

```java

@Transactional
public void selectLeftjoinTest() {

    userRepositoryOptional.ifPresent(userRepository -> {
        // query 1
        Optional<User> optionalUser = userRepository.findById(1l);
        optionalUser.ifPresent(user -> {
            // query 2
            Company company = user.getCompany();
            company.toString();
        });

        // query 3
        userRepository.findByIdFetched(1l);
    });
}



```

```java
public Optional<User> findByIdFetched(Long id) {

    QUser qUser = QUser.user;
    QCompany qCompany = QCompany.company;

    User user = from(qUser)
            .leftJoin(qUser.company, qCompany).fetchJoin()
                .where(qUser.id.eq(id))
                .fetchOne();

    return Optional.ofNullable(user);
}
```

```sql

// query 1
Hibernate:
    select
        user0_.id as id1_1_0_,
        user0_.company_id as company_6_1_0_,
        user0_.created_at as created_2_1_0_,
        user0_.email as email3_1_0_,
        user0_.modified_at as modified4_1_0_,
        user0_.name as name5_1_0_
    from
        users user0_
    where
        user0_.id=?

// query 2
Hibernate:
    select
        company0_.id as id1_0_0_,
        company0_.created_at as created_2_0_0_,
        company0_.created_by as created_3_0_0_,
        company0_.modified_at as modified4_0_0_,
        company0_.name as name5_0_0_
    from
        companies company0_
    where
        company0_.id=?

// query 3
Hibernate:
    select
        user0_.id as id1_1_0_,
        company1_.id as id1_0_1_,
        user0_.company_id as company_6_1_0_,
        user0_.created_at as created_2_1_0_,
        user0_.email as email3_1_0_,
        user0_.modified_at as modified4_1_0_,
        user0_.name as name5_1_0_,
        company1_.created_at as created_2_0_1_,
        company1_.created_by as created_3_0_1_,
        company1_.modified_at as modified4_0_1_,
        company1_.name as name5_0_1_
    from
        users user0_
    left outer join
        companies company1_
            on user0_.company_id=company1_.id
    where
        user0_.id=?

```

2) update, delete
```java
@Override
public long updateName(Long id, String name) {

    QUser qUser = QUser.user;

    return update(qUser).set(qUser.name, name)
                 .where(qUser.id.eq(id)).execute();
}

@Override
public long removeById(Long id) {

    QUser qUser = QUser.user;

    return delete(qUser)
            .where(qUser.id.eq(id))
            .execute();
}
```

```sql
// query update
Hibernate:
    update
        users
    set
        name=?
    where
        id=?
// query delete
Hibernate:
    delete
    from
        users
    where
        id=?
```
3) between

```java
@Override
public List<User> findByCreatedAtBetween(LocalDateTime start, LocalDateTime end) {
    QUser qUser = QUser.user;

    return from(qUser)
            .where(qUser.createdAt.between(start, end))
               .fetch();
}
```

```sql
Hibernate:
    select
        user0_.id as id1_1_,
        user0_.company_id as company_6_1_,
        user0_.created_at as created_2_1_,
        user0_.email as email3_1_,
        user0_.modified_at as modified4_1_,
        user0_.name as name5_1_
    from
        users user0_
    where
        user0_.created_at between ? and ?
```
4) in, in null condition

```java
@Override
public List<User> findByIdIn(List<Long> userIds) {
    QUser qUser = QUser.user;

    return from(qUser)
            .where(qUser.id.in(userIds))
            .fetch();
}
```

```sql
Hibernate:
    select
        user0_.id as id1_1_,
        user0_.company_id as company_6_1_,
        user0_.created_at as created_2_1_,
        user0_.email as email3_1_,
        user0_.modified_at as modified4_1_,
        user0_.name as name5_1_
    from
        users user0_
    where
        user0_.id in (
            ? , ? , ?
        )

// empty condition
Hibernate:
    select
        user0_.id as id1_1_,
        user0_.company_id as company_6_1_,
        user0_.created_at as created_2_1_,
        user0_.email as email3_1_,
        user0_.modified_at as modified4_1_,
        user0_.name as name5_1_
    from
        users user0_
    where
        1=2
```

5) select to DTO
```java
@Override
public List<UserDTO> findUserDTOByIdIn(List<Long> userIds) {

    QUser qUser = QUser.user;
    QCompany qCompany = QCompany.company;

    return from(qUser).select(Projections.fields(UserDTO.class, qUser.name, qUser.email, qCompany.name.as("companyName")))
            .leftJoin(qUser.company, qCompany) // 중요! not fetchJoin
            .where(qUser.id.in(userIds))
            .fetch();
}
```
```sql
Hibernate:
    select
        user0_.name as col_0_0_,
        user0_.email as col_1_0_,
        company1_.name as col_2_0_
    from
        users user0_
    left outer join
        companies company1_
            on user0_.company_id=company1_.id
    where
        user0_.id in (
            ? , ? , ?
        )
```
