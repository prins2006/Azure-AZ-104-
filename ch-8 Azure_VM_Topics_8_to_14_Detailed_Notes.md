# Azure Virtual Machines --- Detailed Notes

## Topics 8--14: VM Service, Architecture, Windows VM, Connection, Resizing, macOS Connection, and Connectivity Troubleshooting

> These notes are written for **AZ-104 / beginner-to-production DevOps
> understanding**.\
> Each topic includes: **meaning, architecture, important settings,
> practical examples, commands, troubleshooting, and interview/exam
> points**.

------------------------------------------------------------------------

# 8. We Are Going to Start with the Azure Virtual Machine Service

## 8.1 What is an Azure Virtual Machine?

An **Azure Virtual Machine (VM)** is a virtual computer running inside
Microsoft Azure.

It provides:

-   CPU
-   RAM
-   Operating system
-   Disk storage
-   Network interface
-   Private IP address
-   Optional public IP address
-   Security controls
-   Monitoring and management

You can use an Azure VM like a physical or on-premises server, but Azure
manages the underlying physical infrastructure.

### Simple example

Suppose a company needs a Windows Server to host an application.

Instead of purchasing:

``` text
Physical Server
     |
     +-- CPU
     +-- RAM
     +-- Disk
     +-- Network
```

the company can create:

``` text
Azure
 |
 +-- Resource Group
      |
      +-- Windows VM
           |
           +-- 2 vCPU
           +-- 8 GB RAM
           +-- OS Disk
           +-- Data Disk
           +-- NIC
           +-- Private IP
           +-- Public IP (optional)
```

------------------------------------------------------------------------

# 8.2 When Do We Use Azure VMs?

VMs are useful when you need:

1.  Full operating-system control
2.  Custom software installation
3.  Legacy applications
4.  Windows Server workloads
5.  Linux server workloads
6.  Custom networking
7.  Applications that are not suitable for PaaS
8.  Development/test environments

### Example

A company has an old application that requires:

``` text
Windows Server
.NET Framework
IIS
Custom Windows service
Specific registry configuration
```

A VM can provide the required OS-level control.

------------------------------------------------------------------------

# 8.3 Azure VM vs Physical Server

  -----------------------------------------------------------------------
  Feature                 Physical Server         Azure VM
  ----------------------- ----------------------- -----------------------
  Hardware                You manage it           Azure manages it

  OS                      You install/manage      You manage

  CPU/RAM                 Physical hardware       Selected VM size

  Scaling                 Usually slow            Can resize/change
                                                  architecture

  Networking              Physical network        Azure virtual network

  Storage                 Physical disks          Managed disks

  Hardware failure        Your responsibility     Azure infrastructure
                                                  handles underlying
                                                  hardware

  Provisioning            Hours/days              Usually minutes

  Cost                    Hardware + maintenance  Pay for Azure
                                                  resources/usage
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 8.4 Important Azure VM Components

A VM normally works with several Azure resources.

``` text
                Azure VM
                   |
        +----------+----------+
        |          |          |
       NIC       OS Disk    Data Disk
        |
   Private IP
        |
   Public IP (optional)
        |
   Network Security Group
        |
   Virtual Network
        |
      Subnet
```

### Main components

#### 1. Virtual Machine

The compute resource.

It defines:

-   OS
-   VM size
-   CPU
-   memory
-   configuration

#### 2. VM Size

Defines the compute capacity.

Example:

``` text
B-series
D-series
E-series
```

A VM size determines resources such as:

``` text
vCPU
RAM
Temporary storage
Network performance
Disk throughput
```

#### 3. OS Disk

Contains the operating system.

Example:

``` text
Windows Server
Ubuntu Linux
Red Hat Enterprise Linux
```

#### 4. Data Disk

Used for application or user data.

Example:

``` text
OS Disk:
C:

Data Disk:
D:
```

For Linux:

``` text
OS Disk:
/

Data Disk:
/data
```

#### 5. Network Interface Card (NIC)

Connects the VM to an Azure Virtual Network.

#### 6. Private IP

Used for communication inside the VNet.

Example:

``` text
VM1: 10.0.1.4
VM2: 10.0.1.5
```

#### 7. Public IP

Optional.

Used when the VM needs internet-facing connectivity.

Example:

``` text
Public IP
    |
Internet
    |
Azure VM
```

#### 8. Network Security Group (NSG)

Controls inbound and outbound network traffic.

Example:

``` text
Allow TCP 3389 -> Windows RDP
Allow TCP 22   -> Linux SSH
Deny everything else
```

------------------------------------------------------------------------

# 8.5 Azure VM Lifecycle

A VM can have different power states.

Common states include:

``` text
Running
Stopped
Stopped (deallocated)
Starting
Stopping
Restarting
```

### Important difference: Stop vs Deallocate

When a VM is **stopped from inside the operating system**, the VM may
remain allocated on Azure infrastructure.

When a VM is **stopped and deallocated**, Azure releases the compute
allocation.

For cost optimization, deallocation is important because compute billing
can stop for supported VM scenarios.

> Storage resources such as managed disks can continue to incur charges
> even when the VM is deallocated.

------------------------------------------------------------------------

# 8.6 Azure VM Creation Flow

Typical VM creation:

