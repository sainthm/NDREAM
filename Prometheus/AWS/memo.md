# memo.md

## 실제 구축하면서 작업 중 필요했던 내용에 대한 간단한 메모장

aws eks describe-cluster --name eks-cluster-xxx --query "cluster.roleArn"

"arn:aws:iam::xxxxxxxxx:role/eksctl-eks-cluster-xx-cluster-ServiceRole-1Q6LCxxxxxxxx"

<br>

### 1단계: 새 Helm 차트 리포지토리를 추가합니다

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add kube-state-metrics https://kubernetes.github.io/kube-state-metrics
helm repo update
```

<br>

### 2단계: Prometheus 네임스페이스 생성

```bash
kubectl create namespace prometheus-namespace
```

<br>

### 3단계: 서비스 계정에 대한 IAM 역할 설정

- 관련링크: [서비스 계정에 대한 IAM 역할](https://docs.aws.amazon.com/ko_kr/prometheus/latest/userguide/set-up-irsa.html#set-up-irsa-ingest)

#### Process

1. 역할 생성을 위한 계정 로그인
2. 스크립트 준비
    - For the ingestion of metrics
    - For the querying of metrics
3. 스크립트 실행을 통한 역할 생성
