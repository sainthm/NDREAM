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

<br>