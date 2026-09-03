# SonarQube, Snyk & Trivy – DevOps Security Interview Guide

## 1. DevOps CI/CD Security Flow

A common DevSecOps pipeline can look like this:

```text
Developer
   |
   v
GitHub
   |
   v
Jenkins
   |
   +----> Build
   |
   +----> Unit Test
   |
   +----> SonarQube Analysis
   |
   +----> Quality Gate
   |
   v
Docker Build
   |
   v
Trivy / Snyk Scan
   |
   v
Nexus / ECR
   |
   v
Kubernetes
```

The main idea is to find security and quality issues before the application reaches production.

---

# 2. SonarQube

## What is SonarQube?

SonarQube is a static code analysis tool. It is integrated into CI/CD pipelines to identify:

- Bugs
- Vulnerabilities
- Security Hotspots
- Code Smells
- Code Duplication
- Test Coverage issues

## Where is SonarQube used?

A common Jenkins flow is:

```text
Jenkins
   |
   v
Build
   |
   v
Unit Test
   |
   v
SonarQube Analysis
   |
   v
Quality Gate
   |
   +---- PASS ----> Continue Pipeline
   |
   +---- FAIL ----> Stop Pipeline
```

---

# 3. SonarQube Quality Gate

## What is a Quality Gate?

A Quality Gate is a set of conditions that determines whether the analyzed code is acceptable for the next stage of the pipeline.

Example:

```text
Bugs             = 0
Vulnerabilities  = 0
Coverage         >= 80%
Code Smells      < 10
Duplications     < 5%
```

If the Quality Gate fails:

```text
Jenkins
   |
   v
SonarQube
   |
   v
Quality Gate FAILED
   |
   X
Pipeline STOP
```

## Interview Question

### Q: Your Jenkins pipeline is failing because SonarQube Quality Gate failed. What will you do?

### Answer:

> First, I will check the SonarQube analysis results and identify why the Quality Gate failed. I will check whether the failure is due to bugs, vulnerabilities, code smells, coverage or duplication. Then I will identify whether it is a new issue or an existing issue. I will work with the developer to fix the issue. After the fix, I will trigger the pipeline again and verify that the Quality Gate passes.

Do not say:

```text
I will bypass SonarQube.
```

A better approach is to investigate, remediate and rerun the pipeline.

---

# 4. Trivy

## What is Trivy?

Trivy is a security scanner used to identify vulnerabilities and misconfigurations in:

- Container images
- Filesystems
- Git repositories
- Kubernetes configurations
- Terraform/IaC
- Secrets
- Software packages
- SBOM-related artifacts

Example:

```bash
trivy image myapp:1.0
```

Example output concept:

```text
Package     CVE          Severity
openssl     CVE-XXXX     HIGH
curl        CVE-XXXX     MEDIUM
glibc       CVE-XXXX     CRITICAL
```

---

# 5. Trivy – CRITICAL Vulnerability Scenario

## Interview Question

### Q: Trivy found a CRITICAL vulnerability in your Docker image. What will you do?

### Answer:

> First, I will check the Trivy report and identify the affected package, CVE, installed version and fixed version. Then I will check whether the vulnerability comes from the application dependency or the Docker base image. If it comes from the base image, I will update the base image. If it is an application dependency, I will upgrade the dependency to a secure version. After rebuilding the image, I will run Trivy again. If the vulnerability is resolved, I will allow the pipeline to continue.

Flow:

```text
Trivy
  |
  v
CRITICAL CVE
  |
  +---- Base Image?
  |        |
  |        +--> Update base image
  |
  +---- Application Dependency?
           |
           +--> Upgrade dependency
                    |
                    v
               Build Image
                    |
                    v
                 Trivy
                    |
                    v
                  PASS
```

---

# 6. Should Every Trivy Vulnerability Fail the Pipeline?

## Interview Question

### Q: Suppose Trivy reports 100 vulnerabilities. Will you stop the deployment?

