# Azure Labs 45--50 --- VM Scale Sets & Azure Web Apps

## Topics

-   45. Lab --- Azure Virtual Machine Scale Set --- Scale Out
-   46. Azure Virtual Machine Scale Set --- Understanding the Scaling
        Process
-   47. Lab --- Azure Virtual Machine Scale Set --- Scale In
-   48. Lab --- Azure Virtual Machine Scale Sets --- Flexible
        Orchestration Mode
-   49. Introduction to Azure Web Apps
-   50. Lab --- Azure Web Apps

------------------------------------------------------------------------

# 45. Lab --- Azure Virtual Machine Scale Set --- Scale Out

## Objective

Learn how to increase the number of VM instances in an Azure Virtual
Machine Scale Set (VMSS).

### What is Scale Out?

**Scale out** means increasing the number of VM instances.

Example:

``` text
Before:
VM1
VM2

After:
VM1
VM2
VM3
VM4
```

So:

``` text
2 VMs → 4 VMs
```

Scale out is also called **horizontal scaling**.

------------------------------------------------------------------------

## Real-World Example

Suppose you have a web application running on two VMs:

``` text
                Users
                  |
                  v
            Load Balancer
              /       \
            VM1       VM2
```

During normal traffic, two VMs are enough.

During a sale:

``` text
                Users
                  |
                  v
            Load Balancer
          /      |      |      \
        VM1     VM2    VM3     VM4
```

Now four VMs handle the traffic.

This is scale out.

------------------------------------------------------------------------

## Portal Lab --- Manual Scale Out

### Step 1 --- Open VM Scale Sets

In Azure Portal:

``` text
Azure Portal
    ↓
Virtual Machine Scale Sets
    ↓
Select your VMSS
```

Example:

``` text
vmss-web-demo
```

### Step 2 --- Open Scaling

Go to:

``` text
VMSS
  ↓
Scaling
```

Depending on the portal experience, you may see instance count or
autoscale configuration.

### Step 3 --- Increase Instance Count

Suppose the current instance count is:

``` text
2
```

Change it to:

``` text
4
```

Then save/apply the change.

Azure creates additional instances.

### Step 4 --- Verify

Go to:

``` text
VMSS
  ↓
Instances
```

You should see:

``` text
Instance 1
Instance 2
Instance 3
Instance 4
```

------------------------------------------------------------------------

## Azure CLI --- Scale Out

List VMSS instances:

``` bash
az vmss list-instances \
  --resource-group rg-vmss-demo \
  --name vmss-web-demo \
  --output table
```

Set capacity to four instances:

``` bash
az vmss scale \
  --resource-group rg-vmss-demo \
  --name vmss-web-demo \
  --new-capacity 4
```

Verify:

``` bash
az vmss show \
  --resource-group rg-vmss-demo \
  --name vmss-web-demo \
  --query sku.capacity
```

Expected:

``` text
4
```

------------------------------------------------------------------------

## Important Difference --- Scale Out vs Scale Up

### Scale Out

``` text
2 VMs → 4 VMs
```

More instances.

### Scale Up

``` text
B2s → D4s
```

A larger VM size.

  Scaling      Meaning            Example
  ------------ ------------------ -----------
  Scale Out    Add instances      2 → 4 VMs
  Scale In     Remove instances   4 → 2 VMs
  Scale Up     Bigger VM          B2s → D4s
  Scale Down   Smaller VM         D4s → B2s

------------------------------------------------------------------------

## Autoscale Scale-Out Example

Suppose:

``` text
Minimum = 2
Default = 2
Maximum = 8
```

Rule:

``` text
CPU > 70%
→ Add 1 VM
```

Current:

``` text
2 VMs
CPU = 85%
```

The autoscale rule can increase capacity:

``` text
2 → 3
```

If CPU remains high and another scaling action is triggered:

``` text
3 → 4
```

It can continue only within the configured limits.

------------------------------------------------------------------------

## Key Learning

Scale out is useful when:

