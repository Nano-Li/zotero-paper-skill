---
id: write-paper-note
description: Generate a note for a single paper, or for a specific detailed question about a single paper. Use ONLY when the user explicitly asks to write, draft, or edit a paper note.
version: 2.1
match: /\b(create|make|write|draft|generate)\b.*\b(note|paper note|reading note|notes?)\b.*\b(for|from|about|on)\b.*\b(paper|article|this)\b/i
match: /\b(note|notes?)\b.*\b(for|from|about|on)\b.*\b(paper|article|this|these)\b/i
match: /\b(save|write|append|add|put)\b.*\b(to\s+)?(note|notes?)\b/i
match: /\b(note|notes?)\b.*\b(save|write|append|add)\b/i
match: /\b(edit|update|modify|rewrite|revise|polish)\b.*\b(note|notes?)\b/i
match: /\b(create|make|new)\b.*\bnote\b/i
match: /\b(write|save|export|send)\b.*\bobsidian\b/i
match: /\bobsidian\b.*\b(note|write|save|export)\b/i
match: /\bto\s+obsidian\b/i
match: /\bobsidian\b.*\bvault\b/i
match: /\b(save|write|export)\b.*\bnote\b.*\b(to\s+)?(file|disk|local|directory|folder)\b/i
match: /\b(note|notes?)\b.*\b(to\s+)?(file|disk|local|directory|folder)\b/i
match: /\b(use|apply|with)\b.*\btemplate\b/i
---

<!-- LLM-FOR-ZOTERO:MANAGED-BEGIN -->

## Write Paper Note

### Where to write

Decide the destination based on what the user says:

- **File-based** (`file_io`): user mentions Obsidian, the notes directory nickname, or "save to file/directory/folder".
- **Zotero note** (`edit_current_note`): user says "create a note", "save to note", "edit my note", or anything without a file destination.

If unclear, default to Zotero note.

### Step 1 — Read content

- If `mineruCacheDir` is available: use `file_io(read, '{mineruCacheDir}/full.md')`.
- Otherwise: use `read_paper` for the overview, then optionally one `search_paper` call for key results/methods if the user wants detail beyond the abstract.
- Keep the read phase minimal: 1 call (MinerU) or 1–2 calls (read_paper/search_paper). Do not read the entire paper section by section.

### Step 2 — Compose the note using the template below

Look up `title` (the paper's full title), `citekey`, `doi`, `journal`, `year`, and **authors** from Zotero item metadata via `read_library(sections:['metadata'])`. Cite papers using **Pandoc citation syntax** `[@citekey]` **only when `citekey` is non-empty**. If `citekey` is missing or empty (common when Better BibTeX is not installed), reference the paper in prose instead (`First-Author et al. (Year)`). **Never emit `[@]`** - an empty citation is a bug.


For **Zotero notes** (`edit_current_note`): omit the YAML frontmatter block entirely. Use only the heading and section structure.

For **file-based notes** (`file_io`): include the full template with YAML frontmatter.

## Note template

Use this template **exactly**.

**FRONTMATTER LOCK**: the 7 fields listed below (`title`, `citekey`, `doi`, `year`, `journal`, `created`, `tags`) are the COMPLETE AND EXCLUSIVE list. You are FORBIDDEN from adding any other field. Explicitly forbidden (non-exhaustive): `authors`, `note_type`, `figure`, `abstract`, `source`, `url`, `keywords`, `added`, `updated`, `status`, `rating`. If you want to record author names, figure labels, abstracts, or any other metadata, put them in the **body text** of the note, not in frontmatter. Do not invent new fields under any circumstance.

```
---
title: "{{paperTitle}}"
citekey: "{{citekey}}"
doi: "{{doi}}"
year: {{year}}
journal: "{{journal}}"
created: {{created}}
tags: [zotero, paper-note]
---

# {{paperTitle}}

## Summary
Summarize the paper's main content in 1-2 sentences.

## Notes
### 逻辑链
Explain the logical support that leads to the paper's conclusions.

### 理论
Explain the theoretical framework or assumptions used by the paper.

### 数值计算方法
Describe the numerical or computational methods used by the paper, if any.

### 实验方法
Describe the experimental design, protocol, data, or empirical setup, if any.

## 创新点
Explain the core innovations that distinguish this paper from related work.

## 图解

```

### How to apply the template

- For **paper notes**, `{{paperTitle}}` is **the full title of the paper itself** (e.g., `"A toolbox for representational similarity analysis"`), looked up from Zotero metadata via `read_library(sections:['metadata'])`. Use the exact same value in both the `title:` frontmatter field and the `# heading`.
- **Filename and `title:` are independent fields.** The filename uses its own three-part pattern (see Step 4b) that MAY include the note subtopic and date; frontmatter `title:` never does. Never copy any part of the filename into `title:`.
- Fill in `{{created}}` with today's date in YYYY-MM-DD format. This is when the note was created, not when the paper was published (that's the `year` field), and not when the Zotero item was created.
- **Required fields that must always be present**: `title`, `created`, `tags`. Never omit these.
- **Look-up fields**: `citekey`, `doi`, `journal`, `year`. If a value is genuinely missing in Zotero metadata, use an empty string (e.g., `doi: ""`) rather than omitting the key — keep the frontmatter shape consistent.
- Keep the template section headings exactly as shown. The headings are intentionally user-facing Chinese labels, while the explanatory text in this skill is written in English for instruction clarity.
- **`图解`** must only contain figure embeds and figure explanations when the user explicitly asks about one or more specific figures. If the user did not ask about any figure, `图解` MUST remain empty: keep the `## 图解` heading, but write nothing under it.