``` text
1. Choose Subscription
        |
2. Choose/Create Resource Group
        |
3. Select VM name
        |
4. Select Region
        |
5. Select Availability options
        |
6. Select Image
        |
7. Select VM Size
        |
8. Configure Administrator
        |
9. Configure Disks
        |
10. Configure Networking
        |
11. Review + Create
```

------------------------------------------------------------------------

# 8.7 Example: Create a Windows VM

Example configuration:

``` text
Resource Group:
rg-vm-lab

VM Name:
win-vm-01

Region:
Central India

Image:
Windows Server

Size:
Small development/test size

Authentication:
Username + Password

Inbound:
RDP 3389
```

After deployment:

``` text
Internet
   |
Public IP
   |
NSG
   |
NIC
   |
Windows VM
```

------------------------------------------------------------------------

# 8.8 Azure CLI Example

Example command:

``` bash
az vm create \
  --resource-group rg-vm-lab \
  --name win-vm-01 \
  --image Win2022AzureEdition \
  --admin-username azureuser \
  --generate-ssh-keys
```

For Windows authentication, you would normally configure the appropriate
Windows administrator credentials/options.

Always check the currently available image SKU in your selected
region/subscription before using a command.

------------------------------------------------------------------------

# 8.9 Important Exam Points

### Question: What resource provides network connectivity to an Azure VM?

**Answer:** Network Interface Card (NIC).

### Question: Where is the operating system stored?

**Answer:** OS managed disk.

### Question: Is a public IP mandatory?

**Answer:** No.

A VM can communicate privately inside a VNet without a public IP.

### Question: What controls network traffic?

**Answer:** Network Security Group (NSG), along with other networking
controls.

------------------------------------------------------------------------

# 9. The Anatomy Behind Building an Azure Virtual Machine

This topic is about understanding **what Azure creates behind the VM
wizard**.

------------------------------------------------------------------------

# 9.1 High-Level Architecture

When you create a VM, you are not creating only one object.

A simplified architecture is:

``` text
                    Azure VM
                       |
        +--------------+--------------+
        |              |              |
      Compute         Storage       Network
        |              |              |
      VM Size       OS Disk       NIC
                     Data Disk       |
                                    |
                               Private IP
                                    |
                              Public IP
                                    |
                                  NSG
                                    |
                                  Subnet
                                    |
                                  VNet
```

------------------------------------------------------------------------

# 9.2 Resource Group

A **Resource Group (RG)** is a logical container for Azure resources.

Example:

``` text
rg-production-app
 |
 +-- VM
 +-- NIC
 +-- Public IP
 +-- OS Disk
 +-- Data Disk
 +-- NSG
```

A resource group does not mean all resources must be physically located
together.

It is primarily a management boundary.

------------------------------------------------------------------------

# 9.3 Region

A region is an Azure geographic location.

Examples:

``` text
Central India
East US
West Europe
Southeast Asia
```

Choosing a region affects:

-   Latency
-   Availability
-   Pricing
-   Service/VM SKU availability
-   Data residency requirements

### Example

If users are primarily in India:

``` text
Users
  |
India
  |
Central India Azure Region
  |
Application VM
```

This can reduce network latency compared with placing the application
far away.

------------------------------------------------------------------------

# 9.4 Availability Zones

Availability Zones are physically separate locations within supported
Azure regions.

Conceptually:

``` text
Region
 |
 +-- Zone 1
 |
 +-- Zone 2
 |
 +-- Zone 3
```

A highly available application can distribute VMs across zones:

``` text
              Load Balancer
                   |
          +--------+--------+
          |                 |
       VM Zone 1          VM Zone 2
```

If one zone experiences a failure, the other zone can continue serving
traffic.

Availability Zone support depends on the region and service/SKU.

------------------------------------------------------------------------

# 9.5 VM Size

VM size determines the available compute resources.

Example:

``` text
VM Size
 |
 +-- vCPU
 +-- RAM
 +-- Network bandwidth
 +-- Temporary storage
 +-- Disk performance limits
```

### Choosing a size

Development:

``` text
Small VM
```

Production application:

``` text
More CPU
More RAM
Higher network/disk performance
```

Database workload:

``` text
Memory + disk performance become important
```

------------------------------------------------------------------------

# 9.6 OS Image

An image provides the operating system and initial software
configuration.

Examples:

``` text
Windows Server
Ubuntu
Red Hat Enterprise Linux
SUSE
```

Conceptually:

``` text
Image
  |
  +-- Operating System
  +-- Base configuration
  +-- Initial software
```

------------------------------------------------------------------------

# 9.7 Authentication

For Windows VMs:

``` text
RDP
 |
Username
Password
```

For Linux VMs:

``` text
SSH
 |
Username
SSH Private Key
```

SSH keys are generally preferred over passwords for Linux
administration.

------------------------------------------------------------------------

# 9.8 Managed Disks

Azure Managed Disks simplify storage management.

Common disk concepts:

``` text
OS Disk
Data Disk
Temporary Disk
```

### OS Disk

Contains the operating system.

### Data Disk

Contains application/user data.

### Temporary Disk

Provides temporary local storage for supported VM sizes.

Do **not** treat temporary disk as durable storage.

