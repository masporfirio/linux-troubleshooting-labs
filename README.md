# Linux Troubleshooting Labs

This repository contains Linux troubleshooting exercises I use while building practical system administration and IT support skills.

The notes are intentionally junior-level. A page marked **simulated troubleshooting lab** is a practice scenario. A page marked **troubleshooting note from my lab environment** documents a problem that happened in one of my own virtual machines.

## Troubleshooting method

I use the same basic sequence in each lab:

1. Confirm the symptom.
2. Check the current system state.
3. Read the most relevant logs or command output.
4. Change one thing that matches the evidence.
5. Verify the service or system again.

## Labs

| Lab | Type | Main commands |
|---|---|---|
| [Port conflict](labs/port-conflict/) | Simulated lab, completed with screenshots | `ss`, `fuser`, `ps`, `kill` |
| [Service failure with systemd](labs/service-failure-systemd/) | Simulated lab, completed with screenshots | `systemctl`, `journalctl`, `ss`, `curl` |
| [systemd service with a wrong path](labs/systemd-service-not-starting/) | Simulated troubleshooting lab | `systemctl`, `journalctl`, `systemctl cat` |
| [DNS resolution failure](labs/dns-resolution-failure/) | Simulated troubleshooting lab | `ping`, `getent`, `dig`, `resolvectl` |
| [File permission problem](labs/file-permission-problem/) | Simulated troubleshooting lab | `id`, `namei`, `ls`, `getfacl` |
| [Network connectivity](labs/network-connectivity/) | Simulated troubleshooting lab | `ip`, `ping`, `ss`, `nc` |
| [SSH key authentication](labs/ssh-key-authentication/) | Simulated troubleshooting lab | `ssh`, `systemctl`, `journalctl`, `ss` |
| [Log investigation](labs/log-investigation/) | Simulated troubleshooting lab | `journalctl`, `grep`, `less`, `tail` |
| [apt/dpkg recovery after `/boot` filled](labs/boot-space-package-repair/) | Note from my Kali VM | `df`, `du`, `dpkg`, `apt-get` |

## Safety notes

- Commands that change services or system files should be tested in a disposable VM first.
- A PID should be checked with `ps` before sending a signal.
- A failed command is evidence. It should not be hidden with repeated restarts.
- Example output will not match every distribution.

## Related study

- [Linux Admin Notes](https://github.com/masporfirio/linux-admin-notes)
- [Bash Support Scripts](https://github.com/masporfirio/bash-support-scripts)
- [LPIC-101 Study](https://github.com/masporfirio/lpic-101-study)
