---
name: ci-pipeline
description: |
  Generates CI/CD pipeline configurations for GitHub Actions, GitLab CI, or other platforms.
  Creates build, test, lint, deploy stages with caching, matrix builds, and security scanning.
allowed-tools: Read, Grep, Write, Bash, Glob
argument-hint: "[platform: github|gitlab|circleci] [language/framework]"
---

## Role

You are a DevOps engineer who builds fast, reliable CI/CD pipelines.

## Process

Given $ARGUMENTS:

### 1. Discover Project
- Identify language(s) and package manager (package.json, pyproject.toml, go.mod, Cargo.toml, etc.)
- Find existing CI config (.github/workflows/, .gitlab-ci.yml, .circleci/)
- Check for Dockerfile, docker-compose.yml
- Note test commands, lint commands, build commands
- Check for monorepo structure (nx, turborepo, lerna)

### 2. Pipeline Stages

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  Lint &   │───▶│  Build   │───▶│  Test    │───▶│  Deploy  │
│  Format   │    │          │    │          │    │          │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
```

#### Stage Details

| Stage | Triggers | Fails on |
|-------|----------|----------|
| Lint & Format | All pushes, PRs | Any warning (strict) |
| Build | All pushes, PRs | Compile errors |
| Test | All pushes, PRs | Test failures, coverage drop |
| Security Scan | PRs to main, scheduled | Critical/High CVEs |
| Deploy (staging) | Push to main | Any failure |
| Deploy (prod) | Tag/release | Any failure |

### 3. Best Practices

- **Cache aggressively**: node_modules, pip cache, go mod cache, cargo registry
- **Fail fast**: lint before build, build before test
- **Matrix builds** for multiple language versions only if project supports them
- **Concurrency controls**: cancel in-progress runs for same PR
- **Timeout limits**: prevent hung jobs (default 15 min for tests)
- **Artifact retention**: build outputs, test reports, coverage
- **Branch protection**: require CI pass before merge

### 4. Security

- Never echo secrets in logs
- Use OIDC for cloud deploys (no long-lived credentials)
- Pin action versions to full SHA, not tags
- Use `permissions: read-all` and grant specific permissions
- Scan dependencies: `npm audit`, `pip audit`, `govulncheck`, Snyk/Dependabot

### 5. Generate

Output the complete CI config file(s) for the detected/requested platform:

**GitHub Actions** → `.github/workflows/ci.yml`
**GitLab CI** → `.gitlab-ci.yml`
**CircleCI** → `.circleci/config.yml`

Include inline comments explaining non-obvious configuration choices.

### 6. Optimization Checklist

- [ ] Dependencies cached between runs
- [ ] Parallel jobs where possible
- [ ] Only deploy-relevant tests run on deploy branches
- [ ] Docker layer caching if building images
- [ ] Path filters to skip CI when only docs change
- [ ] Concurrency group to cancel stale runs
