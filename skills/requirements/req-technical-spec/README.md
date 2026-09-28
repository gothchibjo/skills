# req-technical-spec

Phase 4 of [req-workflow]: turn business requirements into a machine-readable
technical spec, and freeze it.

**Not directly invocable.** The frontmatter sets
`disable-model-invocation: true` — loaded only through the state machine.

## What it does

Two events, both in `03-technical.yaml`.

**`technical` — generate.** Maps every `BR-<n>` to at least one `REQ-<nnn>`:

| Field                 | Derived from                                                                                |
| :-------------------- | :------------------------------------------------------------------------------------------ |
| `id`                  | Topological order — dependencies first, ties by BR order. Deterministic.                    |
| `type`                | `functional` (a behavior), `non-functional`, `business-rule` (a rule), `constraint`         |
| `description`         | One unambiguous EARS-style sentence: "WHEN \<trigger\>, THE SYSTEM SHALL \<outcome\>"       |
| `priority`            | The BR's own `must` / `should` / `could`                                                    |
| `acceptance_criteria` | The BR's criteria, made concrete and checkable                                              |
| `dependencies`        | The BR's `depends on`, plus conservative intra-REQ data/order dependencies                  |
| `verification`        | `method` (`test`, `manual-review`, `inspection`, `simulation`) + named `evidence` artifacts |
| `guard_conditions`    | Seeded from constraints; every `(unresolved)` assumption becomes a guard                    |
| `size_hint`           | `xs`–`l` heuristic from breadth and dependency count                                        |

Then it re-reads and parses the YAML it just wrote, and reports a coverage
table: orphans on both sides, acceptance criteria that cannot be checked,
unresolved assumptions carried forward as guards.

**`freeze` — seal.** Allowed only when the BR is `approved` _and_ the
Assumptions Ledger has no `(unresolved)` items. Sets `status: frozen`, records
who and when, and flips `requirement_freeze.passed: true`.

## Design decisions worth knowing

- **Generation depth depends on approval.** `approved` → the full field set.
  `proposed` → a structural draft only (`id`, `traces_to`, `type`, `title`,
  `description`, `priority`, `acceptance_criteria`) with verification, guards
  and size omitted, and a note that a full regeneration is owed after approval.
  `draft` → refused outright.
- **Coverage holes are reported, never dropped.** A BR with no REQ, or a REQ
  tracing to no BR, is surfaced in the report.
- **Evidence is named generically.** "test report", "traceability matrix",
  "review sign-off" — the tech stack is undecided at this stage, so a
  stack-specific artifact would be a guess.
- **Freezing is one-way.** A frozen spec is sealed: later `technical` events are
  refused, because a scope change at that point is a new PCR (change control).
- **It never touches the requirements.** `01-vision.md` and `02-business.md` are
  read-only here.
- **The gates it does not own stay false.** `plan_approval`,
  `verification_evidence` and `guard_conditions` are flipped by Phase 2.

## Install

Not installed on its own; it ships with the requirements pipeline.

```bash
npx skills add gothchibjo/skills -g -a opencode
```

See [docs/installation.md] for the full matrix.

## Troubleshooting

| Symptom                            | Fix                                                                      |
| :--------------------------------- | :----------------------------------------------------------------------- |
| Generation refused                 | The BR is still `draft`. It must reach `proposed` first.                 |
| A `proposed` BR produced thin REQs | Expected — approve the BR, then regenerate for the full field set.       |
| Freeze refused                     | Both conditions are named: approve the BR, or resolve the ledger.        |
| YAML will not parse                | It re-parses before reporting; fix the reported error before proceeding. |

## License

MIT. See [LICENSE].

<!-- refs -->

[req-workflow]: ../req-workflow/
[docs/installation.md]: ../../../docs/installation.md
[LICENSE]: ../../../LICENSE
