---
name: req-guards
description:
  "Guard-conditions gate (REQ-012) of the frozen-contract pipeline: check a REQ's
  guard_conditions before implementation, log each check, block on violations,
  resume only when clean, and route scope deviations to change control. Phase of
  req-develop; load only via the Skill tool from req-develop."
disable-model-invocation: true
metadata:
  protected: "true"
  copyright: "© 2026 github.com/gothchibjo"
---

# Phase: req-guards

Called by req-develop with the event (`implement` | `resume`) and the target PCR
plus the REQ ids in scope. The cost discipline from req-develop applies.

## Inputs

Read only what the check needs:

- `04-plan.md` front matter (`status`);
- the REQ block for each REQ in scope, from `03-technical.yaml` — grep
  `id: REQ-0nn` and read to the next `id:`, then take its `guard_conditions`;
- the `## Guard Log` table, to find which guards failed before;
- for the pipeline flag in step 4: the full REQ id list
  (`grep -n 'id: REQ-' 03-technical.yaml`) and the latest log row per REQ.

Nothing else from the plan body or the contract.

## Event: implement — guard pre-check

1. Required precondition: `04-plan.md` `status` is `plan_approved` (Plan
   Approval gate first — REQ-011 before REQ-012). If the plan is unapproved,
   refuse: the plan must be approved before any implementation step.
2. For each REQ in scope, take its `guard_conditions` from the frozen YAML and
   check each one in the repo/workspace right now. Several REQs in one event are
   checked in one pass.
3. Append every check to the Guard Log in `04-plan.md` — one row per check, in
   the table under `## Guard Log`:
   ```markdown
   | date | REQ | guard | check | result (pass/fail) |
   ```
   None fails → set `04-plan.md` `status: implementing` and start implementing.
   Any fails → set `status: blocked`, refuse to start, and return every failing
   guard with its check evidence. Do NOT flip any gate off a partial check.
4. The pipeline flag `guard_conditions.passed` is true only when **every** REQ
   in the contract carries a latest Guard Log result of `pass`. Compute it from
   the id list and the log, not from the REQs in scope: a REQ that is unchecked,
   or whose latest row is `fail`, keeps the flag `false` no matter how clean the
   REQs in this event were.

Do not print the REQ text or the guard list back; the log row is the record.

## Event: resume — re-check after a block

Re-run the checks for the guards that failed, and **append the re-check rows**:
the log is append-only, and the last row for each guard must reflect the current
verdict. Still failing → keep `status: blocked`, set
`pipeline_gates.guard_conditions.passed: false`, and restate what is unresolved;
do not start. Clean → set `status: implementing`, re-evaluate the pipeline flag
by step 4, and return the cleared block to req-develop.

## Change control (REQ-015)

Any need to alter a frozen REQ text or skip a guard to "get going" is a scope
change: refuse the deviation, do not update the YAML or the plan, and route the
request to req-workflow (`new`) to register a change-request PCR.
