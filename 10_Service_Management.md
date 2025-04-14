# Systemd Service Management

This section introduces how to manage services (daemons) using `systemctl` under systemd in Oracle Linux.

---

## Service File Locations

- **Unit files (service definitions):**
  ```text
  /usr/lib/systemd/system/
  ```

- **Environment variable overrides:**
  ```text
  /etc/sysconfig/
  ```

---

## Reload All Services

After creating or modifying service unit files:

```bash
systemctl daemon-reload
```

This reloads the systemd manager configuration.

---

## Control Services

Start, stop, restart, or check the status of a service:

```bash
systemctl start <daemon>
systemctl stop <daemon>
systemctl restart <daemon>
systemctl status <daemon>
```

---

## Enable or Disable Services at Boot

Enable a service to auto-start on boot:

```bash
systemctl enable <daemon>
```

Disable a service from starting at boot:

```bash
systemctl disable <daemon>
```

---

## Reference

- [Oracle (2025). "Linux Services," in *Oracle-Base*](https://oracle-base.com/articles/linux/linux-services-systemd)
