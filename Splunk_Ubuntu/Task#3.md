### Objective

The objective of this task is to monitor and detect unauthorized modifications, deletions, or attribute changes in sensitive directories, such as `/etc/`, using **Auditd** for real-time monitoring and **Splunk** for centralized log analysis. By simulating potential threats, such as file tampering or deletion, students will gain hands-on experience in setting up file integrity monitoring, analyzing logs for forensic purposes, and implementing incident response strategies to secure critical system files.

- **Detailed level**: Full
- **Difficulty level**: Medium

What is Auditd?

- **System Activity Logging**: Auditd monitors and records system events like file access, user actions, and configuration changes.
- **Security and Compliance**: Helps meet compliance standards by maintaining a detailed log of critical events.
- **Customizable Rules**: Allows administrators to define specific audit rules to monitor sensitive files, processes, or activities.
- **Real-time Alerts**: Supports real-time notifications for suspicious or unauthorized actions.

### **Tools Used**

1. **Auditd**: For monitoring file integrity.
2. **Splunk Universal Forwarder**: For forwarding logs to Splunk.
3. **Splunk**: For analyzing and visualizing logs.

### **Step-by-Step Guide**

---

### **Step 1: Prerequisites**

1. An **Ubuntu Server** with sudo privileges.
2. **Auditd** installed for file integrity monitoring.
3. **Splunk Universal Forwarder** installed and configured to forward logs.
4. A directory or file to monitor (e.g., `/etc/passwd` or `/etc/`).

---

### **Step 2: Install and Configure Auditd**

1. **Install Auditd**:

   ```bash

   sudo apt update
   sudo apt install auditd -y

   ```

2. **Start and Enable Auditd**:

   ```bash
   systemctl start auditd
   systemctl enable auditd

   ```

3. **Verify Auditd Status**:

   ```bash

   sudo systemctl status auditd

   ```

---

### **Step 3: Set Up File Monitoring Rules**

1. **Add Audit Rules for File Integrity Monitoring**:

   - Open the audit rules configuration file:

     ```bash

     sudo nano /etc/audit/rules.d/audit.rules

     ```

   - Add the following rule to monitor `/etc/` for any changes:

     ```bash

     -w /etc/ -p wa -k file_integrity

     ```

     - `w`: Watch the specified directory.
     - `p`: Permissions to monitor:
       - `w`: Write.
       - `a`: Attribute changes.
     - `k`: Keyword to tag events for easier searching (e.g., `file_integrity`).

2. **Reload Auditd**:

   ```bash
   sudo service auditd restart

   ```

3. **Verify Rules**:

   ```bash

   sudo auditctl -l

   ```

---

### **Step 4: Install and Configure Splunk Universal Forwarder**

1. **Install Splunk Universal Forwarder**:
   - Ensure it is installed and running propoerly
2. **Configure Splunk to Monitor Auditd Logs**:

   - Edit the `inputs.conf` file:

     ```bash

     sudo nano /opt/splunkforwarder/etc/system/local/inputs.conf

     ```

   - Add the following entry:

     ```bash
     [monitor:///var/log/audit/audit.log]
     sourcetype = auditd
     index = linux_file_integrity
     ```

   - Save and restart Splunk Forwarder:

     ```bash

     sudo /opt/splunkforwarder/bin/splunk restart

     ```

3. **Verify Log Forwarding**:

   - Use this query in Splunk to search for Auditd logs:

     ```

     index=linux_file_integrity sourcetype=auditd

     ```

### **Step 5: Simulate Unauthorized Changes**

1. **Modify a File**:

   - Make a change to `/etc/passwd` to simulate unauthorized modification:Add or remove a line, then save and exit.

     ```bash
     sudo nano /etc/passwd

     ```

2. **Delete a File**:

   - Delete a file in `/etc/` (ensure it is non-critical for this test):

     ```bash
     sudo rm /etc/testfile

     ```

3. **List Audit Logs**:

   - Use `ausearch` to find log entries for the simulated actions:

     ```bash
     sudo ausearch -k file_integrity

     ```

---

### **Step 6: Install Auditd for Linux App from Splunkbase**

- Go to More Apps
- Search for **Auditd for Linux App**
- Install the app

### **Step 6: Analyze Logs in Splunk**

1. **Search for File Integrity Events**:
   - Run the following query in Splunk:
     ```
     index=linux_logs sourcetype=auditd key="file_integrity"
     ```
   - This query will filter events tagged with the `file_integrity` keyword.
   - Note: Its all possible because of Aud
2. **Investigate Specific Events**:

   - Analyze the logs for unauthorized modifications. Example:

     ```bash
     type=SYSCALL msg=audit(1667829100.123:123): arch=c000003e syscall=2 success=yes exit=0 a0=7ffc12345 a1=80400 a2=1 items=2 ppid=2345 pid=4567 auid=1000 uid=0 gid=0 euid=0
     ```

3. **Visualize Events**:
   - Create a dashboard to monitor:
     - Files most frequently modified.
     - Users responsible for the changes.
     - Time trends for integrity violations.

---

### **Step 7: Incident Response**

1. **Identify the User**:
   - Extract the user who made the changes from the log.
   - Example: `auid=1000` to a specific user ID.
2. **Restore the File**:
   - If critical files are altered or deleted, restore them from a backup.
3. **Implement Additional Controls**:
   - Set permissions to restrict access to sensitive files:

### \*\*Step 8: Incident Response (IR)

1. **Identify the culprit**:

   - Map `auid` to username:

   ```
   id 1000
   ```

2. **Contain & recover**
   - If `/etc/passwd` modified → restore from backup.
   - If file deleted → recover from `/var/backups/`.
3. **Harden controls**

   - Restrict access to sensitive files:

   ```
   sudo chmod 644 /etc/passwd
   sudo chown root:root /etc/passwd
   ```

   - Enable SELinux/AppArmor policies for extra protection.