-   Traffic increases.
-   CPU becomes high.
-   Request volume increases.
-   More application capacity is required.
-   You want horizontal scaling.

------------------------------------------------------------------------

# 46. Azure Virtual Machine Scale Set --- Understanding the Scaling Process

## Objective

Understand how Azure decides when to add or remove VM instances.

The basic process is:

``` text
Azure Monitor
      ↓
Collect metrics
      ↓
Autoscale rules
      ↓
Evaluate condition
      ↓
Scale Out / Scale In
      ↓
VMSS changes capacity
      ↓
Cooldown / evaluation period
      ↓
Check metrics again
```

------------------------------------------------------------------------

## What is Azure Monitor?

Azure Monitor collects metrics and other monitoring information from
Azure resources.

Examples:

-   Percentage CPU
-   Network activity
-   Requests
-   Application metrics
-   Availability-related information

Autoscaling can use supported metrics to make scaling decisions.

------------------------------------------------------------------------

## Example Scale-Out Rule

Configuration:

``` text
Minimum = 2
Maximum = 6

Rule:
CPU > 70%
Action:
Add 1 instance
```

Current state:

``` text
2 VMs
CPU = 82%
```

Process:

``` text
CPU = 82%
      ↓
CPU > 70%
      ↓
Rule matches
      ↓
Add 1 VM
      ↓
3 VMs
```

After the scaling operation, Azure evaluates the workload again.

------------------------------------------------------------------------

## Example Scale-In Rule

Configuration:

``` text
Minimum = 2
Maximum = 6

Rule:
CPU < 30%
Action:
Remove 1 instance
```

Current:

``` text
5 VMs
CPU = 20%
```

Process:

``` text
CPU = 20%
      ↓
CPU < 30%
      ↓
Rule matches
      ↓
Remove 1 VM
      ↓
4 VMs
```

------------------------------------------------------------------------

## Minimum and Maximum Capacity

Example:

``` text
Minimum = 2
Maximum = 8
```

This means:

``` text
Never go below 2 instances.
Never go above 8 instances.
```

Example:

``` text
Current = 5

High demand:
5 → 6 → 7 → 8

Low demand:
8 → 7 → 6 → 5 → 4 → 3 → 2
```

The autoscale configuration prevents going beyond the limits.

------------------------------------------------------------------------

## Why Cooldown Matters

After scaling, Azure needs time to observe the effect of the change.

Example:

``` text
CPU = 85%
   ↓
Add VM
   ↓
3 VMs
   ↓
Wait / evaluate
   ↓
Check CPU again
```

Without an appropriate waiting/evaluation period, a system could react
too aggressively to temporary changes.

------------------------------------------------------------------------

## Avoiding Scaling Oscillation

Bad configuration:

``` text
Scale out:
CPU > 50%

Scale in:
CPU < 50%
```

Small CPU changes around 50% could repeatedly cause:

``` text
Scale out
   ↓
Scale in
   ↓
Scale out
   ↓
Scale in
```

A wider range is generally better.

Example:

``` text
Scale out:
CPU > 70%

Scale in:
CPU < 30%
```

This provides a larger buffer between the two actions.

------------------------------------------------------------------------

## Important Point

**Load balancing and autoscaling are different.**

### Load Balancer

Distributes traffic:

``` text
Users
  |
  v
Load Balancer
 /    |    \
VM1  VM2  VM3
```

### Autoscaling

Changes the number of instances:

``` text
2 VMs → 4 VMs
```

They can work together:

``` text
                Users
                  |
                  v
            Load Balancer
          /      |      \
        VM1     VM2     VM3
                 ^
                 |
              VMSS
                 ^
                 |
             Autoscale
```

------------------------------------------------------------------------

## Scaling Decision Example

Initial:

``` text
Instances = 2
CPU = 25%
```

No scale-out.

Traffic increases:

``` text
CPU = 78%
```

Scale-out rule:

``` text
CPU > 70%
```

Action:

``` text
2 → 3
```

Later:

