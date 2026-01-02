# my-claude-skills

> A collection of useful Claude Code skills I use for work.


## 🧩 Skills Overview

| Skill | Description | Use Cases |
|-------|-------------|-----------|
| [**🏗️ architect**](skills/architect/SKILL.md) | Generates Mermaid.js architecture diagrams when creating or refactoring modules. Keeps visual documentation in sync with code changes. | New modules, API design, system refactoring, documenting data flows |
| [**🔍 code-reviewer**](skills/code-reviewer/SKILL.md) | Comprehensive code review combining quality, security, and maintainability checks. | PR reviews, feature completion, refactoring, bug fixes |
| [**📝 create-pr**](skills/create-pr/SKILL.md) | Creates a pull request for the current branch with auto-generated concise title and description. | Ready to merge, finishing work, PR creation automation |
| [**✨ kiss-check**](skills/kiss-check/SKILL.md) | Complexity check that forces justification for complex solutions. Before proposing Design Patterns or abstractions, must explain why a simpler approach won't work. | Preventing overengineering, design reviews, architecture decisions |
| [**📋 rfc-generator**](skills/rfc-generator/SKILL.md) | Creates comprehensive RFC (Request for Comments) documents for new features. Guides the user create the document through interactive questioning. | Major features, architecture changes, team proposals |
| [**🔒 security-analysis**](skills/security-analysis/SKILL.md) | Identifies security vulnerabilities in backend code and analyzes security reports. Use for PR security reviews, vulnerability assessments, or analyzing external bug bounty reports against the codebase. | Security audits, PR security checks, bug bounty triage |
| [**📐 specs-generator**](skills/specs-generator/SKILL.md) | Creates comprehensive SPECS.md specification files for new features. Use when the user wants to create specs, specifications, feature documentation, or design documents. | Feature specs, implementation docs, design documentation |
| [**🧪 test-generator**](skills/test-generator/SKILL.md) | Generates unit and integration tests for modified/added code following existing project patterns. | New features, bug fixes, improving test coverage |

## ⚙️ How Skills Work

Skills are modular, reusable instruction sets that Claude Code can invoke to perform complex tasks with consistent quality. Each skill:

- ✅ Has a clear trigger condition
- ✅ Follows a structured workflow
- ✅ Uses specific tools (Read, Grep, Glob, Bash, etc.)
- ✅ Produces predictable output
- ✅ Can reference external documentation

### Anatomy of a Skill

```
skills/
├── architect/
│   └── SKILL.md        # Main instructions
├── code-reviewer/
│   └── SKILL.md
└── security-analysis/
    ├── SKILL.md
    └── references/     # Supporting docs
        ├── owasp-api-top-10.md
        └── severity-assessment.md
```



## 💡 Workflow ideas

- **Start simple**: Use `kiss-check` before architecting complex solutions
- **Document early**: Invoke `architect` when designing new modules
- **Test proactively**: `test-generator` automatically suggests tests for modified code
- **Review thoroughly**: Run `code-reviewer` before merging
- **Plan ahead**: Use `specs-generator` or `rfc-generator` for major features
- **Stay secure**: Always run `security-analysis` on PRs
