# Lab: systemd Service Has the Wrong ExecStart Path

**Type:** Simulated troubleshooting lab.

## Scenario

A custom unit fails immediately because `ExecStart` points to a script that does not exist.

## Objective

Use systemd status, unit-specific logs and the loaded unit file to identify a path error without repeatedly restarting the service.

## Environment

- Linux virtual machine using `systemd`
- `sudo` access in the lab VM

## Lab setup

Create a small script:

```bash
sudo install -d /opt/myapp
printf '#!/bin/sh\necho "myapp started"\nsleep 60\n' | sudo tee /opt/myapp/start.sh >/dev/null
sudo chmod 755 /opt/myapp/start.sh
```

Create `/etc/systemd/system/myapp.service` with an intentionally wrong path:

```ini
[Unit]
Description=Path troubleshooting practice

[Service]
Type=simple
ExecStart=/opt/myapp/missing-start.sh

[Install]
WantedBy=multi-user.target
```

Load the unit and reproduce the failure:

```bash
sudo systemctl daemon-reload
sudo systemctl start myapp.service
```

## Investigation

### 1. Check the service state

```bash
sudo systemctl status myapp.service --no-pager
```

The status should show a failed start and an execution-related error.

### 2. Read the unit logs

```bash
sudo journalctl -u myapp.service -n 30 --no-pager
```

The journal gives more detail than a general `journalctl -xe` search.

### 3. Inspect the configuration systemd loaded

```bash
systemctl cat myapp.service
```

This avoids checking a different copy of the unit by mistake.

### 4. Verify the referenced path

```bash
ls -l /opt/myapp/missing-start.sh
ls -l /opt/myapp/start.sh
```

The first path is missing, while the intended script exists and is executable.

## Fix

Change `ExecStart` to:

```ini
ExecStart=/opt/myapp/start.sh
```

Reload the changed unit and start it again:

```bash
sudo systemctl daemon-reload
sudo systemctl restart myapp.service
```

## Verification

```bash
systemctl is-active myapp.service
sudo systemctl status myapp.service --no-pager
```

Expected result: `active`.

## Cleanup

This removes only the files created for this disposable lab:

```bash
sudo systemctl disable --now myapp.service
sudo rm /etc/systemd/system/myapp.service
sudo rm -r /opt/myapp
sudo systemctl daemon-reload
```

Review the paths before running the cleanup commands.

## What I learned

`systemctl cat` was useful because it showed the exact `ExecStart` path loaded by systemd. The fix required both correcting the path and running `daemon-reload`.
