# AZ-104 — Microsoft Entra External Identities, Licenses & SSPR

## Topics Covered

1. Microsoft Entra ID — Inviting external identities
2. Microsoft Entra ID Licenses
3. Assigning licenses to users
4. Assigning licenses to groups
5. Further points on group-based licensing
6. What is self-service password reset?
7. Enabling self-service password reset

---

# 263. Microsoft Entra ID — Inviting External Identities

## 263.1 What are External Identities?

External identities are users who are outside your organization's Microsoft Entra tenant but need access to your organization's applications or resources.

Common examples:

- Customer
- Vendor
- Consultant
- Partner
- Contractor
- External developer

Microsoft Entra B2B collaboration allows an organization to collaborate with external users while allowing them to use their existing identity.

### Simple example

```text
Your Organization
       |
       | Invite
       v
External Partner
       |
       v
Guest User
       |
       v
Application / Resource
```

---

## 263.2 Microsoft Entra B2B Collaboration

B2B means:

> Business-to-Business collaboration

It allows your organization to collaborate with external users.

Example:

```text
Contoso Tenant
       |
       +-- Employee
       |
       +-- Guest
              |
              +-- External Partner
```

The external user can generally use their existing work, school, or supported identity instead of your organization managing a separate password for them.

---

## 263.3 Guest User

After invitation, the external identity is represented in your directory as a user object whose user type is typically:

```text
Guest
```

The external user must redeem the invitation before accessing the shared resources.

You may see an external user's UPN containing:

```text
#EXT#
```

Example:

```text
rahul_partner.com#EXT#@contoso.onmicrosoft.com
```

The exact generated value depends on the tenant and identity.

---

## 263.4 Why Use Guest Users?

Without B2B:

```text
External User
      |
      v
Create separate local account
      |
      v
Manage another identity
```

With B2B:

```text
External User
      |
      v
Invite
      |
      v
Use existing identity
      |
      v
Access shared resource
```

---

## 263.5 How to Invite an External User

Typical portal flow:

```text
Microsoft Entra admin center
        |
        v
Microsoft Entra ID
        |
        v
Users
        |
        v
New user
        |
        v
Invite external user
```

Enter:

- Email
- Display name
- Optional invitation message

Then select:

```text
Review + invite
```

The external user receives an invitation.

---

## 263.6 Invitation Flow

```text
Administrator
     |
     | Invite
     v
External User
     |
     | Receives invitation
     v
Accept invitation
     |
     v
Authentication
     |
     v
Guest user
     |
     v
Access resource
```

---

## 263.7 External User and Groups

An external guest can be added to a group.

Example:

```text
External User
      |
      v
Guest
      |
      v
Project-External-Users
      |
      v
Application / Resource
```

This can simplify permission management.

---

## 263.8 External User and Applications

You can provide an external user access to a specific application.

Example:

```text
External Partner
       |
       v
Guest User
       |
       v
Sales Application
```

The user does not necessarily need access to everything in the tenant.

This follows the principle of:

> Least privilege

---

## 263.9 Exam Questions

### Question 1

A company needs to allow an external partner to access an application.

**Answer:** Microsoft Entra B2B collaboration.

### Question 2

An external user is invited through B2B collaboration. What type of user is commonly created?

**Answer:** Guest.

### Question 3

Does B2B necessarily require the organization to create and manage the external user's original password?

**Answer:** No. B2B collaboration can allow the external user to use their existing identity.

---

# 264. Microsoft Entra ID Licenses

## 264.1 What is Licensing?

A Microsoft Entra license determines which features and capabilities are available to users and organizations.

Common Microsoft Entra editions include:

```text
Microsoft Entra ID Free
Microsoft Entra ID P1
Microsoft Entra ID P2
```

Always verify current Microsoft licensing documentation when checking exact feature availability because licensing can change.

---

## 264.2 Microsoft Entra ID Free

The Free edition provides basic identity capabilities.

Examples include:

- Users
- Groups
- Basic identity management
- Basic authentication capabilities
- Security defaults

---

## 264.3 Microsoft Entra ID P1

P1 provides additional enterprise identity capabilities.

Important examples include:

- Conditional Access
- More advanced authentication controls
- Self-service password reset capabilities
- Hybrid identity features

For SSPR, verify the exact licensing requirements for the scenario.

---

## 264.4 Microsoft Entra ID P2

P2 provides additional identity protection and governance capabilities.

Examples include:

- Risk-based identity protection
- Privileged Identity Management
- Advanced identity governance capabilities

---

## 264.5 License Comparison

