<p align="center">
  <img src="assets/brand/logo.png" width="128" alt="RusticStack Skills logo: a pixel-art branch in rust with teal and amber leaves on a dark badge, with one violet twig cut off and lifted away">
</p>

<h1 align="center">RusticStack Skills</h1>

<p align="center"><b>Agent skills that prune what a repository doesn't need.</b><br>
Short instruction files for Claude Code, Codex, Cursor and other coding agents. Each one cuts a kind of text that misleads the next reader, human or model.</p>

<p align="center">
  <img src="https://img.shields.io/badge/skills-1-CC420C?style=flat-square" alt="1 skill">
  <img src="https://img.shields.io/badge/Claude_Code-plugin_marketplace-2A2A30?style=flat-square" alt="Claude Code plugin marketplace">
  <img src="https://img.shields.io/badge/works_with-Codex%20%7C%20Cursor-2A2A30?style=flat-square" alt="Works with Codex and Cursor">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-2A2A30?style=flat-square" alt="MIT license"></a>
</p>

## Skills

| Skill | What it does | Install in Claude Code |
| --- | --- | --- |
| [prune-docs-and-comments](skills/prune-docs-and-comments/SKILL.md) | Prunes comments and docs that say nothing the code doesn't, or say something no longer true. | `/plugin install prune-docs-and-comments@rusticstack-skills` |

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

## Adding a skill

1. Create `skills/<name>/SKILL.md`.
2. Add an entry for it to `plugins` in [.claude-plugin/marketplace.json](.claude-plugin/marketplace.json).
3. Add a row to the table above.

## Acknowledgements

The shape of `prune-docs-and-comments` (one test, three ways to fail it, a done checklist, one annotated diff) follows [`prune-tests`](https://github.com/caldoai/skills/blob/main/skills/prune-tests/SKILL.md) from [caldoai/skills](https://github.com/caldoai/skills), MIT licensed.

## License

[MIT](LICENSE)
