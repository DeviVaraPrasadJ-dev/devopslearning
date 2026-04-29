# Devopslearning
---
# DEVOPS ENGINEERING MASTER GUIDE
---
A production‑oriented reference covering DevOps, Cloud, SRE, Platform
Engineering, and everything in between. Each chapter provides theory,
step‑by‑step flows, real‑world commands, debugging playbooks, and best
practices. Written with a senior‑engineer mindset: automation first,
security everywhere, observability by default, and simplicity above all.

## TABLE OF CONTENTS
1. DevOps Fundamentals & Culture
2. Version Control (Git)
3. CI/CD Pipelines & Tools
4. Build Tools & Artifact Management
5. Containerization (Docker)
6. Container Orchestration (Kubernetes)
7. Infrastructure as Code (IaC)
8. Cloud Platforms (AWS, Azure, GCP)
9. Configuration Management & Automation (Ansible)
10. Deployment Strategies & Release Engineering
11. GitOps
12. Security & DevSecOps
13. Monitoring, Observability & Logging
14. Reliability Engineering (SRE)
15. Performance Optimization & FinOps
16. Platform Engineering & Internal Developer Platforms
17. Networking & Storage
18. Data Engineering & Messaging (Kafka, Spark)
19. Advanced Kubernetes
20. Incident Management & Troubleshooting Playbooks
21. System Architecture & Design Patterns
22. Backup, Disaster Recovery & High Availability
23. Supply Chain Security & Zero Trust
24. AI in DevOps (AIOps, MLOps)
25. Best Practices, Common Mistakes & Golden Rules
26. Appendices (Quick Reference Cards)

---

# 1. DEVOPS FUNDAMENTALS & CULTURE

## Concept & Why
DevOps is a cultural and technical movement that unifies development
(Dev) and operations (Ops) to deliver software faster, more reliably,
and at scale. It breaks down silos, promotes shared ownership, and
embeds quality and security throughout the delivery lifecycle.

## Core Principles
- **Collaboration:** Dev, Ops, Security, and Business work as one team.
- **Automation:** Every repetitive task (build, test, deploy, monitor) is
  automated.
- **Measurement:** Everything is measured (deployment frequency, lead time,
  MTTR, change failure rate – DORA metrics).
- **Sharing:** Knowledge, tools, and responsibilities are shared.
- **Continuous Improvement:** Iterate on processes and systems based on
  feedback.

## Implementation Mindset
- Start with a “walking skeleton” of a CI/CD pipeline.
- Adopt infrastructure as code and GitOps for all environments.
- Shift left on security and testing.
- Build observability in from day one.

## Common Anti‑Patterns
- Separating DevOps into a dedicated “DevOps team” that acts as a
  bottleneck.
- Tools without culture change.
- Over‑engineering the pipeline before delivering value.

---

# 2. VERSION CONTROL (GIT)

## Concept & Why
Git is a distributed version control system that captures snapshots of
the entire repository. It enables safe, parallel collaboration and is
the entry point to all automation. Every code change is traceable,
reversible, and auditable.

## Core Workflow
1. Clone a repository   → `git clone <url>`
2. Create a feature branch → `git checkout -b feat/my-feature`
3. Stage changes → `git add .`
4. Commit with a meaningful message → `git commit -m "type: description"`
5. Push to remote → `git push origin feat/my-feature`
6. Open a Pull Request (PR) and get it reviewed.
7. Merge (squash/rebase/merge commit) and delete the branch.

## Branching Strategies
- **Trunk‑Based Development:** Short‑lived feature branches, frequent
  merges to main, fast feedback. Best for mature CI/CD.
- **GitFlow:** Long‑running develop/release branches; suitable for
  versioned products with scheduled releases.
- **GitHub Flow:** Simple feature branches + PRs, direct deploy from main.

## Conflict Resolution
- Pull the latest main, merge into your branch, and resolve conflicts
  locally.
- Use `git mergetool` or IDE tools; always test after resolution.

## Best Practices
- Write atomic commits.
- Follow conventional commit messages (e.g., “feat: add login page”).
- Never commit directly to main.
- Use `.gitignore` to exclude build artefacts, secrets, and local
  configs.
- Sign commits with GPG for integrity.

## Debugging Git Issues
- `git status` – see the current state.
- `git log --oneline --graph` – visualise history.
- `git diff` – inspect unstaged changes.
- `git reflog` – recover lost commits.
- If push is rejected, pull and rebase/merge first.

---

# 3. CI/CD PIPELINES & TOOLS

## Concept & Why
Continuous Integration (CI) automatically builds and tests code on every
commit. Continuous Delivery/Deployment (CD) ensures that the artefact
can be released to production at any time, often automatically.
Why: Fast feedback, reduced integration pain, repeatable releases, and
lower risk.

## Pipeline Flow
Source → Build → Unit Test → Static Analysis → Dependency Scan →
Artifact Build → Image Build → Image Scan → Push to Registry →
Deploy (Dev/Staging) → Integration Tests → Approval → Deploy (Prod)

## Tools
- **Jenkins:** Highly customisable, uses Jenkinsfile (Groovy).
- **GitHub Actions:** YAML workflows triggered by events.
- **GitLab CI:** Integrated with GitLab, YAML‑based.
- **Azure DevOps:** Rich suite with classic and YAML pipelines.

