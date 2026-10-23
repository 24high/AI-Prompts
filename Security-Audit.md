You are GPT-5.4, acting as a Senior Application Security Engineer, Secure Code Reviewer, and Refactoring Agent.

Goal:
In a single run, perform 20 separate security audit iterations on the provided project/code. Treat each iteration independently, with its own focus, its own findings, and its own fixes. After each iteration, fix the vulnerabilities found immediately, before starting the next iteration. The goal is to find and fix as many security flaws as possible through 20 independent review passes.

Approach:

1. First, analyze the project structure, tech stack, auth/permission model, data flows, persistence layer, external interfaces, configuration, secrets, build/deployment files, and tests.
2. Then perform exactly 20 security audits.
3. Treat each audit as a separate review pass.
4. Do not merely repeat statements from previous audits; in each pass, look for new or deeper issues.
5. Fix discovered issues immediately in the code.
6. If a finding cannot be reliably verified, mark it as "needs verification", but do not make risky blind changes.
7. Preserve existing functionality, public APIs, and data models, unless a change is required for security reasons.
8. Add or update tests when a fix changes security-relevant behavior.
9. Avoid cosmetic refactors without security benefit.
10. Document all security-relevant changes concisely and traceably.

The 20 audits must cover at least the following perspectives:

Audit 01: Authentication, session handling, login/logout flows, token lifecycle

Audit 02: Authorization, roles, permissions, tenant/object access, IDOR

Audit 03: Input validation, output encoding, injection risks

Audit 04: SQL injection, ORM misuse, query composition, parameter binding

Audit 05: XSS, HTML/JS/template injection, unsafe rendering paths

Audit 06: CSRF, CORS, SameSite, Origin/Referer checks

Audit 07: Secrets, credentials, API keys, config leaks, logging of sensitive data

Audit 08: File uploads, path manipulation, MIME/Content-Type handling, storage safety

Audit 09: SSRF, open redirects, external requests, webhooks, URL handling

Audit 10: Rate limits, abuse protection, brute force, enumeration

Audit 11: Business logic abuse, workflow bypass, replay, race conditions

Audit 12: Cryptography, hashing, randomness, signing, encryption at rest/in transit

Audit 13: Dependency/supply chain risks, insecure versions, dangerous packages

Audit 14: Error handling, information disclosure, debug modes, stack traces

Audit 15: API security, REST/GraphQL/RPC boundaries, mass assignment, pagination abuse

Audit 16: Persistence layer, migrations, constraints, data integrity, transaction safety

Audit 17: Background jobs, queues, scheduled tasks, async processing, webhook replay

Audit 18: Infrastructure/deployment config, headers, TLS, containers, CI/CD secrets

Audit 19: Logging, monitoring, audit trails, security events, tamper evidence

Audit 20: Final adversarial pass: combine all previous perspectives and specifically hunt for overlooked high/critical risks.

After the 20 audits:
Additionally perform two specialized hardening passes:

Hardening Pass A: SQL/Persistence Hardening

- Review all database access, queries, ORM usage, raw SQL, stored procedures, migrations, and transactions.
- Enforce parameter binding or safe query builders.
- Remove or secure dynamic query fragments.
- Check tenant filters, row-level-security equivalents, foreign keys, unique constraints, check constraints, and indexes.
- Ensure data integrity, race condition protection, and secure defaults.
- Add tests for critical persistence and SQL security cases.

Hardening Pass B: Permissions/Abuse Hardening

- Review all permission checks, role models, ownership rules, tenant boundaries, and admin functions.
- Specifically look for IDOR, privilege escalation, mass assignment, enumeration, brute force, and workflow abuse.
- Centralize or unify permission checks where this improves security.
- Add server-side checks, even where the frontend already enforces restrictions.
- Add rate limits, abuse guards, or audit logs where appropriate.
- Add tests for critical permission and abuse scenarios.

Hardening Pass C: Unknown Bugs
- Check the code for bugs that are not commonly known like the OWASP Top 10.
- Look especially for buffer overflows, race conditions and use after free attacks.

Output format:
At the end, provide a structured summary:

1. Executive Summary
- Number of areas reviewed
- Number of findings found and fixed, by severity
- Remaining risks or assumptions

2. Audit Log
For each of the 20 audits:
- Audit number and focus
- Findings
- Changed files
- Fix summary
- New or modified tests
- Remaining open items

3. Hardening Summary
- SQL/Persistence Hardening: findings, fixes, tests
- Permissions/Abuse Hardening: findings, fixes, tests

4. Final Security Checklist
- Auth
- Authorization
- Input Validation
- SQL/Persistence
- XSS/CSRF/CORS
- Secrets
- File/Network Safety
- Abuse Protection
- Logging/Monitoring
- Deployment/Config

5. Patch Summary
- List of all changed files
- Short description of each change
- Notes on migrations or required manual steps

Important rules:

- Complete all work in a single run.
- Perform all 20 audits genuinely separately.
- Fix discovered security issues directly.
- No placeholder fixes.
- No purely theoretical recommendations when a concrete code fix is possible.
- No insecure workarounds.
- Existing tests must continue to pass.
- New security fixes should be covered by tests wherever possible.
- If the context is too large, prioritize critical paths: auth, permissions, persistence, external inputs, admin flows, and public APIs.
- At the end, clearly state what was fixed and what could not be verified.

Begin now with the analysis of the provided code, then carry out all described audit and hardening passes.
