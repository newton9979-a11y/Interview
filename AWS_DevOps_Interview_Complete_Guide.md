# AWS DevOps Interview Series – Complete Answers & Explanations

# Table of Contents
1. Introduce Yourself
2. Git Branching Strategy
3. Git Merge Conflicts
4. CI/CD
5. Jenkinsfile
6. Scheduling Builds
7. Docker
8. ADD vs COPY
9. ENTRYPOINT vs CMD
10. Dockerfile
11. Sample Dockerfile
12. Enter Running Container
13. Kubernetes Architecture
14. Kubernetes vs Docker Swarm
15. Recover Lost Private Key
16. Connect EC2 Without Key
17. Private and Public Subnet Connectivity
18. Linux Permissions & chmod
19. Check Running Processes
20. Alternatives to ps
21. Check RAM Usage
22. Questions to Ask Interviewer
23. DevSecOps Questions
24. CI/CD Flowcharts

---

# 1. Introduce Yourself

Sample Answer:

Hi, I am an IT professional with experience in Linux Administration, AWS Cloud, Kubernetes, Docker, Jenkins, Git, and Production Support.

I have worked on CI/CD pipelines, containerization, infrastructure monitoring, troubleshooting production incidents, and automation using Shell scripting.

My experience includes:
- AWS (EC2, IAM, VPC, EBS, RDS, CloudWatch)
- Docker & Kubernetes
- Jenkins CI/CD
- Git & GitHub
- Linux Administration
- Monitoring & Troubleshooting

---

# 2. What is Branching Strategy in Git?

Branching strategy defines how teams manage code changes.

Popular Strategies:

## Git Flow

main
│
├── develop
│
├── feature/login
│
├── feature/payment
│
├── release/v1.0
│
└── hotfix/bug

### Advantages
- Structured
- Best for large teams

## Feature Branch Strategy

main
│
├── feature-login
├── feature-payment
└── feature-cart

Most companies use Feature Branch + Pull Requests.

---

# 3. Git Merge Conflicts

Occurs when same lines are modified by multiple developers.

Example:

Developer A:
print("Hello")

Developer B:
print("Hi")

Conflict appears during merge.

Resolution:

git status
git pull
Resolve manually
git add .
git commit

---

# 4. What is CI/CD?

CI = Continuous Integration

Developers frequently merge code.

CD = Continuous Delivery/Deployment

Automatically deploy code.

Flow:

Developer
↓
Git
↓
Jenkins
↓
Build
↓
Test
↓
Docker
↓
Kubernetes
↓
Production

Benefits:
- Faster releases
- Less manual work
- Reduced errors

---

# 5. Jenkinsfile

Example:

```groovy
pipeline {
 agent any

 stages {

  stage('Checkout'){
   steps{
    git 'https://github.com/sample/repo.git'
   }
  }

  stage('Build'){
   steps{
    sh 'mvn clean package'
   }
  }

  stage('Test'){
   steps{
    sh 'mvn test'
   }
  }

  stage('Docker Build'){
   steps{
    sh 'docker build -t app:latest .'
   }
  }

  stage('Deploy'){
   steps{
    sh 'kubectl apply -f deployment.yaml'
   }
  }
 }
}
```

---

# 6. How to Schedule Builds in Jenkins?

Build Triggers → Build periodically

Example:

```bash
H/5 * * * *
```

Runs every 5 minutes.

Cron Format:

Minute Hour Day Month Weekday

---

# 7. What is Docker?

Docker is a containerization platform.

Benefits:
- Lightweight
- Portable
- Fast deployment

Architecture:

Application
↓
Container
↓
Docker Engine
↓
OS

---

# 8. Difference Between ADD and COPY

COPY:
- Copies local files only

ADD:
- Copies files
- Extracts tar files
- Supports URLs

Example:

```dockerfile
COPY app.jar /app/

ADD app.tar.gz /app/
```

---

# 9. ENTRYPOINT vs CMD

ENTRYPOINT:
Fixed command.

CMD:
Default arguments.

Example:

```dockerfile
ENTRYPOINT ["java"]
CMD ["-version"]
```

Output:

java -version

---

# 10. What is Dockerfile?

Text file containing instructions to build Docker image.

Example:

FROM ubuntu
RUN apt install nginx

