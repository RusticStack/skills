---
name: prune-dependencies
description: Prune dependencies the project does not use, with a clean install and build as proof.
argument-hint: "[optional path]"
---

Prune dependencies the project does not use, and prove each removal with a clean install, build, and test run. A run that removes little, and reports the rest, is a valid result.

Every dependency is code someone else can change: it is installed, audited, updated, and trusted on every build. The opposite mistake costs more: a package that looked unused was loaded by a configuration file, needed on another platform, or required at run time and not at build time. So a removal needs proof. When you cannot get proof, keep the dependency and report it.

**0. Set the run up**

- Work only on a clean tree. If anything is uncommitted, stop and ask.
- Find every manifest and its lockfile, in every package of a workspace or monorepo, and the package manager and version the project pins.
- Install from the lockfile as CI does. For each configuration the project supports, run the build, lint, type check, and tests twice, and keep the results. A test whose result differs between the two runs is flaky: note it, and do not count it as evidence. A failure that is already there is reported, not fixed.
- List every declared dependency. The report says how many you examined out of how many are declared; do not call a partial pass done.
- If a rule here conflicts with the task you were given or with the repository's instructions, stop and report the conflict. Do not choose between them.
- Ask once, before editing, what only the owner can decide: whether to commit and on which branch, whether the package is published for others to depend on, and whether upgrades are in scope. They are not, unless the owner says so. Then run to the end without asking again.

**1. Find what uses each dependency**

Keep a dependency only if something uses it: an import in code that runs, a tool the project runs, a plugin a tool loads, or a consumer of the published package. Look at each one the manifest declares.

Prune dependencies that are used by:

**Nothing:** no import, no command, no configuration names it.

For example:

- A package left behind when the code that used it was removed.
- A type-definition or stub package for a library that is gone, or that now ships its own types.
- A plugin or preset for a tool the project no longer uses.
- A package added for an experiment that was never merged.

**The wrong list:** it is used, and declared where it does not belong.

For example:

- A test, lint, or build tool declared as a run-time dependency.
- A run-time import declared only as a development dependency, which works here and fails for whoever installs the package.
- A package declared in the workspace root that only one member uses, or in a member that does not use it.

**Something that already provides it:** the declaration is redundant. Remove it only when nothing uses it; otherwise report it, because the fix changes code or versions.

For example:

- Two packages that do the same job. The one the code does not use is removed. If both are used, report it.
- A package for something the language, the runtime, or the framework now includes at the version the project supports. Report it; switching to the built-in changes code.
- An override, resolution, patch, or pin for a problem that the current versions no longer have. Report it; removing it changes resolved versions.

**A name the code does not import:** it is declared, and what the code imports comes from somewhere else.

For example:

- An import that resolves only because another package happens to bring it in. This one is the opposite fault: a missing declaration. Report it; add it only if the owner agreed.

**2. Leave what is used without an import**

A search for imports proves nothing by itself. Before judging, read `references/hidden-uses.md`. The kinds most often removed by mistake:

- Packages a tool loads by name from configuration: plugins, presets, loaders, parsers, formatters, reporters, drivers, and adapters.
- Packages run as commands: from package scripts, makefiles, git hooks, CI workflows, and container files.
- Peer and optional dependencies, and packages that must be present for another package to work.
- Packages needed on another platform, runtime version, or build configuration, or only in production.
- Packages imported for their side effects, loaded by a computed name, or used only in generated code.
- Pins and overrides that exist for a security fix or a known breakage. Their reason may be in a comment, a commit, or an issue.

A tool's report of unused dependencies is a list of candidates, never a reason to remove.

**3. Prove it**

For each candidate:

- **Search the whole workspace for the name as text,** in every repository you can reach: source, configuration, scripts, CI, container and infrastructure files, and documents. Search for the import name as well, which often differs from the package name, and for every command the package installs.
- **Ask the package manager why it is there** (`npm ls`, `pnpm why`, `cargo tree -i`, `go mod why`, `pip show`, `uv tree`). A package other dependencies need stays installed after you remove the declaration. Then a clean install proves nothing, because code that imports it still finds it. For such a package only the search counts: if any import or command uses it, the declaration stays.
- **Remove it with the package manager's own command** where it has one, and never by editing the lockfile, so the lockfile changes only as much as the removal requires. Where there is no such command (Maven, Gradle, a pip requirements file), edit the manifest.
- **Install clean and run the checks.** Delete the installed packages, install from the new lockfile, then run the build, type check, lint, tests, and the verification commands the package scripts and CI run. A stale installed tree hides a missing package. Where there is no installed tree (Cargo, Go), build from clean. Run nothing that publishes, deploys, migrates, or writes to an outside system.
- **Check each configuration** the project supports: production install without development dependencies, each platform and feature set, each workspace member.
- **Start the program,** where you can. A missing run-time or dynamically loaded package fails at start, not at build.

If you cannot install and build, remove nothing. Report the candidates.

**4. Prune**

- Remove each dependency you have proof for, one at a time or in small groups, so a removal that turns out wrong can be reverted alone.
- Move a dependency to the list it belongs in when that is the fault. In a published package a move changes what its users install, so report it and move only with the owner's word.
- Remove what the dependency alone needed: its own configuration and its type stubs. An entry that names it in a detector's ignore list, or a line in a CI or container file, is a use: keep the dependency and report it.
- Change no versions. If the lockfile shows upgrades you did not ask for, undo and remove more narrowly.
- Change no code. Replacing a package with a few lines of your own, or with a built-in, is a different job; report it as an opportunity.
- Leave pins, overrides, and patches whose reason you cannot find. Report them.
- Add no ignore entry to make a tool's report clean.
- Before finishing, search your diff for added skip, ignore, or suppression markers and for changes to tests, snapshots, CI, coverage thresholds, or test selection. Any such change that this skill does not call for is undone.

**5. Report**

End with a report the owner can review without reading the lockfile diff:

- Coverage: dependencies examined out of dependencies declared.
- The totals: direct dependencies before and after, per manifest, and installed packages before and after, from the package manager's own listing.
- Every dependency removed, with its proof: where you searched, and which installs, builds, and runs passed.
- Every dependency moved between lists, and why.
- What went with each removal: configuration and stubs.
- Every candidate you kept, and what would settle it.
- Missing declarations, redundant packages, stale overrides, and replacements worth considering.
- Lockfile changes other than the removals, which should be none.
- The checks and their results, before and after, for each configuration.
- The limit of the proof: it covers the configurations you installed and ran.

**What done looks like**

- Every declared dependency was examined, and each is either used by something you can name, removed with proof, or listed in the report as kept.
- Every removal has a workspace-wide text search and a clean install, build, and test run behind it.
- No version changed, in the manifest or the lockfile, apart from packages that left.
- No code changed.
- Each misplaced dependency was moved or reported.
- The build, lint, type check, tests, and scripts give the same results as before, under every supported configuration.
- The report's counts match the package manager's listing.

**Example**

```diff
--- a/package.json
+++ b/package.json
   "dependencies": {
@@ removed: no import or command anywhere; left over from the date picker deleted in March; clean install, build, and tests pass @@
-    "moment": "^2.29.4",
@@ moved: only the test files import it @@
-    "msw": "^2.3.0",
     "react": "^18.3.1"
   },
   "devDependencies": {
+    "msw": "^2.3.0",
@@ removed: `axios` ships its own types, and the type check passes without this @@
-    "@types/axios": "^0.14.0",
@@ kept: nothing imports it, and `.eslintrc` names it under "plugins" @@
     "eslint-plugin-import": "^2.29.1",
@@ kept: nothing imports it, and the `prepare` script runs `husky` @@
     "husky": "^9.0.11",
@@ kept: nothing imports it; Sass files are compiled through the bundler, which loads it by name @@
     "sass": "^1.77.0"
   },
@@ kept and reported: an override with no stated reason; `npm ls` shows the patched version is still required by one package @@
   "overrides": { "semver": "^7.5.4" }
--- a/src/api.ts
+++ b/src/api.ts
@@ reported, not changed: `lodash` is imported here and is not declared; it resolves only because another package brings it in @@
 import { debounce } from "lodash";
```
