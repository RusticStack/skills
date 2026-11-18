<p align="center">
  <img src="assets/brand/logo.png" width="128" alt="RusticStack Skills logo: a pixel-art terminal screen in a rusted metal frame, showing green lines of text with one line cut out in orange">
</p>

<h1 align="center">RusticStack Skills</h1>

<p align="center"><b>Agent skills that leave a repository easier to read.</b><br>
Short instruction files for Claude Code, Codex, Cursor and other coding agents. Each one removes or fixes something that costs the next reader, human or model.</p>

<p align="center">
  <img src="https://img.shields.io/badge/skills-5-CC420C?style=flat-square" alt="5 skills">
  <img src="https://img.shields.io/badge/Claude_Code-plugin_marketplace-2A2A30?style=flat-square" alt="Claude Code plugin marketplace">
  <img src="https://img.shields.io/badge/works_with-Codex%20%7C%20Cursor-2A2A30?style=flat-square" alt="Works with Codex and Cursor">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-2A2A30?style=flat-square" alt="MIT license"></a>
</p>

## Skills

| Skill | What it does | Install in Claude Code |
| --- | --- | --- |
| [prune-docs-and-comments](skills/prune-docs-and-comments/SKILL.md) | Prunes comments and docs that say nothing the code doesn't, or say something no longer true. | `/plugin install prune-docs-and-comments@rusticstack-skills` |
| [prune-tests](skills/prune-tests/SKILL.md) | Prunes tests that guard no behavior anyone relies on, with proof for every deletion. | `/plugin install prune-tests@rusticstack-skills` |
| [clarify-code](skills/clarify-code/SKILL.md) | Makes code say what a reader otherwise has to work out, without changing what it does. | `/plugin install clarify-code@rusticstack-skills` |
| [prune-dead-code](skills/prune-dead-code/SKILL.md) | Prunes code that nothing runs and nothing reaches, with proof for every deletion. | `/plugin install prune-dead-code@rusticstack-skills` |
| [prune-dependencies](skills/prune-dependencies/SKILL.md) | Prunes dependencies the project does not use, with a clean install and build as proof. | `/plugin install prune-dependencies@rusticstack-skills` |

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

It reads every file, not a sample, and its report gives the count. For a large repository or several at once, [`references/large-runs.md`](skills/prune-docs-and-comments/references/large-runs.md) covers splitting the work and showing that no code changed when the checks cannot run.

It has had one large trial, a sweep of about twenty repositories, and the lessons from it are in the skill. Review the diff it produces before you merge it.

### What it is built on

The rules come from a review of published research and vendor guidance, including:

- Studies of how comments affect model accuracy. In two studies wrong comments scored worse than no comments. Accurate comments helped in some studies and made no significant difference in another. In one study, defective commented-out code in the prompt raised defects in generated code.
- Studies of repository instruction files. One benchmark found that generated files raised cost without raising task success, and that developer-written files gave a small average gain, also at higher cost. A smaller study found no measurable effect.
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

- **It proves each deletion.** For a test that looks unable to fail, the agent breaks the behavior it claims to guard and runs it; a test that fails stays. A test tied to the implementation is rewritten, not deleted. Reading alone is never the evidence.
- **It never touches a failing test.** A failing test is reported with its output, not deleted, skipped, or weakened.
- **It leaves what only looks useless.** Type-level and compile-fail tests, security checks that something stays removed, characterization tests, snapshots, and saved fuzz inputs stay. [`references/load-bearing.md`](skills/prune-tests/references/load-bearing.md) lists the kinds and ten rules for recognizing one that isn't listed.
- **It changes no production code,** and it does not regenerate snapshots, add skip markers, or edit CI settings and coverage thresholds.
- **It repairs before it deletes.** A test whose assertions never run, for behavior that still exists, is fixed so they run.
- **It reports everything:** each deleted test with its evidence (the break it did not catch, the twin that stays, or the search that found nobody asking for it), and each candidate kept for lack of evidence.

It needs a suite it can run. Without one it deletes nothing and reports its candidates.

It is new and has had no trial run. Review the diff before you merge it.

### What it is built on

