# Azure Administration Notes --- Topics 270--275

> **Topics covered**
>
> 1.  **270. Performing a self-service password reset**
> 2.  **271. Resource tags**
> 3.  **272. Lab --- Moving resources across resource groups**
> 4.  **273. Moving resources across subscriptions**
> 5.  **274. Lab --- Locking resources**
> 6.  **275. Management groups**

These notes are written for **AZ-104 / Azure Administrator** learning,
with practical examples, Azure CLI commands, portal steps,
troubleshooting points, and exam-style questions.

------------------------------------------------------------------------

# 270. Performing a self-service password reset

## 270.1 What is Self-Service Password Reset (SSPR)?

**Microsoft Entra self-service password reset (SSPR)** allows a user to
reset or change their password without contacting an administrator or
help desk.

### Simple example

Suppose:

-   User: `prins@contoso.com`
-   The user forgets the Microsoft Entra password.
-   SSPR is enabled.
-   The user has already registered authentication methods.

The user can select **Forgot my password** during sign-in, verify their
identity, and create a new password.

Microsoft documentation: SSPR lets users reset passwords or unlock
themselves without administrator/help-desk intervention.

## 270.2 Why use SSPR?

Without SSPR:

``` text
User forgets password
        |
        v
Contact Help Desk
        |
        v
Administrator verifies identity
        |
        v
Administrator resets password
        |
        v
User signs in
```

With SSPR:

``` text
User forgets password
        |
        v
Select "Forgot password?"
        |
        v
Verify identity
        |
        v
Create new password
        |
        v
Sign in
```

### Benefits

-   Reduces help-desk workload.
-   Allows users to recover access themselves.
-   Improves password-recovery availability.
-   Provides authentication-based verification before reset.

## 270.3 Important terms

  -----------------------------------------------------------------------
  Term                                Meaning
  ----------------------------------- -----------------------------------
  SSPR                                Self-Service Password Reset

  Microsoft Entra ID                  Microsoft's cloud identity and
                                      access service

  Authentication method               Method used to verify the user's
                                      identity

  Registration                        User configures authentication
                                      methods for SSPR

  Password reset                      User creates a new password

  Password change                     User changes a known password

  Authentication policy               Controls how identity verification
                                      is performed
  -----------------------------------------------------------------------

## 270.4 Typical SSPR flow

``` text
User
 |
 | Forgot password
 v
Microsoft Entra sign-in
 |
 v
SSPR
 |
 | Verify identity
 v
Authentication method
 |
 +---- Authenticator
 +---- SMS
 +---- Voice call
 +---- Other configured method
 |
 v
Enter new password
 |
 v
Password validation
 |
 v
Password reset completed
```

## 270.5 Administrator configuration

Typical configuration process:

1.  Sign in to Azure portal.
2.  Open **Microsoft Entra ID**.
3.  Select **Password reset**.
4.  Configure the SSPR scope.
5.  Configure authentication methods.
6.  Configure registration settings.
7.  Configure notifications/customization as required.
8.  Save the configuration.
9.  Test with a suitable user account.

## 270.6 SSPR scope

The administrator can configure who can use SSPR.

Common options include:

-   **None** --- SSPR isn't enabled for users.
-   **Selected** --- enable SSPR for selected users/groups.
-   **All** --- enable SSPR for users in scope.

### Practical recommendation for a lab

Use a test group:

``` text
SSPR-Test-Users
       |
       +---- user1
       +---- user2
```

Enable SSPR for the test group first.

Then test before expanding the scope.

## 270.7 Authentication methods

The exact available methods depend on tenant configuration and Microsoft
Entra capabilities.

Common examples include:

-   Microsoft Authenticator
-   Mobile phone / SMS
-   Voice call
-   Email in applicable scenarios
-   Security questions in supported configurations

The important idea is:

``` text
Password reset should require identity verification.
```

## 270.8 User registration

Before the user needs SSPR, the user should register the required
authentication information.

Example:

``` text
User signs in
      |
      v
SSPR registration
      |
      +---- Authenticator
      |
      +---- Phone
      |
      v
Registration completed
```

If the user later forgets the password, the registered method can be
used for verification.

## 270.9 User password-reset example

A user forgets the password.

### Step 1

Open the Microsoft sign-in page.

### Step 2

Select:

``` text
Forgot my password
```

### Step 3

Enter the account identity.

### Step 4

Complete the verification challenge.

Example:

``` text
Approve notification in Microsoft Authenticator
```

### Step 5

Enter a new password.

### Step 6

Submit the reset.

### Step 7

Sign in using the new password.

## 270.10 Password reset vs password change

  ------------------------------------------------------------------------
  Feature                 Password reset          Password change
  ----------------------- ----------------------- ------------------------
  User knows old          Usually no              Yes
  password?                                       

  Typical reason          Forgotten password      Routine/security change

  Identity verification   Required according to   Sign-in/authentication
                          configured policy       context applies

  Example                 Forgot password         Change password from
                                                  account settings
  ------------------------------------------------------------------------

