# 🤝 Contributing to Stop The Tugas

Welcome to **Stop The Tugas**! To keep our codebase clean, predictable, and aligned with our MVP scope on **Vercel + Flask + Neon PostgreSQL**, please follow the branching, testing, review, and development guidelines below before opening an issue or pull request (PR).

---

## 🌿 Branching Strategy & Review Rules

We use a two-tier integration workflow centered around **`dev`** and **`main`**:

- **`main` (Production / Release Branch):**
  - Represents verified, release-ready code for deployment and demos.
  - Direct pushes to `main` are prohibited.
  - **Requires Tech Lead review and approval** before merging (typically from `dev` or a release/hotfix PR).
- **`dev` (Integration & Active Development Branch):**
  - Default target branch for feature, fix, design, and documentation PRs.
  - Direct pushes to `dev` are prohibited.
  - **Requires passing CI checks (`pytest` and linting)** and **review/approval from at least one lead of each relevant team** involved in the change (e.g., **Team B**, **Team C**, **Team D**, **UI/UX**, **Flowchart/Algorithm**, and/or **Database Engineering**—whenever a PR touches shared schemas, APIs, or cross-team contracts, get sign-off from at least one lead of every affected team).
- **Feature / Workstream Branches:**
  - Always branch off the latest **`dev`** branch.
  - Suggested naming format:
    - `<track>/<short-kebab-description>`
    - Examples:
      - `team-b/web-push-cron-lock`
      - `team-c/wib-calendar-delete-sync`
      - `team-d/pdf-10mb-crd-validation`
      - `ui-ux/empty-date-and-confirm-modals`
      - `algo/reminder-wib-countdown-spec`
      - `db/neon-schema-without-users`

---

## 🧩 Team Ownership & Boundaries

Respect clear separation of concerns across workstreams:

- **Team B (Reminders & Web Push):** Push subscription registration, `WIB (Asia/Jakarta)` reminder calculation (`H-7`, `H-3`, `H-1`, `H-1 19:00 WIB`, `H-30M`), automatic cancellation/recalculation when agendas are deleted or edited, Service Worker (`sw.js`) + VAPID delivery, and anti-collision cron processing (`claimed_at` + `FOR UPDATE SKIP LOCKED`).
- **Team C (Calendar & Schedules):** Creating and deleting `TUGAS` and `KULIAH` agendas, classroom location (`class_loc`), confirmation prompts before deletion, monthly calendar date markers, daily agenda views in `WIB`, and empty-date states.
- **Team D (Materials & Notes Repository):** **CRD** workflows (**Create, Read, Delete** without **Update**), shared `Course` dropdown alignment, strict `.pdf` (`application/pdf` & PDF signature) and `<= 10 MB` validation, creator metadata, preview/download, and delete confirmation.
- **UI/UX Team:** Design system/tokens, screen references under `design/`, empty states, confirmation modals, validation feedback, and push opt-in UX.
- **Flowchart & Algorithm Team:** Step-by-step algorithms, edge-case rules, and flowcharts under `docs/` aligned with actual implementation.
- **Database Engineering:** Oracle Data Modeler artifacts and Neon PostgreSQL DDL (`database/`), `TIMESTAMPTZ` / `WIB` consistency, foreign key cascades (`ON DELETE CASCADE`), and index/locking behavior.
- **Tech Lead:** Architectural consistency across teams, scope protection, security/env hygiene, and final approval on PRs into `main`.

---

## 🎯 MVP Scope Guardrails

Before starting or reviewing a PR, verify it respects our MVP boundaries:

- **No Login System:** Do not add user authentication, login screens, user accounts, or `USER` foreign keys to core MVP flows.
- **CRD Without Update on Materials:** Do not implement edit/update endpoints or UI flows for uploaded study materials or their metadata.
- **Timezone Consistency:** Always interpret and display schedules and reminders in **`WIB (Asia/Jakarta)`** and store timestamps unambiguously (`TIMESTAMPTZ` in PostgreSQL).
- **Serverless Safety:** Never rely on ephemeral local filesystem writes in production on Vercel—use configured persistent storage for PDFs and environment variables for secrets.

---

## 🧪 Testing Requirements (`pytest` & CI)

To prevent regressions on `dev`, **any PR that adds or modifies backend routes, services, database queries, or business rules must include corresponding automated tests** under `tests/`:

- **Test Structure:**
  - `tests/unit/` — Unit tests for pure logic (e.g., `WIB` countdown intervals, short-notice/past-deadline rules, calendar validation, `.pdf` MIME/header & `<= 10 MB` checks).
  - `tests/integration/` — Route and workflow tests using Flask's `test_client()` and mocked external services (e.g., cron secret auth, `claimed_at` / `FOR UPDATE SKIP LOCKED` handling, `ON DELETE CASCADE` behavior, CRD upload/download/delete).
  - `tests/fixtures/` — Lightweight sample PDFs and test assets (keep fixture files small and non-copyrighted).
- **Local Verification:**
  ```bash
  ruff check .
  pytest -v
  ```
- **PR Approval Rule:** Team leads reviewing a code PR into `dev` should verify that relevant unit/integration tests are included and passing in GitHub Actions (`.github/workflows/ci.yml`) before approving.

---

## 🔄 Pull Request & Review Workflow

1. **Sync & Branch:**
   ```bash
   git checkout dev
   git pull origin dev
   git checkout -b <track>/<short-description>
   ```
2. **Make Focused Changes & Write Tests:**
   - Keep PRs scoped to a single task or issue linked to our **GitHub Project** board and **Milestone**.
   - Add or update `pytest` tests in `tests/unit/` or `tests/integration/` for any new or changed functionality.
   - Update relevant docs (`README.md`, `docs/`, `design/`, or `database/README.md`) if behavior, schema, or routes change.
3. **Self-Check Before Opening PR:**
   - Run `ruff check .` and `pytest -v` locally.
   - Verify no secrets, `.env` files, VAPID private keys, database credentials, or copyrighted files are staged.
   - Verify edge cases (e.g., short-notice/past deadlines, deleting an agenda with pending reminders, non-PDF or `> 10 MB` files, empty dates in `WIB`).
4. **Request Reviews & Merge into `dev`:**
   - Open a PR targeting **`dev`** with a clear summary, testing notes, and issue reference.
   - Request review from **at least one lead of each team** impacted by the PR.
   - Address review feedback promptly; merge only after CI passes and required lead approvals are granted.
5. **Release / Promotion to `main`:**
   - Once a milestone or batch in `dev` is verified and ready, open a PR from `dev` to **`main`**.
   - **Tech Lead** reviews and approves the merge into `main`.

---

## 📌 GitHub Project & Milestones

- Update your issue card status on the **GitHub Project** board (`Backlog` → `Ready` → `In Progress` → `In Review / Testing` → `Done`) as work progresses.
- Keep issues assigned to the appropriate **Milestone** and apply a `blocked` label immediately if waiting on another team or external dependency.

---

## 🔐 Security & Hygiene Rules

- **Never commit `.env`**, API keys, VAPID private keys, Neon connection strings, or cron secrets. Use `.env.example` for placeholder variable names only.
- Keep commits clean and descriptive (e.g., `feat(team-c): add delete confirmation and reminder cleanup` or `test(team-d): add 10MB PDF validation unit tests`).