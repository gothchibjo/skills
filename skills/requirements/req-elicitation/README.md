# req-elicitation

Phase 2 of [req-workflow]: interview the customer until the frontier is empty,
then author the business requirements document.

**Not directly invocable.** The frontmatter sets
`disable-model-invocation: true` — loaded only through the state machine.

## What it does

It runs the [grilling] method against the vision: a design-tree interview where
every decision branches into the decisions hanging off it. The **frontier** is
the set of questions whose prerequisites are settled. The whole frontier is
asked in one round — numbered, each with a recommended answer — then it waits.

```markdown
### CL-7 (awaiting) — 2026-03-14

> Q1 — **Retention**: how long do audit records live? ➡️ 3 years
> Q2 — **Export**: can the client export the report as CSV? ➡️ no
```

When your answers arrive they are appended under that entry and the marker flips
to `(answered)`, then the next round is asked. Nothing is dropped silently:
ambiguity, contradiction and missing detail all become questions.

Once the frontier is empty it stops asking and authors `02-business.md`: Context
& Goals, Stakeholders, Glossary, **Assumptions Ledger**, Non-Goals, and the
atomic BRs sorted `must` → `should` → `could`, each traced to `[V §x]` and
`[CL-n]`.

## Design decisions worth knowing

- **Facts are the agent's job, decisions are yours.** Anything discoverable from
  the filesystem, tools or a sub-agent is looked up, never asked. Judgement
  calls and domain knowledge are put to you, and they wait for your answer.
- **The Clarification Log is append-only.** A `continue` re-poses the questions
  of the last `(awaiting)` entry and resumes exactly there — it does not restart
  the interview.
- **Relayed and verbatim are kept apart.** Your own words are recorded as
  `> CUSTOMER (relay): …`; an exact customer quote is `> VENDOR (verbatim): …`
  and is never edited. That distinction is what keeps the record honest about
  what was actually said.
- **Our wording, not the customer's.** BRs are rewritten into one unambiguous
  atomic concern each, then cross-checked for internal contradictions. A
  contradiction is resolved by asking or marked unresolved — never quietly
  dropped.
- **"Enough for now" still produces a document.** With a non-empty frontier, the
  missing items move into the Assumptions Ledger as `(unresolved)` and the BR is
  authored anyway.
- **Every BR cites a source.** Anything unsourced is either given one, turned
  into a question, or listed in the Assumptions Ledger.

## Install

Not installed on its own; it ships with the requirements pipeline. It calls
[grilling], which ships with the same install.

```bash
npx skills add gothchibjo/skills -g -a opencode
```

See [docs/installation.md] for the full matrix.

## Troubleshooting

| Symptom                               | Fix                                                                             |
| :------------------------------------ | :------------------------------------------------------------------------------ |
| It asks a question you can look up    | Say so — the skill is told facts are its job, and it will dispatch a sub-agent. |
| It stopped with `(awaiting)` entries  | Send the answers; the next round resumes from there.                            |
| BR is marked `proposed`, not approved | Expected. Approval is your explicit action, never inferred.                     |
| An `(unresolved)` item blocks freeze  | Resolve it in the Assumptions Ledger, or accept that freeze will refuse.        |

## License

MIT. See [LICENSE].

<!-- refs -->

[req-workflow]: ../req-workflow/
[grilling]: ../../grilling/
[docs/installation.md]: ../../../docs/installation.md
[LICENSE]: ../../../LICENSE
