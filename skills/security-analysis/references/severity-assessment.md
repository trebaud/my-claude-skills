# Severity Assessment Framework

This document covers CVSS v3.1 scoring and business impact adjustment rules.

## Step 1: Calculate Technical CVSS v3.1 Score

### Base Score Metrics

**Attack Vector (AV)**
- Network (N): Exploitable over network - 0.85
- Adjacent (A): Requires adjacent network access - 0.62
- Local (L): Requires local access - 0.55
- Physical (P): Requires physical access - 0.2

**Attack Complexity (AC)**
- Low (L): No special conditions needed - 0.77
- High (H): Requires specific conditions - 0.44

**Privileges Required (PR)**
- None (N): No authentication needed - 0.85
- Low (L): Basic user privileges - 0.62 (0.68 if Scope changed)
- High (H): Admin privileges needed - 0.27 (0.50 if Scope changed)

**User Interaction (UI)**
- None (N): No user action needed - 0.85
- Required (R): Requires user action - 0.62

**Scope (S)**
- Unchanged (U): Impact limited to vulnerable component
- Changed (C): Impact extends beyond vulnerable component

**Impact Metrics (C/I/A)**
- None (N): No impact - 0.0
- Low (L): Limited impact - 0.22
- High (H): Serious impact - 0.56

### CVSS Score Ranges

| Score | Severity |
|-------|----------|
| 9.0-10.0 | CRITICAL |
| 7.0-8.9 | HIGH |
| 4.0-6.9 | MEDIUM |
| 0.1-3.9 | LOW |
| 0.0 | NONE |

### Documentation Template

```
Technical Assessment:
- CVSS Base Score: [0.0-10.0]
- CVSS Vector: CVSS:3.1/AV:[N|A|L|P]/AC:[L|H]/PR:[N|L|H]/UI:[N|R]/S:[U|C]/C:[N|L|H]/I:[N|L|H]/A:[N|L|H]
- Technical Severity: [CRITICAL|HIGH|MEDIUM|LOW]
- Attack Complexity: [LOW|HIGH]
- Privileges Required: [NONE|LOW|HIGH]
- Exploitability: [description]
```

---

## Step 2: Apply Business Impact Adjustment

**CRITICAL: Business impact HEAVILY influences final severity. A high CVSS score with zero business impact should be downgraded.**

### Business Impact Adjustment Rules

#### 1. NONE/Negligible Business Damage → Downgrade by 1-2 severity levels
Applies when:
- Zero financial loss
- Zero operational disruption
- Only affects attacker's own account
- No user impact
- No reputational risk

Example: A race condition that lets users see their own pending transaction twice has CVSS 7.0 but NONE business impact → Downgrade to LOW.

#### 2. LOW Business Damage → Downgrade by 1 severity level
Applies when:
- Minimal financial exposure (<$1,000)
- Minor operational inconvenience
- Limited user impact
- Easily recoverable

Example: Information disclosure of non-sensitive metadata, minor UI inconsistencies.

#### 3. MODERATE Business Damage → Keep CVSS severity
Applies when:
- Measurable financial risk ($1k-$50k)
- Moderate operational impact
- Affects multiple users
- Requires investigation/remediation effort

#### 4. HIGH Business Damage → Keep or upgrade CVSS severity
Applies when:
- Significant financial risk (>$50k)
- Major operational disruption
- Widespread user impact
- Compliance violations (GDPR, PCI-DSS)
- Reputational damage potential

---

## Step 3: Business Impact Analysis Categories

### Data Impact
- What data is at risk? (PII, payment data, credentials)
- How many users/accounts could be affected?
- What is the data sensitivity level?
- Are there regulatory/compliance implications?
- **Reality check:** Is data actually corrupted/exposed or just inconsistent responses?

### Financial Impact
- Could this lead to direct financial loss?
- What is the potential loss amount?
- Could funds be stolen or misdirected?
- Are payment processors or financial integrations affected?
- **Reality check:** Can attacker gain financially or is it just their own account affected?

### Operational Impact
- Could this disrupt service availability?
- Would this require emergency maintenance?
- How many engineer-hours to fix?
- Does this affect core business flows (purchases, withdrawals, deposits)?
- **Reality check:** Does this cause real operational problems or just edge case inconsistencies?

### Reputation Impact
- Could this be exploited at scale?
- Would exploitation be visible to users?
- Could this damage trust or brand reputation?
- Is this likely to be disclosed publicly?
- **Reality check:** Would users even notice this issue?

