# CEOS

CEOS (Claude EOS) implements the Entrepreneurial Operating System through Claude Code skills. Each EOS component — Vision, Rocks, Scorecard, People Analyzer, etc. — is a standalone skill that reads and writes structured markdown files in a shared git repository.

## Directory Structure

```
ceos/
├── skills/               # Claude Code skills (one per EOS component)
│   └── ceos-*/SKILL.md   # Skill definition files
├── data/                  # All EOS data (git-tracked)
│   ├── vision.md          # V/TO (Vision/Traction Organizer)
│   ├── accountability.md  # Accountability Chart
│   ├── rocks/             # Quarterly Rocks by quarter
│   ├── scorecard/         # Weekly scorecard metrics and entries
│   ├── issues/            # IDS issues (open/ and resolved/)
│   ├── meetings/          # Meeting notes (L10 + kickoff sessions)
│   │   ├── l10/           # Weekly L10 meeting notes
│   │   └── kickoff/       # Focus Day + Vision Building Day sessions
│   ├── processes/         # Core process documentation
│   ├── people/            # People Analyzer evaluations
│   ├── conversations/     # Quarterly conversation records
│   ├── annual/            # Annual planning session records
│   ├── quarterly/         # Quarterly planning session records
│   ├── checkups/          # Organizational Checkup assessments
│   ├── delegate/          # Delegate and Elevate audits
│   ├── clarity/           # Clarity Break reflections
│   └── todos/             # To-Do tracking
├── templates/             # Templates for new data files
├── docs/                  # Documentation (eos-primer.md)
└── setup.sh               # Repository initialization
```

## Workflow Profile

```yaml
workflow:
  base_branch: main
  direct_to_main: true               # No feature branches — commit directly to main
  investigation: light               # Quick search, no Explore agent
  plan_approval: auto                # Auto-approve plans (skills repo)
  user_testing: skip                 # No manual testing needed
  quality_gates: []                  # No automated tests
  review:
    triage: false                    # No review agents
    max_level: NONE
    agents: []
  ship:
    method: direct_push
    target: main
    linear_status: "Done"            # Commit completes the ticket
    deploy_hint: "git push"
  labels:
    auto_detect: false
```

## Workflow

- **Commits directly to `main`** — no feature branches
- **No automated tests** — manual verification of skill files
- **Skills are symlinked** from `~/.claude/skills/ceos-*/` to this repo's `skills/` directory
- **Git-based sync**: Skills run `git pull --ff-only` before reading data and users commit after writes

## Skill Structure Contract

See [`docs/skill-structure.md`](docs/skill-structure.md) for the canonical skill specification:

- **8-section structure** every CEOS skill must follow
- **Data Ownership table** defining which skill writes to which directory
- **Creating New Skills** checklist and files-to-update table
- **Exemplar Skills** to use as reference implementations
