# Forensics
<!DOCTYPE html>
<html lang="en">
<body>

<h1>Using commands and simple tools to maintain the CIA (Confedentiality, Integrity, and Availability) of digital forensic evidence</h1>

<p>This was from and exercise early on in a Cyber Security Bootcamp, and I did not capture any screenshorts, therefore this page will be simple and contain the steps take and explain why they were done.<br>
Also this information was from my first submission, before I went back after grading, but didn't save the revised version. So bear with me and the errors that may be within. I did check most of the terminal commands on a VM and they seemed to work, but didn't have the information the supplied vm had, so that was the best I could do.<br></p>
<h2>DISCLAIMER: to reduce the likelyhood of this document being used to easily get through work that is meant to educate, and help people grow, I will be omitting the pseudo company name given for this assignment, as well as not mentioning the university or course itself.</h2>

<h3>The scenario</h3>



<p>I am playing the role of a Security Analyst at a financial company. It is my job (and those in my department I would assume) to check and ensure infroatmion remains accurate and available. I created a program (a simple script) to run two tools.
1. Log size management using (logrotate) to help automate the process of rotating logs and archiving them for investigation if needed.
2. Log auditing with (auditd) to track events, record the events, detect abuse or unauthorized activity, and create custom reports. <br><br> I was unable to find the complete prompt, as my access to this couse has ended at the time of making this repo. Therefore, there are gaps within the general story.</p>

<h4>Next I'll go into some of the details on setting up the environment (most of the log files were already created).</h4>

<h3>Step 1: Create, Extract, Compress, and Manage tar Backup Archives</h3>

<p>Command to extract the TarDocs.tar archive to the current directory:</p>
<pre><code>tar -xvf TarDocs.tar .
Or 
tar -xvf TarDocs.tar /home/sysadmin/Projects</code></pre>

<p>Command to create the Javaless_Doc.tar archive from the TarDocs/ directory, while excluding the TarDocs/Documents/Java directory:</p>
<pre><code>tar -cvf Javaless_Docs.tar --exclude=TarDocs/Documents/java TarDocs</code></pre>

<p>Command to ensure Java/ is not in the new Javaless_Docs.tar archive:</p>
<pre><code>tar -tvvf Javaless_Docs.tar | grep -w ‘java’</code></pre>
<p>Output gives one file with a java extension, but does not contain the TarDocs/Documents/java folder any more.</p>

<h4>Optional</h4>
<p>Command to create an incremental archive called logs_backup.tar.gz with only changed files to snapshot.file for the /var/log directory:</p>
<pre><code>tar -cvzf logs_backup.tar.gz --listed-incremental=snapshot.file /var/log</code></pre>

<h4>Critical Analysis Question</h4>
<p>Why wouldn't you use the options -x and -c at the same time with tar?</p>
<p>It doesn’t make sense to create (-c) while also extracting (-x) within the same command. It would be quite redundant and very unnecessary.</p>

<h3>Step 2: Create, Manage, and Automate Cron Jobs</h3>
<p>Cron job for backing up the /var/log/auth.log file:</p>
<pre><code>crontab -e 
(I created mine in nano, then added my executable file to the crontab, this is a concatenated version)
0 * * 6 3 sudo tar -cvf auth_backup.tar var/log/auth.log && sudo gzip auth_backup.tgz</code></pre>

<h3>Step 3: Write Basic Bash Scripts</h3>
<p>Brace expansion command to create the four subdirectories:</p>
<pre><code>mkdir ~/backups/{freemem,openlist,diskuse,freedisk}</code></pre>

<p>Paste your system.sh script edits:</p>
<pre><code>#!/bin/bash
free -h >> /home/sysadmin/backups/freemem/free_mem.txt
du -sh >> /home/sysadmin/backups/freedisk/free_disk.txt
lsof >> /home/sysadmin/backups/openlist/open_list.txt
df -Th >> /home/sysadmin/backups/freedisk/free_disk.txt</code></pre>

<p>Command to make the system.sh script executable:</p>
<pre><code>./system.sh</code></pre>

<h4>Optional</h4>
<p>Commands to test the script and confirm its execution:</p>
<pre><code>./system.sh
Then check the listed directories to ensure they have the correct information.</code></pre>

<p>Command to copy system to system-wide cron directory:</p>
<pre><code>sudo cp /home/system.sh /etc/cron.weekly</code></pre>

<h3>Step 4: Manage Log File Sizes</h3>
<p>Run <code>sudo nano /etc/logrotate.conf</code> to edit the logrotate configuration file.</p>
<p>Configure a log rotation scheme that backs up authentication messages to the /var/log/auth.log.</p>
<p>Add your config file edits:</p>
<pre><code>{
Weekly 
Tail -n 7
Notifempty
Compress
Delaycompress
Missingok
}</code></pre>

<h3>Optional Additional Challenge: Check for Policy and File Violations</h3>

<p>Command to verify <code>auditd</code> is active:</p>
<pre><code>sudo auditctl -l</code></pre>
<p>Output =</p>
<pre><code>-w /etc/shadow -p wra -k hashpass_audit
-w /var/log/auth.log -p wra -k hashpass_audit
-w /etc/passwd -p wra -k hashpass_audit</code></pre>

<p>Command to set number of retained logs and maximum log file size:</p>
<pre><code>sudo nano /etc/audit/auditd.conf</code></pre>
<pre><code>Max_log_file:= 35
Max_log:= 7</code></pre>

<p>Command using auditd to set rules for /etc/shadow, /etc/passwd, and /var/log/auth.log:</p>
<pre><code>sudo nano /etc/audit/rules.d/audit.rules
## First rule - delete all
-D

## Increase the buffers to survive stress events.
-b 8192

## This determine how long to wait in burst of events
--backlog_wait_time 0

## Set failure mode to syslog
-f 1

# Files being monitored and criteria for them.
-w /etc/shadow -p wra -k hashpass_audit
-w /var/log/auth.log -p wra -k hashpass_audit
-w /etc/passwd -p wra -k hashpass_audit</code></pre>

<p>Command to restart auditd:</p>
<pre><code>systemctl restart auditd</code></pre>

<p>Command to list all auditd rules:</p>
<pre><code>auditd -l</code></pre>

<p>Command to produce an audit report:</p>
<pre><code>sudo cat /var/log/audit/audit.log</code></pre>

<p>Command to create sudo user</p>
<pre></pre><code>sudo useradd attacker</code> and produce an audit report that lists account modifications:</pre>
<pre><code>sudo useradd attacker
sudo usermod -aG sudoer attacker</code></pre>

<p>Command to use auditd to watch /var/log/cron:</p>
<pre><code>-w /var/log/cron -p rwa -k selinux_changes</code></pre>

<p>Command to verify auditd rules:</p>
<pre><code>sudo auditctl -l</code></pre>

<h3>Optional (Research Activity): Perform Various Log Filtering Techniques</h3>

<p>Command to return journalctl messages with priorities from emergency to error:</p>
<pre><code>#!/bin/bash
sudo journalctl -p 0 ; sudo journalctl -p 1 ; sudo journalctl -p 2 ; sudo journalctl -p 3</code></pre>



</body>
</html>