------------------------------------------------------------------------

# 9.9 NIC

The NIC connects the VM to a subnet.

``` text
VM
 |
NIC
 |
Subnet
 |
VNet
```

The NIC can have:

-   Private IP configuration
-   Public IP association
-   NSG association

------------------------------------------------------------------------

# 9.10 Public IP

A public IP allows external connectivity when configured correctly.

Example:

``` text
Internet
   |
Public IP
   |
NIC
   |
VM
```

But having a public IP does **not automatically mean every port is
open**.

NSG rules and the application/service must also allow the traffic.

------------------------------------------------------------------------

# 9.11 NSG

An NSG contains rules such as:

``` text
Priority 100
Allow TCP 3389 from trusted source

Priority 200
Allow TCP 443

Priority 4096
Deny traffic
```

For Windows:

``` text
RDP = TCP 3389
```

For Linux:

``` text
SSH = TCP 22
```

For web:

``` text
HTTP  = TCP 80
HTTPS = TCP 443
```

------------------------------------------------------------------------

# 9.12 Complete Example

Suppose we deploy:

``` text
Windows Web Server
```

Architecture:

``` text
Internet
   |
Public IP
   |
NSG
   |
NIC
   |
Subnet: 10.0.1.0/24
   |
VNet: 10.0.0.0/16
   |
Windows VM
   |
IIS
   |
Website
```

RDP administration:

``` text
Administrator PC
       |
     TCP 3389
       |
    Public IP
       |
      NSG
       |
      NIC
       |
 Windows VM
```

Website:

``` text
Internet
   |
 TCP 80/443
   |
Public IP
   |
 NSG
   |
 IIS
```

------------------------------------------------------------------------

# 9.13 Production Design Recommendation

Avoid exposing management ports to the entire internet.

Instead of:

``` text
Source: Any
Destination port: 3389
```

prefer:

``` text
Source: Trusted IP / VPN / controlled management network
Destination port: 3389
```

For larger environments, consider Azure-native secure management
approaches such as Azure Bastion instead of exposing RDP/SSH directly to
the public internet.

------------------------------------------------------------------------

# 10. Lab --- Building a Windows Virtual Machine

## 10.1 Objective

Create a Windows VM and understand:

``` text
Resource Group
VM
OS Image
Size
Administrator
Disk
NIC
Public IP
NSG
RDP
```

------------------------------------------------------------------------

# 10.2 Step 1 --- Create Resource Group

Azure Portal:

``` text
Resource groups
    |
Create
```

Example:

``` text
Name:
rg-windows-lab

Region:
Central India
```

------------------------------------------------------------------------

# 10.3 Step 2 --- Create Virtual Machine

Go to:

``` text
Virtual machines
    |
Create
    |
Azure virtual machine
```

Choose:

``` text
Subscription
Resource Group
VM name
Region
Availability option
Security type
Image
VM architecture
Size
```

------------------------------------------------------------------------

# 10.4 Step 3 --- Select Windows Image

Example:

``` text
Windows Server 2022
```

The exact available image/SKU can vary by region and subscription.

------------------------------------------------------------------------

# 10.5 Step 4 --- Authentication

Configure the administrator account.

Example:

``` text
Username:
azureadmin

Password:
StrongPasswordHere
```

Use a strong unique password and do not put real credentials into notes,
scripts, Git repositories, or screenshots.

------------------------------------------------------------------------

# 10.6 Step 5 --- Configure Disks

Typical configuration:

``` text
OS Disk
 |
Windows operating system
```

Optional:

``` text
Data Disk
 |
Application data
```

Example:

``` text
C: -> OS
D: -> Application/Data
```

------------------------------------------------------------------------

# 10.7 Step 6 --- Networking

Azure can create:

``` text
VNet
Subnet
NIC
Public IP
NSG
```

Example:

``` text
VNet:
vnet-lab

Subnet:
subnet-vm

NSG:
nsg-win-vm

Public IP:
pip-win-vm
```

------------------------------------------------------------------------

# 10.8 Step 7 --- RDP Port

RDP normally uses:

``` text
TCP 3389
```

The NSG must allow the required traffic.

Safer design:

``` text
Allow RDP
From: Your trusted public IP
To: VM
Port: 3389
```

Avoid:

``` text
Internet -> 3389 -> VM
```

unless there is a specific controlled reason and additional protection.

------------------------------------------------------------------------

# 10.9 Step 8 --- Create the VM

Click:

``` text
Review + create
```

Azure validates the configuration.

Then:

``` text
Create
```

After deployment, you can inspect:

``` text
VM
 |
Overview
 |
Networking
 |
Disks
 |
Extensions
 |
Identity
 |
Monitoring
```

------------------------------------------------------------------------

# 10.10 Verify the VM

Check:

``` text
VM Status = Running
```

Then inspect:

``` text
Public IP
Private IP
VM size
OS
Disks
NSG
```

------------------------------------------------------------------------

# 11. Lab --- Connecting to the Virtual Machine

For a Windows VM, the common management protocol is **RDP (Remote
Desktop Protocol)**.

------------------------------------------------------------------------

# 11.1 RDP Architecture

``` text
Your Computer
      |
      | TCP 3389
      |
Internet
      |
Azure Public IP
      |
     NSG
      |
     NIC
      |
Windows VM
      |
Windows Remote Desktop
```

