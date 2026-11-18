---
name: prune-tests
description: Prune tests that guard no behavior anyone relies on, with proof for every deletion.
argument-hint: "[optional path]"
---

Prune tests that guard no behavior anyone relies on, and prove each deletion. The aim is a suite people can trust, not a smaller one, and a run that deletes nothing is a valid result.

A test that cannot fail gives false confidence, and a test that fails on every refactor teaches people to ignore failures. The opposite mistake costs more: a deleted guard is gone silently, and nobody learns it until the bug ships. So a deletion needs evidence. When you cannot get evidence, keep the test and report it.

**0. Start from a known state**

- Work only on a clean tree. If anything is uncommitted, stop: pruning and other changes do not go in the same run.
- Run the suite twice and record the results: passing, failing, skipped, expected-to-fail, and any test whose result differs between the two runs.
- A failing test is not a pruning candidate. Never delete, skip, or weaken one to get a green run. Report it with its output.
- If you cannot run the tests, delete nothing. Report the candidates and why you think so.
- If a rule here conflicts with the task you were given or with the repository's instructions, stop and report the conflict. Do not choose between them.

**1. Find what each test depends on**

Keep a test only if it depends on behavior someone relies on: a requirement, a documented API, a reported bug, a consumer, or a policy. It should fail when that behavior regresses, and keep passing through a refactor that preserves it. The code as it stands today is not evidence of what is intended; look for a source outside the code and the test. The exception is legacy code with no specification, where what the code does now is assumed to be required.

A test that depends on one of these is a candidate. Section 3 says what evidence each kind needs, and whether it is deleted, rewritten, or repaired:

**Nothing:** it cannot fail, so it guards nothing.

For example:

- It has no assertion, or its assertions never run: they sit in a branch not taken, a callback never called, or after an early return.
- It asserts on a value the test built itself, or on what a mock was told to return.
- It computes the expected value with the same logic as the code under test.
- It exercises the language, the framework, or a library, and no project code is involved.

**The implementation:** it fails when the code is restructured and the behavior is unchanged.

For example:

- It reads private fields or internal state that no caller can observe.
- Its only assertions are that an in-process collaborator was called, or called in a certain order.
- It pins a constant, a log line, or wording that nothing consumes. In a published library you cannot see the consumers, so tests of public constants, error values, and output formats stay.

**The past:** it guards a decision already made, not a behavior.

For example:

- It asserts that deleted code, a removed flag, or a finished migration is gone, and bringing it back would be a choice, not a regression.
- It was a one-off check for a task that is complete.

**Another test:** the same input and the same asserted facts, at the same layer. Two tests that run the same lines and assert different facts are not duplicates, and neither are a unit test and a contract test of the same behavior, or a hand-written API test and an API report tool. Where one of a pair asserts more, it is not a twin: report the pair. Keep the one a gate or a consumer depends on.

**2. Leave what is load-bearing**

Some tests look like one of the cases above and are the only guard for something. Before pruning, read `references/load-bearing.md`. The kinds that are most often pruned by mistake:

- Tests judged by a tool other than the test runner: type-level tests, compile-fail tests, static assertions, API reports, architecture rules. They do nothing at run time on purpose. Find the tool in the CI configuration before judging them.
- Assertions of absence that are a policy: a removed endpoint stays 404, a dangerous option stays rejected, a secret stays out of the logs.
- Characterization tests that pin odd current behavior of legacy code, and regression tests that cite a bug, an issue, or a CVE.
- Snapshot, golden, and approved files, and saved failing inputs from property and fuzz tests. They are expectations and inputs, not clutter.
- Tests marked skipped or expected-to-fail with a reason. They record a known bug or a test that runs somewhere else. When the bug is fixed the marker goes, not the test.
- Doc tests and examples, tests of error codes and messages that callers match on, accessibility queries, and race, allocation, and benchmark guards.

A code smell is a reason to look at a test, never a reason to delete it.

**3. Prove it**

Reading is not proof. The evidence depends on the kind of candidate:

- **Nothing: break the code.** Break the behavior the test claims to guard: invert the condition, drop the side effect, return a default. Run that test. If it fails, it guards something and stays. If it passes through every break you can make to that behavior, that is your evidence. Then decide: where the behavior still exists and the assertions never ran, repair the test so they run; delete it only where there is nothing to assert.
- **The implementation: name the refactor.** Such a test does fail when the code is broken, so a break proves nothing. Name a rename, extraction, or reordering that preserves behavior and would make it fail. Then rewrite it to assert the outcome through the public interface. Delete it only when another test already asserts that outcome and fails under the break.
- **The past: find who asks for it.** Such a test fails when the removed thing comes back, by design. The evidence is that no requirement, policy, document, issue, or consumer asks for the absence. Say where you looked. If any of them might, keep the test.
- **Another test: break the code against the twin.** The test that stays must fail under the break the deleted one would have caught.

For every break:

- Confirm the test ran. A test that was skipped, ignored, or deselected passes every break. Run it under the tool that judges it (`tsc`, `tsd`, mypy, `-race`, `cargo test --doc`).
- Run it three times when its result decides a deletion. A flaky test can fake either outcome.
- Restore the code and confirm `git diff` shows no change to it.

