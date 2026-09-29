# AZ-104 — Azure RBAC, Custom Roles & Microsoft Entra ID Roles

## Topics Covered

1. Lab — Role assignments for Azure virtual machines — Preparation
2. Lab — Role assignments for Azure virtual machines — Implementation
3. Custom Roles
4. Microsoft Entra ID Roles
5. Lab — Microsoft Entra ID Roles
6. Microsoft Entra ID Custom Roles

---

# 1. Lab — Role Assignments for Azure Virtual Machines — Preparation

## 1.1 What is Azure RBAC?

**Azure Role-Based Access Control (Azure RBAC)** is the authorization system used to control who can access Azure resources and what actions they can perform.

Simple structure:

```text
User / Group / Service Principal
              |
              v
        Role Assignment
              |
              v
        Role Definition
              |
              v
            Scope
              |
              v
       Azure Resource
```

Example:

```text
Prins
  |
  | Virtual Machine Contributor
  |
  v
Resource Group: Production
  |
  +-- VM-01
  +-- VM-02
```

Prins can manage the virtual machines within the assigned scope.

---

# 1.2 Important Azure RBAC Concepts

There are four important concepts:

| Concept            | Meaning                                |
| ------------------ | -------------------------------------- |
| Security principal | Who receives permission                |
| Role definition    | What actions are allowed               |
| Scope              | Where the permission applies           |
| Role assignment    | Connects the principal, role and scope |

---

# 1.3 Security Principal

A security principal can be:

```text
User
Group
Service Principal
Managed Identity
```

Example:

```text
User: prins@example.com
```

can receive:

```text
Virtual Machine Contributor
```

---

# 1.4 Role Definition

A role definition describes what actions are allowed or denied.

Examples:

```text
Owner
Contributor
Reader
Virtual Machine Contributor
Network Contributor
Storage Account Contributor
```

---

# 1.5 Scope

Scope determines **where** the permission works.

Azure RBAC hierarchy:

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

Example:

```text
Subscription
│
├── Resource Group A
│   ├── VM-01
│   └── VM-02
│
└── Resource Group B
    ├── VM-03
    └── VM-04
```

If a user receives:

```text
Virtual Machine Contributor
Scope = Resource Group A
```

the role applies to VMs in that resource group, rather than automatically applying to Resource Group B.

---

# 1.6 Role Assignment

A role assignment connects:

```text
Who
+
What
+
Where
```

For example:

```text
Who:
Prins

What:
Virtual Machine Contributor

Where:
Resource Group A
```

Therefore:

```text
Prins
  |
  +-- Virtual Machine Contributor
          |
          +-- Resource Group A
```

---

# 1.7 Built-in Roles

Azure provides many built-in roles.

### Owner

Can manage resources and manage Azure RBAC access.

```text
Owner
  |
  +-- Manage resources
  +-- Assign Azure RBAC roles
```

### Contributor

Can manage Azure resources but does not have the same ability to manage Azure RBAC role assignments.

```text
Contributor
  |
  +-- Create
  +-- Modify
  +-- Delete
```

### Reader

Can view resources.

```text
Reader
  |
  +-- View
```

but generally cannot:

```text
Create
Modify
Delete
```

---

# 1.8 Virtual Machine Contributor

The **Virtual Machine Contributor** role is designed for managing virtual machines.

It allows management of VMs without granting the broad permissions of Owner.

This is an example of **least privilege**.

Instead of:

```text
Owner
```

give:

```text
Virtual Machine Contributor
```

when the user only needs to manage VMs.

---

# 1.9 Why Use RBAC?

Suppose a company has:

```text
Developer
Cloud Engineer
Security Team
Finance Team
```

They should not all have the same permissions.

Example:

```text
Developer
   ↓
Contributor on Development RG

Cloud Engineer
   ↓
Owner on Production subscription

Finance
   ↓
Reader / cost-related access
```

This follows the principle of:

> **Least privilege**

Give users only the permissions required for their job.

---

# 2. Lab — Role Assignments for Azure Virtual Machines — Implementation

## 2.1 Goal

In this lab, you practice assigning an Azure role to a user for virtual machine management.

