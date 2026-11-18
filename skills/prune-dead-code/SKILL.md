---
name: prune-dead-code
description: Prune code that nothing runs and nothing reaches, with proof for every deletion.
argument-hint: "[optional path]"
---

Prune code that nothing runs and nothing can reach, and prove each deletion before making it. A run that deletes little, and reports the rest, is a valid result.

Dead code is read, searched, compiled, and maintained as if it mattered, and a model takes it as an example of how the project works. The opposite mistake costs more: code that looked dead was called by a name in a string, by another repository, or once a year. So a deletion needs proof that nothing reaches the code. When you cannot get proof, keep it and report it.

**0. Set the run up**

- Work only on a clean tree. If anything is uncommitted, stop and ask.
- For each configuration the project supports, run the build, lint, type check, and tests twice, and keep the results. A test whose result differs between the two runs is flaky: note it, and do not count it as evidence. A failure that is already there is reported, not fixed.
- List the files in scope before reading. The report says how many you examined out of how many you listed; do not call a partial pass done. For a large repository, split the work by top-level directory.
- If a rule here conflicts with the task you were given or with the repository's instructions, stop and report the conflict. Do not choose between them.
- Ask once, before editing, what only the owner can decide: whether to commit and on which branch, whether anything outside this repository uses its code, whether public API may be removed, and which platforms, versions, flags, and jobs are retired. Then run to the end without asking again.
- Find out how the code is reached from outside: what the package exports and publishes, the entry points, the command-line and script names, the routes, the scheduled jobs, the plugins it registers.

**1. Find what reaches each piece of code**

Keep code only if something reaches it: an entry point, a caller, a test of behavior that itself is reached, or something outside the repository. Follow the references from the entry points inward; a function called only by dead code is dead too.

Prune code that is reached by:

**Nothing:** no reference anywhere.

For example:

- A function, class, constant, type, or file that nothing imports or calls.
- An export nothing outside the module uses, in a package that publishes nothing.
- A parameter, field, or variable that is written and never read.
- A style rule, translation key, asset, or template nothing refers to.

**Nothing that can happen:** it is referenced, and the path to it cannot run.

For example:

- Code after a return or a throw, or behind a condition that is always false.
- A branch for a platform, version, or state the project no longer supports.
- A feature flag that is fully rolled out or abandoned, with the branch it no longer takes.
- An error handler for an error that can no longer be raised.

**Only itself:** it is kept alive by things that exist for its sake.

For example:

- Code whose only callers are its own tests.
- A cluster of functions that call each other and are called from nowhere else.
- A compatibility shim for a caller that is gone.

**The past:** it was replaced and left behind.

For example:

- A `v1`, `old`, `legacy`, or `deprecated` copy beside the one in use.
- Commented-out code.
- A script, migration helper, or one-off tool whose job is finished.

Whether a platform is unsupported, a flag is retired, an error can no longer happen, or a job is finished is a fact about the project, not about the code. Take it from the owner or a written source. No build or test can show it.

**2. Leave what is reached in ways a search cannot see**

A search that finds no reference proves nothing by itself. Before judging, read `references/hidden-uses.md`. The kinds most often deleted by mistake:

- Code reached by name at run time: reflection, dependency injection, a handler or class named in a string, a configuration file, a route table, or a template.
- Code reached by convention: framework hooks and lifecycle methods, files a framework finds by their path, test fixtures, command and plugin entry points, serialization methods.
- Code reached from outside: published API, other repositories and packages, scripts in CI, infrastructure, and documentation examples.
- Code reached only sometimes: another platform or build configuration, a feature flag that is off here and on elsewhere, a scheduled or yearly job, an error or recovery path, a migration that old data still needs.
- Code that is present on purpose: an interface kept for stability, a reference implementation tests compare against, a variant required by a trait or protocol.

A tool's report of unused code is a list of candidates, never a reason to delete.

**3. Prove it**

For each candidate:

- **Search the whole workspace for the name as text,** not only as a symbol: source, tests, configuration, templates, scripts, CI, infrastructure, and documents, in every repository you can reach. Search for the parts of a name that could be built by concatenation.
- **Check the surface.** If it is exported from a published package, or the owner said other repositories use this code, a clean search here is not proof.
- **Let the compiler and the tests answer.** Delete it, then run the build, type check, and tests. A failure means it was reached; put it back. A pass is necessary and not sufficient: tests run only what they cover, and code loaded by name fails when the program runs.
- **Run the program** where you can, along the path that would have loaded the code.
- **Check each configuration.** For code behind a platform, feature, or build flag, build or type-check under each one the project supports.
- **For a feature flag,** confirm its state where the flags live, not from the code's default.
- **Where the project has usage data,** such as coverage from production, request logs, or a deprecation counter, use it. Coverage from the test suite shows what tests run, not what users run.

