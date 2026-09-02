# Build Log — what broke and how I fixed it

The lab came together, but not cleanly. These are the problems worth
remembering, roughly in the order I hit them. Most share a theme:
verify each layer before blaming the next one.

## DNS: an install failure that wasn't an install failure

The Wazuh installer died at the dependency step. It looked like a
package problem. It wasn't — `ping 8.8.8.8` worked but
`ping archive.ubuntu.com` failed, so routing was fine and name
resolution was dead. `systemd-resolved` had hung; restarting it
brought resolution back.

Lesson: split the test. Raw IP vs hostname isolates a DNS problem
from a connectivity problem in one step, before you start suspecting
the installer. This came back later — it doesn't survive a reboot
cleanly, so it needed restarting again in Week 2.

## APIPA on the isolated segment (twice)

The dashboard was unreachable from the host. Cause: the host had no
adapter on VMnet2, and when I added one it came up with a 169.254
APIPA address — because I'd deliberately turned DHCP off on that
network. Nothing was handing out addresses, so it self-assigned.
Fix: static-assign the host, `New-NetIPAddress 10.10.10.1`.

Then the exact same symptom on the Windows VM's lab NIC — 169.254
again, same cause, same fix (`10.10.10.20`). Caught it faster the
second time because I recognized the pattern: APIPA on a segment with
no DHCP means "assign this by hand."

## One-way ping: Windows Firewall

Ubuntu could not ping the Windows VM, but Windows could ping Ubuntu.
Windows Firewall blocks inbound ICMP by default — Windows behaving
correctly, not a misconfiguration. I added an allow rule, but scoped
it to 10.10.10.0/24 rather than allowing ICMP from anywhere.

Why scoped: least privilege. The blanket rule costs nothing on a lab
box, but the habit is what transfers to a real one — and a scoped
rule documents its own intent. Anyone reading the firewall knows
exactly what it's for.

## Sysmon running ≠ Sysmon integrated

This was the most useful one. Sysmon installed fine and was writing
events to its own Windows event channel — I confirmed it locally with
`Get-WinEvent`. But nothing showed up in Wazuh.

The Wazuh agent only reads the channels listed in `ossec.conf`, and
Sysmon's channel isn't in the default set. Two working components with
no wire between them. Fix: add a `<localfile>` block pointing at
`Microsoft-Windows-Sysmon/Operational`.

Lesson: a component running is not the same as a component integrated.
I only knew because I checked both ends — local event log AND the
dashboard. If I'd only checked one, I'd have assumed the pipeline
worked when it didn't.

## VMware NAT service hung (Week 2)

Both VMs lost internet after a host reboot. The lab network
(10.10.10.0/24) was completely unaffected — Wazuh kept ingesting, the
agent stayed Active — which is the host-only design working as
intended. The fix was restarting the "VMware NAT Service" on the host.

Worth noting: my first restart attempt targeted a service named
"VMwareNAT", which matches nothing, and failed silently with no error.
The real name has spaces. Lesson: a command returning no error is not
the same as a command that did something — verify the service name
resolves before trusting the restart.

## Correcting myself: not everything was Defender (Week 2)

While running attack atomics, three techniques threw "Access is
denied" from ART's own process launcher (Invoke-Process.ps1). I first
attributed the LSASS block to Defender. But when the same error showed
up on the C2 technique, and `Start-Process` worked where ART's launcher
didn't, it was clear the harness itself was failing to spawn processes
in some cases — not always Defender. I went back and softened the
earlier attribution. Findings get revised when new evidence shows up.

## Open question I didn't chase

A fresh Windows 11 install failed 261 of ~395 CIS benchmark controls
(32%) on the automatic Wazuh scan — I didn't configure that, it ran on
its own. I didn't dig into why out-of-box Windows scores so low against
CIS, but it's a good reminder that "default" and "hardened" are very
different baselines.

![Fresh Windows 11 scoring 32% against CIS benchmark](screenshots/cis-benchmark.png)
