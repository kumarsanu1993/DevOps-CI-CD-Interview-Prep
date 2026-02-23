# 05-top-100-owasp-questions.md: MAANG Standard AppSec Interview Prep

This document contains 100 rigorous, expert-level AppSec questions formatted specifically for MAANG (Meta, Amazon, Apple, Netflix, Google) security engineering and DevSecOps interviews.

## SAST, DAST, IAST & CI/CD Integrations (1-20)

**1. [SAST] How do you write a custom Semgrep rule to detect insecure random number generation in Java?**
**Answer**: Target the explicit instantiation. Rule pattern: `new java.util.Random(...)`. Hook this into GitHub Actions as a PR blocker, enforcing `java.security.SecureRandom`.

**2. [DAST] Why might a standard DAST scan completely miss vulnerabilities in a modern React SPA?**
**Answer**: SPAs use client-side routing and JSON APIs. Standard DAST crawlers fail to populate complex state. Use an OpenAPI/Swagger spec to instruct the DAST tool how to construct JSON payloads.

**3. [IAST] How does IAST prove RCE via Deserialization without exploiting the server?**
**Answer**: IAST instruments the JVM. It watches untrusted data flow from an HTTP controller to `ObjectInputStream.readObject()`. If an attacker passes a Gadget Chain, IAST flags the execution path without successful exploitation.

**4. How do you scale SAST scanning in a massive Monorepo without impacting PR merge times?**
**Answer**: Implement Incremental Scanning. Configure the SAST engine to evaluate strictly the `git diff` boundaries mapping to the PR, returning results in seconds.

**5. What is an AST (Abstract Syntax Tree) in relation to SAST?**
**Answer**: SAST parses code into an AST—a tree representing the semantic structure of the code—allowing accurate Data Flow Analysis rather than simple string matching (regex).

**6. Architecturally, what defines explicit Data Flow Analysis (Taint Tracking)?**
**Answer**: Tracking arbitrary user input (Taint Source) parsing through application logic, ensuring it passes through a proper sanitization function (Cleanser) before reaching a dangerous execution block (Sink).

**7. Why does DAST inherently generate False Negatives when measuring OS Command Injection?**
**Answer**: In a Blind Injection, the server executes the command but returns a standard `200 OK` with no visual output. DAST must use Out-of-Band (OOB) interactions (e.g., DNS queries) to confirm background execution.

**8. What limits IAST structurally?**
**Answer**: Language support. Because it instruments byte code, it inherently relies on explicit agents designed strictly for Java, .NET, Node, Go, or Python.

**9. How do you implement AppSec Gates analyzing DevOps pipelines intelligently?**
**Answer**: Establish Quality Gates validating explicit SAST/SCA scores, failing pipelines dynamically when High/Critical thresholds are breached, isolating vulnerabilities before deployment.

**10. What is a "Dependency Confusion" attack and how do you protect corporate artifact registries?**
**Answer**: Attackers upload identically named packages to the public NPM registry with higher version numbers than internal, private packages. Mitigate by configuring internal Artifact Managers (Artifactory) to strictly prioritize local private repositories.

**11. When evaluating open-source packages, how do you handle Transitive Dependencies securely?**
**Answer**: SCA must execute `npm audit` or parse lockfiles, calculating nested dependency paths to flag the parent package that requires updating.

**12. How does SCA differ from traditional SAST?**
**Answer**: SAST analyzes proprietary source code for logical flaws. SCA maps explicit third-party dependencies against the NVD (National Vulnerability Database) to identify known CVEs.

**(Questions 13-20: Advanced CI/CD AppSec Architectures)**
*   **13.** Use distroless base images to reduce the attack surface for containerized AppSec tooling.
*   **14.** Utilize eBPF (like Falco) to monitor runtime security post-deployment.
*   **15.** Implement SLSA Level 4 for hermetic, reproducible builds.
*   **16.** Store SBOMs centrally in OWASP Dependency-Track.
*   **17.** Rotate pipeline secrets dynamically using HashiCorp Vault.
*   **18.** Enforce branch protections requiring 2-person code reviews for all CI modifications.
*   **19.** Sign container images using Cosign to prevent supply chain tampering.
*   **20.** Aggregate SAST/DAST/SCA outputs via DefectDojo for deduplicated vulnerability routing.

