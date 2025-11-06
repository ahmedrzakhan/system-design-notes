# Bicep Interview Questions & Answers

## 1. What is Bicep?

**Q: Explain Bicep and why use it?**

A: Bicep is a domain-specific language (DSL) for declaring Azure infrastructure. It's a cleaner, simpler alternative to ARM (Azure Resource Manager) JSON templates.

**Key characteristics**:

- Declarative syntax (what you want, not how)
- Simpler than ARM JSON (less verbose)
- Strongly typed (compile-time validation)
- Modular (reusable components)
- Transpiles to ARM JSON
- Full Azure support
- Intellisense and validation
- Parameter and variable support
- String interpolation
- Loops and conditions
- Modules for composition

**vs ARM JSON**:

```
ARM JSON:
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2021-04-01",
      "name": "[concat('stor', uniqueString(resourceGroup().id))]",
      ...
    }
  ]
}

Bicep:
param storageAccountName string = 'stor${uniqueString(resourceGroup().id)}'

resource storageAccount 'Microsoft.Storage/storageAccounts@2021-04-01' = {
  name: storageAccountName
  location: resourceGroup().location
  kind: 'StorageV2'
  sku: {
    name: 'Standard_LRS'
  }
}
```

**Benefits**:

- 30-40% less code than ARM JSON
- Better readability
- Easier debugging
- Type checking
- Modular and reusable
- Same deployment power as ARM

---

## 2. What is basic Bicep syntax?

**Q: Explain Bicep language basics**

A:

**Parameters** (input values):

```bicep
// Simple parameter
param location string

// Parameter with default
param environment string = 'dev'

// Parameter with allowed values
param vmSize string = 'Standard_B2s' = [
  'Standard_B1s'
  'Standard_B2s'
  'Standard_B2ms'
]

// Parameter with description and metadata
@description('The Azure region for resources')
@minLength(1)
@maxLength(90)
param location string = resourceGroup().location

// Complex parameter (object)
param tags object = {
  environment: 'dev'
  owner: 'admin'
}
```

**Variables** (local values):

```bicep
// Simple variable
var storageAccountName = 'stor${uniqueString(resourceGroup().id)}'

// Variable with expression
var vmName = '${appName}-vm-${environment}'

// Variable from parameter
var location = param.location

// Variable as array
var vmSizes = [
  'Standard_B1s'
  'Standard_B2s'
]

// Variable as object
var tags = {
  environment: environment
  createdDate: utcNow()
}
```

**Outputs** (return values):

```bicep
// Simple output
output storageAccountId string = storageAccount.id

// Output with description
@description('The storage account connection string')
output storageConnectionString string = 'DefaultEndpointsProtocol=https;AccountName=${storageAccount.name};...'

// Output object
output resourceIds object = {
  storageId: storageAccount.id
  vmId: virtualMachine.id
}
```

**String interpolation**:

```bicep
// Simple interpolation
param environment string = 'dev'
var appName = 'myapp'
var resourceName = '${appName}-${environment}-vm'  // myapp-dev-vm

// With expressions
var timestamp = utcNow()
var logMessage = 'Resource created at ${timestamp}'

// Nested
var id = '${resourceGroup().id}/${resourceName}'
```

---

## 3. What are resources?

**Q: Declare Azure resources in Bicep**

A: Resources are the infrastructure you want to deploy.

**Resource declaration**:

```bicep
// Basic resource
resource <symbolicName> '<resourceType>@<apiVersion>' = {
  name: '<name>'
  location: '<location>'
  properties: {
    // configuration
  }
}

// Storage account example
resource storageAccount 'Microsoft.Storage/storageAccounts@2021-06-01' = {
  name: storageAccountName
  location: location
  kind: 'StorageV2'
  sku: {
    name: 'Standard_LRS'
  }
  properties: {
    accessTier: 'Hot'
  }
  tags: {
    environment: environment
  }
}

// Virtual Network
resource vnet 'Microsoft.Network/virtualNetworks@2021-02-01' = {
  name: 'myVNet'
  location: location
  properties: {
    addressSpace: {
      addressPrefixes: [
        '10.0.0.0/16'
      ]
    }
    subnets: [
      {
        name: 'subnet1'
        properties: {
          addressPrefix: '10.0.1.0/24'
        }
      }
    ]
  }
}

// Subnet reference
resource subnet 'Microsoft.Network/virtualNetworks/subnets@2021-02-01' = {
  parent: vnet
  name: 'subnet2'
  properties: {
    addressPrefix: '10.0.2.0/24'
  }
}
```

