# DevOps Real-Time Troubleshooting Scenarios — Study Guide

A study guide for DevOps interviews. Read the scenario, think through your own answer, then expand the `Answer` section for root causes, diagnostic commands, the fix, and how to prevent it next time.

**16 scenarios · 5 categories**

## Contents

- [Compute & OS](#compute--os)
  1. [Production EC2 CPU spikes to 100%](#1-production-ec2-cpu-spikes-to-100-aws)
  2. [App runs on EC2 but isn't reachable from the browser](#2-app-runs-on-ec2-but-isnt-reachable-from-the-browser-aws)
  3. [ALB returns 502/504 Bad Gateway intermittently](#3-alb-returns-502504-bad-gateway-intermittently-networking)
  4. [Disk keeps filling up on a production Linux server](#4-disk-keeps-filling-up-on-a-production-linux-server-infra)
- [Containers & Kubernetes](#containers--kubernetes)
  5. [Docker container is running but not accessible](#5-docker-container-is-running-but-not-accessible-k8s)
  6. [Pod is stuck in CrashLoopBackOff](#6-pod-is-stuck-in-crashloopbackoff-k8s)
  7. [Pod stuck in Pending state](#7-pod-stuck-in-pending-state-k8s)
  8. [A node shows status NotReady](#8-a-node-shows-status-notready-k8s)
  9. [HPA isn't scaling despite high load](#9-hpa-isnt-scaling-the-deployment-despite-high-load-k8s)
- [CI/CD](#cicd)
  10. [Jenkins pipeline fails at the build stage](#10-jenkins-pipeline-fails-at-the-build-stage-cicd)
  11. [Pipeline fails only when merging to main](#11-pipeline-fails-only-during-merge-to-main--works-fine-on-feature-branches-cicd)
  12. [Blue-Green Deployment](#12-blue-green-deployment--how-does-it-work-and-when-do-you-use-it-infra)
- [AWS Services](#aws-services)
  13. [RDS storage is full](#13-rds-storage-is-full-aws)
  14. [S3 Access Denied](#14-s3-access-denied-aws)
  15. [Terraform plan shows changes you didn't make](#15-terraform-plan-shows-changes-you-didnt-make-infra)
  16. [Auto Scaling Group isn't launching new instances](#16-auto-scaling-group-isnt-launching-new-instances-under-load-aws)

---

## Compute & OS

### 1. Production EC2 CPU spikes to 100% `AWS`

<details>
<summary><b>Answer</b></summary>

**Likely root causes**
- Runaway process or infinite loop in the application
- Traffic spike beyond current capacity
- A cron job or batch process overlapping with peak hours
- Memory pressure causing excessive garbage collection (CPU thrash)

**Diagnose**
```
# Check the trend first — is this sustained or a spike?
CloudWatch → EC2 → CPUUtilization (1-min granularity)

# SSH in and find the offending process
top          # sort by CPU, note PID
ps aux --sort=-%cpu | head
df -h        # rule out disk pressure
free -m      # rule out memory pressure causing swap thrash
```

**Fix**
- Kill or restart only the affected service — don't reboot the whole box blindly on prod
- If it's load-driven, scale out (Auto Scaling Group) or scale up the instance type
- If it's a bad deploy, roll back first, investigate after

**Prevent**
- CloudWatch alarm at 70–80% CPU with SNS notification
- Auto Scaling policy tied to CPU or request-count target tracking
- Profile the app to find hot code paths before it becomes an incident

> **Interview angle:** Interviewers want to see you separate "stop the bleeding" (restart/scale) from "fix the disease" (root cause + prevention). Say both.

</details>

### 2. App runs on EC2 but isn't reachable from the browser `AWS`

<details>
<summary><b>Answer</b></summary>

**Checklist, in order**
- **Security Group** — is the inbound rule open on 80/443/8080 from the right source (0.0.0.0/0 or the ALB's SG)?
- **NACL** — subnet-level rules are stateless; check both inbound and outbound
- **Load Balancer target group health checks** — is the instance marked *unhealthy*? Check the health-check path and port
- **Web server process** — is nginx/httpd actually running and bound to the right port?
- **App/access logs** — 502/504 from the LB usually means the backend refused or timed out

**Diagnose**
```
curl -I localhost:8080          # from inside the instance — isolates app vs network
sudo ss -tlnp | grep 8080       # is anything actually listening?
sudo systemctl status nginx
tail -f /var/log/nginx/error.log
```

> **Interview angle:** Work outside-in or inside-out consistently — most candidates jump around. Say: "I'd start from the client and work back: DNS → LB → SG/NACL → instance → process."

</details>

### 3. ALB returns 502/504 Bad Gateway intermittently `Networking`

<details>
<summary><b>Answer</b></summary>

**Root causes**
- 502: backend closed the connection or returned a malformed response (app crash, out-of-memory kill)
- 504: backend took longer than the LB's idle timeout to respond
- Keep-alive timeout mismatch between the ALB and the web server (ALB idle timeout must be ≤ web server keep-alive)

**Fix**
- Check target group health-check history and deregistration events
- Increase ALB idle timeout or reduce backend response time for 504s
- Align nginx `keepalive_timeout` with the ALB's idle timeout for 502s
- Check app logs at the exact timestamp of the 502 for crashes/OOM

</details>

### 4. Disk keeps filling up on a production Linux server `Infra`

<details>
<summary><b>Answer</b></summary>

**Diagnose**
```
df -h                       # which mount is full
du -sh /var/log/* | sort -rh | head    # biggest offenders, usually logs
lsof +L1                    # deleted-but-open files still holding space
```

**Common causes & fix**
- Unrotated application logs → configure `logrotate`
- Docker images/containers piling up → `docker system prune`
- A process holding a deleted file open (space not freed until the process restarts)
- Core dumps from a crashing service filling `/var/crash`

**Prevent**
- CloudWatch/Prometheus disk-usage alert at 75–80%
- Centralize logs (CloudWatch Logs, ELK/EFK) and ship off-box instead of retaining locally

</details>

---

## Containers & Kubernetes

### 5. Docker container is running but not accessible `K8s`

<details>
<summary><b>Answer</b></summary>

**Checklist**
- Confirm the port mapping is actually published: `docker ps` shows `0.0.0.0:8080->8080/tcp`?
- Read the container's own logs: `docker logs <container>`
- The app inside the container must bind to `0.0.0.0`, not `127.0.0.1` — binding to localhost makes it unreachable from outside the container network namespace
- If this is Kubernetes: check the **Service type** (ClusterIP vs NodePort vs LoadBalancer) and confirm endpoints exist

```
kubectl get endpoints <service>   # empty = selector doesn't match any pod labels
kubectl get svc <service> -o wide
```

> **Common trap:** an empty Endpoints object almost always means the Service's `selector` doesn't match the Pod's `labels` — not a networking problem at all.

</details>

### 6. Pod is stuck in CrashLoopBackOff `K8s`

<details>
<summary><b>Answer</b></summary>

**Diagnose**
```
kubectl describe pod <pod>      # events section — OOMKilled? failed probe? image pull error?
kubectl logs <pod>               # current attempt
kubectl logs <pod> --previous    # the crash BEFORE the restart — usually more useful
```

**Common causes**
- App crashes immediately on bad config/env var/missing secret
- Liveness probe misconfigured (too aggressive, killing a healthy-but-slow-starting app)
- OOMKilled — container hits its memory `limit`
- Wrong image tag or entrypoint command

**Fix**
- Fix the underlying cause, then `kubectl rollout restart deployment/<name>`
- For OOM: raise the memory limit or fix a leak; for slow starts, add/raise `initialDelaySeconds` on probes

</details>

### 7. Pod stuck in Pending state `K8s`

<details>
<summary><b>Answer</b></summary>

**Root causes**
- Insufficient CPU/memory on any node to satisfy the pod's `requests`
- NodeSelector/affinity/taints excluding all available nodes
- PVC can't be bound — no matching StorageClass or PV, or the zone doesn't match

```
kubectl describe pod <pod>     # Events will name the exact reason
kubectl get nodes -o wide
kubectl describe pvc <pvc>
```

**Fix**
- Scale the node group, or lower the pod's resource requests if over-provisioned
- Remove/adjust taints or tolerations
- Fix the StorageClass or provision the PV in the correct AZ

</details>

### 8. A node shows status NotReady `K8s`

<details>
<summary><b>Answer</b></summary>

**Diagnose**
```
kubectl describe node <node>         # Conditions section: MemoryPressure, DiskPressure, kubelet down?
kubectl get pods -A -o wide | grep <node>
# on the node itself:
sudo systemctl status kubelet
journalctl -u kubelet -f
```

**Common causes**
- kubelet crashed or lost network connectivity to the control plane
- Node under disk or memory pressure
- Container runtime (containerd/Docker) unresponsive

**Fix**
- Restart kubelet/container runtime; if unrecoverable, cordon + drain and replace the node
- `kubectl cordon <node>` then `kubectl drain <node> --ignore-daemonsets` before terminating it

</details>

### 9. HPA isn't scaling the deployment despite high load `K8s`

<details>
<summary><b>Answer</b></summary>

**Common causes**
- Pods have no `resources.requests` set — HPA can't compute a percentage without a baseline
- Metrics Server isn't installed/running, so no metrics are available
- `maxReplicas` already reached, or cluster has no node capacity to schedule new pods (Pending)

```
kubectl get hpa
kubectl top pods
kubectl describe hpa <name>
```

</details>

---

## CI/CD

### 10. Jenkins pipeline fails at the build stage `CI/CD`

<details>
<summary><b>Answer</b></summary>

**Checklist**
- Read the console output first — most answers are right there
- Validate Git credentials/SSH keys haven't expired or rotated
- Confirm the build agent/node is online and has the right labels
- Clean the workspace and rebuild — stale `target/` or `node_modules/` causes phantom failures
- Check Maven/npm dependency resolution — a flaky or unreachable repo (Nexus/Artifactory) will fail the build

```
mvn clean install -X       # verbose output for real error
git ls-remote               # test credentials/connectivity independent of Jenkins
```

> **Interview angle:** Say you'd distinguish "flaky" (retry-and-move-on) failures from "deterministic" (code/config) failures before spending time debugging.

</details>

### 11. Pipeline fails only during merge to main — works fine on feature branches `CI/CD`

<details>
<summary><b>Answer</b></summary>

**Common causes**
- Merge conflicts that auto-merged cleanly in Git but broke the build semantically
- Branch protection/environment secrets that only exist on `main`'s pipeline
- A different (usually stricter) pipeline stage runs only on main — e.g. integration tests, security scans

**Fix**
- Reproduce locally: check out main, merge the feature branch, run the exact same build command
- Compare environment variables/secrets between the branch and main pipelines

</details>

### 12. Blue-Green Deployment — how does it work and when do you use it? `Infra`

<details>
<summary><b>Answer</b></summary>

**How it works**
- Two identical environments: **Blue** (live/current) and **Green** (new version)
- Deploy the new version to Green while Blue keeps serving 100% of traffic
- Test Green in isolation (smoke tests, synthetic checks)
- Switch the router/load balancer to send traffic to Green
- Rollback is just switching the router back to Blue — near-instant, no redeploy needed

**Trade-offs**
- Needs double the infrastructure at cutover time (cost)
- Doesn't catch issues that only show up under real production traffic volume — pair with canary for that
- Stateful services (databases, in-flight sessions) need extra care during the switch

> **Compare to canary:** canary shifts a small % of real traffic gradually; blue-green shifts 100% at once but rolls back instantly. Know both — interviewers often ask you to pick one for a given scenario.

</details>

---

## AWS Services

### 13. RDS storage is full `AWS`

<details>
<summary><b>Answer</b></summary>

**Immediate fix**
- Take a snapshot first if you're going to modify anything risky
- Increase allocated storage (can be done live, with brief I/O impact on some engines)
- Enable Storage Auto Scaling so this doesn't recur

**Also check**
- Bloated transaction/binlogs — check retention settings
- Runaway temp tables from a bad query
- Old snapshots or unused read replicas holding storage

**Prevent**
- CloudWatch alarm on `FreeStorageSpace` at a sensible threshold

</details>

### 14. S3 Access Denied `AWS`

<details>
<summary><b>Answer</b></summary>

**Checklist**
- Bucket policy — does it explicitly allow the action for this principal?
- IAM role/user policy — does it grant `s3:GetObject` (and `s3:ListBucket` if listing)?
- Remember: an explicit `Deny` anywhere (bucket policy, IAM policy, SCP, or an S3 Block Public Access setting) always wins over an `Allow`
- Bucket owner vs object owner mismatch (cross-account uploads) — check the object ACL
- KMS-encrypted objects also need `kms:Decrypt` permission on the key policy, not just S3 permissions

**Prevent**
- Avoid public buckets unless genuinely required; use bucket policies + IAM roles, not long-lived keys

</details>

### 15. Terraform plan shows changes you didn't make `Infra`

<details>
<summary><b>Answer</b></summary>

**Reason:** State drift — the real infrastructure was changed outside of Terraform (manual console edit, another pipeline, or the state file is out of sync with reality).

**Fix**
```
terraform refresh          # sync state with real-world resources (or `plan -refresh-only`)
terraform state list       # inspect what Terraform thinks it owns
terraform import <addr> <id>   # bring an unmanaged resource under management
```

**Prevent**
- Remote backend with locking — S3 + DynamoDB lock table prevents concurrent, conflicting applies
- Ban manual console changes to Terraform-managed resources; enforce via SCPs or tagging conventions
- `terraform plan` in CI on every PR so drift is caught before it compounds

</details>

### 16. Auto Scaling Group isn't launching new instances under load `AWS`

<details>
<summary><b>Answer</b></summary>

**Common causes**
- Already at `MaxSize`
- Scaling policy/alarm never actually triggers — check the CloudWatch alarm's metric and threshold
- Launch template/config references an AMI, subnet, or SG that no longer exists → instances fail to launch silently
- No available IPs left in the target subnet

```
aws autoscaling describe-scaling-activities --auto-scaling-group-name <name>
# describes exactly why the last scale attempt succeeded or failed
```

</details>

---

*Built from your uploaded scenario sheet, expanded with root causes, diagnostic commands, fixes, and prevention for each — plus a few extra scenarios that come up often in DevOps interviews (ALB 5xx, disk-full, HPA, ASG, NotReady nodes, CI/CD-only-on-main failures).*
