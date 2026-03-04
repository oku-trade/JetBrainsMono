# AGENTS.md - JetBrains Mono Font Project

## Project Overview

JetBrains Mono is an open-source typeface designed for developers. This is a **font project**,
not a typical software project. The codebase consists of font source files (Glyphs format),
Python/Kotlin build scripts, shell scripts, and CI configuration. Fonts are licensed under
SIL Open Font License 1.1 (OFL.txt); source code under Apache 2.0.

## Repository Structure

```
sources/                  # Font source files (.glyphs) and build config
  config.yaml             # gftools builder configuration (weights, axes, STAT tables)
  JetBrainsMono.glyphs    # Upright masters (Glyphs 3 format)
  JetBrainsMono-Italic.glyphs  # Italic masters (Glyphs 3 format)
  old-build-scripts/      # Legacy shell-based build scripts (deprecated)
scripts/                  # Build/utility scripts
  generate_variable_webfonts.py  # Converts variable TTF to WOFF2
  GenerateNLVersion.kt    # Generates No-Ligature (NL) variant from TTF
fonts/                    # Built font output (committed to repo)
  otf/                    # Static OTF files (16 styles)
  ttf/                    # Static TTF files (includes NL variants)
  variable/               # Variable TTF files (upright + italic)
  webfonts/               # WOFF2 files (static + variable)
  archives/               # Archived older font versions
.github/workflows/        # CI configuration
install_manual.sh         # Linux font installer script
```

## Build Commands

### Prerequisites

- Python >= 3.9.5
- pip packages: `gftools`, `fonttools[woff]` (see requirements.txt)
- For NL generation: Kotlin runtime + `ttx` command (from fonttools)

### Install Dependencies

```bash
pip install -r requirements.txt
```

### Full Font Build

This is the primary build command. It compiles .glyphs sources into TTF, OTF,
and variable fonts, outputting to the `fonts/` directory:

```bash
gftools builder sources/config.yaml
```

### Generate Variable WOFF2 Webfonts

Run after the main build to produce variable WOFF2 files:

```bash
python scripts/generate_variable_webfonts.py
```

### Complete Build (as CI does it)

```bash
gftools builder sources/config.yaml && python scripts/generate_variable_webfonts.py
```

### Generate No-Ligature (NL) Version

Requires `ttx` (from fonttools) and Kotlin:

```bash
kotlinc -script scripts/GenerateNLVersion.kt
```

### Legacy Build Scripts (deprecated, for reference)

Located in `sources/old-build-scripts/`. These use `fontmake` directly:

```bash
# Static fonts:   sources/old-build-scripts/build-statics.sh
# Variable fonts:  sources/old-build-scripts/build-vf.sh
# Webfonts:        sources/old-build-scripts/build-webfonts.sh
```

## Testing / Validation

There is no automated test suite. Validation is done via:

- **fontbakery**: Google Fonts quality assurance checks (run manually)
  ```bash
  fontbakery check-universal fonts/ttf/*.ttf
  fontbakery check-universal fonts/otf/*.otf
  ```
- **Visual inspection**: Check rendered output at various sizes
- **CI build**: The GitHub Actions workflow (`.github/workflows/build-fonts.yml`)
  builds fonts on push/PR to `master` when `sources/**` changes

## Code Style Guidelines

### Python Scripts (`scripts/generate_variable_webfonts.py`)

- Use **tabs** for indentation (existing convention in this project)
- Standard library imports first, then third-party (`fontTools`)
- Keep scripts minimal and focused on a single task
- Use `print()` with descriptive `INFO:` prefix for progress output
- Use `glob.iglob()` for file pattern matching
- Use `os.path` functions for path manipulation

### Kotlin Scripts (`scripts/GenerateNLVersion.kt`)

- Standard Kotlin style: 4-space indentation in most places
- Group imports: `org.w3c.dom.*` first, then `java.*`, then `javax.*`
- Use Kotlin idioms: extension functions, `forEach`, `?.` safe calls
- Document the script purpose and requirements in a top-level `/** */` comment
- Include `@author` tag for attribution
- Utility/extension functions go after main logic, separated by a comment banner
- Use `fun main()` as entry point (script-style Kotlin)

### Shell Scripts (`install_manual.sh`, old build scripts)

- Use `#!/usr/bin/env bash` or `#!/bin/sh` shebang
- Mark constants with `readonly`
- Use `set -e` for error handling in build scripts
- Quote all variable expansions: `"${variable}"`
- Use functions for logical grouping; name them descriptively
- Print progress with `echo` statements describing each phase
- Use `die()` pattern for fatal errors (print to STDERR, exit 1)

### YAML Configuration (`sources/config.yaml`)

- 2-space indentation
- Font axis order: `wght` first, then `ital`
- Weight values follow standard CSS/OpenType scale: 100-800
  (Thin=100, ExtraLight=200, Light=300, Regular=400, Medium=500,
   SemiBold=600, Bold=700, ExtraBold=800)

### General Conventions

- **Naming**: Font files use PascalCase: `JetBrainsMono-{Weight}{Italic}.{ext}`
  - NL variant: `JetBrainsMonoNL-{Weight}{Italic}.{ext}`
  - Variable: `JetBrainsMono[wght].ttf`, `JetBrainsMono-Italic[wght].ttf`
- **Commit messages**: Descriptive, often reference specific glyphs/Unicode
  codepoints or issue numbers. Examples from history:
  - `"Added IJ ij #578"`
  - `"Fixed typo in ss20 name"`
  - `"upright: path direction of sixPointedBlackStar and notdef updated"`
- **Branch model**: `master` is the main branch
- **Built fonts are committed**: The `fonts/` directory contains generated output
  and is tracked in git. After rebuilding, commit the updated font files.

## CI/CD

GitHub Actions workflow: `.github/workflows/build-fonts.yml`
- Triggers on push/PR to `master` when `sources/**` changes
- Uses Python 3.8, pip cache
- Runs: `gftools builder sources/config.yaml` then `python scripts/generate_variable_webfonts.py`
- Uploads built fonts as artifacts

## Important Notes for Agents

1. **Source of truth**: The `.glyphs` files in `sources/` are the canonical source.
   Never manually edit files in `fonts/` -- they are generated output.
2. **Rebuild after source changes**: Any change to `.glyphs` files or `config.yaml`
   requires a full rebuild (`gftools builder sources/config.yaml`).
3. **Font binaries are committed**: Unlike most projects, built artifacts (`fonts/`)
   are checked into version control. Include them in commits after rebuilding.
4. **OpenType features**: Ligatures and stylistic sets (ss01-ss20, cv01-cv20) are
   defined within the .glyphs source files, not in separate feature files.
5. **NL variant**: "No Ligature" version strips all ligature data from TTF files.
   Generated separately via the Kotlin script after the main build.
6. **Weight axis**: The font supports 8 weights (Thin through ExtraBold) as both
   static instances and a continuous variable axis (`wght` 100-800).
7. **No test suite**: Validation is manual or via fontbakery. Always verify that
   builds complete without errors.
8. **Glyphs app required for source editing**: The `.glyphs` files are in Glyphs 3
   format. You need the Glyphs app (macOS) to edit font masters/glyphs visually.
