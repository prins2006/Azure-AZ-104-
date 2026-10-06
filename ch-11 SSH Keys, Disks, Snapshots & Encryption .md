# Azure Virtual Machines — SSH Keys, Disks, Snapshots & Encryption

## Topics Covered

1. Deploying a Linux VM using SSH Keys
2. Azure Virtual Machine Disks
3. Adding Data Disks
4. What Happens When We Stop an Azure VM
5. Data Disk Snapshots
6. Azure Disk Server-Side Encryption
7. Azure Key Vault
8. Disk Encryption Sets
9. Exam-Based Questions
10. Practical Scenarios
11. Quick Revision

---

# 1. Lab — Deploying a Linux Machine Using SSH Keys

## What is SSH?

SSH stands for:

> Secure Shell

SSH is a secure protocol used to connect to a remote Linux machine.

For example:

```text
Your Laptop
     |
     | SSH
     |
     v
Azure Linux VM
```

Instead of using a username/password, Azure Linux VMs can use an **SSH key pair**.

---

## What is an SSH Key Pair?

An SSH key pair contains two keys:

```text
Private Key
+
Public Key
```

### Private Key

The private key stays on your local machine.

Example:

```text
~/.ssh/id_ed25519
```

Never share your private key.

### Public Key

The public key is placed on the Azure VM.

Example:

```text
~/.ssh/id_ed25519.pub
```

The public key can be shared with the server.

---

## How SSH Authentication Works

```text
                 SSH Connection
Laptop --------------------------------> Azure VM

Private Key                         Public Key
   🔑                                  🔑
   |                                    |
   +------------ Authentication --------+
```

The server verifies that your private key matches the public key stored on the VM.

---

## Generate an SSH Key

On Linux:

```bash
ssh-keygen -t ed25519
```

You may see:

```text
Enter file in which to save the key:
```

Press Enter to use the default location.

Usually:

```text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

Check:

```bash
ls -la ~/.ssh/
```

---

## Connect to an Azure Linux VM

Example:

```bash
ssh -i ~/.ssh/id_ed25519 azureuser@20.10.20.30
```

Where:

```text
-i ~/.ssh/id_ed25519
        |
        +-- Private SSH key

azureuser
        |
        +-- VM username

20.10.20.30
        |
        +-- VM public IP
```

---

## Why SSH Keys Are Better Than Passwords

| SSH Keys                        | Password                      |
| ------------------------------- | ----------------------------- |
| Strong authentication           | Can be guessed/brute-forced   |
| Private key remains local       | Password must be entered      |
| Good for automation             | Less suitable for automation  |
| Can use passphrase              | Password-based authentication |
| Common for Linux administration | Simple but less secure        |

---

# 2. Azure Virtual Machine Disks

An Azure VM normally uses different types of disks.

```text
Azure VM
   |
   +--- OS Disk
   |
   +--- Temporary Disk
   |
   +--- Data Disk
```

---

# OS Disk

The OS disk contains the operating system.

For example:

```text
Ubuntu
Windows Server
Red Hat
SUSE
```

Example:

```text
Azure Linux VM
       |
       +---- OS Disk
              |
              +---- Ubuntu OS
              +---- System files
              +---- Installed packages
```

The OS disk is required for the VM to boot.

---

# Data Disk

A data disk is used to store application data.

Example:

```text
Linux VM
 |
 +-- OS Disk
 |     └── Ubuntu
 |
 +-- Data Disk
       └── Application Data
```

Example:

A database server might use:

```text
OS Disk
   -> Operating System

Data Disk
   -> PostgreSQL database

Another Data Disk
   -> Backup files