## 270.11 Troubleshooting SSPR

### Problem: User cannot reset password

Check:

1.  Is SSPR enabled?
2.  Is the user inside the configured SSPR scope?
3.  Has the user registered the required authentication method?
4.  Is the authentication method available?
5.  Is the new password compliant with the password policy?
6.  Is there an account or directory issue?

### Problem: User isn't prompted for registration

Check:

-   SSPR registration configuration.
-   User/group scope.
-   Authentication method configuration.
-   Registration policy.

## 270.12 AZ-104 exam focus

Know these concepts:

-   Purpose of SSPR.
-   SSPR registration.
-   SSPR scope.
-   Authentication methods.
-   User reset workflow.
-   Difference between administrator password reset and self-service
    reset.

### Exam-style questions

**Q1.** A company wants users to reset forgotten passwords without
help-desk assistance. Which Azure feature should be used?

**Answer:** Microsoft Entra SSPR.

**Q2.** SSPR is enabled, but a user outside the configured group cannot
use it. What should you check?

**Answer:** The user's SSPR scope/group membership.

**Q3.** Why does SSPR require identity verification?

**Answer:** To reduce the risk of an unauthorized person resetting
another user's password.

------------------------------------------------------------------------

# 271. Resource tags

## 271.1 What is an Azure resource tag?

A **tag** is metadata attached to an Azure resource, resource group, or
subscription.

A tag is a:

``` text
Key = Value
```

Example:

``` text
Environment = Production
Department = IT
Owner = Prins
CostCenter = CC1001
Application = Ecommerce
```

Tags help organizations organize resources, track costs, automate
governance, and identify ownership.

## 271.2 Real-world example

Suppose a company has:

``` text
VM-Web-01
VM-Web-02
VM-DB-01
Storage-01
```

Add:

``` text
Environment = Production
Department = Application
Project = Ecommerce
Owner = DevOps
```

Now administrators can identify which resources belong to the project.

## 271.3 Common tagging strategy

A practical strategy:

  Tag           Example
  ------------- ------------
  Environment   Production
  Application   Ecommerce
  Owner         DevOps
  Department    IT
  CostCenter    CC1001
  Project       Website
  ManagedBy     Terraform
  Criticality   High

## 271.4 Why tags are useful

### Organization

Identify resources by application or project.

### Cost management

Example:

``` text
CostCenter = CC1001
```

This can help analyze cost by cost center where supported.

### Automation

Automation scripts can find resources based on tags.

Example:

``` text
Environment = Development
```

A scheduled process might stop development VMs outside working hours.

### Governance

Azure Policy can be used to require or enforce tagging standards.

## 271.5 Tag example

``` text
Resource: web-vm-01

Environment = Production
Application = Web
Owner = DevOps
CostCenter = CC1001
```

## 271.6 Important tag facts

According to current Microsoft documentation:

-   Tags can be applied to supported resources, resource groups, and
    subscriptions.
-   Management groups themselves aren't taggable.
-   Not every Azure resource type supports tags.
-   Tags are stored as plain text.
-   Do **not** store passwords, secrets, tokens, or other sensitive data
    in tags.
-   A resource, resource group, or subscription can have up to **50 tag
    name-value pairs** under the documented limits.
-   Tag names are case-insensitive for operations, while tag values are
    case-sensitive.

## 271.7 Tag inheritance

A common exam trap:

``` text
Resource Group
    |
    +-- Tag: Environment=Production
    |
    +-- VM
    +-- Storage
```

The VM and Storage account **do not automatically inherit** the resource
group's tags.

If an organization wants consistent tags on resources, Azure Policy can
be used to apply or enforce the desired behavior.

## 271.8 Azure Portal --- add a tag

Typical steps:

1.  Open Azure portal.
2.  Open the resource.
3.  Find **Tags**.
4.  Select **Add tags** or edit tags.
5.  Enter key and value.
6.  Select **Save**.

Example:

``` text
Name: Environment
Value: Production
```

## 271.9 Azure CLI --- tag a resource

Example:

``` bash
az resource tag \
  --ids <RESOURCE_ID> \
  --tags Environment=Production Owner=DevOps
```

Another common approach:

``` bash
az tag update \
  --resource-id <RESOURCE_ID> \
  --operation Merge \
  --tags Environment=Production
```

> Check the command syntax available in your installed Azure CLI
> version. Be careful with commands that replace an existing tag set.

## 271.10 View tags

``` bash
az resource show \
  --ids <RESOURCE_ID> \
  --query tags
```

Example output:

``` json
{
  "Environment": "Production",
  "Owner": "DevOps"
}
```

## 271.11 Resource tag scenario

### Scenario

You have 100 VMs.

You want to identify development machines.

Use:

``` text
Environment = Development
```