## Key Principles
- Fail fast: stop the pipeline on the first error.
- Immutable artifacts: build once, promote the same binary/image.
- Shift‑left security: integrate SAST, SCA, and container scanning early.
- Separate build and deploy stages.

## Sample Jenkinsfile (simplified)
```groovy
pipeline {
    agent any
    environment { IMAGE = "repo/app:${env.BUILD_ID}" }
    stages {
        stage('Code Scan') { steps { sh 'sonar-scanner' } }
        stage('Build Image') { steps { sh 'kaniko --context . --destination $IMAGE' } }
        stage('Image Scan') { steps { sh 'trivy image --exit-code 1 --severity HIGH,CRITICAL $IMAGE' } }
        stage('Push') { steps { sh 'docker push $IMAGE' } }
    }
    post { always { cleanWs() } }
}
```
## GitHub Actions Example
```yaml
name: CI
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: mvn clean package
      - run: sonar-scanner
      - run: docker build -t $IMAGE .
      - run: trivy image --severity HIGH,CRITICAL $IMAGE
```
## Debugging Pipeline Failures
- Read the logs of the failed stage carefully.
- Check environment variables and secrets are set.
- Verify network connectivity (dependency repositories).
- For build failures, isolate dependency issues (`mvn dependency:tree`).
- Never bypass a failed quality gate; fix the root cause.

## Rollback & Failure Handling
- Use `kubectl rollout undo deployment/app` for Kubernetes.
- Keep last N successful artifacts in the registry.
- Implement automatic rollback if health probes fail.
```
---
```
# 4. BUILD TOOLS & ARTIFACT MANAGEMENT

## Concept & Why
Build tools automate compilation, testing, and packaging. They handle
dependencies and produce a repeatable artifact.
Why: Consistency, speed, and reproducibility across dev, CI, and
production.

## Maven
- Lifecycle: validate → compile → test → package → install → deploy
- `pom.xml` defines project, dependencies, and plugins.
- Command: `mvn clean install`
- Debug: `mvn dependency:tree` to find conflicts.

## Gradle
- Uses Groovy/Kotlin DSL (`build.gradle`).
- Incremental builds and build cache for speed.
- Command: `gradle build`

## Artifact Repositories
- Nexus, Artifactory, or cloud registries (ECR, ACR, GAR).
- Store built JARs, WARs, Node modules, etc.
- Use snapshot vs release repositories, and clean up old snapshots.

## Container Image Registries
- Docker Hub, ECR, GCR, ACR.
- Tag images with version and commit SHA.
- Scan images before pushing to a production registry.

## Best Practices
- Never store secrets in `pom.xml` or `build.gradle`.
- Use dependency lockdown (e.g., `package-lock.json`, `Gemfile.lock`).
- Reproducible builds: avoid timestamps in artifacts.

---

# 5. CONTAINERIZATION (DOCKER)

## Concept & Why
A container is a lightweight, isolated process that shares the host
kernel. Docker is the de facto tool for building and running them.
Why: “It works on my machine” becomes reality, enabling portable,
fast, and secure deployments.

## Docker Architecture
- Dockerfile → build → Image → run → Container
- Registry stores images for distribution.

## Key Commands
- `docker build -t app:v1 .`
- `docker run -d -p 80:8080 app:v1`
- `docker ps`, `docker logs`, `docker exec -it <id> sh`
- `docker push/pull`

## Dockerfile Best Practices
- Use specific base image tags (avoid `latest`).
- Use multi‑stage builds to reduce size.
- Run as non‑root user.
- Minimise layers by combining RUN commands.
- Example:
```dockerfile
FROM eclipse-temurin:17-jre-alpine AS builder
WORKDIR /app
COPY target/app.jar app.jar

FROM gcr.io/distroless/java17-debian11
COPY --from=builder /app/app.jar /app.jar
EXPOSE 8080
USER 1000
ENTRYPOINT ["java","-jar","/app.jar"]
```
## Kaniko & BuildKit
- Kaniko: builds images inside Kubernetes without Docker daemon.
- BuildKit: modern build engine with parallel builds, secrets, cache.
  Use `docker buildx build` to enable.

## Image Security
- Scan with Trivy or Clair before pushing.
- Use minimal base images (distroless, Chainguard, or WolfiOS).
- Regularly rebuild images to pick up OS patches.

## Debugging Container Issues
- `docker logs <container>` – check application logs.
- `docker inspect <container>` – full configuration.
- `docker exec` to shell into the running container.
- Image pull errors: verify registry URL, image name/tag, and
  credentials (`imagePullSecrets` in Kubernetes).

---

# 6. CONTAINER ORCHESTRATION (KUBERNETES)

## Concept & Why
Kubernetes (K8s) orchestrates containers across a cluster, handling
scheduling, scaling, self‑healing, and service discovery.
Why: It provides a declarative model to manage complex, resilient
distributed systems.

## Core Objects
- Pod: smallest deployable unit (one or more containers).
- Deployment: manages replicas, rolling updates, and rollbacks.
- Service: stable network endpoint (ClusterIP, NodePort, LoadBalancer).
- ConfigMap/Secret: non‑sensitive/sensitive configuration.
- Ingress: HTTP routing to services.
- HPA (HorizontalPodAutoscaler): scales pods based on CPU/memory or
  custom metrics.