```

---

# Temporary Disk

Azure VMs can also have a temporary disk.

It is intended for temporary data such as:

```text
cache
temporary files
swap/page files
```

You should **not store important persistent data** on the temporary disk.

---

# Important Disk Comparison

| Disk           | Purpose                  | Persistent? |
| -------------- | ------------------------ | ----------- |
| OS Disk        | Operating system         | Yes         |
| Data Disk      | Application/data storage | Yes         |
| Temporary Disk | Temporary/cache data     | No          |

---

# Azure Managed Disks

Azure Managed Disks are storage volumes managed by Azure.

You don't have to manually manage storage accounts for normal VM disk management.

Azure manages:

```text
Storage
Availability
Disk placement
Management
```

---

# Common Managed Disk Types

Common Azure disk options include:

```text
Standard HDD
Standard SSD
Premium SSD
Premium SSD v2
Ultra Disk
```

General idea:

| Disk           | Typical Use                        |
| -------------- | ---------------------------------- |
| Standard HDD   | Low-cost workloads                 |
| Standard SSD   | General workloads                  |
| Premium SSD    | Production / performance workloads |
| Premium SSD v2 | High-performance workloads         |
| Ultra Disk     | Very high-performance workloads    |

---

# Example

Suppose you deploy an application server:

```text
VM
 |
 +-- OS Disk
 |     30 GB
 |
 +-- Data Disk
       128 GB
```

The OS disk contains:

```text
Ubuntu
Nginx
Docker
Application packages
```

The data disk contains:

```text
/var/www/data
```

---

# 3. Lab — Adding Data Disks

Suppose we already have:

```text
Azure Linux VM
```

Now we want additional storage.

We create:

```text
Data Disk
   |
   +--- Attach to VM
```

---

## Azure Portal Steps

General process:

```text
Azure Portal
   ↓
Virtual Machines
   ↓
Select VM
   ↓
Disks
   ↓
Create and attach a new disk
   ↓
Select disk size/type
   ↓
Save
```

---

# What Happens After Attaching a Disk?

Attaching a disk does not automatically mean Linux has mounted it.

For example:

```text
Azure
 |
 +-- Disk attached
       |
       v
Linux
 |
 +-- Disk detected
       |
       +-- Partition
       |
       +-- Filesystem
       |
       +-- Mount
```

---

# Check Disks in Linux

Use:

```bash
lsblk
```

Example:

```text
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda      8:0    0   30G  0 disk
├─sda1   8:1    0   29G  0 part /
sdb      8:16   0  128G  0 disk
```

Here:

```text
sda = OS disk
sdb = newly attached data disk
```

---

# Create a Partition

Example:

```bash
sudo fdisk /dev/sdb
```

Then create a partition.

Afterward:

```bash
lsblk
```

You may see:

```text
sdb
└─sdb1
```

---

# Create a Filesystem

Example:

```bash
sudo mkfs.ext4 /dev/sdb1
```

---

# Create Mount Directory

```bash
sudo mkdir /data
```

Mount the disk:

```bash
sudo mount /dev/sdb1 /data
```

Check:

```bash
df -h
```

---

# Verify

```bash
lsblk
```

Example:

```text
sdb
└─sdb1   128G   /data
```

Now:

```bash
cd /data
```

The disk can be used for application data.

---

# Important Production Point

If you manually mount a disk using:

```bash
mount /dev/sdb1 /data
```

the mount may not automatically persist after reboot.

For persistent mounting, configure:

```text
/etc/fstab
```

Prefer using the disk UUID.

Check UUID:

```bash
sudo blkid
```

Example:

```text
/dev/sdb1: UUID="abc123..." TYPE="ext4"
```

Then configure `/etc/fstab`.

---

# 4. What Happens When We Stop an Azure VM?

This is an important Azure concept.

There is a difference between:

```text
Stop from inside OS
```

and:

```text
Stop/Deallocate from Azure
```

---

# Stop From Inside Linux

For example:

```bash
sudo shutdown -h now
```

or:

```bash
sudo poweroff
```

The operating system shuts down.

However, the Azure VM may still be in an allocated state.

---

# Stop/Deallocate From Azure

When you stop/deallocate a VM from Azure:

```text
VM
 ↓
Stopped
 ↓
