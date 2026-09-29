# Azure Owner vs Global Administrator

## 1. Overview

Azure has different permission systems for different types of administration.

The two roles that are commonly confused are:

* **Owner** — Azure RBAC role
* **Global Administrator** — Microsoft Entra ID role

The easiest way to remember them:

> **Owner = Azure resources**
> **Global Administrator = Microsoft Entra ID / identity / tenant**

These roles are **not the same**.

---

# 2. Azure Owner

## What is Azure Owner?

**Owner** is an **Azure RBAC (Role-Based Access Control)** role.

It is used to control access to Azure resources.

An Owner can:

* Manage Azure resources
* Create resources
* Modify resources
* Delete resources
* Assign Azure RBAC roles
* Manage access to Azure resources

### Simple meaning

> An Azure Owner has full management access to Azure resources within the scope where the Owner role is assigned.

---

# 3. What can an Owner manage?

Suppose we have:

```text
Azure Subscription
│
├── Resource Group: Production
│   │
│   ├── Virtual Machine
│   ├── Storage Account
│   ├── Virtual Network
│   └── Database
│
└── Resource Group: Development
    │
    ├── Virtual Machine
    └── Storage Account
```

If a user has **Owner** at the subscription level, they can manage resources throughout that subscription.

For example:

### Virtual Machines

Owner can:

```text
Create VM
Delete VM
Start VM
Stop VM
Resize VM
Change configuration
```

### Storage

Owner can:

```text
Create storage account
Delete storage account
Change configuration
Manage access
```

### Networking

Owner can:

```text
Create VNet
Create subnet
Modify NSG
Create public IP
Configure network resources
```

### RBAC

Owner can:

```text
Assign Owner
Assign Contributor
Assign Reader
Assign other Azure RBAC roles
```

---

# 4. Important Owner Concept — Scope

Azure RBAC permissions depend on **scope**.

Common scopes are:

```text
Management Group
       ↓
Subscription
       ↓
Resource Group
       ↓
Resource
```

For example:

```text
Subscription
│
├── Resource Group A
│   ├── VM
│   └── Storage
│
└── Resource Group B
    ├── VM
    └── Database
```

If you are Owner at:

### Subscription level

You can manage resources under that subscription.

### Resource Group level

You can manage resources inside that resource group.

### Resource level

You can manage that specific resource.

---

# 5. Global Administrator

## What is Global Administrator?

**Global Administrator** is a **Microsoft Entra ID role**.

It is used primarily to manage:

* Microsoft Entra ID
* Users
* Groups
* Applications
* Directory settings
* Identity-related administration
* Microsoft Entra roles

### Simple meaning

> Global Administrator is a highly privileged Microsoft Entra ID administrator responsible for tenant-level identity administration.

---

# 6. What can a Global Administrator manage?

A Global Administrator can manage many Microsoft Entra ID functions.

For example:

## Users

```text
Create users
Delete users
Modify users
Manage user properties
```

## Groups

```text
Create groups
Delete groups
Manage group membership
```

## Applications

```text
App registrations
Enterprise applications
Application permissions
Application configuration
```

## Microsoft Entra roles

A Global Administrator can manage Microsoft Entra administrative roles and related directory administration.

Examples include:

```text
Global Administrator
User Administrator
Groups Administrator
Application Administrator
Security Administrator
```

---

# 7. Azure Owner vs Global Administrator

## Main difference

```text
Azure Owner
     ↓
Azure RBAC
     ↓
Azure Resources
```

Whereas:

```text
Global Administrator
     ↓
Microsoft Entra ID
     ↓
Identity / Tenant
```

---

# 8. Comparison Table

| Feature                  | Azure Owner                               | Global Administrator                    |
| ------------------------ | ----------------------------------------- | --------------------------------------- |
| Permission system        | Azure RBAC                                | Microsoft Entra ID                      |
| Main purpose             | Manage Azure resources                    | Manage identities and tenant            |
| Manage VM                | Yes                                       | Not automatically                       |
| Delete VM                | Yes                                       | Not automatically                       |
| Create Storage Account   | Yes                                       | Not automatically                       |
| Manage VNet              | Yes                                       | Not automatically                       |
| Assign Azure RBAC roles  | Yes                                       | Not simply because of Global Admin role |
| Create Entra users       | No, not automatically                     | Yes                                     |
| Manage Entra groups      | No, not automatically                     | Yes                                     |
| Manage app registrations | No, not automatically                     | Yes                                     |
| Manage tenant settings   | No                                        | Yes                                     |
| Manage Entra roles       | No                                        | Yes                                     |
| Scope                    | Management group/subscription/RG/resource | Microsoft Entra tenant                  |