| Feature | Free | P1 | P2 |
|---|---|---|---|
| Basic users/groups | Yes | Yes | Yes |
| Basic identity management | Yes | Yes | Yes |
| Conditional Access | No | Yes | Yes |
| Advanced identity protection | Limited | More | Advanced |
| Privileged Identity Management | No | No | Yes |
| Advanced governance | Limited | More | Advanced |
| SSPR | Scenario dependent | Yes | Yes |

> Always check current Microsoft documentation for exact licensing requirements for a particular feature.

---

## 264.6 Why Licenses Matter

Example:

```text
User
 |
 v
Microsoft Entra ID P1
 |
 v
P1 features become available
```

Without the required license:

```text
User
 |
 X
Required feature unavailable
```

---

## 264.7 License Assignment

Licenses can generally be assigned:

```text
Directly to users
```

or:

```text
Through groups
```

The second approach is called:

> Group-based licensing

---

# 265. Assigning Licenses to Users

## 265.1 Direct License Assignment

A license can be assigned directly to a user.

Example:

```text
Prins
  |
  v
Microsoft Entra ID P1
```

---

## 265.2 Portal Flow

Typical flow:

```text
Microsoft Entra admin center
       |
       v
Users
       |
       v
All users
       |
       v
Select user
       |
       v
Licenses
       |
       v
Assignments
```

Select the required license and assign it.

---

## 265.3 Example

```text
User:
prins@example.com

License:
Microsoft Entra ID P1
```

Result:

```text
prins@example.com
       |
       v
Microsoft Entra ID P1
```

---

## 265.4 Usage Location

License assignment can require the user's **Usage location** to be set.

Example:

```text
User
 |
 +-- Usage location: India
 |
 +-- License
```

Make sure the user's location is correctly configured before assigning licenses.

---

## 265.5 Why Direct Assignment Can Become Difficult

Imagine:

```text
500 users
```

and licenses are assigned individually:

```text
User 1 -> License
User 2 -> License
User 3 -> License
...
User 500 -> License
```

This becomes difficult to manage.

Group-based licensing can simplify this.

---

# 266. Assigning Licenses to Groups

## 266.1 What is Group-Based Licensing?

Instead of assigning a license individually to every user, assign the license to a group.

Example:

```text
Azure-Developers
       |
       v
Microsoft Entra ID P1
```

Users who are members of the group can receive the assigned license.

---

## 266.2 Example

Suppose:

```text
Azure-Developers
|
+-- Prins
+-- Rahul
+-- Amit
+-- Neha
```

Assign:

```text
Microsoft Entra ID P1
```

to:

```text
Azure-Developers
```

Conceptually:

```text
Azure-Developers
       |
       v
Microsoft Entra ID P1
       |
       +-- Prins
       +-- Rahul
       +-- Amit
       +-- Neha
```

---

## 266.3 Why Group-Based Licensing?

It makes license management easier.

Instead of:

```text
Add license
Add license
Add license
Add license
```

you manage:

```text
Group membership
```

---

## 266.4 Portal Flow

Typical flow:

```text
Microsoft Entra admin center
       |
       v
Groups
       |
       v
Select group
       |
       v
Licenses
       |
       v
Assignments
       |
       v
Select license
```

Assign the required license.

---

## 266.5 New User Example

Suppose a new employee joins:

```text
Neha
```

Add Neha to:

```text
Azure-Developers
```

If the group has a license assigned:

```text
Azure-Developers
       |
       v
Microsoft Entra ID P1
```

Neha can receive the license through group-based licensing.

---

## 266.6 Employee Leaves

Suppose Neha leaves the team.

Remove her from:

```text
Azure-Developers
```

The group-based license assignment is no longer applicable to her through that group.

---

# 267. Further Points on Group-Based Licensing

## 267.1 Group-Based Licensing Flow

```text
Group
  |
  v
License assigned
  |
  v
Users become members
  |
  v
License applied to users
```

---

## 267.2 Multiple Groups

A user can be a member of multiple groups.

Example:

```text
Prins
 |
 +-- Developers
 |      |
 |      +-- License A
 |
 +-- Security-Team
        |
        +-- License B
```

The user may receive licenses from both group assignments.

---

## 267.3 Direct + Group Assignment

A user can have:

```text
Direct license
```

and:

```text
Group-based license
```

The administrator should understand where each license comes from.

---

## 267.4 License Assignment Errors

A group-based license assignment can fail for individual users.

Possible causes include:

- Missing usage location
- Insufficient available licenses
- Conflicting service plans
- Unsupported configuration

Check the user's licensing status and error details.

---

## 267.5 License Availability

Suppose:

```text
Purchased: 100 licenses
Assigned: 100 licenses
Available: 0
```

A new assignment may fail because there are no available licenses.

---

## 267.6 Group-Based Licensing and Dynamic Groups

Group-based licensing can be useful with dynamic groups.

Example:

```text
Dynamic Group
     |
     | Rule:
     | Department = Development
     v
Development Users
     |
     v
License
```

When a user's attributes cause them to enter the group, the group-based license can apply.

---

## 267.7 Important Exam Concept

### Direct Assignment

```text
User
  |
  v
License
```

### Group-Based Assignment

```text
Group
  |
  v
License
  |
  v
Members
```

---

## 267.8 Practical Scenario

### Requirement

A company has 100 developers and all developers need the same Microsoft Entra license.

### Approach

Create:

```text
Developers
```

group.

Assign the license to:

```text
Developers
```

Then manage group membership.

---

# 268. What is Self-Service Password Reset?

## 268.1 SSPR Meaning

SSPR stands for:

> **Self-Service Password Reset**

It allows users to reset or change their password without requiring the help desk for every password problem.

---

## 268.2 Problem Without SSPR

```text
User forgets password
        |
        v
Contact Helpdesk
        |
        v
Helpdesk verifies user
        |
        v
Password reset
```

---

## 268.3 With SSPR

```text
User forgets password
        |
        v
SSPR
        |
        v
Verify identity
        |
        v
Create new password
        |
        v
Access account
```

---

## 268.4 Authentication Verification

Before resetting a password, Microsoft Entra verifies the user's identity using configured authentication methods.

Examples can include:

- Microsoft Authenticator
- SMS
- Voice
- Email OTP
- OATH methods

The exact methods available depend on tenant configuration.

---

## 268.5 SSPR Example

A user forgets their password.

Open:

```text
https://aka.ms/sspr
```

Then:

```text
Enter username
      |
      v
Verify identity
      |
      v
Enter new password
      |
      v
Password changed
```

---

## 268.6 Why SSPR Helps

Without SSPR:

```text
Many password problems
        |
        v
Helpdesk requests
```

With SSPR:

```text
Many password problems
        |
        v
Self-service reset
```

This can reduce helpdesk dependency.

---

## 268.7 SSPR Licensing

Licensing depends on the exact SSPR scenario.

For AZ-104, remember:

```text
SSPR
 |
 v
Check required license
 |
 v
Configure users/groups
 |
 v
Configure authentication methods
```

For hybrid password writeback scenarios, additional licensing requirements apply.

---

# 269. Enabling Self-Service Password Reset

## 269.1 SSPR Configuration

Typical portal path:

```text
Microsoft Entra admin center
       |
       v
Microsoft Entra ID
       |
       v
Protection
       |
       v
Password reset
```

SSPR can be configured for selected users/groups or all users, depending on the tenant configuration.

---

## 269.2 Enable for Selected Users

For a lab, start with a test group.

Example:

```text
SSPR-Test-Group
```

Members:

```text
Prins
Rahul
```

Conceptual configuration:

```text
Password reset
      |
      v
Properties
      |
      v
Selected
      |
      v
SSPR-Test-Group
```

Save the configuration.

---

## 269.3 Enable for All Users

For organization-wide deployment:

```text
Password reset
      |
      v
Properties
      |
      v
All
```

In production, test SSPR with a pilot group before broad deployment.

---

## 269.4 Configure Authentication Methods

Configure the authentication methods that users can use.

Examples:

```text
Microsoft Authenticator
SMS
Email
```

Also configure how many authentication methods are required, according to the organization's policy.

---

## 269.5 User Registration

Users need to register their authentication information.

Conceptually:

```text
User
 |
 v
SSPR Registration
 |
 +-- Authenticator
 +-- Phone
 +-- Email
```

The registration process helps ensure the user has a usable authentication method when a password reset is needed.

---

## 269.6 Test SSPR

Create or use a test user.

Check:

```text
License
   +
SSPR enabled
   +
User included in scope
   +
Authentication methods registered
```

Then open:

```text
https://aka.ms/sspr
```

Test:

```text
Forgot password
      |
      v
Verify identity
      |
      v
Set new password
      |
      v
Sign in
```

---

## 269.7 SSPR for Hybrid Users

In a hybrid environment:

```text
On-Premises Active Directory
          |
          v
Microsoft Entra Connect
          |
          v
Microsoft Entra ID
```

SSPR can be configured with **password writeback**, allowing supported cloud password changes/resets to be written back to on-premises Active Directory.

Conceptually:

```text
User
 |
 v
SSPR
 |
 v
Microsoft Entra ID
 |
 v
Password Writeback
 |
 v
On-Premises Active Directory
```

Password writeback has additional licensing and configuration requirements.

---

## 269.8 SSPR Administrator Permissions

SSPR configuration should use an appropriate least-privileged Microsoft Entra administrative role where possible.

For current Microsoft guidance, check the specific SSPR configuration documentation and required administrative role.

---