Then resources can be filtered/managed based on that metadata.

## 271.12 Tagging mistakes

### Mistake 1 --- Storing secrets

Bad:

``` text
Password = MyPassword123
```

Never store credentials in tags.

### Mistake 2 --- Inconsistent names

Bad:

``` text
environment=prod
Env=Production
ENV=PROD
```

Better:

``` text
Environment=Production
```

### Mistake 3 --- Assuming inheritance

Resource group tags don't automatically become resource tags.

## 271.13 AZ-104 exam focus

Know:

-   Key-value structure.
-   Cost-management use.
-   Governance use.
-   Tag inheritance behavior.
-   50 tag-pair documented limit.
-   Tags are not secrets.
-   Resource-group/subscription tags vs resource tags.

### Exam-style questions

**Q1.** What is the structure of an Azure tag?

**Answer:** Key-value pair.

**Q2.** A resource group has `Environment=Production`. Will a VM
automatically receive the same tag?

**Answer:** No. Resource tags don't automatically inherit resource-group
tags.

**Q3.** Can tags contain passwords?

**Answer:** No. Tags are plain-text metadata and aren't appropriate for
secrets.

------------------------------------------------------------------------

# 272. Lab --- Moving resources across resource groups

## 272.1 What does moving a resource group mean?

An Azure resource can sometimes be moved from:

``` text
Resource Group A
        |
        +---- VM
```

to:

``` text
Resource Group B
        |
        +---- VM
```

This changes the resource's **resource group**, but it does not by
itself move the resource to another Azure region.

## 272.2 Why move resources?

Common reasons:

-   Correct an incorrect resource-group design.
-   Separate production and development resources.
-   Reorganize projects.
-   Change ownership/governance boundaries.
-   Consolidate resources.

## 272.3 Example

Before:

``` text
RG-Development
    |
    +---- web-vm
    +---- storage
```

After:

``` text
RG-Production
    |
    +---- web-vm
```

## 272.4 Important concepts before moving

Always check:

1.  Is the resource type movable?
2.  Are dependencies supported?
3.  Are source and destination scopes valid?
4.  Are you allowed to perform the move?
5.  Are there locks?
6.  Will resource IDs change?
7.  Will RBAC assignments need to be recreated?
8.  Are there references to the old resource ID?

Microsoft maintains a resource-type support list because move support
varies by resource type.

## 272.5 Resource ID changes

A resource ID normally looks similar to:

``` text
/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/<provider>/<type>/<name>
```

If the resource group changes:

``` text
/subscriptions/ABC/resourceGroups/RG-OLD/...
```

may become:

``` text
/subscriptions/ABC/resourceGroups/RG-NEW/...
```

Therefore, any external configuration that explicitly references the old
resource ID may need updating.

## 272.6 Portal process

Typical steps:

1.  Open the source resource group.
2.  Select the resource(s).
3.  Select **Move**.
4.  Choose **Move to another resource group**.
5.  Select/create the destination resource group.
6.  Validate.
7.  Confirm.
8.  Start the move.
9.  Verify the destination.

## 272.7 Azure CLI example

Generic command:

``` bash
az resource move \
  --destination-group <DESTINATION_RG> \
  --ids <RESOURCE_ID>
```

For multiple resources:

``` bash
az resource move \
  --destination-group <DESTINATION_RG> \
  --ids <RESOURCE_ID_1> <RESOURCE_ID_2>
```

## 272.8 Important move behavior

During a resource move, Azure locks the source and destination resource
groups for the duration of the operation. This blocks write/delete
operations in those groups during the move.

The resources themselves can continue operating in many move scenarios;
the move is not the same thing as moving the physical resource to
another region.

## 272.9 Parent and child resources

Generally, specify the **parent resource**.

Example:

``` text
Virtual Machine
   |
   +---- VM Extension
```

When supported, moving the parent moves its child resources
automatically.

You normally cannot move a child resource independently of its parent.

## 272.10 Dependencies

Example:

``` text
VM
 |
 +---- NIC
 |
 +---- Managed Disk
 |
 +---- Public IP
 |
 +---- NSG
```

A move can have dependency requirements.

Before moving a real production workload:

``` text
Identify dependencies
        |
        v
Check move support
        |
        v
Plan move
        |
        v
Validate
        |
        v
Move
        |
        v
Verify application
```

## 272.11 Lab example

### Goal

Move:

``` text
VM: web-vm
Source RG: RG-Old
Destination RG: RG-New
```

### Step 1 --- Create resource groups

``` bash
az group create \
  --name RG-Old \
  --location eastus

az group create \
  --name RG-New \
  --location eastus
```

### Step 2 --- Check resources

``` bash
az resource list \
  --resource-group RG-Old \
  --output table
```

### Step 3 --- Get resource ID

``` bash
az resource list \
  --resource-group RG-Old \
  --query "[].id" \
  --output tsv
```

