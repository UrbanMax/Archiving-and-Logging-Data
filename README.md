# Archiving-and-Logging-Data
This comprehensive guide walks through various system administration tasks, including archive management, cron job scheduling, bash scripting, log file management, auditd configuration, and log filtering techniques. Each step is carefully explained and includes commands to execute tasks efficiently.

Step 1: Create, Extract, Compress, and Manage tar Backup Archives
Command to extract the TarDocs.tar archive to the current directory:

tar xvf 'TarDocs (1).tar'

Command to create the Javaless_Doc.tar archive from the TarDocs/ directory, excluding the TarDocs/Documents/Java directory:

tar cvvf Javaless_Docs.tar --exclude="TarDocs/Documents/Java" TarDocs

Command to ensure Java/ is not in the new Javaless_Docs.tar archive:

tar -tvf Javaless_Docs.tar | grep 'Java'

Optional Command to create an incremental archive called logs_backup.tar.gz with only changed files for the /var/log directory:

tar --listed-incremental=snapshot.file -czvvf logs_backup.tar.gz /var/log

Critical Analysis Question:

You wouldn't use the options -x and -c at the same time with tar because -x (extract) is used to extract files from an archive, and -c (create) is used to create a new archive. Using them together would be contradictory.

Step 2: Create, Manage, and Automate Cron Jobs
Cron job for backing up the /var/log/auth.log file every Wednesday at 6:00 AM:

0 6 * * 3 tar -czf /auth_backup.tgz /var/log/auth.log

Step 3: Write Basic Bash Scripts
Brace expansion command to create the four subdirectories:

mkdir -p backups/{freemem,diskuse,openlist,freedisk}
system.sh script edits:


#!/bin/bash
# Free memory output to a free_mem.txt file
free -h > ~/backups/freemem/free_mem.txt

# Disk usage output to a disk_usage.txt file
du -h > ~/backups/diskuse/disk_usage.txt

# List open files to an open_list.txt file
lsof > ~/backups/openlist/open_list.txt

# Free disk space to a free_disk.txt file
df -h > ~/backups/freedisk/free_disk.txt

Command to make the system.sh script executable:

sudo chmod +x system.sh

Optional Command to test the script and confirm its execution:

./system.sh

Command to copy system.sh to the system-wide cron directory for weekly execution:

sudo cp system.sh /etc/cron.weekly/

Step 4. Manage Log File Sizes

Configure log rotation for /var/log/auth.log:

sudo nano /etc/logrotate.conf

Add the following configuration:

# system-specific logs may be configured here

/var/log/auth.log {
    weekly
    rotate 7
    notifempty
    compress
    delaycompress
    missingok
}

Optional Additional Challenge: Check for Policy and File Violations
Command to verify auditd is active:

sudo systemctl status auditd

Command to set number of retained logs and maximum log file size:

sudo nano /etc/audit/auditd.conf

Add these edits:

max_log_file = 50
num_logs = 10

Command to set auditd rules for specific files:


sudo nano /etc/audit/rules.d/audit.rules
Add these edits:

-w /etc/shadow -p rwa -k hashpass_audit
-w /etc/passwd -p rwa -k userpass_audit
-w /var/log/auth.log -p rwa -k authlog_audit

Command to restart auditd service:

sudo systemctl restart auditd

Command to list all auditd rules:

sudo auditctl -l

Command to produce an audit report:
sudo aureport

Command to create a user and produce an audit report for account modifications:

sudo useradd -m -s /bin/bash attacker

Command to watch /var/log/cron using auditd:

sudo auditctl -w /var/log/cron -p wra -k cron_log_changes

Command to verify auditd rules:

sudo auditctl -l

Optional (Research Activity): Perform Various Log Filtering Techniques
Command to return journalctl messages with priorities from emergency to error:

sudo journalctl -p err

Command to check disk usage of the system journal since the most recent boot:

journalctl --disk-usage

Command to remove all archived journal files except the most recent two:

sudo journalctl --vacuum-files=2

Command to filter log messages with priority levels between zero and two and save output:

sudo journalctl -p 0..2 --output cat > /home/sysadmin/Priority_High.txt

Command to automate the log filtering in a daily cron job:
0 0 * * * sudo journalctl -p 0..2 --output cat > /home/sysadmin//Priority_High.txt
