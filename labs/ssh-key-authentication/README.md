# Lab: SSH Key Authentication Fails

**Type:** Simulated troubleshooting lab.

## Scenario

The SSH service is running and the server is reachable, but public-key authentication fails because the user's `.ssh` directory and `authorized_keys` file have unsafe permissions.

## Objective

Check the client message, service state, listening port, server logs and key-file permissions before changing the SSH configuration.

## Environment

- Two Linux virtual machines or one VM and a Linux client
- OpenSSH client and server
- Console access to the server

## Investigation

### 1. Collect client detail

```bash
ssh -v <user>@<server>
```

Verbose mode shows whether the client offered the expected key and how the server answered. It does not reveal the private key contents.

### 2. Check the server service and port

```bash
sudo systemctl status ssh --no-pager
sudo ss -ltnp | grep ':22'
```

On some distributions the unit is named `sshd` instead of `ssh`.

### 3. Read recent SSH logs

```bash
sudo journalctl -u ssh -n 50 --no-pager
```

The log may report that the key file or home directory has incorrect ownership or modes.

### 4. Inspect the path as the target user

```bash
sudo -u <user> ls -ld /home/<user> /home/<user>/.ssh
sudo -u <user> ls -l /home/<user>/.ssh/authorized_keys
```

## Fix

After confirming the correct account and home directory:

```bash
sudo chown -R <user>:<user> /home/<user>/.ssh
sudo chmod 700 /home/<user>/.ssh
sudo chmod 600 /home/<user>/.ssh/authorized_keys
```

I do not copy or publish the private key. Only the public key belongs in `authorized_keys`.

## Verification

```bash
ssh -v <user>@<server>
sudo journalctl -u ssh -n 20 --no-pager
```

The login should use public-key authentication without a new permission warning.

## What I learned

A running SSH service and an open port only prove that the daemon is available. The client trace and server log were needed to identify an authentication problem instead of a network problem.