All required parts must work.

------------------------------------------------------------------------

# 11.2 Step 1 --- Get Public IP

Azure Portal:

``` text
Virtual Machine
 |
Overview
 |
Public IP address
```

Example:

``` text
20.x.x.x
```

Do not assume the example address is yours.

------------------------------------------------------------------------

# 11.3 Step 2 --- Download RDP File

From the VM:

``` text
Connect
 |
Native RDP
 |
Download RDP file
```

The `.rdp` file contains connection settings.

It does not replace your need for valid authentication.

------------------------------------------------------------------------

# 11.4 Step 3 --- Open RDP

On Windows:

``` text
Remote Desktop Connection
```

Enter:

``` text
Computer:
<VM-public-IP>
```

Then provide the VM administrator credentials.

------------------------------------------------------------------------

# 11.5 What Happens During RDP Connection?

Conceptually:

``` text
Client
  |
  | TCP connection to 3389
  |
Public IP
  |
NSG checks traffic
  |
NIC
  |
Windows networking
  |
Remote Desktop service
  |
Authentication
  |
Windows desktop
```

If any important layer fails, RDP can fail.

------------------------------------------------------------------------

# 11.6 Check Windows RDP Service

Inside the Windows VM, PowerShell can be used to inspect the Remote
Desktop service:

``` powershell
Get-Service TermService
```

Expected state:

``` text
Running
```

------------------------------------------------------------------------

# 11.7 Test Port From a Windows Client

PowerShell:

``` powershell
Test-NetConnection <PUBLIC-IP> -Port 3389
```

Example:

``` powershell
Test-NetConnection 20.x.x.x -Port 3389
```

If:

``` text
TcpTestSucceeded : True
```

the TCP port is reachable from that client.

This does not by itself prove that authentication will succeed.

------------------------------------------------------------------------

# 11.8 Common RDP Problems

### Problem 1 --- VM is stopped

Check:

``` text
VM -> Overview -> Status
```

Start it.

### Problem 2 --- Wrong public IP

Verify the current public IP.

### Problem 3 --- NSG blocks 3389

Check:

``` text
VM
 |
Networking
 |
Inbound port rules
```

### Problem 4 --- Windows firewall blocks RDP

Check Windows Defender Firewall.

### Problem 5 --- Remote Desktop service is stopped

Check:

``` powershell
Get-Service TermService
```

### Problem 6 --- Wrong username/password

Verify credentials.

### Problem 7 --- Network connectivity issue

Test:

``` powershell
Test-NetConnection <IP> -Port 3389
```

------------------------------------------------------------------------

# 12. Lab --- Resizing the Virtual Machine

## 12.1 What is VM Resizing?

Resizing means changing the VM size.

Example:

``` text
Before:
2 vCPU
8 GB RAM

After:
4 vCPU
16 GB RAM
```

This is useful when the workload needs more resources.

------------------------------------------------------------------------

# 12.2 Why Resize?

Common reasons:

### CPU is high

``` text
CPU = 95–100%
```

Increase CPU capacity.

### Memory pressure

``` text
RAM = 90–100%
```

Move to a memory-capable size.

### Application grows

More users require more resources.

### Cost optimization

The VM may be oversized.

You can move to a smaller suitable size.

------------------------------------------------------------------------

# 12.3 Example

Current VM:

``` text
Size:
Small

CPU:
2 vCPU

RAM:
8 GB
```

Application grows:

``` text
CPU consistently > 90%
```

You may resize:

``` text
Medium

CPU:
4 vCPU

RAM:
16 GB
```

------------------------------------------------------------------------

# 12.4 Resize from Azure Portal

Go to:

``` text
Virtual Machine
 |
Size
```

Select a supported size.

Then:

``` text
Resize
```

Azure may need to restart/deallocate the VM depending on the resize and
environment.

------------------------------------------------------------------------

# 12.5 Important: Resize Availability

Not every VM size is available:

-   In every region
-   On every subscription
-   On every VM generation
-   On every hardware cluster
-   At every moment

You may see errors such as:

``` text
NotAvailableForSubscription
```

or availability constraints.

If a size is unavailable:

``` text
Choose another supported size
```

or consider another region if appropriate.

------------------------------------------------------------------------

# 12.6 Resize Impact

Before resizing production systems, consider:

``` text
Downtime
Application availability
Size availability
Cost
Disk compatibility
Network performance
Licensing
Capacity limits
```

------------------------------------------------------------------------

# 12.7 Check VM Size Using Azure CLI

``` bash
az vm show \
  --resource-group rg-vm-lab \
  --name win-vm-01 \
  --query hardwareProfile.vmSize \
  -o tsv
```

Example output:

``` text
Standard_D2s_v5
```

------------------------------------------------------------------------

# 12.8 Change VM Size Using CLI

Example:

``` bash
az vm resize \
  --resource-group rg-vm-lab \
  --name win-vm-01 \
  --size Standard_D4s_v5
```

The exact size must be available for your VM/region/subscription.

------------------------------------------------------------------------

# 12.9 Resize vs Scale Out

These are different concepts.

### Resize / Scale Up

Increase resources of one VM:

