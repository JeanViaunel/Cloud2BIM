# Scheduling Tasks

This section describes how to schedule repetitive or one-time tasks using `cron` and `at` on Oracle Linux.

---

## CRON: Recurring Tasks

The `cron` service executes scheduled commands or scripts at specified time intervals.

### Install `crontab`

```bash
dnf install crontabs
```

### User Permissions

Allow only specific users to schedule cron jobs:

```bash
echo root > /etc/cron.allow
```

Deny all other users (optional):

```bash
echo > /etc/cron.deny
```

---

### Predefined Intervals

Scripts placed in these directories will run automatically at predefined times:

```text
/etc/cron.hourly
/etc/cron.daily
/etc/cron.weekly
/etc/cron.monthly
```

No need to edit crontab directly for these.

---

### Custom Interval Example (Every 10 Minutes)

Create a new cron configuration:

```bash
cp /etc/cron.d/0hourly /etc/cron.d/10mins
```

Edit `/etc/cron.d/10mins`:

```cron
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
*/10 * * * * root run-parts /etc/cron.10mins
```

This runs all scripts inside `/etc/cron.10mins` every 10 minutes.

Create the directory:

```bash
mkdir /etc/cron.10mins
chmod 755 /etc/cron.10mins
```

> `run-parts` will execute all executable scripts in the specified directory.

---

### Edit Crontab Directly (Optional)

To schedule tasks per user:

```bash
crontab -e
```

Example entry (run backup script at 3am daily):

```cron
0 3 * * * /usr/local/bin/backup.sh
```

---

### Manage `cron` Service

```bash
systemctl restart crond
systemctl enable crond
```

---

## AT: One-Time Tasks

The `at` utility schedules a command to run once at a specified time.

### Install `at`

```bash
dnf install at
```

### User Permissions

Allow user:

```bash
echo root > /etc/at.allow
```

Deny others:

```bash
echo > /etc/at.deny
```

---

### Manage `at` Service

```bash
systemctl restart atd
systemctl enable atd
```

---

### Schedule a One-Time Task

Enter interactive mode:

```bash
at 7:55 2024-05-05
```

Then type the command:

```bash
shutdown now -P
```

Press **Ctrl+D** to save and exit.

---

### View and Manage Jobs

List jobs:

```bash
at -l
```

View details:

```bash
at -vc <job_id>
```

Delete job:

```bash
at -d <job_id>
```

---

## Reference

- [Oracle (2025). "Scheduling Tasks on Linux," *Oracle-Base*](https://oracle-base.com/articles/linux/cron-on-linux)
