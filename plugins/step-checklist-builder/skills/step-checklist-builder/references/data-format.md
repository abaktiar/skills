# Data format reference

The page is driven by two JS values inside the `BEGIN DATA … END DATA` markers in
`assets/template.html`: a `STEPS` array and a `HELP` object. Nothing else needs to
change to retarget the page to a new topic.

## Contents
- The `STEPS` array
- Step object fields
- `details[]` block types
- Inline HTML helpers (highlighting)
- The `HELP` map
- Worked example
- Rules & pitfalls

---

## The `STEPS` array

An ordered array of step objects. Render order = array order. Steps are visually
grouped under a heading whenever the `g` value changes, so keep steps that share a
group adjacent and give them the identical `g` string.

## Step object fields

| Field | Required | Meaning |
|---|---|---|
| `g` | yes | Group heading. Reuse the same string across consecutive steps to group them (e.g. `"Provisioning"`). |
| `num` | yes | The large left-hand number, any string: `"00"`, `"01"`, `"14"`. Purely a label. |
| `title` | yes | Step title shown in the card header. |
| `time` | yes | Short meta string, e.g. `"10 min"`, `"setup"`. Shows with a clock glyph. |
| `tags` | no | Array. `"wait"` → amber "Has wait time" badge **and** inclusion in the "Has wait time" filter. `"optional"` → blue "Optional" badge. |
| `details` | yes | Array of content blocks (below). The `flow` blocks within become the checkable items. |

A step is "done" when every `flow` item across all its `details` is checked. Steps
with **no** `flow` items can still be marked done via the "Mark all done" button but
have nothing to tick — give every step at least one `flow` block.

## `details[]` block types

Render top-to-bottom in array order. Mix freely.

### `sec` — sub-section header
```js
{t:'sec', tag:'5.1', label:'AgentCore agent role'}
```
`tag` is optional (shows in accent colour). Use to mirror the source's own
numbering so a long step reads as labelled sub-procedures.

### `p` — paragraph
```js
{t:'p', html:"Intro or explanation. May contain <b>, <code>, <span class='pick'>, <span class='path'>, <i> ..."}
```

### `flow` — numbered, checkable walkthrough  ← the important one
```js
{t:'flow', items:[
  "First action. Tap the number to tick it.",
  "Pick <span class='pick'>Symmetric</span> then click <strong>Next</strong>.",
  "Open <span class='path'>KMS → Customer managed keys</span>."
]}
```
Each string is one tickable step. Numbering runs continuously across **all** flow
blocks within a step (1..N), resetting only at the next step. Make each item a
single concrete action; that granularity is the product.

### `tbl` — field/value table
```js
{t:'tbl', head:["Setting","Value"], rows:[
  ["Key type","<span class='pick'>Symmetric</span>"],
  ["Alias","mepco-analytics"]
]}
```
Cells accept inline HTML. Use for settings/option reference, not for actions.

### `code` — copyable code block
```js
{t:'code', lang:'bash', code:'aws s3 cp ./x s3://bucket/ --recursive'}
```
`lang` is just a label shown in the code header. The body is rendered with
`textContent`, so `< > &` are safe and must NOT be escaped. Avoid backticks and
`${` inside `code` (the data sits in JS template literals). Multiline is fine —
use real newlines inside a normal JS string with `\n`, or a template literal only
if the code contains no backticks.

### `note` — callout
```js
{t:'note', k:'tip', html:"Green tip callout."}
{t:'note', k:'',    html:"Neutral amber callout for an important aside."}
```

## Inline HTML helpers (use inside `p`, `flow`, `tbl`, `note`)

| Markup | Use |
|---|---|
| `<span class='pick'>value</span>` | Green emphasis for the exact option/value to choose. |
| `<span class='path'>Menu → Sub</span>` | Amber pill for a navigation path / location. |
| `<code>literal</code>` | Inline code, identifiers, resource names, short commands. |
| `<strong>Label</strong>` | Button names, screen names, emphasis. |
| `<i>word</i>` | Light emphasis. |

Note: when a `flow` item is checked, `strong`, `pick`, and `u` text inside it gets
struck through automatically — write items so the key terms sit in those tags.

## The `HELP` map

Optional. Keys are step `num` strings. The value is an array of HTML strings shown
as bullets in a red "If you're stuck on this step" panel that appears when the
reader taps **Need help here** on that step.

```js
const HELP = {
  "01":["<b>AccessDenied</b> → re-check permissions.","<b>Wrong region</b> → switch to the target region."],
  "06":["<b>Sync stuck</b> → remove the corrupt file and re-sync."]
};
```
Map each piece of troubleshooting/FAQ content from the source to the step it most
relates to. Omit the key entirely for steps with no troubleshooting.

---

## Worked example (minimal, two steps)

```js
const STEPS = [
{g:"Getting started", num:"00", title:"Prerequisites", time:"setup", details:[
  {t:'p',html:"What you need before starting."},
  {t:'flow',items:[
    "Confirm you have admin access.",
    "Install the CLI and run <code>tool login</code>.",
    "Set the region to <span class='pick'>us-east-2</span>."
  ]},
  {t:'note',k:'tip',html:"Pin the region — it's easy to drift."}
]},
{g:"Main steps", num:"01", title:"Create the resource", time:"10 min", tags:["wait"], details:[
  {t:'sec',tag:'1.1',label:'Create it'},
  {t:'flow',items:[
    "Open <span class='path'>Console → Resources → Create</span>.",
    "Type = <span class='pick'>Standard</span> → <strong>Next</strong>.",
    "Name it <code>my-resource</code> → <strong>Create</strong>."
  ]},
  {t:'tbl',head:["Setting","Value"],rows:[["Type","<span class='pick'>Standard</span>"],["Name","my-resource"]]},
  {t:'code',lang:'bash',code:'tool create resource --name my-resource --type standard'},
  {t:'note',k:'',html:"Provisioning takes a few minutes."}
]}
];

const HELP = {
  "01":["<b>Name taken</b> → append a unique suffix and retry."]
};
```

## Rules & pitfalls

- Keep the engine (everything outside the DATA markers) byte-for-byte unchanged.
- After editing, validate the script parses (see SKILL.md). A single bad quote
  breaks the whole page silently.
- Don't pre-escape `< > &` in `code` bodies; do avoid backticks / `${` there.
- Give every step ≥1 `flow` block so it has checkable items.
- Mirror the source's structure and numbering; don't invent or reorder steps.
