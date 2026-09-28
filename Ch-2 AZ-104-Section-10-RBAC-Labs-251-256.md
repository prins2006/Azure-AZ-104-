# AZ-104 — Section 10: Azure RBAC Labs 251–256

## Topics Covered

| No. | Topic | Main Focus |
|---|---|---|
| 251 | Role-based assignments — Resource level | Assign RBAC at one resource |
| 252 | Role-based assignments — Resource group level | Assign RBAC to a resource group |
| 253 | Role-based assignments — Contributor Role | Understand Contributor permissions |
| 254 | Role-based assignments — User Access Administrator Role | Manage access / role assignments |
| 255 | Role assignments for Azure Storage Accounts | Apply RBAC to Storage |
| 256 | Role Assignment conditions | Fine-grained access with Azure ABAC |

> **AZ-104 memory rule:** Every RBAC question can be broken into **WHO + WHAT + WHERE**.
>
> **WHO** = security principal  
> **WHAT** = role  
> **WHERE** = scope

---

# 251. Role-Based Assignments — Resource Level

## What does resource-level assignment mean?

A role is assigned directly to **one Azure resource**.

```text
Subscription
    |
    +--- Resource Group
           |
           +--- VM-01
           +--- VM-02
```

If a user needs access only to `VM-01`, assign the role at the VM:

```text
User
  |
  +--- Contributor
          |
          +--- VM-01
```

The user does not automatically receive the same role on `VM-02`.

### Why use resource-level scope?

Use it when the user needs the **smallest possible scope**.

Example:

> Developer can manage only VM-01.

```text
WHO   = Developer
WHAT  = Required role
WHERE = VM-01
```

Microsoft recommends using the smallest scope that meets the requirement.

### Scope levels

```text
Management Group
       |
       v
Subscription
       |
       v
Resource Group
       |
       v
Resource
```

### AZ-104 Scenario

> A user must manage one VM but must not manage other VMs in the resource group. Where should you assign the role?

**Answer concept:** Resource-level RBAC.

---

# 252. Role-Based Assignments — Resource Group Level

A role can be assigned at the **resource group** scope.

```text
Production-RG
    |
    +--- VM
    +--- Storage Account
    +--- VNet
```

Example:

```text
User
  |
  +--- Contributor
         |
         +--- Production-RG
```

The assignment can apply to resources inside that resource group.

## Resource vs Resource Group

| Requirement | Scope |
|---|---|
| Manage one VM | Resource |
| Manage all resources in one RG | Resource Group |
| Manage resources across subscription | Subscription |
| Manage resources across multiple subscriptions | Management Group |

### Scenario

> A developer should manage all resources in `Development-RG`, but not resources in `Production-RG`.

Use:

```text
Developer
   |
   +--- Contributor
          |
          +--- Development-RG
```

Do not use subscription scope if resource-group scope is sufficient.

---

# 253. Role-Based Assignments — Contributor Role

## What is Contributor?

The built-in **Contributor** role allows a principal to manage Azure resources according to the permissions in the role.

Conceptually:

```text
Contributor
   |
   +--- Create resources
   +--- Modify resources
   +--- Delete resources
```

Important:

> **Contributor does not grant permission to assign Azure RBAC roles.**

```text
Contributor
    |
    +--- Manage resources       YES
    |
    +--- Assign RBAC roles      NO
```

## Contributor vs Owner

| Capability | Contributor | Owner |
|---|---:|---:|
| View resources | Yes | Yes |
| Create resources | Yes | Yes |
| Modify resources | Yes | Yes |
| Delete resources | Yes | Yes |
| Manage access / RBAC | No | Yes |

### Exam scenario

> A user must create, modify, and delete resources but must not grant permissions to other users.

**Concept:** Contributor.

> A user has Contributor access but cannot assign Reader to another user. Why?

Because Contributor manages resources but does not provide the required access-management permission.

---

# 254. Role-Based Assignments — User Access Administrator

## What is User Access Administrator?

**User Access Administrator** is a built-in Azure role focused on managing access to Azure resources.

Think:

```text
User Access Administrator
          |
          v
    Manage access
          |
          v
    Role assignments
```

## Contributor vs User Access Administrator

| Role | Main purpose |
|---|---|
| Contributor | Manage Azure resources |
| User Access Administrator | Manage access / role assignments |
| Owner | Manage resources + manage access |

