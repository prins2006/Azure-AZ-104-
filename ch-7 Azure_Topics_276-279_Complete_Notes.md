# Azure Administration / AZ-104 Notes --- Topics 276--279

## Topics Covered

1.  **276. Just a quick look into Management Groups**
2.  **277. Lab --- Using the Azure Policy Service**
3.  **278. Lab --- Azure Policy --- Not Allowed Resource Types**
4.  **279. Section Summary**

These notes explain the concepts, practical examples, Azure Portal
steps, Azure CLI examples, troubleshooting, and AZ-104 exam-style
questions.

------------------------------------------------------------------------

# 276. Just a quick look into Management Groups

## 276.1 What is an Azure Management Group?

An **Azure Management Group** is a governance container used to organize
multiple Azure subscriptions.

The hierarchy is:

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

Management groups are useful when an organization has many subscriptions
and wants to apply common governance rules at a higher scope.

Microsoft documents management groups as a scope above subscriptions
where governance such as Azure Policy and RBAC can be applied.

## 276.2 Example

Imagine a company has:

``` text
Contoso Tenant
 |
 +-- Production Management Group
 |       |
 |       +-- Production Subscription 1
 |       +-- Production Subscription 2
 |
 +-- Development Management Group
         |
         +-- Development Subscription
         +-- Test Subscription
```

Instead of configuring every subscription separately, governance can be
applied at the appropriate management-group level.

## 276.3 Why management groups are useful

Management groups help with:

-   Centralized governance
-   Azure Policy
-   RBAC
-   Compliance
-   Organization of subscriptions
-   Enterprise-scale Azure administration

## 276.4 Management group inheritance

Suppose:

``` text
Production-MG
      |
      +-- Policy: Allowed locations = East US
      |
      +-- Subscription-A
      |       |
      |       +-- RG-A
      |              |
      |              +-- VM
      |
      +-- Subscription-B
              |
              +-- RG-B
                     |
                     +-- Storage
```

The policy assignment at the management-group scope can apply to child
subscriptions and their resources according to Azure Policy scope and
inheritance.

## 276.5 Important rule

A management group can contain:

``` text
Management Groups
Subscriptions
```

It does **not** directly contain:

``` text
Resource Groups
Virtual Machines
Storage Accounts
```

Those are lower in the hierarchy.

## 276.6 Management group vs subscription

  -----------------------------------------------------------------------
  Feature                 Management Group        Subscription
  ----------------------- ----------------------- -----------------------
  Position                Above subscription      Below management group

  Contains subscriptions  Yes                     No

  Contains resource       No                      Yes
  groups                                          

  Policy scope            Yes                     Yes

  RBAC scope              Yes                     Yes

  Main purpose            Multi-subscription      Billing/resource
                          governance              boundary
  -----------------------------------------------------------------------

## 276.7 Example: company governance

Suppose the company has 20 subscriptions.

Without management groups:

``` text
Policy
 |
 +-- Subscription 1
 +-- Subscription 2
 +-- Subscription 3
 ...
 +-- Subscription 20
```

With management groups:

``` text
Production-MG
 |
 +-- Subscription 1
 +-- Subscription 2
 +-- Subscription 3
 ...
```

A common governance policy can be assigned to `Production-MG`.

## 276.8 AZ-104 exam points

Remember:

``` text
Management Group
       ↓
Subscription
       ↓
Resource Group
       ↓
Resource
```

Management groups are especially useful for **centralized governance
across multiple subscriptions**.

### Exam-style questions

**Q1.** What is directly below a management group?

**Answer:** Another management group or a subscription.

**Q2.** Can a management group directly contain a VM?

**Answer:** No.

**Q3.** Why would an enterprise use management groups?

**Answer:** To organize subscriptions and apply governance consistently
across multiple subscriptions.

------------------------------------------------------------------------

# 277. Lab --- Using the Azure Policy Service

# 277.1 What is Azure Policy?

**Azure Policy** is a governance service that helps enforce
organizational standards and evaluate Azure resources for compliance.

A policy definition describes:

``` text
IF a resource meets a condition
THEN apply an effect
```

Example:

``` text
IF location is not East US
THEN DENY deployment
```

Microsoft describes Azure Policy as a service for enforcing
organizational standards and assessing compliance at scale. Policy
definitions can be assigned at scopes such as management groups,
subscriptions, resource groups, and resources.
citeturn0search0turn0search9

