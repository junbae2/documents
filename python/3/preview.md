Python3.x
=========

기초
----

### if

> 조건부표현식 > if 조건절을 간단히 표현할수 있다.

```
a = 100
m = ''

if a > 200: m = 'OK!'
else: m = 'More!'

간단히 표현하면
m = 'OK!' if a > 200 else 'More!'
```

심화
----

> Generator expression(생성자표현식)

-	이점은 전체목록을 한번에 작성하지 않으므로 메모리를 덜 사용한다는 점이다. 라고 한다.

```
예1) numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9]
any(x > 1 for num in numbers)

예2) sum(x * 2 for x in numbers)
예3) d = {n: n * 2 for n in numbers}
```

> List comprehension(리스트 내포)

```
예1) numbers = [1, 2, 3]
n2 = [n * 2 for n in numbers]
> n2
 [2, 4, 6] # List 타입이 된다.
```

https://github.com/junbae2/documents.git
