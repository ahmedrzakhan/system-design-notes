# AWS (Amazon Web Services) Interview Questions & Answers

## 1. What is AWS and core services?

**Q: Explain AWS and main offerings**

A: Amazon Web Services is a cloud computing platform providing infrastructure, platforms, and software over the internet.

**Core service categories**:

**Compute**:

- EC2 — Virtual machines
- Lambda — Serverless functions
- ECS/EKS — Container orchestration
- Elastic Beanstalk — Managed platform
- AppRunner — Container app hosting

**Storage**:

- S3 — Object storage
- EBS — Block storage volumes
- EFS — File system
- Glacier — Archival storage
- FSx — Windows file server

**Database**:

- RDS — Relational databases
- DynamoDB — NoSQL
- ElastiCache — In-memory cache
- Neptune — Graph database
- Redshift — Data warehouse

**Networking**:

- VPC — Virtual private cloud
- CloudFront — CDN
- Route 53 — DNS
- ELB/ALB/NLB — Load balancers
- API Gateway — REST API management

**Security & Compliance**:

- IAM — Access management
- Secrets Manager — Secret storage
- KMS — Key management
- ACM — Certificate management
- GuardDuty — Security monitoring

**Developer Tools**:

- CodeCommit — Git repository
- CodeBuild — Build service
- CodePipeline — CI/CD
- CodeDeploy — Deployment service
- CloudFormation — Infrastructure as Code

**Monitoring**:

- CloudWatch — Logs and metrics
- X-Ray — Tracing
- CloudTrail — Audit logs

---

## 2. What is EC2?

**Q: Explain EC2 (Elastic Compute Cloud)**

A: Virtual machines in the cloud. Pay for what you use, can scale up/down.

**Instance types**:

```
T2 — General purpose, burstable (web servers)
T3 — Newer T2, better performance
M5 — General purpose, balanced (applications)
C5 — Compute optimized (processing)
R5 — Memory optimized (databases)
I3 — Storage optimized (NoSQL)
P3 — GPU (ML, HPC)
G3 — Graphics (rendering)
```

**Naming**:

```
t3.medium
├─ Family: t (burstable)
├─ Generation: 3
└─ Size: medium (small, medium, large, xlarge, 2xlarge)
```

**Pricing models**:

```
On-Demand: Pay per hour/second (most expensive, flexible)
Spot: Bid on unused capacity (70% discount, can be terminated)
Reserved: 1-3 year commitment (40-70% discount, fixed)
Dedicated: Physical server (compliance requirements)
Savings Plans: Compute savings (flexible)
```

**Launch instance**:

```bash
# Using AWS CLI
aws ec2 run-instances \
    --image-id ami-0c55b159cbfafe1f0 \
    --instance-type t3.micro \
    --key-name my-key \
    --security-groups my-security-group
```

**Auto Scaling**:

```
Automatically add/remove instances based on demand

Configuration:
- Minimum: 1
- Desired: 2
- Maximum: 10

Triggers:
- CPU > 70% → add instance
- CPU < 30% → remove instance
```

---

## 3. What is Lambda?

**Q: Explain Lambda (serverless)**

A: Run code without managing servers. Pay only for compute time used.

**Characteristics**:

- Stateless functions
- Auto-scaling
- 15 minute max execution
- Limited memory (128MB - 10GB)
- No infrastructure management
- Can be triggered by events

**Supported languages**:

- Node.js, Python, Java, Go, C#, Ruby

**Example**:

```python
# Lambda function
def lambda_handler(event, context):
    name = event.get('name', 'World')
    return {
        'statusCode': 200,
        'body': f'Hello {name}'
    }
```

**Triggers**:

```
API Gateway → Lambda → Database
S3 Event → Lambda → Process file
DynamoDB Stream → Lambda → Notification
CloudWatch Events → Lambda → Scheduled task
SQS → Lambda → Process message
```

**Pricing**:

```
Free tier: 1 million requests + 400,000 GB-seconds per month

Cost:
- $0.20 per 1 million requests
- $0.0000166667 per GB-second

Example:
1000 requests * 512MB * 1 second
= 1000 * 0.5 GB-s = 500 GB-s
= 500 * $0.0000166667 = $0.0083
```

**Best practices**:

