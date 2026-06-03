# david-skills — a personal Claude plugin marketplace

A small [Claude Code plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces)
(just a git repo) that distributes my Claude skills as installable plugins. Plugins
installed from here work in **both Claude Code (CLI/IDE) and the Claude Desktop app**.

## Plugins in this marketplace

### `step-checklist-builder`

Turns any multi-step guide, runbook, setup procedure, SOP, onboarding doc, or ordered
how-to into a **single self-contained interactive HTML "console"** — a dark, polished
dashboard where the reader:

- expands each step for a click-by-click walkthrough,
- ticks off each action (numbered badges flip to green checks),
- watches a live progress ring with done / pending / needs-help counts,
- filters steps by status,
- flags steps they're stuck on to reveal troubleshooting,
- and copies code blocks with one click.

Progress is saved in `localStorage`, so the reader can leave and return mid-task. The
skill is **model-invoked**: Claude triggers it automatically when you ask to turn
instructions into a checklist/tracker page (e.g. "make a checklist page", "turn this
runbook into something I can walk through", "build me a setup tracker").

## Install

> Requires Claude Code v2.1.x or later (`claude --version`).

**From this published repo (recommended):**

```bash
# Add the marketplace (GitHub owner/repo shorthand)
claude plugin marketplace add <your-github-username>/<repo>

# Install the plugin
claude plugin install step-checklist-builder@david-skills
```

Or, inside an interactive Claude Code session, use the slash equivalents:

```text
/plugin marketplace add <your-github-username>/<repo>
/plugin install step-checklist-builder@david-skills
```

After installing, run `/skills` (or `/plugin`) to confirm it loaded. The skill then
triggers automatically, or you can invoke it explicitly with
`/step-checklist-builder:step-checklist-builder`.

**Claude Desktop app:** add the same marketplace from the app's plugin/marketplace
settings (Settings → Plugins/Marketplaces → add `<your-github-username>/<repo>`), then
install `step-checklist-builder`. Because the marketplace is a public git repo, the same
source works across Claude Code and Desktop.

## Local development / testing

```bash
# Validate the marketplace + plugin manifests
claude plugin validate ./my-marketplace

# Add the marketplace from a local path and install
claude plugin marketplace add ./my-marketplace
claude plugin install step-checklist-builder@david-skills

# Iterate without installing
claude --plugin-dir ./my-marketplace/plugins/step-checklist-builder
```

## Repository layout

```
my-marketplace/
├── .claude-plugin/
│   └── marketplace.json                 # registers the plugin(s) below
└── plugins/
    └── step-checklist-builder/
        ├── .claude-plugin/
        │   └── plugin.json              # plugin manifest (name/description/version)
        └── skills/
            └── step-checklist-builder/  # the skill, copied intact
                ├── SKILL.md
                ├── assets/template.html
                └── references/data-format.md
```

## Note on output paths

`SKILL.md` writes its finished HTML to the Claude.ai sandbox locations
(`/home/claude/…`, `/mnt/user-data/outputs/`) and presents it with `present_files`.
Those are conveniences of the Claude.ai / Desktop sandbox. In a plain Claude Code CLI
session those paths may not exist — Claude will simply write the generated `.html` to
your working directory instead. The skill's own bundled references
(`assets/template.html`, `references/data-format.md`) are relative, so the skill is
fully portable.
