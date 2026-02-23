# 04-expert-jenkins.md: Enterprise Scale & Resilience

## 👑 Expert Corner: Scaling the Butler to a Global Logistics Network
At the Expert level, you are solving the fundamental architectural flaws of Jenkins (Single Point of Failure, JVM garbage collection pauses) for enterprise use.

---

## 1. Scaling the Jenkins Master

### Q1: Can you have active-active Jenkins Masters natively?
- **Answer**: No. Open-source Jenkins relies on the file system for storing job configurations, build logs, and state. It is fundamentally a monolithic architecture regarding the master.
- **CloudBees**: The enterprise version of Jenkins (CloudBees CI) offers High Availability (HA) and horizontal scalability (Operations Center managing client masters).
- **Open-Source Workaround**: Run lightweight, isolated masters per team (e.g., Team A gets master-A, Team B gets master-B) managed by JCasC and deployed on Kubernetes, rather than one giant monolithic master for 1,000 developers.

## 2. Resilience and Observability

### Q2: How do you handle Jenkins Master backups and disaster recovery?
- **Answer**: If using JCasC and Job DSL, you don't need to back up the configuration (it's in Git). You only need to back up the `$JENKINS_HOME` directory (specifically build logs and test history, if required for compliance).
- **Best Practice**: Treat the master as ephemeral. If it dies, a new one spins up from Git/JCasC, reconnects to the Git repos, and resumes.

### Q3: How do you monitor Jenkins health?
- **Answer**: Use the Prometheus metrics plugin to expose JVM metrics (Heap usage, GC pauses), Executor usage, and Queue length to Grafana.
- **Alerting**: Alert if the build queue length is constantly growing (indicating not enough agents) or if the Master JVM heap is near 100% (indicating potential OutOfMemory crashes).

## 3. Security Hardening

### Q4: How do you secure Jenkins at an enterprise level?
- **Answer**:
  - **Authentication**: Connect Jenkins to an Identity Provider (SSO via SAML/OIDC to Okta/Azure AD) rather than using the internal user database.
  - **Authorization**: Implement Role-Based Access Control (RBAC). Developers can build logs, Admins can manage plugins. (Often requires Matrix Authorization Strategy plugin).
  - **Agent Security**: Ensure agents cannot execute code on the master. 
  - **Audit Logging**: Ship all Jenkins logs (who clicked what, who changed what job) to a centralized logging system (ELK/Splunk) for compliance audits.