**Checklist before writing the note — verify each item:**
1. `title:` value is the paper's full title from Zotero - NOT the filename, NOT the figure/subtopic label, NOT the date.
2. Frontmatter contains exactly the 7 keys shown above, in that order, and NO others.
3. You did not add `authors`, `note_type`, `figure`, `abstract`, or any other field.
4. `created:` is today's date in YYYY-MM-DD.
5. `tags:` is present.
6. If the user did not ask about a specific figure, leave `## 图解` empty; keep only the `## 图解` heading and do not invent figure explanations.
7. You identified the `{notetitle}` subtopic (figure label, section name, topic) separately — it goes into the filename in Step 4b, never into `title:`.

### Step 3 — Include figures

**If the user asked about a specific figure, you MUST include that figure in the note.** 

#### For Zotero notes (`edit_current_note`)

- Use `![Caption](file:///{mineruCacheDir}/images/filename.png)`. The `edit_current_note` tool auto-imports `file://` images as Zotero embedded attachments.
- Place figure embeds and their explanations under the `## 图解` section.

#### For file-based notes (`file_io`)

**Hard rules — these embeds do NOT render inline in Obsidian or most markdown viewers. NEVER produce them:**

- NEVER use `file:///...` URLs. They are blocked inline for security.
- NEVER use absolute filesystem paths like `/Users/...`, `~/...`, or `C:\...`.
- NEVER use `|width` syntax inside `![alt](url)`. Width suffixes only work inside `![[wikilink]]` embeds, which we do NOT use here. The `|` ends up as literal text and the image is not resized.

**Algorithm — follow exactly:**

1. Create the destination directory: `run_command` with `mkdir -p "{attachmentsPath}/{sanitized-paper-title}"`. The folder is named after the **paper title only** (no subtopic, no date) so multiple notes about the same paper share the same images folder.
2. Copy image files from `{mineruCacheDir}/images/` to `{attachmentsPath}/{sanitized-paper-title}/` using `run_command`. Copy images BEFORE writing the note file.
3. Compute the **relative path from the note's directory to the image file**. Use `..` to climb to the common ancestor, then descend to the image. Count path segments deterministically — don't guess.
4. Embed with `![<caption>](<relative-path>)`. Nothing else.

**Worked example:**

```
Note path:    {vault}/Logs/paper-notes/Nili2014.md
Image path:   {vault}/Logs/imgs/Nili2014/figure-2.jpg
Note folder:  {vault}/Logs/paper-notes/
Relative:     ../imgs/Nili2014/figure-2.jpg
Write:        ![Figure 2. RSA toolbox schematic](../imgs/Nili2014/figure-2.jpg)
```

**Negative examples — never produce any of these:**

- `![Figure 2](file:///Users/.../figure-2.jpg)` — `file://` renders as a broken-image icon in Obsidian.
- `![Figure 2](/Users/.../figure-2.jpg)` — absolute path is outside the vault; viewers refuse.
- `![Figure 2|400](../imgs/foo.jpg)` — `|400` becomes literal alt text; image is not resized.
- `![[imgs/foo/figure-2.jpg]]` — wiki-link embed; we use standard markdown only.

