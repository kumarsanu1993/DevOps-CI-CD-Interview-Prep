# 01-beginner-owasp.md: The Fundamentals & Testing Basics

## 🐣 Beginner's Corner: The "Doctor's Exam" Analogy
Think of application security testing like diagnosing a patient.
- **SAST**: Reviewing the patient's DNA and family history before they even get sick (Checking the static code early).
- **DAST**: Poking the patient to see if it hurts without knowing what's inside (Testing the running app from the outside).
- **IAST**: Having a microscopic camera inside the patient while you test their reflexes (Monitoring internal code execution while simulating external attacks).

---

## 1. Core AppSec Testing Concepts & Examples

The user specifically requested examples for these critical testing methodologies:

### Q1: What is SAST (Static Application Security Testing)?
- **Answer**: It scans the raw source code or byte code *without running it*. It looks for known dangerous functions (e.g., using `MD5`, concatenating raw inputs into SQL).
- **Example Scenario**: A developer writes `String query = "SELECT * FROM users WHERE user_id = " + req.getParameter("id");`. A SAST tool parses this syntax tree, sees user input going directly into a SQL statement, and flags it as a "SQL Injection Risk" directly in the IDE.
- **Example Tools**: SonarQube, Checkmarx, Veracode, Fortify, Semgrep, GitHub Advanced Security (CodeQL).

### Q2: What is DAST (Dynamic Application Security Testing)?
- **Answer**: It behaves like an automated hacker. It communicates with a *running* web application via HTTP endpoints, crawling pages and submitting malicious payloads (like `<script>alert(1)</script>`) into forms to see if the server reflects it back or crashes.
- **Example Scenario**: The CI/CD pipeline deploys the app to a "Staging" environment. The DAST tool automatically crawls `https://staging.myapp.com/login`, inputs `' OR 1=1 --` into the password field, and checks if it successfully logs in.
- **Example Tools**: OWASP ZAP (Zed Attack Proxy), Burp Suite Enterprise, Acunetix, Qualys.

### Q3: What is IAST (Interactive Application Security Testing)?
- **Answer**: The hybrid evolution of SAST and DAST. You attach a monitoring agent to the running application (like a New Relic or Datadog agent). As regular tests (or DAST scans) run against the app, the IAST agent watches the data flow from the HTTP request, through the frameworks, down to the database natively.
- **Example Scenario**: A QA engineer runs an automated Cypress test script that logs into the app. The IAST agent mounted inside the JVM observes that the `username` field from the Cypress test was passed unmodified into a `java.sql.Statement.executeQuery()` function. It instantly flags an SQL injection flaw with 100% accuracy, showing exact line numbers, without requiring an actual hack to succeed.
- **Example Tools**: Contrast Security, Synopsys Seeker.

---

## 2. Introduction to OWASP

### Q4: What is OWASP?
- **Answer**: The Open Worldwide Application Security Project. It is a nonprofit foundation that works to improve the security of software.

### Q5: What is the OWASP Top 10?
- **Answer**: A globally recognized standard awareness document for developers and web application security. It represents a broad consensus about the most critical security risks to web applications.

### Q6: Can you list some of the classic OWASP Top 10 Vulnerabilities?
- **Answer**: Over the years, the categories evolve, but classics include:
  1. Broken Access Control
  2. Cryptographic Failures
  3. Injection (SQL, NoSQL, Command)
  4. Insecure Design
  5. Security Misconfiguration
  6. Vulnerable and Outdated Components
  7. Identification and Authentication Failures
  8. Software and Data Integrity Failures
  9. Security Logging and Monitoring Failures
  10. Server-Side Request Forgery (SSRF)
