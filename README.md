# karpathy-llm-wiki

**A reusable skill for building Karpathy-style LLM wikis with Claude Code, Cursor, Codex, and other Agent Skills tools.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/Astro-Han/karpathy-llm-wiki?style=social)](https://github.com/Astro-Han/karpathy-llm-wiki)
[![GitHub forks](https://img.shields.io/github/forks/Astro-Han/karpathy-llm-wiki?style=social)](https://github.com/Astro-Han/karpathy-llm-wiki)
[![Agent Skills](https://img.shields.io/badge/Agent_Skills-compatible-blue)](https://agentskills.io)
[![Install](https://img.shields.io/badge/Install-npx_add--skill-green)](https://github.com/Astro-Han/karpathy-llm-wiki#install)

<p align="center">
  <img src="assets/karpathy-tweet.png" alt="Karpathy's tweet about LLM Wiki" width="560">
</p>

`karpathy-llm-wiki` packages [Karpathy's LLM Wiki idea](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) into one installable [Agent Skills](https://agentskills.io) skill. Your coding agent ingests sources into `raw/`, compiles durable knowledge pages into `wiki/`, answers questions with citations, generates higher-level insights, and lints the wiki for consistency.

## What Is an LLM Wiki?

An **LLM wiki** is a knowledge system where the LLM maintains structured wiki pages instead of re-searching raw documents on every question. New sources are compiled into durable markdown pages, cross-references are updated over time, and answers cite the wiki pages that already contain the synthesized knowledge.

This skill gives you four operations:

| Operation | What it does | Output |
|-----------|--------------|--------|
| **Ingest** | Collects a source into `raw/` and compiles it into the wiki | New or updated wiki pages |
| **Query** | Searches the wiki and answers with citations | Grounded answers linking to markdown pages |
| **Insights** | Synthesizes business ideas, reframings, actions, workflow updates, or publishable posts from the wiki | Chat-only insights or saved pages under `wiki/Insights/` |
| **Lint** | Checks index integrity, links, and wiki health | Auto-fixes plus reported issues |

See [SKILL.md](SKILL.md) for the full skill specification.

## LLM Wiki vs RAG

| Approach | Knowledge lives in | When synthesis happens | Good for |
|----------|--------------------|------------------------|----------|
| **RAG** | Raw chunks and embeddings | At query time | Broad retrieval across large corpora |
| **LLM Wiki** | Curated markdown pages | During ingest and maintenance | Compounding knowledge, summaries, and durable cross-links |

This skill is optimized for the wiki model: knowledge that improves over time instead of re-deriving relationships on every query.

## Usage Stats

Based on a production knowledge base maintained daily since April 2026:

- **94** wiki articles across **13** topic directories
- **99** source materials ingested
- **87** operation log entries in the last 7 days

See [examples/](examples/) for sample wiki pages, source files, and operation logs.

## Install

```bash
npx add-skill Astro-Han/karpathy-llm-wiki
```

Works with any tool that supports the [Agent Skills](https://agentskills.io) standard.

## Quick Start

### 1. Ingest your first source

Give the skill a URL, a file, or pasted text:

> "Ingest this article: https://example.com/attention-is-all-you-need"

The skill stores the source in `raw/`, then compiles or updates the right knowledge pages in `wiki/`.

### 2. Ask your wiki a question

> "What do I know about attention mechanisms?"

The skill searches the wiki and answers with citations linking back to your markdown pages.

### 3. Generate insights

> "Form insights from the latest wiki changes"

The skill can answer in chat, or save durable insight pages under `wiki/Insights/` when you ask it to update the wiki.

### 4. Keep the wiki healthy

> "Lint my wiki"

Checks for broken links, missing index entries, stale cross-references, and related issues.

## How the Workflow Works

The core idea from Karpathy: the LLM maintains the wiki while the human focuses on choosing sources and asking good questions.

```text
your-project/
├── raw/            ← Immutable source material
│   └── topic/
│       └── 2026-04-03-source-article.md
├── wiki/           ← Compiled knowledge pages maintained by the LLM
│   ├── topic/
│   │   ├── concept-name.md
│   │   └── subsection/
│   │       └── nested-concept.md
│   ├── Insights/
│   │   └── useful-synthesis.md
│   ├── index.md    ← Global table of contents
│   └── log.md      ← Append-only operation log
```

Topic directories can contain nested subsection directories when they make retrieval clearer. The global `wiki/index.md` mirrors that effective hierarchy so both humans and agents can discover top-level topics, subsections, and saved insight pages.

Each new source can update multiple pages, strengthen cross-references, record contradictions, and optionally trigger an insight pass. That is what makes the wiki compound over time.

## Obsidian and Dataview

Wiki pages use standard markdown with Dataview-compatible inline fields:

```markdown
Type:: article
Updated:: 2026-05-27
Sources:: Karpathy, 2026
Raw:: [source](../../raw/topic/source.md)
Tags:: "retrieval", "wiki"
Use cases:: "answer questions", "maintain knowledge"
Related questions:: "How should an LLM wiki evolve?"
```

This keeps the wiki readable in any markdown editor while making it useful inside Obsidian Dataview queries. Multi-value metadata such as `Tags`, `Use cases`, and `Related questions` is stored as quoted comma-separated inline values so Dataview can parse them as arrays.

## Insight Generation

Insights are first-class wiki artifacts, not throwaway summaries. They live under `wiki/Insights/`, cite the underlying wiki pages, and can be revised as the knowledge base changes.

Supported insight types:

| Type | Best for |
|------|----------|
| **Business Idea** | Product, startup, service, or monetizable directions |
| **Wise Thought** | Non-obvious reframings and belief challenges |
| **Action Advice** | Concrete next actions |
| **Pipeline Update** | Improvements to life, work, learning, or wiki workflows |
| **Social Question or Post** | Questions or post drafts worth publishing |

There are two flows:

| Flow | When it runs | What happens |
|------|--------------|--------------|
| **Automatic** | After ingest | The skill reviews the newly changed knowledge pages and saves only high-signal insights when the update warrants it |
| **Manual** | On request | You choose a topic, latest changes, article set, or insight type; the skill either answers in chat or writes/revises pages in `wiki/Insights/` |

Saved insight pages update `wiki/index.md` and append an `insights` entry to `wiki/log.md`, just like other durable wiki changes.

## Tool Compatibility

This skill follows the [agentskills.io](https://agentskills.io) open standard:

| Tool | Install method |
|------|----------------|
| Claude Code | `npx add-skill Astro-Han/karpathy-llm-wiki` |
| Cursor | `npx add-skill Astro-Han/karpathy-llm-wiki` |
| Codex CLI | Copy to `.agents/skills/karpathy-llm-wiki/` |
| OpenCode | `npx add-skill Astro-Han/karpathy-llm-wiki` |
| Other tools | Copy `SKILL.md` and `references/` into the tool's skill directory |

## FAQ

### What is the difference between an LLM wiki and a personal wiki?

An LLM wiki is maintained by the model. It updates summaries, cross-links, index entries, and contradictions as new material arrives. A normal personal wiki depends on manual editing.

### What sources can I ingest?

Web pages, papers, blog posts, PDFs, markdown files, text files, and pasted text. The skill converts everything into markdown under `raw/` and compiles it into `wiki/`.

### Is this production-ready?

The workflow is based on a real knowledge base with 94 articles and 99 sources maintained daily since April 2026. The repo includes examples, templates, and a design spec.

## Inspired By

Unofficial community implementation of the workflow from [Karpathy's LLM Wiki idea](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). The value here is the reusable workflow, prompt structure, and battle-tested knowledge-compilation rules.

See also: [lucasastorian/llmwiki](https://github.com/lucasastorian/llmwiki), [atomicmemory/llm-wiki-compiler](https://github.com/atomicmemory/llm-wiki-compiler).

## License

[MIT](LICENSE)