## Injection & DOM Architectures (21-40)

**21. In a Node.js + PostgreSQL app, how do you perform SQLi when the developer used parameterized queries in an `ORDER BY` clause?**
**Answer**: Parameterized queries do not inherently parameterize SQL Identifiers (like Column names). Passing user input to `ORDER BY` breaks parameterization. Use a strict hardcoded whitelist for sort variables.

**22. Explain Second-Order SQL Injection.**
**Answer**: The payload successfully passes into the database safely (e.g., via an ORM). Later, a different architectural component (like a batch script) reads that DB field and concatenates it into a raw query without an ORM.

**23. How do you bypass a basic WAF specifically looking for `<script>` tags?**
**Answer**: Utilize implicit Event Handlers inside structural HTML elements (e.g., `<img src=x onerror=alert(init)>`).

**24. What makes DOM-based XSS substantially more difficult to detect via WAF?**
**Answer**: The payload sits in the URL Fragment `#`, which the browser never sends to the server. The client-side JS reads it and modifies the DOM natively. Server logs and WAFs never see it.

**25. How do you implement robust anti-XSS architectures natively in React?**
**Answer**: React implicitly encodes `{variables}`. However, using `dangerouslySetInnerHTML` bypasses this. If required, strictly run inputs through DOMPurify before setting them.

**26. Explain GraphQL Introspection Abuse.**
**Answer**: Enabling introspection in production allows an attacker to automatically download the entire API schema, mapping hidden administrative mutations perfectly.

**27. How does XML External Entity (XXE) manipulation extract AWS metadata?**
**Answer**: An attacker defines an external entity referencing `http://169.254.169.254/latest/meta-data/iam/security-credentials/`. The vulnerable XML parser inherently fetches the IAM STS tokens and echoes them back.

**28. How do you definitively prevent XXE in modern Java microservices?**
**Answer**: Completely and globally disable Document Type Definitions (DTDs) across all XML parsing configurations.

**29. What is an OS Command Injection payload that bypasses space restrictions?**
**Answer**: Using Internal Field Separators (`$IFS`), e.g., `cat$IFS/etc/passwd`.

**30. How does Prototype Pollution lead to Remote Code Execution (RCE) in Node.js?**
**Answer**: Attackers manipulate `__proto__` to inject arbitrary properties into the global Object prototype. If the application later uses an undefined property to construct a shell command (e.g., `child_process.exec()`), the polluted property takes over.

**(Questions 31-40: NoSQLi, LDAP Injection, SSRF)**
*   **31.** NoSQL Injection bypasses Auth by passing objects instead of strings: `{"password": {"$ne": null}}`.
*   **32.** Blind SSRF can be exploited using time delays or out-of-band DNS interactions.
*   **33.** SSRF can bypass internal network perimeters to access Redis/Memcached.
*   **34.** Mitigate SSRF by forcing IMDSv2 in AWS.
*   **35.** Mitigate SSRF by using strict allowable DNS whitelists.
*   **36.** LDAP Injection uses unescaped variables in Active Directory lookups.
*   **37.** Insecure Deserialization in Python heavily exploits the `pickle` module.
*   **38.** Java deserialization exploits often leverage the `Apache Commons Collections` library.
*   **39.** Stop Deserialization by using primitive data formats (JSON/Protobuf) instead of native Objects.
*   **40.** Cloud-native SSRF can also be used to query internal Kubernetes `etcd` configurations if unrestricted.

## Broken Authentication, Identity & Secrets (41-60)

