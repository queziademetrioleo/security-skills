---
name: owasp-top10
description: Full OWASP Top 10 (2021) assessment workflow. Provides a structured checklist and test cases for each of the 10 risk categories: Broken Access Control, Cryptographic Failures, Injection, Insecure Design, Security Misconfiguration, Vulnerable Components, Authentication Failures, Data Integrity, Logging/Monitoring, and SSRF. Use for comprehensive baseline assessments.
---

# OWASP Top 10 (2021) Assessment

You are a security engineer conducting a baseline OWASP Top 10 assessment. Work through each category systematically, collecting evidence and marking items as pass, fail or not applicable.

## Assessment Template

For each category, mark each test item:
- ✅ Pass — tested and no issue found
- ❌ Fail — vulnerability confirmed (document with evidence)
- ⚠️ Partial — some controls present but incomplete
- N/A — not applicable to this application

---

## A01 — Broken Access Control

The most common and impactful category. Verify that users can only access resources they are authorized for.

**Checklist:**

- [ ] Unauthenticated access to protected endpoints
- [ ] Regular user can access admin endpoints
- [ ] User A can access User B's data (IDOR)
- [ ] HTTP method bypass (GET vs POST vs PUT on same resource)
- [ ] Missing access control on API endpoints that have it on the UI
- [ ] Directory traversal: `/api/files/../../../etc/passwd`
- [ ] Mass assignment: can user update fields they shouldn't (e.g., `role`, `is_admin`)

**Tests:**
```bash
# Test unauthenticated access
curl -s "BASE_URL/api/users"
curl -s "BASE_URL/api/admin/dashboard"

# IDOR
curl -s "BASE_URL/api/users/1" -H "Authorization: Bearer USER_2_TOKEN"
curl -s "BASE_URL/api/users/2" -H "Authorization: Bearer USER_2_TOKEN"

# Mass assignment
curl -s -X PATCH "BASE_URL/api/users/me" \
  -d '{"role":"admin","is_admin":true}'
```

---

## A02 — Cryptographic Failures

Sensitive data exposed due to weak or absent encryption.

**Checklist:**

- [ ] Sensitive data transmitted over HTTP (not HTTPS)
- [ ] Weak TLS version (TLS 1.0 or 1.1 accepted)
- [ ] Sensitive data in URL parameters (passwords, tokens in query strings)
- [ ] Sensitive data in responses that shouldn't be there (passwords, full card numbers)
- [ ] Hardcoded secrets in client-side code (JS bundles)
- [ ] Passwords stored as MD5 or SHA1 (check via breach data if applicable)
- [ ] Encryption keys exposed in source code

**Tests:**
```bash
# Check TLS
curl -s -v "https://target.com" 2>&1 | grep "SSL connection\|TLS"

# Check for secrets in JS
curl -s "TARGET_URL/bundle.js" | grep -oP '(apiKey|secret|token|password)["\s:=]+["a-zA-Z0-9]{20,}'

# Check if HTTP redirects to HTTPS or if HTTP is accepted
curl -s -o /dev/null -w "%{http_code}" "http://target.com"
```

---

## A03 — Injection

Untrusted data sent to interpreters.

**Checklist:**

- [ ] SQL injection in query parameters
- [ ] SQL injection in POST body (JSON fields)
- [ ] NoSQL injection (MongoDB operators in JSON)
- [ ] Reflected XSS in URL parameters
- [ ] Stored XSS in user-generated content fields
- [ ] DOM-based XSS in JavaScript
- [ ] OS command injection
- [ ] LDAP injection
- [ ] SSTI (Server-Side Template Injection)
- [ ] XML/XXE injection

**→ Use the `injection-testing` skill for detailed test procedures**

---

## A04 — Insecure Design

Design flaws that cannot be fixed by implementation alone.

**Checklist:**

- [ ] No rate limiting on resource-intensive operations
- [ ] Business logic bypasses (skip steps in a workflow)
- [ ] No input validation on business-critical fields (e.g., negative prices, negative quantities)
- [ ] Predictable resource identifiers (sequential IDs)
- [ ] Sensitive operations without confirmation step
- [ ] No abuse prevention for free-tier features

**Tests:**
```bash
# Negative price test
curl -s -X POST "BASE_URL/api/order" \
  -d '{"product_id":1,"quantity":-999,"price":-100}'

# Skip payment step
curl -s -X POST "BASE_URL/api/order/complete" \
  -d '{"order_id":"UNPAID_ORDER_ID"}'
```

---

## A05 — Security Misconfiguration

**Checklist:**

