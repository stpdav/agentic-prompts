# Addendum to `optimise-agent-context.md`

Append these to the prompt produced in "Optimizing agent files for better context" (6 Oct 2026). Everything else in that prompt stands; this adds what the kody repo scan and the Claude Code memory docs contribute.

## Changes to existing steps

- Step 4 router budget: 20 lines after the formatter, not ~40. Kody holds a quarter-million-line codebase at 20 and its agents skip sub-indexes anyway. Enforce it (below) rather than aim for it.
- Step 4 index entries: point at an index per area (`docs/contributing/index.md`, `docs/principles/index.md`, skills dir), not at every leaf. The leaf indexes carry `path - when to open it` tables.
- Step 5 focused docs: each page is one topic, present tense, one real example from this repo where a rule needs illustrating, 20-60 lines. A page that grows past one topic becomes a directory with a short `index.md`. Prefer `docs/principles/<rule>.md` for durable engineering rules and `docs/contributing/<topic>.md` for how-the-system-works pages.

## New steps

### Harness files, one source
- Canonical file is `AGENTS.md`. `CLAUDE.md` is the single line `@AGENTS.md` plus any Claude-Code-only lines below it; Claude Code reads `CLAUDE.md`, not `AGENTS.md`. A symlink works on macOS/Linux; use the import on Windows. Mention a path without importing it by wrapping it in backticks.
- `.cursor/rules/*`, `copilot-instructions.md`, `GEMINI.md`, `.windsurfrules`: reduce to a pointer at `AGENTS.md` and the indexes. Delete `.cursorrules` and other legacy single-file formats once relocated.
- Where a harness supports path-scoped rules (Claude Code `.claude/rules`, Cursor globs), use them only for guidance tied to a file pattern - e.g. a rule that loads when editing `*.test.*` and points at the testing doc. Not for general guidance.

### Promote rules to checkers
For every KEEP or MOVE line that is a rule rather than a fact, apply the strongest cheap guardrail that needs no guessing:
1. Lint rule, type, or formatter setting when the violation is visible in one file.
2. Validate-script check for inventory constraints (file sizes, dead links, forbidden phrases).
3. A test when the failure is behavioural.
4. Prose only when none of the above is reasonable; then describe the failure mode and stop.

Rule of thumb: a review comment or agent mistake repeated twice becomes a check, not a sentence.

### Mandatory checks, wired into the validate command and CI
- `AGENTS.md` line budget: 20 lines, measured after the formatter, fails on exceed, never grandfathered into a snapshot. Exactly one such check; do not add a second.
- Markdown file-reference check: relative links and inline repo paths must exist in the tree.
- Optional: reject changelog phrasing in docs ("now we", "no longer", "previously") so pages describe the present system.

### Two principle pages
- `docs/principles/lean-agent-context.md`: agents load guidance on demand; the entry map is tiny; detail one hop away; no restating across files. Rules: `AGENTS.md` is an index held by the line budget; detail lives in focused docs or skills; progressive disclosure; update the source-of-truth page in the same change as the behaviour; leave a pointer when guidance moves; delete docs no session reads.
- `docs/principles/examples-over-prose.md`: one example matching current code; a multi-step procedure becomes a script; a page keeps when / command / failure.

### Verify additions
- Open each harness if available and confirm the router loads and the import or symlink resolves. For Claude Code, `/context` lists `CLAUDE.md` and the imported `AGENTS.md` under memory files.
- Re-read every leaf as a fresh agent: every link resolves; nothing is stated twice.
