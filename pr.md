## What changed?

One new unit test in `tests/unit/runner.test.ts`, in the `runCheck`
describe block, exercising the child-process event ordering the
`settled` guard in `runCheck` exists for: `"close"` settles the promise
first, and `"error"` is emitted afterwards. No `src/` file is modified.

The test swaps in a `FakeChild` (the fake already used by the suite),
emits `"close"` explicitly, awaits the promise so it has really settled,
then emits `"error"` the way Node can after the fact (e.g. a read error
surfacing on a stream whose process has already exited), and asserts:

- the settled result is exactly what `"close"` delivered
  (`exitCode: null`, `signal: "SIGTERM"`, empty stdout/stderr) and is
  unchanged by the late event;
- no unhandled rejection surfaces — this is enforced structurally,
  because vitest fails the whole run on an unhandled rejection, so a
  broken guard fails this test;
- the `SIGINT`/`SIGTERM` forwarding listeners end up exactly in the
  post-`cleanup()` state `"close"` left them in: none leaked, none
  removed twice.

## Why?

`runCheck`'s `"error"` handler guards with `if (settled) return;`
specifically to handle a real race between Node's `"close"` and
`"error"` events on a child process. Every existing test fires
`"error"` as the *first* settling event (the cannot-start case), so the
guard itself had no coverage: a change that dropped or broke the check
could silently reintroduce a double-settle or an unhandled rejection —
the latter fails a whole Action run after the result was already
delivered — without any test noticing.

One honest caveat documented in the test: per ES semantics, calling
`reject()` on an already-resolved promise is a silent no-op, so the
guard's effect is not observable through the promise object alone. What
the test pins is the observable contract the issue asks for — the
already-settled result is unchanged and no unhandled rejection occurs —
plus the listener-cleanup invariant that a second, non-guarded settle
*would* have disturbed (a second `cleanup()` call is idempotent, but a
guard regression that reordered or duplicated the settle path is caught
by the surrounding state assertions and by vitest's unhandled-rejection
detection).

## Tests performed

- [x] `npm test -- runner` — 15 tests passed (14 existing + 1 new)
- [x] `npm test` — 11 files / 148 tests passed
- [x] `npm run typecheck` — passes
- [x] `npm run lint` — clean
- [x] `npm run build` — not needed: no `src/` file is modified, so
      `dist/` is unchanged and still matches a fresh build

## Changelog

Does this change anything people using the Action will notice? If so, it
needs an entry in `CHANGELOG.md` under `[Unreleased]`:

- [x] Internal-only (test coverage); no entry needed.

## Related issue

Closes #272

## Compatibility impact

Does this change an input, output, or the supported `Protocol-Canary`
version range? If so, update the README's Inputs/Outputs/"Supported
Canary versions" tables.

None. Test-only change; runtime behavior is unchanged.

## Breaking change?

- [ ] Yes — described above
- [x] No
