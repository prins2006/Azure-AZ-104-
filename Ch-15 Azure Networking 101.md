# Azure Networking 101 — Complete Notes (Topics 71–79)

## Table of Contents

1. [Networking Basics](#71-networking-basics)
2. [IP Addresses](#72-ip-addresses)
3. [Subnet Mask and CIDR Blocks](#73-subnet-mask-and-cidr-blocks)
4. [Introduction to Azure Virtual Networks](#74-introduction-to-azure-virtual-networks)
5. [Lab: Building an Azure Virtual Network](#75-lab-building-an-azure-virtual-network)
6. [Understanding Subnets](#76-understanding-subnets)
7. [Lab: Deploying an Azure VM](#77-lab-deploying-an-azure-vm-to-the-network)
8. [Lab: Deploying Another VM to Another Subnet](#78-lab-deploying-another-vm-to-another-subnet)
9. [Understanding IP Addresses in Azure](#79-understanding-ip-addresses-in-azure)
10. [AZ-104 Practice Questions](#az-104-practice-questions)

---

# 71. Networking Basics

## 1. What Is a Computer Network?

A computer network is a group of devices connected to communicate and exchange data.

Examples:

* Computers and laptops
* Servers and virtual machines
* Routers and switches
* Firewalls
* Printers
* Mobile devices

### Real-World Example

When you open a website, your computer sends a request to a web server. The server processes the request and sends the website's data back to your computer.

```text
Client
  |
  | HTTP/HTTPS Request
  v
Network
  |
  v
Web Server
  |
  | HTTP/HTTPS Response
  v
Client
```

## 2. Important Networking Devices

| Device        | Purpose                                 | Example                    |
| ------------- | --------------------------------------- | -------------------------- |
| NIC           | Connects a device to a network          | Azure VM network interface |
| Switch        | Connects devices in a local network     | Office LAN                 |
| Router        | Forwards traffic between networks       | Home router                |
| Firewall      | Allows or blocks network traffic        | Azure Firewall             |
| Load Balancer | Distributes traffic among servers       | Multiple web VMs           |
| DNS Server    | Resolves domain names to IP addresses   | Resolving `example.com`    |
| Gateway       | Provides connectivity to other networks | Azure VPN Gateway          |

## 3. What Is an IP Address?

An IP address is a logical address used to identify a network interface and route traffic.

Example:

```text
Laptop IP: 192.168.1.10
Server IP: 192.168.1.20
```

If both devices are on a reachable network and security rules permit traffic, they can communicate.

## 4. What Is a Port?

A port identifies a service or application endpoint on a device.

| Port | Service    | Purpose                |
| ---- | ---------- | ---------------------- |
| 22   | SSH        | Linux remote access    |
| 53   | DNS        | Name resolution        |
| 80   | HTTP       | Web traffic            |
| 443  | HTTPS      | Encrypted web traffic  |
| 3306 | MySQL      | Database communication |
| 5432 | PostgreSQL | Database communication |
| 3389 | RDP        | Windows remote desktop |

Example:

```text
IP Address: 10.0.1.4
Port: 22
Protocol: TCP
```

This represents an attempt to connect to the SSH service at `10.0.1.4`.

## 5. What Is a Network Protocol?

A protocol defines rules for communication between devices.

* **IP:** Provides addressing and packet routing.
* **TCP:** Provides reliable, ordered delivery.
* **UDP:** Sends datagrams without TCP's delivery guarantees.
* **HTTP/HTTPS:** Used for web communication.
* **SSH:** Secure remote command-line access.
* **DNS:** Resolves domain names to IP addresses.

## 6. How Networking Works in Azure

When you create an Azure virtual machine:

1. You select a virtual network and subnet.
2. Azure connects the VM through a virtual network interface (NIC).
3. The NIC receives a private IP address.
4. Azure networking routes traffic between reachable destinations.
5. Network Security Groups and other security controls determine whether traffic is allowed.
6. A public IP can provide an internet-facing endpoint when configured.

**Important:** An IP address alone does not guarantee connectivity. Routing, NSGs, firewalls, and the destination service must also be configured correctly.

---

# 72. IP Addresses

## 1. What Is IPv4?

IPv4 uses a 32-bit address represented by four decimal octets.

Example:

```text
192.168.1.10
```

Each octet contains 8 bits and ranges from 0 to 255.

```text
192       .168       .1         .10
11000000  .10101000  .00000001  .00001010
```

Total IPv4 address space:

```text
2^32 = 4,294,967,296 possible addresses
```

Some addresses are reserved for private networks, loopback, documentation, multicast, and other special purposes.

## 2. Private IP Addresses

Private IP addresses are used within private networks and are not directly routable across the public internet.

The three private IPv4 ranges are:

| Private Range                     | CIDR             |
| --------------------------------- | ---------------- |
| `10.0.0.0` – `10.255.255.255`     | `10.0.0.0/8`     |
| `172.16.0.0` – `172.31.255.255`   | `172.16.0.0/12`  |
| `192.168.0.0` – `192.168.255.255` | `192.168.0.0/16` |

Example:

```text
10.0.1.4
```

This can be the private IP address of an Azure VM.

## 3. Public IP Addresses

A public IP address is internet-routable and can be associated with supported Azure resources.

Example:

```text
Public IP: 20.50.60.70
```

The address above is illustrative.

A public IP does not automatically make every service accessible. NSGs, firewalls, routing, and service configuration still control connectivity.

## 4. Private IP vs Public IP

| Feature                 | Private IP                                     | Public IP                                       |
| ----------------------- | ---------------------------------------------- | ----------------------------------------------- |
| Main purpose            | Private network communication                  | Internet-facing connectivity                    |
| Example                 | `10.0.1.4`                                     | An allocated Azure public IP                    |
| Direct internet routing | Not directly routable over the public internet | Internet-routable                               |
| Typical use             | Backend VM or database                         | Public frontend endpoint                        |
| Security requirements   | Still requires security controls               | Requires careful exposure and security controls |

## 5. Static IP vs Dynamic IP

### Dynamic Private IP

Azure automatically allocates an available IP address from the subnet.

### Static Private IP

You specify an available address in the subnet.

Example:

```text
VNet: 10.0.0.0/16
Subnet: 10.0.1.0/24
VM private IP: 10.0.1.10
```

Static IPs are useful when other systems depend on a predictable private address.

Azure's dynamic allocation method can retain an address through some VM stop/start operations. Do not assume it changes every time the VM restarts; allocation behavior depends on the resource and operation.

## 6. What Is IPv6?

IPv6 uses 128-bit addresses.

Example:

```text
2001:db8:abcd:1::10
```

IPv6 provides a much larger address space than IPv4.

## 7. What Is a Loopback Address?

The IPv4 loopback address is:

```text
127.0.0.1
```

It refers to the local machine.

Example:

```bash
curl http://127.0.0.1:3000
```

This tests a service running locally on port 3000. It does not prove that another computer can reach the service.

## 8. Linux Commands for IP Addresses

### Display network interfaces

```bash
ip addr
```

### Display routing information

```bash
ip route
```

### Display assigned IP addresses

```bash
hostname -I
```

### Display the neighbor table

```bash
ip neigh
```

These commands are useful when troubleshooting connectivity inside Azure Linux VMs.

---

# 73. Subnet Mask and CIDR Blocks

## 1. What Is a Subnet?

A subnet is a smaller logical network created from a larger IP address range.

Example:

```text
VNet: 10.0.0.0/16

Web Subnet: 10.0.1.0/24
App Subnet: 10.0.2.0/24
DB Subnet:  10.0.3.0/24
```

Each subnet can contain resources serving different purposes.

## 2. What Is a Subnet Mask?

A subnet mask identifies which bits belong to the network prefix and which bits belong to the host portion.

Example:

```text
IP Address:  192.168.1.10
Subnet Mask: 255.255.255.0
CIDR:        /24
```

The first 24 bits are the network prefix. The remaining 8 bits represent the address portion within the subnet.

## 3. What Does CIDR Mean?

CIDR stands for Classless Inter-Domain Routing.

Example:

```text
10.0.1.0/24
```

Here:

* `10.0.1.0` is the network address.
* `/24` means 24 bits form the network prefix.
* The remaining 8 bits determine the total address range.

## 4. CIDR Reference Table

| CIDR  | Subnet Mask       | Total IPv4 Addresses | Azure-Usable IPv4 Addresses |
| ----- | ----------------- | -------------------: | --------------------------: |
| `/16` | `255.255.0.0`     |               65,536 |                      65,531 |
| `/20` | `255.255.240.0`   |                4,096 |                       4,091 |
| `/24` | `255.255.255.0`   |                  256 |                         251 |
| `/25` | `255.255.255.128` |                  128 |                         123 |
| `/26` | `255.255.255.192` |                   64 |                          59 |
| `/27` | `255.255.255.224` |                   32 |                          27 |
| `/28` | `255.255.255.240` |                   16 |                          11 |

Azure reserves five IPv4 addresses in each subnet. The usable counts shown are address capacity, not a guarantee that all addresses are currently available.

## 5. CIDR Calculation Formula

For IPv4:

```text
Total addresses = 2^(32 - prefix length)
```

Example: `/24`

```text
2^(32 - 24)
= 2^8
= 256 total addresses
```

Example: `/26`

```text
2^(32 - 26)
= 2^6
= 64 total addresses
```

For a typical Azure IPv4 subnet:

```text
Azure-usable addresses = Total addresses - 5
```

Therefore, a `/26` subnet has:

```text
64 - 5 = 59 Azure-usable IPv4 addresses
```

## 6. CIDR Range Example

Consider:

```text
192.168.1.0/26
```

A `/26` contains 64 addresses.

| Subnet             | Address Range   |
| ------------------ | --------------- |
| `192.168.1.0/26`   | `.0` – `.63`    |
| `192.168.1.64/26`  | `.64` – `.127`  |
| `192.168.1.128/26` | `.128` – `.191` |
| `192.168.1.192/26` | `.192` – `.255` |

Each subnet contains 64 total addresses.

## 7. Important CIDR Rules

* A smaller prefix number means a larger address range.
* A larger prefix number means a smaller address range.
* Subnets in the same VNet must not overlap.
* A subnet must fit inside the VNet address space.
* Leave sufficient address capacity for future resources.
* Plan non-overlapping address ranges when connecting VNets.

---

# 74. Introduction to Azure Virtual Networks

## 1. What Is an Azure VNet?

An Azure Virtual Network (VNet) is a logically isolated network in Azure that enables resources to communicate through private IP addresses and configured network connectivity.

A VNet is similar to a private network in an organization's infrastructure.

Example:

```text
Azure Subscription
└── Resource Group
    └── VNet: 10.0.0.0/16
        ├── Web Subnet: 10.0.1.0/24
        │   └── Web VM
        ├── App Subnet: 10.0.2.0/24
        │   └── App VM
        └── DB Subnet: 10.0.3.0/24
            └── Database VM
```

## 2. Important Azure Networking Components

| Component     | Purpose                                         |
| ------------- | ----------------------------------------------- |
| VNet          | Defines the Azure network and address space     |
| Subnet        | Divides a VNet into smaller ranges              |
| NIC           | Connects a VM to a subnet                       |
| Private IP    | Supports private network communication          |
| Public IP     | Provides an internet-routable endpoint          |
| NSG           | Filters inbound and outbound traffic            |
| Route Table   | Defines custom routes                           |
| VNet Peering  | Connects VNets through Azure networking         |
| VPN Gateway   | Provides supported encrypted VPN connectivity   |
| Azure Bastion | Provides browser-based management access to VMs |

## 3. How Two VMs Communicate

Example:

```text
VM1: 10.0.1.4
VM2: 10.0.2.4
```

Both VMs belong to the same VNet.

Communication flow:

1. VM1 sends traffic to VM2's private IP.
2. Azure evaluates the applicable routes.
3. Applicable NSG rules and other network controls determine whether traffic is permitted.
4. The destination VM receives the traffic if the network and guest configuration allow it.
5. The application processes the request if it is listening on the destination port.

Azure normally provides the required default routes between subnets in the same VNet.

## 4. Can Two Subnets Communicate Automatically?

Default VNet routing supports communication between subnets.

However, traffic can fail because of:

* NSG rules
* Operating-system firewalls
* Custom routes
* Incorrect IP addresses
* Services not listening on the required ports

Two subnets in the same VNet do not normally require VNet peering.

## 5. Traditional Networking vs Azure Networking

| Traditional Networking    | Azure Networking                              |
| ------------------------- | --------------------------------------------- |
| Physical network          | Virtual network                               |
| Physical NIC              | Virtual NIC                                   |
| LAN subnet                | Azure subnet                                  |
| Physical router           | Azure-managed routing infrastructure          |
| Firewall                  | NSGs, Azure Firewall, and other controls      |
| Office network connection | VPN Gateway or supported private connectivity |

---

# 75. Lab: Building an Azure Virtual Network

## 1. Lab Objective

Create a VNet and two subnets using Azure Portal or Azure CLI.

## 2. Lab Configuration

| Setting            | Value            |
| ------------------ | ---------------- |
| Resource Group     | `rg-network-lab` |
| VNet Name          | `vnet-prins-lab` |
| Region             | `centralindia`   |
| VNet Address Space | `10.0.0.0/16`    |
| Web Subnet         | `snet-web`       |
| Web Subnet Range   | `10.0.1.0/24`    |
| App Subnet         | `snet-app`       |
| App Subnet Range   | `10.0.2.0/24`    |

Use a region supported by your subscription.

## 3. Create the VNet Using Azure Portal

1. Open the Azure Portal.
2. Search for **Virtual networks**.
3. Select **Create**.
4. Select the subscription.
5. Create or select the resource group.
6. Enter the VNet name `vnet-prins-lab`.
7. Select the region.
8. Configure the address space as `10.0.0.0/16`.
9. Add the `snet-web` subnet with `10.0.1.0/24`.
10. Add the `snet-app` subnet with `10.0.2.0/24`.
11. Select **Review + create**.
12. After validation succeeds, select **Create**.

## 4. Create a Resource Group Using Azure CLI

Sign in:

```bash
az login
```

List subscriptions:

```bash
az account list --output table
```

Select the intended subscription:

```bash
az account set --subscription "<SUBSCRIPTION_ID>"
```

Create the resource group:

```bash
az group create \
  --name rg-network-lab \
  --location centralindia
```

Explanation:

* `az group create` creates a resource group.
* `--name` defines its name.
* `--location` specifies its metadata location.

## 5. Create the VNet and First Subnet

```bash
az network vnet create \
  --resource-group rg-network-lab \
  --name vnet-prins-lab \
  --location centralindia \
  --address-prefixes 10.0.0.0/16 \
  --subnet-name snet-web \
  --subnet-prefixes 10.0.1.0/24
```

Explanation:

* `az network vnet create` creates the VNet.
* `--address-prefixes` sets the VNet address space.
* `--subnet-name` specifies the initial subnet.
* `--subnet-prefixes` defines the subnet range.

## 6. Create the Second Subnet

```bash
az network vnet subnet create \
  --resource-group rg-network-lab \
  --vnet-name vnet-prins-lab \
  --name snet-app \
  --address-prefixes 10.0.2.0/24
```

## 7. Verify the VNet

```bash
az network vnet show \
  --resource-group rg-network-lab \
  --name vnet-prins-lab \
  --output table
```

List subnets:

```bash
az network vnet subnet list \
  --resource-group rg-network-lab \
  --vnet-name vnet-prins-lab \
  --output table
```

## 8. Lab Validation Checklist

* [ ] Resource group exists.
* [ ] VNet exists.
* [ ] VNet address space is `10.0.0.0/16`.
* [ ] Web subnet is `10.0.1.0/24`.
* [ ] App subnet is `10.0.2.0/24`.
* [ ] Subnets do not overlap.
* [ ] The VNet is in the intended region.

---

# 76. Understanding Subnets

## 1. Why Do We Need Subnets?

Subnets help organize resources, allocate IP addresses, and apply security and routing policies.

Example:

```text
VNet: 10.0.0.0/16
|
├── Web Subnet: 10.0.1.0/24
│   └── Web Servers
|
├── App Subnet: 10.0.2.0/24
│   └── Application Servers
|
└── DB Subnet: 10.0.3.0/24
    └── Database Servers
```

## 2. Azure Reserved IPv4 Addresses

For a subnet such as `10.0.1.0/24`, Azure reserves five addresses.

| Address      | Purpose                               |
| ------------ | ------------------------------------- |
| `10.0.1.0`   | Network address                       |
| `10.0.1.1`   | Default gateway                       |
| `10.0.1.2`   | Azure DNS mapping                     |
| `10.0.1.3`   | Azure DNS mapping                     |
| `10.0.1.255` | Reserved for the subnet's end address |

The subnet contains 256 total addresses and 251 Azure-usable IPv4 addresses.

## 3. What Is an NSG?

A Network Security Group filters network traffic using security rules.

Example policy:

| Source   | Destination   |     Port | Intended Action           |
| -------- | ------------- | -------: | ------------------------- |
| Internet | Web tier      |  TCP 443 | Allow HTTPS               |
| Web tier | App tier      | TCP 3000 | Allow application traffic |
| App tier | Database tier | TCP 3306 | Allow MySQL traffic       |
| Internet | Database tier | TCP 3306 | Deny direct access        |

This is an illustrative design. Effective traffic decisions depend on the actual rule priorities, source/destination ranges, direction, default rules, and all applicable security controls.

## 4. How NSG Rules Work

NSG rules include:

* Priority
* Source
* Source port
* Destination
* Destination port
* Protocol
* Action: Allow or Deny
* Direction: Inbound or Outbound

Lower numeric priority values are evaluated before higher numeric values.

An explicit higher-priority deny can block traffic that a lower-priority rule would otherwise allow.

## 5. Create a Subnet Using Azure CLI

```bash
az network vnet subnet create \
  --resource-group rg-network-lab \
  --vnet-name vnet-prins-lab \
  --name snet-db \
  --address-prefixes 10.0.3.0/24
```

Verify:

```bash
az network vnet subnet show \
  --resource-group rg-network-lab \
  --vnet-name vnet-prins-lab \
  --name snet-db \
  --output json
```

## 6. Special Azure Subnets

Some Azure services require specially named and sized subnets.

For example, Azure Bastion requires a subnet named:

```text
AzureBastionSubnet
```

The required subnet size depends on the Bastion configuration and applicable Azure requirements. Verify the requirements for the selected Bastion deployment before creating it.

A Bastion deployment referencing a nonexistent subnet will fail.

## 7. Subnet Design Best Practices

* Use non-overlapping ranges.
* Keep sufficient free IP capacity.
* Separate application tiers when it helps security and management.
* Apply NSGs according to the principle of least privilege.
* Do not assume that different subnets are automatically isolated.
* Plan dedicated subnets for services that require them.

---

# 77. Lab: Deploying an Azure VM to the Network

## 1. Lab Objective

Deploy a Linux VM into the existing `snet-web` subnet.

## 2. Architecture

```text
Resource Group: rg-network-lab
|
└── VNet: vnet-prins-lab
    |
    └── snet-web: 10.0.1.0/24
        |
        └── VM: vm-web-01
            |
            └── NIC
                |
                └── Private IP
```

## 3. Deploy Through Azure Portal

1. Open **Virtual machines**.
2. Select **Create → Azure virtual machine**.
3. Choose the resource group `rg-network-lab`.
4. Enter `vm-web-01`.
5. Select a region compatible with the VNet.
6. Choose an available VM size.
7. Configure Linux authentication using an SSH public key.
8. Open the **Networking** tab.
9. Select `vnet-prins-lab`.
10. Select `snet-web`.
11. Review the public IP and inbound access settings.
12. Select **Review + create**.
13. Validate and deploy.

## 4. Deploy Through Azure CLI

The following example uses Ubuntu 22.04 and a small VM size. The size must be available in your subscription and region.

```bash
az vm create \
  --resource-group rg-network-lab \
  --name vm-web-01 \
  --location centralindia \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --vnet-name vnet-prins-lab \
  --subnet snet-web \
  --admin-username azureuser \
  --generate-ssh-keys
```

If your subscription cannot deploy `Standard_B1s`, select an available SKU using Azure's available VM sizes and quota information.

The `--generate-ssh-keys` option creates or uses local SSH keys. Keep your private key secure.

## 5. Verify the VM

```bash
az vm show \
  --resource-group rg-network-lab \
  --name vm-web-01 \
  --show-details \
  --output table
```

List IP addresses:

```bash
az vm list-ip-addresses \
  --resource-group rg-network-lab \
  --name vm-web-01 \
  --output table
```

## 6. Connect to the VM

Example:

```bash
ssh azureuser@10.0.1.4
```

Replace the example IP with the actual VM private IP.

This requires a network path to the private IP and the appropriate SSH, NSG, firewall, and authentication configuration.

Possible connection methods:

* Azure Bastion
* VPN
* An authorized jump host
* A suitably secured public endpoint

## 7. Troubleshoot SSH

Inside the VM, after gaining access:

```bash
ip addr
```

Displays network interfaces and addresses.

```bash
ip route
```

Displays the routing table.

```bash
sudo systemctl status ssh
```

Checks the SSH service status.

```bash
sudo ss -lntp
```

Shows listening TCP ports.

Check:

1. The VM is running.
2. The NIC is attached to the correct subnet.
3. The source can reach the VM.
4. Applicable NSGs permit TCP 22.
5. The guest firewall permits SSH.
6. The SSH service is listening.
7. The username and private key are correct.

---

# 78. Lab: Deploying Another VM to Another Subnet

## 1. Lab Objective

Deploy a second VM in `snet-app` and test communication between the two VMs.

## 2. Architecture

```text
VNet: 10.0.0.0/16
|
├── snet-web: 10.0.1.0/24
│   └── vm-web-01
│       └── Example IP: 10.0.1.4
|
└── snet-app: 10.0.2.0/24
    └── vm-app-01
        └── Example IP: 10.0.2.4
```

The IP addresses are illustrative.

## 3. Deploy the Second VM

```bash
az vm create \
  --resource-group rg-network-lab \
  --name vm-app-01 \
  --location centralindia \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --vnet-name vnet-prins-lab \
  --subnet snet-app \
  --admin-username azureuser \
  --generate-ssh-keys
```

Verify both VMs:

```bash
az vm list-ip-addresses \
  --resource-group rg-network-lab \
  --output table
```

## 4. Test Connectivity

From VM 1, test the private IP of VM 2:

```bash
ping -c 4 10.0.2.4
```

A failed ping does not automatically mean that routing is broken. ICMP may be blocked.

A TCP connectivity test is often more useful when you know which service should be reachable.

## 5. Install Nginx on VM 2

Connect to VM 2 and run:

```bash
sudo apt update
sudo apt install nginx -y
sudo systemctl enable --now nginx
```

Check the listening ports:

```bash
sudo ss -lntp
```

From VM 1, test the web service:

```bash
curl -I http://10.0.2.4
```

Replace the IP address with VM 2's actual private IP.

If the connection fails, check the NSGs, routing, guest firewall, and Nginx status.

## 6. Test TCP Connectivity

If `nc` is installed on VM 1:

```bash
nc -vz 10.0.2.4 80
```

This checks whether a TCP connection to port 80 can be established. It does not start a web server.

## 7. Real-World Three-Tier Application

```text
Internet
   |
   v
Public Web Endpoint
   |
   v
Web Tier
   |
   | Required application port
   v
Backend API
   |
   | Required database port
   v
Database Tier
```

Security recommendations:

* Allow only required user-facing traffic to the web tier.
* Permit backend traffic only from authorized sources.
* Restrict database traffic to the application tier and other approved clients.
* Avoid exposing databases directly to the public internet.
* Use private networking wherever practical.

## 8. Important Networking Rules

* VMs in different subnets of the same VNet normally communicate using default VNet routes.
* VNet peering is generally not required between subnets in the same VNet.
* Two separate VNets generally need peering, VPN, or another appropriate connection for private communication.
* Network connectivity also depends on security rules, routing, and the destination application.

---

# 79. Understanding IP Addresses in Azure

## 1. How Azure Assigns IP Addresses

When a VM is connected to a VNet, its NIC contains an IP configuration referencing a subnet.

Example:

```text
VNet: 10.0.0.0/16
  |
  └── Subnet: 10.0.1.0/24
        |
        └── VM: vm-web-01
              |
              └── NIC
                    |
                    └── Private IP: 10.0.1.4
```

The private IP belongs to the NIC's Azure IP configuration.

## 2. Private IP Address

Private IP addresses are used for communication between resources over private network paths.

Example:

```text
Web VM:     10.0.1.4
Backend VM: 10.0.2.4
Database:   10.0.3.4
```

Traffic between these VMs is possible when routing and security configurations permit it.

## 3. Public IP Address

A public IP provides an internet-routable endpoint when associated with a supported Azure resource.

Example:

```text
Internet
   |
   v
Public IP / Public Frontend
   |
   v
Azure Network Resource
   |
   v
Private VM
```

The exact architecture depends on the resource. A public IP does not automatically open every port.

## 4. Static Private IP Configuration

To configure a static private IP in the Azure Portal:

1. Open the VM.
2. Select **Networking**.
3. Open the attached NIC.
4. Select **IP configurations**.
5. Select the required IP configuration.
6. Change the private IP allocation method to **Static**.
7. Enter an available address within the subnet.
8. Save the configuration.

Example:

```text
Subnet: 10.0.1.0/24
Desired private IP: 10.0.1.10
```

Make sure the address is valid and available.

## 5. Find the VM's NIC Name

```bash
az vm show \
  --resource-group rg-network-lab \
  --name vm-web-01 \
  --query "networkProfile.networkInterfaces[].id" \
  --output tsv
```

List its IP configurations:

```bash
az network nic ip-config list \
  --resource-group rg-network-lab \
  --nic-name "<NIC_NAME>" \
  --output table
```

Configure a static private IP:

```bash
az network nic ip-config update \
  --resource-group rg-network-lab \
  --nic-name "<NIC_NAME>" \
  --name ipconfig1 \
  --private-ip-address 10.0.1.10
```

Replace `<NIC_NAME>` and `ipconfig1` with the actual values. Ensure the target address is available in the subnet.

Verify:

```bash
az network nic show \
  --resource-group rg-network-lab \
  --name "<NIC_NAME>" \
  --query "ipConfigurations[].{Name:name,PrivateIP:privateIPAddress,Subnet:subnet.id}" \
  --output table
```

## 6. Find Public and Private IP Addresses

```bash
az vm list-ip-addresses \
  --resource-group rg-network-lab \
  --name vm-web-01 \
  --output table
```

This command shows IP information associated with the VM.

## 7. What Is a Default Gateway?

A default gateway is the next hop used when a destination does not match a more specific route.

Linux command:

```bash
ip route
```

Example output:

```text
default via 10.0.1.1 dev eth0
```

This is an illustrative route. In Azure, the first usable IPv4 address in a subnet is reserved as the default gateway address.

## 8. What Is DNS?

DNS translates domain names into IP addresses.

Example:

```text
api.example.com -> IP address
```

Azure VMs can use Azure-provided DNS or appropriately configured custom DNS servers. Azure Private DNS zones can resolve private names for supported resources and connected VNets.

Example:

```bash
curl http://api.internal.example
```

This works only if the name resolves correctly and the destination is reachable.

## 9. Azure IP Address Best Practices

* Use private IPs for internal communication.
* Use public IPs only when required.
* Use static private IP allocation when a stable address is needed.
* Protect SSH and RDP endpoints.
* Use Bastion or private connectivity for administration where appropriate.
* Review NIC, subnet, NSG, and route configurations when troubleshooting.
* Avoid overlapping IP ranges when planning VNet connections.

---

# AZ-104 Practice Questions

## 1. Quick Questions

### Q1. What does an Azure VNet provide?

**Answer:** A logically isolated network in Azure that supports private network communication between connected resources.

### Q2. How many total IPv4 addresses exist in a /24 subnet?

**Answer:** 256.

Calculation:

```text
2^(32 - 24) = 256
```

### Q3. How many Azure-usable IPv4 addresses exist in a /26 subnet?

**Answer:** 59.

Calculation:

```text
Total = 2^(32 - 26) = 64
Usable = 64 - 5 = 59
```

### Q4. Do two subnets in the same VNet normally require VNet peering?

**Answer:** No. Azure provides default routes for communication within the same VNet.

### Q5. What is the purpose of an NSG?

**Answer:** To filter inbound and outbound traffic using security rules.

### Q6. Which is a private IPv4 address?

Options:

* A. `8.8.8.8`
* B. `10.1.2.3`
* C. `1.1.1.1`
* D. `203.0.113.25`

**Answer:** B. `10.1.2.3`.

### Q7. Which Linux command displays IP addresses?

**Answer:**

```bash
ip addr
```

### Q8. A VM has a private IP but cannot be accessed through SSH. What should you check?

**Answer:** NSGs, routes, guest firewall, SSH service status, connectivity, and authentication.

---

## 2. Scenario-Based Questions

### Scenario 1: Two VMs Cannot Communicate

VM1:

```text
10.0.1.4
```

VM2:

```text
10.0.2.4
```

Both VMs belong to the same VNet, but VM1 cannot connect to VM2 on TCP port 3000.

**Troubleshooting steps:**

1. Verify the destination application is listening on port 3000.
2. Check NSG rules on the NICs and subnets.
3. Check the guest firewall.
4. Verify the routes and any custom routing.
5. Test connectivity with a suitable TCP tool.

Do not create VNet peering just because the VMs are in different subnets of the same VNet.

### Scenario 2: Choose a Subnet Size

A team needs at least 50 Azure-usable IPv4 addresses.

Options:

* `/26`: 64 total, 59 Azure-usable addresses.
* `/27`: 32 total, 27 Azure-usable addresses.

**Answer:** `/26`.

### Scenario 3: Secure VM Administration

A Linux VM must be administered without exposing SSH directly to the public internet.

**Answer:** Consider Azure Bastion or private connectivity such as VPN, together with appropriate identity and access controls.

### Scenario 4: Database Security

A database VM must accept connections from an application VM but not from arbitrary internet sources.

**Answer:** Use private networking and narrowly scoped NSG rules. Avoid direct public database exposure.

### Scenario 5: Overlapping VNets

```text
VNet-A: 10.0.0.0/16
VNet-B: 10.0.1.0/24
```

The address spaces overlap.

**Question:** Can you directly establish standard VNet peering?

**Answer:** No. Standard Azure VNet peering does not support overlapping address spaces. Plan non-overlapping ranges before connecting the VNets.

---

# Practical Revision Checklist

* [ ] Explain the difference between public and private IP addresses.
* [ ] Calculate the total addresses in a CIDR block.
* [ ] Calculate Azure-usable IPv4 addresses.
* [ ] Explain how a VNet differs from a subnet.
* [ ] Create a VNet using Azure Portal.
* [ ] Create a VNet using Azure CLI.
* [ ] Deploy a VM into a specific subnet.
* [ ] Deploy two VMs into different subnets.
* [ ] Test private communication between VMs.
* [ ] Troubleshoot NSG and firewall problems.
* [ ] Configure a static private IP.
* [ ] Explain how Azure reserves five IPv4 addresses per subnet.
* [ ] Explain why different subnets in the same VNet normally do not require peering.
* [ ] Explain why overlapping address ranges cause connectivity problems.

---

# Cleanup: Remove Lab Resources

After completing the lab, remove the resource group if you no longer need its resources.

**Warning:** This deletes all resources inside the resource group, including VMs, disks, NICs, public IPs, and the VNet. Confirm that it contains only disposable lab resources before running the command.

```bash
az group delete \
  --name rg-network-lab \
  --yes \
  --no-wait
```

Verify the deletion later in the Azure Portal or CLI.

---

# Final Revision Summary

| Topic | Main Concept                                                |
| ----- | ----------------------------------------------------------- |
| 71    | Networking devices, protocols, ports, and IP addresses      |
| 72    | IPv4, IPv6, private/public, and static/dynamic IPs          |
| 73    | Subnet masks, CIDR, address ranges, and subnet calculations |
| 74    | Azure VNets and network components                          |
| 75    | Building a VNet and creating subnets                        |
| 76    | Subnet design, Azure-reserved IPs, and NSGs                 |
| 77    | Deploying a VM into a subnet                                |
| 78    | Deploying two VMs and testing private connectivity          |
| 79    | Azure private/public IP configuration and DNS               |

**Next recommended topic:** Networking 102 — Network Security Groups (NSGs), effective security rules, and troubleshooting network connectivity.