```
// Keep functions small and focused
// Use environment variables for config
// Return quickly (for cost)
// Use Lambda layers for shared code
// Set appropriate timeout
// Monitor CloudWatch logs
```

---

## 4. What is S3?

**Q: Explain S3 (Simple Storage Service)**

A: Object storage service. Store and retrieve data at any time.

**Concepts**:

```
Bucket — Container (like folder)
Object — File + metadata
Key — File path
Region — Geographic location
```

**Storage classes**:

```
Standard — Frequently accessed ($0.023/GB)
Intelligent-Tiering — Automatic optimization
Glacier — Long-term archival ($0.004/GB)
Deep Archive — Very long-term ($0.00099/GB)
```

**Pricing example**:

```
100 GB of files in Standard: 100 * $0.023 = $2.30/month
1000 PUT requests: 1000 * $0.005 per 1000 = $0.005
1000 GET requests: 1000 * $0.0004 per 1000 = $0.0004
```

**Using S3**:

```bash
# Create bucket
aws s3 mb s3://my-bucket

# Upload file
aws s3 cp file.txt s3://my-bucket/

# List files
aws s3 ls s3://my-bucket/

# Download file
aws s3 cp s3://my-bucket/file.txt ./

# Sync directory
aws s3 sync ./local-folder s3://my-bucket/remote-folder
```

**Features**:

```
Versioning — Keep multiple versions
Lifecycle policies — Auto-delete/archive old files
Replication — Copy across regions
Access Control — ACLs, bucket policies
Encryption — At-rest and in-transit
CORS — Cross-origin requests
```

**Use cases**:

- Static website hosting
- Backup and archive
- Data lake
- Media storage
- Big data analytics

---

## 5. What is RDS?

**Q: Explain RDS (Relational Database Service)**

A: Managed database service. AWS handles backups, patches, scaling.

**Supported databases**:

- MySQL/MariaDB
- PostgreSQL
- Oracle
- SQL Server
- Aurora (AWS proprietary, better performance)

**Deployment models**:

```
Single-AZ — One availability zone (cheap, risky)
Multi-AZ — Automatic failover, synchronous replication
Read Replicas — Asynchronous copies for read scaling
```

**Instance classes**:

```
db.t3.micro — Burstable, free tier eligible
db.t3.small — Small application
db.m5.large — General purpose
db.r5.xlarge — Memory optimized
db.x1e.xlarge — Extreme memory
```

**Backup and recovery**:

```
Automated backups — 1-35 days retention
Manual snapshots — Persistent until deleted
Point-in-time restore — Restore to any second
```

**Example setup**:

```python
import pymysql

connection = pymysql.connect(
    host='mydb.c9akciq32.us-east-1.rds.amazonaws.com',
    user='admin',
    password='password',
    database='mydb'
)

cursor = connection.cursor()
cursor.execute('SELECT * FROM users')
results = cursor.fetchall()
```

**Scaling**:

```
Vertical — Larger instance (downtime needed)
Horizontal — Read replicas (read-only copies)
Aurora — Auto-scaling, better performance
```

---

## 6. What is DynamoDB?

**Q: Explain DynamoDB (NoSQL)**

A: Managed NoSQL database. Highly available, auto-scaling, predictable performance.

**Data model**:

```
Table — Collection of items
Item — Row with attributes
Attribute — Column
Primary Key — Unique identifier

Types:
- Partition key (hash key) — Required
- Sort key (range key) — Optional
```

**Example**:

```
Table: Users
├─ Partition key: user_id
├─ Sort key: timestamp
└─ Attributes:
   ├─ name: "John"
   ├─ email: "john@example.com"
   └─ metadata: {...}
```

**Consistency models**:

```
Strongly Consistent — Latest data (slower)
Eventually Consistent — May have stale data (faster, default)
```

**Pricing**:

```
On-Demand:
- $1.25 per million write units
- $0.25 per million read units
- $1.25 per million read units for leased read capacity

Provisioned:
- Fixed capacity at predictable cost
- Good for predictable workloads
```

**Using DynamoDB**:

```python
import boto3

dynamodb = boto3.resource('dynamodb', region_name='us-east-1')
table = dynamodb.Table('Users')

# Put item
table.put_item(Item={
    'user_id': '123',
    'name': 'John',
    'email': 'john@example.com'
})

# Get item
response = table.get_item(Key={'user_id': '123'})
print(response['Item'])

# Query
response = table.query(KeyConditionExpression='user_id = :uid',
    ExpressionAttributeValues={':uid': '123'})

# Scan (slow, don't use in production)
response = table.scan()

# Update
table.update_item(
    Key={'user_id': '123'},
    UpdateExpression='SET #n = :val',
    ExpressionAttributeNames={'#n': 'name'},
    ExpressionAttributeValues={':val': 'Jane'}
)

# Delete
table.delete_item(Key={'user_id': '123'})
```

**Global Secondary Index (GSI)**:

```
Query data by non-key attributes
Example: Query by email instead of user_id
Separate throughput from main table
```

---

## 7. What is VPC?

**Q: Explain VPC (Virtual Private Cloud)**

A: Private network in AWS. Control IP addresses, subnets, routing, security.

**Components**:

```
VPC — Network boundary (e.g., 10.0.0.0/16)
  ├─ Public Subnet — Accessible from internet (10.0.1.0/24)
  ├─ Private Subnet — Not accessible from internet (10.0.2.0/24)
  ├─ Internet Gateway — Connect to internet
  ├─ NAT Gateway — Allow private to access internet
  ├─ Route Table — Define routes
  └─ Security Groups — Firewall rules
```

**Architecture example**:

```
Internet
    ↓
Internet Gateway
    ↓
Public Subnet (ALB, NAT)
    ↓
Private Subnet (EC2, Lambda)
    ↓
Database Subnet (RDS)
```

**Security Groups** (stateful firewall):

```
Inbound rules:
- Allow HTTP (port 80) from 0.0.0.0/0
- Allow HTTPS (port 443) from 0.0.0.0/0
- Allow SSH (port 22) from my-ip/32
- Allow 5432 (PostgreSQL) from app-sg

Outbound rules:
- Allow all to 0.0.0.0/0
```

**Network ACLs** (stateless firewall):

```
Different from security groups
Applied at subnet level
Order matters (numbered rules)
Both allow and deny rules
```

**Creating VPC**:

```bash
# Create VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# Create public subnet
aws ec2 create-subnet \
    --vpc-id vpc-123 \
    --cidr-block 10.0.1.0/24 \
    --availability-zone us-east-1a

# Create Internet Gateway
aws ec2 create-internet-gateway

# Attach to VPC
aws ec2 attach-internet-gateway \
    --internet-gateway-id igw-123 \
    --vpc-id vpc-123
```

---

## 8. What is IAM?

**Q: Explain IAM (Identity and Access Management)**

A: Manage who (users, roles) can do what (permissions) on AWS resources.

**Components**:

```
Users — People accessing AWS
Roles — Assume temporary permissions
Groups — Collections of users
Policies — Permissions documents
```

