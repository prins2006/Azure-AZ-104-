# Azure Compute — Virtual Machine Scale Sets & Azure Web Apps

## Topics Covered

1. Azure Virtual Machine Scale Sets
2. Lab — Azure Virtual Machine Scale Sets
3. Lab — VM Scale Set Scale Out
4. Azure VM Scale Set — Understanding the Scaling Process
5. Lab — VM Scale Set Scale In
6. Lab — VM Scale Sets — Flexible Orchestration Mode
7. Section Quiz — Compute: Virtual Machines
8. Introduction to Azure Web Apps
9. Lab — Azure Web Apps

---

# 43. Azure Virtual Machine Scale Sets (VMSS)

## What is Azure Virtual Machine Scale Set?

Azure Virtual Machine Scale Sets (VMSS) allow you to create and manage a group of identical or similar Virtual Machines.

Instead of manually creating:

    VM1
    VM2
    VM3
    VM4
    VM5

you can create a VM Scale Set and Azure can manage the VM instances for you.

Example:

    VM Scale Set
          |
    -----------------
    |       |       |
   VM1     VM2     VM3

If traffic increases:

    VM1
    VM2
    VM3
    VM4
    VM5

Azure can automatically increase or decrease the number of VM instances.

---

# Why do we need VM Scale Sets?

Imagine you have an e-commerce website.

Normal traffic:

    10:00 AM
    1,000 users
    2 VMs are enough

During a sale:

    8:00 PM
    50,000 users
    2 VMs are not enough

You could manually create additional VMs, but this is slow and difficult.

VMSS can automatically scale:

    Low traffic
        ↓
      2 VMs

        ↓

    High traffic
        ↓
      5 VMs

        ↓

    Traffic decreases
        ↓
      2 VMs

This is called:

    Autoscaling

---

# Main Benefits of VMSS

## 1. Automatic scaling

VMSS can increase or decrease VM instances based on demand.

Example:

    CPU > 70%
        ↓
    Add VM

    CPU < 30%
        ↓
    Remove VM

---

## 2. High availability

You can distribute VM instances across Availability Zones.

Example:

    Zone 1       Zone 2       Zone 3
      |            |            |
     VM1          VM2          VM3

If one zone has a problem, instances in other zones can continue serving users.

---

## 3. Load balancing

VMSS can work with Azure Load Balancer or Application Gateway.

Example:

                 Users
                   |
                   ↓
             Load Balancer
                   |
        ---------------------
        |         |         |
       VM1       VM2       VM3

The load balancer distributes incoming traffic.

---

## 4. Centralized management

Instead of managing each VM independently, you manage the scale set.

For example:

    Update VM image
    Change VM size
    Configure autoscaling
    Manage instances

---

# Important VMSS Concepts

## Instance

An instance is an individual VM inside a scale set.

Example:

    VMSS
     |
     +--- Instance 1
     +--- Instance 2
     +--- Instance 3

If your scale set has:

    Capacity = 3

then normally you have 3 VM instances.

---

# Minimum, Default and Maximum Instances

Autoscaling commonly uses:

    Minimum
    Default
    Maximum

Example:

    Minimum = 2
    Default = 2
    Maximum = 10

Meaning:

    Never go below 2 VMs.

    Normally start with 2 VMs.

    Never go above 10 VMs.

---

# Example

Suppose:

    Minimum = 2
    Default = 3
    Maximum = 8

Traffic increases.

Azure may scale:

    3 → 4 → 5 → 6 → 7 → 8

But it will never go above:

    8

If traffic decreases:

    8 → 7 → 6 → 5 → 4 → 3 → 2

But it will never go below:

    2

---

# VMSS Use Case

Suppose you have an API application.

Normal traffic:

    2,000 requests/minute

You need:

    2 VMs

During a marketing campaign:

    20,000 requests/minute

You need:

    6 VMs

After the campaign:

    2,000 requests/minute

You only need:

    2 VMs

VMSS can automatically manage this.

---

# VMSS Architecture

Example:

                        Internet
                           |
                           ↓
                  Azure Load Balancer
                           |
             ---------------------------
             |            |            |
            VM1          VM2          VM3
             \            |            /
              \           |           /
                 VM Scale Set
                       |
                 Autoscale Rules
                       |
                Azure Monitor


---

# VMSS Scaling

There are two main scaling directions.

## Scale Out

Increase the number of VM instances.

Example:

    2 VMs → 5 VMs

Used when:

    Traffic increases
    CPU increases
    Requests increase

---

## Scale In

Decrease the number of VM instances.

Example:

    5 VMs → 2 VMs

Used when:

    Traffic decreases
    CPU decreases
    Demand decreases

---

# Scale Out vs Scale In

| Feature | Scale Out | Scale In |
|---|---|---|
| Meaning | Add VMs | Remove VMs |
| Example | 2 → 5 | 5 → 2 |
| Trigger | High demand | Low demand |
| Goal | Improve capacity | Reduce cost |
| Cost | Increases | Decreases |

---

# VMSS Scaling Based on CPU

Example:

    CPU > 70%
        ↓
    Add 1 VM

    CPU < 30%
        ↓
    Remove 1 VM

