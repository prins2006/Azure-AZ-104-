# Azure Networking — Network Security Groups (NSG) and Application Security Groups (ASG)

## AZ-104 Exam Preparation + Practical Labs + Production Scenarios

---

# Table of Contents

1. Introduction to Network Security Groups
2. How NSGs Work
3. Inbound and Outbound Rules
4. Lab 82 — NSG Getting Started
5. Lab 83 — Understanding Rule Priorities
6. Lab 84 — Inter-VM Communication
7. Lab 85 — Outbound Rules
8. Lab 86 — ICMP Protocol
9. Lab 87 — Two-Tier Architecture Setup
10. Lab 88 — Two-Tier Architecture Implementation
11. Lab 89 — NSGs at Subnet Level
12. Lab 90 — NSG on Subnet and Network Interface
13. Lab 91 — Application Security Groups
14. NSG vs ASG vs Azure Firewall
15. Troubleshooting Practical Scenarios
16. Production Best Practices
17. AZ-104 Exam Questions
18. Hands-On Practice Tasks
19. Quick Revision Cheat Sheet

---

# 1. Introduction to Network Security Groups

## 1.1 What is a Network Security Group?

A Network Security Group (NSG) is an Azure networking resource used to control network traffic entering and leaving Azure virtual networks.

An NSG contains security rules that allow or deny traffic based on:

- Source IP address or source network.
- Destination IP address or destination network.
- Source port.
- Destination port.
- Protocol: TCP, UDP, ICMP, or Any.
- Direction: Inbound or Outbound.
- Priority: The order in which rules are evaluated.

### Real-world example

Suppose you have an Azure virtual machine running Ubuntu with Nginx installed.

Your VM has a public IP address, and you want users to access your website through HTTP.

You want the following behavior:

- Allow HTTP traffic on port 80.
- Allow HTTPS traffic on port 443.
- Allow SSH on port 22 only from your trusted public IP.
- Block other unwanted inbound traffic.

An NSG helps implement these network access rules.

| Traffic | Port | Desired action |
|---|---:|---|
| HTTP | 80/TCP | Allow |
| HTTPS | 443/TCP | Allow |
| SSH from trusted IP | 22/TCP | Allow |
| SSH from other IPs | 22/TCP | Deny |
| Unrequested inbound traffic | Any | Usually denied by default rules, subject to applicable higher-priority rules |

**Important:** An NSG filters network traffic. It does not install software, patch a VM, or replace authentication and operating-system firewalls.

## 1.2 Where can an NSG be associated?

An NSG can be associated with:

1. A subnet in an Azure virtual network.
2. A network interface (NIC) attached to an Azure virtual machine.

It is not directly attached to the VM's operating system.

Example:

    Virtual Network: vnet-prod
    |
    +-- Subnet: web-subnet
    |   |
    |   +-- NSG: nsg-web-subnet
    |   +-- VM: web-vm
    |
    +-- Subnet: app-subnet
    |   |
    |   +-- NSG: nsg-app-subnet
    |   +-- VM: app-vm

The subnet-level NSG applies to traffic crossing the subnet boundary. The NIC-level NSG applies to traffic at that network interface.

---

# 2. How Network Security Groups Work

## 2.1 Basic traffic flow

When a client connects to an Azure VM, network traffic must pass through the relevant Azure network security controls.

For example, an internet user accesses an Nginx website hosted on an Azure VM.

    Internet User
          |
          | HTTP Request: TCP/80
          v
    Azure Public IP
          |
          v
    Network Security Rules
          |
          v
    VM Network Interface
          |
          v
    Ubuntu Operating System
          |
          v
    Nginx Web Server
          |
          v
    Website Response

The exact network path depends on the architecture, but this diagram illustrates the main concept.

The website works only if the relevant network controls allow the connection and the application is correctly configured.

## 2.2 NSG rules contain these properties

| Property | Meaning | Example |
|---|---|---|
| Name | Name of the rule | Allow-HTTP |
| Priority | Evaluation order | 200 |
| Source | Where traffic originates | Internet |
| Source port | Originating port | * |
| Destination | Target of traffic | Any |
| Destination port | Service port | 80 |
| Protocol | Traffic protocol | TCP |
| Direction | Inbound or Outbound | Inbound |
| Action | Allow or Deny | Allow |

The asterisk (`*`) means any port in the applicable port field.

For a typical inbound HTTP rule, the source port is `*`, while the destination port is `80`.

## 2.3 Default NSG rules

Azure creates default security rules when an NSG is created.

Common default inbound rules include:

| Priority | Rule | Action |
|---:|---|---|
| 65000 | AllowVnetInBound | Allow |
| 65001 | AllowAzureLoadBalancerInBound | Allow |
| 65500 | DenyAllInBound | Deny |

Common default outbound rules include:

| Priority | Rule | Action |
|---:|---|---|
| 65000 | AllowVnetOutBound | Allow |
| 65001 | AllowInternetOutBound | Allow |
| 65500 | DenyAllOutBound | Deny |

These are simplified names for the default rules.

**Important:** Azure evaluates lower numerical priorities first. Custom rules with priorities from 100 to 4096 are commonly used, while default rules have priorities of 65000 or higher.

