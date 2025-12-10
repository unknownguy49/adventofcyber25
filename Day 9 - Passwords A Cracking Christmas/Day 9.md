# Advent of Cyber 2025 - Day 9 - Passwords: A Cracking Christmas

## 1. Introduction

With time between Easter and Christmas being destabilised, the once-quiet systems of The Best Festival Company began showing traces of encrypted data buried deep within their servers. Sir Carrotbane, stumbled upon a series of locked PDF and ZIP files labelled “North Pole Asset List.” Rumours spread that these could contain fragments of Santa’s master gift registry, critical information that could help Malhare control the festive balance between both worlds.

Sir Carrotbane sets out to crack the encryption, learning how weak passwords can expose even the most guarded secrets. Can the Elves adapt fast and prevent their secrets from being discovered?
<br></br>

### Learning Objectives
---

- How password-based encryption protects files such as PDFs and ZIP archives.
- Why weak passwords make encrypted files vulnerable.
- How attackers use dictionary and brute-force attacks to recover passwords.
- A hands-on exercise: cracking the password of an encrypted file to reveal its contents.
- The importance of using strong, complex passwords to defend against these attacks.

A few simple points to remember:

- The strength of protection depends almost entirely on the password. Short or common passwords can be guessed; long, random passwords are far harder to break.
- Different file formats use different algorithms and key derivation methods. For example, PDF encryption and ZIP encryption differ in details (how the key is derived, salt use, number of hash iterations). That affects how easy or hard cracking is.
- Many consumer tools still support legacy or weak modes (particularly older ZIP encryption). That makes some encrypted archives much easier to attack than modern, well-implemented schemes.
- Encryption protects data confidentiality only. It does not prevent someone with access to the encrypted file from trying to guess the password offline.

To make it simple, encryption makes the contents unreadable unless the correct password is known. If the password is weak, an attacker can simply try likely passwords until one works.
  <br></br>

### Connecting to the Machine
---

Start your target machine by clicking the Start Machine button below. The machine will need about 2 minutes to fully boot. In case you can not see it, click the Show Split View button at the top of the page.

Alternatively, you can use the credentials below to connect to the target machine via SSH from your own THM VPN connected machine:
```
Username: ubuntu
Password: AOC2025Ubuntu!
IP address: 10.49.182.72
Connection via SSH: ssh ubuntu@10.49.182.72
```
<br></br>

## 2. Attacks Against Encrypted Files

### How Attackers Recover Weak Passwords
---

Attackers generally **do not break encryption algorithms** directly — modern cryptography is far too strong.
Instead, they attack the **password** protecting the encrypted file.

Two primary techniques are used:
#### **Dictionary Attacks**

* Use a **predefined list of passwords** (wordlist).
* Common lists: `rockyou.txt`, `common-passwords.txt`.
* Fast and effective because many people choose weak or predictable passwords.
* Wordlists often contain:

  * leaked passwords
  * common patterns
  * simple substitutions (password123, qwerty, etc.)

#### **Mask / Brute-Force Attacks**

Brute-force → tests every possible combination.
Downside: time grows exponentially with length/complexity.

Mask attacks → greatly reduce search space by **guessing known patterns**, e.g.:

```
?l?l?l?d?d
(three lowercase letters + two digits)
```

This balances speed + completeness.

#### **Practical Tips Used by Attackers (and Defenders Should Know)**

* Start with dictionary attacks (fast wins).
* If unsuccessful, create **targeted wordlists** (org names, project names, personal data).
* Use mask attacks for short or patterned passwords.
* Use **GPU acceleration** (hashcat) to speed up cracking.
* Monitor system resource usage — cracking is CPU/GPU intensive and detectable.

<br></br>

### Exercise: Cracking Protected Files
---

Files are located in the **Desktop** directory.

```
cd Desktop
```

#### **1. Confirm the File Type**

Use the `file` command:

```
file flag.pdf
file flag.zip
```

This determines whether you'll use:

* **PDF tools** (pdfcrack, john via pdf2john)
* **ZIP tools** (fcrackzip, john via zip2john)

#### **2. Tools Available**

| File Type | Tools                      |
| --------- | -------------------------- |
| PDF       | pdfcrack, pdf2john → john  |
| ZIP       | fcrackzip, zip2john → john |
| General   | hashcat, john              |

---

#### **3. Dictionary Attack (fastest method)**

#### **PDF Example – Using pdfcrack + rockyou.txt**

```
pdfcrack -f flag.pdf -w /usr/share/wordlists/rockyou.txt
```

pdfcrack will output the recovered password:

```
found user-password: naughtylist
```

#### **ZIP Example – Using john**

1. Convert ZIP to John-readable hash:

```
zip2john flag.zip > ziphash.txt
```

2. Run John:

```
john --wordlist=/usr/share/wordlists/rockyou.txt ziphash.txt
```

John will output something like:

```
winter4ever   (flag.zip/flag.txt)
```

Session complete.

<br></br>

### Detection of Cracking Activity
---

Offline cracking doesn't generate login failures, but **endpoint behaviour gives it away**.

### **Key telemetry sources:**
#### **Process Creation Signals**

Identify binaries:

* john
* hashcat
* fcrackzip
* pdfcrack
* zip2john
* pdf2john.pl
* 7z / qpdf

Look for command-line traits:

* `--wordlist`
* `--mask`
* `-a 3`
* `rockyou.txt`
* references to generated hash files

