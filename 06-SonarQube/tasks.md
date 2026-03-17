# Module 6 – Tasks | SonarQube Deep Dive

This module simulates how teams enforce code quality and security in real environments.

You will:

- Setup SonarQube  
- Analyze multiple applications  
- Fix real issues  
- Integrate with CI/CD pipelines  
- Work with coverage and quality gates  

Complete tasks in sequence.

---

## Task 1 – SonarQube Setup

**Objective**

Install and run SonarQube.

**What You Need to Do**

- Run SonarQube using Docker or VM  
- Access UI in browser  
- Login and change default credentials  

**Deliverables**

- SonarQube dashboard access proof  

**Interview Connect**

- What is SonarQube used for  

---

## Task 2 – Understand SonarQube UI and Concepts

**Objective**

Understand core components.

**What You Need to Do**

Explore:

- Projects  
- Issues  
- Quality Gates  
- Quality Profiles  

Create `sonarqube-basics.md` explaining:

- Bugs vs vulnerabilities vs code smells  

**Deliverables**

- sonarqube-basics.md  

**Interview Connect**

- What is technical debt  

---

## Task 3 – First Code Analysis (Manual)

**Objective**

Run first scan manually.

**What You Need to Do**

- Install Sonar Scanner  
- Create sample project  
- Run scan using sonar-project.properties  

**Deliverables**

- Scan logs  
- Project visible in SonarQube  

**Interview Connect**

- How SonarQube analyzes code  

---

## Task 4 – Java Project Analysis

**Objective**

Analyze Java project.

**What You Need to Do**

- Use Maven project  

Run:

mvn clean verify sonar:sonar

Check issues in dashboard

**Deliverables**

- Analysis report  

**Interview Connect**

- How Sonar integrates with Maven  

---

## Task 5 – Node.js Project Analysis

**Objective**

Analyze Node.js project.

**What You Need to Do**

- Use Node project  
- Run Sonar scanner  
- Analyze results  

**Deliverables**

- Node project report  

**Interview Connect**

- Why JS projects need linting + analysis  

---

## Task 6 – Python Project Analysis

**Objective**

Analyze Python project.

**What You Need to Do**

- Use Python project  
- Configure scanner  
- Run analysis  

**Deliverables**

- Python analysis report  

**Interview Connect**

- Static analysis vs runtime errors  

---

## Task 7 – Quality Gates Setup

**Objective**

Control code quality.

**What You Need to Do**

- Create custom quality gate  

Add conditions:

- Coverage threshold  
- No critical vulnerabilities  

Apply to project

**Deliverables**

- Quality gate configuration  

**Interview Connect**

- Why quality gates are important  

---

## Task 8 – Fix Code Issues

**Objective**

Improve code quality.

**What You Need to Do**

Identify:

- Bugs  
- Code smells  

Fix issues in code

Re-run analysis

**Deliverables**

- Before/after comparison  

**Interview Connect**

- How teams reduce technical debt  

---

## Task 9 – Code Coverage (Node.js with JEST)

**Objective**

Measure test coverage.

**What You Need to Do**

- Setup JEST  
- Generate coverage report  
- Integrate with SonarQube  

**Deliverables**

- Coverage report  

**Interview Connect**

- Why coverage is important  

---

## Task 10 – Code Coverage (Python)

**Objective**

Measure Python coverage.

**What You Need to Do**

- Use coverage tool  
- Generate report  
- Integrate with SonarQube  

**Deliverables**

- Coverage report  

**Interview Connect**

- Difference between coverage and quality  

---

## Task 11 – GitHub Actions Integration

**Objective**

Run Sonar in CI pipeline.

**What You Need to Do**

- Add Sonar scan step in workflow  
- Use secrets for token  
- Run pipeline  

**Deliverables**

- Pipeline execution with Sonar  

**Interview Connect**

- Why integrate Sonar in CI  

---

## Task 12 – GitLab CI Integration

**Objective**

Integrate Sonar with GitLab.

**What You Need to Do**

- Update .gitlab-ci.yml  
- Add Sonar scan stage  
- Verify results  

**Deliverables**

- GitLab pipeline proof  

**Interview Connect**

- CI vs quality gate enforcement  

---

## Task 13 – Quality Gate Failure Handling

**Objective**

Fail pipeline on bad code.

**What You Need to Do**

- Configure pipeline to fail if quality gate fails  
- Test with bad code  

**Deliverables**

- Failed pipeline proof  

**Interview Connect**

- Why blocking bad code is important  

---

## Task 14 – SSL Setup for SonarQube

**Objective**

Secure SonarQube.

**What You Need to Do**

- Configure HTTPS  
- Use reverse proxy (Nginx or similar)  
- Access SonarQube securely  

**Deliverables**

- HTTPS access proof  

**Interview Connect**

- Why SSL is required  

---

## Task 15 – Multi-Project Analysis

**Objective**

Handle multiple projects.

**What You Need to Do**

- Add multiple projects in SonarQube  
- Analyze each  
- Compare metrics  

**Deliverables**

- Multiple project dashboards  

**Interview Connect**

- How companies manage many services  

---

## Task 16 – Custom Quality Profiles

**Objective**

Customize analysis rules.

**What You Need to Do**

- Create custom quality profile  
- Enable/disable rules  
- Apply to project  

**Deliverables**

- Profile configuration  

**Interview Connect**

- Why default rules are not enough  

---

## Task 17 – Branch Analysis

**Objective**

Analyze feature branches.

**What You Need to Do**

- Run scan for different branches  
- Compare results  

**Deliverables**

- Branch analysis proof  

**Interview Connect**

- Why branch-level analysis matters  

---

## Task 18 – Real Project Analysis (Java)

**Objective**

Analyze real-world project.

**What You Need to Do**

- Use Java project  
- Run full analysis  
- Document issues  

**Deliverables**

- Report with findings  

**Interview Connect**

- What kind of issues are common in Java apps  

---

## Task 19 – Real Project Analysis (Node.js with JEST)

**Objective**

Combine analysis and coverage.

**What You Need to Do**

- Run tests with coverage  
- Run Sonar scan  
- Validate results  

**Deliverables**

- Combined report  

**Interview Connect**

- How testing improves code quality  

---

## Task 20 – Real Project Analysis (Python)

**Objective**

Analyze Python project deeply.

**What You Need to Do**

- Run analysis  
- Fix issues  
- Re-run scan  

**Deliverables**

- Improvement proof  

**Interview Connect**

- Code quality challenges in Python  

---

## Task 21 – Notes and Documentation

**Objective**

Document learnings.

**What You Need to Do**

Create `notes-module-6.md` including:

- SonarQube workflow  
- Quality gates  
- Coverage integration  
- Common issues and fixes  

**Deliverables**

- Notes file  

**Interview Connect**

- How do you explain SonarQube in interviews  

---

## Learn in Public

After completing this module:

- Push all analysis configs and reports  
- Share:  

What issues you found  

What you fixed  

Coverage improvements  

Suggested tags:

#Batch13 #SonarQube #CodeQuality #DevOps #StaticAnalysis #LearnInPublic

---

## Outcome of Tasks

After completing these tasks, you will be able to:

- Run static code analysis across multiple stacks  
- Integrate SonarQube into CI/CD pipelines  
- Enforce quality gates in real workflows  
- Improve code quality using actionable insights  
- Work with coverage reports and metrics  
- Build production-ready quality pipelines  