``` text
VM
2 CPU
   |
   v
VM
4 CPU
```

This is called:

``` text
Vertical scaling
Scale up
```

### Scale Out

Add more VM instances:

``` text
       Load Balancer
        /          \
      VM1          VM2
```

This is:

``` text
Horizontal scaling
Scale out
```

------------------------------------------------------------------------

# 12.10 When to Use Which?

  Situation                                 Approach
  ----------------------------------------- -----------------
  One VM needs more CPU                     Resize
  One VM needs more RAM                     Resize
  Need high availability                    Scale out
  Need to handle more traffic               Often scale out
  Need to reduce cost                       Resize down
  Application supports multiple instances   Scale out

------------------------------------------------------------------------

# 13. Connecting to the Azure Virtual Machine from macOS

This topic explains how a Mac can connect to an Azure VM.

------------------------------------------------------------------------

# 13.1 Windows VM from macOS

For Windows VM:

``` text
macOS
  |
RDP client
  |
Internet
  |
Azure Public IP
  |
TCP 3389
  |
Windows VM
```

A Mac requires an RDP-compatible client/application.

Microsoft provides a Remote Desktop client for macOS through its
supported Microsoft remote desktop offering.

The exact application name/UI can change over time, so use the current
Microsoft-supported client from the official Microsoft source.

------------------------------------------------------------------------

# 13.2 Basic Steps

### Step 1

Get the VM public IP:

``` text
Azure Portal
 |
Virtual Machine
 |
Overview
 |
Public IP
```

### Step 2

Verify RDP access is permitted.

Check:

``` text
Networking
 |
NSG
 |
Inbound rules
 |
TCP 3389
```

### Step 3

On macOS, open your RDP client.

Create a new connection.

Example:

``` text
PC name:
20.x.x.x
```

### Step 4

Enter the Windows VM administrator account.

### Step 5

Connect.

------------------------------------------------------------------------

# 13.3 SSH to a Linux VM from macOS

macOS includes an SSH client in Terminal.

Example:

``` bash
ssh azureuser@20.x.x.x
```

If a private key is required:

``` bash
ssh -i ~/.ssh/id_ed25519 azureuser@20.x.x.x
```

Set appropriate private-key permissions:

``` bash
chmod 600 ~/.ssh/id_ed25519
```

------------------------------------------------------------------------

# 13.4 SSH Architecture

``` text
macOS Terminal
      |
      | TCP 22
      |
Azure Public IP
      |
     NSG
      |
     NIC
      |
Linux VM
      |
SSH service
```

------------------------------------------------------------------------

# 13.5 Troubleshooting SSH

Test the port:

``` bash
nc -vz <PUBLIC-IP> 22
```

Example:

``` bash
nc -vz 20.x.x.x 22
```

You can also use:

``` bash
ssh -v azureuser@20.x.x.x
```

The `-v` option provides verbose connection information.

For more debugging:

``` bash
ssh -vvv azureuser@20.x.x.x
```

------------------------------------------------------------------------

# 13.6 Common SSH Problems

### Permission denied

Possible causes:

``` text
Wrong username
Wrong private key
Wrong permissions
Incorrect authorized_keys
```

Check:

``` bash
chmod 600 ~/.ssh/id_ed25519
```

### Connection timed out

Likely network-level issue:

``` text
NSG
Public IP
Routing
Firewall
VM state
```

### Connection refused

The machine may be reachable, but the SSH service may not be listening
on the expected port.

Inside Linux:

``` bash
sudo systemctl status ssh
```

or on some distributions:

``` bash
sudo systemctl status sshd
```

------------------------------------------------------------------------

# 14. Troubleshooting Connectivity

Connectivity troubleshooting should be systematic.

Do not immediately assume:

``` text
"Azure is down"
```

Instead, test each layer.

------------------------------------------------------------------------

# 14.1 Connectivity Troubleshooting Model

Use this sequence:

``` text
1. Is the VM running?
        |
2. Does it have the expected IP?
        |
3. Is the NIC attached?
        |
4. Is the subnet correct?
        |
5. Does the NSG allow traffic?
        |
6. Is routing correct?
        |
7. Is the guest OS firewall allowing traffic?
        |
8. Is the service running?
        |
9. Is the application listening on the expected port?
        |
10. Are credentials correct?
```

------------------------------------------------------------------------

# 14.2 Layer 1 --- Check VM Power State

Azure Portal:

``` text
Virtual Machine
 |
Overview
 |
Status
```

CLI:

``` bash
az vm get-instance-view \
  --resource-group rg-vm-lab \
  --name win-vm-01 \
  --query instanceView.statuses
```

If the VM is stopped, start it.

------------------------------------------------------------------------

# 14.3 Layer 2 --- Check IP Address

Check:

``` text
Public IP
Private IP
```

A common mistake is using an old public IP address.

If the public IP resource is not static/preserved as expected, its
address can change after certain lifecycle events.

Always verify the current address in Azure.

------------------------------------------------------------------------

# 14.4 Layer 3 --- Check NIC

Go to:

``` text
VM
 |
Networking
 |
Network Interface
```

Verify:

``` text
NIC exists
NIC is attached
Private IP is present
NSG association is expected
```

------------------------------------------------------------------------