## 277.2 Azure Policy vs RBAC

This is an important AZ-104 distinction.

### RBAC

RBAC answers:

``` text
WHO can perform an action?
```

Example:

``` text
Prins
 |
 +-- Contributor
       |
       +-- Can manage resources
```

### Azure Policy

Policy answers:

``` text
WHAT configurations/actions are allowed or required?
```

Example:

``` text
Policy:
Only East US resources allowed
```

Even if a user has Contributor permission, a policy can deny a
deployment that violates the policy.

## 277.3 Azure Policy basic structure

A policy contains:

``` text
Policy Definition
      |
      +-- Conditions
      |
      +-- Effect
```

Example:

``` text
IF
  resource type = Microsoft.Compute/virtualMachines

AND
  location != eastus

THEN
  deny
```

## 277.4 Policy definition

A **policy definition** describes the rule.

Example:

``` text
Name:
Allowed locations

Rule:
Only allow resources in East US
```

Microsoft provides many built-in policy definitions, and administrators
can also create custom policies. citeturn0search0turn0search2

## 277.5 Policy assignment

Creating a policy definition does not by itself enforce it.

You need an:

``` text
Assignment
```

The assignment connects the policy definition to a scope.

Example:

``` text
Policy Definition
      |
      v
Allowed Locations
      |
      v
Assignment
      |
      v
Subscription
```

## 277.6 Policy scope

A policy can be assigned at appropriate Azure scopes such as:

``` text
Management Group
Subscription
Resource Group
Resource
```

A policy assigned higher in the hierarchy can affect child scopes unless
excluded.

## 277.7 Important Azure Policy objects

  -----------------------------------------------------------------------
  Object                              Meaning
  ----------------------------------- -----------------------------------
  Policy Definition                   Rule describing desired state

  Assignment                          Applies definition to a scope

  Initiative                          Group of multiple policy
                                      definitions

  Compliance                          Shows whether resources meet policy

  Exemption                           Excludes specific scopes/resources
                                      from a policy assignment

  Remediation                         Corrects applicable existing
                                      non-compliant resources for
                                      supported effects
  -----------------------------------------------------------------------

## 277.8 Common policy effects

Some important effects are:

### Audit

``` text
Resource violates policy
       |
       v
Resource is marked non-compliant
```

The resource is not blocked.

### Deny

``` text
Resource violates policy
       |
       v
Deployment blocked
```

Microsoft documents that the `deny` effect prevents matching resource
requests and can return a 403/forbidden result. citeturn0search11

### Modify

Can add, update, or remove supported properties/tags during resource
creation or update, and can support remediation of existing resources in
supported scenarios. citeturn0search7

### Append

Adds specified properties to a request in supported scenarios.
citeturn0search5

### DeployIfNotExists

Can deploy related resources/configuration when the required
resource/configuration doesn't exist, subject to policy requirements.

### AuditIfNotExists

Checks whether a related resource/configuration exists and reports
compliance.

## 277.9 Policy workflow

``` text
Create/choose policy definition
             |
             v
        Assign policy
             |
             v
        Select scope
             |
             v
       Azure evaluates
             |
       +-----+------+
       |            |
    Compliant   Non-compliant
       |            |
       v            v
     Allow      Audit/Deny/
                Modify/etc.
```

## 277.10 Portal lab --- assign a policy

### Step 1 --- Open Azure Policy

In Azure Portal:

``` text
Search
  ↓
Policy
  ↓
Assignments
```

### Step 2 --- Select Assign policy

Choose:

``` text
Assign Policy
```

### Step 3 --- Select Scope

For a lab, use a test resource group.

Example:

``` text
Subscription:
Azure subscription 1

Resource group:
RG-AZ104-Policy
```

### Step 4 --- Select Policy Definition

Search for a built-in policy.

Example:

``` text
Allowed locations
```

### Step 5 --- Configure parameters

Select the allowed location.

Example:

``` text
East US
```

### Step 6 --- Review enforcement

For a real deny policy, enforcement should be understood carefully
before enabling it broadly.

For testing, Microsoft recommends validating policies in a limited scope
and can use enforcement mode `doNotEnforce` where appropriate so the
policy can be evaluated without triggering the policy effect.
citeturn0search4

