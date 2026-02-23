# 01-beginner-gitlab-ci-cd.md: The Fundamentals

## 🐣 Beginner's Corner: The "Scripted Sequence" Analogy
Think of GitLab CI/CD as a highly organized sequence of scripts running in isolated boxes.
- **Pipeline**: The whole process from code push to deployment.
- **Stage**: A phase in the pipeline (e.g., Build, Test, Deploy). Like an act in a play.
- **Job**: A specific task within a stage (e.g., Unit Test, Linting). They run in parallel inside a stage.
- **Runner**: The isolated box (VM or Container) that executes the Job's script.
- **`.gitlab-ci.yml`**: The script that defines the entire show.

---

## 1. Core Concepts

### Q1: What is GitLab CI/CD?
- **Answer**: It is a tool built into GitLab for software development through Continuous Integration (CI), Continuous Delivery (CD), and Continuous Deployment (CD). You don't need a separate server like Jenkins.

### Q2: How does a GitLab Pipeline structure work?
- **Answer**: It uses a YAML file `.gitlab-ci.yml` at the root of the project.
- **Hierarchy**: Pipeline > Stages (run sequentially) > Jobs (run in parallel within a stage).
  - If Stage 1 (Build) fails, Stage 2 (Test) will not start.

### Q3: What is a GitLab Runner?
- **Answer**: A lightweight, highly scalable agent (written in Go) that runs the jobs defined in the `.gitlab-ci.yml` file and sends the results back to GitLab.
- **Types**: Shared Runners (provided by GitLab), Specific/Group Runners (hosted by you on your own servers or Kubernetes clusters).

### Q4: What are the primary ways to run a Job?
- **Shell Executor**: Runs tasks directly on the host machine.
- **Docker Executor**: Runs tasks inside a fresh Docker container for every job (Most common and recommended for clean states).

---

## 2. Basic Configuration

### Q5: What does a basic `.gitlab-ci.yml` look like?
```yaml
stages:
  - build
  - test

build_job:
  stage: build
  script:
    - echo "Building the app..."
    - npm install

test_job:
  stage: test
  script:
    - echo "Testing the app..."
    - npm test
```

### Q6: How do you use a specific Docker image for a job?
- **Answer**: Provide the `image` keyword at the global or job level.
```yaml
test_job:
  image: node:16
  stage: test
  script:
    - npm test
```

### Q7: Variables in GitLab CI/CD?
- **Answer**: You can define custom variables in the YAML file or in the GitLab UI (Project > Settings > CI/CD > Variables). UI variables can be Masked (hidden in logs) or Protected (only available on protected branches/tags).
