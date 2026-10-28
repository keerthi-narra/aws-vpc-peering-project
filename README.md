# AWS VPC Peering Project

**Project:** Connecting Two Private Networks Using AWS VPC Peering  
**Student Name:** Narra Keerthi Reddy                               
**Batch:** [Your Batch]  
**AWS Region:** Asia Pacific (Sydney) — `ap-southeast-2`   
**Completion Date:** 28 October 2026    

## 1. Project Overview

This project demonstrates how to establish private communication between two Amazon Virtual Private Clouds (VPCs) using VPC Peering.

Two VPCs with non-overlapping IPv4 CIDR blocks were configured. A bastion host in VPC A provided administrative SSH access to two private EC2 application instances. Route tables and security groups were configured to control communication between the networks.

The project was implemented using the AWS Management Console and tested using private IP addresses.

## 2. Objectives

- Create two VPCs with non-overlapping CIDR blocks.
- Configure public and private subnets with appropriate route tables.
- Launch a bastion host and two private EC2 instances.
- Establish a VPC peering connection.
- Configure forward and return routes.
- Control SSH and ICMP traffic using security groups.
- Test private connectivity between application instances.
- Perform a controlled ICMP failure and recovery test.
- Document the cleanup of project resources.

## 3. Architecture

The architecture consists of two VPCs connected through a VPC peering connection.

### VPC A

- VPC CIDR: `10.10.0.0/16`
- Public subnet: `10.10.1.0/24`
- Private subnet: `10.10.2.0/24`
- Bastion private IP: `10.10.1.10`
- App-A private IP: `10.10.2.10`

### VPC B

- VPC CIDR: `10.20.0.0/16`
- Private subnet: `10.20.1.0/24`
- App-B private IP: `10.20.1.10`

The bastion host provides SSH access from the administrator's computer to the private instances. App-A and App-B communicate through VPC peering using their private IP addresses.

Architecture diagram: `docs/architecture.png` (to be added).

## 4. AWS Resource Inventory

| Resource | Name | Resource ID |
|---|---|---|
| VPC A | `ewb-vpc-a` | `vpc-030aeff11da561a18` |
| VPC B | `ewb-vpc-b` | `vpc-0d9ab69f6185e6992` |
| Public subnet A | `ewb-a-public` | `subnet-05c8ba03678ed52de` |
| Private subnet A | `ewb-a-private` | `subnet-06ac47d63156aea43` |
| Private subnet B | `ewb-b-private` | `subnet-0f9729f451e2ac2cd` |
| Internet Gateway | `ewb-igw-a` | `igw-05456be81e68372be` |
| Public route table | `ewb-rt-a-public` | `rtb-06d98fbc05ad211c5` |
| Private route table A | `ewb-rt-a-private` | `rtb-0f69e34cf8868a21d` |
| Private route table B | `ewb-rt-b-private` | `rtb-050cd80fd91115329` |
| VPC peering | `ewb-vpc-peering` | `pcx-065e6374381912571` |
| Bastion EC2 | `ewb-bastion` | `i-09b583320c753445e` |
| App-A EC2 | `ewb-app-a` | `i-03b2a01b36b1bf11f` |
| App-B EC2 | `ewb-app-b` | `i-09cb338976d325390` |

These IDs document the original lab resources. The resources were subsequently cleaned up.

## 5. Route Table Configuration

### VPC A public route table

| Destination | Target | Purpose |
|---|---|---|
| `10.10.0.0/16` | Local | Internal VPC A communication |
| `0.0.0.0/0` | Internet Gateway | Internet access for the public subnet |
| `10.20.0.0/16` | VPC peering | Route from the bastion subnet toward VPC B |

Associated subnet: `ewb-a-public`.

### VPC A private route table

| Destination | Target | Purpose |
|---|---|---|
| `10.10.0.0/16` | Local | Internal VPC A communication |
| `10.20.0.0/16` | VPC peering | Route from App-A toward VPC B |

Associated subnet: `ewb-a-private`.

### VPC B private route table

| Destination | Target | Purpose |
|---|---|---|
| `10.20.0.0/16` | Local | Internal VPC B communication |
| `10.10.0.0/16` | VPC peering | Return route toward VPC A |

Associated subnet: `ewb-b-private`.

## 6. Security Groups

### Bastion — `ewb-sg-bastion`

- Inbound SSH, TCP port 22, from the administrator's trusted public IPv4 address using `/32`.
- Default outbound traffic allowed.

### App-A — `ewb-sg-app-a`

- SSH, TCP port 22, from `10.10.1.10/32`.
- ICMP Echo Request from `10.20.1.10/32`.
- Default outbound traffic allowed.

### App-B — `ewb-sg-app-b`

- SSH, TCP port 22, from `10.10.1.10/32`.
- ICMP Echo Request from `10.10.2.10/32`.
- Default outbound traffic allowed.

The application instances had no public IP addresses. SSH administration was performed through the bastion host.

## 7. Connectivity Testing

The following tests were part of the project validation.

| Test | Expected or observed result |
|---|---|
| Baseline ping before peering | Failed; the original screenshot is unavailable |
| Forward ping: App-A to App-B | Successful, 4 packets received and 0% packet loss |
| Reverse ping: App-B to App-A | Successful, 4 packets received and 0% packet loss |
| SSH to App-B through bastion | Successful |
| Controlled ICMP failure | Temporarily remove App-B's ICMP Echo Request rule and verify ping failure |
| ICMP recovery | Restore the rule and verify successful ping again |

The original baseline failure screenshot was not retained. The controlled ICMP failure and recovery should be supported by the saved test screenshots where available.

## 8. Forward and Return Traffic Paths

### Forward path

App-A (`10.10.2.10`) sends traffic to App-B (`10.20.1.10`). The VPC A private route table directs traffic for `10.20.0.0/16` to the peering connection. VPC B's local route delivers the traffic to App-B, subject to security-group rules.

### Return path

App-B sends its response to `10.10.2.10`. VPC B's private route table directs traffic for `10.10.0.0/16` to the peering connection. VPC A's local route delivers the response to App-A.

Both forward and return routes are required for successful two-way communication.

## 9. Cleanup

After the testing and evidence collection, the lab resources were removed. The three EC2 instances were terminated, the VPC peering connection and its routes were deleted, the project security groups and custom route tables were deleted, and the project Internet Gateway was deleted.

EBS volumes, snapshots, and Elastic IPs were reported as empty in the Sydney Region. The final state of both project VPCs should be verified and documented.

## 10. Key Learnings

- VPC peering connects networks with non-overlapping CIDR blocks.
- An active peering connection alone does not enable communication; routes and security groups must also be configured.
- Forward and return routes are both necessary.
- Private EC2 instances can be accessed securely through a bastion host without public IP addresses.
- Security groups restrict traffic by source, protocol, and port.
- Controlled failure and recovery tests help verify that network security rules work as intended.
- Cleanup is important to avoid unnecessary AWS resource charges.

## 11. Project Status

The AWS lab resources were created, tested, and subsequently cleaned up. This repository documents the original configuration, test results, and lessons learned; it does not represent a currently running AWS environment.

## 12. Repository Structure

```text
aws-vpc-peering-project/
├── README.md
├── docs/
│   ├── YourName_EWB_VPC_Peering.pdf
│   └── architecture.png
├── screenshots/
├── network-config/
│   └── resource-inventory.md
├── cleanup.md
└── .gitignore
```

Screenshots and the final PDF should be added before submission. No private keys, passwords, access tokens, or AWS credentials should be committed.