### Important permission concept

To create a role assignment, the principal needs permission such as:

```text
Microsoft.Authorization/roleAssignments/write
```

### Scenario

> An administrator needs to assign Azure RBAC roles but does not need broad resource-management permissions.

**Think:** User Access Administrator.

---

# 255. Role Assignments for Azure Storage Accounts

Azure Storage access has an important distinction:

```text
Management plane
        vs
Data plane
```

## Management plane

Controls the Storage Account resource itself.

Examples:

```text
Create Storage Account
Delete Storage Account
Change configuration
Manage resource settings
```

## Data plane

Controls the actual data.

Examples:

```text
Read blobs
Write blobs
Delete blobs
Read queue messages
```

### Important Storage data roles

```text
Storage Blob Data Reader
Storage Blob Data Contributor
Storage Blob Data Owner
```

## Storage Blob Data Reader

Used when a user needs to read blob data.

```text
User
  |
  +--- Storage Blob Data Reader
          |
          +--- Read blob data
```

## Storage Blob Data Contributor

Used for broader blob-data access, including reading and writing according to the role definition.

## Storage Blob Data Owner

Provides broad blob-data management permissions defined by the role.

### Important exam concept

Do not automatically assume:

```text
Contributor
```

means:

```text
Can read/write blob contents
```

Azure resource-management roles and Storage data roles are different concepts.

### Scenario

> A user should read blob data but should not modify it.

Think:

```text
Storage Blob Data Reader
```

> A user needs to read and modify blob data.

Think:

```text
Storage Blob Data Contributor
```

---

# 256. Role Assignment Conditions

## What is a role assignment condition?

A role assignment condition is an **optional additional check** added to an Azure RBAC role assignment.

It provides more fine-grained access control.

Think:

```text
RBAC
  |
  +--- WHO
  +--- ROLE
  +--- SCOPE
  |
  +--- CONDITION
```

## Why use conditions?

Normal RBAC can sometimes be broader than required.

Example:

```text
User
  |
  +--- Storage Blob Data Reader
          |
          +--- Storage Account
```

Maybe the user should not read every blob.

A condition can narrow the access based on supported attributes.

Example:

```text
User
  |
  +--- Storage Blob Data Reader
          |
          +--- Condition:
                 Project = Development
```

Azure ABAC extends Azure RBAC with attribute-based conditions.

## Common condition attributes

Depending on the supported action, conditions can use attributes such as:

```text
Project
Container name
Blob path
Blob index tag
Private link
Subnet
Current UTC time
Principal attributes
Request attributes
Resource attributes
```

## Simple example

```text
Storage Account
    |
    +--- Container A
    +--- Container B
```

Assignment:

```text
User
  |
  +--- Storage Blob Data Reader
          |
          +--- Condition:
                 Container = A
```

The condition further filters the permissions granted by that assignment.

## Important: Condition is not a general "Deny"

A role assignment condition is an additional filter on the permissions granted by that assignment.

It is **not** a general explicit-deny mechanism.

Also remember that Azure RBAC is additive. If the same user has another unconditional role assignment that grants the required access, that other assignment can still allow access.

Example:

```text
Assignment 1:
Storage Blob Data Contributor
Scope = Storage Account
Condition = Project = A

Assignment 2:
Storage Blob Data Contributor
Scope = Subscription
No condition
```

Assignment 2 may provide broader access.

Therefore, when troubleshooting conditions, inspect:

```text
Direct assignments
+
Inherited assignments
+
Assignments at broader scopes
```

---

# Role Assignment Condition — Portal Flow

Conceptually:

```text
Resource
   |
   v
Access control (IAM)
   |
   v
Add role assignment
   |
   v
Select role
   |
   v
Select principal
   |
   v
Conditions (optional)
   |
   v
Review + assign
```

Conditions are available for supported storage data roles.

---

# ⭐ 251–256 Quick Comparison

| Topic | Main Concept |
|---|---|
| 251 | Assign role to one resource |
| 252 | Assign role to a resource group |
| 253 | Contributor manages resources but not RBAC access |
| 254 | User Access Administrator manages access / role assignments |
| 255 | Use Storage data roles for Storage data access |
| 256 | Conditions provide more fine-grained access |

---

# ⭐ WHO + WHAT + WHERE + CONDITION

For advanced RBAC questions:

```text
WHO?
User / Group / Service Principal / Managed Identity

WHAT?
Reader / Contributor / Owner / Storage data role

WHERE?
Management Group
Subscription
Resource Group
Resource

CONDITION?
Optional additional restriction
```

Example:

```text
WHO
Developer

WHAT
Storage Blob Data Reader

WHERE
Storage Account

CONDITION
Project = Development
```

---

# AZ-104 Exam Question Types

Microsoft does not publish the exact live exam questions. Prepare for scenario-based and interactive question formats.

## Type 1 — Choose the Scope

> A developer must manage all resources in `Dev-RG` but must not manage resources in `Prod-RG`.

Think:

```text
WHO   = Developer
WHAT  = Contributor
WHERE = Dev-RG
```

**Concept:** Resource Group scope.

---

## Type 2 — Choose the Role

> A user must manage Azure resources but must not assign permissions to other users.

**Concept:** Contributor.

---

## Type 3 — Access Management

> An administrator needs to assign Azure RBAC roles but does not need broad resource-management permissions.

**Concept:** User Access Administrator.

---

## Type 4 — Storage Data Access

> A user needs to read blobs but should not modify blob data.

**Concept:** Storage Blob Data Reader.

---

## Type 5 — Condition

> A user should read only blobs that have `Project=Development`.

**Concept:**

```text
Azure RBAC
+
Role assignment condition
+
Azure ABAC
```

---

## Type 6 — Inheritance

> A user has a role at subscription scope. What happens to resources below that subscription?

Think:

```text
Subscription
     |
     +--- Resource Group
            |
            +--- Resources
```

The assignment can apply to child scopes.

---

## Type 7 — Troubleshooting

> A user has a condition on a Storage Blob Data role but can still access data that appears outside the condition.

Investigate:

```text
Other role assignments
        +
Inherited assignments
        +
Broader scopes
```

---

# ⭐ Practice Questions

### Q1
A user should manage only one VM. Which RBAC scope should be considered?

### Q2
A developer should manage every resource in `Development-RG`. Which scope is appropriate?

### Q3
What is the main difference between Contributor and Owner?

### Q4
Can Contributor assign RBAC roles to other users?

### Q5
What is the main purpose of User Access Administrator?

### Q6
What is the difference between Azure resource-management permissions and Storage data permissions?

### Q7
Which role would you consider when a user only needs to read blob data?

### Q8
What is Azure ABAC?

### Q9
What is a role assignment condition?

### Q10
Can an Azure RBAC condition be used as a general explicit deny?

### Q11
A user has a conditional Storage Blob Data Reader assignment at a resource group but also has an unconditional assignment at subscription scope. What should you investigate?

### Q12
For an RBAC question, what three things should you identify first?

```text
WHO
WHAT
WHERE
```

---

# ⭐ Final Revision Diagram

```text
                    Azure RBAC
                        |
          +-------------+-------------+
          |             |             |
         WHO           WHAT          WHERE
          |             |             |
        User          Role          Scope
        Group         Reader        MG
        SP            Contributor   Subscription
        MI            Owner         Resource Group
                      Storage       Resource
                      roles
                        |
                        v
                   CONDITION
                        |
                        v
                     Azure ABAC
```

# ⭐ Most Important Things to Remember

1. **Resource level** → one specific resource.
2. **Resource group level** → resources inside one RG.
3. **Contributor** → manages resources, not RBAC role assignments.
4. **User Access Administrator** → manages access / role assignments.
5. **Storage data roles** → control access to Storage data.
6. **Role assignment conditions** → provide finer-grained restrictions.
7. **Conditions are not general explicit deny rules.**
8. **RBAC is additive** → always check other assignments and inherited access.
9. Use the **smallest scope** that satisfies the requirement.
10. For every scenario ask:

```text
WHO?
WHAT ROLE?
WHERE?
CONDITION?
```

## Official Microsoft Learn References

- AZ-104 Study Guide: https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-104
- Azure RBAC role assignments: https://learn.microsoft.com/en-us/azure/role-based-access-control/role-assignments
- Azure ABAC / role assignment conditions: https://learn.microsoft.com/en-us/azure/role-based-access-control/conditions-overview
- Add/edit role assignment conditions: https://learn.microsoft.com/en-us/azure/role-based-access-control/conditions-role-assignments-portal