**Resource references**:

```bicep
// Reference existing resource
resource existingStorage 'Microsoft.Storage/storageAccounts@2021-06-01' existing = {
  name: existingStorageAccountName
}

// Access resource properties
var storageName = existingStorage.name
var storageId = existingStorage.id
var storageKey = existingStorage.listKeys().keys[0].value

// Use in another resource
resource vnet 'Microsoft.Network/virtualNetworks@2021-02-01' = {
  name: 'myVNet'
  location: location
  properties: {
    addressSpace: {
      addressPrefixes: [
        '10.0.0.0/16'
      ]
    }
  }
  dependsOn: [
    existingStorage  // Explicit dependency
  ]
}
```

---

## 4. What are symbolic names and dependencies?

**Q: Resource relationships and ordering**

A:

**Symbolic names**:

```bicep
// Declare resource with symbolic name
resource storageAccount 'Microsoft.Storage/storageAccounts@2021-06-01' = {
  name: 'mystorageaccount'
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
}

// Reference by symbolic name
var storageId = storageAccount.id
var storageName = storageAccount.name

// Use in parent relationship
resource container 'Microsoft.Storage/storageAccounts/blobServices/containers@2021-06-01' = {
  parent: storageAccount
  name: 'container1'
}
```

**Implicit dependencies**:

```bicep
// Bicep automatically detects dependencies
resource vnet 'Microsoft.Network/virtualNetworks@2021-02-01' = {
  name: 'myVNet'
  location: location
  properties: {
    addressSpace: {
      addressPrefixes: ['10.0.0.0/16']
    }
  }
}

// This depends on vnet automatically (references it)
resource nic 'Microsoft.Network/networkInterfaces@2021-02-01' = {
  name: 'myNic'
  location: location
  properties: {
    ipConfigurations: [
      {
        name: 'ipconfig1'
        properties: {
          subnet: {
            id: '${vnet.id}/subnets/subnet1'  // Reference creates dependency
          }
        }
      }
    ]
  }
}
```

**Explicit dependencies**:

```bicep
resource storageAccount 'Microsoft.Storage/storageAccounts@2021-06-01' = {
  name: 'mystorageaccount'
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
}

resource appServicePlan 'Microsoft.Web/serverfarms@2021-01-15' = {
  name: 'myAppServicePlan'
  location: location
  sku: {
    name: 'B1'
    tier: 'Basic'
  }
  dependsOn: [
    storageAccount  // Explicit dependency
  ]
}
```

---

## 5. What are loops and conditions?

**Q: Iteration and conditional logic**

A:

**For loops** (iteration):

```bicep
// Loop over array
param vmCount int = 3
param vmsizes array = ['Standard_B1s', 'Standard_B2s', 'Standard_B2ms']

resource vm 'Microsoft.Compute/virtualMachines@2021-03-01' = [for i in range(0, vmCount): {
  name: 'vm-${i}'
  location: location
  properties: {
    hardwareProfile: {
      vmSize: vmSizes[i]
    }
  }
}]

// Loop with index
resource nics 'Microsoft.Network/networkInterfaces@2021-02-01' = [for (nic, index) in vmConfigs: {
  name: '${nic.name}-nic'
  location: location
  properties: {
    ipConfigurations: [
      {
        name: 'ipconfig1'
        properties: {
          privateIPAddress: nic.privateIP
        }
      }
    ]
  }
}]

// Nested loop
@minValue(1)
@maxValue(10)
param replicaCount int = 3

resource instances 'Microsoft.Compute/virtualMachines@2021-03-01' = [for region in regions: {
  for i in range(0, replicaCount): {
    name: '${region}-vm-${i}'
    location: region
    // ...
  }
}]
```

**If conditions**:

```bicep
param deployStorage bool = true
param environment string = 'dev'

// Conditional resource
resource storageAccount 'Microsoft.Storage/storageAccounts@2021-06-01' = if (deployStorage) {
  name: 'mystorageaccount'
  location: location
  kind: 'StorageV2'
  sku: {
    name: 'Standard_LRS'
  }
}

// Conditional property
resource appServicePlan 'Microsoft.Web/serverfarms@2021-01-15' = {
  name: 'myAppServicePlan'
  location: location
  sku: {
    name: environment == 'prod' ? 'P1V2' : 'B1'  // Ternary operator
    tier: environment == 'prod' ? 'PremiumV2' : 'Basic'
  }
}

// Conditional in array
param tags object = {
  environment: environment
  costCenter: environment == 'prod' ? 'prod-cost' : 'dev-cost'
}

// Optional parameters based on condition
resource vm 'Microsoft.Compute/virtualMachines@2021-03-01' = {
  name: 'myVM'
  location: location
  properties: {
    hardwareProfile: {
      vmSize: 'Standard_B2s'
    }
    osProfile: environment == 'prod' ? {
      computerName: 'prodvm'
      adminUsername: 'azureuser'
      adminPassword: vmPassword  // Only in prod
    } : null
  }
}
```

---

## 6. What are functions?

**Q: Built-in and custom functions**

A:

**Built-in functions**:

```bicep
// String functions
var upper = toUpper('hello')  // 'HELLO'
var lower = toLower('HELLO')  // 'hello'
var substring = substring('hello', 0, 3)  // 'hel'
var indexOf = indexOf('hello', 'l')  // 2
var contains = contains('hello', 'll')  // true

// Array functions
var array = ['a', 'b', 'c']
var arrayLength = length(array)  // 3
var first = first(array)  // 'a'
var last = last(array)  // 'c'
var joined = join(array, ',')  // 'a,b,c'
var filtered = filter(array, x => x != 'b')  // ['a', 'c']

// Object functions
var obj = { name: 'John', age: 30 }
var keys = keys(obj)  // ['name', 'age']
var values = values(obj)  // ['John', 30]

// Numeric functions
var max = max(1, 2, 3)  // 3
var min = min(1, 2, 3)  // 1
var sum = sum(1, 2, 3)  // 6

// Resource functions
var uniqueId = uniqueString(resourceGroup().id)
var resourceGroupId = resourceGroup().id
var subscriptionId = subscription().subscriptionId

// Deployment functions
var deployment = deployment()  // Current deployment info
var environmentVersion = environment().name  // 'AzureCloud'
```

**Custom functions**:

```bicep
// Define function
@minLength(1)
func generateStorageName(environment string, appName string): string => 'stor${appName}${environment}${uniqueString(resourceGroup().id)}'

// Use function
param appName string
param environment string

var storageName = generateStorageName(environment, appName)

resource storageAccount 'Microsoft.Storage/storageAccounts@2021-06-01' = {
  name: storageName
  location: location
  kind: 'StorageV2'
  sku: {
    name: 'Standard_LRS'
  }
}

// Function with complex logic
func buildTags(environment string, owner string, costCenter string): object => {
  return {
    environment: environment
    owner: owner
    costCenter: costCenter
    createdDate: utcNow('u')
    version: '1.0'
  }
}

var resourceTags = buildTags('dev', 'admin', 'it-dept')

// Recursive function
@export()
func fibonacci(n int): int => n <= 1 ? n : fibonacci(n - 1) + fibonacci(n - 2)
```

---

## 7. What are modules?

**Q: Code reuse and composition**

A: Modules are reusable Bicep files that define sets of resources.

**Module structure**:

```bicep
// storage.bicep - Reusable storage module
param location string = resourceGroup().location
param accountName string
param sku string = 'Standard_LRS'
param kind string = 'StorageV2'

param tags object = {}

resource storageAccount 'Microsoft.Storage/storageAccounts@2021-06-01' = {
  name: accountName
  location: location
  kind: kind
  sku: {
    name: sku
  }
  properties: {
    accessTier: 'Hot'
  }
  tags: tags
}

output storageId string = storageAccount.id
output storageName string = storageAccount.name
output storageKey string = storageAccount.listKeys().keys[0].value
```

**Using modules**:

```bicep
// main.bicep
param location string = resourceGroup().location
param environment string = 'dev'

// Use module
module storage 'storage.bicep' = {
  name: 'storageDeployment'
  params: {
    location: location
    accountName: 'mystorageaccount'
    sku: environment == 'prod' ? 'Standard_GRS' : 'Standard_LRS'
    kind: 'StorageV2'
    tags: {
      environment: environment
      owner: 'admin'
    }
  }
}

// Use module output
var storageId = storage.outputs.storageId
var storageName = storage.outputs.storageName

// Another module
module vnet 'vnet.bicep' = {
  name: 'vnetDeployment'
  params: {
    location: location
    addressPrefix: '10.0.0.0/16'
  }
}

// Loop over modules
param locations array = ['eastus', 'westus']

module storageByRegion 'storage.bicep' = [for region in locations: {
  name: 'storage-${region}'
  params: {
    location: region
    accountName: 'stor${region}${uniqueString(resourceGroup().id)}'
  }
}]
```

**Module scope**:

```bicep
// Deploy to resource group (default)
module rg1Resources 'resources.bicep' = {
  name: 'deployment1'
  params: {}
}

// Deploy to subscription
module subscriptionLevel 'subscription.bicep' = {
  name: 'subscriptionDeployment'
  scope: subscription()
  params: {}
}

// Deploy to management group
module mgmtLevel 'management.bicep' = {
  name: 'managementDeployment'
  scope: managementGroup('my-mg')
  params: {}
}

// Deploy to specific resource group
module crossRgResources 'resources.bicep' = {
  name: 'crossRgDeployment'
  scope: resourceGroup('mySubscription', 'otherResourceGroup')
  params: {}
}
```

---

## 8. What are variables and symbolic references?

**Q: Variable types and expressions**

A:

**Variable types**:

```bicep
// Primitive types
var stringVar: string = 'hello'
var intVar: int = 42
var boolVar: bool = true

// Array types
var stringArray: array = ['a', 'b', 'c']
var intArray: int[] = [1, 2, 3]
var mixedArray: array = ['string', 42, true]

// Object types
var tagObject: object = {
  environment: 'dev'
  owner: 'admin'
}

// Union types
var flexible: string | int = 42  // Can be string or int

// Nullable types (can be null)
var nullable: string? = null

// Type inference (Bicep infers type)
var inferred = 'hello'  // Inferred as string
var calculated = 1 + 2  // Inferred as int
var combined = '${variable}'  // Inferred as string
```

**Symbolic references**:

```bicep
// Reference resource by symbolic name
resource storageAccount 'Microsoft.Storage/storageAccounts@2021-06-01' = {
  name: 'mystorageaccount'
  location: location
  kind: 'StorageV2'
  sku: {
    name: 'Standard_LRS'
  }
}

// Access properties
var storageName = storageAccount.name  // Resource name
var storageId = storageAccount.id  // Full resource ID
var storageLocation = storageAccount.location  // Location
var storageType = storageAccount.type  // Microsoft.Storage/storageAccounts

// Call function on resource
var storageKeys = storageAccount.listKeys()

// Use in expressions
resource container 'Microsoft.Storage/storageAccounts/blobServices/containers@2021-06-01' = {
  parent: storageAccount  // Reference by symbolic name
  name: 'container1'
}

// Computed properties
var storageUri = 'https://${storageAccount.name}.blob.core.windows.net'
var connectionString = 'BlobEndpoint=${storageUri};SharedAccessSignature=${storageAccount.listAccountSas().value}'
```

---

## 9. What are metadata and validation?

**Q: Add constraints and documentation**

A:

**Parameter decorators**:

```bicep
// Description
@description('The Azure region for resources')
param location string

// Length constraints
@minLength(1)
@maxLength(90)
param name string

// Numeric constraints
@minValue(1)
@maxValue(100)
param count int

// Allowed values
@allowed([
  'dev'
  'staging'
  'prod'
])
param environment string

// Pattern (regex)
@minLength(5)
@maxLength(24)
@metadata({
  description: 'Storage account name must be 5-24 characters'
})
param storageAccountName string

// Common metadata
@export()  // Export for use in other Bicep files
param exportedValue string

// Multiple decorators
@description('VM size')
@allowed([
  'Standard_B1s'
  'Standard_B2s'
  'Standard_B2ms'
])
param vmSize string = 'Standard_B2s'
```

**Variable decorators**:

```bicep
// Internal variable with metadata
@metadata({
  description: 'Generated from parameters'
})
var computedName = '${appName}-${environment}'

// Export variable for use in modules
@export()
var baseImageUri = 'https://image.registry.azure.io/base:latest'
```

**Output decorators**:

```bicep
@description('The ID of the created storage account')
output storageAccountId string = storageAccount.id

@description('The connection string for the storage account')
@secure()  // Hide in logs
output connectionString string = 'DefaultEndpointsProtocol=https;...'
```

---

## 10. What are advanced patterns?

**Q: Complex Bicep scenarios**

A:

**User-defined types**:

```bicep
// Define custom type
type vmConfig = {
  name: string
  size: string
  imagePublisher: string
  imageOffer: string
  imageSku: string
}

// Use custom type
param vmConfigs vmConfig[] = []

resource vm 'Microsoft.Compute/virtualMachines@2021-03-01' = [for config in vmConfigs: {
  name: config.name
  location: location
  properties: {
    hardwareProfile: {
      vmSize: config.size
    }
    storageProfile: {
      imageReference: {
        publisher: config.imagePublisher
        offer: config.imageOffer
        sku: config.imageSku
        version: 'latest'
      }
    }
  }
}]
```

**Bicep parameter files**:

```bicep
// main.bicep
param location string = resourceGroup().location
param environment string
param vmCount int
param tags object

resource vm 'Microsoft.Compute/virtualMachines@2021-03-01' = [for i in range(0, vmCount): {
  name: 'vm-${i}'
  location: location
  tags: tags
  // ...
}]

// parameters.json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "value": "eastus"
    },
    "environment": {
      "value": "prod"
    },
    "vmCount": {
      "value": 3
    },
    "tags": {
      "value": {
        "environment": "production",
        "owner": "admin"
      }
    }
  }
}
```

**Conditional deployment with dependencies**:

```bicep
param deployDatabase bool = true
param deployAppService bool = true

// Conditional resource
resource database 'Microsoft.Sql/servers@2019-06-01' = if (deployDatabase) {
  name: 'sqlserver-${uniqueString(resourceGroup().id)}'
  location: location
  properties: {
    administratorLogin: sqlAdminLogin
    administratorLoginPassword: sqlAdminPassword
  }
}

// Depends on database only if deployed
resource appService 'Microsoft.Web/sites@2021-01-15' = if (deployAppService) {
  name: 'app-${uniqueString(resourceGroup().id)}'
  location: location
  properties: {
    serverFarmId: appServicePlan.id
    connectionStrings: deployDatabase ? [
      {
        name: 'DefaultConnection'
        connectionString: deployDatabase ? 'Server=tcp:${database.properties.fullyQualifiedDomainName}...' : ''
      }
    ] : []
  }
  dependsOn: deployDatabase ? [
    database
  ] : []
}
```

---

## 11. What is deployment?

**Q: Deploy Bicep templates**

A:

**Via Azure CLI**:

```bash
# Deploy to resource group
az deployment group create \
  --name myDeployment \
  --resource-group myResourceGroup \
  --template-file main.bicep \
  --parameters @parameters.json

# Deploy with inline parameters
az deployment group create \
  --name myDeployment \
  --resource-group myResourceGroup \
  --template-file main.bicep \
  --parameters location=eastus environment=prod

# Validate before deployment
az deployment group validate \
  --resource-group myResourceGroup \
  --template-file main.bicep \
  --parameters @parameters.json

# What-if (preview changes)
az deployment group what-if \
  --resource-group myResourceGroup \
  --template-file main.bicep \
  --parameters @parameters.json
```

**Via PowerShell**:

```powershell
# Deploy
New-AzResourceGroupDeployment `
  -Name myDeployment `
  -ResourceGroupName myResourceGroup `
  -TemplateFile main.bicep `
  -TemplateParameterFile parameters.json

# What-if
Get-AzResourceGroupDeploymentWhatIfResult `
  -ResourceGroupName myResourceGroup `
  -TemplateFile main.bicep
```

**Via ARM API**:

```bash
# First compile Bicep to ARM JSON
az bicep build --file main.bicep

# Then deploy using ARM API
curl -X PUT \
  -H "Authorization: Bearer $token" \
  -H "Content-Type: application/json" \
  -d @template.json \
  https://management.azure.com/subscriptions/{subscriptionId}/resourcegroups/{resourceGroup}/providers/microsoft.resources/deployments/{deploymentName}?api-version=2021-04-01
```

---

## 12. What are best practices?

**Q: Bicep design patterns**

A:

**Naming conventions**:

