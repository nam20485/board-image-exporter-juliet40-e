# Tasks

This file documents repetitive tasks and their workflows for the PCB Renderer CLI project.

## Run TruffleHog Security Scan

**Last performed:** N/A - Setup phase

**Files to modify:** None (read-only operation)

**Steps:**
1. Use PowerShell script for Windows: `./scripts/security/run-trufflehog.ps1`
2. Or use Bash script for Linux/Mac: `./scripts/security/run-trufflehog.sh`
3. Review output for any detected secrets
4. Update `security/trufflehog-allowlist.yml` if false positives need exclusion

**Important notes:**
- TruffleHog runs automatically in CI on push/PR to main branch
- Pre-commit hook available via `git config core.hooksPath .githooks`
- Never commit actual secrets - use `.env` files excluded from git

---

## Memory Bank Update

**Last performed:** 2026-01-28

**Files to modify:**
- `.kilocode/rules/memory-bank/context.md` - Update current work focus and recent changes
- `.kilocode/rules/memory-bank/product.md` - Update if product scope changes
- `.kilocode/rules/memory-bank/architecture.md` - Update if architecture changes
- `.kilocode/rules/memory-bank/tech.md` - Update if technology stack changes

**Steps:**
1. Review all project files for changes since last update
2. Update `context.md` with current work focus, recent changes, and next steps
3. Update other files if significant changes occurred in product, architecture, or tech
4. Ensure consistency across all files (prioritize brief.md if conflicts)

**Important notes:**
- Must review ALL memory bank files during update
- Focus particularly on context.md as it tracks current state
- Update when: discovering new patterns, after significant changes, or when explicitly requested

---

## Add New Sample Board File

**Last performed:** N/A - Setup phase

**Files to modify:**
- `plan_docs/boards/<new_board>.json` - Create new board file
- `tests/invalid_boards/` - If creating an invalid board for testing

**Steps:**
1. Follow existing board JSON schema from `plan_docs/boards/board_alpha.json`
2. Include required fields: schemaVersion, metadata, boundary, stackup, nets
3. Add components, traces, vias as needed for test scenario
4. Place invalid boards in `tests/invalid_boards/` with descriptive name

**Important notes:**
- 26 sample boards already exist covering various scenarios
- Board files use MICRON or MILLIMETER units
- Invalid boards should test one specific error condition each

---

## Run Full Test Suite

**Last performed:** N/A - No source code yet

**Steps:**
1. Ensure virtual environment activated: `uv sync`
2. Run all tests: `uv run pytest`
3. Run with coverage: `uv run pytest --cov=pcb_renderer`
4. Run property-based tests: `uv run pytest -m hypothesis`
5. Update snapshots if needed: `uv run pytest --snapshot-update`
6. Run linting: `uv run ruff check .`
7. Run type checking: `uv run pyright`

**Important notes:**
- All checks must pass before committing
- Snapshot tests verify visual output hasn't changed unexpectedly
- Property-based tests use Hypothesis for invariant checking

---

## Implement New Validation Rule

**Last performed:** N/A - Planning phase

**Files to modify:**
- `pcb_renderer/errors.py` - Add new error code to taxonomy
- `pcb_renderer/validate.py` - Implement validation logic
- `tests/test_validate.py` - Add unit tests
- `tests/invalid_boards/` - Create test fixture if needed

**Steps:**
1. Define error code in `errors.py` with code, severity, message template
2. Add validation function in `validate.py`
3. Integrate into `validate_board()` orchestration function
4. Write unit tests covering valid and invalid cases
5. Create invalid board JSON fixture if integration test needed
6. Run test suite to verify

**Important notes:**
- 14 error codes defined in architecture (see architecture.md)
- Validation should collect all errors, not fail fast
- Include JSON path in error for precise location reporting

