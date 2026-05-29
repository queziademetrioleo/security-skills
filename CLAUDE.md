# Security Skills — Claude Code Plugin

This repo contains security testing skills for Claude Code.

## Ethical Use

Always confirm written authorization before running any security test. These skills send real HTTP requests to the target system. Never run against production systems without explicit approval from the system owner.

## How Skills Work

Each skill in this repo follows the Claude Code skills format:
- `SKILL.md` — instructions loaded into Claude's context
- `resources/` — payloads, templates, checklists Claude references during testing
- `plugin.json` — metadata for the marketplace registry

## Skill Invocation

Skills are invoked with `/skill-name [target]`. Example:
```
/web-recon https://target.example.com
/api-security-audit https://api.example.com
/ai-llm-security https://chat.example.com
```
