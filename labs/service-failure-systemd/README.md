# Lab: systemd Service Fails Because a Port Is Busy

**Type:** Simulated troubleshooting lab, completed with screenshots.

This lab adds `systemd` and journal logs to the basic [port conflict lab](../port-conflict/).

## Scenario

`myapp.service` starts a Python web server on TCP port `8080`. Another process is already using the port, so systemd reports a failed service.

## Objective

Use service status, logs and socket information to connect the systemd failure to the port conflict.

## Environment

- Debian-based Linux virtual machine
- `systemd`
- Python 3 and `curl`
- `sudo` access in the lab VM

## Investigation

### 1. Check the service state

```bash
sudo systemctl status myapp.service --no-pager
```

This shows whether systemd started the unit, its exit status and recent messages. It is a quick first check, not the complete diagnosis.

### 2. Read logs for this unit

```bash
sudo journalctl -u myapp.service -n 30 --no-pager
```

Filtering by unit keeps the first review focused. The expected error is `Address already in use`.

### 3. Check the port

```bash
sudo ss -ltnp | grep ':8080'
```

The listener information links the application error to a running process.

### 4. Verify the process

```bash
ps -fp <PID>
```

This check comes before stopping anything.

## Fix

Stop the confirmed test process, then start the systemd service again:

```bash
kill <PID>
sudo systemctl start myapp.service
```

## Verification

```bash
systemctl is-active myapp.service
sudo ss -ltnp | grep ':8080'
curl -I http://127.0.0.1:8080
```

The checks answer three different questions: whether systemd considers the service active, whether it is listening, and whether it responds over HTTP.

If the unit is meant to start at boot, check that separately:

```bash
systemctl is-enabled myapp.service
```

`enabled` and `active` are not the same state.

## Helper script

The included `service_status.sh` prints service status and the last 30 journal lines:

```bash
./service_status.sh myapp.service
```

## What I learned

Restarting a failed service does not remove the cause. The journal message led to the port check, and the HTTP request verified more than the service status alone.

## Screenshots

- [Failed service](screenshots/service_failed.png)
- [Journal error](screenshots/journalctl_error.png)
- [Port conflict](screenshots/port_conflict.png)
- [Running service](screenshots/service_running.png)
- [HTTP test](screenshots/curl_test.png)
