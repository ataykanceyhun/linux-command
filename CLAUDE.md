# CLAUDE.md

This file provides guidance for AI assistants working in this repository.

## Project Overview

This is a **Turkish-language Linux operational runbook documentation repository**. It contains professional, repeatable operational procedures for Linux system administration tasks. There is no application code — all content is Markdown with embedded Bash command examples.

**Primary language:** Turkish (technical terms and commands remain in English/original form)

## Repository Structure

```
linux-command/
├── .github/
│   ├── workflows/
│   │   └── docs-quality.yml    # CI: markdown lint + link check
│   └── pull_request_template.md
├── .markdownlint.json           # Linting config (MD013, MD031, MD032 disabled)
├── CLAUDE.md                    # This file
├── CONTRIBUTING.md              # Contribution workflow
├── README.md                    # Project index and quick start
├── RUNBOOK_QUALITY_CHECKLIST.md # Pre-PR quality gate
├── STYLE_GUIDE.md               # Canonical style rules
├── network/
│   ├── README.md
│   └── firewall/
│       └── iptables-basics.md
├── storage/
│   ├── README.md
│   ├── lvm/
│   │   └── resize-physical-volume.md
│   ├── vdo/
│   │   ├── create-vdo.md
│   │   └── extend-vdo-capacity.md
│   └── xfs/
│       └── create-xfs-on-lvm.md
└── templates/
    └── runbook-template.md      # Copy this when creating new runbooks
```

## Mandatory Runbook Structure

Every runbook **must** contain exactly these 9 headers in this order:

1. `# Baslik` — Title with metadata table directly below it
2. `## Amac` — Purpose (1–2 sentences)
3. `## Kapsam` — Scope (affected systems / environments)
4. `## Onkosullar` — Prerequisites (packages, permissions, variable table)
5. `## Risk ve Geri Donus (Rollback)` — Risks, backup commands, rollback commands
6. `## Adimlar` — Steps (pre-check → change → persist)
7. `## Dogrulama` — Validation (minimum 2 commands, at least one showing the changed state)
8. `## Sorun Giderme` — Troubleshooting (minimum 2 distinct failure scenarios)
9. `## Referanslar` — References (man pages, official docs)

### Metadata Table

Place this immediately under the `# Baslik` heading:

```markdown
| Alan          | Deger              |
| --- | --- |
| Risk          | <Low|Medium|High>  |
| Son Dogrulama | <YYYY-MM-DD>       |
| Tahmini Sure  | <e.g. 15 dk>       |
| Kesinti Etkisi| <Yok|Kismi|Tam>    |
```

## Style Conventions

### Language

- Write in **Turkish** for explanatory prose.
- Keep Linux/technical terms in English (e.g., `volume group`, `logical volume`, `interface`).
- Use short distribution-specific notes where behavior differs:
  - `RHEL/CentOS notu: ...`
  - `Ubuntu/Debian notu: ...`

### File Naming

- **Always** `kebab-case` and ASCII-only (no Turkish special characters in filenames).
- Examples: `resize-physical-volume.md`, `create-vdo.md`, `iptables-basics.md`

### Command Blocks

- Wrap all commands in fenced ` ```bash ``` ` blocks.
- **Never** include prompt symbols (`$` or `#`) — only the raw command.
- Multi-line commands use backslash continuation.
- Every critical step must be immediately followed by a validation command.

### Placeholder Naming

Use these exact placeholder tokens consistently across all runbooks:

| Placeholder | Meaning |
| --- | --- |
| `<disk_device>` | Physical disk (e.g. `sdb`, `nvme0n1`) |
| `<vg_name>` | LVM volume group name |
| `<lv_name>` | LVM logical volume name |
| `<vdo_lv_name>` | VDO logical volume name |
| `<vdo_pool_size>` | VDO physical pool size |
| `<vdo_virtual_size>` | VDO virtual capacity |
| `<mount_point>` | Filesystem mount path |
| `<interface_name>` | Network interface name |
| `<source_cidr>` | Source network in CIDR notation |
| `<destination_ip>` | Target IP address |
| `<line_number>` | iptables rule line number |

### Security Guardrail Flow

For any risky operation, the `## Adimlar` section **must** follow this order:

1. **On kontrol** — snapshot current state (`lsblk`, `pvs`, `vgs`, `lvs`, `iptables -S`, etc.)
2. **Yedekleme** — backup config before changes (`cp /etc/fstab /etc/fstab.bak.$(date +%F-%H%M%S)`, `iptables-save`, `vgcfgbackup`, etc.)
3. **Degisiklik** — apply the change
4. **Kalici hale getirme** — persist if needed (`/etc/fstab`, `iptables-save > /etc/sysconfig/iptables`, etc.)

The `## Risk ve Geri Donus (Rollback)` section must document both backup commands and the matching rollback/recovery commands.

## Adding New Runbooks

1. Copy `templates/runbook-template.md` to the appropriate topic directory.
2. Place under `storage/` or `network/` (create a subdirectory as needed).
3. Name the file in `kebab-case` ASCII.
4. Fill in all 9 mandatory sections.
5. Add an entry to `README.md` (both the topic section and the catalog table).
6. Add an entry to the relevant topic `README.md` (e.g., `storage/README.md`).
7. Validate against `RUNBOOK_QUALITY_CHECKLIST.md` before opening a PR.

## CI / Quality Gates

GitHub Actions runs on every push to `main` and on all pull requests:

| Job | Tool | What it checks |
| --- | --- | --- |
| `markdown-lint` | `markdownlint-cli2` | All `**/*.md` files against `.markdownlint.json` rules |
| `link-check` | `lychee` | All links in `**/*.md` are reachable (accepts 200, 206, 429; retries 3×) |

**CI must pass before merging.** Fix lint or broken links before requesting review.

### Markdown Lint Rules (`.markdownlint.json`)

All default rules are enabled except:

- `MD013` — line length limit (disabled)
- `MD031` — blank line around code blocks (disabled)
- `MD032` — blank line around lists (disabled)

## Commit and PR Conventions

### Commit Message Format

```
docs(<scope>): <short description in Turkish or English>
```

Examples:

```
docs(storage): LVM fiziksel volume buyutme adimlari eklendi
docs(network): iptables temel kurallar duzeltildi
docs(template): onkosullar bolumu guncellendi
```

### PR Description (use `.github/pull_request_template.md`)

- **Amac** — What change was made and why
- **Kapsam** — Which files/topics are affected
- **Risk** — Low / Medium / High with justification
- **Dogrulama adimlari** — Steps a reviewer can take to verify the change

### PR Checklist (from `CONTRIBUTING.md`)

- [ ] File name is `kebab-case` and ASCII
- [ ] All 9 mandatory headers present in correct order
- [ ] Commands include validation steps
- [ ] Rollback commands documented
- [ ] Metadata table is complete
- [ ] `RUNBOOK_QUALITY_CHECKLIST.md` verified
- [ ] CI checks pass

## Development Branch

Active development happens on the `claude/add-claude-documentation-ZivlP` branch. The stable branch is `main`. Push to the designated feature branch; never push directly to `main`.

## Key Reference Files

| File | Purpose |
| --- | --- |
| `STYLE_GUIDE.md` | Authoritative style rules — read before writing any runbook |
| `CONTRIBUTING.md` | Contribution workflow and PR checklist |
| `RUNBOOK_QUALITY_CHECKLIST.md` | Pre-PR quality gate checklist |
| `templates/runbook-template.md` | Starting point for every new runbook |
| `.markdownlint.json` | Linting rule overrides |
| `.github/workflows/docs-quality.yml` | CI pipeline definition |