If the repository has a mutation tool, run it on the files the candidates cover, before and after the deletion. No mutant caught before may be missed after. Leave out, and report, any mutant whose result changes between repeated runs.

`references/proof.md` has commands for listing tests, running one test, attributing coverage to a test, checking that assertions ran, and mutation tools, for the common ecosystems.

**4. Prune**

- Delete each test you have evidence against.
- Rewrite instead of deleting when the test covers behavior someone relies on but reaches it through the implementation: assert the outcome through the public interface. Where assertions never run and the behavior still exists, repair the test so they run.
- Change no production code. Report code that exists only for a deleted test, such as a hook or a helper method, with its callers; the owner decides.
- Never regenerate expectation files to get to green. `-u`, `-update`, `accept`, and `overwrite` modes bless whatever the code does now. Remove only the entries that belonged to deleted tests.
- Add no skip, only, focus, ignore, or expected-failure marker. Do not edit CI configuration, coverage thresholds, or test selection.
- Remove fixtures and helpers that only deleted tests used, after searching the whole repository, including other packages, for other users. Search for the name, not the import: pytest fixtures are used by name. A helper that is exported, published, or registered as a plugin stays and is reported.
- Check what a deletion would trip before you make it: coverage thresholds, required CI job names, and owner rules on the test paths. If it would trip one, keep the test and report it.
- Before finishing, search your diff for added skip, only, focus, ignore, or expected-failure markers and for changes to CI, coverage, or test-selection files. Any hit is undone.

**5. Report**

End with a report the owner can review without reading the whole diff:

- The totals: tests before and after, files changed and deleted, lines removed and added. Take line counts from `git diff --numstat` against the starting commit and test counts from the runner's own listing.
- Every test deleted, with the reason and the evidence: the break you made and the result, or where you looked for a requirement.
- Every test rewritten or repaired: what it asserted and what it asserts now.
- Every candidate you kept for lack of evidence, and what would settle it. Include tests for which you found no requirement and no proof either way.
- Every failing, flaky, skipped, or expected-to-fail test you found, untouched.
- Production code that now has no test or no caller.
- The suite results and coverage before and after, and the commands and tool versions you ran.
- The limit of the proof: it covers the breaks you made, not every possible fault.

**What done looks like**

- Every test you judged and kept fails when the behavior it names is broken, or is in the report as kept without proof.
- Every deletion has its evidence recorded: a break the test did not catch, a twin that catches it, or the search that found nobody asking for it.
- No test was deleted, skipped, or weakened because it failed.
- No expectation file was regenerated, and no marker, CI setting, or threshold changed.
- The diff changes no production code.
- The suite gives the same results as before for every test that remains, and every gate still passes.
- The report's totals match `git diff --numstat` and the runner's test count.

**Example**

```diff
--- a/tests/cart.test.ts
+++ b/tests/cart.test.ts
@@ deleted: with `total()` changed to return 0, this still passed; it asserts on the mock's own return value @@
-test("uses the price service", () => {
-  priceService.get.mockReturnValue(10);
-  expect(priceService.get("sku-1")).toBe(10);
-});
@@ rewritten: it failed when `addItem` was split in two with no change in behavior; it now asserts the outcome @@
 test("adding an item updates the total", () => {
   cart.addItem("sku-1", 2);
-  expect(cart._recalculate).toHaveBeenCalledTimes(1);
+  expect(cart.total()).toBe(20);
 });
@@ repaired: the assertion sat in a callback that was never awaited, so it never ran @@
-test("rejects an empty cart", () => {
-  checkout(emptyCart).catch((e) => expect(e.code).toBe("EMPTY_CART"));
+test("rejects an empty cart", async () => {
+  await expect(checkout(emptyCart)).rejects.toMatchObject({ code: "EMPTY_CART" });
 });
@@ deleted: same input and same asserted fact as "adding an item updates the total" @@
-test("total reflects added items", () => {
-  cart.addItem("sku-1", 2);
-  expect(cart.total()).toBe(20);
-});
@@ kept: an absence that is a policy, though it reads like history @@
 test("the legacy export endpoint stays removed", async () => {
   expect((await get("/v1/export")).status).toBe(404);
 });
@@ kept: it pins a rounding quirk and cites the bug; breaking the rounding makes it fail @@
 // Regression: #482, totals off by one cent for three-way splits.
 test("splits 0.10 three ways as 0.04, 0.03, 0.03", () => {
   expect(split(0.1, 3)).toEqual([0.04, 0.03, 0.03]);
 });
--- a/tests/types.test-d.ts
+++ b/tests/types.test-d.ts
@@ kept: it does nothing at run time; `tsd` in CI is what judges it @@
 expectType<Cart>(createCart());
--- a/tests/__snapshots__/cart.test.ts.snap
+++ b/tests/__snapshots__/cart.test.ts.snap
@@ untouched: no deleted test had a snapshot, and nothing was regenerated @@
```
