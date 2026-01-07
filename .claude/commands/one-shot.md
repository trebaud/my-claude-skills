---
description: Autonomously deliver a complete feature from idea to PR
argument-hint: [feature-description]
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Skill
---

# One-Shot Feature Delivery

Autonomous SDLC orchestrator - executes all stages without stopping, auto-fixes aggressively.

## Core Principles

- Full autonomy - no approval needed
- Aggressive auto-fixes (max 2 attempts per issue)
- Permissive execution - continue through all stages
- Always regenerate docs based on current code
- Work on current branch only
- Detailed logging of all actions

## Context

Input: $ARGUMENTS
```bash
!`git status`
!`git branch --show-current`
!`git diff --name-only`
!`git diff --staged --name-only`
!`git log --oneline -5`
```

---

## Stage 1: Planning

Generate SPECS.md and RFC:

1. Call `specs-generator` skill - interactive questioning
2. Call `rfc-generator` skill - include security analysis
3. Always regenerate if exists
4. Save to working directory

**Auto-fix**: Missing/outdated/incomplete → regenerate

**Log**:
```
### Stage 1: Planning
Status: [Complete]
Actions: Generated SPECS.md, RFC-[name].md
Issues: [N]
Artifacts: SPECS.md, RFC-[name].md
```

---

## Stage 2: Design

Generate diagrams and KISS validation:

1. Call `architect` skill - generate Mermaid diagrams in ARCHITECTURE.md
2. Call `kiss-check` skill - validate simplicity
3. If KISS violations → call `refactoring-assistant` skill to simplify
4. Update ARCHITECTURE.md

**Auto-fix**: Missing diagrams → generate; Overengineered → refactor

**Log**:
```
### Stage 2: Design
Status: [Complete]
Actions: ARCHITECTURE.md created/updated, KISS check passed
Issues: [N]
- [Severity] [Issue]: Auto-fixed: [Yes/No]
Artifacts: ARCHITECTURE.md
```

---

## Stage 3: Development

Implement based on SPECS.md:

1. Analyze SPECS.md and RFC
2. Implement following existing patterns
3. Use `kiss-check` proactively
4. Auto-fix: lint/type/style/import errors
5. Commit incrementally

**Auto-fix**: Compilation/type/lint → fix immediately

**Log**:
```
### Stage 3: Development
Status: [Complete]
Actions: Implemented [N] files, [Y] commits
Issues: [N]
- [Severity] [Issue]: Auto-fixed: [Yes/No]
Artifacts: [file list]
```

---

## Stage 4: Testing

Generate and run tests:

1. Call `test-generator` skill for new/modified code
2. Call `test-runner` agent to execute tests
3. Auto-fix: setup/import/assertion errors (max 2 attempts)
4. Log all failures

**Auto-fix**: Missing tests → generate; Failures → attempt fix

**Log**:
```
### Stage 4: Testing
Status: [Complete]
Actions: [N] tests generated, results: [Passing: X, Failing: Y]
Issues: [N]
- [Test name] ([file:line]): Auto-fixed: [Yes/No]
Artifacts: [test files]
```

---

## Stage 5: Quality

Code review and security:

1. Call `code-reviewer` skill
2. Call `security-analysis` skill
3. Auto-fix: trivial/moderate issues (max 2 attempts each)
4. Log critical issues even if fixed

**Auto-fix**: Style/performance/security → attempt fix

**Log**:
```
### Stage 5: Quality
Status: [Complete]
Actions: Reviewed [N] files, security analysis done
Issues: [N]
- [Critical/Warning/Suggestion] [Issue]: Auto-fixed: [Yes/No] (attempts: X)
Artifacts: [security reports]
```

---

## Stage 6: Deploy

Create PR:

1. `git add .`
2. Create commit with descriptive message
3. Call `create-pr` skill - include diagram, migration guide, known issues
4. Verify PR created

**Auto-fix**: Missing docs → generate; PR fail → retry

**Log**:
```
### Stage 6: Deploy
Status: [Complete]
Actions: Committed changes, created PR #[number]
Issues: [N]
Artifacts: PR URL
```

---

## Final Report

```
# One-Shot Complete

## Summary
Feature: [name]
Duration: [X min]
Status: [Ready / Needs review / Blocking issues]

## Stage Results
| Stage | Status | Issues | Auto-Fixed | Manual Review |
|-------|--------|--------|------------|--------------|
| Planning | ✅/⚠️/❌ | [N] | [N] | Yes/No |
| Design | ✅/⚠️/❌ | [N] | [N] | Yes/No |
| Development | ✅/⚠️/❌ | [N] | [N] | Yes/No |
| Testing | ✅/⚠️/❌ | [N] | [N] | Yes/No |
| Quality | ✅/⚠️/❌ | [N] | [N] | Yes/No |
| Deploy | ✅/⚠️/❌ | [N] | [N] | Yes/No |

## Manual Intervention
[Issues needing manual review]
1. [Priority] [Issue] (Stage, file:line)
   - Description
   - Why auto-fix failed
   - Recommended action

## Artifacts
- SPECS.md
- RFC-[name].md
- ARCHITECTURE.md
- PR: #[number] - [url]
- [Files created/modified]

## Next Steps
1. Review manual intervention items
2. Test: git checkout main && git pull
3. Review PR at [url]
4. Merge when ready
```

---

## Auto-Fix Logic

```
for each issue:
  attempts = 0
  while attempts < 2:
    fix = attempt_fix(issue)
    if fix.success:
      log.success("Auto-fixed")
      break
    attempts += 1
  if attempts >= 2:
    log.warning("Auto-fix failed, manual review needed")
```

**Categories**:
- **Trivial** (100% auto-fix): formatting, imports, types
- **Moderate** (70-80%): performance, KISS, error handling
- **Critical** (50-60%): security patches, breaking changes
- **Never auto-fix**: business logic, migrations, config

---

## Execution Order

1. Planning → 2. Design → 3. Development → 4. Testing → 5. Quality → 6. Deploy → Report

**NEVER STOP between stages - ALWAYS CONTINUE**

## Success Criteria

✅ All stages attempted
✅ PR created
✅ Code compiles
✅ Tests run
✅ Docs generated
✅ All issues documented
