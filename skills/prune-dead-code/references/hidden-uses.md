# Code that is used in ways a search cannot see

Read this before judging any code dead. The recognition rules come first, because they cover the framework this file does not list.

Keeping code that turned out to be dead costs some reading. Deleting code that was reached by a string, another repository, or a job that runs once a year costs an outage. When unsure, keep it and report it.

## Recognition rules

1. **A name can be data.** Anything that turns a string into code can reach a symbol with no reference to it: reflection, dynamic import, dependency injection, a registry, an event bus, a serializer. Search for the name as text, and for the pieces it could be assembled from.
2. **A path can be a call.** Frameworks load files by where they are and what they are called. A file with no importer may be a route, a page, a migration, a command, a test, or a plugin.
3. **A decorator or attribute is a caller.** Code marked with one is registered with something: a route, a task, a fixture, a signal handler, a command, an export to another language.
4. **Public means unknown callers.** Anything exported from a published package, a shared library, or a service's API may be used by code you cannot search.
5. **The build you ran is one of several.** Code can be dead on this platform, in this feature set, in this build mode, and live in another.
6. **Rare is not never.** Error handling, recovery, rollback, a migration for old data, a yearly or end-of-period job, and an operator's manual script run seldom and matter most when they do.
7. **Other files reach in.** CI workflows, Docker and infrastructure files, package scripts, makefiles, cron tables, documentation examples, and other repositories call code by name or path.
8. **Some code exists to be present.** A method an interface or protocol requires, a symbol kept for binary or source compatibility, a reference implementation that tests compare against.
9. **Tests alone do not make code live.** Code reached only from its own tests is a candidate. Code that tests use as a helper for other tests is a test helper, and an exported or published helper may have users in other repositories: keep it and report it.
10. **A tool's unused list is where to look, not what to delete.** Every detector has blind spots, and they are exactly the cases above.

## Reached by name at run time

- Reflection and dynamic lookup: `getattr`, `importlib`, `globals()`, `Class.forName`, `Type.GetType`, `Activator.CreateInstance`, `Reflect`, `send` and `const_get` in Ruby, `require()` or `import()` with a computed path.
- Dependency injection and registries: Spring components and beans, .NET service registration and assembly scanning, Angular and NestJS providers, Guice and Dagger modules, service loaders (`META-INF/services`), Python entry points.
- Names in configuration: handler and class names in YAML, JSON, TOML, or XML; route tables; serverless function definitions; job and queue names; logging and ORM configuration.
- Serialization: fields, getters, setters, and constructors used by JSON, XML, ORM, or protobuf mapping; enum variants that appear in stored data or on the wire.
- Templates and markup: functions, filters, components, and properties named in HTML, JSX, Vue, Svelte, Jinja, Razor, or similar templates; CSS classes built by concatenation.
- Message and event names: handlers bound by a string topic, command, or event name.

## Reached by convention

- Framework lifecycle and magic methods: constructors, destructors, `__dunder__` methods, `toString`, `equals`, `hashCode`, React and Vue lifecycle methods, `main`.
- Files found by path: Next.js, Nuxt, SvelteKit, and Remix routes; Rails and Django apps, models, migrations, and management commands; test files and `conftest.py`; Go `init` functions and `_test.go` files; Storybook stories.
- Tests and fixtures: pytest fixtures used by name, `setUp` and `tearDown`, parametrized cases, benchmark and fuzz functions.
- Command and plugin entry points: `bin` entries, console scripts, CLI subcommands registered by decorator, editor and build plugins.
- Foreign interfaces: `extern "C"`, `#[no_mangle]`, `//export`, JNI methods, P/Invoke, WebAssembly exports, symbols a linker script names.
- Macros and code generation: code used only by a macro expansion, a derive, or generated code; and the generator's own input.

## Reached from outside the repository

- Published packages: everything reachable from the package's exports, the type declarations, and the documented API.
- Other repositories, services, and scripts in the organization.
- CI workflows, deployment and infrastructure definitions, container entry points, package scripts, makefiles, and git hooks.
- HTTP routes, RPC methods, GraphQL fields, webhooks, and command-line flags that users call.
- Database objects used by reports, other applications, or an operator.

## Reached only sometimes

- Platform and architecture: `#ifdef`, `cfg(...)`, build tags, `process.platform`, `Platform.OS`, file suffixes such as `_windows.go` or `.ios.tsx`.
- Build modes and features: debug versus release, Cargo features, compile-time flags, tree-shaken entry points, server versus client bundles.
- Feature flags: the code's default is not the flag's state. Check the flag service or configuration, in every environment.
- Scheduled and manual work: cron jobs, end-of-month and end-of-year jobs, backfills, data repair scripts, incident tools.
- Error and recovery paths: retries, fallbacks, rollbacks, handlers for errors that are rare but possible.
- Old data: migrations, upgraders, and parsers for formats that existing files, rows, or clients still use.

## What the detectors miss

Use the repository's own tool if it has one, and treat its output as candidates.

| Ecosystem | Tools | Common blind spots |
| --- | --- | --- |
| JavaScript, TypeScript | knip, ts-prune, ESLint `no-unused-vars`, TypeScript `noUnusedLocals` | dynamic imports, files found by path, names in templates and configuration |
| Python | vulture, Ruff `F401` and `F841` | `getattr`, decorators that register, `__all__`, framework hooks, fixtures |
| Go | `deadcode` (golang.org/x/tools), staticcheck `U1000` | `//go:linkname`, build tags and platforms other than the one analysed (run it once per configuration), a method needed only to satisfy an interface |
| Rust | the `dead_code` and `unused` lints, Clippy | other features and targets, macros, `pub` items in a library |
| Java, Kotlin | IDE inspections, ProGuard and R8 reports | reflection, dependency injection, serialization, service loaders |
| C# | IDE analyzer `IDE0051`, compiler warning `CS0169` | reflection, dependency injection, XAML bindings, serialization |

A project that already silences a detector for a symbol, with a keep rule, an allowlist entry, or a suppression, is telling you that symbol is reached in a way the tool cannot see.
