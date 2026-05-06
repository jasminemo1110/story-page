---
name: story-page
description: Generate a Claude-styled HTML "story page" recording the arc of a vibe-coding project — from the user's original idea to key design decisions — formatted as a curated dialogue between the project owner and Claude. Use when the user invokes /story or asks to "make a story page / 故事页 / 复盘页面" for the current project. Output is editable plain HTML with all text in a top-level JSON block.
---

# Story Page Skill

Generate an HTML page that retells the creative arc of the user's project as a stylized dialogue between the **project owner** (the user) and **Claude**, intended for short-video recording.

## ⚠️ RULE 0 — MUST READ BEFORE GENERATING

**Before doing anything else, you MUST use the Read tool to load `template.html` from this skill folder (`~/.claude/skills/story-page/template.html`).**

The output story page must be **structurally byte-identical** to that template — same HTML, same CSS, same render JS — with **only** the contents of `<script id="story-data" type="application/json">...</script>` replaced.

Do NOT generate the HTML from memory, from training data, or from "what a story page should look like." Past versions of this skill had a different UI (e.g. a gradient line under chapter titles, name labels in bubbles, a different cover tag) — those are wrong. Only the current `template.html` is the source of truth.

How to write the output file correctly:
1. `Read` `~/.claude/skills/story-page/template.html`
2. Compose the JSON content for the project's story (per the rules below)
3. `Write` the destination file using the template's exact HTML/CSS/JS, with your JSON spliced into the `<script id="story-data">` block

If you find yourself typing CSS rules or render-JS by hand, you are doing it wrong. Stop and re-read the template.

## Trigger

- `/story` (no args) → auto-extract the story from the current project
- `/story <prompt>` → user provides direction, e.g. `/story 重点讲跑道概念的诞生` or `/story 只讲后半段，从旅行账本开始`

## Output

Write a single self-contained HTML file. The output directory is `<project-root>/files/` if that dir exists, otherwise `<project-root>/`.

**Filename rule (series-aware):**
- First time: `story.html`
- Second time: `story-2.html`
- Third time: `story-3.html`
- ...etc.

Detect existing volumes by globbing `story.html` and `story-*.html` in the output directory. Never overwrite an earlier volume — always create the next one. (See **Incremental generation** below for how to fill it.)

The file is built from `template.html` (in this skill folder). The template separates **content (JSON)** from **style (CSS + render JS)**. You only need to write the JSON `<script id="story-data">` block — leave everything else identical to the template.

## Step-by-step

### 0. Load the template

`Read` `~/.claude/skills/story-page/template.html` first. Do not skip. Everything below assumes you have it open.

### 1. Gather material (do this even when the user gives a prompt — never skip)

In parallel:
- `git log --oneline -50` for the commit arc, plus `git log --reverse --format="%ai %s" | head -5` to find the project's earliest commit (used for the `meta` month)
- Read `CLAUDE.md` if present (project intent, design decisions)
- Read `README.md` if present
- Glance at the main entry file(s) to understand what the project actually does
- **Read recent session transcripts** at `~/.claude/projects/<project-path-hash>/*.jsonl` (sort by mtime, take the last 2–3 sessions). Project path hash is the cwd with `/` replaced by `-`. These contain real user phrasings and the actual back-and-forth — use them to recover authentic dialogue when possible. If unreadable or absent, fall back to default voice (see step 4).
- **Check for prior volumes** in the output directory (see **Incremental generation** below). If any exist, read the latest one's `<script id="story-data">` JSON to extract: prior `title`, `cover_tag`, `user_name`, `created_by`, all prior chapter titles, and the highest chapter number used.
- If the user gave a prompt, treat it as a **direction filter** (e.g., "重点讲跑道概念" / "已讲过的不要重复" / "突出我父母在用这点"). The prompt overrides defaults but doesn't replace material gathering.

### 1b. Incremental generation (when prior volumes exist)

If `story.html` (and possibly `story-2.html`, etc.) already exists in the output directory, this run is a **continuation**, not a fresh story.