The default rules cannot be deleted or modified, but custom rules can override them by using a higher evaluation precedence (a lower numerical priority).

### Example

Suppose the default `AllowVnetInBound` rule permits traffic from a virtual network.

You create a custom rule:

    Priority: 200
    Source: 10.10.1.0/24
    Destination: Any
    Destination port: 22
    Protocol: TCP
    Action: Deny

For matching SSH traffic from that subnet, the custom deny rule is evaluated before the default allow rule.

## 2.4 NSGs are stateful

An NSG is stateful. This means that when an allowed connection is established, response traffic for that connection is automatically permitted by the NSG.

Example:

    Client 10.10.1.10
          |
          | TCP request to port 80
          v
    Web VM 10.10.2.10
          |
          | TCP response
          v
    Client 10.10.1.10

If the inbound request is allowed and the connection is established, you do not normally need a separate inbound rule for the return packets.

However, this does not mean all new connections in the reverse direction are allowed. A new connection is evaluated against the applicable security rules.

---

# 3. Inbound and Outbound Rules

## 3.1 Inbound rules

Inbound rules control new traffic entering the resource or subnet protected by the NSG.

Examples:

- Internet users accessing an Nginx website.
- An administrator connecting through SSH.
- A load balancer sending requests to a backend VM.
- A frontend VM connecting to a backend application on port 5000.

Example inbound HTTP rule:

    Direction: Inbound
    Priority: 200
    Source: Internet
    Source port ranges: *
    Destination: Any
    Destination port ranges: 80
    Protocol: TCP
    Action: Allow

This rule allows matching inbound TCP traffic on destination port 80, subject to other applicable network controls.

## 3.2 Outbound rules

Outbound rules control new traffic leaving the resource or subnet protected by the NSG.

Examples:

- A VM downloading packages from Ubuntu repositories.
- An application connecting to an external API.
- A backend VM connecting to a database.
- A VM initiating an internet connection.

Example outbound rule:

    Direction: Outbound
    Priority: 300
    Source: Any
    Source port ranges: *
    Destination: 10.10.3.0/24
    Destination port ranges: 3306
    Protocol: TCP
    Action: Allow

This permits matching outbound TCP traffic to destination port 3306 in the specified network, subject to other applicable controls.

## 3.3 Source port vs destination port

This distinction is important for the AZ-104 exam.

When a client accesses a website:

    Client: 192.0.2.10:51520
              |
              | TCP
              v
    Web Server: 10.10.1.10:80

- Source IP: `192.0.2.10`
- Source port: `51520`
- Destination IP: `10.10.1.10`
- Destination port: `80`

The client usually uses a temporary source port, while the web server listens on its service port.

Therefore, the normal inbound HTTP rule uses:

    Source port: *
    Destination port: 80

## 3.4 Common service ports

| Service | Protocol | Port |
|---|---|---:|
| SSH | TCP | 22 |
| HTTP | TCP | 80 |
| HTTPS | TCP | 443 |
| DNS | UDP/TCP | 53 |
| RDP | TCP | 3389 |
| MySQL | TCP | 3306 |
| PostgreSQL | TCP | 5432 |
| MongoDB | TCP | 27017 |
| Custom Node.js application | TCP | 3000 |
| Custom API | TCP | 5000 |

Ports depend on the application's configuration. These are common defaults, not mandatory values.

---

# 4. Lab 82 — Network Security Groups: Getting Started

## Objective

Create an NSG and understand how security rules affect VM connectivity.

## Example architecture

    Internet
       |
       v
    Public IP
       |
       v
    NSG: nsg-web
       |
       v
    Ubuntu VM
       |
       v
    Nginx: TCP/80

## Step 1: Create an NSG

In the Azure portal:

1. Search for **Network security groups**.
2. Select **Create**.
3. Choose your subscription.
4. Select your resource group.
5. Enter the NSG name: `nsg-web`.
6. Choose the appropriate region.
7. Select **Review + create**.
8. Select **Create**.

## Step 2: Associate the NSG

Open `nsg-web`.

Choose either:

- **Subnets** to associate the NSG with a subnet.
- **Network interfaces** to associate the NSG with a VM's NIC.

Select the appropriate resource and save the association.

## Step 3: Add an inbound HTTP rule

Navigate to **Inbound security rules** and add:

    Source: Any
    Source port ranges: *
    Destination: Any
    Service: HTTP
    Destination port ranges: 80
    Protocol: TCP
    Action: Allow
    Priority: 200
    Name: Allow-HTTP

For a real production website, consider whether the source should be restricted or whether the website should be exposed through a load balancer, application gateway, or another controlled entry point.

## Step 4: Test Nginx

On the Ubuntu VM:

    sudo apt update
    sudo apt install nginx -y
    sudo systemctl enable --now nginx
    sudo systemctl status nginx

Check the local web server:

    curl http://localhost

Expected result: HTML from the Nginx welcome page.

From your client machine, browse to:

    http://<VM-PUBLIC-IP>

Replace `<VM-PUBLIC-IP>` with the actual public IP of your VM.