If you cannot build and run the tests, delete nothing. Report the candidates.

**4. Prune**

- Delete each piece you have proof for. What it alone kept alive, such as helpers, imports, and types, is a new candidate and needs the same proof.
- A test goes with the code only if it names the deleted symbol and tests nothing else. List each one. Any other test that fails means the code was reached: put it back.
- A mention in configuration, CI, infrastructure, or documentation is a use. Keep the code and report it.
- Delete whole files and directories when everything in them goes, and fix the indexes, exports, and build lists that named them.
- For a finished feature flag, keep the branch that runs, remove the other, and remove the flag from where it is defined.
- Delete commented-out code. Git history holds it.
- Change no behavior. Do not rewrite or tidy what stays; that is a different job.
- Remove no public API unless the owner said to. Report it as a candidate with what you found.
- Add no suppression, exclusion, skip, or ignore entry. Never regenerate snapshot or expectation files; remove only the entries of deleted tests. If a deletion would drop coverage below a threshold, keep the code and report it.
- Before finishing, search your diff for added skip, ignore, or suppression markers and for changes to tests, snapshots, CI, coverage thresholds, or test selection. Any such change that this skill does not call for is undone.
- Keep each deletion its own reviewable change where you can, so one that turns out wrong can be reverted alone.

**5. Report**

End with a report the owner can review without reading the whole diff:

- Coverage: files examined out of files listed.
- The totals: files changed and deleted, lines removed and added, from `git diff --numstat` against the starting commit. It does not count lines in binary files; list those separately.
- Every deletion, grouped by kind, with its proof: where you searched, and which builds and tests ran.
- Every test deleted with its code, by name, and the test count before and after.
- What else fell out after each deletion.
- Every candidate you kept, and what would settle it: public API, a name that appears in a string, a flag whose state you could not confirm, a platform you could not build.
- Feature flags removed, with the state you confirmed and where.
- The checks and their results, before and after, for each configuration.
- The limit of the proof: it covers this workspace and these configurations, not callers you cannot see.

**What done looks like**

- Every candidate was either deleted with proof or listed in the report as kept, with what would settle it.
- Every deletion has a workspace-wide text search and a passing build and test run behind it.
- No public API was removed without the owner's word.
- No behavior changed, and nothing that stays was rewritten.
- No suppression or ignore entry was added, and no snapshot was regenerated.
- The build, lint, type check, and tests give the same results as before, under every supported configuration. The only tests missing are the ones listed in the report as deleted with their code.
- The report's totals match `git diff --numstat` and the runner's test count.

**Example**

```diff
--- a/src/export.ts
+++ b/src/export.ts
@@ deleted: no reference in the workspace as a symbol or as text; build and tests pass without it @@
-function toLegacyCsv(rows: Row[]): string {
-  return rows.map((r) => Object.values(r).join(";")).join("\n");
-}
@@ kept: the type says `format` is "csv" | "json", but types are gone at run time, and this is what bad input hits @@
   throw new Error(`unknown format: ${format}`);
--- a/src/checkout.ts
+++ b/src/checkout.ts
@@ flag removed: the owner confirmed `new_tax_engine` is retired, and it is 100% on in the flag service; the old branch and the flag definition go @@
-  const tax = flags.enabled("new_tax_engine") ? taxEngine.quote(cart) : legacyTax(cart);
+  const tax = taxEngine.quote(cart);
--- a/src/legacyTax.ts
+++ /dev/null
@@ deleted with `legacyTax.test.ts`, which tested nothing else: its only caller was the branch above @@
--- a/src/handlers/refund.ts
+++ b/src/handlers/refund.ts
@@ kept: no import anywhere, and `routes.yaml` names it as "handlers/refund#process" @@
 export function process(event: RefundEvent) {
@@ kept: exported from the published package, so callers outside this repository may use it; reported @@
 export function parseLegacyId(id: string): number {
@@ kept: runs only on Windows; the Windows build was not available here, so it is reported, not judged @@
 function normalizeDriveLetter(path: string): string {
```
