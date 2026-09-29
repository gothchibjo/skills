---
name: req-register
description:
  "Create a new PCR record in the active repo: register the customer's vision
  verbatim as PART A of requirements/PCR-NNNN-<slug>/01-vision.md, open an empty
  PART B Clarification Log, write skeletons of 02-business.md and
  03-technical.yaml, and add the PCR row to requirements/README.md. Phase of the
  req-workflow state machine; load only via the Skill tool from req-workflow,
  never on your own."
disable-model-invocation: true
metadata:
  protected: "true"
  copyright: "© 2026 github.com/gothchibjo"
---

# Phase: req-register

Creates a new PCR record in the active repo. Receives the vision source (text or
file path) and any metadata the user already gave.

## 1. Determine root and registry

- Root R = `git rev-parse --show-toplevel` (fallback: cwd). Registration root =
  `R/requirements/`; create if absent.
- Registry = `R/requirements/README.md`; create if absent: heading
  `# Requirements Registry` and a table `| PCR | Title | Status | Updated |`.

## 2. Allocate the PCR id

- List `R/requirements/PCR-<nnnn>-<slug>/` directories. nextId = max `<nnnn>` +
  1, zero-padded to 4 (`PCR-0001`).
- `slug` = kebab-case of the title: lowercase latin, only `[a-z0-9-]`, keep ≤ 40
  chars.

## 3. Registration questions (one batch)

Ask for anything still unknown (Question tool):

- requester — who submitted the request;
- title — short, may stay in the customer's language;
- channel — email / meeting / paste / git issue / etc.;
- artifact_language — `ru` or `en` (the artifacts are ours: CL questions, BR,
  YAML; the vision itself stays in its source language).
- source_language — infer from the vision text. Do not re-ask what was already
  supplied.

## 4. Split check

If the vision obviously bundles several independent feature clusters, ask
whether to register one PCR per cluster (default: yes — split by connectedness,
so each PCR stays independently plannable and reviewable). Each cluster gets its
own id/folder and its own PART A excerpt (verbatim slice). If declined, register
as one PCR.

## 5. Write the record

`R/requirements/PCR-NNNN-<slug>/`

- `01-vision.md` front matter:
  ```yaml
  ---
  pcr: PCR-0001
  title: ...
  slug: ...
  status: registered
  requester: ...
  channel: ...
  received: <YYYY-MM-DD>
  source_language: ...
  artifact_language: ...
  participants: [analyst, customer]
  ---
  ```
  body:
  ```markdown
  # PART A — Verbatim Vision (As-Is)

  > Immutable section: never edited after registration. Byte-for-byte as
  > received.

  <vision text exactly as received — defects, contradictions and all. No fixes,
  no normalization, no inferred sections.>

  # PART B — Clarification Log (append-only)

  > Never rewritten; only appended to by req-elicitation.
  ```
- `02-business.md` skeleton:
  ```markdown
  ---
  pcr: PCR-0001
  title: ...
  status: draft
  revised: <YYYY-MM-DD>
  based_on: 01-vision.md
  approved_by: null
  approved_at: null
  revisions: []
  ---

  # Context & Goals

  # Stakeholders

  # Glossary

  # Assumptions Ledger

  # Non-Goals

  # Atomic Business Requirements
  ```
- `03-technical.yaml` skeleton:
  ```yaml
  pcr: PCR-0001
  title: ...
  status: draft
  revised: <YYYY-MM-DD>
  frozen_by: null
  frozen_at: null
  pipeline_gates:
    requirement_freeze: { required: true, passed: false }
    plan_approval: { required: true, passed: false }
    verification_evidence: { required: true, passed: false }
    guard_conditions: { required: true, passed: false }
  requirements: []
  ```
- Registry row: `| PCR-0001 | <title> | registered | <today> |`.

## 6. Report back to req-workflow

Return `pcr`, the record folder path, `slug`, and the note that the next event
is elicitation round 1 (fresh record, PART B empty).
