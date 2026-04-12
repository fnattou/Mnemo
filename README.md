# Lean-Mnemo

Token-efficient external memory format for AI agents, using `.aicontext` files.

---

## Core Principles

1. **Two-level access** — Index files (lightweight overview) loaded first; detail files loaded on demand
2. **Hierarchical structure** — Each folder has its own `index.aicontext`; don't flatten to root
3. **Token-based abbreviations** — Only multi-token terms (3+ tokens) worth abbreviating; dictionary stored in `dict.aicontext`
4. **Merge updates** — Keep one entry per topic file; update with latest date instead of appending
5. **Concise writing** — Bullets, not paragraphs; remove filler

---

## Directory Structure

```
memory/
  dict.aicontext                    # Abbreviation dictionary (optional, load first)
  index.aicontext                   # Root index: lists categories
  decisions/
    index.aicontext                 # Lists decision topic files
    auth.aicontext                  # One topic per file
    db.aicontext
    architecture/
      index.aicontext
  tasks/
    index.aicontext
    [topic files]
  bugs/
    index.aicontext
    [topic files]
```

---

## File Formats

### Topic File (`decisions/auth.aicontext`)

```yaml
2026-04-10: JWT + Redis caching strategy
  - Stateless session management via JWT tokens
  - Redis for token revocation
  - Scales horizontally
  - Trade-off: complexity vs simplicity
```

**Rules:**
- One topic per file (e.g., auth.aicontext contains only auth strategy, not mixed decisions)
- YAML format with timestamp (YYYY-MM-DD)
- Bullet points only (concise)
- When topic revisited: update existing entry with new date; don't append duplicate entries
- Keep only the latest state; history not needed

### Index File (`decisions/index.aicontext`)

```yaml
# decisions/ - Architecture and design decisions

dir: architecture/ | System architecture decisions
dir: infra/ | Infrastructure and deployment decisions

file: auth.aicontext | JWT + Redis authentication strategy
file: db.aicontext | PostgreSQL selection rationale
```

**Rules:**
- Lists immediate children only (files and subdirectories)
- Each entry: path + one-line description
- Update when files added/removed

### Dictionary File (`dict.aicontext`)

```yaml
# Multi-token abbreviations (optional)
SMS: session management strategy
DTL: database transaction logging
```

**Rules:**
- Only include terms that:
  - Are 3+ tokens when written out
  - Appear 3+ times in context
  - Are domain-specific (reused across files)
- Load first when reading memory files

**Note:** At small-to-medium scale (~100 topics), abbreviations yield modest savings (~5%) with meaningful maintenance overhead. The primary compaction gains come from prose removal and two-level access, not abbreviations. Skip `dict.aicontext` unless your memory grows large enough that the savings justify the upkeep.

---

## Mnemo vs RAG

RAG optimizes for **scale**. Mnemo optimizes for **density**.

| | Mnemo | RAG |
|---|---|---|
| Best for | ~100 topics, deterministic recall | 1000+ documents, fuzzy search |
| Token cost | Low — hand-curated bullets | Medium — raw chunks passed as-is |
| Setup | Text editor only | Vector DB + embedding model |
| Tuning | None | Chunk size, overlap, retrieval K |
| Recall guarantee | ✅ Explicit file read | ❌ Search can miss |
| Offline | ✅ | ❌ API required |

**RAG's hidden costs:** vector DB ops, embedding API fees, chunk strategy tuning, retrieval quality evaluation.

**Use Mnemo when** you have a bounded set of decisions, rules, or context that must be reliably recalled — and you want zero infrastructure.

**Use RAG when** you have large unstructured corpora and fuzzy search is acceptable.

---

## Token Efficiency

Scenario: 10 project topics (auth, DB schema, API design, etc.), equivalent to ~10 typical wiki pages.

| What you load | Tokens | vs. raw docs |
|---|---|---|
| Raw Markdown docs (all) | ~2,920 | 100% |
| Mnemo — all files (without dict) | ~929 | 31.8% |
| Mnemo — all files | ~881 | 30.2% |
| Mnemo — index only | ~13 | <1% |
| Mnemo — index + 1 topic | ~194 | 6.6% |

The two main drivers of compaction:
- **Removing prose**: Keep only decision bullets, remove background paragraphs (~68% reduction)
- **Two-level access**: Index acts as a table of contents; load full topics only when needed

`dict.aicontext` (abbreviations) adds a further ~5% on top, but at small scale the maintenance overhead rarely justifies it.

> See [benchmark/scenario_10topics/](benchmark/scenario_10topics/) for full scenario data (raw/ and memory/ directories).

---

## saveMemory Skill Implementation

Claude reads referenced context, extracts key insights, determines category/topic, writes to appropriate file with timestamp, updates all affected indices.

**Algorithm:**
1. Parse user input (e.g., "conversation history")
2. Read referenced content
3. Analyze: extract 3-5 key insights
4. Determine: category (decisions/tasks/bugs) + topic (auth/db/etc)
5. Check: does `memory/{category}/{topic}.aicontext` exist?
   - Yes → read, merge with new info, update date
   - No → create file
6. Write entry (timestamp + bullets)
7. Update `memory/{category}/index.aicontext`
8. If new category created: update `memory/index.aicontext`
9. Suggest dict entries if multi-token terms appear 3+ times

**Example:**

```
User: /saveMemory "conversation about JWT vs OAuth"

Claude's decision:
  - Extract: "JWT chosen for stateless design, horizontal scaling"
  - Category: decisions
  - Topic: auth
  - Writes to: memory/decisions/auth.aicontext
  - Updates: memory/decisions/index.aicontext, memory/index.aicontext
```

---

## Portability

1. Copy `CLAUDE.md`, `memory-format.aicontext`, and `.claude/commands/saveMemory.md` to the new project
2. Create `memory/index.aicontext` and `memory/{category}/index.aicontext`
3. Call `/saveMemory` — Claude creates topic files automatically
4. All operations use standard Read/Write/Edit tools → no external dependencies