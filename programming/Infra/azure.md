# Azure (Microsoft Azure) Interview Questions & Answers

## 1. What is Azure and core services?

**Q: Explain Microsoft Azure and main offerings**

A: Microsoft Azure is a cloud computing platform providing infrastructure, platforms, and software services.

**Core service categories**:

**Compute**:

- Virtual Machines — VMs (Windows/Linux)
- App Service — Web/mobile apps, APIs
- Azure Container Instances — Managed containers
- Azure Kubernetes Service (AKS) — Orchestration
- Azure Functions — Serverless
- Batch — Large-scale computing
- Cloud Services — Legacy platform

**Storage**:

- Blob Storage — Objects (like S3)
- File Share — SMB file shares
- Queue Storage — Message queues
- Table Storage — NoSQL
- Disk Storage — Persistent volumes

**Database**:

- Azure SQL Database — Managed SQL Server
- Azure Database for MySQL/PostgreSQL — Open source
- Azure Cosmos DB — Global NoSQL
- Azure Data Lake — Big data
- Azure Synapse Analytics — Data warehouse

**Networking**:

- Virtual Networks (VNets) — Private networks
- Load Balancer — Network level (Layer 4)
- Application Gateway — Application level (Layer 7)
- API Management — API gateway
- Azure Front Door — Global load balancing/CDN
- ExpressRoute — Dedicated network connection

**Security & Compliance**:

- Azure Active Directory (Entra ID) — Identity management
- Key Vault — Secret storage
- Security Center — Security monitoring
- DDoS Protection — Attack mitigation
- Firewall — Network security

**Analytics & Monitoring**:

- Azure Monitor — Logs and metrics
- Application Insights — Application monitoring
- Azure Log Analytics — Data analysis
- Power BI — Business intelligence

**Developer Tools**:

- Azure DevOps — CI/CD, project management
- Azure Repos — Git repositories
- Azure Pipelines — Build and release
- Azure Boards — Issue tracking
- GitHub Integration — Deep integration

---

## 2. What are Virtual Machines?

**Q: Explain Azure Virtual Machines**

A: Scalable on-demand computing resources. Deploy Windows or Linux VMs.

**VM series**:

```
B-series — Burstable (dev/test)
D-series — General purpose
E-series — Memory optimized
F-series — Compute optimized
G-series — Storage optimized
H-series — High-performance computing
L-series — Storage intensive
M-series — Memory intensive
N-series — GPU
```

**Naming convention**:

```
Standard_D2s_v3
├─ Standard — Performance level
├─ D — Series (general purpose)
├─ 2 — Number of vCPUs
├─ s — Premium storage capable
└─ v3 — Version
```

**Creating VM**:

```bash
# Using Azure CLI
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys \
    --size Standard_B2s
```

**Availability options**:

```
Availability Set — Protect against hardware failure
                   VM spread across multiple racks
                   99.95% availability

Availability Zone — Unique physical location in region
                    Redundant power, network, cooling
                    99.99% availability

Scale Set — Auto-scaling group of identical VMs
           Load balancing built-in
```

**Pricing**:

```
Pay-as-you-go — Hourly billing (most expensive)
Reserved Instances — 1-3 year commitment (35-72% discount)
Spot VMs — Unused capacity (up to 90% discount, can be evicted)
```

**Lifecycle**:

```
Started — Running (charged)
Stopped — Deallocated (not charged for compute, only storage)
Deleted — Removed (not charged)
```

---

## 3. What is Azure App Service?

**Q: Explain App Service**

A: Managed platform for building and hosting web apps, mobile apps, and APIs.

**Hosting options**:

```
Web Apps — ASP.NET, Node.js, Python, Java, PHP
Mobile Apps — Backend for mobile
API Apps — REST APIs
```

**App Service Plans**:

```
Free tier — 1 GB storage, no scaling
Shared tier — 1 GB storage, no scaling
Basic — 10 GB storage, manual scaling, no SSL
Standard — 50 GB storage, auto-scaling, SSL
Premium — 250 GB storage, app-to-app communication, staging slots
Isolated — Dedicated environment, highest performance
```

