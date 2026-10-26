
# AWS VPC Peering Architecture — Technical Documentation

## Project Overview

This project documents the design, implementation, and validation of an AWS Virtual Private Cloud (VPC) Peering architecture. The implementation demonstrates private communication between two isolated VPCs using EC2 instances, route tables, security groups, and a bastion host.

The documentation covers network architecture, CIDR allocation, subnet segmentation, routing configuration, secure administrative access, connectivity testing, troubleshooting, and security best practices.

## Project Objectives

- Design two VPCs with non-overlapping IPv4 CIDR ranges.
- Configure public and private subnets.
- Establish private communication through VPC Peering.
- Implement routing in both directions.
- Configure security groups to control network traffic.
- Access a private EC2 instance through a bastion host.
- Validate network connectivity using private IP addresses.
- Document the implementation process and technical findings.

## Technology Stack

- Amazon Virtual Private Cloud (VPC)
- Amazon EC2
- VPC Peering
- Route Tables
- Internet Gateway
- Security Groups
- Linux and SSH
- ICMP / Ping
- AWS Management Console

## Network Architecture

```mermaid
flowchart LR
    Internet((Internet)) --> IGW[Internet Gateway]
    IGW --> PublicRT[Public Route Table]
    PublicRT --> Bastion[Bastion EC2]
    Bastion --> AppA[App-A EC2<br/>10.10.2.10]
    AppA <-->|VPC Peering| AppB[App-B EC2<br/>10.20.1.10]
    AppA --- RouteA[VPC-A Private Route Table]
    AppB --- RouteB[VPC-B Private Route Table]
```

**Architecture summary:**

- VPC-A hosts the bastion host and App-A.
- VPC-B hosts App-B.
- VPC Peering enables private communication between the VPCs.
- Route tables direct traffic to the peering connection.
- Security groups control which traffic is permitted.
- The bastion host provides an SSH entry point to App-A.

<img width="1536" height="1024" alt="AWS VPC Peering Architecture Diagram" src="https://github.com/user-attachments/assets/d58a5f31-361e-464c-9766-1885e6f2d0e4" />

## Network Configuration

| Resource | CIDR / Private IP | Purpose |
|---|---|---|
| VPC-A | `10.10.0.0/16` | Primary application network |
| VPC-A public subnet | `10.10.1.0/24` | Bastion host |
| VPC-A private subnet | `10.10.2.0/24` | App-A |
| VPC-B | `10.20.0.0/16` | Peer application network |
| VPC-B private subnet | `10.20.1.0/24` | App-B |
| Bastion EC2 | `10.10.1.10` | Secure SSH entry point |
| App-A EC2 | `10.10.2.10` | Private application instance |
| App-B EC2 | `10.20.1.10` | Peer application instance |

**Note:** Confirm the resource names and IP addresses against your AWS console before publication. The bastion's public IP may change unless an Elastic IP is assigned.

## Implementation Methodology 

### Step 1: Create VPC-A

**Configuration:** Create VPC-A with CIDR `10.10.0.0/16`.

**Reason:** A VPC provides an isolated virtual network in which AWS resources can communicate using private IP addresses. The selected CIDR defines the address space available to the network.

**Screenshot:**  
<img width="1366" height="768" alt="Screenshot (148)" src="https://github.com/user-attachments/assets/83d16186-7e93-4242-8da6-d9bcc80c2025" />

<img width="1366" height="768" alt="Screenshot (149)" src="https://github.com/user-attachments/assets/50f07030-d9c1-45b3-9ce3-99021785add6" />

<img width="1366" height="768" alt="Screenshot (150)" src="https://github.com/user-attachments/assets/08b57fa6-8c91-4e18-85d2-a77a894ee343" />


### Step 2: Create VPC-A Subnets

Create the following subnets:

- Public subnet: `10.10.1.0/24`
- Private subnet: `10.10.2.0/24`

**Reason:** Subnet segmentation separates the internet-facing administrative entry point from the private application workload. This supports a more controlled network design.

**Screenshot:**  
<img width="1366" height="768" alt="Screenshot (150)" src="https://github.com/user-attachments/assets/bebd14de-0506-44e6-9ba6-0f0ac652b938" />

<img width="1366" height="768" alt="Screenshot (151)" src="https://github.com/user-attachments/assets/4928a3a4-c2fa-4a5c-9b5e-270d2fe8eab5" />

<img width="1366" height="768" alt="Screenshot (152)" src="https://github.com/user-attachments/assets/c5c1d455-0e1c-4797-aca4-f6cefa6da755" />

<img width="1366" height="768" alt="Screenshot (152)" src="https://github.com/user-attachments/assets/16af14e9-fa6f-4392-aa33-77e0a92cfebf" />

