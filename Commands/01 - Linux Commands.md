---
tags:
  - cybersecurity/reference
  - linux
  - command-reference
status: maintained
---

# Linux Commands

Commands and shell patterns I have documented while working with Ubuntu, Linux administration and OverTheWire Bandit.

## Process and resource inspection

| Command used / pattern        | What it does                                                                         | General use and useful options                                                                                                                                            | Where I used it                                                                                                                                                                                |
| ----------------------------- | ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ps aux`                      | Displays a snapshot of running processes with user, PID, CPU and memory information. | `ps` reports process state; `a` includes processes associated with terminals, `u` uses a user-oriented format, and `x` includes processes without a controlling terminal. | [Linux Administration — processes](../Cybersecurity/02-Linux%20Administration/Linux%20Administration.md#1-processes)                                                                                  |
| `ps aux --sort=-%mem \| head` | Sorts the process list by memory use and shows the first rows.                       | Change the sort key to `-%cpu` for CPU use. `head -n 20` shows a larger sample.                                                                                           | [Linux Administration — process resource checks](../Cybersecurity/02-Linux%20Administration/Linux%20Administration.md#1-processes)                                                                    |
| `top` / `htop`                | Show live process and resource activity.                                             | `top` is commonly available by default; `htop` is an interactive alternative that may need installation.                                                                  | [Linux Administration — process tools](../Cybersecurity/02-Linux%20Administration/Linux%20Administration.md#1-processes)                                                                              |
| `id`                          | Shows the current user's UID, primary GID and supplementary groups.                  | `id USER` inspects another account when permissions allow. Useful for checking membership in groups such as `kvm` or `libvirt`.                                           | [KVM — user group membership](../Cybersecurity/01-Infrastructure/Virtualization/01%20-%20KVM.md#6-user-group-membership) · [Bandit — ownership and groups](../CTF%20%26%20Practice/OverTheWire%20Bandit%20Notes.md#bandit-level-6) |

## Services and systemd

| Command used / pattern | What it does | General use and useful options | Where I used it |
|---|---|---|---|
| `systemctl status SERVICE` | Shows a service's current state and recent status information. | Use `systemctl status SERVICE_NAME`; `--no-pager` avoids opening a pager. | [Linux Administration — systemd](../Cybersecurity/02-Linux%20Administration/Linux%20Administration.md#2-systemd-and-services) |
| `systemctl start SERVICE` / `stop SERVICE` | Starts or stops a service for the current runtime. | `start` and `stop` affect current state; they do not by themselves change boot-time enablement. | [Linux Administration — service lifecycle](../Cybersecurity/02-Linux%20Administration/Linux%20Administration.md#2-systemd-and-services) |
| `systemctl restart SERVICE` / `reload SERVICE` | Restarts a service or asks it to reload supported configuration. | `reload` is only meaningful if that service supports reload; otherwise use the documented restart procedure. | [Linux Administration — restart and reload](../Cybersecurity/02-Linux%20Administration/Linux%20Administration.md#2-systemd-and-services) |
| `systemctl enable SERVICE` / `disable SERVICE` | Controls whether a service is enabled to start automatically at boot. | `enable --now SERVICE` enables and starts it; `disable --now SERVICE` disables and stops it. | [Linux Administration — boot enablement](../Cybersecurity/02-Linux%20Administration/Linux%20Administration.md#2-systemd-and-services) |
| `systemctl is-active SERVICE` / `is-enabled SERVICE` | Returns the active runtime state or boot enablement state. | Useful in scripts and quick checks; `active` and `enabled` describe different things. | [Linux Administration — service-state checks](../Cybersecurity/02-Linux%20Administration/Linux%20Administration.md#2-systemd-and-services) · [Host inventory — libvirtd check](../Cybersecurity/01-Infrastructure/Host%20Inventory%20%26%20Lab%20Setup.md#virtualization-readiness-checks-i-performed) |
| `systemctl list-units --type=service --all 'virt*' 'libvirt*'` | Lists matching service units, including inactive units. | Useful when identifying whether a host uses the legacy `libvirtd` daemon or modular libvirt services. | [libvirt — daemon inspection](../Cybersecurity/01-Infrastructure/Virtualization/03%20-%20libvirt.md#my-connection-and-service-checks) |

## Listening sockets and open files

| Command used / pattern | What it does | General use and useful options | Where I used it |
|---|---|---|---|
| `sudo ss -tulpn` | Lists listening TCP/UDP sockets and associated process information. | `-t` TCP, `-u` UDP, `-l` listening, `-p` process, `-n` numeric output. Root privileges may be needed to show process details for all sockets. | [Linux Administration — listening sockets](../Cybersecurity/02-Linux%20Administration/Linux%20Administration.md#3-listening-ports-and-sockets) |
| `sudo ss -lntup` | Lists listening TCP and UDP sockets with numeric addresses and process information. | `-a` can include non-listening sockets; `-l` restricts the output to listening sockets. | [Linux Administration — socket inspection](../Cybersecurity/02-Linux%20Administration/Linux%20Administration.md#3-listening-ports-and-sockets) |
| `sudo lsof -i -P -n` | Lists open files associated with network connections. | `-i` selects network files, `-P` keeps port numbers numeric, and `-n` avoids DNS resolution. Add a protocol or port filter to narrow results. | [Linux Administration — lsof network view](../Cybersecurity/02-Linux%20Administration/Linux%20Administration.md#3-listening-ports-and-sockets) |

## Firewall command patterns

> These commands are documented as UFW examples in my Linux notes. Their presence here does not assert that these exact rules are currently active on my host or VMs.

| Command used / pattern | What it does | General use and useful options | Where I used it |
|---|---|---|---|
| `sudo ufw status verbose` | Shows UFW status and configured rules with additional detail. | `sudo ufw status numbered` displays numbered rules that can be referenced when managing them. | [Linux Administration — UFW](../Cybersecurity/02-Linux%20Administration/Linux%20Administration.md#4-linux-firewall-ufw) |
| `sudo ufw allow 443/tcp` | Adds a rule allowing inbound TCP traffic to port 443. | Restrict by source when appropriate, for example `ufw allow from SOURCE to any port PORT proto tcp`. Review the rule before exposing a service. | [Linux Administration — UFW allow example](../Cybersecurity/02-Linux%20Administration/Linux%20Administration.md#4-linux-firewall-ufw) |
| `sudo ufw deny 22/tcp` | Adds a rule denying inbound TCP traffic to port 22. | Firewall rules can lock out remote administration. Confirm console access and rule order before changing SSH access. | [Linux Administration — UFW deny example](../Cybersecurity/02-Linux%20Administration/Linux%20Administration.md#4-linux-firewall-ufw) |
| `sudo ufw allow out 53/udp` | Adds an outbound UDP rule for port 53, commonly used for DNS. | UFW can distinguish inbound (`in`) and outbound (`out`) direction and protocol. | [Linux Administration — UFW outbound example](../Cybersecurity/02-Linux%20Administration/Linux%20Administration.md#4-linux-firewall-ufw) |

## Logs and troubleshooting

| Command used / pattern | What it does | General use and useful options | Where I used it |
|---|---|---|---|
| `journalctl -u SERVICE` | Reads journal entries associated with a systemd unit. | Add `-b` for the current boot, `--since` / `--until` for a time window, and `-n 100` for recent lines. | [Linux Administration — journal logs](../Cybersecurity/02-Linux%20Administration/Linux%20Administration.md#5-logs-and-troubleshooting) |
| `journalctl -u SERVICE -f` | Follows new journal entries for a unit in real time. | Similar to `tail -f`; press `Ctrl+C` to stop following. | [Linux Administration — live service logs](../Cybersecurity/02-Linux%20Administration/Linux%20Administration.md#5-logs-and-troubleshooting) |
| `journalctl -b` | Shows journal entries from the current boot. | Use `-b -1` to inspect the previous boot when persistent journal data is available. | [Linux Administration — boot troubleshooting](../Cybersecurity/02-Linux%20Administration/Linux%20Administration.md#5-logs-and-troubleshooting) |

## File, text and shell utilities

| Command used / pattern | What it does | General use and useful options | Where I used it |
|---|---|---|---|
| `ls -a` / `ls -la` / `ls -l` | Lists directory contents, including hidden entries (`-a`) and detailed metadata (`-l`). | `-h` makes sizes human-readable when combined with long listing; `-la` combines hidden and detailed output. | [Bandit — hidden files](../CTF%20%26%20Practice/OverTheWire%20Bandit%20Notes.md#bandit-level-3) · [Bandit — permissions and ownership](../CTF%20%26%20Practice/OverTheWire%20Bandit%20Notes.md#bandit-level-6) |
| `cat -- "filename"` | Prints file contents; `--` marks the end of options so a filename beginning with `-` is treated as a filename. | Quote paths containing spaces. For large files, use `less`; for binary content, use `file` first rather than assuming it is text. | [Bandit — filename beginning with a dash](../CTF%20%26%20Practice/OverTheWire%20Bandit%20Notes.md#bandit-level-1) · [Bandit — inspect candidate files](../CTF%20%26%20Practice/OverTheWire%20Bandit%20Notes.md#bandit-level-4) · [Bandit — read matching file](../CTF%20%26%20Practice/OverTheWire%20Bandit%20Notes.md#bandit-level-5) |
| `file ./inhere/*` | Identifies file types using file signatures and content heuristics. | Useful when an extension is missing or misleading. It does not guarantee that a file is safe to open. | [Bandit — identify the human-readable file](../CTF%20%26%20Practice/OverTheWire%20Bandit%20Notes.md#bandit-level-4) |
| `find . -type f -size 1033c ! -executable` | Searches recursively for regular files exactly 1033 bytes long that are not executable by the current user. | `-name` filters by filename, `-user` / `-group` by ownership, `-mtime` by modification time, `-maxdepth` limits recursion, and `-exec` runs a command on matches. | [Bandit — search by size and execute permission](../CTF%20%26%20Practice/OverTheWire%20Bandit%20Notes.md#bandit-level-5) |
| `find / -type f -size 33c -user bandit7 -group bandit6 2>/dev/null` | Searches from `/` for files matching type, size, user and group while redirecting standard error to `/dev/null`. | `2>/dev/null` hides error output; it can also hide useful diagnostic information, so use it only when appropriate. | [Bandit — find file by ownership and size](../CTF%20%26%20Practice/OverTheWire%20Bandit%20Notes.md#bandit-level-6) |
| `for file in ./*; do ...; done` | Iterates over entries matched by a shell glob. | Quote variable expansions such as `"$file"`; check `-f` when the loop should process regular files only. | [Bandit — loop over candidate files](../CTF%20%26%20Practice/OverTheWire%20Bandit%20Notes.md#bandit-level-4) |
| `for file in *\ *; do mv "$file" "${file// /_}"; done` | Renames names containing spaces by replacing spaces with underscores. | This is a Bash parameter-expansion example; preview filenames first and make sure the target names will not collide. | [Bandit — rename filenames containing spaces](../CTF%20%26%20Practice/OverTheWire%20Bandit%20Notes.md#bandit-level-2) |
| `2>/dev/null` | Redirects standard error (file descriptor 2) to `/dev/null`. | `>file` redirects standard output; `>file 2>&1` redirects both standard output and standard error to the same destination. Redirection order matters. | [Bandit — standard streams and redirection](../CTF%20%26%20Practice/OverTheWire%20Bandit%20Notes.md#bandit-level-6) |
| `ssh user@host -p PORT` | Opens an encrypted SSH session to a remote host and TCP port. | `-p` specifies the port; `-i KEY_FILE` selects a private key. Keep private keys and passwords out of notes and screenshots. | [Bandit — SSH fundamentals](../CTF%20%26%20Practice/OverTheWire%20Bandit%20Notes.md#what-is-ssh-secure-shell-) · [Bandit — level connection examples](../CTF%20%26%20Practice/OverTheWire%20Bandit%20Notes.md#bandit-level-0) |

## Wazuh-related Linux commands

For Wazuh-specific operations, see [06 - Wazuh Operations Commands](06%20-%20Wazuh%20Operations%20Commands.md). The general `systemctl` and `journalctl` syntax remains covered in the sections above.

## Related notes

- [Linux Administration](../Cybersecurity/02-Linux%20Administration/Linux%20Administration.md)
- [OverTheWire Bandit](../CTF%20%26%20Practice/OverTheWire%20Bandit%20Notes.md)
- [Host Inventory & Lab Setup](../Cybersecurity/01-Infrastructure/Host%20Inventory%20%26%20Lab%20Setup.md)


## File and text inspection used during Wazuh troubleshooting

| Command used / pattern | What it does | General use and useful options | Where I used it |
|---|---|---|---|
| `sudo cp SOURCE DESTINATION` | Copies a file, used here to preserve the original Wazuh Manager/Filebeat configuration before editing. | Keep the backup path explicit. A backup is useful only if I know which original file it represents and can restore it deliberately. | [Project 01 — archive configuration backup](../Projects/Project%2001%20-%20Windows%20Endpoint%20Detection%20%26%20Investigation/07%20-%20Wazuh%20Raw%20Archive%20Collection%20and%20Indexing.md#1-enable-json-archives-on-the-manager) |
| `grep -n 'PATTERN' FILE` | Searches text and prints matching line numbers. | `-F` treats the pattern as fixed text; `-E` enables extended regular expressions; `-i` ignores case; `-A N` / `-B N` show context lines. | [Project 01 — inspect Filebeat configuration](../Projects/Project%2001%20-%20Windows%20Endpoint%20Detection%20%26%20Investigation/07%20-%20Wazuh%20Raw%20Archive%20Collection%20and%20Indexing.md#2-enable-archive-indexing-in-filebeat) |
| `tail -n 40 FILE` | Prints the last 40 lines of a file. | Change the number after `-n`; `tail -f FILE` follows appended log lines. Useful for a quick recent-log sample, not a substitute for filtering by time or component. | [Project 01 — inspect Filebeat application logs](../Projects/Project%2001%20-%20Windows%20Endpoint%20Detection%20%26%20Investigation/07%20-%20Wazuh%20Raw%20Archive%20Collection%20and%20Indexing.md#how-i-verified-each-stage) |
| `find PATH -maxdepth 2 -type f -printf '%p (%s bytes)\n'` | Lists files under a path, limited to two directory levels, and prints each path and size. | `-type f` selects regular files; `-maxdepth` limits traversal; `-printf` formats output (GNU `find`). | [Project 01 — inspect Filebeat registry files](../Projects/Project%2001%20-%20Windows%20Endpoint%20Detection%20%26%20Investigation/07%20-%20Wazuh%20Raw%20Archive%20Collection%20and%20Indexing.md#how-i-verified-each-stage) |
