# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Educational course platform for "Python Programming - Semester 1" (Ukrainian language). This is a static documentation site built with MkDocs Material, not a Python application.

## Commands

```bash
# Setup
python3 -m venv env
source env/bin/activate
pip3 install -r requirements.txt

# Serve locally with live reload
mkdocs serve --livereload
```

## Architecture

- **docs/** — Course content in Markdown (Ukrainian). Organized by modules with paired lecture/practice files.
- **mkdocs.yml** — Site configuration, navigation, theme settings, and markdown extensions.
- **overrides/main.html** — Extends base template to add a footer with GitHub issue link.
- **requirements.txt** — MkDocs and related dependencies (pinned versions).

### Content naming convention

Files follow `NN-<topic>-<type>.md` pattern (e.g., `01-variables-lecture.md`, `02-variables-practice.md`). New content must also be added to the `nav` section in `mkdocs.yml`.

## Key Details

- All documentation is in **Ukrainian**. Maintain this when editing content.
- Practice assignments build incrementally within each module.
- Site deploys to GitHub Pages at `https://kurotych.com/ua/courses/programming-1sem/` (built by the `kurotych.github.io` repo, which includes this repo as a git submodule).
- Markdown extensions: `admonition`, `pymdownx.details`, `pymdownx.highlight`, `pymdownx.superfences` (with **mermaid** diagram support), `pymdownx.snippets`, `pymdownx.inlinehilite`, `toc` with permalinks.
- Do not create guides for windows
- In code samples, Cyrillic is allowed **only inside comments**. Everything else — string literals, `print()` arguments, log messages, variable/function names, terminal output blocks, error messages — must be Latin-only. This applies to any text that would appear in stdout/stderr, over the network, or as code identifiers.
- Python examples always do complete. They must work after copying into empty (*.py) document