**Example: Deploy Node.js**:

```bash
# Create App Service Plan
az appservice plan create \
    --name myAppServicePlan \
    --resource-group myResourceGroup \
    --sku B1 \
    --is-linux

# Create Web App
az webapp create \
    --resource-group myResourceGroup \
    --plan myAppServicePlan \
    --name myWebApp \
    --runtime "node|18"

# Deploy from GitHub
az webapp deployment source config \
    --resource-group myResourceGroup \
    --name myWebApp \
    --repo-url https://github.com/user/repo \
    --branch master \
    --manual-integration
```

**Deployment slots**:

```
Production — Live app
Staging — Pre-production testing
Testing — QA environment

Features:
- Warm up before swap
- Swap traffic between slots
- Route percentage of traffic
```

**Auto-scaling**:

```
Rules: CPU > 70% → Add instance
       CPU < 30% → Remove instance
       Memory > 80% → Add instance

Scale both horizontally (instances)
and vertically (compute size)
```

---

## 4. What is Azure Storage?

**Q: Explain Azure Storage services**

A: Massively scalable cloud storage for data.

**Storage types**:

**Blob Storage** (objects):

```
Hot tier — Frequently accessed, highest cost
Cool tier — Infrequently accessed, medium cost
Archive tier — Long-term archival, lowest cost

Use cases:
- Documents, images, videos
- Backups
- Log files
- Static website hosting
```

**File Share** (SMB):

```
Azure Files — SMB file shares
NFS — Network file system

Use cases:
- SMB file shares for Windows
- NFS for Linux
- Lift and shift from on-premises
```

**Queue Storage** (messaging):

```
Store messages for processing
Guaranteed delivery
Visibility timeout
Dead letter queue

Use case: Decouple applications
```

**Table Storage** (NoSQL):

```
Key-value store
Scalable
Partition key + Row key = unique ID

Similar to DynamoDB but less feature-rich
Being replaced by Cosmos DB
```

**Example: Blob Storage**:

```python
from azure.storage.blob import BlobServiceClient

connection_string = "DefaultEndpointsProtocol=https;..."
blob_service_client = BlobServiceClient.from_connection_string(connection_string)

# Upload blob
container_client = blob_service_client.get_container_client("mycontainer")
with open("file.txt", "rb") as data:
    blob_client = container_client.upload_blob("file.txt", data, overwrite=True)

# Download blob
download_stream = blob_client.download_blob()
print(download_stream.readall())

# List blobs
for blob in container_client.list_blobs():
    print(blob.name)

# Delete blob
blob_client.delete_blob()
```

**Pricing**:

```
Hot: $0.0184/GB/month
Cool: $0.01/GB/month (30-day minimum)
Archive: $0.004/GB/month (180-day minimum, retrieval costs)
```

---

## 5. What is Azure SQL Database?

**Q: Explain Azure SQL Database**

A: Fully managed relational database. Microsoft SQL Server in the cloud.

**Deployment models**:

```
Single database — Dedicated to one application
Elastic pool — Share resources across databases
Managed instance — Full SQL Server compatibility
```

**Service tiers**:

```
Basic — Development, light workloads, 5 DTUs
Standard — Production workloads, 10-3000 DTUs
Premium — Performance demanding, 125-4000 DTUs
```

**DTU vs vCore**:

```
DTU — Database Transaction Unit (bundled compute)
vCore — Virtual cores (buy compute separately)

DTU: Simpler, fixed pricing
vCore: More control, better for complex workloads
```

**Example: Create and query**:

```python
import pyodbc

# Connect
server = 'myserver.database.windows.net'
database = 'mydb'
username = 'azureuser'
password = 'password'

connection_string = f'Driver={{ODBC Driver 17 for SQL Server}};Server={server};Database={database};Uid={username};Pwd={password}'
cnxn = pyodbc.connect(connection_string)

# Query
cursor = cnxn.cursor()
cursor.execute('SELECT * FROM Users')
for row in cursor:
    print(row)

# Insert
cursor.execute('INSERT INTO Users (name, email) VALUES (?, ?)', ('John', 'john@example.com'))
cnxn.commit()
```

