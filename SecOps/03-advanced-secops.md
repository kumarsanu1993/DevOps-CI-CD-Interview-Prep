# 03-advanced-secops.md: Cloud Security & Architecture

## 🚀 Advanced Corner: Zero Trust & Posture Management
At this level, you focus on securing immense cloud environments, ensuring continuous compliance, and architecting systems assuming the network is already hostile.

---

## 1. Zero Trust Architecture

### Q1: What is Zero Trust?
- **Answer**: A security framework that discards the old "Castle and Moat" idea (where everything inside the VPN is trusted). Zero Trust dictates: "Never trust, always verify."
- **Implementation**: Every single API call, database query, or user action requires independent authentication and authorization, regardless of whether it originates from inside or outside the corporate network.

### Q2: How does mTLS (Mutual TLS) fit into Zero Trust?
- **Answer**: In a service mesh (like Istio), mTLS ensures that not only does the client verify the server's certificate (standard TLS), but the server also verifies the client's certificate. This guarantees that `Microservice A` securely identifying itself to `Microservice B` cannot be spoofed.

## 2. Cloud Security Posture Management (CSPM)

### Q3: What is CSPM?
- **Answer**: CSPM tools (like AWS Security Hub or Prisma Cloud) continuously monitor your cloud infrastructure (AWS/GCP/Azure) to ensure it complies with security policies.
- **Use Case**: Instantly altering or alerting if a DevOps engineer accidentally makes an S3 bucket publicly readable, or opens port 22 (SSH) to the entire world (`0.0.0.0/0`).

### Q4: Explain CWPP (Cloud Workload Protection Platform).
- **Answer**: While CSPM looks at the *configuration* of the cloud (the AWS APIs), CWPP looks at the *internals* of the workloads (the VMs, containers, and serverless functions), protecting them from runtime threats, malware, and unauthorized execution.

## 3. Incident Response & Operations

### Q5: What is SIEM (Security Information and Event Management)?
- **Answer**: Systems (like Splunk, Datadog SIEM, or ELK) that aggregate logs from every server, firewall, and application in the company, applying correlation rules to detect anomalies (e.g., "A user logged in from New York and China within 5 minutes of each other").

### Q6: Explain the difference between Black-box, White-box, and Gray-box penetration testing.
- **Black-box**: Penetration testers are given zero prior knowledge of the internal architecture. They attack from the outside like a standard hacker.
- **White-box**: Testers are given full source code, architecture diagrams, and admin credentials to find hidden logical flaws.
- **Gray-box**: A hybrid approach; testers have user-level credentials and partial environmental knowledge.

### Q7: What is DAST vs IAST?
- **Interactive Application Security Testing (IAST)**: Unlike DAST which guesses from the outside, IAST instruments the application from the *inside* (using an agent attached to the JVM/Node process). When a test hits an endpoint, the IAST agent watches the data flow perfectly through the code to the database, achieving massive accuracy with zero false positives.
