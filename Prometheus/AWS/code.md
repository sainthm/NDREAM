# Service Account role 부분만 확인!!

# This script creates an Amazon VPC, an Amazon EKS cluster, and service account role.

WORKLOAD_REGION='eu-west-1'
WORKLOAD_REGION_AZ1='eu-west-1a'
WORKLOAD_REGION_AZ2='eu-west-1b'
WORKLOAD_CIDR_BLOCK='192.168.32.0/19'
WORKLOAD_CIDR_BLOCK_SUBNET1='192.168.32.0/22'
WORKLOAD_CIDR_BLOCK_SUBNET2='192.168.37.0/22'
SERVICE_ACCOUNT_NAME=amp-iamproxy-ingest-service-account

WORKLOAD_ACCOUNT_ID=$(aws sts get-caller-identity | jq .Account -r)

aws configure set region $WORKLOAD_REGION

# Setup VPC
WORKLOAD_VPCID=$(aws ec2 create-vpc \
  --cidr-block ${WORKLOAD_CIDR_BLOCK} | jq .Vpc.VpcId -r)
aws ec2 create-tags --resources $WORKLOAD_VPCID \
  --tags Key=Name,Value=EKS-AMP-Workload
aws ec2 modify-vpc-attribute --vpc-id $WORKLOAD_VPCID \
  --enable-dns-hostnames
aws ec2 modify-vpc-attribute --vpc-id $WORKLOAD_VPCID \
  --enable-dns-support

# Create an internet gateway and attaches it to the VPC
WORKLOAD_IGW=$(aws ec2 create-internet-gateway | jq -r '.InternetGateway.InternetGatewayId')
aws ec2 attach-internet-gateway \
    --internet-gateway-id $WORKLOAD_IGW \
    --vpc-id $WORKLOAD_VPCID

# Create subnets and configure route table
WORKLOAD_RT=$(aws ec2 describe-route-tables \
    --query 'RouteTables[].RouteTableId' \
    --filters Name=vpc-id,Values=$WORKLOAD_VPCID \
    --output text)
aws ec2 create-route --route-table-id $WORKLOAD_RT \
    --gateway-id $WORKLOAD_IGW \
    --destination-cidr-block '0.0.0.0/0'
    
WORKLOAD_SUBNET1=$(aws ec2 create-subnet --vpc-id $WORKLOAD_VPCID \
    --cidr-block $WORKLOAD_CIDR_BLOCK_SUBNET1 \
    --availability-zone $WORKLOAD_REGION_AZ1 | jq -r '.Subnet.SubnetId')
WORKLOAD_SUBNET2=$(aws ec2 create-subnet --vpc-id $WORKLOAD_VPCID \
    --cidr-block $WORKLOAD_CIDR_BLOCK_SUBNET2 \
    --availability-zone $WORKLOAD_REGION_AZ2 | jq -r '.Subnet.SubnetId')
aws ec2 modify-subnet-attribute --map-public-ip-on-launch \
    --subnet-id $WORKLOAD_SUBNET1
aws ec2 modify-subnet-attribute --map-public-ip-on-launch \
    --subnet-id $WORKLOAD_SUBNET2
aws ec2 associate-route-table --route-table-id $WORKLOAD_RT \
    --subnet-id $WORKLOAD_SUBNET1
aws ec2 associate-route-table --route-table-id $WORKLOAD_RT \
    --subnet-id $WORKLOAD_SUBNET2

# Amazon EKS cluster creation
eksctl create cluster workload \
  --vpc-public-subnets $WORKLOAD_SUBNET1,$WORKLOAD_SUBNET2
eksctl utils associate-iam-oidc-provider --cluster=workload --approve


CLUSTER_OIDC_PROVIDER=$(aws eks describe-cluster --name workload \
  --query "cluster.identity.oidc.issuer" \
  --output text | sed -e "s/^https:\/\///")
cat > trustPolicy.json << EOF
{
  "Version": "2012-10-17",
  "Statement": [    
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::${WORKLOAD_ACCOUNT_ID}:oidc-provider/${CLUSTER_OIDC_PROVIDER}"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
        "${CLUSTER_OIDC_PROVIDER}:sub": "system:serviceaccount:prometheus:amp-iamproxy-ingest-service-account"
        }
      }
    }
  ]
}
EOF

# Create an IAM role for Kubernetes service account
aws iam create-role \
    --role-name EKS-AMP-ServiceAccount-Role \
    --assume-role-policy-document file://trustPolicy.json \
    --description "IAM role to be used by a K8s service account to assume cross account role"


echo "export WORKLOAD_ACCOUNT_ID=${WORKLOAD_ACCOUNT_ID}" >> delete.env
echo "export WORKLOAD_VPCID=${WORKLOAD_VPCID}" >> delete.env
echo "export WORKLOAD_IGW=${WORKLOAD_IGW}" >> delete.env
echo "export WORKLOAD_RT=${WORKLOAD_RT}" >> delete.env
echo "export WORKLOAD_SUBNET1=${WORKLOAD_SUBNET1}" >> delete.env
echo "export WORKLOAD_SUBNET2=${WORKLOAD_SUBNET2}" >> delete.env
echo "export WORKLOAD_RT=${WORKLOAD_RT}" >> delete.env
echo "export CLUSTER_NAME=workload" >> delete.env
echo "export ROLE_NAME=EKS-AMP-ServiceAccount-Role" >> delete.env

echo -e "# Copy this into your Central account terminal:\n\nexport WORKLOAD_ACCOUNT_ID=$WORKLOAD_ACCOUNT_ID\n"



## EC2 sample yaml
global:
  scrape_interval: 15s
  external_labels:
    monitor: 'prometheus'

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:8000']

remote_write:
  -
    url: https://aps-workspaces.my-region.amazonaws.com/workspaces/my-workspace-id/api/v1/remote_write
    queue_config:
        max_samples_per_send: 1000
        max_shards: 200
        capacity: 2500
    sigv4:
         region: my-region