## Desired State Flow
1. User applies YAML → `kubectl apply -f deployment.yaml`
2. API Server stores object in etcd.
3. Controller Manager notices difference between desired and current →
   creates pods.
4. Scheduler assigns pods to nodes.
5. Kubelet on node starts containers.

## Key Commands
- `kubectl get pods -n <ns>`
- `kubectl describe pod <name>` – debug events.
- `kubectl logs <pod> -c <container>`
- `kubectl exec -it <pod> -- sh`
- `kubectl rollout status deployment/app`
- `kubectl rollout undo deployment/app`

## YAML Examples
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app
  template:
    metadata:
      labels:
        app: app
    spec:
      containers:
      - name: app
        image: repo/app:v1
        ports:
        - containerPort: 8080
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
```
---
```yaml
apiVersion: v1
kind: Service
metadata:
  name: app-svc
spec:
  selector:
    app: app
  ports:
  - port: 80
    targetPort: 8080
  type: LoadBalancer
```
## Common Pod Phases & Debugging
- CrashLoopBackOff: check logs for application error, missing config,
  or OOMKilled.
- ImagePullBackOff: incorrect image name/tag, missing
  imagePullSecrets, network issue.
- Pending: insufficient CPU/memory, no matching node, or volume
  binding.
- OOMKilled: increase memory limits or fix memory leak.
- Use `kubectl describe pod` and `kubectl get events` as first steps.

## Best Practices
- Always define resource requests and limits.
- Use readiness and liveness probes.
- Run containers as non‑root and read‑only filesystem if possible.
- Use namespaces to separate environments.
- Apply network policies to restrict traffic.

---

# 7. INFRASTRUCTURE AS CODE (IAC)

## Concept & Why
IaC manages infrastructure (servers, networks, databases) through
machine‑readable definition files, not manual processes.
Why: Version control, repeatability, consistency, and collaboration.

## Tools
- Terraform: declarative, provider‑based, state‑driven.
- Pulumi: uses general‑purpose languages (TypeScript, Python) for
  infrastructure.
- Crossplane: Kubernetes‑native, uses CRDs to manage cloud resources.

## Terraform Workflow
1. Write configuration (.tf files).
2. `terraform init` – initialise providers and backends.
3. `terraform plan` – preview changes.
4. `terraform apply` – apply changes.
5. Store state remotely (S3 + DynamoDB locking) for team use.
6. Use modules for reusable components.

## Terraform Example (AWS EC2)
```hcl
data "aws_ami" "latest" {
  most_recent = true
  owners      = ["amazon"]
  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.latest.id
  instance_type = "t2.micro"
  tags = { Name = "web-server" }
}
```
## Terraform State & Remote Backend
- State maps real resources to configuration; keep it secure and locked.
- Use S3 backend with DynamoDB for locking and versioning.

## Debugging Terraform
- `terraform validate` for syntax.
- `terraform plan` to see drift.
- Check provider credentials and API limits.
- `terraform refresh` to update state from real world.
- For dependency errors, check remote state and module versions.

## Other IaC Tools
- Chef, Puppet (agent‑based configuration management)
- Cloud‑specific: AWS CloudFormation, Azure ARM/Bicep

---

# 8. CLOUD PLATFORMS (AWS, AZURE, GCP)

## Concept & Why
Cloud provides on‑demand compute, storage, and networking. The big
three share core concepts but differ in details.
Why: Elasticity, global reach, managed services, pay‑as‑you‑go.

## AWS Core Services
- Compute: EC2, Auto Scaling Groups (ASG), Launch Templates, Lambda
- Storage: S3 (object), EBS (block), EFS (file)
- Networking: VPC, subnets, security groups, ALB/ELB, Route53
- Identity: IAM roles, policies (least privilege)
- Container: EKS, ECR, ECS Fargate

## Azure Core Services
- Compute: Virtual Machines, VM Scale Sets, AKS, Functions
- Storage: Blob Storage, Managed Disks, Files
- Networking: VNet, NSG, Azure Load Balancer, DNS
- Identity: Azure AD, Managed Identities

## GCP Core Services
- Compute: Compute Engine, GKE, Cloud Run, Cloud Functions
- Storage: Cloud Storage, Persistent Disk
- Networking: VPC, firewall rules, Cloud Load Balancing
- Identity: IAM, service accounts

## Multi‑Cloud & Hybrid
- Use Terraform or Crossplane to abstract differences
- Challenges: data egress costs, operational complexity
- Hybrid cloud: on‑prem + cloud (Azure Arc, AWS Outposts)

## Security & Cost
- Always apply least privilege IAM
- Encrypt data at rest and in transit
- Enable cost budgets, use spot/preemptible instances for non‑critical workloads, rightsize resources

---

# 9. CONFIGURATION MANAGEMENT & AUTOMATION (ANSIBLE)

## Concept & Why
Ansible automates provisioning, configuration, and application
deployment using agentless SSH.
Why: Idempotent, simple YAML playbooks, large community modules.

## Key Concepts
- Inventory: defines hosts and groups
- Playbook: list of plays (tasks) applied to hosts
- Module: idempotent unit of work (e.g., `copy`, `yum`, `service`)
- Roles: reusable collection of tasks, handlers, templates

## Example Playbook
```yaml
- name: Install Nginx
  hosts: webservers
  become: yes
  tasks:
    - name: Ensure nginx installed
      apt:
        name: nginx
        state: present
    - name: Start nginx
      service:
        name: nginx
        state: started
