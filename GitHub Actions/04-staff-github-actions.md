# 04-staff-github-actions.md: Enterprise Scale & DevSecOps

## 👑 Staff Corner: Governance, Scaling, and Hardening
At the Staff/Principal level, you are designing CI/CD systems for hundreds of developers across thousands of repositories.

---

## 1. Enterprise Infrastructure (Runners)

### Q1: How do you scale Self-Hosted Runners for 1,000+ developers?
- **Answer**: You cannot use static EC2 instances. You must use **Ephemeral Runners** combined with Autoscaling.
- **Tools**: Actions Runner Controller (ARC) on Kubernetes.
- **Pattern**: When a workflow is queued, a webhook triggers ARC to spin up a new K8s pod with the runner agent. Once the job finishes, the pod is destroyed (ephemeral).
- **Why Ephemeral**: Security (clean slate, no cross-contamination between jobs), Cost (scale to zero).

### Q2: What are the security risks of Self-Hosted Runners on public repos?
- **Answer**: Extremely high risk. A malicious PR can execute code on your internal network.
- **Mitigation**: Never use self-hosted runners for public repositories without heavy isolation. Always mandate approval for first-time contributors before CI runs.

## 2. Governance and Reusability at Scale

### Q3: How do you enforce compliance pipelines across an Enterprise?
- **Answer**: Using **Required Workflows** (via Organization settings) or **Reusable Workflows with centralized governance**.
- **Strategy**: Define standard CI templates in a centralized `.github` organization repo. All child repos must call these templates. Use GitHub API/tools to audit repo configurations.

### Q4: How do you manage Dependabot integration with Actions?
- **Answer**: Dependabot creates automated PRs for dependency updates.
- **Automation**: Use an Action to auto-approve and auto-merge Dependabot PRs for minor/patch versions if all CI checks pass. Ensure secrets are not automatically exposed to Dependabot PRs (using specific Dependabot secrets).

## 3. Pipeline Security Hardening

### Q5: What is Supply Chain Security in the context of Actions?
- **Answer**: Protecting the tools and dependencies used *in* your CI/CD.
- **Practices**:
  - Pinning Actions to strict **commit SHAs** instead of mutable tags (`@v2`, which can change).
  - Using `dependabot` to keep Action versions updated.
  - Signing provenance data for artifacts (SLSA framework) to prove the binary was built by a specific workflow.
  - Least Privilege: Ensuring `GITHUB_TOKEN` permissions are restricted to `contents: read` by default.
