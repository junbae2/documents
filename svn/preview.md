설치하기
========

OSX
---

> svn 을 설치한다

```
$ brew install --universal subversion
 - 이때 homebrew를 사용했는데, homebrew가 설치되어 있지 않다면, 설치부
 - /usr/local/bin/svn
```

> root 디렉토리를 결정한다

```
$ mkdir /Users/{myuser}/{svn_root}
$ cd /Users/{myuser}/{svn_root}
```

> repository 를 생성한다.

```
$ svnadmin create --fs-type fsfs {repository}
 - 타입은 변경가능
 - 위치에 repository 가 생성된것을 확인
```

> 디렉토리권한을 수정한다

```
$ chmod -R g+w {repository}
```

> 설정변경

-	서버설정
-	{repository}/conf/svnserve.conf

```
[general]
...
anon-access = none # 익명사용자 접근불가
auth-access = write # 인증사용자 쓰기가능
...
password-db = passwd # 상용자/비번 등록파일 passwd
...
authz-db = authz # 그룹별 권한관리파일 authz
...
```

-	계정/비밀번호
-	{repository}/conf/passwd

```
...
[users]
ward.b=ward.b
...
```

-	그룹/권한
-	{repository}/conf/authz

```
[groups]
...
read=user1, user2, user3
read_write=user5
...
[{repository}:/]
@read=r
@read_write=rw
```

> 재시작

```
$ ps -ef | grep svnserve
- 종료

$ svnserve -d -r ~/svn/ --listen-port 3690
- 시작
```
