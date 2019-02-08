
## spring cloud config server 에 curl 을 통해 요청하는 테스트를 해 보았다.

package tree

---
/message.properties

/message-beta.properties

/message_en.properties

/message_en-beta.properties
.
.

/common-resources/application.yml

/common-resources/application-beta.yml

---

case1>

```application.yml
server:
  port: 8999

spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: https://github.com/junbae2/resources.git
          skipSslValidation: true
```


테스트1) $ curl -X GET http://localhost:8999/message/default/master

```json
{
    "name": "message",
    "profiles": [
        "default"
    ],
    "label": "master",
    "version": "c23a9b1d7d78de868019d777dddcccf9a2caa52b",
    "state": null,
    "propertySources": [
        {
            "name": "https://github.com/junbae2/resources/message.properties",
            "source": {
                "greeting": "hello"
            }
        }
    ]
}
```

name 을 'message' 로 했더니 propertySources 가 조회 되었다.

테스트2) $ curl -X GET http://localhost:8999/application/default/master

```json
{
    "name": "application",
    "profiles": [
        "default"
    ],
    "label": "master",
    "version": "c23a9b1d7d78de868019d777dddcccf9a2caa52b",
    "state": null,
    "propertySources": [

    ]
}
```

name 을 'application' 으로 했더니 propertySources 가 없다

case2> searchPaths: common-resources 를 추가해 봤다

```application.yml
server:
  port: 8999

spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: https://github.com/junbae2/resources
          skipSslValidation: true
          searchPaths: common-resources
```

테스트3) $ curl -X GET http://localhost:8999/message/default/master

```json
{
    "name": "message",
    "profiles": [
        "default"
    ],
    "label": "master",
    "version": "c23a9b1d7d78de868019d777dddcccf9a2caa52b",
    "state": null,
    "propertySources": [
        {
            "name": "https://github.com/junbae2/resources/common-resources/application.yml",
            "source": {
                "profile": "default"
            }
        },
        {
            "name": "https://github.com/junbae2/resources/message.properties",
            "source": {
                "greeting": "hello"
            }
        }
    ]
}
```

1)과 동일한 요청을 했더니, "searchPaths" 의 경로에 있는 application.yml(profile default), 가 propertySources 에 조회 되었다.

테스트4) $ curl -X GET http://localhost:8999/application/beta/master

```json
{
    "name": "application",
    "profiles": [
        "beta"
    ],
    "label": "master",
    "version": "c23a9b1d7d78de868019d777dddcccf9a2caa52b",
    "state": null,
    "propertySources": [
        {
            "name": "https://github.com/junbae2/resources/common-resources/application-beta.yml",
            "source": {
                "spring.profiles.active": "beta"
            }
        },
        {
            "name": "https://github.com/junbae2/resources/common-resources/application.yml",
            "source": {
                "profile": "default"
            }
        }
    ]
}
```

name 을 application 으로 하고, profile 을 beta 로 했더니, 예상대로 application-beta.yml 이 조회 되고, application.yml 이 조회되었다. profile default 는 기본으로 조회되는것 같다.

테스트5) $ curl -X GET http://localhost:8999/message/beta/master

```json
{
    "name": "message",
    "profiles": [
        "beta"
    ],
    "label": "master",
    "version": "c23a9b1d7d78de868019d777dddcccf9a2caa52b",
    "state": null,
    "propertySources": [
        {
            "name": "https://github.com/junbae2/resources/common-resources/application-beta.yml",
            "source": {
                "spring.profiles.active": "beta"
            }
        },
        {
            "name": "https://github.com/junbae2/resources/message-beta.properties",
            "source": {
                "greeting": "hello"
            }
        },
        {
            "name": "https://github.com/junbae2/resources/common-resources/application.yml",
            "source": {
                "profile": "default"
            }
        },
        {
            "name": "https://github.com/junbae2/resources/message.properties",
            "source": {
                "greeting": "hello"
            }
        }
    ]
}
```

예사대로 message-beta.properties 와 appplication-beta.yml 이 조회 되었는데, default 역시 message, application 두가지 모두 조회 되었다.

테스트6) 이번에는 좀 다르게, message_ja 를 조회 하기위에 아래와 같이 요청 하였다.

$ curl -X GET http://localhost:8999/message_ja/beta/master

```json
{
    "name": "message_en",
    "profiles": [
        "beta"
    ],
    "label": "master",
    "version": "c9754bacf611f78c42a7a77ef9c54d0fe980b99e",
    "state": null,
    "propertySources": [
        {
            "name": "https://github.com/junbae2/resources/common-resources/application-beta.yml",
            "source": {
                "spring.profiles.active": "beta"
            }
        },
        {
            "name": "https://github.com/junbae2/resources/message_en-beta.properties",
            "source": {
                "greeting": "hello-beta"
            }
        },
        {
            "name": "https://github.com/junbae2/resources/common-resources/application.yml",
            "source": {
                "profile": "default"
            }
        },
        {
            "name": "https://github.com/junbae2/resources/message_en.properties",
            "source": {
                "greeting": "hello-default"
            }
        }
    ]
}
```
예상한것 처럼 message_ja 의 default, beta application 의 default, beta 가 모두 조회되었다.
