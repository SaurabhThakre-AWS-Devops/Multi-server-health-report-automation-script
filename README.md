# VM Health Monitoring Automation System

## Overview

This project automates the health monitoring of multiple Linux servers.

The monitoring server connects to multiple remote servers using SSH, collects system metrics (CPU, Memory, Disk), generates an HTML health report, converts it into a PDF report, and sends the report through email automatically.

The complete process is scheduled using Linux Cron Job for automatic execution.

---

# Architecture

```
                 +----------------------+
                 |  Management Server   |
                 |                      |
                 | Health Script        |
                 | SSH Client           |
                 | wkhtmltopdf          |
                 | ssmtp                |
                 | cron                 |
                 +----------+-----------+
                            |
                            |
                    SSH Connection
                            |
        -----------------------------------------
        |              |             |            |
        |              |             |            |
   Docker VM1     Docker VM2    Docker VM3   Docker VM4
   172.18.0.3     172.18.0.4   172.18.0.5  172.18.0.6


Remote Servers Collect:

- Hostname
- CPU Usage
- Memory Usage
- Disk Usage

Then:

Metrics → HTML Report → PDF Report → Email Notification
```

---

# Environment Setup

## Management Server Requirements

Install required packages:

```bash
apt update

apt install -y sshpass

apt install -y wkhtmltopdf

apt install -y ssmtp
```

---

# Remote Server Setup

For testing, Docker containers were used as remote servers.

Example containers:

```
server1
server2
server3
server4
server5
```

Each container works as an independent Linux server.

---

# SSH Configuration on Remote Servers

Login inside each container.

## Install SSH Server

```bash
apt update

apt install -y openssh-server sudo
```

---

## Create Monitoring User

```bash
adduser agm123
```

Password:

```
123
```

---

## Provide Sudo Permission

```bash
echo "agm123 ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
```

---

## Start SSH Service

Create SSH directory:

```bash
mkdir -p /run/sshd
```

Start SSH daemon:

```bash
/usr/sbin/sshd
```

Verify SSH service:

```bash
ps -ef | grep [s]shd
```

Expected:

```
root       sshd
```

---

# Test SSH Connectivity

Install sshpass on management server:

```bash
apt install -y sshpass
```

Test connection:

```bash
sshpass -p "123" ssh \
-o StrictHostKeyChecking=no \
-o UserKnownHostsFile=/dev/null \
agm123@172.18.0.3 hostname
```

Expected output:

```
container hostname
```

Repeat for all servers.

---

# Server Inventory Configuration

Create server list file:

```bash
nano servers.txt
```

Add all monitored servers:

```
172.18.0.3
172.18.0.4
172.18.0.5
172.18.0.6
```

The monitoring script reads this file and checks every server.

---

# Email Configuration

Install ssmtp:

```bash
apt install -y ssmtp
```

Edit configuration:

```bash
nano /etc/ssmtp/ssmtp.conf
```

Configuration:

```
root=your_email@gmail.com

mailhub=smtp.gmail.com:465

AuthUser=your_email@gmail.com

AuthPass=APP_PASSWORD

UseTLS=YES

UseSTARTTLS=NO

FromLineOverride=YES
```

Note:

- Gmail App Password is required.
- Do not use your normal Gmail password.

---

# Health Monitoring Script

Script Name:

```
multi_server_health_report.sh
```

Execution flow:

```
Read servers.txt

        |
        v

Connect Remote Servers using SSH

        |
        v

Collect Server Metrics

        |
        +--> CPU Usage
        |
        +--> Memory Usage
        |
        +--> Disk Usage
        |
        +--> Hostname

        |
        v

Determine Server Health

        |
        v

Generate HTML Report

        |
        v

Convert HTML Report to PDF

        |
        v

Send PDF Report via Email
```

---

# Run Script Manually

Give execution permission:

```bash
chmod +x multi_server_health_report.sh
```

Run:

```bash
./multi_server_health_report.sh
```

---

# Sample Output

```
[INFO] Starting VM Health Report Generation

[INFO] Metrics - CPU: 0%
Memory: 35%
Disk: 7%

[SUCCESS] PDF report created

[INFO] PDF created:
/tmp/vm-health-reports/vm_health_report.pdf

[SUCCESS] Email sent successfully

[SUCCESS] Report sent successfully

[INFO] Cleaned up old reports
```

---

# Generated Report Contains

## Server Information

- Server Name
- Report Date


## Resource Monitoring

| Resource | Usage | Status |
|----------|-------|--------|
| CPU | % | Normal/Warning/Critical |
| Memory | % | Normal/Warning/Critical |
| Disk | % | Normal/Warning/Critical |


## Filesystem Monitoring

Displays:

- Filesystem
- Mount point
- Size
- Used Space
- Usage Percentage


## Health Assessment

Displays overall server health:

```
HEALTHY
```

or

```
UNHEALTHY
```

---

# Cron Job Automation

Edit cron:

```bash
crontab -e
```

Example:

Run every day at 9 AM:

```bash
0 9 * * * /path/multi_server_health_report.sh >> /var/log/vm_health_monitor.log 2>&1
```

---

# Verify Cron Execution

Check logs:

```bash
cat /var/log/vm_health_monitor.log
```

Check generated reports:

```bash
ls /tmp/vm-health-reports/
```

---

# Cleanup

Old reports are automatically deleted after 30 days.

Cleanup command:

```bash
find "$REPORT_DIR" \
-name "vm_health_report_*.pdf" \
-mtime +30 \
-delete
```

---

# Troubleshooting

## SSH Connection Issue

Check SSH service:

```bash
ps -ef | grep sshd
```

Restart SSH:

```bash
/usr/sbin/sshd
```

---

## Email Issue

Check ssmtp configuration:

```bash
cat /etc/ssmtp/ssmtp.conf
```

Test email:

```bash
echo -e "Subject: Test\n\nHello" | ssmtp your_email@gmail.com
```

---

## PDF Generation Issue

Check wkhtmltopdf:

```bash
wkhtmltopdf --version
```

---

# Final Result

The automation provides:

✅ Multi-server monitoring  
✅ Automated metric collection  
✅ HTML dashboard report  
✅ PDF report generation  
✅ Email notification  
✅ Cron-based scheduling  
✅ Automatic old report cleanup  
