# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Role
The repository is a job‑search assistant framework. Claude acts as a career advisor and application assistant for the user, automatically:

1.  Evaluates job postings against the candidate profile.
2.  Generates a tailored CV and cover letter in LaTeX.
3.  Lints and adapts skills, templates and commands.
4.  Prepares interview talking points.
5.  Advises on career strategy.

## Candidate Profile
The candidate profile is stored in this file in the placeholder form shown in the original repo. The `/setup` workflow replaces the placeholders with the user's details.

## Repository Structure (high‑level view)
- **/cv/** – modern‑cv LaTeX templates.
- **/cover_letters/** – custom `cover.cls` and cover‑letter templates.
- **/.claude/commands/** – user‑invokable command handlers (`/apply`, `/setup`, `/scrape`, `/rank`, …).
- **/.claude/skills/** – reusable skills that implement the core logic (job‑evaluation, PDF verification, ATS parsing, etc.).
- **/.agents/skills/** – behind‑the‑scenes job‑portal CLIs (e.g. `jobbank-search`).
- **/tools/** – helper scripts for CI, PDF verification, salary lookup, etc.
- **/tests/** – unit and integration tests for the command and skill code.

## Development Commands
The repo contains a CI pipeline, but the following commands can be run locally for debugging or regression testing:

| Command | Purpose | Notes |
|---------|---------|-------|
| `bun install` in each folder under `.agents/skills/**/cli` | Install TypeScript dependencies for the job‑portal CLIs. | The CLIs are lightweight; running `bun install` is sufficient. |
| `npm run lint` in the repository root | Lint all TypeScript and Python files with the repo‑specific eslint and flake8 rules. | Aligns with `tests/test_lint_skills.py`. |
| `python -m pytest` | Run the full test suite, including unit tests for skills and CLI behaviours. | Tests use `pytest` and are straightforward to run. |
| `python -m pytest tests/test_*.py` | Run a single test file. | For quick debugging of a specific module. |
| `python tools/verify_pdf.py <file>.pdf --dump-text <file>.txt` | Verify a compiled PDF’s page count and extract the text layer for later ATS checks. | This is used by `/apply` during the verification step. |

## Workflow Commands
| Command | Typical flow | Example output |
|---------|-------------|----------------|
| `/setup` | Guides the user through importing documents or typing profile data, writes the placeholder fields in this file and the skill files. | Generates `CLAUDE.md`, `01-candidate-profile.md`, etc. |
| `/scrape` | Runs all installed job‑portal CLIs, aggregates results and deduplicates them. Output is a markdown table of matching postings. | See `tests/test_scrape_provenance.py` for expected format. |
| `/apply <URL-or-text>` | 1. Parses the posting<br>2. Scores fit using the **job‑evaluation** skill<br>3. Drafts a CV/letter<br>4. Spawns a reviewer agent<br>5. Revises and compiles PDFs<br>6. Runs ATS and PDF checks | Final PDF paths in `cv/` and `cover_letters/`. |
| `/rank` | Scores a bulk list of posting URLs produced by `/scrape` and returns a ranked shortlist. | Includes fit score and key gaps. |
| `/interview <application-id>` | Builds a prep pack for an upcoming interview. | Generates STAR examples and mock Q&A. |
| `/outcome` | Records the result of a submitted application into `documents/applications/<company>_<role>/outcome.md`. | Adds interview feedback or offer details. |
|
## Extending the Framework
- **Adding a new portal skill** – Create a folder in `.agents/skills/`, copy an existing CLI structure, update the `SKILL.md` manifest, and run `bun install`. Register with `/add-portal` to integrate.
- **Adding a custom CV or cover‑letter template** – Place the template files under `templates/` and run `/add-template` to register. The command records the compile command (e.g. `xelatex %i`) and activates the template for `/apply`.
- **Adding new commands** – Write a handler in `.claude/commands/` and add a dealer entry in `.claude/settings.json`. Tests in `tests/` should be added to ensure behaviour.

## Running Tests in CI
The CI workflow in `.github/workflows/ci.yml` runs the steps:

1.  `bun install` for the job‑portal CLIs.
2.  `npm run lint` for lint‑checks.
3.  `python -m pytest` for the Python tests.
4.  `./check_framework_version.py` to ensure method‑level consistency.
5.  Smoke‑test LaTeX compilation for the stock templates.

For local debugging you can mimic the CI by running the sequence of commands above.

## Security & Integrity
All commands run without side‑effects beyond updating tracked files. No secrets are touched unless stored in a per‑fork `.claude/settings.json`.          
