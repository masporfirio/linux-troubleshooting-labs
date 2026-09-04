# Lab: DNS Resolution Failure

**Type:** Simulated troubleshooting lab.

## Scenario

A Linux machine can reach an external IP address but cannot resolve domain names.

## Objective

Separate basic connectivity from DNS resolution, identify the resolver configuration and verify the result after correcting it.

## Environment

- Debian or Ubuntu virtual machine
- `systemd-resolved` in this example
- `dnsutils` already installed for `dig`, `host` and `nslookup`

## Investigation

### 1. Check the local address and route

```bash
ip addr
ip route
```

This confirms that the interface has an address and that a default route exists.

### 2. Test connectivity without DNS

```bash
ping -c 3 1.1.1.1
```

If this works, the machine can reach an external IP. It does not prove that DNS works.

### 3. Test name resolution

```bash
getent hosts example.com
ping -c 3 example.com
```

`getent` uses the system name-service configuration. A failure here, together with successful IP connectivity, points toward DNS.

### 4. Inspect the resolver

```bash
resolvectl status
cat /etc/resolv.conf
```

I check both because `/etc/resolv.conf` may be a file or a symlink managed by another service.

### 5. Query DNS directly

```bash
dig example.com
host example.com
nslookup example.com
```

`dig` shows which server answered and whether the response contained an address.

## Fix

The permanent fix depends on which service manages the connection. In this simulated `systemd-resolved` case, set a temporary DNS server on the correct interface:

```bash
sudo resolvectl dns <interface> 1.1.1.1
sudo resolvectl flush-caches
```

I would then correct the connection profile in NetworkManager, Netplan or the VM network configuration. Editing a managed `/etc/resolv.conf` directly may be overwritten.

## Verification

```bash
resolvectl query example.com
getent hosts example.com
ping -c 3 example.com
```

## What I learned

`ping example.com` mixes two tests. Checking an IP first and then using `getent` or `dig` made it easier to separate routing from name resolution.