### Answer:

> Not necessarily. I would define a security policy based on severity and exploitability. For example, we may block production deployment for CRITICAL and HIGH vulnerabilities, while MEDIUM and LOW findings can be tracked and remediated according to the organization's security SLA. I would also investigate false positives and whether a fixed version is available.

Example policy:

```text
CRITICAL  -> BLOCK
HIGH      -> BLOCK
MEDIUM    -> Depending on policy
LOW       -> Report
UNKNOWN   -> Investigate
```

The exact policy should be agreed with the organization's security team.

---

# 7. Snyk

## What is Snyk?

Snyk is a developer-focused security platform that can identify vulnerabilities in:

- Application code
- Open-source dependencies
- Container images
- Infrastructure as Code

---

# 8. SCA – Software Composition Analysis

## What is SCA?

SCA means:

```text
Software Composition Analysis
```

It checks third-party/open-source dependencies for known vulnerabilities.

For example, a Java application may have:

```text
pom.xml
   |
   +--> Spring
   +--> Log4j
   +--> Jackson
   +--> Tomcat
```

SCA can identify whether one of these dependencies contains a known vulnerability.

Example:

```text
Application
     |
     v
pom.xml
     |
     v
Snyk / Dependency Scanner
     |
     v
Vulnerable dependency
     |
     v
Upgrade dependency
```

---

# 9. SonarQube vs Snyk vs Trivy

This is a very common interview question.

| Tool | Main Purpose |
|---|---|
| SonarQube | Source-code quality and security analysis |
| Snyk | Code, open-source dependencies, containers and IaC |
| Trivy | Container/image, filesystem, IaC, Kubernetes and secrets scanning |

Easy way to remember:

```text
SonarQube
    |
    +--> Is my CODE good and secure?

Snyk
    |
    +--> Are my dependencies/code/container/IaC secure?

Trivy
    |
    +--> Is my IMAGE / IaC / artifact secure?
```

There can be overlap between these tools. The organization should define which tool is the source of truth for each security check.

---

# 10. Real-Time DevOps Scenario

## Interview Question

### Q: Developer says the application is working, but Trivy is blocking your production deployment. What will you do?

### Answer:

> I won't immediately bypass the security scan. I will first understand the vulnerability, severity, affected component, exploitability and whether a fixed version is available. If a fix exists, I will update the dependency or base image and rebuild the image. If there is no fix available, I will discuss the risk with the security and application teams and follow the approved exception process if the organization allows it. Any exception should be documented and have a remediation plan.

This demonstrates DevSecOps maturity.

---

# 11. False Positive Scenario

## Interview Question

### Q: Trivy/Snyk reports a vulnerability, but the developer says it doesn't affect the application. What do you do?

### Answer:

> I will validate the finding instead of immediately ignoring it. I will check the vulnerable package, version, dependency path and whether the vulnerable functionality is actually used. If it is confirmed as a false positive, I will follow the organization's approved ignore or exception mechanism and document the reason.

Never simply say:

```text
I'll just ignore it.
```

---

# 12. Where Should Trivy Run?

A good CI/CD design is:

```text
Git Checkout
     |
     v
Maven Build
     |
     v
Unit Test
     |
     v
SonarQube
     |
     v
Quality Gate
     |
     v
Docker Build
     |
     v
Trivy Image Scan
     |
     v
Docker Push
     |
     v
Nexus / ECR
     |
     v
Kubernetes Deploy
```

## Why scan before pushing?

You don't want a vulnerable image to become a release candidate in your container registry.

---

# 13. Jenkins – Fail Pipeline Based on Trivy Severity

## Interview Question

### Q: How do you make Jenkins fail when Trivy finds CRITICAL or HIGH vulnerabilities?

Example:

```bash
trivy image --severity HIGH,CRITICAL --exit-code 1 myapp:${BUILD_NUMBER}
```

