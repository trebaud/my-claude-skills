---
name: kiss-check
description: Complexity check that forces justification for complex solutions. Before proposing Design Patterns or abstractions, must explain why a simpler approach won't work.
tools: Read, Grep, Glob, AskUserQuestion
---

# The Refuse-to-Overengineer

A behavioral skill to keep codebases simple and maintainable. Forces complexity justification before proposing sophisticated solutions.

## Core Philosophy

> "The simplest solution that works is usually the best."

Every abstraction, pattern, or framework adds cognitive load. This skill ensures complexity is introduced only when genuinely necessary.

## Automatic Triggers

This skill activates when Claude is about to suggest:

- Design patterns (Factory, Strategy, Observer, etc.)
- Abstract base classes
- Generic type parameters beyond basic usage
- Service layers or repositories
- Event-driven architectures
- Plugin systems
- Configuration frameworks
- Custom DSLs or builders

## Complexity Check Protocol

Before proposing ANY complex solution, complete this checklist:

### 1. Simple Alternative Analysis

**REQUIRED**: First propose the simplest possible solution:

```markdown
## Simplest Approach
[Describe the straightforward if/else, direct function, or inline solution]

**Lines of code**: ~[X]
**New files**: [0 or N]
**New abstractions**: [0 or N]
```

### 2. Justify Complexity

Only proceed with complexity if you can answer YES to at least one:

| Question | If YES, complexity may be warranted |
|----------|-------------------------------------|
| Will this exact pattern be reused 3+ times? | Creates genuine DRY benefits |
| Does the simple version have a concrete bug risk? | Not theoretical, actual bugs |
| Is there a measurable performance requirement? | With specific metrics |
| Does existing codebase already use this pattern? | Consistency matters |

### 3. Complexity Cost Statement

For any non-trivial solution, state:

```markdown
## Complexity Cost
- **Files added**: [N]
- **Lines of code**: [X] (vs [Y] for simple version)
- **Concepts to understand**: [List new abstractions]
- **Future maintenance**: [Who needs to understand this?]
```

## Red Flags to Challenge

When you see yourself writing these, STOP and reconsider:

| Red Flag | Question to Ask |
|----------|-----------------|
| `interface I...` | "Do I have 2+ implementations right now?" |
| `abstract class` | "Is this inheritance necessary today?" |
| `Factory`, `Builder` | "Is `new Thing()` insufficient?" |
| `Strategy pattern` | "Will a simple if/else work?" |
| `Event emitters` | "Can I just call the function directly?" |
| `Config objects` | "Can I use constants or params?" |
| `Generic<T>` | "Is this actually used with multiple types?" |
| `Service layer` | "Am I just wrapping a function?" |

## Decision Framework

```
                    ┌─────────────────────┐
                    │ Need to add feature │
                    └─────────┬───────────┘
                              │
                    ┌─────────▼───────────┐
                    │ Can I do it with    │
                    │ if/else or direct   │───YES──▶ Do that.
                    │ function call?      │
                    └─────────┬───────────┘
                              │ NO
                    ┌─────────▼───────────┐
                    │ Will this pattern   │
                    │ be reused 3+ times  │───NO───▶ Do the simple thing.
                    │ in existing code?   │
                    └─────────┬───────────┘
                              │ YES
                    ┌─────────▼───────────┐
                    │ Does codebase       │
                    │ already have this   │───YES──▶ Use existing pattern.
                    │ pattern?            │
                    └─────────┬───────────┘
                              │ NO
                    ┌─────────▼───────────┐
                    │ State complexity    │
                    │ cost and get user   │
                    │ approval first      │
                    └─────────────────────┘
```

## Output Format When Complexity Is Proposed

```markdown
## Solution Proposal

### Simple Version (Recommended)
```[language]
// Direct, inline solution
```
**Pros**: Easy to understand, no new abstractions
**Cons**: [Any actual drawbacks]

### Complex Version (Alternative)
```[language]
// Pattern-based solution
```
**Pros**: [Genuine benefits]
**Cons**: More code, new concepts, maintenance burden

### Why Complex Might Be Needed
- [ ] Reused 3+ times: [Yes/No - where?]
- [ ] Concrete bug risk in simple: [Yes/No - what?]
- [ ] Performance requirement: [Yes/No - what metric?]
- [ ] Matches existing patterns: [Yes/No - where?]

### Recommendation
[Simple/Complex] because [specific reason]
```

## Examples

### Bad: Overengineered
```typescript
// For ONE use case:
interface PaymentProcessor { process(amount: number): Promise<void>; }
class StripeProcessor implements PaymentProcessor { ... }
class PaymentProcessorFactory { create(type: string): PaymentProcessor { ... } }
```

### Good: Simple
```typescript
// For ONE use case:
async function processStripePayment(amount: number): Promise<void> {
  // Direct implementation
}
```

### When Complex IS Right
```typescript
// Already have 4 payment processors with shared interface
// Adding a 5th, pattern is established
class PayPalProcessor implements PaymentProcessor { ... }
```

## Remember

- Three similar lines of code > one premature abstraction
- Delete code is better than clever code
- Future requirements are imaginary until they arrive
- "What if we need to..." is not a valid reason
