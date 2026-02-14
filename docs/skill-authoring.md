# Skill Authoring Guide

How to write a new CEOS skill or improve an existing one. This guide is for both EOS practitioners and developers.

## When to Write a New Skill

Before creating a new skill, ask these questions:

| Question | If Yes | If No |
|----------|--------|-------|
| Is this a core EOS tool not yet covered? | New skill | Check existing skills |
| Does it extend an existing skill's workflow? | Add a mode to the existing skill | New skill may be warranted |
| Is it EOS-specific? | Belongs in CEOS | Consider a general-purpose skill instead |
| Do multiple teams need this? | Good candidate | Maybe keep it as a custom skill in your fork |

The current 5 skills cover the most-used EOS tools. Potential future skills include `ceos-process` (documenting core processes), `ceos-accountability` (managing the Accountability Chart interactively), or `ceos-quarterly` (orchestrating the full quarterly planning session).

## Skill File Structure

Every CEOS skill lives in `skills/ceos-NAME/SKILL.md`. The file has two parts: YAML frontmatter and the skill body.

### YAML Frontmatter

```yaml
---
name: ceos-example
description: Use when [trigger condition — when should Claude invoke this skill]
---
```

**Important:** The `description` should say **when** to use the skill, not **what** it does. Claude uses the description to decide whether to invoke the skill, so it needs to match user intent.

| Good Description | Bad Description |
|-----------------|----------------|
| "Use when setting, tracking, or scoring quarterly Rocks" | "Manages quarterly Rocks with three modes" |
| "Use when running or reviewing a Level 10 weekly meeting" | "Facilitates L10 meetings with 7 sections" |

### Skill Body Sections

Every CEOS skill follows the same 5-section structure:

#### 1. When to Use

A bullet list of trigger phrases — things a user might say that should activate this skill.

```markdown
## When to Use

- "Set rocks for this quarter" or "let's plan our quarterly priorities"
- "Rock status" or "how are our rocks tracking?"
- "Score rocks" or "end of quarter review"
```

Include both formal EOS terminology ("IDS this") and natural language ("we have a problem"). Teams vary in how much EOS jargon they use.

#### 2. Context

Tell Claude what it needs to know before executing the skill.

**Repo detection** (required in every CEOS skill):

```markdown
### Finding the CEOS Repository

Search upward from the current directory for the `.ceos` marker file.
This file marks the root of the CEOS repository.

If `.ceos` is not found, stop and tell the user:
"Not in a CEOS repository. Clone your CEOS repo and run setup.sh first."
```

**Key files table** — list every file/directory the skill reads or writes:

```markdown
### Key Files

| File | Purpose |
|------|---------|
| `data/rocks/QUARTER/` | Rock files for each quarter |
| `templates/rock.md` | Template for new Rock files |
```

**Data format details** — document any YAML frontmatter fields, status enums, or naming conventions the skill needs to understand.

#### 3. Process

Step-by-step instructions for Claude to follow. This is the core of the skill.

**Use modes** when a skill handles multiple workflows. Each mode has its own step sequence:

```markdown
## Process

### Mode: Setting Rocks
[Steps for creating new Rocks]

### Mode: Tracking Rocks
[Steps for weekly status updates]

### Mode: Scoring Rocks
[Steps for end-of-quarter review]
```

**Be specific.** Claude follows these instructions literally. Include:
- What to read and display
- What questions to ask the user
- How to validate input
- What to write and where
- Example output formats

#### 4. Output Format

Describe what the skill's output should look like for each mode.

```markdown
## Output Format

**Setting:** Show each Rock file before writing. End with a summary table.
**Tracking:** Status table with milestone progress.
**Scoring:** Quarter scorecard with completion rate.
```

#### 5. Guardrails

Constraints that prevent the skill from doing the wrong thing. These are the most important part of a well-designed skill.

```markdown
## Guardrails

- **Always show diff before writing.** Never modify a file without approval.
- **One owner per Rock.** Suggest splitting if user tries to share.
- **3-7 per person.** Warn if outside this range.
```