Watch for potfiles / state:

* `~/.john/john.pot`
* `~/.hashcat/hashcat.potfile`
* `john.rec`

### **GPU Artefacts**

GPU cracking is very loud:

* `nvidia-smi` shows long-running hashcat/john processes
* Consistent 90–100% GPU load
* High power draw + fan spikes
* Libraries load: `libcuda.so`, `OpenCL.dll`, etc.

### **Network Indicators**

While cracking is offline, attackers often:

* Download wordlists (rockyou, SecLists)
* Pull repos, tools, or GPU drivers
* Use package installs (`apt install john hashcat`)

### **Unusual File Reads**

Repeated heavy reads of:

* Wordlists
* Encrypted files
* Converted hash files

### **Sample Detection Rules**

**Sysmon (Windows)** – detect known cracking tools:

```
(ProcessName="C:\Program Files\john\john.exe" OR
 ProcessName="C:\Tools\hashcat\hashcat.exe" OR
 CommandLine="*pdf2john.pl*" OR
 CommandLine="*zip2john*")
```

**Linux auditd example:**

```
auditctl -w /usr/share/wordlists/rockyou.txt -p r -k wordlists_read
auditctl -a always,exit -F arch=b64 -S execve -F exe=/usr/bin/john -k crack_exec
auditctl -a always,exit -F arch=b64 -S execve -F exe=/usr/bin/hashcat -k crack_exec
```

**Sigma Rule Example:**

```
title: Password Cracking Tools Execution
id: 9f2f4d3e-4c16-4b0a-bb3a-7b1c6c001234
status: experimental
logsource:
  product: windows
  category: process_creation
detection:
  selection_name:
    Image|endswith:
      - '\john.exe'
      - '\hashcat.exe'
      - '\fcrackzip.exe'
      - '\pdfcrack.exe'
      - '\7z.exe'
      - '\qpdf.exe'
  selection_cmd:
    CommandLine|contains:
      - '--wordlist'
      - 'rockyou.txt'
      - 'zip2john'
      - 'pdf2john'
      - '--mask'
      - ' -a 3'
  condition: selection_name or selection_cmd
level: medium
```

### **Incident Response Playbook**

1. Isolate host if malicious activity is detected
2. Gather triage artefacts (process list, GPU stats, open files, memory dumps)
3. Preserve working dirs, wordlists, hashes, shell history
4. Review decrypted files → look for exfiltration/movement
5. Determine if activity was authorized
6. Rotate keys/passwords; enforce MFA
7. Provide user education; ensure safe sandboxes

<br></br>

## 3. Side Quest: Finding the Secret Key

### Enumeration
---
During enumeration of the target user’s home directory, I identified a suspicious hidden KeePass database:

```
ls -la
...
-rw------- 1 ubuntu ubuntu 419413 .Passwords.kdbx
```

KeePass databases (`.kdbx`) typically contain stored credentials, making this an immediate point of interest.

Verification:

```
file .Passwords.kdbx
```

Result:

```
Keepass password database 2.x KDBX
```

This confirmed it was a valid KeePass 2 database.

<br></br>

### Transferring the KeePass Database to My Local Machine
---
Since cracking would be more efficient locally, I copied the file from the target to my own system.

From my attacking machine:

```
scp ubuntu@<TARGET_IP>:/home/ubuntu/.Passwords.kdbx ~/Downloads/
```

The file successfully appeared in my local `~/Downloads` directory.

<br></br>

### Extracting the KeePass Hash for Cracking
---
Inside my local machine’s Downloads folder, I extracted the hash from the KeePass file:

```
keepass2john .Passwords.kdbx > kdbx.hash
```

Checking the extracted hash:

```
head -n 2 kdbx.hash
```

Hash extraction was successful.

<br></br>

### Cracking the KeePass Master Password
---
Using John the Ripper and the rockyou wordlist, I attempted a dictionary attack:

```
john --wordlist=/usr/share/wordlists/rockyou.txt kdbx.hash
```

John successfully cracked the password:

```
harrypotter (.Passwords)
```

The KeePass master password is:
**harrypotter**

To confirm:

```
john --show kdbx.hash
```
<br></br>

### Opening the KeePass Database
---
With the password recovered, I opened the KeePass database locally using the KeePassXC GUI:
```
keepassxc .Passwords.kdbx
```

I entered password:

```
harrypotter
```

The database opened successfully.

<br></br>

### Retrieving the Hidden Content
---
Inside the KeePass DB, I found:
* An **image entry**

It contained the hidden message required for the challenge:
**tit_for_tat**

<br></br>

## **Final Answers**

| Question                                        | Answer                            |
| ----------------------------------------------- | --------------------------------- |
| What is the flag inside the encrypted PDF?      | **THM{Cr4ck1ng_PDFs_1s_34$y}**    |
| What is the flag inside the encrypted ZIP file? | **THM{Cr4ck1n6_z1p$_1s_34$yyyy}** |
| Side Quest Key                                  | **tit_for_tat**                   |

<br></br>

## **Conclusion**

* Password cracking focuses on weak passwords, not breaking encryption algorithms.
* Dictionary and mask attacks are the most effective techniques.
* Tools like **john**, **hashcat**, **pdfcrack**, and **fcrackzip** make the process efficient.
* High GPU usage, repeated file reads, and known binary execution are strong behavioural indicators.
* Monitored environments can detect and respond to password-cracking attempts.

---