<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:0d1117,50:b91c1c,100:0d1117&height=200&section=header&text=Security%20Skills&fontSize=60&fontColor=ffffff&fontAlignY=38&desc=Claude%20Code%20Skills%20for%20AI%20%26%20Web%20Security%20Testing&descAlignY=58&descSize=18" width="100%"/>

<br/>

[![Skills](https://img.shields.io/badge/Skills-12-red?style=for-the-badge&logo=shield&logoColor=white)](./plugins)
[![AI Security](https://img.shields.io/badge/AI%20Security-5%20Skills-blueviolet?style=for-the-badge&logo=openai&logoColor=white)](./plugins/ai-llm-security)
[![OWASP](https://img.shields.io/badge/OWASP-Top%2010-orange?style=for-the-badge&logo=owasp&logoColor=white)](./plugins/owasp-top10)
[![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](./LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Compatible-black?style=for-the-badge&logo=anthropic&logoColor=white)](https://claude.ai/code)

<br/>

> **12 Claude Code skills** for security testing of web apps, APIs, AI/LLM systems and ML models.  
> Built for red teamers, security engineers and developers who want structured, repeatable assessments.

<br/>

</div>

---

## Skill Map

```
security-skills/
│
├── 🤖  AI Security (OWASP LLM Top 10)
│   ├── ai-llm-security          → Prompt injection, jailbreak, SSRF via tools
│   ├── rag-security             → Document poisoning, retrieval bypass, tenant leakage
│   ├── agentic-systems-security → Tool abuse, orchestrator bypass, privilege escalation
│   ├── llm-api-security         → API key exposure, proxy bypass, cost abuse
│   └── ai-model-security        → Model stealing, membership inference, adversarial inputs
│
├── 🌐  Web & API Security (OWASP Top 10)
│   ├── web-recon                → Fingerprinting, JS bundle analysis, attack surface
│   ├── api-security-audit       → Auth bypass, CORS, IDOR, rate limiting
│   ├── firebase-security        → API key abuse, Security Rules, user enumeration
│   ├── broken-auth-testing      → JWT attacks, session security, brute force, MFA
│   ├── injection-testing        → SQL, XSS, command injection, SSTI
│   └── owasp-top10              → Full OWASP Top 10 (2021) checklist
│
└── 📄  Reporting
    └── pentest-report           → CVSS-scored reports with PoC and remediation
```

---

## Skills

<table>
  <thead>
    <tr>
      <th>Category</th>
      <th>Skill</th>
      <th>What It Tests</th>
      <th>Standard</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="5"><b>🤖 AI Security</b></td>
      <td><a href="./plugins/ai-llm-security"><code>ai-llm-security</code></a></td>
      <td>Prompt injection (5 variants), jailbreak, SSRF via agent tools, architecture exposure via SSE, model DoS, indirect injection</td>
      <td>OWASP LLM Top 10</td>
    </tr>
    <tr>
      <td><a href="./plugins/rag-security"><code>rag-security</code></a></td>
      <td>Document poisoning via upload, retrieval bypass with adversarial queries, cross-tenant data leakage, vector DB direct access, embedding key exposure</td>
      <td>OWASP LLM Top 10</td>
    </tr>
    <tr>
      <td><a href="./plugins/agentic-systems-security"><code>agentic-systems-security</code></a></td>
      <td>Tool enumeration, code/file/DB/email tool abuse, orchestrator bypass, sub-agent direct access, cross-session memory leakage, privilege escalation via tool chaining</td>
      <td>OWASP LLM Top 10</td>
    </tr>
    <tr>
      <td><a href="./plugins/llm-api-security"><code>llm-api-security</code></a></td>
      <td>API key discovery in JS/APK/repos, proxy bypass, system prompt extraction, model enumeration, token exhaustion and cost abuse</td>
      <td>OWASP LLM Top 10</td>
    </tr>
    <tr>
      <td><a href="./plugins/ai-model-security"><code>ai-model-security</code></a></td>
      <td>Model extraction via API probing, membership inference, training data extraction, adversarial inputs, content filter evasion, ML supply chain (pickle, dataset poisoning)</td>
      <td>OWASP LLM Top 10</td>
    </tr>
    <tr>
      <td rowspan="6"><b>🌐 Web & API</b></td>
      <td><a href="./plugins/web-recon"><code>web-recon</code></a></td>
      <td>HTTP headers, JS bundle analysis, hardcoded secrets, endpoint discovery, cookie audit, CORS check, technology fingerprinting</td>
      <td>OWASP Top 10</td>
    </tr>
    <tr>
      <td><a href="./plugins/api-security-audit"><code>api-security-audit</code></a></td>
      <td>Unauthenticated access, IDOR, CORS misconfiguration, rate limiting absence, exposed Swagger/OpenAPI docs, HTTP method abuse, sensitive data in responses</td>
      <td>OWASP A01–A05</td>
    </tr>
    <tr>
      <td><a href="./plugins/firebase-security"><code>firebase-security</code></a></td>
      <td>API key restrictions, user enumeration, self-registration abuse, Firestore Security Rules, Storage access control, Firebase App Check</td>
      <td>OWASP A02, A07</td>
    </tr>
    <tr>
      <td><a href="./plugins/broken-auth-testing"><code>broken-auth-testing</code></a></td>
      <td>JWT alg:none, weak secret brute force, algorithm confusion, session fixation, cookie flags, brute force protection, MFA bypass, password reset flaws</td>
      <td>OWASP A07</td>
    </tr>
    <tr>
      <td><a href="./plugins/injection-testing"><code>injection-testing</code></a></td>
      <td>SQL injection, NoSQL injection, reflected/stored/DOM XSS, OS command injection, SSTI (Jinja2, FreeMarker), LDAP injection, HTTP header injection</td>
      <td>OWASP A03</td>
    </tr>
    <tr>
      <td><a href="./plugins/owasp-top10"><code>owasp-top10</code></a></td>
      <td>Full assessment covering all 10 OWASP categories with pass/fail checklists and test commands for each</td>
      <td>OWASP Top 10 (2021)</td>
    </tr>
    <tr>
      <td><b>📄 Reporting</b></td>
      <td><a href="./plugins/pentest-report"><code>pentest-report</code></a></td>
      <td>Generates structured reports with executive summary, CVSS scoring, proof-of-concept evidence, risk table and remediation roadmap</td>
      <td>NIST SP 800-115</td>
    </tr>
  </tbody>
</table>

---

## Quick Start

**Invoke any skill by name in Claude Code:**

```bash
# AI Security
/ai-llm-security https://chat.target.com
/rag-security https://docs-chat.target.com
/agentic-systems-security https://agent.target.com
/llm-api-security https://app.target.com
/ai-model-security https://model-api.target.com

# Web & API Security
/web-recon https://target.com
/api-security-audit https://api.target.com
/firebase-security https://app.target.com
/broken-auth-testing https://target.com/login
/injection-testing https://target.com
/owasp-top10 https://target.com

# After testing
/pentest-report
```

**Or follow the recommended flow:**

```
1. /web-recon           → map attack surface
2. /api-security-audit  → test all API endpoints found
3. /broken-auth-testing → deep-dive on auth
4. /injection-testing   → test all input fields
5. /pentest-report      → generate deliverable report
```

**For AI systems:**

```
1. /ai-llm-security          → baseline LLM security checks
2. /rag-security              → if system uses a knowledge base
3. /agentic-systems-security  → if system has multiple agents/tools
4. /llm-api-security          → check for exposed provider API keys
5. /pentest-report            → document findings
```

---

## Installation

**Install from this repo:**

```bash
# Clone and copy skills to Claude Code skills directory
git clone https://github.com/queziademetrioleo/security-skills
cp -r security-skills/plugins/* ~/.claude/skills/
```

**Or install a single skill:**

```bash
cp -r security-skills/plugins/ai-llm-security ~/.claude/skills/
```

---

## Repo Structure

```
security-skills/
├── .claude-plugin/
│   └── marketplace.json          # Plugin registry (all 12 skills)
├── plugins/
│   └── <skill-name>/
│       ├── .claude-plugin/
│       │   └── plugin.json       # Skill metadata and version
│       ├── README.md             # Usage docs
│       └── skills/
│           └── <skill-name>/
│               ├── SKILL.md      # Skill instructions loaded into Claude
│               └── resources/    # Payloads, checklists, templates
├── CLAUDE.md                     # Context for Claude Code
└── README.md
```

---

## Ethical Use

These skills are intended for:

- ✅ Authorized penetration testing engagements
- ✅ Security audits of systems you own
- ✅ CTF competitions and security research in controlled environments
- ✅ Developer security training and education

**Do not use against systems without explicit written authorization.**

---

## Contributing

1. Fork this repo
2. Create a new skill in `plugins/<skill-name>/`
3. Follow the structure in any existing skill
4. Add your skill to `.claude-plugin/marketplace.json`
5. Open a pull request

---

<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:0d1117,50:b91c1c,100:0d1117&height=100&section=footer" width="100%"/>

Made for the security community · Built with [Claude Code](https://claude.ai/code)

</div>