``` text
CPU = 55%
```

No additional scale-out.

Traffic decreases:

``` text
CPU = 20%
```

Scale-in rule:

``` text
CPU < 30%
```

Action:

``` text
3 → 2
```

If 2 is the minimum, scaling stops there.

------------------------------------------------------------------------

# 47. Lab --- Azure Virtual Machine Scale Set --- Scale In

## Objective

Learn how to reduce the number of VM instances when workload decreases.

### What is Scale In?

Scale in means removing VM instances.

Example:

``` text
6 VMs → 3 VMs
```

This is horizontal scaling in the opposite direction from scale out.

------------------------------------------------------------------------

## Why Scale In?

Suppose:

``` text
Day:
6 VMs
High traffic

Night:
2 VMs
Low traffic
```

Keeping six VMs running all night may waste resources.

Scale in allows capacity to match demand.

------------------------------------------------------------------------

## Portal Lab

### Step 1 --- Open VMSS

``` text
Azure Portal
    ↓
Virtual Machine Scale Sets
    ↓
Select VMSS
```

### Step 2 --- Open Scaling

``` text
VMSS
  ↓
Scaling
```

### Step 3 --- Reduce Instance Count

Suppose:

``` text
Current = 5
```

Change:

``` text
5 → 3
```

Save/apply the change.

Azure removes instances until the requested capacity is reached.

### Step 4 --- Verify

Go to:

``` text
VMSS
  ↓
Instances
```

Expected:

``` text
Instance 1
Instance 2
Instance 3
```

------------------------------------------------------------------------

## Azure CLI --- Scale In

Set capacity to three:

``` bash
az vmss scale \
  --resource-group rg-vmss-demo \
  --name vmss-web-demo \
  --new-capacity 3
```

Check capacity:

``` bash
az vmss show \
  --resource-group rg-vmss-demo \
  --name vmss-web-demo \
  --query sku.capacity
```

List instances:

``` bash
az vmss list-instances \
  --resource-group rg-vmss-demo \
  --name vmss-web-demo \
  --output table
```

------------------------------------------------------------------------

## Autoscale Scale-In Example

Configuration:

``` text
Minimum = 2
Maximum = 8
```

Rule:

``` text
CPU < 30%
→ Remove 1 instance
```

Current:

``` text
5 VMs
CPU = 20%
```

Possible progression:

``` text
5 → 4 → 3 → 2
```

It should not go below the configured minimum.

------------------------------------------------------------------------

## Production Warning --- Stateful Applications

Be careful when removing VM instances if important data exists only on a
VM's local disk.

Bad design:

``` text
VM1 → Important local data
VM2 → Important local data
VM3 → Important local data
```

If VM3 is removed, data stored only there may become unavailable.

Better design:

``` text
VM1 \
VM2  \
VM3   → External database/storage
```

Examples:

-   Azure SQL
-   Azure Database services
-   Azure Storage
-   Other appropriate external data stores

Stateless application tiers are generally easier to scale.

------------------------------------------------------------------------

## Scale-In Scenario

``` text
Peak traffic
    ↓
8 VMs

Traffic decreases
    ↓
6 VMs

Later
    ↓
4 VMs

Low traffic
    ↓
2 VMs
```

This can reduce resource consumption while maintaining the required
minimum capacity.

------------------------------------------------------------------------

# 48. Lab --- Azure VM Scale Sets --- Flexible Orchestration Mode

## Objective

Understand Flexible orchestration and how it differs from Uniform
orchestration.

------------------------------------------------------------------------

## Uniform Orchestration

Uniform orchestration is designed for a standardized group of VM
instances.

Think:

``` text
VMSS
 |
 +-- VM1
 +-- VM2
 +-- VM3
 +-- VM4
```

The VMs are generally created and managed as a consistent fleet.

### Good Use Case

A web tier where all servers run:

``` text
Ubuntu
Nginx
Same application
Same configuration
```

------------------------------------------------------------------------

## Flexible Orchestration