If the page does not load, check the NSG, VM public IP, routing, Nginx status, and operating-system firewall.

**Security warning:** Do not expose SSH to the entire internet just to make a lab easier. Use Azure Bastion, a VPN, or a trusted source IP where possible.

---

# 5. Lab 83 — Network Security Groups: Understanding Priorities

## 5.1 What is rule priority?

Priority determines the order in which NSG security rules are evaluated.

A lower number means a higher priority.

Example:

| Priority | Rule | Action |
|---:|---|---|
| 100 | Deny SSH from Internet | Deny |
| 200 | Allow SSH from trusted IP | Allow |
| 300 | Allow HTTP | Allow |
| 65500 | Default deny inbound | Deny |

Rules are evaluated from the lowest numerical priority upward.

When a rule matches the traffic, its action is applied. Azure does not continue to a lower-priority rule to reverse that decision.

## 5.2 Example: Allow and deny rules conflict

Suppose you configure:

    Rule 1:
    Priority: 100
    Source: Internet
    Destination port: 22
    Action: Deny

    Rule 2:
    Priority: 200
    Source: 203.0.113.10/32
    Destination port: 22
    Action: Allow

Both rules match SSH traffic from `203.0.113.10` because that IP is part of the Internet source range.

Since priority 100 is evaluated first, SSH from that IP is denied.

The more specific allow rule never gets the chance to permit that traffic.

### Correct configuration

    Rule 1:
    Priority: 100
    Source: 203.0.113.10/32
    Destination port: 22
    Action: Allow

    Rule 2:
    Priority: 200
    Source: Internet
    Destination port: 22
    Action: Deny

Now the trusted IP is allowed, and other matching Internet SSH traffic is denied.

Use the actual public IP of your trusted administrator rather than the documentation-only example IP above.

## 5.3 Priority rules to remember

- Lower numerical value = higher priority.
- The same NSG cannot have two custom rules with the same priority.
- Custom NSG rules have priorities between 100 and 4096.
- Default rules have higher numerical values than custom rules.
- More-specific rules are not automatically evaluated before less-specific rules; priority controls evaluation.
- Inbound and outbound rules are evaluated separately.

## 5.4 Practical exercise

Create these rules:

| Priority | Source | Destination port | Action |
|---:|---|---:|---|
| 100 | Your trusted public IP `/32` | 22 | Allow |
| 200 | Internet | 22 | Deny |
| 300 | Internet | 80 | Allow |
| 310 | Internet | 443 | Allow |

For each rule, choose TCP unless your application requires a different protocol.

Test SSH from the trusted IP, HTTP access, and HTTPS access after configuring TLS.

---

# 6. Lab 84 — Network Security Groups: Inter-VM Communication

## Objective

Understand how NSGs control traffic between VMs in a virtual network.

## Example architecture

    Virtual Network: 10.10.0.0/16
    |
    +-- Web subnet: 10.10.1.0/24
    |      |
    |      +-- web-vm: 10.10.1.10
    |
    +-- App subnet: 10.10.2.0/24
           |
           +-- app-vm: 10.10.2.10

Assume the web application on `web-vm` needs to communicate with `app-vm` on TCP port 5000.

## 6.1 Required network rules

At the application VM's subnet or NIC, allow inbound TCP port 5000 from the web subnet:

    Direction: Inbound
    Source: 10.10.1.0/24
    Destination: 10.10.2.10
    Destination port: 5000
    Protocol: TCP
    Action: Allow
    Priority: 200

If a subnet-level NSG and NIC-level NSG both apply, both must allow the flow.

Also confirm that the application listens on an address reachable from the other VM, rather than only on `127.0.0.1`.

## 6.2 Test connectivity

From the web VM:

    ping 10.10.2.10

This tests ICMP reachability only if ICMP is allowed and supported along the path.

For TCP port 5000, use:

    nc -vz 10.10.2.10 5000

If `nc` is not installed on Ubuntu:

    sudo apt update
    sudo apt install netcat-openbsd -y

Check the application listener on the app VM:

    sudo ss -lntp

Look for port 5000.

## 6.3 Why might VM-to-VM communication fail?

Possible causes:

1. The NSG blocks the traffic.
2. The application is not running.
3. The application listens only on localhost.
4. The destination port is incorrect.
5. The operating-system firewall blocks the connection.
6. The VMs are in different VNets without working connectivity.
7. A route table or another network control changes the traffic path.
8. A NIC-level NSG denies the traffic even though the subnet NSG allows it.

**Production principle:** Allow only the required source network, destination, and port. Avoid opening all ports between application tiers.

---

# 7. Lab 85 — Network Security Groups: Outbound Rules

## 7.1 Why outbound rules matter

Outbound rules determine which new connections a VM or subnet can initiate.

For example, an Ubuntu VM may need to download packages:

    Ubuntu VM
       |
       | Outbound TCP/443
       v
    Ubuntu Package Repository

If the required outbound traffic is denied, commands such as `apt update` may fail.

## 7.2 Example: Allow HTTPS outbound

    Direction: Outbound
    Priority: 200
    Source: Any
    Source port ranges: *
    Destination: Internet
    Destination port ranges: 443
    Protocol: TCP
    Action: Allow

