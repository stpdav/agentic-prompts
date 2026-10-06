# Prompt: install testing principles + router-style agent docs

Paste everything below the line into your agent (Claude Code, Cursor, Codex, etc.) at the root of any repo. Works for new and existing repos.

---

You are setting up this repository so that every future agent session writes small, behaviour-focused, high-value tests and loads only the context it needs. Do the work directly in the repo. Do not ask me questions you can answer by reading the repo.

## 1. Discover before you write

Inspect the repo and determine:

- Package manager, language(s), test runner(s), test file naming, how tests are run (scripts, CI config).
- Existing test helpers/factories, global setup files, mock configuration, console/log guards.
- Which test "flavours" exist or should exist (pure unit, integration against real local infra, transport/API smoke, browser E2E).
- Whether an `AGENTS.md`, `CLAUDE.md`, `.cursorrules`, or similar already exists, and whether `docs/` exists.

If the repo is empty or has no tests yet, pick sensible defaults for the detected stack and say which you chose.

## 2. Create `docs/contributing/testing-principles.md`

Write it for THIS repo. Every runner name, filename suffix, helper path, and command must be real and verified by reading the repo. Delete anything that does not apply. Structure:

### Opening statement

Two or three sentences: this codebase favours small, readable test suites with explicit setup and minimal magic; each test follows a meaningful workflow end-to-end even if that makes it longer and assertion-heavy.

### Test flavour decision matrix

A table with one row per flavour that exists in this repo. Columns: `Flavour / command`, `Use when`, `Avoid when`. Rule at the top: choose the lightest flavour that can falsify the behaviour. Include a short concrete example showing the same feature tested at two levels (e.g. spy on a function in a unit test vs assert on real rows in an integration test).

### Principles (adapt wording to the stack, keep the intent)

- Fewer, longer tests: when assertions belong to one workflow, put them in one test.
- Treat each test like a manual tester's script: one setup, then as many actions and assertions as the journey needs.
- Never split one flow into many tiny tests to satisfy "one assertion per test". Multiple related assertions in one test are a feature.
- Flat test files: top-level `test(...)`, no nested `describe` blocks.
- No `beforeEach`/`afterEach` or equivalent shared setup; inline setup per test.
- No shared mutable state across tests. If the next assertion depends on the same object/request/response, it belongs in the same test.
- Do not test what the type system or compiler already guarantees.
- Only use disposable/cleanup patterns (`using`, fixtures with teardown, etc.) when there is real cleanup. Otherwise skip them.
- Helpers return ready-to-run objects (factory pattern); explicitly imported, never globals.
- Test names state intent and expected outcome: "auth handler returns 400 for invalid JSON".
- Tests must be runnable offline: no public internet, no third-party services; use local fakes and fixtures.
- Keep the bar for adding a test high. Raise it further for slower integration and E2E tests.
- Fast unit tests for server/domain logic. E2E only for a very small set of critical happy-path journeys, never edge cases.
- Assert intermediate states inside the workflow that causes them rather than writing isolated tests for incidental loading or transition states.
- Do not add a regression test for every bug. Add one only when the flow matters enough to pay the maintenance cost.
- Never assert that a string blob "contains" descriptive copy. Test behaviour: structured output, user-visible outcomes, stable public contracts.
- Never write tests whose only value is pinning configuration-style prose (tool descriptions, hints, warnings, labels). Test the behaviour instead.
- If a logging/console guard exists, document how to assert on expected logs and how to silence incidental ones with an explicit allowlist. Never blanket-silence output; that hides regressions.
- If mocks are auto-reset globally, say so, and say that each test must inline the setup it needs.
- Document the exact command(s) to run each flavour, and any pitfalls (e.g. commands that accidentally match the wrong spec files).

### Agent-specific anti-patterns (include this section verbatim)

Agents tend to over-test. Before adding any test, check it is not one of these:

- A test per function parameter or per branch when one workflow test would cover the real behaviour.
- A test that re-asserts the type signature, a default value, or that a constant equals itself.
- A test that mocks the thing under test, or mocks so much that only the mocks are being asserted.
- A snapshot or `toContain` on prose, error text, or UI copy.
- A test added "for coverage" with no plausible failure it would catch.
- An E2E test for something a unit test can falsify in milliseconds.
- A `describe` tree with `beforeEach` that hides what the test actually depends on.

If in doubt, do not add the test. Prefer extending the nearest existing workflow test.

### Examples

Two short, real examples from this stack: one plain workflow test with inline setup and several assertions, and one using a factory with real cleanup. Use real imports from this repo.

## 3. Make `AGENTS.md` (or `CLAUDE.md`) a router, not a manual

Create or trim the always-loaded agent file to the minimum:

- One or two sentences: what this project is and where the agent is. Nothing else about architecture, structure, or commands that the agent can discover from the repo.
- Keep ONLY non-default rules that trip agents up on every task today (e.g. "run `<validate command>` before finishing; git hooks are unreliable in agents"). If there are more than a handful, move them to a doc and link it.
- A line stating the file is intentionally brief and that agents must not add content here; detailed guidance lives in focused docs.
- An index of focused docs with one line each describing when to open it. First entry:
  `docs/contributing/testing-principles.md` - read this before writing or changing any test.
- Harness-specific gotchas (cloud agents, sandboxes, CI) go in their own doc, linked from the index.

Delete from the always-loaded file anything that is discoverable by reading the repo: repository structure, package manager, Node/Python version, how to run tests, coding standards, library lists. Aim for well under 50 lines.

If the repo uses `.cursor/rules`, `.github/copilot-instructions.md`, or similar, make them point at the same docs instead of duplicating content.

## 4. Verify

- Run the full test command(s) to confirm nothing broke.
- Open `docs/contributing/testing-principles.md` as if you were a fresh agent and check every path, command, and helper it names exists.
- Report: files created/changed, line count of the always-loaded agent file before and after, and anything you deliberately left out because the repo didn't need it.

## 5. Ongoing rule (add this to the testing principles doc)

Context is not free. Do not grow `AGENTS.md`. When a new rule is needed, put it in the most specific focused doc and link it from the index. Periodically ask: has any agent session actually read this doc? If not, delete it.
