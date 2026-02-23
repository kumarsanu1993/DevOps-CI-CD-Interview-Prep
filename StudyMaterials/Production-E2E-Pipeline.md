# End-to-End Production CI/CD Pipeline Architecture

This document outlines a rigorous, production-grade DevSecOps CI/CD pipeline deploying a containerized application to Amazon EKS. It encompasses the exact stages necessary for MAANG-level compliance, including exhaustive security scans and artifact provenance.

## 🔗 Accompanying Code Implementations
The theoretical stages listed below are explicitly implemented in the following technology-specific files within this repository:
- **GitHub Actions**: [06-e2e-pipeline.yml](../GitHub%20Actions/06-e2e-pipeline.yml)
- **GitLab CI/CD**: [06-e2e-pipeline.yml](../GitLab%20CI-CD/06-e2e-pipeline.yml)
- **Jenkins**: [06-e2e-Jenkinsfile](../Jenkins/06-e2e-Jenkinsfile)
- **ArgoCD (GitOps Pull Execution)**: [06-e2e-argocd-application.yaml](../ArgoCD/06-e2e-argocd-application.yaml)
- **Spinnaker (Canary CD Execution)**: [06-e2e-spinnaker-pipeline.json](../Spinnaker/06-e2e-spinnaker-pipeline.json)

---

## 🏗️ The 10-Stage Production Pipeline

### 1. Code Checkout
- **Purpose**: Retrieve the source code from the Git repository.
- **Best Practice**: Perform a shallow clone (`fetch-depth: 1` in GitHub Actions) to save bandwidth and execution time. Ensure the checkout uses an explicitly scoped, ephemeral GitHub Token or SSH key rather than a broad personal access token (PAT).

### 2. Secret Scanning (GitLeaks)
- **Purpose**: Ensure no hardcoded credentials, API keys, or tokens accidentally made it into the codebase or git history.
- **Execution**: Run `gitleaks detect` across the commit diff. If high-entropy strings or known token formats (e.g., `AKIA...` for AWS) are found, the build fails instantly before any compilation occurs.

### 3. Static Application Security Testing (SAST)
- **Purpose**: Analyze the raw source code for logical security flaws (e.g., SQL Injection, XSS) using tools like SonarQube, Checkmarx, or Semgrep.
- **Execution**: The SAST tool parses the code into an Abstract Syntax Tree (AST) to track data flow. A Quality Gate is enforced; if Critical or High vulnerabilities are detected, the pipeline terminates.

### 4. Filesystem SCA Scanning (Trivy FS)
- **Purpose**: Evaluate the open-source dependencies (e.g., `package.json`, `pom.xml`, `requirements.txt`).
- **Execution**: Running `trivy fs .` analyzes the repository for vulnerable third-party libraries globally recognized in the NVD (National Vulnerability Database). This prevents Supply Chain attacks.

### 5. Application Build & Unit Testing
- **Purpose**: Compile the code and execute isolated logic tests.
- **Execution**: Run standard test runners (e.g., `npm run test`, `mvn test`, `pytest`). Code coverage (e.g., JaCoCo) is evaluated, and the build artifact (e.g., `.jar`, `dist/`) is generated.

### 6. Container Image Building
- **Purpose**: Package the application artifact and its runtime into an immutable Docker image.
- **Process**: Execute `docker build`.
- **Best Practice**: Use **Multi-Stage Builds** to ensure development tools (like compilers) are left out of the final image. Use **Distroless** or Alpine base images to drastically reduce the attack surface.

### 7. Container Image Scanning (Trivy Image)
- **Purpose**: Scan the newly compiled Docker image for OS-level vulnerabilities before it ever touches a registry.
- **Execution**: Run `trivy image myapp:latest`. If an outdated underlying OS packages (e.g., an old version of `curl` or `openssl`) contains a Critical CVE, the pipeline breaks and the image is dropped.

