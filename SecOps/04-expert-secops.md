# 04-expert-secops.md: Governance, Threat Modeling & Supply Chain

## 👑 Expert Corner: Enterprise Risk & Secure Supply Chains
At the Expert/Principal level, you are not just configuring security tools; you are building organizational frameworks, managing executive risk, and defending against state-sponsored supply chain vectors.

---

## 1. Supply Chain Security

### Q1: What is SLSA (Supply-chain Levels for Software Artifacts)?
- **Answer**: A formalized framework created by Google to prevent tampering, improve integrity, and secure packages flowing through the CI/CD pipeline.
- **Levels**: Ranging from Level 1 (scripted build) to Level 4 (hermetic, reproducible builds, two-person reviews, and non-bypassable provenance signatures).
- **Goal**: Defeating attacks like the SolarWinds hack, where attackers injected malware into the build server itself.

### Q2: What is an SBOM (Software Bill of Materials)?
- **Answer**: A formal, machine-readable inventory (using formats like SPDX or CycloneDX) detailing exactly every library, framework, and transitive dependency compiled into software. It is critical for enterprise compliance and immediate impact analysis during massive zero-day outbreaks.

## 2. Threat Modeling & Risk

### Q3: How do you conduct Threat Modeling?
- **Answer**: Analyzing an architecture diagram *before* code is written to identify structural flaws.
- **Frameworks**:
  - **STRIDE**: Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege.
  - **DREAD**: Damage, Reproducibility, Exploitability, Affected users, Discoverability.

### Q4: Explain the concept of "Blast Radius."
- **Answer**: The measurement of how far collateral damage extends if a specific component is compromised.
- **Expert-level design**: Segmenting networks, isolating AWS accounts using AWS Organizations, and utilizing strict IAM boundaries so that if a frontend web server is compromised, the blast radius does not reach the internal financial database.

## 3. Cryptography & Secrets Architecture at Scale

### Q5: How does Dynamic Secrets generation work in HashiCorp Vault?
- **Answer**: Instead of permanently storing a database password in Vault, Vault is granted root access to the DB. When a microservice requests access, Vault acts as a broker, *creating a brand new, unique SQL user/password* on the fly, handing it to the app, and automatically destroying the SQL user when the TTL (Time to Live) expires.
- **Benefit**: Zero permanent credentials to steal or rotate manually.

### Q6: What is a KMS Envelope Encryption Architecture?
- **Answer**: For encrypting massive amounts of data, you cannot rely entirely on a Cloud KMS (Key Management Service) due to API latency and cost.
- **Envelope pattern**: You request a Data Encryption Key (DEK) from KMS. You use the DEK locally to encrypt the massive database. You then ask KMS to encrypt the DEK itself using the Master Key (CMK), and you store the *encrypted DEK* alongside the encrypted data.

## 4. Governance & Chaos Engineering

### Q7: What is Security Chaos Engineering?
- **Answer**: Proactively injecting security failures (e.g., shutting down a firewall, altering an IAM role to overly permissive) in a staging or production environment to verify that SIEM alerts trigger accurately and that automated remediation scripts instantly fix the breach. 
