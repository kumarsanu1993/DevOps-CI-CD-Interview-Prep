# 01-beginner-github-actions.md: Fundamentals of GitHub Actions

## 🐣 Beginner's Corner: The "Assembly Line" Analogy
Think of GitHub Actions like a factory assembly line.
- **Workflow**: The entire factory floor for a specific product.
- **Job**: A specific workstation (e.g., painting station).
- **Step**: An individual worker doing a task at that station (e.g., applying red paint).
- **Action**: A pre-packaged tool the worker uses (e.g., a specific paint sprayer).
- **Runner**: The physical factory building where this happens (GitHub-hosted or self-hosted server).
- **Event**: The manager blowing the whistle to start the line (e.g., a push to `main` branch).

---

## 1. Core Concepts

### Q1: What is GitHub Actions?
- **Answer**: It's a CI/CD and automation platform native to GitHub. It allows you to automate your software development workflows directly in your repository.
- **Use Cases**: Running tests on pull requests, deploying code after a merge, publishing packages, automating issue triaging.

### Q2: Workflows vs Jobs vs Steps?
- **Workflow**: A configurable automated process made up of one or more jobs. Defined in a `.yml` file inside `.github/workflows/`.
- **Job**: A set of steps that execute on the SAME runner. By default, multiple jobs run *in parallel*.
- **Step**: An individual task that can either run commands (shell script) or an Action. Steps run in *sequence* and share the same filesystem within a job.

### Q3: What is a Runner?
- **GitHub-hosted Runners**: Virtual machines hosted by GitHub (Ubuntu, Windows, macOS). Clean state for every job.
- **Self-hosted Runners**: Your own machines (EC2, on-prem) registered with GitHub. Gives you control over hardware, OS, and network access (e.g., accessing internal databases).

### Q4: How do you trigger a Workflow (Events)?
- **Push**: Triggered when code is pushed to a branch (`on: push`).
- **Pull Request**: Triggered when a PR is opened, synchronized, or closed (`on: pull_request`).
- **Schedule**: Triggered at a specific time using cron syntax (`on: schedule`).
- **Manual**: Triggered manually from the UI or via API (`on: workflow_dispatch`).

---

## 2. Basic Configuration

### Q5: What does a basic workflow file look like?
```yaml
name: CI
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run a one-line script
        run: echo Hello, world!
```

### Q6: What is `actions/checkout`?
- **Answer**: It is a standard GitHub Action that checks out your repository under `$GITHUB_WORKSPACE`, so your workflow can access the code. It is usually the first step in a build/test job.

### Q7: How do you pass data between steps?
- **Answer**: Usually via the filesystem or using specific output parameters (e.g., `echo "MY_VAR=value" >> $GITHUB_ENV` or `>> $GITHUB_OUTPUT`).

### Q8: What are Environment Variables in Actions?
- **Answer**: GitHub provides default variables (like `GITHUB_SHA`, `GITHUB_REF`). You can also set custom environment variables at the workflow, job, or step level using the `env` keyword.
