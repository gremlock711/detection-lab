# Detection Lab

A home lab for practicing detection engineering: build a SIEM pipeline,
run real attack techniques against it, write detections, and tune out
the false positives.

Built solo over three weeks as a portfolio project. The interesting
part isn't that the rules fire — it's the notes/ folder, where I work
through why they fire, what they miss, and where rule-based detection
hits its limits.

## Topology

    host (Windows 11 desktop) 10.10.10.1
      |
      +-- VMnet2 host-only  10.10.10.0/24  (isolated lab network)
      |
      +----------------------+----------------------+
      |                                             |
    ubuntu-wazuh 10.10.10.10          win11-victim 10.10.10.20
    +--------------------+            +--------------------+
    | Wazuh manager      |            | Wazuh agent        |
    | indexer            |  <--1514---| Sysmon             |
    | dashboard :443     |   events   | (SwiftOnSecurity)  |
    +--------------------+            +--------------------+

    Both VMs: 8 GB RAM / 4 vCPU, VMware Workstation Pro
    Second NIC on each (NAT) used only during build; disconnected
    during attack detonation so the victim is fully isolated.

## Stack

- Ubuntu 22.04 + Wazuh 4.9.2 (manager, indexer, dashboard)
- Windows 11 Enterprise eval + Sysmon (SwiftOnSecurity config)
- Atomic Red Team for attack execution
- MITRE ATT&CK for technique selection and rule mapping

## What's here

    rules/     custom Wazuh detection rules (100100-series)
    testing/   each technique: atomic run, what fired, what didn't
    notes/     tuning decisions, FP analysis, troubleshooting
    excel-access-reconciliation/  separate Excel exercise

## Techniques covered

Eight ATT&CK techniques across six tactics:

| ID | Technique | Tactic | Result |
|----|-----------|--------|--------|
| T1547.001 | Registry Run Key | Persistence | rule 100100 |
| T1053.005 | Scheduled Task | Persistence | rule 100101 |
| T1112 | Modify Registry | Defense Evasion | rule 100102 |
| T1059.001 | PowerShell (encoded) | Execution | rule 100103 |
| T1071.001 | Web Protocol C2 | Command & Control | rule 100104 |
| T1003.001 | LSASS Memory | Credential Access | blocked by Defender |
| T1055 | Process Injection | Defense Evasion | ran undetected |
| T1078 | Valid Accounts | Persistence / Evasion | rule 100105+100106 |

The last two are findings in their own right: one technique the
endpoint control stopped before the SIEM saw anything, and one that
ran successfully while generating no SIEM telemetry at all. Not every
attack is catchable with host event rules, and documenting where the
gaps are matters as much as the detections that work.

## Build notes

See notes/ for the full build log, including what broke:
DNS resolver failures, APIPA on the isolated segment, the
Sysmon-to-Wazuh integration gap, and Wazuh field-matching quirks.
