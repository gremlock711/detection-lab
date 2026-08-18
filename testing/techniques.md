# Chosen techniques — locked Aug 18

| ID | Name | Sysmon EID | Day |
|----|------|-----------|-----|
| T1547.001 | Run Key persistence | 13 | Wed 19 |
| T1053.005 | Scheduled Task | 1 | Fri 21 |
| T1112 | Modify Registry | 13 | Fri 21 |
| T1059.001 | PowerShell | 1 | Fri 21 |
| T1003.001 | LSASS memory access | 10 | Sat 22 |
| T1055 | Process Injection | 8/10 | Sat 22 |
| T1071.001 | Web protocol C2 | 3 | Sat 22 |
| T1078 | Valid Accounts | 4624/4625 (Security) | Mon 24 |

Eight. Not nine.

Per technique: run atomic -> find telemetry -> write rule ->
trigger it -> tune FPs -> write up what fired and what didn't.
