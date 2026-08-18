## firewall scoping — why 10.10.10.0/24 not blanket allow
- least privilege. open what's needed, nothing more
- blanket rule costs nothing on a lab box, but the habit transfers
- scoped rule documents its own intent

## sysmon running but nothing in wazuh
- sysmon was writing to its own event channel fine
- wazuh agent only reads channels listed in ossec.conf
- sysmon's channel isn't in the default list
- component running != component integrated
- verified at BOTH ends: Get-WinEvent locally, then dashboard