**If the figure image cannot be found** in the MinerU cache, tell the user clearly. Do NOT fall back to `file:///`, absolute paths, or any of the negative examples above.

### Step 4a — Write to Zotero (`edit_current_note`)

**Creating notes** (mode: `create`):
- Notes are created directly without a confirmation card.
- In **paper chat** (active item exists): default to `target: 'item'` — attaches the note to the active paper.
- In **library chat** (no active item): do not create a standalone note. Ask the user to open or specify the target paper first.
- If the paper already has a single child note, the tool auto-appends your content with an `<hr/>` separator. Just call `edit_current_note(mode:'create')`.
- If the paper has **multiple** child notes and the user wants to append, ask which note to write to before proceeding.

**Editing existing notes** (mode: `edit`):
- Edits always show a diff review card for the user to approve.
- PREFER `patches` (find-and-replace pairs) over `content` (full rewrite) — patches are faster.
- Use mode `edit` for: append to specific position, insert, delete, rewrite sections.

**Format:**
- Pass Markdown by default. When the user explicitly requests HTML output or provides an HTML template (e.g., Better Notes templates with inline styles), write HTML with inline styles directly.

### Step 4b — Write to file (`file_io`)

**Prerequisites:**
- The user's notes directory path and default folder are provided in the system prompt under "Notes directory configuration". If missing, tell the user to configure the notes directory in the plugin preferences (Settings > Agent tab).
- The default folder is used when the user doesn't specify a folder. If the user specifies a different folder, write there instead.

**Filename pattern (default):** `{papertitle}-{notetitle}-{date}.md`

Three components, joined by single hyphens:

- **`{papertitle}`** — sanitized paper title from Zotero metadata.
- **`{notetitle}`** — the specific aspect or subtopic the note covers. Derive it from the user's request:
  - "notes on figure 1" / "summarize figure 3" → `figure-1` / `figure-3`
  - "methodology summary" / "methods notes" → `methodology`
  - "discussion notes" / "notes on the discussion" → `discussion`
  - "key findings" / "summarize the findings" → `key-findings`
  - "summarize this paper" / "reading notes for this paper" (no specific aspect) → **omit `{notetitle}` entirely**, along with the hyphen that would precede it.
- **`{date}`** — today's date in `YYYY-MM-DD`. This is the same value you write into frontmatter `created:` — reuse it, don't look up a different date.

**Sanitizer for each component** — lowercase, replace any run of non-alphanumeric characters with a single hyphen, strip leading/trailing hyphens, collapse consecutive hyphens to one, trim each component to ~80 characters.

**Never double up hyphens** when a component is omitted. `{papertitle}--{date}.md` is wrong; the correct form is `{papertitle}-{date}.md`.

**Worked examples:**

| User request | Filename |
|---|---|
| "summary notes about figure 1 to my obsidian note" | `stable-and-dynamic-coding-for-working-memory-figure-1-2026-04-16.md` |
| "create a reading note for this paper" | `stable-and-dynamic-coding-for-working-memory-2026-04-16.md` |
| "methodology summary" | `stable-and-dynamic-coding-for-working-memory-methodology-2026-04-16.md` |

**Writing steps:**

1. Construct the file path: `{notesDirectoryPath}/{folder}/<filename>.md`, using the native path separator from the runtime platform section.
2. Call `file_io(write, filePath, noteContent)`.
3. If writing fails, report the error clearly with the attempted path.

**Filename is independent of frontmatter.** The frontmatter `title:` stays the paper's full title per the template. Do NOT put the subtopic or the date into `title:`.


### Key rules

- **Never** output the full note text in chat. Always use `edit_current_note` or `file_io`.
- Use the note template above — frontmatter is locked to the 7 fields shown; do not add or remove fields.
- Use `[@citekey]` Pandoc syntax inline **only when `citekey` is non-empty**. When `citekey` is missing or empty, reference the paper in prose instead (`First-Author et al. (Year)`). **Never emit `[@]`.**
- Use the native path separator provided in the runtime platform section. Never mix separators.

### Budget
Total tool calls: 2–5 (read content, read metadata, optionally copy figures, write note).

<!-- LLM-FOR-ZOTERO:MANAGED-END -->
