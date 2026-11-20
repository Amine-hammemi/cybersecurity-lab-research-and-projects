# Portfolio Evidence Plan

This checklist identifies useful visual evidence to add to the repository. Capture each item only after reproducing the state yourself. Do not create screenshots just to make the folder look fuller.

## Priority 1 — Core lab proof

| Suggested filename | Capture | What it demonstrates |
|---|---|---|
| `01-virt-manager-overview.png` | virt-manager showing the lab VMs and their configured state | The virtualization environment exists |
| `02-libvirt-network-xml.png` | `virsh net-dumpxml isolated-lab` output | The isolated network definition, bridge and DHCP range |
| `03-vm-nic-attachments.png` | `virsh domiflist Kali`, `virsh domiflist wazuh`, and `virsh domiflist win11` output | Each VM has only the `isolated-lab` NIC attached |
| `04-wazuh-agent-active.png` | Wazuh Dashboard agent list showing the Windows endpoint as active | Endpoint enrollment and recent connectivity |
| `05-sysmon-eventviewer.png` | Event Viewer, Sysmon Operational channel, with a clearly identified Event ID | Local Sysmon telemetry generation |
| `06-wazuh-alert-details.png` | One Wazuh alert expanded to show timestamp, rule, agent and event fields | Alert triage and field interpretation |

## Priority 2 — Investigation evidence

| Suggested filename | Capture | What it demonstrates |
|---|---|---|
| `07-process-tree.png` | Process Explorer or PowerShell output showing a parent-child process relationship | Windows process investigation |
| `08-event-correlation.png` | Related Sysmon events around one controlled action | Timeline construction and event correlation |
| `09-wireshark-arp-icmp.png` | Wireshark capture filtered with `arp || icmp` | Packet-level network analysis |
| `10-wazuh-investigation-timeline.png` | A concise timeline built from related endpoint events | An investigation workflow, not just dashboard browsing |

## Screenshot standards

- Capture the actual application or terminal; do not recreate output in an image editor.
- Include enough context to identify the VM, tool, event channel or command.
- Crop unrelated desktop content.
- Hide credentials, tokens, email addresses, personal usernames and unrelated host details.
- Keep the original full-resolution image where practical.
- Add a short caption in the relevant Markdown note explaining the observation and its significance.

## Where to place evidence

Use the `Evidence/` directory at the repository root for captured images. The example below is a future embed path; add the screenshot after you have captured and reviewed it:

```markdown
![Wazuh agent status](Evidence/04-wazuh-agent-active.png)

*The Windows 11 agent was shown as active at the time of capture.*
```

Adjust the relative path based on the note's folder depth. In Obsidian, use a wikilink embed if preferred.

## Evidence integrity

A screenshot proves only what is visible at the time it was captured. It does not, by itself, prove long-term availability, correct detection, or independent mastery. Pair important screenshots with the command, configuration excerpt, event fields or repeatable steps that produced them.