### Step 4 --- Move the resource

``` bash
az resource move \
  --destination-group RG-New \
  --ids <RESOURCE_ID>
```

### Step 5 --- Verify

``` bash
az resource list \
  --resource-group RG-New \
  --output table
```

## 272.12 Common move problems

### Error: Resource type doesn't support move

Cause:

The particular resource type has a move restriction.

Solution:

Check Microsoft's supported-resource list and service-specific guidance.

### Error: Read-only lock

A read-only lock on the source, destination, or subscription can prevent
a move.

Remove/change the lock if you are authorized and the move is approved.

### Error: Dependency problem

A dependent resource may need to be moved with its parent or may not
support the requested move.

### Error: RBAC issue

Role assignments can be affected because resource IDs/scopes can change.

Review access after the move.

## 272.13 Exam-style questions

**Q1.** Does moving a VM to another resource group automatically move it
to another Azure region?

**Answer:** No.

**Q2.** What happens to the resource ID when the resource group changes?

**Answer:** The resource ID changes because the resource-group portion
of the ID changes.

**Q3.** Can every Azure resource be moved between resource groups?

**Answer:** No. Move support depends on the resource type.

**Q4.** What happens to source and destination resource groups during a
move?

**Answer:** They are locked for the duration of the move operation.

------------------------------------------------------------------------

# 273. Moving resources across subscriptions

## 273.1 What does cross-subscription move mean?

A resource can, when supported, be moved from:

``` text
Subscription-A
    |
    +---- RG-A
          |
          +---- VM
```

to:

``` text
Subscription-B
    |
    +---- RG-B
          |
          +---- VM
```

## 273.2 Why move resources between subscriptions?

Examples:

-   Department receives its own subscription.
-   Project moves from development to production subscription.
-   Subscription restructuring.
-   Cost-management separation.
-   Organizational changes.

## 273.3 Important requirement

For a normal cross-subscription resource move, the source and
destination subscriptions generally need to be associated with the
**same Microsoft Entra tenant**.

A resource move isn't a method for moving resources into a different
Microsoft Entra tenant.

## 273.4 Cross-subscription dependency problem

Suppose:

``` text
Subscription A
 |
 +-- RG-Web
      |
      +-- VM
      +-- NIC
      +-- Disk
```

If these resources have dependencies, they may need to be moved
together.

For cross-subscription moves, Microsoft documents that the resource and
its dependent resources need to be in the same resource group and moved
together in applicable scenarios.

## 273.5 Three-step planning model

A useful way to understand a complex cross-subscription move:

``` text
Step 1
Put required dependent resources together
        |
        v
Step 2
Move resources to destination subscription
        |
        v
Step 3
Reorganize them into destination resource groups
```

## 273.6 Portal process

Typical process:

1.  Open the resource group.
2.  Select resources.
3.  Select **Move**.
4.  Select **Move to another subscription**.
5.  Select destination subscription.
6.  Select destination resource group.
7.  Validate.
8.  Confirm the implications.
9.  Start move.
10. Verify resources in destination subscription.

## 273.7 Azure CLI example

Generic pattern:

``` bash
az resource move \
  --destination-subscription-id <DESTINATION_SUBSCRIPTION_ID> \
  --destination-group <DESTINATION_RG> \
  --ids <RESOURCE_ID_1> <RESOURCE_ID_2>
```

## 273.8 Permissions

The account performing the move needs appropriate permissions at the
source and destination scopes.

Microsoft documents permissions including:

``` text
Source:
Microsoft.Resources/subscriptions/resourceGroups/moveResources/action

Destination:
Microsoft.Resources/subscriptions/resourceGroups/write
```

Exact permissions can vary depending on the resources and operation.

## 273.9 RBAC after moving

Important concept:

``` text
Resource moves
      |
      v
Resource scope/resource ID changes
      |
      v
Existing role assignments may need review/recreation
```

Microsoft documents that active role assignments on moved resources can
become orphaned and should be reviewed.

## 273.10 Subscription quotas

Before moving resources into a destination subscription:

``` text
Check destination subscription
        |
        +---- Quotas
        +---- Resource providers
        +---- Permissions
        +---- Policy
        +---- Locks
        +---- Region/resource support
```

The destination subscription must be capable of hosting the resources.

## 273.11 Tenant vs subscription

This is an important AZ-104 distinction.

``` text
Microsoft Entra tenant
        |
        +---- Subscription A
        |
        +---- Subscription B
        |
        +---- Subscription C
```

Subscriptions can be separate billing/management boundaries while being
associated with the same tenant.

A resource move between subscriptions is not the same thing as moving a
resource between tenants.

## 273.12 Resource move vs resource migration

### Resource move

Changes management location such as:

``` text
Resource Group
Subscription
```

### Region migration

Changes the Azure physical/geographic region.

These are different operations.

## 273.13 Exam-style questions

