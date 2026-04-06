# Module 1 – Answers | DevOps & DevSecOps Fundamentals

> **Role context:** Infrastructure and Cloud Analyst (Intermediate) — Air Canada
> **Interview depth:** Production-grade, enterprise-level answers

---

## Task 1 – DevOps, DevSecOps & Why They Exist

---

### Q: Walk me through the evolution from Traditional SDLC to DevSecOps.

In traditional SDLC — whether Waterfall or early Agile — Development, QA, and Operations were completely siloed. Dev would write code for months, throw it "over the wall" to QA, who'd find defects late, and then throw it to Ops for a release that could take weeks. The feedback loop was broken, releases were risky, and security was a last-minute audit before go-live.

**Agile** shortened the development cycle to sprints, but Operations was still disconnected — you'd deliver faster code that still deployed slowly. That's where **DevOps** came in: it's a culture and practice of breaking the wall between Dev and Ops, automating the path from code commit to production. You measure success by deployment frequency, lead time for changes, mean time to recovery (MTTR), and change failure rate — the DORA metrics.

But DevOps alone didn't solve security. Security was still being done at the end — a penetration test before release, maybe a SAST scan as an afterthought. In production-grade environments, that's too late and too expensive. A defect found in production costs 10x more to fix than one found at commit time.

**DevSecOps** integrates security as a first-class citizen at every phase of the pipeline — from IDE plugins that catch secrets at commit time, to SAST/DAST in the CI pipeline, to runtime container security in Kubernetes, to compliance-as-code in IaC. At Air Canada, where you're dealing with PCI-DSS for payment data, PIPEDA for Canadian passenger data, and SOC 2 for cloud operations, shift-left security isn't optional — it's the only viable model.

---

### Q: What is Shift-Left Security and why does it matter in a cloud environment?

Shift-Left means moving security validation as early as possible in the development lifecycle — ideally to the developer's workstation before code even hits a pull request. The term comes from visualizing the SDLC as a timeline left to right: `code → build → test → deploy → operate`. Shifting left means you're catching issues before they travel rightward.

In practice this means:

- **Pre-commit hooks** using `gitleaks` or `trufflehog` to block secrets from entering the repo
- **SAST** (Static Application Security Testing) like SonarQube or Checkmarx running in the PR pipeline
- **SCA** (Software Composition Analysis) like Snyk or Dependabot catching vulnerable dependencies at build time
- **IaC scanning** with Checkov or tfsec so your Terraform doesn't deploy open S3 buckets or permissive security groups
- **Container image scanning** with Trivy before the image is pushed to ECR/ACR

In a cloud context this matters even more because a misconfigured IAM role or a publicly exposed storage account isn't just a dev mistake — it's an attack surface in production within seconds of `terraform apply`.

---

### Evolution Summary

```
Traditional SDLC → Agile → CI/CD → DevOps → DevSecOps
     (Months)     (Sprints) (Automated) (Unified) (Secure by default)
```

---

### Comparison Table: Traditional SDLC vs DevOps vs DevSecOps

| Dimension          | Traditional SDLC          | DevOps                     | DevSecOps                          |
|--------------------|---------------------------|----------------------------|------------------------------------|
| Release cadence    | Monthly / Quarterly       | Daily / Weekly             | Daily / Weekly                     |
| Team structure     | Siloed Dev, QA, Ops       | Unified Dev + Ops          | Dev + Ops + Security               |
| Security phase     | End-of-project audit      | Ad-hoc                     | Every pipeline stage               |
| Feedback loop      | Weeks to months           | Hours to days              | Minutes (pipeline gate)            |
| Risk model         | Big-bang releases         | Small incremental          | Small + secure incremental         |
| Tools              | Manual deployment scripts | CI/CD, IaC                 | CI/CD + SAST/DAST/SCA/IaC scanning |
| Failure recovery   | Major incident, slow fix  | Automated rollback         | Automated rollback + audit trail   |

---

## Task 2 – How DevOps Works in Real Companies

---

### Q: Describe an end-to-end production DevOps workflow.

**1. Code Commit (Developer Workstation)**
Developer writes code in a feature branch. Pre-commit hooks run `gitleaks` and lint checks. A pull request is opened to `main` or `develop`.

**2. CI Pipeline Triggers (GitHub Actions / Azure DevOps / Jenkins)**
- Unit tests and code coverage gates (fail if < 80%)
- SAST scan via SonarQube — Quality Gate must pass
- Dependency vulnerability scan via Snyk / Dependabot
- IaC scan via Checkov if Terraform changes are present
- Build artifact (JAR, Docker image, Lambda zip)

**3. Artifact Creation & Secure Storage**
- Docker image built, scanned with Trivy for CVEs
- Image tagged with commit SHA and pushed to ECR / ACR (private registry)
- Build artifact published to Nexus or JFrog Artifactory with checksum verification

**4. Deploy to Staging / Pre-Prod**
- Kubernetes manifest or Helm chart applied to staging namespace
- Integration tests and DAST scan (OWASP ZAP) run against staging
- Performance baseline checked