Deallocated
```

The compute resources are released.

For many VM types, this means you stop paying the VM's compute charge.

However, attached resources such as managed disks and some networking resources can still incur charges.

---

# Stopped vs Deallocated

| State                 | Compute Resources    | VM Compute Billing    |
| --------------------- | -------------------- | --------------------- |
| Running               | Allocated            | Yes                   |
| Stopped inside OS     | May remain allocated | Can continue          |
| Stopped + Deallocated | Released             | Compute billing stops |

---

# Important Exam Point

Remember:

> **Stopping a VM is not always the same as deallocating a VM.**

For cost optimization, deallocation is important.

---

# What Happens to Data?

Suppose:

```text
VM
 |
 +-- OS Disk
 +-- Data Disk
```

When VM is deallocated:

```text
VM compute resources
        ↓
      Released

Managed Disks
        ↓
       Remain
```

Your managed disk data remains.

---

# Example

You have a development VM.

You only use it:

```text
9 AM → 6 PM
```

At 6 PM, you deallocate it.

At 9 AM, start it again.

This can reduce compute costs compared with leaving it running 24/7.

---

# 5. Lab — Data Disk Snapshot

## What is a Snapshot?

A snapshot is a point-in-time copy of a managed disk.

Example:

```text
Data Disk
   |
   +---- Snapshot
           |
           +---- Point-in-time copy
```

---

# Why Use Snapshots?

Snapshots can be used for:

* Backup-related workflows
* Testing
* Creating disks
* Disaster recovery scenarios
* Copying disk state
* Development/testing

---

# Example

Suppose:

```text
Data Disk
128 GB
```

contains:

```text
Application
Database files
Configuration
```

Before making a major change:

```text
Data Disk
   |
   +---- Create Snapshot
```

If something goes wrong, the snapshot can be used as a source for creating a disk.

---

# Snapshot Flow

```text
Existing Disk
     |
     v
 Create Snapshot
     |
     v
 Snapshot
     |
     v
 Create Managed Disk
     |
     v
 Attach to VM
```

---

# Important Point

A snapshot is **not the same thing as attaching another disk**.

A snapshot is a point-in-time representation of a disk.

---

# Example Scenario

Before upgrading a database:

```text
Database Data Disk
        |
        v
Create Snapshot
        |
        v
Upgrade Database
        |
        v
Problem?
        |
        v
Use snapshot to create a disk
```

---

# Snapshot vs Backup

They are related but not identical concepts.

| Snapshot                                     | Backup                                   |
| -------------------------------------------- | ---------------------------------------- |
| Point-in-time disk copy                      | Dedicated backup/recovery solution       |
| Useful for disk-level recovery               | Designed for broader backup requirements |
| Can create a disk from it                    | Managed through backup policies/services |
| Not automatically a complete backup strategy | Better for scheduled retention           |

For production environments, don't assume that creating one snapshot is a complete backup strategy.

---

# 6. Azure Disks — Server-Side Encryption

## What is Encryption?

Encryption converts readable data into protected data.

Example:

```text
Original Data
     |
     v
Encryption
     |
     v
Encrypted Data
```

If someone gets unauthorized access to the underlying storage, the encrypted data is protected.

---

# Server-Side Encryption (SSE)

Azure managed disks support **server-side encryption**.

The encryption happens on the Azure storage infrastructure.

Conceptually:

```text
VM
 |
 v
Managed Disk
 |
 v
Azure Storage
 |
 v
Encryption
```

---

# Types of Encryption Keys

Azure disk encryption can use:

```text
Platform-managed keys
```

or:

```text
Customer-managed keys
```

---

# Platform-Managed Keys

Azure manages the encryption keys.

Advantages:

* Simple
* Less administration
* Default encryption experience for many managed disk scenarios

---

# Customer-Managed Keys

With customer-managed keys:

```text
Customer
   |
   v
Azure Key Vault
   |
   v
Encryption Key
   |
   v