Flexible orchestration provides more flexibility for VM instance
management.

It is useful when your VM-based workload needs more individual control
rather than treating every VM as an identical instance.

Conceptually:

``` text
VMSS
 |
 +-- VM1 → Web workload
 |
 +-- VM2 → API workload
 |
 +-- VM3 → Worker workload
```

The exact architecture should still follow the application's design and
Azure service limitations.

------------------------------------------------------------------------

## Why Use Flexible Orchestration?

Potential reasons include:

-   More flexible VM management.
-   More control over individual VM instances.
-   Workloads that are not perfectly identical.
-   Scenarios requiring a VMSS management boundary while retaining more
    VM-level flexibility.

------------------------------------------------------------------------

## Uniform vs Flexible

  -----------------------------------------------------------------------
  Feature                 Uniform                 Flexible
  ----------------------- ----------------------- -----------------------
  Main idea               Standardized VM fleet   Flexible VM management

  VM configuration        Generally consistent    More individual
                                                  flexibility

  Best suited for         Homogeneous workloads   More varied VM
                                                  workloads

  Management style        Fleet-oriented          More VM-oriented
                                                  flexibility
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## Important Exam Understanding

Do not memorize:

``` text
Flexible = no scaling
```

That is incorrect.

The important distinction is the **orchestration/management model**, not
simply whether scaling is possible.

------------------------------------------------------------------------

## Practical Example

### Uniform

You operate 20 identical Nginx servers:

``` text
VM1 → Nginx
VM2 → Nginx
VM3 → Nginx
...
VM20 → Nginx
```

Uniform orchestration is a natural fit for this standardized fleet.

### Flexible

You have a VM-based environment where individual instances require more
flexibility in their configuration and management.

Flexible orchestration can be considered.

------------------------------------------------------------------------

## Lab Verification Checklist

After creating or reviewing a Flexible VMSS, verify:

``` text
VMSS
 ↓
Orchestration configuration
 ↓
Instances
 ↓
Networking
 ↓
Scaling configuration
```

Also verify that the selected VM size, region, subscription quota, and
other required resources are available.

------------------------------------------------------------------------

# 49. Introduction to Azure Web Apps

## What is Azure Web Apps?

Azure Web Apps is a capability of **Azure App Service**, a Platform as a
Service (PaaS) offering.

It lets you host web applications and APIs without directly managing the
underlying virtual machines.

------------------------------------------------------------------------

## Traditional VM Hosting

With an Azure VM, you may manage:

``` text
Azure VM
   ↓
Operating System
   ↓
Updates / Patching
   ↓
Web Server
   ↓
Runtime
   ↓
Application
```

For example:

``` text
Azure VM
   ↓
Ubuntu
   ↓
Nginx
   ↓
Node.js
   ↓
Application
```

You are responsible for much more infrastructure management.

------------------------------------------------------------------------

## Azure App Service

With App Service:

``` text
Azure App Service
      ↓
    Web App
      ↓
 Application
```

Azure manages much of the underlying platform/infrastructure.

You can focus more on:

-   Application code
-   Configuration
-   Deployment
-   Scaling
-   Monitoring

------------------------------------------------------------------------

## PaaS

Azure Web Apps is a **PaaS** solution.

PaaS means:

``` text
Platform as a Service
```

The cloud provider manages more of the underlying infrastructure than
with IaaS.

------------------------------------------------------------------------

## VM vs Web App

  Feature                  Azure VM                Azure Web App
  ------------------------ ----------------------- ----------------------------------
  Service model            IaaS                    PaaS
  OS management            Customer                Azure manages platform
  VM access                Yes                     No traditional VM administration
  Infrastructure control   High                    Lower
  Maintenance              More                    Less
  Web hosting              Possible                Primary use
  Scaling                  VM/VMSS                 App Service scaling
  Best for                 Custom infrastructure   Web/API applications

------------------------------------------------------------------------

# App Service Plan

One of the most important concepts is the **App Service Plan**.

A Web App runs in an App Service Plan.

