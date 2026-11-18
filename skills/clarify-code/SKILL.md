---
name: clarify-code
description: Make code say what a reader otherwise has to work out, without changing what it does.
argument-hint: "[optional path]"
---

Change code so that it says what a reader otherwise has to work out or be told in a comment, and change nothing about what it does.

Unclear code is paid for on every read, by people and by models, and a comment that explains it goes stale. The opposite mistake costs more: a refactor that changes behavior, or one that adds a layer nobody needed. So every change here keeps behavior, is checked by the tests, and removes more than it adds. When you cannot check a change, do not make it.

**0. Set the run up**

- Work only on a clean tree, and record `git status --porcelain`. If files are already changed, stop and ask.
- Run the build, lint, type check, and tests once, and keep the results. They are how each change is checked. A failure that is already there is reported, not fixed.
- Find what covers the code in scope. Code with no test that would notice a mistake gets only the changes a compiler or a rename tool can verify.
- Ask once, before editing, what only the owner can decide: whether to commit, on which branch, and whether public names may change.

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
- The same logic copied where the copies must stay the same.

**Nothing:** code a reader must study and that does no work.

For example:

- Code that cannot run: after a return, behind a condition that is never true, a parameter nothing reads, an import nothing uses.
- A wrapper that only forwards, an interface with one implementation and no second in sight, an option nobody sets.
- A variable that only renames another, a `return x == true`, an `else` after a return.

**2. Leave what is fixed by something outside**

Some names and shapes are not yours to improve, because something outside the code depends on them exactly as they are:

- Public API: exported names, signatures, and their order. Changing them breaks callers you cannot see.
- Names that leave the program: serialized fields, database columns, configuration keys, environment variables, routes, event names, metric and log keys.
- Names found by convention or reflection: framework hooks, test names, handlers bound by string, anything a template or a build script looks up.
- Generated code, vendored code, and code another team owns.
- Code written oddly on purpose, with a comment saying why: a hot loop, a workaround, a deliberate copy.
- Duplication in tests. A test that repeats its setup is easier to read than one that hides it.
- Code that looks dead and may not be: called by reflection, by another repository, behind a feature flag, or only on one platform. Search before removing, and report what you could not settle.

**3. Change it**

- Rename with the language's rename tooling or the compiler as the check, not with search and replace.
- Name a value where it is used, and give it one home if it is shared.
- Flatten with guard clauses. Extract a function only when the piece has a name that says more than its body.
- Delete what does no work, once a search shows nothing uses it.
- Prefer removing a layer to adding one. Three similar lines are clearer than a helper invented for them, and no change here introduces a new abstraction, pattern, or dependency.
- When a change makes a comment redundant, delete the comment. Keep every comment that says why.
- Keep each change one kind of thing, small enough to review, and run the checks after it. If a check fails, undo the change; do not adjust the test to fit.
- Follow the conventions of the code around you, even where you would choose differently.

Change no behavior. That includes the things that look harmless: the order of side effects, what gets evaluated and when, error messages and types, logging, performance of a hot path, and what happens on bad input. A bug you find is reported, not fixed, because fixing it changes behavior and belongs in its own change.

**4. Report**

End with a report the owner can review without reading the whole diff:

- The totals: files changed, lines removed and added, from `git diff --numstat` against the starting commit.
- The changes, grouped by kind: renames (old name, new name), values named, functions flattened or split, code deleted.
- Every comment removed because the code now says it.
- What you found and left: bugs, code that looks dead and could not be proven so, public names that deserve a change, code with no test where a restructure was skipped.
- The checks and their results, before and after.

**What done looks like**

- No comment is left that only explains what the code does.
- Every name says what the thing is, and one thing has one name.
- No literal carries meaning that is not named.
- Nothing is left that cannot run or does no work, as far as a search could show.
- The run added no abstraction and no dependency, and removed at least as many lines as it added.
- No public name, wire name, or behavior changed.
- The build, lint, type check, and tests give the same results as before, and no test was edited to get there.

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