# 14.5 Layer 4 --- Check NSG

Suppose you need RDP.

Required:

``` text
Protocol: TCP
Destination port: 3389
Source: Trusted source
Action: Allow
```

For SSH:

``` text
TCP 22
```

For HTTPS:

``` text
TCP 443
```

------------------------------------------------------------------------

# 14.6 NSG Rule Priority

Azure NSG rules have priorities.

Example:

``` text
Priority 100
Deny TCP 3389 from Internet

Priority 200
Allow TCP 3389 from My-IP
```

The lower numerical priority is evaluated first.

Therefore:

``` text
100 < 200
```

The priority 100 deny can prevent the later allow from taking effect.

### Important

When troubleshooting NSGs, inspect:

``` text
Source
Destination
Protocol
Port
Direction
Action
Priority
```

------------------------------------------------------------------------

# 14.7 Layer 5 --- Check Routing

Azure networking uses routes to determine where traffic goes.

Check:

``` text
VNet
 |
Subnet
 |
Route tables
 |
Routes
```

A custom route can change normal traffic flow.

Example:

``` text
VM
 |
Subnet
 |
UDR
 |
Network Virtual Appliance
 |
Firewall
 |
Internet
```

If routing is incorrect, the packet may never reach its destination.

------------------------------------------------------------------------

# 14.8 Layer 6 --- Check Guest OS Firewall

Azure networking can be correct while the OS blocks the connection.

## Windows

Windows Defender Firewall may block:

``` text
RDP
HTTP
HTTPS
Custom ports
```

## Linux

Check firewall tools such as:

``` bash
sudo ufw status
```

or:

``` bash
sudo firewall-cmd --list-all
```

depending on the Linux distribution.

------------------------------------------------------------------------

# 14.9 Layer 7 --- Check Service

A port can be allowed but the application may not be running.

Linux:

``` bash
sudo systemctl status nginx
```

Check listening ports:

``` bash
sudo ss -lntp
```

Example:

``` text
LISTEN 0 511 0.0.0.0:80
```

This means a service is listening on TCP port 80 on all IPv4 interfaces.

------------------------------------------------------------------------

# 14.10 Layer 8 --- Check Application

Suppose:

``` text
NSG allows TCP 8080
```

but the application listens on:

``` text
127.0.0.1:8080
```

External clients may still be unable to reach it.

A service intended to accept network traffic may need to listen on the
appropriate interface, such as:

``` text
0.0.0.0:8080
```

rather than only:

``` text
127.0.0.1:8080
```

The correct binding depends on the application and security design.

------------------------------------------------------------------------

# 14.11 Test Connectivity From Linux

### Ping

``` bash
ping <IP>
```

But remember:

**Ping uses ICMP**, not TCP.

A failed ping does not necessarily mean TCP connectivity is unavailable
because ICMP may be blocked.

### Test TCP port

``` bash
nc -vz <IP> 22
```

or:

``` bash
nc -vz <IP> 80
```

### Curl

``` bash
curl -I http://<IP>
```

For HTTPS:

``` bash
curl -vk https://<IP>
```

------------------------------------------------------------------------

# 14.12 Test Connectivity From Windows

PowerShell:

``` powershell
Test-NetConnection <IP> -Port 3389
```

Example:

``` powershell
Test-NetConnection 20.x.x.x -Port 3389
```

For HTTPS:

``` powershell
Test-NetConnection example.com -Port 443
```

------------------------------------------------------------------------

# 14.13 Test DNS

If the user connects using a hostname:

``` bash
nslookup example.com
```

or:

``` bash
dig example.com
```

If DNS resolves to the wrong IP:

``` text
DNS problem
```

If DNS is correct but the port fails:

``` text
Network/service problem
```

------------------------------------------------------------------------

# 14.14 Common Connectivity Symptoms

## Symptom: Timeout

Example:

``` text
Connection timed out
```

Likely areas:

``` text
VM stopped
Wrong IP
NSG
Route
Firewall
Network path
```

------------------------------------------------------------------------

## Symptom: Connection refused

Example:

``` text
Connection refused
```

Usually means the destination is reachable but nothing is accepting the
connection on that port, or an active device/service is rejecting it.

Check:

``` text
Service status
Listening port
Guest firewall
```

------------------------------------------------------------------------

## Symptom: Permission denied

For SSH:

``` text
Permission denied (publickey)
```

Check:

``` text
Username
Private key
Authorized keys
Key permissions
SSH configuration
```

------------------------------------------------------------------------

## Symptom: RDP authentication failure

Check:

``` text
Username
Password
Account status
RDP configuration
Windows policies
```

------------------------------------------------------------------------

# 14.15 Azure Network Watcher

Azure Network Watcher provides network diagnostic capabilities.

Useful tools include:

``` text
IP flow verify
Next hop
Connection troubleshoot
Packet capture
Topology
```

------------------------------------------------------------------------

# 14.16 IP Flow Verify

IP Flow Verify can help determine whether traffic is allowed or denied
by NSG rules.

Conceptually:

``` text
Source
   |
Destination
   |
Protocol
   |
Port
   |
NSG evaluation
   |
Allow / Deny
```

Example:

``` text
Source: Your IP
Destination: VM
Protocol: TCP
Port: 3389
```

