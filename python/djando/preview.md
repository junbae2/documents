Django
======

setting.py
----------

> USE_TZ = True or False

```
setting.py
..
USE_TZ = True
..


$(가상환경) python manage.py shell
.
.
>>> import {appname}.models from {Model}
>>> import datetime
>>>A
>>> {Model}.objects.filter(created_ymdt__date=datetime.date(2018, 6, 27)).count()
0 !! 왜!!
>>>
```

USE_TZ 가 True 인경우 지정한 Time Zone 이 templates, forms 에서만 적용된다. 위 예제에서 조건에 맞는 데이터가 1건 데이터베이스에 있지만, 결과는 0 이다. USE_TZ 값 False 로변경 후 아래와 같이 테스트 결과를 얻었다.

```
setting.py
..
USE_TZ = False
..

$(가상환경) python manage.py shell
.
.
>>> import {appname}.models from {Model}
>>> import datetime
>>>
>>> {Model}.objects.filter(created_ymdt__date=datetime.date(2018, 6, 27)).count()
1
>>>

```

manage.py
---------

> django shell 테스트

```
${가상환경} python manage.py shell
>>> django 테스트 가 가능한 console 이 활성화 된다.
```
