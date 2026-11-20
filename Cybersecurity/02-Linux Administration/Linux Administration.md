# Linux Administration

> My notes on the Linux administration concepts I use while operating my cybersecurity lab.

## 1. Processes

- A **process** is a running instance of a program.
- Each process has a Process ID (**PID**).
- A process uses system resources such as CPU, memory, files and network sockets.
- A service may run one or more processes; a service and a process are not the same thing.

### Commands I use

```bash
ps aux
ps aux --sort=-%mem | head
top
htop
```

- `ps aux`: list running processes.
- `top` / `htop`: monitor processes and resource usage interactively.
- `ps aux --sort=-%mem | head`: quickly find processes using the most memory.

## 2. systemd and services

- `systemd` is the init system and service manager used by Ubuntu.
- It starts system services during boot and manages their lifecycle.
- A service is commonly described by a systemd unit, such as `wazuh-manager.service`.

| Command | Purpose |
|---|---|
| `systemctl status SERVICE` | Show service state and recent logs |
| `systemctl start SERVICE` | Start a stopped service |
| `systemctl stop SERVICE` | Stop a running service |
| `systemctl restart SERVICE` | Stop and start the service again |
| `systemctl reload SERVICE` | Ask the service to reload configuration, if supported |
| `systemctl enable SERVICE` | Enable automatic startup at boot |
| `systemctl disable SERVICE` | Disable automatic startup at boot |
| `systemctl enable --now SERVICE` | Enable and start immediately |
| `systemctl is-active SERVICE` | Check whether it is currently active |
| `systemctl is-enabled SERVICE` | Check whether it is enabled at boot |

Example:

```bash
sudo systemctl status wazuh-manager
sudo systemctl restart wazuh-manager
```

**Important:** `enable` does not necessarily start a service immediately. `start` does not necessarily enable it for future boots.

## 3. Listening ports and sockets

A process can listen for incoming network connections on a port.

```bash
sudo ss -tulpn
sudo ss -lntup
sudo lsof -i -P -n
```

Useful `ss` options:

- `-t`: TCP sockets.
- `-u`: UDP sockets.
- `-l`: listening sockets.
- `-p`: show associated process information.
- `-n`: show numeric addresses and ports.

A listening port does not automatically mean that the service is reachable remotely. Firewall rules, routing and network configuration also matter.

## 4. Linux firewall — UFW

- **UFW (Uncomplicated Firewall)** is a front end for managing Linux firewall rules.
- It can control incoming and outgoing traffic.
- Rules can consider protocol, port, source/destination address and interface.

```bash
sudo ufw status verbose
sudo ufw allow 443/tcp
sudo ufw deny 22/tcp
sudo ufw allow out 53/udp
```

My lab has separate firewall boundaries:

- Ubuntu host firewall.
- Wazuh VM firewall.
- Other VM firewalls, if configured.

A rule on the host does not automatically configure the firewall inside a guest VM.

## 5. Logs and troubleshooting

```bash
journalctl -u wazuh-manager
journalctl -u wazuh-manager -f
journalctl -b
```

- `journalctl`: query logs collected by the systemd journal.
- `-u`: filter by unit/service.
- `-f`: follow new log entries.
- `-b`: show logs from the current boot.

## 6. What I have practised

- Checked Linux processes and resource usage while running virtual machines.
- Inspected systemd service states.
- Checked listening ports with `ss`.
- Worked with UFW on the Wazuh VM and Ubuntu host.
- Checked Wazuh services during lab validation.

## 7. What I still need to practise

- Troubleshooting a service that repeatedly fails to start.
- Following a service's logs and identifying the root cause.
- Understanding process trees and parent/child relationships on Linux.
- Writing firewall rules safely and verifying their effect.

## Command reference

- [Linux command reference](../../Commands/01%20-%20Linux%20Commands.md)
- [Wazuh operations command reference](../../Commands/06%20-%20Wazuh%20Operations%20Commands.md)

## Related notes

- [Virtual Networking](../01-Infrastructure/Virtualization/05%20-%20Virtual%20Networking.md)
- [Host Inventory & Lab Setup](../01-Infrastructure/Host%20Inventory%20%26%20Lab%20Setup.md)
- [Wazuh Architecture](../04-Security%20Monitoring/Wazuh%20Architecture.md)
- [Wazuh SOC Lab](../04-Security%20Monitoring/Wazuh%20SOC%20Lab.md)

## Official resources

- [Ubuntu Server documentation](https://ubuntu.com/server/docs)
- [systemd documentation](https://www.freedesktop.org/wiki/Software/systemd/)
- [Ubuntu UFW documentation](https://help.ubuntu.com/community/UFW)