Result:

``` text
Allowed
```

or:

``` text
Denied
```

This is very useful when an NSG rule is suspected.

------------------------------------------------------------------------

# 14.17 Connection Troubleshoot

Connection troubleshoot can help diagnose connectivity between
endpoints.

Example:

``` text
VM
 |
Connection troubleshoot
 |
Destination:
20.x.x.x:443
```

It can help identify issues in the network path.

------------------------------------------------------------------------

# 14.18 Packet Capture

Packet capture can be useful for deeper network troubleshooting.

Example scenario:

``` text
Client sends packets
       |
       v
VM receives?
       |
       +-- No
       |
       +-- Yes
            |
            v
        Application
```

Packet captures should be used carefully because they can contain
sensitive traffic information.

------------------------------------------------------------------------

# 14.19 Complete Troubleshooting Example

### Problem

You created a Windows VM but RDP is not working.

### Step 1 --- Check VM

``` text
VM status = Running
```

Good.

### Step 2 --- Check public IP

``` text
Public IP = 20.x.x.x
```

Good.

### Step 3 --- Check RDP NSG rule

You find:

``` text
TCP 3389
Source = Your IP
Action = Allow
```

Good.

### Step 4 --- Test from client

``` powershell
Test-NetConnection 20.x.x.x -Port 3389
```

Result:

``` text
TcpTestSucceeded : False
```

Continue.

### Step 5 --- Check Windows firewall

Inside the VM, verify the RDP firewall rules.

### Step 6 --- Check Remote Desktop service

``` powershell
Get-Service TermService
```

If:

``` text
Stopped
```

start/fix the service according to the VM configuration.

### Step 7 --- Test again

``` powershell
Test-NetConnection 20.x.x.x -Port 3389
```

If:

``` text
TcpTestSucceeded : True
```

network-level connectivity is working.

If RDP still fails, investigate:

``` text
Credentials
RDP configuration
Windows policies
User permissions
```

------------------------------------------------------------------------

# 14.20 Production Troubleshooting Flow

Use this practical flow:

``` text
                 Connection Problem
                         |
                         v
                 Is VM running?
                    /        \
                  No          Yes
                  |            |
                Start       Correct IP?
                              /    \
                            No      Yes
                            |        |
                         Fix IP    NSG?
                                    / \
                                  No   Yes
                                  |     |
                               Fix NSG  Route?
                                         / \
                                       No   Yes
                                       |     |
                                    Fix route  OS firewall?
                                                 / \
                                               Yes  No
                                               |     |
                                            Fix FW  Service?
                                                       / \
                                                     No   Yes
                                                     |     |
                                                  Start   App config?
                                                          |
                                                        Test
```

------------------------------------------------------------------------

# Practical Commands Cheat Sheet

## Azure CLI

### Login

``` bash
az login
```

### List VMs

``` bash
az vm list -o table
```

### Show VM

``` bash
az vm show \
  --resource-group <RESOURCE_GROUP> \
  --name <VM_NAME>
```

### VM status

``` bash
az vm get-instance-view \
  --resource-group <RESOURCE_GROUP> \
  --name <VM_NAME> \
  --query instanceView.statuses
```

### Start

``` bash
az vm start \
  --resource-group <RESOURCE_GROUP> \
  --name <VM_NAME>
```

### Stop/deallocate

``` bash
az vm deallocate \
  --resource-group <RESOURCE_GROUP> \
  --name <VM_NAME>
```

### Restart

``` bash
az vm restart \
  --resource-group <RESOURCE_GROUP> \
  --name <VM_NAME>
```

### Show VM size

``` bash
az vm show \
  --resource-group <RESOURCE_GROUP> \
  --name <VM_NAME> \
  --query hardwareProfile.vmSize \
  -o tsv
```

### Resize

``` bash
az vm resize \
  --resource-group <RESOURCE_GROUP> \
  --name <VM_NAME> \
  --size <AVAILABLE_VM_SIZE>
```

------------------------------------------------------------------------

# Linux Connectivity Commands

## Check service

``` bash
sudo systemctl status ssh
```

``` bash
sudo systemctl status nginx
```

## Check listening ports

``` bash
sudo ss -lntp
```

## Check firewall

``` bash
sudo ufw status
```

## Test TCP port

``` bash
nc -vz <IP> <PORT>
```

## SSH verbose mode

``` bash
ssh -v user@<IP>
```

## HTTP test

``` bash
curl -I http://<IP>
```

## DNS

``` bash
nslookup example.com
```

------------------------------------------------------------------------

# Windows Connectivity Commands

## Check RDP service

``` powershell
Get-Service TermService
```

## Test TCP

``` powershell
Test-NetConnection <IP> -Port 3389
```

## Check network configuration

``` powershell
ipconfig
```

## Test DNS

``` powershell
nslookup example.com
```

------------------------------------------------------------------------

# Important Comparisons

## VM vs VM Size

``` text
VM
= Virtual computer

VM Size
= CPU/RAM/network/storage capability assigned to the VM
```

------------------------------------------------------------------------

## Public IP vs Private IP

  Public IP                        Private IP
  -------------------------------- -----------------------------------------------
  Internet-facing address          VNet/internal address
  Used for external connectivity   Used for internal communication
  Optional                         Required for normal VNet connectivity
  Should be protected              Internal but still requires security controls

