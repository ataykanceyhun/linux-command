# CLAUDE.md — AI Assistant Guide for linux-command

This file provides context, conventions, and workflow instructions for AI
assistants working in this repository.

## Project Overview

This is a **Turkish-language documentation repository** containing professional
Linux operations runbooks. It is a documentation-only project (no code to build
or run). All content is written in Turkish; technical commands and Linux terms
remain in their original English form.

**Goals:**

- Provide repeatable, production-safe operational procedures
- Enforce pre-check → backup → change → validate → rollback discipline
- Maintain a lightweight, scalable Markdown knowledge base

## Repository Structure

```
linux-command/
├── .github/
│   ├── workflows/docs-quality.yml   # CI: markdownlint + lychee link check
│   └── pull_request_template.md     # PR description template
├── .markdownlint.json               # Markdown lint overrides
├── CLAUDE.md                        # This file
├── CONTRIBUTING.md                  # Contribution workflow
├── README.md                        # Project overview + runbook index
├── RUNBOOK_QUALITY_CHECKLIST.md     # Quality gate checklist
├── STYLE_GUIDE.md                   # Writing standards
├── templates/
│   └── runbook-template.md          # Boilerplate for new runbooks
├── storage/
│   ├── README.md
│   ├── lvm/resize-physical-volume.md
│   ├── vdo/create-vdo.md
│   ├── vdo/extend-vdo-capacity.md
│   └── xfs/create-xfs-on-lvm.md
└── network/
    ├── README.md
    └── firewall/iptables-basics.md
```

New topic areas follow the same pattern: a top-level directory with its own
`README.md` and sub-directories per subject (e.g., `storage/lvm/`).

## Runbook Catalogue

| Domain  | File                                    | Risk   |
| ------- | --------------------------------------- | ------ |
| Storage | `storage/lvm/resize-physical-volume.md` | Medium |
| Storage | `storage/vdo/create-vdo.md`             | High   |
| Storage | `storage/vdo/extend-vdo-capacity.md`    | High   |
| Storage | `storage/xfs/create-xfs-on-lvm.md`     | High   |
| Network | `network/firewall/iptables-basics.md`   | High   |

## Mandatory Runbook Structure

Every runbook **must** contain exactly these nine headings in this order:

1. `# <Title>` — document title
2. `## Amac` — purpose (1-2 sentences)
3. `## Kapsam` — scope (affected systems, environments)
4. `## Onkosullar` — prerequisites (packages, permissions, variables)
5. `## Risk ve Geri Donus (Rollback)` — risks, backup command, rollback command
6. `## Adimlar` — step-by-step procedure
7. `## Dogrulama` — validation (minimum 2 commands)
8. `## Sorun Giderme` — troubleshooting (minimum 2 failure modes)
9. `## Referanslar` — references (internal docs, man pages)

Immediately below the `# Title` heading, each runbook includes a metadata table:

```markdown
| Alan            | Deger              |
| --------------- | ------------------ |
| Risk            | <Low|Medium|High>  |
| Son Dogrulama   | <YYYY-MM-DD>       |
| Tahmini Sure    | <e.g. 15 dk>       |
| Kesinti Etkisi  | <Yok|Kismi|Tam>    |
```

## Writing Conventions

### Language

- Primary language: **Turkish**
- Linux commands, package names, and technical terms stay in English
- Distribution-specific notes use these prefixes:
  - `RHEL/CentOS notu: ...`
  - `Ubuntu/Debian notu: ...`

### File naming

- ASCII characters only, `kebab-case` (e.g., `resize-physical-volume.md`)
- Place files under the correct domain directory

### Commands

- All commands go inside ` ```bash ` fenced blocks — no exceptions
- **No shell prompts** (`$` or `#`) inside code blocks
- Every critical step is followed by at least one validation command
- Use the standard placeholder format: `<disk_device>`, `<vg_name>`,
  `<lv_name>`, `<mount_point>`, `<interface_name>`, `<source_cidr>`,
  `<destination_ip>`

### Safety flow for risky commands

Runbooks covering destructive operations must follow this sequence inside
`## Adimlar`:

1. Pre-check (`lsblk`, `pvs`, `vgs`, `lvs`, current config snapshot)
2. Backup (`cp /etc/fstab /etc/fstab.bak.$(date +%F-%H%M%S)`,
   `iptables-save`, `vgcfgbackup`, etc.)
3. Apply change
4. Validate
5. Rollback command (also in `## Risk ve Geri Donus`)

### Validation section

- Minimum **2 commands**
- At least one must show the effect of the change clearly
  (e.g., `df -hT`, `mount`, `iptables -S`)

