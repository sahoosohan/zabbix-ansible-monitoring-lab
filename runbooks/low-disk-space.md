# Runbook: Low disk space

## Alert
Zabbix trigger from "Linux by Zabbix agent", for example "Free disk space is less than 20% on volume /". Check the exact name under Monitoring > Problems.

## Severity
Match the severity set on the trigger in Zabbix.

## Impact
A full disk can stop services, break logging, and prevent updates or database writes.

## Diagnose
Run on the affected host:

    df -h
    sudo du -xh / --max-depth=2 2>/dev/null | sort -rh | head -15
    sudo journalctl --disk-usage

Look for large log files, old package caches, or a runaway application writing data.

## Resolve
1. Clean the package cache: `sudo apt clean`
2. Remove unused packages: `sudo apt autoremove`
3. Shrink old journal logs: `sudo journalctl --vacuum-time=7d`
4. Delete or archive large files you identified (check what they are first).
5. If the disk is simply too small, plan to extend it.

## Escalate if
- Free space does not recover after cleanup
- The large files belong to a database or critical application
- The disk fills again within hours

## Verify
- `df -h` shows enough free space
- The Zabbix problem moves to Resolved

## Example from this lab
Not triggered in this lab. This runbook is based on the standard template trigger.
