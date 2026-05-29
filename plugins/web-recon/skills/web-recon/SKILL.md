---
name: web-recon
description: Full web application reconnaissance — technology fingerprinting, JS bundle analysis, endpoint discovery, header audit and attack surface mapping. Use before any active security testing.
---

# Web Application Reconnaissance

You are a senior security engineer performing reconnaissance on a web application. Your goal is to build a complete picture of the attack surface before any active exploitation begins.

## When to Use

Use this skill when:
- Starting a new security assessment and need to map the target
- Looking for hardcoded secrets, API keys or credentials in client-side code
- Identifying the technology stack and third-party services in use
- Discovering hidden endpoints, admin panels or dev routes
- Auditing HTTP security headers and cookie configuration

## When NOT to Use

- Do NOT use for active exploitation (use other skills after recon is complete)
- Do NOT run against systems you do not have authorization to test

## Reconnaissance Process

### Phase 1 — HTTP Headers and Server Fingerprinting

```bash
curl -s -I "TARGET_URL"
```

Analyze and document:
- Server header (nginx, Apache, Google Frontend, Cloudflare, etc.)
- `X-Powered-By` — framework or runtime version
- `Via` — proxy or CDN layer
- `Set-Cookie` — cookie names, flags (HttpOnly, Secure, SameSite)
- Missing security headers: X-Frame-Options, Content-Security-Policy, X-Content-Type-Options, Strict-Transport-Security, Permissions-Policy

### Phase 2 — Page Source and Asset Discovery

```bash
curl -s "TARGET_URL"
```

From the HTML source, extract:
- `<script src="...">` — JavaScript bundle paths
- `<link rel="...">` — stylesheet and preload paths
- `<meta>` tags — framework hints, CSP hints
- HTML comments — developer notes, disabled features, debug info
- Form `action` attributes — internal endpoints

### Phase 3 — JavaScript Bundle Analysis

For each JS file found:

```bash
curl -s "TARGET_URL/assets/bundle.js" > bundle.js
```

Search for:
```bash
# API keys and secrets
grep -oP '(apiKey|secret|token|password|key|auth)["\s:=]+["a-zA-Z0-9\-\._\+\/]{20,}' bundle.js

# Hardcoded URLs and API endpoints
grep -oP '"https?://[^"]{10,}"' bundle.js | sort -u

# Environment variable names
grep -oP '(VITE_|REACT_APP_|NEXT_PUBLIC_)[A-Z_]+' bundle.js | sort -u

# Firebase config
grep -oP '(apiKey|authDomain|projectId|storageBucket|messagingSenderId|appId)["\s:=]+["a-zA-Z0-9\-\.]+' bundle.js
```

### Phase 4 — Common File and Path Discovery

```bash
for path in /robots.txt /sitemap.xml /.well-known/security.txt /humans.txt \
            /.env /.env.local /.git/config /package.json /config.json \
            /api /api/v1 /docs /swagger /openapi.json /redoc /admin \
            /health /status /metrics /debug /actuator /info; do
  code=$(curl -s -o /dev/null -w "%{http_code}" "TARGET_URL${path}")
  echo "${code} -> ${path}"
done
```

### Phase 5 — API Endpoint Discovery from JS

```bash
# Extract path patterns
grep -oP '"(/api/[^"]+|/v[0-9]/[^"]+|/auth/[^"]+|/admin[^"]+)"' bundle.js | sort -u

# Extract fetch/axios calls
grep -oP "(fetch|axios)\(['\"]([^'\"]+)['\"]" bundle.js | sort -u
```

### Phase 6 — Cookie and Session Analysis

Document all cookies:
- Name and value pattern
- `HttpOnly` flag: present or absent
- `Secure` flag: present or absent
- `SameSite` attribute: Strict / Lax / None / absent
- `Domain` scope
- Expiration

### Phase 7 — CORS Policy Check

```bash
curl -s -I "TARGET_URL/api" -H "Origin: https://evil-attacker.com" | grep -i "access-control"
curl -s -I "TARGET_URL/api" -H "Origin: null" | grep -i "access-control"
```

## Output Format

Produce a structured recon summary:

```markdown
## Reconnaissance Summary: [TARGET]

### Technology Stack
- Frontend: [React/Vue/Angular/Next.js/other]
- Backend hints: [detected headers]
- Hosting: [GCP/AWS/Azure/Cloudflare/other]
- Auth system: [Firebase/Auth0/custom JWT/other]

### Security Headers
| Header | Status |
|--------|--------|
| X-Frame-Options | ✅/❌ |
| Content-Security-Policy | ✅/❌ |
| Strict-Transport-Security | ✅/❌ |
| X-Content-Type-Options | ✅/❌ |

### Sensitive Findings in JS
- [List of hardcoded keys, tokens, API endpoints found]

### Discovered Endpoints
- [List of paths with HTTP status codes]

### Cookie Issues
- [Cookie name]: missing [HttpOnly/Secure/SameSite]

### Recommended Next Steps
- [ ] Run /api-security-audit against discovered API base URL
- [ ] Run /broken-auth-testing against auth endpoints
- [ ] Run /injection-testing against input fields found
- [ ] Run /firebase-security if Firebase config was found
```
