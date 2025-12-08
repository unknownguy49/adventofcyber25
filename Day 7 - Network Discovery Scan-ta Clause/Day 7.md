# Advent of Cyber 2025 - Day 7 - Network Discovery: Scan-ta Clause

## 1. Introduction

Christmas preparations are delayed - HopSec has breached our QA environment and locked us out! Without it, the TBFC projects can't be tested, and our entire SOC-mas pipeline is frozen. To make things worse, the server is slowly transforming into a twisted EAST-mas node.

Can you uncover HopSec's trail, find a way back into tbfc-devqa01, and restore the server before the bunny's takeover is complete? For this task, you'll need to check every place to hide, every opened port that bunnies left unprotected. Good luck!

<br></br>

### Learning Objectives

---

- Learn the basics of network service discovery with Nmap
- Learn core network protocols and concepts along the way
- Apply your knowledge to find a way back into the server
  <br></br>

### Connecting to the Machine

---

Start your target machine by clicking the Start Machine button below. The machine will need about 2 minutes to fully boot. Additionally, start your AttackBox by clicking the Start AttackBox button below. The AttackBox will start in split view. In case you can not see it, click the Show Split View button at the top of the page.
<br></br>

## 2. Discover Network Services

### Discovering Exposed Services

---

Even though access to the QA server was lost, we still know its IP address: **10.49.147.14**. With that, we can counterattack, enumerate services, and take back control from the bad bunnies.

1. Identify the target → `tbfc-devqa01 (10.49.147.14)`
2. Scan the host for open ports
3. Explore accessible services (SSH, HTTP, hidden ports)
4. Exploit exposed services to retrieve **three secret keys**
5. Use the keys to unlock the web admin panel
6. Perform on-host enumeration to reveal the final flag

Throughout the task, **keep track of all three key parts**, in format:

```
KEYNAME:KEY
```

<br></br>

#### **The Simplest Port Scan**

---

Run a basic Nmap scan:

```
nmap 10.49.147.14
```

Results:

```
22/tcp   open  ssh
80/tcp   open  http
```

Meaning:

- **SSH** (port 22) → remote management
- **HTTP** (port 80) → defaced web page (`Pwned by HopSec`)

Accessible at:
`http://10.49.147.14`

<br></br>

#### **Scanning the Whole Range of Ports**

---

Scan all 0–65535 TCP ports with service banners:

```
nmap -p- --script=banner 10.49.147.14
```

Results:

```
22/tcp      open ssh
80/tcp      open http
21212/tcp   open trinket-agent (vsFTPd banner)
25251/tcp   open TBFC maintd v0.2
```

Findings:

- **21212/tcp → FTP server (vsFTPd 3.0.5)**
- **25251/tcp → Custom TBFC maintenance daemon**

We now explore both.

<br></br>

#### **Enumerating the FTP Server (Port 21212)**

---

Connect anonymously:

```
ftp 10.49.147.14 21212
```

List files:

```
ls
-rw-r--r--  tbfc_qa_key1
```

Retrieve the file:

```
get tbfc_qa_key1 -
```

→ **First key part recovered:** `3aster_`

<br></br>

#### **Enumerating the TBFC Custom Service (Port 25251)**

---

Since it's a custom TCP service, use **Netcat**:

```
nc -v 10.49.147.14 25251
```

Output:

```
TBFC maintd v0.2
Type HELP for commands.
```

Enter:

```
GET KEY
```

→ **Second key part recovered:** `15_th3_`

Press **CTRL+C** to exit.

<br></br>

#### **UDP Scanning for Hidden Services**

---

Until now, only TCP ports were scanned. Run a UDP scan:

```
nmap -sU 10.49.147.14
```

Open port detected:

```
53/udp open domain
```

Port **53** = DNS service.

Query DNS TXT records using `dig`:

```
dig @10.49.147.14 TXT key3.tbfc.local +short
```

→ **Third key part recovered:** `n3w_xm45`

Now we have all three keys.

<br></br>

#### **Accessing the Web Admin Panel**

---

Visit the website:

`http://10.49.147.14`

Combine the three keys:

```
3aster_15_th3_n3w_xm45
```

Enter into the secret admin console to gain host access.

<br></br>

### On-Host Service Discovery (Admin Console)

---

Inside the console, run:

```
ss -tunlp
```

or `netstat` on older systems.

You will see:

- FTP on **21212**
- TBFC app on **25251**
- SSH on **22**
- HTTP on **80**
- DNS on **53**
- MySQL on **3306**
- Additional localhost-only services (8000, 7681, etc.)

Since we are local, we can access the MySQL database without a password.

#### **Exploring MySQL**

```
mysql -D tbfcqa01 -e "show tables;"
```

```
flags
```

Retrieve the final flag:

```
mysql -D tbfcqa01 -e "select * from flags;"
```

→ **THM{4ll_s3rvice5_d1sc0vered}**

<br></br>

## Final Answers

| Question                             | Answer                           |
| ------------------------------------ | -------------------------------- |
| What evil message is on the website? | **Pwned by HopSec**              |
| First key part (FTP)?                | **3aster\_**                     |
| Second key part (TBFC app)?          | **15*th3***                      |
| Third key part (DNS)?                | **n3w_xm45**                     |
| Which port was MySQL running on?     | **3306**                         |
| Final flag from database?            | **THM{4ll_s3rvice5_d1sc0vered}** |

<br></br>

## Conclusion

- Initial Nmap scan identified basic services (SSH, HTTP).
- Full-range Nmap scan uncovered hidden services (FTP on 21212, TBFC app on 25251).
- Anonymous FTP login leaked **key 1**.
- TBFC custom daemon leaked **key 2**.
- UDP scan + DNS TXT record revealed **key 3**.
- Combined keys allowed entry into secret admin panel.
- On-host enumeration exposed a local MySQL database containing the final flag.
- This exercise demonstrates how exposed services across TCP/UDP can leak sensitive data if not secured.

---
