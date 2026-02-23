# 01-beginner-jenkins.md: The Fundamentals

## 🐣 Beginner's Corner: The "Butler" Analogy
Think of Jenkins as a very reliable, somewhat old-fashioned butler.
- **Master Node**: The Butler giving orders and keeping track of the schedule.
- **Agent/Worker Node**: The specialized staff doing the actual heavy lifting (baking, painting, building).
- **Freestyle Job**: A quickly written sticky note of instructions for the Butler.
- **Pipeline (Jenkinsfile)**: A formal, typed-out recipe for the Butler to follow step-by-step.
- **Plugins**: Different tools you give to the Butler so he can interact with new things (like giving him a megaphone to talk to Slack, or keys to AWS).

---

## 1. Core Concepts

### Q1: What is Jenkins?
- **Answer**: An open-source automation server written in Java. It is widely used for continuous integration and continuous delivery (CI/CD).
- **Pros**: Massive plugin ecosystem, highly customizable, free.
- **Cons**: Configuration can drift ("snowflake servers"), older UI, managing the master node can be painful.

### Q2: Master vs Agent Architecture?
- **Master (Controller)**: Handles scheduling, dispatching builds to agents, recording results, and hosting the UI. *You should not run heavy builds on the Master.*
- **Agent (Node)**: A machine (VM, Docker container, physical server) that connects to the Master and executes the jobs.

### Q3: Freestyle Project vs Pipeline?
- **Freestyle**: Configured entirely via the UI. Easy to start, but hard to version control and audit. Changes are lost if not backed up.
- **Pipeline (Jenkinsfile)**: "Pipeline-as-Code". The CI/CD steps are written in a file (usually `Jenkinsfile`) stored in the source code repository. This allows version control, code review, and auditability.

### Q4: What are Plugins?
- **Answer**: Extensions that add functionality to Jenkins. Since Jenkins is just a runner, it relies on plugins to integrate with Git, Docker, Kubernetes, AWS, Slack, etc.

---

## 2. Basic Configuration

### Q5: What is a `Jenkinsfile`?
- **Answer**: A text file that contains the definition of a Jenkins Pipeline. It uses Groovy syntax.
- **Two types**: Declarative (newer, stricter, easier to read) and Scripted (older, more flexible Groovy code).

### Q6: What does a basic Declarative Pipeline look like?
```groovy
pipeline {
    agent any // Run on any available Jenkins agent
    stages {
        stage('Build') {
            steps {
                echo 'Building...'
                sh 'make build'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
                sh 'make test'
            }
        }
    }
}
```

### Q7: How do you trigger a Jenkins build?
- **Source Code Management (SCM) Polling**: Jenkins checks Git periodically for changes (Not recommended, high load).
- **Webhooks**: GitHub/GitLab actively notifies Jenkins when a push happens (Recommended).
- **Schedule**: Cron syntax (e.g., nightly builds at 2 AM).
- **Manual**: Clicking "Build Now" in the UI.
