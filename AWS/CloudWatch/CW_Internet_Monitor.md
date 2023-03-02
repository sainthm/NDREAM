# Amazon CloudWatch Internet Monitor 미리 보기 – 인터넷 성능에 대한 종단 간 가시성 제공

- Main URL: [Amazon CloudWatch Internet Monitor 미리 보기 – 인터넷 성능에 대한 종단 간 가시성 제공](https://aws.amazon.com/ko/blogs/korea/cloudwatch-internet-monitor-end-to-end-visibility-into-internet-performance-for-your-applications/)


<br>

- 인터넷 문제가 애플리케이션 성능과 가용성에 어떻게 영향을 미칠 수 있는지에 대한 가시성을 제공
- 인터넷 문제를 진단하는 데 걸리는 시간을 며칠에서 몇 분으로 줄일 수 있음
- 애플리케이션 코드를 계측할 필요가 없음
- AWS Management Console의 CloudWatch 콘솔에서 서비스를 활성화하고 즉시 사용 가능!!


<br>

## PoC

- 콘솔 접속 위치: CloudWatch Dashboard > Application monitoring 탭 > Internet Monitor 선택

<br>

### Add resources 항목에 추가 가능한 리소스들

- VPC
  - Specific 한 VPC 선택 가능
- CloudFront
  - Distribution
- Workspaces
  - 해당 내용은 AWS 가이드에 포함


<br>

### Maximun city-networks to monitor 항목

- Monitoring limit
  - Up to 500,000 city-networks
  - Up to 50,000 city-networks
  - Up to 5,000 city-networks
  - Up to 100 city-networks
  - Other - please specify (1 to 500,000)

<br>

## 설정에 대한 AWS 가이드

- Internet monitor를 배포한 순간부터, Internet Monitor에서는 자동으로 애플리케이션의 리소스 로그에 따라 데이터를 수집하기 시작
-  VPC Flow Logs, CloudFront 로그 또는 기타 로그 유형을 활성화하거나 비용을 지불할 필요가 없습니다.