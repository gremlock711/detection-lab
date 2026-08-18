# Access Reconciliation — Excel practice exercise

Synthetic roster + system access list. Built to practice
reconciliation logic and Excel tooling, not a real audit.

## Method
- XLOOKUP from access list -> roster, dragged down
- #N/A = no matching employee record (ghost)
- returns a value but fields disagree = orphaned / transfer / status drift
- ran it the other direction too, roster -> access
- one direction finds ghosts, the other finds unprovisioned employees
- conceptually a LEFT JOIN with a null check

## Found
- 2 ghosts: PRM1099, PRM1088 (roster ends at PRM1024)
- 3 orphaned: PRM1003, PRM1006, PRM1017 terminated, access still active
- 1 transfer gap: PRM1014 roster says 4102, access says 5203
- 3 unprovisioned: PRM1021, 1022, 1023 (all hired Jul 2026)
- 1 pending never completed: PRM1018
- status drift: Actve, ACTIVE, pendng, Pending, Deactivated

## What tripped me up
- exercise was seeded so an incomplete lookup range flags a real
  employee as a ghost. verify record counts across both sources first.
- also caught myself about to report a duplicate record that
  isn't in the data. same failure, one level up.
