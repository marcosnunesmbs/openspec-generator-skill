---
name: openspec-generator
description: Analyze an existing project and generate or update OpenSpec spec artifacts. Use this skill whenever the user wants to create or update specs for a project using OpenSpec, initialize an openspec/ folder, document system behavior, generate spec.md files from existing code, map features to requirements and scenarios, audit existing specs for gaps, or identify new domains not yet covered. Trigger this skill when the user mentions "openspec", "create specs", "generate specifications", "update specs", "audit specs", "document system behavior", "initialize openspec", "specs missing", or asks to analyze a codebase and produce structured requirements. Also trigger for brownfield projects where the user wants to capture existing behavior formally.
---

# OpenSpec Generator

Skill for analyzing an existing project and generating or updating specification artifacts following the OpenSpec structure and philosophy.

## OpenSpec Philosophy (Summary)

```
fluid not rigid       — no phase gates, work on what makes sense
iterative not waterfall — learn while building
easy not complex      — light setup, minimal ceremony
brownfield-first      — works with existing codebases
```

**Specs** describe how the system **currently behaves** (source of truth).  
**Changes** are proposed modifications, living in separate folders until merged.

---

## Generation Process

### Step 0 — Check Preconditions and Determine Mode (REQUIRED)

**First**, check the project state to determine the **operating mode**:

```bash
# 1. Check if openspec/ folder exists
[ ! -d "openspec" ] && echo "ERROR: openspec/ not found" && exit 1

# 2. Check if openspec/specs/ exists
[ ! -d "openspec/specs" ] && echo "ERROR: openspec/specs/ not found" && exit 1

# 3. Count existing specs and list covered domains
SPECS_COUNT=$(find openspec/specs -name "spec.md" | wc -l)
echo "Specs found: $SPECS_COUNT"
find openspec/specs -name "spec.md" | sort
```

**Verification result determines the mode:**

| Situation | Mode | Action |
|-----------|------|--------|
| `openspec/` doesn't exist | ❌ Blocked | Inform: "Initialize OpenSpec first by creating `openspec/` and `openspec/specs/`." |
| `openspec/specs/` doesn't exist | ❌ Blocked | Inform: "Create the `openspec/specs/` folder before continuing." |
| `openspec/specs/` **empty** | 🟢 **Initial Mode** | Proceed to Step 1 — generate specs from scratch |
| `openspec/specs/` **has specs** | 🔵 **Audit Mode** | Proceed to Step 1-A — audit and expand |

---

### Step 1 — Explore the Project (Initial Mode)

> **Skip to Step 1-A if in Audit Mode.**

Explore the project to identify all domains:

```bash
# Relevant file structure
find . -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" \) \
  | grep -v node_modules | grep -v dist | grep -v .next | head -80

# Folder structure
ls -la src/ 2>/dev/null || ls -la app/ 2>/dev/null || ls -la

# Project context
cat package.json 2>/dev/null | head -40
cat README.md 2>/dev/null | head -50
```

**What to look for:**
- Feature/domain folders (`auth/`, `payments/`, `users/`, `orders/`)
- API routes (`routes.ts`, `controller.ts`, `api/`)
- Models/entities (`schema`, `model`, `entity`, `dto`)
- Configuration (`.env.example`)
- Existing tests — reveal implicit expected behavior

---

### Step 1-A — Audit Existing Specs (Audit Mode)

When `openspec/specs/` already has content, the goal is **to identify gaps** — uncovered domains and missing requirements in existing specs.

#### 1-A.1 — Read all existing specs

```bash
# List already covered domains
find openspec/specs -name "spec.md" | sort

# Read each existing spec
for spec in $(find openspec/specs -name "spec.md"); do
  echo "=== $spec ==="
  cat "$spec"
  echo ""
done
```

#### 1-A.2 — Map domains from code

```bash
# Explore project structure to find undocumented domains
find . -type f \( -name "*.ts" -o -name "*.tsx" \) \
  | grep -v node_modules | grep -v dist | grep -v ".spec.ts" \
  | grep -E "(controller|service|route|module)" | sort

ls src/ 2>/dev/null || ls app/ 2>/dev/null
```

#### 1-A.3 — Identify gaps

Build two maps:

**Domains in code but WITHOUT spec:**
- List code folders/features that don't have a corresponding folder in `openspec/specs/`
- These domains need new specs

**Domains WITH spec but possible gaps:**
- Compare controllers/routes with existing requirements
- Identify endpoints or behaviors without documented scenarios
- Check if critical edge cases (errors, permissions, limits) are covered

#### 1-A.4 — Present diagnosis to user

Before writing anything, present the analysis results:

```
📋 OpenSpec Diagnosis

✅ Already covered domains (X specs):
  - auth/        → Y requirements, Z scenarios
  - payments/    → Y requirements, Z scenarios

🆕 Domains without spec (found in code):
  - notifications/  → detected in src/notifications/
  - reports/        → detected in src/reports/

⚠️  Gaps in existing specs:
  - auth/: endpoint POST /auth/refresh not documented
  - payments/: refund scenario missing

Would you like me to:
[1] Create specs for new domains
[2] Fill gaps in existing specs
[3] Do both
```

Wait for confirmation or instruction from user before proceeding.

---

### Step 2 — Identify Domains (Initial Mode)

Map the functional areas of the system:

| Organization Type | Example Domains |
|---------------------|----------------------|
| By feature         | `auth`, `payments`, `search`, `notifications` |
| By component      | `api`, `frontend`, `workers`, `database` |
| By bounded context | `ordering`, `fulfillment`, `inventory` |

For each domain, record:
- **Name** — folder in `openspec/specs/`
- **Responsibility** — what it does
- **Evidence sources** — which files reveal the behavior

---

### Step 3 — Generate or Update spec.md

#### For new domains — create `openspec/specs/{domain}/spec.md`:

```markdown
# {Domain} Specification

## Purpose
High-level description of what this domain does in the system.

## Scope
What's **included** in this spec:
- {behavior 1}
- {behavior 2}

What's **out of scope**:
- {what's not here and why}

## Requirements

### Requirement: {Behavior Name}
The system SHALL/MUST/SHOULD {observable behavior description}.

#### Scenario: {Happy case}
- GIVEN {precondition}
- WHEN {action or event}
- THEN {expected result}
- AND {additional result, if any}

#### Scenario: {Error or edge case}
- GIVEN {precondition}
- WHEN {invalid action or exceptional situation}
- THEN {how the system responds}
```

#### For gaps in existing specs — add to the corresponding spec:

Insert missing requirements or scenarios in the existing `## Requirements` section, keeping the style and format of the original spec. **Do not rewrite what already exists.**

**Content rules:**

| ✅ Include | ❌ Avoid |
|------------|-----------|
| Externally observable behavior | Internal class/function names |
| Inputs, outputs, error conditions | Library or framework choices |
| External constraints (security, privacy) | Step-by-step implementation details |
| Testable scenarios (Given/When/Then) | Detailed execution plans |

**RFC 2119 Keywords:**
- **MUST / SHALL** — absolute requirement
- **SHOULD** — recommended, but exceptions exist
- **MAY** — optional

---

### Step 4 — Update OpenSpec README

If `openspec/README.md` doesn't exist, create it. If it already exists, **update only the domain table** to include new ones:

```markdown
# OpenSpec — {Project Name}

Behavioral specifications of the system, organized by domain.

## Structure

### specs/
Source of truth — how the system currently behaves.

| Domain | Description |
|---------|-----------|
| [auth](specs/auth/spec.md) | Authentication and session management |
| [payments](specs/payments/spec.md) | Payment processing |
| ... | ... |

### changes/
Proposed modifications. Each change lives in its own folder until merged.

## Conventions

- Requirements use RFC 2119 keywords (SHALL, MUST, SHOULD, MAY)
- Scenarios follow Given/When/Then format
- Specs describe **behavior**, not implementation
```

---

### Step 5 — Validate Quality

**Checklist per generated or modified spec:**
- [ ] Has clear and concise `## Purpose`?
- [ ] Each `### Requirement:` uses RFC 2119 keyword?
- [ ] Each requirement has at least 1 scenario?
- [ ] Scenarios cover happy path AND at least 1 edge case?
- [ ] Doesn't mention internal classes, methods, or libs?
- [ ] Is the described behavior **verifiable/testable**?
- [ ] In Audit Mode: was existing content preserved?

---

## Strategy by Project Type

### NestJS / Express (Backend API)
Analyze: `controller`, `routes`, `dto`, `guard`, `middleware`, tests `*.spec.ts`  
Typical domains: `auth`, `users`, `{principal-entity}`, `notifications`

### React / Next.js (Frontend)
Analyze: `pages/`, `app/`, forms, data fetching hooks, `store/`, `context/`  
Typical domains: `ui`, `navigation`, `forms`, `{principal-feature}`

### Full-stack (e.g., Quantix — React + NestJS)
Divide by end-to-end feature:
- `openspec/specs/auth/` — authentication
- `openspec/specs/transactions/` — transactions
- `openspec/specs/accounts/` — accounts and balances
- `openspec/specs/reports/` — reports and exports

---

## Expected Output

When complete, report to user:

**Initial Mode:**
- Identified domains and created specs
- Requirements per domain
- Areas with weak evidence (need human review)

**Audit Mode:**
- New domains created
- Gaps filled in existing specs
- Items requiring human decision (ambiguous behavior in code)
- Domains in code still without sufficient coverage

---

## References

- [OpenSpec Concepts](references/openspec-concepts.md) — Full philosophy and format
