# SonarQube, Snyk & Trivy – DevOps Interview Questions and Answers

## 1. SonarQube Quality Gate Failure

### Interview Question
**What will you do if the SonarQube Quality Gate fails?**

### Answer
> First, I will check the SonarQube report to understand why the Quality Gate failed. I will check for bugs, vulnerabilities, code smells, code coverage and duplication. Then I will identify whether the issue is newly introduced or an existing issue. I will coordinate with the developer to fix the issue. Once it is fixed, I will run the pipeline again and verify that the Quality Gate passes. I would not bypass the Quality Gate without an approved exception.

### Practical Flow

```text
Jenkins
   |
   v
SonarQube Scan
   |
   v
Quality Gate FAILED
   |
   v
Check SonarQube Report
   |
   +--> Bug?
   +--> Vulnerability?
   +--> Coverage?
   +--> Code Smell?
   |
   v
Developer Fix
   |
   v
Run Pipeline Again
   |
   v
Quality Gate PASS
```

---

## 2. SonarQube Authentication / Token Problem

### Interview Question
**Jenkins is unable to authenticate with SonarQube. How will you troubleshoot?**

### Answer
> First, I will verify whether the SonarQube server is reachable from the Jenkins server. Then I will check the SonarQube URL and Jenkins credentials configuration. I will verify that the token is valid, has not expired or been revoked, and has the required permissions. I will also check whether the correct credentials ID is configured in the Jenkins pipeline. Finally, I will rerun the pipeline and check the Jenkins console logs.

### Things to Check

```text
1. SonarQube URL
2. Jenkins credentials
3. Token validity
4. Token permissions
5. Jenkins credentialsId
6. Network connectivity
7. SonarQube server logs
8. Jenkins console output
```

---

## 3. SonarQube Scanner Can't Connect to Server

### Interview Question
**SonarScanner cannot connect to SonarQube. What will you check?**

### Answer
> I will first check whether the SonarQube service is running. Then I will verify connectivity from the Jenkins server to the SonarQube server. I will check DNS resolution, IP address, port, firewall and the SonarQube URL. If connectivity is working, I will check authentication and scanner configuration.

### Troubleshooting Commands

```bash
ping <sonarqube-server>
nc -zv <sonarqube-server> 9000
curl http://<sonarqube-server>:9000
```

### Check

```text
SonarQube service
Jenkins network
Security Group
Firewall
DNS
Proxy
SonarQube URL
Authentication
```

### Interview Tip

> I troubleshoot the connectivity layer first and the authentication layer second.

---

## 4. Trivy Finds CRITICAL CVE

### Interview Question
**Trivy found a CRITICAL vulnerability in your Docker image. What will you do?**

### Answer
> First, I will check the Trivy report and identify the CVE, affected package, installed version and fixed version. Then I will determine whether the vulnerability comes from the application dependency or the Docker base image. If a fixed version is available, I will upgrade the package or base image, rebuild the image and run Trivy again. I will allow deployment only after the vulnerability is remediated or an approved security exception is provided.

### Example

```bash
trivy image myapp:1.0
```

### Remediation Flow

```text
Identify CVE
     |
     v
Find affected package
     |
     v
Check fixed version
     |
     v
Upgrade
     |
     v
Docker Build
     |
     v
Trivy Scan Again
     |
     v
PASS
```

---

## 5. Trivy Finds Vulnerability in Docker Base Image

### Interview Question
**Trivy reports a vulnerability in your Docker base image. What will you do?**

### Answer
> First, I will confirm that the vulnerability is coming from the base image. I will check the installed package and whether a fixed package or newer base image is available. I will update the Dockerfile to use an approved and patched base image, rebuild the image and run Trivy again. I will also make sure we are using a supported base image and not an outdated or end-of-life image.

### Example

Old:

```dockerfile
FROM ubuntu:20.04
```

Update according to the organization's approved supported image/version:

```dockerfile
FROM ubuntu:<approved-patched-version>
```

Build:

```bash
docker build -t myapp:2.0 .
```

Scan:

```bash
trivy image myapp:2.0
```

### Important Interview Point

> A vulnerability in the base image can affect every application image built from that base image, so keeping base images patched is important.

---

## 6. Trivy False Positive

### Interview Question
**Trivy reports a vulnerability, but the developer says it is a false positive. What will you do?**