The important concept is:

```text
exit-code 0 -> Jenkins continues
exit-code 1 -> Jenkins fails
```

Flow:

```text
Trivy
  |
  +--> No HIGH/CRITICAL
  |        |
  |        v
  |      PASS
  |
  +--> HIGH/CRITICAL found
           |
           v
         FAIL
           |
           v
       Jenkins STOP
```

---

# 14. Vulnerability vs Misconfiguration

## Vulnerability

A vulnerability is a known security weakness in a package or software component.

Example:

```text
openssl
CVE-XXXX
CRITICAL
```

## Misconfiguration

A misconfiguration is an insecure configuration.

Example Kubernetes configuration:

```yaml
securityContext:
  privileged: true
```

Other examples may include:

```yaml
hostNetwork: true
```

or overly permissive cloud/IaC permissions.

---

# 15. Kubernetes Security Scanning

## Interview Question

### Q: How do you scan Kubernetes configurations?

### Answer:

> I can scan Kubernetes manifests before deployment using security scanning tools such as Trivy or Snyk IaC. The objective is to identify insecure configurations before they reach the cluster.

Example:

```bash
trivy config deployment.yaml
```

Or:

```bash
trivy config .
```

---

# 16. Secret Detection

## Interview Question

### Q: Can Trivy detect secrets?

Yes.

Trivy can scan images, filesystems and repositories for exposed secrets such as:

- API keys
- Passwords
- Tokens
- Credentials

Example:

```text
Docker Image
     |
     v
Trivy
     |
     +--> Vulnerability
     +--> Secret
     +--> Misconfiguration
     +--> License
```

Important DevOps practice:

```text
Never hard-code:
- Passwords
- API keys
- Cloud credentials
- Database credentials
```

Use a proper secret-management solution.

---

# 17. Complete DevSecOps Pipeline

A strong production-oriented flow is:

```text
                     Developer
                         |
                         v
                      GitHub
                         |
                         v
                      Jenkins
                         |
              +----------+----------+
              |                     |
              v                     v
           Build                Unit Test
              |                     |
              +----------+----------+
                         |
                         v
                    SonarQube
                         |
                         v
                    Quality Gate
                         |
                    +----+----+
                    |         |
                  PASS       FAIL
                    |         |
                    v         X
                Docker Build
                    |
                    v
               Snyk / Trivy
                    |
              +-----+-----+
              |           |
            PASS         FAIL
              |           |
              v           X
          Nexus / ECR
              |
              v
         Kubernetes
              |
              v
         Production
```

---

# 18. How to Handle Security Failures as a DevOps Engineer

Use this general approach:

```text
1. Identify the failure
        |
        v
2. Check severity
        |
        v
3. Identify affected component
        |
        v
4. Check whether a fix exists
        |
        v
5. Remediate
        |
        v
6. Rebuild
        |
        v
7. Scan again
        |
        v
8. Validate result
        |
        v
9. Continue deployment
```

If there is no immediate fix:

```text
No Fix Available
       |
       v
Assess Risk
       |
       v
Security / Application Team
       |
       v
Approved Exception?
       |
   +---+---+
   |       |
  YES      NO
   |       |
   v       v
Document  Fix Before
Exception Deployment
```

---

# 19. Best Interview Answer to Memorize

## Q: How do you handle security scanning in your DevOps pipeline?

### Answer:

> We integrate security scanning into the CI/CD pipeline. After the application is built and tested, we run SonarQube for static code analysis and enforce the Quality Gate. After creating the Docker image, we use Trivy or Snyk to scan the image for vulnerabilities. We define security thresholds, such as blocking critical and high-severity vulnerabilities. If a scan fails, I investigate the affected package or configuration, identify whether a fixed version is available, remediate it and rebuild the image. If there is no immediate fix, I follow the organization's approved security exception process rather than bypassing the scan.

---

# 20. Top 10 Hands-On Interview Scenarios

