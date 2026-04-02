# OpenSpec — Concepts and Reference

## Philosophy

OpenSpec is built around four principles:

```
fluid not rigid       — no phase gates, work on what makes sense
iterative not waterfall — learn while building, refine as you go
easy not complex      — light setup, minimal ceremony
brownfield-first      — works with existing codebases, not just greenfield
```

## High-Level Structure

```
openspec/
├── specs/           # Source of truth — how the system works now
│   ├── auth/
│   │   └── spec.md
│   └── payments/
│       └── spec.md
└── changes/         # Proposed modifications (one folder per change)
    └── add-oauth/
        ├── spec.delta.md
        ├── design.md
        └── tasks.md
```

## spec.md Format

```markdown
# {Domain} Specification

## Purpose
High-level description of the domain.

## Requirements

### Requirement: {Name}
The system SHALL/MUST/SHOULD {observable behavior}.

#### Scenario: {Case}
- GIVEN {precondition}
- WHEN {action}
- THEN {result}
- AND {additional result}
```

## RFC 2119 Keywords

| Keyword | Meaning |
|---------|-------------|
| MUST / SHALL | Absolute requirement — no exceptions |
| SHOULD | Recommended — justified exceptions exist |
| MAY | Optional |
| MUST NOT / SHALL NOT | Absolutely prohibited |

## What a Spec Is (and Isn't)

**Is:** behavior contract  
**Is NOT:** implementation plan

✅ Externally observable behavior  
✅ Inputs, outputs, error conditions  
✅ Constraints (security, privacy, performance)  
✅ Testable scenarios (Given/When/Then)

❌ Internal class/function names  
❌ Library/framework choices  
❌ Implementation details  
❌ Step-by-step execution plans

## Rigor Levels

### Lite (default — use in most cases)
- Short requirements focused on behavior
- Clear scope and non-scope
- Some concrete acceptance scenarios

### Full (only for high-risk changes)
- Cross-team or cross-repo changes
- API/contract changes
- Data migrations
- Security/privacy requirements
- Where ambiguity causes expensive rework

## Changes vs Specs

**Specs** (`openspec/specs/`) = current source of truth  
**Changes** (`openspec/changes/`) = proposed modifications

A change can contain:
- `spec.delta.md` — what changes in the spec
- `design.md` — implementation decisions
- `tasks.md` — concrete work to do

When a change is archived (merged), its deltas are incorporated into the specs.