This rule permits matching outbound HTTPS traffic, subject to the other applicable controls.

If a higher-priority deny rule matches, the connection will still be blocked.

## 7.3 Example: Restrict a backend VM

Suppose the backend VM should communicate with a database but should not initiate arbitrary internet connections.

A possible policy is:

    Allow backend -> database on TCP/3306
    Deny backend -> Internet for other prohibited destinations

Carefully design rule priority, required platform dependencies, DNS, package repositories, monitoring, and management access before implementing restrictive outbound policies.

A blanket deny rule can unintentionally break required services.

## 7.4 Test outbound connectivity

Check DNS resolution:

    nslookup example.com

Check HTTPS connectivity:

    curl -I https://example.com

Check package repositories:

    sudo apt update

A failure does not automatically mean the NSG is responsible. DNS configuration, routing, proxy settings, firewall rules, and internet egress configuration can also cause failures.

## 7.5 Important Azure egress consideration

Do not assume that a VM always has internet access just because an NSG permits outbound traffic.

Internet access also depends on the network's egress configuration, public IP or NAT configuration where applicable, routes, and Azure platform behavior. Explicit outbound connectivity design is important in production.

---

# 8. Lab 86 — Network Security Groups: ICMP Protocol

## 8.1 What is ICMP?

ICMP stands for Internet Control Message Protocol.

It is used for network diagnostic and error messages.

Common examples include:

- Echo request.
- Echo reply.
- Destination unreachable.
- Time exceeded.

The `ping` command commonly uses ICMP Echo Request and Echo Reply messages.

Example:

    ping 10.10.2.10

If a reply is received, the destination responded to the ICMP echo request.

However, a failed ping does not necessarily mean the VM is offline.

## 8.2 Does ICMP use TCP or UDP ports?

No. ICMP does not use TCP or UDP ports.

For example:

    TCP -> Port 22, 80, 443
    UDP -> Port 53, 123
    ICMP -> No TCP/UDP port

Therefore, allowing TCP port 22 does not permit ICMP ping traffic.

## 8.3 Configure an ICMP rule

Create an inbound rule on the destination VM's NSG:

    Direction: Inbound
    Source: 10.10.1.0/24
    Destination: Any
    Protocol: ICMP
    Action: Allow
    Priority: 200

If the Azure portal offers protocol selections such as TCP, UDP, ICMP, and Any, select ICMP.

The exact protocol options can depend on the interface or deployment method. Where the portal or rule API does not expose the needed ICMP option, verify the available rule configuration instead of trying to substitute a TCP port.

## 8.4 Test ICMP

From the source VM:

    ping 10.10.2.10

If the ping fails, check:

1. The destination NSG's inbound rules.
2. The source NSG's outbound rules.
3. The destination operating-system firewall.
4. The destination IP address.
5. Network routing and connectivity.
6. Whether the destination supports and responds to ICMP.

**Interview point:** Ping tests ICMP reachability, not whether an application is listening on TCP port 80 or 443.

A server can block ping and still serve a website successfully.

---

# 9. Lab 87 — Two-Tier Architecture: Setup

## 9.1 What is a two-tier architecture?

A two-tier architecture separates an application into two logical layers.

For a common web application, these are:

1. Web/frontend tier.
2. Application/backend tier.

The frontend receives user requests and communicates with the backend to process application logic.

Example:

    User
      |
      | HTTP/HTTPS
      v
    Web Tier
    10.10.1.0/24
      |
      | Application TCP/5000
      v
    Application Tier
    10.10.2.0/24

This example has two application tiers. A database could be added as a separate third tier.

## 9.2 Why separate the tiers?

Security:

- Users access the public-facing frontend.
- The backend is not directly exposed to the internet.
- Only the required frontend-to-backend traffic is permitted.

Operations:

- Frontend and backend can scale independently.
- Different configurations can be applied to each tier.
- Network access is easier to audit.

## 9.3 Suggested lab resources

Create the following resources:

| Resource | Example |
|---|---|
| Virtual network | `vnet-app` |
| VNet address space | `10.10.0.0/16` |
| Web subnet | `10.10.1.0/24` |
| App subnet | `10.10.2.0/24` |
| Web VM | `web-vm` |
| App VM | `app-vm` |
| Web NSG | `nsg-web` |
| App NSG | `nsg-app` |

These are example addresses and names. Check for address-space conflicts before creating resources.

## 9.4 Security design

Web tier:

    Allow inbound HTTP/HTTPS as required.
    Restrict administration access.
    Do not expose backend-only ports to the Internet.

Application tier:

    Allow inbound application traffic only from the web tier.
    Do not expose the application port publicly.
    Restrict administration access through a controlled management path.

A subnet-level NSG is a convenient place to implement rules for the entire tier. A NIC-level NSG can add additional restrictions for an individual VM.

---

# 10. Lab 88 — Two-Tier Architecture: Implementation

## Objective

Implement communication between a web tier and an application tier using NSGs.

## 10.1 Example traffic requirements

Assume:

- Web VM: `10.10.1.10`
- App VM: `10.10.2.10`
- Nginx: TCP/80
- Backend API: TCP/5000

Required flows:

| Source | Destination | Port | Action |
|---|---|---:|---|
| Internet | Web VM | TCP/80 | Allow if public HTTP is required |
| Internet | Web VM | TCP/443 | Allow if public HTTPS is required |
| Web subnet | App VM | TCP/5000 | Allow |
| Internet | App VM | TCP/5000 | Deny |
| Untrusted sources | VM SSH | TCP/22 | Deny |

For SSH, a higher-priority deny rule can override a broad allow rule. Configure trusted administration access carefully.

## 10.2 Configure the web NSG

Add the necessary inbound web rules.

For example:

    Allow-HTTP
    Direction: Inbound
    Source: Internet
    Destination port: 80/TCP
    Action: Allow
    Priority: 200

If HTTPS is configured:

    Allow-HTTPS
    Direction: Inbound
    Source: Internet
    Destination port: 443/TCP
    Action: Allow
    Priority: 210

Allowing port 443 does not configure TLS. You must also configure a certificate and HTTPS listener.

## 10.3 Configure the application NSG

Create an inbound rule:

    Name: Allow-Web-to-App
    Direction: Inbound
    Priority: 200
    Source: 10.10.1.0/24
    Source port ranges: *
    Destination: 10.10.2.10
    Destination port ranges: 5000
    Protocol: TCP
    Action: Allow

Do not create a broad inbound rule allowing TCP/5000 from the Internet.

## 10.4 Configure the backend application

A backend service must listen on a reachable interface.

For example, if a Node.js service listens only on:

    127.0.0.1:5000

then other VMs cannot connect to it using the VM's private IP.

The service may need to listen on the VM's private interface or on `0.0.0.0`, subject to appropriate host firewall and application security controls.

Check listeners:

    sudo ss -lntp

Example output:

    LISTEN 0 511 0.0.0.0:5000 0.0.0.0:*

This indicates a listener on all IPv4 interfaces, not just localhost.

**Security warning:** Listening on `0.0.0.0` does not itself make the service publicly accessible, but it makes it available on all local IPv4 interfaces. Use network and host firewall rules to control access.

## 10.5 Test the architecture

From the web VM:

    nc -vz 10.10.2.10 5000

If the TCP connection succeeds, the network path and service listener are likely functioning.

If it fails, check:

- Backend application status.
- Listening address and port.
- App subnet NSG.
- App NIC NSG.
- Web subnet or NIC outbound rules.
- Operating-system firewall.
- Effective routes and the VM's private IP.

## 10.6 Production architecture improvement

A production design may use:

    Internet
       |
       v
    Application Gateway / Load Balancer
       |
       v
    Web or frontend tier
       |
       v
    Private application tier
       |
       v
    Private database tier

Application Gateway is a Layer 7 web traffic service with features such as HTTP routing and optional Web Application Firewall capabilities. A Load Balancer primarily distributes network traffic at Layer 4.

Use the service that matches the application and security requirements.

---

# 11. Lab 89 — Network Security Groups: Subnets

## 11.1 What is subnet-level NSG association?

A subnet-level NSG applies security rules to traffic crossing the subnet boundary.

For example:

    VNet: 10.10.0.0/16
    |
    +-- web-subnet: 10.10.1.0/24
    |      |
    |      +-- web-vm-1
    |      +-- web-vm-2
    |
    +-- app-subnet: 10.10.2.0/24
           |
           +-- app-vm-1
           +-- app-vm-2

If `nsg-web` is associated with `web-subnet`, its rules can protect the applicable traffic for resources in that subnet.

This is useful when several VMs should follow the same network access policy.

## 11.2 Benefits of subnet-level NSGs

- Centralized control for a group of VMs.
- Consistent security policies.
- Easier administration as VMs are added.
- Suitable for web, application, and database subnet boundaries.
- Reduced need to configure the same rules independently on each VM.

## 11.3 Example: Application subnet

Suppose the application subnet is:

    10.10.2.0/24

You want to allow application traffic only from the web subnet:

    Source: 10.10.1.0/24
    Destination: 10.10.2.0/24
    Protocol: TCP
    Destination port: 5000
    Action: Allow

This policy should be implemented on the destination-side security controls as an inbound rule, with matching outbound permissions on the source-side controls where necessary.

## 11.4 Important subnet rule concept

A subnet-level NSG does not automatically mean all traffic from one VM to another VM in the same subnet is blocked.

By default, Azure's NSGs include an `AllowVnetInBound` rule, which permits matching virtual-network traffic unless a higher-priority rule denies it.

To enforce stricter segmentation within a subnet, create appropriate custom rules and test the effective policy.

---

# 12. Lab 90 — NSG on Subnet and Network Interface

## 12.1 Difference between subnet and NIC association

| Feature | Subnet-level NSG | NIC-level NSG |
|---|---|---|
| Associated with | Subnet | Network interface |
| Scope | Resources using the subnet | Traffic for that NIC |
| Best use | Shared tier-wide policies | VM-specific restrictions |
| Applies to every VM in subnet? | Yes, for applicable subnet traffic | No |
| Can both be used? | Yes | Yes |

