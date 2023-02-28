# AWS CLI 환경에서의 role-switching (Cross-account 환경)

- 파일 생성일: 2023.02.28

<br>

## 방법 (크게 두 가지)

1. **mfa_serial** 활용 방법
2. **assume-role** 활용 방법

<br>

## 준비물:

- MFA Serial
  - Serial 값 확인 방법(메인 로그인 계정): AWS >> IAM >> Users >> Specific user >> Security credentials >> MFA >> Identifier
  - arn:aws:iam::123412341234:mfa/xxxx (xxxx 항목에는 다양한 값 가능: Authy 등)
- Role ARN

<br>

## 환경정보:

<br>

## 작업사항:


1. **mfa_serial** 활용 방법 (금방 끝남, 설정이 간단)


```bash
# vim ~/.aws/credentias 에 아래 내용 추가
# 기존에 [default] profile 은 존재해야함 >> 이 때 사용하는 [default] 프로파일에는 메인 로그인 계정 IAM User의 Key set 값 필요

[원하는 프로필 이름]
source_profile = default
mfa_serial = arn:aws:iam::123412341234:mfa/xxxx
role_arn = arn:aws:iam::432143214321:role/role
```


2. **assume-role** 활용 방법 (명령어가 김, 복붙이 귀찮을 수 있음, 코드 짜면 어느정도 사람손 안타게 구현 가능)

```bash
aws sts assume-role --role-arn arn:aws:iam::yyyyyyyyyyyy:role/역할명 --role-session-name 아무거나 --serial-number arn:aws:iam::xxxxxxxxxxxx:mfa/사용자명 --token-code MFA인증번호

```

<br>


```json
{
  "Credentials": {
    "AccessKeyId": "ZZZZZZZZZZZZZZZZZZZZZ",
    "SecretAccessKey": "YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY",
    "SessionToken": "시크릿 액세스 키를 초월하는 엄청나게 긴 문자열, 체감상 10배는 더 김",
    "Expiration": "2020-07-28T12:15:05+00:00"
  },
  "AssumedRoleUser": {
    "AssumedRoleId": "OOOOOOOOOOOOOOOOOOOOO:세션명",
    "Arn": "arn:aws:sts::yyyyyyyyyyyy:assumed-role/역할명/세션명"
  }
}
```

<br>


```bash
export AWS_ACCESS_KEY_ID=ZZZZZZZZZZZZZZZZZZZZZ (위에 출력된 액세스 키)
export AWS_SECRET_ACCESS_KEY=YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY (위에 출력된 액세스 키)
export AWS_SESSION_TOKEN=시크릿 액세스 키를 초월하는 엄청나게 긴 문자열, 체감상 10배는 더 김 (위에 출력된 세션 토큰값)
```


<br>