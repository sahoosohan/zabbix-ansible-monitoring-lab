# Runbook: Service down on a host

## Alert
A Zabbix trigger for a stopped service or failed check (for example a systemd service item or a port check). The template does not include one by default; you create it for the services that matter.

## Severity
Set according to how important the service is.

## Impact
Users or other systems that depend on the service cannot use it, even though the host is up.

## Diagnose
Run on the affected host:

    systemctl status <service>
    sudo journalctl -u <service> -n 50 --no-pager
    ss -tlnp | grep <port>

Look for a failed start, a config error in the logs, a full disk, or a port conflict.

## Resolve
1. Start the service: `sudo systemctl start <service>`.
2. If it fails, read the log lines, fix the cause (config, disk, permissions), and start again.
3. Enable it on boot: `sudo systemctl enable <service>`.

## Escalate if
- The service fails again right after a restart
- The logs point to a problem you cannot fix (data corruption, missing dependency)
- It is a critical service and the outage lasts more than the agreed time

## Verify
- `systemctl status <service>` shows active (running)
- The service responds on its port
- The Zabbix problem moves to Resolved

## Example from this lab
Not triggered in this lab. The closest real case is the Zabbix agent itself stopping, which is covered in host-unreachable.md.