**5. Production Deployment (with approval gate)**
- Change advisory board (CAB) approval or automated promotion policy
- Deployment via Blue-Green or Canary strategy (zero-downtime)
- Smoke tests verify critical paths are healthy

**6. Monitoring & Observability**
- Prometheus + Grafana for metrics
- Loki or Datadog for logs
- Jaeger / X-Ray for distributed traces
- Alerts on SLO breaches, error rate spikes, latency degradation
- PagerDuty on-call rotation for critical services

At each stage there is a feedback mechanism back to the developer — not just "it failed" but "here's the specific CVE, here's the SonarQube rule, here's the offending line."

---

### CI/CD Flow Diagram

```
Developer
    │
    ▼
[Feature Branch] ──► Pre-commit hooks (secrets scan, lint)
    │
    ▼
[Pull Request] ──► Code Review + SAST (SonarQube) + SCA (Snyk)
    │
    ▼
[CI Pipeline] ──► Unit Tests → Build → Image Scan (Trivy) → Push to Registry
    │
    ▼
[Deploy: Staging] ──► Integration Tests → DAST (OWASP ZAP) → Performance Baseline
    │
    ▼
[Approval Gate] ──► Manual or Automated Policy Check (CAB / OPA)
    │
    ▼
[Deploy: Production] ──► Canary (5% → 25% → 100%) with auto-rollback on SLO breach
    │
    ▼
[Monitoring] ──► Prometheus + Grafana + Loki + Alerts + PagerDuty On-Call
```

---

### Q: What are the key DevOps roles and what does each person actually do day-to-day?

**DevOps / Platform Engineer**
Owns the CI/CD platform, writes and maintains pipelines, manages IaC for cloud infrastructure (Terraform modules, Ansible playbooks), handles Kubernetes cluster upgrades, and builds internal developer tooling. Day-to-day: reviewing pipeline failures, writing Helm charts, onboarding new microservices into the platform.

**Cloud Infrastructure Analyst** *(the role at Air Canada)*
Focuses on cloud resource management, cost optimization, infrastructure provisioning, and working with dev teams to right-size and architect cloud workloads. Day-to-day: reviewing AWS Cost Explorer for anomalies, writing Terraform for new environments, troubleshooting networking issues (VPC peering, Transit Gateway, security groups), and supporting engineering teams.

**Security Engineer / DevSecOps Engineer**
Owns security tooling in the pipeline, manages CSPM tools like AWS Security Hub or Prisma Cloud, responds to security findings, and works with compliance teams for audits (PCI-DSS, PIPEDA, SOC 2).

**SRE (Site Reliability Engineer)**
Owns production reliability. Defines SLOs (e.g., 99.95% availability for the booking API), manages incident response, writes runbooks, performs blameless post-mortems, and drives toil reduction through automation.

---

### Q: Explain Canary Deployment — when would you use it at Air Canada?

A Canary deployment is a release strategy where you route a small percentage of production traffic to the new version — say 5% — while the remaining 95% continues to hit the stable version. You monitor error rates, latency, and business metrics for the canary group. If everything looks healthy after a defined period, you gradually shift more traffic:

```
5% → 25% → 50% → 100%
```

If the canary shows elevated errors, you immediately cut traffic back to 0% with no customer impact at scale.

**At Air Canada**, I'd use Canary for critical customer-facing APIs — the flight search, booking, or check-in service. These services have high traffic volume and zero tolerance for widespread failure. A Blue-Green swap would expose 100% of users immediately if there's a regression. A Canary limits the blast radius to 5% while giving you real production signal.

**Implementation options:**
- **Kubernetes**: Argo Rollouts or Istio service mesh for weighted traffic splitting
- **AWS**: CodeDeploy Canary with Lambda, or weighted target groups in ALB
- **Azure**: Azure Deployment Slots + Traffic Manager weighted routing

**Key discipline**: Define your success criteria *before* the deployment starts — not after. If p99 latency exceeds 500ms or error rate goes above 0.1% in the canary group, the deployment is automatically rolled back. That is production-grade — not just "watch it and see."

---

## Key Interview Tips for Air Canada Context

| Area | What to Emphasize |
|---|---|
| **Compliance** | PCI-DSS (payments), PIPEDA (Canadian passenger data), SOC 2 (cloud ops) |
| **Availability** | Airline booking = revenue-critical; any downtime = direct financial loss; target 99.95%+ SLO |
| **Cloud platforms** | Air Canada uses AWS and Azure — know both (VPC/VNet, IAM/RBAC, EKS/AKS, S3/Blob, CloudWatch/Azure Monitor) |
| **Quantify impact** | "Reduced deployment time from 2 hours to 15 minutes", "caught 12 critical CVEs before prod", "saved $40K/month in compute costs via right-sizing" |
| **Bridge role** | An Infrastructure & Cloud Analyst bridges dev teams and cloud operations — emphasize translating infrastructure concepts to non-technical stakeholders |
| **DORA metrics** | Deployment frequency, lead time, MTTR, change failure rate — know these and how your work improves them |
