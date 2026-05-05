# eMPHRe — Electronic Mental and Public Health Registry

## Project overview

**eMPHRe** is a web-based mental health registry system designed for use by Pasig City LGU health facilities in the Philippines. The system manages patient records, clinical assessments, treatment tracking, and follow-up scheduling across the public health network.

A working interactive HTML prototype has been developed and is the current deliverable. The prototype lives in a single self-contained HTML file with all CSS and JavaScript inline.

**Current prototype file:** `mh_registry_prototype.html`

---

## System architecture

The system has three clinical modules and four shared supporting modules.

### Clinical modules

| Module | Description |
|--------|-------------|
| **Mental Health & Neurological** | Core MH patient registry and clinical workflow |
| **Substance Abuse** | DDAPTP-managed registry and workflow (same sub-modules, separate access) |

Each clinical module contains these sub-modules in sequence:
1. Patient Registry
2. Clinical Assessments
3. Treatment Care List
4. Follow-up Tracking
5. Reporting Engine

### Supporting (admin) modules

- User Management
- Health Facility Registry
- Audit Logs
- Data Import / Export

### User roles and access

| Role | Module access |
|------|--------------|
| MH Nurse Coordinator (Facility) | MH & Neurological only |
| MH Nurse Coordinator (LGU) | MH & Neurological only |
| MH Physician (LGU) | MH & Neurological only |
| DDAPTP Coordinator (Facility) | Substance Abuse only |
| DDAPTP Nurse Coordinator (LGU) | Substance Abuse only |
| DDAPTP Physician (LGU) | Substance Abuse only |
| System Administrator | Supporting/admin modules only |

The logged-in user in the current prototype is **Greg Hermo, RN** with role **MH Nurse Coordinator (Facility)** assigned to **Bagong Katipunan Health Center**.

---

## Prototype — current state

### What is built

The prototype is a fully interactive single-page HTML file. It does not connect to a backend — all data is held in a JavaScript `PATIENTS` array in memory.

#### Patient registry (landing page)

- **Privacy-first design:** patient list is NOT shown by default. Staff must search by Accession No., Last Name, or First Name to see any results
- Minimum one field required to trigger a search
- "No patient found" empty state with clear message
- Results show: Health Facility, Accession No., First Name, Middle Name, Last Name, Sex, Age, Barangay, Status
- **New Patient** button opens a 3-step wizard modal

#### New patient registration (3-step modal)

**Step 1 — Basic identification** (required fields trigger duplicate check)
- First Name *(required)*
- Middle Name
- Surname *(required)*
- Date of birth → Age (auto-computed, editable ±stepper)
- Sex *(required)*

**Step 2 — Duplicate check**
- Searches existing records by surname + first name prefix + sex
- Shows possible matches with confidence badges (High / Partial)
- "Use this record" shortcut to avoid duplicates
- "Proceed as new patient" if no match