```bicep
// Parameters: descriptive, lowercase with camelCase
param storageAccountName string
param environmentName string
param vnetAddressPrefix string

// Variables: descriptive, lowercase with camelCase
var uniqueSuffix = uniqueString(resourceGroup().id)
var resourceTags = { environment: environment }

// Resources: descriptive, typically kebab-case in Azure
resource storageAccount 'Microsoft.Storage/storageAccounts@2021-06-01' = {
  name: 'stor-${appName}-${environment}-${uniqueSuffix}'
}

// Outputs: descriptive, lowercase with camelCase
output storageAccountId string = storageAccount.id
output storageAccountName string = storageAccount.name
```

**Parameter organization**:

```bicep
// 1. Required parameters first
param location string

// 2. Optional parameters with defaults
param environment string = 'dev'
param tags object = {}

// 3. Parameters with constraints
@minLength(3)
@maxLength(24)
param storageAccountName string

// 4. Documented parameters
@description('The size of the virtual machine')
@allowed(['Standard_B1s', 'Standard_B2s'])
param vmSize string = 'Standard_B1s'
```

**Module structure**:

```bicep
// Organize modules by function
// modules/
//   ├── storage.bicep
//   ├── networking/
//   │   ├── vnet.bicep
//   │   └── nsg.bicep
//   └── compute/
//       ├── vm.bicep
//       └── appservice.bicep

// main.bicep uses modules
module storage 'modules/storage.bicep' = {
  name: 'storageDeployment'
  params: {
    location: location
    accountName: storageAccountName
  }
}

module networking 'modules/networking/vnet.bicep' = {
  name: 'networkingDeployment'
  params: {
    location: location
    vnetName: vnetName
  }
}
```

**Error handling and defaults**:

```bicep
// Always provide defaults for optional parameters
param environment string = 'dev'
param location string = resourceGroup().location
param tags object = {
  environment: environment
  createdDate: utcNow('u')
}

// Use validation
@allowed(['dev', 'staging', 'prod'])
param environment string = 'dev'

// Provide meaningful defaults
param vmCount int = 1
@minValue(1)
@maxValue(100)
param vmCount int = 1

// Compute derived values
var resourceNamePrefix = '${appName}-${environment}'
var uniqueSuffix = uniqueString(resourceGroup().id)
```

---

## 13. What is testing and validation?

**Q: Verify Bicep templates**

A:

**Bicep validation**:

```bash
# Validate syntax
az bicep validate --file main.bicep

# Validate with parameters
az bicep validate \
  --file main.bicep \
  --parameters @parameters.json

# Build (transpile to ARM JSON)
az bicep build --file main.bicep

# Build with output file
az bicep build \
  --file main.bicep \
  --outfile template.json
```

**Deployment validation**:

```bash
# What-if (preview changes without applying)
az deployment group what-if \
  --resource-group myResourceGroup \
  --template-file main.bicep \
  --parameters @parameters.json

# Validate deployment
az deployment group validate \
  --resource-group myResourceGroup \
  --template-file main.bicep \
  --parameters @parameters.json

# Test deployment (dry-run)
az deployment group create \
  --resource-group myResourceGroup \
  --template-file main.bicep \
  --parameters @parameters.json \
  --no-wait
```

**Testing patterns**:

```bicep
// Test different scenarios via different parameter files
// parameters-dev.json (development)
// parameters-staging.json (staging)
// parameters-prod.json (production)

// Run tests
for env in dev staging prod; do
  echo "Testing $env..."
  az deployment group validate \
    --resource-group "rg-$env" \
    --template-file main.bicep \
    --parameters "parameters-$env.json"
done
```

---

## 14. What is real-world example?

**Q: Complete Bicep template**

A: Multi-environment infrastructure

