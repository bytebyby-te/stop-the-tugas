# AGENTS.md — Stop The Tugas

This file provides instructions for AI coding agents working in **Stop The Tugas** (Flask + PostgreSQL on Neon + Web Push + Vercel). Explicit instructions in the user's prompt override this file.

---

## 1. Project & MVP Guardrails
- **No Login System:** Do not add user authentication, login flows, user accounts, or `USER` foreign keys to core MVP features.
- **Timezone (`WIB`):** Always use `zoneinfo.ZoneInfo("Asia/Jakarta")` for WIB date/time calculations and display; store timestamps in PostgreSQL as `TIMESTAMPTZ`. Avoid naive `datetime.now()` without explicit timezone awareness.
- **Team B (Reminders & Cron):** Support `TUGAS` (`H-7`, `H-3`, `H-1`) and `KULIAH` (`H-1 19:00 WIB`, `H-30M`), recalculate/cancel pending reminders on agenda updates/deletes, and use `claimed_at` + `FOR UPDATE SKIP LOCKED` in PostgreSQL to prevent duplicate cron processing.
- **Team C (Calendar & Agendas):** Support creating and deleting `TUGAS` and `KULIAH` agendas with `class_loc` for classes, monthly/daily WIB views, empty-date handling, and delete confirmation.
- **Team D (Materials Repository):** Enforce **CRD without Update** (no edit/update endpoints or UI for materials/metadata), allow `.pdf` only (`application/pdf` + `%PDF-` header check) up to **10 MB** (`10485760` bytes), and use persistent storage on Vercel rather than ephemeral local disk writes in production.

---

## 2. Commands & Quality Gates
Run relevant checks after modifying Python code or tests, and only mark a task complete when tests and linters pass:
- **Install dependencies:** `pip install -r requirements.txt` (or `uv sync` if using `uv`)
- **Run tests:** `pytest -v`
- **Lint & format:** `ruff check .` and `ruff format --check .`
- **Type check (if configured):** `mypy app` or `pyrefly check`

---

## 3. Python Best Practices & Code Style
- **Python Version & Typing:** Target modern Python 3.11+ with type hints on public functions, service helpers, and route handlers (`dict[str, Any]`, `list[...]`, `X | None`).
- **Flask Structure:** Keep HTTP parsing, status codes, and validation thin in `app/routes/`; place business logic (WIB countdowns, cron locking, PDF validation, database queries) in `app/services/` and `app/models/`.
- **Database Safety:** Always use parameterized queries (`%s` / bound parameters)—never interpolate user input into SQL strings. Wrap multi-step claims/writes in explicit transactions (`BEGIN` / `COMMIT` / `ROLLBACK`).
- **Error Handling & Idioms:** Follow EAFP (`try` / `except` with specific exceptions rather than bare `except:`), use `pathlib.Path` instead of `os.path`, prefer f-strings, and write concise Google-style docstrings for non-trivial public functions.
- **Testing & PR Readiness:** Because PRs into `dev` that add or modify routes, services, queries, or business rules require tests, always add or update unit tests in `tests/unit/` and/or integration tests in `tests/integration/` (using `tests/conftest.py` and `tests/fixtures/`), and run `pytest -v` and `ruff check .` before finishing.

---

## 4. Boundaries (Always Do / Ask First / Never Do)
- **Always do:**
  - Keep changes scoped to the requested issue or task.
  - Add or update corresponding `pytest` tests whenever changing logic, endpoints, or validation rules.
  - Update related documentation (`README.md`, `docs/`, `database/README.md`) when routes, schemas, or workflows change.
  - Check `.gitignore` and never stage or commit secrets.
- **Ask first:**
  - Before adding new external packages to `requirements.txt`.
  - Before modifying database schema DDL (`database/physical/schema.sql`, `migrations/`) or shared cross-team API contracts.
- **Never do:**
  - Commit `.env`, VAPID private keys, database credentials, cron secrets, or copyrighted PDFs.
  - Push directly to `main` or `dev` or modify unrelated files outside your task scope.
  - Delete or weaken existing tests just to make a failing test pass.