For a DevOps Engineer, practice these scenarios:

### 1. SonarQube Quality Gate failure

```text
Jenkins -> SonarQube -> Quality Gate FAILED
```

Know how to identify the failing condition and remediate it.

### 2. SonarQube authentication/token problem

```text
Jenkins -> SonarQube
             |
             X
       Authentication Failed
```

Know how to check credentials/token configuration.

### 3. SonarQube scanner cannot connect

Check:

```text
DNS
Network
URL
Port
Firewall
SonarQube service
Authentication
```

### 4. Trivy finds CRITICAL CVE

Know how to:

```text
Identify CVE
   ->
Find affected package
   ->
Check fixed version
   ->
Upgrade
   ->
Rebuild
   ->
Rescan
```

### 5. Vulnerability in Docker base image

Example:

```dockerfile
FROM old-base-image
```

Update to an approved patched base image and rebuild.

### 6. Trivy false positive

Validate:

```text
Package
Version
Dependency path
Exploitability
Fixed version
```

Then use the organization's approved exception/ignore process if appropriate.

### 7. Snyk dependency vulnerability

Check:

```text
pom.xml
   |
   v
Dependency
   |
   v
Vulnerability
   |
   v
Upgrade dependency
   |
   v
Build + Test
```

### 8. Secret detected in Docker image

Do not simply ignore it.

```text
Secret found
    |
    v
Remove secret
    |
    v
Rotate/revoke exposed credential
    |
    v
Use secret management
    |
    v
Rebuild image
    |
    v
Rescan
```

### 9. Kubernetes YAML misconfiguration

Example:

```yaml
privileged: true
```

Scan and remediate the manifest before deployment.

### 10. Jenkins automatically blocks deployment

Understand how scan exit codes and pipeline conditions can enforce:

```text
CRITICAL -> BLOCK
HIGH     -> BLOCK
MEDIUM   -> Policy based
LOW      -> Report
```

---

# 21. Interview Cheat Sheet

| Question | Key Point |
|---|---|
| What is SonarQube? | Static code analysis |
| What is Quality Gate? | Determines whether code passes quality/security conditions |
| What is Trivy? | Vulnerability/misconfiguration/secrets scanner |
| What is Snyk? | Code, dependency, container and IaC security platform |
| What is SCA? | Software Composition Analysis |
| What is CVE? | Common Vulnerabilities and Exposures identifier |
| Should all vulnerabilities block deployment? | Based on organizational security policy |
| What if Trivy finds CRITICAL? | Investigate → remediate → rebuild → rescan |
| What if no fix exists? | Risk assessment + approved exception process |
| What if finding is false positive? | Validate and document approved exception/ignore |
| Where does Trivy run? | Typically after image build and before registry/deployment |
| Can Trivy scan Kubernetes YAML? | Yes |
| Can Trivy detect secrets? | Yes |
| How does Jenkins stop on Trivy failure? | Scanner exit code + pipeline condition |
| What is DevSecOps? | Integrating security throughout the DevOps lifecycle |

---

# 22. Simple Memory Trick

Remember the tools like this:

```text
SONARQUBE
    |
    +--> CODE QUALITY
    +--> BUGS
    +--> CODE SMELLS
    +--> CODE SECURITY
    +--> QUALITY GATE


SNYK
    |
    +--> DEPENDENCIES
    +--> CODE
    +--> CONTAINERS
    +--> IaC


TRIVY
    |
    +--> CONTAINER IMAGES
    +--> VULNERABILITIES
    +--> MISCONFIGURATION
    +--> SECRETS
    +--> IaC
    +--> KUBERNETES
```

## Most Important DevOps Principle

```text
Do NOT bypass security findings blindly.

Identify
   ->
Validate
   ->
Remediate
   ->
Rebuild
   ->
Rescan
   ->
Deploy
```

This is the approach you should demonstrate in a real DevOps interview.
