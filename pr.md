## What changed?

`renderExecutionFailureMarkdown` in `src/summary.ts` no longer embeds the
raw execution-failure diagnostic inside a fixed ```` ```text ```` fence.
It now computes the longest run of backticks the diagnostic contains and
opens/closes the fence with a run strictly longer than that (minimum 3,
with 20 as the working floor for nested embedding). The placeholder for
an empty diagnostic and every other part of the rendered summary are
unchanged; only the fence length adapts.

Two new tests in `tests/unit/summary.test.ts` cover the issue's
acceptance criterion and its adversarial neighbor:

- a realistic compiler-style diagnostic whose body quotes its own
  ```` ```rust ```` block renders with an opening fence longer than any
  run inside the diagnostic, exactly one opening fence line and one
  closing fence line, and the verbatim diagnostic content preserved;
- a worst-case diagnostic consisting only of backtick runs (including
  runs of equal length) also stays inside the fence.

Both tests assert the actual CommonMark property, not an incidental
fence length: any line whose leading backtick run reaches fence length
must be one of the two fence lines, so the diagnostic can never close
the block early.

## Why?

Execution-failure diagnostics come straight from an external process's
stderr, so the Action cannot assume that text is safe to embed verbatim
inside a fence it does not control. A diagnostic containing its own
triple-backtick block (plausible from a compiler error or panic message
quoting code) closed the fixed fence early, and the rest of the
diagnostic rendered as broken markdown in the job summary — on the
exact page users are sent to when the Action fails hardest.

Fenced code blocks are closed only by a closing fence of at least the
opening fence's length, so a fence longer than every backtick run in
the content can never be closed early (CommonMark spec §Fenced code
blocks). This is the spec-sanctioned strategy for embedding arbitrary
text and requires no escaping or mangling of the diagnostic.

## Tests performed

- [x] `npm test -- summary` — 18 tests passed (16 existing + 2 new)
- [x] `npm test` — 11 files / 149 tests passed
- [x] `npm run typecheck` — passes
- [x] `npm run lint` — clean
- [x] `npm run build` — passes; `dist/index.js` and `dist/index.js.map`
      rebuilt and committed so the committed bundle matches a fresh
      build of the changed `src/summary.ts`

## Changelog

Does this change anything people using the Action will notice? If so, it
needs an entry in `CHANGELOG.md` under `[Unreleased]`:

- [x] `CHANGELOG.md` updated under `[Unreleased]` → `### Fixed` — the
      rendered execution-failure summary is user-visible output.

## Related issue

Closes #269

## Compatibility impact

Does this change an input, output, or the supported `Protocol-Canary`
version range? If so, update the README's Inputs/Outputs/"Supported
Canary versions" tables.

None. The summary remains a job-summary markdown document; only the
fence length of the execution-failure diagnostic block adapts, which is
what the rendered output was always *meant* to be. No input, output, or
supported-version change; no consumer parses this fence length.

## Breaking change?

- [ ] Yes — described above
- [x] No
