# OpenSpec Generator

A skill for analyzing existing projects and generating or updating specification artifacts following the OpenSpec structure and philosophy.

## What is OpenSpec?

OpenSpec is a lightweight specification framework built on four principles:

```
fluid not rigid       — no phase gates, work on what makes sense
iterative not waterfall — learn while building, refine as you go
easy not complex      — light setup, minimal ceremony
brownfield-first      — works with existing codebases, not just greenfield
```

**Specs** describe how the system **currently behaves** (source of truth).  
**Changes** are proposed modifications, living in separate folders until merged.

## OpenSpec Generator Skill

This skill helps you:

- ✅ Initialize an `openspec/` folder structure in your project
- ✅ Generate specification documents from existing code
- ✅ Audit existing specs for gaps and missing domains
- ✅ Map features to requirements and scenarios
- ✅ Document system behavior formally

### Trigger Keywords

This skill activates when you mention:
- "openspec"
- "create specs" / "generate specifications"
- "update specs" / "audit specs"
- "document system behavior"
- "initialize openspec"
- "specs missing"
- "analyze codebase and produce structured requirements"

## Installation

### For Claude Desktop / Claude Code

1. **Locate your Claude config folder:**
   - **Windows**: `%APPDATA%\Claude\claude.json`
   - **macOS**: `~/Library/Application Support/Claude/claude.json`
   - **Linux**: `~/.config/Claude/claude.json`

2. **Add the skill path to your configuration:**

```json
{
  "skills": [
    {
      "name": "openspec-generator",
      "path": "PATH_TO_SKILL/openspec-generator"
    }
  ]
}
```

3. **Restart Claude** to load the new skill.

### For VS Code (GitHub Copilot)

1. **Find your VS Code extensions folder:**
   - **Windows**: `%USERPROFILE%\.vscode\extensions`
   - **macOS**: `~/.vscode/extensions`
   - **Linux**: `~/.vscode/extensions`

2. **Create a custom instructions file** for Copilot:
   
   Create `.vscode/copilot-instructions.md` in your workspace with:

```markdown
# Custom Copilot Instructions

You have access to the OpenSpec Generator skill. When the user wants to:
- Create or update specs using OpenSpec
- Initialize an openspec/ folder
- Document system behavior
- Generate spec.md files from existing code
- Audit existing specs for gaps

Use the skill located at: PATH_TO_SKILL/openspec-generator
```

3. **Or add to VS Code settings** (`settings.json`):

```json
{
  "github.copilot.chat.instructions": "You have access to OpenSpec Generator skill..."
}
```

### For Cursor IDE

1. Open Cursor settings
2. Navigate to **AI Settings** → **Custom Instructions**
3. Add:

```
When asked to create specifications, document system behavior, or work with OpenSpec:
- Use the OpenSpec Generator skill from: PATH_TO_SKILL/openspec-generator
```

### For Other AI Code Editors

**General method** — Add skill reference in your AI assistant's configuration:

```json
{
  "skill": "openspec-generator",
  "path": "PATH_TO_SKILL/openspec-generator",
  "description": "Generate OpenSpec specification artifacts from existing codebases"
}
```

## Usage

Once installed, simply ask your AI assistant:

> "Generate OpenSpec specs for my project"  
> "Initialize OpenSpec in this codebase"  
> "Audit the existing specs and find gaps"  
> "Document how the auth system works"

The skill will:

1. **Check preconditions** — Verify `openspec/` and `openspec/specs/` exist
2. **Explore the project** — Identify domains and structure
3. **Generate or update specs** — Create `spec.md` files with requirements and scenarios
4. **Validate quality** — Ensure specs follow OpenSpec format

## OpenSpec Folder Structure

```
openspec/
├── specs/                    # Source of truth — how the system works
│   ├── auth/
│   │   └── spec.md
│   └── payments/
│       └── spec.md
└── changes/                  # Proposed modifications
    └── add-oauth/
        ├── spec.delta.md
        ├── design.md
        └── tasks.md
```

## Spec Format

```markdown
# {Domain} Specification

## Purpose
High-level description of what this domain does.

## Scope
What's included and what's excluded.

## Requirements

### Requirement: {Behavior Name}
The system SHALL/MUST/SHOULD {observable behavior}.

#### Scenario: {Case Name}
- GIVEN {precondition}
- WHEN {action or event}
- THEN {expected result}
- AND {additional result, if any}
```

## RFC 2119 Keywords

| Keyword | Meaning |
|---------|---------|
| MUST / SHALL | Absolute requirement — no exceptions |
| SHOULD | Recommended — justified exceptions exist |
| MAY | Optional |
| MUST NOT | Absolutely prohibited |

## Requirements

- **Do**: Describe externally observable behavior
- **Do**: Include inputs, outputs, error conditions
- **Don't**: Mention internal classes, methods, or libraries
- **Don't**: Include implementation details

## Additional Resources

- [OpenSpec Concepts](./skill/openspec-generator/references/openspec-concepts.md) — Complete philosophy and format reference
- [OpenSpec Repository](https://github.com/Fission-AI/OpenSpec/) — The OpenSpec repository
## License

MIT
