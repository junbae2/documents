

create table TEST (
  id bigint(20) auto_increment,
  a bigint(20) not null,
  b varchar(20) not null,
  c int not null,
  d datetime not null,
  primary key (id),
  key idx_test_a_b_c_d(a,b,c,d)
);


## 순서대로

```
explain
select * from TEST
  where a = 1
  and b = '2000000000'
  and c = 200
  and d = now();

결과 ----------------------------------------------------------------
1	SIMPLE	TEST		ref	idx_test_a_b_c_d	idx_test_a_b_c_d	99	const,const,const,const	1	100	Using index
```

## d 빼기
```
explain
select * from TEST
  where a = 1
  and b = '2000000000'
  and c = 200;

결과 ---------------------------------------------------------------
1	SIMPLE	TEST		ref	idx_test_a_b_c_d	idx_test_a_b_c_d	94	const,const,const	1	100	Using index

```

## d, c 빼기

```
explain
select * from TEST
  where a = 1
  and b = '2000000000';

결과 ---------------------------------------------------------------
1	SIMPLE	TEST		ref	idx_test_a_b_c_d	idx_test_a_b_c_d	90	const,const	1	100	Using index
```

## d, c, b 빼기

```
explain
select * from TEST
  where a = 1;

결과 ---------------------------------------------------------------
1	SIMPLE	TEST		ref	idx_test_a_b_c_d	idx_test_a_b_c_d	8	const	1	100	Using index
```

## b 빼기

```
explain
select * from TEST
  where a = 1
  and c = 200
  and d < now();

결과 ---------------------------------------------------------------
1	SIMPLE	TEST		ref	idx_test_a_b_c_d	idx_test_a_b_c_d	8	const	1	5	Using where; Using index
```

## b, d 빼기

```
explain
select * from TEST
  where a = 1
  and c = 200;

결과 ---------------------------------------------------------------
1	SIMPLE	TEST		ref	idx_test_a_b_c_d	idx_test_a_b_c_d	8	const	1	10	Using where; Using index
```

## a 빼기
```
explain
select * from TEST
  where b = '2000000000'
  and c = 200;
  and d < now();

결과 ---------------------------------------------------------------
1	SIMPLE	TEST		index	idx_test_a_b_c_d	idx_test_a_b_c_d	99		372	1	Using where; Using index
```
type 이 index 가 되어 버렸다. 나빠졌다.

## a, c 순서바꾸기
```
explain
select * from TEST
  where c = 200
  and b = '2000000000'
  and a = 200
  and d < now();

결과 ---------------------------------------------------------------
1	SIMPLE	TEST		ref	idx_test_a_b_c_d	idx_test_a_b_c_d	99	const,const,const,const	1	100	Using index
```
