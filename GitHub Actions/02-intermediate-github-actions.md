# 02-intermediate-github-actions.md: Beyond the Basics

## 📚 Intermediate Corner: Caching & Reusability
As your workflows grow, they can become slow and repetitive. This section focuses on making pipelines faster and more modular.

---

## 1. Secrets and Variables

### Q1: How do you handle Secrets?
- **Answer**: By defining Secrets in the Repository Settings. They are encrypted and passed into workflows via `${{ secrets.MY_SECRET }}`.
- **Scope**: Organization, Repository, or Environment level.
- **Limits**: GitHub redacts secrets from logs, but you must be careful not to output them to files or URLs.

### Q2: What are Configuration Variables?
- **Answer**: Non-sensitive configuration data (e.g., `APP_ENV`, `INFRA_REGION`). Defined in repo settings and accessed via `${{ vars.MY_VAR }}`.

## 2. Speed and Optimization

### Q3: What is caching in GitHub Actions and why is it used?
- **Answer**: `actions/cache` stores dependencies (like `node_modules`, `.m2` for Maven) between workflow runs.
- **Why**: Drastically reduces build times. Instead of downloading MB/GB of dependencies every run, it fetches them from the cache based on a key (often a hash of lockfiles like `package-lock.json`).

### Q4: Explain the Matrix Build strategy.
- **Answer**: Matrix allows you to run a single job configuration multiple times with different variables.
- **Example**: Testing a Node.js app on `v14`, `v16`, and `v18` simultaneously.
```yaml
strategy:
  matrix:
    node-version: [14.x, 16.x, 18.x]
```

## 3. Modularity

### Q5: What are Reusable Workflows?
- **Answer**: Workflows that can be called by other workflows (`uses: docs/my-workflow.yml@main`).
- **Why**: Keeps code DRY (Don't Repeat Yourself). E.g., a standard "Deploy to AWS" workflow reused by 10 different repos.

### Q6: What are Composite Actions?
- **Answer**: Similar to reusable workflows, but defined as a single Action. It bundles multiple *steps* into one step.
- **Difference**: Reusable workflows call full *workflows* (jobs/events), while Composite Actions bundle *steps* within a job.

### Q7: How do you pass artifacts between jobs?
- **Answer**: Since jobs run on different runners, you must use `actions/upload-artifact` in the first job and `actions/download-artifact` in the second job to transfer files (e.g., compiled binaries, test reports).
