# Runbook: High memory usage

## Alert
Zabbix trigger from "Linux by Zabbix agent", for example "High memory utilization". Check the exact name under Monitoring > Problems.

## Severity
Match the severity set on the trigger in Zabbix.

## Impact
The host may start swapping and slow down. In the worst case the kernel kills processes (out-of-memory).

## Diagnose
Run on the affected host:

    free -h
    ps aux --sort=-%mem | head -10
    sudo dmesg | grep -i -E 'out of memory|oom' | tail

Look for one process using most of the memory, or memory growing steadily over time (a leak).

## Resolve
1. If it is an expected job, wait for it to finish.
2. Restart the leaking service: `sudo systemctl restart <service>`.
3. Stop the offending process: `kill <PID>`.
4. If memory is simply too small for the workload, plan to add RAM.

## Escalate if
- Memory stays high after the service restart
- The OOM killer is stopping important processes
- The host is swapping heavily or unresponsive

## Verify
- `free -h` shows normal available memory
- The Zabbix problem moves to Resolved

## Example from this lab
Not triggered in this lab. The lab VMs have 1-2 GB of RAM, so this alert is realistic there.