### Step 7 --- Create assignment

Select:

``` text
Review + create
```

Then:

``` text
Create
```

## 277.11 Check compliance

Go to:

``` text
Azure Portal
  ↓
Policy
  ↓
Compliance
```

You may see:

``` text
Compliant
Non-compliant
Not registered / Not applicable
```

The exact compliance state depends on the policy and evaluation.

## 277.12 Azure CLI --- list policies

``` bash
az policy definition list \
  --output table
```

## 277.13 Find a specific policy

``` bash
az policy definition list \
  --query "[?contains(displayName, 'Allowed locations')].{Name:name,DisplayName:displayName}" \
  --output table
```

## 277.14 Show a policy definition

``` bash
az policy definition show \
  --name <POLICY_DEFINITION_NAME>
```

Azure CLI provides commands for creating, listing, showing, updating,
and deleting policy definitions. citeturn0search3

## 277.15 Assign a policy with Azure CLI

Generic example:

``` bash
az policy assignment create \
  --name allowed-locations \
  --display-name "Allowed locations policy" \
  --policy <POLICY_DEFINITION_ID> \
  --scope <SCOPE>
```

For a resource-group scope:

``` bash
az policy assignment create \
  --name allowed-locations \
  --display-name "Allowed locations policy" \
  --policy <POLICY_DEFINITION_ID> \
  --scope /subscriptions/<SUBSCRIPTION_ID>/resourceGroups/<RESOURCE_GROUP>
```

## 277.16 List policy assignments

``` bash
az policy assignment list \
  --output table
```

## 277.17 Check policy compliance

Portal:

``` text
Policy
  ↓
Compliance
```

CLI can also be used with Azure Policy commands and resource graph/query
tooling depending on the compliance information being retrieved.

## 277.18 Important concept --- policy does not equal RBAC

Example:

``` text
User has Contributor
       |
       v
Tries to create VM
       |
       v
Azure Policy says:
VM SKU not allowed
       |
       v
Deployment denied
```

Contributor permission does not automatically bypass Azure Policy.

## 277.19 Policy assignment inheritance

Example:

``` text
Management Group
       |
       +-- Policy Assignment
       |
       +-- Subscription
              |
              +-- Resource Group
                     |
                     +-- VM
```

The assignment can apply down the hierarchy.

Azure Policy also supports exclusions/exemptions for appropriate
scenarios. citeturn0search0

## 277.20 Best practice

For a new policy:

``` text
Define narrowly
      ↓
Test
      ↓
Audit
      ↓
Review compliance
      ↓
Enable enforcement
      ↓
Monitor continuously
```

Microsoft recommends testing policies on a limited subset before broader
deployment to reduce unintended effects. citeturn0search4

------------------------------------------------------------------------

# 278. Lab --- Azure Policy --- Not Allowed Resource Types

## 278.1 What is "Not allowed resource types"?

Azure has a built-in policy called:

``` text
Not allowed resource types
```

Its purpose is to restrict specified Azure resource types from being
deployed.

Microsoft lists this built-in policy with `Audit`, `Deny`, and
`Disabled` effects. citeturn0search2

## 278.2 Real-world example

Suppose an organization does not allow:

``` text
Microsoft.Network/publicIPAddresses
```

Policy:

``` text
IF resource type = Microsoft.Network/publicIPAddresses
THEN Deny
```

Then a user tries:

``` text
Create Public IP
       |
       v
Azure Policy evaluation
       |
       v
Resource type is prohibited
       |
       v
Deployment denied
```

## 278.3 Why use this policy?

It can help:

-   Reduce unnecessary services.
-   Control resource types.
-   Reduce attack surface.
-   Control costs.
-   Enforce organizational standards.
-   Prevent unauthorized services from being deployed.

## 278.4 Example scenario

Company rule:

``` text
Developers cannot deploy:
Azure Kubernetes Service
```

The administrator can configure a policy that blocks:

``` text
Microsoft.ContainerService/managedClusters
```

at the required scope.

## 278.5 Portal lab

### Step 1 --- Create a test resource group

Example:

``` text
RG-Policy-Lab
```

### Step 2 --- Open Policy

``` text
Azure Portal
   ↓
Policy
   ↓
Assignments
```

### Step 3 --- Assign Policy