Rules:
- **Do not repeat** chapter topics already covered in prior volumes. Use prior chapter titles as a "do not repeat" list. New chapters must come from material **after** the latest volume's creation (later commits, later sessions, new design decisions).
- **Inherit** these fields from the latest prior volume: `cover_tag`, `title`, `user_name`, `user_name_full`, `created_by`. Do not change them — same series should look like the same series.
- **Refresh** these fields: `subtitle` (reflects the new chapter focus), `meta` (use current month, refresh tags if focus shifted).
- **Set** `volume` to (highest existing volume + 1). E.g., if `story.html` and `story-2.html` exist, set `volume: 3`.
- **Set** `chapter_start` to (highest chapter number across all prior volumes + 1). Chapter numbering must be **continuous across the series** — if Vol.1 ended at CHAPTER 07, Vol.2 starts at CHAPTER 08.
- If there isn't enough new material for at least 3 chapters, **stop and tell the user** rather than padding. Suggest they wait until more development happens.

### 2. Extract chapter candidates

Look for:
- The **original problem / pain point** that motivated the project (almost always Chapter 1)
- **Architectural commitments** ("no database", "single file", "local-only") — each is usually one chapter
- **"Aha" moments / pivots** — places where the design changed because of a realization
- **Signature features** that define what makes this project different
- The **current state** or what's next (optional closing chapter)

Reject:
- Pure bug fixes
- Refactors that don't change the user-facing concept
- Anything purely technical with no narrative weight

### 3. Pick 4–7 chapters

Aim for 4–7. Fewer than 4 feels thin; more than 7 loses the viewer. Each chapter should map to **one decision or one realization**, not one feature or one commit.

Chapter titles should be **noun-phrase or short evocative phrases** in Chinese, like:
- ✅ "最初的疑问" / "最小可用的形状" / "跑道的诞生" / "一个意外的需求"
- ❌ "添加数据库" / "实现旅行账本功能" / "Bug 修复"

### 4. Write the dialogue

Per chapter: **1–3 turns** of dialogue.

**Default authorship rule (important)**: the user is the project owner and the source of ideas. Default every new direction, every "what if we...", every constraint to **them** as the originator. Claude's role is to respond, validate, surface tradeoffs, or expand. This reflects how the project actually was built — the user knew what they wanted, Claude helped implement.

**Avoid the boring Q&A pattern.** Vary the dialogue shapes across chapters:
- 用户 提出方向 → Claude 给出方案 (经典型)
- 用户 直接断言/吐槽 → Claude 点头并补一个细节 (验证型)
- 用户 提出方向 → Claude 给 ABC 三个备选 → 用户 拍板 (协商型)
- 用户 提出反对/疑问 → Claude 让步或澄清 (修正型)
- 用户 临时想到一个新需求 → Claude 顺势接住 (插入型)

A good story page uses **at least 2–3 different shapes** across its chapters.

**Sourcing real dialogue**: If session transcripts (step 1) contain a moment that fits a chapter, paraphrase the user's actual phrasing rather than inventing one. Their real voice is the gold standard.

**Voice guidance**:
- 用户: direct, inquisitive, sometimes pushes back. Will often state preference before asking "可以吗？"
- Claude: concise, surfaces tradeoffs, ends with a concrete recommendation
- Use `*关键词*` for orange-highlighted emphasis — **works on both sides**, not just Claude
- Avoid jargon dumps. Audience is short-video viewers, not engineers.

Use **narrate** (旁白) at most once per chapter, only when context is needed before dialogue starts.

Use **note** (批注) sparingly — not every chapter needs one. Insert when a design decision deserves a meta-comment ("this decision shaped everything that came after").

### 5. Fill the JSON

Open `template.html`, replace the `<script id="story-data">` block content. **Do not modify the CSS or the render JS.**

