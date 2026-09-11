# ISIC-949 Civic Membership Organization Coordination Actor

Administrative coordination for civic membership organizations (community groups, civic associations, and other membership-based organizations).

## Scope

This actor manages **administrative coordination only**:

- Event/meeting scheduling logistics
- Dues-processing administrative logistics (tracking, reminders)
- Non-content supply coordination (consumables)
- Staff shift scheduling (proposals only, never binding)
- Safety and conduct concerns (escalated to humans)

## Out of Scope (Hard-Coded Blocks)

The actor **cannot and will not** make decisions on:

- Membership-eligibility or expulsion
- Religious doctrine or belief content
- Political positions or advocacy stances
- Dues amounts or financial waivers
- Disciplinary action

## Governor Hard Checks

All proposals are subject to three **permanent, un-overridable** checks:

1. **Member/Record Verification** — Targets must exist and be verified in the store.
2. **Effect Requirement** — All effects must be `:propose` (proposals only, no auto-commits without escalation).
3. **Scope Exclusion** — Proposals mentioning excluded topics are rejected permanently.

## Operations

| Operation | Description | Effect |
|-----------|-------------|--------|
| `:schedule-member-event` | Meeting/event scheduling | `:propose` |
| `:coordinate-dues-processing-logistics` | Dues tracking/reminders | `:propose` |
| `:coordinate-supply-request` | Non-content supply coordination | `:propose` |
| `:schedule-staff-shift-proposal` | Admin shift proposals (never binding) | `:propose` |
| `:flag-safety-concern` | Safety/conduct concerns for human review | `:propose` (escalates) |

## Rollout Phases

| Phase | Operations | Escalation |
|-------|-----------|-----------|
| 0 | Read-only | All escalate |
| 1 | Event scheduling + Dues logistics | All escalate |
| 2 | + Supply + Staff shift | All escalate |
| 3 | Full auto-commit | Safety concerns always escalate |

## Architecture

**All modules are `.cljc`** (portable across Clojure and ClojureScript):

- **store.cljc** — In-memory member/event/account directory + append-only ledger
- **advisor.cljc** — Proposal enrichment (deterministic demo; production uses LLM)
- **governor.cljc** — Three HARD checks (no overrides)
- **operation.cljc** — StateGraph-equivalent flow: intake → advise → govern → decide → commit | hold | escalate
- **phase.cljc** — Phase definitions (0–3)
- **sim.cljc** — 5 demo scenarios
- **test.cljc** — 20 comprehensive test cases

## Tests

Run all tests:

```bash
kbb --backend sci -m civicmembershiporg.test
```

**Test coverage (20 cases)**:

- **Store** (5): member lookup, all members, event lookup, account lookup, ledger append
- **Governor** (7): member unverified, effect not :propose, scope exclusions (membership eligibility, religious doctrine, political position, dues waiver), flag-safety-concern allowed
- **Operation** (5): event scheduling (happy path), unverified member rejection, safety escalation, dues logistics, supply request
- **Phase** (3): phase 0 (read-only), phase 1 (event + dues auto-commit), phase 3 (full auto-commit)

## Demo

Run simulation with 5 scenarios:

```bash
kbb --backend sci -m civicmembershiporg.sim
```

Scenarios:

1. Happy path — event scheduling for verified member
2. Hard check — unverified member blocked
3. Scope exclusion — religious-doctrine decision blocked
4. Scope exclusion — political-position decision blocked
5. Escalation — safety concern always escalates

## License

AGPL-3.0
