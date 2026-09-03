# AWS DevOps Interview Handbook – Set 24


---
🚀 𝗔𝗪𝗦 𝗗𝗲𝘃𝗢𝗽𝘀 𝗜𝗻𝘁𝗲𝗿𝘃𝗶𝗲𝘄 𝗦𝗲𝗿𝗶𝗲𝘀 – 𝗦𝗲𝘁 𝟮𝟰

So as a part of my interview preparation journey, I have gathered a set of DevOps interview questions from various sources, which I will be sharing regularly, If you are preparing for the interviews, especially in AWS DevOps, feel free to take a look.

1. Are you a part of requirement planning, Design, deployment or monitoring ? Which one ? 

Ans : I am mainly involved in deployment and monitoring activities. I work on CI/CD pipelines, application deployments, Kubernetes, AWS infrastructure, production support, and monitoring. I also participate in requirement discussions and provide inputs during design, but my primary responsibilities are deployment, production support, troubleshooting, and monitoring.

---

2. How are you communicating with your customers ? Through private IP or public IP ?
For an interview, you can answer like this:

It depends on the architecture and the type of customer access. For secure internal or private connectivity, we generally use private IPs through VPN, VPC peering, or dedicated connectivity. For customers accessing applications over the internet, we expose the application through a public endpoint such as a Load Balancer or DNS, rather than directly exposing the server’s public IP.

In production, we prefer private communication wherever possible and use controlled public endpoints only when external customer access is required.

Short answer:

“For internal communication, we use private IPs. For external customers, we use a public endpoint/load balancer with DNS, while keeping backend servers on private IPs.”

---

3. If it is private IP, Can you explain me in terms of VPC's ?

Yes. For an interview, explain it using a simple AWS VPC architecture.

Interview Answer

If we are communicating with customers through a private IP, the customer and our application need to have private network connectivity.

For example, suppose our application is running inside an AWS VPC in a private subnet. The application has a private IP, such as 10.0.2.10.

If the customer also has their application/network in another VPC or an on-premises data center, we can establish connectivity using VPC Peering, Transit Gateway, VPN, or Direct Connect.

Once the network connectivity and routing are configured, the customer can access our application using its private IP or private DNS name, without exposing the application directly to the public internet.

Example
Customer VPC
10.1.0.0/16
     |
     | VPN / VPC Peering / Transit Gateway
     |
     ↓
AWS Application VPC
10.0.0.0/16
     |
     ↓
Private Subnet
10.0.2.0/24
     |
     ↓
Application
10.0.2.10

For example:

Customer → 10.0.2.10:8080

The important pieces are:

VPC CIDRs → connectivity → route tables → security groups → application

⭐ Good interview statement

“In a private-IP communication model, our application resides in a private subnet inside a VPC. Customer connectivity is established through VPN, VPC Peering, Transit Gateway, or Direct Connect depending on the architecture. We configure the appropriate routes and security-group rules so the customer can reach the application over its private IP or private DNS, without exposing the backend to the internet.”

Interview tip: If they ask “Which one have you personally worked on?”, don't claim all of these. Mention the specific AWS networking components you have actually worked with.

Yes. For your profile, I would recommend you confidently focus on AWS VPC Peering as the one concept to explain deeply.

🎯 Interview Question

“Which private connectivity method have you personally worked with?”

You can answer:

I have worked with VPC Peering for private communication between AWS VPCs.
For example, if an application is running in one VPC and another application or customer environment is in a different VPC, we can establish a VPC Peering connection between them.

After creating the peering connection, we configure the route tables in both VPCs so that traffic destined for the other VPC's CIDR is routed through the peering connection. We also configure security groups and network ACLs as required.

This allows the applications to communicate using their private IP addresses, without sending the traffic over the public internet.

Simple architecture
Customer VPC
10.1.0.0/16
     |
     | VPC Peering
     |
     ↓
Application VPC
10.0.0.0/16
     |
     ↓
Private Subnet
     |
     ↓
App Server
10.0.2.10
🔥 If interviewer asks: “What did you configure?”

Say:

“I mainly worked with VPC CIDRs, VPC Peering, route tables, and security groups. I made sure the routes were configured on both sides and that the required application ports were allowed through the security groups.”

Remember this flow

VPC → Peering → Route Table → Security Group → Private IP Communication

That's a very good answer for a DevOps/SRE interview because it shows you understand not just VPC Peering, but how traffic actually flows.

---


4. A EC2 instance created in a private subnet, What would you do to connect to that EC2 instance, How can you configure the connectivity.
Yes. This is a very common AWS interview question.

Question

An EC2 instance is created in a private subnet. How would you connect to that EC2 instance? How would you configure the connectivity?

There are several approaches. The most common is Bastion Host / Jump Server.

1. Bastion Host — most common answer

Architecture:

                    Internet
                       |
                    Internet
                    Gateway
                       |
                Public Subnet
                       |
                 Bastion Host
                  (Public EC2)
                       |
                  Private IP
                       |
                Private Subnet
                       |
                 Private EC2

You configure:

Bastion Host

EC2 in a public subnet
Public/Elastic IP
Security Group allows SSH 22 from your trusted IP only.

Private EC2

No public IP
Security Group allows SSH 22 from the Bastion Host's Security Group

Example:

Bastion-SG
Inbound:
TCP 22
Source: My-Office-IP/32
Private-EC2-SG
Inbound:
TCP 22
Source: Bastion-SG

Then:

ssh -i key.pem ec2-user@<bastion-public-ip>

From the Bastion:

ssh -i key.pem ec2-user@<private-ec2-private-ip>
2. AWS Systems Manager Session Manager — preferred in modern environments

You can avoid a Bastion Host altogether.

Your Laptop
     |
     v
AWS Systems Manager
     |
     v
Private EC2

Requirements include:

SSM Agent running on the EC2
IAM instance role with appropriate SSM permissions
Network connectivity from the private instance to SSM endpoints, typically through VPC endpoints or suitable outbound connectivity such as a NAT Gateway.

