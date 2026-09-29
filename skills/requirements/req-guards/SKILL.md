---
name: req-guards
description:
  "Guard-conditions gate (REQ-012) of the frozen-contract pipeline: before any
  implementation step, check the REQ's guard_conditions from 03-technical.yaml,
  record each check in the pipeline's guard log, block on violations, and resume
  only when the failing guards are clean. Also routes scope deviations from
  frozen REQ texts to change control (REQ-015). Phase of the req-develop state
  machine; load only via the Skill tool from req-develop."
disable-model-invocation: true
metadata:
  protected: "true"
  copyright: "© 2026 github.com/gothchibjo"
---

# Phase: req-guards

Called by req-develop with the event (`implement` | `resume`) and the target
PCR.

## Inputs

Read `03-technical.yaml` (frozen), `04-plan.md` (plan + guard log), and the
planned REQ to work on.

## Event: implement — guard pre-check

1. Required precondition: `04-plan.md` `status` is `plan_approved` (Plan
   Approval gate first — REQ-011 before REQ-012). If the plan is unapproved,
   refuse: the plan must be approved before any implementation step.
2. For the REQ to be worked on, take its `guard_conditions` from the frozen YAML
   and check each one in the repo/workspace right now.
3. Append every check to the Guard Log in `04-plan.md`:
   ```markdown
   | date | REQ | guard | check | result (pass/fail) |
   ```
   None fails → announce clean and start implementing; set `04-plan.md`
   `status: implementing`. Any fails → set `status: blocked`, refuse to start,
   and list every failing guard with its check evidence. Do NOT flip any gate.
4. If all guard checks for the pipeline are clean at this point, set
   `pipeline_gates.guard_conditions.passed: true`. Whenever any guard is
   failing, ensure it is `false`.

## Event: resume — re-check after a block

Re-run the checks for the failed guards. Still failing → keep `status: blocked`
and restate what is unresolved; do not start. Clean → set `status: implementing`
and report the blocked period was cleared.

## Change control (REQ-015)

Any need to alter a frozen REQ text or skip a guard to "get going" is a scope
change: refuse the deviation, do not update the YAML or the plan, and route the
request to req-workflow (`new`) to register a change-request PCR.