---

# 9. Important: Owner does NOT mean Global Administrator

Suppose:

```text
Prins
   ↓
Owner
   ↓
Azure Subscription
```

This does **not automatically mean**:

```text
Prins
   ↓
Global Administrator
   ↓
Microsoft Entra ID
```

Being Azure Owner does not automatically give the user every Microsoft Entra administrative permission.

---

# 10. Important: Global Administrator does NOT simply mean Owner

Suppose:

```text
Rahul
   ↓
Global Administrator
   ↓
Microsoft Entra ID
```

This does not mean Rahul is automatically an Azure Owner in the normal RBAC sense for every Azure resource.

For Azure resource access, Azure RBAC permissions and scope still matter.

---

# 11. Why are they separate?

Microsoft separates:

## Identity administration

from:

## Resource administration

This separation allows organizations to divide responsibilities.

For example:

```text
Identity Team
     ↓
Microsoft Entra ID
     ↓
Users / Groups / Applications
```

and:

```text
Cloud Team
     ↓
Azure RBAC
     ↓
VM / Storage / Network / Database
```

This is useful for security and least-privilege administration.

---

# 12. Real-World Example

Imagine a company called:

```text
ABC Technologies
```

They have:

```text
Microsoft Entra Tenant
        │
        ├── Employees
        ├── Groups
        └── Applications
        │
        ↓
Azure Subscription
        │
        ├── Production VM
        ├── Storage
        ├── VNet
        └── Database
```

The company gives:

```text
Rahul
Global Administrator
```

Rahul manages:

```text
Users
Groups
Applications
Identity
Tenant settings
```

The company gives:

```text
Prins
Owner
```

Prins manages:

```text
VMs
Storage
Networks
Databases
Azure RBAC
```

Therefore:

```text
Rahul → Identity administration

Prins → Azure resource administration
```

---

# 13. Example Question

## Question

A user needs to create a virtual machine and delete an existing virtual machine.

Which permission system should be considered?

### Answer

**Azure RBAC**

An appropriate role could be:

```text
Owner
```

or, if access management is not required:

```text
Contributor
```

---

# 14. Example Question — Microsoft Entra

## Question

An administrator needs to create new users in Microsoft Entra ID.

Which type of role should be considered?

### Answer

A **Microsoft Entra ID administrative role**.

For example:

```text
User Administrator
```

A Global Administrator can also perform broad directory administration.

---

# 15. Example Question — Access to VM

## Scenario

You have:

```text
User: Alex

Role:
Global Administrator
```

Alex tries to manage an Azure VM.

Should you automatically assume Alex can manage the VM?

### Answer

**No.**

Global Administrator and Azure RBAC are different permission systems.

For Azure resource management, check:

```text
Azure RBAC
+
Role
+
Scope
```

For example:

```text
Owner
Contributor
Virtual Machine Contributor
Reader
```

---

# 16. Example — Owner at Resource Group Level

Suppose:

```text
Subscription
│
├── RG-Production
│   ├── VM-01
│   ├── VM-02
│   └── Storage-01
│
└── RG-Development
    ├── VM-03
    └── Storage-02
```

Prins is:

```text
Owner
Scope: RG-Production
```

Then Prins can manage resources in:

```text
RG-Production
```

But this assignment does not automatically make Prins Owner of:

```text
RG-Development
```

This demonstrates the importance of **scope**.

---

# 17. Azure RBAC Scope

Azure RBAC commonly follows this hierarchy:

```text
Management Group
        ↓
Subscription
        ↓
Resource Group
        ↓
Resource
```

Permissions inherited from a higher scope can apply to lower scopes.

Example:

```text
Owner
Scope = Subscription
        ↓
Resource Group
        ↓
VM
        ↓
Storage
```

An Owner assignment at the subscription level can therefore provide access to resources under that subscription.

---

# 18. Owner vs Contributor

This is another important AZ-104 comparison.

## Owner

Owner can:

```text
Manage resources
+
Manage Azure RBAC access
```

## Contributor

Contributor can:

```text
Manage resources
```