Select:

``` text
Assign Policy
```

### Step 4 --- Scope

Select:

``` text
Subscription
  ↓
RG-Policy-Lab
```

For a learning lab, use a test resource group rather than an important
production scope.

### Step 5 --- Policy definition

Search:

``` text
Not allowed resource types
```

Select it.

### Step 6 --- Parameters

Choose the resource type(s) to deny.

Example:

``` text
Microsoft.Network/publicIPAddresses
```

### Step 7 --- Policy effect

Use the appropriate effect available in the selected
definition/assignment.

For a blocking lab, the effective behavior should be:

``` text
Deny
```

### Step 8 --- Create assignment

Select:

``` text
Review + create
```

Then:

``` text
Create
```

## 278.6 Test the policy

Try to create the prohibited resource.

Example:

``` text
Create Public IP
```

Azure Policy evaluates the request.

Expected result:

``` text
Deployment failed
```

A common error is:

``` text
RequestDisallowedByPolicy
```

Microsoft documents this error as occurring when an Azure Policy
prevents a deployment because the resource does not comply with an
assigned policy. citeturn0search14

## 278.7 Understanding the error

You may see something similar to:

``` text
Code:
RequestDisallowedByPolicy

Message:
Resource was disallowed by policy.

Policy assignment:
Not allowed resource types

Policy definition:
Not allowed resource types
```

The important information is:

``` text
Which policy assignment?
Which policy definition?
Which resource was blocked?
```

## 278.8 Troubleshooting RequestDisallowedByPolicy

### Step 1

Read the deployment error.

Look for:

``` text
RequestDisallowedByPolicy
```

### Step 2

Find the policy assignment.

Portal:

``` text
Policy
  ↓
Assignments
```

### Step 3

Open the policy.

Check:

``` text
Scope
Parameters
Exclusions
Effect
```

### Step 4

Check whether another policy is also involved.

Multiple policies can affect the same request.

### Step 5

If the deployment should be allowed, the administrator must review the
policy assignment and its parameters/exclusions rather than simply
changing the user's RBAC role.

## 278.9 Important exam scenario

Question:

> A user has Contributor access but receives `RequestDisallowedByPolicy`
> when creating a resource. What is the likely reason?

Answer:

``` text
An Azure Policy assignment is blocking the resource deployment.
```

Changing the user to Owner does not automatically make a policy-denied
resource deployment valid.

## 278.10 Allowed resource types vs Not allowed resource types

These are easy to confuse.

### Allowed resource types

Concept:

``` text
Only listed resource types are allowed.
```

Example:

``` text
Allowed:
Microsoft.Compute/virtualMachines
Microsoft.Storage/storageAccounts
```

Everything outside the allowed list can be denied by the policy.

### Not allowed resource types

Concept:

``` text
Specified resource types are prohibited.
```

Example:

``` text
Not allowed:
Microsoft.Network/publicIPAddresses
```

Other resource types can remain allowed unless another policy restricts
them.

## 278.11 Policy logic example

### Not allowed

``` text
IF
  type = Microsoft.Network/publicIPAddresses

THEN
  deny
```

### Result

``` text
Create VM
      -> depends on other policies and resources

Create Storage Account
      -> not blocked by this particular policy

Create Public IP
      -> DENIED
```

## 278.12 Existing resources

A common misconception is:

> "If I assign a deny policy, Azure immediately deletes existing
> prohibited resources."

This is **not** how the deny effect works.

A deny policy prevents applicable create/update requests that violate
the policy.

Existing resources may be evaluated for compliance, but the policy does
not automatically delete them.

## 278.13 Policy vs resource lock

  ------------------------------------------------------------------------
  Feature                 Azure Policy             Resource Lock
  ----------------------- ------------------------ -----------------------
  Main purpose            Governance/compliance    Prevent accidental
                                                   modification/deletion

  Can deny resource       Yes, with Deny           No
  creation                                         

  Can control resource    Yes                      No
  configuration                                    

  Can block deletion      Policy can govern        Yes
                          specific                 
                          actions/configurations   
                          depending on effect      

  Example                 Don't allow public IP    Don't delete production
                          resources                VM
  ------------------------------------------------------------------------

