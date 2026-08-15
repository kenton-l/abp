# AGENTS.md

## Cursor Cloud specific instructions

This repository is the **ABP Framework source** (not an ABP-based app): a large polyglot
monorepo. The **.NET framework is the primary product**; the Angular workspace under
`npm/ng-packs` is a secondary UI component.

### Toolchain (already provided by the environment)
- **.NET SDK 10.0.100** (pinned in `global.json`) is installed at `/usr/share/dotnet`
  and symlinked to `/usr/local/bin/dotnet`.
- **PowerShell 7 (`pwsh`)** is installed as a dotnet global tool at `~/.dotnet/tools`
  (already on `PATH` via `~/.bashrc`). It is only needed for the `build/*.ps1` scripts;
  you can also drive builds/tests with `dotnet` directly.
- Node 22 + Yarn 1 are available for the Angular workspace.

### Building & testing the .NET framework (the main product)
- Canonical CI flow (see `.github/workflows/build-and-test.yml`) runs from `build/`:
  `pwsh ./build-all.ps1` then `pwsh ./test-all.ps1` over the core solution set listed
  in `build/common.ps1` (framework + core modules). Add `-f` for the full set. This is
  heavy (100+ projects).
- For focused work, target a single `.slnx` or `.csproj` directly, e.g.
  `dotnet test framework/test/Volo.Abp.Core.Tests/Volo.Abp.Core.Tests.csproj`.
- Solution files use the new **`.slnx`** XML format (there are no classic `.sln`/`.slnf`).
- Central Package Management: package versions live in `Directory.Packages.props`; the
  ABP version is in `common.props`.

### Test infrastructure — no external services needed
- **EF Core tests** use **in-memory SQLite** (created/migrated in-process). No DB server.
- **MongoDB tests** use **MongoSandbox**, which self-hosts an embedded `mongod` per run
  (downloaded via NuGet). No external MongoDB server; it does need to exec the bundled
  binary, which works in this environment.
- Redis-related tests only assert DI registration; no live Redis is required.
- Non-obvious: the first `dotnet test` restores + builds hundreds of transitive projects,
  so an initial EF Core/MongoDB test run can take a couple of minutes even for one project.

### Running a sample application
- There is no single deployable app. The quickest runnable host that exercises core
  framework functionality (module system + Autofac DI + configuration) is
  `framework/test/SimpleConsoleDemo` (prints `Hello ABP!`).
- Gotcha: run it from its **output directory** so it can find `appsettings.json`
  (`dotnet run` uses the shell CWD, not the project dir, so config lookup fails from
  the repo root). Run the built dll instead:
  `cd framework/test/SimpleConsoleDemo/bin/Debug/net10.0 && dotnet SimpleConsoleDemo.dll`.
- Full template host apps under `templates/app/aspnet-core` require a real relational DB
  (run `*.DbMigrator` first) and, for tiered setups, an auth server — not needed for the
  core test/build workflow.

### Angular workspace (`npm/ng-packs`) — known caveat
- This is an Nx 22 / Angular 21 workspace. A fresh `yarn install` currently fails under
  Yarn classic (v1) with `Invariant Violation: could not find a copy of vite to link ...
  node_modules/vitest/node_modules` (vitest 4 / vite 7 hoisting), and no `yarn.lock` is
  committed. Treat the Angular packages as an **optional** component; the .NET framework
  build/test does not depend on them.
