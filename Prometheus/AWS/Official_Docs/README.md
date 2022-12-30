# [공식사용자가이드]Amazon Managed Service for Prometheus

- 노션 페이지 생성일: 2022.12.27

<br>

## Main docs URL:[Amazon Managed Service for Prometheus Documentation](https://docs.aws.amazon.com/prometheus/?icmpid=docs_homepage_mgmtgov)

<br>

## User guide URL:

[What is Amazon Managed Service for Prometheus?](https://docs.aws.amazon.com/prometheus/latest/userguide/what-is-Amazon-Managed-Service-Prometheus.html)

- [공식 소개 페이지 글](https://github.com/sainthm/NDREAM/tree/main/Prometheus/AWS/Official_Introduction)과 겹치는 내용은 생략

<br>

## Supported Regions:

- US East (Ohio)
- US East (N. Virginia)
- US West (Oregon)
- Asia Pacific (Singapore)
- Asia Pacific (Sydney)
- Asia Pacific (Tokyo)
- Europe (Frankfurt)
- Europe (Ireland)
- Europe (London)
- Europe (Stockholm)

<br>

### Pricing:

- Ingestion and storage of metrics
- Storage charge (Based on the compressed size of metric samples and metadata)

[Amazon Managed Service for Prometheus Pricing | Fully Managed Prometheus | Amazon Web Services](https://aws.amazon.com/ko/prometheus/pricing/)

<br>

# Getting started:

### 기본 작업:

- 계정 생성
- IAM 사용자 및 key 생성

<br>

## 프로메테우스 작업:

### 개요:

- Workspace 생성
- Workspace에 프로메테우스 메트릭 ingest
- 프로메테우스 메트릭 query

<br>

## Create a workspace:

### Workspace:

- A workspace is a logical space dedicated to the storage and querying of Prometheus metrics.


<br>

### AWS CLI를 활용한 설치 방법

```python
# Workspace alias 설정
# Alias가 유니크할 필요는 X
# 내부적으로 고유의 workspace ID를 부여받음

# Workspace 생성 명령어
aws amp create-workspace [ --alias my-first-workspace]

# Workspace 상태 확인 명령어
aws amp describe-workspace --workspace-id my-workspace-id
```

<br>

## Ingest Prometheus metrics to the workspace:

### Remote write:

Ingesting metrics is done using Prometheus remote write. 

Remote write enables the sending of samples to a remote storage destination.

[Configuration | Prometheus](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#remote_write)


<br>

### Secure the ingestion of your metrics

- Using AWS PrivateLink with Amazon Managed Service for Prometheus
    - Public internet endpoint
    - VPC endpoint through AWS PrivateLink
        
        [Using Amazon Managed Service for Prometheus with interface VPC endpoints](https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-and-interface-VPC.html)
        
- Authentication and authorization
    - IAM roles (Prometheus server, Grafana server)
    - AWS Signature Version 4 (AWS SigV4)
        
        [Signing AWS API requests](https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html)
        

<br>

### **Ingestion methods and how to set them up**

- Ingest metrics from a Prometheus server
- Use AWS Distro for OpenTelemetry

<br>

> 고가용성을 위해, 다수의 프로메테우스 서버(인스턴스)를 생성하여 하나의 AMP workspace로 보낼 경우, deduplication 작업 필요 
> (겹치는 메트릭에 대해서는 중복으로 과금)

<br>

[Send high-availability data with Prometheus or the Prometheus Operator](https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-ingest-dedupe.html)


<br>

## **Quick Start: Set up ingestion from a new Prometheus server using Helm:**

- 시나리오:
    - Amazon EKS cluster 환경
    - default 설정으로 메트릭 전달
    - Helm CLI 3.0 or later

<br>

### Step:

1. Add new Helm chart repo
2. Create a Prometheus namespace
3. Set up IAM roles for service accounts
4. Set up the new server and start ingesting metrics

```yaml
# yaml 파일

# IAM_PROXY_PROMETHEUS_ROLE_ARN >> 변경 필요
# WORKSPACE_ID >> 변경 필요
# AWS_REGION >> 변경 필요

## The following is a set of default values for prometheus server helm chart which enable remoteWrite to AMP
## For the rest of prometheus helm chart values see: https://github.com/prometheus-community/helm-charts/blob/main/charts/prometheus/values.yaml
##
serviceAccounts:
  server:
    name: amp-iamproxy-ingest-service-account
    annotations: 
      eks.amazonaws.com/role-arn: ${IAM_PROXY_PROMETHEUS_ROLE_ARN}
server:
  remoteWrite:
    - url: https://aps-workspaces.${AWS_REGION}.amazonaws.com/workspaces/${WORKSPACE_ID}/api/v1/remote_write
      sigv4:
        region: ${AWS_REGION}
      queue_config:
        max_samples_per_send: 1000
        max_shards: 200
        capacity: 2500
```

<br>

```bash
# Prometheus server 생성 명령어

# prometheus-chart-name >> Prometheus release name 으로 변경 필요
# prometheus-namespace >> Prometheus namespace 값으로 변경 필요
helm install prometheus-chart-name prometheus-community/prometheus -n prometheusnamespace \
-f my_prometheus_values_yaml
```

<br>

### 유저가이드 내에 있는 모든 실행 예시

- **Set up ingestion from a new Prometheus server using Helm**
- **Set up ingestion from an existing Prometheus server in Kubernetes on EC2**
- **Set up ingestion from an existing Prometheus server in Kubernetes on Fargate**
- **Set up metrics ingestion using AWS Distro for Open Telemetry on an Amazon Elastic Kubernetes Service cluster**
- **Set up metrics ingestion from Amazon ECS using AWS Distro for Open Telemetry**
- **Set up metrics ingestion from an Amazon EC2 instance using remote write**

<br>

### **Send high-availability data with Prometheus or the Prometheus Operator**

- Deduplication 셋팅
- 고가용성을 위한 설정 (+이중과금을 막기 위한 설정)
- 설정 적용 시, AMP는 하나의 인스턴스를 leader replica로 설정하고 데이터 샘플을 해당 레플리카를 통해서만 ingest 함
- Leader replica가 30초 동안 AMP로 데이터 샘플을 보내지 않으면, AMP는 자동으로 다른 프로메테우스 인스턴스를 리더 레플리카로 선정하고 새로운 리더로 부터 데이터를 ingest

<br>

- 설정을 위해서 붙여야하는 라벨명: ```cluster```
- 각각의 레플리카를 그룹에서 구별하기 위해 붙이는 라벨명: ```__replica__```
- De-duplication 작업을 위해서는 위의 두 개의 라벨이 모두 필요: cluster , ```__replica__```


<br>


### **Send high-availability data to Amazon Managed Service for Prometheus with the Prometheus community Helm chart**

- 헬름 차트를 이용한 고가용성 설정 방법
- external_labels 설정을 통해 HA 구성임을 명시적으로 표현 필요
- Multiple replicas 구성을 위해서는 명시적으로 다른 레플리카 값으로 여러번 배포해야함
  - replica 라벨에 대해 auto-set 적용을 위해서는 prometheus-operator Helm chart 활용 필요

