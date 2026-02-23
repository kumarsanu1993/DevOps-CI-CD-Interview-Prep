# 02-intermediate-jenkins.md: Best Practices & Integrations

## 📚 Intermediate Corner: Pipelines & Credentials
This level focuses on writing DRY (Don't Repeat Yourself) pipelines, handling secure data, and structuring projects effectively.

---

## 1. Pipeline Deep Dive

### Q1: Declarative vs Scripted Pipeline?
- **Declarative**: Uses strict blocks (`pipeline`, `agent`, `stages`). Easier to validate, provides pre-defined hooks (`post` actions). Recommended for 95% of use cases.
- **Scripted**: Uses the `node` block. It is a full Groovy script, allowing immense flexibility, `try/catch` logic, and complex loops. Harder to maintain.

### Q2: What is a Shared Library?
- **Answer**: A way to share Groovy code across multiple Jenkinsfiles in different repositories.
- **Why**: If you have 50 microservices that all build Java the exact same way, you write the logic once in a Shared Library and call it: `@Library('my-shared-library') _` followed by `buildJavaApp()`.

### Q3: How do you use Docker in Jenkins Pipelines?
- **Answer**: The Pipeline can spin up a specific Docker container to run steps inside it, ensuring a clean environment without installing tools on the Jenkins agent directly.
```groovy
pipeline {
    agent {
        docker { image 'maven:3.8.1-jdk-11' }
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package' // runs inside the container
            }
        }
    }
}
```

## 2. Security and Credentials

### Q4: How do you securely handle passwords and API keys?
- **Answer**: Using the Jenkins Credentials plugin. You store the secret in Jenkins (as Secret Text, Username/Password, etc.) and bind it to environment variables in the pipeline.
- **Example**:
```groovy
pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY = credentials('aws-access-key-id')
    }
    // ...
}
```
- **Never**: Hardcode secrets in the `Jenkinsfile` or print them to stdout (`echo`).

### Q5: What is Masking?
- **Answer**: Jenkins attempts to mask (replace with `****`) known credentials in the console output. However, it's not foolproof, so developers must still avoid echoing secrets.

## 3. Workflow Control

### Q6: How do you pause a pipeline for manual approval?
- **Answer**: Using the `input` step. Often used before deploying to Production.
```groovy
stage('Deploy to Prod') {
    steps {
        input message: 'Deploy to Production?', ok: 'Yes, Deploy'
        sh './deploy.sh production'
    }
}
```

### Q7: How do you handle failure notifications?
- **Answer**: By using the `post` block in a Declarative pipeline.
```groovy
post {
    failure {
        slackSend channel: '#alerts', message: "Build Failed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
    }
    success {
        echo 'It worked!'
    }
}
```
