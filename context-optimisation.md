# Prompt: Optimise this repo's agent context for cost and signal

You are auditing and rewriting the always-loaded agent context in this repository. Goal: cut the tokens every session pays for before the user types a word, without losing anything the agent cannot rediscover on its own.

Grounding (verified Oct 2026):
- Anthropic removed >80% of Claude Code's system prompt for Claude 5-generation models with no measurable loss on its coding evals (claude.dev, 24 Jul 2026). Six old rules are now myths: rules -> judgement, examples -> interface design, everything upfront -> progressive disclosure, repetition -> simple tool descriptions, memory in CLAUDE.md -> auto-memory, prose specs -> rich references (code, tests, mockups).
- ETH Zurich study (arXiv 2602.11988, Feb 2026): AGENTS.md-style files raise inference cost ~20% on average. LLM-generated files lowered success ~3%, developer-written ones raised it ~4%. Instructions are followed; repository overviews are not useful. Conclusion: keep only non-standard requirements, and measure before claiming a gain.

## Scope

Find every file injected into context at session start, for every harness this repo supports:
`AGENTS.md`, `CLAUDE.md` (root, nested, `~/.claude/CLAUDE.md` if referenced), `.claude/rules/*`, `.cursor/rules/*`, `.cursorrules`, `.github/copilot-instructions.md`, `GEMINI.md`, `.windsurfrules`, auto-loaded skills (`.claude/skills/*/SKILL.md` without an invocation trigger), and MCP server configs whose tool schemas load eagerly.

## Procedure

1. Inventory. List each always-loaded file with line count and an estimated token count (chars / 4 is fine). Sum them. This is the baseline.
2. Classify every line or block into exactly one bucket:
   - KEEP: true on every task, every time, and not derivable from the repo. Typically: one-paragraph statement of what the repo is, genuine gotchas (non-standard layout, a convention that contradicts the ecosystem default, a landmine that has bitten agents repeatedly), a current harness bug workaround.
   - MOVE: real but situational (testing principles, auth structure, deploy steps, debugging tips, style beyond the linter). Goes to a focused doc under `docs/agent/` or a skill, reached by a one-line pointer.
   - REPLACE WITH REFERENCE: a rule that is already enforced or expressed in code. Point to the linter config, the test suite, the type file, the example implementation. Code beats prose.
   - DELETE: discoverable by reading the repo (package manager, runtime version, directory tree, how to run tests, framework in use), generic advice ("write clean code", "follow best practices"), restated defaults, session memories, boilerplate like "this file provides guidance to...".
3. Resolve conflicts. Search the whole set for instructions that contradict each other or over-specify (e.g. "never write comments" vs "document as appropriate"). Prefer deleting the constraint and letting the model match the surrounding code over adding a tie-breaker rule.
4. Rewrite the root file as a router. Target shape, in this order, under ~40 lines:
   - What this repo is (2-4 lines).
   - Every-task gotchas (bulleted, each one line, each earned by a real repeated failure).
   - Harness-specific landmines only if they are current; mark each with the bug or reason so it can be retired.
   - Index: one line per focused doc or skill, as `path - when to read it`. Flat where possible; add a sub-index only when the index itself exceeds ~25 entries.
   - Final line: "This file is intentionally brief. Do not add to it; put detail in the indexed docs."
5. Build the focused docs. Each one answers a single question an agent would ask mid-task. Split rather than grow; an agent should be able to open one file and get only what it needs. Lint prose the same way: if the code shows it, cite the path instead of describing it.
6. Deferred loading. Where the harness supports it, convert eagerly loaded skills to on-demand (clear trigger in the description) and MCP tool sets to lazy/searchable loading. Remove MCP servers not used in the last 30 days of transcripts if that history is available.
7. Verify. Every path in the index exists. No instruction appears in two places. No kept line is derivable from the repo. Re-count tokens.

## Decision rules

- "Could the agent learn this with one `ls`, one `cat package.json`, or one grep?" -> DELETE.
- "Would an agent do the wrong thing on most tasks without this line?" If no -> MOVE or DELETE.
- "Is this a preference or a judgement call?" -> state the outcome, not the procedure, or leave it to the model.
- "Does this exist as code, config, or a test?" -> REPLACE WITH REFERENCE.
- Do not invent new rules while tidying. Do not soften a deletion into a shorter version of the same rule.
- If this repo is still worked by pre-Claude-5-class models, keep guardrail rules in a clearly labelled `docs/agent/legacy-model-rules.md` and point to it conditionally rather than inlining them.

## Output

1. Baseline vs result table: file, tokens before, tokens after, % change, total.
2. Change log: one line per moved/deleted/replaced block with its bucket and one-phrase reason.
3. The new root file(s) in full, plus any new focused docs.
4. Measurement plan: how to confirm this helped. Minimum: over the next N sessions, sample transcripts and record (a) which indexed docs were actually opened, (b) task completion, (c) tokens per task; delete docs nobody opens after a month; re-run this audit when the model generation changes.
5. Open questions: anything you were unsure whether to keep, with your recommendation.

Do not run this as a one-off. Treat it as a periodic subtraction pass, not a rewrite.
