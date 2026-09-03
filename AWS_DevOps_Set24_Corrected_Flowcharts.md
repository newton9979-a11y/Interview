# AWS DevOps Interview Handbook Set 24 (Corrected Flowchart Edition)

## Corrected AWS Private Connectivity Flow

```text
Customer Network / Customer VPC
10.1.0.0/16
        │
        │ VPN / VPC Peering / Transit Gateway
        ▼
AWS VPC
10.0.0.0/16
        │
        ▼
Route Tables
        │
        ▼
Security Groups
        │
        ▼
Private Subnet
10.0.2.0/24
        │
        ▼
Application Server
10.0.2.10
```

Traffic Flow:

```text
Customer
   │
   ▼
Connectivity (VPN / Peering / TGW)
   │
   ▼
Route Table
   │
   ▼
Security Group
   │
   ▼
Application
```

---

## Corrected Bastion Host Architecture

```text
Internet
   │
   ▼
Internet Gateway
   │
   ▼
Public Subnet
   │
   ▼
Bastion Host
(Public IP)
   │ SSH
   ▼
Private Subnet
   │
   ▼
Private EC2
(No Public IP)
```

Security Group Flow:

```text
Your Laptop
     │ TCP 22
     ▼
Bastion-SG
     │
     ▼
Private-EC2-SG
     │
     ▼
Private EC2
```

---

## Corrected AWS Systems Manager Architecture

```text
Administrator Laptop
          │
          ▼
AWS Console / AWS CLI
          │
          ▼
Systems Manager
          │
          ▼
SSM Agent
          │
          ▼
Private EC2
```

Requirements:

```text
IAM Role
    +
SSM Agent
    +
VPC Endpoint / NAT Gateway
    =
Session Manager Access
```

---

## Corrected VPN Access Architecture

```text
Client Network
192.168.10.0/24
         │
         ▼
Site-to-Site VPN
         │
         ▼
Virtual Private Gateway
         │
         ▼
AWS VPC
10.0.0.0/16
         │
         ▼
Route Tables
         │
         ▼
Security Groups
         │
 ┌───────┴────────┐
 ▼                ▼

40 EC2          60 EC2
client-sg       default-sg
Allowed         Blocked
```

---

## Production AWS Architecture (Interview Grade)

```text
Users
  │
  ▼
Route53
  │
  ▼
CloudFront
  │
  ▼
AWS WAF
  │
  ▼
Application Load Balancer
  │
  ▼
Private Subnets
  │
  ▼
EC2 / EKS
  │
  ▼
RDS Multi-AZ
  │
  ▼
Encrypted Storage

Security Layers
────────────────
IAM
Security Groups
NACL
KMS
CloudTrail
CloudWatch
GuardDuty
```

## Terraform Update (2026)

Recommended:
- S3 Remote Backend
- Versioning Enabled
- Encryption Enabled
- use_lockfile = true

Legacy:
- DynamoDB locking should only be mentioned for older implementations.

