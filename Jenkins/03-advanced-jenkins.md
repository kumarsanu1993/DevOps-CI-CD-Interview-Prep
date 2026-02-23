# 03-advanced-jenkins.md: Architecture and Containerization

## 🚀 Advanced Corner: High Availability & Ephemeral Agents
At this level, you manage Jenkins as an infrastructure component, not just a UI tool.

---

## 1. Jenkins Configuration as Code (JCasC)

### Q1: What is JCasC and why is it important?
- **Answer**: Jenkins Configuration as Code (JCasC) allows defining the entire Jenkins master configuration (plugins, security realms, credentials, system settings) in a YAML file.
- **Problem it solves**: The "Snowflake Server" problem where a Jenkins master is configured via UI clicks, making it impossible to reproduce if it crashes or needs an upgrade.
- **Benefit**: You can treat Jenkins infrastructure like application code. Store the YAML in Git, and deploy Jenkins via an automated pipeline.

## 2. Dynamic Agents with Kubernetes

### Q2: How do you scale Jenkins agents dynamically?
- **Answer**: By integrating Jenkins with a cloud provider (AWS EC2 plugin) or Kubernetes (Kubernetes plugin).
- **How it works with K8s**:
  1. A build is triggered on the master.
  2. The master communicates with the Kubernetes API to spin up a new Pod specifically for this build.
  3. The steps run inside this ephemeral Pod.
  4. Once finished, the Pod is terminated, saving cost and ensuring a clean state.

### Q3: What does a Jenkinsfile look like using K8s agents?
```groovy
pipeline {
    agent {
        kubernetes {
            yaml '''
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: maven
                image: maven:3-alpine
                command: ['cat']
                tty: true
            '''
        }
    }
    stages {
        stage('Build') {
            steps {
                container('maven') {
                    sh 'mvn version'
                }
            }
        }
    }
}
```

## 3. Advanced Job DSL

### Q4: What is the Job DSL Plugin?
- **Answer**: While JCasC configures the *server*, the Job DSL plugin programs the creation of *Jobs*.
- **Use Case**: Automatically scanning a GitHub organization and creating a Multibranch Pipeline job for every repository it finds, without human intervention. (Also commonly solved by the GitHub Organization Folder plugin).

### Q5: How do you handle Artifact Management?
- **Answer**: Jenkins is bad at storing large files long-term. You should push built artifacts (JARs, Docker Images) to an external artifact repository like Artifactory, Nexus, AWS S3, or a Docker Registry as parts of the pipeline, using Jenkins only as a passthrough.
