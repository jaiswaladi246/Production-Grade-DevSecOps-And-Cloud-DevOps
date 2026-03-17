# Module 5 – Tasks | CI/CD Tools (GitHub Actions, GitLab CI/CD, Jenkins)

This module simulates a real enterprise CI/CD environment.

You will:

- Build pipelines using GitHub Actions, GitLab CI/CD, and Jenkins  
- Work with real application repositories  
- Implement automation using webhooks and runners/agents  
- Handle secrets, RBAC, and secure configurations  
- Debug failures and design production-ready pipelines  

Complete tasks in sequence.

---

## Phase 1 – CI/CD Foundations

### Task 1 – CI/CD Basics and Workflow Understanding

**Objective**

Understand core CI/CD concepts and workflow design.

**What You Need to Do**

Create a file `cicd-basics.md`

Explain:

- CI vs CD vs Continuous Deployment  
- Pipeline stages (Build, Test, Deploy)  
- Comparison between GitHub Actions, GitLab CI/CD, and Jenkins  

**Deliverables**

- cicd-basics.md  

**Interview Connect**

- Difference between CI/CD and DevOps  

---

### Task 2 – Project Setup for Pipelines

**Objective**

Prepare application repositories for pipeline implementation.

**What You Need to Do**

Create `module-5-projects/`

Add:

- One Node.js application  
- One Java or Python application  

Ensure both can build and run locally

**Deliverables**

- Project structure  
- Local run proof  

**Interview Connect**

- Why pipelines fail even if code works locally  

---

## Phase 2 – GitHub Actions

### Task 3 – GitHub Actions Basic Pipeline

**Objective**

Create a basic CI workflow.

**What You Need to Do**

Create `.github/workflows/main.yml`

Add:

- Trigger on push  
- Install dependencies  
- Run build/test  

**Deliverables**

- Workflow YAML  
- Successful pipeline run  

**Interview Connect**

- What are runners in GitHub Actions  

---

### Task 4 – Multi-Step Workflow

**Objective**

Simulate a real pipeline structure.

**What You Need to Do**

Add stages:

- Build  
- Test  
- Artifact upload  

Use artifact upload action

**Deliverables**

- Updated workflow  
- Artifact proof  

**Interview Connect**

- Why artifacts are used in pipelines  

---

### Task 5 – Secrets and Environment Variables

**Objective**

Secure sensitive data in pipelines.

**What You Need to Do**

- Add secrets in repository settings  
- Use them inside workflow  
- Ensure values are not exposed in logs  

**Deliverables**

- YAML using secrets  

**Interview Connect**

- Why secrets should not be hardcoded  

---

### Task 6 – Deployment Using GitHub Actions

**Objective**

Automate deployment process.

**What You Need to Do**

- Deploy application to a VM or cloud instance  
- Use SSH-based deployment  
- Restart service after deployment  

**Deliverables**

- Deployment logs  

**Interview Connect**

- Difference between CD and Continuous Deployment  

---

## Phase 3 – GitLab CI/CD

### Task 7 – GitLab Runner Setup

**Objective**

Understand execution model in GitLab.

**What You Need to Do**

- Install GitLab Runner  
- Register runner with project  
- Verify runner status  

**Deliverables**

- Runner status proof  

**Interview Connect**

- Shared vs specific runners  

---

### Task 8 – GitLab Pipeline Part 1

**Objective**

Create a basic pipeline.

**What You Need to Do**

Create `.gitlab-ci.yml`

Define:

- stages (build, test)  
- simple jobs  

**Deliverables**

- Successful pipeline execution  

**Interview Connect**

- Difference between GitHub and GitLab pipelines  

---

### Task 9 – GitLab Pipeline Part 2 (Advanced)

**Objective**

Enhance pipeline for production use.

**What You Need to Do**

Add:

- artifacts  
- caching  
- environment variables  
- rules or conditions  

**Deliverables**

- Advanced pipeline YAML  

**Interview Connect**

- What is caching in CI/CD  

---

### Task 10 – GitLab Deployment Pipeline

**Objective**

Deploy application using GitLab pipeline.

**What You Need to Do**

- Deploy application using pipeline  
- Use SSH or container-based approach  
- Verify application availability  

**Deliverables**

- Deployment proof  

**Interview Connect**

- Why GitLab is widely used in enterprises  

---

## Phase 4 – Jenkins Core

### Task 11 – Jenkins Setup and Exploration

**Objective**

Understand Jenkins architecture and UI.

**What You Need to Do**

- Install Jenkins  
- Configure admin user  
- Explore dashboard and plugins  

**Deliverables**

- Running Jenkins instance  

**Interview Connect**

- Why Jenkins is still widely used  

---

### Task 12 – Jenkins Freestyle Job

**Objective**

Create and run a basic job.

**What You Need to Do**

- Create Freestyle job  
- Add build step (Node/Java build)  
- Trigger manually  

**Deliverables**

- Job execution logs  

**Interview Connect**

- Freestyle vs pipeline jobs  

---

### Task 13 – Jenkins Declarative Pipeline

**Objective**

Implement pipeline as code.

**What You Need to Do**

- Create Jenkinsfile  
- Define stages (build, test, deploy)  

**Deliverables**

- Jenkinsfile  
- Pipeline execution  

**Interview Connect**

- Declarative vs scripted pipeline  

---

### Task 14 – Jenkins Multibranch Pipeline

**Objective**

Automate builds for branches.

**What You Need to Do**

- Create multibranch pipeline  
- Connect repository  
- Trigger build on new branch  

