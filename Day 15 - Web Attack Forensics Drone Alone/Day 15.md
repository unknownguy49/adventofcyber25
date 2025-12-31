# Advent of Cyber 2025 - Day 15 - Web Attack Forensics: Drone Alone

## 1. Introduction

TBFC’s drone scheduler web UI is getting strange, long HTTP requests containing Base64 chunks. Splunk raises an alert: “Apache spawned an unusual process.” On some endpoints, these requests cause the web server to execute shell code, which is obfuscated and hidden within the Base64 payloads. For this room, your job as the Blue Teamer is to triage the incident, identify compromised hosts, extract and decode the payloads and determine the scope.

You’ll use Splunk to pivot between web (Apache) logs and host-level (Sysmon) telemetry.

Follow the investigation steps below; each corresponds to a Splunk query and investigation goal.

<br></br>

### Learning Objectives
---

- Detect and analyze malicious web activity through Apache access and error logs
- Investigate OS-level attacker actions using Sysmon data
- Identify and decode suspicious or obfuscated attacker payloads
- Reconstruct the full attack chain using Splunk for Blue Team investigation

<br></br>

### Connecting to the Machine
---

Start your target VM by clicking the Start Machine button below. The machine will need about 3 minutes to fully boot. Additionally, start your AttackBox by clicking the Start AttackBox button below. The AttackBox will start in split view. In case you can not see it, click the Show Split View button at the top of the page.

<br></br>

## 2. Web Attack Forensics

### Logging into Splunk
---

After starting the **AttackBox** and the **target machine**, allow around **3 minutes** for all services to fully boot.

Access the Splunk dashboard using Firefox:

```
http://MACHINE_IP:8000/en-US/app/search/search/
```

#### **Login Details**

* **Username:** provided in the lab
* **Password:** provided in the lab
* **Connection:** HTTP

Once authenticated, you will be redirected to the **Search & Reporting** page.

> ⚠️ Important: Adjust the **time range** to **“Last 7 days”** or **“All time”** to avoid missing events.

This investigation follows **Log McBlue**, a blue team analyst tracing the attacker’s actions through Splunk logs.

<br></br>

### Detect Suspicious Web Commands
---

The first step is to identify possible **command injection attempts** in Apache access logs.

#### **Splunk Query**

```
index=windows_apache_access 
(cmd.exe OR powershell OR "powershell.exe" OR "Invoke-Expression") 
| table _time host clientip uri_path uri_query status
```

#### **Purpose**

This query searches for:

* System command execution attempts
* PowerShell usage
* Suspicious parameters passed through web requests

The presence of these indicators suggests an attacker attempting to execute system commands via a vulnerable CGI script such as `hello.bat`.

#### **Base64 Analysis**

Some results contain **Base64-encoded PowerShell commands**.

Example encoded string:

```
VABoAGkAcwAgAGkAcwAgAG4AbwB3ACAATQBpAG4AZQAhACAATQBVAEEASABBAEEASABBAEEA
```

Decoding the string using a Base64 decoder reveals the attacker’s intent and confirms malicious activity.

<br></br>

### Inspect Apache Error Logs
---

Next, we investigate **Apache error logs** to confirm whether malicious requests reached backend execution.

#### **Splunk Query**

```
index=windows_apache_error 
("cmd.exe" OR "powershell" OR "Internal Server Error")
```

#### **View Setting**

Set **View: Raw** to inspect full error messages.

#### **Why This Matters**
* A **500 Internal Server Error** often indicates backend processing
* Errors involving `cmd.exe` or `powershell` suggest execution attempts
* Confirms whether attacks were blocked or partially executed

Requests like:

```
/cgi-bin/hello.bat?cmd=powershell
```

triggering errors strongly indicate exploitation attempts.

<br></br>

### Trace Suspicious Process Creation
---

To confirm OS-level execution, we pivot to **Sysmon logs**.

#### **Splunk Query**

```
index=windows_sysmon ParentImage="*httpd.exe"
```

#### **Analysis**

This query identifies processes spawned by Apache (`httpd.exe`).

#### **Why This Is Critical**
Apache should **not** spawn system utilities.

Suspicious example:

```
ParentImage = C:\Apache24\bin\httpd.exe
Image        = C:\Windows\System32\cmd.exe
```

This confirms:

* Command injection succeeded
* Web server executed OS-level commands
* The attacker breached the application layer into the OS

<br></br>

### Confirm Attacker Enumeration Activity

---

After gaining execution, attackers often run **reconnaissance commands**.

#### **Splunk Query**

```
index=windows_sysmon *cmd.exe* *whoami*
```

#### **Purpose**

The `whoami` command is commonly used to:

* Identify the running user
* Determine privilege level
* Assess next attack steps

Finding this confirms **post-exploitation activity**, proving the attacker successfully executed commands on the host.

<br></br>

### Identify Encoded PowerShell Payloads
---

Finally, we search for **encoded PowerShell execution**, commonly used to evade detection.

#### **Splunk Query**

```
index=windows_sysmon 
Image="*powershell.exe" 
(CommandLine="*enc*" OR CommandLine="*-EncodedCommand*" OR CommandLine="*Base64*")
```

#### **Outcome**

* If no results appear, encoded payloads were blocked
* If results exist, decoding reveals attacker intent

In this case, no execution was confirmed — meaning the encoded payload did not successfully run.

<br></br>

## **Final Answers**

| Question                                                                       | Answer             |
| ------------------------------------------------------------------------------ | ------------------ |
| What is the reconnaissance executable file name?                               | **whoami.exe**     |
| What executable did the attacker attempt to run through the command injection? | **PowerShell.exe** |

<br></br>

## **Conclusion**

* Splunk effectively correlates **web logs** and **Sysmon events**
* Apache access logs reveal command injection attempts
* Error logs confirm backend interaction
* Sysmon proves OS-level command execution
* `whoami.exe` confirms attacker reconnaissance
* Encoded PowerShell detection helps identify obfuscated payloads
* This investigation demonstrates a full attack chain from **web request → OS execution**

---