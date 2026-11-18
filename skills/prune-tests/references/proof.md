# Commands for proving a test guards something

Use the repository's own scripts where they exist; these are the underlying commands. Check a flag against the tool's help before relying on it, since versions differ.

A break made by hand needs no tool: edit the code, run one test, restore, and confirm `git diff` shows no change to that file.

## List tests, and find the ones that do not run

| Ecosystem | List | Skipped and ignored |
| --- | --- | --- |
| Python (pytest) | `pytest --collect-only -q` | `pytest -rxXs` |
| JavaScript (Jest) | `npx jest --listTests` (files only) | search for `.skip`, `.only`, `xit`, `test.failing` |
| JavaScript (Vitest) | `npx vitest list` | search for `.skip`, `.only`, `.fails` |
| Rust | `cargo test -- --list` | `cargo test -- --ignored` |
| Go | `go test -list . ./...` (top-level tests) | search for `t.Skip` |

## Run one test

| Ecosystem | Command |
| --- | --- |
| Python | `pytest path/to/test_file.py::test_name` |
| Jest | `npx jest path/to/file.test.ts -t "test name"` |
| Vitest | `npx vitest run path/to/file.test.ts -t "test name"` |
| Rust | `cargo test module::path::test_name -- --exact` |
| Go | `go test -run '^TestName$' ./pkg` |

## Which test runs which line

- Python: set `dynamic_context = test_function` under `[run]` in the coverage.py configuration, or run `pytest --cov=<package> --cov-context=test`, then `coverage html --show-contexts`.
- JavaScript and TypeScript: StrykerJS with `"coverageAnalysis": "perTest"` (its default). Jest and Vitest alone give coverage per run, not per test.
- .NET: Stryker.NET with `coverage-analysis` set to `perTest` (its default).
- Go and Rust: run the single test with coverage (`go test -run '^TestName$' -coverprofile=cover.out ./pkg`, `cargo llvm-cov --lcov --output-path lcov.info -- test_name`).

Coverage shows that a line ran. It does not show that a fault on that line would be caught.

## Does anything check that an assertion ran

- Jest and Vitest: `expect.hasAssertions()` or `expect.assertions(n)` inside the test fails it when no assertion ran.
- ESLint `jest/expect-expect` and RuboCop `RSpec/NoExpectationExample` find tests with no assertion in the source. They do not find an assertion that is present and never reached.
- PHPUnit reports a test with no assertion as risky by default.

## Mutation tools

Run on the files the candidates cover, not the whole repository. A surviving mutant can be equivalent to the original code, so it is evidence, not a verdict.

| Ecosystem | Tool | Command |
| --- | --- | --- |
| Python | mutmut | `mutmut run "my_module*"` |
| JavaScript, TypeScript | StrykerJS | `npx stryker run`, with `mutate` set to the files |
| Rust | cargo-mutants | `cargo mutants -f 'src/path/*.rs'` (results in `mutants.out/`) |
| Go | gremlins | `gremlins unleash` |
| Java, Kotlin | PIT | `mvn test-compile org.pitest:pitest-maven:mutationCoverage`, with `targetClasses` and `targetTests` |
| C# | Stryker.NET | `dotnet stryker -m "**/File.cs"` |

A function whose whole body can be replaced by a default return with no test failing is called pseudo-tested. The Descartes engine for PIT automates that check for Java; by hand it is the first break to try.

Tools that mutate in place can leave edits behind if interrupted. Check `git status` after a run.
