---
id: fix-note-latex
description: Fix malformed LaTeX math syntax inside table cells in a Zotero note attached to the current item. Use ONLY when the user explicitly asks to repair, fix, correct, polish, or normalize LaTeX/formulas/math in note tables.
version: 1
match: /\b(fix|repair|correct|normalize|polish|clean)\b.*\b(latex|math|formula|equation|公式)\b.*\b(table|tables|表格)\b/i
match: /\b(table|tables|表格)\b.*\b(latex|math|formula|equation|公式)\b.*\b(fix|repair|correct|normalize|polish|clean)\b/i
match: /\b(note|notes?|笔记)\b.*\b(table|tables|表格)\b.*\b(latex|math|formula|equation|公式)\b/i
match: /\b(latex|math|formula|equation|公式)\b.*\b(note|notes?|笔记)\b.*\b(table|tables|表格)\b/i
---



## Fix Note Table LaTeX

### Goal

Repair malformed LaTeX commands inside table cells in an existing Zotero note.

This skill is a narrow repair pass. Do not summarize the paper, rewrite the note, change table structure, or edit prose outside table-cell math.

### Step 1 — Find the target note

To find notes attached to the current Zotero item, use the active item ID from the Zotero context and call read_library with sections: ["notes"]. Then count and inspect the returned child notes array for that item.

- If no child notes are found, tell the user that the current item has no attached notes to repair.
- If exactly one child note is found, use that note as the target.
- If multiple child notes are found, do NOT edit any note. List every candidate note with its `noteId` and title, then ask the user to choose one.
- If the user already provided or confirmed a `noteId`, use that note as the target.

For editing a known Zotero note by `noteId`, MUST use `zotero_script(mode: "write")` with this edit path:

`zotero_script(mode: "write")`
→ `Zotero.Items.getAsync(noteId)`
→ `item.isNote()`
→ `item.getNote()`
→ `env.snapshot(item)`
→ modify note HTML
→ `item.setNote(newHtml)`
→ `await item.saveTx()`

This path directly targets the specified note item and does not depend on Zotero UI focus or active note state.

### Step 2 — Inspect only note tables

Read the target note content and identify note tables only.

Zotero notes are commonly stored as HTML. In that case, inspect only `<table>` elements and their `<td>`/`<th>` cells. If the note content is Markdown rather than HTML, inspect Markdown pipe tables instead. A Markdown pipe table is a block of consecutive lines that contains `|` column separators and a separator row such as `|---|---|`.

Scope limits:

- Repair only math inside table cells.
- Repair only math spans delimited by `$...$` or `$$...$$` inside those table cells.
- Do NOT edit prose outside tables.
- Do NOT edit non-table formulas.
- Do NOT add new values, units, rows, columns, captions, explanations, or citations.
- Do NOT convert tables into bullet lists.

### Step 3 — Repair LaTeX command backslashes

Inside table-cell math spans only, add missing leading backslashes to malformed LaTeX commands.

Common repairs:

| Malformed | Correct |
|---|---|
| `times` | `\times` |
| `mathrm{...}` | `\mathrm{...}` |
| `mu` before units | `\mu` |
| `circ` | `\circ` |
| `sim` | `\sim` |
| `pm` | `\pm` |
| `lambda` | `\lambda` |
| `theta` | `\theta` |
| `beta` | `\beta` |
| `tau` | `\tau` |
| `sqrt{...}` | `\sqrt{...}` |
| `frac{...}{...}` | `\frac{...}{...}` |
| `AA` for angstrom units | `\AA` |

Examples of allowed repairs:

```markdown
$4times10^{16} mathrm{W/cm^2}$ -> $4\times10^{16}\ \mathrm{W/cm^2}$
$40times40 mumathrm{m}^2$ -> $40\times40\ \mu\mathrm{m}^2$
$3^circ$ -> $3^\circ$
$sim300 mathrm{eV}$ -> $\sim300\ \mathrm{eV}$
$qsim1.78 mathrm{AA^{-1}}$ -> $q\sim1.78\ \mathrm{\AA^{-1}}$
```

### Step 4 — Write the minimal edit

Use `zotero_script(mode: "write")` to edit the known note item by `noteId`.

- Patch only the malformed table-cell math strings.
- Preserve all other note HTML or Markdown byte-for-byte where possible.
- Do not output the full repaired note in chat.
- If no malformed table-cell LaTeX is found, tell the user no repair was needed and do not edit the note.

### Checklist before editing

1. You identified the target note. If multiple notes exist, you listed `noteId` and title and waited for the user to choose.
2. Every planned edit is inside an HTML table cell (`<td>`/`<th>`) or a Markdown pipe table cell.
3. Every planned edit is inside `$...$` or `$$...$$`.
4. Every planned edit only adds missing LaTeX backslashes or spacing needed for valid LaTeX.
5. No prose, rows, columns, headings, citations, or non-table formulas are changed.
6. Edit by `noteId` with `zotero_script(mode: "write")`, not by active UI note state.

### Budget

Total tool calls: 2–4 (read notes, optionally ask user to choose, edit note).