Example requirement:

> Give a user permission to manage virtual machines in a specific resource group.

---

# 2.2 Azure Portal Method

Go to:

```text
Azure Portal
    |
    v
Resource Groups
    |
    v
Select Resource Group
    |
    v
Access control (IAM)
```

Then:

```text
Add
  |
  v
Add role assignment
```

Select a role such as:

```text
Virtual Machine Contributor
```

Then select:

```text
User, group, or service principal
```

Select the user.

Finally:

```text
Review + assign
```

---

# 2.3 Understand What Happened

Suppose:

```text
User = Prins

Role = Virtual Machine Contributor

Scope = RG-Production
```

The resulting assignment is:

```text
Prins
  |
  +-- Virtual Machine Contributor
          |
          +-- RG-Production
```

This means the user's permissions apply within that scope.

---

# 2.4 Azure CLI

First check the current subscription:

```bash
az account show
```

List subscriptions:

```bash
az account list -o table
```

Set the required subscription:

```bash
az account set --subscription "<subscription-name-or-id>"
```

---

# 2.5 Find the Resource Group

```bash
az group list -o table
```

Example:

```text
Name            Location
--------------  --------
RG-Production   eastus
RG-Development  eastus
```

---

# 2.6 Find the User

For Entra users:

```bash
az ad user list -o table
```

You can search for a specific user:

```bash
az ad user show --id user@example.com
```

---

# 2.7 Assign Virtual Machine Contributor

Example:

```bash
az role assignment create \
  --assignee user@example.com \
  --role "Virtual Machine Contributor" \
  --scope "/subscriptions/<SUBSCRIPTION-ID>/resourceGroups/RG-Production"
```

The important parameters are:

```text
--assignee
```

Who receives the permission.

```text
--role
```

Which role is assigned.

```text
--scope
```

Where the permission applies.

---

# 2.8 Check Role Assignments

```bash
az role assignment list \
  --assignee user@example.com \
  -o table
```

You can also specify a scope:

```bash
az role assignment list \
  --assignee user@example.com \
  --scope "/subscriptions/<SUBSCRIPTION-ID>/resourceGroups/RG-Production" \
  -o table
```

---

# 2.9 Remove a Role Assignment

```bash
az role assignment delete \
  --assignee user@example.com \
  --role "Virtual Machine Contributor" \
  --scope "/subscriptions/<SUBSCRIPTION-ID>/resourceGroups/RG-Production"
```

---

# 2.10 Important Practical Scenario

### Requirement

A developer needs to:

```text
Start VM
Stop VM
Restart VM
Modify VM configuration
```

But does not need to:

```text
Assign Azure RBAC roles
```

A role such as:

```text
Virtual Machine Contributor
```

may be more appropriate than:

```text
Owner
```

This demonstrates least privilege.

---

# 3. Custom Roles

## 3.1 What is a Custom Role?

Azure provides many built-in roles.

Sometimes none of them exactly match your organization's requirements.

Then you can create a:

> **Custom Azure RBAC role**

A custom role allows you to define exactly which Azure actions are allowed.

---

# 3.2 Why Create a Custom Role?

Suppose an employee needs:

```text
Start VM
Stop VM
Restart VM
```

but should NOT be able to:

```text
Delete VM
Create VM
Assign RBAC roles
```

A broad built-in role might provide more permissions than required.

A custom role can be designed around the required operations.

---

# 3.3 Custom Role Structure

A custom role definition can contain:

```text
Name
Description
Actions
NotActions
DataActions
NotDataActions
AssignableScopes
```

Example structure:

```json
{
  "Name": "Custom VM Operator",
  "Description": "Can manage selected VM operations",
  "Actions": [
    "Microsoft.Compute/virtualMachines/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Compute/virtualMachines/deallocate/action"
  ],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": [
    "/subscriptions/<SUBSCRIPTION-ID>"
  ]
}
```

---

# 3.4 Actions

`Actions` define management-plane operations.

Examples:

```text
Microsoft.Compute/virtualMachines/read
```

```text
Microsoft.Compute/virtualMachines/start/action
```