### Answer
> I will not immediately ignore the finding. First, I will validate the vulnerability by checking the CVE, affected package, installed version, dependency path and whether the vulnerable functionality is actually present or used. I will also check the vulnerability details and available fixed versions. If it is confirmed to be a false positive, I will follow the organization's approved exception or ignore mechanism and document the reason.

### Process

```text
Trivy Finding
     |
     v
Validate CVE
     |
     v
Check Package + Version
     |
     v
Check Dependency Path
     |
     v
Is it actually exploitable?
     |
   +---+---+
   |       |
  YES      NO
   |       |
   v       v
Remediate  Approved Exception
           + Documentation
```

### Don't Say
> I'll just add it to the ignore list.

### Better Answer
> I will validate it first and use an approved exception process if it is genuinely a false positive.

---

## 7. Snyk Dependency Vulnerability

### Interview Question
**Snyk reports a vulnerability in a dependency. What will you do?**

### Answer
> I will identify the vulnerable dependency and check the installed version and recommended fixed version. Then I will determine whether it is a direct or transitive dependency. If a secure version is available, I will update the dependency, run the application tests and rebuild the application. Then I will run Snyk again to verify that the vulnerability has been resolved.

### Example Flow

```text
pom.xml
   |
   v
Snyk
   |
   v
Vulnerable dependency
   |
   v
Check fixed version
   |
   v
Update pom.xml
   |
   v
Maven Build + Test
   |
   v
Snyk Scan
   |
   v
PASS
```

### Direct Dependency

```xml
<dependency>
    <groupId>example</groupId>
    <artifactId>example-library</artifactId>
    <version>OLD_VERSION</version>
</dependency>
```

### Transitive Dependency

**Question:** What if it is a transitive dependency?

**Answer:**
> I will identify which direct dependency is bringing it into the application. If possible, I will upgrade the parent dependency. If necessary, I can explicitly manage the transitive dependency version, provided it is compatible with the application.

---

## 8. Secret Detected in Docker Image

### Interview Question
**Trivy detects a secret inside your Docker image. What will you do?**

### Answer
> I will treat it as a security incident rather than simply ignoring the scan. First, I will identify what type of secret was exposed. If the credential may have been used, I will revoke or rotate it immediately. Then I will remove the secret from the Dockerfile, source code or build process and use a proper secret-management mechanism. I will rebuild the image and scan it again. I will also investigate whether the exposed secret was pushed to a registry or repository.

### Flow

```text
Secret Detected
      |
      v
Identify Secret
      |
      v
Was it exposed/used?
      |
      v
Rotate / Revoke
      |
      v
Remove from Source/Image
      |
      v
Use Secret Management
      |
      v
Rebuild Image
      |
      v
Trivy Scan
      |
      v
PASS
```

### Never Do This

```dockerfile
ENV DB_PASSWORD=SuperSecret123
```

### Better Approach

Use an appropriate secret-management solution such as:

```text
AWS Secrets Manager
Kubernetes Secrets
HashiCorp Vault
Jenkins Credentials
```

depending on the organization's architecture.

---

## 9. Kubernetes YAML Misconfiguration

### Interview Question
**Your security scanner reports a Kubernetes YAML misconfiguration. What will you do?**

### Answer
> I will review the scanner finding and identify the insecure Kubernetes configuration. I will check why the configuration was used and whether it is actually required. If it is unnecessary, I will remove or correct it. Then I will scan the manifest again and only deploy after the security check passes.

### Example

Potentially insecure:

```yaml
securityContext:
  privileged: true
```

Potentially safer when privileged access is not required:

```yaml
securityContext:
  privileged: false
```

Another example:

```yaml
hostNetwork: true
```

The correct fix depends on the application's actual requirements.

### Scan

```bash
trivy config deployment.yaml
```

### Flow

```text
Kubernetes YAML
       |
       v
Security Scan
       |
       v
Misconfiguration
       |
       v
Understand Requirement
       |
       v
Fix YAML
       |
       v
Scan Again
       |
       v
Deploy
```

---

## 10. Jenkins Automatically Blocks Deployment Based on Security Severity

### Interview Question
**How would you configure Jenkins to block deployment when CRITICAL or HIGH vulnerabilities are found?**