### User Impact
- How many users are affected?
- What is the severity of impact per user?
- Could users lose money or access?
- Would users need to take action (password resets, etc.)?
- **Reality check:** Does end state differ from expected state for the user?

### Abuse Potential
- Can this be automated/scripted?
- What is the effort required to exploit?
- Could attackers monetize this vulnerability?
- **Reality check:** What does the attacker actually gain?

### Documentation Template

```
BUSINESS IMPACT ANALYSIS:
- Data at risk: [type and volume - be specific about actual vs theoretical risk]
- Affected users: [number or percentage]
- Financial risk: [estimated amount or NONE/LOW/MEDIUM/HIGH - must justify]
- Operational disruption: [NONE/MINOR/MODERATE/SEVERE - be realistic]
- Compliance impact: [regulations affected or NONE]
- Reputation risk: [LOW/MEDIUM/HIGH/CRITICAL - consider if users would notice]
- Abuse potential: [LOW/MEDIUM/HIGH - what does attacker actually gain?]
- Already exploited: [YES/NO/UNKNOWN - check logs]
- **REALITY CHECK:** [One sentence summary of actual business damage]
```

---

## Step 4: Determine Final Severity

### Final Severity Matrix

| Technical CVSS | Business Impact | FINAL SEVERITY |
|----------------|-----------------|----------------|
| CRITICAL (9-10) | HIGH | CRITICAL |
| CRITICAL (9-10) | MODERATE | HIGH |
| CRITICAL (9-10) | LOW | MEDIUM |
| CRITICAL (9-10) | NONE | LOW |
| HIGH (7-8.9) | HIGH | HIGH |
| HIGH (7-8.9) | MODERATE | HIGH |
| HIGH (7-8.9) | LOW | MEDIUM |
| HIGH (7-8.9) | NONE | LOW |
| MEDIUM (4-6.9) | HIGH | HIGH |
| MEDIUM (4-6.9) | MODERATE | MEDIUM |
| MEDIUM (4-6.9) | LOW | LOW |
| MEDIUM (4-6.9) | NONE | LOW |
| LOW (0.1-3.9) | Any | LOW |

### Documentation Template

```
SEVERITY ASSESSMENT:

Technical Assessment:
- CVSS Base Score: [score]
- CVSS Vector: [vector string]
- Technical Severity: [CRITICAL/HIGH/MEDIUM/LOW]

Business Impact Adjustment:
- Financial Impact: [NONE/LOW/MEDIUM/HIGH - with justification]
- Operational Impact: [NONE/MINOR/MODERATE/SEVERE]
- User Impact: [NONE/LOW/MEDIUM/HIGH]
- Reputational Risk: [NONE/LOW/MEDIUM/HIGH]
- Business Damage: [One sentence reality check]

FINAL SEVERITY: [CRITICAL/HIGH/MEDIUM/LOW]
Rationale: [Explain why business impact adjusted the technical severity up/down]
```

---

## Step 5: Priority and Timeline

Based on **FINAL SEVERITY** (business-adjusted), not technical CVSS:

| FINAL Severity | Priority | Timeline | When to Use |
|----------------|----------|----------|-------------|
| CRITICAL | IMMEDIATE | 24-48 hours | Active exploitation, data breach in progress, significant financial loss occurring |
| HIGH | HIGH PRIORITY | 1 week | Moderate-to-high business impact, potential for significant damage |
| MEDIUM | MEDIUM PRIORITY | 2-4 weeks | Moderate business impact, limited real-world exploitation potential |
| LOW | LOW PRIORITY | Next sprint | Minimal/zero business impact, theoretical vulnerabilities |

### Priority Documentation Template

```
REMEDIATION PRIORITY:

Priority: [IMMEDIATE/HIGH/MEDIUM/LOW]
Timeline: [24-48h / 1 week / 2-4 weeks / Next sprint]

Justification:
- FINAL Severity: [severity]
- Key business factors: [list main reasons for this priority]
- Risk if delayed: [what could happen if not fixed in timeline]
```

---

## Common Overstatement Patterns

Researchers often overstate these impacts. Always verify with code evidence:

1. **"Financial loss"** - Often only affects attacker's own account
2. **"Data breach"** - Often just information disclosure with no real impact
3. **"Ledger inconsistencies"** - Often cosmetic or self-correcting
4. **"Unauthorized access"** - May be to non-sensitive data
5. **"Account takeover"** - May require unrealistic attack conditions
6. **"Race condition exploitation"** - May have no meaningful outcome
7. **"Bypass authentication"** - May only bypass to public areas

Always ask: **"What can the attacker actually DO with this, and what BUSINESS DAMAGE results?"**
