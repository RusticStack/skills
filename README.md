<p align="center">
  <img src="assets/brand/logo.png" width="128" alt="RusticStack Skills logo: a pixel-art branch in rust with teal and amber leaves on a dark badge, with one violet twig cut off and lifted away">
</p>

<h1 align="center">RusticStack Skills</h1>

<p align="center"><b>Agent skills that prune what a repository doesn't need.</b><br>
Short instruction files for Claude Code, Codex, Cursor and other coding agents. Each one cuts a kind of text that misleads the next reader, human or model.</p>

<p align="center">
  <img src="https://img.shields.io/badge/skills-2-CC420C?style=flat-square" alt="2 skills">
  <img src="https://img.shields.io/badge/Claude_Code-plugin_marketplace-2A2A30?style=flat-square" alt="Claude Code plugin marketplace">
  <img src="https://img.shields.io/badge/works_with-Codex%20%7C%20Cursor-2A2A30?style=flat-square" alt="Works with Codex and Cursor">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-2A2A30?style=flat-square" alt="MIT license"></a>
</p>

## Skills

| Skill | What it does | Install in Claude Code |
| --- | --- | --- |
| [prune-docs-and-comments](skills/prune-docs-and-comments/SKILL.md) | Prunes comments and docs that say nothing the code doesn't, or say something no longer true. | `/plugin install prune-docs-and-comments@rusticstack-skills` |
| [prune-tests](skills/prune-tests/SKILL.md) | Prunes tests that guard no behavior anyone relies on, with proof for every deletion. | `/plugin install prune-tests@rusticstack-skills` |

## Install

With the [skills CLI](https://github.com/vercel-labs/skills), which works with Claude Code, Cursor, Codex and other agents:

```
npx skills add rusticstack/skills
```

Or, in Claude Code, add this repository as a plugin marketplace, then install a skill with the command in its row of the table above:

```
/plugin marketplace add rusticstack/skills
```

To install by hand, copy a skill's folder into `~/.claude/skills/`.

## prune-docs-and-comments

Run it on a repository or a path:

```
/prune-docs-and-comments
/prune-docs-and-comments src/billing
```

The agent asks one question of every comment and document: what does it depend on? Prose stays only if it depends on intent the code cannot express, such as why the code is this way, what must stay true, or what a caller may rely on. It goes if it depends on:

- **The past.** It narrates an edit, describes code that is gone, or is a finished plan or status report. Git history already holds it.
- **Nothing.** It restates the code beside it, so no reader learns anything from it.
- **The implementation.** It copies values, file trees or call lists out of the code, and is wrong the first time someone forgets to update it.

What makes it safe to run:

- **It changes no code.** The diff contains comments and documents only, so the build, lint, type check and tests give the same results before and after.
- **It leaves what tools read.** Linter and type-checker suppressions, build tags, license headers, doctests, expected-output comments in tests, and docstrings that become command-line help or API schemas stay exactly as they are. [`references/load-bearing.md`](skills/prune-docs-and-comments/references/load-bearing.md) lists the known forms for more than twenty languages and formats, and ten rules for recognizing one that isn't listed.
- **It keeps when unsure.** A deleted reason is gone for good, so prose the agent cannot fault stays.
- **It needs evidence for staleness.** A comment is stale when the agent can point to the code that contradicts it. Age is not evidence.
- **It reports what it won't settle.** When a comment and the code disagree and the code may be the one that is wrong, both stay and the disagreement goes in the report.

Agent instruction files (`CLAUDE.md`, `AGENTS.md`, rule files) get the same test, held more strictly, because every line in them is read on every task. Directory tours, style rules a linter already enforces, and generic advice go. Exact commands, traps, and the owner's stated preferences stay.

The skill is new. Review the diff it produces before you merge it.

### What it is built on

The rules come from a review of published research and vendor guidance, including:

- Studies of how comments affect model accuracy. Wrong comments scored worse than no comments, accurate comments helped, and commented-out code in the prompt raised defects in generated code.
- Studies of repository instruction files, which found that generated files raised cost without raising task success, while short developer-written ones helped.
- Comment taxonomies and code-comment inconsistency research from software engineering.
- The comment sections of the Google style guides, the Linux kernel coding style, PEP 8 and PEP 257, the Go doc comment guide, and the Rust API guidelines.
- The documentation of the linters, type checkers, build tools and doc generators whose directives live in comments.

## prune-tests

Run it on a repository or a path:

```
/prune-tests
/prune-tests tests/billing
```

The agent asks what each test depends on. A test stays only if it depends on behavior someone relies on: a requirement, a documented API, a reported bug, a consumer, or a policy. It is a candidate to go if it depends on:

- **Nothing.** It has no assertion that runs, or it asserts on a value it built itself.
- **The implementation.** It fails when the code is restructured and the behavior is unchanged.
- **The past.** It guards a decision already made, such as deleted code staying deleted.
- **Another test.** The same input and the same asserted facts, at the same layer.

What makes it safe to run:

- **It proves each deletion.** The agent breaks the behavior a test claims to guard and runs the test. A test that fails stays. Reading alone is never the evidence.
- **It never touches a failing test.** A failing test is reported with its output, not deleted, skipped, or weakened.
- **It leaves what only looks useless.** Type-level and compile-fail tests, security checks that something stays removed, characterization tests, snapshots, and saved fuzz inputs stay. [`references/load-bearing.md`](skills/prune-tests/references/load-bearing.md) lists the kinds and ten rules for recognizing one that isn't listed.
- **It changes no production code,** and it does not regenerate snapshots, add skip markers, or edit CI settings and coverage thresholds.
- **It repairs before it deletes.** A test whose assertions never run, for behavior that still exists, is fixed so they run.
- **It reports everything:** each deleted test with the break that it did not catch, and each candidate kept for lack of evidence.

It needs a suite it can run. Without one it deletes nothing and reports its candidates.

Like the other skill, it is new. Review the diff before you merge it.

### What it is built on

- Studies of pseudo-tested code, where a method's body can be removed without any test failing, and of tests that pass while their assertions never run.
- Research on coverage and mutation testing, which found that matching coverage does not show a deletion is safe.
- Published reports and benchmarks of coding agents deleting or rewriting tests to get a green run, and of which guardrails reduced it.
- The testing chapters of *Software Engineering at Google*, Kent Beck's test desiderata, and the documentation of the snapshot, type-test, property-test, and mutation tools it names.

## Adding a skill

1. Create `skills/<name>/SKILL.md`.
2. Add an entry for it to `plugins` in [.claude-plugin/marketplace.json](.claude-plugin/marketplace.json).
3. Add a row to the table above.

## Acknowledgements

The shape of both skills (one test, a few ways to fail it, a done checklist, one annotated diff) follows [`prune-tests`](https://github.com/caldoai/skills/blob/main/skills/prune-tests/SKILL.md) from [caldoai/skills](https://github.com/caldoai/skills), MIT licensed.

## License

[MIT](LICENSE)
