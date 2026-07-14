---
name: doc-cleanup
description: Audit and clean Markdown documentation for context rot. Use when docs may contain completed-work logs, executed plans, stale claims, contradictions, dead paths, or drifted authorities, especially in agent-required reading.
---

# Doc cleanup

Remove context rot from Markdown without erasing load-bearing rules or useful
history. Treat every doc claim as a hypothesis and the repository as evidence.

Work in two passes: diagnose and report without editing, then execute only after
the user approves the proposed cleanup. Approval to edit does not imply approval
to commit.

## 1. Inventory the full scope

Enumerate every in-scope `*.md` file, excluding generated, dependency, VCS, and
user-excluded paths. Record each file's line count and role:

- **entry**: `AGENTS.md`, `CLAUDE.md`, or equivalent
- **required**: an entry doc instructs agents to read it
- **reference**: consulted conditionally
- **historical**: retained for lookup, not current guidance

Follow the reading pointers in every entry doc. Weight findings by reading
frequency: rot in required reading costs more than rot in an archive.

Inventory is complete when every in-scope Markdown file has a role and every
entry-doc reading pointer is accounted for as resolved, dead, or external.

## 2. Diagnose without editing

Read every in-scope doc deeply enough to test it for these rot patterns:

1. **Completed-work logs** — finished checklists, phase logs, implementation
   narratives, post-mortems, and verification transcripts presented as live
   guidance.
2. **Executed plans** — shipped plans still written as intent, including
   superseded analysis and rejected options.
3. **Internal contradictions** — competing values, corrections below stale
   tables, or dated addenda that reverse earlier guidance.
4. **Stale claims** — references to renamed symbols, changed constants, wrong
   line numbers, or work described as unmerged after it landed.
5. **Dead paths** — missing files, obsolete machine paths, moved repositories,
   or removed tools.
6. **Authority drift** — a purported source of truth that no longer matches the
   implemented behavior or a later authoritative decision.

For each suspected hit, capture:

- exact file and line location
- the claim or content at issue
- repository evidence from file lookup, search, configuration, tests, or Git
  history
- confidence and any unresolved ambiguity
- the smallest safe disposition: keep, correct, condense, archive, or banner

Code proves implemented behavior, not necessarily intended behavior. When a doc
may define the intended contract, report the mismatch as unresolved authority
drift unless repository evidence establishes which authority superseded it.

Mark load-bearing content that must survive: hard rules, active decisions,
recurring traps, do-this guidance, and domain vocabulary. A trap remains current
even when the story of discovering it is historical.

Diagnosis is complete when every inventoried file is either represented by at
least one evidence-backed finding or explicitly marked clean, and every factual
finding has been checked against repository evidence.

## 3. Report and stop

Present findings from highest to lowest `reading frequency × misdirection risk`.
For each one, show the claim, evidence, consequence, and proposed disposition.
Then give a per-file cleanup plan, list unresolved decisions separately, and
state which load-bearing content will remain.

Stop for approval. The diagnostic pass must leave the worktree unchanged.

Reporting is complete when the user can approve, reject, or amend every proposed
file change without additional investigation.

## 4. Execute the approved plan

Re-check each factual claim immediately before writing it. Apply only approved
changes, using this per-file playbook:

- **Archive history that must remain conveniently discoverable.** Put a verbatim
  copy or clearly bounded extract in `docs/archive/` with a collision-resistant
  source-and-date name. Record its source path and Git blob or content hash in
  the cleanup report or a sidecar manifest, leaving verbatim content untouched.
  Verify a full-file archive against the captured pre-edit source; verify an
  extract against its recorded source bounds. Git history alone is sufficient
  when discoverability does not justify a live archive.
- **Rewrite live guidance as current facts.** Add a short archive pointer when
  one was created. Derive every retained factual claim from current evidence,
  rather than carrying it forward unchecked.
- **Point to authorities.** Name the owning schema, configuration, or source file
  instead of duplicating volatile values.
- **Condense completed work.** Replace finished checklists with a short outcome;
  preserve open items in their original wording.
- **Preserve traps.** Turn recurring gotchas into short imperative guidance
  before moving their historical narrative.
- **Banner unresolved drift.** State the mismatch and the currently operative
  authority; avoid silently rewriting a contract whose ownership is unclear.
- **Repair entry docs.** Fix dead reading pointers and stale feature claims, and
  keep the required-reading set minimal.

Execution is complete when every approved finding has a corresponding diff or a
documented reason no change was needed, and every retained factual claim touched
by the rewrite has fresh evidence.

## 5. Verify and hand off

Verify that:

- every relative link in each touched file resolves
- full-file archives match their captured pre-edit sources byte-for-byte, and
  extracts match their recorded source bounds
- load-bearing content identified during diagnosis survives in live guidance
- the diff contains only approved Markdown changes
- no new contradiction was introduced across touched docs

Report the changed files, archive mappings, corrected contradictions, unresolved
authority decisions, and verification performed. Offer a docs-only commit with a
specific message; create it only when the user authorizes the commit.

The cleanup is complete when every approved finding is accounted for, all checks
pass, and the user has a reviewable diff plus any remaining decisions.