```
## Idempotency
- Running the playbook multiple times results in the same desired state

## Debugging Ansible
- Use `-v` or `-vvv` for verbosity
- Check connectivity: `ansible all -m ping`
- Check gather facts for OS differences
- Syntax check: `ansible
-playbook --syntax-check playbook.yml`

---

# 10. DEPLOYMENT STRATEGIES & RELEASE ENGINEERING

## Deployment Strategies
- **Rolling Update**: Gradual replacement of instances. Default in K8s Deployments. Good for stateless apps.
- **Blue‑Green**: Maintain two environments (blue/active, green/idle). Switch traffic instantly. Zero downtime, fast rollback.
- **Canary**: Route a small percentage of traffic to new version, monitor, then increase. Lowers risk.
- **Feature Flags**: Decouple deployment from release. Toggle features without redeploying.

## Release Engineering
- Semantic Versioning (SemVer): MAJOR.MINOR.PATCH
- CI generates versioned artifacts, store in registry
- Deploy pipelines promote artifacts through environments (dev→QA→staging→prod)
- Rollback: `kubectl rollout undo` or switch traffic back
- Roll‑forward: Fix forward in code and deploy a new version

## Best Practices
- Always have a tested rollback plan
- Use health probes and metrics to validate deployment success
- Gradual rollout with automated canary analysis

---

# 11. GITOPS

## Concept & Why
GitOps uses Git as the single source of truth for infrastructure and
application configuration. An operator (Argo CD, Flux) continuously
reconciles the cluster with the Git repo.
Why: Complete audit trail, drift detection, consistency, secure pull model.

## Flow
1. Developer merges a configuration change (e.g., new image tag) into the environment repo.
2. Argo CD detects the change and syncs the cluster.
3. If drift occurs (manual change), Argo CD can auto‑heal by reverting to the Git state.

## Argo CD
- Application CRD defines what to sync from which repo
- `argocd app sync myapp`
- `argocd app diff myapp` to see drift

## Debugging GitOps
- Check Argo CD/Flux logs
- Verify repository URL and credentials
- Ensure the desired state is valid Kubernetes YAML
- If sync fails, inspect the error message in the operator

---

# 12. SECURITY & DEVSECOPS

## Concept & Why
DevSecOps integrates security into every phase of the delivery
pipeline, shifting left to catch vulnerabilities early.
Why: Prevent breaches, comply with regulations, reduce cost of fixes.

## Security Toolchain
- SAST: SonarQube (code quality, bugs, code smells)
- SCA: Snyk, Dependabot (dependency vulnerabilities)
- Container Scanning: Trivy, Clair (image CVEs)
- DAST: OWASP ZAP (dynamic testing)
- Secrets Management: HashiCorp Vault, AWS Secrets Manager, Azure Key Vault
- Policy as Code: OPA/Gatekeeper, Kyverno
- Runtime Security: Falco, Tetragon

## Pipeline Security
- Run scanners at the build stage
- Fail the build on HIGH/CRITICAL findings
- Sign artifacts with Cosign and verify before deployment
- Never hardcode secrets; use environment masking and vault

## Secrets Management
- Dynamic secrets (short‑lived) are preferred
- Inject secrets at runtime via CSI drivers (Secrets Store CSI) or Vault Agent
- Rotate credentials automatically

## Image Security
- Use minimal, updated base images (Chainguard, distroless)
- Scan images in CI, block vulnerable ones
- Run containers as non‑root, read‑only root filesystem

## Debugging Security Issues
- Check scanner logs for specific CVEs and fix recommendations
- For policy violations, read the admission controller logs (Kyverno/OPA)
- Audit IAM roles regularly; `aws iam get-account-authorization-details`

---

# 13. MONITORING, OBSERVABILITY & LOGGING

## Concept & Why
Monitoring: collecting metrics. Observability: deriving internal
system state from outputs (metrics, logs, traces).
Why: Detect issues, understand system behaviour, and data‑driven decisions.

## Golden Signals (Google SRE)
- Latency, Traffic, Errors, Saturation
- Monitor them for every service

## Tool Stack
- Metrics: Prometheus (scrapes), Alertmanager (alerts), Grafana (dashboards)
- Logs: Elasticsearch, Fluentd/Fluent Bit, Kibana (ELK) or Loki
- Traces: Jaeger, Tempo, OpenTelemetry
- Unified: Grafana Alloy, OpenTelemetry collector

## Prometheus Setup
- `prometheus.yml` defines scrape targets
- Query with PromQL: `rate(http_requests_total[5m])`

## Grafana
- Build dashboards with panels for CPU, memory, latency, error rate
- Set alerts in Grafana or Alertmanager

## Logging Architecture
- Application → stdout/stderr → Fluent Bit → Elasticsearch/Loki
- Structure logs in JSON for easier parsing

## Correlation
- Inject request IDs and propagate through services
- Link logs, traces, and metrics using a common trace ID

## Debugging Observability
- Check if metric endpoint is reachable (`curl localhost:9100/metrics`)
- Verify Fluent Bit/Logstash connectivity
- Use `kubectl logs` to see logs inside a pod if central logging fails
- `tcpdump` on high ports to see network flows

---

# 14. RELIABILITY ENGINEERING (SRE)

## Core Concepts
- SLI (Service Level Indicator): actual measurement (e.g., latency p95)
- SLO (Service Level Objective): target for SLI (p95 < 200ms over 30 days)
- SLA (Service Level Agreement): contractual promise to customers
- Error Budget: allowed failure (1 – SLO). If exhausted, halt feature releases and focus on reliability.

## Incident Management Flow
1. Detect (alert fired) → Acknowledge → Join incident bridge
2. Roles: Incident Commander, Operations, Communications
3. Mitigation first (rollback, scale, failover); root cause later
4. Post‑Incident Review (blameless RCA)
5. Track action items to prevent recurrence

## Error Budget Policy
- When error budget is burnt fast, freeze deployments
- Automate canary analysis to stay within budget

## Debugging Reliability Issues
- Check error budget dashboards and burn rate alerts
- Correlate metrics, logs, and traces; identify what changed (deployment, config, traffic)

---

# 15. PERFORMANCE OPTIMIZATION & FINOPS

## Performance Tuning
- CPU/memory: adjust Kubernetes requests/limits, profile app
- Caching: Redis for hot data, CDN for static assets
- Database: indexing, connection pooling, read replicas
- Code: avoid N+1 queries, use async processing

## Load Testing
- Tools: k6, JMeter, Locust
- Establish baseline, ramp up, monitor Golden Signals

## FinOps (Cloud Cost Management)
- Rightsizing: match resources to actual usage
- Use Reserved Instances/Savings Plans for steady workloads, Spot for fault‑tolerant
- Auto‑scaling to match demand
- Lifecycle policies for storage (S3 Intelligent‑Tiering)
- Continuous cost monitoring (AWS Cost Explorer, Kubecost)

## Debugging Performance
- Slow app: check CPU/memory metrics, DB query times, upstream latency
- Use `kubectl top`, `htop`, `pg_stat_statements`
- Trace requests with Jaeger to find bottlenecks

---

# 16. PLATFORM ENGINEERING & INTERNAL DEVELOPER PLATFORMS

## Concept & Why
Building a self‑service platform (IDP) that provides paved paths
(golden paths) for developers to deploy and operate their services.
Why: Reduces cognitive load, ensures best practices, and accelerates delivery at scale.

## Components
- Service catalog (Backstage)
- CI/CD templates (GitHub Actions, Jenkins shared libraries)
- Infrastructure modules (Terraform, Crossplane)
- Observability defaults (pre‑configured dashboards, alerts)
- Policy guardrails (Kyverno, OPA)

## Golden Paths
- Pre‑defined, supported ways to build a service (e.g., Spring Boot + PostgreSQL + K8s). Developers start from a template and get production readiness automatically.

## Platform as a Product
- Treat platform with SLOs, feedback loops, and iteration

## Debugging Platform Issues
- Check platform API logs, template versions, and permission boundaries. Validate that the platform’s own SLOs are met.

---

# 17. NETWORKING & STORAGE

## Networking Fundamentals
- DNS: name to IP; latency matters; cache heavily
- Load Balancer: L4 (TCP), L7 (HTTP) – ALB, Nginx, HAProxy
- Reverse Proxy: terminates SSL, routes requests (Nginx, Envoy)
- TLS/SSL: encrypt data in transit; use TLS 1.3, rotate certificates

## Kubernetes Networking
- Pod‑to‑Pod communication via CNI (Calico, Cilium)
- Service types: ClusterIP, NodePort, LoadBalancer, ExternalName
- Ingress & Gateway API: HTTP routing
- Network Policies: restrict traffic; default deny then allow

## Storage Types
- Block (SAN, EBS, Persistent Disks) – for databases, low‑latency
- Object (S3, Blob, GCS) – for files, backups, static assets
- File (NFS, EFS, GlusterFS) – shared storage across pods

## Kubernetes Persistent Volumes
- PV/PVC lifecycle: provision → bind → use
- Use storage classes for dynamic provisioning

## Debugging Networking
- `curl -v`, `nslookup`, `dig`, `traceroute`, `tcpdump`
- Check Kubernetes endpoint objects: `kubectl get endpoints`
- Network policy issues: `kubectl describe networkpolicy`

---

# 18. DATA ENGINEERING & MESSAGING (KAFKA, SPARK)

## Apache Kafka
- Distributed event streaming platform
- Producers → Brokers → Consumers
- Topics partitioned for scale
- Use for asynchronous decoupling, event sourcing, real‑time analytics

## Kafka Monitoring
- Consumer lag: `kafka-consumer-groups --describe`
- Broker metrics: under‑replicated partitions, disk usage

## Apache Spark
- Batch and stream processing
- Spark Streaming for real‑time, Spark SQL for analytics
- Integrate with Kafka, S3

## DevOps Role in Data Pipelines
- CI/CD for pipeline code (dbt, Airflow DAGs)
- Monitor data freshness, quality, and pipeline failures
- Airflow: DAGs as code, use Kubernetes Executor

## Debugging Kafka/Spark
- Check consumer group lag; if high, scale consumers
- Dead letter queue (DLQ) for poisonous messages
- Spark: check driver logs, executor OOM

---

# 19. ADVANCED KUBERNETES

## Helm
- Package manager for K8s; `helm install myapp ./chart`
- Use values.yaml for environment‑specific config
- Helm hooks for pre/post install jobs

## StatefulSets & DaemonSets
- StatefulSet: for stateful apps with stable network ID and storage
- DaemonSet: run one pod per node (e.g., logging agent)

## RBAC
- Role/ClusterRole define permissions; RoleBinding binds to users/groups
- Always apply least privilege

## Network Policies
- `ingress` and `egress` rules to restrict traffic

## Service Mesh (Istio)
- Manages traffic (canary, retries), security (mTLS), observability
- Sidecar proxy (Envoy) injected per pod

## Autoscaling
- HPA based on CPU/memory or custom metrics (Prometheus adapter)
- Vertical Pod Autoscaler (VPA) adjusts requests/limits
- Cluster Autoscaler & Karpenter for nodes

## Debugging Advanced K8s
- HPA not scaling: check metrics server, custom metrics API
- Network policy blocking: check policy rules with `kubectl describe netpol`
- Helm release stuck: `helm rollback`, check hook logs

---

# 20. INCIDENT MANAGEMENT & TROUBLESHOOTING PLAYBOOKS

## General Approach
1. Identify symptom → check metrics, logs, events
2. Mitigate impact (rollback, scale, traffic shift)
3. Perform root cause analysis (RCA) using the 5 Whys
4. Implement long‑term fix and prevent recurrence

## High‑Frequency Playbooks
- **Pod CrashLoopBackOff**: `kubectl logs` & `describe`; check OOM or config
- **ImagePullBackOff**: verify image name, registry auth, network
- **High CPU/Memory**: scale, profile, check for leaks
- **Slow API**: trace requests, check DB, add caching
- **Deployment Failure**: `kubectl rollout status`, revert if needed
- **DNS Failure**: `nslookup service name`; check CoreDNS pods
- **Node NotReady**: check kubelet, disk/memory pressure
- **Certificate Expiry**: `openssl x509 -dates`; automated renewal

## Full Incident Timeline (Production Outage Example)
- T0: Alert fires → Acknowledge, join bridge
- T1: Triage dashboards: latency spike, errors high, CPU normal → suspect downstream dependency
- T2: Check pods: all running → `curl` shows slow response
- T3: Logs reveal database timeout errors
- T4: Database check: high disk IO, recent deployment introduced unoptimised query
- T5: Mitigation: rollback deployment
- T6: Verify recovery, repeat until metrics normalise
- T7: Root cause: lack of query performance review
- T8: Permanent fix: add slow‑query alert, enforce DB review in deployment process

---

# 21. SYSTEM ARCHITECTURE & DESIGN PATTERNS

## Key Patterns
- **Three‑Tier**: presentation, logic, data
- **Microservices**: independently deployable, owned by teams
- **Event‑Driven**: services react to events (Kafka), loose coupling
- **CQRS & Event Sourcing**: separate read/write models, store events

## High Availability & Scalability
- Deploy across multiple AZs/regions
- Stateless services behind load balancers; stateful services with replication
- Horizontal scaling preferred

## Resilience Patterns
- Circuit Breaker, Retries with exponential backoff, Bulkhead, Timeout, Graceful degradation
- Chaos Engineering: inject failures (Gremlin, LitmusChaos) to test resilience

## Disaster Recovery
- Define RTO (time to recover) and RPO (max data loss)
- Regular backups (snapshots), cross‑region replication
- Test restore process quarterly

---

# 22. BACKUP, DISASTER RECOVERY & HIGH AVAILABILITY

## Backup Strategies
- Database: automated snapshots + point‑in‑time recovery
- Persistent volumes: snapshots or Velero for K8s
- Configuration: Git (source of truth)
- Verify backups: actual restore in sandbox

## Disaster Recovery
- Active‑Passive: standby environment in another region, traffic switched via DNS
- Active‑Active: both regions serving traffic, lower RTO but higher cost

## High Availability
- Eliminate single points of failure: redundant load balancers, DB replicas
- K8s control plane HA (managed by cloud provider)

---

# 23. SUPPLY CHAIN SECURITY & ZERO TRUST

## Software Supply Chain
- From code to deployment, every step must be verifiable
- SLSA framework (Level 1‑4) for build integrity
- SBOM (Software Bill of Materials) to track dependencies

## Artifact Signing
- Use Cosign to sign container images and verify during admission (Kyverno policy)
- Rekor transparency log for audit

## Zero Trust Principles
- Never trust, always verify; micro‑segmentation
- Every request authenticated and authorised
- Use mTLS between services (Istio, Linkerd)

## Runtime Threat Detection
- Falco, Tetragon for anomalous syscalls
- Automated response: kill pod, isolate namespace

---

# 24. AI IN DEVOPS (AIOPS, MLOPS)

## AIOps
- AI/ML used for anomaly detection, root cause analysis, and automated remediation
- Tools: Splunk ITSI, Dynatrace, or custom models
- Guardrails: human approval for critical actions

## MLOps
- Extend CI/CD to ML: version data, model, code
- Pipeline: data prep → training → evaluation → packaging → deployment (model serving)
- Model monitoring for drift and accuracy

## Caution
- AI assists, does not replace engineering judgement
- Start with simple rule‑based alerts, add AI incrementally

---

# 25. BEST PRACTICES, COMMON MISTAKES & GOLDEN RULES

## Best Practices
- Automate everything: builds, tests, deployments, scaling
- Infrastructure as Code and GitOps for every environment
- Observability: metrics, logs, traces from day one
- Security: shift left, least privilege, secrets never in plain text
- Cost: tag resources, set alerts, right‑size, use spot/preemptible
- Documentation: runbooks, architecture diagrams, decision records

## Common Mistakes
- Hardcoding secrets in code or config
- No monitoring/alerting
- No tested rollback strategy
- Over‑engineering (microservices before needed)
- Ignoring cost until it’s too late
- Running containers as root
- Neglecting CI/CD pipeline security

## Golden Rules
- Design for failure: every component can and will fail
- Simplicity is the ultimate sophistication; avoid unnecessary complexity
- The network is unreliable; add retries, timeouts, and circuit breakers
- If you can’t measure it, you can’t fix it
- A rollback is always safer than a risky fix‑forward in an emergency
- Culture eats tools for breakfast

---

# 26. APPENDICES – QUICK REFERENCE CARDS

## Git Cheat Sheet
- `git clone`, `checkout -b`, `add`, `commit`, `push`, `pull --rebase`,
  `log --oneline --graph`, `stash`, `reflog`

## Docker Commands
- `build -t`, `run -d -p`, `exec -it`, `logs`, `ps`, `inspect`, `system prune`

## Kubernetes `kubectl`
- `get pods/deploy/svc`, `describe`, `logs -f`, `exec`, `apply -f`,
  `delete`, `rollout status/undo`, `top`, `cordon/uncordon`

## Terraform Snippets
- `init`, `plan`, `apply -auto-approve`, `destroy`, `state list`, `import`

## Troubleshooting Quick Lookup
- Pod Pending → `describe`, check resources/node
- CrashLoop → `logs`, check OOM, config, probes
- ImagePull → `describe`, verify imagePullSecrets and name
- Service unreachable → `get endpoints`, check selector
- Slow DB → `EXPLAIN`, index, connection pool
- Build fail → `mvn dependency:tree`, check network

---

### Ingress YAML
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-svc
            port:
              number: 80
  tls:
  - hosts:
    - app.example.com
    secretName: app-tls-secret
```
---