**Q1.** Can a resource normally be moved across subscriptions that
belong to different Microsoft Entra tenants?

**Answer:** No, the standard move operation doesn't support moving
resources to a new Microsoft Entra tenant.

**Q2.** What should you check before moving a resource to another
subscription?

**Answer:** Move support, dependencies, permissions, locks, quotas,
policies, resource providers, and destination requirements.

**Q3.** Does a subscription move automatically change the Azure region?

**Answer:** No.

**Q4.** Why can RBAC need review after a move?

**Answer:** The resource's scope/resource ID can change, and role
assignments may become orphaned or need to be recreated.

------------------------------------------------------------------------

# 274. Lab --- Locking resources

## 274.1 What is an Azure resource lock?

A **resource lock** protects an Azure resource, resource group, or
subscription from accidental deletion or modification.

Locks are an additional protection layer.

## 274.2 Two important lock types

### CanNotDelete

Portal name:

``` text
Delete
```

CLI/API name:

``` text
CanNotDelete
```

Meaning:

``` text
Read       -> Allowed
Modify     -> Allowed
Delete     -> Blocked
```

### ReadOnly

Portal name:

``` text
Read-only
```

CLI/API name:

``` text
ReadOnly
```

Meaning:

``` text
Read       -> Allowed
Modify     -> Blocked
Delete     -> Blocked
```

## 274.3 Easy memory trick

``` text
CanNotDelete
    = You can change it
    = You cannot delete it

ReadOnly
    = You can read it
    = You cannot change it
    = You cannot delete it
```

## 274.4 Lock hierarchy

Locks can be applied to:

``` text
Subscription
     |
     +---- Resource Group
              |
              +---- Resource
```

A lock can affect child resources through inheritance.

## 274.5 Example

Suppose:

``` text
RG-Production
 |
 +---- VM
 +---- Storage
 +---- Database
```

Apply:

``` text
CanNotDelete
```

to `RG-Production`.

The resources under that scope are protected from deletion through the
lock inheritance model.

## 274.6 Important: lock overrides user permissions

Even if a user has permission to delete a resource, a lock can prevent
the operation.

Example:

``` text
User
 |
 +---- Owner
 |
 +---- Delete permission
 |
 v
Resource has CanNotDelete
 |
 v
Delete operation blocked
```

## 274.7 Portal lab

### Goal

Create a Delete lock on a production resource.

### Steps

1.  Open Azure portal.
2.  Open the target resource.
3.  Open **Locks**.
4.  Select **Add**.
5.  Enter a lock name.

Example:

``` text
Lock name: Protect-Web-VM
```

6.  Select lock type:

``` text
Delete
```

7.  Add notes if required.
8.  Create the lock.

## 274.8 Test the lock

Try to delete the resource.

Expected behavior:

``` text
Delete operation
      |
      v
Azure checks lock
      |
      v
CanNotDelete lock found
      |
      v
Delete blocked
```

## 274.9 Azure CLI

Create a CanNotDelete lock:

``` bash
az lock create \
  --name Protect-Production \
  --lock-type CanNotDelete \
  --resource-group RG-Production
```

List locks:

``` bash
az lock list \
  --resource-group RG-Production \
  --output table
```

Delete a lock:

``` bash
az lock delete \
  --name Protect-Production \
  --resource-group RG-Production
```

> You need sufficient permissions to create/delete locks.

## 274.10 ReadOnly example

``` bash
az lock create \
  --name Production-ReadOnly \
  --lock-type ReadOnly \
  --resource-group RG-Production
```

With ReadOnly:

``` text
Read resource    -> Yes
Update resource  -> No
Delete resource  -> No
```

## 274.11 Important storage-account trap

A lock on a storage account is not equivalent to protecting all data
inside the account.

Microsoft documents that locks primarily protect **control-plane**
operations. Data-plane operations can behave differently.

Therefore:

``` text
Resource lock != backup
Resource lock != data protection
Resource lock != ransomware protection
```

Use appropriate backup, retention, recovery, and data-protection
mechanisms for data protection.

## 274.12 Lock troubleshooting

### Problem

You try to move a resource and receive an error related to a read-only
lock.

### Reason

A read-only lock can prevent resource moves.

### Solution

Check locks on:

-   Source resource group.
-   Destination resource group.
-   Source subscription.
-   Destination subscription, as applicable.

Remove or adjust the lock only if authorized and the move is approved.

## 274.13 Lock vs RBAC

These are different concepts.

  -----------------------------------------------------------------------
  Feature                 RBAC                    Lock
  ----------------------- ----------------------- -----------------------
  Controls access         Yes                     No

  Determines who can      Yes                     No
  perform actions                                 

  Protects from           Indirectly              Yes
  accidental                                      
  deletion/modification                           

  Can override user       Permission model        Lock can block
  permission?                                     operations

  Example                 Contributor             CanNotDelete
  -----------------------------------------------------------------------

