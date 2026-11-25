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
plus the REQ ids in scope. Cost rules P1–P6 from req-develop apply.

## Inputs

Read only what the check needs:

- `04-plan.md` front matter (`status`);
- the REQ block for each REQ in scope, from `03-technical.yaml` — grep the id
  and read its `guard_conditions`, not the whole contract;
- the Guard Log table, to avoid double-logging the same check.

Nothing else from the plan body or the contract.

## Event: implement — guard pre-check

1. Required precondition: `04-plan.md` `status` is `plan_approved` (Plan
   Approval gate first — REQ-011 before REQ-012). If the plan is unapproved,
   refuse: the plan must be approved before any implementation step.
2. For each REQ in scope, take its `guard_conditions` from the frozen YAML and
   check each one in the repo/workspace right now. Several REQs in one event are
   checked in one pass.
3. Append every check to the Guard Log in `04-plan.md`:
   ```markdown
   | date | REQ | guard | check | result (pass/fail) |
   ```
   None fails → announce clean and start implementing; set `04-plan.md`
   `status: implementing`. Any fails → set `status: blocked`, refuse to start,
   and list every failing guard with its check evidence. Do NOT flip any gate.
4. If all guard checks for the pipeline are clean at this point, edit
   `pipeline_gates.guard_conditions.passed: true`. Whenever any guard is
   failing, ensure it is `false`.

Do not print the REQ text or the guard list back; the log row is the record.

## Event: resume — re-check after a block

Re-run the checks for the failed guards only. Still failing → keep
`status: blocked` and restate what is unresolved; do not start. Clean → set
`status: implementing` and report the blocked period was cleared.

## Change control (REQ-015)

Any need to alter a frozen REQ text or skip a guard to "get going" is a scope
change: refuse the deviation, do not update the YAML or the plan, and route the
request to req-workflow (`new`) to register a change-request PCR.