## 278.14 Policy vs RBAC

  Feature         RBAC                               Policy
  --------------- ---------------------------------- ------------------------------
  Main question   Who can do it?                     What is allowed/required?
  Example         Contributor can create resources   Public IP not allowed
  Scope           Management group to resource       Management group to resource
  Compliance      Not primary purpose                Core purpose

## 278.15 CLI example --- find the policy

``` bash
az policy definition list \
  --query "[?contains(displayName, 'Not allowed resource types')].{Name:name,DisplayName:displayName}" \
  --output table
```

## 278.16 CLI example --- show policy

``` bash
az policy definition show \
  --name <POLICY_NAME>
```

## 278.17 CLI example --- list assignments

``` bash
az policy assignment list \
  --output table
```

## 278.18 Lab cleanup

After completing the lab:

1.  Remove/delete the test policy assignment.
2.  Verify the policy is no longer enforcing the test restriction.
3.  Delete test resources/resource group if no longer required.
4.  Check Azure costs.

Example:

``` text
Policy
  ↓
Assignments
  ↓
Delete test assignment
```

Do not remove organization-wide policies without authorization.

------------------------------------------------------------------------

# 279. Section Summary

This section brings together the governance topics.

## 279.1 Management Groups

Management groups organize subscriptions.

``` text
Management Group
       ↓
Subscription
       ↓
Resource Group
       ↓
Resource
```

Use them for:

-   Centralized governance
-   Policy
-   RBAC
-   Multi-subscription organization

## 279.2 Azure Policy

Azure Policy evaluates resources against organizational rules.

``` text
Policy Definition
       ↓
Assignment
       ↓
Scope
       ↓
Evaluation
       ↓
Compliance / Effect
```

## 279.3 Policy definition

Defines:

``` text
Condition
+
Effect
```

Example:

``` text
IF
  location != eastus

THEN
  deny
```

## 279.4 Policy assignment

A policy becomes useful at a scope when it is assigned.

``` text
Policy Definition
        |
        v
Policy Assignment
        |
        v
Management Group / Subscription / Resource Group / Resource
```

## 279.5 Policy effects

Remember these common effects:

  Effect              Basic meaning
  ------------------- --------------------------------------------------
  Audit               Report non-compliance
  Deny                Block request
  Modify              Change/add/remove supported properties
  Append              Add supported properties to request
  DeployIfNotExists   Deploy related configuration/resource if missing
  AuditIfNotExists    Audit related configuration/resource if missing
  Disabled            Policy isn't actively enforced

## 279.6 Not allowed resource types

Purpose:

``` text
Prevent specified resource types from being deployed.
```

Example:

``` text
Not allowed:
Microsoft.Network/publicIPAddresses
```

Attempt:

``` text
Create Public IP
```

Result under a Deny assignment:

``` text
RequestDisallowedByPolicy
```

## 279.7 Policy vs RBAC vs Lock

This is one of the most important revision tables.

  Technology         Main question
  ------------------ ---------------------------------------------------
  RBAC               Who can perform an action?
  Azure Policy       What configuration/action is allowed or required?
  Resource Lock      Should a protected resource be changed/deleted?
  Management Group   How do we organize/govern many subscriptions?
  Tags               How do we label and organize resources?

## 279.8 Example enterprise architecture

``` text
Microsoft Entra Tenant
        |
        v
Management Group
        |
        +------------------------+
        |                        |
        v                        v
Production MG             NonProduction MG
        |                        |
        v                        v
Prod Subscription          Dev Subscription
        |                        |
        v                        v
Resource Groups            Resource Groups
        |                        |
        v                        v
Resources                  Resources

Policies:
- Allowed locations
- Not allowed resource types
- Required tags
- Approved SKUs

RBAC:
- Reader
- Contributor
- Owner

Locks:
- CanNotDelete
- ReadOnly

Tags:
- Environment
- Owner
- CostCenter
```

# AZ-104 Exam Questions --- Section 276--279

## Question 1

A company has 50 Azure subscriptions and wants one governance rule to
apply to all production subscriptions. What should the administrator
use?

**Answer:** A management group containing the production subscriptions.

------------------------------------------------------------------------

## Question 2

Which hierarchy is correct?

A. Resource → Resource Group → Subscription → Management Group

B. Management Group → Subscription → Resource Group → Resource

C. Subscription → Management Group → Resource → Resource Group

**Answer:** B.