Conceptually:

``` text
App Service Plan
       |
   +---+---+
   |   |   |
 App1 App2 App3
```

The App Service Plan provides the compute environment for the
applications.

------------------------------------------------------------------------

## Can Multiple Apps Share a Plan?

Yes.

Example:

``` text
App Service Plan
       |
       +-- Website
       |
       +-- API
       |
       +-- Admin Portal
```

Sharing can reduce cost, but all apps consume resources from the same
plan capacity.

------------------------------------------------------------------------

# App Service Scaling

## Scale Up

Scale up means moving to a more capable App Service pricing tier/size.

Conceptually:

``` text
Smaller resources
       ↓
Larger resources
```

This is vertical scaling.

------------------------------------------------------------------------

## Scale Out

Scale out means increasing the number of application instances.

Example:

``` text
1 instance
     ↓
3 instances
```

This is horizontal scaling.

------------------------------------------------------------------------

## Scale Up vs Scale Out

  Type        Meaning                       Example
  ----------- ----------------------------- -----------------------
  Scale Up    More resources per instance   Smaller → Larger tier
  Scale Out   More instances                1 → 4 instances

------------------------------------------------------------------------

# Supported Application Stacks

Azure App Service supports several application stacks, depending on the
platform and currently available runtime versions.

Common examples include:

-   .NET
-   Java
-   Node.js
-   Python
-   PHP

Always check the currently supported runtime versions when creating a
new app.

------------------------------------------------------------------------

# Application Settings

App Service provides application settings that can be exposed to the
application as environment variables.

Example:

``` text
DATABASE_URL
API_URL
NODE_ENV
```

Node.js example:

``` javascript
const databaseUrl = process.env.DATABASE_URL;
const environment = process.env.NODE_ENV;
```

This is preferable to hardcoding environment-specific configuration in
source code.

------------------------------------------------------------------------

# Deployment

Applications can be deployed using supported mechanisms such as:

-   GitHub
-   Azure DevOps
-   ZIP deployment
-   Local Git where supported
-   Containers
-   Other supported deployment methods

A common workflow is:

``` text
Developer
    ↓
GitHub
    ↓
Deployment
    ↓
Azure App Service
    ↓
Web Application
```

------------------------------------------------------------------------

# Deployment Slots

App Service can provide deployment slots on supported pricing
tiers/configurations.

Example:

``` text
Production
    |
    +-- Staging
```

You can deploy a new version to staging:

``` text
Staging
   ↓
Version 2
```

Test it, then swap it into production.

Conceptually:

``` text
Before:
Production → Version 1
Staging    → Version 2

After swap:
Production → Version 2
Staging    → Version 1
```

This can reduce deployment risk.

------------------------------------------------------------------------

# HTTPS

Production applications should use HTTPS.

Example:

``` text
https://example.com
```

HTTPS encrypts traffic between the client and the application endpoint.

------------------------------------------------------------------------

# Monitoring and Logs

Application monitoring and logs help troubleshoot:

-   Application startup errors
-   HTTP errors
-   Deployment problems
-   Runtime issues
-   Performance problems

Use Azure monitoring/logging capabilities appropriate to the
application.

------------------------------------------------------------------------

# When to Use Azure Web Apps

Use Azure Web Apps when:

-   You want to host a web application.
-   You want PaaS.
-   You don't want to manage the underlying VM.
-   You need supported application runtimes.
-   You want easy deployment.
-   You need built-in scaling capabilities.
-   You want easier application operations.

------------------------------------------------------------------------

# When to Use Azure VM Instead

Use a VM when:

-   You need OS-level access.
-   You need custom system software.
-   You need a specific OS configuration.
-   You need custom networking/software configuration.
-   The workload is not well suited to App Service.

------------------------------------------------------------------------

# 50. Lab --- Azure Web Apps

## Objective

Create an Azure Web App using Azure App Service.

Architecture:

``` text
Internet
   |
   v
Azure App Service
   |
   v
Web App
   |
   v
Application
```

