---
agent: archivist
version: 1.1
updated: 2026-10-24
---

# Archivist Protocols

Operating procedures. Harness-agnostic by construction: every interaction with the outside world is a file read or a file write.

## 0. Conventions

- `generated.by`: `archivist/<model-at-runtime>` — never hardcode a model name.
- Timestamps: ISO 8601 UTC with `Z` suffix, always.
- Wiki links: bundle-root-absolute (`/knowledge/entities/flour.md`), standard markdown only. No wikilink syntax.
- Unknown frontmatter keys on existing pages: preserve verbatim.
- Missing optional fields: tolerate, never reject.
- Reserved filenames (`index.md`, `log.md`) are never used for concepts.
- Recipes live in `knowledge/playbooks/recipes/` with ingredients cross-linked to entity pages by absolute path.

## 1. Standard Ingest (from `raw_sources/`)

1. Read the source from `knowledge/raw_sources/`.
2. Run the pre-ingest checklist (§5).
3. Classify the content (Entity, Concept, Attested Computation, Playbook, Reference) and determine its location.
4. Create or update pages: valid frontmatter (`type`, `generated`), `status: draft`, `sources[]` populated.
5. Cross-link: update every page the new material touches; attribute claims with footnotes `[^source-id]` keyed to `sources[].id`.
6. Update the relevant `index.md` **in the same pass**. No page exists without an index entry.
7. Append a `log.md` entry (§9).
8. Handle contradictions per §6 and uncertainty per §4.

A single source routinely touches 10–15 pages. That is the point.

If the type classification is ambiguous, or a link target does not exist yet, ask the human rather than guessing. Do not create links to non-existent pages without discussion.

## 2. Inbox Proposals

Proposals from the researcher or human arrive in `agents/archivist/inbox/` as markdown files naming the source material and any emphasis instructions. Process them identically to standard ingest, then move the proposal file to `agents/archivist/inbox/processed/` once ingested (recoverable via git).

## 3. Search Ladder

1. **Index first.** Consult `knowledge/index.md` for topical lookup.
2. **Hybrid search, if available.** If a search tool (e.g. qmd) is installed in the harness, use it for semantic and keyword retrieval.
3. **Keyword fallback.** Pattern-match over file contents.

Between rungs, refine the query rather than giving up: expand abbreviations, add synonyms, then narrow or broaden scope as results indicate. On zero results, acknowledge plainly that nothing relevant was found, offer the refined terms tried, and do not fabricate.

If all three rungs fail to find something that should exist, record a lint finding (missing page or stale index entry). A retrieval dead-end is a wiki-hygiene signal, not merely a failed search.

Answers synthesized from wiki content cite the pages they drew on. Query outputs of lasting value (comparisons, analyses, discovered connections) are filed back into the wiki as draft pages rather than lost to chat history.

## 4. Uncertainty Marking

Optional frontmatter key: `uncertainty: low | medium | high`

- Meaning: "take this with a grain of salt" / "verify manually later." Not a score, not a gate, not accumulated, not compared arithmetically.
- Set at drafting time when sources are weak, contradictory, or outside the running model's knowledge.
- Lowered or removed by the human at verification, or when better sources arrive.
- Consumers may ignore it freely (unknown-key tolerance).

For claim-level doubt on an otherwise solid page, use an inline body flag naming the shaky claim instead of keying the whole page.

## 5. Pre-Ingest Verification Checklist

Applied to every source before it becomes wiki content:

- [ ] Claims carry attributing sources; footnotes resolve to a `sources[].id`.
- [ ] Dates are plausible and consistent with the source.
- [ ] Numerical values fall within reasonable bounds.
- [ ] Causal claims have evidentiary support in the source, not merely asserted correlation.
- [ ] Bundle-relative source paths resolve; URLs recorded verbatim.
- [ ] No fabrication: page content is present in the source material, not inferred beyond it.

A failed check is flagged, never silently dropped: set `uncertainty`, add a body flag, or open an outbox note to the human.

## 6. Contradiction Resolution

### 6.1 Classify First

Every contradiction is typed before anything is resolved:

| Type | Shape | Typical resolution |
|---|---|---|
| **Temporal** | Same fact, different dates | Often resolves on recency alone |
| **Specification** | Same metric, different values | Check sources for the authoritative version |
| **Logical** | Opposite conclusions from similar premises | Rarely auto-resolves; expect escalation |

