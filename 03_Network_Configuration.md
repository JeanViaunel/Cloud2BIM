# Network Configuration

This section describes how to configure network settings in Oracle Linux 9, including interface management, hostname, routing, DNS, and system files.

---

## Checking Network State

Show the current network status:

```bash
ip addr
```

or simply:

```bash
ip a
```

---

## Hostname and General Network Settings

### `/etc/sysconfig/network`

This file holds top-level networking configurations such as hostname and gateway:

```bash
NETWORKING=yes
HOSTNAME=master
GATEWAY=10.0.0.254
```

> In Red Hat family, use `hostnamectl` to change or display the hostname.

Set new hostname:

```bash
hostnamectl set-hostname <new-hostname>
```

Show current hostname:

```bash
hostnamectl
```

or:

```bash
hostname
```

---

## Network Interface Configuration

### Oracle Linux 8: `/etc/sysconfig/network-scripts/`

Use legacy `ifup` and `ifdown` commands:

Turn off interface:

```bash
ifdown eth0
```

Turn on interface:

```bash
ifup eth0
```

Show all interface states:

```bash
ifstat
```

> Make sure `ONBOOT=yes` is set in the configuration script to enable interfaces at boot time.

---

### Oracle Linux 9: `/etc/NetworkManager/system-connections/`

Use `nmcli` to manage interfaces:

Turn off connection:

```bash
nmcli conn down <device>
```

Turn on connection:

```bash
nmcli conn up <device>
```

Show connection info:

```bash
nmcli conn show <device>
```

or:

```bash
nmcli conn show
```

Rename a connection:

```bash
nmcli conn modify <old-name-or-UUID> connection.id <new-name>
```

Create a new configuration:

```bash
nmcli connection add type ethernet ifname <device_name> con-name <device_name> \
ipv4.method manual ipv4.addresses "<ip>/<netmask>" ipv4.gateway "<gateway>" \
ipv4.dns "<dns1> <dns2>" ipv4.dns-search ""
```

Example:

```bash
nmcli connection add type ethernet ifname ens192 con-name ens192 \
ipv4.method manual ipv4.addresses "10.112.12.100/24" ipv4.gateway "10.112.12.254" \
ipv4.dns "8.8.8.8 8.8.4.4" ipv4.dns-search ""
```

Example configuration file:

```ini
[connection]
id=ens192
uuid=<uuid>
type=ethernet
autoconnect-priority=-999
interface-name=ens192
zone=<zone_of_firewall>

[ethernet]

[ipv4]
address1=10.112.12.100/24,10.112.12.254
method=manual
route-metric=<route-metric>

[ipv6]
method=ignore

[proxy]
```

> `route-metric` sets the priority when multiple routes are defined.

---

## `/etc/hosts`

Local name resolution entries, e.g.:

```
10.0.0.1      master
10.0.0.2      frontend
10.0.0.101    slave
```

---

## `/etc/resolv.conf`

DNS configuration file. Multiple `nameserver` entries are allowed:

```
search aiengineer.tw
nameserver 8.8.8.8
nameserver 8.8.4.4
```

---

## Routes

Add a route:

```bash
ip route add <network_ip>/<netmask> via <gateway>
```

Example:

```bash
ip route add 61.56.0.0/16 via 10.112.12.254
```

List current routes:

```bash
ip route list
```

---

## Restarting Network

```bash
systemctl restart NetworkManager

nmcli networking off
nmcli networking on
nmcli networking connectivity
```

---

## Reference

- [Oracle (2025). "Linux Network Configuration," in *Oracle-Base*](https://oracle-base.com/articles/linux/linux-network-configuration#networking_files)
