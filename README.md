# Linux_interview_qna
# Linux Interview Questions and Answers README

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

## Log Management

### 1. How do you find and list log files older than 7 days in the `/var/log` folder?
**Answer:**  
Use the `find` command:  
- To find all `.log` files: `find /var/log -type f -name "*.log"` (`-type f` for files, `-type d` for directories).  
- To find files older than 7 days: `find /var/log -type f -name "*.log" -mtime +7`.  
- To list with details: `find /var/log -type f -name "*.log" -mtime +7 -exec ls -ltr {} \;` ({} holds file details).  
- To remove them: `find /var/log -type f -name "*.log" -mtime +7 -exec rm {} \;`.

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

# Compress logs older than 7 days (but newer than 30 days) if not already gzipped
find "$LOG_DIR" -type f -name "*.log" -mtime +7 -mtime -30 ! -name "*.gz" -exec gzip {} \; -exec echo "[$(date)] Compressed: {}" >> "$LOG_FILE" \;

# Delete compressed logs older than 30 days
find "$LOG_DIR" -type f -name "*.gz" -mtime +30 -exec rm -f {} \; -exec echo "[$(date)] Deleted: {}" >> "$LOG_FILE" \;

echo "[$(date)] Log rotation completed successfully" >> "$LOG_FILE"
```
Schedule: `crontab -e` and add `0 0 * * * /path/to/log_rotation.sh` for daily at midnight.

### 3. How do you perform bulk user creation using a CSV file?
**Answer:**  
Steps:
- Check if the CSV file exists.  
- Ignore the header (line 1) and loop through the rest.  
- Use a while loop to read username and password.  
- Create user with `useradd`.  
- Set password with `chpasswd` (non-interactive).  

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

### 4. How would you write a Bash script to monitor service health?
**Answer:**  
A basic script checks if a service is running and monitors CPU/memory usage.

Example script (`service_health_monitor.sh`):
```bash
#!/bin/bash
SERVICE="nginx"  # Change to your service
LOG_FILE="service_health.log"

if systemctl is-active --quiet "$SERVICE"; then
    echo "[$(date)] $SERVICE is running" >> "$LOG_FILE"
else
    echo "[$(date)] $SERVICE is down! Attempting restart..." >> "$LOG_FILE"
    systemctl restart "$SERVICE"
fi

# Check CPU and memory
CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | awk '{print $2 + $4}')
MEM_USAGE=$(free -m | awk '/Mem/{printf "%.2f", $3/$2 * 100.0}')

echo "[$(date)] CPU: $CPU_USAGE% | Memory: $MEM_USAGE%" >> "$LOG_FILE"
```
Schedule via cron for regular checks.

## File Management

### 5. How do you find and delete files over 100 MB in `/var/`?
**Answer:**
```bash
find /var/ -type f -size +100M -exec rm -i {} \;
```

## User and Login Tracking

### 6. How do you get the list of users who logged in today?
**Answer:**
```bash
last -F | grep "$(date '+%b %e')" | awk '{print $1}' | sort | uniq
```

## Web Server Troubleshooting

### 7. If a website doesn’t load (not accessible) and you have access to the server, how would you investigate?
**Answer:**
- Check if the web server is running: `systemctl status nginx`  
- Inspect logs: `tail -f /var/log/nginx/error.log`  
- Verify document root: `/var/www/html/index.html` exists  
- Check configuration: `nginx -t`  

## Text Processing

### 8. Using the sed command, how do you remove the first and last line of a file?
**Answer:**
```bash
# View
sed '1d;$d' file.txt

# In-place
sed -i '1d;$d' file.txt
```

## Cloud Instance Access

### 9. If you've lost your PEM file, how would you access a cloud instance?
**Answer:**
- PEM files cannot be restored.  
- Use AWS Session Manager or EC2 Instance Connect.  
- Generate a new key: `ssh-keygen -t rsa -f new_key`.  
- Add public key to `~/.ssh/authorized_keys`.  
- Connect: `ssh -i new_key user@instance-ip`.  

## Disk Space Management

### 10. If `/var` is almost 90% full, what would be your next steps?
**Answer:**
- Check usage: `du -sh /var/*`  
- Clean old logs, cache, or crashes  
- Configure logrotate  

## Performance Optimization

### 11. If a Linux server is slow due to high CPU utilization, how would you fix it?
**Answer:**
- Identify process: `top` or `htop`  
- Kill or `renice` non-critical processes  
- Investigate logs for root cause  

## Nginx Application Issues

### 12. If an application on Nginx returns "connection refused," how would you fix it?
**Answer:**
- Check service: `systemctl status nginx`  
- Verify firewall & ports  
- AWS: security groups for 80/443  

**502 Bad Gateway:** Backend service down/misconfigured.  
**404 Not Found:** File missing or misconfigured location block.  

## SSH Troubleshooting

### 13. If SSH to an instance fails, what could be the causes?
**Answer:**
- Wrong IP address  
- PEM file permission issues  
- Firewall/security groups blocking port 22  
- SSH service not running  

---

## Additional Resources
- Use `man <command>` for more details.  
- Practice in VirtualBox or AWS EC2.  
- Tools: `htop`, `logrotate`, `systemctl`.  

---
If you have suggestions or more questions to add, open an **issue** or **pull request**!
