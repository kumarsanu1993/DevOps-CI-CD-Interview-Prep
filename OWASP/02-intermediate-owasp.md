# 02-intermediate-owasp.md: The Classics (Injection & XSS)

## 📚 Intermediate Corner: Understanding the Exploits
This level focuses on the mechanics of the most infamous vulnerabilities on the OWASP Top 10 and how developers must mitigate them.

---

## 1. Injection Vulnerabilities

### Q1: What is SQL Injection (SQLi)?
- **Answer**: Occurs when untrusted user input is passed directly into a database query without sanitization. An attacker can insert their own SQL commands to read, modify, or delete the entire database.
- **Example Vulnerable Code**: `query = "SELECT * FROM users WHERE name = '" + username + "'";`
- **Mitigation**: ALWAYS use Parameterized Queries (Prepared Statements) or an ORM (Object-Relational Mapper) like Prisma or Hibernate. 
- **Safe Code**: `db.query("SELECT * FROM users WHERE name = ?", [username]);`

### Q2: What is OS Command Injection?
- **Answer**: Occurs when an application passes unsafe user-supplied data to a system shell.
- **Example**: A ping utility app takes an IP address and runs `os.system("ping " + ip_address)`. An attacker enters `127.0.0.1; rm -rf /`.

### Q3: What is NoSQL Injection?
- **Answer**: Even without SQL, NoSQL databases (like MongoDB) can be injected if logic isn't sanitized. For example, passing JSON objects where strings are expected (e.g., passing `{"$ne": null}` as a username to bypass login checks).

## 2. Client-Side Attacks

### Q4: What is Cross-Site Scripting (XSS)?
- **Answer**: Occurs when an application includes untrusted data in a web page without proper validation or escaping via the DOM. It allows attackers to execute malicious scripts specifically in the victim’s browser to steal session cookies or deface the site.
- **Types**:
  - **Stored XSS**: Malicious script is permanently stored on the server (e.g., in a forum post).
  - **Reflected XSS**: Malicious script is bounced off a web server natively through a URL parameter (e.g., `?search=<script>...`).
  - **DOM-based XSS**: Malicious data modifies the DOM entirely within the client-side JavaScript executing natively.
- **Mitigation**: Encode all data organically before rendering it in the DOM. Use modern frameworks (React/Angular) which automatically encode HTML by default. Implement a Content Security Policy (CSP).

### Q5: What is Cross-Site Request Forgery (CSRF)?
- **Answer**: An attacker tricks a victim's browser into executing an unwanted action on a trusted site where the victim is already authenticated. (e.g., A victim clicks a malicious link that silently submits a hidden form transferring funds from their bank).
- **Mitigation**: 
  - Using Anti-CSRF Tokens (unique, unpredictable values required to submit state-changing forms).
  - Setting the `SameSite` cookie attribute natively (`SameSite=Lax` or `SameSite=Strict`).

## 3. Broken Access Control

### Q6: What is IDOR (Insecure Direct Object Reference)?
- **Answer**: A critical form of Broken Access Control. Occurs when an application provides direct access to objects based purely on user-supplied input cleanly bypassing authorization checks.
- **Example**: User A authenticates and accesses their receipt at `api/receipts/1001`. They manually change the URL to `api/receipts/1002` and cleanly download User B's receipt because the server only verified they were logged in, not that they actually owned receipt 1002.
- **Mitigation**: Validate access controls at the data layer for every single request natively verifying the authenticated user explicitly owns the requested resource ID intrinsically.