**Backup and recovery**:

```
Point-in-time restore — Recover to any point (last 35 days)
Geo-restore — Restore from geo-replicated backup
Long-term retention — 7-35 years retention
```

**High availability**:

```
Locally redundant — 99.99% availability
Zone redundant — 99.995% availability (replicated across zones)
Geo-redundant — Multi-region replication
```

---

## 6. What is Azure Cosmos DB?

**Q: Explain Cosmos DB (global NoSQL)**

A: Globally distributed, multi-model database with guaranteed low latency.

**Data models**:

```
SQL API — Document database (like MongoDB)
MongoDB API — Wire protocol compatible
Cassandra API — Column store
Table API — Key-value
Gremlin API — Graph database
```

**Key features**:

```
Global distribution — Read/write in 99+ regions
Multi-master replication — Write anywhere
Guaranteed latency — < 10ms for 99th percentile
Automatic scaling — Based on usage
```

**Consistency levels**:

```
Strong — Latest data, slowest
Bounded staleness — Within certain lag
Session — Consistent within session
Consistent prefix — Respects order
Eventual — Best performance, might be stale
```

**Example**:

```python
from azure.cosmos import CosmosClient, PartitionKey

endpoint = "https://mydb.documents.azure.com:443/"
key = "your-key"
client = CosmosClient(endpoint, key)

# Create database
database = client.create_database_if_not_exists(id="mydb")

# Create container
container = database.create_container_if_not_exists(
    id="users",
    partition_key=PartitionKey(path="/user_id")
)

# Insert item
item = {
    "id": "1",
    "user_id": "user1",
    "name": "John",
    "email": "john@example.com"
}
container.create_item(body=item)

# Query
query = "SELECT * FROM c WHERE c.name = @name"
items = list(container.query_items(
    query=query,
    parameters=[{"name": "@name", "value": "John"}]
))

# Update
item["name"] = "Jane"
container.upsert_item(body=item)

# Delete
container.delete_item(item, partition_key="user1")
```

**Pricing**:

```
Provisioned throughput — Pay for RU/s
Serverless — Pay per operation (no minimum)

Example: 400 RU/s = ~$24/month
```

---

## 7. What is Azure Virtual Network (VNet)?

**Q: Explain networking in Azure**

A: Private network in Azure. Control IP addresses, subnets, routing, and security.

**Components**:

```
VNet — Address space (e.g., 10.0.0.0/16)
  ├─ Subnet — Subdivided address space (10.0.1.0/24)
  ├─ Network Security Group (NSG) — Firewall rules
  ├─ Route Table — Define traffic paths
  ├─ Service Endpoints — Direct connection to Azure services
  └─ Private Link — Private connectivity to services
```

**Network Security Groups**:

```
Inbound rules:
- Allow HTTP (port 80) from 0.0.0.0/0
- Allow HTTPS (port 443) from 0.0.0.0/0
- Allow SSH (port 22) from my-ip/32
- Allow 5432 (PostgreSQL) from app-subnet

Outbound rules:
- Allow all to 0.0.0.0/0 (default)

Stateful firewall — Return traffic automatically allowed
```

**Creating VNet**:

```bash
# Create VNet
az network vnet create \
    --resource-group myResourceGroup \
    --name myVNet \
    --address-prefix 10.0.0.0/16

# Create subnet
az network vnet subnet create \
    --resource-group myResourceGroup \
    --vnet-name myVNet \
    --name mySubnet \
    --address-prefix 10.0.1.0/24

# Create NSG
az network nsg create \
    --resource-group myResourceGroup \
    --name myNSG

# Add NSG rule
az network nsg rule create \
    --resource-group myResourceGroup \
    --nsg-name myNSG \
    --name allowHTTP \
    --priority 100 \
    --source-address-prefixes 0.0.0.0/0 \
    --destination-port-ranges 80 \
    --access Allow \
    --protocol Tcp
```

**VNet Peering**:

```
Connect two VNets directly
No internet gateway needed
Transitive peering not supported (need additional peering)
```

---

## 8. What is Azure Active Directory (Entra ID)?

**Q: Explain identity and access management**

A: Microsoft's cloud identity service. Manage users, groups, and permissions.

**Concepts**:

```
Users — Individual accounts
Groups — Collections of users
Applications — SaaS apps, enterprise apps
Service Principals — Identity for applications
Managed Identity — Azure resources authenticate without credentials
```

**Authentication**:

```
Multi-factor authentication (MFA)
Passwordless authentication
Single sign-on (SSO)
Conditional access (based on device, location, etc.)
```

**Hybrid Identity**:

```
Sync on-premises AD with Azure AD (Entra Connect)
Seamless authentication
Hybrid multi-factor authentication
```

**Example: Register app**:

```python
from azure.identity import ClientSecretCredential

# Create credential
credential = ClientSecretCredential(
    tenant_id="your-tenant-id",
    client_id="your-client-id",
    client_secret="your-client-secret"
)

# Use credential to access Azure resources
from azure.keyvault.secrets import SecretClient

secret_client = SecretClient(
    vault_url="https://myvault.vault.azure.net/",
    credential=credential
)

secret = secret_client.get_secret("my-secret")
print(secret.value)
```

---

## 9. What is Azure Key Vault?

**Q: Explain secret management**

A: Centralized service to store and manage secrets, keys, and certificates.

**What to store**:

```
- Database passwords
- API keys
- Connection strings
- SSH keys
- SSL certificates
- Encryption keys
```

**Features**:

```
Access control (RBAC)
Auditing (logging all access)
Automatic rotation
Soft delete (recover within 90 days)
Azure Private Link (private access)
```

**Example**:

```python
from azure.keyvault.secrets import SecretClient
from azure.identity import DefaultAzureCredential

# Create client
credential = DefaultAzureCredential()
client = SecretClient(
    vault_url="https://myvault.vault.azure.net/",
    credential=credential
)

# Set secret
client.set_secret("dbpassword", "mySecurePassword123")

# Get secret
secret = client.get_secret("dbpassword")
print(secret.value)

# List secrets
for secret_properties in client.list_properties_of_secrets():
    print(secret_properties.name)

# Delete secret
client.begin_delete_secret("dbpassword")
```

**Pricing**:

```
Operations — $0.03 per 10,000 operations
Per certificate — $1/month
HSM-backed keys — Additional cost
```

---

## 10. What is Azure Container Registry (ACR)?

**Q: Explain container registry**

A: Manage Docker and OCI container images. Private container registry.

**Features**:

```
Private registry (control access)
Geo-replication (multi-region)
Webhook (trigger on image push)
Image scanning (vulnerability detection)
Build support (build images in cloud)
```

**Using ACR**:

```bash
# Login
az acr login --name myregistry

# Tag image
docker tag myapp:latest myregistry.azurecr.io/myapp:latest

# Push image
docker push myregistry.azurecr.io/myapp:latest

# List images
az acr repository list --name myregistry

# Build image
az acr build \
    --registry myregistry \
    --image myapp:latest \
    --file Dockerfile .
```

**Webhook example** (trigger CI/CD):

```bash
az acr webhook create \
    --registry myregistry \
    --name mywebhook \
    --actions push \
    --uri https://myserver.com/webhook
```

---

## 11. What is Azure Kubernetes Service (AKS)?

**Q: Explain Kubernetes orchestration**

A: Managed Kubernetes service. Deploy and manage containerized applications.

**Features**:

```
Managed control plane (Azure handles master)
Auto-scaling (nodes and pods)
Built-in monitoring (with Container Insights)
Azure DevOps integration
GPU support
```

**Creating cluster**:

```bash
# Create AKS cluster
az aks create \
    --resource-group myResourceGroup \
    --name myCluster \
    --node-count 3 \
    --vm-set-type VirtualMachineScaleSets \
    --generate-ssh-keys

# Get credentials
az aks get-credentials \
    --resource-group myResourceGroup \
    --name myCluster

# Deploy application
kubectl apply -f deployment.yaml

# Scale nodes
az aks scale \
    --resource-group myResourceGroup \
    --name myCluster \
    --node-count 5
```

