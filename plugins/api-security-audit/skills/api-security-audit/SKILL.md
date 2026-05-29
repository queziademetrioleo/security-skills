---
name: api-security-audit
description: API security testing covering authentication bypass, broken access control, CORS misconfiguration, rate limiting absence, exposed documentation, IDOR and sensitive data exposure. Use against REST/GraphQL APIs.
---

# API Security Audit

You are a security engineer auditing a REST or GraphQL API. Systematically test every security control layer.

## When to Use

- Auditing a REST or GraphQL API for authentication and authorization flaws
- Looking for CORS misconfigurations that allow cross-origin credential theft
- Verifying rate limiting is in place on sensitive endpoints
- Checking if API documentation is accidentally exposed in production
- Testing for IDOR (Insecure Direct Object References)

## Audit Phases

### Phase 1 — Endpoint Discovery and Documentation Exposure

```bash
# Check if API docs are public
for path in /docs /swagger /openapi.json /redoc /api-docs /swagger.json \
            /swagger-ui.html /api/swagger /v1/docs /v2/docs; do
  code=$(curl -s -o /dev/null -w "%{http_code}" "API_BASE${path}")
  echo "${code} -> ${path}"
done
```

**Risk:** Public documentation gives attackers a complete map of every endpoint, parameter and schema — eliminating reconnaissance time entirely.

### Phase 2 — Authentication Testing

**2a. Missing authentication (no token required)**
```bash
curl -s -X POST "API_BASE/sensitive-endpoint" \
  -H "Content-Type: application/json" \
  -d '{"test": "data"}'
```

**2b. Invalid token accepted**
```bash
curl -s "API_BASE/protected" \
  -H "Authorization: Bearer invalid_token_test_123"
```

**2c. Expired token accepted**
```bash
# Use a known-expired token from a previous session
curl -s "API_BASE/protected" \
  -H "Authorization: Bearer EXPIRED_TOKEN_HERE"
```

**2d. Token without required claims**
```bash
# Craft a minimal JWT with algorithm=none
# Header: {"alg":"none","typ":"JWT"}
# Payload: {"sub":"test","role":"user"}
NONE_JWT="eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJzdWIiOiJ0ZXN0Iiwicm9sZSI6InVzZXIifQ."
curl -s "API_BASE/admin" -H "Authorization: Bearer ${NONE_JWT}"
```

### Phase 3 — Broken Access Control and IDOR

```bash
# Create two legitimate accounts, get their IDs
# Then access account B's resources with account A's token

# Access other user's data with own token
curl -s "API_BASE/users/OTHER_USER_ID/profile" \
  -H "Authorization: Bearer MY_TOKEN"

# Access admin endpoint with regular user token
curl -s "API_BASE/admin/users" \
  -H "Authorization: Bearer REGULAR_USER_TOKEN"

# Vertical privilege escalation: change own role
curl -s -X PATCH "API_BASE/users/MY_ID" \
  -H "Authorization: Bearer MY_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"role": "admin"}'
```

### Phase 4 — CORS Misconfiguration

```bash
# Test if arbitrary origin is reflected
curl -s -I "API_BASE/sensitive" \
  -H "Origin: https://evil-attacker.com" | grep -i "access-control"

# Test null origin (file:// attack)
curl -s -I "API_BASE/sensitive" \
  -H "Origin: null" | grep -i "access-control"

# Test subdomain injection
curl -s -I "API_BASE/sensitive" \
  -H "Origin: https://evil.legitimate-domain.com" | grep -i "access-control"
```

**Critical combination:** `Access-Control-Allow-Origin: [reflected]` + `Access-Control-Allow-Credentials: true` = CORS credential theft vulnerability.

### Phase 5 — Rate Limiting

```bash
# Send 20 concurrent requests to a sensitive endpoint
for i in $(seq 1 20); do
  curl -s -o /dev/null -w "Request $i: %{http_code}\n" \
    -X POST "API_BASE/login" \
    -H "Content-Type: application/json" \
    -d '{"email":"test@test.com","password":"wrong'${i}'"}' &
done
wait
```

Check response headers for: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `Retry-After`.

If all return 200 with no rate limit headers → **rate limiting is absent**.

### Phase 6 — Security Headers

```bash
curl -s -I "API_BASE/"
```

Check for:
- `X-Content-Type-Options: nosniff`
- `Strict-Transport-Security`
- `Content-Security-Policy`
- `X-Frame-Options`

### Phase 7 — Sensitive Data Exposure

- Inspect all API responses for PII (CPF, email, phone, address)
- Look for internal IDs, stack traces, version numbers in error responses
- Check if verbose error messages reveal database type or ORM

```bash
# Trigger a validation error and inspect the response
curl -s -X POST "API_BASE/endpoint" \
  -H "Content-Type: application/json" \
  -d '{"invalid_field": true}'
```

### Phase 8 — HTTP Method Abuse

```bash
for method in GET POST PUT PATCH DELETE OPTIONS HEAD TRACE; do
  code=$(curl -s -o /dev/null -w "%{http_code}" -X "${method}" "API_BASE/resource")
  echo "${method}: ${code}"
done
```

## Risk Classification

| Finding | Severity |
|---------|----------|
| Endpoint accessible without any token | Critical |
| Admin endpoint accessible with user token | Critical |
| CORS reflects arbitrary origin + credentials allowed | High |
| No rate limiting on auth endpoints | High |
| Public API docs in production | High |
| IDOR on user resources | High |
| Verbose error messages with stack traces | Medium |
| Missing security headers | Medium |
| Expired tokens accepted | Medium |

## Output

Document each finding with:
1. Endpoint affected
2. HTTP method and payload used
3. Expected vs actual response
4. Severity and CVSS estimate
5. Remediation recommendation
