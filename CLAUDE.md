<!-- -*- fill-column: 80 -*- -->
<div style="width: 768px; margin-left: 28px;">
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is Combobulate?

Combobulate is an Emacs minor mode for structured editing and navigation using Emacs 29's built-in tree-sitter library. It provides language-aware movement and code manipulation commands that work consistently across all supported languages.

## Build & Test Commands

```bash
# Byte-compile all .el files
make byte-compile

# Run the full test suite (byte-compiles first)
make run-tests

# Build test harnesses from fixture definitions (run after modifying fixtures)
make build-tests

# Clean compiled files and generated test harnesses
make clean-tests

# Rebuild combobulate-rules.el from tree-sitter grammar JSONs
make rebuild-relationships

# Download latest grammars and rebuild rules
make download-relationships

# Run tests in Docker (for reproducible environment)
make docker-run-tests
```

The test runner command is:
```
emacs --batch --no-init-file --chdir ./tests/ -L .. -L . -l tests/.ts-setup.el -l ert <test-files> -f ert-run-tests-batch-and-exit
```

## Architecture

### Core Subsystems

**`combobulate.el`** — Entry point. Requires all subsystems and activates `combobulate-mode`.

**`combobulate-procedure.el`** — The heart of the system. Defines a declarative DSL for selecting AST nodes based on type, position, parent/ancestor constraints, and fields. All navigation and editing commands ultimately use procedures to determine which nodes to operate on.

**`combobulate-rules.el`** (303 KB, auto-generated) — Grammar relationship tables mapping every tree-sitter node type to its valid children and fields for each supported language. **Never edit by hand** — regenerate with `make rebuild-relationships` from `build/build-relationships.py`.

**`combobulate-navigation.el`** — Movement commands: up/down hierarchy (`C-M-u`/`C-M-d`), sibling navigation (`C-M-n`/`C-M-p`), defun navigation.

**`combobulate-manipulation.el`** — Code editing: drag/reorder nodes, clone, kill, splice (remove parent keeping children), indentation handling.

**`combobulate-envelope.el`** — Code templating and the "carousel" interface: an interactive preview system for complex multi-step edits.

**`combobulate-cursor.el`** — Multiple cursor and field editor: places cursors at syntactically significant positions, sequence editing for HTML attributes, function arguments, etc.

**`combobulate-query.el`** — Interactive tree-sitter S-expression query builder with syntax highlighting, xref integration, and query history.

**`combobulate-setup.el`** — Language registration and keymap setup. Wires procedures to commands for each language mode.

**`combobulate-interface.el`** — Abstraction layer over `treesit-*` functions. Use these `combobulate-*` wrappers rather than calling treesit directly.

### Language Modules

Each language has its own file (`combobulate-python.el`, `combobulate-js-ts.el`, etc.) that defines:

- **Procedures** for each operation type: `procedures-sibling`, `procedures-hierarchy`, `procedures-edit`, `procedures-sexp`, `procedures-defun`
- **Envelope templates** for code generation
- **Pretty-print functions** for node name display
- Language-specific customization variables

To add a new language: create `combobulate-<lang>.el`, define the procedure sets, and register it in `combobulate-setup.el`.

### Test Infrastructure

Tests use ERT (Emacs Regression Testing) with a fixture-based approach:

- Raw fixtures live in `tests/fixtures/` as code snippets with before/after states
- `make build-tests` runs `generate-harnesses.el` to produce `tests/test-*.gen.el` files
- **Never edit `.gen.el` files** — they are regenerated from fixtures
- `tests/.ts-setup.el` configures the test environment and loads tree-sitter grammars

### Key Design Patterns

1. **Procedures are declarative**: Rather than imperative tree traversal, procedures describe constraints on what nodes to find. A procedure says "find a sibling node of type X where the cursor is positioned before it" — the engine evaluates this.

2. **Language parity**: The same user-facing commands work across all languages by dispatching through the language's registered procedures.

3. **Carousel for complex edits**: Multi-step transformations (e.g., changing a for-loop to a list comprehension) use the carousel interface to preview and accept changes incrementally.

4. **Generated rules**: `combobulate-rules.el` is derived from tree-sitter grammar JSON files via `build/build-relationships.py`. This keeps the rules accurate as grammars evolve.
</div>