---

# 11. Sample Dockerfile

```dockerfile
FROM openjdk:17

WORKDIR /app

COPY target/app.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java","-jar","app.jar"]
```

Build:

```bash
docker build -t app .
```

Run:

```bash
docker run -d -p 8080:8080 app
```

---

# 12. How to Enter Running Container?

```bash
docker ps
docker exec -it container_id bash
```

Alternative:

```bash
docker exec -it container_id sh
```

---

# 13. Kubernetes Architecture

Master Plane:

- API Server
- Scheduler
- Controller Manager
- ETCD

Worker Node:

- Kubelet
- Kube Proxy
- Container Runtime

Flow:

User
↓
API Server
↓
Scheduler
↓
Worker Node
↓
Pod

---

# 14. Kubernetes vs Docker Swarm

| Feature | Kubernetes | Swarm |
|----------|------------|--------|
| Scaling | Advanced | Basic |
| Networking | Advanced | Simple |
| Community | Large | Smaller |
| Enterprise Usage | High | Medium |

---

# 15. Can Lost Private Key Be Recovered?

No.

AWS does not allow downloading private key again.

Best Practice:
- Create AMI
- Replace key pair
- Use SSM Session Manager

---

# 16. Connect EC2 if Key Lost

Method 1:
SSM Session Manager

Method 2:
Detach root volume
Attach to another EC2
Modify authorized_keys

Method 3:
Create new AMI

---

# 17. Connect Private Subnet with Public Subnet

Using Route Tables and NAT Gateway.

Internet
↓
IGW
↓
Public Subnet
↓
NAT Gateway
↓
Private Subnet

---

# 18. Linux Permissions and chmod

Permissions:

r = Read
w = Write
x = Execute

Example:

755

Owner = 7 (rwx)
Group = 5 (r-x)
Others = 5 (r-x)

```bash
chmod 755 file.sh
```

---

# 19. Command to Check Running Processes

```bash
ps -ef
```

or

```bash
ps aux
```

---

# 20. Other Commands

```bash
top
htop
pgrep
pidof
pstree
```

---

# 21. Command to Check RAM Usage

```bash
free -h
```

Also:

```bash
top
htop
vmstat
```

---

# 22. Questions to Ask Interviewer

- What are current DevOps challenges?
- Which cloud platform do you use?
- What CI/CD tools are used?
- How is on-call support managed?
- What growth opportunities exist?

---

# 23. DevSecOps Questions

## How do you secure a CI/CD Pipeline?

- RBAC
- MFA
- Secrets Manager
- Vulnerability Scanning
- Code Review

Tools:
- SonarQube
- Trivy
- Snyk

Flow:

Developer
↓
Git Scan
↓
Build
↓
Security Scan
↓
Deploy

---

## Security Challenges

- Secrets exposure
- Vulnerable images
- Misconfigured IAM
- Open Security Groups

---

## Prevent DDoS in AWS

Services:

- AWS Shield
- AWS WAF
- CloudFront
- Route53

---

## Compliance in DevSecOps

- Automated Audits
- AWS Config
- CloudTrail
- Security Hub

---

## Patch Automation

AWS Systems Manager Patch Manager

Flow:

Patch Baseline
↓
Maintenance Window
↓
Automatic Patching

---

## OWASP Top 10

- Injection
- Broken Authentication
- XSS
- SSRF
- Security Misconfiguration

Mitigation:
- Input Validation
- WAF
- Secure Coding
- Dependency Scanning

---

## Security in IaC

Tools:

- Checkov
- Terrascan
- tfsec

Scan Terraform before deployment.

---

## API Security

- OAuth
- JWT
- API Gateway
- Rate Limiting
- WAF

---

## DevSecOps Tools

- SonarQube
- Trivy
- Snyk
- Checkov
- OWASP ZAP
- AWS Security Hub

---

## Security Breach Handling

1. Identify Incident
2. Isolate Resources
3. Analyze Logs
4. Patch Vulnerability
5. Recover Systems
6. RCA Report

Flow:

Detection
↓
Containment
↓
Investigation
↓
Recovery
↓
Lessons Learned

---

# Final Interview Tip

Always explain:
- What
- Why
- How
- Real-time Example

This approach greatly improves AWS DevOps interview performance.
