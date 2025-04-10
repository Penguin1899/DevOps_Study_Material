## Extended Logging in Linux: journald, Structured Logging, and Logrotate

### journald (Systemd Journal)

How it works:
- All output from systemd services goes into a binary journal file.
Logs are stored in:
- Volatile memory: /run/log/journal/ (lost on reboot)
- Persistent storage: /var/log/journal/ (only if enabled)

Enable persistent journaling:
```
sudo mkdir -p /var/log/journal
sudo systemd-tmpfiles --create --prefix /var/log/journal
sudo systemctl restart systemd-journald
```

### Structured Logging with journald

Systemd journal supports structured logging, meaning logs have:
- Priority levels
- Timestamps
- Source (service name, PID, etc.)
- Fields like MESSAGE, SYSLOG_IDENTIFIER, etc.

Example:
```
journalctl -u nginx.service --output=json-pretty
```

You’ll see structured JSON output:
```
{
  "_SYSTEMD_UNIT": "nginx.service",
  "MESSAGE": "Started A high performance web server and a reverse proxy server.",
  "_PID": "1142",
  "_HOSTNAME": "myserver",
  "__REALTIME_TIMESTAMP": "1683129389028384"
}
```

Filter logs with fields:
```
journalctl PRIORITY=3   # Only errors
journalctl _PID=1142
```

### Custom Logs from Your Scripts
If your script is writing logs with echo, you can send structured logs to journald:

Using systemd-cat
```
echo "An error occurred" | systemd-cat -t myapp -p err
```
- -t is tag (service name)
- -p is priority (emerg, alert, crit, err, warning, notice, info, debug)

Then view:
```
journalctl -t myapp
```

You can even wrap it into your script:
```
#!/bin/bash
systemd-cat -t myapp -p info <<< "Service is starting"
```

## logrotate – Rotating Traditional Log Files

What is it?
- logrotate manages logs that grow over time by:
    - Archiving old logs
    - Compressing them (e.g., .gz)
    - Deleting older files
    - Running pre/post scripts

Why is it needed?
- If your custom service or app writes to /var/log/myapp.log, you don’t want it growing forever.

Logrotate Configuration
- Global config: /etc/logrotate.conf
- App-specific configs: /etc/logrotate.d/myapp

Example: /etc/logrotate.d/myapp
```
/var/log/myapp.log {
    daily
    rotate 7
    compress
    missingok
    notifempty
    create 0640 myuser mygroup
    postrotate
        systemctl restart myapp.service > /dev/null 2>&1 || true
    endscript
}
```
Explanation:
- daily: Rotate once a day
- rotate 7: Keep 7 days of logs
- compress: Use gzip
- create: Recreate log after rotation
- postrotate: Restart app if needed

Manually Test Logrotate
```
sudo logrotate -d /etc/logrotate.d/myapp      # Dry-run
sudo logrotate -f /etc/logrotate.d/myapp      # Force rotate
```

## Summary – When to Use What?

| Situation                               | Use                                          |
|-----------------------------------------|----------------------------------------------|
| Service using systemctl                 | Logs go to journald, use journalctl         |
| Custom scripts writing logs to files     | Manage with logrotate                        |
| Need structured querying                | Use journald with JSON output                |
| Want to tag logs from scripts           | Use systemd-cat                              |

---
## Interview Examples:

- **Question**: Where do systemd service logs go by default?

To the system journal. View them with journalctl -u servicename.

---
- **Question**: How do you prevent logs from being lost on reboot?

Enable persistent journaling by creating /var/log/journal/.

---
- **Question**: Your app writes to a file. How to prevent log growth?

Set up a logrotate config in /etc/logrotate.d/myapp

---
- **Question**: Scenario: You want to run a shell script as a systemd service that logs everything

Script Path: /usr/local/bin/backup.sh
Log Path: /var/log/backup.log

Step 1: The Script (backup.sh)
```
#!/bin/bash

LOGFILE="/var/log/backup.log"

echo "[$(date)] Backup script started" | systemd-cat -t backup-service -p info
echo "[$(date)] Backup script started" >> "$LOGFILE"

# Simulate work
sleep 2

# Logging both to file and journald
echo "[$(date)] Performing backup..." | systemd-cat -t backup-service -p info
echo "[$(date)] Performing backup..." >> "$LOGFILE"

sleep 2

echo "[$(date)] Backup completed successfully." | systemd-cat -t backup-service -p notice
echo "[$(date)] Backup completed successfully." >> "$LOGFILE"
```

Ensure the script is executable:

```
sudo chmod +x /usr/local/bin/backup.sh
```

Step 2: The systemd Service File
Path: /etc/systemd/system/backup.service

```
[Unit]
Description=Daily Backup Service
After=network.target

[Service]
ExecStart=/usr/local/bin/backup.sh
User=nobody
Restart=on-failure
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

Step 3: Reload and Enable the Service
```
sudo systemctl daemon-reload
sudo systemctl enable backup.service
sudo systemctl start backup.service
```

Step 4: View Logs
Using journalctl
```
journalctl -t backup-service
```

View log file
```
cat /var/log/backup.log
```

Step 5: Setup Logrotate
Path: /etc/logrotate.d/backup

```
/var/log/backup.log {
    weekly
    rotate 4
    compress
    missingok
    notifempty
    create 0644 nobody nobody
    postrotate
        systemctl restart backup.service > /dev/null 2>&1 || true
    endscript
}
```
Final Notes (For Interview/Production)
- You’re covering both journald + traditional logs, which is great for flexibility.
- Using systemd-cat tags logs nicely and allows you to filter easily.
- With logrotate, you prevent log bloat.
- Running as nobody for minimal privilege — bonus points.

