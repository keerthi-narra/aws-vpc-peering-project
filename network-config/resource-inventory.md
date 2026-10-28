# AWS VPC Peering Project — Resource Inventory

## 1. Project Overview

- **Project:** AWS VPC Peering
- **AWS Region:** Asia Pacific (Sydney) — `ap-southeast-2`
- **VPC Peering Connection:** `ewb-vpc-peering`
- **Final status:** AWS project resources were cleaned up after testing.

## 2. VPC Inventory

| VPC Name | CIDR Block | VPC ID |
|---|---|---|
| ewb-vpc-a | `10.10.0.0/16` | `vpc-030aeff11da561a18` |
| ewb-vpc-b | `10.20.0.0/16` | `vpc-0d9ab69f6185e6992` |

The two VPCs use non-overlapping IPv4 CIDR ranges.

## 3. Subnet Inventory

| VPC | Subnet Name | CIDR Block | Subnet ID |
|---|---|---|---|
| VPC A | ewb-a-public | `10.10.1.0/24` | `subnet-05c8ba03678ed52de` |
| VPC A | ewb-a-private | `10.10.2.0/24` | `subnet-06ac47d63156aea43` |
| VPC B | ewb-b-private | `10.20.1.0/24` | `subnet-0f9729f451e2ac2cd` |

## 4. EC2 Instance Inventory

| Instance | Role | Private IP |
|---|---|---|
| ewb-bastion | Bastion host | `10.10.1.10` |
| ewb-app-a | Application host in VPC A | `10.10.2.10` |
| ewb-app-b | Application host in VPC B | `10.20.1.10` |

All three instances were terminated during cleanup.

## 5. Route Table Inventory

### VPC A — Public Route Table

- **Name:** `ewb-rt-a-public`
- **Route table ID:** `rtb-06d98fbc05ad211c5`
- **Associated subnet:** `ewb-a-public` (`10.10.1.0/24`)

| Destination | Target |
|---|---|
| `10.10.0.0/16` | Local |
| `0.0.0.0/0` | Internet Gateway `igw-05456be81e68372be` |
| `10.20.0.0/16` | VPC peering connection `pcx-065e6374381912571` |

### VPC A — Private Route Table

- **Name:** `ewb-rt-a-private`
- **Route table ID:** `rtb-0f69e34cf8868a21d`
- **Associated subnet:** `ewb-a-private` (`10.10.2.0/24`)

| Destination | Target |
|---|---|
| `10.10.0.0/16` | Local |
| `10.20.0.0/16` | VPC peering connection `pcx-065e6374381912571` |

### VPC B — Private Route Table

- **Name:** `ewb-rt-b-private`
- **Route table ID:** `rtb-050cd80fd91115329`
- **Associated subnet:** `ewb-b-private` (`10.20.1.0/24`)

| Destination | Target |
|---|---|
| `10.20.0.0/16` | Local |
| `10.10.0.0/16` | VPC peering connection `pcx-065e6374381912571` |

**Note:** These are the project's intended configured routes before cleanup. Verify them against your saved screenshots if you need to confirm every route and association.

## 6. VPC Peering Connection

- **Name:** `ewb-vpc-peering`
- **Connection ID:** `pcx-065e6374381912571`
- **Requester:** VPC A — `10.10.0.0/16`
- **Accepter:** VPC B — `10.20.0.0/16`
- **Observed pre-cleanup status:** Active
- **Final cleanup:** Peering connection deleted; related route table entries were removed.

## 7. Security Group Inventory

### Bastion Security Group

- **Name:** `ewb-sg-bastion`
- **Security group ID:** `sg-0229cdf463f11fb6e`

| Direction | Protocol | Port/Type | Source/Destination |
|---|---|---|---|
| Inbound | TCP | 22 (SSH) | Your public IPv4 address `/32` used during the lab |

### App-A Security Group

- **Name:** `ewb-sg-app-a`
- **Security group ID:** `sg-0634ec6353820e3b1`

| Direction | Protocol | Port/Type | Source/Destination |
|---|---|---|---|
| Inbound | TCP | 22 (SSH) | `10.10.1.10/32` |
| Inbound | ICMP Echo Request | Ping | `10.20.1.10/32` |

### App-B Security Group

- **Name:** `ewb-sg-app-b`
- **Security group ID:** `sg-0a889666a1856f01a`

| Direction | Protocol | Port/Type | Source/Destination |
|---|---|---|---|
| Inbound | TCP | 22 (SSH) | `10.10.1.10/32` |
| Inbound | ICMP Echo Request | Ping | `10.10.2.10/32` |

The three custom project security groups were deleted during cleanup. The outbound rules were left at their default allow-all configuration during testing.

## 8. Connectivity Test Results

| Test | Result |
|---|---|
| App-A to App-B ping after peering | 4/4 replies; 0% packet loss |
| App-B to App-A ping after peering | 4/4 replies; 0% packet loss |
| SSH to App-B through bastion using ProxyJump | Successful |
| Controlled ICMP rule removal | Failure observed according to the test process; retain a screenshot if available |
| ICMP rule restoration | Connectivity recovered according to the test process; retain a screenshot if available |

The original baseline ping failure screenshot was not saved. Do not represent a different screenshot as proof of that baseline test.

## 9. Cleanup Summary

The following project resources were reported deleted or terminated:

- Three EC2 instances
- Project VPC peering connection
- Three custom project security groups
- Three project subnets
- Three custom project route tables
- Project Internet Gateway
- Both project VPCs

EBS volumes, snapshots, and Elastic IPs were reported empty. Preserve screenshots as evidence where available.

## 10. Learning Outcomes

- Created two non-overlapping VPCs and configured their subnets.
- Configured route tables for public access and private cross-VPC communication.
- Established VPC peering and configured routes in both directions.
- Used security group rules to control SSH and ICMP traffic.
- Verified forward and reverse connectivity and SSH access through a bastion host.
- Performed cleanup to reduce the risk of ongoing AWS charges.