**Deliverables**

- Branch-based pipeline execution  

**Interview Connect**

- Importance of multibranch pipelines  

---

### Task 15 – Webhook Integration

**Objective**

Automate build triggers.

**What You Need to Do**

- Configure webhook in GitHub or GitLab  
- Trigger Jenkins job on code push  
- Optionally use Generic Webhook Trigger plugin  

**Deliverables**

- Automatic trigger proof  

**Interview Connect**

- Webhook vs polling  

---

## Phase 5 – Jenkins Advanced

### Task 16 – Master and Agent Setup

**Objective**

Distribute build workloads.

**What You Need to Do**

- Setup Jenkins agent  
- Connect to master  
- Execute jobs on agent  

**Deliverables**

- Agent execution logs  

**Interview Connect**

- Benefits of distributed builds  

---

### Task 17 – Docker as Jenkins Agent

**Objective**

Use container-based execution.

**What You Need to Do**

- Configure Docker agent  
- Run pipeline inside container  

**Deliverables**

- Pipeline logs from container  

**Interview Connect**

- Why ephemeral agents are preferred  

---

### Task 18 – Parameters and Variables

**Objective**

Make pipelines dynamic.

**What You Need to Do**

- Add build parameters  
- Use environment variables  
- Use credentials binding  

**Deliverables**

- Parameterized pipeline  

**Interview Connect**

- Why parameterized builds are used  

---

### Task 19 – Post Actions and Notifications

**Objective**

Handle pipeline outcomes.

**What You Need to Do**

- Add post actions for success and failure  
- Print logs or messages  
- Optional notification setup  

**Deliverables**

- Pipeline logs with post actions  

**Interview Connect**

- Importance of notifications in CI/CD  

---

### Task 20 – Jenkins Backup and Restore

**Objective**

Ensure Jenkins data safety.

**What You Need to Do**

- Write backup script  
- Backup job configurations  
- Restore in new Jenkins instance  

**Deliverables**

- Backup script  
- Restore validation  

**Interview Connect**

- Recovery strategies for Jenkins  

---

### Task 21 – Upstream and Downstream Jobs

**Objective**

Understand job dependencies.

**What You Need to Do**

Create two jobs:

- Build job  
- Deploy job  

Configure downstream trigger

**Deliverables**

- Chained job execution  

**Interview Connect**

- Pipeline vs job chaining  

---

### Task 22 – Jenkins Shared Library

**Objective**

Reuse pipeline code across projects.

**What You Need to Do**

- Create shared library repository  
- Add reusable functions in vars/  
- Use shared library in pipeline  

**Deliverables**

- Shared library repo  
- Pipeline using library  

**Interview Connect**

- Why shared libraries are important  

---

## Phase 6 – Security and Production Setup

### Task 23 – RBAC in Jenkins

**Objective**

Control access.

**What You Need to Do**

- Install role-based plugin  
- Create roles and assign users  

**Deliverables**

- Role configuration proof  

**Interview Connect**

- Importance of RBAC  

---

### Task 24 – LDAP Integration (Documentation)

**Objective**

Understand enterprise authentication.

**What You Need to Do**

Create `jenkins-ldap.md`

Document setup and benefits

**Deliverables**

- Documentation file  

**Interview Connect**

- LDAP vs local authentication  

---

### Task 25 – HashiCorp Vault Integration

**Objective**

Secure secret management.

**What You Need to Do**

- Setup Vault  
- Integrate with Jenkins  
- Fetch secrets in pipeline  

**Deliverables**

- Pipeline using Vault  

**Interview Connect**

- Why Vault is preferred  

---

### Task 26 – TLS and Domain Setup

**Objective**

Secure Jenkins access.

**What You Need to Do**

- Configure domain and DNS  
- Enable HTTPS using SSL  

**Deliverables**

- Secure Jenkins URL  

**Interview Connect**

- Why HTTPS is mandatory  

---

## Phase 7 – Troubleshooting and Real Scenarios

### Task 27 – Debugging Failed Pipelines

**Objective**

Handle real-world issues.

**What You Need to Do**

- Introduce failures intentionally  
- Debug and fix issues  

**Deliverables**

- Before and after logs  

**Interview Connect**

- Debugging strategies  

---

### Task 28 – Scenario-Based Pipeline Implementation

**Objective**

Simulate production workflow.

**What You Need to Do**

Build full pipeline:

- Build  
- Test  
- Deploy  
- Notify  

Use Jenkins or GitLab

**Deliverables**

- Complete working pipeline  

**Interview Connect**

- Designing CI/CD for real systems  

---

## Notes and Documentation

### Task 29 – Module Notes

**Objective**

Centralize learning.

**What You Need to Do**

Create `notes-module-5.md` including:

- GitHub Actions workflow  
- GitLab pipeline structure  
- Jenkins architecture  
- Common failures and fixes  

**Deliverables**

- Notes file  

**Interview Connect**

- Common CI/CD interview topics  

---

## Learn in Public

After completing this module:

- Push all pipelines to your repository  
- Write a summary of your learnings  
- Document failures and solutions  

Suggested tags:

#Batch13 #CICD #Jenkins #GitHubActions #GitLab #DevOps #LearnInPublic

---

## Outcome of Module 5

After completing this module, you will be able to:

- Build pipelines across GitHub Actions, GitLab, and Jenkins  
- Automate deployments using CI/CD  
- Use agents, runners, and containers for execution  
- Secure pipelines using RBAC and Vault  
- Troubleshoot pipeline failures  
- Design production-grade CI/CD workflows  