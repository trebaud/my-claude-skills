# My Agent Config

> A collection of useful agent skills and subagents I use for work.


## 🧩 Skills Overview

| Skill | Description | Use Cases |
|-------|-------------|-----------|
| [**🏗️ architect**](skills/architect/SKILL.md) | Generates Mermaid.js architecture diagrams for modules and refactors. Keeps visual docs in sync. | New modules, API design, refactoring, data flows |
| [**🔍 code-reviewer**](skills/code-reviewer/SKILL.md) | Comprehensive code review covering quality, security, and maintainability. | PR reviews, feature completion, refactoring, bug fixes |
| [**📝 create-pr**](skills/create-pr/SKILL.md) | Creates pull requests with auto-generated title and description. | Ready to merge, finishing work, PR automation |
| [**✨ kiss-check**](skills/kiss-check/SKILL.md) | Forces justification for complex solutions. Must explain why simpler won't work. | Preventing overengineering, design reviews, architecture decisions |
| [**📋 rfc-generator**](skills/rfc-generator/SKILL.md) | Creates RFC documents for new features through interactive questioning. | Major features, architecture changes, team proposals |
| [**🔒 security-analysis**](skills/security-analysis/SKILL.md) | Identifies security vulnerabilities and analyzes security reports. | Security audits, PR checks, bug bounty triage |
| [**📐 specs-generator**](skills/specs-generator/SKILL.md) | Creates comprehensive SPECS.md files for new features and design docs. | Feature specs, implementation docs, design documentation |
| [**🧪 test-generator**](skills/test-generator/SKILL.md) | Generates unit and integration tests following existing patterns. | New features, bug fixes, improving test coverage |
| [**🔧 refactoring-assistant**](skills/refactoring-assistant/SKILL.md) | Performs safe refactoring with automated test verification. | KISS violations, overengineered code, duplicate code |
| [**🐛 debug**](skills/debug/SKILL.md) | Test-first bug debugging system. Creates reproducing tests, then uses subagents to implement fixes proven by passing tests. | Bug reports, unexpected behavior, regression issues |
| [**🎯 interview**](skills/interview/SKILL.md) | Interviews about implementation plans by asking non-obvious technical questions about decisions, tradeoffs, and constraints. | Plan validation, requirement clarification, design reviews |


## 🤖 Agents Overview

| Agent | Description | Use Cases |
|-------|-------------|-----------|
| [**🧪 test-runner**](agents/test-runner.md) | Runs tests and provides failure analysis without attempting fixes. | Running tests, analyzing test failures, debugging test suites |
| [**📊 kiss-enforcer**](agents/kiss-enforcer.md) | Enforces simplicity using the kiss-check skill. Identifies overengineering and ensures complexity is only necessary. | Code reviews, architecture decisions, preventing overengineering |



## 🚀 Skills & Agents in the SDLC

Skills and agents seamlessly integrate into your software development lifecycle, automating routine tasks and ensuring quality at every stage. Here's how they collaborate to supercharge your workflow:

```mermaid
flowchart LR
    A[Planning<br/>specs-generator, rfc-generator, interview] --> B[Design<br/>architect, kiss-check, interview]
    B --> C[Development<br/>kiss-check, refactoring-assistant]
    C --> D[Testing<br/>test-generator, test-runner, debug]
    D --> E[Review & Deploy<br/>code-reviewer, security-analysis, create-pr]
    E --> F[Maintenance<br/>security-analysis, code-reviewer, debug]
    
    G[/one-shot command/] -.-> A
    G -.-> B
    G -.-> C
    G -.-> D
    G -.-> E
```

- **Planning**: Use `specs-generator` for new features and `interview` to validate implementation plans.
- **Design**: Run `architect` early to visualize your system, use `interview` to clarify requirements, and `kiss-check` to challenge complexity.
- **Building**: Let `kiss-check` challenge your complex ideas—simpler is often better.
- **Refactoring**: Use `refactoring-assistant` to simplify overengineered code safely.
- **Testing**: Pair `test-generator` with `test-runner` and `debug` for comprehensive testing workflows.
- **Shipping**: Deploy with `code-reviewer` and `security-analysis` to catch issues before production.
- **Maintaining**: Schedule regular `security-analysis` runs and use `debug` for systematic bug resolution.


---

## 🎢 One-Shot Command

| Command | Description | Use Cases |
|---------|-------------|-----------|
| [**one-shot**](commands/one-shot.md) | Autonomously deliver a complete feature from idea to PR. Executes all SDLC stages with auto-fixes. | New features, end-to-end automation, rapid prototyping |

> 💡 `/one-shot [feature description]` automates the entire SDLC from idea to PR. It's fast, powerful, and might be plotting world domination while you're not looking. Just kidding (probably). Use responsibly! 🤖