### ConfigMap & Secret YAML
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database-url: "jdbc:postgresql://db:5432/app"
  log-level: "debug"
---
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  database-password: cGFzc3dvcmQ=   # base64 encoded
```
---

### NetworkPolicy YAML
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
spec:
  podSelector:
    matchLabels:
      tier: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: frontend
    ports:
    - protocol: TCP
      port: 8080
```
---

### RBAC Example YAML
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: deployer
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-manager
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "create", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: deployer-pod-manager
subjects:
- kind: ServiceAccount
  name: deployer
roleRef:
  kind: Role
  name: pod-manager
  apiGroup: rbac.authorization.k8s.io
```
---

### HorizontalPodAutoscaler (HPA) YAML
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```
---

### ArgoCD Application YAML
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/org/repo
    targetRevision: HEAD
    path: k8s/overlays/prod
  destination:
    server: https://kubernetes.default.svc
    namespace: prod
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```
---

### Helm values.yaml snippet
```yaml
# values.yaml
replicaCount: 3
image:
  repository: nginx
  tag: "1.25"
  pullPolicy: IfNotPresent
service:
  type: ClusterIP
  port: 80
ingress:
  enabled: false
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi
```
---

## 27. ADDITIONAL ADVANCED CONFIGURATIONS

### Prometheus Alert Rule
```yaml
groups:
- name: example
  rules:
  - alert: HighErrorRate
    expr: rate(http_requests_total{status="500"}[5m]) > 0.1
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "High 5xx error rate ({{ $value }} req/s)"
```
### Grafana Dashboard Panel (JSON snippet)
```json
{
  "id": 1,
  "title": "Request Latency p95",
  "type": "graph",
  "targets": [
    {
      "expr": "histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))",
      "legendFormat": "p95"
    }
  ],
  "yaxes": [
    { "format": "s", "label": "Seconds" }
  ]
}
```
---

### Full Multi‑Stage Dockerfile with Tests
```dockerfile
# Stage 1: Build
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package