## 269.9 SSPR Troubleshooting

If a user cannot use SSPR, check:

```text
1. Is SSPR enabled?
2. Is the user included in the selected group?
3. Does the user have the required license?
4. Has the user registered authentication methods?
5. Are the configured authentication methods available?
6. Is the password managed in the expected location?
7. Is password writeback configured for hybrid scenarios?
```

---

# AZ-104 Exam Revision

## External Identities

```text
External user
      |
      v
B2B Collaboration
      |
      v
Guest
```

---

## Licensing

### Direct

```text
User
 |
 v
License
```

### Group-Based

```text
Group
 |
 v
License
 |
 v
Members
```

---

## SSPR

```text
Forgot password
      |
      v
SSPR
      |
      v
Verify identity
      |
      v
New password
```

---

## Hybrid SSPR

```text
SSPR
 |
 v
Microsoft Entra ID
 |
 v
Password Writeback
 |
 v
On-Premises AD
```

---

# Important Differences

| Topic | Main Concept |
|---|---|
| External identities | Collaborate with users outside your organization |
| B2B | External/guest collaboration |
| Entra licensing | Controls access to licensed features |
| Direct licensing | License assigned directly to user |
| Group-based licensing | License assigned to group and applied to members |
| SSPR | Users reset/change passwords themselves |
| SSPR authentication | Verifies user before reset |
| Password writeback | Writes supported password changes back to on-premises AD |

---

# Scenario-Based AZ-104 Questions

## Question 1

A company wants an external consultant to access an internal application without creating a normal employee account.

**Answer:** Microsoft Entra B2B collaboration.

---

## Question 2

An external B2B user is invited to the tenant. What user type is commonly used?

**Answer:** Guest.

---

## Question 3

100 developers require the same Microsoft Entra license.

**Answer:** Group-based licensing can simplify management.

---

## Question 4

A company assigns a license directly to a user.

What type of assignment is this?

**Answer:** Direct user-based license assignment.

---

## Question 5

A user forgets their Microsoft Entra password and needs to reset it without contacting the helpdesk.

**Answer:** Self-Service Password Reset (SSPR).

---

## Question 6

An administrator wants to test SSPR with only a few users before enabling it for everyone.

**Answer:** Enable SSPR for a selected test group.

---

## Question 7

An SSPR user cannot reset their password.

What should you check?

**Answer:**

- SSPR enabled?
- Correct user/group scope?
- Required license?
- Authentication methods registered?

---

## Question 8

A hybrid user needs password reset changes written back to on-premises Active Directory.

**Answer:** Password writeback.

---

# Final Revision Diagram

```text
                   MICROSOFT ENTRA ID
                          |
        +-----------------+-----------------+
        |                 |                 |
        v                 v                 v
External Users       Licensing             SSPR
        |                 |                 |
        v                 v                 v
      B2B           Direct / Group      Password Reset
        |            Assignment              |
        v                 |                  v
      Guest               v            Authentication
                         License             |
                                             v
                                      New Password
```

---

# Final Memory Tricks

## External Identities

> **External user -> B2B -> Guest**

## Licensing

> **User -> Direct license**

> **Group -> License -> Members**

## SSPR

> **Forgot password -> Verify identity -> Reset password**

## Hybrid SSPR

> **SSPR -> Password Writeback -> On-Premises AD**

---

# Most Important AZ-104 Points

Remember:

1. B2B = external collaboration.
2. External B2B users are commonly Guest users.
3. Licenses can be assigned directly to users.
4. Licenses can be assigned to groups.
5. Group-based licensing simplifies large-scale license management.
6. SSPR allows users to reset/change passwords without helpdesk involvement.
7. SSPR requires appropriate licensing for the scenario.
8. SSPR can be enabled for selected users/groups or all users.
9. Users need appropriate authentication methods registered.
10. Password writeback is used for supported hybrid scenarios.
11. When troubleshooting SSPR, check license, scope, registration, authentication methods, and hybrid configuration.

---

# One-Line Revision

```text
B2B        -> External Users
Licensing  -> User / Group
SSPR       -> Self Password Reset
Writeback  -> Cloud -> On-Premises AD
```

# Microsoft Learn

- Microsoft Entra B2B collaboration: https://learn.microsoft.com/en-us/entra/external-id/what-is-b2b
- Microsoft Entra licensing: https://learn.microsoft.com/en-us/entra/fundamentals/licensing
- SSPR licensing: https://learn.microsoft.com/en-us/entra/identity/authentication/concept-sspr-licensing
- How SSPR works: https://learn.microsoft.com/en-us/entra/identity/authentication/concept-sspr-howitworks
- Enable SSPR: https://learn.microsoft.com/en-us/entra/identity/authentication/tutorial-enable-sspr
