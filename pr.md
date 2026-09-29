## What changed?

Four new unit tests in `tests/unit/output.test.ts` for `parseReport`'s
non-object validation branch. No `src/` file is modified.

The new tests feed `parseReport` the JSON scalars `null`, `42`, a JSON
string, and `true`, and assert each throws `InvalidReportError` with the
guard's `"not a report object"` message — the branch the issue asks to
cover with `parseReport("null")` and `parseReport("42")`, plus its two
remaining scalar neighbors so the guard is pinned for every JSON
non-object value (`typeof`-object arrays aside).

## Why?

`parseReport` rejects non-object JSON with
`typeof parsed !== "object" || parsed === null`. The existing
"rejects JSON that is not an object" test passes `[1, 2, 3]` — but an
array *passes* that `typeof === "object"` check, so it falls through to
the later `schemaVersion` check and is rejected there instead. The
non-object guard itself had no coverage proving it rejects the values
it exists to reject, and a regression that inverted or dropped it could
silently turn `null`/`42` input into a confusing "missing schemaVersion"
error (or worse, a crash on field access) rather than the intended
clear message.

These tests pin the distinct branch, including its specific
`not a report object` message, so both the rejection and its wording
are contract.

## Tests performed

- [x] `npm test -- output` — 21 tests passed (17 existing + 4 new)
- [x] `npm test` — 11 files / 151 tests passed
- [x] `npm run typecheck` — passes
- [x] `npm run lint` — clean
- [x] `npm run build` — not needed: no `src/` file is modified, so
      `dist/` is unchanged and still matches a fresh build

## Changelog

Does this change anything people using the Action will notice? If so, it
needs an entry in `CHANGELOG.md` under `[Unreleased]`:

- [x] Internal-only (test coverage); no entry needed.

## Related issue

Closes #270

## Compatibility impact

Does this change an input, output, or the supported `Protocol-Canary`
version range? If so, update the README's Inputs/Outputs/"Supported
Canary versions" tables.

None. Test-only change; runtime behavior is unchanged.

## Breaking change?

- [ ] Yes — described above
- [x] No