Think:

``` text
RBAC = Who can do what?
Lock = Even if allowed, should this operation be blocked?
```

## 274.14 Exam-style questions

**Q1.** Which lock allows modification but blocks deletion?

**Answer:** CanNotDelete.

**Q2.** Which lock blocks modification and deletion?

**Answer:** ReadOnly.

**Q3.** A user is Owner but cannot delete a locked resource. Why?

**Answer:** The resource lock blocks the delete operation.

**Q4.** Does a storage-account resource lock guarantee that blob data
cannot be deleted through all interfaces?

**Answer:** No. Locks primarily protect control-plane operations;
data-plane operations require separate data-protection controls.

------------------------------------------------------------------------

# 275. Management groups

## 275.1 What is an Azure Management Group?

An **Azure management group** is a container above subscriptions.

It provides a hierarchical way to apply:

-   Azure Policy
-   RBAC
-   Governance
-   Compliance
-   Management settings

across multiple subscriptions.

## 275.2 Azure hierarchy

Remember this hierarchy:

``` text
Microsoft Entra Tenant
        |
        v
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

A more complete organizational example:

``` text
Tenant
 |
 +-- Corporate Management Group
       |
       +-- Production Management Group
       |      |
       |      +-- Production Subscription
       |
       +-- Development Management Group
              |
              +-- Development Subscription
              +-- Test Subscription
```

## 275.3 Why use management groups?

Imagine an enterprise with:

``` text
20 subscriptions
```

Instead of applying a policy separately to every subscription:

``` text
Policy
 |
 +-- Subscription 1
 +-- Subscription 2
 +-- Subscription 3
 ...
 +-- Subscription 20
```

Use a management group:

``` text
Management Group
 |
 +-- Subscription 1
 +-- Subscription 2
 +-- Subscription 3
 ...
 +-- Subscription 20
```

Apply governance at the management-group scope.

The child subscriptions inherit applicable conditions.

## 275.4 Management group example

Company:

``` text
Contoso
 |
 +-- Production-MG
 |      |
 |      +-- Prod-Subscription-01
 |      +-- Prod-Subscription-02
 |
 +-- NonProduction-MG
        |
        +-- Dev-Subscription-01
        +-- Test-Subscription-01
```

Possible policy:

``` text
Production-MG
   |
   +---- Allowed locations:
         East US
         West Europe
```

The subscriptions underneath inherit the policy assignment as
applicable.

## 275.5 Management groups and Azure Policy

Management groups are especially useful with Azure Policy.

Example:

``` text
Management Group
       |
       v
Azure Policy:
Allowed locations = East US
       |
       +---- Subscription A
       +---- Subscription B
       +---- Subscription C
```

This gives centralized governance.

## 275.6 Management groups and RBAC

Azure RBAC can also be assigned at management-group scope.

Example:

``` text
Management Group
 |
 +---- Reader role
       |
       +---- Subscription A
       +---- Subscription B
       +---- Subscription C
```

The assignment can be inherited by child scopes according to Azure RBAC
scope/inheritance rules.

## 275.7 Important hierarchy rule

Only certain objects can be direct children of a management group.

A management group can contain:

-   Other management groups.
-   Subscriptions.

It does not directly contain resource groups or resources.

## 275.8 Management group depth

Azure management groups support hierarchical organization.

A common structure is:

``` text
Root Management Group
       |
       +-- Organization
              |
              +-- Production
              |      |
              |      +-- Subscription
              |
              +-- NonProduction
                     |
                     +-- Subscription
```

Avoid creating unnecessarily complicated hierarchies. Design the
hierarchy around actual governance requirements.

## 275.9 Creating a management group in Portal

Typical steps:

1.  Open Azure portal.
2.  Search for **Management groups**.
3.  Select **Management groups**.
4.  Select **Create**.
5.  Enter:
    -   Management group ID.
    -   Display name.
6.  Create the management group.
7.  Add/move subscriptions as appropriate.

## 275.10 Azure CLI example

A management group can be created using Azure CLI.

Generic pattern:

``` bash
az account management-group create \
  --name <MANAGEMENT_GROUP_ID> \
  --display-name "Production"
```

List management groups:

``` bash
az account management-group list \
  --output table
```

Show a management group:

``` bash
az account management-group show \
  --name <MANAGEMENT_GROUP_ID>
```

## 275.11 Azure PowerShell

List management groups:

``` powershell
Get-AzManagementGroup
```

Get one management group:

``` powershell
Get-AzManagementGroup -GroupId "Production"
```

## 275.12 Moving a subscription into a management group

Conceptually:

``` text
Subscription
     |
     | move
     v
