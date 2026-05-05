# Johndesi Ventures — Company Website

[![Deploy to S3 + CloudFront](https://github.com/bankolejohn/johndesi_website_s3/actions/workflows/deploy.yml/badge.svg)](https://github.com/bankolejohn/johndesi_website_s3/actions/workflows/deploy.yml)

Production website for [Johndesi Ventures](https://johndesiventures.website) — a DevOps and cloud infrastructure consultancy specialising in autonomous pipelines, AI-driven workflows, and spec-driven architecture.

---

## Live Site

**[https://johndesiventures.website](https://johndesiventures.website)**

---

## Architecture

```
GitHub (source) ──► GitHub Actions (CI/CD) ──► S3 (storage) ──► CloudFront (CDN + HTTPS)
                                                                        │
                                                         johndesiventures.website (Namecheap DNS)
```

| Layer | Service | Purpose |
|-------|---------|---------|
| Source control | GitHub | Version control and CI/CD trigger |
| CI/CD | GitHub Actions + OIDC | Keyless deployment via IAM role assumption |
| Storage | AWS S3 | Static file hosting |
| CDN | AWS CloudFront | Global edge delivery + HTTPS termination |
| TLS | AWS ACM | SSL/TLS certificate (DNS validated) |
| DNS | Namecheap | Domain management → CloudFront CNAME |

---

## Repository Structure

```
.
├── index.html                      # Single-page website (HTML + CSS + JS inline)
├── .github/
│   └── workflows/
│       └── deploy.yml              # CI/CD pipeline — deploys on push to main
├── .gitignore
└── README.md
```

---

## CI/CD Pipeline

Every push to `main` automatically:

1. Syncs `index.html` to the S3 bucket
2. Creates a CloudFront invalidation to purge the cache

### Authentication — OIDC (no stored keys)

The pipeline uses **OpenID Connect (OIDC)** to authenticate with AWS — no long-lived access keys are stored in GitHub Secrets.

How it works:
- GitHub mints a short-lived JWT token scoped to this repository and workflow run
- AWS IAM validates the token against the registered GitHub OIDC provider
- The workflow assumes the `GitHubActions-JohndesiVentures-Deploy` IAM role
- Temporary credentials are issued for the duration of the job only

The IAM role is locked to this repository via a trust policy condition:
```json
"token.actions.githubusercontent.com:sub": "repo:bankolejohn/johndesi_website_s3:*"
```

### IAM Role Permissions

The role has minimal, least-privilege permissions:

```
s3:PutObject        → arn:aws:s3:::johndesiventures.website/*
s3:DeleteObject     → arn:aws:s3:::johndesiventures.website/*
s3:ListBucket       → arn:aws:s3:::johndesiventures.website
cloudfront:CreateInvalidation → distribution/E1IZF9XNUW1C2A
```

Nothing else. No IAM access, no EC2, no other buckets.

---

## Local Development

No build step required. Open directly in a browser:

```bash
open index.html
```

Or serve locally with Python:

```bash
python3 -m http.server 8080
# visit http://localhost:8080
```

---

## Deployment

Deployment is fully automated via the CI/CD pipeline. To deploy manually:

```bash
# Upload to S3
aws s3 sync . s3://johndesiventures.website \
  --exclude "*" \
  --include "index.html" \
  --content-type "text/html" \
  --cache-control "no-cache, no-store, must-revalidate"

# Invalidate CloudFront cache
aws cloudfront create-invalidation \
  --distribution-id E1IZF9XNUW1C2A \
  --paths "/*"
```

---

## Infrastructure

| Resource | Value |
|----------|-------|
| S3 Bucket | `johndesiventures.website` (us-east-1) |
| CloudFront Distribution | `E1IZF9XNUW1C2A` |
| CloudFront Domain | `d38a6nr1wygrfc.cloudfront.net` |
| ACM Certificate | us-east-1 (DNS validated) |
| IAM Role | `GitHubActions-JohndesiVentures-Deploy` |

---

## Contact

**John Bankole** — Founder & Principal DevOps Engineer  
[LinkedIn](https://www.linkedin.com/in/john-bankole/) · [GitHub](https://github.com/bankolejohn) · [admin@johndesiventures.website](mailto:admin@johndesiventures.website)