<img width="1366" height="768" alt="Screenshot (153)" src="https://github.com/user-attachments/assets/b11009d8-22ad-4718-8fd9-1604f23cf357" />

### Step 3: Configure the Internet Gateway

Create an Internet Gateway, attach it to VPC-A, and configure the public route table with:

`0.0.0.0/0 → Internet Gateway`

Associate the public route table with the public subnet and ensure the bastion has a public IPv4 address.

**Reason:** The Internet Gateway enables internet communication for appropriately routed public resources. A route alone is not sufficient; the instance also needs suitable addressing and security rules.

**Screenshot:**  
<img width="1366" height="768" alt="Screenshot (155)" src="https://github.com/user-attachments/assets/5da3e594-56fe-458f-9f92-9c31f395ce0c" />

<img width="1366" height="768" alt="Screenshot (156)" src="https://github.com/user-attachments/assets/6c23fe15-a117-4259-bd2b-6508c50d62b0" />

### Step 4: Create VPC-B

Create VPC-B with CIDR `10.20.0.0/16`.

**Reason:** The second VPC represents a separate network environment. Its CIDR does not overlap with VPC-A, allowing AWS routing to distinguish destinations unambiguously.

**Screenshot:**  
<img width="1366" height="768" alt="Screenshot (172)" src="https://github.com/user-attachments/assets/21cbdc14-d34b-4915-857b-1013070b44c5" />

<img width="1366" height="768" alt="Screenshot (173)" src="https://github.com/user-attachments/assets/de0cd22f-aae0-4a75-8906-d3f939b0174a" />

### Step 5: Create the VPC-B Private Subnet

Create a private subnet in VPC-B with CIDR `10.20.1.0/24`.

**Reason:** This subnet hosts App-B without requiring a public IP address for private communication with App-A.

**Screenshot:**  
<img width="1366" height="768" alt="Screenshot (174)" src="https://github.com/user-attachments/assets/941eb645-e4da-4c93-87dd-624eeb1587b5" />
<img width="1366" height="768" alt="Screenshot (175)" src="https://github.com/user-attachments/assets/4389b83f-af80-423c-b694-37fa6f5087d3" />

<img width="1366" height="768" alt="Screenshot (176)" src="https://github.com/user-attachments/assets/5ea42900-ac84-49ef-b355-b974e5c4533e" />

### Step 6: Configure Route Tables and Subnet Associations

Create or verify the route tables for the public subnet in VPC-A, the private subnet in VPC-A, and the private subnet in VPC-B. Associate each subnet with its intended route table.

**Reason:** Route tables determine how traffic leaves a subnet. Correct subnet associations are essential because a route added to an unrelated route table will not affect the intended traffic.

**Screenshot:**  
FOR VPC A :
<img width="1366" height="768" alt="Screenshot (159)" src="https://github.com/user-attachments/assets/bcd63de9-82a1-4032-9699-7a369159e805" />

<img width="1366" height="768" alt="Screenshot (162)" src="https://github.com/user-attachments/assets/8ecb9759-d1ce-4e08-b76c-9dd5f12e561d" />

<img width="1366" height="768" alt="Screenshot (165)" src="https://github.com/user-attachments/assets/62d6ec63-3808-4c5e-945f-5acaa14d00af" />

FOR VPC B: 
<img width="1366" height="768" alt="Screenshot (174)" src="https://github.com/user-attachments/assets/4a0ea67c-ee32-46be-bdfc-ba57c6df084a" />

<img width="1366" height="768" alt="Screenshot (175)" src="https://github.com/user-attachments/assets/e7cd2571-25a9-4a2d-b0af-69e93433fcf3" />

<img width="1366" height="768" alt="Screenshot (177)" src="https://github.com/user-attachments/assets/4eb202d7-d3f0-471c-a616-66eb2fed8de0" />

<img width="1366" height="768" alt="Screenshot (178)" src="https://github.com/user-attachments/assets/8d4827e8-63c8-4dcd-90ec-a0f5f5ca884c" />

<img width="1366" height="768" alt="Screenshot (179)" src="https://github.com/user-attachments/assets/41a9f4c3-3296-4537-86c7-a3cd5ff64459" />

### Step 7: Configure Security Groups

Configure security groups using least-privilege access:

- Bastion: allow SSH (TCP 22) only from your trusted public IP.
- App-A: allow SSH from the bastion security group or its private address, according to the configuration.
- App-A and App-B: allow the required ICMP traffic for ping testing.

**Reason:** Security groups act as stateful virtual firewalls. Restricting inbound traffic reduces unnecessary exposure and helps protect application instances.

**Screenshot:**  
<img width="1366" height="768" alt="Screenshot (182)" src="https://github.com/user-attachments/assets/d2a4121e-011b-4319-8481-77abb50b6d3a" />

