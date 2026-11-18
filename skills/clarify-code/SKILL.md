---
name: clarify-code
description: Make code say what a reader otherwise has to work out, without changing what it does.
argument-hint: "[optional path]"
---

Change code so that it says what a reader otherwise has to work out or be told in a comment, and change nothing about what it does.

Unclear code is paid for on every read, by people and by models, and a comment that explains it goes stale. The opposite mistake costs more: a refactor that changes behavior, or one that adds a layer nobody needed. So every change here keeps behavior, is checked, and adds no layer. When you cannot check a change, do not make it. A run that changes little, and reports the rest, is a valid result.

**0. Set the run up**

- Work only on a clean tree. If anything is uncommitted, stop and ask.
- For each configuration the project supports, run the build, lint, type check, and tests twice, and keep the results. A test whose result differs between the two runs is flaky: note it, and do not count it as evidence. A failure that is already there is reported, not fixed. These checks are how each change is verified. Code under a test that already fails cannot be checked, so leave it alone.
- List the files in scope before reading. The report says how many you read out of how many you listed; do not call a partial pass done.
- Find what covers the code in scope. To learn whether a test would notice a mistake, break the function, run its tests, and restore it. If nothing fails, the code is untested, and it gets only the changes a compiler or a rename tool can verify.
- If a rule here conflicts with the task you were given or with the repository's instructions, stop and report the conflict. Do not choose between them.
- Ask once, before editing, what only the owner can decide: whether to commit, on which branch, and whether public names may change. Then run to the end without asking again. For a large repository, split the work by top-level directory.

**1. Find what the reader has to work out**

Change code only where a reader must rebuild in their head something the code could state. If a comment exists only to explain what the code does, the code is the thing to fix.

Look for meaning the code hides in:

**A name:** it says less than the thing is, or something else.

For example:

- A name that needs a comment to say what it holds: `d`, `tmp2`, `data`, `flag`, `handle()`.
- A name that is no longer true: `userList` holding a map, `getUser` that also writes.
- One thing with several names, or several things with one.
- A boolean named so that the reader has to negate it twice.

**A value:** a literal whose meaning lives in someone's memory.

For example:

- A number or string that means something: `86400`, `"3"`, `retries > 5`.
- A bare `true` or `null` at a call site, where nobody can tell what it switches.
- The same literal in several places that must change together.

**A shape:** the structure is harder to follow than the logic it carries.

For example:

- The main path buried under nested conditions that guard clauses would flatten.
- One function doing several jobs, with comments marking where each begins.
- A condition long enough that it needs a comment to say what it tests.
- The same logic copied where the copies must stay the same. Report this one: merging copies adds an abstraction, and that is the owner's decision.

**Nothing:** code a reader must study and that does no work, where the compiler or linter can show it within the file.

For example:

- Code after a return, an unused local variable, an unused import, a private function nothing in the file calls.
- A private wrapper that only forwards to one function.
- A variable that only renames another, a `return x == true`, an `else` after a return.

Anything wider is not this skill's to delete: an unused export, an interface with one implementation, an option nobody sets, a branch that looks impossible. A search that finds no reference does not prove code dead. Report these, or run `prune-dead-code`.

**2. Leave what is fixed by something outside**

Some names and shapes are not yours to improve, because something outside the code depends on them exactly as they are:

- Public API: exported names, signatures, and their order. Changing them breaks callers you cannot see.
- Names that leave the program: serialized fields, database columns, configuration keys, environment variables, routes, event names, metric and log keys.
- Names found by convention or reflection: framework hooks, test names, handlers bound by string, anything a template or a build script looks up.
- Generated code, vendored code, and code another team owns.
- Code written oddly on purpose, with a comment saying why: a hot loop, a workaround, a deliberate copy.
- Duplication in tests. A test that repeats its setup is easier to read than one that hides it.
- Code that looks dead and may not be: called by reflection, by another repository, behind a feature flag, or only on one platform. Report it.

**3. Change it**

