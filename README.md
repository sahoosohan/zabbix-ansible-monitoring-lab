# Linux Infrastructure Monitoring with Zabbix and Ansible

A home-lab project: monitor a Linux VM with Zabbix 7.0, automate the agent setup with Ansible, and document the response to common alerts as runbooks.

## Architecture

    Laptop (VirtualBox, bridged network 192.168.88.0/24)
    |
    +-- zbx-server   192.168.88.50   Ubuntu 24.04
    |     Zabbix 7.0 server + MySQL + Apache + web UI
    |     Ansible control node
    |
    +-- linux-client 192.168.88.51   Ubuntu 24.04
          Zabbix Agent 2 (monitored host, installed by Ansible)

## What is built

- Zabbix server with web UI, monitoring a Linux host with the "Linux by Zabbix agent" template
- Custom trigger `High CPU on {HOST.NAME}`, tested with `stress-ng` (it fired and resolved)
- Dashboard "Infrastructure Overview": host availability, problems, CPU and memory graphs, top hosts
- Ansible playbook (`ansible/install_agent_linux.yml`) that installs and configures Zabbix Agent 2
- Five runbooks in `runbooks/` for common alerts

## Ansible automation

The playbook installs the Zabbix repository package and `zabbix-agent2`, sets `Server`, `ServerActive` and `Hostname` in the agent config, restarts the agent when the config changes, and enables it on boot.

Verified by reverting `linux-client` to a clean snapshot with no agent:

- First run: `ok=6 changed=4 failed=0`, and the host turned green in Zabbix
- Second run: `changed=0` (idempotent)

Run it from the control node:

    cd ansible
    ansible-playbook -i inventory.ini install_agent_linux.yml

## Runbooks

- [High CPU](runbooks/high-cpu.md)
- [Host unreachable / agent not available](runbooks/host-unreachable.md)
- [Low disk space](runbooks/low-disk-space.md)
- [High memory](runbooks/high-memory.md)
- [Service down](runbooks/service-down.md)

The CPU and host-unreachable runbooks are based on alerts seen in this lab. The other three are based on standard template triggers and were not triggered here.

## Screenshots

See the `screenshots/` folder: dashboard, hosts list, CPU problem (active and resolved), and both Ansible run recaps.

## Honest notes

- The CPU trigger threshold is 5% only because the throttled lab VM reaches about 15-17% under load. A real environment would use a much higher value.
- `admin1` has passwordless sudo on `linux-client` so Ansible can run unattended. This is a lab shortcut and should not be used on real servers.
- A Windows host is **not** part of this project yet.
- No notification channel (email or Telegram) is configured yet.

## Tools

Zabbix 7.0, Ansible, VirtualBox, Ubuntu Server 24.04, MySQL, Apache.