Suppose:

    Current instances = 2

CPU becomes:

    85%

Scale out:

    2 → 3

If CPU remains high:

    3 → 4

If traffic later decreases:

    CPU = 20%

Scale in:

    4 → 3

Then:

    3 → 2

---

# Important Point

Autoscaling does NOT mean:

    CPU high = instantly create unlimited VMs

Azure respects:

    Minimum capacity
    Maximum capacity
    Scaling rules
    Cooldown period
    Monitoring data

---

# VMSS Scaling Cooldown

Cooldown is a waiting period after a scaling operation.

Example:

    CPU > 70%
        ↓
    Add VM
        ↓
    Wait for cooldown
        ↓
    Check metrics again

Why?

Because creating a VM takes time.

Without cooldown:

    CPU high
       ↓
    Add VM
       ↓
    CPU still high
       ↓
    Add VM
       ↓
    CPU still high
       ↓
    Add VM

This could cause unnecessary scaling.

---

# VMSS Images

VMSS instances are created from a VM image/configuration.

Common options include:

    Ubuntu
    Windows Server
    Custom Images

Example:

    Ubuntu 24.04
          |
          ↓
       VMSS
       / | \
     VM1 VM2 VM3

---

# VMSS and Availability Zones

For higher availability:

    Zone 1 → VM1
    Zone 2 → VM2
    Zone 3 → VM3

If Zone 1 becomes unavailable:

    VM2 + VM3
    can continue operating.

---

# VMSS and Load Balancer

A common production architecture:

                         Users
                           |
                           ↓
                    Azure Load Balancer
                           |
             ---------------------------
             |            |            |
            VM1          VM2          VM3
             |            |            |
             -------- VMSS ------------
                           |
                      Application

---

# When should you use VMSS?

Use VMSS when:

- You need multiple VM instances.
- Application traffic changes.
- You need automatic scaling.
- You need high availability.
- You want to reduce manual VM management.
- You have stateless workloads.
- You need load-balanced application servers.

---

# When VMSS may not be ideal

If your application requires a single unique server with local state, VMSS may not be the best choice.

Example:

    Special database server
    Unique application server
    Server with important local-only data

For these workloads, other Azure services may be more appropriate.

---

# 44. Lab — Azure Virtual Machine Scale Sets

## Goal

Create a VM Scale Set and understand how multiple VM instances are managed.

Typical architecture:

                    VM Scale Set
                         |
          -----------------------------
          |             |             |
         VM1           VM2           VM3

---

# Step 1 — Open Azure Portal

Go to:

    Azure Portal
        ↓
    Virtual Machine Scale Sets
        ↓
    Create

---

# Step 2 — Select Subscription

Choose the Azure subscription.

Example:

    Subscription:
    Azure subscription 1

---

# Step 3 — Resource Group

Create or select a Resource Group.

Example:

    Resource Group:
    rg-vmss-demo

---

# Step 4 — Scale Set Name

Example:

    vmss-web-demo

The name should be unique where required.

---

# Step 5 — Region

Choose an Azure region.

Example:

    East US

or:

    Central India

The actual available VM sizes and features can vary by region/subscription.

---

# Step 6 — Orchestration Mode

You may see:

    Flexible

or:

    Uniform

This is an important AZ-104 concept.

---

# Uniform Orchestration

Uniform orchestration is designed around a group of similar VM instances.

Example:

    VMSS
     |
    ----------------
    |      |       |
   VM1    VM2     VM3

The instances are generally based on the same configuration.

---

# Flexible Orchestration

Flexible orchestration provides more flexibility in managing VM instances.

It supports scenarios where you need more control over individual VMs.

Example:

    VMSS
     |
    ------------------------
    |          |           |
   VM1        VM2         VM3
    |          |           |
 different   different   different
 configuration as needed

Flexible mode is useful when workloads are not perfectly identical.

---

# Step 7 — Select VM Image

Example:

    Ubuntu Server

or:

    Windows Server

For Linux labs:

    Ubuntu

is commonly used.

---

# Step 8 — VM Size

Example:

    Standard_B2s

The available sizes depend on:

    Region
    Subscription quota
    Availability

---

# Step 9 — Instance Count

Example:

    2

Azure creates:

    VM1
    VM2

---

# Step 10 — Networking

A VMSS can be placed inside:

    Virtual Network
        |
        +--- Subnet

Example:

    VNet:
    vnet-demo

    Subnet:
    subnet-web

---

# Step 11 — Review + Create

Azure validates the configuration.

Then:

    Create

Azure creates:

    VM Scale Set
        |
        +--- Instance 1
        +--- Instance 2

---

# Verify Instances

Go to:

    Virtual Machine Scale Set
        ↓
    Instances

You should see individual VM instances.

Example:

    Instance 0
    Instance 1

---

# Important Lab Learning

You should understand the relationship:

    VM Scale Set
          |
          +--- VM Instance
          +--- VM Instance
          +--- VM Instance

The scale set is the management layer.

The instances are the actual VMs.

---

# Azure CLI Example

Create a resource group:

```bash
az group create \
  --name rg-vmss-demo \
  --location eastus