------------------------------------------------------------------------

## Question 3

What does an Azure Policy definition contain?

**Answer:** Conditions/rules that determine compliance and an effect to
take when the conditions are met.

------------------------------------------------------------------------

## Question 4

A user has Contributor permissions but cannot create a resource and
receives `RequestDisallowedByPolicy`. What should you investigate?

**Answer:** Azure Policy assignments affecting the deployment
scope/resource.

------------------------------------------------------------------------

## Question 5

Which Azure Policy effect blocks a matching resource request?

**Answer:** Deny.

------------------------------------------------------------------------

## Question 6

What is the purpose of the "Not allowed resource types" policy?

**Answer:** To prevent specified resource types from being deployed.

------------------------------------------------------------------------

## Question 7

Does a Deny policy delete existing resources that violate the policy?

**Answer:** No. It prevents applicable create/update requests that
violate the policy; existing resources can be evaluated for compliance.

------------------------------------------------------------------------

## Question 8

Which technology answers "Who can perform this operation?"

**Answer:** Azure RBAC.

------------------------------------------------------------------------

## Question 9

Which technology answers "Are resources configured according to
organizational standards?"

**Answer:** Azure Policy.

------------------------------------------------------------------------

## Question 10

What is the main purpose of a management group?

**Answer:** Organizing and governing multiple Azure subscriptions.

------------------------------------------------------------------------

# Quick Revision Sheet

## Management Groups

``` text
Management Group
       ↓
Subscription
       ↓
Resource Group
       ↓
Resource
```

## Azure Policy

``` text
Definition
    ↓
Assignment
    ↓
Scope
    ↓
Evaluation
    ↓
Effect
```

## Policy Effects

``` text
Audit       = Report
Deny        = Block
Modify      = Change
Append      = Add
DeployIfNotExists = Deploy if missing
AuditIfNotExists  = Audit if missing
```

## Not Allowed Resource Types

``` text
Specified resource type
        ↓
Policy evaluation
        ↓
Deny
        ↓
RequestDisallowedByPolicy
```

## RBAC vs Policy

``` text
RBAC
"What can this identity do?"

Policy
"What resource configurations are allowed?"
```

## Final AZ-104 Memory Trick

``` text
MG = Many subscriptions
Policy = Rules
Assignment = Applies the rule
Deny = Blocks
Audit = Reports
RBAC = Permissions
Lock = Protection
Tag = Metadata
```

------------------------------------------------------------------------

# Practical Lab Checklist

Use a **test resource group** for these exercises.

-   [ ] Create a test resource group.
-   [ ] Open Azure Policy.
-   [ ] Explore Definitions.
-   [ ] Find `Not allowed resource types`.
-   [ ] Create a test policy assignment.
-   [ ] Select a resource type to deny.
-   [ ] Try creating that resource.
-   [ ] Observe `RequestDisallowedByPolicy`.
-   [ ] Open Policy → Compliance.
-   [ ] Inspect the policy assignment.
-   [ ] Remove the test assignment.
-   [ ] Verify the restriction is gone.
-   [ ] Review management-group hierarchy.
-   [ ] Review RBAC vs Policy vs Locks.
-   [ ] Delete unused lab resources to avoid unnecessary charges.

------------------------------------------------------------------------

# Official Microsoft Learn References

-   Azure Policy overview:
    https://learn.microsoft.com/en-us/azure/governance/policy/overview
-   Azure Policy documentation:
    https://learn.microsoft.com/en-us/azure/governance/policy/
-   Assign Azure Policy in the portal:
    https://learn.microsoft.com/en-us/azure/governance/policy/assign-policy-portal
-   Built-in Azure Policy definitions:
    https://learn.microsoft.com/en-us/azure/governance/policy/samples/built-in-policies
-   Azure Policy definition structure:
    https://learn.microsoft.com/en-us/azure/governance/policy/concepts/definition-structure
-   Evaluate the impact of a new policy:
    https://learn.microsoft.com/en-us/azure/governance/policy/concepts/evaluate-impact
-   Resolve RequestDisallowedByPolicy:
    https://learn.microsoft.com/en-us/azure/azure-resource-manager/troubleshooting/error-policy-requestdisallowedbypolicy
-   Azure management groups:
    https://learn.microsoft.com/en-us/azure/governance/management-groups/
