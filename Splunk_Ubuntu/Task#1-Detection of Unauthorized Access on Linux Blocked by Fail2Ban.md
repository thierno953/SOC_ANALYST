**Objective**: Set up monitoring for unauthorized access attempts, trigger Fail2Ban to block malicious IPs, and analyze logs in Splunk.

- `Detailed level`: High
- `Difficulty level`: Medium

---

### **Step-by-Step Instructions**

### **Step 1: Prerequisites**

1. Ensure you have an **Ubuntu Server** ready for this task.
2. Install the necessary tools:
   - Fail2Ban for SSH protection.
   - Splunk Universal Forwarder for log forwarding.

---

### **Step 2: Install and Configure Fail2Ban**

1. **Install Fail2Ban**:

   ```bash

   sudo apt update
   sudo apt install fail2ban -y

   systemctl restart fail2ban
   fail2ban-client status sshd
   ls /etc/fail2ban/filter.d/recidive.conf
   ```

2. **Configure Fail2Ban for SSH**:

   - Open the Fail2Ban configuration file:

     ```bash

     sudo nano /etc/fail2ban/jail.local

     ```

   - Add the following lines to protect the SSH service:

     ```
     [DEFAULT]
     ignoreip = 127.0.0.1/8 ::1 <TON_IP_FIXE>
     dbfile = /var/lib/fail2ban/fail2ban.sqlite3
     logtarget = SYSLOG

     [sshd]
     enabled = true
     port = ssh
     logpath = /var/log/auth.log
     maxretry = 3
     bantime = 600
     findtime = 600

     [recidive]
     enabled  = true
     filter   = recidive
     logpath  = /var/log/fail2ban.log
     bantime  = 604800
     findtime = 86400
     maxretry = 5
     action   = iptables-allports[name=recidive]

     ```

   - Save and close the file.

3. **Add Restart Fail2Ban** to apply changes:

   ```bash
   /opt/splunkforwarder/bin/splunk add monitor /var/log/fail2ban.log
   sudo systemctl restart fail2ban

   ```

4. **Verify Fail2Ban Status**:

   ```bash
   sudo fail2ban-client reload
   sudo fail2ban-client status
   sudo fail2ban-client status recidive
   ```

   Look for the `sshd` jail in the output.

5. Create index and sourcetype in Splunk platform

   ```css
   Create index and sourcetype and name them as fail2ban_logs
   ```

---

### **Step 3: Simulate Unauthorized SSH Access**

1.  ```bash
    ssh user@<target-ip>

    ```

### **Step 3: Simulate a Brute-Force Attack with Hydra**

1. **Prepare a wordlist** with common passwords:

   - Use a pre-existing wordlist (e.g., `/usr/share/wordlists/rockyou.txt`) or create a custom one:

     ```bash

     echo -e "password123\nadmin123\n12345678\ntestpass" > passwords.txt

     ```

2. **Run Hydra** to perform the brute-force attack:

   ```bash

   hydra -l testuser -P passwords.txt <target-ip> ssh

   ```

   - Replace `<target-ip>` with the IP address of the Ubuntu server.

3. **Observe Fail2Ban Action**:

   - As the attack progresses, Fail2Ban will detect multiple failed login attempts and block the attacking IP.
   - Check Fail2Ban logs on the server:

     ```bash

     tail -f /var/log/fail2ban.log

     ```

4. **Monitor Fail2Ban Logs**:

   ```bash

   tail -f /var/log/fail2ban.log
   sudo fail2ban-client status sshd
   sudo fail2ban-client status recidive
   watch -n 1 "fail2ban-client status sshd | grep 'Banned IP list'"

   ```

   - Check for entries indicating that Fail2Ban has banned the attacking IP.

---

### **Step 4: Configure Splunk for Log Monitoring**

1. Make sure your Universal forwarder is up and running on Ubuntu server.
2. **Set up log forwarding for Fail2Ban**:

   - Create an input configuration:

     ```bash

     sudo nano /opt/splunkforwarder/etc/system/local/inputs.conf

     ```

   - Add the following configuration to monitor Fail2Ban logs:

     ```
     [monitor:///var/log/syslog]
     disabled = false
     index = linux_os_logs
     sourcetype = syslog
     host = server-ssh-prod

     [monitor:///var/log/auth.log]
     disabled = false
     index = security_incidents
     sourcetype = linux_secure
     whitelist = Failed password|Invalid user|authentication failure
     host = server-ssh-prod

     [monitor:///var/log/fail2ban.log]
     disabled = false
     index = fail2ban_logs
     sourcetype = fail2ban
     host = ubuntu-prod-ssh

     ```

   - Save and restart the Splunk Universal Forwarder:

     ```bash

     sudo /opt/splunkforwarder/bin/splunk restart

     ```

---

### **Step 5: Analyze Logs in Splunk**

1. **Log in to Splunk** and navigate to the Search and Reporting App.
2. Use the following SPL query to identify blocked IPs:

   ```

   index=linux_logs sourcetype=fail2ban action="ban"

   index="security_incidents" sourcetype="linux_secure" "Failed password"

   (index="fail2ban_logs" OR index="security_incidents")
   | rex field=_raw "Ban\s(?<banned_ip>\d+\.\d+\.\d+\.\d+)"
   | stats count AS attempts BY banned_ip
   | sort -attempts

   index="fail2ban_logs"
   | rex field=_raw "Ban\s(?<banned_ip>\d+\.\d+\.\d+\.\d+)"
   | timechart span=1h count BY banned_ip

   ```

3. **Visualize Results**:
   - Create a dashboard to monitor trends of failed login attempts and banned IPs.

---

### **Step 6: Validate Incident Response**

1. **Validate Banned IPs**:

   ```bash

   sudo fail2ban-client status sshd

   ```

   Verify the attacking IP is listed as banned.

2. **Unban IPs (if needed)**:

   ```bash

   sudo fail2ban-client set sshd unbanip <banned-ip>

   ```
