AWS CLI 설치/설정
=================

mac os
------

### 설치

#### 1. pip 설치

#### 2. awscli 설치

```
$ sudo pip install --upgrade --user awscli
```

### 설정

awscli 를 사용하기 위해서는, 사용자 등록이 먼저 진행되어야 한다.

#### 1. 엑세스key 와 보안 key 추출

```
$ more credentials.csv
User name,Password,Access key ID,Secret access key,Console login link
ju--a-2,,AKIA---LIB4---2EYPQ,h--SD9s---R93RI/r---Oe---OShelhbU--IOH,https://54010---7354.signin.aws.amazon.com/console
```

credentials.csv는 IAM 사용자 생성과정에서 다운로드 받을수 있다.[AWS 계정의 IAM 사용자 생성](https://docs.aws.amazon.com/ko_kr/IAM/latest/UserGuide/id_users_create.html#id_users_create_console) 을 참고 하자.

#### 2. 계정구성

```
$ aws configure
AWS Access Key ID [None]: AKIA---LIB4---2EYPQ
AWS Secret Access Key [None]: h--SD9s---R93RI/r---Oe---OShelhbU--IOH
Default region name [None]: ap-northeast-2
Default output format [None]:
```

끝이다. 아래와 같이 생성된 사용자를 확인할수 있다.

```
$ aws iam list-users
{
    "Users": [
        {
            "Path": "/",
            "UserName": "ju--a-2",
            "UserId": "AKIA---LIB4---2EYPQ",
            "Arn": "arn:aws:iam::54010---7354:user/ju--a-2",
            "CreateDate": "2018-09-03T11:13:29Z",
            "PasswordLastUsed": "2018-09-05T07:42:07Z"
        }
    ]
}
```

이제 CLI 를 사용할수 있다.