### 8. Push to Container Registry (Amazon ECR)
- **Purpose**: Securely store the vetted image.
- **Execution**: Authenticate via OIDC (OpenID Connect) to AWS STS to receive short-lived credentials. Tag the image with the Git commit SHA securely (e.g., `myapp:7a2f1c8`) and push to Amazon Elastic Container Registry (ECR).
- **Compliance**: Sign the image cryptographically using `Cosign` to guarantee provenance natively.

### 9. Infrastructure & Application Deployment (Amazon EKS)
- **Purpose**: Deploy the new image to the Kubernetes cluster.
- **Push Execution (GitHub/GitLab/Jenkins)**: Instruct the CI runner to run `kubectl set image` or apply a Helm upgrade directly. 
- **Pull Execution (ArgoCD)**: Instead of the CI pipeline pushing to the cluster, the CI pipeline strictly updates the Image Tag inside a Git Infrastructure Repository. ArgoCD runs natively inside the EKS cluster, detects the `git diff`, and actively pulls/syncs the Git state securely to the cluster (GitOps).
- **Complex Execution (Spinnaker)**: If native `kubectl` rolling updates aren't enough, Spinnaker is triggered (often by the ECR publish event). It implements advanced deployment architectures like Blue/Green (Red/Black) deployments, instantly shifting cluster traffic.

### 10. Post-Deployment Testing & Canary Rollbacks
- **Purpose**: Verify the application is functioning safely.
- **Smoke/Integration Validation**: Provide a quick internal HTTP ping to `/healthz` (Smoke Test). Afterwards, trigger a Postman or Cypress suite against the staging database validating backend logic.
- **Automated Rollbacks (Spinnaker+Kayenta)**: Advanced CD relies on Canary analysis. Spinnaker evaluates Prometheus/Datadog metrics (e.g., 500 Error rates, CPU Spikes). If the new image triggers anomalies in the first 10 minutes, Spinnaker natively and instantly aborts the deployment and rolls back the Pods automatically.

---

## 🧪 Testing the CI/CD Code (Pipeline-as-Code)

A true DevSecOps engineer doesn't just write pipelines; they explicitly test the YAML/Groovy code dictating the pipeline itself natively before it ever executes in production.

### 1. GitHub Actions Testing
- **Unit Testing**: Use **`act`** (a local tool for running GitHub Actions inside Docker). A developer can run `act push` on their laptop to verify the YAML syntax and job execution steps. Use `actionlint` specifically to catch strict YAML syntax errors, missing secrets, or invalid expressions before pushing.
- **Integration/Smoke Testing**: Create a "Test Repository" specifically for pipelines. Trigger the reusable workflow inside the test repo using dummy code, asserting that the jobs execute and produce the expected dummy artifact.

### 2. GitLab CI/CD Testing
- **Unit Testing**: Use the **GitLab CI Lint API** (`/api/v4/ci/lint`). You can pipe your `.gitlab-ci.yml` locally via `curl` into the GitLab API to explicitly validate the YAML grammar. Additionally, use `gitlab-runner exec docker <job_name>` to run a single job locally.
- **Integration Testing**: Utilize a `.gitlab-ci.yml` strictly in a dedicated testing branch that runs `include:` parsing your custom templates intelligently and executing them against a mock registry or artifact repository.

### 3. Jenkins Testing
- **Unit Testing**: Use **Jenkins Pipeline Unit** testing framework (a Java/Groovy library). It allows you to mock Jenkins steps natively (e.g., mocking `sh` or `git` commands) and assert that your Groovy logic (loops, if/else, parallel blocks) executes correctly. [Note: Mocking is essential here; write Groovy tests using Spock or JUnit to test the Jenkinsfile exactly identically to standard application code].
- **Smoke/Integration Testing**: Run a secondary Jenkins master natively. Use it specifically as a CI layer for your Shared Libraries. Trigger a job using the shared pipeline code to execute against a mock application repository, confirming the E2E sequence natively finishes reporting success to your testing frameworks.

---
### The Golden Rule
Any modification to the `Jenkinsfile`, `.github/workflows/`, or `.gitlab-ci.yml` must explicitly be treated as application code and subjected to explicit PR reviews, Unit Testing, and Security scanning natively. 