```text
Microsoft.Compute/virtualMachines/restart/action
```

---

# 3.5 NotActions

`NotActions` can exclude management operations from a broader action set.

Conceptually:

```text
Actions
  |
  +-- Allow
  |
  +-- NotActions
       |
       +-- Exclude
```

Important:

> `NotActions` does not represent an explicit deny system. It removes actions from the role's allowed action set.

---

# 3.6 DataActions

`DataActions` are used for operations against data within certain Azure resources.

This is different from management-plane operations.

Think:

```text
Actions
   ↓
Manage the Azure resource

DataActions
   ↓
Access data inside the resource
```

Example concept:

```text
Storage Account
     |
     +-- Management
     |
     +-- Blob data
```

---

# 3.7 AssignableScopes

`AssignableScopes` defines where the custom role can be assigned.

Example:

```json
"AssignableScopes": [
  "/subscriptions/<SUBSCRIPTION-ID>"
]
```

This means the custom role can be assigned within that subscription scope.

---

# 3.8 Create Custom Role Using JSON

Save a role definition:

```text
custom-vm-role.json
```

Example:

```json
{
  "Name": "Custom VM Operator",
  "Description": "Can perform selected VM operations",
  "Actions": [
    "Microsoft.Compute/virtualMachines/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Compute/virtualMachines/deallocate/action"
  ],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": [
    "/subscriptions/<SUBSCRIPTION-ID>"
  ]
}
```

Create the role:

```bash
az role definition create \
  --role-definition custom-vm-role.json
```

---

# 3.9 List Custom Roles

```bash
az role definition list \
  --custom-role-only true \
  -o table
```

---

# 3.10 Show a Role

```bash
az role definition list \
  --name "Custom VM Operator"
```

---

# 3.11 Delete a Custom Role

```bash
az role definition delete \
  --name "Custom VM Operator"
```

Be careful when deleting roles that are already being used.

---

# 4. Microsoft Entra ID Roles

## 4.1 What is Microsoft Entra ID?

Microsoft Entra ID is Microsoft's cloud identity and access management service.

It manages:

```text
Users
Groups
Applications
Identities
Authentication
Directory
```

---

# 4.2 Microsoft Entra Roles vs Azure RBAC

This distinction is extremely important.

### Azure RBAC

Controls:

```text
Azure resources
```

Examples:

```text
VM
Storage
VNet
Database
Resource Group
Subscription
```

### Microsoft Entra roles

Control:

```text
Identity and directory administration
```

Examples:

```text
Users
Groups
Applications
Tenant
Directory
```

---

# 4.3 Global Administrator

Global Administrator is a highly privileged Microsoft Entra role.

It provides broad administrative capabilities across Microsoft Entra ID and related Microsoft services.

Think:

```text
Global Administrator
       |
       +-- Users
       +-- Groups
       +-- Applications
       +-- Directory
       +-- Tenant administration
```

---

# 4.4 User Administrator

User Administrator focuses on user administration.

Examples include managing many aspects of:

```text
Users
```

It is more limited than Global Administrator.

---

# 4.5 Groups Administrator

Groups Administrator is focused on managing groups.

Think:

```text
Groups Administrator
        |
        +-- Groups
```

---

# 4.6 Application Administrator

Application Administrator is focused on managing applications within Microsoft Entra ID.

Examples:

```text
Enterprise applications
Application registrations
```

The exact permissions should always be checked against the current Microsoft Entra role definition because Microsoft can change role capabilities.

---

# 4.7 Security Administrator

Security Administrator is focused on security-related administration within the Microsoft identity/security ecosystem.

The important exam concept is:

> Microsoft Entra roles are different from Azure RBAC roles.

---

# 4.8 Azure Owner vs Global Administrator

| Feature                 | Azure Owner       | Global Administrator               |
| ----------------------- | ----------------- | ---------------------------------- |
| Permission system       | Azure RBAC        | Microsoft Entra roles              |
| Main purpose            | Azure resources   | Identity/tenant                    |
| VM management           | Yes, within scope | Not automatically                  |
| Storage management      | Yes, within scope | Not automatically                  |
| VNet management         | Yes, within scope | Not automatically                  |
| Assign Azure RBAC roles | Yes               | Not simply because of Global Admin |
| Manage users            | Not automatically | Yes                                |
| Manage groups           | Not automatically | Yes                                |
| Manage applications     | Not automatically | Yes                                |
| Tenant administration   | No                | Yes                                |

