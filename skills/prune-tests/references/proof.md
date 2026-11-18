# Commands for proving a test guards something

Use the repository's own scripts where they exist; these are the underlying commands. Check a flag against the tool's help before relying on it, since versions differ.

A break made by hand needs no tool: edit the code, run one test, restore, and confirm `git diff` shows no change to that file.

## List tests, and find the ones that do not run

| Ecosystem | List | Skipped and ignored |
| --- | --- | --- |
| Python (pytest) | `pytest --collect-only -q` | `pytest -rxXs` |
| JavaScript (Jest) | `npx jest --listTests` (files only) | search for `.skip`, `.only`, `xit`, `test.failing` |
| JavaScript (Vitest) | `npx vitest list` (`--json=<file>` to save it, `--filesOnly` for files) | search for `.skip`, `.only`, `.fails` |
| Rust | `cargo test -- --list` | `cargo test -- --ignored` |
| Go | `go test -list . ./...` (top-level tests) | search for `t.Skip` |

## Run one test

| Ecosystem | Command |
| --- | --- |
| Python | `pytest path/to/test_file.py::test_name` |
| Jest | `npx jest path/to/file.test.ts -t "test name"` |
| Vitest | `npx vitest run path/to/file.test.ts -t "test name"` |
| Rust | `cargo test -- --exact module::path::test_name` |
| Go | `go test -run '^TestName$' ./pkg` |

## Which test runs which line

- Python: set `dynamic_context = test_function` under `[run]` in the coverage.py configuration, or run `pytest --cov=<package> --cov-context=test`, then `coverage html --show-contexts`. coverage.py finds tests by name (functions starting with `test`), and lines run before the first test fall in an empty context.
- JavaScript, TypeScript, and .NET: Jest and Vitest give coverage per run, not per test. StrykerJS (`"coverageAnalysis": "perTest"`, its default) and Stryker.NET (`coverage-analysis` set to `perTest` in its configuration file, its default) use per-test coverage to pick the tests for each mutant; whether their reports let you read it back per test is not confirmed. Per-test analysis assumes the tests are independent and safe in any order.
- Go: run the single test with coverage, `go test -run '^TestName$' -coverprofile=cover.out ./pkg`.
- Rust: `cargo llvm-cov --lcov --output-path lcov.info` gives coverage for a run. Narrowing it to one test with a trailing filter is not confirmed; check `cargo llvm-cov --help`.

Coverage shows that a line ran. It does not show that a fault on that line would be caught.

## Does anything check that an assertion ran

- Jest and Vitest: `expect.hasAssertions()` or `expect.assertions(n)` inside the test fails it when no assertion ran.
- ESLint `jest/expect-expect` and RuboCop `RSpec/NoExpectationExample` find tests with no assertion in the source. They do not find an assertion that is present and never reached.
- PHPUnit reports a test with no assertion as risky by default.

## Mutation tools

Run on the files the candidates cover, not the whole repository, where the tool can be narrowed. A surviving mutant can be equivalent to the original code, so it is evidence, not a verdict.

| Ecosystem | Tool | Command |
| --- | --- | --- |
| Python | mutmut | `mutmut run "my_module*"` |
| JavaScript, TypeScript | StrykerJS | `npx stryker run`, with `mutate` set to the files |
| Rust | cargo-mutants | `cargo mutants -f 'src/path/*.rs'` (results in `mutants.out/`; exit code 2 means mutants were missed) |
| Go | gremlins | `gremlins unleash` (no file filter confirmed; run it per module) |
| Java, Kotlin | PIT | `mvn test-compile org.pitest:pitest-maven:mutationCoverage`, with `targetClasses` and `targetTests` |
| C# | Stryker.NET | `dotnet stryker -m "**/File.cs"` |

A function that the tests run, and whose whole body can be removed or replaced by a default return with no test failing, is called pseudo-tested. The Descartes engine for PIT automates that check for Java; by hand it is the first break to try.

A tool that mutates the real tree can leave edits behind if interrupted. cargo-mutants does with `--in-place`, and marks them `/* ~ changed by cargo-mutants ~ */`. Check `git status` after a run.

cargo-mutants moves the previous `mutants.out` to `mutants.out.old` on the next run, so copy `caught.txt` out before the second run of a before-and-after comparison.