Disk Encryption
```

This provides greater control over key lifecycle and access.

---

# Why Use Customer-Managed Keys?

Organizations may need:

* Key rotation control
* Key access control
* Compliance requirements
* Separation of duties
* Ability to manage key lifecycle

---

# Important Exam Concept

Do not confuse:

```text
Server-Side Encryption
```

with:

```text
Azure Disk Encryption
```

They are related to protecting disk data but use different mechanisms and terminology.

For AZ-104, understand the distinction between managed disk SSE and VM guest-level disk encryption technologies.

---

# 7. Lab — Azure Key Vault Service

## What is Azure Key Vault?

Azure Key Vault is a service used to securely store and manage:

```text
Secrets
Keys
Certificates
```

---

# Simple Example

Suppose your application needs a database password.

Bad approach:

```text
app.py

DB_PASSWORD="MyPassword123"
```

The password is stored directly in code.

This is risky.

Better approach:

```text
Application
     |
     v
Azure Key Vault
     |
     v
Database Password
```

The application retrieves the secret securely.

---

# Key Vault Objects

Azure Key Vault mainly works with:

### Secrets

Examples:

```text
Database password
API key
Connection string
Token
```

### Keys

Used for:

```text
Encryption
Signing
Cryptographic operations
```

### Certificates

Used for:

```text
TLS/SSL
HTTPS
Certificate management
```

---

# Example

Suppose:

```text
Application
   |
   | Need DB password
   v
Key Vault
   |
   +---- DB_PASSWORD
   |
   v
Application
```

The password does not need to be hardcoded into the application source code.

---

# Key Vault and Managed Identity

A very important Azure concept is:

> Managed Identity

Instead of storing Key Vault credentials inside the application, an Azure resource can use a managed identity.

Example:

```text
Azure VM
   |
   | Managed Identity
   v
Azure Key Vault
   |
   v
Secret
```

This removes the need to store credentials in the VM/application.

---

# Example Scenario

Suppose a Linux VM runs a web application.

The application needs:

```text
DATABASE_PASSWORD
```

Instead of:

```text
export DATABASE_PASSWORD="password123"
```

You can use:

```text
VM
 |
 +-- Managed Identity
       |
       v
   Key Vault
       |
       +-- Database Password
```

This is a more secure architecture.

---

# Key Vault Access

Access to Key Vault can involve Azure identity and authorization mechanisms such as:

```text
Microsoft Entra ID
RBAC
Key Vault access policies
```

For modern Azure designs, Azure RBAC is commonly preferred where appropriate.

---

# 8. Lab — Disk Encryption Sets

## What is a Disk Encryption Set?

A Disk Encryption Set (DES) is an Azure resource that allows Azure managed disks and other supported resources to use a customer-managed key.

The key is stored in:

```text
Azure Key Vault
```

---

# Architecture

```text
                 Azure Key Vault
                       |
                       |
                 Customer Key
                       |
                       v
              Disk Encryption Set
                       |
                       v
                Managed Disk
                       |
                       v
                     VM
```

---

# Why Do We Need Disk Encryption Sets?

Suppose an organization says:

> "We need to control the encryption key used to encrypt our Azure managed disks."

You can use:

```text
Key Vault
     +
Disk Encryption Set
     +
Managed Disk
```

---

# Example

Imagine:

```text
Production VM
      |
      +---- OS Disk
      |
      +---- Data Disk
```

The company requires customer-managed encryption keys.

Architecture:

```text
Azure Key Vault
       |
       | Customer-managed key
       v
Disk Encryption Set
       |
       +----------------+
       |                |
       v                v
    OS Disk          Data Disk
       |
       v
    Azure VM
```

---

# High-Level Configuration Flow

1. Create Azure Key Vault.
2. Create an encryption key in Key Vault.
3. Create a Disk Encryption Set.
4. Associate the encryption key with the DES.
5. Give the DES identity appropriate permissions to use the key.
6. Configure supported managed disks to use the DES.

---

# Important Permission Concept

The Disk Encryption Set has an identity.

That identity needs appropriate access to the Key Vault key.

Conceptually:

```text
Disk Encryption Set Identity
          |
          | Permission
          v
      Key Vault Key