- Studies of pseudo-tested code, where a method's body can be removed without any test failing, and of tests that pass while their assertions never run.
- Research on coverage and mutation testing, which found coverage only weakly tied to fault detection. The skill infers from it that matching coverage does not show a deletion is safe; no study tested that directly.
- User reports of coding agents deleting or rewriting tests to get a green run, and one benchmark of impossible tasks that measured test tampering and which guardrails reduced it.
- The testing chapters of *Software Engineering at Google*, Kent Beck's test desiderata, and the documentation of the snapshot, type-test, property-test, and mutation tools it names.

## clarify-code

Run it on a repository or a path:

```
/clarify-code
/clarify-code src/billing
```

Where `prune-docs-and-comments` keeps a comment that explains unclear code, this skill fixes the code so the comment is no longer needed. The agent looks for meaning the code hides in:

- **A name** that says less than the thing is, or something else.
- **A value** whose meaning lives in someone's memory, such as a bare `86400` or `true`.
- **A shape** that is harder to follow than the logic it carries, such as a main path under four levels of nesting.
- **Nothing:** code that does no work, where the compiler or linter can show it within the file. Anything wider is reported or left to `prune-dead-code`.

What keeps it safe:

- **It changes no behavior.** A bug it finds is reported, not fixed.
- **Every change is checked.** It runs the checks after each change and undoes one that fails. Code with no test gets only what a compiler or a rename tool can verify.
- **It leaves what something outside depends on:** exported names, fields that go over the wire, database columns, names found by reflection, and code written oddly on purpose.
- **It adds no layers.** No new abstraction, pattern, dependency, or suppression. Merging duplicated logic is reported, not done.

This one is written from general refactoring practice. It was audited against the research behind the first two skills, and has had no research pass of its own and no trial run. Review its diff with more care.

## prune-dead-code

```
/prune-dead-code
/prune-dead-code src/billing
```

The agent follows references inward from the entry points and looks for code reached by nothing, by a path that cannot run, only by its own tests, or left behind after a replacement. Finished feature flags go with the branch they no longer take.

- **A clean search is not proof.** Each deletion needs a text search of the whole workspace and a passing build and test run without the code, under every configuration the project supports. A test is deleted with the code only if it tests nothing else, and each one is listed.
- **It leaves what a search cannot see.** Code reached by a name in a string, by a file's path, from another repository, or once a year stays. [`references/hidden-uses.md`](skills/prune-dead-code/references/hidden-uses.md) lists the kinds.
- **It removes no public API** unless the owner says to, changes no behavior, and adds no suppression to quiet a tool.

## prune-dependencies

```
/prune-dependencies
```

The agent checks each declared dependency for a use: an import in code that runs, a command the project runs, a plugin a tool loads, or a consumer of the published package. It removes the unused, moves the misplaced to the right list, and reports redundant packages, stale overrides, and imports that have no declaration.

- **The proof is a clean install.** After each removal it deletes the installed packages, installs from the new lockfile, and runs the build, tests, and verification scripts. A stale installed tree hides a missing package. Where another package keeps the dependency installed anyway, only the search counts.
- **It leaves what is used without an import:** plugins named in configuration, packages run as commands, peer and optional dependencies, and pins that exist for a security fix. [`references/hidden-uses.md`](skills/prune-dependencies/references/hidden-uses.md) lists the kinds.
- **It changes no versions and no code.** Upgrades are out of scope unless the owner asks, and replacing a package with your own code is reported as an opportunity.

Like `clarify-code`, these two are written from general practice. They were audited against the research behind the other two skills, and have had no research pass of their own and no trial run.

## Adding a skill

1. Create `skills/<name>/SKILL.md`.
2. Add an entry for it to `plugins` in [.claude-plugin/marketplace.json](.claude-plugin/marketplace.json).
3. Add a row to the table above.

## Acknowledgements

The shape of these skills (one test, a few ways to fail it, a done checklist, one annotated diff) follows [`prune-tests`](https://github.com/caldoai/skills/blob/main/skills/prune-tests/SKILL.md) from [caldoai/skills](https://github.com/caldoai/skills), MIT licensed.

## License

[MIT](LICENSE)
