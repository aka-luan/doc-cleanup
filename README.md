# doc-cleanup

**An agent skill that finds and removes context rot from your repo's markdown docs.**

Your `CLAUDE.md` / `AGENTS.md` ecosystem is quietly lying to your agents. Every
"read this before you code" doc accumulates finished checklists, executed
plans, dead file paths, and facts the code outgrew months ago. Agents pay for
that twice: once in context tokens on **every session**, and again in wrong
conclusions — because a stale fact in an "authoritative" doc beats a correct
fact the agent never thought to look up.

`doc-cleanup` is a [skill](https://docs.claude.com/en/docs/claude-code) (a
`SKILL.md` playbook) that audits every markdown doc in a repo, verifies each
claim against the actual code, reports what's hurting your agents, and — after
you approve — archives the history and rewrites the live docs down to current
facts.

## Results from its first production run

A real Next.js/Hono monorepo, ~3 weeks of agent-driven development, docs
faithfully updated the whole time — and still:

| Doc (agent required-reading) | Before | After | |
|---|---:|---:|---|
| Roadmap / milestones | 445 lines | 33 lines | −93% |
| Design-system spec | 443 lines | 181 lines | −59% |
| Pricing plan | 197 lines | 52 lines | −74% |
| **Total required reading** | **1,085 lines** | **266 lines** | **−75%** |

Nothing was lost — 919 lines of history moved verbatim to `docs/archive/`,
still greppable, no longer in every session's context.

The line count wasn't even the important part. The audit caught, in one pass:

- A pricing doc stating the enabled payment methods **two different ways in
  the same file**, one with a line reference that no longer existed.
- A pricing table "corrected" by a footnote three paragraphs below it —
  the table itself never edited. Agents read tables; footnotes are a coin flip.
- Two features marked *"implemented locally — awaiting merge/deploy"* that had
  been merged to `main` days earlier (`git merge-base --is-ancestor` says so).
- A gotchas file warning agents about the import graph of a component that
  had been **renamed weeks earlier**.
- A hard rule in `AGENTS.md` pointing at a sibling repo via a Windows drive
  path (`A:\Dev\...`) on a machine that had since moved to WSL — the rule was
  literally unfollowable.
- `README.md` and `AGENTS.md` both advertising a feature ("custom URLs") that
  doesn't exist in the product.

Every one of those is a trap a coding agent would have walked into.

## The six rot patterns it hunts

1. **Completed-work logs** — checklists that are ~all `[x]`, phase logs full
   of ✅, bug post-mortems, verification transcripts. History belongs in git
   and archives, not in every session's context window.
2. **Executed plans never rewritten** — a plan whose steps all shipped reads
   as *intent* but gets consumed as *fact*, and its pre-decision analysis
   (superseded tables, rejected options) actively misleads.
3. **Internal contradictions** — supersession notes instead of edits, the
   same fact with two values in one doc, stacked dated addenda that reverse
   each other.
4. **Stale facts vs code** — renamed files, changed constants, wrong
   `file.ts:NN` refs, "awaiting merge" for merged work. Every claim is
   verified by grepping the repo before it's flagged: *doc claims are
   hypotheses; the code is the evidence.*
5. **Dead paths** — old OS paths after machine migrations, moved sibling
   repos, deleted tooling.
6. **Authority drift** — a doc crowned "source of truth" that the code has
   legitimately outgrown. Left alone, agents will "fix" correct code back
   toward the stale spec.

Equally important is what it **refuses to delete**: hard rules, recurring
traps, do-not-do warnings, domain glossaries. A gotcha discovered
historically is not history — it's the most valuable content agent docs have.

## How it works

Five phases, with a hard gate in the middle:

1. **Inventory** — every `*.md` by size, weighted by how often agents must
   read it (a 150-line doc on the "read before you code" list costs more than
   a 400-line doc read monthly).
2. **Diagnose** *(read-only)* — hunt the six patterns, verifying each finding
   against the code.
3. **Report** — findings worst-first with concrete evidence. **Stops here for
   your approval. It never silently rewrites docs.**
4. **Execute** — archive verbatim (diff-verified against `git HEAD`), rewrite
   live docs to re-verified current facts, point at source-of-truth code
   instead of duplicating values, banner drifted spec docs ("code wins"),
   collapse done checklists, extract surviving traps from archived logs.
5. **Verify & commit** — every relative link resolves, `git status` shows
   only intended files, docs-only commit.

The execute phase is deliberately delegable: the skill has the main agent
write a self-contained spec of verified facts and hand the bulk rewriting to
a cheaper model, then review the diff line-by-line. In the first run that
review caught the subagent flattening "Premium *unlocks* manual moderation"
into "moderation: manual" — a table cell that would have misstated the
product's default behavior.

## Install

**One-liner (via [`skills`](https://www.npmjs.com/package/skills), works for Claude Code and other agents):**

```bash
npx skills add aka-luan/doc-cleanup
```

Run it inside a project for a per-project install, or add `-g` for global.

**Manual, Claude Code global (all projects):**

```bash
git clone https://github.com/aka-luan/doc-cleanup ~/.claude/skills/doc-cleanup
```

**Manual, Claude Code single project:**

```bash
git clone https://github.com/aka-luan/doc-cleanup .claude/skills/doc-cleanup
```

**Other harnesses:** it's one `SKILL.md` file with YAML frontmatter
(`name` + `description`). Drop it wherever your agent discovers skills
(e.g. `.agents/skills/doc-cleanup/`).

## Use

```
/doc-cleanup
```

or just ask naturally — the skill triggers on things like:

> "evaluate all the md docs on this project and what might be hurting agents thinking"

(which is, verbatim, the request that produced this skill).

## When to run it

- Entry docs feel heavy and sessions start slow.
- An agent confidently cites something you know is outdated.
- After a big milestone ships — plans and checklists rot fastest right after
  they succeed.
- After a machine/OS migration or a repo move.
- Periodically. Docs rot at the speed you ship; agentic repos ship fast.

## Philosophy

- **Archive, don't delete.** History stays greppable in `docs/archive/`,
  out of the context window.
- **The repo is the evidence.** No fact gets written into a live doc without
  being re-verified against code at write time.
- **Point, don't copy.** Docs should reference the constant/schema that owns
  a value, not restate it — restated values are future contradictions.
- **Traps outlive history.** Recurring gotchas get extracted and kept;
  narratives about fixing them get archived.

## License

[MIT](LICENSE)
