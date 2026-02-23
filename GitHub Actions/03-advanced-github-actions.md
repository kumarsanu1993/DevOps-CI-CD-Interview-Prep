# 03-advanced-github-actions.md: Architecture and Security

## 🚀 Advanced Corner: Custom Actions & Security
At this level, you are building your own tools, managing complex deployment strategies, and securing your pipelines.

---

## 1. Custom Actions Development

### Q1: What types of Custom Actions can you build?
- **JavaScript Actions**: Run directly on the runner machine (Node.js). Fast and platform-independent (Linux, Mac, Windows).
- **Docker Actions**: Packaged in a Docker container. Ensures environment consistency but only runs on Linux runners.
- **Composite Actions**: Combining existing CLI tools or actions into a single reusable step.

### Q2: How do you handle inputs and outputs in Custom Actions?
- **Answer**: Defined in `action.yml`. Inputs are accessed via `core.getInput()` (in JS) or environment variables. Outputs are set via `core.setOutput()` or writing to the `$GITHUB_OUTPUT` file.

## 2. Identity and Access Management

### Q3: What is OIDC (OpenID Connect) in GitHub Actions?
- **Answer**: A secure way to authenticate with cloud providers (AWS, Azure, GCP) without storing long-lived credentials as secrets.
- **How it works**: The workflow requests a short-lived token from the cloud provider by passing an OIDC token proving it runs from a specific GitHub repo/branch.
- **Why**: Eliminates the risk of leaked static credentials.

## 3. Environments and Deployments

### Q4: How do GitHub Environments work?
- **Answer**: Environments (like "Production", "Staging") allow you to set specific protection rules and secrets.
- **Features**:
  - Required Reviewers: Manual approval before a job runs (e.g., before deploying to Prod).
  - Wait timers: Delay a job for X minutes.
  - Deployment Branches: Only let `main` branch trigger deployment.

### Q5: How do you handle concurrency in Actions?
- **Answer**: The `concurrency` block ensures only one workflow/job runs at a time.
- **Scenario**: If a user pushes 3 commits quickly, you can cancel the previous redundant builds and only deploy the latest commit by setting `concurrency: group-name` with `cancel-in-progress: true`.