# Stage 2: Unit & Integration Tests
FROM build AS test
RUN mvn verify

# Stage 3: Production Image
FROM gcr.io/distroless/java17-debian11
COPY --from=build /app/target/app.jar /app.jar
EXPOSE 8080
USER 1000
ENTRYPOINT ["java","-jar","/app.jar"]
```
---

### Kubernetes StatefulSet Example
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres-db
spec:
  serviceName: "postgres"
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:16
        ports:
        - containerPort: 5432
        env:
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
```
---

### Istio VirtualService + DestinationRule (Canary)
```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: app-vs
spec:
  hosts:
  - app.example.com
  http:
  - match:
    - headers:
        canary:
          exact: "true"
    route:
    - destination:
        host: app-svc
        subset: v2
  - route:
    - destination:
        host: app-svc
        subset: v1
      weight: 90
    - destination:
        host: app-svc
        subset: v2
      weight: 10
```
```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: app-dr
spec:
  host: app-svc
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```
---

### Terraform Remote State (S3 + DynamoDB)
```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-lock"
    encrypt        = true
  }
}
```
---

### Jenkins Pipeline – Multi‑Environment Deployment
```groovy
pipeline {
    agent any
    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: 'Select target environment')
    }
    environment {
        IMAGE = "myapp:${env.BUILD_ID}"
    }
    stages {
        stage('Build & Test') {
            steps { sh 'mvn clean verify' }
        }
        stage('Image Build & Scan') {
            steps {
                sh 'kaniko --context . --destination $IMAGE'
                sh 'trivy image --exit-code 1 --severity HIGH,CRITICAL $IMAGE'
            }
        }
        stage('Deploy to Dev') {
            when { expression { params.ENVIRONMENT == 'dev' } }
            steps { sh 'kubectl set image deployment/app app=$IMAGE --namespace=dev' }
        }
        stage('Deploy to Staging') {
            when { expression { params.ENVIRONMENT == 'staging' } }
            steps { sh 'kubectl set image deployment/app app=$IMAGE --namespace=staging' }
        }
        stage('Deploy to Production') {
            when { expression { params.ENVIRONMENT == 'prod' } }
            input { message 'Approve production deployment?' }
            steps { sh 'kubectl set image deployment/app app=$IMAGE --namespace=prod' }
        }
    }
    post {
        failure { notifySlack("Pipeline failed for ${params.ENVIRONMENT}") }
    }
}
```
---