**41. Why is session fixation a critical OWASP vulnerability?**
**Answer**: An attacker forces a victim to use an attacker-known session ID. When the victim logs in, the attacker hijacks that identical session. Mitigate by regenerating session IDs post-login.

**42. How does MFA bypass occur natively via IDOR?**
**Answer**: The MFA verification API (`/verify-mfa`) accepts an attacker-supplied `user_id` as a POST parameter instead of implicitly extracting the identity from the encrypted JWT context.

**43. Is storing JWTs in `localStorage` secure against XSS?**
**Answer**: No. Any XSS vulnerability can read `localStorage.getItem('token')` and exfiltrate it. Utilize `HttpOnly` combined `Secure` cookies.

**44. What defines a JSON Web Token (JWT) structurally?**
**Answer**: A cryptographically signed string using Base64Url encoding containing three segments: Header (algorithm), Payload (claims), and Signature (verification).

**45. Why is setting `alg: none` dangerous inside a JWT parser?**
**Answer**: Attackers strip the JWT signature and set the Algorithm header to `none`. Vulnerable backend libraries implicitly accepted the payload bypassing signature validation.

**46. Explain the explicit difference between OAuth 2.0 and SAML.**
**Answer**: SAML is an XML-based framework utilized for Enterprise SSO. OAuth 2.0 is an authorization framework using JSON/REST designed to delegate access via Bearer Tokens.

**47. Why is OpenID Connect (OIDC) necessary on top of OAuth 2.0?**
**Answer**: OAuth 2.0 explicitly handles Authorization (delegated access) but provides no assertions validating identity natively. OIDC adds ID Tokens (JWTs) defining exactly *who* the user is (Authentication).

**48. How do you implement a secure "Forgot Password" token mechanism?**
**Answer**: Generate a CSPRNG token, store exclusively its salted hash in the database, and strictly enforce a 15-minute Time To Live (TTL).

**49. What is a "Pass-the-Hash" attack inside Windows AD internal networks?**
**Answer**: Extracting NTLM hashed passwords from memory (LSASS) and rebroadcasting identical payloads to authenticate implicitly without the plaintext password.

**50. What makes BCrypt/Argon2 superior to MD5 for password cryptography?**
**Answer**: They inherently utilize cryptographic "Salt" (defeating rainbow tables) and tunable "Work Factors" (consuming massive CPU/Memory) explicitly defeating GPU brute-forcing natively.

**(Questions 51-60: API Keys, IAM boundaries, Vaults)**
*   **51.** Secure API key issuance requires generating opaque strings with high entropy.
*   **52.** A compromised API key embedded in a mobile APK must be revoked instantly.
*   **53.** AWS KMS Envelope Encryption significantly reduces massive database encryption API latency.
*   **54.** Dynamic Secrets in HashiCorp Vault eliminate permanent database credentials.
*   **55.** K8s native secrets are only Base64 encoded by default, requiring KMS integration.
*   **56.** Utilize `git filter-repo` to systematically purge historical leaked secrets inside repositories.
*   **57.** TruffleHog scans git repos utilizing high-entropy Shannon algorithms mapping explicit secrets.
*   **58.** Principle of Least Privilege dictates explicitly scoping AWS IAM policies to unique resource ARNs.
*   **59.** Cross-account IAM roles mitigate hardcoding access keys inside CI/CD runners.
*   **60.** AWS STS issues ephemeral credentials with a mandatory TTL limit.

## Cross-Site Request Forgery (CSRF) & Access Controls (61-80)

**61. What is the explicit difference between Stored and Reflected XSS?**
**Answer**: Stored XSS saves the payload deeply into the database, hitting every user rendering that page. Reflected bounces explicitly off the server URL parameters evaluating against a clicked victim.

**62. How do you effectively mitigate XSS natively in legacy frameworks?**
**Answer**: By implementing strictly Context-Aware output encoding organically before inserting data into HTML, JS, or CSS execution blocks natively.