**Step 3 — Patient & case details**
- Barangay *(required, dropdown)*
- Address
- Occupation
- Contact No. 1 & 2
- Health Facility *(auto-selected to logged-in user's facility)*
- PhilHealth ID Number (PIN)
- Guardian/Relative Name
- Guardian Relationship (Parent, Sibling, Child, Friend, Others)
- Accession No. *(auto-generated `MH-000-YY-NNN`, editable)*
- Date of Initial Assessment *(defaults to today)*
- Case Type: New / Old *(radio)*
- Referral Source: Walk-in / School / Workplace (with referral letter) / Hospital/Clinic *(radio)*

#### Patient detail view

Accessed by clicking **View →** in the search results. Shows:
- **Sidebar:** patient profile, identification, contact, guardian, case details, status badge
- **Breadcrumb** navigation back to registry
- **Six tabbed sections** (all show historical data as tables, newest-first):

| Tab | Contents |
|-----|----------|
| Complaints & History | Chief complaints (patient/guardian), HPI, onset, reliability, past medical, family history |
| Examination | Mental state examination (MSE) summary across visits |
| Diagnosis | ICD-10/11 codes and MHGAP conditions across visits |
| Treatment | Psychosocial interventions and medications (with dispensed qty) |
| Referral | Referral destinations, remarks, follow-up dates |
| History of Visits | Full visit summary table with **View →** button per row |

Each tab has a **Follow-up consult** button once at least one assessment exists.

#### Patient status values

`Active` | `Referred` | `Recovered` | `Lost to Followup` | `Deceased`

#### New assessment (3-step modal)

Triggered on first visit (no prior assessments).

**Step 1 — Complaints & History**
- Consultation/assessment date *(required, defaults today)*
- Chief complaint from patient *(required)*
- Chief complaint from guardian *(required)*
- Onset, Reliability
- History of Present Illness
- Past Medical History
- Family History

**Step 2 — Mental State Examination (MSE)**

14 MSE items:
1. Appearance — radio (Kempt / Unkempt)
2. Behavior/Mannerism — textarea
3. Eye contact — radio (With / Without)
4. Level of consciousness — radio + Others specify (Alert / Stuporous / Confused / Others)
5. Attitude — radio + Others specify (Cooperative / Suspicious / Apathetic / Others)
6. Orientation — checkboxes (Time / Place / Person / Event)
7. Speech and language — radio + Others specify (Normal / Soft / Loud / Pressured / Others)
8. Mood — radio + Others specify (Neutral / Anxious / Irritable / Cheerful / Euphoric / Depressed / Others)
9. Affect — radio + Others specify (Congruent / Incongruent / Blunted / Flat / Labile / Others)
10. Thought process — radio (Linear / Goal oriented / Tangential / Loose associations / Disorganized)
11. Hallucinations — checkboxes (Auditory / Visual)
12. Self-harm & Suicidal behavior — checkboxes (With suicide attempt / Currently suicidal) → conditional reveal for method, risk assessment text, remarks
13. Homicidal/Harm to others — checkbox → conditional remarks field
14. Risk factors & Psychosocial concerns — checkboxes (10 options including bullying, abuse, substance use, PWD, housing instability, etc.)

> Suicide risk flag in Step 2 auto-highlights **Safety planning and suicide watch** in Step 3 and shows a red warning banner.

**Step 3 — Diagnosis & Intervention**

- **ICD-10/11 code** — searchable autocomplete from 80+ MH-related ICD-10 codes
- **MHGAP conditions** — checkboxes (9 options)
- **Psychosocial interventions** — 14 checkboxes (Psychoeducation through General Intervention)
- **Pharmacological interventions** — table with: Medication (datalist autocomplete from 120+ MH medications), Frequency (OD/BD/TD/QID/PRN/QWEEK), Dispensed qty (auto-computed from frequency × 30-day supply, editable)
- **Referral** — 16 destination checkboxes + remarks textarea
- **Follow-up** — date field (own section, separate from referral)

#### Follow-up consult modal

Triggered by **Follow-up consult** button on any tab of a patient with existing assessments. Single scrollable form (no step wizard). Header shows which previous visit it continues from.

Contains three **collapsible sections** showing read-only history tables:

1. **Complaints & History** — table of all previous visits' complaints + "+ Add notes for this visit" button that reveals new-entry fields
2. **Diagnosis** — table of all previous diagnoses + "+ Add / update diagnosis" button (ICD autocomplete + MHGAP checkboxes)
3. **Psychosocial Issues / Concerns** — table of all previous risk factors + "+ Add / update concerns" (checkboxes)

> **Previous visit data is strictly read-only.** Editing prior records is intentionally not supported in the current workflow. A separate edit workflow is planned but not yet scoped.

Followed by always-visible sections:
- Psychosocial interventions (checkboxes, start unchecked)
- Pharmacological interventions (med table, pre-filled from previous visit, editable)
- Referral (fresh checkboxes + remarks)
- Follow-up date (own section)

#### Visit Detail modal

Opened by **View →** buttons in history tables and History of Visits tab. Shows a complete read-only record of one visit.

- **Meta bar at top:** Provider name + role-color badge + health facility
- **7 collapsible sections:** Complaints & History, Mental State Examination, Risk Assessment, Diagnosis, Treatment, Referral, Follow-up
- Smart expand/collapse defaults: Risk Assessment always open; sections with no data auto-collapse with a "Not recorded" badge; Risk Assessment shows a red "⚠ Risk flagged" badge if suicide/homicidal flags were set

---

## Data model

### Patient object

```js
{
  facility: String,       // Health facility name
  accession: String,      // e.g. "MH-000-25-001"
  first: String,
  middle: String,
  last: String,
  sex: 'M' | 'F',
  age: Number,
  dob: String,            // ISO date "YYYY-MM-DD"
  barangay: String,
  address: String,
  contact1: String,
  contact2: String,
  occupation: String,
  pin: String,            // PhilHealth ID
  guardian: String,
  relationship: String,
  caseType: 'New' | 'Old',
  referralSource: String,
  status: 'Active' | 'Referred' | 'Recovered' | 'Lost to Followup' | 'Deceased',
  assessments: Assessment[],
}
```

### Assessment object

```js
{
  date: String,             // ISO date
  cc_patient: String,
  cc_guardian: String,
  onset: String,
  reliability: String,
  hpi: String,
  pastMedical: String,
  familyHistory: String,
  mse: {
    appearance: String,
    behavior: String,
    eyeContact: String,
    consciousness: String,
    attitude: String,
    orientation: String[],
    speech: String,
    mood: String,
    affect: String,
    thoughtProcess: String,
    hallucinations: String[],
    selfHarm: {
      attempted: Boolean,
      current: Boolean,
      method: String,
      riskAssessment: String,
      remarks: String,
    },
    homicidal: Boolean,
    homicidalRemarks: String,
    riskFactors: String[],
  },
  diagnosis: {
    icd: String,            // Full ICD code + description
    mhgap: String[],
  },
  treatment: {
    psychosocial: String[],
    meds: [{ name: String, freq: String, dispensed: Number }],
  },
  referral: {
    referTo: String[],
    remarks: String,
    followUp: String,       // ISO date
  },
  provider: {
    name: String,
    role: String,
    facility: String,
  },
  _isFollowUp: Boolean,     // true for follow-up consults
}
```

---

## Seeded test data

10 patients with accession numbers `MH-000-25-001` through `MH-000-25-010`, covering all MHGAP condition categories:

| Accession | Patient | Diagnosis | Status | Visits |
|-----------|---------|-----------|--------|--------|
| 001 | Maria Lourdes Dela Cruz, F/38 | GAD (F41.1) | Active | 1 |
| 002 | Jose Emmanuel Buenaventura, M/22 | MDD moderate + suicidal ideation (F32.1) | Active | 1 |
| 003 | Rosario Villanueva, F/55 | Schizophrenia paranoid (F20.0) | Active | 2 |
| 004 | Kevin Ong, M/16 | MDD mild + NSSI (F32.0) | Active | 1 |
| 005 | Anastacia Florendo, F/67 | Dementia Alzheimer's (F00.1) | Referred | 1 |
| 006 | Danilo Pagtalunan, M/29 | Alcohol dependence (F10.2) | Active | 1 |
| 007 | Luz Macapagal, F/44 | PTSD post-fire (F43.1) | Recovered | 2 |
| 008 | Ramon Jr. Espiritu, M/34 | Bipolar I manic (F31.1) | Active | 1 |
| 009 | Aireen Domingo, F/31 | Panic disorder (F41.0) | Lost to Followup | 1 |
| 010 | Eduardo Tolentino, M/58 | MDD severe + high suicide risk (F32.3) | Deceased | 1 |

---

## Reference data included

### ICD-10 MH codes
~80 F-codes plus relevant Z-codes (X84, Z03.2, Z65.4, Z91.5), covering dementia, psychosis, mood disorders, anxiety, OCD, PTSD, personality disorders, intellectual disability, ASD, ADHD, conduct disorders.

### MHGAP conditions (9)
Depression, Anxiety, Psychosis, Epilepsy, Child & Adolescent Mental & Behavioral Disorder, Dementia, Self-harm/Suicide attempt, Disorders due to Substance Use, Other Significant Mental Health Concerns

### Psychosocial interventions (14)
Psychoeducation → General Intervention (includes Katatagan Plus, KKDK, CBT Specialist, PSP/Debriefing, Safety Planning, etc.)

### Referral destinations (16)
Emergency, Psychiatry, Neurology, Specialties, Pediatrics/Developmental, Psychologist, VAWC/BCPC, CSWDO, WCPU, School, Smoking Cessation Clinic, MHSATOP, CHAMP Specialty Clinic, HC/SHC, Social Hygiene Clinic, Others

### Medication datalist (~120 items)
SSRIs, SNRIs, TCAs, atypical antidepressants, typical + atypical antipsychotics, mood stabilizers, anticonvulsants, benzodiazepines, anti-dementia agents, stimulants (ADHD), anticholinergics (EPS), substance use medications.

### Dispensed quantity logic
`freqToDispensed(freq)` → 30-day supply default:
- OD → 30 tabs
- BD → 60 tabs
- TD → 90 tabs
- QID → 120 tabs
- PRN → 30 tabs
- QWEEK → 5 tabs

---

## Key UX decisions

| Decision | Rationale |
|----------|-----------|
| No free-scroll patient list | MH data is sensitive; requires intentional search to access records |
| Age auto-computed but editable stepper | Some patients cannot establish DOB; age estimate still required |
| Duplicate check before patient creation | Prevents accidental duplicate records in multi-facility LGU context |
| Follow-up as single-page form (no steps) | Returning consults are faster; no need to re-enter full MSE every visit |
| Previous visit data read-only | Audit integrity; no silent edits to clinical records |
| Safety planning auto-highlighted on suicide flag | Clinical safety — ensures care provider doesn't miss the intervention |
| Referral and Follow-up as separate sections | Conceptually distinct: referral is to another provider, follow-up is return to same team |
| Dispensed qty auto-computed from frequency | Reduces data entry; 30-day supply is standard for community MH programs |

---

## Validation rules (current)

- New patient Step 1: First Name, Surname, Sex all required before duplicate check
- New patient Step 3: Barangay required; Case Type and Referral Source required
- New assessment Step 1: Date, both Chief Complaints required
- Follow-up: Consultation date required
- All validation shows inline red banner + field highlighting (no `alert()` — blocked in sandboxed iframes)

---

## Known gaps / planned work

- [ ] Complete `dispensed` qty capture in `readFollowUpMeds()` and `saveFollowUp()` — partially implemented
- [ ] Wire freq→qty auto-compute in follow-up form's med table
- [ ] Show `dispensed` qty in Treatment tab history table and Visit Detail modal
- [ ] Edit workflow for previous assessment data (intentionally deferred)
- [ ] Epilepsy-specific assessment fields
- [ ] Dementia-specific assessment fields (MMSE score capture)
- [ ] Screening tool score capture (PHQ-9, GAD-7, PCL-5, CAGE, MMSE)
- [ ] Patient status change workflow (with reason and date)
- [ ] Report generation (HC MHP Report format)
- [ ] Backend API integration (currently all in-memory JS)
- [ ] Authentication / session management (currently hardcoded user)
- [ ] Substance Abuse module (DDAPTP workflow, separate from MH)
- [ ] Neurological module (epilepsy-specific)
- [ ] User management screens
- [ ] Health Facility Registry screens
- [ ] Audit log screens
- [ ] Mobile responsiveness improvements

---

## Technology stack (prototype)

- **Single-file HTML** — all CSS and JS inline, no build step
- **Vanilla JavaScript** — no frameworks
- **CSS custom properties** — follows Claude.ai design token conventions (`--color-*`, `--border-radius-*`, etc.) for theming compatibility
- **HTML `<datalist>`** — medication autocomplete (native, no JS overhead)
- **Custom JS autocomplete** — ICD-10 code search with match highlighting
- **No external dependencies** — fully offline-capable

---

## Philippines-specific context

- **LGU:** Local Government Unit (Pasig City)
- **Health Centers:** barangay-level primary care facilities
- **MHSATOP:** Mental Health Services and Treatment Outreach Program
- **CHAMP:** Community Health and Multi-disciplinary Program (specialty clinic)
- **DDAPTP:** Drug Dependency Assessment Program and Treatment Protocol
- **Barangays in scope:** Bagong Ilog, Bagong Katipunan, Bambang, Caniogan, Kapasigan, Kapitolyo, Manggahan, Pinagbuhatan, San Miguel, Santa Lucia, Ugong
- **PhilHealth PIN format:** `XX-XXXXXXXXX-X`
- **Accession number format:** `MH-000-YY-NNN` (MH prefix, 3-digit facility code, 2-digit year, 3-digit sequence)
- **MHGAP:** WHO Mental Health Gap Action Programme — the clinical framework used in the assessments

---

## File inventory

| File | Description |
|------|-------------|
| `mh_registry_prototype.html` | Main interactive prototype (all-in-one) |
| `emphre_system_architecture_v2.svg` | System architecture diagram (editable) |
| `emphre_system_architecture_v2.jpg` | System architecture diagram (JPEG export) |
| `EMPHRE_PROJECT_CONTEXT.md` | This file |
