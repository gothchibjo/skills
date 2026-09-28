# req-verification

Phase 4 of [req-develop]: the verification-evidence gate (REQ-013) and the
traceability chain (REQ-014). A REQ is not done because code exists — it is done
because the evidence its contract asked for exists.

**Not directly invocable.** The frontmatter sets
`disable-model-invocation: true` — loaded only through the state machine.

## What it does

**`verify` — record evidence per REQ.** For each REQ you claim is done, it maps
the concrete artifact that proves it against the `verification.evidence` and
`verification.method` declared in the frozen YAML:

| Method          | Evidence                                   |
| :-------------- | :----------------------------------------- |
| `test`          | A runnable check or test report            |
| `inspection`    | Reviewed file paths plus a review sign-off |
| `manual-review` | A review verdict or traceability matrix    |
| `simulation`    | A run artifact or logs                     |

Rows are appended to the Evidence Matrix in `04-plan.md`:

```markdown
| REQ     | verification.method | evidence artifact          | status |
| ------- | ------------------ | -------------------------- | ------ |
| REQ-004 | test               | tests/e2e/export.spec.ts   | done   |
| REQ-007 | manual-review      | review sign-off 2026-03-14 | waived |
```

**`complete` — close the gate.** Allowed only when every REQ in the frozen YAML
has a `done` or explicitly `waived` row. It then flips
`pipeline_gates.verification_evidence.passed: true`, sets `04-plan.md` to
`done`, and appends the traceability chain
`BR-<n> → REQ-<n> → evidence artifact` for every REQ.

## Design decisions worth knowing

- **Evidence must match the declared method.** A test report does not close a
  REQ whose method is `manual-review`; the row stays `open`.
- **A waiver is a recorded exception, not silence.** A `waived` row requires a
  written reason in the matrix, and remains visible forever.
- **No evidence, no closure.** A REQ with neither an evidence row nor a waiver
  cannot be claimed complete.
- **Gaps are listed, never hidden.** Every break in the `BR → REQ → evidence`
  chain is reported explicitly, including at `complete`.
- **A gap is not fixed by editing the contract.** Changing a REQ to match the
  evidence is refused and routed to change control (REQ-015) via [req-workflow].

## Install

Not installed on its own; it ships with the development pipeline.

```bash
npx skills add gothchibjo/skills -g -a opencode
```

See [docs/installation.md] for the full matrix.

## Troubleshooting

| Symptom                           | Fix                                                                            |
| :-------------------------------- | :----------------------------------------------------------------------------- |
| A REQ stays `open`                | Its evidence does not match the declared method. Produce the right artifact.   |
| `complete` refused                | The REQs lacking evidence are named.                                           |
| You need to waive something       | Mark it `waived` with a reason; it is allowed, but it is recorded, not hidden. |
| It refuses to "fix" a failing REQ | Correct — that would edit a frozen contract. Open a change-request PCR.        |

## License

MIT. See [LICENSE].

<!-- refs -->

[req-develop]: ../req-develop/
[req-workflow]: ../req-workflow/
[docs/installation.md]: ../../../docs/installation.md
[LICENSE]: ../../../LICENSE