- [ ] Default credentials active
- [ ] Unnecessary features enabled (debug mode, verbose errors, directory listing)
- [ ] Public API documentation in production (Swagger, OpenAPI)
- [ ] Detailed error messages with stack traces
- [ ] Missing security headers (CSP, X-Frame-Options, HSTS, etc.)
- [ ] CORS misconfiguration (arbitrary origin + credentials)
- [ ] Cloud storage buckets publicly readable
- [ ] Admin panels accessible from the internet

**Tests:**
```bash
# Security headers
curl -s -I "TARGET_URL" | grep -iE "(x-frame|content-security|strict-transport|x-content-type)"

# Public docs
curl -s -o /dev/null -w "%{http_code}" "TARGET_URL/swagger"
curl -s -o /dev/null -w "%{http_code}" "TARGET_URL/docs"

# Error verbosity
curl -s "TARGET_URL/api/nonexistent"
```

---

## A06 — Vulnerable and Outdated Components

**Checklist:**

- [ ] Frontend libraries with known CVEs
- [ ] Backend framework version disclosed and outdated
- [ ] Dependencies with known vulnerabilities (npm audit, pip-audit)
- [ ] Server software version disclosed in headers
- [ ] Docker images with outdated base OS

**Tests:**
```bash
# Server version disclosure
curl -s -I "TARGET_URL" | grep -iE "(server|x-powered-by)"

# If source code is accessible:
npm audit
pip-audit
```

---

## A07 — Identification and Authentication Failures

**Checklist:**

- [ ] Weak password policy (min length, complexity)
- [ ] No brute force protection on login
- [ ] JWT algorithm confusion or weak secret
- [ ] Session tokens not invalidated on logout
- [ ] Cookie missing HttpOnly/Secure/SameSite flags
- [ ] Password reset tokens predictable or reusable
- [ ] MFA bypassable

**→ Use the `broken-auth-testing` skill for detailed test procedures**

---

## A08 — Software and Data Integrity Failures

**Checklist:**

- [ ] Auto-update mechanisms without signature verification
- [ ] Deserialization of untrusted data
- [ ] CI/CD pipeline accessible or injectable
- [ ] CDN scripts loaded without Subresource Integrity (SRI) hashes

**Tests:**
```bash
# Check for SRI on CDN scripts
curl -s "TARGET_URL" | grep -oP '<script[^>]+src="https://[^"]*"[^>]*>' | grep -v "integrity="
```

---

## A09 — Security Logging and Monitoring Failures

**Checklist:**

- [ ] Failed login attempts not logged or alerted
- [ ] No alerting on multiple failed authentication attempts
- [ ] No audit trail for sensitive operations (delete, admin actions)
- [ ] Log data not protected from injection
- [ ] No incident response capability observed

**Assessment approach:** Ask the client team or observe during testing whether alerts were triggered. Try 20 failed logins — was there any automated response?

---

## A10 — Server-Side Request Forgery (SSRF)

**Checklist:**

- [ ] URL fetch endpoints that accept user-supplied URLs
- [ ] PDF/screenshot generation from user URLs
- [ ] Webhooks that accept user-configured URLs
- [ ] Import/export features that fetch remote resources
- [ ] AI agent tools that make outbound requests based on user input

**Tests:**
```bash
# Test any URL fetch endpoint
curl -s -X POST "BASE_URL/api/fetch" \
  -d '{"url":"http://169.254.169.254/latest/meta-data/"}'

# Blind SSRF via Burp Collaborator equivalent
curl -s -X POST "BASE_URL/api/webhook" \
  -d '{"callback_url":"http://your-server.com/ssrf-test"}'
```

---

## Final Checklist Summary

After completing all tests, fill in this summary:

| Category | Status | Critical Findings |
|----------|--------|------------------|
| A01 — Broken Access Control | ❌/✅/⚠️ | [count] |
| A02 — Cryptographic Failures | ❌/✅/⚠️ | [count] |
| A03 — Injection | ❌/✅/⚠️ | [count] |
| A04 — Insecure Design | ❌/✅/⚠️ | [count] |
| A05 — Security Misconfiguration | ❌/✅/⚠️ | [count] |
| A06 — Vulnerable Components | ❌/✅/⚠️ | [count] |
| A07 — Authentication Failures | ❌/✅/⚠️ | [count] |
| A08 — Data Integrity | ❌/✅/⚠️ | [count] |
| A09 — Logging and Monitoring | ❌/✅/⚠️ | [count] |
| A10 — SSRF | ❌/✅/⚠️ | [count] |

**→ Use the `pentest-report` skill to generate the final report from these findings**
