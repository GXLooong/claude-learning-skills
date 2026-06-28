---
name: karpathy-llm-wiki
description: "Use when building or maintaining a personal LLM-powered knowledge base. Triggers: ingesting sources into a wiki, querying wiki knowledge, linting wiki quality, 'add to wiki', 'what do I know about', or any mention of 'LLM wiki' or 'Karpathy wiki'."
---

# Karpathy LLM Wiki

Build and maintain a personal knowledge base using LLMs. You manage two directories: `raw/` (immutable source material) and `wiki/` (compiled knowledge articles). Sources go into raw/, you compile them into wiki articles, and the wiki compounds over time.

Core ideas from Karpathy:
- "The LLM writes and maintains the wiki; the human reads and asks questions."
- "The wiki is a persistent, compounding artifact."

## Architecture

Three layers, all under the user's project root:

**raw/** — Immutable source material. You read, never modify. Organized by topic subdirectories (e.g., `raw/machine-learning/`).

**wiki/** — Compiled knowledge articles. You have full ownership. Organized by topic subdirectories, one level only: `wiki/<topic>/<article>.md`. Contains two special files:
- `wiki/index.md` — Global index. One row per article, grouped by topic, with link + summary + Updated date.
- `wiki/log.md` — Append-only operation log.

**SKILL.md** (this file) — Schema layer. Defines structure and workflow rules.

Templates live in `references/` relative to this file. Read them when you need the exact format for raw files, articles, archive pages, or the index.

### Initialization

Triggers only on the first Ingest. Check whether `raw/` and `wiki/` exist. Create only what is missing; never overwrite existing files:

- `raw/` directory (with `.gitkeep`)
- `wiki/` directory (with `.gitkeep`)
- `wiki/index.md` — heading `# Knowledge Base Index`, empty body
- `wiki/log.md` — heading `# Wiki Log`, empty body

If Query or Lint cannot find the wiki structure, tell the user: "Run an ingest first to initialize the wiki." Do not auto-create.

---

## Ingest

Fetch a source into raw/, then compile it into wiki/. Always both steps, no exceptions.

### Fetch (raw/)

1. Get the source content using whatever web or file tools your environment provides. If nothing can reach the source, ask the user to paste it directly.

2. Pick a topic directory. Check existing `raw/` subdirectories first; reuse one if the topic is close enough. Create a new subdirectory only for genuinely distinct topics.

3. Save as `raw/<topic>/YYYY-MM-DD-descriptive-slug.md`.
   - Slug from source title, kebab-case, max 60 characters.
   - Published date unknown → omit the date prefix from the file name (e.g., `descriptive-slug.md`). The metadata Published field still appears; set it to `Unknown`.
   - If a file with the same name already exists, append a numeric suffix (e.g., `descriptive-slug-2.md`).
   - Include metadata header: source URL, collected date, published date.
   - Preserve original text. Clean formatting noise. Do not rewrite opinions.

4. **If the source is a conversation/dialogue** (user-AI turns, not an article/webpage), you MUST add a **对话索引 (Conversation Index)** at the top of the raw file, right after the metadata header. This enables efficient navigation without reading the entire file:

   ```markdown
   # [Title]
   
   > URL: [url] | Collected: [date] | Total Turns: N | Type: conversation
   
   ## 对话索引
   
   | Turn | Question | Key Topics | Char Offset |
   |------|----------|------------|-------------|
   | 1 | "user's question summary" | topic1, topic2 | ~0 |
   | 2 | "next question" | topic3 | ~5000 |
   | 3 | "..." | ... | ~12000 |
   ...
   ```
   
   When you later need to find specific turns in a large raw file, use this index to jump to the right Char Offset. For files >200KB, the index is the ONLY efficient way to navigate.

5. **When compiling wiki from conversation raw**, the wiki article's `## 你的提问脉络` section MUST include ALL user questions from the conversation. This is the primary way learning-companion judges the user's understanding depth. Format:

   ```markdown
   ## 你的提问脉络
   
   ### [Concept/Subtopic]
   1. "exact user question" — 追问了 [aspect]，说明 [depth assessment]
   2. "next question" — ...
   ```

   See `references/raw-template.md` for the exact format.

### Compile (wiki/)

Determine where the new content belongs:

- **Same core thesis as existing article** → Merge into that article. Add the new source to Sources/Raw. Update affected sections.
- **New concept** → Create a new article in the most relevant topic directory. Name the file after the concept, not the raw file.
- **Spans multiple topics** → Place in the most relevant directory. Add See Also cross-references to related articles elsewhere.

These are not mutually exclusive. A single source may warrant merging into one article while also creating a separate article for a distinct concept it introduces. In all cases, check for factual conflicts: if the new source contradicts existing content, annotate the disagreement with source attribution. When merging, note the conflict within the merged article. When the conflicting content lives in separate articles, note it in both and cross-link them.

#### Merge Rules for "Same Topic, New Conversation" (Repeated Learning)

This is the most common case: the user has a new conversation about a topic they already have a wiki article for. The raw is a separate file (different date), the wiki article gets updated. Here's exactly how:

**1. Raw file** → Always a new file. `raw/<topic>/<new-date>-<slug>.md`. Never modify old raw files.

**2. Wiki — Sources & Raw fields** → Append the new raw file link to Sources and Raw fields in the wiki article header (semicolon-separated).

**3. Wiki — 知识概要 (Knowledge Summary)** → Only update if the new conversation introduces a genuinely new concept. If it's just deeper exploration of existing concepts, leave the summary as-is.

**4. Wiki — 你的提问脉络 (Question Thread)** → **Always append.** Add new questions to the relevant concept group(s). If a concept group already has questions from a previous conversation, add the new ones at the end with the new date prefix:
   ```markdown
   ### Concept Name
   *2026-06-27:* 1. "earlier question" — depth assessment
   *2026-06-28:* 15. "new question" — updated depth assessment
   ```
   Never delete old questions — they are the historical record of the user's learning trajectory.

**5. Wiki — 💭 你的理解 (Personal Understanding)** → **Append with date layer.** Do NOT overwrite old understanding. Add a new dated subsection:
   ```markdown
   ## 💭 你的理解
   
   ### 2026-06-28（新对话）
   - [new insights, questions, breakthroughs]
   
   ### 2026-06-27
   - [previous understanding preserved]
   ```
   This preserves the evolution of understanding over time.

**6. Wiki — See Also** → Check if new cross-references are needed. Add if the new conversation linked to other concepts not previously referenced.

**7. wiki/log.md** → Append a learning record under the new date with concept + depth + insight. Also update profile.md's 最近学习于 timestamp and知识地图 if needed.

**8. wiki/index.md** → Refresh the Updated date for the modified article.

**9. wiki/log.md** → Append the operation record.

**10. Raw index** → If the new raw is a conversation >200KB, add a 对话索引 at the top.

See `references/article-template.md` for article format. Key points:
- Sources field: author, organization, or publication name + date, semicolon-separated.
- Raw field: markdown links to raw/ files, semicolon-separated.
- Relative paths from `wiki/<topic>/` use `../../raw/<topic>/<file>.md` (two levels up to project root).

### Conversation Source Detection

When the source material is a **conversation transcript** (dialogue between user and AI, not a standalone article/webpage), you MUST do extra work to preserve the user's personal understanding. This is what makes the wiki truly personal — it's not just another encyclopedia, it's the user's thinking companion.

**How to detect a conversation source:**
- Source contains back-and-forth dialogue (user/assistant turns)
- User's own words, questions, reactions, or "aha moments" are embedded
- The user pastes a chat log or says "ingest this conversation" / "ingest our discussion"
- The source contains the user rephrasing concepts in their own words

**When conversation is detected, the wiki article MUST include a `## 💭 你的理解` section:**

Extract from the conversation:
- The user's original questions (what they were curious about)
- Any moments where the user rephrased the concept in their own words
- The user's analogies or ways of framing the concept
- Any doubts, confusions, or caveats the user expressed
- What clicked for them ("aha moments")

Format in the wiki article:
```markdown
## 💭 你的理解

*记录于 YYYY-MM-DD，来自对话*

**你的疑问：**
- [用户原始提出的问题]
- [追问]

**你的表达：**
> [用户用自己的话描述这个概念的方式]

**你的洞察：**
- [用户自己发现的关联或类比]
```

If the conversation does NOT contain any of the above (e.g., it's just a command-and-response), skip this section. Never invent or fabricate the user's understanding.

**Cross-reference with learning-companion:** If a `wiki/profile.md` exists (maintained by learning-companion), check it for the user's knowledge level in related areas. Use this to calibrate how the wiki article is written — match the user's terminology level and note connections to concepts they already know.

### Cascade Updates

After the primary article, check for ripple effects:

1. Scan articles in the same topic directory for content affected by the new source.
2. Scan `wiki/index.md` entries in other topics for articles covering related concepts.
3. Update every article whose content is materially affected. Each updated file gets its Updated date refreshed.

Archive pages are never cascade-updated (they are point-in-time snapshots).

### Post-Ingest

**Archive write order (must follow this sequence):**

```
1. raw/ 文件            ← 先落地原始素材（只写不读，最安全）
2. wiki/[topic]/*.md    ← 再编译/更新 wiki 文章（核心产出）
3. wiki/index.md        ← 更新目录卡片
4. wiki/log.md          ← 记录操作
5. wiki/profile.md      ← 最后更新画像（依赖前几步结论）
```

This order ensures: even if interrupted mid-way, raw and wiki are complete. Missing index entries can be fixed by lint; missing log entries can be added later.

Update `wiki/index.md`: add or update entries for every touched article. When adding a new topic section, include a one-line description. The Updated date reflects when the article's knowledge content last changed, not the file system timestamp. See `references/index-template.md` for format.

Append to `wiki/log.md`:

```
## [YYYY-MM-DD] ingest | <primary article title>
- Updated: <cascade-updated article title>
- Updated: <another cascade-updated article title>
```

Omit `- Updated:` lines when no cascade updates occur.

---

## Query

Search the wiki and answer questions. Examples of triggers:
- "What do I know about X?"
- "Summarize everything related to Y"
- "Compare A and B based on my wiki"

### Steps

1. Read `wiki/index.md` to locate relevant articles.
2. Read those articles and synthesize an answer.
3. Prefer wiki content over your own training knowledge. Cite sources with markdown links: `[Article Title](wiki/topic/article.md)` (project-root-relative paths for in-conversation citations; within wiki/ files, use paths relative to the current file).
4. Output the answer in the conversation. Do not write files unless asked.

### Archiving

When the user explicitly asks to archive or save the answer to the wiki:

1. Write the answer as a new wiki page. See `references/archive-template.md`. When converting conversation citations to the archive page, rewrite project-root-relative paths (e.g., `wiki/topic/article.md`) to file-relative paths (e.g., `../topic/article.md` or `article.md` for same-directory).
   - Sources: markdown links to the wiki articles cited in the answer.
   - No Raw field (content does not come from raw/).
   - File name reflects the query topic, e.g., `transformer-architectures-overview.md`.
   - Place in the most relevant topic directory.
2. Always create a new page. Never merge into existing articles (archive content is a synthesized answer, not raw material).
3. Update `wiki/index.md`. Prefix the Summary with `[Archived]`.
4. Append to `wiki/log.md`:
   ```
   ## [YYYY-MM-DD] query | Archived: <page title>
   ```

### Date-Based Query

When user asks to review learning by date (e.g., "what did I learn yesterday?", "回顾昨天的学习", "最近学了什么"), answer by querying the wiki's time-based records.

**How the wiki stores time information (two layers of granularity):**

| Layer | File | Format | What it answers |
|-------|------|--------|----------------|
| Operations + Learning | `wiki/log.md` | `## YYYY-MM-DD operation \| details` | "What happened on [date]?" |
| Articles | `wiki/index.md` | `Updated: YYYY-MM-DD` per entry | "Which articles changed on [date]?" |

log.md is the primary source for time-based queries — it records both operations (ingest/lint) and learning sessions (concepts + depth + insights).

**Query procedure:**

1. **Parse the date reference**: "yesterday" → today-1, "last week" → range, "2026-06-26" → exact date.

2. **Query log.md**: grep for `## YYYY-MM-DD` entries in the target date range. This gives both what operations happened AND what concepts were learned.

3. **Supplement with index.md**: check for articles with Updated dates in the range.

4. **Synthesize**: Answer with a chronological summary from log.md. Use index.md for what was touched.

4. **Be honest**: If no records exist for the queried date, say so. "Your wiki doesn't have records for [date]. The nearest entry is from [closest date]."

5. **Cross-reference with learning-companion**: If a time-based question comes in and learning-companion is available, note that learning-companion's Review Mode has richer synthesis capability (it knows user's depth markers, insights, and knowledge blind spots).

---

## Lint

Quality checks on the wiki. Two categories with different authority levels.

### Deterministic Checks (auto-fix)

Fix these automatically:

**Index consistency** — compare `wiki/index.md` against actual wiki/ files (excluding index.md and log.md):
- File exists but missing from index → add entry with `(no summary)` placeholder. For Updated, use the article's metadata Updated date if present; otherwise fall back to file's last modified date.
- Index entry points to nonexistent file → mark as `[MISSING]` in the index. Do not delete the entry; let the user decide.

**Internal links** — for every markdown link in wiki/ article files (body text and Sources metadata), excluding Raw field links (validated by Raw references below) and excluding index.md/log.md (handled above):
- Target does not exist → search wiki/ for a file with the same name elsewhere.
  - Exactly one match → fix the path.
  - Zero or multiple matches → report to the user.

**Raw references** — every link in a Raw field must point to an existing raw/ file:
- Target does not exist → search raw/ for a file with the same name elsewhere.
  - Exactly one match → fix the path.
  - Zero or multiple matches → report to the user.

**See Also** — within each topic directory:
- Add obviously missing cross-references between related articles.
- Remove links to deleted files.

### Heuristic Checks (report only)

These rely on your judgment. Report findings without auto-fixing:

- Factual contradictions across articles
- Outdated claims superseded by newer sources
- Missing conflict annotations where sources disagree
- Orphan pages with no inbound links from other wiki articles
- Missing cross-topic references
- Concepts frequently mentioned but lacking a dedicated page
- Archive pages whose cited source articles have been substantially updated since archival

### Post-Lint

Append to `wiki/log.md`:

```
## [YYYY-MM-DD] lint | <N> issues found, <M> auto-fixed
```

---

## Conventions

- Standard markdown with relative links throughout.
- wiki/ supports one level of topic subdirectories only. No deeper nesting.
- Today's date for log entries, Collected dates, and Archived dates. Updated dates reflect when the article's knowledge content last changed. Published dates come from the source (use `Unknown` when unavailable).
- Inside wiki/ files, all markdown links use paths relative to the current file. In conversation output, use project-root-relative paths (e.g., `wiki/topic/article.md`).
- Ingest updates both `wiki/index.md` and `wiki/log.md`. Archive (from Query) updates both. Lint updates `wiki/log.md` (and `wiki/index.md` only when auto-fixing index entries). Plain queries do not write any files.
