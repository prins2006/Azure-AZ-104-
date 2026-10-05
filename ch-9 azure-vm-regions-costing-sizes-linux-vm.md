# Azure Virtual Machines --- AZ-104 Notes

## 15. Lab -- Setting up a Web Server on the Virtual Machine

### What is a Web Server?

A web server receives HTTP/HTTPS requests from clients and returns web
content.

Common web servers: - Nginx - Apache - IIS

### Basic Architecture

``` text
User Browser
     |
     | HTTP Request
     v
Azure Public IP
     |
     v
Linux VM
     |
     v
Nginx Web Server
     |
     v
Web Page
```

### Install Nginx on a Linux VM

Connect to the VM:

``` bash
ssh azureuser@<PUBLIC-IP>
```

Update packages:

``` bash
sudo apt update
```

Install Nginx:

``` bash
sudo apt install nginx -y
```

Check Nginx:

``` bash
sudo systemctl status nginx
```

Start Nginx:

``` bash
sudo systemctl start nginx
```

Enable Nginx at boot:

``` bash
sudo systemctl enable nginx
```

Check listening ports:

``` bash
sudo ss -tulpn | grep nginx
```

Default ports:

``` text
HTTP  -> 80
HTTPS -> 443
```

### Azure NSG Requirement

Installing Nginx is not enough. Azure must allow inbound HTTP traffic
through the Network Security Group (NSG).

Example rule:

``` text
Protocol: TCP
Port: 80
Source: Any
Action: Allow
```

Then open:

``` text
http://<PUBLIC-IP>
```

------------------------------------------------------------------------

## 16. Understanding Azure Regions

### What is an Azure Region?

An Azure region is a geographical area containing one or more Azure
datacenters.

Examples:

``` text
East US
Central India
West Europe
Southeast Asia
UK South
```

### Why Region Selection Matters

Region selection affects:

-   Latency
-   Availability
-   Pricing
-   Service availability
-   Data residency
-   Disaster recovery
-   Compliance

For users mainly located in India, a region such as Central India may
generally provide lower latency than a distant region.

### Azure Region vs Availability Zone

A **Region** is a geographical Azure location.

An **Availability Zone** is a physically separate datacenter location
within an Azure region.

Example:

``` text
Central India
 ├── Zone 1
 ├── Zone 2
 └── Zone 3
```

Availability Zones help protect workloads from individual datacenter
failures.

### Region Pair

Azure regions may have a paired region for disaster recovery and
platform recovery purposes.

``` text
Primary Region
      |
      | Recovery / Replication
      v
Paired Region
```

------------------------------------------------------------------------

## 17. Costing for Azure Resources

### Why Azure Cost Matters

Azure resources generally use usage-based pricing.

Examples of resources that can contribute to cost:

``` text
Virtual Machine
Storage
Public IP
Database
Network traffic
Backup
```

### Main Factors Affecting VM Cost

#### 1. VM Size

A larger VM generally costs more.

``` text
B1s
  ↓
B2s
  ↓
D2s_v5
  ↓
D4s_v5
```

#### 2. Running Time

A VM running continuously generally costs more than one that is used
only during working hours.

#### 3. Disks

Common disk types include:

``` text
Standard HDD
Standard SSD
Premium SSD
Ultra Disk
```

#### 4. Networking

Network usage and data transfer can contribute to costs.

### Azure Pricing Calculator

The Azure Pricing Calculator can be used to estimate costs before
deployment.

``` text
Select Service
      ↓
Choose Region
      ↓
Choose Configuration
      ↓
Enter Expected Usage
      ↓
Estimate Cost
```

### Cost Management

Azure Cost Management can be used to:

-   View costs
-   Analyze spending
-   Create budgets
-   Monitor usage
-   Identify expensive resources

Example:

``` text
Monthly Budget = $50
Current Cost   = $35
Remaining      = $15
```

------------------------------------------------------------------------

## 18. Virtual Machine Sizes

### What is a VM Size?

A VM size determines the resources and capabilities available to a
virtual machine.

It can affect:

-   vCPUs
-   RAM
-   Temporary storage
-   Maximum data disks
-   Network performance

Example:

``` text
Standard_B1s
Standard_D2s_v5
```

### Common VM Families

  Family     Typical Purpose
  ---------- -----------------------------
  B-series   Burstable workloads
  D-series   General-purpose workloads
  E-series   Memory-intensive workloads
  F-series   Compute-intensive workloads
  L-series   Storage-intensive workloads
  N-series   GPU workloads

### B-Series

B-series VMs are useful for workloads with variable CPU requirements.

Examples:

``` text
Development Server
Testing Environment
Small Web Server
```

### D-Series

D-series VMs are commonly used for general-purpose workloads.

Examples:

``` text
Web Application
Application Server
Development Environment
```

### E-Series

E-series VMs are designed for workloads requiring more memory.

Examples:

``` text
Database
Large Application
Memory-intensive Workload
```

### Choosing a VM Size

Consider:

``` text
CPU Requirement
      +
RAM Requirement
      +
Disk Requirement
      +
Network Requirement
      +
Workload
      +
Cost
```

------------------------------------------------------------------------

## 19. Lab -- Building a Linux Virtual Machine

### Azure Linux VM Architecture

``` text
Internet
    |
    v
Public IP
    |
    v
NIC
    |
    v
Network Security Group
    |
    v
Linux VM
    |
    +---- OS Disk
    |
    +---- Data Disk
```

### Creating a Linux VM

In Azure Portal:

``` text
Azure Portal
    ↓
Virtual Machines
    ↓
Create
    ↓
Azure Virtual Machine
```

Configure:

``` text
Subscription
Resource Group
VM Name
Region
Availability Options
Image
VM Size
```

Example:

``` text
VM Name: linux-vm
Region: Central India
Image: Ubuntu
```

### Authentication

For Linux VMs, SSH key authentication is recommended.

``` text
SSH Public Key
SSH Private Key
```

The public key is placed on the VM.

The private key should remain securely with you.

``` text
Your Computer
    |
    | Private Key
    v
SSH Authentication
    |
    v
Azure Linux VM
```

### Networking

A Linux VM commonly uses:

``` text
Virtual Network
Subnet
Network Interface
Public IP
Network Security Group
```

### Important NSG Ports

  Service   Protocol     Port
  --------- ---------- ------
  SSH       TCP            22
  HTTP      TCP            80
  HTTPS     TCP           443

### Check VM from Azure CLI

``` bash
az vm list -o table
```

Check VM instance status:

``` bash
az vm get-instance-view \
  --resource-group <RESOURCE-GROUP> \
  --name <VM-NAME> \
  --query instanceView.statuses
```

------------------------------------------------------------------------

## 20. Lab -- Connecting to the Linux Machine

### What is SSH?

SSH stands for **Secure Shell**.

It is used to securely connect to remote Linux machines.

Default SSH port:

``` text
22
```

### SSH Connection

Basic command:

``` bash
ssh azureuser@<PUBLIC-IP>
```

Using a private key:

``` bash
ssh -i ~/.ssh/my-key.pem azureuser@<PUBLIC-IP>
```

### SSH Authentication Flow

``` text
Your Computer
      |
      | SSH Request
      | Port 22
      v
Azure NSG
      |
      | Allow TCP 22
      v
Linux VM
      |
      | Verify SSH Key
      v
Login Successful
```

### Private Key Permissions

Private keys should not be accessible by other users.

``` bash
chmod 600 ~/.ssh/my-key.pem
```

Check:

``` bash
ls -l ~/.ssh/my-key.pem
```

Expected permissions are similar to:

``` text
-rw-------
```

### Troubleshooting SSH

#### 1. Check Public IP

``` bash
az vm show \
  --resource-group <RESOURCE-GROUP> \
  --name <VM-NAME> \
  --show-details \
  --query publicIps \
  -o tsv
```

#### 2. Check NSG

Make sure TCP port 22 is allowed.

#### 3. Check VM State

``` bash
az vm get-instance-view \
  --resource-group <RESOURCE-GROUP> \
  --name <VM-NAME> \
  --query instanceView.statuses
```

#### 4. Check SSH Key

``` bash
ssh -i ~/.ssh/my-key.pem azureuser@<PUBLIC-IP>
```

#### 5. Test Port 22

``` bash
nc -zv <PUBLIC-IP> 22
```

Note: `ping` may fail even when SSH works because ICMP may not be
allowed.

------------------------------------------------------------------------

# AZ-104 Revision

## Azure VM Components

``` text
Resource Group
      |
      +---- Virtual Machine
      |
      +---- NIC
      |
      +---- Public IP
      |
      +---- NSG
      |
      +---- OS Disk
      |
      +---- Data Disk
      |
      +---- Virtual Network
              |
              +---- Subnet
```

## Quick Comparison

  Topic               Meaning
  ------------------- ----------------------------------------------
  Region              Geographical Azure location
  Availability Zone   Separate datacenter within a region
  VM Size             Determines CPU/RAM and other VM capabilities
  NSG                 Controls network traffic
  Public IP           Allows internet-facing connectivity
  NIC                 Connects VM to a network
  OS Disk             Contains operating system
  Data Disk           Stores application/data files
  SSH                 Secure remote Linux connection
  Port 22             Default SSH port
  Port 80             Default HTTP port
  Port 443            Default HTTPS port

# Important Interview Questions

## 1. What is an Azure region?

An Azure region is a geographical location containing one or more Azure
datacenters where Azure resources can be deployed.

## 2. What is an Availability Zone?

An Availability Zone is a physically separate location within an Azure
region that provides protection against datacenter-level failures.

## 3. What determines the cost of an Azure VM?

Main factors include VM size, running time, disk configuration,
networking, and other attached Azure resources.

## 4. What is a VM size?

A VM size defines the compute resources and capabilities available to a
virtual machine, such as vCPU, memory, disk, and networking limits.

## 5. What is an NSG?

A Network Security Group controls inbound and outbound network traffic
using security rules.

## 6. What port is used by SSH?

``` text
TCP 22
```

## 7. How do you connect to a Linux Azure VM?

``` bash
ssh azureuser@<PUBLIC-IP>
```

or:

``` bash
ssh -i <PRIVATE-KEY> azureuser@<PUBLIC-IP>
```

## 8. Why might SSH fail even when the VM is running?

Possible reasons:

``` text
NSG does not allow TCP 22
Wrong public IP
Wrong username
Wrong private key
SSH service is not running
Network connectivity problem
VM is not actually running
```

## 9. Why is region selection important?

Because it affects:

``` text
Latency
Pricing
Service availability
Data residency
Compliance
Availability
```

## 10. Why would you choose B-series instead of a larger VM?

For workloads with low average CPU usage but occasional CPU bursts,
B-series can be a cost-effective option.

# Practical Flow to Remember

``` text
1. Select Azure Region
        ↓
2. Select VM Size
        ↓
3. Select Linux Image
        ↓
4. Configure Authentication
        ↓
5. Create VNet/Subnet
        ↓
6. Configure NSG
        ↓
7. Create NIC
        ↓
8. Attach OS Disk
        ↓
9. Deploy VM
        ↓
10. Get Public IP
        ↓
11. Allow SSH (22)
        ↓
12. Connect using SSH
        ↓
13. Install Nginx
        ↓
14. Allow HTTP (80)
        ↓
15. Access Web Server
```
