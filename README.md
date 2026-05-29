# Security Skills

> Claude Code skills for security testing of web applications, APIs, AI/LLM systems and mobile apps.

Built for security engineers, red teamers and developers who want structured, repeatable security assessments powered by Claude Code.

## Skills Available

### AI Security (OWASP LLM Top 10)

| Skill | Description |
|-------|-------------|
| [`ai-llm-security`](./plugins/ai-llm-security) | Core OWASP LLM Top 10: prompt injection, jailbreak, SSRF via tools, architecture exposure, model DoS |
| [`rag-security`](./plugins/rag-security) | RAG pipelines: document poisoning, retrieval bypass, cross-tenant leakage, context injection |
| [`agentic-systems-security`](./plugins/agentic-systems-security) | Multi-agent systems (LangGraph, CrewAI, ADK): tool abuse, orchestrator bypass, privilege escalation |
| [`llm-api-security`](./plugins/llm-api-security) | LLM API wrappers (OpenAI, Anthropic, Gemini): key exposure, proxy bypass, cost abuse, prompt extraction |
| [`ai-model-security`](./plugins/ai-model-security) | Model deployments: extraction/stealing, membership inference, training data extraction, adversarial inputs |

### Web and API Security (OWASP Top 10)

| Skill | Description |
|-------|-------------|
| [`web-recon`](./plugins/web-recon) | Reconnaissance, fingerprinting, JS bundle analysis, attack surface mapping |
| [`api-security-audit`](./plugins/api-security-audit) | Auth bypass, CORS, rate limiting, IDOR, exposed docs |
| [`firebase-security`](./plugins/firebase-security) | Exposed API keys, Security Rules, user enumeration |
| [`broken-auth-testing`](./plugins/broken-auth-testing) | Token validation, session security, brute force, MFA bypass |
| [`injection-testing`](./plugins/injection-testing) | SQL, XSS, command injection, SSTI |
| [`owasp-top10`](./plugins/owasp-top10) | Full OWASP Top 10 (2021) assessment checklist |

### Reporting

| Skill | Description |
|-------|-------------|
| [`pentest-report`](./plugins/pentest-report) | Structured markdown reports with CVSS, PoC and remediation roadmap |

## Installation

Add this repo as a plugin source in Claude Code:

```bash
claude mcp add security-skills https://github.com/queziademetrioleo/security-skills
```

Or install a single skill manually by copying the skill directory into `~/.claude/skills/`.

## Usage

Invoke any skill by name in your Claude Code session:

```
/web-recon https://target.com
/api-security-audit https://api.target.com
/ai-llm-security https://chat.target.com
/pentest-report
```

## Scope and Ethics

These skills are intended for:
- Authorized penetration testing engagements
- Security audits of systems you own or have written permission to test
- CTF competitions and security research in controlled environments
- Developer security training

**Do not use against systems without explicit written authorization.**

## Structure

```
security-skills/
├── .claude-plugin/
│   └── marketplace.json       # Plugin registry
├── plugins/
│   └── <skill-name>/
│       ├── .claude-plugin/
│       │   └── plugin.json    # Skill metadata
│       ├── README.md          # Usage documentation
│       └── skills/
│           └── <skill-name>/
│               ├── SKILL.md   # Main skill instructions
│               └── resources/ # Templates, payloads, checklists
├── CLAUDE.md
└── README.md
```

## Contributing

1. Fork the repo
2. Create a new skill directory following the structure above
3. Add your skill to `.claude-plugin/marketplace.json`
4. Open a pull request
