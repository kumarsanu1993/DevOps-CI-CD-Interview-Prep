# 03-advanced-gitlab-ci-cd.md: Advanced Pipelines & Modularity

## 🚀 Advanced Corner: Complex Workflows
At this level, you build pipelines for mono-repos, use CI/CD templates, and trigger pipelines dynamically.

---

## 1. Modular Pipelines (`include`)

### Q1: How do you use the `include` keyword?
- **Answer**: It allows you to compose your `.gitlab-ci.yml` by including other YAML files. Great for DRYing out pipelines across an organization.
- **Methods**:
  - `include: local`: File in the same repo.
  - `include: file`: File from another GitLab repo.
  - `include: remote`: File from an external URL.
  - `include: template`: Built-in GitLab templates (e.g., Auto DevOps templates).

### Q2: What is the `extends` keyword?
- **Answer**: An elegant alternative to YAML anchors for reusing configuration within the same file.
```yaml
.deploy_template: # Hidden job (starts with a dot)
  stage: deploy
  script:
    - echo "Deploying to $ENV_NAME"

deploy_dev:
  extends: .deploy_template
  variables:
    ENV_NAME: "Development"
```

## 2. Multi-Project & Child Pipelines

### Q3: What is a Parent-Child Pipeline?
- **Answer**: A parent pipeline triggers a separate, child pipeline defined in a different YAML file. The child pipeline's status reflects back to the parent.
- **Use Case**: Mono-repos. If you have a backend and frontend in one repo, you can use `rules:changes` to only trigger the frontend child pipeline if frontend files changed.
```yaml
trigger_frontend:
  trigger:
    include: frontend/.gitlab-ci.yml
    strategy: depend # Wait for child to finish
  rules:
    - changes:
      - frontend/**/*
```

### Q4: What is a Multi-Project Pipeline?
- **Answer**: Triggering a pipeline in an entirely *different repository*.
```yaml
deploy_infrastructure:
  trigger: my-group/terraform-infra
```

## 3. Dynamic Pipelines

### Q5: What is a Dynamic Child Pipeline?
- **Answer**: When the parent pipeline runs a script that *generates* a `.gitlab-ci.yml` file, and then triggers it as a localized child pipeline.
- **Use case**: You have 50 dynamically generated deployment targets, and you generate the exact YAML configuration needed on the fly using a Python or Bash script in the first stage.
