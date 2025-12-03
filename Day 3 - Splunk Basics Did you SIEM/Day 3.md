# Advent of Cyber 2025 - Day 3 - Splunk Basics: Did you SIEM?

## 1. Introduction

It’s almost Christmas in Wareville, and the team of The Best Festival Company (TBFC) is busy preparing for the big celebration. Everything is running smoothly until the SOC dashboard flashes red. A ransom message suddenly appears: 

<img src="https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1763099571417.png">
<br></br>
The message comes from King Malhare, the jealous ruler of HopSec Island, who’s tired of Easter being forgotten. He’s sent his Bandit Bunnies to attack TBFC’s systems and turn Christmas into his new holiday, EAST-mas.
<br></br>
With McSkidy missing and the network under attack, the TBFC SOC team will utilize Splunk to determine how the ransomware infiltrated the system and prevent King Malhare’s plan from being compromised before Christmas.
<br></br>

### Learning Objectives

---

- Ingest and interpret custom log data in Splunk
- Create and apply custom field extractions
- Use Search Processing Language (SPL) to filter and refine search results
- Conduct an investigation within Splunk to uncover key insights
  <br></br>

### Connecting to the Machine

---

Start your VM by clicking the Start Machine button below. The machine will need about 2 -3 minutes to fully boot. Once the machine is up and running, you can connect to the Splunk SIEM by visiting https://10-49-184-237.reverse-proxy.cell-prod-ap-south-1b.vm.tryhackme.com in your browser.

Note: If you get a 502 error when accessing the link, please give the Splunk instance more time to fully boot up.

<br></br>

## 2. Log Analysis with Splunk

### Exploring the Logs
---
On the Splunk instance, the data has been pre-ingested for us to investigate the incident. On the Splunk interface, click on Search & Reporting on the left panel.

On the next page, type:

```
index=main
```

in the search bar to show all ingested logs. Select **All time** as the time frame.

After running the query, two pre-ingested datasets appear.
We can verify this by clicking the **sourcetype** field.

**The two datasets are:**

* **web_traffic** – events related to web connections
* **firewall_logs** – firewall events (allowed/blocked), webserver IP: **10.10.1.15**

<br></br>

### Initial Triage
---
Run:
```
index=main sourcetype=web_traffic
```

Breakdown:

* Retrieves all web_traffic logs from main index
* Time range currently set to All time
* Timeline histogram shows distribution of 17,172 events
* Selected fields: host, source, sourcetype
* Interesting fields: Splunk-generated (#date_hour), plus user_agent, path, client_ip
* Event details show extracted fields such as user_agent, path, status, client_ip

<br></br>

### Visualizing the Logs Timeline
---
Chart the total event count per day:
```
index=main sourcetype=web_traffic | timechart span=1d count
```
Then click **Visualization** for graph view.

To show results in descending order:
```
index=main sourcetype=web_traffic | timechart span=1d count | sort by count | reverse
```

A clear spike appears — the period when King Malhare launched the attack.

<br></br>


### Anomaly Detection
---

### User Agent

Click **user_agent** in the left panel.
Legitimate agents (Mozilla variants) are mixed with large volumes of suspicious user agents.

### client_ip

Inspect **client_ip** — one IP clearly stands out.

### path
Inspect **path** — reveals numerous suspicious URIs.

<br></br>

### Filtering out Benign Values
---
Exclude known normal browser traffic:

```
index=main sourcetype=web_traffic user_agent!=*Mozilla* user_agent!=*Chrome* user_agent!=*Safari* user_agent!=*Firefox*
```

The output shows only suspicious user agents.
Click **client_ip** to find the IP responsible — use this IP for <REDACTED> in following queries.

<br></br>

### Narrowing Down Suspicious IPs
---

```
sourcetype=web_traffic user_agent!=*Mozilla* user_agent!=*Chrome* user_agent!=*Safari* user_agent!=*Firefox* 
| stats count by client_ip 
| sort -count 
| head 5
```

This reveals the top malicious IP used by the Bandit Bunnies.

<br></br>

### Tracing the Attack Chain
---

Replace **<REDACTED>** with the suspicious IP.

### Reconnaissance (Footprinting)

```
sourcetype=web_traffic client_ip="<REDACTED>" AND path IN ("/.env", "/*phpinfo*", "/.git*") | table _time, path, user_agent, status
```

Confirms probing using curl/wget with 404/403/401 responses.

### Enumeration (Vulnerability Testing)

```
sourcetype=web_traffic client_ip="<REDACTED>" AND path="*..*" OR path="*redirect*"
```

Then refined:

```
sourcetype=web_traffic client_ip="<REDACTED>" AND path="*..\/..\/*" OR path="*redirect*" | stats count by path
```

Shows attempts to read sensitive files via path traversal.

### SQL Injection Attack

```
sourcetype=web_traffic client_ip="<REDACTED>" AND user_agent IN ("*sqlmap*", "*Havij*") | table _time, path, status
```

Confirms sqlmap/Havij and time-based SQLi using SLEEP(5).

### Exfiltration Attempts

```
sourcetype=web_traffic client_ip="<REDACTED>" AND path IN ("*backup.zip*", "*logs.tar.gz*") | table _time path, user_agent
```

Shows exfiltration of compressed log files.

### Ransomware Staging & RCE

```
sourcetype=web_traffic client_ip="<REDACTED>" AND path IN ("*bunnylock.bin*", "*shell.php?cmd=*") | table _time, path, user_agent, status
```

Shows remote code execution and ransomware execution:
`cmd=./bunnylock.bin`

<br></br>
---

### Correlate Outbound C2 Communication
---

Pivot to firewall logs:

```
sourcetype=firewall_logs src_ip="10.10.1.5" AND dest_ip="<REDACTED>" AND action="ALLOWED" 
| table _time, action, protocol, src_ip, dest_ip, dest_port, reason
```

Confirms outbound C2 connection.
ACTION=ALLOWED and REASON=C2_CONTACT verify malware communication.

<br></br>

---

### Volume of Data Exfiltrated
---

```
sourcetype=firewall_logs src_ip="10.10.1.5" AND dest_ip="<REDACTED>" AND action="ALLOWED" 
| stats sum(bytes_transferred) by src_ip
```

Shows large volume of data moved to the attacker’s C2 server.

<br></br>

### Final Answers
---

| Question                                                                  | Answer            |
| ------------------------------------------------------------------------- | ----------------- |
| What is the attacker IP found attacking and compromising the web server?  | **198.51.100.55** |
| Which day was the peak traffic in the logs?                               | **2025-10-12**    |
| What is the count of Havij user_agent events found in the logs?           | **993**           |
| How many path traversal attempts to access sensitive files were observed? | **658**           |
| How many bytes were transferred to the C2 server?                         | **126167**        |

<br></br>

### Conclusion
---
* Attacker identified from abnormal web_traffic logs
* Reconnaissance via curl/wget probing configuration files
* Path traversal, redirect testing, and vulnerability scanning
* SQL injection confirmed using sqlmap/Havij user agents
* Exfiltration using backup.zip / logs.tar.gz
* RCE via webshell and execution of bunnylock.bin
* Firewall logs confirm outbound C2 communication
* Significant data transfer to attacker IP

---