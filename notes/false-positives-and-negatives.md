# False Positives, False Negatives, and the Tradeoff Between Them

The rules working was the easy part. The interesting part was watching
where detection breaks down — and it breaks in two opposite directions.

## False positive: rule 100100 (Run key persistence)

During a testing session, Microsoft Edge's auto-updater tripped my
Run-key rule 5 times in 22 seconds. All legitimate. Registry Run keys
are written constantly by normal software — installers, updaters,
browsers — so a rule watching that behavior inherits all of that noise.

## False negative: T1055 (process injection)

The MockingJay RWX injection atomic ran successfully — it spawned
Notepad, so the shellcode executed — and my SIEM generated no useful
alert. The technique is specifically built to avoid the API calls
(VirtualAlloc, WriteProcessMemory, CreateRemoteThread) that behavioral
detection watches for. Attack succeeded, detection missed it. That's a
false negative.

## Why 100103 (encoded PowerShell) stayed clean

In the same session, rule 100103 had 0 false positives even though I
ran PowerShell repeatedly. Almost nothing legitimate base64-encodes
its own commands, so the rule only fired on the actual atomic. Same
machine, same uptime as the noisy Run-key rule — the difference is
entirely the base rate of the behavior being watched.

## The tradeoff

False positives and false negatives pull against each other:

- Tighten a rule to cut false positives, and you risk missing real
  variants — more false negatives.
- Loosen it to catch everything, and false positives explode.

My own rules landed on both sides. 100100 was loose enough to catch the
technique but also caught Edge. 100103 was tight and clean but would
miss PowerShell abuse that isn't encoded. There's no setting that wins
both — every detection is a deliberate choice of where to sit on that
line, and it depends on the environment, the threat, and how much
analyst attention there is to spend.

## The bigger picture

Even an idle lab box generates a constant stream of events. A real
environment is that times thousands, all day. That's where alert
fatigue comes from — too many low-signal alerts, and the real one gets
missed. Deciding what to alert on isn't just "can I detect this," it's
"is this detection worth an analyst's attention."

## Note: LSASS (T1003.001) was neither

Worth distinguishing — the LSASS credential-dump wasn't a false
negative. Defender blocked it before it executed, so there was never
anything for the SIEM to detect. That's a defense-in-depth outcome: the
endpoint control caught what a host-based SIEM rule couldn't have. A
false negative means the attack *ran* and I missed it; here it never ran.
