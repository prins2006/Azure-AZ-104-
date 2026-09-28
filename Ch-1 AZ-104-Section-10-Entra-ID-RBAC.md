# AZ-104 --- Section 10: Manage Azure Identities and Governance

## Topics Covered

  -----------------------------------------------------------------------
  No.                     Topic                   Main Focus
  ----------------------- ----------------------- -----------------------
  246                     What is Microsoft Entra Identity and
                          ID                      authentication

  247                     Creating a user in      Users
                          Microsoft Entra ID      

  248                     Logging in as the new   Authentication and
                          user                    access

  249                     Let's deploy some       Azure resources and
                          resources               permissions

  250                     Introduction to         Authorization / RBAC
                          Role-Based Access       
                          Control                 

  251                     Role-based assignments  RBAC scope
                          --- Resource level      
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 246. What is Microsoft Entra ID?

## Definition

**Microsoft Entra ID** is Microsoft's cloud-based identity and access
management service.

It helps answer:

-   Who are you?
-   How do you prove your identity?
-   What are you allowed to access?

### Basic flow

``` text
User
  |
  | Login
  v
Microsoft Entra ID
  |
  | Authentication
  v
Azure
  |
  | Authorization through RBAC
  v
Azure Resources
```

## Important Terms

  Term             Meaning
  ---------------- ---------------------------------------------
  Tenant           An organization's Microsoft Entra directory
  User             Identity representing a person
  Group            Collection of users
  Guest user       External user invited into the directory
  Authentication   Proving who you are
  Authorization    Determining what you can access
  RBAC             Controls access to Azure resources

## Authentication vs Authorization

### Authentication

Authentication means:

> **Who are you?**

Example:

``` text
Username + Password + MFA
             |
             v
     Authentication
        successful
```

### Authorization

Authorization means:

> **What are you allowed to do?**

Example:

``` text
User
 |
 +--- Reader role
       |
       +--- Can view resources
       +--- Cannot modify resources
```

### AZ-104 Scenario

**Question:**

A user can successfully sign in to Azure but cannot delete a virtual
machine. What is the likely issue?

**Concept:**

``` text
Authentication = Successful
Authorization  = Insufficient permissions
```

This points toward **Azure RBAC**.

------------------------------------------------------------------------

# 247. Lab --- Creating a User in Microsoft Entra ID

You should understand how to create and manage users.

Typical user information includes:

``` text
Name
Username
Password
Location
Job title
Department
Contact information
```

Example:

``` text
Name: Rahul Patel
Username: rahul@company.onmicrosoft.com
```

## Member vs Guest

### Member

Usually represents someone who belongs to the organization.

``` text
Company Employee
       |
       v
     Member
```

### Guest

Usually represents an external person.

``` text
External Consultant
       |
       v
      Guest
```

## Important User Management Concepts

Know these concepts:

-   Create users
-   Update user properties
-   Delete users
-   Restore recently deleted users
-   Add users to groups
-   Assign licenses
-   Manage external/guest users
-   Self-Service Password Reset (SSPR)

## AZ-104 Scenario

**Question:**

An external consultant needs access to your Azure environment. What type
of user should you create?

**Concept:**

``` text
External person
      |
      v
Guest user
```

------------------------------------------------------------------------

# 248. Lab --- Logging in as the New User

This topic is mainly about understanding:

``` text
Identity
Authentication
Authorization
```

Suppose you create:

``` text
User: testuser
```

The user can authenticate:

``` text
testuser
   |
   v
Microsoft Entra ID
   |
   v
Login successful
```

However:

> Successful login does NOT automatically give the user permission to
> manage every Azure resource.

Example:

``` text
Login
  |
  v
Successful
  |
  v
Azure Portal
  |
  v
Access Resource Group
  |
  v
Permission denied
```

Why?

Because the user may not have the required **Azure RBAC role**.

------------------------------------------------------------------------

# 249. Lab --- Let's Deploy Some Resources

This topic connects identity with Azure resources.

Typical Azure hierarchy:

``` text
Management Group
       |
       v
Subscription
       |
       v
Resource Group
       |
       +------ VM
       |
       +------ Storage Account
       |
       +------ Network
```

The important question is:

> Who can create or manage these resources?

This is an **authorization / RBAC** question.

## Example

Suppose:

``` text
User: Rahul
Role: Contributor
Scope: Dev-RG
```

Rahul can manage resources within the scope according to the permissions
included in the Contributor role.

If the role is:

``` text
Reader
```

the user can generally view resources but cannot perform write
operations.

------------------------------------------------------------------------

# 250. Introduction to Role-Based Access Control

## What is RBAC?

**RBAC = Role-Based Access Control**

Azure RBAC answers:

> **Who can do what on which Azure resource?**

The three most important components are:

``` text
Security Principal
        +
      Role
        +
      Scope
```