but does not have the same ability to manage Azure RBAC access.

### Memory trick

```text
Owner = Contributor + Access Management
```

---

# 19. Owner vs Reader

## Owner

```text
Read resources
Create resources
Modify resources
Delete resources
Manage RBAC
```

## Reader

```text
View resources
```

Reader cannot normally:

```text
Create
Modify
Delete
```

resources.

---

# 20. Global Administrator vs User Administrator

These are both Microsoft Entra roles.

## Global Administrator

Broad directory administration.

```text
Tenant
Users
Groups
Applications
Directory settings
Roles
```

## User Administrator

Focused mainly on user administration.

```text
Create users
Manage users
Delete users
Reset certain user credentials/passwords
```

The important concept is:

> Microsoft Entra roles control identity-related administration.

---

# 21. Two Permission Systems

This is one of the most important concepts to understand.

Azure has:

## System 1 — Azure RBAC

Used for:

```text
Azure resources
```

Examples:

```text
VM
Storage
VNet
Database
Key Vault
App Service
```

Common roles:

```text
Owner
Contributor
Reader
Virtual Machine Contributor
Storage Blob Data Contributor
```

---

## System 2 — Microsoft Entra Roles

Used for:

```text
Identity and tenant administration
```

Examples:

```text
Users
Groups
Applications
Directory
Identity settings
```

Common roles:

```text
Global Administrator
User Administrator
Groups Administrator
Application Administrator
Security Administrator
```

---

# 22. Easy Architecture Diagram

```text
                    Microsoft Cloud
                          │
             ┌────────────┴────────────┐
             │                         │
       Microsoft Entra ID          Azure Resources
             │                         │
             │                         │
       Users / Groups             VM / Storage
       Applications               VNet / Database
       Tenant                     App Service
             │                         │
             │                         │
      Global Administrator          Owner
      User Administrator            Contributor
      Groups Administrator          Reader
```

---

# 23. Think of It Like a Company

A simple analogy:

## Global Administrator

Think:

> **Head of Identity / Directory**

They manage:

```text
Who are the employees?
Who has an account?
What groups exist?
What applications are registered?
How is the organization's identity system configured?
```

## Azure Owner

Think:

> **Cloud Resource Administrator**

They manage:

```text
Which VM exists?
Which storage account exists?
Which network exists?
Who has access to Azure resources?
```

---

# 24. Another Easy Analogy

Imagine a university.

### Microsoft Entra ID

Contains:

```text
Students
Teachers
Staff
Groups
Applications
```

A Global Administrator manages this identity system.

### Azure

Contains:

```text
VMs
Servers
Databases
Storage
Networks
```

An Azure Owner manages these resources.

So:

```text
Global Administrator
        ↓
Who are you?

Azure Owner
        ↓
What Azure resources can you manage?
```

---

# 25. Global Administrator and Azure Resource Access

A particularly important concept is that Microsoft Entra administration and Azure resource authorization are separate.

If a Global Administrator needs Azure resource access, there are mechanisms available to **elevate access** to Azure resources.

Conceptually:

```text
Global Administrator
        ↓
Elevate access
        ↓
Azure RBAC access
        ↓
Azure resources
```

This is an administrative capability and should be used carefully because it can provide very broad Azure access.

---

# 26. Why Least Privilege Matters

You should not give everyone:

```text
Owner
```

or:

```text
Global Administrator
```

unless their job requires it.

Instead, assign the smallest permission required.

Example:

If a person only needs to view a VM:

```text
Reader
```

If they need to manage VMs but not assign RBAC:

```text
Virtual Machine Contributor
```

If they need complete resource and access management:

```text
Owner
```

For identity tasks, choose the appropriate Microsoft Entra role rather than automatically assigning Global Administrator.

---

# 27. Common AZ-104 Exam Confusion

## Question Type 1

> Which role allows a user to manage Azure resources and assign Azure RBAC roles?

### Think:

```text
Owner
```

---

## Question Type 2

> Which role is used for broad Microsoft Entra ID administration?

### Think:

```text
Global Administrator
```

---

## Question Type 3

> A user is Global Administrator but cannot manage an Azure VM. Why?

### Think:

```text
Global Administrator
≠
Azure RBAC Owner
```

Check Azure RBAC assignment and scope.

---

## Question Type 4

> A user is Owner of a resource group but cannot manage users in Microsoft Entra ID.