Then:

aws ssm start-session --target <instance-id>

This is often the more secure approach because you don't need to expose SSH to the internet.

3. VPN / Direct Connect

For corporate environments:

Corporate Network
       |
   VPN / Direct Connect
       |
      VPC
       |
Private EC2

Users connect to the corporate network/VPN and then access the EC2 through its private IP.

Interview answer

You can say:

"If the EC2 instance is in a private subnet, it doesn't have a public IP, so I cannot directly SSH to it from the internet. I can configure connectivity using a Bastion Host in a public subnet, allowing SSH from my trusted IP to the Bastion and allowing SSH from the Bastion Security Group to the private EC2. Alternatively, I would prefer AWS Systems Manager Session Manager because it provides secure access without exposing SSH or deploying a Bastion Host. In a corporate environment, I can also use Site-to-Site VPN or Direct Connect to access the private subnet."

⭐ Remember
Private EC2
   |
   +-- Bastion Host
   |
   +-- SSM Session Manager
   |
   +-- VPN
   |
   +-- Direct Connect

Best interview keywords:
Private IP + Security Groups + Route Tables + Bastion Host + SSM + VPN/Direct Connect.

---

5. Suppose your clients wants to connect to 40 EC2 instance out of 100 EC2 instances, which is configured in a private subnet under the VPN, How can you provide connectivity ?
This is a very common AWS networking interview question. The key is: you don't give the client access to all 100 EC2 instances. You control access to only the required 40.

Interview answer

If the client needs connectivity to only 40 out of 100 EC2 instances, I would keep all EC2 instances in private subnets and provide VPN connectivity to the VPC.

Then I would control which instances the client can access using security groups and routing.

I would place the 40 required EC2 instances behind a dedicated security group, for example client-access-sg. The VPN client traffic would be allowed only to this security group on the required application ports, such as 22, 80, or 443.

The remaining 60 instances would not allow traffic from the client's VPN CIDR in their security groups. Therefore, even though the VPN provides connectivity to the VPC, the client can reach only the authorized 40 instances.

Architecture
                 Client Network
                192.168.10.0/24
                       |
                       |
                 Site-to-Site VPN
                       |
                       ↓
              AWS Virtual Private Gateway
                       |
                       ↓
                 AWS VPC
              10.0.0.0/16
                       |
          ┌────────────┴────────────┐
          ↓                         ↓
    Private Subnet A          Private Subnet B
       40 EC2                    60 EC2
          |                         |
    client-access-sg          restricted-sg
          |                         |
       ALLOWED                    DENIED
🔑 What controls the access?

Think about these 4 layers:

VPN → establishes private connectivity between client and AWS VPC.
Route tables → ensure traffic can reach the required subnet/instances.
Security Groups → allow client VPN CIDR only to the 40 authorized instances.
Network ACLs → additional subnet-level control if required.
⭐ Important interview point

Don't say:

"I'll create VPN access to 40 EC2 instances."

Instead say:

"The VPN provides connectivity to the VPC. I restrict access to the required 40 instances using routing and security-group rules."

That's the technically stronger answer.

If interviewer asks: “How exactly will you identify those 40?”

You can say:

“I would put those 40 instances in dedicated private subnets or, preferably, associate them with a dedicated security group such as client-access-sg. The VPN client CIDR would be allowed only in that security group's inbound rules. The other 60 instances would have different security groups without that rule.”

One important correction: Security groups aren't attached to a subnet; they're attached to ENIs/instances. So for 40 selected EC2 instances, a dedicated security group is usually the cleanest control.

---

6. How can you explain the security aspects to your clients, who wants to migrate from on-prem to public cloud ?
For an interview, answer this as a security approach during cloud migration, not just as a list of AWS security services.

🎯 Interview Answer

When a client wants to migrate from on-premises to public cloud, I first understand their security requirements, compliance requirements, applications, data classification, and existing network architecture.

From the AWS side, I follow a defense-in-depth and least-privilege approach.

At the network level, I would design the VPC with public and private subnets. Internet-facing components such as load balancers can be placed in public subnets, while application servers and databases remain in private subnets. We use Security Groups, Network ACLs, route tables, VPN or Direct Connect as required.

At the identity level, I use IAM with least-privilege permissions, MFA, and role-based access rather than sharing credentials.

For data security, I use encryption both at rest and in transit. AWS KMS can be used for encryption-key management, and TLS can be used for application communication.

For monitoring and auditing, I would use services such as CloudTrail, CloudWatch, and security monitoring tools to detect and investigate suspicious activity.

Finally, I would regularly review access, patching, vulnerabilities, backups, logging, and compliance requirements to maintain the security posture after migration.

🧠 Easy way to remember
             CLOUD SECURITY
                   |
     ┌─────────────┼─────────────┐
     ↓             ↓             ↓
   Network       Identity       Data
     |             |             |
   VPC          IAM/Roles      Encryption
   SG           MFA            KMS
   NACL         Least          TLS
   VPN          Privilege
     |
     └─────────────┬─────────────┘
                   ↓
          Monitoring & Audit
          CloudTrail / CW
                   ↓
             Compliance
🔥 If interviewer asks: “What will you check before migration?”

Say:

“I would first perform a security assessment of the existing on-prem environment, identify sensitive data and compliance requirements, map existing firewall and access rules, design the AWS network and IAM model, define encryption and logging requirements, and then validate the security controls before moving workloads to production.”

⭐ One strong sentence

“My approach is to follow the shared-responsibility model, least privilege, defense in depth, encryption, network isolation, and continuous monitoring.”

That sentence is excellent to use at the end of your answer.

---



7. Say the clients from their company land (Company internet) wants to migrate to cloud (public cloud), What you can suggest for this in terms of security ?
For an interview, I would answer this in terms of secure connectivity from the company's on-premises network to the public cloud.

