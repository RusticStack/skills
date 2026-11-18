# Dependencies that are used without an import

Read this before judging any dependency unused. The recognition rules come first, because they cover the tool this file does not list.

Keeping an unused dependency costs install time and an audit entry. Removing one that was loaded by name, needed in production, or required on another platform breaks a build or a deployment, sometimes weeks later. When unsure, keep it and report it.

## Recognition rules

1. **Configuration is a caller.** Tools load packages named in their configuration: plugins, presets, loaders, transformers, parsers, reporters, formatters. Read every configuration file, including the ones inside `package.json`, `pyproject.toml`, and `Cargo.toml`.
2. **A command is a use.** A package can exist only for the executable it installs. Search scripts, makefiles, hooks, CI, and container files for every command name it provides.
3. **The package name is not the import name.** `pyyaml` is imported as `yaml`, `beautifulsoup4` as `bs4`, `Pillow` as `PIL`; a scoped npm package may be imported by a subpath; a Rust crate with hyphens is used with underscores, or renamed in the manifest.
4. **Something else may require it to be present.** Peer dependencies, optional backends, database drivers, and plugins are chosen at run time by the package that needs them.
5. **The install you ran is one of several.** Development, production, each platform, each runtime version, each feature set, and each workspace member can need different packages.
6. **An import can be for its effect.** A polyfill, an instrumentation or tracing package, a type augmentation, a registration, or a stylesheet is imported and never referenced again.
7. **Build time and run time differ.** A package absent from the build may be loaded when the program starts, and a package used only by the build may be absent at run time by design.
8. **A pin has a reason.** An override, a resolution, a patch, an exact version, or a duplicate entry usually marks a security fix or a known breakage, even when nobody wrote down which.
9. **Published packages have consumers.** A dependency of a library is part of what its users install. Moving it between lists changes what they get.
10. **A tool's unused list is where to look, not what to remove.** Every detector misses the cases above.

## Loaded by name from configuration

- JavaScript and TypeScript: ESLint plugins, configs, and parsers; Babel presets and plugins; PostCSS, Tailwind, and Stylelint plugins; webpack loaders, Vite and Rollup plugins; Jest and Vitest environments, transformers, and reporters; Prettier plugins; TypeScript `types`, `plugins`, and `extends`; commitlint and semantic-release configs.
- Python: pytest plugins (loaded by entry point with no import at all), Sphinx extensions and themes, mypy plugins, Django `INSTALLED_APPS` and middleware, Celery and SQLAlchemy drivers named in a URL, build backends in `[build-system]`.
- Rust: build dependencies used in `build.rs`, proc-macro crates, crates enabled only by a feature, `[patch]` and `[replace]` entries.
- Go: tools tracked in a `tools.go` file or the `tool` directive, packages imported with `_` for registration, such as database drivers.
- Java and Kotlin: annotation processors, Gradle and Maven plugins, JDBC drivers and logging backends found on the classpath, service-loader implementations.
- .NET: analyzers, source generators, MSBuild SDKs and tasks, providers registered by name.
- Ruby and PHP: gems required by Bundler groups, Rails engines, Composer plugins, and packages found by autoload or discovery.

## Used as a command

Search for the executable's name, which often differs from the package's:

- Package scripts and their pre and post hooks, `npx`, `pnpm exec`, `bunx`, `uv run`, `poetry run`, `cargo` subcommands, `go run` and `go tool`.
- Makefiles, task runners, git hooks (`husky`, `lefthook`, `pre-commit`), and editor tasks.
- CI workflows, container files, deployment scripts, and infrastructure definitions.
- Documentation that tells contributors to run it.

## Required to be present

- Peer dependencies of something you keep. Read them from the package's own `peerDependencies` or from `npm ls` and `pnpm why`; recent npm and pnpm install peers without a warning.
- Optional backends: database drivers, cache and queue clients, image, compression, and crypto libraries chosen by a setting or by what is installed.
- Type packages and stubs that another package's types refer to.
- Native add-ons, platform-specific binaries, and their build tools.
- Runtime shims: polyfills, `tslib`, `regenerator-runtime`, `core-js`, helpers that compiled output imports and source does not.

## Needed somewhere else

- Production only: process managers, servers, drivers, and monitoring agents started by a deployment and never by tests.
- Another platform or architecture: optional dependencies, environment markers (`sys_platform`, `python_version`), `cfg(target_os)` dependencies, build constraints.
- Another runtime version: backports and compatibility packages the oldest supported version still needs.
- Another feature set: Python extras, Cargo features, optional peer features.
- Another workspace member, or the root on behalf of all members.

## Pins, overrides, and patches

`overrides`, `resolutions`, `pnpm.overrides`, `patchedDependencies`, patch files, `[patch]` in Cargo, `replace` and `exclude` in `go.mod`, constraints files, and exact pins. Look for the reason in a nearby comment, the commit that added it, a linked advisory or issue. Remove one only when the reason is gone: the fixed version is now the minimum everything asks for, or the package it patched has left. Otherwise report it.

## What the detectors miss

Use the repository's own tool if it has one, and treat its output as candidates.

| Ecosystem | Tools | Common blind spots |
| --- | --- | --- |
| JavaScript, TypeScript | knip, depcheck | plugins named in configuration, commands, peer dependencies, runtime shims |
| Python | deptry, pip-check-reqs | import names that differ, plugins loaded by entry point, extras, markers |
| Rust | cargo-machete, cargo-udeps | features and targets not built, renamed crates, build and proc-macro crates |
| Go | `go mod why -m`, `go mod tidy` | `tidy` rewrites `go.mod` and `go.sum`, so it is an edit, not a report; it counts imports under every build tag and platform at once |
| Java, Kotlin | `mvn dependency:analyze`, Gradle dependency-analysis plugin | reflection, runtime-only drivers, annotation processors |
| .NET | IDE and SDK unused-reference analysis | analyzers, generators, runtime-loaded providers |

A project that already lists a dependency in a detector's ignore configuration is telling you it is used in a way the tool cannot see.