### Answer
> I would integrate the security scanner into the Jenkins pipeline and configure the scanner to return a failure exit code when the defined severity threshold is detected. Jenkins will use that exit code to fail the stage and prevent the deployment stage from executing.

### Example Trivy Command

```bash
trivy image --severity HIGH,CRITICAL --exit-code 1 myapp:${BUILD_NUMBER}
```

### Important Concept

```text
exit-code 0 -> Jenkins continues
exit-code 1 -> Jenkins fails
```

### Jenkins Pipeline Example

```groovy
stage('Trivy Scan') {
    steps {
        sh '''
            trivy image             --severity HIGH,CRITICAL             --exit-code 1             myapp:${BUILD_NUMBER}
        '''
    }
}

stage('Deploy') {
    steps {
        sh 'kubectl apply -f deployment.yaml'
    }
}
```

### Flow

```text
Trivy Scan
    |
    v
Severity?
    |
    +---- LOW/MEDIUM ----> Continue
    |
    +---- HIGH/CRITICAL --> Exit Code 1
                              |
                              v
                         Jenkins FAIL
                              |
                              v
                       Deployment BLOCKED
```

---

# 11. Should Every Trivy Vulnerability Fail the Pipeline?

### Interview Question
**Suppose Trivy reports 100 vulnerabilities. Will you stop the deployment?**

### Answer
> Not necessarily. I would define a security policy based on severity and exploitability. For example, we may block production deployment for CRITICAL and HIGH vulnerabilities, while MEDIUM and LOW findings can be tracked and remediated according to the organization's security SLA. I would also investigate false positives and whether a fixed version is available.

### Example Policy

```text
CRITICAL  -> BLOCK
HIGH      -> BLOCK
MEDIUM    -> Depending on policy
LOW       -> Report
UNKNOWN   -> Investigate
```

---

# 12. SonarQube vs Snyk vs Trivy

### Interview Question
**What is the difference between SonarQube, Snyk and Trivy?**

| Tool | Main Purpose |
|---|---|
| SonarQube | Source-code quality and security analysis |
| Snyk | Code, open-source dependencies, containers and IaC |
| Trivy | Container/image, filesystem, IaC, Kubernetes and secrets scanning |

### Easy Way to Remember

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

---

# 13. Complete DevSecOps Pipeline

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

# 14. General Security Failure Handling

Use this approach for almost every security issue:

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

# 15. One Strong Interview Answer to Memorize

### Question
**Explain how you handle security scanning in your DevOps pipeline.**

### Answer

> We integrate security scanning into the CI/CD pipeline. After the application is built and tested, we run SonarQube for static code analysis and enforce the Quality Gate. After creating the Docker image, we use Trivy or Snyk to scan the image for vulnerabilities. We define security thresholds, such as blocking critical and high-severity vulnerabilities. If a scan fails, I investigate the affected package or configuration, identify whether a fixed version is available, remediate it and rebuild the image. If there is no immediate fix, I follow the organization's approved security exception process rather than bypassing the scan.

---

# 16. Interview Mindset

For every security issue, remember:

```text
IDENTIFY
   ↓
VALIDATE
   ↓
UNDERSTAND IMPACT
   ↓
CHECK FIX
   ↓
REMEDIATE
   ↓
REBUILD
   ↓
RESCAN
   ↓
PASS
   ↓
DEPLOY
```

If there is no fix:

```text
No Fix
  ↓
Risk Assessment
  ↓
Security Team
  ↓
Approved Exception
  ↓
Document
  ↓
Remediation Plan
```

## Key DevOps Principle

> **Do not bypass security findings blindly.**

The interviewer is checking whether you can safely handle a failed production pipeline without bypassing security controls.

---

# 17. Quick Interview Cheat Sheet

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
| What if finding is false positive? | Validate and document approved exception |
| Where does Trivy run? | Typically after image build and before registry/deployment |
| Can Trivy scan Kubernetes YAML? | Yes |
| Can Trivy detect secrets? | Yes |
| How does Jenkins stop on Trivy failure? | Scanner exit code + pipeline condition |
| What is DevSecOps? | Integrating security throughout the DevOps lifecycle |

---

# 18. Final Memory Trick

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

## Final Rule

```text
Security Finding
      |
      v
Identify
      |
      v
Validate
      |
      v
Remediate
      |
      v
Rebuild
      |
      v
Rescan
      |
      v
Deploy
```