```

Without the required permission, encryption operations can fail.

---

# Key Vault vs Disk Encryption Set

| Azure Key Vault          | Disk Encryption Set                                        |
| ------------------------ | ---------------------------------------------------------- |
| Stores/manages keys      | References a Key Vault key for disk encryption             |
| Stores secrets           | Used with managed disk encryption                          |
| Stores certificates      | Provides association between disk and customer-managed key |
| General security service | Disk encryption-related resource                           |

---

# 9. Complete Architecture Example

Consider a production web server:

```text
                    Azure
                      |
              +-------+-------+
              |               |
           Key Vault        Linux VM
              |               |
        Customer Key       Managed Identity
              |               |
              v               |
       Disk Encryption Set   |
              |               |
              +-------+-------+
                      |
                  Managed Disk
                      |
                  Application
```

---

# Example Production Scenario

A company hosts an application on an Azure Linux VM.

Requirements:

1. Secure Linux login.
2. Store application data separately.
3. Encrypt disks.
4. Control encryption keys.
5. Store database credentials securely.

Solution:

```text
Requirement                Azure Solution

Secure Linux login     →   SSH Keys

Application storage    →   Managed Data Disk

Disk protection        →   Server-Side Encryption

Customer key control   →   Key Vault + Disk Encryption Set

DB password            →   Key Vault Secret

