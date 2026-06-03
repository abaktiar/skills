---
name: step-checklist-builder
description: >-
  Turn any multi-step guide, runbook, setup procedure, SOP, onboarding doc, or
  ordered how-to into a single self-contained interactive HTML "console" — a
  dark, polished dashboard where the reader expands each step for a click-by-click
  walkthrough, ticks off each action (numbered badges flip to green checks),
  watches a live progress ring with done / pending / needs-help counts, filters
  by status, flags steps where they're stuck to reveal troubleshooting, and
  copies code blocks. Progress saves locally so they can return mid-task.
  Use this skill whenever the user wants to turn instructions, steps, a setup
  guide, a deployment/provisioning checklist, a runbook, an SOP, or an onboarding
  flow into a trackable, checkable web page — even if they just say "make a
  checklist page", "turn this guide into something I can walk through", "build me
  a setup tracker", or "same as before but for a new topic". Prefer this skill
  over hand-rolling HTML when the deliverable is a step-by-step tracker.
---

# Step Checklist Builder

Generates a self-contained interactive HTML page from ordered step content. The
look and behaviour are fixed (a deliberate dark "engineering console" theme using
IBM Plex fonts); only the **content** changes per topic. This consistency is the
point — the user gets the same familiar tool every time.

## What the output does

- Each step is a collapsible card with a big number, title, and time/badge meta.
- Expanding a step reveals a **numbered walkthrough** whose items are the
  checkboxes — tapping an item's number flips it to a green check and strikes the
  text. A step auto-completes when all its items are ticked; "Mark all done"
  toggles them together.
- A sticky dashboard shows a progress ring + **Done / Pending / Need help** counts
  and filter chips (All / Pending / Done / Need help / Has wait time).
- "Need help here" flags a step (red) and reveals a contextual troubleshooting
  panel for that step, if one was provided.
- Code blocks have copy-to-clipboard buttons. Progress persists via localStorage.

## Workflow

1. **Read the source.** Take the user's guide / steps (pasted text, an uploaded
   file, or content already in the conversation). If a file, read it first.
2. **Read `references/data-format.md`.** It defines the exact step + block shapes.
   Do this before writing data — the block grammar is small but specific.
3. **Copy the template.** `cp assets/template.html /home/claude/<name>.html`
   (or straight to `/mnt/user-data/outputs/`). Never rebuild the CSS/JS engine by
   hand — it is the reusable, tested part.
4. **Fill the header placeholders** (HTML comments in the file): `<!--PAGETITLE-->`,
   `<!--EYEBROW-->`, `<!--TITLE-->`, `<!--SUB-->`, `<!--PILL-->`, `<!--FOOTER-->`,
   `<!--DONEMSG-->`. Replace the text between each pair, keep the tags' structure
   (e.g. keep the `<span class="accent">…</span>` inside `<!--TITLE-->` for the
   coloured word, keep `<span class="dot"></span>` inside the pill).
5. **Replace the DATA block.** Between the `BEGIN DATA — EDIT THIS` and
   `END DATA — EDIT THIS` markers, replace the example `STEPS` array and `HELP`
   map with content generated from the source. Leave everything outside the
   markers untouched.
6. **Map the source faithfully.** Preserve the source's own numbering and
   sub-section structure (e.g. "5.1", "5.2") using `sec` blocks. Turn each
   concrete action ("click X", "select Y", "run Z") into a `flow` item so it
   becomes individually checkable — that granularity is what makes the page
   useful. Put the exact values to choose in `<span class='pick'>…</span>`, nav
   paths in `<span class='path'>…</span>`, literals/commands in `<code>` or
   `code` blocks. Lift any troubleshooting/FAQ content into the `HELP` map keyed
   by the step it relates to.
7. **Validate, then deliver.** Run the syntax check below, copy to
   `/mnt/user-data/outputs/`, and present with `present_files`.

## Validate before delivering

The most common breakage is a malformed `STEPS`/`HELP` array. Always run:

```bash
node -e "const h=require('fs').readFileSync('PATH.html','utf8');const m=h.match(/<script>([\s\S]*?)<\/script>/);new Function(m[1]);console.log('JS OK')"
```

If it errors, the data block has a syntax problem (usually an unescaped quote, a
stray backtick, or a `${` inside a `code` string). Fix and re-run.

### Gotchas that matter

- The `code` field is rendered with `textContent`, so `<` `>` `&` inside code are
  safe — do NOT pre-escape them. But avoid backticks and `${` in code strings,
  since the data is inside JS template literals (backtick-delimited). If the
  source code contains a backtick, switch that one `code` value to a normal
  JS string with escaped quotes, or replace the backtick.
- Class names matter: the badge number is `.fnum` and the step-header number is
  `.num` — they are intentionally different. Don't touch the engine's classes.
- Keep the localStorage key (`checklist_progress_v1`) unless the user wants two
  different checklists open without sharing saved state — then give each its own
  key string.
- This skill is content-only. Do **not** change the colour theme, fonts, or
  layout unless the user explicitly asks; the value is a consistent tool.

## Examples

**Trigger:** "Here's our onboarding runbook — make it something new hires can walk
through and tick off." → read the runbook, map each task to a `flow` item grouped
by phase, deliver the HTML.

**Trigger:** "Same setup-console thing you made before, but for our Postgres
migration steps." → copy the template, fill header for the migration, generate
`STEPS` from the migration steps, deliver.

**Trigger:** "Turn this 12-step deployment checklist into a trackable page with
copy buttons for the commands." → each command becomes a `code` block inside the
relevant step's `details`.