## 12.2 What happens if both NSGs are applied?

For a typical new connection between two VMs, the applicable outbound and inbound rules must permit the traffic.

For example:

    Web VM
       |
       | Outbound security evaluation
       v
    Web subnet NSG / Web NIC NSG
       |
       v
    App subnet NSG / App NIC NSG
       |
       v
    App VM

At the destination, inbound traffic must be permitted by the applicable subnet and NIC NSGs.

At the source, outbound traffic must be permitted by the applicable subnet and NIC NSGs.

If any required NSG denies the flow, the connection will not work.

### Example

Web VM subnet NSG:

    Allow outbound TCP/5000 to 10.10.2.0/24

Web VM NIC NSG:

    Deny outbound TCP/5000 to 10.10.2.0/24

App VM subnet NSG:

    Allow inbound TCP/5000 from 10.10.1.0/24

Even though the destination subnet allows the traffic, the source NIC denies it. The connection fails.

## 12.3 Effective security rules

When troubleshooting, inspect effective security rules for the VM's NIC in the Azure portal.

Effective rules help you understand the security policy resulting from applicable NSG associations and other supported network security configurations.

Also inspect effective routes when routing may be involved.

Useful Azure CLI commands:

    az network nic list-effective-nsg \
      --resource-group <RESOURCE-GROUP> \
      --name <NIC-NAME>

This command retrieves effective NSG information for the specified NIC, subject to your permissions and the resource configuration.

To inspect NIC details:

    az network nic show \
      --resource-group <RESOURCE-GROUP> \
      --name <NIC-NAME> \
      --output table

Replace the placeholders with actual resource names.

---

# 13. Lab 91 — Application Security Groups

## 13.1 What is an Application Security Group?

An Application Security Group (ASG) allows you to group VM network interfaces logically based on their application role.

Instead of repeatedly using individual IP addresses in NSG rules, you can reference an ASG as the source or destination in supported NSG rules.

Examples:

- `asg-web`
- `asg-app`
- `asg-db`

ASGs simplify the management of application-oriented network rules within supported Azure virtual network configurations.

**Important:** An ASG is not a firewall by itself. It is a logical grouping used in supported NSG rules.

## 13.2 Example without ASGs

Suppose your application has three frontend VMs:

    web-vm-1: 10.10.1.10
    web-vm-2: 10.10.1.11
    web-vm-3: 10.10.1.12

The backend runs on TCP port 5000.

Without ASGs, you might have to maintain a list of source IP addresses in an NSG rule.

As VMs are added or their private IP addresses change, the rule may require maintenance.

## 13.3 Example with ASGs

Create:

    asg-web
    asg-app

Add the web VM NICs to `asg-web`.

Add the application VM NICs to `asg-app`.

Then create an NSG rule:

    Name: Allow-Web-to-App
    Direction: Inbound
    Source: asg-web
    Destination: asg-app
    Protocol: TCP
    Destination port: 5000
    Action: Allow
    Priority: 200

This represents a logical rule permitting matching traffic from NICs in the web ASG to NICs in the app ASG.

The referenced ASGs must be configured correctly, and the NICs must be eligible for the ASG and rule combination.

## 13.4 How to create an ASG

In the Azure portal:

1. Search for **Application security groups**.
2. Select **Create**.
3. Choose the subscription and resource group.
4. Choose the region.
5. Enter a name such as `asg-web`.
6. Select **Review + create**.
7. Create the ASG.
8. Open the ASG and associate the relevant VM network interfaces.
9. Open the relevant NSG.
10. Create or update an NSG rule to use the ASG as its source or destination.

ASG and NSG resources must meet the supported regional and virtual network requirements for the intended configuration.

## 13.5 Why use ASGs?

Suppose your application grows:

    Initially:
    2 web VMs
    2 app VMs

    Later:
    10 web VMs
    8 app VMs

With ASGs, the NSG rule can continue to reference the logical groups, while you manage group membership as network interfaces are added or removed.

This makes the policy easier to maintain.

## 13.6 ASG limitations and considerations

- An ASG is not a standalone security enforcement point.
- ASGs are used in supported NSG security rule configurations.
- They are associated with network interfaces, not directly with an application process.
- They do not replace subnet design, route configuration, or firewall policy.
- ASGs do not automatically inspect HTTP requests or provide a Web Application Firewall.
- ASGs are not a substitute for identity-based application authorization.
- ASG-based rules must satisfy Azure's supported network and resource constraints.

---

# 14. NSG vs ASG vs Azure Firewall

These services solve different problems.

| Feature | NSG | ASG | Azure Firewall |
|---|---|---|---|
| Main purpose | Allow or deny network flows | Group NICs by application role | Centralized network traffic filtering |
| Is it a security enforcement point? | Yes | No, used through NSG rules | Yes |
| Contains allow/deny rules? | Yes | No independent traffic rules | Yes |
| Can filter by ports and protocols? | Yes | No independent filtering | Yes |
| Logical grouping of application NICs? | No | Yes | Not its primary purpose |
| Centralized network firewall features? | Limited to NSG functionality | No | Yes |
| HTTP application-layer inspection? | No | No | Available through supported features, but not equivalent to an application-specific WAF |
| Common use | Subnet and NIC filtering | Readable, role-based NSG rules | Centralized egress, ingress, and network filtering |

