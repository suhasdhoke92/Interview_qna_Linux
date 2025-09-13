
# Linux Interview Questions and Answers 

This README provides a compiled set of Linux interview questions and answers, focusing on system administration, scripting, and troubleshooting. It's based on common scenarios encountered in Linux environments, such as log management, user administration, performance issues, and web server problems. The content is structured in a Q&A format for easy reference and preparation.

These questions cover practical commands, scripts, and step-by-step troubleshooting. Feel free to contribute or expand this repository with more examples.

## Table of Contents
- [Log Management](#log-management)
- [Automation and Scripting](#automation-and-scripting)
- [File Management](#file-management)
- [User and Login Tracking](#user-and-login-tracking)
- [Web Server Troubleshooting](#web-server-troubleshooting)
- [Text Processing](#text-processing)
- [Cloud Instance Access](#cloud-instance-access)
- [Disk Space Management](#disk-space-management)
- [Performance Optimization](#performance-optimization)
- [Nginx Application Issues](#nginx-application-issues)
- [SSH Troubleshooting](#ssh-troubleshooting)

---

## Log Management

### 1. How do you find and list log files older than 7 days in the `/var/log` folder?
**Answer:**  
Use the `find` command:  
- To find all `.log` files:  
  ```bash
  find /var/log -type f -name "*.log"
  ```  
- To find files older than 7 days:  
  ```bash
  find /var/log -type f -name "*.log" -mtime +7
  ```  
- To list with details:  
  ```bash
  find /var/log -type f -name "*.log" -mtime +7 -exec ls -lh {} \;
  ```  
- To remove them (with confirmation):  
  ```bash
  find /var/log -type f -name "*.log" -mtime +7 -exec rm -i {} \;
  ```  

⚠️ In production, log rotation (`logrotate`) is preferred over manual deletion.

---

## Automation and Scripting

### 2. How can you automate log rotation via a daily cron job to compress logs older than 7 days and delete logs older than 30 days?
**Answer:**  
Steps:  
1. Verify the directory exists.  
2. Find and compress logs older than 7 days (but newer than 30).  
3. Find and delete logs older than 30 days.  
4. Schedule via cron.  

Example script (`log_rotation.sh`):  
```bash
#!/bin/bash
LOG_DIR="/var/log/myapp"
LOG_FILE="/var/log/myapp/log_rotation.log"

if [ ! -d "$LOG_DIR" ]; then
    echo "[$(date)] Log directory does not exist" >> "$LOG_FILE"
    exit 1
fi

# Compress logs older than 7 days but newer than 30 days
find "$LOG_DIR" -type f -name "*.log" -mtime +7 -mtime -30 ! -name "*.gz"     -exec gzip {} \; -exec echo "[$(date)] Compressed: {}" >> "$LOG_FILE" \;

# Delete compressed logs older than 30 days
find "$LOG_DIR" -type f -name "*.gz" -mtime +30     -exec rm -f {} \; -exec echo "[$(date)] Deleted: {}" >> "$LOG_FILE" \;

echo "[$(date)] Log rotation completed successfully" >> "$LOG_FILE"
```

Schedule with cron:  
```bash
crontab -e
0 0 * * * /path/to/log_rotation.sh
```

---

### 3. How do you perform bulk user creation using a CSV file?
**Answer:**  
Steps:  
- Ensure CSV exists.  
- Skip header line.  
- Read username and password.  
- Create user with `useradd`.  
- Set password with `chpasswd`.  

Example script (`bulk_user_creation.sh`):  
```bash
#!/bin/bash
CSV_FILE="users.csv"
LOG_FILE="user_creation.log"

if [ ! -f "$CSV_FILE" ]; then
    echo "[$(date)] CSV file does not exist" >> "$LOG_FILE"
    exit 1
fi

tail -n +2 "$CSV_FILE" | while IFS=',' read -r username password; do
    if id "$username" &>/dev/null; then
        echo "[$(date)] User $username already exists" >> "$LOG_FILE"
        continue
    fi
    useradd -m "$username" && echo "$username:$password" | chpasswd
    echo "[$(date)] Created user: $username" >> "$LOG_FILE"
done

echo "[$(date)] Bulk user creation completed" >> "$LOG_FILE"
```

---

### 4. How would you write a Bash script to monitor service health?
**Answer:**  
Check if service is running and log CPU/memory usage.

Example script (`service_health_monitor.sh`):  
```bash
#!/bin/bash
SERVICE="nginx"  # change as needed
LOG_FILE="service_health.log"

if systemctl is-active --quiet "$SERVICE"; then
    echo "[$(date)] $SERVICE is running" >> "$LOG_FILE"
else
    echo "[$(date)] $SERVICE is down! Attempting restart..." >> "$LOG_FILE"
    systemctl restart "$SERVICE"
fi

CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | awk '{print $2 + $4}')
MEM_USAGE=$(free -m | awk '/Mem/{printf "%.2f", $3/$2 * 100.0}')

echo "[$(date)] CPU: $CPU_USAGE% | Memory: $MEM_USAGE%" >> "$LOG_FILE"
```

Schedule with cron for regular checks.

---

## File Management

### 5. How do you find and delete files over 100 MB in `/var/`?
**Answer:**  
```bash
find /var/ -type f -size +100M -exec rm -i {} \;
```  
The `-i` flag prompts confirmation before deletion.

---

## User and Login Tracking

### 6. How do you get the list of users who logged in today?
**Answer:**  
```bash
last -F | grep "$(date '+%b %e')" | awk '{print $1}' | sort | uniq
```  

This extracts usernames from today's logins.

---

## Web Server Troubleshooting

### 7. If a website doesn’t load (not accessible) and you have access to the server where it's hosted, how would you investigate?
**Answer:**  
1. Check if the service is running:  
   ```bash
   systemctl status nginx
   ```  
2. Inspect logs:  
   ```bash
   tail -f /var/log/nginx/error.log
   ```  
3. Verify document root files exist.  
4. Test configuration:  
   ```bash
   nginx -t
   ```  
5. Ensure firewall/security groups allow traffic on port 80/443.

---

## Text Processing

### 8. Using the sed command, how do you remove the first and last line of a file?
**Answer:**  
```bash
sed '1d;$d' file.txt
```  

For in-place edit:  
```bash
sed -i '1d;$d' file.txt
```

---

## Cloud Instance Access

### 9. If you've lost your PEM file, how would you access a cloud instance? Can you restore the deleted PEM file? How would you connect to the cloud provider instance?
**Answer:**  
- PEM files **cannot be restored**.  
- Alternatives:  
  - Use AWS Session Manager or EC2 Instance Connect (if enabled).  
  - Generate a new key pair and add its public key to `~/.ssh/authorized_keys` of the instance.  
  - Access by attaching the disk to another instance if locked out.  

---

## Disk Space Management

### 10. If `/var` is almost 90% full, what would be your next steps?
**Answer:**  
1. Check usage:  
   ```bash
   du -sh /var/*
   ```  
2. Investigate large subdirectories (`/var/log`, `/var/cache`, `/var/spool`).  
3. Archive/remove old logs.  
4. Configure `logrotate`.  
5. Clear package caches:  
   ```bash
   apt clean   # Debian/Ubuntu
   yum clean all  # RHEL/CentOS
   ```  

---

## Performance Optimization

### 11. If a Linux server is slow due to high CPU utilization, how would you fix it?
**Answer:**  
1. Identify processes:  
   ```bash
   top
   htop
   ```  
2. Kill non-critical processes:  
   ```bash
   kill <PID>
   ```  
3. Lower priority for heavy processes:  
   ```bash
   renice 19 -p <PID>
   ```  
4. Check logs, consider scaling resources.

---

## Nginx Application Issues

### 12. If an application deployed on Nginx returns "connection refused," how would you fix it? Also, explain common scenarios for 502 Bad Gateway and 404 Not Found errors.
**Answer:**  
- **Connection refused**: service not running, wrong port, or firewall blocking.  
  - Check Nginx status.  
  - Ensure firewall/security groups allow traffic.  

- **502 Bad Gateway**: Nginx can't reach upstream.  
  - Backend crashed, wrong upstream config, or timeouts.  

- **404 Not Found**: file/resource missing.  
  - Check document root and Nginx location blocks.  

---

## SSH Troubleshooting

### 13. If SSH to an instance fails, what could be the causes and how would you troubleshoot?
**Answer:**  
- Wrong IP address.  
- Incorrect PEM permissions (must be 600).  
- Firewall blocking port 22.  
- Cloud security group rules missing SSH.  
- SSH service down (`systemctl status ssh`).  

---

## Additional Resources

- Use `man` pages for command help.  
- Tools: `htop`, `logrotate`, `systemctl`.  
- Practice in VMs or cloud labs.

---