------------------------------------------------------------------------

## 1. Security Principal --- WHO?

Who receives the permission?

Examples:

``` text
User
Group
Service Principal
Managed Identity
```

Example:

``` text
Prins
  |
  v
Security Principal
```

------------------------------------------------------------------------

## 2. Role --- WHAT?

What actions can the security principal perform?

Examples:

``` text
Reader
Contributor
Owner
User Access Administrator
```

------------------------------------------------------------------------

## 3. Scope --- WHERE?

Where does the permission apply?

Azure RBAC has four important scopes:

``` text
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

### Important Rule

> Higher scope generally means broader access.

> Lower scope generally means narrower access.

------------------------------------------------------------------------

# Important Azure RBAC Roles

## Reader

Reader can generally:

``` text
View resources
```

But cannot generally:

``` text
Create
Modify
Delete
```

------------------------------------------------------------------------

## Contributor

Contributor can generally:

``` text
Create resources
Modify resources
Delete resources
```

But:

> Contributor cannot manage RBAC role assignments.

------------------------------------------------------------------------

## Owner

Owner can:

``` text
Manage resources
+
Manage access
```

### Easy memory

``` text
Contributor = Manage resources

Owner = Manage resources + Manage access
```

------------------------------------------------------------------------

## User Access Administrator

This role is primarily related to:

``` text
Managing user access
Managing role assignments
```

------------------------------------------------------------------------

# 251. Role-Based Assignments --- Resource Level

This topic is about assigning an Azure RBAC role at a specific resource.

Suppose:

``` text
VM1
```

You want Rahul to manage only this VM.

Instead of:

``` text
Subscription
    |
    +--- Contributor
```

you can assign the role at:

``` text
VM1
 |
 +--- Contributor
```

This creates a narrower permission scope.

------------------------------------------------------------------------

# Azure RBAC Scope

There are four important levels:

  -----------------------------------------------------------------------
  Scope                   Example                 General Access Area
  ----------------------- ----------------------- -----------------------
  Management Group        Company MG              Multiple subscriptions

  Subscription            Production Subscription Resources within
                                                  subscription

  Resource Group          Production-RG           Resources inside RG

  Resource                VM1                     One specific resource
  -----------------------------------------------------------------------

## Scope Example

``` text
Subscription
    |
    +--- Resource Group A
    |       |
    |       +--- VM1
    |       +--- Storage1
    |
    +--- Resource Group B
            |
            +--- VM2
```

If you assign:

``` text
Rahul
Contributor
Resource Group A
```

the role applies to resources under Resource Group A.

It does not automatically give Rahul Contributor access to Resource
Group B.

------------------------------------------------------------------------

# RBAC Inheritance

A role assignment at a parent scope can apply to child scopes.

Example:

``` text
Subscription
    |
    +--- Contributor
           |
           +--- Resource Group A
           |      |
           |      +--- VM1
           |      +--- Storage1
           |
           +--- Resource Group B
                  |
                  +--- VM2
```

If Rahul has Contributor at the **subscription level**, the assignment
can apply to resources below that subscription.

If Rahul has Contributor only at **Resource Group A**, it applies to
resources under Resource Group A, not Resource Group B.

------------------------------------------------------------------------

# Entra Roles vs Azure RBAC Roles

This is a very important distinction.

## Microsoft Entra Roles

Used primarily for managing:

``` text
Identity
Users
Groups
Directory
Entra services
```

Examples:

``` text
Global Administrator
User Administrator
Groups Administrator
```

## Azure RBAC Roles

Used for managing:

``` text
Azure Resources
Virtual Machines
Storage
Networks
Resource Groups
Subscriptions
```

Examples:

``` text
Reader
Contributor
Owner
Virtual Machine Contributor
Storage Blob Data Reader
```

### Easy memory

``` text
Entra Role
    |
    v
Identity / Directory administration


Azure RBAC
    |
    v
Azure Resource access
```

------------------------------------------------------------------------

# ⭐ The Most Important RBAC Framework

Whenever you see an RBAC question, immediately think:

``` text
WHO + WHAT + WHERE
```

## WHO?

``` text
User
Group
Service Principal
Managed Identity
```

## WHAT?

``` text
Reader
Contributor
Owner
Other Azure role
```

## WHERE?

``` text
Management Group
Subscription
Resource Group
Resource
```

Example:

``` text
WHO?
Developer

WHAT?
Contributor

WHERE?
Development Resource Group
```

Therefore:

``` text
Developer
   |
   +--- Contributor
          |
          +--- Development RG
