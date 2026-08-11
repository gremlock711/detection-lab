# Detection Lab

Home lab for endpoint detection engineering: Sysmon telemetry on
Windows 11, shipped to Wazuh on Ubuntu, exercised with Atomic Red Team.

Status: WIP

## Topology

+-- DESKTOP HOST (Windows 11, 32 GB) -------------+
| |
| VMware Workstation Pro |
| |
| +--------------+ +--------------+ |
| | win11-victim | | ubuntu-wazuh | |
| | 10.10.10.20 |------->| 10.10.10.10 | |
| | 8 GB / 4 CPU | logs | 8 GB / 4 CPU | |
| +--------------+ +--------------+ |
| +---- VMnet2 (host-only) ----+ |
| |
+-------------------------------------------------+


Log path: Sysmon -> Windows Event Log -> Wazuh agent -> TCP 1514
-> Wazuh manager (decoders + rules) -> indexer -> dashboard (443)

## Layout

- `rules/` detection rules
- `testing/` atomics run, what fired, what didn't
- `notes/` tuning decisions and FP analysis
- `access-audit/` Excel access reconciliation writeup

## Build log

See `notes/build-log.md`