**Deployment example**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: myregistry.azurecr.io/myapp:latest
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 500m
              memory: 512Mi
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 80
  selector:
    app: myapp
```

---

## 12. What is Azure Functions?

**Q: Explain serverless computing**

A: Run code without managing servers. Event-driven, auto-scaling.

**Triggers**:

```
HTTP — Web requests
Timer — Scheduled tasks
Storage — Blob/Queue changes
Cosmos DB — Document changes
Queue/Topic — Message processing
Service Bus — Message queue
Event Hub — Streaming data
```

**Example: HTTP trigger**:

```python
import azure.functions as func
import json

def main(req: func.HttpRequest) -> func.HttpResponse:
    name = req.params.get('name')
    if not name:
        try:
            req_body = req.get_json()
            name = req_body.get('name')
        except ValueError:
            pass

    if name:
        return func.HttpResponse(
            json.dumps({"message": f"Hello {name}"}),
            status_code=200
        )
    else:
        return func.HttpResponse(
            "Please pass a name in the query string or request body",
            status_code=400
        )
```

**Example: Timer trigger**:

```python
import azure.functions as func
import datetime

def main(mytimer: func.TimerRequest) -> None:
    if mytimer.past_due:
        print('The timer is past due!')

    print(f'Python timer trigger function executed at {datetime.datetime.utcnow()}')
```

**Pricing**:

```
Free tier: 1 million requests + 400,000 GB-seconds/month
Pay-as-you-go:
- $0.20 per 1 million requests
- $0.000016667 per GB-second
```

---

## 13. What is Azure DevOps?

**Q: Explain CI/CD pipeline**

A: Tools for planning, developing, deploying, and monitoring applications.

**Components**:

```
Azure Repos — Git repositories
Azure Pipelines — CI/CD
Azure Boards — Work tracking
Azure Artifacts — Package management
Azure Test Plans — Manual testing
```

**Pipeline example**:

```yaml
trigger:
  - main

pool:
  vmImage: "ubuntu-latest"

steps:
  - task: UseDotNet@2
    inputs:
      version: "6.0.x"

  - task: DotNetCoreCLI@2
    inputs:
      command: "restore"

  - task: DotNetCoreCLI@2
    inputs:
      command: "build"
      arguments: "--configuration Release"

  - task: DotNetCoreCLI@2
    inputs:
      command: "test"

  - task: DotNetCoreCLI@2
    inputs:
      command: "publish"
      publishWebProjects: true
      arguments: "--configuration Release --output $(Build.ArtifactStagingDirectory)"

  - task: PublishBuildArtifacts@1
    inputs:
      pathToPublish: "$(Build.ArtifactStagingDirectory)"
      artifactName: "drop"

  - task: AzureWebApp@1
    inputs:
      azureSubscription: "myServiceConnection"
      appType: "webAppLinux"
      appName: "mywebapp"
      package: "$(Build.ArtifactStagingDirectory)/**/*.zip"
```

---

## 14. What is Azure Monitor and Application Insights?

**Q: Explain monitoring and observability**

A: Comprehensive monitoring of applications and infrastructure.

**Azure Monitor**:

```
Metrics — Performance data
Logs — Event data
Alerts — Notifications on thresholds
Workbooks — Custom dashboards
```

**Application Insights**:

```
Application performance monitoring (APM)
Distributed tracing
Dependency tracking
Custom events
Failed request tracing
Exception tracking
```

**Example**:

```python
from applicationinsights import TelemetryClient
from applicationinsights.common import Request

# Create client
tc = TelemetryClient('instrumentation-key')

# Track event
tc.track_event('MyEvent', {'property1': 'value1'})

# Track metric
tc.track_metric('MyMetric', 42)

# Track exception
try:
    1/0
except:
    tc.track_exception()

# Track request
request = Request("POST", "http://example.com/api/users")
request.success = True
request.response_code = 200
tc.track_request(request)

