# Detecting Cobalt Strike's PSExec

## Introduction

Cobalt Strike's psexec command is an implementation of the popular PsExec tool, which is a part of Microsoft's Sysinternals Suite. It's a lightweight telnet-replacement that lets you execute processes on other systems. Cobalt Strike's version is utilized to execute payloads on remote systems, as part of the post-exploitation process.

When the psexec command is invoked within Cobalt Strike, the following steps occur:

1. **Service Creation:** The tool first creates a new service on the target system. This service is responsible for executing the desired payload. The service is typically created with a random name to avoid easy detection.
2. **File Transfer:** Cobalt Strike then transfers the payload to the target system, often to the ADMIN$ share. This is typically done using the SMB protocol.
3. **Service Execution:** The newly created service is then started, which in turn executes the payload. This payload can be a shellcode, an executable, or any other file type that can be executed.
4. **Service Removal:** After the payload has been executed, the service is stopped and deleted from the target system to minimize traces of the intrusion.
5. **Communication:** If the payload is a beacon or another type of backdoor, it will typically establish communication back to the Cobalt Strike team server, allowing for further commands to be sent and executed on the compromised system.

Cobalt Strike's psexec works over port 445 (SMB), and it requires local administrator privileges on the target system. Therefore, it's often used after initial access has been achieved and privileges have been escalated.



## How Cobalt Strike PSExec Traffic Looks Like

<img width="1107" height="1071" alt="image" src="https://github.com/user-attachments/assets/b074a6cf-d306-43ed-9096-0fb1b60a2779" />

**Image Source:** https://thedfirreport.com/2021/08/29/cobalt-strike-a-defenders-guide/

## Related Evidence

- **Related Directory:** `/home/htb-student/module_files/cobalt_strike_psexec`
- **Related Splunk Index:** `cobalt_strike_psexec`
- **Related Splunk Sourcetype:** `bro:smb_files:json`


## Detecting Cobalt Strike's PSExec With Splunk & Zeek Logs

Now let's explore how we can identify Cobalt Strike's PSExec, using Splunk and Zeek logs.

```shellsession id="m4x7qn"
index="cobalt_strike_psexec"
sourcetype="bro:smb_files:json"
action="SMB::FILE_OPEN" 
name IN ("*.exe", "*.dll", "*.bat")
path IN ("*\\c$", "*\\ADMIN$")
size>0
```

<img width="751" height="361" alt="image" src="https://github.com/user-attachments/assets/02002881-3aa8-42f6-9ba7-4d6107c8a92d" />

## 🔑 Key Takeaway

Cobalt Strike's PSExec uses SMB over port 445 to transfer and execute payloads on remote systems through temporary services. The Splunk query searches for executable files opened through SMB, specifically `.exe`, `.dll`, and `.bat` files transferred to administrative shares such as `C$` and `ADMIN$`.
