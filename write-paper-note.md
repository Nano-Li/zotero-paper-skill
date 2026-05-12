---
id: write-paper-note
description: Generate a note for a single paper, or for a specific detailed question about a single paper. Use ONLY when the user explicitly asks to write, draft, or edit a paper note.
version: 2.2
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

**FRONTMATTER LOCK**: when YAML frontmatter is used for file-based notes, the 7 fields listed below (`title`, `citekey`, `doi`, `year`, `journal`, `created`, `tags`) are the COMPLETE AND EXCLUSIVE list. You are FORBIDDEN from adding any other frontmatter field. Explicitly forbidden frontmatter fields (non-exhaustive): `authors`, `note_type`, `figure`, `abstract`, `source`, `url`, `keywords`, `added`, `updated`, `status`, `rating`. If you want to record author names, figure labels, abstracts, or any other metadata, put them in the **body text** of the note, not in frontmatter. Do not invent new frontmatter fields under any circumstance.

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

## 1. 一句话总结

## 2. 摘要与结论

### 摘要

### 结论概括

## 3. 引言解读

### 逻辑链

### 与前人工作的比较

- 前人局限：
- 本文改进：

## 4. 实验细节

### 激光参数

| 参数 | 数值 | 备注 |
|---|---|---|

### 材料参数

| 参数 | 数值 | 备注 |
|---|---|---|

### 测量系统

| 参数 | 数值 | 备注 |
|---|---|---|

### 对我有用的参数

### 实验流程

## 5. 理论与模拟方法

### 理论方法

### 模拟方法

## 6. 机制与结果

### 作者提到的机制

### 我的思考

### 关键结果

## 7. 局限性

## 8. 我的笔记

## 9. 重要图表

```

### How to apply the template

- `{{paperTitle}}` is **the full title of the paper itself** (e.g., `"A toolbox for representational similarity analysis"`), looked up from Zotero metadata via `read_library(sections:['metadata'])`. Use the exact same value in both the `title:` frontmatter field and the `# heading`.
- **Filename and `title:` are independent fields.** The filename uses its own three-part pattern (see Step 4b) that MAY include the note subtopic and date; frontmatter `title:` never does. Never copy any part of the filename into `title:`.
- Fill in `{{created}}` with today's date in YYYY-MM-DD format. This is when the note was created, not when the paper was published (that's the `year` field), and not when the Zotero item was created.
- For **file-based notes**, required frontmatter fields that must always be present are `title`, `created`, and `tags`. Never omit these.
- For **file-based notes**, look-up frontmatter fields are `citekey`, `doi`, `journal`, and `year`. If a value is genuinely missing in Zotero metadata, use an empty string (e.g., `doi: ""`) rather than omitting the key — keep the frontmatter shape consistent.
- Keep the template section headings exactly as shown. Except for necessary proper nouns, abbreviations, units, equations, variables, and the required original-English excerpt paragraphs in `### 摘要` and `### 结论概括` (see the rule for `## 2. 摘要与结论` below), all filled note content MUST be Chinese.
- Use valid LaTeX for important physical parameters, equations, and symbolic variables. If `full.md` contains a correct LaTeX formula, preserve that LaTeX formula rather than rewriting it as plain text. Every LaTeX command inside `$...$` or `$$...$$` MUST include its leading backslash, e.g., `\times`, `\mathrm{}`, `\mu`, `\circ`, `\sim`, `\pm`, `\lambda`, `\theta`, `\sqrt{}`, and `\frac{}{}`. In Markdown pipe tables, check table-cell formulas especially carefully because the same backslash syntax is still required inside table cells.
- **`## 1. 一句话总结`**: write about one concise Chinese sentence summarizing the paper's main content.
- **`## 2. 摘要与结论`**: under `### 摘要`, write two paragraphs: first reproduce the original English abstract from the paper, then provide a concise Chinese translation or summary. Under `### 结论概括`, write two paragraphs: first excerpt the key original English conclusion sentences from the paper, then provide a concise Chinese summary. Only these first paragraphs may be English excerpts.
- **`## 3. 引言解读`**: under `### 逻辑链`, use bullet points, one concise Chinese sentence per bullet. Follow this bullet-list pattern: one bullet for the existing problem, one bullet for the proposed method, and one bullet for the obtained conclusion or result. Under `### 与前人工作的比较`, fill exactly two bullets: first the previous limitation, then this work's improvement. Each bullet should be one concise Chinese sentence and include key parameters when available.
- **`## 4. 实验细节`**: fill the three parameter tables strictly in the shown table format. Only write parameters explicitly reported by the paper. If a table has no relevant parameters, leave the table empty; do not write `N/A`, `None`, or inferred values. `### 对我有用的参数` must remain completely empty. Under `### 实验流程`, use short bullet points, one step per bullet. Example: `- 对准泵浦光和探测光，并校准延迟线。`
- **`## 5. 理论与模拟方法`**: focus on key model names, theory terms, simulation methods, boundary conditions, assumptions, and important model parameters. Do not write generic descriptions.
- **`## 6. 机制与结果`**: under `### 作者提到的机制`, state the physical mechanisms explicitly discussed by the authors. `### 我的思考` must remain completely empty. Under `### 关键结果`, summarize the experimental or computational results in one to two concise Chinese sentences.
- **`## 7. 局限性`**: use bullet points, one concise Chinese sentence per limitation. Avoid long generic paragraphs and do not invent limitations not supported by the paper.
- **`### 对我有用的参数`**, **`### 我的思考`**, and **`## 8. 我的笔记`**: always leave these three sections completely empty. They are reserved for the human researcher. Do not write prompts, placeholders, inferred ideas, suggestions, or comments.
- **`## 9. 重要图表`** must only contain figure embeds and figure explanations when the user explicitly asks about one or more specific figures. If the user did not ask about any figure, `## 9. 重要图表` MUST remain empty: keep the heading, but write nothing under it.