Memory trick:

```text
OWNER
  ↓
Azure Resources

GLOBAL ADMINISTRATOR
  ↓
Identity / Tenant
```

---

# 5. Lab — Microsoft Entra ID Roles

## 5.1 Goal

The purpose of this lab is to understand how to assign and manage Microsoft Entra administrative roles.

---

# 5.2 Azure Portal

Open:

```text
Azure Portal
```

Then:

```text
Microsoft Entra ID
      |
      v
Roles and administrators
```

You will see Microsoft Entra roles such as:

```text
Global Administrator
User Administrator
Groups Administrator
Application Administrator
Security Administrator
```

---

# 5.3 Assign a Microsoft Entra Role

Select a role.

Example:

```text
User Administrator
```

Then:

```text
Add assignments
```

Select the required user.

Then:

```text
Review + assign
```

---

# 5.4 Important Difference

When you assign:

```text
User Administrator
```

you are assigning a:

> **Microsoft Entra role**

When you assign:

```text
Virtual Machine Contributor
```

you are assigning an:

> **Azure RBAC role**

These are different systems.

---

# 5.5 Azure CLI

List Microsoft Entra users:

```bash
az ad user list -o table
```

List Microsoft Entra groups:

```bash
az ad group list -o table
```

For Microsoft Entra directory role management, Azure CLI commands and Microsoft Graph interfaces may be used depending on the operation and current tooling.

Always distinguish these from:

```bash
az role assignment
```

which is for Azure RBAC.

---

# 5.6 Azure RBAC vs Entra Role Assignment

### Azure RBAC

Example:

```bash
az role assignment create \
  --assignee user@example.com \
  --role "Virtual Machine Contributor" \
  --scope "/subscriptions/<SUBSCRIPTION-ID>/resourceGroups/RG-Production"
```

### Microsoft Entra role

The role is assigned within:

```text
Microsoft Entra ID
    |
    v
Roles and administrators
```

Do not confuse the two.

---

# 6. Microsoft Entra ID Custom Roles

## 6.1 What is a Microsoft Entra Custom Role?

Microsoft Entra ID also supports **custom directory roles**.

They allow organizations to create a role with a specific set of directory permissions rather than giving a user a broad built-in role.

Conceptually:

```text
Built-in Entra Role
       |
       +-- Predefined permissions

Custom Entra Role
       |
       +-- Organization-defined permissions
```

---

# 6.2 Why Use a Custom Entra Role?

Suppose an administrator needs a very specific task:

```text
Manage selected directory settings
```

but should not receive:

```text
Global Administrator
```

A custom role can be considered if the required permissions are supported.

This is another example of:

> **Least privilege**

---

# 6.3 Azure RBAC Custom Role vs Entra Custom Role

This is extremely important.

### Azure RBAC Custom Role

Used for:

```text
Azure resources
```

Example:

```text
VM
Storage
Network
Database
```

### Microsoft Entra Custom Role

Used for:

```text
Microsoft Entra directory operations
```

Example:

```text
Users
Groups
Directory operations
```

Therefore:

```text
Azure Custom Role
        ↓
Azure Resource Permissions

Entra Custom Role
        ↓
Directory Permissions
```

---

# 6.4 Comparison

| Feature             | Azure RBAC Custom Role          | Entra Custom Role         |
| ------------------- | ------------------------------- | ------------------------- |
| System              | Azure RBAC                      | Microsoft Entra ID        |
| Main purpose        | Azure resources                 | Directory/identity        |
| Scope               | Azure resource hierarchy        | Microsoft Entra directory |
| Used for VMs        | Yes                             | No                        |
| Used for storage    | Yes                             | No                        |
| Used for users      | Not as directory administration | Yes                       |
| Used for groups     | Not as directory administration | Yes                       |
| Defined permissions | Azure resource actions          | Directory permissions     |

