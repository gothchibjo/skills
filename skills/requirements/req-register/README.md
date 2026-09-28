# req-register

Phase 1 of [req-workflow]: open a new PCR record from an incoming vision.

**Not directly invocable.** The frontmatter sets
`disable-model-invocation: true` — the agent will not reach for it, and it is
loaded only through the state machine.

## What it does

- Resolves the project root with `git rev-parse --show-toplevel` and creates
  `requirements/` if it is missing.
- Allocates the next id: highest existing `PCR-<nnnn>` plus one, zero-padded to
  four digits.
- Derives a kebab-case `slug` from the title, ≤ 40 characters.
- Asks for whatever is still unknown in **one batch**: requester, title,
  channel, `artifact_language` (`ru`/`en`) and `source_language`. Nothing
  already supplied is asked twice.
- Writes the record skeleton: PART A verbatim, an empty PART B, and skeletons of
  `02-business.md` and `03-technical.yaml` with the four gates at
  `passed: false`.
- Adds the row to the registry in `requirements/README.md`.

## The split check

If the vision obviously bundles several independent feature clusters, the skill
asks whether to register **one PCR per cluster** — the default. Split by
connectedness, so each PCR stays independently plannable and reviewable. Each
cluster gets its own id, folder and its own verbatim PART A slice. Declining
registers it as one PCR.

## Design decisions worth knowing

- **PART A is immutable and byte-for-byte.** The vision is stored as received,
  defects, contradictions and all — no fixes, no normalization, no inferred
  sections. Everything downstream cites it, so tidying it up here would destroy
  the traceability back to what the customer actually said.
- **`artifact_language` and `source_language` are different things.** Artifacts
  we write (clarification log, BR, YAML) are in our language; the vision itself
  stays in the customer's.
- **The skeleton is complete, not a stub.** `03-technical.yaml` gets its full
  `pipeline_gates` block up front, so every later phase writes into a shape that
  already exists.
- **The title may stay in the customer's language** — the slug is latin
  kebab-case either way.

## Install

Not installed on its own; it ships with the requirements pipeline.

```bash
npx skills add gothchibjo/skills -g -a opencode
```

See [docs/installation.md] for the full matrix.

## Troubleshooting

| Symptom                            | Fix                                                                        |
| :--------------------------------- | :------------------------------------------------------------------------- |
| Wrong id allocated                 | Ids come from directory names. Renaming a folder re-derives the next id.   |
| It registered one PCR for a bundle | That is the default. Decline the split when you want one record instead.   |
| Asking for language twice          | The values were supplied; the skill is told to infer from the vision text. |

## License

MIT. See [LICENSE].

<!-- refs -->

[req-workflow]: ../req-workflow/
[docs/installation.md]: ../../../docs/installation.md
[LICENSE]: ../../../LICENSE
