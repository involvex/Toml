# Toml — Agent Instructions

PowerShell module for reading and writing [TOML](https://toml.io) data, with full TOML 1.0.0 specification support. Pure PowerShell parser and serializer — no external TOML runtime dependency.

## Project

- Module name: `Toml`
- Entry point: `src/` → built into `./output/Toml/Toml.psm1` + `Toml.psd1` via `build.ps1`
- Public commands: `ConvertFrom-Toml`, `ConvertTo-Toml`, `Import-Toml`, `Export-Toml`, `Format-Toml`, `Test-Toml`, `Merge-Toml`
- Tests: `tests/Toml.Tests.ps1` (Pester 6)
- Lint: PSScriptAnalyzer over `./src`
- Published to: PowerShell Gallery (`Install-PSResource -Name Toml`)
- Docs: https://psmodule.io/Toml
- Issues: https://github.com/PSModule/Toml/issues

Note: `CLAUDE.md`, `GEMINI.md`, and `.github/copilot-instructions.md` do not exist in this repository; this file was authored from `README.md`, `CONTRIBUTING.md`, `build.ps1`, and the repository layout.

## Useful Commands

All commands run in PowerShell 7+. Use `pwsh`, not Windows PowerShell 5.1.

### Build the module locally

```powershell
pwsh -File .\build.ps1
```

Output goes to `./output/Toml/`. Optional parameters:

```powershell
pwsh -File .\build.ps1 -Version 1.2.3 -Prerelease preview
```

### Install / import the module

From the PowerShell Gallery:

```powershell
Install-PSResource -Name Toml
Import-Module -Name Toml
```

From a local build:

```powershell
Import-Module .\output\Toml\Toml.psd1 -Force
```

### Run the test suite

```powershell
Invoke-Pester -Path .\tests\Toml.Tests.ps1
```

Requires Pester 6.x (`#Requires -Modules @{ ModuleName = 'Pester'; ModuleVersion = '6.0.0'; MaximumVersion = '6.*' }`).

### Lint the source

```powershell
Invoke-ScriptAnalyzer -Path .\src -Recurse -Severity Error,Warning
```

### Common usage examples

Parse a TOML string:

```powershell
$doc = ConvertFrom-Toml -InputObject @'
[database]
host = "localhost"
port = 5432
'@
$doc.Data.database.host   # "localhost"
```

Import from a file:

```powershell
$doc = Import-Toml -Path './config.toml'
$doc.FilePath             # absolute path
$doc.Data                 # OrderedDictionary
```

Serialize to TOML:

```powershell
$toml = ConvertTo-Toml -InputObject ([ordered]@{ title = 'App'; version = 1 })
```

Export to a file:

```powershell
Export-Toml -InputObject $config -Path './output.toml'
```

Normalize TOML text:

```powershell
Format-Toml -Path 'Cargo.toml' -Indent 4
```

Validate without throwing:

```powershell
Test-Toml -InputObject $tomlText
```

Merge two documents:

```powershell
Merge-Toml -BaseObject $defaults -OverrideObject $overrides -Strategy LastWins
```

Per-command help:

```powershell
Get-Help ConvertFrom-Toml -Examples
Get-Help Import-Toml -Examples
```

## Technologies

| Layer | Technology | Version / notes |
|---|---|---|
| Runtime | PowerShell | 7.6 LTS required (build.ps1 requires ≥ 7.0) |
| Tests | Pester | 6.x |
| Lint | PSScriptAnalyzer | Error and Warning severity |
| Build/publish framework | PSModule | https://github.com/PSModule/Process-PSModule |
| Parser/serializer | Pure PowerShell | No external TOML runtime dependency |
| Package format | PowerShell module | `.psm1` + `.psd1`, published to PowerShell Gallery |
| Specification | TOML | 1.0.0 |
| File encoding | UTF-8 without BOM | For all files written by the module |
| Line endings | CRLF by default | `.gitattributes`: `* text eol=autocrlf` |

## Best Practices

- **PowerShell version**: target PowerShell 7.6 LTS. Use `pwsh` for all local commands.
- *)odule build**: always build through `.uild.ps1` before running tests locally; do not edit files under `output/`.
- **Imports**: import the local manifest with `-Force` so a rebuilt module is reloaded:
  `Import-Module .\output\Toml\Toml.psd1 -Force`.
- **Hashtables**: use `[ordered]` hashtables where key order matters (tables and inline tables).
- **Comment-based help**: add comment-based help to every public function; `Get-Help <Command> -Examples` should work.
- **Tests**: write Pester tests for public commands and parser edge cases in `tests/Toml.Tests.ps1`; place shared fixtures in `tests/BeforeAll.ps1a.
- **Lint**: keep `Invoke-ScriptAnalyzer -Path ./src -Recurse -Severity Error,Warning` clean (no errors or warnings).
- **Type mapping** (TOML → PowerShell): String → `[string]`, Integer → `[long]`, Float → `[double]`, Boolean → `[bool]`, Offset date-time → `[System.DateTimeOffset]`, Local date-time/date → `[System.DateTime]`, Local time → `[System.TimeSpan]`, Array → `[object[]]`, Table → `[ordered]` hashtable, Array of tables → `[object[]]` of hashtables.
- **Spec compliance**: reject duplicate keys and table redefinitions per TOML 1.0.0.
- **Encoding**: write files as UTF-8 without BOM; respect `.gitattributes` (CRLF default, LF for `*.sh`, `*.svg`).

## Guidelines

### Contributing (from CONTRIBUTING.md)

1. Open an issue for bugs, feature requests, or unclear documentation (include PowerShell version via `$PSVersionTable`, a minimal reproducing TOML snippet, expected vs. actual behavior).
2. Fork the repo and create a feature branch from `main`.
3. Make focused, well-scoped changes.
4. Add or update tests in `tests/` for any changed behavior.
5. Run the local build and test suite:

   ```powershell
   pwsh -File .\build.ps1
   Import-Module .\output\Toml\Toml.psd1 -Force
   Invoke-Pester -Path .\tests\Toml.Tests.ps1
   ```

6. Ensure PSScriptAnalyzer reports no errors or warnings:

   ```powershell
   Invoke-ScriptAnalyzer -Path .\src -Recurse -Severity Error,Warning
   ```

7. Open a pull request with a clear description of the change and the problem it solves.

### Source layout (`src/`)

- `classes/public/` — public classes (e.g. `TomlDocument`)
- `enums/` — enums exported as type accelerators (e.g. `TomlValueKind`)
- `functions/private/` — private helper functions
- `functions/public/` — one file per public command, exported via `Export-ModuleMember`
- `header.ps1` / `finally.ps1` — optional build header/footer

Follow this structure when adding code; `build.ps1` assembles the `.psm1` in the order: header → init → enums → classes → private functions → public functions → finally.

### Windows / shell compatibility

This repo's global guidance requires PowerShell-compatible commands: use `dir`/`Get-ChildItem`, `Remove-Item`, `Copy-Item`, `Move-Item`, `Get-Content`/`cat`, `Set-Content`, `Select-String` (or `rg`). Never use `rm -rff, `cp`, `mv`, or interactive editors/pagers in non-interactive shells.

### Do not

- Do not edit generated files under `output/`.
- Do not commit `testResults.xml` or `output/` (ignored in `.gitignore`).
- Do not use npm/yarn/pnpm; this is a PowerShell module with no Node.js toolchain.