- Rename with the language's rename tooling or the compiler as the check, not with search and replace. Rename tools miss strings, templates, and reflection, and a dynamically typed language has no compiler to catch a miss, so search for the old name as text afterwards.
- Name a value where it is used, and give it one home if it is shared.
- Flatten with guard clauses. Extract a function only when the piece has a name that says more than its body.
- Delete what does no work, where the compiler or linter confirms nothing uses it.
- Prefer removing a layer to adding one. Three similar lines are clearer than a helper invented for them, and no change here introduces a new abstraction, pattern, or dependency.
- When a change you made makes a comment redundant, delete that comment. Leave every other comment alone, including accurate ones that describe how the code works.
- Keep each change one kind of thing, small enough to review, and run the checks after it. If a check fails, undo the change. A rename may touch the tests that use the name; no assertion or expected value changes.
- Follow the conventions of the code around you, even where you would choose differently.
- Add no suppression, skip, or ignore marker.
- Before finishing, search your diff for added skip, ignore, or suppression markers and for changes to tests, snapshots, CI, coverage thresholds, or test selection. Any such change that this skill does not call for is undone.

Change no behavior. That includes the things that look harmless: the order of side effects, what gets evaluated and when, error messages and types, logging, performance of a hot path, and what happens on bad input. A bug you find is reported, not fixed, because fixing it changes behavior and belongs in its own change.

**4. Report**

End with a report the owner can review without reading the whole diff:

- Coverage: files read out of files listed.
- The totals: files changed, lines removed and added, from `git diff --numstat` against the starting commit, with new files counted.
- The changes, grouped by kind: renames (old name, new name), values named, functions flattened or split, code deleted.
- Every comment removed because the code now says it.
- What you found and left: bugs, code that looks dead and could not be proven so, public names that deserve a change, code with no test where a restructure was skipped.
- The checks and their results, before and after.

**What done looks like**

- Every file on the list was read, and every unclear spot found was either changed with a passing check or listed in the report as left.
- No comment is left that a change in this run made redundant.
- The run added no abstraction, no dependency, and no suppression.
- No public name, wire name, or behavior changed.
- The build, lint, type check, and tests give the same results as before, and no assertion or expected value was edited to get there.

**Example**

```diff
--- a/src/session.ts
+++ b/src/session.ts
@@ renamed and named: the comments were explaining a name and a number @@
-// time to live, in seconds (one day)
-const t = 86400;
+const SESSION_TTL_SECONDS = 24 * 60 * 60;
@@ flattened: the main path was three levels deep; behavior and the order of checks are the same @@
 export function refresh(session: Session | null): Session {
-  if (session) {
-    if (!session.revoked) {
-      if (session.expiresAt > now()) {
-        return extend(session, t);
-      } else {
-        throw new SessionError("expired");
-      }
-    } else {
-      throw new SessionError("revoked");
-    }
-  } else {
-    throw new SessionError("missing");
-  }
+  if (!session) throw new SessionError("missing");
+  if (session.revoked) throw new SessionError("revoked");
+  if (session.expiresAt <= now()) throw new SessionError("expired");
+  return extend(session, SESSION_TTL_SECONDS);
 }
@@ deleted: a wrapper that only forwarded, with one caller, now calling `extend` directly @@
-function doExtend(s: Session, n: number) {
-  return extend(s, n);
-}
@@ named: nobody could tell what `true` switched at the call site @@
-  store.save(session, true);
+  const overwriteExisting = true;
+  store.save(session, overwriteExisting);
@@ kept: exported, so callers outside this repository may use the name @@
 export function chkSess(id: string): boolean {
@@ kept: a field name that goes over the wire @@
   return { sess_id: session.id, exp: session.expiresAt };
@@ kept: odd on purpose, and the comment says why @@
   // Compared byte by byte in constant time; do not replace with ===.
   return timingSafeEqual(a, b);
@@ reported, not fixed: `<=` treats a session that expires this instant as expired, and the docs say it is valid @@
```