### CloudWatch Alarm (AWS)
```json
{
  "AlarmName": "HighCPUAlarm",
  "MetricName": "CPUUtilization",
  "Namespace": "AWS/EC2",
  "Statistic": "Average",
  "Period": 300,
  "EvaluationPeriods": 2,
  "Threshold": 80,
  "ComparisonOperator": "GreaterThanThreshold",
  "Dimensions": [ { "Name": "InstanceId", "Value": "i-1234567890abcdef0" } ],
  "AlarmActions": [ "arn:aws:sns:us-east-1:123456789012:ops-team" ]
}
```
---

### Azure Monitor Metric Alert (example)
```json
{
  "type": "Microsoft.Insights/metricAlerts",
  "name": "high-cpu-alert",
  "properties": {
    "description": "Alert when CPU exceeds 80%",
    "severity": 2,
    "enabled": true,
    "scopes": ["/subscriptions/.../resourceGroups/rg/providers/Microsoft.Compute/virtualMachines/vm1"],
    "evaluationFrequency": "PT1M",
    "windowSize": "PT5M",
    "criteria": {
      "odata.type": "Microsoft.Azure.Monitor.SingleResourceMultipleMetricCriteria",
      "allOf": [
        {
          "name": "HighCPU",
          "metricName": "Percentage CPU",
          "operator": "GreaterThan",
          "threshold": 80,
          "timeAggregation": "Average"
        }
      ]
    }
  }
}
```
---

