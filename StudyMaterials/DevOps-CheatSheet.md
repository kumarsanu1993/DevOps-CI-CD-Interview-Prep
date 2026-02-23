# Study Materials: DevOps & CI/CD Cheat Sheet

This is a generated cheat sheet providing a quick reference guide to CI/CD core concepts. It serves as study material for the interview repository.

## 🐙 GitHub Actions Reference

### Essential Workflow Triggers
- **push**: Triggered on pushing to a branch.
- **pull_request**: Triggered when a pull request is opened or updated.
- **schedule**: Uses cron syntax (e.g., `cron: '0 0 * * *'`).
- **workflow_dispatch**: Allows manual triggering from the UI.

### Common Context Variables
- `${{ github.repository }}`: The name of the repo (e.g., `octocat/Hello-World`).
- `${{ github.sha }}`: The commit SHA that triggered the workflow.
- `${{ github.ref }}`: Defines the branch or tag ref.
- `${{ secrets.MY_SECRET }}`: Accessing an encrypted secret.

---

## 🦊 GitLab CI/CD Reference

### Core YAML Keywords
- **stages**: Defines the global order of execution (build, test, deploy).
- **image**: The Docker image used to execute the job.
- **script**: The actual shell commands executing the workload.
- **artifacts**: Defines files to save and share between stages (e.g., paths).
- **cache**: Stores dependencies globally to speed up pipelines.

### Environment Variables
- `$CI_COMMIT_REF_NAME`: Branch or tag name.
- `$CI_JOB_TOKEN`: A token used for authentication with GitLab API during a job.
- `$CI_PIPELINE_ID`: Unique ID of the current pipeline.

---

## 👨‍🍳 Jenkins Reference

### Jenkinsfile (Declarative) Structure
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Building...'
            }
        }
    }
}
```

### Essential Concepts
- **JCasC**: Jenkins Configuration as Code (configuring Jenkins master via YAML).
- **Shared Libraries**: Reusable Groovy code loaded via `@Library`.
- **RBAC**: Role-Based Access Control mapped via external IAM (Active Directory).
- **Agents (Nodes)**: The worker instances that actually execute the builds.

This document serves as an aggregate quick-reference guide. For deep dives, refer to the Expert-Level `100 Questions` modules in each respective folder.