------------------------------------------------------------------------

# Prerequisites

You need:

-   Azure subscription
-   Resource Group
-   Permission to create App Service resources
-   An available region
-   A supported App Service Plan/SKU

Be aware that available SKUs and quotas can vary by subscription and
region.

------------------------------------------------------------------------

# Method 1 --- Azure Portal

## Step 1 --- Open Azure Portal

Go to:

``` text
Azure Portal
    ↓
Create a resource
    ↓
Web App
```

------------------------------------------------------------------------

## Step 2 --- Basics

Select:

``` text
Subscription
Resource Group
```

Example:

``` text
Resource Group:
rg-webapp-demo
```

------------------------------------------------------------------------

## Step 3 --- Web App Name

Choose a globally unique app name.

Example:

``` text
prins-webapp-demo-2026
```

The name affects the default hostname.

------------------------------------------------------------------------

## Step 4 --- Publish

Select:

``` text
Code
```

when deploying normal application source code.

For container-based applications, select the appropriate container
option.

------------------------------------------------------------------------

## Step 5 --- Runtime Stack

Choose the runtime required by your application.

Example:

``` text
Node.js
```

Select an available supported version.

------------------------------------------------------------------------

## Step 6 --- Operating System

Choose:

``` text
Linux
```

or:

``` text
Windows
```

depending on the application requirements.

------------------------------------------------------------------------

## Step 7 --- Region

Select an available region.

Example:

``` text
Central India
```

or another suitable region.

------------------------------------------------------------------------

## Step 8 --- App Service Plan

Create or select an App Service Plan.

Example:

``` text
ASP-webapp-demo
```

Choose a suitable pricing tier for the lab.

------------------------------------------------------------------------

## Step 9 --- Review + Create

Select:

``` text
Review + create
```

Then:

``` text
Create
```

------------------------------------------------------------------------

# Method 2 --- Azure CLI

## Step 1 --- Create Resource Group

``` bash
az group create \
  --name rg-webapp-demo \
  --location eastus
```

------------------------------------------------------------------------

## Step 2 --- Create App Service Plan

Example Linux plan:

``` bash
az appservice plan create \
  --name asp-webapp-demo \
  --resource-group rg-webapp-demo \
  --sku B1 \
  --is-linux
```

The exact SKU may not be available for every subscription/region.

------------------------------------------------------------------------

## Step 3 --- Create Web App

Example:

``` bash
az webapp create \
  --resource-group rg-webapp-demo \
  --plan asp-webapp-demo \
  --name unique-webapp-name \
  --runtime "NODE:22-lts"
```

If the runtime string is rejected, list currently supported runtimes:

``` bash
az webapp list-runtimes \
  --os-type linux
```

Then select an available Node.js runtime.

------------------------------------------------------------------------

# Verify the Web App

List Web Apps:

``` bash
az webapp list \
  --resource-group rg-webapp-demo \
  --output table
```

------------------------------------------------------------------------

# Get Default Hostname

``` bash
az webapp show \
  --resource-group rg-webapp-demo \
  --name unique-webapp-name \
  --query defaultHostName \
  --output tsv
```

The command returns the default hostname.

Open it in your browser.

------------------------------------------------------------------------

# Configure Application Settings

Example:

``` bash
az webapp config appsettings set \
  --resource-group rg-webapp-demo \
  --name unique-webapp-name \
  --settings \
  NODE_ENV=production
```

Add another setting:

``` bash
az webapp config appsettings set \
  --resource-group rg-webapp-demo \
  --name unique-webapp-name \
  --settings \
  API_URL=https://api.example.com
```

------------------------------------------------------------------------

# View Application Settings

``` bash
az webapp config appsettings list \
  --resource-group rg-webapp-demo \
  --name unique-webapp-name \
  --output table
```

Do not expose passwords or secret values unnecessarily.

------------------------------------------------------------------------

# Node.js Example Application

Create:

``` text
app.js
```

Example:

``` javascript
const http = require("http");

const port = process.env.PORT || 3000;

const server = http.createServer((req, res) => {
    res.writeHead(200, {
        "Content-Type": "text/plain"
    });

    res.end("Hello from Azure Web App!");
});

server.listen(port, () => {
    console.log(`Server running on port ${port}`);
});
```

------------------------------------------------------------------------

# package.json

Example:

``` json
{
  "name": "azure-webapp-demo",
  "version": "1.0.0",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  }
}
```

------------------------------------------------------------------------

# Why Use process.env.PORT?

Cloud platforms can provide the port through an environment variable.

Therefore:

``` javascript
const port = process.env.PORT || 3000;
```

means:

``` text
Use Azure-provided PORT
       ↓
If unavailable locally
       ↓
Use 3000
```

This makes the application easier to run both locally and in the cloud.

------------------------------------------------------------------------

# Deployment Concept

A typical deployment can look like:

``` text
Local Project
     |
     v
Git Repository
     |
     v
Deployment
     |
     v
Azure App Service
     |
     v
Web App
```

------------------------------------------------------------------------

# Logs

For troubleshooting, enable/view App Service logs as appropriate and
use:

``` bash
az webapp log tail \
  --resource-group rg-webapp-demo \
  --name unique-webapp-name
```

Use logs to investigate:

-   Application startup failures
-   Runtime errors
-   Deployment problems
-   Configuration issues

------------------------------------------------------------------------

# Common Lab Troubleshooting

## Problem 1 --- App name already exists

Error:

``` text
Name is already taken
```

Solution:

Use a globally unique name.

Example:

``` text
prins-webapp-20261007
```

------------------------------------------------------------------------

## Problem 2 --- Runtime not supported

If:

``` bash
--runtime "NODE:22-lts"
```

does not work, check:

``` bash
az webapp list-runtimes \
  --os-type linux
```

Select a currently supported runtime.

------------------------------------------------------------------------

## Problem 3 --- SKU unavailable

If the selected plan SKU is unavailable:

-   Try another supported SKU.
-   Try another region.
-   Check subscription quota/limits.
-   Check current App Service availability.

------------------------------------------------------------------------

## Problem 4 --- Application does not start

Check:

``` text
Application logs
Startup command
Runtime version
package.json
Application port
Environment variables
```

For Node.js, verify:

``` json
"scripts": {
  "start": "node app.js"
}
```

------------------------------------------------------------------------

# Lab Verification Checklist

After completing the lab, verify:

``` text
[ ] Resource Group created
[ ] App Service Plan created
[ ] Web App created
[ ] Runtime selected
[ ] Default hostname works
[ ] Application deployed
[ ] Application settings configured
[ ] Logs checked
[ ] Scaling options reviewed
```

------------------------------------------------------------------------

# Lab 45--50 Quick Revision

## Lab 45 --- Scale Out

``` text
More demand
    ↓
More VM instances
```

Example:

``` text
2 → 4 VMs
```

------------------------------------------------------------------------

## Topic 46 --- Scaling Process

``` text
Monitor metrics
    ↓
Evaluate rule
    ↓
Scale
    ↓
Wait/evaluate
    ↓
Repeat if required
```

------------------------------------------------------------------------

## Lab 47 --- Scale In

``` text
Less demand
    ↓
Fewer VM instances
```

Example:

``` text
6 → 2 VMs
```

------------------------------------------------------------------------

## Lab 48 --- Flexible Orchestration

``` text
More flexible VM instance management
```

Compared with Uniform:

``` text
Uniform → standardized fleet
Flexible → more individual VM flexibility
```

------------------------------------------------------------------------

## Topic 49 --- Azure Web Apps

``` text
Azure App Service
       ↓
     Web App
       ↓
Application
```

PaaS service.

------------------------------------------------------------------------

## Lab 50 --- Azure Web Apps

Main flow:

``` text
Resource Group
      ↓
App Service Plan
      ↓
Web App
      ↓
Deploy Application
      ↓
Configure Settings
      ↓
Test
      ↓
Monitor
```

