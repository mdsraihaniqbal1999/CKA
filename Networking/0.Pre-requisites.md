# Networking Basics with Linux Commands

This guide explains fundamental networking concepts using Linux commands and examples. It covers how to connect computers via switches, how routing works across networks, and how to configure and verify networking components using commands like `ip link`, `ip addr`, `ip route`, and `route`. This is written in a GitHub-friendly `README.md` format.

---

## 🧩 Connecting Two Computers via a Switch

### 🖧 Setup
- Two computers (Host A and Host B)
- A switch
- Each computer is connected to the switch via a network interface card (NIC), typically `eth0`.

```
[Host A eth0] ---- [Switch] ---- [Host B eth0]
```

### 🔧 Commands
#### View Network Interfaces
```bash
ip link
```
- Shows all available interfaces.
- `eth0` will be the primary wired interface on most systems.

#### Assign IP Addresses
```bash
ip addr add 192.168.1.10/24 dev eth0  # On Host A
ip addr add 192.168.1.20/24 dev eth0  # On Host B
```

#### Bring the Interface Up
```bash
ip link set eth0 up
```

#### View Assigned IP Addresses
```bash
ip addr show
```

---

## 🌐 Communication Across Networks

To allow systems in different networks to communicate, a **router** is needed.

```
[Host A 192.168.1.10] -- [Router eth0 192.168.1.1 | eth1 10.0.0.1] -- [Host C 10.0.0.2]
```

### 🧭 Purpose of the Router
- Routes traffic between different subnets (networks)

### 🧭 Purpose of `route` and `ip route`

#### `ip route`
- Displays or modifies the IP routing table
- Modern replacement for `route`

#### Example
```bash
ip route add 10.0.0.0/24 via 192.168.1.1
```
- Adds a route to reach network `10.0.0.0/24` via gateway `192.168.1.1`

---

## 🛣️ Understanding Gateway vs Route

| Term     | Definition |
|----------|-------------|
| **Gateway** | A device (usually a router) that connects your network to another network |
| **Route**   | An entry in the routing table that tells the system where to send traffic |

### 🔍 View Routes
```bash
ip route show
```

---

## 🖥️ Host A to Host C via Host B (Router)

```
Host A (192.168.1.10) --- (eth0) Host B (eth1) --- Host C (10.0.0.2)
```
- Host B acts as a **router** between the two subnets

### Configuration Steps
#### On Host B
```bash
# Assign IPs
ip addr add 192.168.1.1/24 dev eth0
ip addr add 10.0.0.1/24 dev eth1

# Enable IP forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward
```

#### On Host A
```bash
ip route add 10.0.0.0/24 via 192.168.1.1
```

#### On Host C
```bash
ip route add 192.168.1.0/24 via 10.0.0.1
```

---

## 🧪 Commands Recap

| Command                      | Purpose |
|-----------------------------|---------|
| `ip link`                   | Show or manage interfaces |
| `ip addr`                   | Show or assign IP addresses |
| `ip addr add <IP> dev <IF>`| Assign IP address |
| `ip route`                  | Show routing table |
| `ip route add`             | Add a route to routing table |
| `route`                     | Legacy tool for routing |
| `cat /proc/sys/net/ipv4/ip_forward` | Check if IP forwarding is enabled |

---

## ⚠️ Persistence Note

By default, these configurations **do not persist** after a system reboot.
To make them persistent:

- Use network configuration files (e.g., `/etc/network/interfaces`, `/etc/netplan/*`, or `/etc/sysconfig/network-scripts/*` depending on your distro)
- Enable IP forwarding persistently via `/etc/sysctl.conf`:

```bash
net.ipv4.ip_forward = 1
```

Then apply:
```bash
sysctl -p
```

---

## ✅ Final Notes
- Always verify interface status with `ip link`
- Use `ping` or `traceroute` to test connectivity between hosts
- Check firewall rules if pings fail