# Flush
tc.flush()
```

---

## 15. What is Azure Load Balancer and Application Gateway?

**Q: Load balancing in Azure**

A:

**Load Balancer** (Layer 4 — Network):

```
- Distribute traffic at network level
- Supports UDP and TCP
- High performance, low latency
- Good for non-HTTP protocols
- NLB equivalent in AWS
```

**Application Gateway** (Layer 7 — Application):

```
- Route based on hostnames, paths, ports
- SSL termination
- Web Application Firewall (WAF)
- Session affinity
- Multi-site hosting
- ALB equivalent in AWS
```

**Creating Application Gateway**:

```bash
az network application-gateway create \
    --name myGateway \
    --resource-group myResourceGroup \
    --vnet-name myVNet \
    --subnet mySubnet \
    --capacity 2 \
    --sku Standard_v2 \
    --http-settings-cookie-based-affinity Disabled \
    --frontend-port 80 \
    --http-settings-port 80 \
    --http-settings-protocol Http \
    --public-ip-address myPublicIP \
    --cert-file path/to/cert.pfx \
    --cert-password password
```

---

## 16. What is Azure Resource Manager (ARM)?

**Q: Infrastructure as Code**

A: Manage Azure resources declaratively using templates.

**ARM template structure**:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string",
      "defaultValue": "myVM"
    }
  },
  "variables": {
    "storageAccountName": "[concat('stor', uniqueString(resourceGroup().id))]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2021-04-01",
      "name": "[variables('storageAccountName')]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "StorageV2",
      "properties": {}
    },
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2021-03-01",
      "name": "[parameters('vmName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "hardwareProfile": {
          "vmSize": "Standard_B2s"
        }
      }
    }
  ],
  "outputs": {
    "storageAccountId": {
      "type": "string",
      "value": "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
    }
  }
}
```

**Deploying**:

```bash
az deployment group create \
    --resource-group myResourceGroup \
    --template-file template.json \
    --parameters vmName=myVM
```

---

## 17. What is cost optimization?

**Q: Reduce Azure costs**

A:

**Strategies**:

**1. Reserved Instances**:

```
1-year or 3-year commitment
30-72% discount
Good for predictable workloads
```

**2. Spot instances**:

```
Up to 90% discount
Can be evicted
Good for fault-tolerant workloads
Batch processing
```

**3. Sizing**:

```
Use Azure Advisor recommendations
Scale down unused resources
Right-size VMs based on metrics
```

**4. Storage optimization**:

```
Use cheaper storage tiers
Delete old snapshots
Archive old data
Lifecycle policies
```

**5. Network optimization**:

```
Use zone-redundant storage (cheaper than geo-redundant)
Bandwidth egress costs
Virtual network peering instead of VPN
```

**Cost analysis**:

```bash
# Get cost by resource group
az costmanagement query \
    --timeframe MonthToDate \
    --type Usage \
    --dataset granularity=Daily \
    dataset grouping=ResourceGroup
```

---

## 18. What is disaster recovery and business continuity?

**Q: Plan for failures**

A:

**Backup strategies**:

```
Azure Backup — Backup VMs, databases
Azure Site Recovery — Replicate to secondary region
Geo-redundant storage — Automatic replication
```

**Recovery metrics**:

```
RTO — Recovery Time Objective (downtime tolerance)
RPO — Recovery Point Objective (data loss tolerance)
```

**Example: Site Recovery**:

```bash
# Create recovery services vault
az backup vault create \
    --name myVault \
    --resource-group myResourceGroup

# Enable replication
az site-recovery fabric create \
    --resource-group myResourceGroup \
    --vault-name myVault \
    --name primaryFabric \
    --type Azure
```

---

## 19. What is security best practices?

**Q: Secure Azure infrastructure**

A:

**Identity & Access**:

```
- Use Managed Identity for resources
- Enable MFA
- RBAC (Role-Based Access Control)
- Principle of least privilege
- Regular access reviews
```

**Network Security**:

```
- Use VNet (don't use default)
- Private subnets for sensitive resources
- NSG rules (firewall)
- Service Endpoints/Private Links
- DDoS Protection
```

**Data Protection**:

```
- Encryption at rest (Storage Service Encryption)
- Encryption in transit (TLS)
- Azure Disk Encryption
- SQL Database encryption
- Key Vault for keys
```

**Compliance**:

```
- Azure Compliance Manager
- Policy enforcement (Azure Policy)
- Audit logging (Azure Monitor)
- Regular security assessments
```

---

## 20. What is real-world example?

**Q: Complete Azure application**

A: E-commerce platform

```
Frontend:
Azure App Service (Web Apps)
Static Web Apps (CDN + hosting)
Azure Front Door (global load balancing)

Backend:
App Service (APIs)
Azure Functions (serverless operations)
Logic Apps (workflow automation)

Database:
Azure SQL Database (relational)
Azure Cosmos DB (product catalog)
Azure Cache for Redis (sessions/cache)

Storage:
Blob Storage (product images)
CDN (serve images globally)
File Share (shared content)

Security:
Azure Active Directory (authentication)
Key Vault (secrets, encryption keys)
Azure DDoS Protection
Web Application Firewall

DevOps:
Azure DevOps (CI/CD)
Azure Container Registry (images)
Azure Kubernetes Service (orchestration)

Monitoring:
Application Insights
Azure Monitor
Log Analytics

Backup:
Azure Backup
Azure Site Recovery
Geo-redundant storage
```

**ARM Template (simplified)**:

```json
{
  "resources": [
    {
      "type": "Microsoft.Web/serverfarms",
      "name": "myAppServicePlan",
      "apiVersion": "2021-01-15",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "B1",
        "tier": "Basic"
      }
    },
    {
      "type": "Microsoft.Web/sites",
      "name": "myWebApp",
      "apiVersion": "2021-01-15",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', 'myAppServicePlan')]"
      ],
      "properties": {
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', 'myAppServicePlan')]"
      }
    },
    {
      "type": "Microsoft.Sql/servers",
      "name": "myServer",
      "apiVersion": "2019-06-01",
      "location": "[resourceGroup().location]",
      "properties": {
        "administratorLogin": "azureuser",
        "administratorLoginPassword": "[listKeys(resourceId('Microsoft.KeyVault/vaults/secrets', 'myVault', 'sqlPassword'), '2021-04-01').value]"
      }
    }
  ]
}
```

---

## Azure vs AWS Comparison

**Azure Advantages**:

```
- Better for enterprises (Office 365, Active Directory)
- Hybrid cloud (on-premises integration)
- Enterprise agreement discounts
- Windows/SQL Server optimization
- AI services integration
```

**AWS Advantages**:

```
- Larger market share
- More services overall
- Pricing transparency
- Larger community
- More mature
```

**Service Mapping**:

```
AWS          Azure
EC2          Virtual Machines
Lambda       Azure Functions
S3           Blob Storage
RDS          Azure SQL Database
DynamoDB     Cosmos DB
VPC          VNet
IAM          Entra ID/RBAC
CloudFormation → ARM Templates
CodePipeline → Azure Pipelines
CloudWatch   → Azure Monitor
```

---

## Azure Interview Tips

1. **Core services** — VMs, App Service, SQL Database, Functions
2. **Networking** — VNet, NSG, App Gateway, Load Balancer
3. **Security** — Entra ID, RBAC, Key Vault, encryption
4. **Storage** — Blob, Files, Queues
5. **Databases** — SQL, Cosmos DB, when to use each
6. **Containers** — ACR, AKS
7. **Serverless** — Functions, Logic Apps
8. **DevOps** — Azure DevOps, pipelines, IaC
9. **Monitoring** — Application Insights, Azure Monitor
10. **Cost optimization** — Reserved instances, right-sizing
11. **Disaster recovery** — Backup, replication, redundancy
12. **Identity** — Entra ID, Managed Identity
13. **Hybrid** — ExpressRoute, hybrid integration
14. **ARM Templates** — Infrastructure as Code
15. **Real projects** — Production Azure experience
