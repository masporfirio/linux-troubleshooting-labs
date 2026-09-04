# Lab: Port Conflict

**Type:** Simulated troubleshooting lab, completed with screenshots.

## Scenario

A simple web server should listen on TCP port `8080`, but a second copy fails with `Address already in use`.

## Objective

Find which process owns the port, confirm what it is, stop the correct process and verify that the port can be used again.

## Environment

- Linux virtual machine
- Python 3 standard library web server
- Two terminal sessions

## Investigation

### 1. Reproduce the conflict

Start a local test server:

```bash
python3 -m http.server 8080
```

In a second terminal, run the same command. The second process should fail because only one listener can bind to the same address and port.

### 2. Check the listening socket

```bash
ss -ltnp | grep ':8080'
```

`ss` confirms that the port is listening and may show the process name and PID. Without sufficient permissions, the process details may be missing.

### 3. Confirm the PID

```bash
fuser 8080/tcp
ps -fp <PID>
```

`fuser` gives another way to find the PID. `ps` is the important safety check: it confirms which process would receive the signal.

## Fix

After confirming that the PID belongs to the test server:

```bash
kill <PID>
```

I use the normal `TERM` signal first. A forced `kill -9` is not the default response because it does not let the process clean up.

## Verification

Check that the old listener is gone:

```bash
ss -ltnp | grep ':8080'
```

No matching line means the port is free. Start the server again and test it locally:

```bash
python3 -m http.server 8080
curl -I http://127.0.0.1:8080
```

## What I learned

The error message identifies a resource conflict, but it does not identify the safe process to stop. Checking the socket and then confirming the PID with `ps` avoids acting on the wrong process.

## Screenshots

- [Address already in use](screenshots/01-address-already-in-use.png)
- [Port check with ss](screenshots/02-ss-port-check.png)
- [Process identification with fuser](screenshots/03-fuser-process.png)
- [Port available after the fix](screenshots/04-resolved-port-conflict.png)
