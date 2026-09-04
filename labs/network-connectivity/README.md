# Lab: Network Connectivity

**Type:** Simulated troubleshooting lab.

## Scenario

A Linux VM cannot reach a service on another host. The interface was left down after a network change.

## Objective

Check the connection in layers: link, address, route, gateway, remote IP, DNS and the application port.

## Environment

- Linux virtual machine
- A known gateway address
- A test service and port

## Investigation

### 1. Check link state

```bash
ip link
```

The expected interface shows `state DOWN`, so higher-level tests cannot work yet.

### 2. Check addresses

```bash
ip addr
```

This shows whether the interface has an IPv4 or IPv6 address after it is brought up.

### 3. Check routes

```bash
ip route
```

For an external destination, I expect a route that matches it, usually a default route through the gateway.

### 4. Test one layer at a time

```bash
ping -c 3 <gateway-ip>
ping -c 3 <remote-ip>
getent hosts <service-name>
nc -vz <service-name> <port>
```

These checks separate local routing, remote reachability, DNS and the application port. Some networks block ICMP, so a failed ping is not the only test.

## Fix

Bring up the confirmed interface:

```bash
sudo ip link set dev <interface> up
```

If a connection manager is in use, reconnect through that manager so the address and routes are restored. For NetworkManager:

```bash
nmcli device status
sudo nmcli device connect <interface>
```

## Verification

```bash
ip link show <interface>
ip addr show <interface>
ip route
nc -vz <service-name> <port>
```

## What I learned

Testing the remote service first did not show where the failure started. Checking the link and route before DNS and ports made the order of the investigation clearer.