Production Management Group
```

After the subscription becomes a child of the management group, it can
inherit governance settings from that parent.

## 275.13 Management group vs resource group

This is an important exam comparison.

  Feature                           Management Group                  Resource Group
  --------------------------------- --------------------------------- -------------------------
  Main purpose                      Organize/govern subscriptions     Organize resources
  Contains                          Management groups/subscriptions   Resources
  Scope                             Above subscription                Below subscription
  Policy                            Can apply                         Can apply
  RBAC                              Can assign                        Can assign
  Contains VM directly?             No                                Yes
  Used for enterprise governance?   Yes                               Yes, but at lower scope

## 275.14 Management group vs subscription

  -----------------------------------------------------------------------
  Feature                 Management Group        Subscription
  ----------------------- ----------------------- -----------------------
  Position                Above subscription      Below management group

  Billing boundary        No                      Yes

  Can contain             Yes                     No
  subscriptions                                   

  Can contain resource    No                      Yes
  groups                                          

  Governance              Excellent for           Subscription-level
                          multi-subscription      governance
                          governance              
  -----------------------------------------------------------------------

## 275.15 Management group inheritance

Example:

``` text
Production-MG
 |
 +-- Policy: Allowed locations
 |
 +-- Subscription-A
 |      |
 |      +-- RG-A
 |             |
 |             +-- VM
 |
 +-- Subscription-B
        |
        +-- RG-B
               |
               +-- Storage
```

A policy assigned at `Production-MG` can flow down to child scopes.

This is one of the most important management-group concepts for AZ-104.

## 275.16 Management group design example

For a company:

``` text
Tenant Root
 |
 +-- Platform
 |     |
 |     +-- Network-Subscription
 |     +-- Security-Subscription
 |
 +-- Workloads
       |
       +-- Production
       |     |
       |     +-- Prod-Subscription
       |
       +-- NonProduction
             |
             +-- Dev-Subscription
             +-- Test-Subscription
```

Possible governance:

``` text
Platform
   -> Security policies

Production
   -> Strong production policies

NonProduction
   -> Development/test policies
```

## 275.17 Management group permissions

Moving subscriptions or management groups requires appropriate
permissions at the relevant scopes.

For example, Microsoft documents management-group permissions such as:

``` text
Microsoft.management/managementgroups/write
Microsoft.management/managementgroups/subscriptions/write
Microsoft.Authorization/roleAssignments/write
Microsoft.Authorization/roleAssignments/delete
Microsoft.Management/register/action
```

Exact requirements depend on the operation and current hierarchy.

## 275.18 Management group troubleshooting

### Problem: Cannot see management groups

Possible causes:

-   Insufficient permissions.
-   Wrong Microsoft Entra directory/tenant.
-   Subscription/account is associated with another directory.

### Problem: Cannot move a subscription

Check:

-   Permissions on current parent.
-   Permissions on target parent.
-   Ownership/RBAC inheritance.
-   Management-group hierarchy rules.
-   Whether the operation would remove required access.

## 275.19 AZ-104 exam focus

Know:

-   Management groups are above subscriptions.
-   They organize multiple subscriptions.
-   They support centralized governance.
-   Azure Policy can be assigned at management-group scope.
-   RBAC can be assigned at management-group scope.
-   Child scopes can inherit applicable governance/access.
-   Management groups can contain subscriptions and other management
    groups.
-   Resource groups are below subscriptions.

### Exam-style questions

**Q1.** A company has 30 Azure subscriptions and wants to apply one
policy to all production subscriptions. What should it use?

**Answer:** A management group containing the production subscriptions.

**Q2.** What is the scope order from highest to lowest?

**Answer:**

``` text
Management Group
    ↓
Subscription
    ↓
Resource Group
    ↓
Resource
```

**Q3.** Can a management group directly contain a virtual machine?

**Answer:** No. Management groups contain management groups and
subscriptions.

**Q4.** Can Azure Policy be assigned at management-group scope?

**Answer:** Yes.

**Q5.** Can RBAC be assigned at management-group scope?

**Answer:** Yes.

------------------------------------------------------------------------

# Quick Revision --- Topics 270--275

## 1. SSPR

``` text
Forgot password
      ↓
Verify identity
      ↓
Reset password
```

**Key:** Microsoft Entra SSPR.

## 2. Tags

``` text
Key = Value
```

Example:

``` text
Environment = Production
```

**Key:** Metadata, organization, cost analysis, governance.

## 3. Move across resource groups

``` text
RG-A
 |
 +-- VM

       MOVE

RG-B
 |
 +-- VM
```

**Key:** Resource group changes; Azure region doesn't automatically
change.

## 4. Move across subscriptions

``` text
Subscription-A
      |
      +-- Resource

             MOVE

Subscription-B
      |
      +-- Resource
```

**Key:** Check tenant, move support, dependencies, permissions, quotas,
policies, locks, and RBAC.

## 5. Locks

``` text
CanNotDelete
    = Modify YES
    = Delete NO

ReadOnly
    = Modify NO
    = Delete NO
```

**Key:** Locks protect against accidental changes/deletion.

## 6. Management groups

``` text
Management Group
       ↓