## Example decision

Use an NSG when you need to allow TCP/443 to a web VM.

Use an ASG when you need a rule that allows traffic from the web application NIC group to the backend application NIC group.

Use Azure Firewall when your design requires centralized network firewall policy and controlled traffic paths.

Use Azure Application Gateway with WAF when you need supported web application-layer protection against HTTP attacks.

---

# 15. Troubleshooting Practical Scenarios

## Scenario 1: SSH is not working

Command:

    ssh azureuser@<VM-PUBLIC-IP>

Check:

1. Is the VM running?
2. Is the public IP correct?
3. Is TCP/22 allowed by the applicable NSGs?
4. Is the SSH daemon running?
5. Is the OS firewall allowing SSH?
6. Is the SSH key correct?
7. Is a route or other network control blocking the path?

On Ubuntu:

    sudo systemctl status ssh
    sudo ss -lntp | grep ':22'

Do not open SSH to the entire Internet as the default fix.

## Scenario 2: Nginx is installed but the website is not accessible

Check the service:

    sudo systemctl status nginx

Check the listener:

    sudo ss -lntp | grep ':80'

Test locally:

    curl -I http://localhost

Then inspect the NSG rules, VM public IP, routing, host firewall, and application gateway or load balancer configuration if present.

## Scenario 3: Web VM cannot connect to the backend

Test:

    nc -vz 10.10.2.10 5000

Check:

- Backend service is running.
- Backend listens on the correct interface.
- Inbound NSG rule permits TCP/5000.
- Outbound NSG rules permit the flow.
- NIC-level NSGs do not block it.
- The source and destination addresses are correct.
- The route between the subnets is valid.
- The OS firewall allows the connection.

## Scenario 4: Ping fails but the website works

This can be normal.

Ping uses ICMP, while a website normally uses TCP/80 or TCP/443.

The network policy might allow web traffic but deny ICMP.

Do not conclude that the server is down based on ping alone.

## Scenario 5: A VM cannot download packages

Run:

    sudo apt update
    curl -I https://example.com
    nslookup example.com

Check outbound NSG rules, DNS, routing, NAT or public egress configuration, and host firewall settings.

## Scenario 6: A rule allows traffic, but the connection still fails

Remember that the full path matters.

For a VM-to-VM connection, inspect:

1. Source NIC and subnet NSGs.
2. Destination NIC and subnet NSGs.
3. Routing.
4. Application listener.
5. Operating-system firewall.
6. DNS resolution if using hostnames.
7. Other network security controls in the path.

Use effective NSG rules and effective routes to help isolate the issue.

---

# 16. Production Best Practices

## 16.1 Apply least privilege

Allow only the traffic that is actually required.

Prefer:

    Source: Web subnet
    Destination: App subnet
    Port: 5000/TCP
    Action: Allow

Avoid:

    Source: Any
    Destination: Any
    Ports: *
    Protocol: Any
    Action: Allow

## 16.2 Restrict administrative access

For SSH and RDP:

- Prefer Azure Bastion, VPN, or a controlled management network.
- If public administrative access is required, restrict it to trusted source IP addresses where feasible.
- Use strong authentication and keys.
- Avoid unnecessary public IP addresses on VMs.

## 16.3 Separate application tiers

Keep backend and database services off direct public access unless the architecture explicitly requires it.

Allow only the necessary application flows between tiers.

## 16.4 Document rules

Use descriptive rule names:

    Allow-Trusted-SSH
    Allow-Internet-HTTPS
    Allow-Web-to-App
    Deny-Untrusted-SSH

Document the purpose, owner, source, destination, port, and expected expiration of temporary rules.

## 16.5 Test before enforcing restrictive outbound rules

A broad outbound deny can disrupt:

- Package updates.
- Monitoring agents.
- DNS resolution.
- Required APIs.
- Management and deployment services.

Test in a controlled environment before applying the policy to production.

## 16.6 Monitor and review

Regularly review unused rules, overly broad source ranges, public exposure, and changes to application dependencies.

Use Azure monitoring and network troubleshooting tools where appropriate.

---

# 17. AZ-104 Exam Questions

## Question 1

You want to allow SSH only from your trusted public IP. Which source should you configure?

A. Internet  
B. Any  
C. Your trusted public IP with `/32`  
D. VirtualNetwork

**Answer: C**

Explanation: A `/32` represents one IPv4 address. It limits the source to that single address.

## Question 2

Which rule is evaluated first: priority 100 or priority 300?

**Answer:** Priority 100.

Explanation: A lower numerical priority is evaluated earlier.

## Question 3

An NSG allows TCP port 80 but ping fails. Is the website necessarily unavailable?

**Answer:** No.

Explanation: TCP/80 and ICMP are different protocols. Ping can fail while HTTP works.

## Question 4

A subnet NSG allows traffic, but a NIC NSG denies it. Will the connection work?

