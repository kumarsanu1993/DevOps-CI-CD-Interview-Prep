# 01-beginner-secops.md: The Fundamentals

## 🐣 Beginner's Corner: The "Airport Security" Analogy
Think of SecOps (Security Operations) and DevSecOps like an airport.
- **Shift Left**: Checking tickets and IDs *before* people get to the scanning machines (catching bad code early in the IDE).
- **SAST (Static Application Security Testing)**: The X-ray machine scanning luggage without opening it (analyzing code without running it).
- **DAST (Dynamic Application Security Testing)**: The security guard asking you questions or swab-testing your bag (testing the running application from the outside).
- **Firewall/WAF**: The border control checking passports (filtering incoming traffic).
- **IAM (Identity & Access Management)**: Your boarding pass deciding which gate you can enter.

---

## 1. Core Concepts

### Q1: What is DevSecOps?
- **Answer**: It stands for Development, Security, and Operations. It is the philosophy of integrating security practices within the DevOps process from the very beginning (shifting left), rather than bolting security on as an afterthought at the end.
- **Goal**: Make everyone responsible for security, automating checks to speed up secure delivery.

### Q2: What does "Shift Left" mean?
- **Answer**: Moving security testing earlier (to the "left") in the software development lifecycle (SDLC).
- **Why**: It is vastly cheaper and faster to fix a security bug discovered by a developer in their IDE than it is to fix a vulnerability found in production by an attacker.

### Q3: What is the CIA Triad?
- **Answer**: The foundational model of information security.
  - **Confidentiality**: Only authorized people can view the data (e.g., encryption).
  - **Integrity**: Data has not been tampered with (e.g., hashing/checksums).
  - **Availability**: Systems and data are accessible when needed (e.g., DDoS protection, backups).

---

## 2. Basic Security Testing

### Q4: SAST vs. DAST?
- **SAST (Static Analysis)**: "White-box" testing. It reads the raw source code looking for dangerous patterns (like SQL injection flaws or hardcoded passwords). Runs during the Build phase.
- **DAST (Dynamic Analysis)**: "Black-box" testing. It interacts with the compiled, running application, trying to aggressively hack it via web requests. Runs during the Test/Staging phase.

### Q5: What is SCA (Software Composition Analysis)?
- **Answer**: A tool that scans your `package.json`, `pom.xml`, or `requirements.txt` to see if you are using open-source libraries that have known vulnerabilities (CVEs) or licensing issues.
- **Example Tools**: Dependabot, Snyk, Black Duck.

### Q6: What is a CVE?
- **Answer**: Common Vulnerabilities and Exposures. It is a dictionary of publicly known information security vulnerabilities and exposures. Every known bug gets a unique ID (e.g., CVE-2021-44228 for Log4j).

---

## 3. Operations & Perimeters

### Q7: What is the Principle of Least Privilege (PoLP)?
- **Answer**: Providing a user, program, or process with only the bare minimum permissions necessary to perform its specific function, and nothing more.

### Q8: What is the difference between Authentication (AuthN) and Authorization (AuthZ)?
- **AuthN**: *Who are you?* (Logging in with a username/password or MFA).
- **AuthZ**: *What are you allowed to do?* (Checking if your user role has admin rights to delete a database).