Subscription
       ↓
Resource Group
       ↓
Resource
```

**Key:** Centralized governance across subscriptions.

------------------------------------------------------------------------

# Important AZ-104 Comparison Table

  -----------------------------------------------------------------------
  Topic                   Main Purpose            Important Exam Point
  ----------------------- ----------------------- -----------------------
  SSPR                    User password recovery  User can reset password
                                                  after identity
                                                  verification

  Tags                    Resource metadata       Key-value pairs

  Resource Group Move     Reorganize resources    Resource ID changes

  Subscription Move       Reorganize subscription Same Microsoft Entra
                          ownership/management    tenant for standard
                                                  moves

  CanNotDelete Lock       Prevent deletion        Modification still
                                                  allowed

  ReadOnly Lock           Prevent changes         Modification and
                                                  deletion blocked

  Management Group        Governance across       Above subscription
                          subscriptions           scope
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# Practical AZ-104 Lab Sequence

A good hands-on order is:

``` text
1. Create Resource Group
        ↓
2. Create a test resource
        ↓
3. Add resource tags
        ↓
4. Move resource to another Resource Group
        ↓
5. Verify resource ID/scope
        ↓
6. Create a second subscription if available
        ↓
7. Practice cross-subscription move with a supported resource
        ↓
8. Create CanNotDelete lock
        ↓
9. Try deleting resource
        ↓
10. Create ReadOnly lock
        ↓
11. Test modification
        ↓
12. Create Management Group
        ↓
13. Add subscriptions
        ↓
14. Review inherited governance
```

> **Lab safety:** Use test resources and monitor your Azure costs.
> Remove test resources, locks, and management-group objects only when
> appropriate and when you understand their dependencies.

------------------------------------------------------------------------

# Common AZ-104 Exam Traps

## Trap 1

**Question:** Resource group tag automatically appears on every
resource.

**Answer:** False.

------------------------------------------------------------------------

## Trap 2

**Question:** CanNotDelete prevents modifications.

**Answer:** False.

``` text
CanNotDelete:
Modify = allowed
Delete = blocked
```

------------------------------------------------------------------------

## Trap 3

**Question:** ReadOnly only prevents deletion.

**Answer:** False.

It prevents updates and deletes.

------------------------------------------------------------------------

## Trap 4

**Question:** Moving a resource group changes its Azure region.

**Answer:** False.

A resource-group/subscription move is different from a region migration.

------------------------------------------------------------------------

## Trap 5

**Question:** Every Azure resource can be moved.

**Answer:** False.

Move support depends on resource type and dependencies.

------------------------------------------------------------------------

## Trap 6

**Question:** Management groups contain resource groups.

**Answer:** False.

The management-group hierarchy directly contains management groups and
subscriptions.

------------------------------------------------------------------------

## Trap 7

**Question:** Resource locks replace backups.

**Answer:** False.

Locks and backups solve different problems.

------------------------------------------------------------------------

## Trap 8

**Question:** Tags are a secure place for passwords.

**Answer:** False.

Tags are plain-text metadata.

------------------------------------------------------------------------

# Final Memory Map

``` text
                    AZURE GOVERNANCE
                          |
       +------------------+------------------+
       |                  |                  |
      SSPR              Tags              Locks
       |                  |                  |
 Password recovery     Metadata       Protect resources
       |                  |                  |
       +------------------+------------------+
                          |
                     MANAGEMENT
                          |
                Management Groups
                          |
                    Subscriptions
                          |
                   Resource Groups
                          |
                      Resources
```

------------------------------------------------------------------------

# One-Minute Revision

### SSPR

**Self-service password reset** allows users to reset forgotten
passwords after configured identity verification.

### Tags

**Tags = key/value metadata.**

Example:

``` text
Environment=Production
```

### Move resource group

Changes the resource's resource-group scope. The resource ID changes;
the region does not automatically change.

### Move subscription

Moves supported resources to another subscription. Check tenant,
dependencies, permissions, quotas, policies, locks, and RBAC.

### CanNotDelete

Can read and modify, but cannot delete.

### ReadOnly

Can read, but cannot modify or delete.

### Management Group

A governance scope **above subscriptions** used to organize
subscriptions and apply policies/RBAC at scale.

------------------------------------------------------------------------

# Official Microsoft Learn References

-   Microsoft Entra SSPR:
    https://learn.microsoft.com/en-us/azure/active-directory/authentication/quickstart-sspr
-   Azure resource tags:
    https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/tag-resources
-   Move Azure resources:
    https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/move-resources-overview
-   Move resources to another resource group/subscription:
    https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/move-resource-group-and-subscription
-   Azure resource move support:
    https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/move-support-resources
-   Azure resource locks:
    https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/lock-resources
-   Azure management groups:
    https://learn.microsoft.com/en-us/azure/governance/management-groups/
