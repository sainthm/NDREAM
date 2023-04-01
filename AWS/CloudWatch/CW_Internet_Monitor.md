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
- VPC Flow Logs, CloudFront 로그 또는 기타 로그 유형을 활성화하거나 비용을 지불할 필요 X



<br>

## 설정된 사항에 대해, 확인한 사항

<br>

### Overview

- Health scores
  - 5분 단위로 찍히는 것으로 보임
  - UTC 및 로컬 타임존은 당연히 지원
  - 이슈가 생기면 빨간 영역으로 찍힘
  - 지원 항목은 아래 세 개로 보임
    - Availability score
    - Performance score
    - Helath event score < 95%
- Internet traffic overview
  - 위에서 빨갛게 찍힌 영역에 대해, 세계지도위에 이슈 마크로 출력
  - 마우스 커서를 올려보면, Impact on traffic 항목이 출력 (영향도가 퍼센트로 나옴, 확인한 사항은 17.21%, 연계로 Performance score는 82.56% 으로 감소)
- Health events
  - 해당 항목에서는 영향에 대해, 아래의 컬럼에 대한 값을 보여줌
    - Client location (ex. Kansas City, Missouri, United States)
    - Client network (ex. 특정 플랫폼)
    - Service location (ex. us-west-2 [리전])
    - Traffic impact (ex. 17.21%)
    - Impact type (ex. Internet performance issue)
- Select an event location to see more details
  - 위의 항목에서 Client location 의 특정 지역을 클릭하면 출력
  - 아래의 항목에 대해, 값이 출력됨
    - Past event (Started at / Ended)
    - Event duration (ex. 00h 05m 00s)
    - Impact to overall traffic (ex. 17.21%)
    - Imapct at selected client location (ex. 100%)
    - Imapairment alaysis
      - Event status (ex. resloved)
      - Imparment tpye (ex. Performance)

<br>

### Historical explorer

- 출력 결과에 대한 추세를 이해하고 해당 위치 및 네트워크 제공업체와 관련된 이전 데이터 참조 가능
- 최대 18개월 전까지 아래의 데이터 확인 가능
  - Performance score
  - Availability score
  - Bytes transferred
  - Round-trip time (RTT): 말 그대로 왕복 (Request to return a response)
- All events
  - Lists health events for your application. For example, traffic impact of N% for a health event for Location C and ISP A means that N% of traffic from CloudFront towards Location C over ISP A is experiencing an availability or performance drop. 
  - (Note that when service location is a Region, measurements and events represent connectivity at a Regional level, **not between end-user locations** and Availability Zones.)


<br>


### Traffic insights

- Traffic insights filter
  - 아래의 항목에 대해, 필터링 설정 가능
    - AWS location
    - Country
    - Subdivision
    - Metro
    - City
    - Network providers
- Overall traffic
  - 아래의 항목에 대해, 데이터 출력
    - Total traffic (bytes)
    - Total traffic in (bytes)
    - Total traffic out (bytes)
    - Average TTFB (Time To First Byte: HTTP 요청을 했을 때, 처음 byte[정보]가 브라우저에 도달하는 시간)
    - Availability score
    - Performance score
- City-networks traffic monitoring
  - 아래의 두 개의 값에 대한 그래프 출력
    - CityNetworksFor100PercentTraffic (100%에 대한 값을 출력)
    - CityNetworksMonitored (실제 값을 출력)
- Top 10 client locations
  - 말 그대로 Top 10 출력 (반대로 Worst 10도 출력 가능)
  - Sorting 조건은 아래의 세 가지 이며, 상세한 설정은 조건 밑에 서술
    - Sorting field
      - Total traffic
      - Total traffic in
      - Total traffic out
      - Average TTFB
      - Availability score
      - Performance score
    - Granularity (생전 처음 보는 단어, 사전 상 의미는 "낟알 모양, 입상(粒狀); 입도(粒度)" 라고 나오지만 이렇게 쓰고 **세부사항** 혹은 **부분** , **명세** 로 읽어도 문제는 없어보임)
      - City
      - Subdivision
      - Country
      - Metro
    - Sorting order
      - Highest to lowest
      - Lowest to highest
- Traffic optimization suggestions
  - Top 네트워크에 대해, 셋업 추천
  - EC2 / CloudFront 두 가지 서비스에 대해, include 선택 가능

<br>

### Monitor details

- 생성한 CW Internet Monitor 에 대한, 설정 정보를 확인 가능
- Monitor details
  - Monitor name
  - Monitor ARN
  - Status info
  - Monitoring limit
  - Status
  - Created at
  - Modified at
- Monitored resources
  - Resource ID
  - Resource name
  - Service (EC2 / CloudFront)
  - AWS Region (CloudFront 의 경우, 공란)
  - Account ID
- Tags
  - 설정한 태그 확인 가능


<br>