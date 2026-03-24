---
name: security-analysis
description: Identifies security vulnerabilities in backend code using OWASP API Security Top 10. Use when reviewing PRs for security, performing vulnerability assessments, analyzing bug bounty reports against the codebase, or when the code-reviewer skill needs a security check.
allowed-tools: Read, Grep, Glob, WebFetch
---

# Security Analysis

Identify security vulnerabilities in the backend codebase using OWASP API Security Top 10 as the classification framework.

## Core Principles

**Business Impact Drives Severity**: Technical CVSS scores are starting points. Apply business impact adjustment to get FINAL SEVERITY. High CVSS with zero business damage = LOW severity.

**Verify in Code**: Don't trust theoretical claims. Locate vulnerabilities in the codebase, trace data flow, and assess realistic impact.

**Reality Check**: Can attacker actually gain financially? Does this cause real operational problems? Would users notice?

## Reference Documents

**For OWASP classification**: See @references/owasp-api-top-10.md
**For severity scoring**: See @references/severity-assessment.md

## Security Review

Focus on HIGH confidence vulnerabilities only.

### Process
1. Identify user-controlled input in the diff
2. Trace data flow to sensitive operations
3. Verify vulnerability exists (not just potential)
4. Check against OWASP API Top 10

### Output Format
```
🔒 SECURITY ISSUE: [Vulnerability Type]

[Brief description]

Steps to reproduce: [How to exploit]
Recommended fix: [Specific change]
```
