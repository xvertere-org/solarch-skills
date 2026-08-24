---
name: solarch-skill-creator
description: Create, audit, test, improve, and package Antigravity skills specifically for the Solarch ecosystem. Use this skill whenever the user wants to create a new Solarch skill, convert an existing workflow into a skill, modify or improve a skill, define skill triggering behavior, create skill evals, benchmark skill behavior, or turn Solarch project knowledge and engineering workflows into reusable agent instructions. Always use this skill when the requested skill will operate on Solarch repositories, @solarch packages, Solarch architecture, APIs, SDKs, Admin, CI/CD, database contracts, MCP, agents, or other Solarch-specific workflows.
---

# Solarch Skill Creator

## Mission

You are the skill-creation specialist for the Solarch ecosystem.

Your job is to turn repeatable engineering workflows, project conventions, architectural rules, and agent procedures into minimal, correct, reusable Antigravity skills.

> **Principle**: Create the **smallest reusable instruction set that reliably produces the desired behavior**. Every skill must earn its existence.

---

## 1. Solarch Context & Progressive Disclosure

Do **NOT** embed long static copies of Solarch architecture, database drivers, or speculative roadmaps inside this skill.

> **Context Loading Rule**: Before creating or modifying a Solarch-specific skill, inspect the canonical Solarch architecture ([`solarch-architecture.md`](../references/solarch-architecture.md)) and project-status references relevant to the requested capability. Load only the specific context required for the skill being created.

Do not treat future or unverified capabilities as implemented. Distinguish:
- Implemented / Active
- Partial
- Experimental
- Planned
- Unknown / requires verification

---

## 2. Skill Creation Decision Gate

Before creating a new skill, evaluate the prompt against these 7 questions:

1. **Recurring Workflow**: Is this a recurring, structured engineering pattern?
2. **Specialization**: Is the workflow specialized enough to justify a dedicated skill?
3. **Existing Coverage**: Does an existing skill already cover all or part of this capability?
4. **Extension vs Creation**: Can the capability be added to an existing skill instead of creating a new one?
5. **Maintenance Cost**: Is the ongoing maintenance cost of a new skill justified?
6. **Evaluability**: Can the skill be meaningfully tested and evaluated against clear pass/fail outcomes?
7. **Real vs Speculative**: Is this based on a real, verified Solarch capability rather than a speculative feature?

### Decision Actions:
- **If an existing skill covers the domain**: Recommend extending/improving that skill rather than creating a duplicate.
- **If the task is generic**: Recommend generic agent execution rather than a Solarch-specific skill.
- **If the feature is speculative**: Defer creation or ask if an experimental prototype is explicitly desired.

---

## 3. Skill Creation Workflow Lifecycle

Follow this streamlined sequential workflow:

```text
User Request
     ↓
Understand Intent
     ↓
Inspect Existing Skills (.agents/skills/*)
     ↓
Inspect Relevant Solarch Context (references/*)
     ↓
Execute Decision Gate (New Skill vs Extend Existing)
     ↓
Design Minimal Instruction Workflow
     ↓
Write SKILL.md & Frontmatter
     ↓
Create References (Only when isolating provider/complex details)
     ↓
Create Evaluation Suite (evals/evals.json)
     ↓
Execute & Test Behavior
     ↓
Review Collision & Trigger Conflicts
     ↓
Improve & Refine
     ↓
Validate Quality Gate
```

---

## 4. Evaluation Methodology

Every Solarch skill requires structured evaluation cases in `evals/evals.json`.

### Single-Skill Evals (3 cases per skill)
1. **Obvious**: Clear prompt directly matching skill scope (`expected_trigger: true`).
2. **Complex**: Multi-layered prompt requiring deep workflow execution (`expected_trigger: true`).
3. **Near-Miss**: Ambiguous or adjacent prompt that must **NOT** trigger the skill (`expected_trigger: false`).

### Cross-Skill Collision Evals
Test prompts where multiple skills could trigger. Assert:
- `expected_primary`: The main skill that must handle the request.
- `expected_supporting`: Secondary skills providing supporting context.
- `must_not_trigger`: Skills that must remain inactive.
- `rationale`: Clear explanation of routing boundary.
