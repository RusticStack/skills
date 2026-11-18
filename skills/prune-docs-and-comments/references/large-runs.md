# Large runs, and proving that no code changed

Read this when the scope is a large repository or several repositories, or when the build and tests cannot run.

## Coverage

- Build the file list before any reading: `rg --files --hidden -g '!.git'`. Without `--hidden` the listing skips `.github`, `.githooks`, and agent configuration directories.
- Count comment lines per file as well (`rg -c` with the language's comment marker) so a file with many comments and no edits stands out.
- After the pass, compare the files you read with the list. A script that walks the list can drop files silently, for example on names with hyphens or spaces. A file missing from your results is unread, not absent.

## Splitting the work

- Split a repository of hundreds of files by top-level directory, and give the documents to a separate part. One agent on a large repository stalls and loses its run.
- Give each part the file list for its directories, the off-limits list from `git status --porcelain`, and the coverage requirement.
- Launching an agent returns at once. Wait for the completion notices before starting more, and stay under the limit on concurrent agents; a launch over the limit fails with an error that is easy to miss.
- Two agents in one repository must not run a formatter or a fixer over the whole repository in write mode. Each formats only its own files.
- Run the checks from one place, one at a time. Checks run alongside other work can fail with no output and tell you nothing.

## Proving that no code changed

Use the project's build, type check, and tests when they run. When they cannot, because dependencies are not installed or a service is missing, compare the code with its comments removed, before and after:

- **TypeScript and JavaScript:** transpile the committed copy and the working copy with whitespace minified and compare the output, for example `bun build --no-bundle --minify-whitespace <file>`. Put the committed copy beside the original under a temporary name so its imports resolve; a copy in another directory always differs.
- **Go:** compare the token streams with comments dropped, using `go/scanner`.
- **Any language:** `git diff -U0`, then check that every changed line is a comment or sits in a document.

A comment-only diff can still change behavior where a tool reads comments. That is what `load-bearing.md` is for; this check does not replace it.

Say in the report which proof you used for each repository.

## Several repositories

- After deleting or renaming a document, search every repository in the workspace for links to it.
- Ask the owner's questions for all repositories at once, at the start, including which branch each one pushes to.
- Deliver one report for the whole run, with coverage and check results per repository.