Secure application     →   Managed Identity
```

---

# 10. Important Commands

## Check SSH Directory

```bash
ls -la ~/.ssh/
```

---

## Generate SSH Key

```bash
ssh-keygen -t ed25519
```

---

## Connect to VM

```bash
ssh -i ~/.ssh/id_ed25519 azureuser@PUBLIC_IP
```

---

## List Linux Disks

```bash
lsblk
```

---

## Check Disk Usage

```bash
df -h
```

---

## Check Block Devices

```bash
sudo fdisk -l
```

---

## Check Disk UUID

```bash
sudo blkid
```

---

## Mount Disk

```bash
sudo mount /dev/sdb1 /data
```

---

## Check Mounted Disks

```bash
df -h
```

or:

```bash
lsblk
```

---

# 11. AZ-104 Exam-Based Questions

## Q1. You need to securely connect to an Azure Linux VM without using a password. What should you use?

A. Storage Account
B. SSH key pair
C. Azure Policy
D. Network Security Group

### Answer

**B. SSH key pair**

---

# Q2. Where should the private SSH key be stored?

A. Azure VM public IP
B. Azure Key Vault only
C. Local client securely
D. NSG

### Answer

**C. Local client securely**

---

# Q3. Which key is placed on the Linux VM?

A. Private key
B. Public key
C. Recovery key
D. Session key

### Answer

**B. Public key**

---

# Q4. You need additional persistent storage for an Azure VM. What should you use?

A. Temporary disk
B. Data disk
C. RAM
D. Swap only

### Answer

**B. Data disk**

---

# Q5. Which disk normally contains the operating system?

A. Data disk
B. Temporary disk
C. OS disk
D. Snapshot

### Answer

**C. OS disk**

---

# Q6. An application requires persistent data storage. Which disk should be used?

A. Temporary disk
B. Data disk
C. Cache
D. RAM

### Answer

**B. Data disk**

---

# Q7. What happens to managed disks when an Azure VM is deallocated?

A. They are automatically deleted
B. They remain available
C. They are converted into snapshots
D. They lose their data

### Answer

**B. They remain available**

---

# Q8. Your company wants to reduce VM compute costs when a development VM is not being used. What should you do?

A. Restart the VM
B. Deallocate the VM
C. Create a snapshot
D. Increase disk size

### Answer

**B. Deallocate the VM**

---

# Q9. What is the main purpose of an Azure disk snapshot?

A. Increase CPU
B. Create a point-in-time copy of a disk
C. Create a network interface
D. Increase RAM

### Answer

**B. Create a point-in-time copy of a disk**

---

# Q10. You want to recover a disk to an earlier point in time. What can you use?

A. Snapshot
B. NSG
C. Public IP
D. Load Balancer

### Answer

**A. Snapshot**

---

# Q11. Where are Azure managed disks encrypted?

A. Only inside the Linux operating system
B. On the Azure storage infrastructure
C. Only inside the user's laptop
D. Inside DNS

### Answer

**B. On the Azure storage infrastructure**

---

# Q12. Your organization wants to use its own encryption key for Azure managed disks. What should you use?

A. NSG
B. Azure DNS
C. Key Vault + Disk Encryption Set
D. Azure Bastion

### Answer

**C. Key Vault + Disk Encryption Set**

---

# Q13. Which Azure service stores encryption keys?

A. Azure Key Vault
B. Azure Load Balancer
C. Azure Monitor
D. Azure DNS

### Answer

**A. Azure Key Vault**

---

# Q14. Which Key Vault object is appropriate for storing a database password?

A. Certificate
B. Secret
C. Disk
D. Snapshot

### Answer

**B. Secret**

---

# Q15. Which Key Vault object is used for cryptographic keys?

A. Key
B. Secret
C. VM
D. Disk

### Answer

**A. Key**

---

# Q16. Which Key Vault object can be used for HTTPS certificate management?

A. Secret
B. Certificate
C. Snapshot
D. Data disk

### Answer

**B. Certificate**

---

# Q17. What is the purpose of a Disk Encryption Set?

A. Create VM passwords
B. Connect managed disks to a customer-managed encryption key
C. Create public IPs
D. Configure DNS

### Answer

**B. Connect managed disks to a customer-managed encryption key**

---

# Q18. Where is the customer-managed encryption key typically stored?

A. VM RAM
B. Azure Key Vault
C. NSG
D. Public IP

### Answer

**B. Azure Key Vault**

---

# Q19. A VM application needs a secret from Key Vault. You don't want to store credentials in the application code. What should you use?

A. Managed Identity
B. Public IP
C. Temporary disk
D. NSG

### Answer

**A. Managed Identity**

---

# Q20. What is the recommended place for an application secret such as an API key?

A. Source code
B. README.md
C. Azure Key Vault
D. Docker image layer

### Answer

**C. Azure Key Vault**

---

# 12. Scenario-Based AZ-104 Questions

## Scenario 1 — Linux VM Login

Your company deploys 20 Linux VMs. Administrators need secure SSH access without sharing passwords.

### Question

What should you configure?

### Answer

Use **SSH key-based authentication**.

Each administrator should securely manage their private key while the corresponding public key is configured for VM access.

---

# Scenario 2 — Application Storage

A Linux VM runs a web application. The application generates 500 GB of persistent data.

### Question

Where should you store the data?

### Answer

Use an Azure **managed data disk**.

Do not use the temporary disk for important persistent data.

---

# Scenario 3 — Development Cost

A developer VM is used only during working hours.

### Question

What should you do outside working hours?

### Answer

**Deallocate the VM**.

This releases VM compute resources and can reduce compute charges.

Remember that associated resources such as disks can continue to incur charges.

---

# Scenario 4 — Before Major Change

An administrator wants to make a major change to a data disk and wants a point-in-time copy first.

### Question

What should they create?

### Answer

Create a **snapshot of the data disk** before making the change.

---

# Scenario 5 — Database Password

A web application needs a database password.

The developer currently has:

```text
DB_PASSWORD=password123
```

inside the application configuration.

### Question

What is a better Azure solution?

### Answer

Store the password as a **Key Vault secret** and allow the application to access it securely, preferably using **Managed Identity**.

---

# Scenario 6 — Customer-Controlled Encryption

A company requires:

> "Azure managed disks must use a customer-managed encryption key."

### Question

Which Azure services should you consider?

### Answer

```text
Azure Key Vault
       +
Customer-managed key
       +
Disk Encryption Set
       +
Managed Disk
```

---

# 13. Important Differences for AZ-104

## OS Disk vs Data Disk

```text
OS Disk
   ↓
Operating System

Data Disk
   ↓
Application Data
```

---

## Data Disk vs Temporary Disk

| Data Disk                     | Temporary Disk                           |
| ----------------------------- | ---------------------------------------- |
| Persistent                    | Temporary                                |
| Suitable for application data | Suitable for temporary data              |
| Managed disk                  | Local temporary storage                  |
| Data survives VM deallocation | Data should not be treated as persistent |

---

## Snapshot vs Disk

| Snapshot                                          | Disk                   |
| ------------------------------------------------- | ---------------------- |
| Point-in-time copy                                | Actual storage volume  |
| Used as a source for recovery/copy                | Attached to VM         |
| Not directly equivalent to a normal attached disk | Used by OS/application |

---

## Key Vault vs Disk Encryption Set

```text
Key Vault
    ↓
