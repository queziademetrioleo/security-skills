# web-recon

Performs full reconnaissance on a web application target before any active testing.

## What it does

1. Fetches HTTP response headers and identifies security header gaps
2. Downloads and analyzes JavaScript bundles for hardcoded secrets, API keys, endpoints and framework hints
3. Checks `robots.txt`, `sitemap.xml`, `/.well-known/`, common config file paths
4. Identifies cookies and their security flags
5. Maps all discovered URLs, API base paths and external domains
6. Fingerprints the tech stack (framework, CDN, cloud provider, auth system)

## Usage

```
/web-recon https://target.com
```

## Output

A structured reconnaissance report with:
- Technology stack identified
- All endpoints and API paths found
- Hardcoded secrets or keys found in JS
- Security header gaps
- Cookie security issues
- Recommended next skills to invoke
