Gradle 을 알아보자
==================

기본 Task
---------

> init

```
$ gradle init

!type 욥선이 붙을수 있는데, 참고 하자
https://docs.gradle.org/current/userguide/build_init_plugin.html#sec:build_init_types
```

현재 디렉토리에 gradle 관련 기본파일들을 생성해 준다.

gradlew
-------

```
$ gradle wrapper --gradle-version ${version}
...

BUILD SUCCESSFUL in 0s
1 actionable task: 1 executed
```

현재 디렉토리에 아래파일들이 생성된다.

-	.gradle 디렉토리
-	gradlew 실행파일(gradlew.bat)

gradlew 로 편리하게 gradle 을 실행할수(? 표현이 정확한가?) 있다

```
$ gradlew build
$ gradlew run
```

Tip
---

> Dependency 버젼을 환경변수로 관리하기

1.gradle.properties 작성하기

```
springDataVersion = 1.8.0.RELEASE
vertxVersion = 3.5.1
```

2.build.gradle 에서 쓰기

```
dependencies {
    compile "io.vertx:vertx-core:$vertxVersion"
    compile "org.springframework.data:spring-data-jpa:$springDataVersion"
    ...
}
```
