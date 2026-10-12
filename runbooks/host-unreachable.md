# Runbook: Host unreachable / Zabbix agent not available

## Alert
Zabbix trigger: "Zabbix agent is not available" (from the template "Linux by Zabbix agent"). Check the exact trigger name under Monitoring > Problems.
The host's ZBX icon turns red or grey in Monitoring > Hosts.

## Severity
Match the severity set on the trigger in Zabbix (Average in the default template).

## Impact
Zabbix has no data for the host. Other alerts for that host (CPU, memory, disk) cannot fire while it is unreachable, so problems may go unnoticed.

## Diagnose
Work from the Zabbix server (zbx-server) outward:

1. Is the host reachable on the network?

       ping -c 3 192.168.88.51

2. Is the agent port open?

       nc -zv 192.168.88.51 10050

3. Can you log in over SSH?

       ssh admin1@192.168.88.51

If ping fails, the VM is probably powered off or has no network. Check the VM in VirtualBox.
If ping works but port 10050 is closed, the agent is stopped or a firewall is blocking it.

On the host itself:

    systemctl status zabbix-agent2
    sudo tail -n 30 /var/log/zabbix/zabbix_agent2.log
    sudo grep -E '^(Server|ServerActive|Hostname)=' /etc/zabbix/zabbix_agent2.conf
    sudo ufw status

Check that Server= is the Zabbix server IP (192.168.88.50) and Hostname= matches the host name in the Zabbix UI exactly.

## Resolve
1. VM powered off: start it and wait about a minute.
2. Agent stopped: `sudo systemctl start zabbix-agent2`, and make sure it starts on boot: `sudo systemctl enable zabbix-agent2`.
3. Wrong config: fix Server/ServerActive/Hostname, then `sudo systemctl restart zabbix-agent2`.
4. Agent missing or broken: re-run the Ansible playbook from zbx-server:

       ansible-playbook -i inventory.ini install_agent_linux.yml

5. Firewall blocking: allow port 10050 from the Zabbix server.

## Escalate if
- The host does not respond after a restart
- The VM or machine will not boot
- Several hosts go unreachable at the same time (likely a network or Zabbix server problem)

## Verify
- `ping` and `nc -zv <ip> 10050` succeed
- The ZBX icon in Monitoring > Hosts turns green (can take 1-2 minutes)
- The problem moves to Resolved

## Example from this lab
After linux-client was restarted, Zabbix raised "agent not available" and then recovered on its own once the VM and agent were back. The same alert appeared when the VM was powered off during snapshot work, and re-running the Ansible playbook rebuilt the agent from a clean machine.
