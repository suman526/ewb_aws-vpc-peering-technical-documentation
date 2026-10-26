
# AWS VPC Peering Architecture — Concepts

## Concepts

---

### 1. Amazon Virtual Private Cloud (VPC)

A **Virtual Private Cloud (VPC)** is a logically isolated virtual network in AWS. It defines an IP address space and contains resources such as subnets, route tables, and network interfaces.

VPC-A and VPC-B are separate networks. Creating two VPCs does not automatically allow them to communicate. A connectivity mechanism, suitable routes, and traffic permissions are required.

VPCs provide network isolation and let you define where resources are placed and how they communicate.

---

### 2. CIDR and IPv4 Address Planning

**CIDR** stands for *Classless Inter-Domain Routing*. A CIDR block describes an IP address range.

The project uses the following address ranges:

- `10.10.0.0/16` — Address range for VPC-A.
- `10.20.0.0/16` — Address range for VPC-B.
- `10.10.2.0/24` — Subnet containing App-A.
- `10.20.1.0/24` — Subnet containing App-B.

A `/16` prefix describes a larger address range than a `/24` prefix.

The VPC CIDRs in this design do not overlap. Overlapping CIDRs prevent a normal VPC Peering configuration because destination address ranges would be ambiguous.

CIDR planning organizes address allocation and enables route tables to distinguish destinations.

---

### 3. Subnets

A **subnet** is a subdivision of a VPC's IP address range and exists in one Availability Zone. Its route table determines how traffic is directed. The labels *public* and *private* describe the subnet's routing and reachability design.

This project uses:

- **Public subnet:** `10.10.1.0/24` for the bastion.
- **Private subnet:** `10.10.2.0/24` for App-A.
- **Private subnet:** `10.20.1.0/24` for App-B.

Subnet segmentation separates administrative access from application workloads and supports different routing and security policies.

---

### 4. Public and Private Subnets

A subnet is generally considered **public** when its route table has a route to an Internet Gateway and resources have appropriate public addressing and security rules.

A **private subnet** does not use a direct Internet Gateway route for direct internet reachability.

A private instance can have outbound internet access if a separate egress design, such as a NAT Gateway, is configured. VPC Peering does not provide internet access by itself.

Keeping application instances private reduces their direct exposure to the internet.

---

### 5. Internet Gateway (IGW)

An **Internet Gateway (IGW)** is a VPC component that enables internet communication for resources with suitable public addressing, routing, and security rules.

Example VPC-A public route table:

| Destination | Target |
|---|---|
| `10.10.0.0/16` | `local` |
| `0.0.0.0/0` | Internet Gateway (`ewb-igw-a`) |

The local route enables communication within the VPC's address space. The default route, `0.0.0.0/0`, directs traffic not matched by a more specific route to the Internet Gateway.

The Internet Gateway provides the internet-facing network path to the bastion's public IPv4 address.

---

### 6. Route Tables and Routing

A **route table** contains destination-and-target rules that determine where traffic is sent. Each subnet is associated with a route table, either explicitly or through the VPC's main route table.

#### Required Private Routes

| Route Table | Destination | Target |
|---|---|---|
| VPC-A private route table | `10.10.0.0/16` | `local` |
| VPC-A private route table | `10.20.0.0/16` | VPC Peering connection |
| VPC-B private route table | `10.20.0.0/16` | `local` |
| VPC-B private route table | `10.10.0.0/16` | VPC Peering connection |

AWS uses the most specific matching route. The peering route tells AWS where to send packets whose destination belongs to the other VPC.

A peering connection alone does not automatically install routes. Both sides must have appropriate routes in the route tables associated with the participating subnets.

---

### 7. VPC Peering

**VPC Peering** is a private network connection between two VPCs. It allows resources to communicate using private IP addresses when routing and security rules permit it.

In this project:

- **Requester:** VPC-A (`10.10.0.0/16`)
- **Accepter:** VPC-B (`10.20.0.0/16`)
- **Required state:** `Active`

The peering request must be accepted where required, and the routes must be added separately.

VPC Peering is **not transitive**. If VPC-A peers with VPC-B and VPC-B peers with VPC-C, this does not automatically permit VPC-A to reach VPC-C through VPC-B.

Peering enables private connectivity without routing inter-VPC traffic through the public internet.

---

### 8. Amazon EC2

**Amazon Elastic Compute Cloud (EC2)** provides virtual server instances in AWS. An instance is placed in a subnet and uses network interfaces, security groups, and route tables for network connectivity.

The roles in this project are:

- **Bastion:** Administrative entry point in the public subnet.
- **App-A:** Private instance in VPC-A.
- **App-B:** Private instance in VPC-B.

---

### 9. Security Groups

A **security group** is a stateful virtual firewall associated with an EC2 network interface. It controls allowed inbound and outbound traffic.

Security groups determine which network connections are permitted to reach an instance and which outbound connections it can initiate.

---

### 10. Network ACLs

A **network access control list (network ACL)** is an optional subnet-level traffic filter.

Unlike security groups, network ACLs are **stateless**, so both inbound and outbound rules must permit the relevant traffic.

If connectivity fails despite correct routes and security groups, inspect the network ACLs.

---

### 11. Bastion Host

A **bastion host** is a controlled entry point for accessing resources that do not have direct public reachability.

In this project, the bastion is in VPC-A's public subnet. The administrator connects to it over SSH, then forwards a connection to App-A's private IP.

A bastion is not automatically secure just because it is a bastion. Restrict SSH sources, protect credentials, patch the operating system, and monitor administrative access.

It demonstrates a way to administer a private instance without assigning App-A a public IP address.

---

### 12. SSH and Key-Based Authentication

**Secure Shell (SSH)** is a protocol for encrypted remote login and command execution. Linux EC2 instances commonly use key-based authentication.

Example command to connect to the bastion:

```bash
ssh -i "C:\path\to\ewb-lab.pem" ec2-user@BASTION_PUBLIC_IP
```

SSH provides encrypted administrative access. The private key must be kept confidential and must never be committed to GitHub.

#### SSH ProxyCommand

`ProxyCommand` allows an SSH client to use another command—in this case, an SSH connection to the bastion—to forward the connection to a private destination.

Example for Windows PowerShell:

```powershell
ssh -i "C:\path\to\ewb-lab.pem" `
  -o IdentitiesOnly=yes `
  -o ProxyCommand="ssh -i C:\path\to\ewb-lab.pem -o IdentitiesOnly=yes -W %h:%p ec2-user@BASTION_PUBLIC_IP" `
  ec2-user@10.10.2.10
```

Replace the example path and bastion address with the actual values.

The `-W %h:%p` option forwards standard input and output to the requested destination host and port through the bastion.

This lets the local SSH client reach App-A's private IP through the bastion without giving App-A a public IP address.

---

### 13. ICMP and Ping

**Internet Control Message Protocol (ICMP)** supports diagnostic messages. The `ping` utility uses ICMP Echo Request and Echo Reply messages.

From App-A, run:

```bash
ping -c 4 10.20.1.10
```

This sends four ICMP Echo Requests to App-B's private IP.

A successful ping indicates that ICMP echo communication worked at the time of the test.

A failed ping does not necessarily mean routing is broken. Security groups, network ACLs, host firewalls, or the destination instance may block ICMP.

Ping is a simple connectivity test across the peered VPCs.

---

### 14. Private IP Communication

App-A (`10.10.2.10`) reaches App-B (`10.20.1.10`) using private addresses.

When routes and security rules are correct, packets traverse the VPC Peering connection.

This validates the intended private network path rather than using public IP addresses for inter-VPC communication.

---

## Author

**SUMAN YADAV**
