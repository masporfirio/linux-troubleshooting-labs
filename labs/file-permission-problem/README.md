# Lab: File Permission Problem

**Type:** Simulated troubleshooting lab.

## Scenario

A support user receives `Permission denied` when reading a shared file. The file belongs to the expected group, but one directory in the path does not allow group traversal.

## Objective

Check the user's identity, every part of the path and any ACLs before changing permissions.

## Environment

- Linux virtual machine
- A test user and group
- ACL tools already installed if `getfacl` is used

## Investigation

### 1. Confirm the user and groups

```bash
id <user>
groups <user>
```

This checks whether the user is actually a member of the group that should have access.

### 2. Check the file

```bash
ls -l /srv/support/reports/status.txt
```

The file mode and owner are only part of the path.

### 3. Check every directory component

```bash
namei -l /srv/support/reports/status.txt
```

Directories need execute permission for traversal. A readable file can still be unreachable through a restricted directory.

### 4. Check ACLs

```bash
getfacl /srv/support/reports/status.txt
getfacl /srv/support/reports
```

An ACL can add or restrict effective permissions beyond the basic mode shown by `ls -l`.

## Fix

In this scenario, the intended group is `support` and only that group should traverse the reports directory:

```bash
sudo chgrp support /srv/support/reports
sudo chmod 750 /srv/support/reports
```

If the user is missing from the group:

```bash
sudo usermod -aG support <user>
```

The user needs a new login session before the group membership is normally available.

## Verification

```bash
sudo -u <user> test -r /srv/support/reports/status.txt
sudo -u <user> head /srv/support/reports/status.txt
```

## What I learned

Changing the file to mode `777` would hide the real problem and grant more access than needed. `namei -l` showed that the blocked directory, not the file itself, was the cause.