Schema:
```json
{
  "cover_tag": "XX × Claude Code 共创",
  "title": "故事标题（用 {accent: \"词\"} 包裹想橘色高亮的词）",
  "subtitle": "副标题，一句话定调",
  "meta": "2026 年 4 月 · 标签 · 标签",
  "volume": null,
  "chapter_start": 1,
  "user_name": "我",
  "user_name_full": "项目主人的名字",
  "created_by": "",
  "chapters": [
    {
      "title": "章节名",
      "narrate": "旁白（可省略）",
      "turns": [
        { "who": "user",   "text": ["第一段", "第二段"] },
        { "who": "claude", "text": ["回应内容里可以用 *关键词* 标橘色"] }
      ],
      "note": "批注（可省略）"
    }
  ]
}
```

Notes:
- `text` is always an array; each item is one paragraph.
- `who` is `"user"` or `"claude"`.
- For `cover_tag`, `user_name`, `user_name_full`, `created_by`: try to derive from git config (`git config user.name`) or ask the user for their preferred display name on the first run. Fall back to `"我"` / `"我 × Claude Code 共创"` if no name is available. On continuation runs (Vol.2+), inherit these from the prior volume — do not re-ask.
- `volume`: omit (or `null`) for Vol.1. Set to `2`, `3`, ... for continuation runs. Renders as "Vol.2" badge next to the title.
- `chapter_start`: omit (or `1`) for Vol.1. For continuations, set to (highest prior chapter number + 1) so chapter numbers continue across the series.
- `created_by`: optional. If set, renders as "Created by XXX" in the page footer.
- `closing`: the page's closing line above the watermark. Write it in the **same language as the rest of the content**, and include today's date. Examples:
  - Chinese: `"故事讲到 2026 年 5 月 5 日 · 未完待续"`
  - English: `"Story stands as of May 5, 2026 · to be continued"`
  - Other: use the conventional date format and "to be continued" idiom of that language.
  Always stamp today's actual date — this freezes the generation moment into the page.
- `meta` has **3 tags** (optionally 4), separated by ` · `:
  1. **时间** — birth month from earliest git commit, formatted as `2026 年 4 月`
  2. **技术** — primary tech stack (e.g. `React + FastAPI`, `Next.js + SQLite`)
  3. **总结** — one short noun phrase summarizing what the project does (e.g. `自驾地图`, `个人记账`)
  4. *(optional)* — a 4th tag only if there's a meaningful angle not covered above
- `*emphasis*` works on both user and claude sides — use it on either to highlight key phrases.
- `{br}` inside `title` inserts a manual line break (renders as `<br>`). Useful when an auto-wrapped title breaks at an awkward word — author chooses where the break lands.
- **Narrate text** (`narrate` field) is written in first person from the user's perspective — always use `"我"` rather than the user's name (e.g. `"我的父母退休后迷上了自驾"` not `"<姓名>的父母退休后迷上了自驾"`).

### 6. Report back

After writing the file, report to the user:

1. The output path
2. **A numbered list of chapter titles you chose**
3. **2–3 lines on what you deliberately left out** (e.g., "舍掉了：早期的 Excel 导出脚本、合并账本逻辑——前者偏工具向，后者过于技术细节")
4. Invite: "想加一章讲 ___ 或者改某章的取舍，告诉我即可。"

This list is the user's hook for catching omissions — it is **mandatory**.

## Style rules (do not violate)

- **Match the project's primary language.** Detect from `README.md` / `CLAUDE.md` / recent session transcripts / source comments. If the project is Chinese-led (CN docs, CN dialogue), generate in Chinese. If English-led, generate in English. Same for other languages. When in doubt, default to Chinese (since this skill was authored in a Chinese context). Apply this to all narrative text: title, subtitle, meta, narrate, dialogue, note, **and the `closing` field**.
- **Claude 橘 `#D97757`** is the brand color — never substitute green/blue/etc.
- Page is for **short-video recording** — content should look good when paused mid-frame, not require scrolling to make sense
- Keep the JSON readable — one chapter per object, generous newlines, no minification
- Do not add new HTML structure beyond what the template's render JS supports. If you need a new content type, that's a follow-up to discuss with the user — don't invent it inline.

## Example

See `example.html` in this skill folder — the canonical reference (the 记账 project's story page).