**Answer:** No.

Explanation: Both applicable NSGs must permit the required flow.

## Question 5

Which Azure feature groups VM NICs by application role for use in NSG rules?

A. Azure Firewall  
B. Application Security Groups  
C. Route tables  
D. Public IP addresses

**Answer: B**

Explanation: ASGs allow NSG rules to reference logical groups of network interfaces.

## Question 6

Where can an NSG be associated?

A. Only a virtual machine's operating system  
B. Only a public IP  
C. A subnet or network interface  
D. Only a storage account

**Answer: C**

## Question 7

A web VM needs to access a backend on TCP/5000. Which rule is the most restrictive suitable option?

A. Allow all ports from Internet  
B. Allow TCP/5000 from the web subnet  
C. Allow all protocols from Any  
D. Allow TCP/22 from Internet

**Answer: B**

## Question 8

What does the default `DenyAllInBound` rule do?

**Answer:** It denies inbound traffic that has not already matched a higher-precedence rule.

## Question 9

What does NSG statefulness mean?

**Answer:** Return traffic for an allowed established connection is automatically permitted by the NSG. It does not permit all new connections in the reverse direction.

## Question 10

A VM has both a subnet-level NSG and a NIC-level NSG. The subnet NSG allows TCP/443, but the NIC NSG denies the same inbound flow. What happens?

**Answer:** The flow is blocked because the NIC-level NSG denies it.

---

# 18. Hands-On Practice Tasks

Complete these tasks in your own Azure lab.

## Task 1 — Basic web security

- [ ] Create an NSG named `nsg-web`.
- [ ] Associate it with your web subnet or NIC.
- [ ] Allow inbound TCP/80.
- [ ] Install and start Nginx.
- [ ] Test using the VM's public IP, if configured.

## Task 2 — Restrict SSH

- [ ] Allow TCP/22 from your trusted public IP.
- [ ] Add a lower-precedence deny rule for other Internet SSH traffic.
- [ ] Confirm the trusted connection still works.
- [ ] Keep a recovery path available before changing remote-access rules.

## Task 3 — Inter-VM communication

- [ ] Create two VMs in the same VNet.
- [ ] Run a service on TCP/5000 on the destination VM.
- [ ] Allow traffic from the source VM or source subnet.
- [ ] Test using `nc -vz`.

## Task 4 — Test ICMP

- [ ] Add an appropriate ICMP rule.
- [ ] Run `ping` between the VMs.
- [ ] Remove or disable the test rule when no longer required.

## Task 5 — Two-tier architecture

- [ ] Create a web subnet.
- [ ] Create an application subnet.
- [ ] Configure the web tier to reach the application tier.
- [ ] Deny direct Internet access to the backend application port.
- [ ] Verify the expected connections and blocked connections.

## Task 6 — Subnet vs NIC NSG

- [ ] Apply an NSG to the subnet.
- [ ] Apply a second NSG to a NIC.
- [ ] Test a connection permitted by one NSG but denied by the other.
- [ ] Inspect effective security rules.

## Task 7 — Application Security Groups

- [ ] Create `asg-web`.
- [ ] Create `asg-app`.
- [ ] Add the appropriate NICs to each ASG.
- [ ] Create a rule allowing `asg-web` to reach `asg-app` on TCP/5000.
- [ ] Test the application connection.

---

# 19. Quick Revision Cheat Sheet

| Topic | Remember |
|---|---|
| NSG | Filters network traffic |
| Inbound rule | Controls incoming new traffic |
| Outbound rule | Controls outgoing new traffic |
| Priority | Lower number = evaluated first |
| Stateful behavior | Allows return traffic for an allowed established connection |
| TCP/22 | Common SSH port |
| TCP/80 | Common HTTP port |
| TCP/443 | Common HTTPS port |
| ICMP | Used by ping; no TCP/UDP port |
| Subnet NSG | Shared policy for applicable subnet traffic |
| NIC NSG | Policy for applicable traffic at a specific NIC |
| Both NSGs present | Both must permit the required flow |
| ASG | Logical group of NICs referenced in supported NSG rules |
| Default inbound deny | Denies unmatched inbound traffic |
| Effective NSG | Helps inspect the applied security rules |
| Least privilege | Permit only the required sources, destinations, and ports |

---

# Final Interview Summary

**Network Security Group (NSG):** An Azure resource that filters inbound and outbound network traffic using rules based on source, destination, port, protocol, direction, priority, and action.

**Application Security Group (ASG):** A logical grouping of network interfaces that makes NSG rules easier to manage by application role instead of maintaining individual IP addresses.

**Subnet-level NSG:** Applies security rules to applicable traffic crossing a subnet boundary.

**NIC-level NSG:** Applies security rules to applicable traffic for an individual network interface.

**Rule priority:** Lower numerical priorities are evaluated first. A matching deny rule can prevent traffic even if a later rule allows it.

**Two-tier architecture:** Separates the web tier from the application tier and permits only the required communication between them.

**Production principle:** Design security around least privilege, test effective rules and routing, and never assume that one successful allow rule guarantees end-to-end connectivity.