🎯 Interview Answer

If the client is currently using their company network/internet and wants to migrate their applications to a public cloud like AWS, I would not expose the internal applications directly to the public internet.

First, I would design a secure VPC with public and private subnets. Internet-facing components such as a Load Balancer can be placed in the public subnet, while application servers and databases can remain in private subnets.

For connectivity between the company's on-premises network and AWS, I would use Site-to-Site VPN initially. If the client requires high bandwidth and consistent private connectivity, I would consider AWS Direct Connect.

I would also implement Security Groups, Network ACLs, IAM least privilege, encryption, MFA, logging and monitoring.

So the overall approach would be secure private connectivity + network segmentation + least-privilege access + encryption + continuous monitoring.

Simple architecture
Company On-Premises
       |
       | Secure VPN
       | or Direct Connect
       ↓
   AWS VPC
 10.0.0.0/16
       |
 ┌─────┴──────────┐
 ↓                ↓
Public Subnet   Private Subnet
     |                |
   ALB            App Servers
                      |
                   Database
                  (Private)
⭐ If they ask: "Why not use the company internet directly?"

Say:

“For sensitive enterprise workloads, I would avoid direct exposure of private applications to the public internet. VPN or Direct Connect provides controlled connectivity, while Security Groups and IAM restrict who can access the resources.”

🧠 Remember this for the interview

On-prem → VPN/Direct Connect → VPC → Private Subnet → Application → Database

And for security:

IAM + SG + NACL + Encryption + Logging + Monitoring.


8. How can you explain the different layers in cloud to the client while migrating from on-prem to cloud, e.g. network layer(in terms of network)?
For this interview question, explain the migration in layers. The interviewer is mainly checking whether you understand how an on-prem application maps to cloud architecture.

🎯 Interview Answer

When migrating an application from on-premises to the public cloud, I explain the architecture layer by layer. This helps the client understand where security and connectivity controls are implemented.

First is the Network Layer. We design the AWS VPC, CIDR ranges, public and private subnets, route tables, Internet Gateway, NAT Gateway, Security Groups and Network ACLs. For connectivity with the on-premises environment, we can use Site-to-Site VPN or Direct Connect.

Second is the Compute Layer. Depending on the application, we can use EC2, Auto Scaling, containers or Kubernetes. We keep application servers in private subnets where possible.

Third is the Application Layer. We deploy the application using services such as EC2, ECS or EKS and expose it through a Load Balancer when required.

Fourth is the Database and Storage Layer. Databases such as RDS can be deployed in private subnets, and storage can use services such as S3, EBS or EFS depending on the requirement.

Fifth is the Security and Identity Layer. We use IAM, roles, MFA, Security Groups, encryption with KMS, secrets management and least-privilege access.

Finally, the Monitoring Layer provides visibility using CloudWatch, CloudTrail, logging and alerting.

So I would explain the migration as Network → Compute → Application → Data → Security → Monitoring, while maintaining security and availability at every layer.

🏗️ Simple architecture to remember
              ON-PREMISES
                  |
           VPN / Direct Connect
                  |
                  ↓
        ┌─────────────────────┐
        │     NETWORK LAYER   │
        │ VPC / Subnets       │
        │ Routes / SG / NACL  │
        └──────────┬──────────┘
                   ↓
        ┌─────────────────────┐
        │    COMPUTE LAYER    │
        │ EC2 / ECS / EKS     │
        └──────────┬──────────┘
                   ↓
        ┌─────────────────────┐
        │ APPLICATION LAYER   │
        │ ALB / Application   │
        └──────────┬──────────┘
                   ↓
        ┌─────────────────────┐
        │ DATA / STORAGE      │
        │ RDS / S3 / EBS      │
        └─────────────────────┘

       Security + IAM + Monitoring
             across all layers
⭐ If they specifically ask: “Explain the Network Layer”

Say:

“At the network layer, I start by designing the VPC and CIDR ranges. Then I divide the VPC into public and private subnets across multiple Availability Zones. Public subnets can contain internet-facing load balancers, while application and database servers remain private. I configure route tables, Internet/NAT Gateways, Security Groups and NACLs. For on-premises connectivity, I use VPN or Direct Connect. Finally, I make sure only the required traffic is allowed between the different layers.”

🧠 Interview shortcut

Remember:

VPC → Subnets → Routes → Connectivity → Security → Application

This is a strong way to answer because you're showing the client how traffic flows, not simply naming AWS services.

---


9. Can you create an IAM policy, which can identify whether a user has logged in via VPN connection from certain point to certain point, & can allow and deny the access ?
For this interview question, explain the migration in layers. The interviewer is mainly checking whether you understand how an on-prem application maps to cloud architecture.

🎯 Interview Answer

When migrating an application from on-premises to the public cloud, I explain the architecture layer by layer. This helps the client understand where security and connectivity controls are implemented.

First is the Network Layer. We design the AWS VPC, CIDR ranges, public and private subnets, route tables, Internet Gateway, NAT Gateway, Security Groups and Network ACLs. For connectivity with the on-premises environment, we can use Site-to-Site VPN or Direct Connect.

Second is the Compute Layer. Depending on the application, we can use EC2, Auto Scaling, containers or Kubernetes. We keep application servers in private subnets where possible.

Third is the Application Layer. We deploy the application using services such as EC2, ECS or EKS and expose it through a Load Balancer when required.

Fourth is the Database and Storage Layer. Databases such as RDS can be deployed in private subnets, and storage can use services such as S3, EBS or EFS depending on the requirement.

Fifth is the Security and Identity Layer. We use IAM, roles, MFA, Security Groups, encryption with KMS, secrets management and least-privilege access.

Finally, the Monitoring Layer provides visibility using CloudWatch, CloudTrail, logging and alerting.

So I would explain the migration as Network → Compute → Application → Data → Security → Monitoring, while maintaining security and availability at every layer.