### Why?

Because:

```text
Owner
=
Azure Resource Management
```

It does not automatically provide Microsoft Entra administrative permissions.

---

## Question Type 5

> A user needs to create an Entra ID user.

Think:

```text
Microsoft Entra role
```

rather than:

```text
Azure RBAC role
```

---

# 28. Interview Answer

If an interviewer asks:

> "What is the difference between Azure Owner and Global Administrator?"

You can answer:

> **Azure Owner is an Azure RBAC role that provides full management of Azure resources within its assigned scope, including the ability to manage Azure RBAC access. Global Administrator is a Microsoft Entra ID role that provides broad administrative control over the organization's identity and tenant. They are separate permission systems, so being Owner does not automatically make someone Global Administrator, and being Global Administrator does not simply mean they are an Owner of every Azure resource.**

---

# 29. Very Short Interview Answer

> **Owner manages Azure resources, while Global Administrator manages Microsoft Entra identity and tenant administration. Owner belongs to Azure RBAC, whereas Global Administrator belongs to Microsoft Entra roles.**

---

# 30. Important Keywords to Remember

### Azure Owner

```text
Azure RBAC
Resources
Subscription
Resource Group
VM
Storage
Network
Database
Role Assignment
Access Management
```

### Global Administrator

```text
Microsoft Entra ID
Tenant
Users
Groups
Applications
Identity
Directory
Entra Roles
```

---

# 31. Quick Revision Table

| If the question mentions... | Think about...       |
| --------------------------- | -------------------- |
| VM                          | Azure RBAC           |
| Storage Account             | Azure RBAC           |
| Virtual Network             | Azure RBAC           |
| Resource Group              | Azure RBAC           |
| Subscription                | Azure RBAC           |
| Assign Azure RBAC role      | Owner                |
| Create Entra user           | Microsoft Entra role |
| Manage Entra groups         | Microsoft Entra role |
| Manage tenant               | Microsoft Entra      |
| App registration            | Microsoft Entra      |
| Global Administrator        | Microsoft Entra      |
| Owner                       | Azure RBAC           |

---

# 32. Final Memory Trick

Remember:

```text
OWNER
  ↓
Azure Resources
  ↓
VM / Storage / Network / Database
```

And:

```text
GLOBAL ADMINISTRATOR
  ↓
Microsoft Entra ID
  ↓
Users / Groups / Applications / Tenant
```

### One sentence

> **OWNER = "What can I manage in Azure?"**

> **GLOBAL ADMINISTRATOR = "What can I manage in the organization's identity/tenant?"**

---

# 33. AZ-104 Exam Focus

For AZ-104, make sure you understand these concepts:

1. Azure RBAC
2. Role assignments
3. Role definitions
4. Scope
5. Owner
6. Contributor
7. Reader
8. Resource-specific roles
9. Microsoft Entra ID
10. Microsoft Entra administrative roles
11. Global Administrator
12. User Administrator
13. Groups Administrator
14. Azure RBAC vs Microsoft Entra roles
15. Role inheritance
16. Least privilege
17. Privileged access and elevation concepts

The most important distinction is:

```text
Azure RBAC
    ↓
Azure Resource Access

Microsoft Entra Roles
    ↓
Identity / Tenant Administration
```

---

# 34. Final Summary

| Concept                    | Azure Owner       | Global Administrator              |
| -------------------------- | ----------------- | --------------------------------- |
| Type                       | Azure RBAC role   | Microsoft Entra role              |
| Main area                  | Azure resources   | Identity/tenant                   |
| VM management              | Yes, within scope | Not automatically                 |
| Storage management         | Yes, within scope | Not automatically                 |
| Network management         | Yes, within scope | Not automatically                 |
| Azure RBAC assignments     | Yes               | Not simply from Global Admin role |
| User management            | Not automatically | Yes                               |
| Group management           | Not automatically | Yes                               |
| Application administration | Not automatically | Yes                               |
| Tenant administration      | No                | Yes                               |
| Main exam keyword          | **Resources**     | **Identity**                      |

## Final Rule

```text
OWNER
→ Azure Resource Management

GLOBAL ADMINISTRATOR
→ Microsoft Entra / Identity Management
```

**Do not memorize only the role names. Understand which permission system each role belongs to.**

That distinction is extremely important for Azure administration and AZ-104 questions.
