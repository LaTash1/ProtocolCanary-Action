## What changed?

One new unit test in `tests/unit/canary.test.ts` covering the ambiguous
checksum-manifest branch of `selectExpectedChecksum` (exercised through
`ensureCanaryInstalled`, the suite's established observation point for
private helpers). No `src/` file is modified.

The test mocks a published checksum manifest containing multiple
`stellar-canary`-named entries for platforms other than the runner's —
e.g. `stellar-canary-darwin-x64` and `stellar-canary-win32-arm64` when
running on Linux — with no entry matching the current platform/arch,
and asserts that `ensureCanaryInstalled` still succeeds, no
`InstallationFailedError` is thrown, nothing is reinstalled, and the
degradation is the deliberate debug log
("no entry for this platform") that `verifyInstalledBinary` emits when
`selectExpectedChecksum` returns `undefined`.

## Why?

`selectExpectedChecksum` prefers an exact name match, then a
platform+arch match, then a platform-only match, then a single
unambiguous candidate. When a real published manifest names several
platform-specific binaries and none of them is for this runner's
platform, every branch falls through and the function returns
`undefined` — the ambiguous-manifest case, distinct from the
"no stellar-canary-named entries at all" case already covered by the
"falls back to commit/tag pinning when no checksum manifest is
published" test.

It must degrade to commit/tag pinning (the pre-checksum behavior per
`SECURITY.md`) exactly like that case, not fail the install because a
release happened to ship binaries for other platforms. That specific
fallthrough had no test, so a change that turned the fallthrough into a
throw — or into silently trusting the first unrelated platform's
checksum — would not have been caught.

The platform pair is derived from the actual `process.platform` at
runtime, so the test is correct on any runner OS the suite executes on.

## Tests performed

- [x] `npm test -- canary` — 19 tests passed (18 existing + 1 new)
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

Closes #273

## Compatibility impact

Does this change an input, output, or the supported `Protocol-Canary`
version range? If so, update the README's Inputs/Outputs/"Supported
Canary versions" tables.

None. Test-only change; runtime behavior is unchanged.

## Breaking change?

- [ ] Yes — described above
- [x] No
