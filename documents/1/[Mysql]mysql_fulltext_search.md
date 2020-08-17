

1) creation table

```mysql
mysql> CREATE TABLE opening_lines (
       id INT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
       opening_line TEXT(500),
       author VARCHAR(200),
       title VARCHAR(200)
       ) ENGINE=InnoDB;
```

2) creation full text search index

```mysql
mysql> CREATE FULLTEXT INDEX idx ON opening_lines(opening_line);
Query OK, 0 rows affected, 1 warning (0.19 sec)
Records: 0  Duplicates: 0  Warnings: 1
```

3) create table with full text search index

```mysql
mysql> CREATE TABLE opening_lines (
       id INT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
       opening_line TEXT(500),
       author VARCHAR(200),
       title VARCHAR(200),
       FULLTEXT idx (opening_line)
       ) ENGINE=InnoDB;
```
https://dev.mysql.com/doc/refman/8.0/en/fulltext-boolean.html

n-gram 파서 관련
http://mysqldba.tistory.com/287

alter table

@Query(value = "SELECT * FROM accounts WHERE (COALESCE(first_name,'') LIKE %:firstName% AND COALESCE(last_name,'') LIKE %:lastName%)", nativeQuery = true)
public List<Account> searchByFirstnameAndLastname(@Param("firstName")String firstName,@Param("lastName")String lastName);




---
http://mysqldba.tistory.com/287

N-gram