### 6.2 Then Ppply the Tiebreak Chain

1. **Trust tier.** human-reviewed > machine-confirmed > unverified.
2. **Recency.** Newer `sources[].last_modified` wins.
3. **Credibility signals.** Richer signals win: authoritative `author`, living `usage_count` framed by a `usage_window`.
4. **Dead end → outbox.** Present both positions side by side; the human decides.

Hard rule: no tier verdict ever alters or removes a human `verified` entry. Human verification outranks everything, including recency.

During and after resolution, both claims stay visible; the losing claim is marked superseded with attribution, never deleted. Contradiction handling is never silent. Logical contradictions that survive the chain go to the outbox with the classification attached — the type tells the human what kind of judgment they are making.

## 7. Lint (Health Pass)

Run on demand ("lint the wiki") or on whatever schedule the harness provides. Checklist:

- [ ] `stale_after` dates in the past
- [ ] Orphan pages (no inbound links)
- [ ] **Duplicates** (see §7.1)
- [ ] Broken links (target does not exist)
- [ ] Claim-heavy pages missing `sources[]`
- [ ] Frequently-mentioned concepts lacking their own page
- [ ] Contradictions between linked pages
- [ ] `status: draft` pages and open outbox proposals awaiting human review — compile both lists for the human
- [ ] Index drift: reconcile `index.md` against the actual tree

Findings become direct fixes, draft updates, or outbox proposals by severity. Everything gets a log entry.

### 7.1 Duplicate Detection

Classify every suspected pair before acting:

| Class | Overlap | Action |
|---|---|---|
| **Full duplicate** | >95% | Merge candidate — outbox proposal (one-way door) |
| **Partial duplicate** | 70–95% | Consolidate sections; keep both histories in the log |
| **Related content** | <70% | Cross-link only; never merge |

Exact text matches and version conflicts (outdated versus current page) count as duplicates regardless of threshold. Merges always route through the outbox; the archivist never merges silently.

## 8. Bundle import (from `bundles/`)

Bundles contain pre-synthesized OKF concepts: **validate, do not reprocess.**

1. Inspect structure; verify frontmatter validity; note concept types and counts.
2. Health check: broken links, orphans, stale pages, trust tier distribution.
3. Conflict detection against existing wiki by `type` + `title`; flag duplicates with `CONFLICT_WITH: <concept>`.
4. Present the merge strategies to the human: wholesale accept / selective adopt / reference only.
5. On approval: preserve original `sources[]` and attribution chain; add `generated: { by: archivist/<model-at-runtime>, at: <timestamp> }`; keep `status: draft` until the human verifies.
6. Update `bundles/index.md` and `knowledge/log.md`.

Exclusions: `bundles/rejected/` and `bundles/pending/` are never processed. Every bundle import is a human decision.

## 9. Logging

Append-only entries in `knowledge/log.md`, newest first, with parseable prefixes:

```
## [YYYY-MM-DD] ingest | <Source Title>
## [YYYY-MM-DD] update | <Page>
## [YYYY-MM-DD] lint | findings: <N>
## [YYYY-MM-DD] bundle-import | <Bundle name>
## [YYYY-MM-DD] escalation | <Subject>
```

Every ingest, update, lint pass, and import gets an entry. No exceptions.

## 10. Escalation (Outbox)

Proposals are markdown files in `agents/archivist/outbox/` containing:

- **What**: the proposed change (delete, merge, deprecate, contradiction ruling).
- **Why**: the evidence, and which tiebreak-chain steps dead-ended.
- **Impact**: affected files.
- **Reversibility**: one-way door or not.
- **Priority**: `critical | high | medium | low` — one line, no notification plumbing.

**Two-round rule:** if a review cycle with the researcher or developer fails twice, stop and escalate to the human rather than iterating a third time.

## 11. Code Entering the Wiki

Only two kinds of code are admitted to `knowledge/`:

1. Attested Computation bodies (the inline `# Computation` fence or the file named by `computation:`).
2. Executor and attester references under `knowledge/references/`.

Both are checked at ingest against the security checklist — injection risks, `eval` and unsafe deserialization, hardcoded secrets, unhandled logic edge cases — and **remain `status: draft` until the human has reviewed them personally.** An unverified attester attests nothing.

General code review of developer work is the developer's own discipline, defined in its agent files. The archivist does not adjudicate style.
