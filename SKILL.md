---
name: karpathy-llm-wiki
description: "Use when building or maintaining a personal LLM-powered knowledge base. Triggers: ingesting sources into a wiki, querying wiki knowledge, linting wiki quality, 'add to wiki', 'what do I know about', or any mention of 'LLM wiki' or 'Karpathy wiki'."
---

# Karpathy LLM Wiki

Build and maintain a personal knowledge base using LLMs. You manage two directories: `raw/` (immutable source material) and `wiki/` (compiled knowledge articles). Sources go into raw/, you compile them into wiki articles, and the wiki compounds over time.

Core ideas from Karpathy:
- "The LLM writes and maintains the wiki; the human reads and asks questions."
- "The wiki is a persistent, compounding artifact."

This skill supports four workflows:
- **Ingest** — fetch a source into `raw/`, then compile it into `wiki/`
- **Query** — answer questions from the wiki and optionally archive the synthesis
- **Lint** — keep the wiki structurally healthy
- **Insights** — synthesize actionable, cross-article insights and save them back into the wiki

## Architecture

Three layers, all under the user's project root:

**raw/** — Immutable source material. You read, never modify. Organized by topic subdirectories (e.g., `raw/machine-learning/`).

**wiki/** — Compiled knowledge articles. You have full ownership. Organized by topic directories with optional nested subsection directories: `wiki/<topic>/<article>.md` or `wiki/<topic>/<subsection>/<article>.md`. Use nesting only when it improves retrieval clarity; avoid deep trees without a strong reason. 

Contains two special files:
- `wiki/index.md` — Global index. One row per article, grouped by topic and, when relevant, subgrouped by subsection path, with link + summary + Updated date.
- `wiki/log.md` — Append-only operation log. When articles live in subsections, log entries should use enough path context to make the location unambiguous.

**SKILL.md** (this file) — Schema layer. Defines structure and workflow rules.

Templates live in `references/` relative to this file. Read them when you need the exact format for raw files, articles, archive pages, insight pages, or the index.

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

   See `references/raw-template.md` for the exact format.

### Compile (wiki/)

Determine where the new content belongs:

- **Same core thesis as existing article** → Merge into that article. Add the new source to Sources/Raw. Update affected sections.
- **New concept** → Create a new article in the most relevant topic or subsection directory. Name the file after the concept, not the raw file.
- **Spans multiple topics** → Place in the most relevant directory in the hierarchy. Add See Also cross-references to related articles elsewhere.

These are not mutually exclusive. A single source may warrant merging into one article while also creating a separate article for a distinct concept it introduces. In all cases, check for factual conflicts: if the new source contradicts existing content, annotate the disagreement with source attribution. When merging, note the conflict within the merged article. When the conflicting content lives in separate articles, note it in both and cross-link them.

See `references/article-template.md` for article format. Key points:
- Article and insight metadata should use Dataview-compatible inline field syntax: `Field:: value`.
- Sources field: author, organization, or publication name + date. When multiple sources are present, keep them in one concise inline field value.
- Raw field: markdown links to raw/ files. When multiple raw sources are present, keep them in one concise inline field value.
- Tags field: short thematic markers for cross-cutting retrieval. Keep them concise, lowercase when practical, and store multiple items as a quoted comma-separated inline list, for example `Tags:: "retrieval", "wiki"`.
- Use cases field: short phrases naming concrete tasks where the note is useful. Keep each use case brief and practical, and store multiple items as a quoted comma-separated inline list.
- Related questions field: short natural-language questions the note helps answer. Keep each question concise and retrieval-friendly, and store multiple items as a quoted comma-separated inline list.
- Raw field links must be relative from the article's actual location to project root. Example: `../../raw/<topic>/<file>.md` from `wiki/<topic>/article.md`, or `../../../raw/<topic>/<file>.md` from `wiki/<topic>/<subsection>/article.md`.

### Cascade Updates

After the primary article, check for ripple effects:

1. Scan articles in the same local directory first, then nearby subsection directories within the same topic tree, for content affected by the new source.
2. Scan `wiki/index.md` entries in other topics or subsection branches for articles covering related concepts.
3. Update every article whose content is materially affected. Each updated file gets its Updated date refreshed.

Archive pages are never cascade-updated (they are point-in-time snapshots).

### Post-Ingest

Update `wiki/index.md`: add or update entries for every touched article. When adding a new topic section, include a one-line description. When adding a subsection, preserve the folder hierarchy in the index with clear nested grouping or path labels so readers can see where the page lives. The Updated date reflects when the article's knowledge content last changed, not the file system timestamp. See `references/index-template.md` for format.

Append to `wiki/log.md`:

```
## [YYYY-MM-DD] ingest | <primary article title>
- Updated: <cascade-updated article title>
- Updated: <another cascade-updated article title>

If titles are ambiguous or the page lives in a subsection, include path context such as `AI / Second Brain / <article title>`.
```

Omit `- Updated:` lines when no cascade updates occur.

### Auto-Insights After Ingest

After the ingest is complete, run a brief insight pass over the newly created or updated knowledge pages. This pass is part of the default workflow, but it should stay compact and high-signal.

1. Review the primary article plus any cascade-updated articles.
2. Decide whether they warrant new or revised insights. Skip insight generation if the ingest is too small, too repetitive, or does not change practical understanding.
3. If insights are warranted, save them into `wiki/Insights/` using the insight workflow below.
4. Update `wiki/index.md` for every created or revised insight page.
5. Append to `wiki/log.md`:

```
## [YYYY-MM-DD] insights | Auto after ingest: <primary article title>
- Updated: <insight page title>
- Updated: <another insight page title>
```

Omit the `insights` log entry when no insight page changed. Do not modify the structure of the log beyond using the existing heading-plus-updates pattern.

---

## Query

Search the wiki and answer questions. Examples of triggers:
- "What do I know about X?"
- "Summarize everything related to Y"
- "Compare A and B based on my wiki"

### Steps

1. Read `wiki/index.md` to locate relevant articles.
2. Read those articles and synthesize an answer.
3. Prefer wiki content over your own training knowledge. Cite sources with markdown links such as `[Article Title](wiki/topic/article.md)` or `[Article Title](wiki/topic/subsection/article.md)` (project-root-relative paths for in-conversation citations; within wiki/ files, use paths relative to the current file).
4. Output the answer in the conversation. Do not write files unless asked.

### Archiving

When the user explicitly asks to archive or save the answer to the wiki:

1. Write the answer as a new wiki page. See `references/archive-template.md`. When converting conversation citations to the archive page, rewrite project-root-relative paths (e.g., `wiki/topic/article.md` or `wiki/topic/subsection/article.md`) to file-relative paths appropriate to the archive page location (e.g., `../topic/article.md`, `../topic/subsection/article.md`, `../../sibling-subsection/article.md`, or `article.md` for same-directory).
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

### Additional Rules
- You can use `wiki/Insights/` as a relevant articles, but  do not let them override more grounded topic articles when they conflict. Prefer the underlying knowledge pages when resolving disagreements.

---

## Insights

Synthesize new understanding from the wiki and store it back into the wiki as insight pages. Insights are not raw notes, not summaries, and not free-form journaling. They should create leverage: a business direction, a sharp reframing, a concrete next action, a system upgrade, or a publishable prompt/post.

### Triggers

Examples of triggers:
- "Generate insights on the latest wiki changes"
- "Gain Python-related insights"
- "Update my business ideas with the wiki knowledge"
- "Suggest social media posts from my wiki"
- "Save insights to wiki"

### Steps

1. Read `wiki/index.md` to identify relevant source articles and any existing insight pages.
2. Read the selected knowledge pages and the most relevant existing insight pages.
3. Generate only the strongest insights for the requested scope. Favor fewer, sharper outputs over broad coverage.
4. If the user asked for insights only in chat, output them with citations and do not write files. By default, insights should be generated as new articles under `wiki/Insights/` with proper metadata and citations.
5. If the user asked to save or update insights, write or revise pages in `wiki/Insights/`, update `wiki/index.md`, and append to `wiki/log.md`:

```
## [YYYY-MM-DD] insights | <scope description>
- Updated: <insight page title>
- Updated: <another insight page title>
```

Omit `- Updated:` lines when only one page was changed.

### Scope Selection

Choose the narrowest scope that matches the user's request:
- **By topic** — read the relevant topic section from `wiki/index.md`, then the matching articles
- **By article set** — read the specific requested pages
- **By latest changes** — prefer pages with the newest `Updated` dates, optionally using `wiki/log.md` as a routing hint
- **By insight type** — gather the best supporting pages for that type even if they span topics

When the user says "latest changes" without more detail, default to the most recently updated knowledge pages in `wiki/`, excluding `wiki/Insights/` unless the task is explicitly about refining existing insights.

### Insight Types

Use one of these five explicit types:
1. **Business Idea** — a concrete product, startup, service, or monetizable direction. Must include `Audience`, `Pain`, and `First step`.
2. **Wise Thought** — a non-obvious reframing that challenges an existing belief. Must include `Belief challenged`.
3. **Action Advice** — a concrete action that improves money, usefulness, or progress toward goals. Must include `Next action`.
4. **Pipeline Update** — a change to life/work/learning/wiki workflow. Must include `Stop`, `Start`, and `Keep`.
5. **Social Question or Post** — a question worth publishing or a ready post draft. Must include either `Question` or `Post draft`.

Each insight must:
- cite at least 1-2 concrete wiki articles
- synthesize or reprioritize, not merely summarize
- stay grounded in evidence from the wiki
- avoid generic motivation or vague self-help phrasing
- prefer practical utility while keeping a balanced tone

### Storing Insights

Insights live under `wiki/Insights/`. This topic can contain broad living pages and more focused pages when the idea cluster becomes large enough to deserve its own file.

Default storage behavior:
- Reuse an existing insight page when the new insight meaningfully extends or sharpens it
- Create a new page when the idea is clearly distinct or would make an existing page unfocused
- Update existing insight pages instead of duplicating the same insight with new wording

See `references/insight-template.md` for the exact page format.

Key points:
- `Sources` must link to wiki articles, not raw files
- `Updated` reflects when the insight meaningfully changed
- `Status` tracks the working state of the insight, such as `new`, `active`, `shipped`, `superseded`, or `rejected`
- Use relative links from `wiki/Insights/` to other wiki pages, for example `../Flutter/article.md` or `../AI/Second Brain/article.md`

---

## Lint

Quality checks on the wiki. Two categories with different authority levels.

### Deterministic Checks (auto-fix)

Fix these automatically:

**Index consistency** — compare `wiki/index.md` against actual wiki/ files recursively (excluding index.md and log.md):
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

**See Also** — within each topic tree:
- Add obviously missing cross-references between related articles in the same section, nearby subsections, or parent topic when the relationship is strong.
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
- The language of section headings and structural labels inside a wiki page or insight page must match the language of the page itself. If the note is written in Russian, headings should also be in Russian; if the note is written in English, headings should also be in English. Keep metadata field names that are part of the schema exactly as required unless the schema itself is intentionally localized.
- `Tags`, `Use cases`, and `Related questions` are retrieval metadata, not mini-summaries. Keep every item short. Do not repeat folder names, long sentences, or the article title in slightly different words. Store multi-value fields in Dataview-friendly quoted comma-separated inline lists so Dataview parses them as arrays instead of one long string.
- wiki/ supports topic directories plus nested subsection directories when they make retrieval or grouping clearer. Prefer the shallowest structure that still makes the sectioning useful.
- `wiki/Insights/` is a normal topic directory and may also use subsection nesting when needed.
- `wiki/index.md` should mirror the effective hierarchy of the wiki so a reader can discover both top-level topics and their subsections from the index alone.
- `wiki/log.md` should keep the same heading-plus-updates format, but may include topic/subsection path context in titles or update bullets when that improves clarity.
- Today's date for log entries, Collected dates, and Archived dates. Updated dates reflect when the article's or insight's knowledge content last changed. Published dates come from the source (use `Unknown` when unavailable).
- Inside wiki/ files, all markdown links use paths relative to the current file. In conversation output, use project-root-relative paths (e.g., `wiki/topic/article.md` or `wiki/topic/subsection/article.md`).
- Ingest updates both `wiki/index.md` and `wiki/log.md`. Auto-insights after ingest may update both as well. Archive (from Query) updates both. Manual insight saves update both. Lint updates `wiki/log.md` (and `wiki/index.md` only when auto-fixing index entries). Plain queries or chat-only insight requests do not write any files.