Good guardrails prevent:
- Data loss (always show before writing)
- EOS anti-patterns (shared ownership, too many Rocks)
- Scope creep (don't modify files that belong to other skills)
- Silent failures (validate inputs, check for missing files)

## Data Integration Patterns

### Reading Data

Skills read from the `data/` directory tree. Always check if files exist before trying to read them, and tell the user to run `setup.sh init` if key files are missing.

```markdown
Read all files in `data/rocks/[current-quarter]/`.
If the directory doesn't exist, create it.
If no Rock files exist: "No Rocks set for this quarter yet."
```

### Using Templates

New data files should be based on templates in `templates/`. This ensures consistent structure across companies.

```markdown
Use `templates/rock.md` as the template. Fill in:
- Frontmatter: id, title, owner, quarter, status=on_track, created=today
- Body: measurable outcome and milestones from the user
```

### Writing Data

**Always show a diff or preview before writing.** This is the single most important guardrail. CEOS data is business-critical — a corrupted V/TO or lost Issue could affect real business decisions.

```markdown
Show the file before writing. Ask: "Create this Rock?"
```

### ID Generation

Skills that create numbered files (Rocks, Issues) must generate unique IDs:

```markdown
Read existing files in the directory.
Find the highest existing ID number.
Increment by 1 for the new file.
Check BOTH open/ and solved/ directories (for Issues).
```

## Cross-Skill Coordination

CEOS skills are **loosely coupled**. A skill can mention another skill but should never auto-invoke it.

| Do This | Don't Do This |
|---------|---------------|
| "Consider creating an Issue with `ceos-ids`" | Automatically calling ceos-ids |
| "Check Rock alignment with `ceos-vto`" | Reading and modifying the V/TO directly |
| "This off-track metric should go to the Issues list" | Creating an Issue file without the user's say-so |

Why? The user controls the workflow. During an L10 meeting, the facilitator decides when to switch from Scorecard Review to IDS. Skills should suggest, not command.

## Security & Scope Boundaries

CEOS skills run inside Claude Code with the user's full permissions. A skill can read files, write files, and execute commands. This power requires discipline — skills should do the minimum necessary and stay within well-defined boundaries.

### Scope Rules

Every CEOS skill should follow these scope rules:

| Rule | Why |
|------|-----|
| **Only access `data/` and `templates/`** | Company EOS data lives here — nothing else should be touched |
| **Never reference external URLs or APIs** | Skills should work offline; external calls could exfiltrate data |
| **Never handle credentials** | No asking for API keys, passwords, or tokens |
| **Never instruct shell commands** | File operations go through Claude's built-in tools, not `bash` |
| **Always show before writing** | The user must approve every file change (the "diff before write" rule) |
| **Stay in the CEOS repo** | The `.ceos` marker detection ensures skills don't operate outside the repo |

### Security Manifest (Optional Frontmatter)

You can add optional fields to your skill's YAML frontmatter to declare its scope explicitly. These help reviewers audit your skill quickly:

```yaml
---
name: ceos-example
description: Use when [trigger condition]
file-access: [data/example/, templates/example.md]
tools-used: [Read, Write, Glob]
---
```

| Field | Purpose | Example |
|-------|---------|---------|
| `file-access` | Directories and files the skill reads or writes | `[data/rocks/, data/vision.md, templates/rock.md]` |
| `tools-used` | Claude Code tools the skill relies on | `[Read, Write, Glob, Grep]` |

These fields are **not enforced at runtime** — they're a declaration of intent that makes code review faster. Reviewers compare what the manifest says against what the skill body actually does.

### What NOT to Put in a Skill

| Don't Do This | Why | What to Do Instead |
|---------------|-----|-------------------|
| `Read ~/.ssh/id_rsa` | Accesses files outside CEOS repo | Only read from `data/` and `templates/` |
| `Run curl https://api.example.com/...` | External network call | Skills should work offline |
| `Ask the user for their API key` | Credential harvesting risk | Skills don't need external service credentials |
| `Write to /tmp/export.csv` | Writes outside CEOS repo | Write to `data/` directory |
| `Delete all files in data/rocks/` | Destructive bulk operation | Modify individual files with user confirmation |
| `Automatically invoke ceos-ids` | Breaks user control principle | Suggest the skill; let user invoke it |

### Well-Scoped vs Suspicious: Examples

**Well-scoped skill manifest:**

```yaml
---
name: ceos-rocks
description: Use when setting, tracking, or scoring quarterly Rocks
file-access: [data/rocks/, templates/rock.md]
tools-used: [Read, Write, Glob]
---
```

This skill reads Rock files, creates new ones from a template, and lists directory contents. All within `data/` and `templates/`. Straightforward to review.

**Suspicious skill manifest:**

```yaml
---
name: ceos-export
description: Use when exporting EOS data
file-access: [data/, ~/.config/export/]
tools-used: [Read, Write, Bash, WebFetch]
---
```

Red flags: accesses files outside the repo (`~/.config/`), uses `Bash` (shell commands), and uses `WebFetch` (external network access). A reviewer should ask: why does an export skill need network access and shell commands?

## Testing Your Skill

CEOS skills are tested manually with Claude Code:

1. **Install the skill**: Run `./setup.sh` to symlink your skill into `~/.claude/skills/`
2. **Start Claude Code**: `claude` in your CEOS repo directory
3. **Try each trigger phrase**: Does Claude recognize the skill?
4. **Test each mode**: Walk through every workflow path
5. **Test edge cases**:
   - What happens with no data files? (first-time setup)
   - What happens with malformed frontmatter?
   - What happens if the user gives unexpected input?
   - What if required files are missing?

## Submitting Your Skill

1. **Open an Issue first** — describe the EOS process your skill implements
2. **Fork the repo** and create your skill in `skills/ceos-NAME/SKILL.md`
3. **Follow the naming convention**: `ceos-` prefix, lowercase, hyphen-separated
4. **Update docs** — add your skill to [skill-reference.md](skill-reference.md)
5. **Submit a PR** with:
   - The SKILL.md file
   - Before/after examples showing the skill in action
   - Which EOS tool or process it implements

See [CONTRIBUTING.md](../CONTRIBUTING.md) for the full contribution process.

## Quick Reference

| Element | Convention |
|---------|-----------|
| Skill directory | `skills/ceos-NAME/` |
| Skill file | `SKILL.md` (exactly this name) |
| Naming | `ceos-` prefix, lowercase, hyphens |
| Frontmatter | `name` + `description` (when to use, not what it does) |
| Sections | When to Use, Context, Process, Output Format, Guardrails |
| Repo detection | Search upward for `.ceos` marker |
| Data files | Markdown + YAML frontmatter in `data/` |
| Templates | `templates/` directory, used as basis for new files |
| Diff before write | Always — no exceptions |
| Cross-skill | Mention, don't invoke |
