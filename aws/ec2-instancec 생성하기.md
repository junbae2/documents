AMI
---

보안그룹
--------

### VPC ID 확인

```
aws ec2 describe-vpcs
```

### 보안그룹생성

```
$ aws ec2 create-security-group \
--group-name HelloWorld \
--description "Hello World Demo" \
--vpc-id vpc-7d021915
{
    "GroupId": "xg-xxxxxxxx"
}
```

#### 인바운드허용

port: 22, 3000

```
$ aws ec2 authorize-security-group-ingress \
--group-id xg-xxxxxxxx \
--protocol tcp \
--port 22 \
--cidr 0.0.0.0/0

$ aws ec2 authorize-security-group-ingress \
--group-id xg-xxxxxxxx \
--protocol tcp \
--port 3000 \
--cidr 0.0.0.0/0
```

#### 결과확인

```
$ aws ec2 describe-security-groups --group-id xg-xxxxxxxx --output text
```

### SSH키 생성하기

```
$ aws ec2 create-key-pair --key-name EffectiveDevOpsAWS
{
    "KeyFingerprint": "XXXXXX",
    "KeyMaterial": "-----BEGIN RSA PRIVATE KEY-----\nXXXXX\n-----END RSA PRIVATE KEY-----",
    "KeyName": "EffectiveDevOpsAWS"
}

$echo -e "KeyMaterial 의 값" > 저장파일명.pem

```

### EC2 인스턴스 띄우기

먼저 ec2 이미지를 확인하자

```
$ aws ec2 describe-images --filters "Name=description, Values=Amazon Linux AMI * x86_64 HVM GP2" --query 'Images[*].[CreationDate, Description, ImageId]' --output text | sort -k 1 | tail
```

생성 > 실행

```
$ aws ec2 run-instances \
--instance-type t2.micro \
--key-name EffectiveDevOpsAWS \
--security-group-ids xg-xxxxxxxx \
--image-id ami-xx--2xxx1--cxx
```

상태보기

```
$ aws ec2 describe-instance-status --instance-ids i-xxx82xx1---xxx
```

공인IP확인하기

```
$ aws ec2 describe-instances \
--instance-ids i-000xxxf---x \
--query "Reservations[*].Instances[*].PublicIpAddress"
```
