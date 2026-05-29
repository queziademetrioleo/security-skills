---
name: firebase-security
description: Firebase-specific security audit. Tests exposed API keys in JS bundles, Security Rules misconfigurations, user enumeration via Identity Toolkit API, Firestore and Storage access control, and authentication bypass. Use when a Firebase config is found in client-side code.
---

# Firebase Security Audit

You are a security engineer auditing a Firebase-based application. Firebase exposes a client-side API key and config by design — the security relies on Firebase Security Rules and proper restrictions. When these are misconfigured, the impact is severe.

## When to Use

- A Firebase `apiKey`, `authDomain`, or `projectId` was found in client-side JavaScript
- Auditing Firestore, Realtime Database, or Firebase Storage for access control
- Testing Firebase Authentication for user enumeration or bypass
- Verifying the API key has proper HTTP referrer restrictions

## Phase 1 — Extract Firebase Config

Search JavaScript bundles for the Firebase config object:

```bash
curl -s "APP_URL/assets/bundle.js" | \
  grep -oP '(apiKey|authDomain|projectId|storageBucket|messagingSenderId|appId|measurementId)["\s:=]+["a-zA-Z0-9\-\.]+' | \
  sort -u
```

Record all values:
```
apiKey:           AIzaSy...
authDomain:       project-id.firebaseapp.com
projectId:        project-id
storageBucket:    project-id.appspot.com
messagingSenderId: 123456789
appId:            1:123:web:abc123
```

## Phase 2 — API Key Restriction Audit

Test whether the API key is restricted to allowed HTTP referrers:

```bash
# If key is unrestricted, these will all succeed:
curl -s "https://identitytoolkit.googleapis.com/v1/accounts:signInWithPassword?key=API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"email":"test@test.com","password":"test","returnSecureToken":true}'

# Expected if key is restricted: 400 API_KEY_HTTP_REFERRER_BLOCKED
# Expected if key is unrestricted: 400 INVALID_LOGIN_CREDENTIALS (key works, wrong password)
```

## Phase 3 — User Enumeration

Firebase historically had different error codes for existing vs non-existing users. Test:

```bash
# Test with a known/suspected real email
curl -s -X POST "https://identitytoolkit.googleapis.com/v1/accounts:signInWithPassword?key=API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"email":"known@company.com","password":"wrongpass","returnSecureToken":true}'

# Test with a clearly fake email
curl -s -X POST "https://identitytoolkit.googleapis.com/v1/accounts:signInWithPassword?key=API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"email":"definitelynotreal_xyz@company.com","password":"wrongpass","returnSecureToken":true}'
```

Compare error messages:
- `INVALID_LOGIN_CREDENTIALS` for both → enumeration mitigated ✅
- Different errors (e.g., `EMAIL_NOT_FOUND` vs `WRONG_PASSWORD`) → enumeration possible ❌

Also test via `createAuthUri`:

```bash
curl -s -X POST "https://identitytoolkit.googleapis.com/v1/accounts:createAuthUri?key=API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"identifier":"target@company.com","continueUri":"https://app.com/"}'
```

If `allProviders` is returned in the response → email exists in the system.

## Phase 4 — Self-Registration Abuse

```bash
# Test if anyone can create an account
curl -s -X POST "https://identitytoolkit.googleapis.com/v1/accounts:signUp?key=API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"email":"attacker@evil.com","password":"Test1234!","returnSecureToken":true}'
```

- `ADMIN_ONLY_OPERATION` → sign-up is restricted to admins ✅
- Returns `idToken` → anyone can create an account ❌

## Phase 5 — Firestore Security Rules Testing

With a valid Firebase ID token, test Firestore collection access:

```bash
TOKEN="FIREBASE_ID_TOKEN"
PROJECT="your-project-id"

# Try to read all documents in a collection
curl -s "https://firestore.googleapis.com/v1/projects/${PROJECT}/databases/(default)/documents/users" \
  -H "Authorization: Bearer ${TOKEN}"

# Try to read admin collection
curl -s "https://firestore.googleapis.com/v1/projects/${PROJECT}/databases/(default)/documents/admin" \
  -H "Authorization: Bearer ${TOKEN}"

# Try to write to a collection
curl -s -X POST \
  "https://firestore.googleapis.com/v1/projects/${PROJECT}/databases/(default)/documents/users" \
  -H "Authorization: Bearer ${TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"fields":{"test":{"stringValue":"security_test"}}}'
```

## Phase 6 — Firebase Storage Access

```bash
BUCKET="your-project-id.appspot.com"
TOKEN="FIREBASE_ID_TOKEN"

# List all files in storage root
curl -s "https://firebasestorage.googleapis.com/v0/b/${BUCKET}/o" \
  -H "Authorization: Bearer ${TOKEN}"

# Try without any token (public bucket?)
curl -s "https://firebasestorage.googleapis.com/v0/b/${BUCKET}/o"

# Download a specific file
curl -s "https://firebasestorage.googleapis.com/v0/b/${BUCKET}/o/path%2Ffile.pdf?alt=media" \
  -H "Authorization: Bearer ${TOKEN}"
```

## Phase 7 — Firebase App Check

Check if Firebase App Check is enforced:

```bash
# Request without App Check token — if App Check is enforced, this should fail
curl -s -X POST "https://identitytoolkit.googleapis.com/v1/accounts:signInWithPassword?key=API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"email":"test@test.com","password":"test"}'

# A 400 with "APP_NOT_AUTHORIZED" means App Check is enforced ✅
# Returning INVALID_LOGIN_CREDENTIALS means App Check is NOT enforced ❌
```

## Risk Classification

| Finding | Severity |
|---------|----------|
| API key unrestricted (no HTTP referrer restriction) | High |
| Anyone can create accounts (self-registration open) | High |
| User enumeration possible via different error codes | Medium |
| Firestore collection readable without authorization | Critical |
| Firebase Storage files downloadable without auth | High |
| App Check not enforced (key usable from any origin) | Medium |
| `messagingSenderId` exposed (FCM notification abuse) | Low |

## Remediation Reference

| Issue | Fix |
|-------|-----|
| Unrestricted API key | Google Cloud Console → APIs & Services → Credentials → HTTP referrer restriction |
| Open self-registration | Firebase Console → Authentication → Sign-in method → disable email sign-up |
| Insecure Security Rules | Firebase Console → Firestore/Storage → Rules → require `request.auth != null` |
| App Check not enforced | Firebase Console → App Check → Enforce for each service |