For app A:
<img width="1366" height="768" alt="Screenshot (184)" src="https://github.com/user-attachments/assets/97f52a00-9e3a-42cb-b83b-a70cc08fb400" />

<img width="1366" height="768" alt="Screenshot (187)" src="https://github.com/user-attachments/assets/a2ae5087-a0b6-4b7b-81dc-3e19211418ad" />

For app B:
<img width="1366" height="768" alt="Screenshot (190)" src="https://github.com/user-attachments/assets/b12af9cf-bdfc-4db0-a36c-992fbc716d0e" />

<img width="1366" height="768" alt="Screenshot (193)" src="https://github.com/user-attachments/assets/d09e3a3f-91fd-4b80-849a-5a32f68d2e76" />

### Step 8: Create the EC2 Key Pair

Create or select the SSH key pair used for administrative access. The key used in this project was named `ewb-lab.pem`.

**Reason:** SSH key-based authentication allows secure access without relying on password-based login.

**Security requirement:** Never upload private key files, AWS access keys, passwords, or other credentials to GitHub.

### Step 9: Launch the EC2 Instances

Launch the instances in their intended subnets:

- Bastion in the VPC-A public subnet.
- App-A in the VPC-A private subnet.
- App-B in the VPC-B private subnet.

Verify the private IP addresses and security group associations.

**Reason:** These instances provide the endpoints required to demonstrate administrative access and private communication across the peering connection.

**Screenshot:**  
<img width="1366" height="768" alt="Screenshot (196)" src="https://github.com/user-attachments/assets/2e1d304a-aacd-41c5-9bf8-21456e66cbe1" />

<img width="1366" height="768" alt="Screenshot (197)" src="https://github.com/user-attachments/assets/90de115c-efcb-4844-9236-203854d11012" />

App A:
<img width="1366" height="768" alt="Screenshot (200)" src="https://github.com/user-attachments/assets/636c4aea-12b1-48ab-be47-17bb0b418018" />

<img width="1366" height="768" alt="Screenshot (201)" src="https://github.com/user-attachments/assets/81115c76-b174-4650-945d-7dadc1debbc8" />

<img width="1366" height="768" alt="Screenshot (202)" src="https://github.com/user-attachments/assets/6408d821-2c53-415c-b34e-bb409731355c" />

App B:
<img width="1366" height="768" alt="Screenshot (206)" src="https://github.com/user-attachments/assets/b1abc20c-0f05-41fa-9fd2-6f7b6a5cb0c3" />

<img width="1366" height="768" alt="Screenshot (207)" src="https://github.com/user-attachments/assets/399d8f31-4e9b-4f76-bf3c-0c379037a760" />

<img width="1366" height="768" alt="Screenshot (208)" src="https://github.com/user-attachments/assets/f3ad6237-085b-488a-90be-40e1e88af336" />

### Step 10: Connect to the Bastion Host

From Windows PowerShell, connect to the bastion using its current public IPv4 address:

```powershell
ssh -i "C:\Users\YOUR_WINDOWS_USER\Desktop\ewb-lab.pem" ec2-user@BASTION_PUBLIC_IP
```

Replace the placeholders with your actual Windows username and bastion public IP. Confirm the correct login username for the selected AMI.

**Reason:** The bastion provides a controlled SSH entry point for reaching a private instance without assigning that instance a public IP address.

**Screenshot:**  
<img width="1366" height="373" alt="Screenshot (216)" src="https://github.com/user-attachments/assets/417a0461-1902-4e75-9058-518d1032ee5d" />

### Step 11: Access App-A Through the Bastion

Use SSH ProxyCommand from Windows PowerShell:

```powershell
ssh -i "C:\Users\YOUR_WINDOWS_USER\Desktop\ewb-lab.pem" `
  -o IdentitiesOnly=yes `
  -o ProxyCommand="ssh -i C:\Users\YOUR_WINDOWS_USER\Desktop\ewb-lab.pem -o IdentitiesOnly=yes -W %h:%p ec2-user@BASTION_PUBLIC_IP" `
  ec2-user@10.10.2.10