**63. Explain the mechanics driving CSRF.**
**Answer**: A victim visits `attacker.com`. The attacker's HTML silently forces a POST request targeting `bank.com/transfer`. If the victim is logged in, the browser implicitly attaches the `bank.com` cookies, transferring funds perfectly.

**64. How does the `SameSite` cookie attribute mitigate CSRF?**
**Answer**: Setting `SameSite=Lax` or `Strict` entirely instructs the victim's browser NEVER to attach session cookies to cross-origin POST requests initiated entirely from third-party attacker domains.

**65. What is IDOR (Insecure Direct Object Reference)?**
**Answer**: A form of Broken Access Control where an application provides direct access to database objects based explicitly on user-supplied input (e.g., changing URL `id=1` to `id=2`) flawlessly bypassing authorization verification.

**66. How do you mitigate IDOR inherently at MAANG scale?**
**Answer**: Implementing strict object-level authorization natively inside the Data Access Layer (DAL), inherently requiring the backend to verify the User ID matches the Object Owner ID natively before executing the SQL response.

**67. What establishes a secure CORS (Cross-Origin Resource Sharing) architecture?**
**Answer**: Enforcing exact strict whitelists matching backend domains via `Access-Control-Allow-Origin`. Never utilize implicit `*` wildcards on authenticated API routes.

**68. Describe an explicit Path Traversal payload modifying backend structures.**
**Answer**: Submitting `../../../../etc/shadow` effectively breaking out of implicit image directories smoothly accessing raw system files if not canonicalized securely.

**69. How does Content Security Policy (CSP) fundamentally kill XSS?**
**Answer**: Injecting strict HTTP headers instructing the browser exactly which remote domains are inherently authorized to serve scripts. It strictly blocks inline executing `<script>` elements natively.

**70. What is a SSRF attack bouncing natively via Webhooks?**
**Answer**: A user registers `http://localhost/admin` as a Webhook endpoint. The backend server inherently trusts and POSTs data exactly to its own administrative interface gracefully bypassing external network firewalls.

**(Questions 71-80: Rate Limiting, WAF, Edge Security)**
*   **71.** WAFs heavily rely entirely on OWASP Core Rule Sets.
*   **72.** API Rate Limiting mitigates basic brute forcing and targeted application layer DDoS.
*   **73.** GraphQL query depth limiting prevents inherently complex nested query DoS attacks.
*   **74.** Implement strict pagination caps to mitigate explicit unpaginated database exhaustion.
*   **75.** Use hCaptcha/reCAPTCHA implicitly validating explicit organic human interaction.
*   **76.** Avoid returning exactly `User not found` to protect against Username Enumeration.
*   **77.** AWS Shield Advanced mitigates extensive Volumetric DDoS dynamically.
*   **78.** Replay Attacks are natively mitigated by generating exact cryptographic `nonce` timestamps.
*   **79.** Utilize strict Mutual TLS (mTLS) validating both the client and server cryptographically.
*   **80.** API Gateways (Kong/Apigee) inherently centralize authentication offloading microservice load.

## Compliance, Threat Modeling & Executive DevSecOps (81-100)

**81. Difference between Authentication vs Authorization failures?**
**Answer**: Broken authentication involves compromising the user identity (weak passwords, stolen JWTs). Broken Access Control involves bypassing permissions natively (IDOR, Privilege escalation).

**82. How heavily does STRIDE impact threat modeling?**
**Answer**: Designing system diagrams evaluating specifically Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, and Elevation of Privilege natively before writing code.

**83. What mandates compliance inside SOC2 Type II requirements inherently?**
**Answer**: Demonstrating extensive long-term architectural stability mapping strictly verified audit logs natively aggregating identity access matrices explicitly proving Separation of Duties over 6-12 months.

**84. Why implement an Immutable Infrastructure architecture?**
**Answer**: Disabling SSH completely and forcing cluster node replacements inherently eradicates persistent OS malware naturally maintaining explicit High Availability dynamically.

