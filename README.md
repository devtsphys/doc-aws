# AWS Services Reference Cheat Sheet

## Table of Contents

1. [Compute Services](#compute-services)
1. [Storage Services](#storage-services)
1. [Database Services](#database-services)
1. [Networking & Content Delivery](#networking--content-delivery)
1. [Application Deployment](#application-deployment)
1. [Monitoring & Logging](#monitoring--logging)
1. [Security & Identity](#security--identity)
1. [Developer Tools & Testing](#developer-tools--testing)
1. [Analytics & Data Processing](#analytics--data-processing)
1. [Messaging & Integration](#messaging--integration)

-----

## Compute Services

### EC2 (Elastic Compute Cloud)

**Virtual servers in the cloud**

**Instance Types:**

- **t2/t3**: Burstable, general purpose (t2.micro free tier eligible)
- **m5/m6**: Balanced compute, memory, networking
- **c5/c6**: Compute optimized
- **r5/r6**: Memory optimized
- **p3/p4**: GPU instances for ML/AI

**Key Concepts:**

- **AMI (Amazon Machine Image)**: Template for EC2 instances
- **Security Groups**: Virtual firewall (stateful)
- **Key Pairs**: SSH authentication
- **Auto Scaling Groups**: Automatic scaling based on demand
- **Placement Groups**: Control instance placement (cluster, spread, partition)

**CLI Examples:**

```bash
# Launch instance
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t2.micro \
  --key-name MyKeyPair \
  --security-group-ids sg-903004f8

# Stop instance
aws ec2 stop-instances --instance-ids i-1234567890abcdef0

# Create AMI
aws ec2 create-image --instance-id i-1234567890abcdef0 --name "MyServer"

# Describe instances
aws ec2 describe-instances --filters "Name=instance-state-name,Values=running"
```

**Best Practices:**

- Use Auto Scaling for high availability
- Reserve instances for steady workloads (up to 75% savings)
- Spot instances for fault-tolerant workloads (up to 90% savings)
- Use IAM roles instead of access keys

-----

### Lambda

**Serverless compute - run code without managing servers**

**Key Features:**

- Pay per invocation (first 1M requests/month free)
- Max execution time: 15 minutes
- Memory: 128 MB to 10 GB
- Supports: Node.js, Python, Java, Go, .NET, Ruby, Custom Runtime

**Event Sources:**

- API Gateway, S3, DynamoDB, Kinesis, SNS, SQS, EventBridge, CloudWatch Events

**CLI Examples:**

```bash
# Create function
aws lambda create-function \
  --function-name myFunction \
  --runtime python3.9 \
  --role arn:aws:iam::123456789012:role/lambda-role \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip

# Invoke function
aws lambda invoke \
  --function-name myFunction \
  --payload '{"key":"value"}' \
  output.txt

# Update function code
aws lambda update-function-code \
  --function-name myFunction \
  --zip-file fileb://new-function.zip

# Add environment variables
aws lambda update-function-configuration \
  --function-name myFunction \
  --environment Variables={DB_HOST=localhost,DB_PORT=5432}
```

**Lambda Layers:**

- Share code/libraries across functions
- Max 5 layers per function
- Up to 250 MB total (unzipped)

**Best Practices:**

- Keep functions small and focused
- Use environment variables for configuration
- Enable CloudWatch Logs for debugging
- Use VPC only when necessary (adds cold start time)
- Optimize package size for faster cold starts

-----

### ECS (Elastic Container Service)

**Container orchestration platform**

**Launch Types:**

- **Fargate**: Serverless, AWS manages infrastructure
- **EC2**: You manage EC2 instances

**Key Components:**

- **Task Definition**: Blueprint (Docker image, CPU, memory, ports)
- **Service**: Runs and maintains tasks
- **Cluster**: Logical grouping of tasks/services

**CLI Examples:**

```bash
# Create cluster
aws ecs create-cluster --cluster-name my-cluster

# Register task definition
aws ecs register-task-definition --cli-input-json file://task-definition.json

# Create service
aws ecs create-service \
  --cluster my-cluster \
  --service-name my-service \
  --task-definition my-task:1 \
  --desired-count 2 \
  --launch-type FARGATE

# Update service
aws ecs update-service \
  --cluster my-cluster \
  --service my-service \
  --desired-count 3
```

**Task Definition Example (JSON):**

```json
{
  "family": "web-app",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "containerDefinitions": [{
    "name": "web",
    "image": "nginx:latest",
    "portMappings": [{
      "containerPort": 80,
      "protocol": "tcp"
    }]
  }]
}
```

-----

### EKS (Elastic Kubernetes Service)

**Managed Kubernetes platform**

**Key Features:**

- Certified Kubernetes conformant
- Managed control plane
- Auto-scaling with Cluster Autoscaler
- Integration with AWS services (ALB, EBS, IAM)

**Setup Commands:**

```bash
# Create cluster
eksctl create cluster \
  --name my-cluster \
  --region us-west-2 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 3

# Update kubeconfig
aws eks update-kubeconfig --name my-cluster --region us-west-2

# Deploy application
kubectl apply -f deployment.yaml

# Scale deployment
kubectl scale deployment/my-app --replicas=5
```

-----

### Elastic Beanstalk

**PaaS for application deployment**

**Supported Platforms:**

- Java, .NET, PHP, Node.js, Python, Ruby, Go, Docker

**Key Features:**

- Automatic provisioning (EC2, RDS, Load Balancer, Auto Scaling)
- Blue/green deployments
- Rolling updates
- Health monitoring

**CLI Examples:**

```bash
# Initialize application
eb init -p python-3.8 my-app --region us-east-1

# Create environment
eb create production-env

# Deploy application
eb deploy

# View status
eb status

# SSH into instance
eb ssh

# View logs
eb logs
```

**Configuration (.ebextensions):**

```yaml
option_settings:
  aws:elasticbeanstalk:environment:
    EnvironmentType: LoadBalanced
  aws:autoscaling:asg:
    MinSize: 2
    MaxSize: 6
  aws:elasticbeanstalk:environment:process:default:
    HealthCheckPath: /health
```

-----

## Storage Services

### S3 (Simple Storage Service)

**Object storage with 99.999999999% durability**

**Storage Classes:**

- **S3 Standard**: Frequent access, low latency
- **S3 Intelligent-Tiering**: Auto cost optimization
- **S3 Standard-IA**: Infrequent access, lower cost
- **S3 One Zone-IA**: Single AZ, lower cost
- **S3 Glacier Instant Retrieval**: Archive, millisecond access
- **S3 Glacier Flexible Retrieval**: Archive, minutes to hours
- **S3 Glacier Deep Archive**: Lowest cost, 12-hour retrieval

**Key Features:**

- Versioning (protect from accidental deletion)
- Lifecycle policies (auto transition/delete)
- Replication (same-region or cross-region)
- Encryption (SSE-S3, SSE-KMS, SSE-C)
- Access control (Bucket policies, ACLs, IAM)

**CLI Examples:**

```bash
# Create bucket
aws s3 mb s3://my-unique-bucket-name

# Upload file
aws s3 cp file.txt s3://my-bucket/

# Upload directory (recursive)
aws s3 sync ./local-dir s3://my-bucket/remote-dir/

# Download file
aws s3 cp s3://my-bucket/file.txt ./

# List objects
aws s3 ls s3://my-bucket/

# Delete object
aws s3 rm s3://my-bucket/file.txt

# Enable versioning
aws s3api put-bucket-versioning \
  --bucket my-bucket \
  --versioning-configuration Status=Enabled

# Set lifecycle policy
aws s3api put-bucket-lifecycle-configuration \
  --bucket my-bucket \
  --lifecycle-configuration file://lifecycle.json
```

**Lifecycle Policy Example:**

```json
{
  "Rules": [{
    "Id": "Archive old files",
    "Status": "Enabled",
    "Transitions": [{
      "Days": 30,
      "StorageClass": "STANDARD_IA"
    }, {
      "Days": 90,
      "StorageClass": "GLACIER"
    }],
    "Expiration": {
      "Days": 365
    }
  }]
}
```

**S3 Event Notifications:**

- Trigger Lambda, SQS, SNS on object creation/deletion
- Use for automated processing pipelines

-----

### EBS (Elastic Block Store)

**Block storage for EC2 instances**

**Volume Types:**

- **gp3/gp2**: General purpose SSD (default)
- **io2/io1**: Provisioned IOPS SSD (databases)
- **st1**: Throughput optimized HDD (big data)
- **sc1**: Cold HDD (infrequent access)

**Key Features:**

- Snapshots (incremental backups to S3)
- Encryption at rest
- Multi-Attach (io1/io2 only, up to 16 instances)
- Elastic Volumes (resize without downtime)

**CLI Examples:**

```bash
# Create volume
aws ec2 create-volume \
  --size 100 \
  --volume-type gp3 \
  --availability-zone us-east-1a

# Attach volume
aws ec2 attach-volume \
  --volume-id vol-1234567890abcdef0 \
  --instance-id i-1234567890abcdef0 \
  --device /dev/sdf

# Create snapshot
aws ec2 create-snapshot \
  --volume-id vol-1234567890abcdef0 \
  --description "Backup on 2024-01-15"

# Copy snapshot to another region
aws ec2 copy-snapshot \
  --source-region us-east-1 \
  --source-snapshot-id snap-1234567890abcdef0 \
  --destination-region us-west-2
```

-----

### EFS (Elastic File System)

**Managed NFS file system**

**Use Cases:**

- Shared storage across multiple EC2 instances
- Container storage
- Web serving, content management

**Performance Modes:**

- **General Purpose**: Low latency (default)
- **Max I/O**: Higher throughput, higher latency

**Throughput Modes:**

- **Bursting**: Scales with storage size
- **Provisioned**: Fixed throughput regardless of size

**CLI Examples:**

```bash
# Create file system
aws efs create-file-system \
  --performance-mode generalPurpose \
  --throughput-mode bursting \
  --encrypted

# Create mount target
aws efs create-mount-target \
  --file-system-id fs-1234567890abcdef0 \
  --subnet-id subnet-12345678 \
  --security-groups sg-12345678

# Mount on EC2 (from instance)
sudo mount -t efs fs-1234567890abcdef0:/ /mnt/efs
```

-----

### Storage Gateway

**Hybrid cloud storage integration**

**Types:**

- **File Gateway**: NFS/SMB interface to S3
- **Volume Gateway**: iSCSI block storage
- **Tape Gateway**: Virtual tape library

-----

## Database Services

### RDS (Relational Database Service)

**Managed relational databases**

**Supported Engines:**

- MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, Aurora

**Key Features:**

- Automated backups (up to 35 days)
- Multi-AZ for high availability
- Read replicas for scaling reads
- Automated patching
- Encryption at rest and in transit

**CLI Examples:**

```bash
# Create DB instance
aws rds create-db-instance \
  --db-instance-identifier mydb \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --master-username admin \
  --master-user-password MyPassword123 \
  --allocated-storage 20 \
  --backup-retention-period 7

# Create read replica
aws rds create-db-instance-read-replica \
  --db-instance-identifier mydb-replica \
  --source-db-instance-identifier mydb

# Create snapshot
aws rds create-db-snapshot \
  --db-snapshot-identifier mydb-snapshot-20240115 \
  --db-instance-identifier mydb

# Restore from snapshot
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier mydb-restored \
  --db-snapshot-identifier mydb-snapshot-20240115

# Enable Multi-AZ
aws rds modify-db-instance \
  --db-instance-identifier mydb \
  --multi-az \
  --apply-immediately
```

**Connection Example (Python):**

```python
import psycopg2

conn = psycopg2.connect(
    host="mydb.c9akciq32.us-east-1.rds.amazonaws.com",
    database="mydb",
    user="admin",
    password="MyPassword123"
)
```

-----

### DynamoDB

**NoSQL key-value/document database**

**Key Features:**

- Single-digit millisecond latency
- Auto-scaling
- Global tables (multi-region)
- DynamoDB Streams (change data capture)
- Point-in-time recovery

**Capacity Modes:**

- **On-Demand**: Pay per request
- **Provisioned**: Specify RCU/WCU (cheaper for predictable workload)

**CLI Examples:**

```bash
# Create table
aws dynamodb create-table \
  --table-name Users \
  --attribute-definitions \
    AttributeName=UserId,AttributeType=S \
    AttributeName=Email,AttributeType=S \
  --key-schema \
    AttributeName=UserId,KeyType=HASH \
  --global-secondary-indexes \
    IndexName=EmailIndex,Keys=[{AttributeName=Email,KeyType=HASH}],Projection={ProjectionType=ALL},ProvisionedThroughput={ReadCapacityUnits=5,WriteCapacityUnits=5} \
  --billing-mode PROVISIONED \
  --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5

# Put item
aws dynamodb put-item \
  --table-name Users \
  --item '{"UserId": {"S": "123"}, "Name": {"S": "John"}, "Email": {"S": "john@example.com"}}'

# Get item
aws dynamodb get-item \
  --table-name Users \
  --key '{"UserId": {"S": "123"}}'

# Query
aws dynamodb query \
  --table-name Users \
  --index-name EmailIndex \
  --key-condition-expression "Email = :email" \
  --expression-attribute-values '{":email":{"S":"john@example.com"}}'

# Scan (expensive, avoid in production)
aws dynamodb scan --table-name Users
```

**Best Practices:**

- Design for uniform data distribution
- Use partition keys with high cardinality
- Avoid scans, use queries with indexes
- Enable DynamoDB Streams for event-driven architecture

-----

### Aurora

**MySQL/PostgreSQL compatible, 5x faster than standard MySQL**

**Key Features:**

- Up to 15 read replicas
- Storage auto-scales up to 128 TB
- Aurora Serverless for variable workloads
- Global Database (< 1 second RPO)
- Backtrack (rewind DB without restoring)

-----

### ElastiCache

**Managed Redis or Memcached**

**Use Cases:**

- Session storage
- Caching database queries
- Real-time analytics
- Leaderboards

**CLI Examples:**

```bash
# Create Redis cluster
aws elasticache create-cache-cluster \
  --cache-cluster-id my-redis-cluster \
  --cache-node-type cache.t3.micro \
  --engine redis \
  --num-cache-nodes 1

# Create replication group (Redis)
aws elasticache create-replication-group \
  --replication-group-id my-redis-rg \
  --replication-group-description "My Redis Cluster" \
  --cache-node-type cache.t3.medium \
  --num-cache-clusters 3 \
  --automatic-failover-enabled
```

-----

## Networking & Content Delivery

### VPC (Virtual Private Cloud)

**Isolated network in AWS**

**Components:**

- **Subnets**: Public (internet gateway) vs Private
- **Route Tables**: Control traffic routing
- **Internet Gateway**: Connect VPC to internet
- **NAT Gateway**: Allow private subnet internet access
- **VPC Peering**: Connect VPCs
- **VPN/Direct Connect**: Connect on-premises to AWS

**CIDR Examples:**

- 10.0.0.0/16 (65,536 IPs)
- 10.0.1.0/24 (256 IPs)
- 10.0.1.0/28 (16 IPs)

**CLI Examples:**

```bash
# Create VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# Create subnet
aws ec2 create-subnet \
  --vpc-id vpc-1234567890abcdef0 \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a

# Create internet gateway
aws ec2 create-internet-gateway

# Attach IGW to VPC
aws ec2 attach-internet-gateway \
  --vpc-id vpc-1234567890abcdef0 \
  --internet-gateway-id igw-1234567890abcdef0

# Create NAT gateway
aws ec2 create-nat-gateway \
  --subnet-id subnet-1234567890abcdef0 \
  --allocation-id eipalloc-1234567890abcdef0

# Create route table
aws ec2 create-route-table --vpc-id vpc-1234567890abcdef0

# Add route to internet gateway
aws ec2 create-route \
  --route-table-id rtb-1234567890abcdef0 \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id igw-1234567890abcdef0
```

**Security Groups vs NACLs:**

- **Security Groups**: Stateful, instance-level, allow rules only
- **NACLs**: Stateless, subnet-level, allow and deny rules

-----

### ELB (Elastic Load Balancing)

**Distribute traffic across multiple targets**

**Types:**

- **ALB (Application)**: HTTP/HTTPS, Layer 7, path-based routing
- **NLB (Network)**: TCP/UDP, Layer 4, ultra-low latency
- **GLB (Gateway)**: Layer 3, third-party appliances
- **CLB (Classic)**: Legacy, both Layer 4 and 7

**ALB Features:**

- Host-based routing: `api.example.com` → Target Group A
- Path-based routing: `/api/*` → Target Group B
- WebSocket support
- HTTP/2 support
- SSL/TLS termination

**CLI Examples:**

```bash
# Create ALB
aws elbv2 create-load-balancer \
  --name my-alb \
  --subnets subnet-12345678 subnet-87654321 \
  --security-groups sg-12345678 \
  --scheme internet-facing \
  --type application

# Create target group
aws elbv2 create-target-group \
  --name my-targets \
  --protocol HTTP \
  --port 80 \
  --vpc-id vpc-1234567890abcdef0 \
  --health-check-path /health

# Register targets
aws elbv2 register-targets \
  --target-group-arn arn:aws:elasticloadbalancing:... \
  --targets Id=i-1234567890abcdef0 Id=i-0987654321fedcba0

# Create listener
aws elbv2 create-listener \
  --load-balancer-arn arn:aws:elasticloadbalancing:... \
  --protocol HTTP \
  --port 80 \
  --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:...

# Create listener rule (path-based)
aws elbv2 create-rule \
  --listener-arn arn:aws:elasticloadbalancing:... \
  --conditions Field=path-pattern,Values='/api/*' \
  --priority 10 \
  --actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:...
```

-----

### Route 53

**Scalable DNS and domain registration**

**Routing Policies:**

- **Simple**: Single resource
- **Weighted**: Split traffic by percentage
- **Latency**: Route to lowest latency region
- **Failover**: Active-passive failover
- **Geolocation**: Route based on user location
- **Geoproximity**: Route based on resource location
- **Multi-value**: Return multiple IPs (client-side load balancing)

**CLI Examples:**

```bash
# Create hosted zone
aws route53 create-hosted-zone \
  --name example.com \
  --caller-reference $(date +%s)

# Create A record
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch file://change-batch.json
```

**change-batch.json:**

```json
{
  "Changes": [{
    "Action": "CREATE",
    "ResourceRecordSet": {
      "Name": "www.example.com",
      "Type": "A",
      "TTL": 300,
      "ResourceRecords": [{"Value": "192.0.2.1"}]
    }
  }]
}
```

**Health Checks:**

- Monitor endpoint health
- Trigger failover based on health
- SNS notifications on health check failures

-----

### CloudFront

**Content Delivery Network (CDN)**

**Features:**

- Edge locations worldwide (400+)
- HTTPS support
- Custom SSL certificates
- Lambda@Edge for customization
- Origin: S3, ALB, EC2, custom HTTP server

**CLI Examples:**

```bash
# Create distribution
aws cloudfront create-distribution \
  --origin-domain-name my-bucket.s3.amazonaws.com \
  --default-root-object index.html

# Create invalidation (clear cache)
aws cloudfront create-invalidation \
  --distribution-id E1234567890ABC \
  --paths "/*" "/images/*"
```

**Use Cases:**

- Static website hosting (S3 + CloudFront)
- Video streaming
- API acceleration
- DDoS protection (with AWS Shield)

-----

### API Gateway

**Create, publish, and manage APIs**

**Types:**

- **REST API**: RESTful APIs
- **HTTP API**: Lower cost, faster (modern choice)
- **WebSocket API**: Real-time two-way communication

**Features:**

- Lambda integration
- Request/response transformation
- API keys and usage plans
- Throttling and rate limiting
- CORS support
- Custom domains

**CLI Examples:**

```bash
# Create REST API
aws apigateway create-rest-api --name 'My API'

# Get resources
aws apigateway get-resources --rest-api-id abc123

# Create resource
aws apigateway create-resource \
  --rest-api-id abc123 \
  --parent-id xyz789 \
  --path-part users

# Create method
aws apigateway put-method \
  --rest-api-id abc123 \
  --resource-id xyz789 \
  --http-method GET \
  --authorization-type NONE

# Create integration (Lambda)
aws apigateway put-integration \
  --rest-api-id abc123 \
  --resource-id xyz789 \
  --http-method GET \
  --type AWS_PROXY \
  --integration-http-method POST \
  --uri arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/arn:aws:lambda:us-east-1:123456789012:function:myFunc/invocations

# Deploy API
aws apigateway create-deployment \
  --rest-api-id abc123 \
  --stage-name prod
```

**Endpoint URL:**

```
https://abc123.execute-api.us-east-1.amazonaws.com/prod/users
```

-----

## Application Deployment

### CodeCommit

**Git repositories managed by AWS**

```bash
# Clone repository
git clone https://git-codecommit.us-east-1.amazonaws.com/v1/repos/MyRepo

# Configure credentials
git config --global credential.helper '!aws codecommit credential-helper $@'
git config --global credential.UseHttpPath true
```

-----

### CodeBuild

**Compile source code, run tests, produce deployable artifacts**

**buildspec.yml Example:**

```yaml
version: 0.2

phases:
  install:
    runtime-versions:
      python: 3.9
    commands:
      - pip install -r requirements.txt
  
  pre_build:
    commands:
      - echo Running tests...
      - pytest
  
  build:
    commands:
      - echo Building application...
      - python setup.py build
  
  post_build:
    commands:
      - echo Build completed on `date`

artifacts:
  files:
    - '**/*'
  base-directory: dist

cache:
  paths:
    - '/root/.cache/pip/**/*'
```

**CLI Examples:**

```bash
# Start build
aws codebuild start-build --project-name MyProject

# View build logs
aws codebuild batch-get-builds --ids MyProject:12345
```

-----

### CodeDeploy

**Automate application deployments to EC2, Lambda, ECS**

**Deployment Types:**

- **In-place**: Stop app, deploy new version (downtime)
- **Blue/Green**: Deploy to new instances, switch traffic (zero downtime)

**appspec.yml Example (EC2):**

```yaml
version: 0.0
os: linux
files:
  - source: /
    destination: /var/www/html
hooks:
  BeforeInstall:
    - location: scripts/install_dependencies.sh
      timeout: 300
  ApplicationStart:
    - location: scripts/start_server.sh
      timeout: 300
  ValidateService:
    - location: scripts/validate.sh
      timeout: 300
```

**CLI Examples:**

```bash
# Create deployment
aws deploy create-deployment \
  --application-name MyApp \
  --deployment-group-name MyDeploymentGroup \
  --s3-location bucket=my-bucket,key=app.zip,bundleType=zip

# Get deployment status
aws deploy get-deployment --deployment-id d-1234567890
```

-----

### CodePipeline

**Continuous delivery orchestration**

**Stages:**

1. **Source**: CodeCommit, GitHub, S3
1. **Build**: CodeBuild
1. **Test**: CodeBuild, third-party tools
1. **Deploy**: CodeDeploy, ECS, Elastic Beanstalk, CloudFormation

**CLI Example:**

```bash
# Create pipeline
aws codepipeline create-pipeline --cli-input-json file://pipeline.json

# Start pipeline execution
aws codepipeline start-pipeline-execution --name MyPipeline
```

-----

### CloudFormation

**Infrastructure as Code (IaC)**

**Template Example (YAML):**

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Simple EC2 instance'

Parameters:
  InstanceType:
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t2.small
      - t2.medium

Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: !Ref InstanceType
      KeyName: MyKeyPair
      SecurityGroups:
        - !Ref MySecurityGroup
      Tags:
        - Key: Name
          Value: MyServer

  MySecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow HTTP and SSH
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

Outputs:
  InstanceId:
    Description: Instance ID
    Value: !Ref MyEC2Instance
  PublicIP:
    Description: Public IP address
    Value: !GetAtt MyEC2Instance.PublicIp
```

**CLI Examples:**

```bash
# Validate template
aws cloudformation validate-template --template-body file://template.yaml

# Create stack
aws cloudformation create-stack \
  --stack-name my-stack \
  --template-body file://template.yaml \
  --parameters ParameterKey=InstanceType,ParameterValue=t2.small

# Update stack
aws cloudformation update-stack \
  --stack-name my-stack \
  --template-body file://template.yaml

# Delete stack
aws cloudformation delete-stack --stack-name my-stack

# Describe stack
aws cloudformation describe-stacks --stack-name my-stack

# List stacks
aws cloudformation list-stacks --stack-status-filter CREATE_COMPLETE
```

**Change Sets (Preview Changes):**

```bash
# Create change set
aws cloudformation create-change-set \
  --stack-name my-stack \
  --change-set-name my-changes \
  --template-body file://template.yaml

# View changes
aws cloudformation describe-change-set \
  --stack-name my-stack \
  --change-set-name my-changes

# Execute change set
aws cloudformation execute-change-set \
  --stack-name my-stack \
  --change-set-name my-changes
```

-----

### SAM (Serverless Application Model)

**Framework for serverless applications**

**template.yaml Example:**

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31

Globals:
  Function:
    Timeout: 30
    Runtime: python3.9

Resources:
  HelloWorldFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: hello_world/
      Handler: app.lambda_handler
      Events:
        HelloWorld:
          Type: Api
          Properties:
            Path: /hello
            Method: get
      Environment:
        Variables:
          TABLE_NAME: !Ref MyTable

  MyTable:
    Type: AWS::Serverless::SimpleTable
    Properties:
      PrimaryKey:
        Name: id
        Type: String

Outputs:
  HelloWorldApi:
    Description: "API Gateway endpoint URL"
    Value: !Sub "https://${ServerlessRestApi}.execute-api.${AWS::Region}.amazonaws.com/Prod/hello/"
```

**SAM CLI Commands:**

```bash
# Initialize new project
sam init

# Build application
sam build

# Test locally
sam local start-api
sam local invoke HelloWorldFunction -e events/event.json

# Deploy
sam deploy --guided

# View logs
sam logs -n HelloWorldFunction --tail
```

-----

### CDK (Cloud Development Kit)

**Define infrastructure using programming languages**

**Supported Languages:** TypeScript, JavaScript, Python, Java, C#, Go

**Example (Python):**

```python
from aws_cdk import (
    Stack,
    aws_s3 as s3,
    aws_lambda as lambda_,
    aws_apigateway as apigw,
    RemovalPolicy
)
from constructs import Construct

class MyStack(Stack):
    def __init__(self, scope: Construct, id: str, **kwargs):
        super().__init__(scope, id, **kwargs)
        
        # S3 bucket
        bucket = s3.Bucket(self, "MyBucket",
            versioned=True,
            removal_policy=RemovalPolicy.DESTROY,
            auto_delete_objects=True
        )
        
        # Lambda function
        fn = lambda_.Function(self, "MyFunction",
            runtime=lambda_.Runtime.PYTHON_3_9,
            handler="index.handler",
            code=lambda_.Code.from_asset("lambda"),
            environment={
                "BUCKET_NAME": bucket.bucket_name
            }
        )
        
        # Grant permissions
        bucket.grant_read_write(fn)
        
        # API Gateway
        api = apigw.LambdaRestApi(self, "MyApi",
            handler=fn,
            proxy=False
        )
        
        items = api.root.add_resource("items")
        items.add_method("GET")
        items.add_method("POST")
```

**CDK CLI Commands:**

```bash
# Initialize project
cdk init app --language python

# List stacks
cdk list

# Synthesize CloudFormation template
cdk synth

# Deploy
cdk deploy

# Diff changes
cdk diff

# Destroy
cdk destroy
```

-----

## Monitoring & Logging

### CloudWatch

**Monitoring and observability service**

**Components:**

- **Metrics**: Numerical data points over time
- **Logs**: Application and system logs
- **Alarms**: Trigger actions based on metrics
- **Events/EventBridge**: Event-driven automation
- **Dashboards**: Visualize metrics

**Default EC2 Metrics (5-minute intervals):**

- CPUUtilization
- NetworkIn/NetworkOut
- DiskReadOps/DiskWriteOps
- StatusCheckFailed

**Custom Metrics:**

- Memory usage (requires CloudWatch agent)
- Disk space
- Custom application metrics

**CLI Examples:**

```bash
# Put metric data
aws cloudwatch put-metric-data \
  --namespace "MyApp" \
  --metric-name ResponseTime \
  --value 234 \
  --unit Milliseconds

# Get metric statistics
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
  --start-time 2024-01-15T00:00:00Z \
  --end-time 2024-01-15T23:59:59Z \
  --period 3600 \
  --statistics Average

# Create alarm
aws cloudwatch put-metric-alarm \
  --alarm-name HighCPU \
  --alarm-description "Alert when CPU exceeds 80%" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:MyTopic

# Delete alarm
aws cloudwatch delete-alarms --alarm-names HighCPU
```

**CloudWatch Logs:**

```bash
# Create log group
aws logs create-log-group --log-group-name /aws/lambda/myfunction

# Create log stream
aws logs create-log-stream \
  --log-group-name /aws/lambda/myfunction \
  --log-stream-name 2024/01/15/stream

# Put log events
aws logs put-log-events \
  --log-group-name /aws/lambda/myfunction \
  --log-stream-name 2024/01/15/stream \
  --log-events timestamp=1610000000000,message="Error occurred"

# Query logs (Insights)
aws logs start-query \
  --log-group-name /aws/lambda/myfunction \
  --start-time 1610000000 \
  --end-time 1610086400 \
  --query-string 'fields @timestamp, @message | filter @message like /ERROR/'

# Filter log events
aws logs filter-log-events \
  --log-group-name /aws/lambda/myfunction \
  --filter-pattern "ERROR"
```

**Log Insights Query Examples:**

```
# Find errors
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 20

# Count requests by status code
fields @timestamp, statusCode
| stats count() by statusCode

# Parse JSON logs
fields @timestamp, @message
| parse @message '{"user":"*","action":"*"}' as user, action
| filter action = "login"

# Calculate percentiles
fields @timestamp, duration
| stats avg(duration), pct(duration, 50), pct(duration, 95), pct(duration, 99)
```

-----

### CloudWatch Agent

**Collect system metrics and logs**

**Installation (EC2):**

```bash
# Download and install
wget https://s3.amazonaws.com/amazoncloudwatch-agent/linux/amd64/latest/amazon-cloudwatch-agent.rpm
sudo rpm -U amazon-cloudwatch-agent.rpm

# Configure
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard

# Start agent
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config \
  -m ec2 \
  -s \
  -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json
```

**Config Example (JSON):**

```json
{
  "metrics": {
    "namespace": "MyApp",
    "metrics_collected": {
      "mem": {
        "measurement": [
          {"name": "mem_used_percent", "rename": "MemoryUsage"}
        ],
        "metrics_collection_interval": 60
      },
      "disk": {
        "measurement": [
          {"name": "used_percent", "rename": "DiskUsage"}
        ],
        "metrics_collection_interval": 60,
        "resources": ["*"]
      }
    }
  },
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/myapp.log",
            "log_group_name": "/aws/ec2/myapp",
            "log_stream_name": "{instance_id}"
          }
        ]
      }
    }
  }
}
```

-----

### X-Ray

**Distributed tracing for microservices**

**Features:**

- Service map visualization
- Request tracing across services
- Performance bottleneck identification
- Error analysis

**SDK Integration (Python):**

```python
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core import patch_all

# Patch libraries (boto3, requests, etc.)
patch_all()

# Manual instrumentation
@xray_recorder.capture('process_order')
def process_order(order_id):
    # Add metadata
    xray_recorder.put_metadata('order_id', order_id)
    xray_recorder.put_annotation('customer_type', 'premium')
    
    # Your code here
    result = fetch_order(order_id)
    return result
```

**Lambda Integration:**

```python
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core import patch_all

patch_all()

def lambda_handler(event, context):
    # Automatically traced
    return {
        'statusCode': 200,
        'body': 'Success'
    }
```

-----

### EventBridge

**Serverless event bus**

**Event Sources:**

- AWS services (EC2, S3, etc.)
- Custom applications
- SaaS partners

**Use Cases:**

- React to state changes
- Schedule tasks (cron-like)
- Cross-account event routing

**CLI Examples:**

```bash
# Create rule (scheduled)
aws events put-rule \
  --name DailyBackup \
  --schedule-expression "cron(0 2 * * ? *)"

# Add Lambda target
aws events put-targets \
  --rule DailyBackup \
  --targets "Id"="1","Arn"="arn:aws:lambda:us-east-1:123456789012:function:BackupFunction"

# Create rule (event pattern)
aws events put-rule \
  --name EC2StateChange \
  --event-pattern '{"source":["aws.ec2"],"detail-type":["EC2 Instance State-change Notification"]}'

# Put custom event
aws events put-events \
  --entries '[{"Source":"myapp","DetailType":"order.created","Detail":"{\"orderId\":\"123\"}"}]'
```

**Event Pattern Examples:**

```json
// EC2 instance state change
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": {
    "state": ["terminated"]
  }
}

// S3 object created
{
  "source": ["aws.s3"],
  "detail-type": ["Object Created"],
  "detail": {
    "bucket": {
      "name": ["my-bucket"]
    }
  }
}
```

-----

### SNS (Simple Notification Service)

**Pub/Sub messaging service**

**Features:**

- Multiple subscribers per topic
- Fan-out pattern
- SMS, email, HTTP/S, Lambda, SQS endpoints
- Message filtering

**CLI Examples:**

```bash
# Create topic
aws sns create-topic --name MyTopic

# Subscribe (email)
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:MyTopic \
  --protocol email \
  --notification-endpoint user@example.com

# Subscribe (Lambda)
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:MyTopic \
  --protocol lambda \
  --notification-endpoint arn:aws:lambda:us-east-1:123456789012:function:MyFunction

# Publish message
aws sns publish \
  --topic-arn arn:aws:sns:us-east-1:123456789012:MyTopic \
  --message "System alert: High CPU usage detected" \
  --subject "Alert"

# Publish with attributes (for filtering)
aws sns publish \
  --topic-arn arn:aws:sns:us-east-1:123456789012:MyTopic \
  --message "Order placed" \
  --message-attributes '{"event_type":{"DataType":"String","StringValue":"order"}}'
```

**Filter Policy Example:**

```json
{
  "event_type": ["order", "payment"],
  "amount": [{"numeric": [">", 100]}]
}
```

-----

### SQS (Simple Queue Service)

**Fully managed message queuing**

**Queue Types:**

- **Standard**: Unlimited throughput, at-least-once delivery, best-effort ordering
- **FIFO**: Exactly-once processing, ordered, 3000 msg/sec with batching

**Features:**

- Dead Letter Queue (DLQ) for failed messages
- Visibility timeout (default 30 seconds)
- Message retention: 1 minute to 14 days
- Long polling for cost efficiency

**CLI Examples:**

```bash
# Create queue
aws sqs create-queue --queue-name MyQueue

# Create FIFO queue
aws sqs create-queue \
  --queue-name MyQueue.fifo \
  --attributes FifoQueue=true,ContentBasedDeduplication=true

# Send message
aws sqs send-message \
  --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/MyQueue \
  --message-body "Hello from SQS"

# Send message (FIFO)
aws sqs send-message \
  --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/MyQueue.fifo \
  --message-body "Order #123" \
  --message-group-id "orders" \
  --message-deduplication-id "order-123"

# Receive messages
aws sqs receive-message \
  --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/MyQueue \
  --max-number-of-messages 10 \
  --wait-time-seconds 20

# Delete message
aws sqs delete-message \
  --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/MyQueue \
  --receipt-handle "AQEBwJnK...="

# Purge queue (delete all messages)
aws sqs purge-queue \
  --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/MyQueue
```

**Lambda + SQS Integration:**

```python
import json

def lambda_handler(event, context):
    for record in event['Records']:
        message_body = record['body']
        print(f"Processing: {message_body}")
        
        # Process message
        # If exception occurs, message returns to queue
        
    return {
        'statusCode': 200,
        'body': json.dumps('Success')
    }
```

-----

## Security & Identity

### IAM (Identity and Access Management)

**Manage access to AWS services**

**Components:**

- **Users**: Individual accounts
- **Groups**: Collection of users
- **Roles**: Assumed by services or users
- **Policies**: JSON documents defining permissions

**Policy Structure:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": "203.0.113.0/24"
        }
      }
    }
  ]
}
```

**Common Policy Examples:**

**S3 Read-Only:**

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "s3:GetObject",
      "s3:ListBucket"
    ],
    "Resource": [
      "arn:aws:s3:::my-bucket",
      "arn:aws:s3:::my-bucket/*"
    ]
  }]
}
```

**EC2 Full Access:**

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": "ec2:*",
    "Resource": "*"
  }]
}
```

**CLI Examples:**

```bash
# Create user
aws iam create-user --user-name alice

# Create group
aws iam create-group --group-name developers

# Add user to group
aws iam add-user-to-group --user-name alice --group-name developers

# Attach policy to group
aws iam attach-group-policy \
  --group-name developers \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Create role
aws iam create-role \
  --role-name LambdaExecutionRole \
  --assume-role-policy-document file://trust-policy.json

# Attach policy to role
aws iam attach-role-policy \
  --role-name LambdaExecutionRole \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

# Create access key
aws iam create-access-key --user-name alice

# List policies attached to user
aws iam list-attached-user-policies --user-name alice
```

**Trust Policy (for Role):**

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "Service": "lambda.amazonaws.com"
    },
    "Action": "sts:AssumeRole"
  }]
}
```

**Best Practices:**

- Enable MFA for root and privileged users
- Use roles instead of access keys for EC2
- Follow principle of least privilege
- Rotate credentials regularly
- Use IAM Access Analyzer to identify overly permissive policies

-----

### KMS (Key Management Service)

**Manage encryption keys**

**Features:**

- Symmetric and asymmetric keys
- Automatic key rotation
- CloudTrail logging
- Integration with AWS services

**CLI Examples:**

```bash
# Create key
aws kms create-key --description "My encryption key"

# Create alias
aws kms create-alias \
  --alias-name alias/mykey \
  --target-key-id 1234abcd-12ab-34cd-56ef-1234567890ab

# Encrypt data
aws kms encrypt \
  --key-id alias/mykey \
  --plaintext "Hello World" \
  --output text \
  --query CiphertextBlob | base64 --decode > encrypted.bin

# Decrypt data
aws kms decrypt \
  --ciphertext-blob fileb://encrypted.bin \
  --output text \
  --query Plaintext | base64 --decode

# Enable key rotation
aws kms enable-key-rotation --key-id 1234abcd-12ab-34cd-56ef-1234567890ab
```

-----

### Secrets Manager

**Store and rotate secrets**

**Use Cases:**

- Database credentials
- API keys
- OAuth tokens

**CLI Examples:**

```bash
# Create secret
aws secretsmanager create-secret \
  --name MyDatabasePassword \
  --secret-string '{"username":"admin","password":"MyP@ssw0rd"}'

# Retrieve secret
aws secretsmanager get-secret-value --secret-id MyDatabasePassword

# Update secret
aws secretsmanager update-secret \
  --secret-id MyDatabasePassword \
  --secret-string '{"username":"admin","password":"NewP@ssw0rd"}'

# Enable automatic rotation
aws secretsmanager rotate-secret \
  --secret-id MyDatabasePassword \
  --rotation-lambda-arn arn:aws:lambda:us-east-1:123456789012:function:RotateSecret \
  --rotation-rules AutomaticallyAfterDays=30
```

**Lambda Code Example:**

```python
import boto3
import json

def lambda_handler(event, context):
    sm = boto3.client('secretsmanager')
    
    # Get secret
    response = sm.get_secret_value(SecretId='MyDatabasePassword')
    secret = json.loads(response['SecretString'])
    
    username = secret['username']
    password = secret['password']
    
    # Use credentials
    # ...
```

-----

### WAF (Web Application Firewall)

**Protect web applications from exploits**

**Rule Types:**

- IP set matching
- SQL injection protection
- XSS protection
- Rate limiting
- Geo blocking

**CLI Example:**

```bash
# Create web ACL
aws wafv2 create-web-acl \
  --name MyWebACL \
  --scope REGIONAL \
  --default-action Allow={} \
  --rules file://rules.json

# Associate with ALB
aws wafv2 associate-web-acl \
  --web-acl-arn arn:aws:wafv2:us-east-1:123456789012:regional/webacl/MyWebACL/... \
  --resource-arn arn:aws:elasticloadbalancing:us-east-1:123456789012:loadbalancer/app/my-alb/...
```

-----

### Shield

**DDoS protection**

**Tiers:**

- **Shield Standard**: Free, automatic protection
- **Shield Advanced**: $3000/month, enhanced protection, 24/7 DRT support

-----

### GuardDuty

**Threat detection service**

**Features:**

- Monitors CloudTrail, VPC Flow Logs, DNS logs
- Machine learning anomaly detection
- Integration with SecurityHub

-----

### Inspector

**Automated security assessment**

**Scans:**

- EC2 instances
- Container images in ECR
- Lambda functions

**Findings:**

- CVE vulnerabilities
- Network reachability issues
- CIS benchmarks

-----

## Developer Tools & Testing

### CloudShell

**Browser-based shell with AWS CLI pre-installed**

**Features:**

- 1 GB persistent storage per region
- Pre-authenticated with your credentials
- No additional cost

-----

### Systems Manager (SSM)

**Operational hub for AWS resources**

**Key Features:**

**Parameter Store:**

```bash
# Put parameter
aws ssm put-parameter \
  --name /myapp/database/password \
  --value "MySecurePassword" \
  --type SecureString

# Get parameter
aws ssm get-parameter \
  --name /myapp/database/password \
  --with-decryption
```

**Session Manager (SSH alternative):**

```bash
# Start session
aws ssm start-session --target i-1234567890abcdef0

# Port forwarding
aws ssm start-session \
  --target i-1234567890abcdef0 \
  --document-name AWS-StartPortForwardingSession \
  --parameters "portNumber=3306,localPortNumber=3306"
```

**Run Command:**

```bash
# Execute command on EC2
aws ssm send-command \
  --instance-ids i-1234567890abcdef0 \
  --document-name "AWS-RunShellScript" \
  --parameters 'commands=["sudo yum update -y"]'
```

-----

### CodeArtifact

**Artifact repository for package management**

**Supported Formats:**

- npm, PyPI, Maven, NuGet

**CLI Example:**

```bash
# Create repository
aws codeartifact create-repository \
  --domain my-domain \
  --repository my-repo

# Login (npm)
aws codeartifact login \
  --tool npm \
  --domain my-domain \
  --repository my-repo

# Publish package
npm publish
```

-----

### Device Farm

**Test mobile apps on real devices**

**Supported Platforms:**

- iOS, Android, web apps

-----

## Analytics & Data Processing

### Athena

**Serverless SQL queries on S3**

**Features:**

- Standard SQL
- Pay per query ($5 per TB scanned)
- Integration with QuickSight for visualization
- Support for Parquet, ORC, JSON, CSV

**CLI Examples:**

```bash
# Start query execution
aws athena start-query-execution \
  --query-string "SELECT * FROM my_table WHERE date = '2024-01-15'" \
  --result-configuration OutputLocation=s3://my-results-bucket/ \
  --query-execution-context Database=my_database

# Get query results
aws athena get-query-results --query-execution-id <execution-id>
```

**Create Table (DDL):**

```sql
CREATE EXTERNAL TABLE logs (
  timestamp STRING,
  request_method STRING,
  status_code INT,
  response_time FLOAT
)
PARTITIONED BY (year INT, month INT, day INT)
STORED AS PARQUET
LOCATION 's3://my-logs-bucket/';
```

-----

### EMR (Elastic MapReduce)

**Big data processing with Hadoop, Spark**

**Supported Frameworks:**

- Apache Spark, Hadoop, Hive, Presto, Flink

**CLI Example:**

```bash
# Create cluster
aws emr create-cluster \
  --name "My Spark Cluster" \
  --release-label emr-6.10.0 \
  --applications Name=Spark \
  --instance-type m5.xlarge \
  --instance-count 3 \
  --use-default-roles

# Submit Spark job
aws emr add-steps \
  --cluster-id j-XXXXXXXXXXXXX \
  --steps Type=Spark,Name="My Spark Job",ActionOnFailure=CONTINUE,Args=[--class,org.example.MyApp,s3://my-bucket/app.jar]
```

-----

### Kinesis

**Real-time data streaming**

**Services:**

- **Kinesis Data Streams**: Real-time streaming
- **Kinesis Data Firehose**: Load streams into S3, Redshift, Elasticsearch
- **Kinesis Data Analytics**: SQL queries on streams
- **Kinesis Video Streams**: Video streaming

**Data Streams CLI:**

```bash
# Create stream
aws kinesis create-stream \
  --stream-name MyStream \
  --shard-count 2

# Put record
aws kinesis put-record \
  --stream-name MyStream \
  --partition-key user123 \
  --data "Hello Kinesis"

# Get records
aws kinesis get-shard-iterator \
  --stream-name MyStream \
  --shard-id shardId-000000000000 \
  --shard-iterator-type LATEST

aws kinesis get-records --shard-iterator <iterator>
```

**Firehose Example:**

```bash
# Create delivery stream
aws firehose create-delivery-stream \
  --delivery-stream-name MyFirehose \
  --s3-destination-configuration \
    RoleARN=arn:aws:iam::123456789012:role/FirehoseRole,\
BucketARN=arn:aws:s3:::my-bucket

# Put record
aws firehose put-record \
  --delivery-stream-name MyFirehose \
  --record Data=`echo "data" | base64`
```

-----

### Glue

**ETL and data catalog service**

**Components:**

- **Data Catalog**: Metadata repository
- **Crawlers**: Discover and catalog data
- **ETL Jobs**: Transform data (Python or Scala)

**CLI Examples:**

```bash
# Create crawler
aws glue create-crawler \
  --name my-crawler \
  --role GlueServiceRole \
  --database-name my_database \
  --targets S3Targets=[{Path=s3://my-bucket/data/}]

# Start crawler
aws glue start-crawler --name my-crawler

# Create job
aws glue create-job \
  --name my-etl-job \
  --role GlueServiceRole \
  --command Name=glueetl,ScriptLocation=s3://my-bucket/scripts/transform.py

# Start job run
aws glue start-job-run --job-name my-etl-job
```

**Glue ETL Script Example (Python):**

```python
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job

args = getResolvedOptions(sys.argv, ['JOB_NAME'])
sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args['JOB_NAME'], args)

# Read from Data Catalog
datasource = glueContext.create_dynamic_frame.from_catalog(
    database="my_database",
    table_name="raw_data"
)

# Transform
transformed = datasource.filter(lambda x: x["status"] == "active")

# Write to S3
glueContext.write_dynamic_frame.from_options(
    frame=transformed,
    connection_type="s3",
    connection_options={"path": "s3://my-bucket/processed/"},
    format="parquet"
)

job.commit()
```

-----

### Redshift

**Data warehousing**

**Features:**

- Columnar storage
- Parallel query execution
- ANSI SQL support
- Integration with S3, Athena, QuickSight

**CLI Examples:**

```bash
# Create cluster
aws redshift create-cluster \
  --cluster-identifier my-cluster \
  --node-type dc2.large \
  --master-username admin \
  --master-user-password MyPassword123 \
  --number-of-nodes 2

# Load data from S3
COPY my_table
FROM 's3://my-bucket/data/'
IAM_ROLE 'arn:aws:iam::123456789012:role/RedshiftRole'
FORMAT AS PARQUET;
```

-----

### QuickSight

**Business intelligence and visualization**

**Features:**

- Interactive dashboards
- ML-powered insights
- Embedded analytics
- Pay-per-session pricing

-----

## Messaging & Integration

### Step Functions

**Orchestrate distributed applications**

**State Types:**

- Task (Lambda, ECS, Batch, SNS, SQS, etc.)
- Choice (conditional branching)
- Parallel (execute branches in parallel)
- Wait (delay)
- Fail/Succeed (terminal states)

**State Machine Example (JSON):**

```json
{
  "Comment": "Order processing workflow",
  "StartAt": "ValidateOrder",
  "States": {
    "ValidateOrder": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:ValidateOrder",
      "Next": "CheckInventory"
    },
    "CheckInventory": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:CheckInventory",
      "Next": "IsAvailable"
    },
    "IsAvailable": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.available",
          "BooleanEquals": true,
          "Next": "ProcessPayment"
        }
      ],
      "Default": "OutOfStock"
    },
    "ProcessPayment": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:ProcessPayment",
      "Next": "ShipOrder"
    },
    "ShipOrder": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:ShipOrder",
      "End": true
    },
    "OutOfStock": {
      "Type": "Fail",
      "Error": "OutOfStock",
      "Cause": "Item not available"
    }
  }
}
```

**CLI Examples:**

```bash
# Create state machine
aws stepfunctions create-state-machine \
  --name MyStateMachine \
  --definition
```