# Runbook: High CPU on a host

## Alert
Zabbix trigger: `High CPU on {HOST.NAME}`
Expression: `avg(/linux-client/system.cpu.util,1m)>5`
Note: the threshold is 5% only because the lab VM is throttled and reaches about 15-17% under load. A production threshold would be much higher (for example 80-90%).

## Severity
Match the severity set on the trigger in Zabbix (Warning in a typical setup).

## Impact
The host is slower than normal. Services on it may respond slowly or time out if CPU stays high.

## Diagnose
Run these on the affected host (over SSH):

    top -o %CPU
    ps aux --sort=-%cpu | head -10
    uptime

Check for:
- One process using most of the CPU (runaway job, stress test, stuck service)
- A recent deployment or scheduled job
- Load average well above the number of CPU cores

In Zabbix, open Monitoring > Hosts > Graphs for the host and check when CPU started rising.

## Resolve
1. If it is an expected job, wait for it to finish.
2. If it is a runaway process, stop it politely first: `kill <PID>`. Use `kill -9 <PID>` only if it does not exit.
3. If a service is misbehaving, restart it: `sudo systemctl restart <service>`.
4. If caused by a test, stop it (for example `pkill stress-ng`).

## Escalate if
- CPU stays high for more than 15 minutes after the steps above
- The offending process is a critical service you cannot restart
- The host becomes unresponsive

## Verify
- `top` shows CPU back to normal
- The Zabbix problem moves to Resolved (it can take a few minutes)
- The host's CPU graph returns to its usual level

## Example from this lab
`stress-ng --cpu 1` on linux-client raised the alert. Stopping it cleared the problem after about 5 minutes.
