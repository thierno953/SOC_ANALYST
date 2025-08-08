# Wazuh + VirusTotal Integration for Malware Detection and Auto-Removal

- Python installed on the Windows endpoint: [https://www.python.org/downloads/windows/](https://www.python.org/downloads/windows/)

- Wazuh documentation reference: [https://documentation.wazuh.com/current/proof-of-concept-guide/detect-remove-malware-virustotal.html#windows-endpoint](https://documentation.wazuh.com/current/proof-of-concept-guide/detect-remove-malware-virustotal.html#windows-endpoint)

- Virustotal: [https://www.virustotal.com/gui/home/upload](https://www.virustotal.com/gui/home/upload)

#### Enable Real-time Monitoring of the Downloads Folder

- Edit `ossec.conf` on the Windows Agent:

```sh
<directories realtime="yes">C:\Users\Administrator\Downloads</directories>
```

#### Compile the Response Script on the Windows Agent

- Install `pyinstaller` and check version:

```sh
PS C:\Users\Administrator> pip install pyinstaller
PS C:\Users\Administrator> pyinstaller --version
```

- Save the following script as `"remove-threat.py"`:

```sh
# Copyright (C) 2015-2025, Wazuh Inc.
# All rights reserved.

import os, sys, json, datetime, stat, pathlib

if os.name == 'nt':
    LOG_FILE = "C:\\Program Files (x86)\\ossec-agent\\active-response\\active-responses.log"
else:
    LOG_FILE = "/var/ossec/logs/active-responses.log"

ADD_COMMAND, DELETE_COMMAND, CONTINUE_COMMAND, ABORT_COMMAND = 0, 1, 2, 3
OS_SUCCESS, OS_INVALID = 0, -1

class message:
    def __init__(self):
        self.alert = ""
        self.command = 0

def write_debug_file(ar_name, msg):
    with open(LOG_FILE, mode="a") as log_file:
        log_file.write(f"{datetime.datetime.now():%Y/%m/%d %H:%M:%S} {ar_name}: {msg}\n")

def setup_and_check_message(argv):
    input_str = next(sys.stdin, "")
    msg_obj = message()

    try:
        data = json.loads(input_str)
    except ValueError:
        write_debug_file(argv[0], "Failed to decode JSON input.")
        msg_obj.command = OS_INVALID
        return msg_obj

    msg_obj.alert = data
    command = data.get("command")
    msg_obj.command = ADD_COMMAND if command == "add" else DELETE_COMMAND if command == "delete" else OS_INVALID

    if msg_obj.command == OS_INVALID:
        write_debug_file(argv[0], f"Invalid command: {command}")

    return msg_obj

def send_keys_and_check_message(argv, keys):
    keys_msg = json.dumps({
        "version": 1,
        "origin": {"name": argv[0], "module": "active-response"},
        "command": "check_keys",
        "parameters": {"keys": keys}
    })
    write_debug_file(argv[0], keys_msg)
    print(keys_msg)
    sys.stdout.flush()

    input_str = next(sys.stdin, "")
    try:
        data = json.loads(input_str)
    except ValueError:
        write_debug_file(argv[0], "Failed to decode JSON response.")
        return OS_INVALID

    action = data.get("command")
    return CONTINUE_COMMAND if action == "continue" else ABORT_COMMAND if action == "abort" else OS_INVALID

def secure_delete_file(filepath_str, ar_name):
    filepath = pathlib.Path(filepath_str)

    if '::' in filepath_str:
        raise Exception(f"Refusing to delete ADS/NTFS stream: {filepath_str}")

    if os.path.islink(filepath):
        raise Exception(f"Refusing to delete symbolic link: {filepath}")

    attrs = os.lstat(filepath).st_file_attributes
    if attrs & stat.FILE_ATTRIBUTE_REPARSE_POINT:
        raise Exception(f"Refusing to delete reparse point: {filepath}")

    resolved_filepath = filepath.resolve()
    if not resolved_filepath.is_file():
        raise Exception(f"Target is not a regular file: {resolved_filepath}")

    os.remove(resolved_filepath)

def main(argv):
    write_debug_file(argv[0], "Started")
    msg = setup_and_check_message(argv)

    if msg.command < 0:
        sys.exit(OS_INVALID)

    if msg.command == ADD_COMMAND:
        alert = msg.alert["parameters"]["alert"]
        keys = [alert["rule"]["id"]]
        action = send_keys_and_check_message(argv, keys)

        if action != CONTINUE_COMMAND:
            if action == ABORT_COMMAND:
                write_debug_file(argv[0], "Aborted")
                sys.exit(OS_SUCCESS)
            write_debug_file(argv[0], "Invalid command")
            sys.exit(OS_INVALID)

        try:
            file_path = alert["data"]["virustotal"]["source"]["file"]
            if os.path.exists(file_path):
                secure_delete_file(file_path, argv[0])
                write_debug_file(argv[0], json.dumps(msg.alert) + " Successfully removed threat")
            else:
                write_debug_file(argv[0], f"File does not exist: {file_path}")
        except Exception as e:
            write_debug_file(argv[0], f"{json.dumps(msg.alert)}: Error removing threat: {str(e)}")
    else:
        write_debug_file(argv[0], "Invalid command")

    write_debug_file(argv[0], "Ended")
    sys.exit(OS_SUCCESS)

if __name__ == "__main__":
    main(sys.argv)
```

#### Compile the script:

```sh
PS C:\Users\Administrator> cd .\Desktop
PS C:\Users\Administrator\Desktop> pyinstaller -F remove-threat.py
```

#### Move into the generated `dist/` directory:

```sh
PS C:\Users\Administrator\Desktop> cd dist
```

#### Restart the Wazuh agent:

```sh
PS C:\Users\Administrator> Restart-Service -Name wazuh
```

## Wazuh Manager Configuration

- Edit `/var/ossec/etc/ossec.conf` on the Wazuh Manager:

```sh
<integration>
  <name>virustotal</name>
  <api_key><YOUR_VIRUS_TOTAL_API_KEY></api_key>
  <group>syscheck</group>
  <alert_format>json</alert_format>
</integration>

<command>
  <name>remove-threat</name>
  <executable>remove-threat.exe</executable>
  <timeout_allowed>no</timeout_allowed>
</command>

<active-response>
  <disabled>no</disabled>
  <command>remove-threat</command>
  <location>local</location>
  <rules_id>87105</rules_id>
</active-response>
```

![WAZUH](/Wazuh/assets/16.png)

#### Define custom detection rules `/var/ossec/etc/rules/local_rules.xml`:

```sh
<group name="virustotal">
  <rule id="100092" level="12">
    <if_sid>657</if_sid>
    <match>Successfully removed threat</match>
    <description>$(parameters.program) removed threat at $(parameters.alert.data.virustotal.source.file)</description>
  </rule>

  <rule id="100093" level="12">
    <if_sid>657</if_sid>
    <match>Error removing threat</match>
    <description>Error removing threat at $(parameters.alert.data.virustotal.source.file)</description>
  </rule>
</group>
```

#### Restart Wazuh Manager:

```sh
sudo systemctl restart wazuh-manager
```

![WAZUH](/Wazuh/assets/17.png)

#### Test with EICAR (Standard Test File)

- Download the EICAR test file (safe, non-malicious):

```sh
PS C:\Users\Administrator> Invoke-WebRequest -Uri https://secure.eicar.org/eicar.com.txt -OutFile eicar.txt
```

- Move it to the monitored folder:

```sh
PS C:\Users\Administrator> cp .\eicar.txt C:\Users\Administrator\Downloads
```

- Once the file is detected and flagged by **VirusTotal**, Wazuh will log the event and trigger the `remove-threat.exe` script to automatically delete the file from the system.

![WAZUH](/Wazuh/assets/18.png)
