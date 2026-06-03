# abaktiar-skills — a personal Claude plugin marketplace

A [Claude Code plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces)
(just a git repo) that distributes my Claude skills as installable plugins. Plugins
installed from here work in **both Claude Code (CLI/IDE) and the Claude Desktop app**.

- **Marketplace name:** `abaktiar-skills`
- **Repo:** `abaktiar/skills` (https://github.com/abaktiar/skills)
- **Owner:** Al baktiar

## Plugins in this marketplace

### `step-checklist-builder`

Turns any multi-step guide, runbook, setup procedure, SOP, onboarding doc, or ordered
how-to into a **single self-contained interactive HTML "console"** — a dark, polished
dashboard where the reader expands each step, ticks off each action (numbered badges flip
to green checks), watches a live progress ring (done / pending / needs-help), filters by
status, flags steps they're stuck on to reveal troubleshooting, and copies code blocks.
Progress is saved in `localStorage`. Model-invoked: Claude triggers it automatically when
you ask to turn instructions into a checklist/tracker page.

## Install

> Requires Claude Code v2.1.x or later (`claude --version`).

```bash
# Add the marketplace (GitHub owner/repo shorthand)
claude plugin marketplace add abaktiar/skills

# Install the plugin
claude plugin install step-checklist-builder@abaktiar-skills
```

Inside an interactive Claude Code session, the slash equivalents are
`/plugin marketplace add abaktiar/skills` then `/plugin install <plugin>@abaktiar-skills`.
Run `/skills` (or `/plugin`) to confirm they loaded.

**Claude Desktop app:** add the same marketplace from Settings → Plugins/Marketplaces
(`abaktiar/skills`), then install the plugin(s). Because the marketplace is a git repo, the
same source works across Claude Code and Desktop.

## Local development / testing

```bash
# Validate the marketplace + each plugin manifest
claude plugin validate ./my-marketplace
claude plugin validate ./my-marketplace/plugins/step-checklist-builder

# Add from a local path and install
claude plugin marketplace add ./my-marketplace
claude plugin install step-checklist-builder@abaktiar-skills

# Iterate on one plugin without installing
claude --plugin-dir ./my-marketplace/plugins/step-checklist-builder
```

## Repository layout

```
my-marketplace/
├── .claude-plugin/
│   └── marketplace.json                     # registers the plugin below
└── plugins/
    └── step-checklist-builder/
        ├── .claude-plugin/plugin.json
        └── skills/step-checklist-builder/    # skill, copied intact
            ├── SKILL.md
            ├── assets/template.html
            └── references/data-format.md
```

## Notes

- **`step-checklist-builder` output paths.** `SKILL.md` writes its finished HTML to the
  Claude.ai sandbox locations (`/home/claude/…`, `/mnt/user-data/outputs/`). Those are
  conveniences of the Claude.ai / Desktop sandbox; in a plain Claude Code CLI session
  Claude simply writes the generated `.html` to your working directory. The skill's bundled
  references are relative, so the skill is fully portable.