🏗️ Simple architecture to remember
              ON-PREMISES
                  |
           VPN / Direct Connect
                  |
                  ↓
        ┌─────────────────────┐
        │     NETWORK LAYER   │
        │ VPC / Subnets       │
        │ Routes / SG / NACL  │
        └──────────┬──────────┘
                   ↓
        ┌─────────────────────┐
        │    COMPUTE LAYER    │
        │ EC2 / ECS / EKS     │
        └──────────┬──────────┘
                   ↓
        ┌─────────────────────┐
        │ APPLICATION LAYER   │
        │ ALB / Application   │
        └──────────┬──────────┘
                   ↓
        ┌─────────────────────┐
        │ DATA / STORAGE      │
        │ RDS / S3 / EBS      │
        └─────────────────────┘

       Security + IAM + Monitoring
             across all layers
⭐ If they specifically ask: “Explain the Network Layer”

Say:

“At the network layer, I start by designing the VPC and CIDR ranges. Then I divide the VPC into public and private subnets across multiple Availability Zones. Public subnets can contain internet-facing load balancers, while application and database servers remain private. I configure route tables, Internet/NAT Gateways, Security Groups and NACLs. For on-premises connectivity, I use VPN or Direct Connect. Finally, I make sure only the required traffic is allowed between the different layers.”

🧠 Interview shortcut

Remember:

VPC → Subnets → Routes → Connectivity → Security → Application

This is a strong way to answer because you're showing the client how traffic flows, not simply naming AWS services.

For cloud migration planning, a CMDB can help map and track infrastructure dependencies.
Ad
9. Can you create an IAM policy, which can identify whether a user has logged in via VPN connection from certain point to certain point, & can allow and deny the access ?


Yes. This is an IAM policy condition question. The important point is that IAM can restrict access based on the source IP address seen by AWS, but IAM itself does not directly determine “the user is connected through VPN.”

🎯 Interview Answer

Yes, we can create an IAM policy using the aws:SourceIp condition key to allow or deny access based on the source IP address.

For example, if the company's VPN uses a fixed public IP range, we can allow access only when the request originates from that IP range. If the request comes from outside that range, IAM can deny the access.

For more granular controls, we can also use conditions such as aws:SourceVpce for VPC endpoints or other identity/network conditions depending on the architecture.

Example IAM Policy

Suppose the company's VPN exits through:

203.0.113.0/24

