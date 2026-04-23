# paket-dependency-groups

## Probe metadata

| Field | Value |
|---|---|
| Pattern | paket-dependency-groups |
| Target framework | net8.0 |
| NuGet source | https://api.nuget.org/v3/index.json |
| Storage mode | none |
| Dependency groups | Main (default), Build, Test |
| Generated | 2026-04-22 |

## Feature exercised

This probe exercises Paket's dependency group feature. Three named groups are
declared in `paket.dependencies`: the implicit Main group (no `group` keyword),
an explicit `group Build`, and an explicit `group Test`. The `paket.lock` uses
`GROUP <name>` separators to partition packages by group. This validates that
Mend's UA engine correctly parses all GROUP sections in `paket.lock` and
detects dependencies from ALL groups, not just the default Main group.

## Expected dependency tree

### Main group (default, no group keyword)

| Package | Resolved version | Has transitives |
|---|---|---|
| Newtonsoft.Json | 13.0.3 | No |
| Serilog | 3.1.1 | Yes |

Transitives from Main:

| Package | Resolved version | Required by |
|---|---|---|
| Serilog.Sinks.Console | 4.1.0 | Serilog |

### group Build

| Package | Resolved version | Has transitives |
|---|---|---|
| Fake.Core.Target | 6.0.0 | Yes |

Transitives from Build:

| Package | Resolved version | Required by |
|---|---|---|
| Fake.Core.Process | 6.0.0 | Fake.Core.Target |
| Fake.Core.Trace | 6.0.0 | Fake.Core.Process |
| FSharp.Core | 6.0.7 | Fake.Core.Target, Fake.Core.Process, Fake.Core.Trace |

### group Test

| Package | Resolved version | Has transitives |
|---|---|---|
| NUnit | 3.14.0 | No |
| NUnit3TestAdapter | 4.5.0 | No |

### Detection expectations

- Mend must detect **10 unique packages** in total across all three groups.
- The `GROUP Build` and `GROUP Test` sections must not be silently skipped.
- A known failure mode is that Mend reads only the first (Main) lockfile
  section and stops at the first `GROUP` keyword, yielding only 3 packages
  instead of 10. This probe targets that specific parser gap.
- `storage: none` must not prevent detection — Mend reads `paket.lock`, not
  the local packages folder.
- All packages from all groups should appear in the Mend dependency tree.

## File structure

```
paket-dependency-groups/
├── paket.dependencies       # Three groups: Main, Build, Test
├── paket.lock               # GROUP separators for each named group
├── expected-tree.json       # Expected Mend dependency tree (probe format)
├── README.md                # This file
└── src/
    └── MyProject/
        ├── MyProject.csproj # SDK-style net8.0 project
        └── paket.references # References with group sections
```
