# 02-intermediate-secops.md: Hardening & Container Security

## 📚 Intermediate Corner: Building Secure Infrastructure
At this level, you move beyond checking code syntax and start securing the environments where the code runs: Containers, Secrets, and Network boundaries.

---

## 1. Secrets Management

### Q1: Why shouldn't you commit secrets to Git, and how do you fix it if you do?
- **Why**: Git history is immutable by default. Any developer who clones the repo has the password forever.
- **How to fix**: You cannot just delete the file and commit again (it remains in the git history). You must use tools like `git filter-repo` or BFG Repo-Cleaner to rewrite history, and *immediately rotate/revoke the compromised credential*.

### Q2: How do Secret Managers (like HashiCorp Vault or AWS Secrets Manager) work?
- **Answer**: They act as a central, highly secure digital vault. Applications authenticate to the Vault at runtime to request the secret dynamically, injecting it into memory (`/dev/shm` or env variables) without ever writing it to a physical configuration file.

## 2. Container & Image Security

### Q3: How do you secure a Dockerfile?
- **Non-Root User**: Never run the application as `root` (e.g., use `USER 1001`).
- **Minimal Base Images**: Use Alpine or Distroless images to reduce the attack surface.
- **Multi-Stage Builds**: Compile the code in an early stage, and only copy the compiled binary to the final image (leaving the compiler and source code behind).

### Q4: What is Container Image Scanning?
- **Answer**: Utilizing tools (like Trivy or Clair) to unpack a Docker image before it reaches a registry and mapping every installed OS package (e.g., `apt-get install curl`) against a database of CVEs.

## 3. Network Security & Cryptography

### Q5: What is the difference between Hashing and Encryption?
- **Encryption**: Two-way math. You lock data with a key, and it can be unlocked with a key (e.g., AES-256). Used for data at rest.
- **Hashing**: One-way math. You pass data into a function to get a fixed-length string (e.g., SHA-256). You cannot reverse a hash perfectly. Used for storing passwords safely.

### Q6: What is a WAF (Web Application Firewall)?
- **Answer**: A firewall specifically sitting in front of web applications (Layer 7) inspecting inbound HTTP/HTTPS traffic. It blocks common web attacks identified by the OWASP Top 10 (like SQLi, Cross-Site Scripting, and botnets).

### Q7: Explain Data at Rest vs. Data in Transit.
- **Data at Rest**: Data sitting on a hard drive/database. Secured by Disk Encryption (AES-encryption, AWS KMS).
- **Data in Transit**: Data moving across the internet. Secured by TLS (SSL/HTTPS certificates).
- **Data in Use**: Data currently in RAM/CPU processing. Secured by Confidential Computing (Hardware enclaves like AWS Nitro).