------------------------------------------------------------------------

# Interview Questions --- Labs 45--50

## Q1. What is scale out?

Adding more instances to handle increased workload.

------------------------------------------------------------------------

## Q2. What is scale in?

Removing instances when workload decreases.

------------------------------------------------------------------------

## Q3. What is the difference between scale out and scale up?

Scale out adds instances.

Scale up increases resources of an instance.

------------------------------------------------------------------------

## Q4. What controls VMSS autoscaling?

Autoscale rules use supported Azure Monitor metrics and configured
capacity limits.

------------------------------------------------------------------------

## Q5. Why are minimum and maximum instance counts important?

They prevent autoscaling from going below or above the configured
capacity boundaries.

------------------------------------------------------------------------

## Q6. What is Azure App Service?

A PaaS platform for hosting web applications, APIs, and supported
application runtimes.

------------------------------------------------------------------------

## Q7. What is an App Service Plan?

It provides the compute environment/resources for App Service
applications.

------------------------------------------------------------------------

## Q8. Can multiple Web Apps share an App Service Plan?

Yes.

------------------------------------------------------------------------

## Q9. What is a deployment slot?

A separate App Service deployment environment used for scenarios such as
staging and testing before production.

------------------------------------------------------------------------

## Q10. Why should application secrets not be hardcoded?

Hardcoded secrets can be exposed through source code, repositories,
logs, or builds. Use secure configuration/secret-management approaches
instead.

------------------------------------------------------------------------

# AZ-104 Scenario Questions

### Scenario 1

A website receives very high traffic during the evening and low traffic
at night.

**Question:** What should you configure?

**Answer:** Autoscaling with appropriate scale-out and scale-in rules.

------------------------------------------------------------------------

### Scenario 2

You have 2 VM instances and need 5 during peak traffic.

**Question:** What operation is this?

**Answer:** Scale out.

------------------------------------------------------------------------

### Scenario 3

You have 8 VM instances at peak time and only need 2 at night.

**Question:** What operation is this?

**Answer:** Scale in.

------------------------------------------------------------------------

### Scenario 4

You need to host a Node.js application but do not want to manage the
Linux operating system.

**Answer:** Azure App Service / Web App.

------------------------------------------------------------------------

### Scenario 5

You need SSH access and complete control over the operating system.

**Answer:** Azure VM or an appropriate VM-based architecture such as
VMSS.

------------------------------------------------------------------------

### Scenario 6

You have multiple web applications and want them to use the same App
Service compute environment.

**Answer:** Multiple Web Apps can share an App Service Plan.

------------------------------------------------------------------------

### Scenario 7

You want to test Version 2 before moving it into production.

**Answer:** Use a deployment slot when supported by the App Service
plan/configuration.

------------------------------------------------------------------------

# Final Architecture Comparison

## VMSS

``` text
                     Internet
                        |
                        v
                 Load Balancer
                        |
          +-------------+-------------+
          |             |             |
         VM1           VM2           VM3
          \             |             /
           \            |            /
                VM Scale Set
                     |
                  Autoscale
```

Best when you need VM-level control.

------------------------------------------------------------------------

## Azure Web App

``` text
                     Internet
                        |
                        v
                 Azure App Service
                        |
                     Web App
                        |
                  Application
```

Best when you want managed PaaS web hosting.

------------------------------------------------------------------------

# Final Memory Notes

``` text
Scale Out  = MORE INSTANCES
Scale In   = FEWER INSTANCES

Scale Up   = BIGGER INSTANCE
Scale Down = SMALLER INSTANCE

VMSS       = GROUP OF VMs

Uniform    = STANDARDIZED VM FLEET
Flexible   = MORE FLEXIBLE VM MANAGEMENT

Web App    = PAAS WEB HOSTING

App Service Plan = COMPUTE ENVIRONMENT FOR WEB APPS

Autoscale:
HIGH DEMAND → SCALE OUT
LOW DEMAND  → SCALE IN
```
