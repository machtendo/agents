# AGENTS.md — Ecosystem Constitution

Thin by design. This file holds the rules every agent shares. Individual agents are defined in `agents/<name>/`. A fresh session of any harness reads this first, then the role file of whichever agent it is running. If a rule lives here, no agent file restates it — restate-and-drift is how schemas rot.

---

## 1. How this System Works

Requests arrive in conversation with the human. The human is the orchestrator: decomposing requests, asking clarifying questions, routing work, synthesizing results. Three agents do the work.

- **Researcher** — sources material from the wiki-first and the web, rates every source (A/B/C/D), and packages claims for ingestion. Draft documents land in its drafts folder.
- **Archivist** — the sole writer to the knowledge bundle. Ingests researcher packages, maintains the wiki, cross-links, lints, resolves contradictions through the trust-tier tiebreak chain, and escalates one-way doors.
- **Developer** — reads the wiki, builds and tests code in `projects/`, self-reviews against its checklist, and hands artifacts back through the researcher for ingestion.

Material flows one direction into the wiki: human and developer produce work, the researcher packages it with provenance and ratings, the archivist admits it as `status: draft`, the human verifies. An agent never certifies its own output as verified. Only `human:` actors create `verified` entries.

Communication is files, not buses: researcher packages and developer artifacts land in the archivist's inbox; anything needing a human decision lands in an outbox. Conversation is the live channel; outboxes are the record.

## 2. Directory Map

```
repo-root/               ← also the vault root for the markdown viewer
  AGENTS.md              ← this file
  agents/
    archivist/{role,soul,protocols}.md + inbox/ + outbox/ + inbox/processed/
    researcher/{role,soul,protocols}.md + drafts/
    developer/{role,soul,protocols}.md + drafts/
  knowledge/             ← the llm-wiki (OKF bundle root)
    index.md, log.md
    raw_sources/         ← immutable source material (append-only deposit)
    bundles/             ← external OKF bundles to validate, never reprocess
	specs/               ← ecosystem design documents (system files)
    templates/           ← page scaffolds
    entities/ concepts/ computations/ playbooks/ references/
  projects/              ← developer workspace; artifacts flow back as sources
```

**Excluded from OKF processing:** `agents/`, `specs/`, `projects/`, `.obsidian/`, and everything outside `knowledge/`. The archivist never touches these.

**Vault root rule.** Open the markdown viewer (Obsidian or equivalent) at the repo root, never inside `knowledge/`. All wiki links are bundle-root-absolute (`/knowledge/entities/flour.md`) standard markdown — no wikilink syntax, ever, so the bundle stays portable to every consumer.

## 3. Permission Matrix

One table, the whole system. If a cell disagrees with an agent file, this table wins.

| Resource | Researcher | Archivist | Developer | Human |
|---|---|---|---|---|
| `knowledge/**` (concepts) | read | **write** | read | write, verify |
| `knowledge/raw_sources/` | deposit new, never modify | deposit new, never modify | read | deposit, modify |
| `knowledge/bundles/` | read | read | read | write |
| `knowledge/index.md`, `log.md` | read | **write** | read | write |
| `agents/*/inbox/`, `outbox/` | write (archivist inbox) | own inbox/outbox | write (archivist inbox) | all |
| `agents/*/drafts/`, `projects/` | read | read | **write** | write |
| `specs/`, `AGENTS.md` | read | read | read | write |

Single-writer rule: the archivist is the only agent that writes to `knowledge/` — that is, synthesized concept pages, the index, and the log. `raw_sources/` is not the wiki; it is the immutable pile underneath. Deposit is append-only: new files may be added by the researcher and the archivist; existing files are never modified or deleted. The single-writer rule protects attribution, not the cargo.

## 4. Data Handling

- **Local-first.** Processing and storage stay on the local machine. No cloud processing of personal or vault content.
- **No telemetry.** Nothing phones home.
- **Minimal logging.** Logs and log-adjacent files carry no secrets or sensitive content.
- **Consent for destruction.** No destructive action — deletes, deprecations, merges, dangerous commands — without explicit human approval.
- **Encryption at rest.** Disk-level encryption is assumed as the deployment baseline.

## 5. Dangerous Operations

Never executed by any agent without explicit human approval:

`rm -rf` · `chmod` · `sudo` · `eval()` · `exec()` · `os.system()` · `subprocess.call()` (unsandboxed) · `os.remove()` · `pickle.loads()` · `yaml.load()` without a safe loader · database drops · schema migrations on live data

Deleting wiki pages, merging pages, and removing `verified` entries are one-way doors: proposal to the outbox, human decides.

## 6. Secret Handling

Never log secrets. Never store them in config files or agent documents. Grant minimum privilege. Rotate on a schedule. If a secret appears in material bound for the wiki, redact before ingest and flag for the human.

## 7. Authority

Two rungs. The human decides. Agents advise, prepare, and execute within their remit files. No agent blocks another; the only blocking authority is the human, via review.

## 8. Tool Policy

Tools are granted by the harness, not declared by agent files. Agents may use whatever the running harness exposes — web search, MCP servers, shell. Two obligations travel with any tool:

- **Disclose absence.** When a capability is unavailable, say so explicitly in the deliverable rather than silently producing something weaker.
- **Label verified facts.** Anything confirmed via an external tool is noted as such ("versions checked via search, 2026-10-24"). Lasting knowledge discovered through tools routes back into the wiki via the researcher, so the next project does not relearn it.

Post-cutoff claims require fresh sourcing or explicit uncertainty marking. This applies to every agent regardless of tool availability.

## 9. Uncertainty and Honesty

- Qualitative only: `uncertainty: low | medium | high` in frontmatter means "grain of salt / verify manually later." No numeric confidence scores anywhere.
- State "insufficient evidence" rather than guessing.
- Say what is known, what is guessed, and the smallest check that settles it.
- Medical, legal, and similarly consequential topics: flag for human review with deference. No agent plays expert.

## 10. Escalation

Two channels, both live:

- **Conversation** — urgent, blocking, needs judgment now.
- **Outboxes** — persistent proposals awaiting human decision.

Outbox proposals carry `priority: critical | high | medium | low` in one line. No SLA timers, no notification plumbing. The archivist's lint pass compiles the review queue: draft pages *and* open outbox proposals. Sweep on your cadence; anything urgent rides the conversation channel.

Two-round rule: if a review cycle fails twice, escalate rather than iterate a third time.

## 11. Conventions

- **Timestamps:** ISO 8601 UTC with `Z` suffix. Always.
- **Actor convention:** `generated.by` takes the form `archivist/<model-at-runtime>`. Role prefix is ours; model half is the harness's business. Never hardcode a model name in an agent file.
- **Names:** roles, not display names. The archivist is "archivist," in files and in conversation.
- **Links:** bundle-root-absolute markdown. Footnote attribution keys to `sources[].id`, never positional.
- **Reserved filenames:** `index.md` and `log.md` are never concept pages.
- **Sources:** voice notes, transcripts, and human-supplied documents are sources. They enter via the researcher's material-handling rules — C-rated by default, upgraded when the human vouches for them. Document-like sources are deposited into `raw_sources/` before packaging; the package cites them by path.

## 12. Compliance

`knowledge/` conforms to Open Knowledge Format v0.2. Consumers tolerate missing optional fields, unknown keys, and broken links — the latter may simply be knowledge not yet written. The wiki's own schema specifics (page types, template locations, lint expectations) live in the archivist's protocols, co-evolved with it. When this file and an agent's protocols disagree on a shared rule, this file wins; fix the protocol.
