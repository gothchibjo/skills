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
| `glossary`            | Load-bearing terms, definitions copied from the PCR's terms slice                           |

Then it re-reads and parses the YAML it just wrote, and reports a coverage
table: orphans on both sides, acceptance criteria that cannot be checked,
unresolved assumptions carried forward as guards, glossary terms included and
any open conflict.

**`freeze` — seal.** Allowed only when the BR is `approved`, the Assumptions
Ledger has no `(unresolved)` items, and no glossary conflict is open. Sets
`status: frozen`, records who and when, and flips
`requirement_freeze.passed: true`.

## Design decisions worth knowing

- **Generation depth depends on approval.** `approved` → the full field set.
  `proposed` → a structural draft only (`id`, `traces_to`, `type`, `title`,
  `description`, `priority`, `acceptance_criteria`) with verification, guards,
  size and glossary omitted, and a note that a full regeneration is owed after
  approval. `draft` → refused outright.
- **Coverage holes are reported, never dropped.** A BR with no REQ, or a REQ
  tracing to no BR, is surfaced in the report.
- **A glossary conflict is a block, not a warning.** Every load-bearing term
  must be defined in the PCR's terms slice; a definition that contradicts the
  canon stops generation and refuses freeze. Other findings are reported and
  allowed through, but a REQ built on a wrong definition passes review while
  being wrong.
- **Definitions are copied, never re-worded.** The YAML `glossary` block and the
  terms slice must match; a mismatch is drift and freeze refuses.
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

See [docs/installation.md] for alternative installation methods.

## Troubleshooting

| Symptom                            | Fix                                                                                             |
| :--------------------------------- | :---------------------------------------------------------------------------------------------- |
| Generation refused                 | The BR is still `draft`. It must reach `proposed` first.                                        |
| A `proposed` BR produced thin REQs | Expected — approve the BR, then regenerate for the full field set.                              |
| Freeze refused                     | The named condition is unmet: approve the BR, resolve the ledger, or resolve the term conflict. |
| A term used in a REQ is undefined  | Reported, not invented: add it to the terms slice (and the canon) first.                        |
| YAML will not parse                | It re-parses before reporting; fix the reported error before proceeding.                        |

## License

MIT. See [LICENSE].

<!-- refs -->

[req-workflow]: ../req-workflow/
[docs/installation.md]: ../../../docs/installation.md
[LICENSE]: ../../../LICENSE