**Policy example**:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": "arn:aws:s3:::my-bucket/*"
    },
    {
      "Effect": "Deny",
      "Action": ["s3:DeleteObject"],
      "Resource": "*"
    }
  ]
}
```

**Best practices**:

```
1. Principle of least privilege
   - Only grant minimum needed permissions

2. Use roles, not access keys
   - Roles have temporary credentials
   - Can be assumed by services

3. Enable MFA
   - Multi-factor authentication

4. Rotate credentials regularly
   - Access keys, passwords

5. Use temporary credentials
   - STS (Security Token Service)

6. Audit with CloudTrail
   - Track who did what
```

**Creating user and role**:

```bash
# Create user
aws iam create-user --user-name john

# Create policy
aws iam create-policy --policy-name S3Access \
    --policy-document file://policy.json

# Attach policy to user
aws iam attach-user-policy \
    --user-name john \
    --policy-arn arn:aws:iam::123456789:policy/S3Access

# Create role for EC2
aws iam create-role --role-name EC2Role \
    --assume-role-policy-document file://trust-policy.json

# Create instance profile
aws iam create-instance-profile --instance-profile-name EC2Profile

# Add role to profile
aws iam add-role-to-instance-profile \
    --instance-profile-name EC2Profile \
    --role-name EC2Role
```

---

## 9. What is CloudFormation?

**Q: Explain CloudFormation (Infrastructure as Code)**

A: Define AWS infrastructure using templates (JSON/YAML).

**Benefits**:

- Infrastructure as code (version control)
- Reproducible deployments
- Stack management (create/update/delete all resources)
- Cost estimation
- Change sets (preview changes)

**Template structure**:

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: "VPC with EC2 instance"

Parameters:
  InstanceType:
    Type: String
    Default: t3.micro

Mappings:
  AMIMap:
    us-east-1:
      AMI: ami-0c55b159cbfafe1f0

Resources:
  MyVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsHostnames: true

  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MyVPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: us-east-1a

  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !FindInMap [AMIMap, !Ref "AWS::Region", AMI]
      InstanceType: !Ref InstanceType
      SubnetId: !Ref PublicSubnet

Outputs:
  InstanceId:
    Value: !Ref MyEC2Instance
    Description: Instance ID

  VPCId:
    Value: !Ref MyVPC
    Description: VPC ID
```

**Deployment**:

```bash
# Create stack
aws cloudformation create-stack \
    --stack-name my-stack \
    --template-body file://template.yaml \
    --parameters ParameterKey=InstanceType,ParameterValue=t3.small

# List stacks
aws cloudformation list-stacks

# Get stack info
aws cloudformation describe-stack-resources --stack-name my-stack

# Update stack
aws cloudformation update-stack \
    --stack-name my-stack \
    --template-body file://template.yaml

# Delete stack
aws cloudformation delete-stack --stack-name my-stack
```

---

## 10. What is API Gateway?

**Q: Explain API Gateway**

A: Managed service to create, publish, and manage APIs.

**Features**:

```
REST APIs — Traditional HTTP
HTTP APIs — Cheaper, simpler
WebSocket APIs — Real-time communication
```

**Architecture**:

```
Client
  ↓
API Gateway
  ├─ Authentication (API keys, auth tokens)
  ├─ Rate limiting
  ├─ Caching
  ├─ Logging
  └─ Routing
    ↓
Lambda/EC2/HTTP Endpoint
```

**Creating REST API**:

```bash
# Create API
aws apigateway create-rest-api --name MyAPI

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

# Set Lambda integration
aws apigateway put-integration \
    --rest-api-id abc123 \
    --resource-id xyz789 \
    --http-method GET \
    --type AWS_PROXY \
    --integration-http-method POST \
    --uri arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/arn:aws:lambda:us-east-1:123456789:function:MyFunction/invocations
```

**API Gateway + Lambda** (common pattern):

```python
# Lambda handler
def lambda_handler(event, context):
    http_method = event['httpMethod']
    path = event['path']

    if http_method == 'GET' and path == '/users':
        return {
            'statusCode': 200,
            'body': json.dumps({'users': [...]})
        }
    elif http_method == 'POST':
        body = json.loads(event['body'])
        # Create user...
        return {
            'statusCode': 201,
            'body': json.dumps({'id': '123'})
        }
    else:
        return {
            'statusCode': 404,
            'body': json.dumps({'error': 'Not found'})
        }
```

---

## 11. What is CloudWatch?

**Q: Explain CloudWatch (monitoring)**

A: Centralized monitoring for AWS resources and applications.

**Components**:

```
Metrics — Data points (CPU, memory, etc.)
Logs — Text output from applications
Alarms — Automated actions based on metrics
Events — Trigger workflows
```

**Metrics**:

```
AWS provided:
- EC2: CPU, disk reads/writes, network
- RDS: Database connections, queries
- Lambda: Invocations, errors, duration

Custom metrics:
- Can send any metric from application
```

**CloudWatch Logs**:

```python
import boto3

logs = boto3.client('logs')

# Create log group
logs.create_log_group(logGroupName='/my-app/logs')

# Put log events
logs.put_log_events(
    logGroupName='/my-app/logs',
    logStreamName='2024-01-01',
    logEvents=[
        {
            'timestamp': int(time.time() * 1000),
            'message': 'Application started'
        }
    ]
)

# Query logs
response = logs.start_query(
    logGroupName='/my-app/logs',
    startTime=int(time.time()) - 3600,
    endTime=int(time.time()),
    queryString='fields @timestamp, @message | filter @message like /ERROR/'
)
```

**Alarms**:

```
Alarm states:
- OK — Metric within threshold
- ALARM — Metric outside threshold
- INSUFFICIENT_DATA — Not enough data

Actions:
- Send SNS notification
- Execute Lambda
- Auto-scale
- Stop/terminate instance
```

**Dashboard**:

```
Visualize multiple metrics
Create custom dashboards
Real-time or past data
```

---

## 12. What is Elastic Load Balancer?

**Q: Explain load balancing**

A: Distribute traffic across multiple targets for high availability and scalability.

**Types**:

**Application Load Balancer (ALB)** — Layer 7 (Application):

```
- Route based on hostnames, paths, ports
- Best for web apps, microservices
- Example: /api/* → API service, /images/* → image service
```

**Network Load Balancer (NLB)** — Layer 4 (Transport):

```
- Extreme performance, low latency
- Best for non-HTTP protocols, gaming, IoT
- Millions of requests per second
```

**Classic Load Balancer (ELB)** — Deprecated:

```
- Legacy load balancer
- Not recommended for new projects
```

**Architecture**:

```
User Traffic
    ↓
Load Balancer (public subnet)
    ├─ Target 1 (private subnet)
    ├─ Target 2 (private subnet)
    └─ Target 3 (private subnet)

Health checks:
- Periodic checks to all targets
- Remove unhealthy targets
- Return healthy when ready
```

**Creating ALB**:

```bash
# Create target group
aws elbv2 create-target-group \
    --name my-targets \
    --protocol HTTP \
    --port 80 \
    --vpc-id vpc-123

# Register targets
aws elbv2 register-targets \
    --target-group-arn arn:... \
    --targets Id=i-123 Id=i-456

# Create load balancer
aws elbv2 create-load-balancer \
    --name my-lb \
    --subnets subnet-123 subnet-456 \
    --security-groups sg-789

# Create listener
aws elbv2 create-listener \
    --load-balancer-arn arn:... \
    --protocol HTTP \
    --port 80 \
    --default-actions Type=forward,TargetGroupArn=arn:...
```

---

## 13. What is CloudFront?

**Q: Explain CloudFront (CDN)**

A: Content Delivery Network. Cache and serve content from edge locations globally.

**Benefits**:

```
- Lower latency (serve from nearest edge)
- Reduce origin load
- DDoS protection
- SSL/TLS termination
- Cost savings (data transfer)
```

**Origins**:

```
- S3 bucket
- EC2 instance
- ELB
- Custom HTTP endpoint
```

**Caching**:

```
Default cache behaviors:
- HTML: 24 hours (TTL)
- CSS/JS: 1 year
- Images: 1 year

Cache key:
- By default: URL
- Custom: Include headers, query params, cookies
```

**Distributions**:

```
Web distribution — For websites
RTMP distribution — For streaming (deprecated)
```

**Creating distribution**:

```bash
aws cloudfront create-distribution \
    --distribution-config file://config.json
```

**Example config**:

```json
{
  "CallerReference": "unique-id",
  "Comment": "My distribution",
  "DefaultRootObject": "index.html",
  "Origins": {
    "Quantity": 1,
    "Items": [
      {
        "Id": "myS3Origin",
        "DomainName": "mybucket.s3.amazonaws.com",
        "S3OriginConfig": {}
      }
    ]
  },
  "DefaultCacheBehavior": {
    "TargetOriginId": "myS3Origin",
    "ViewerProtocolPolicy": "redirect-to-https",
    "TrustedSigners": {
      "Enabled": false,
      "Quantity": 0
    },
    "ForwardedValues": {
      "QueryString": false,
      "Cookies": { "Forward": "none" }
    },
    "MinTTL": 0
  },
  "Enabled": true
}
```

---

## 14. What is deployment patterns?

**Q: Common AWS deployment architectures**

A:

**Simple web app**:

```
ALB → EC2 Auto-Scaling Group → RDS
S3 (static files) ← CloudFront ← Users
```

**Microservices**:

```
API Gateway → Lambda (multiple functions)
         ↓
    DynamoDB/RDS
```

**Serverless app**:

```
S3 (static site) → CloudFront
API Gateway → Lambda → DynamoDB
```

**High-availability**:

```
Multi-AZ:
- ALB across AZs
- Auto-scaling across AZs
- RDS Multi-AZ
- Route 53 for DNS failover
```

**Blue-Green deployment**:

```
Blue (current) — Running in production
Green (new) — New version ready

Deploy to Green, test, then switch traffic
Quick rollback if issues
```

---

## 15. What is cost optimization?

**Q: Reduce AWS costs**

A:

**Strategies**:

**1. Right-sizing**:

```
- Analyze CloudWatch metrics
- Downsize overprovisioned instances
- Use smaller instance types
- Remove unused resources
```

**2. Reserved Instances/Savings Plans**:

```
- 1-year or 3-year commitments
- 40-70% discount vs on-demand
- Good for steady baseline
```

**3. Spot Instances**:

```
- Bid on unused capacity
- 70% discount
- Can be interrupted
- Good for fault-tolerant workloads
```

**4. Storage optimization**:

```
- Use cheaper storage classes
- Delete old snapshots
- Compress data
- Use S3 Intelligent-Tiering
```

**5. Data transfer**:

```
- CloudFront reduces data transfer
- Same-region transfers are free
- Use VPC endpoints to avoid internet gateway costs
```

**6. Shutdown unused resources**:

```
- Stop development instances
- Delete unattached EBS volumes
- Remove unused security groups
- Delete old snapshots
```

**Cost tracking**:

```bash
# Get cost and usage
aws ce get-cost-and-usage \
    --time-period Start=2024-01-01,End=2024-01-31 \
    --granularity MONTHLY \
    --metrics "UnblendedCost" \
    --group-by Type=DIMENSION,Key=SERVICE

# Use Cost Explorer console for visualization
```

---

## 16. What is disaster recovery?

**Q: Business continuity and resilience**

A: Plan for failures and minimize downtime/data loss.

**Metrics**:

```
RTO — Recovery Time Objective (how fast to recover)
RPO — Recovery Point Objective (acceptable data loss)

Example:
- RTO: 1 hour (must be back online in 1 hour)
- RPO: 15 minutes (lose max 15 minutes of data)
```

**Strategies**:

**Backup & Restore** (cheap, slow):

```
- Regular backups to S3/Glacier
- RTO: hours, RPO: hours
- Best for non-critical systems
```

**Pilot Light** (low-cost):

```
- Minimal version running in DR region
- Scale up if needed
- RTO: 10s minutes, RPO: minutes
```

**Warm Standby** (moderate cost):

```
- Full system running at reduced capacity
- Scale up on failure
- RTO: minutes, RPO: seconds
```

**Hot Standby** (expensive):

```
- Full duplicate system running
- Instant failover
- RTO: seconds, RPO: seconds
- Active-active replication
```

**Implementation**:

```python
# Enable RDS automated backups
aws rds modify-db-instance \
    --db-instance-identifier mydb \
    --backup-retention-period 30 \
    --preferred-backup-window "03:00-04:00"

# Enable cross-region replication
aws rds create-db-instance-read-replica \
    --db-instance-identifier mydb-replica \
    --source-db-instance-identifier mydb \
    --source-region us-east-1

# S3 cross-region replication
aws s3api put-bucket-replication \
    --bucket my-bucket \
    --replication-configuration file://replication.json
```

---

## 17. What is security best practices?

**Q: Secure AWS infrastructure**

A:

**Identity & Access**:

```
- Use IAM roles, not access keys
- Enable MFA
- Principle of least privilege
- Regular access reviews
- Rotate credentials
```

**Network Security**:

```
- Use VPC (not default VPC)
- Private subnets for databases
- Security groups (stateful firewall)
- NACLs for additional control
- VPC Flow Logs for monitoring
```

**Data Protection**:

```
- Encryption at rest (KMS)
- Encryption in transit (TLS)
- S3 Block Public Access
- Secrets Manager for credentials
- Database encryption
```

**Monitoring & Logging**:

```
- CloudTrail for audit logs
- CloudWatch for metrics
- VPC Flow Logs
- WAF for web applications
- GuardDuty for threats
```

**Compliance**:

```
- Config rules for compliance
- Automated remediation
- Audit reports
- Regular security assessments
```

---

## 18. What is auto-scaling?

**Q: Automatically add/remove resources**

A: Scale based on demand or schedule.

**Types**:

**Target tracking**:

```
Keep metric at target value
Example: CPU at 70%
- If > 70%: Add instance
- If < 70%: Remove instance
```

**Step scaling**:

```
Different actions at different thresholds
If CPU > 80%: Add 2 instances
If CPU > 90%: Add 4 instances
If CPU < 30%: Remove 1 instance
```

**Scheduled scaling**:

```
Predictable patterns
Example: Scale up at 9 AM, scale down at 6 PM
```

**Configuration**:

```bash
aws autoscaling create-auto-scaling-group \
    --auto-scaling-group-name my-asg \
    --launch-configuration my-lc \
    --min-size 2 \
    --max-size 10 \
    --desired-capacity 4 \
    --availability-zones us-east-1a us-east-1b

aws autoscaling put-scaling-policy \
    --auto-scaling-group-name my-asg \
    --policy-name scale-up \
    --policy-type TargetTrackingScaling \
    --target-tracking-configuration file://config.json
```

---

## 19. What is common AWS patterns?

**Q: Typical architectures**

A:

**Serverless blog**:

```
S3 (static) → CloudFront
API Gateway → Lambda → DynamoDB
S3 → Lambda → email notifications
```

**Mobile app backend**:

```
AppSync (GraphQL) → Lambda → DynamoDB
Cognito → Authentication
S3 → Image storage
```

**Real-time analytics**:

```
Kinesis → Lambda → S3
S3 → Athena → QuickSight (visualization)
```

**Data pipeline**:

```
S3 (source) → Lambda → AWS Glue
AWS Glue → S3 (processed)
Redshift → Analytics
```

---

## 20. What is real-world example?

**Q: Complete AWS project**

A: E-commerce application

```
Frontend:
S3 + CloudFront (React app)
Route 53 (DNS)

Backend:
API Gateway → ALB → Auto-scaling EC2
EC2 → RDS (PostgreSQL)

Cache:
ElastiCache (Redis) for sessions

Storage:
S3 for product images
CloudFront for CDN

Search:
Elasticsearch for product search

Queue:
SQS for order processing
SNS for notifications

Monitoring:
CloudWatch for logs/metrics
CloudTrail for audit
X-Ray for tracing

Security:
VPC with public/private subnets
Security groups
IAM roles
KMS encryption
```

**Infrastructure as Code** (CloudFormation):

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: E-commerce application

Resources:
  # VPC
  AppVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16

  # RDS Database
  AppDatabase:
    Type: AWS::RDS::DBInstance
    Properties:
      Engine: postgres
      InstanceClass: db.t3.small
      AllocatedStorage: 100
      MasterUsername: admin
      MasterUserPassword: !Sub "{{resolve:secretsmanager:dbpassword:SecretString:password}}"

  # ALB
  AppLoadBalancer:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      Scheme: internet-facing
      Type: application

  # Auto-scaling group
  AppASG:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      MinSize: 2
      MaxSize: 10
      DesiredCapacity: 4
      LaunchConfigurationName: !Ref LaunchConfig

  # S3 for static assets
  AssetBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: my-app-assets
      PublicAccessBlockConfiguration:
        BlockPublicAcls: true

  # CloudFront distribution
  CDN:
    Type: AWS::CloudFront::Distribution
    Properties:
      DistributionConfig:
        Enabled: true
        DefaultRootObject: index.html
        Origins:
          - DomainName: !GetAtt AssetBucket.DomainName
            Id: S3Origin

Outputs:
  APIEndpoint:
    Value: !GetAtt AppLoadBalancer.DNSName

  CDNDomain:
    Value: !GetAtt CDN.DomainName
```

---

## AWS Interview Tips

1. **Core services** — EC2, S3, RDS, Lambda, VPC
2. **Pricing models** — On-demand, Reserved, Spot
3. **Architecture** — Design scalable, resilient systems
4. **Security** — IAM, encryption, VPC, least privilege
5. **Monitoring** — CloudWatch, CloudTrail, X-Ray
6. **Databases** — RDS, DynamoDB, when to use each
7. **Networking** — VPC, subnets, security groups, load balancing
8. **Deployment** — Auto-scaling, CloudFormation, CI/CD
9. **Cost optimization** — Right-sizing, Reserved Instances
10. **Disaster recovery** — Backup, replication, failover
11. **Serverless** — Lambda, API Gateway, DynamoDB
12. **DevOps tools** — CodePipeline, CodeDeploy, CodeBuild
13. **Real projects** — Production experience with AWS
14. **Certifications** — AWS Solutions Architect helpful
15. **Hands-on** — Building actual systems on AWS