Stores customer-managed key

Disk Encryption Set
    ↓
Uses the key for supported Azure disk encryption scenarios
```

---

## Platform-Managed vs Customer-Managed Keys

| Platform-Managed        | Customer-Managed                           |
| ----------------------- | ------------------------------------------ |
| Azure manages keys      | Customer controls key lifecycle            |
| Simpler                 | More administration                        |
| Less configuration      | Greater control                            |
| Common default approach | Useful for compliance/control requirements |

---

# 14. Final Revision Diagram

```text
                       AZURE VM
                          |
             +------------+------------+
             |            |            |
          OS Disk      Data Disk   Temp Disk
             |            |
          Ubuntu       App Data
                          |
                     Snapshot
                          |
                          v
                    Point-in-time
                       copy


             CUSTOMER-MANAGED ENCRYPTION

                    Key Vault
                       |
                 Customer Key
                       |
                       v
              Disk Encryption Set
                       |
                       v
                  Managed Disk
                       |
                       v
                     VM


                 SECURE APPLICATION

                  Azure VM
                     |
              Managed Identity
                     |
                     v
                 Key Vault
                     |
          +----------+----------+
          |          |          |
       Secret       Key     Certificate
```

---

# 15. AZ-104 Quick Revision

Remember these points:

1. **SSH public key** → stored/configured on the Linux VM.
2. **SSH private key** → securely kept by the client/user.
3. **OS disk** → contains the operating system.
4. **Data disk** → persistent application/data storage.
5. **Temporary disk** → temporary data; don't depend on it for persistence.
6. **Managed disk** → Azure-managed VM storage.
7. **Snapshot** → point-in-time copy of a disk.
8. **Deallocate VM** → releases compute resources.
9. **Managed disks remain** after VM deallocation.
10. **Key Vault** → stores secrets, keys, and certificates.
11. **Managed Identity** → avoids storing credentials in application code.
12. **Server-Side Encryption** → protects managed disk data at the storage layer.
13. **Customer-managed key** → gives the organization greater key control.
14. **Disk Encryption Set** → associates supported Azure resources with a customer-managed key.
15. **Key Vault + DES + managed disk** → common architecture for customer-managed disk encryption.

---

# 16. Most Important Exam Questions to Practice

### Question 1

You need to connect securely to an Azure Linux VM without passwords.

**Answer:** SSH key pair.

### Question 2

You need persistent storage for application files.

**Answer:** Azure managed data disk.

### Question 3

You need a point-in-time copy of a managed disk.

**Answer:** Disk snapshot.

### Question 4

You want to reduce compute costs when a VM is not required.

**Answer:** Deallocate the VM.

### Question 5

You need to store an application database password securely.

**Answer:** Azure Key Vault Secret.

### Question 6

You don't want application credentials stored in code.

**Answer:** Use Managed Identity with Key Vault.

### Question 7

You need customer-controlled encryption for Azure managed disks.

**Answer:** Azure Key Vault + customer-managed key + Disk Encryption Set.

### Question 8

A VM is deallocated. What happens to its managed data disk?

**Answer:** The managed disk remains and its data is preserved.

### Question 9

Which Key Vault object is used for an encryption key?

**Answer:** Key.

### Question 10

Which Key Vault object is used for an API password?

**Answer:** Secret.

---

# One-Line Memory Trick

```text
SSH → Secure VM Login
OS Disk → Operating System
Data Disk → Persistent Data
Temp Disk → Temporary Data
Snapshot → Point-in-Time Copy
Deallocate → Release VM Compute
Key Vault → Secrets + Keys + Certificates
Managed Identity → Passwordless Azure Resource Access
DES → Customer-Managed Disk Encryption
```