**85. What differentiates Falco against traditional SAST/DAST configurations?**
**Answer**: Falco operates dynamically utilizing eBPF (Extended Berkeley Packet Filter) evaluating explicit kernel calls exclusively blocking malicious process runtime behavior natively.

**86. How does Zero Trust specifically secure internal Kubernetes setups?**
**Answer**: Deploying a Service Mesh natively enforcing strict mTLS automatically between all proxy sidecars inherently discarding the traditional internal VPN "Castle and Moat" vulnerability paradigm entirely.

**87. Explain Cloud Security Posture Management (CSPM).**
**Answer**: CSPM tools continuously evaluate explicit cloud configurations parsing underlying AWS APIs natively guaranteeing compliance (e.g., verifying zero S3 buckets remain publicly accessible globally).

**88. Define an explicit Container Escape matrix natively.**
**Answer**: Applications mapping heavy privileged capabilities implicitly mounting explicit Docker sockets `docker.sock` explicitly mapping native cgroups granting application root-level API execution dynamically escaping internal structures beautifully.

**89. What establishes a cryptographically secure Enterprise SSO?**
**Answer**: SAML entirely combined smoothly with Azure AD (Entra) inherently bridging identity management exclusively enforcing explicit conditional access policies gracefully reliably implicitly.

**90. How does OWASP ASVS specifically upgrade basic vulnerability testing?**
**Answer**: It shifts basic Top 10 lists into highly scalable testing matrices, evaluating requirements directly against exact Application Security controls across critical B2B deployments. It sets specific compliance thresholds (Level 1, 2, or 3) depending on the sensitivity of the data processed.

**(Questions 91-100: Executive Risk & Security Methodologies)**
*   **91. Bug Bounties**: Using Bug Bounties (like HackerOne) integrates explicit external Black-box penetration feedback from the community, providing an asymmetrical defense against zero-days.
*   **92. Penetration Testing**: White-box penetration testing maps the complete source code and internal logic flawlessly, finding deep logical flaws that black-box DAST tools simply cannot reach.
*   **93. Chaos Engineering**: Security Chaos Engineering explicitly injects malicious failures (e.g., terminating a firewall, opening an IAM port) to proactively prove that the SIEM alerts dynamically trigger and automated remediation executes correctly.
*   **94. IMDSv2 vs SSRF**: Enforcing explicit IMDSv2 blocks Cloud SSRF natively by requiring a PUT request to establish a session token with a specific TTL, breaking standard 1-step GET payloads.
*   **95. GuardDuty**: Implementing rigorous AWS GuardDuty maps explicit ML threat vectors reliably, using VPC Flow Logs and CloudTrail to identify compromised EC2 instances or exposed IAM credentials automatically.
*   **96. Threat Matrix**: DREAD evaluates vulnerabilities comprehensively: Damage potential, Reproducibility, Exploitability, Affected Users, and Discoverability, prioritizing real-world risk over theoretical CVE scores.
*   **97. Blast Radius**: A core architectural concept evaluating exactly what gets compromised if a specific component falls. Mitigate by confining the radius—using AWS Organizations, strict IAM boundaries, and network segmentation so a frontend breach cannot pivot to the core financial databases.
*   **98. Shift Left Philosophy**: Automating OWASP validations early in the SDLC seamlessly saves enterprise budgets. Fixing a vulnerable library via SCA in the IDE costs nothing; fixing it post-breach in production costs millions.
*   **99. OIDC vs Static Keys**: OIDC securely facilitates short-lived, verifiable JWT identities globally, entirely replacing the need to distribute static, highly-privileged API keys to third-party CI/CD runners (like GitHub Actions deploying to AWS).
*   **100. Security Culture**: Security is a shared responsibility reliably maintained by building paved roads. DevSecOps succeeds not by creating friction and gates, but by engineering secure defaults (like enforced SAST templates) that developers transparently consume. 

*/ End. Note: MAANG interviews focus intensely on the exact "WHY" behind every mitigation mapping to architectural system design concepts.*