### Troubleshooting section

- Minimum **2 distinct failure modes**, each with:
  - Symptom
  - Likely cause
  - Resolution command

## Creating a New Runbook

1. Copy `templates/runbook-template.md` to the appropriate directory.
2. Name the file in ASCII `kebab-case`.
3. Fill in all nine sections and the metadata table.
4. Follow `STYLE_GUIDE.md` throughout.
5. Self-review using `RUNBOOK_QUALITY_CHECKLIST.md`.
6. Update the relevant `README.md` index and the root `README.md` catalogue
   table.
7. Open a PR using `.github/pull_request_template.md`.

## CI/CD

The `docs-quality` GitHub Actions workflow (`.github/workflows/docs-quality.yml`)
runs on every push to `main` and on every pull request:

| Job              | Tool                          | What it checks           |
| ---------------- | ----------------------------- | ------------------------ |
| `markdown-lint`  | `markdownlint-cli2-action@v20`| All `**/*.md` files      |
| `link-check`     | `lychee-action@v2`            | All links inside `**/*.md`|

Lychee settings: `--max-retries 3`, `--retry-wait-time 2`, accepts HTTP
200, 206, 429. Link checks fail the build on broken links.

**Both CI jobs must pass before merging.**

### Running checks locally

```bash
# Markdown lint (requires markdownlint-cli2)
npx markdownlint-cli2 "**/*.md"

# Link check (requires lychee)
lychee --verbose --max-retries 3 --accept 200,206,429 "**/*.md"
```

### Disabled markdownlint rules (`.markdownlint.json`)

| Rule  | Description                             |
| ----- | --------------------------------------- |
| MD013 | Line length limit                       |
| MD031 | Blank lines around fenced code blocks   |
| MD032 | Blank lines around lists                |

Do not re-enable these rules without team discussion.

## Git Workflow

### Branches

- Stable branch: `main`
- Feature/docs branches: descriptive names, e.g., `add-network-bonding-runbook`
- AI-generated branches follow the pattern: `claude/<topic>-<id>`

### Commit messages

Recommended format from `CONTRIBUTING.md`:

```
docs(<scope>): <short description in Turkish or English>
```

Examples:

```
docs(storage): LVM PV buyutme runbook eklendi
docs(network): iptables temelleri guncellendi
```

Keep commit messages short and imperative. No period at the end.

### Pull requests

Use `.github/pull_request_template.md`. Fill in:

- **Amac** — what problem this solves
- **Kapsam** — which files and topics changed
- **Risk** — Low / Medium / High with a brief note
- **Dogrulama** — how the change was tested/reviewed
- **Checklist** — all boxes checked before requesting review

## Quality Checklist Summary

Before opening a PR, verify all items in `RUNBOOK_QUALITY_CHECKLIST.md`:

**Content quality**

- [ ] `## Amac` is one clear sentence
- [ ] `## Kapsam` defines the impact area
- [ ] `## Onkosullar` lists packages, permissions, environment
- [ ] Rollback steps are actionable and testable
- [ ] Steps are in chronological order with copyable commands
- [ ] Validation commands measure success
- [ ] At least 2 failure modes in troubleshooting

**Technical quality**

- [ ] File name is ASCII `kebab-case`
- [ ] All code blocks use ` ```bash `
- [ ] Placeholders use standard format (`<vg_name>`, etc.)
- [ ] No broken internal links

**Operational quality**

- [ ] Backup command present for production-risk steps
- [ ] Pre-check captures current state before changes
- [ ] Rollback steps are genuinely executable
- [ ] Distro differences noted where applicable

## Key Conventions for AI Assistants

1. **Language**: Write all prose in Turkish. Keep commands in English.
2. **No new structure**: Do not invent new heading names or section orders.
   Use the nine mandatory headings exactly as specified.
3. **No shell prompts**: Never add `$` or `#` to code blocks.
4. **Always include rollback**: Every runbook editing system state must have a
   backup and rollback command.
5. **Placeholder consistency**: Use only the approved placeholder names listed
   above.
6. **Update the index**: When adding a runbook, update both the domain
   `README.md` and the root `README.md` catalogue table.
7. **CI must pass**: After any change, verify that markdown lint rules are not
   violated. Do not disable additional markdownlint rules.
8. **Metadata is mandatory**: Every runbook file must have the metadata table
   directly below its `# Title`.
9. **Minimum validation**: The `## Dogrulama` section needs at least 2 commands.
10. **Minimum troubleshooting**: The `## Sorun Giderme` section needs at least
    2 distinct failure modes.
