# Lab: Investigate Failed SSH Logins

**Type:** Simulated troubleshooting lab.

## Scenario

A test user reports several failed SSH login attempts before a successful login. The goal is to review only the relevant time window and confirm whether the failure was authentication or service availability.

## Objective

Filter service logs, find the failure message and compare it with service and network state.

## Environment

- Debian or Ubuntu virtual machine
- OpenSSH server
- Console or `sudo` access to read authentication logs

## Investigation

### 1. Confirm the service is available

```bash
systemctl is-active ssh
sudo ss -ltnp | grep ':22'
```

An active service and listening port make a service outage less likely.

### 2. Review a short time window

```bash
sudo journalctl -u ssh --since '15 minutes ago' --no-pager
```

Limiting the time range avoids searching unrelated entries from earlier sessions.

### 3. Filter authentication failures

On a system that writes `/var/log/auth.log`:

```bash
sudo grep -i 'failed password' /var/log/auth.log | tail -n 20
sudo grep -ic 'failed password' /var/log/auth.log
```

`tail` keeps the review small. `grep -c` gives a count without turning the count alone into a security conclusion.

### 4. Read the full lines safely

```bash
sudo less /var/log/auth.log
```

I use `less` instead of printing a large log with `cat`. Any hostnames, usernames or IP addresses should be removed before sharing a screenshot or note.

## Fix

In this practice scenario, the client used the wrong test username. Retry with the correct lab account:

```bash
ssh <correct-user>@<server>
```

The log should be treated as evidence. A real series of unexpected failures would require a separate security review instead of changing authentication settings immediately.

## Verification

```bash
sudo journalctl -u ssh --since '5 minutes ago' --no-pager
last -n 5
```

The new entries should show a successful login for the intended test account.

## What I learned

The service status answered whether SSH was running. The authentication log answered a different question: why this user's login failed.
