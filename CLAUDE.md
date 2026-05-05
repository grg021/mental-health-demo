# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**eMPHRe** — Electronic Mental and Public Health Registry. A clinical workflow prototype for Pasig City LGU (Philippines) mental health facilities. The full domain spec, data model, seeded test data, validation rules, and planned-but-unbuilt features live in `EMPHRE_PROJECT_CONTEXT.md` — read it before changing clinical logic, terminology, or workflow steps. It is the source of truth for *intent*; the HTML file is the source of truth for *current behavior*.

### `context/` folder

A local reference folder (gitignored — `context/.gitignore` is `*`, so nothing here is committed). Treat it as read-only background material, not as files to edit:

- `emphre_system_architecture_v2.svg` / `.jpg` — current system architecture diagram. View this (especially the SVG) when you need to understand the three clinical modules + four supporting modules + role/access matrix at a glance.
- `emphre_system_architecture.svg` / `.jpg` — earlier v1 of the diagram, kept for reference. Prefer v2.
- `mh_registry_prototype.html` and `EMPHRE_PROJECT_CONTEXT.md` — snapshots identical to the root copies. Edit only the root files; do not modify the ones in `context/`.

## Running / "building"

There is no build, no package manager, no tests, no lint. The entire app is one file: `mh_registry_prototype.html` (~4450 lines, all CSS + JS inline, vanilla JS, no external runtime dependencies).

To run: open the file directly in a browser (`open mh_registry_prototype.html` on macOS). After edits, just reload — there is nothing to recompile.

## Architecture

**Single-page, in-memory.** State is a top-level `const PATIENTS = [...]` array seeded with 10 fixtures (lines ~1571–2186). There is no persistence: refresh discards all changes. Do not add backend integration without checking with the user — it's intentionally a frontend-only prototype.

**View switching:** two `.container` divs (`#registryView`, `#patientView`) toggle via `display`. Three modals overlay both: `#modal` (new patient wizard), `#assessModal` (assessment / follow-up — reused for both flows), `#visitDetailModal` (read-only visit view).

**Flow shapes — these differ deliberately, don't unify them:**
- New patient → 3-step wizard (`renderStep1/2/3`, `savePatient`)
- New assessment (first visit) → 3-step wizard (`renderAssessStep1/2/3`, `saveAssessment`)
- Follow-up consult (subsequent visit) → single scrollable form, NOT stepped (`openFollowUp`, `renderFollowUpForm`, `saveFollowUp`). Returning consults are faster and don't re-collect full MSE.

**Read-only history is a hard rule.** Previous-visit data shown in follow-up modal and Visit Detail modal must never be editable. New entries always append to `assessments[]` — never mutate prior entries. An edit workflow is intentionally deferred.

**Code is sectioned by banner comments** like `/* =================== TOAST =================== */`. Use those (and the function name index — `grep -n '^\s*function ' mh_registry_prototype.html`) to navigate; the file is too long to read top-to-bottom productively.

## Conventions specific to this codebase

- **No `alert()` / `confirm()` / `prompt()`** — the prototype is often viewed inside sandboxed iframes that block them silently. Use the existing `showToast()` for success/info, and the inline red-banner pattern (`showValidationError(msg, fieldIds)`) for validation.
- **Auto-computed fields stay editable.** Age from DOB, dispensed qty from frequency (`freqToDispensed`), accession number, today's date defaults — all compute a sensible default but the user can override. Don't lock them.
- **Suicide-risk flag in MSE Step 2 has a cross-step side effect:** it auto-checks "Safety planning and suicide watch" in Step 3 and shows a red banner. Preserve this when touching either step.
- **Reference data (ICD-10 list, medication datalist, barangays, MHGAP conditions, referral destinations) is hardcoded inline.** When extending, keep the existing categorization and Filipino-context items (MHSATOP, CHAMP, VAWC/BCPC, CSWDO, WCPU, etc. — see EMPHRE_PROJECT_CONTEXT.md "Philippines-specific context").
- **Logged-in user is hardcoded** as Greg Hermo, RN / MH Nurse Coordinator (Facility) / Bagong Katipunan Health Center, in the header markup and used by `getProvider()` to stamp new assessments. There is no auth.
- **CSS uses custom properties** (`--primary`, `--text`, `--radius`, etc.) defined in `:root`. Use the tokens, don't hardcode colors.
- **Accession number format:** `MH-000-YY-NNN` (MH prefix, 3-digit facility code, 2-digit year, 3-digit sequence). The `000` facility code is a placeholder in the seed data.

## When editing

- Prefer surgical edits inside the existing function. Don't refactor the file into modules or introduce a build step unless the user explicitly asks — the single-file, no-dependencies property is a deliberate constraint (offline-capable, easy to share).
- After non-trivial UI changes, open the file in a browser and exercise the flow you touched. Type checking does not exist here; the only verification is clicking through.