**Checklist before writing the note — verify each item:**
1. For Zotero notes, omit the YAML frontmatter entirely and start from `# {{paperTitle}}`. For file-based notes, include the YAML frontmatter.
2. For file-based notes, `title:` value is the paper's full title from Zotero - NOT the filename, NOT the figure/subtopic label, NOT the date.
3. For file-based notes, frontmatter contains exactly the 7 keys shown above, in that order, and NO others.
4. For file-based notes, you did not add `authors`, `note_type`, `figure`, `abstract`, or any other extra frontmatter field.
5. For file-based notes, `created:` is today's date in YYYY-MM-DD.
6. For file-based notes, `tags:` is present.
7. `### 逻辑链`, `### 实验流程`, and `## 7. 局限性` must all use bullet lists, with one concise Chinese sentence per bullet. Logic Chain example: `- 现有方法存在xxx问题。` `- 本文提出xxx方法。` `- 最终得到xxx结论。`
8. `### 对我有用的参数`, `### 我的思考`, and `## 8. 我的笔记` are completely empty.
9. All math expressions use valid LaTeX syntax. Check table cells especially carefully: every physical parameter written in LaTeX inside a Markdown pipe table MUST use correct backslash commands. Every LaTeX command has its leading backslash. Correct table-cell example:

   ```markdown
   | 参数 | 数值 | 备注 |
   |---|---|---|
   | NIR 强度 | $4\times10^{16}\ \mathrm{W/cm^2}$ | 泵浦光强 |
   | 焦斑尺寸 | $40\times40\ \mu\mathrm{m}^2$ | 反应区处 |
   ```
10. You identified the `{notetitle}` subtopic (figure label, section name, topic) separately — it goes into the filename in Step 4b, never into `title:`.
11. If the user did not ask about a specific figure, leave `## 9. 重要图表` empty; keep only the heading and do not invent figure explanations.

### Step 3 — Include figures

**If the user asked about a specific figure, you MUST include that figure in the note.** 

#### For Zotero notes (`edit_current_note`)

- Use `![Caption](file:///{mineruCacheDir}/images/filename.png)`. The `edit_current_note` tool auto-imports `file://` images as Zotero embedded attachments.
- Place figure embeds and their explanations under the `## 9. 重要图表` section.

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
- Use the note template above. For file-based notes, frontmatter is locked to the 7 fields shown; do not add or remove fields. For Zotero notes, omit frontmatter entirely.
- Use `[@citekey]` Pandoc syntax inline **only when `citekey` is non-empty**. When `citekey` is missing or empty, reference the paper in prose instead (`First-Author et al. (Year)`). **Never emit `[@]`.**
- Use the native path separator provided in the runtime platform section. Never mix separators.

### Budget
Total tool calls: 2–5 (read content, read metadata, optionally copy figures, write note).

<!-- LLM-FOR-ZOTERO:MANAGED-END -->