```

Replace the Windows username and bastion public IP before executing the command.

**Reason:** ProxyCommand forwards the SSH connection through the bastion to App-A's private IP. This demonstrates access to a private instance through a controlled entry point.

**Screenshot:**  
<img width="1366" height="409" alt="Screenshot (217)" src="https://github.com/user-attachments/assets/5dc5513f-bfb2-41d3-889c-e807299dd95f" />

### Step 12: Verify App-A's Private IP

Run the following command after logging in to App-A:

```bash
hostname -I
```

Verify that the output includes `10.10.2.10`.

**Reason:** This confirms that the SSH session reached the expected private instance and provides a baseline before connectivity testing.

**Screenshot:**  
<img width="1366" height="568" alt="Screenshot (220)" src="https://github.com/user-attachments/assets/b9b0b574-c5ba-419f-9424-15f858b4573a" />

### Step 13: Establish a Baseline Connectivity Test

From App-A, test the private IP of App-B:

```bash
ping -c 4 10.20.1.10
```

If peering routes are not yet configured, this test is expected to fail in the intended isolated network design.

**Reason:** A baseline test helps compare connectivity before and after peering configuration. Ping uses ICMP, so the result also depends on security group, network ACL, and host firewall rules.

### Step 14: Create and Accept the VPC Peering Connection

Create a VPC Peering connection between:

- VPC-A: `10.10.0.0/16`
- VPC-B: `10.20.0.0/16`

Accept the request and wait until the connection status is **Active**.

**Reason:** The peering connection establishes the relationship required for private routing between the two VPCs. It does not automatically create the necessary routes.

**Screenshot:**  
<img width="1366" height="768" alt="Screenshot (223)" src="https://github.com/user-attachments/assets/e87bb978-067d-4171-9c0c-bcbbd92684d8" />

<img width="1366" height="768" alt="Screenshot (224)" src="https://github.com/user-attachments/assets/9090f90e-7026-497a-a1b1-451651df841d" />

<img width="1366" height="768" alt="Screenshot (225)" src="https://github.com/user-attachments/assets/1a94d215-66bd-4493-82b3-29d09a4a5aa7" />

<img width="1366" height="768" alt="Screenshot (226)" src="https://github.com/user-attachments/assets/a65fb524-8c2d-431d-a628-0e0a35cc4546" />

### Step 15: Configure Routes in Both VPCs

Add the following routes to the relevant private route tables:

| Route table | Destination | Target |
|---|---|---|
| VPC-A private route table | `10.20.0.0/16` | VPC Peering connection |
| VPC-B private route table | `10.10.0.0/16` | VPC Peering connection |

**Reason:** Both sides need routes to direct traffic destined for the other VPC through the peering connection. Missing a route in either direction can prevent communication.

**Screenshot — VPC-A route:**  
<img width="1366" height="768" alt="Screenshot (229)" src="https://github.com/user-attachments/assets/c6512d69-fb58-4c39-9dfb-07208ebcc6ca" />


**Screenshot — VPC-B route:**  
<img width="1366" height="768" alt="Screenshot (232)" src="https://github.com/user-attachments/assets/f90df3a7-fdb3-4203-8bb8-68e5fedb2578" />

### Step 16: Validate Private Connectivity

From App-A, execute:

```bash
ping -c 4 10.20.1.10
```

The App-A-to-App-B ping was successfully tested in this project.

If you also configured the necessary ICMP rules and performed a reverse-direction test, run this from App-B:

```bash
ping -c 4 10.10.2.10
```

**Reason:** Connectivity testing provides evidence that routing and traffic controls permit communication between the private instances. A successful ping validates ICMP reachability at the time of the test.

**Screenshot:**  
<img width="1366" height="607" alt="Screenshot (239)" src="https://github.com/user-attachments/assets/9d8b94fa-5a7e-4ef2-8fce-0cba9ad557e9" />

## Validation Summary

| Validation item | Result |
|---|---|
| VPC CIDR planning | Two non-overlapping CIDR ranges configured |
| Bastion SSH access | Successfully connected during the lab |
| App-A access through bastion | Successfully connected during the lab |
| App-A private IP verification | `10.10.2.10` |
| App-A to App-B ICMP test | Successful |
| VPC Peering routes | Configured in both directions |

## Troubleshooting

If private communication fails, verify:

1. The VPC Peering connection is active.
2. The VPC CIDR ranges do not overlap.
3. VPC-A has a route to `10.20.0.0/16` through peering.
4. VPC-B has a route to `10.10.0.0/16` through peering.
5. Each subnet is associated with the correct route table.
6. Security groups permit the required traffic.
7. Network ACLs and Linux host firewalls do not block the traffic.
8. The destination private IP address is correct.

## Security Considerations

- Restrict SSH access to trusted source IP addresses.
- Keep application instances private unless public access is explicitly required.
- Apply least-privilege security group rules.
- Never commit private keys, AWS credentials, or passwords.
- Review screenshots for sensitive account information before publishing.
- Remove unused AWS resources to reduce unnecessary costs.

## Conclusion

This project documents the implementation and validation of private communication between two AWS VPCs using VPC Peering. It demonstrates practical cloud networking concepts, controlled SSH access, routing configuration, security controls, and connectivity verification.

## Author

- **Name:** [SUMAN YADAV]
- **Project:** AWS VPC Peering Architecture
- **Documentation:** Technical implementation and validation for EWB PVT LTD.
---