------------------------------------------------------------------------

## OS Disk vs Data Disk

  OS Disk                          Data Disk
  -------------------------------- ---------------------------------
  Contains operating system        Contains application/user data
  Required for normal VM boot      Optional
  Windows/Linux system files       Database/files/application data
  Usually attached automatically   Added when required

------------------------------------------------------------------------

## RDP vs SSH

  RDP                         SSH
  --------------------------- ----------------------------
  Commonly used for Windows   Commonly used for Linux
  TCP 3389                    TCP 22
  Graphical remote desktop    Command-line remote access
  Windows Remote Desktop      OpenSSH

------------------------------------------------------------------------

## Resize vs Scale Out

  Resize                           Scale Out
  -------------------------------- --------------------------------
  Increase/decrease VM resources   Add/remove VM instances
  Vertical scaling                 Horizontal scaling
  CPU/RAM changes                  Number of instances changes
  Can require restart              Often used with load balancing

------------------------------------------------------------------------

# AZ-104 / Interview Questions

## 1. What is an Azure VM?

An Azure VM is an on-demand virtual server in Azure that provides
control over an operating system, compute resources, storage, and
networking.

------------------------------------------------------------------------

## 2. What are the main resources associated with a VM?

Common resources include:

``` text
VM
NIC
OS Disk
Data Disks
VNet
Subnet
NSG
Public IP (optional)
```

------------------------------------------------------------------------

## 3. Is a public IP required for an Azure VM?

No.

A VM can operate with private networking only.

------------------------------------------------------------------------

## 4. What port does RDP use?

``` text
TCP 3389
```

------------------------------------------------------------------------

## 5. What port does SSH use?

Normally:

``` text
TCP 22
```

------------------------------------------------------------------------

## 6. What is an NSG?

A Network Security Group contains rules that allow or deny network
traffic to supported Azure network resources such as NICs and subnets.

------------------------------------------------------------------------

## 7. Why can RDP fail even when the VM is running?

Possible causes:

``` text
Wrong IP
NSG blocking 3389
Route issue
Windows firewall
RDP service problem
Authentication problem
```

------------------------------------------------------------------------

## 8. What is VM resizing?

Changing the VM size to change its compute resources.

Example:

``` text
2 vCPU / 8 GB
        |
      Resize
        |
4 vCPU / 16 GB
```

------------------------------------------------------------------------

## 9. What is the difference between stop and deallocate?

Stopping a VM and deallocating it are not necessarily the same.
Deallocation releases the compute allocation and is commonly used when
you want to stop supported VM compute charges. Disks and other attached
resources can continue to incur charges.

------------------------------------------------------------------------

## 10. Can every VM be resized to every size?

No.

Availability depends on:

``` text
Region
Subscription capacity
VM generation
SKU availability
Architecture
Other constraints
```

------------------------------------------------------------------------

## 11. Does having a public IP automatically expose RDP?

No.

Network security rules, routing, guest firewall, and the RDP service all
matter.

------------------------------------------------------------------------

## 12. What would you check first when SSH is not working?

A practical order is:

``` text
1. VM running?
2. Correct public/private IP?
3. NSG allows TCP 22?
4. Route correct?
5. Guest firewall?
6. SSH service running?
7. Port listening?
8. Correct username/key?
```

------------------------------------------------------------------------

# Real-World Scenario

## Scenario

A company hosts a Windows web application on Azure.

Architecture:

``` text
                    Internet
                       |
                    HTTPS 443
                       |
                Azure Public IP
                       |
                      NSG
                       |
                 Load Balancer
                   /       \
                  /         \
              VM-01        VM-02
             Windows       Windows
                |             |
               IIS           IIS
                |             |
             App Data / Shared Backend
```

Administrators need management access.

A secure design should avoid unnecessarily exposing RDP to the entire
internet.

Possible management architecture:

``` text
Administrator
      |
Secure management path
      |
Azure Bastion / controlled private connectivity
      |
VNet
      |
Windows VM
```

This separates:

``` text
Application traffic
```

from:

``` text
Administrative traffic
```

------------------------------------------------------------------------

# Final Revision Summary

Remember these points:

``` text
Azure VM
= Compute resource

VM Size
= CPU/RAM/performance capacity

OS Disk
= Operating system

Data Disk
= Application/user data

NIC
= Network connection for VM

Private IP
= Internal VNet communication

Public IP
= Optional external connectivity

NSG
= Network traffic rules

RDP
= Windows remote administration, normally TCP 3389

SSH
= Linux remote administration, normally TCP 22

Resize
= Vertical scaling

Scale Out
= Horizontal scaling

Network Watcher
= Azure network diagnostics
```

## Most Important Troubleshooting Rule

When a VM connection fails, do not check only one thing.

Check the complete path:

``` text
Client
  ↓
DNS / IP
  ↓
Internet / Private Network
  ↓
Azure Route
  ↓
NSG
  ↓
NIC
  ↓
Guest OS Firewall
  ↓
Service
  ↓
Application
  ↓
Authentication
```

This layered approach is one of the most important practical skills for
Azure administration and DevOps.