---

# 6.5 Example Scenario

## Requirement

A company wants:

```text
Helpdesk Team
```

to manage certain user properties.

But they should not receive:

```text
Global Administrator
```

The organization can evaluate whether a:

```text
Custom Microsoft Entra role
```

can provide exactly the required permissions.

---

# 7. Important Difference: Azure RBAC Custom Role vs Entra Custom Role

Remember this diagram:

```text
                         Microsoft Cloud
                               |
               ┌───────────────┴───────────────┐
               |                               |
        Microsoft Entra ID                 Azure
               |                               |
               |                               |
       Entra Roles                       Azure RBAC
               |                               |
       Global Administrator                Owner
       User Administrator                 Contributor
       Groups Administrator               Reader
       Custom Entra Role                  Custom RBAC Role
               |                               |
          Identity                        Resources
               |                               |
       Users / Groups                  VM / Storage / VNet
       Applications                   Database / App Service
```

---

# 8. Common Exam Traps

## Trap 1

> Global Administrator = Azure Owner

**Incorrect.**

They are different permission systems.

---

## Trap 2

> Owner can automatically manage Microsoft Entra users.

**Incorrect.**

Azure Owner is an Azure RBAC role.

---

## Trap 3

> Reader can modify an Azure VM.

**Incorrect.**

Reader is primarily read/view access.

---

## Trap 4

> Contributor can assign Azure RBAC roles.

**Incorrect.**

Contributor can manage resources but does not have the same RBAC access-management capability as Owner.

---

## Trap 5

> A custom Azure RBAC role is used to manage Microsoft Entra users.

**Incorrect.**

Azure RBAC custom roles are for Azure resource permissions.

Microsoft Entra custom roles are for directory permissions.

---

# 9. Scenario-Based Questions

## Question 1

A user needs to manage Azure virtual machines but does not need to manage Azure RBAC permissions.

Which role type should you consider?

### Answer

A VM-specific role such as:

```text
Virtual Machine Contributor
```

rather than broad:

```text
Owner
```

---

# Question 2

A user needs to view Azure resources but cannot modify them.

Which role?

### Answer

```text
Reader
```

---

# Question 3

A user needs to manage Azure resources and assign Azure RBAC roles.

Which built-in role?

### Answer

```text
Owner
```

---

# Question 4

An administrator needs broad Microsoft Entra directory administration.

Which role?

### Answer

```text
Global Administrator
```

---

# Question 5

An administrator needs to manage users but does not need the full privileges of Global Administrator.

Which type of role should be considered?

### Answer

A more focused Microsoft Entra role such as:

```text
User Administrator
```

---

# Question 6

A company wants a special Azure permission set that does not exist as a built-in Azure role.

What should they consider?

### Answer

```text
Azure RBAC Custom Role
```

---

# Question 7

A company wants a special Microsoft Entra directory permission set.

What should they consider?

### Answer

```text
Microsoft Entra Custom Role
```

---

# Question 8

A user is Owner of a resource group but cannot create a Microsoft Entra user.

Why?

### Answer

Because:

```text
Owner
=
Azure RBAC
```

while user administration is controlled through:

```text
Microsoft Entra roles
```

---

# 10. Practical Lab Flow

A useful hands-on practice sequence is:

```text
Step 1
Create/select Azure subscription

        ↓

Step 2
Create Resource Group

        ↓

Step 3
Create Virtual Machine

        ↓

Step 4
Open Access Control (IAM)

        ↓

Step 5
Assign Virtual Machine Contributor

        ↓

Step 6
Verify role assignment

        ↓

Step 7
Create/test Custom Azure RBAC Role

        ↓

Step 8
Open Microsoft Entra ID

        ↓

Step 9
Open Roles and administrators

        ↓

Step 10
Assign an Entra role

        ↓

Step 11
Understand Entra Custom Roles
```

---

# 11. Important Commands for Revision

## Subscription

```bash
az account show
```

```bash
az account list -o table
```

```bash
az account set --subscription "<subscription-id>"
```

---

## Resource Groups