```bicep
// main.bicep - Complete production template
param location string = resourceGroup().location

@allowed(['dev', 'staging', 'prod'])
param environment string = 'dev'

@minLength(3)
@maxLength(24)
param appName string

param tags object = {
  environment: environment
  owner: 'devops'
  costCenter: 'it'
}

// Computed variables
var uniqueSuffix = uniqueString(resourceGroup().id)
var resourcePrefix = '${appName}-${environment}'
var storageAccountName = 'stor${appName}${environment}${uniqueSuffix}'

// Storage module
module storage 'modules/storage.bicep' = {
  name: 'storageDeployment'
  params: {
    location: location
    accountName: storageAccountName
    sku: environment == 'prod' ? 'Standard_GRS' : 'Standard_LRS'
    tags: tags
  }
}

// VNet module
module networking 'modules/networking.bicep' = {
  name: 'networkingDeployment'
  params: {
    location: location
    vnetName: '${resourcePrefix}-vnet'
    addressPrefix: '10.0.0.0/16'
    subnets: [
      {
        name: 'web-subnet'
        addressPrefix: '10.0.1.0/24'
      }
      {
        name: 'db-subnet'
        addressPrefix: '10.0.2.0/24'
      }
    ]
    tags: tags
  }
}

// Database
resource sqlServer 'Microsoft.Sql/servers@2019-06-01' = {
  name: '${resourcePrefix}-sql-${uniqueSuffix}'
  location: location
  properties: {
    administratorLogin: sqlAdminLogin
    administratorLoginPassword: sqlAdminPassword
  }
  tags: tags
}

resource sqlDatabase 'Microsoft.Sql/servers/databases@2019-06-01' = {
  parent: sqlServer
  name: '${appName}db'
  location: location
  sku: {
    name: environment == 'prod' ? 'S1' : 'S0'
    tier: 'Standard'
  }
}

// App Service
resource appServicePlan 'Microsoft.Web/serverfarms@2021-01-15' = {
  name: '${resourcePrefix}-plan'
  location: location
  sku: {
    name: environment == 'prod' ? 'P1V2' : 'B1'
    tier: environment == 'prod' ? 'PremiumV2' : 'Basic'
  }
}

resource webApp 'Microsoft.Web/sites@2021-01-15' = {
  name: '${resourcePrefix}-app-${uniqueSuffix}'
  location: location
  identity: {
    type: 'SystemAssigned'
  }
  properties: {
    serverFarmId: appServicePlan.id
    appSettings: [
      {
        name: 'STORAGE_ACCOUNT_NAME'
        value: storage.outputs.storageAccountName
      }
      {
        name: 'STORAGE_ACCOUNT_KEY'
        value: storage.outputs.storageAccountKey
      }
      {
        name: 'DATABASE_CONNECTION_STRING'
        value: 'Server=tcp:${sqlServer.properties.fullyQualifiedDomainName};Database=${sqlDatabase.name};'
      }
    ]
  }
  tags: tags
}

// Outputs
output storageAccountId string = storage.outputs.storageAccountId
output webAppUrl string = 'https://${webApp.properties.defaultHostName}'
output sqlServerFqdn string = sqlServer.properties.fullyQualifiedDomainName
```

---

## 15. What are Bicep vs Terraform?

**Q: Bicep vs Infrastructure as Code tools**

A:

**Bicep**:

```
Advantages:
- Native to Azure (first-class support)
- Simple syntax (DSL designed for Azure)
- Transpiles to ARM JSON
- Full Azure API coverage
- Intellisense in VS Code
- No state management complexity

Disadvantages:
- Azure only (not multi-cloud)
- Less mature than Terraform
- Smaller community
- Limited to Azure best practices
```

**Terraform**:

```
Advantages:
- Multi-cloud (AWS, GCP, Azure, etc.)
- Large community and resources
- More mature
- State management explicit
- HCL language more flexible

Disadvantages:
- State file management complexity
- Steeper learning curve
- Multi-cloud adds complexity
- Can be overkill for Azure-only shops
```

**Comparison**:

```
Feature         Bicep       Terraform
Cloud support   Azure only  Multi-cloud
Syntax          Easy        Medium
State mgmt      Built-in    Manual
Community       Growing     Mature
For Azure       Excellent   Good
For multi-cloud Not suitable Excellent
```

---

## Bicep Interview Tips

1. **Basic syntax** — Parameters, variables, outputs
2. **Resources** — Declaration, properties, references
3. **Dependencies** — Implicit and explicit
4. **Loops and conditions** — For, if
5. **Functions** — Built-in and custom
6. **Modules** — Reusability and composition
7. **Deployment** — CLI, PowerShell, ARM API
8. **Validation** — What-if, validate
9. **Best practices** — Naming, organization
10. **vs ARM JSON** — Why Bicep is better
11. **vs Terraform** — Use cases, trade-offs
12. **Production patterns** — Multi-environment, scaling
13. **Error handling** — Validation, constraints
14. **Performance** — Large-scale deployments
15. **Real projects** — Production experience