```

------------------------------------------------------------------------

# AZ-104 Question Types You Should Prepare For

Microsoft does not publish the exact live exam questions. The exam can
use different interaction formats, including multiple-choice and
scenario-based interactions.

For these topics, prepare for the following question styles.

------------------------------------------------------------------------

## Type 1 --- Direct Knowledge

**Question:**

What service provides identity and access management for Azure?

**Think:**

``` text
Microsoft Entra ID
```

------------------------------------------------------------------------

## Type 2 --- Scenario-Based

**Question:**

A company wants to give an employee access to all resources in
`Production-RG`, but not resources in other resource groups. Which scope
should be used?

**Think:**

``` text
Production-RG
      |
      v
Resource Group scope
```

------------------------------------------------------------------------

## Type 3 --- Role Selection

**Question:**

A user needs to view Azure resources but must not modify them. Which
role should be assigned?

**Think:**

``` text
Reader
```

------------------------------------------------------------------------

## Type 4 --- Role Comparison

**Question:**

Which role allows a user to manage Azure resources and manage access?

**Think:**

``` text
Owner
```

------------------------------------------------------------------------

## Type 5 --- Inheritance

**Question:**

A user has Contributor access at the subscription level. What resources
can the user manage?

Understand:

``` text
Subscription
     |
     +--- Resource Group
             |
             +--- Resource
```

The role can apply to child scopes.

------------------------------------------------------------------------

## Type 6 --- Drag and Drop

You may need to match concepts with definitions.

Example:

``` text
User
Group
Reader
Contributor
Resource Group
```

Know what each one represents.

------------------------------------------------------------------------

## Type 7 --- Build List

You may be asked to arrange the steps for assigning an RBAC role.

Conceptually:

``` text
1. Identify who needs access
2. Select the required role
3. Select the scope
4. Assign the role
```

Remember:

``` text
WHO
 ↓
WHAT
 ↓
WHERE
 ↓
ASSIGN
```

------------------------------------------------------------------------

## Type 8 --- Case Study

Example environment:

``` text
Company
 |
 +--- Subscription
 |
 +--- Production RG
 |      |
 |      +--- VM
 |      +--- Storage
 |
 +--- Development RG
        |
        +--- VM
```

Requirements:

``` text
Developer A → Manage Development
Developer B → View Production
Administrator → Manage resources and access
```

You need to determine:

``` text
WHO
 ↓
WHAT ROLE
 ↓
WHICH SCOPE
```

------------------------------------------------------------------------

# ⭐ Quick Revision Table

  Requirement                  Think About
  ---------------------------- ---------------------------
  Who is the user?             Entra ID
  External user                Guest
  Prove identity               Authentication
  What can user access?        Authorization
  Azure resource permissions   Azure RBAC
  Only view resources          Reader
  Manage resources             Contributor
  Manage resources + access    Owner
  Manage role assignments      User Access Administrator
  Access entire subscription   Subscription scope
  Access one resource group    Resource Group scope
  Access one VM                Resource scope
  Multiple subscriptions       Management Group scope

------------------------------------------------------------------------

# ⭐ 10 Questions You Should Practice

## Q1

What is Microsoft Entra ID?

## Q2

What is the difference between authentication and authorization?

## Q3

What is the difference between an Entra role and an Azure RBAC role?

## Q4

What is Azure RBAC?

## Q5

What are the four RBAC scopes?

## Q6

What is the difference between Reader, Contributor, and Owner?

## Q7

Can a Contributor assign RBAC roles to another user?

## Q8

An external consultant needs access to Azure. What type of Entra user is
appropriate?

## Q9

A developer needs access only to one resource group. At which scope
should the role be assigned?

## Q10

A user needs to manage one VM but should not receive access to other
resources. What should you consider?

------------------------------------------------------------------------

# Final Revision Diagram

``` text
                 Microsoft Entra ID
                        |
                        v
                      User
                        |
                        v
                 Authentication
                        |
                        v
                  Azure Portal
                        |
                        v
                 Azure RBAC
                        |
          +-------------+-------------+
          |             |             |
         WHO           WHAT          WHERE
          |             |             |
        User         Role          Scope
        Group        Reader        MG
        SP           Contributor   Subscription
        MI           Owner         Resource Group
                     Other roles   Resource
```

## The most important sentence to remember

> **Azure RBAC determines WHO can do WHAT at WHICH SCOPE.**

``` text
WHO   = Security Principal
WHAT  = Role
WHERE = Scope
```

------------------------------------------------------------------------

# Exam Preparation Priority for These Topics

### High Priority

1.  Azure RBAC
2.  RBAC scopes
3.  Reader vs Contributor vs Owner
4.  Role inheritance
5.  Entra ID vs Azure RBAC
6.  User vs Guest user
7.  Authentication vs Authorization

### Medium Priority

8.  User properties
9.  Groups
10. Role assignment workflow

### Remember

Do not memorize only the definitions. Practice **scenario questions**
where you identify:

``` text
WHO?
WHAT?
WHERE?
```

That approach is especially useful for AZ-104 RBAC questions.
