# 02-intermediate-gitlab-ci-cd.md: Artifacts, Caching & Rules

## 📚 Intermediate Corner: Data Passing & Conditional Execution
This level focuses on passing data between jobs, speeding up pipelines, and controlling exactly when jobs should run.

---

## 1. Handling Data (Artifacts & Cache)

### Q1: Cache vs Artifacts? What is the difference?
- **Cache**: Used to speed up subsequent pipeline runs by storing downloaded dependencies (like `node_modules`). NOT guaranteed to exist on the next run. Used *between pipeline runs*.
- **Artifacts**: Used to pass intermediate build results (like a compiled binary or test report) from one stage to a later stage. Guaranteed to exist and can be downloaded from the UI. Used *between stages of the same pipeline*.

### Q2: How do you configure an Artifact?
- **Answer**: Use the `artifacts` keyword, specifying `paths` and `expire_in`.
```yaml
build_job:
  stage: build
  script:
    - make build
  artifacts:
    paths:
      - bin/
    expire_in: 1 week
```
- By default, all subsequent jobs download the artifacts from previous stages.

### Q3: How do you configure global caching?
```yaml
cache:
  key: ${CI_COMMIT_REF_SLUG} # Cache per branch
  paths:
    - node_modules/
```

## 2. Controlling Job Execution

### Q4: How do you run jobs conditionally?
- **Answer**: Historically using `only` / `except`, but modern pipelines use the much more powerful `rules` keyword.

### Q5: How do the `rules` keyword work?
- **Answer**: It evaluates an array of conditions. The job runs according to the first matching rule.
```yaml
deploy_prod:
  stage: deploy
  script:
    - ./deploy.sh
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
      when: manual # Requires a human click
    - if: $CI_COMMIT_BRANCH != "main"
      when: never # Don't add job to pipeline
```

### Q6: What is `allow_failure`?
- **Answer**: The job can fail without stopping the progression of the pipeline. Useful for experimental tests or non-critical linting.
```yaml
lint_code:
  script: npm run lint
  allow_failure: true
```

## 3. Environments

### Q7: What are GitLab Environments?
- **Answer**: Logical targets (like "staging" or "production") that group deployments.
- **Why**: Allows you to track what commit is deployed where, view deployment history, and handle approvals directly in the GitLab UI.
```yaml
deploy_job:
  stage: deploy
  script:
    - ./deploy-to-prod.sh
  environment:
    name: production
    url: https://my-app.com
```
