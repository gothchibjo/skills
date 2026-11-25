---
name: req-elicitation
description:
  "Clarify and formalize business requirements for a PCR: run the grilling
  interview in rounds, append each round to PART B as CL entries, and author
  02-business.md once the frontier is empty. Maintains the glossary canon and the
  stakeholder registry with stable ids. Phase of req-workflow; load only via the
  Skill tool from req-workflow."
disable-model-invocation: true
metadata:
  protected: "true"
  copyright: "© 2026 github.com/gothchibjo"
---

# Phase: req-elicitation

Clarifies a PCR's vision into formalized business requirements. Called by
req-workflow with the event (`new` | `continue` | `answers`) and the target PCR.
Cost rules R1–R6 from req-workflow apply; §1 spells out the sliced reads this
phase needs.

## 1. Load the record — as little as the event needs

- **Every round.** `01-vision.md` front matter (`artifact_language`), PART A
  (the vision is the interview's subject) and **only** the latest CL entry and
  any `(awaiting)` entry — grep for `### CL-` and `(awaiting)`, do not read PART
  B whole. `02-business.md` front matter only.
- **Authoring the BR (§6).** Now read PART B once — every answered round — and
  the `02-business.md` sections being rewritten. This is the second legitimate
  whole-document read in Phase 1: it happens once per BR revision, not once per
  event.
- **A term in play.** grep the canon (and `glossary_ref:`) for the term, its
  aliases and its id — never read the whole glossary, which is project-wide and
  keeps growing.
- **An actor in play.** grep `requirements/stakeholders.md` for the name — never
  read the roster whole.
- **`status`-style resume.** Front matter, the awaiting marker and the open
  questions from grep are enough; no body reads.

Then derive the next ids without reading either project-wide file to the end:

```bash
# next CL / G / S, without reading the file to the end
grep -o 'CL-[0-9]*' 01-vision.md | grep -o '[0-9]*' | sort -n | tail -1
grep -o 'G-[0-9]*' <canon> | grep -o '[0-9]*' | sort -n | tail -1
grep -o 'S-[0-9]*' requirements/stakeholders.md | grep -o '[0-9]*' | sort -n | tail -1
```

Zero-pad to 3 for `G` and `S`, none for `CL`. If `stakeholders.md` is absent,
create it with heading `# Stakeholders` and the registry table header.

## 2. Interpret the event

- `new` (first round): no CL yet → open round 1.
- `continue` without answers: re-pose the questions of the latest `(awaiting)`
  CL — resume exactly there, do not restart.
- `answers`: append the pasted replies to the first `(awaiting)` CL, flip it to
  `(answered)`, then continue with the next round.

## 3. Run the interview (grilling method)

Design-tree interview: every decision branches into the decisions hanging off
it. The frontier = questions whose prerequisites are already settled. Ask the
whole frontier in one round: number each question and give a recommended answer
(`➡️ ...`). Wait for answers before the next round.

- Facts are yours to find; decisions and domain knowledge belong to the customer
  proxy (the terminal user). Never ask for anything you can look up — but look
  it up with grep or a sliced read, and reach for a subagent only when the
  answer genuinely spans many files. A subagent answers in a short digest; do
  not have it dump files back.
- A term that the canon or `glossary_ref:` defines differently (same term or
  alias, other meaning) is a contradiction: ask a numbered question quoting both
  definitions. Never resolve it by overwriting the canon.
- Who is affected, who decides, who can block, and what each wants from this
  change are domain decisions: ask them. Never infer a person's role or
  authority.
- Nothing is ignored silently: ambiguity, contradiction, and missing detail all
  become questions. Deliver and record each round fully.

## 4. Persist each round into PART B (append-only, composed once)

Compose the round **once**. Append exactly that text to PART B and return the
same text to req-workflow, which shows it to the user — do not write the
questions for the file and again for the chat.

```markdown
### CL-<n> (awaiting) — <YYYY-MM-DD>

> Q1 — <title>: <question> ➡️ <recommended answer> Q2 — <title>: <question> ➡️
> <recommended answer>
```

When answers arrive, append them under that CL and flip the heading marker to
`(answered)`:

- User's own words → `> CUSTOMER (relay): <text>`.
- Exact customer quote / correspondence → `> VENDOR (verbatim): <exact text>` —
  never edited. The terminal user is the authoritative customer proxy; the
  distinction keeps the record honest about what is verbatim versus relayed.

## 5. Converge

Done when the frontier is empty — no spec-changing question left. Then stop
asking and author the business document (§6). If the user says "enough for now"
with the frontier non-empty, move the missing items into the Assumptions Ledger
as unresolved and still author the document.

## 6. Author `02-business.md`

Front matter: `status: proposed`, `revised: <today>`, `revisions: []` (keep
`pcr`, `based_on`, `approved_by/at` fields).

Fill the skeleton's sections once. On a rework, edit only the sections the
feedback touches — never regenerate the whole document, and never rewrite
sections whose text did not change.

- **Context & Goals** — 2–5 crisp sentences.
- **Stakeholders for This Change** — only those who matter here, one line each.
  New actors are added to the registry first (§8), so every row carries an
  `S-<nnn>` id; the requester from `01-vision.md` must appear.

  ```markdown
  | id    | actor       | interest in this change               | position |
  | :---- | :---------- | :------------------------------------ | :------- |
  | S-001 | Бухгалтерия | Нужен экспорт с НДС, иначе не подпишет | blocks   |
  ```

  `position`: `supports | neutral | blocks`. Identity, role and authority live
  in the registry (§8), not here.

- **Terms Used in This PCR** — the slice this PCR relies on, one line each. New
  terms are appended to the canon first (§7), so every row here carries a
  `G-<nnn>` id. Definitions are copied from the canon as of authoring: this copy
  is the seal, so later canon edits cannot change what this contract means.

  ```markdown
  | id    | term      | definition (as used) | origin          |
  | :---- | :-------- | :------------------- | :-------------- |
  | G-001 | заказ     | ...                  | canon           |
  | G-014 | предзаказ | ...                  | new in PCR-0001 |
  ```

- **Assumptions Ledger** — table: assumption/decision, source (`[V §x]` /
  `[CL-n]`), resolved. Every unresolved item is marked `(unresolved)`.
- **Non-Goals** — explicitly out of scope.
- **Atomic Business Requirements** — the core. Rules:
  - Our wording, not the customer's: concise, unambiguous, one atomic concern
    per requirement, sorted by priority. Cross-check for internal contradictions
    — a contradiction is resolved via a question round or marked unresolved,
    never silently dropped.
  - Format per BR:
    ```markdown
    ### BR-01 — <short title>

    - description: <one atomic behavior or service level; testable at business
      level>
    - priority: must | should | could
    - rationale: <why; from [V §x] / [CL-n]>
    - acceptance criteria (business): <measurable outcome or rule>
    - traces to: [V §x], [CL-n]
    - depends on: BR-0x (or none)
    ```
  - Sort: `must` first, then `should`, then `could`.

Traceability backstop: every BR cites at least one source (`[V §x]` or
`[CL-n]`); anything unsourced is sourced, turned into an unanswered question, or
listed in the Assumptions Ledger.

## 7. Maintain the canon

The canon is `requirements/glossary.md` (or the path in the registry header). If
it does not exist, create it with the heading `# Project Glossary` and this
table header. One table, append-only:

```markdown
| id    | term  | aliases | definition | source              | status |
| :---- | :---- | :------ | :--------- | :------------------ | :----- |
| G-001 | заказ | order   | ...        | PCR-0001/2026-11-25 | active |
```

- Match by id, then by term or alias, case-insensitively. A term already defined
  with the same meaning is referenced by its id, never re-added.
- A new term: append one row with the next id, `source: PCR-<nnnn>/<today>`,
  `status: active`.
- A term defined only in `glossary_ref:` is appended to the canon on first use,
  with `source: <glossary_ref path>` and the definition copied, not re-worded;
  then it is referenced by its new id.
- A different meaning for an existing term is a conflict. Do not write it: ask
  the question (§3) and, while it is open, record it in the Assumptions Ledger
  as `(unresolved)` with both definitions and their sources. That entry blocks
  `freeze`.
- Only an explicit user answer may reshape a term: append the new row, then flip
  the old row's `status` to `superseded-by G-<nnn>`. Flipping `status` is the
  only mutation an existing row ever receives.
- Write definitions in `artifact_language`; keep the term in the customer's
  words and put synonyms in `aliases`. Never write `glossary_ref:`. Never read
  the canon end to end — grep the term, then edit the row.

## 8. Maintain the stakeholder registry

`requirements/stakeholders.md`, one table. Rows are immutable except `status`:

```markdown
| id    | actor       | role         | type     | authority      | status | source              |
| :---- | :---------- | :----------- | :------- | :------------- | :----- | :------------------ |
| S-001 | Бухгалтерия | finance team | internal | decision-maker | active | PCR-0001/2026-11-25 |
```

- `type`: `internal | external`; `authority`:
  `decision-maker | influencer | recipient | blocker`; `status`:
  `active | inactive | superseded-by S-<nnn>`. Role, type and authority are
  decisions — ask, never guess. `source` is `PCR-<nnnn>/<today>`.
- Match actors by name, case-insensitively, by grepping the table. An actor
  already present with the same role and authority is referenced by id, never
  re-added.
- A changed role or authority is not an edit: after the user confirms, append
  the new row and flip the old one to `superseded-by S-<nnn>`. A person leaving
  is `inactive`, never a deletion.
- Every `S-<nnn>` in the PCR slice must exist in the registry. A duplicate actor
  with conflicting role or authority is a warning, not a block: report it and
  ask which row is current.
- Approval authority is the one hard check, and it belongs to req-workflow's
  `approve` step, not here.

## 9. Report back to req-workflow

Return a compact result: the round text verbatim when a CL is awaiting answers
(req-workflow shows it), BR composed (ready for `br_review`), or an error. Name
any open glossary conflict, the terms added to the canon, and the stakeholder
rows added or changed. Do not address the user — req-workflow owns that.