### Sample Runbook – On‑Call Checklist
- **Step 1**: Acknowledge alert within 1 minute (PagerDuty, Opsgenie)
- **Step 2**: Join incident bridge; assign Incident Commander, Ops Lead, Communications
- **Step 3**: Triage – check dashboards (Grafana) for metrics: latency, error rate, CPU, memory
- **Step 4**: If application issue, check pod logs (`kubectl logs -n <ns> <pod>`)
- **Step 5**: If infrastructure issue, verify nodes, network, database connectivity
- **Step 6**: Mitigate – rollback deployment (`kubectl rollout undo deployment/<name>`), scale up, or failover
- **Step 7**: Communicate status to stakeholders (Slack, email) every 15–30 minutes
- **Step 8**: After service restored, preserve logs and metrics for post‑incident review
- **Step 9**: Schedule blameless RCA within 24–48 hours
- **Step 10**: Document timeline, root cause, and action items in runbook/wiki

### SRE Error Budget Calculation Example
- **SLI**: Request success rate (non‑5xx) measured over a 30‑day window
- **SLO**: 99.9% success rate (i.e., 0.1% error budget)
- **Allowed errors per month** = (1 – 0.999) × total requests in 30 days
  - Example: 100 requests/sec → 100 × 86,400 × 30 ≈ 259.2 million requests/month
  - Allowed errors = 0.001 × 259,200,000 = 259,200 errors/month
- **Burn rate threshold**: If 10% of error budget consumed in 1 hour → alert
  - Fast burn = 10% of 259,200 = 25,920 errors in 1 hour → alert triggers
- **Action**: When error budget depleted, halt feature deployments and focus on reliability improvements


----------
**END OF DOCUMENT**
----------