```bash
az group list -o table
```

---

## Azure RBAC assignments

```bash
az role assignment list -o table
```

```bash
az role assignment list \
  --assignee user@example.com \
  -o table
```

---

## Assign Azure RBAC role

```bash
az role assignment create \
  --assignee user@example.com \
  --role "Virtual Machine Contributor" \
  --scope "/subscriptions/<SUBSCRIPTION-ID>/resourceGroups/RG-Production"
```

---

## Delete Azure RBAC assignment

```bash
az role assignment delete \
  --assignee user@example.com \
  --role "Virtual Machine Contributor" \
  --scope "/subscriptions/<SUBSCRIPTION-ID>/resourceGroups/RG-Production"
```

---

## Custom Azure RBAC roles

List custom roles:

```bash
az role definition list \
  --custom-role-only true \
  -o table
```

Create:

```bash
az role definition create \
  --role-definition custom-role.json
```

Delete:

```bash
az role definition delete \
  --name "Custom VM Operator"
```

---

# 12. Exam Revision — One Page

## Azure RBAC

```text
WHO?
 ↓
User / Group / Service Principal / Managed Identity

WHAT?
 ↓
Role Definition

WHERE?
 ↓
Scope

RESULT
 ↓
Role Assignment
```

---

## Scope hierarchy

```text
Management Group
       ↓
Subscription
       ↓
Resource Group
       ↓
Resource
```

---

## Common Azure roles

```text
Owner
 ↓
Resources + RBAC access management

Contributor
 ↓
Resources

Reader
 ↓
View resources
```

---

## Microsoft Entra roles

```text
Global Administrator
 ↓
Broad tenant administration

User Administrator
 ↓
Users

Groups Administrator
 ↓
Groups

Application Administrator
 ↓
Applications
```

---

## Custom roles

```text
Azure RBAC Custom Role
 ↓
Azure resource permissions

Microsoft Entra Custom Role
 ↓
Directory permissions
```

---

# 13. Most Important AZ-104 Concepts

Make sure you can explain these without memorizing:

### 1. What is Azure RBAC?

Authorization system for Azure resources.

### 2. What is a role assignment?

The relationship between:

```text
Security Principal
+
Role
+
Scope
```

### 3. What is scope?

The level where a permission applies.

```text
Management Group
Subscription
Resource Group
Resource
```

### 4. Owner vs Contributor?

```text
Owner
=
Contributor-like resource management
+
RBAC access management
```

### 5. Owner vs Global Administrator?

```text
Owner
=
Azure resources

Global Administrator
=
Microsoft Entra identity/tenant
```

### 6. What is a custom Azure role?

An organization-defined Azure RBAC role containing selected permissions.

### 7. What is a Microsoft Entra custom role?

An organization-defined Microsoft Entra directory role containing selected directory permissions.

---

# 14. Final Memory Map

```text
                         AZURE ACCESS CONTROL
                                |
                ┌───────────────┴───────────────┐
                |                               |
          AZURE RBAC                       ENTRA ROLES
                |                               |
         Azure Resources                  Identity/Tenant
                |                               |
       ┌────────┼────────┐              ┌───────┼────────┐
       |        |        |              |       |        |
     Owner  Contributor Reader       Global   User    Groups
                                     Admin    Admin   Admin
       |
       └── Custom RBAC Role

                                           |
                                           └── Custom Entra Role
```

---

# 15. Final Rule to Remember

> **If the question is about an Azure resource, think Azure RBAC.**

Examples:

```text
VM
Storage
VNet
Database
Resource Group
Subscription
```

> **If the question is about identity or the Microsoft Entra directory, think Microsoft Entra roles.**

Examples:

```text
User
Group
Application
Directory
Tenant
Identity administration
```

And finally:

```text
OWNER
→ Azure resource administration

GLOBAL ADMINISTRATOR
→ Microsoft Entra / identity administration

CUSTOM RBAC ROLE
→ Custom Azure resource permissions

CUSTOM ENTRA ROLE
→ Custom directory permissions
```

This distinction is one of the most important concepts to understand for **AZ-104 Azure Administrator** questions.
