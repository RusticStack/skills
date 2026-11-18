# Tests that look useless and are the only guard for something

Read this before pruning. The recognition rules come first, because they protect a kind of test this file does not list.

Keeping a test that turned out to be worthless costs a few seconds per run. Deleting the only guard for a behavior costs a bug nobody is told about. When unsure, keep it and report it.

## Recognition rules

1. **Ask what change would make it fail, and who would be hurt.** A security regression, a library upgrade, a broken consumer, a changed compile error, or a wrong documented example is an answer that keeps the test.
2. **Find the judge before judging.** A test with no run-time assertion is live if another tool checks it. Search the CI configuration and scripts for the type checker, the compiler step, the doc-test flag, the API report, the lint, and the coverage gate.
3. **Expectation and replay files are inputs.** A file that holds expected output or a saved failing case is half of a test. Deleting it, or regenerating it until the run is green, removes the check.
4. **Judge an absence by the requirement behind it.** "This route must stay 404" is a policy. "This module no longer exists" is history.
5. **Read reason strings and identifiers.** A ticket, issue, or CVE number in a name, comment, or marker means a known bug.
6. **Odd expected values are a signal.** A value that looks wrong, copied from what the code returned, with no specification, is a characterization test. Keep it, and report the oddity as a possible bug.
7. **Check who else uses a helper.** A fixture or helper that is exported, public, published, or behind a feature flag may be used by other packages or repositories.
8. **Duplicates must match the layer.** A unit test, a contract test, a compile-fail test, and a documented example can look alike and guard different things.
9. **Passing almost always is not worthless.** Race, interleaving, allocation, and fuzz tests exist for the rare run.
10. **A test of a library may be an alarm.** If project code relies on the library behavior the test pins, it tells you when an upgrade breaks you. Only a test with no project code behind it is a candidate.

## Judged by another tool

These pass at run time whatever the code does. The check happens elsewhere.

- TypeScript: `*.test-d.ts` with `expectType`, `expectAssignable`, `expectError` (tsd); `expectTypeOf` (expect-type, checked by `tsc`); `// @ts-expect-error` above a line that must not compile.
- Python: `typing.assert_type`, checked by mypy or pyright; stub tests (`stubtest`) and their allowlists.
- Rust: `compile_fail` doc tests; trybuild cases with their `*.stderr` files; `tests/public-api.txt` (cargo public-api).
- C and C++: `static_assert`.
- API reports: `*.api.md` (API Extractor), Go `apidiff`, japicmp, cargo-semver-checks.
- Architecture and convention rules: ArchUnit (including frozen rules and their violation store), NetArchTest, import-linter contracts, dependency-cruiser rules, ESLint `no-restricted-imports`, license-header checks.

## Expectation and replay files

- Snapshots: Jest and Vitest `.snap` files and inline snapshots, insta `.snap` and pending `.snap.new`.
- Golden and approved files: Go `testdata/`, `*.approved.*`.
- Saved failing inputs, replayed on every run: `.hypothesis/`, `proptest-regressions/`, Go `testdata/fuzz/<FuzzName>/`, a hard-coded fast-check `seed` and `path`.
- Recorded wire formats: old binary or JSON fixtures, names containing `v1`, `legacy`, or `compat`, protobuf `reserved` fields.

After deleting a test that owned a snapshot, remove that test's entries only. Jest and Vitest fail a CI run on entries that no test owns. Do not clean up with an update-all mode, which also re-records real failures: `jest -u`, `vitest -u`, `go test -update`, `cargo insta accept`, `TRYBUILD=overwrite`.

## Looks like the past

- An absence that is a policy: a removed endpoint returns 404, a dangerous option is rejected, a token, password, or key does not appear in output or logs.
- A regression test tied to a bug, issue, or CVE. It fails on the commit before the fix.
- A schema or migration check: every model change has a migration, a migration can be reversed.

## Looks like the implementation

- A characterization test of legacy code: it records what the code does, with no claim that it is right. It is often the only record that someone depends on the behavior.
- A wire-format or stored-data compatibility test, which looks like it pins internals and guards data already written.
- A test of an error code, sentinel error, or exception type that callers match on. Message wording is a contract only where something consumes it.
- A consumer-driven contract (Pact) and its provider verification.

## Looks like nothing

- A type-level test (see above).
- An accessibility assertion: a query by role or label, or an axe run. It can read as "the browser draws a button" and be the only check of the accessible name.
- A smoke, import, or start test that guards packaging, an entry point, or an optional dependency. `pytest.importorskip` skips silently when the extra is not installed, so check that CI installs it.
- A learning or contract test of a third-party library that project code relies on.
- A documented example that runs: Python `>>>` doctests, Go `Example` functions with `// Output:`, Rust doc tests, Elixir `iex>` examples, Markdown code blocks run by a docs test.

## Disabled, skipped, expected to fail

`xfail`, `test.failing`, `@expectedFailure`, `#[ignore]`, `t.Skip`, `@Disabled`, with a reason or an issue link, record a known bug or a slow test run elsewhere. Report them. When the bug is fixed, the marker is removed and the test stays.

## Rare-run guards

- Go tests that matter under `-race`; loom models in Rust (`loom::model`, `cfg(loom)`).
- Allocation assertions (`testing.AllocsPerRun`), benchmark jobs in CI.
- Property and fuzz tests, whose value is the inputs nobody thought of.

## Gates a deletion can trip

- Coverage thresholds: Jest `coverageThreshold`, Vitest `coverage.thresholds`, coverage.py `fail_under`, cargo-llvm-cov `--fail-under-lines`.
- Required status checks that name a job, a tag, or a test.
- Owner rules (`CODEOWNERS`) covering test paths, snapshots, or API reports.

Check these before deleting, and report any that the deletion would trip. Do not lower a threshold or rename a job to make the deletion pass.
