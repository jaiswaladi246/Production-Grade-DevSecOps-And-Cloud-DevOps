# Module 7 – Tasks | Security Tools (Trivy, OWASP, Prowler, SBOM)

This module simulates how teams implement security checks across applications, containers, and cloud environments.

You will:

- Scan applications and images  
- Identify vulnerabilities  
- Fix security issues  
- Work with SBOM  
- Integrate security into CI/CD  

Complete tasks in sequence.

---

## Task 1 – DevSecOps Basics

**Objective**

Understand security in DevOps.

**What You Need to Do**

Create `security-basics.md`

Explain:

- DevSecOps  
- Shift-left security  
- Types of vulnerabilities  

**Deliverables**

- security-basics.md  

**Interview Connect**

- What is DevSecOps  

---

## Task 2 – Install Security Tools

**Objective**

Prepare environment.

**What You Need to Do**

Install:

- Trivy  
- OWASP dependency check tool  
- Prowler  

Verify installation

**Deliverables**

- Version outputs  

**Interview Connect**

- Why multiple tools are required  

---

## Task 3 – Trivy Filesystem Scan

**Objective**

Scan local project.

**What You Need to Do**

- Run Trivy on a project directory  

Analyze output:

- vulnerabilities  
- severity levels  

**Deliverables**

- Scan report  

**Interview Connect**

- What is CVE  

---

## Task 4 – Trivy Docker Image Scan

**Objective**

Scan container images.

**What You Need to Do**

- Pull a Docker image  
- Run Trivy scan  
- Identify vulnerabilities  

**Deliverables**

- Image scan report  

**Interview Connect**

- Why container scanning is critical  

---

## Task 5 – Fix Vulnerabilities in Image

**Objective**

Improve image security.

**What You Need to Do**

- Update base image or packages  
- Rebuild image  
- Re-run scan  

**Deliverables**

- Before and after scan results  

**Interview Connect**

- How to reduce vulnerabilities  

---

## Task 6 – OWASP Dependency Check (Java/Node)

**Objective**

Scan dependencies.

**What You Need to Do**

- Run OWASP dependency check  
- Identify vulnerable libraries  

**Deliverables**

- Dependency report  

**Interview Connect**

- Why third-party dependencies are risky  

---

## Task 7 – Fix Dependency Issues

**Objective**

Improve dependency security.

**What You Need to Do**

- Upgrade vulnerable libraries  
- Re-run scan  

**Deliverables**

- Updated report  

**Interview Connect**

- How dependency management affects security  

---

## Task 8 – Prowler Setup

**Objective**

Prepare cloud security tool.

**What You Need to Do**

- Install Prowler  
- Configure credentials  
- Run basic scan  

**Deliverables**

- Initial scan output  

**Interview Connect**

- What is cloud misconfiguration  

---

## Task 9 – Prowler Cloud Security Scan

**Objective**

Scan cloud environment.

**What You Need to Do**

- Run Prowler scan  

Identify:

- IAM issues  
- S3 misconfigurations  
- security gaps  

**Deliverables**

- Scan report  

**Interview Connect**

- Common AWS security mistakes  

---

## Task 10 – Fix Cloud Issues

**Objective**

Improve cloud security posture.

**What You Need to Do**

- Fix at least 2–3 issues found  
- Re-run scan  

**Deliverables**

- Before/after results  

**Interview Connect**

- How to secure cloud environments  

---

## Task 11 – SBOM Basics

**Objective**

Understand SBOM.

**What You Need to Do**

Create `sbom-basics.md`

Explain:

- What is SBOM  
- Why it is important  

**Deliverables**

- sbom-basics.md  

**Interview Connect**

- Why SBOM is required  

---

## Task 12 – Generate SBOM

**Objective**

Create SBOM for project.

**What You Need to Do**

- Use tool (Trivy or Syft)  
- Generate SBOM file  

**Deliverables**

- SBOM file  

**Interview Connect**

- What information SBOM contains  

---

## Task 13 – Analyze SBOM

**Objective**

Understand dependencies.

**What You Need to Do**

- Review SBOM  
- Identify risky components  

**Deliverables**

- Analysis notes  

**Interview Connect**

- How SBOM helps in security  

---

## Task 14 – CI/CD Integration (GitHub Actions)

**Objective**

Add security to pipeline.

**What You Need to Do**

- Add Trivy scan in workflow  
- Fail pipeline on high severity  

**Deliverables**

- Pipeline execution  

**Interview Connect**

- Why security should be in CI  

---

## Task 15 – CI/CD Integration (GitLab)

**Objective**

Integrate security checks.

**What You Need to Do**

- Add scan stage in .gitlab-ci.yml  
- Run Trivy or OWASP scan  

**Deliverables**

- Pipeline logs  

**Interview Connect**

- Security gates in pipelines  

---

## Task 16 – Security Policy Implementation

**Objective**

Enforce rules.

**What You Need to Do**

Define:

- allowed severity levels  
- blocking conditions  

Apply in pipeline

**Deliverables**

- Policy configuration  

**Interview Connect**

- What is security policy  

---

## Task 17 – Real Project Scan (End-to-End)

**Objective**

Apply all tools together.

**What You Need to Do**

Scan:

- code  
- dependencies  
- container  

Fix issues

Re-scan

**Deliverables**

- Full report  

**Interview Connect**

- How companies handle security  

---

## Task 18 – Notes and Documentation

**Objective**

Consolidate learning.

**What You Need to Do**

Create `notes-module-7.md` including:

- Trivy usage  
- OWASP checks  
- Prowler findings  
- SBOM concepts  
- Common issues and fixes  

**Deliverables**

- Notes file  

**Interview Connect**

- Explain security workflow  

---

## Learn in Public

After completing this module:

Share:

vulnerabilities found  

fixes applied  

tools used  

Suggested tags:

#Batch13 #DevSecOps #Security #Trivy #OWASP #Prowler #SBOM #LearnInPublic

---

## Outcome of Tasks

After completing these tasks, you will be able to:

- Scan applications and containers for vulnerabilities  
- Identify and fix dependency and image issues  
- Perform cloud security checks  
- Generate and analyze SBOM  
- Integrate security into CI/CD pipelines  
- Apply DevSecOps practices in real environments  