We can allow access only from that range:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::company-private-bucket/*",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": "203.0.113.0/24"
        }
      }
    }
  ]
}
What happens?
User
  |
  ↓
Company VPN
  |
  ↓
VPN Public IP
203.0.113.x
  |
  ↓
AWS
  |
  ↓
IAM Policy
  |
  ├── Source IP matches → ✅ Allow
  |
  └── Source IP doesn't match → ❌ Deny
🔥 Very important interview point

Don't say:

❌ "IAM identifies whether the user is connected to VPN."

Say:

✅ "IAM can evaluate the source IP of the request. If our VPN has a known/fixed egress IP, we can use aws:SourceIp to restrict access to requests coming through that network."

Also, “from certain point to certain point” can mean a time period. If the interviewer means time-based access, that's a different IAM condition using aws:CurrentTime / DateGreaterThan / DateLessThan.

---

10. How would create 5 different AWS accounts for a development team, how would you plan an authentication and authorization and what's the different between them ?
This is a good AWS IAM + AWS Organizations interview question. The key is to separate authentication from authorization.

🎯 Interview Answer

If I need to create 5 different AWS accounts for a development team, I would manage them centrally using AWS Organizations rather than managing each account independently.

I would create separate accounts based on environments or workloads, for example:

AWS Organization
      |
      ├── Dev Account
      ├── QA Account
      ├── Test Account
      ├── Staging Account
      └── Shared Services Account

For authentication, I would use AWS IAM Identity Center integrated with the company's identity provider, such as Microsoft Entra ID/Active Directory. Developers would use their corporate identity and SSO, instead of creating individual IAM users in every AWS account.

For authorization, I would create permission sets such as Developer, ReadOnly, and Administrator and assign them to the appropriate users or groups for each AWS account.

I would also use Service Control Policies (SCPs) through AWS Organizations to establish guardrails across accounts. For example, we can prevent developers from disabling CloudTrail or using certain restricted AWS services.

This gives us centralized authentication, controlled authorization, account isolation, and consistent security policies.

🔐 Authentication vs Authorization
Authentication	Authorization
Who are you?	What are you allowed to do?
Verifies identity	Determines permissions
IAM Identity Center / SSO	IAM policies / Permission Sets
Corporate IdP	Roles and policies
Login	Access to AWS resources
Example

Suppose Newton is a developer.

Newton
   |
   ↓
Corporate SSO
   |
   ↓
IAM Identity Center
   |
   ↓
Developer Permission Set
   |
   ├── Dev Account    → Full development access
   ├── QA Account     → Limited access
   ├── Test Account   → Limited access
   ├── Staging        → ReadOnly
   └── Production     → No access
⭐ Strong interview point

If they ask:

“Why create 5 accounts instead of 5 VPCs?”

Say:

“Separate AWS accounts provide stronger isolation for security, billing, access control, quotas, and operational boundaries. We can still use multiple VPCs within each account when required.”

🧠 Remember this flow

Organizations → Accounts → IAM Identity Center → Groups → Permission Sets → SCP Guardrails

And the simplest way to remember the difference:

Authentication = Who are you?
Authorization = What can you access?

11. Linux command for
 • The current shell in which your working on
 • Default shell of CentOS and Linux OS
 • Create a file and edit a file 
 
12. Have you done any security patching in the project, If yes how ?
Yes. For an interview, you can answer based on a typical DevOps/SRE production environment like this:

🎯 Interview Answer

Yes, I have been involved in security patching activities in production environments. The process generally starts with identifying vulnerabilities from security or vulnerability-scanning reports.

We first analyze the affected servers, operating systems, packages, and application dependencies and prioritize the vulnerabilities based on their severity, such as Critical or High.

Before applying patches, we take the necessary backup/snapshot and test the patches in a lower environment such as Dev or QA. After validation, we raise a change request and schedule the production activity during an approved maintenance window.

For Linux servers, we use package-management tools such as yum or dnf to update the required packages. For example:

sudo yum update

or for a specific package:

sudo yum update <package-name>

After patching, we verify the server status, application health, services, logs, and connectivity. We then perform vulnerability scanning again to confirm that the vulnerability has been remediated.

If there is any issue, we follow the rollback plan using the backup or snapshot.

🔄 Simple flow to remember
Vulnerability Scan
       ↓
Identify & Prioritize
       ↓
Backup / Snapshot
       ↓
Test in Dev / QA
       ↓
Change Request
       ↓
Production Maintenance Window
       ↓
Apply Security Patch
       ↓
Application Validation
       ↓
Rescan
       ↓
Close Vulnerability
⭐ If they ask: “What tools did you use?”

You can mention only the tools you have actually used. A safe answer is:

“We used vulnerability scanning reports to identify the vulnerabilities, Linux package-management tools for OS patching, and our change-management process for production implementation. After patching, we validated the application and performed a rescan.”

Interview tip: Don't say you personally performed patching with a particular tool unless you actually did. If your role was mainly coordination, validation, and production support, say that clearly.




13. How can you restricts the malicious IP's in EC2 instances ?
13. How can you restrict malicious IPs in EC2 instances?

For an interview, give a layered security answer:

Yes. If we identify a malicious IP trying to access our EC2 instances, we can block it at different layers depending on the architecture.

At the AWS network level, we can use AWS Network Firewall or AWS WAF when the traffic is going through a supported load balancer/application layer. We can also use Security Groups to allow only trusted IP ranges, but Security Groups are not designed as a large IP blacklist.

At the subnet level, we can use Network ACLs to deny specific malicious IP addresses.

At the EC2 operating-system level, we can use the Linux firewall such as iptables, nftables, or firewalld to block an IP directly on the server.

For a large or continuously changing list of malicious IPs, I would prefer an automated security solution rather than manually maintaining rules on every EC2 instance.

Example: Linux firewalld

To block:

203.0.113.50

you could use:

sudo firewall-cmd --permanent \
  --add-rich-rule='rule family="ipv4" source address="203.0.113.50" reject"

sudo firewall-cmd --reload

Verify:

sudo firewall-cmd --list-all
🧠 Architecture
Internet
    |
    ↓
AWS WAF / Network Firewall
    |
    ↓
Load Balancer
    |
    ↓
Security Group
    |
    ↓
Network ACL
    |
    ↓
EC2
    |
    ↓
OS Firewall
(firewalld / nftables)
⭐ Strong interview answer

“I would first identify where the malicious traffic enters the architecture. For web applications, I would prefer WAF-based protection; for network-level filtering, Network Firewall or NACLs can be used; and for host-level protection, I can use the Linux firewall. I would also automate detection and blocking where possible instead of manually blocking IPs on individual EC2 instances.”

Important: Don't say “Security Group can deny a malicious IP.” AWS Security Groups are allow-list based and don't support explicit deny rules. For explicit IP blocking, NACLs or appropriate firewall/WAF controls are better.

---

14. Which scripting knowledge you have ,Where have you used python ?
For your DevOps/SRE interview, keep this answer practical and don't overclaim advanced Python.

🎯 Interview Answer

I have scripting experience mainly with Bash/Shell scripting and Python. I have used scripting for automation, monitoring, log analysis, health checks, and routine operational tasks.

With Python, I have mainly used it for automating repetitive tasks and working with APIs. For example, I have written scripts to check application or server status, process logs, retrieve information from AWS services, and perform basic automation.

In DevOps, I have also used Bash scripting for Linux administration, deployment activities, file operations, service checks, and troubleshooting.

Python is particularly useful when the automation requires more complex logic, API integration, JSON processing, or interaction with AWS services.

🔥 If interviewer asks: “Where exactly did you use Python?”

You can say:

“I used Python mainly for operational automation. For example, instead of manually checking multiple servers or application endpoints, I can write a Python script to check their status and generate a report. I have also used Python for API calls and processing JSON responses from cloud services.”

Example Python you can discuss

Application health check:

import requests

url = "http://application:8080/health"

response = requests.get(url, timeout=5)

if response.status_code == 200:
    print("Application is UP")
else:
    print("Application is DOWN")
🧠 Remember for interview
Bash
 ├── Linux administration
 ├── Deployment
 ├── Service checks
 ├── File operations
 └── Troubleshooting

Python
 ├── Automation
 ├── API calls
 ├── JSON processing
 ├── Health checks
 └── AWS automation
⭐ One-line answer

“I have hands-on experience with Bash and Python. I mainly use Bash for Linux and deployment automation, while Python is useful for API integration, AWS automation, health checks, and processing operational data.”

---

15. Can you name some of the modules you have used in python for project ?
I have used Python modules such as requests, boto3, json, os, subprocess, logging, and datetime.

requests – for REST API calls and health checks
boto3 – for interacting with AWS services
json – for parsing and processing JSON responses
os – for environment variables and file/system operations
subprocess – for executing Linux commands from Python
logging – for application and automation logs
datetime – for date and time-related operations
⭐ Strong interview answer

“In my DevOps automation work, I have mainly used boto3 for AWS automation, requests for API calls and health checks, json for processing API responses, os and subprocess for system-level automation, and logging and datetime for operational scripts.”

Example
import boto3
import requests
import json
import os
import subprocess
import logging
from datetime import datetime

Interview tip: If they ask “Which module did you use most?”, say boto3 if you have actually worked with AWS automation. Then be ready to explain one small use case, such as retrieving EC2 instance information or checking AWS resources.

16. How do you manage the state-file in your environment ?
16. How do you manage the Terraform state file in your environment?

For a DevOps interview, a strong answer is:

We manage the Terraform state file using a remote backend, rather than storing terraform.tfstate locally or committing it to Git.

In AWS, I prefer using an S3 bucket as the remote backend. The state file is stored centrally, encrypted, and access is controlled through IAM.

For state locking, depending on the Terraform/backend setup, we use the supported locking mechanism so that two engineers or pipelines don't modify the state simultaneously.

We also enable S3 versioning so that previous versions of the state can be recovered if required.

Access to the state bucket is restricted to authorized users and the CI/CD pipeline because the state file can contain sensitive infrastructure information.

Before making changes, we run terraform plan, review the changes, and then use the CI/CD pipeline to apply them.

🏗️ Typical Architecture
Developer / Jenkins
       |
       | terraform plan/apply
       ↓
   Terraform
       |
       ↓
+--------------------+
|   S3 Backend       |
|                    |
| terraform.tfstate  |
| Encryption         |
| Versioning         |
+--------------------+
       |
       ↓
   AWS Resources
Example backend
terraform {
  backend "s3" {
    bucket = "company-terraform-state"
    key    = "dev/terraform.tfstate"
    region = "ap-south-1"

    encrypt = true
  }
}
🔥 Important interview points

Remember these:

Remote Backend
→ S3

Encryption
→ Protect the state

Versioning
→ Recover previous state

Access Control
→ IAM

Locking
→ Prevent concurrent state modifications

CI/CD
→ Controlled plan and apply

⭐ Short interview answer

“We store Terraform state remotely in an encrypted S3 backend with versioning and controlled IAM access. We use state locking to prevent concurrent modifications and never commit the state file to Git. Terraform plan is reviewed before apply, and production changes are normally executed through our CI/CD pipeline.”

---

17. What is state-file locking, how can you achieve that ?
17. What is Terraform state-file locking, and how can you achieve it?
🎯 Interview Answer

State-file locking prevents multiple users or CI/CD pipelines from modifying the same Terraform state at the same time.

For example, if two engineers run terraform apply simultaneously, both could try to update the same state file, which can cause state corruption or inconsistent infrastructure.

To avoid this, Terraform uses a state-locking mechanism provided by the backend. When one process is modifying the state, Terraform acquires a lock. Another process trying to modify the same state has to wait or fails until the lock is released.

AWS Example

A common setup is:

             Terraform
                 |
          terraform apply
                 |
                 ↓
        ┌─────────────────┐
        │   S3 Backend     │
        │                  │
        │ terraform.tfstate│
        └────────┬────────┘
                 |
          State Locking

For current Terraform with an S3 backend, S3 backend locking can be enabled with:

terraform {
  backend "s3" {
    bucket       = "company-terraform-state"
    key          = "prod/terraform.tfstate"
    region       = "ap-south-1"
    encrypt      = true
    use_lockfile = true
  }
}

Then initialize:

terraform init
🔥 Interview example

“Suppose Jenkins is running terraform apply for production and at the same time another engineer runs terraform apply. The first process acquires the state lock. The second process cannot modify the same state simultaneously. Once the first operation completes, the lock is released.”

⚠️ Important distinction for interviews

You may hear the older AWS architecture:

S3 → State File
DynamoDB → State Lock

Historically, Terraform commonly used a DynamoDB table for S3 backend locking. For current Terraform S3 backends, S3 lockfiles (use_lockfile = true) are the current locking mechanism, while DynamoDB-based locking is deprecated.

⭐ Short answer to remember

“State locking prevents concurrent Terraform operations from modifying the same state file. In our AWS environment, we use an S3 remote backend with state locking enabled. This ensures only one Terraform operation can modify a particular state at a time.”

---


18. How can you fix the manual done changes to EC2 instances, in terraform ?
18. How can you fix manually done changes to EC2 instances in Terraform?

This is a Terraform drift question.

🎯 Interview Answer

If someone manually changes an EC2 instance outside Terraform—for example, changing the instance type, security group, tags, or other Terraform-managed configuration—it creates configuration drift.

I would first run:

terraform plan

Terraform compares the current AWS infrastructure with the configuration and state it knows about and shows the difference.

If the Terraform code represents the configuration we actually want, I would review the plan and run:

terraform apply

Terraform will try to bring the infrastructure back to the configuration defined in the .tf files.

If the manual change was intentional and should become the new standard, I would update the Terraform code to reflect that change, then run terraform plan and terraform apply.

So, I don't simply overwrite every manual change—I first determine whether the change was unauthorized drift or an approved change.

Example

Terraform says:

instance_type = "t3.medium"

But someone manually changes the EC2 instance to:

t3.large

Run:

terraform plan

Terraform may show:

~ instance_type = "t3.large" -> "t3.medium"

If t3.medium is the desired configuration:

terraform apply

Terraform changes it back.

🔥 If the manual change should be retained

Suppose the team intentionally changed:

t3.medium → t3.large

Then update Terraform:

instance_type = "t3.large"

and run:

terraform plan
terraform apply

Now Terraform and AWS are aligned.

🧠 Remember this for interview
Manual AWS Change
       ↓
Configuration Drift
       ↓
terraform plan
       ↓
Is change intentional?
     ↙       ↘
   NO         YES
   ↓           ↓
apply       Update .tf
   ↓           ↓
AWS ← Terraform → AWS
⭐ Short interview answer

“I handle manual EC2 changes as Terraform drift. I first run terraform plan to identify the difference. If the manual change is unauthorized, I apply the Terraform configuration to restore the desired state. If it was an approved change, I update the Terraform code first and then apply it. This keeps Terraform as the source of truth.”

---

19. Why do we need terraform module ?
19. Why do we need Terraform Modules?
🎯 Interview Answer

We use Terraform modules to organize and reuse infrastructure code. Instead of writing the same Terraform configuration repeatedly for different environments or projects, we create a reusable module and pass different input variables.

Modules help us maintain reusability, consistency, standardization, and easier maintenance.

For example, if we need to create VPCs for Dev, QA, and Production, instead of writing three separate VPC configurations, we can create one reusable VPC module and provide different CIDRs and environment-specific values.

Example
Terraform Project
       |
       ├── modules/
       │    ├── vpc/
       │    ├── ec2/
       │    └── rds/
       |
       ├── dev/
       ├── qa/
       └── prod/

You could have one VPC module:

module "vpc" {
  source = "./modules/vpc"

  vpc_cidr = "10.0.0.0/16"
  environment = "dev"
}

For QA:

module "vpc" {
  source = "./modules/vpc"

  vpc_cidr = "10.1.0.0/16"
  environment = "qa"
}

Same module, different inputs.

🔥 Main benefits
Benefit	Meaning
Reusability	Write once, use multiple times
Standardization	Same architecture across environments
Maintainability	Change module in one place
Consistency	Reduces configuration differences
Scalability	Easier to manage large infrastructure
Organization	Keeps Terraform code structured
⭐ Strong interview answer

“Terraform modules allow us to create reusable and standardized infrastructure components. For example, we can create a common VPC module and reuse it across Dev, QA, and Production by passing different variables. This reduces code duplication, improves consistency, and makes infrastructure easier to maintain.”

🧠 Remember

Module = Write once → Reuse many times → Standardize infrastructure
20. How you are deploying the changes in Jenkins ?
20. How are you deploying changes through Jenkins?

For your DevOps/SRE interview, you can answer like this:

We use Jenkins as the CI/CD automation tool. When a developer commits and pushes code to Git, Jenkins is triggered through a webhook.

First, Jenkins checks out the code from Git. Then we perform build and unit testing, followed by code-quality and security scans such as SonarQube, Snyk or Trivy depending on the project.

After successful validation, we build a Docker image, tag it with the build number or Git commit ID, and push it to Amazon ECR.

Then Jenkins deploys the new image to the target environment, such as Kubernetes/EKS. We update the Deployment with the new image and monitor the rollout.

After deployment, we perform health checks and verify the application. For production, we normally use an approval gate/change process before deployment.

🔄 Jenkins deployment flow
Developer
    ↓
Git Push
    ↓
Jenkins Webhook
    ↓
Checkout Code
    ↓
Build
    ↓
Unit Tests
    ↓
SonarQube / Security Scan
    ↓
Docker Build
    ↓
Push Image → ECR
    ↓
Approval (Production)
    ↓
Deploy → Kubernetes / EKS
    ↓
Rollout Status
    ↓
Health Check
Example Kubernetes deployment

Jenkins can execute:

kubectl set image deployment/frontend \
  frontend=123456789.dkr.ecr.ap-south-1.amazonaws.com/frontend:$BUILD_NUMBER \
  -n production

Then verify:

kubectl rollout status deployment/frontend -n production

If the deployment has a problem:

kubectl rollout undo deployment/frontend -n production
⭐ Strong interview answer

“We follow a CI/CD pipeline where Jenkins checks out the code, builds and tests it, performs quality and security scans, builds and pushes the Docker image to ECR, and then deploys the image to Kubernetes. We verify the rollout and application health after deployment. For production, we have an approval gate and follow the change-management process.”

🧠 Remember

Git → Jenkins → Build → Test → Scan → Docker → ECR → Deploy → Verify → Rollback

21. Is it terraform, scripting, CLI which one your using to deploy in Jenkins ?
21. Is it Terraform, scripting, or CLI that you use to deploy through Jenkins?

For your interview, I recommend answering CLI + Terraform, depending on what is being deployed.

We use both Terraform and CLI commands in Jenkins, depending on the deployment requirement.

Terraform is mainly used for infrastructure provisioning and changes, such as creating or modifying VPCs, EC2, security groups, IAM, and other AWS resources.

For application deployment, especially Kubernetes deployments, we use CLI commands such as kubectl from the Jenkins pipeline. Jenkins deploys the Docker image that was built and pushed to ECR.

We also use shell scripting within Jenkins for small automation and to execute commands.

Example
Jenkins
   |
   ├── Terraform
   |      └── Infrastructure
   |          VPC / EC2 / IAM / SG
   |
   ├── Shell Script
   |      └── Automation
   |
   └── kubectl CLI
          └── Application Deployment
              Kubernetes / EKS
If interviewer asks: "Which one do you mainly use?"

Say:

“For infrastructure, we primarily use Terraform. For application deployment to Kubernetes, we use kubectl CLI commands through Jenkins. Shell scripting is used to automate supporting tasks.”

⭐ Example Jenkins stages
stage('Terraform') {
    steps {
        sh 'terraform plan'
        sh 'terraform apply -auto-approve'
    }
}

stage('Deploy') {
    steps {
        sh 'kubectl set image deployment/frontend frontend=$IMAGE'
        sh 'kubectl rollout status deployment/frontend'
    }
}
🧠 Easy interview formula

Infrastructure → Terraform
Application → kubectl/CLI
Automation → Shell/Python

This is a much better answer than saying “we use only Terraform”, because Terraform and kubectl solve different deployment problems.

---


22. How GitHub is connected to Jenkins? is it pull based or push based ?
22. How is GitHub connected to Jenkins? Is it pull-based or push-based?
🎯 Interview Answer

GitHub and Jenkins can be connected using different mechanisms, but in our CI/CD setup, we generally use a GitHub webhook, which is push-based triggering.

When a developer pushes code to GitHub, GitHub sends a webhook notification to Jenkins. Jenkins receives the event and then starts the pipeline.

However, once Jenkins starts, Jenkins pulls the actual source code from GitHub using Git.

So, technically:

Trigger = Push-based
Code retrieval = Pull-based

🔄 Flow
Developer
    |
    | git push
    ↓
GitHub Repository
    |
    | Webhook
    ↓
Jenkins
    |
    | git clone / git fetch
    ↓
Source Code
    |
    ↓
Build → Test → Scan → Deploy
⭐ Very important interview point

If interviewer asks:

“Is Jenkins pulling or GitHub pushing?”

Say:

“The trigger is push-based because GitHub sends a webhook to Jenkins when code is pushed. But Jenkins then pulls the source code from GitHub. So the trigger is push-based, while source-code checkout is pull-based.”

Another method

Jenkins can also poll GitHub periodically, for example:

Jenkins → GitHub
          "Any new changes?"

That's polling, but webhook-based integration is generally preferred because it triggers the pipeline immediately instead of waiting for the next polling interval.

---

23. Where your storing the project related secrets, tokens in the cloud environments ?
23. Where do you store project-related secrets and tokens in cloud environments?

For an AWS/DevOps interview, a strong answer is:

We don't store project secrets, passwords, API tokens, or database credentials directly in the source-code repository or in plain-text configuration files.

In AWS, we primarily use AWS Secrets Manager for sensitive credentials and secrets. For less-sensitive configuration values, we can use AWS Systems Manager Parameter Store.

Access to these secrets is controlled using IAM roles and least-privilege permissions. Applications running on EC2 or EKS retrieve the required secrets at runtime using their IAM role rather than hardcoding credentials.

We also make sure secrets are encrypted using AWS KMS, and access is audited through CloudTrail.

🔐 Example architecture
Developer
    |
    ↓
GitHub
(no passwords/tokens)
    |
    ↓
Jenkins
    |
    ↓
AWS Secrets Manager
    |
    ↓
IAM Role
    |
    ↓
Application
(EC2 / EKS)
Example

Instead of putting this in Git:

username: admin
password: MyPassword123

we store the credential in Secrets Manager:

DB_USERNAME
DB_PASSWORD
API_TOKEN

Then the application retrieves it securely at runtime.

⭐ If interviewer asks: "What about Jenkins credentials?"

Say:

“For Jenkins-specific credentials, we use the Jenkins Credentials Store or integrate Jenkins with a centralized secret-management solution. The pipeline accesses the credential through Jenkins rather than hardcoding it in the Jenkinsfile.”

🧠 Remember

Secrets Manager → passwords, API tokens, DB credentials

Parameter Store → configuration parameters / simpler secrets

IAM Role → controls who can retrieve them

KMS → encryption

CloudTrail → auditing

And most importantly:

“Never hardcode secrets in Git, Dockerfiles, Kubernetes YAML, or Jenkinsfiles.”

24. Where will you store secrets on a application level (if the application running in EC2) ? Any cloud native service ?
24. Where will you store secrets at the application level if the application is running on EC2?

A strong interview answer is:

If the application is running on an EC2 instance, I would use a cloud-native secret-management service such as AWS Secrets Manager rather than storing passwords or tokens directly in the application configuration or on the EC2 filesystem.

I would create the secret in AWS Secrets Manager and give the EC2 instance an IAM role with permission to retrieve only the required secret. The application can then retrieve the secret at runtime using the AWS SDK or AWS CLI.

The secret is encrypted, typically using AWS KMS, and access can be audited through CloudTrail.

Architecture
Application
     |
     | AWS SDK
     ↓
EC2 Instance
     |
     | IAM Role
     ↓
AWS Secrets Manager
     |
     ↓
Encrypted Secret
(DB Password / API Token)
Example

Instead of:

# ❌ Don't do this
db.username=admin
db.password=MyPassword123
api.token=abc123

Store them in:

AWS Secrets Manager
        ↓
DB_USERNAME
DB_PASSWORD
API_TOKEN

The EC2 instance gets permission through its IAM Instance Profile/Role.

⭐ If interviewer asks: "Why not environment variables?"

You can say:

“Environment variables are better than hardcoding secrets, but for sensitive production credentials I prefer a centralized secret-management service such as AWS Secrets Manager. It provides encryption, access control, rotation capabilities, and auditing.”

🧠 Remember

EC2 Application → IAM Role → Secrets Manager → KMS encryption

That's the cleanest cloud-native answer for this question.

---

25. What are the security feature of S3 , which we can apply to the application ?
25. What are the security features of S3 that we can apply to an application?

Interview-ready answer:

“For S3 security, I mainly focus on access control, encryption, and monitoring. We keep the bucket private and avoid public access. We use IAM roles and bucket policies with least privilege. We enable S3 Block Public Access, encrypt data using SSE-S3 or SSE-KMS, and enable versioning to protect against accidental deletion. For sensitive data, we can also use VPC endpoints so applications access S3 privately without going through the public internet. CloudTrail and S3 access logging can be used for auditing.”

Key S3 security features
Security Feature	Purpose
Block Public Access	Prevent accidental public exposure
IAM Roles/Policies	Control which application/users can access S3
Bucket Policy	Resource-level access control
SSE-S3 / SSE-KMS	Encryption at rest
TLS/HTTPS	Encryption in transit
Versioning	Protect against accidental deletion/overwrite
S3 VPC Endpoint	Private access from VPC
CloudTrail	Audit API activity
S3 Object Lock	Prevent deletion/modification for required retention
Least Privilege	Give application only required permissions
Example: EC2 application accessing S3
Application on EC2
       |
       | IAM Role
       ↓
     S3 Bucket
       |
       ├── Block Public Access
       ├── Bucket Policy
       ├── SSE-KMS Encryption
       ├── Versioning
       └── CloudTrail

Important interview point:
Don't give the EC2 application an AWS access key/secret key. Attach an IAM role to the EC2 instance and give that role only the required S3 permissions, such as s3:GetObject for a specific bucket/prefix.

One-line answer to remember:

“S3 security = Private bucket + Block Public Access + IAM least privilege + Bucket Policy + Encryption + HTTPS + Versioning + VPC Endpoint + Auditing.”

---