# Advent of Cyber 2025 - Day 22 - C2 Detection: Command & Carol

## 1. Introduction

The TBFC is very wary since the last series of attacks by the underlings of King Malhare. They are on full alert for anything happening. But they are getting restless; it is too quiet. Sir Elfo of the TBFC takes the initiative and proposes to hunt for Command and Control traffic using the meticulously collected network traffic. A majority of the TBFC elves object, we don't have the time to go through so much network traffic! Sir Elfo chuckles, don't fret, for I have a powerful tool to assist us! I present to you RITA, Real Intelligence Threat Analytics. We just need to convert our PCAP file to Zeek logs, and RITA will do the rest! Anyone can do it, just follow today's tasks.

<br></br>

### Learning Objectives
---

- Convert a PCAP to Zeek logs
- Use RITA to analyze Zeek logs
- Analyze the output of RITA

<br></br>

### Connecting to the Machine
---

Start your target machine by clicking the Start Machine button below. The machine will open in split view and need about 2 minutes to fully boot. In case you can not see it, click the Show Split View button at the top of the page

<br></br>

## 2. Detecting C2 with RITA

### The Magic of RITA
---

Real Intelligence Threat Analytics (**RITA**) is an open-source framework developed by **Active Countermeasures**. Its primary goal is to **detect Command and Control (C2) communication** by analysing network traffic.

Key capabilities of RITA include:

* C2 beacon detection
* DNS tunneling detection
* Long connection detection
* Data exfiltration detection
* Threat intelligence correlation
* Severity scoring of connections
* Identifying number of hosts communicating with an external IP
* Tracking first-seen timestamps of external hosts

RITA works by correlating multiple network attributes such as IP addresses, ports, timestamps, and connection durations to identify suspicious communication patterns.

<br></br>

### How RITA Detects C2 Activity
---

RITA performs analytics on normalized Zeek log data and evaluates several behavioural indicators, including:

* Periodic connection intervals
* Excessive DNS queries
* Long Fully Qualified Domain Names (FQDNs)
* Random or high-entropy subdomains
* Large data volumes over time
* Use of non-standard ports
* Short-lived or self-signed certificates
* Matches against known malicious IPs from threat intelligence feeds

These analytics allow RITA to identify suspicious connections even when traffic is encrypted.

<br></br>

### Zeek Logs as RITA Input
---

RITA only accepts **Zeek logs** as input.

Zeek is an open-source Network Security Monitoring (NSM) tool that:

* Observes network traffic passively
* Does not block traffic or use signatures
* Converts PCAP data into structured logs

Zeek supports:

* Transaction data (e.g., HTTP, DNS logs)
* Extracted content data (files, artifacts)

PCAP files must first be converted into Zeek logs before RITA can analyze them.

<br></br>

### Converting PCAP to Zeek Logs
---

Navigate to the home directory and list contents:

```bash
ls
```

Two relevant directories exist:

* `pcaps` – contains packet captures
* `zeek_logs` – destination for Zeek output

Convert a PCAP to Zeek logs using:

```bash
zeek readpcap pcaps/AsyncRAT.pcap zeek_logs/asyncrat
```

Zeek processes the PCAP and generates multiple structured log files.

<br></br>

### Reviewing Zeek Log Output
---

Generated Zeek logs include:

* `conn.log`
* `dns.log`
* `http.log`
* `ssl.log`
* `files.log`
* `x509.log`
* `notice.log`
* and others

These logs store enriched metadata used by RITA during analysis.

<br></br>

### Importing Zeek Logs into RITA
---

Import the Zeek logs into RITA:

```bash
rita import --logs ~/zeek_logs/asyncrat/ --database asyncrat
```

During import:

* Logs are parsed
* Threat intel feeds are updated
* Analytics modules are executed

Once complete, the dataset is ready for inspection.

<br></br>

### Viewing RITA Analysis Results
---

View analysis results with:

```bash
rita view asyncrat
```

The RITA interface displays:

* Search bar
* Results pane
* Details pane

Smaller datasets may produce fewer results and are more prone to false positives.

<br></br>

### Understanding the RITA Interface
---

#### **Search Bar**

* Activated using `/`
* Supports filters and sorting
* `?` displays available search fields

#### **Results Pane**

Displays:

* Severity score
* Source and destination
* Beacon likelihood
* Connection duration
* Subdomain usage
* Threat intel matches

#### **Details Pane**

Shows:

* Threat Modifiers
* Connection metadata

<br></br>

### Threat Modifiers Explained
---

Threat modifiers influence severity scoring and include:

* MIME type / URI mismatch
* Rare signature
* Prevalence
* First seen timestamp
* Missing host header
* Large outgoing data volume
* No direct connections

These modifiers help identify stealthy or unusual behaviour.

<br></br>

### Analyzing Suspicious Entries
---

Two notable findings:

* `sunshine-bizrate-inc-software.trycloudflare.com`
* IP `91.134.150.150`

Indicators observed:

* Long FQDN
* VirusTotal flags
* Rare TLS signatures
* Long connection duration
* Non-standard ports

These characteristics strongly suggest C2 activity.

<br></br>

### Hands-On RITA Challenge
---

Analyze `~/pcaps/rita_challenge.pcap` using:

1. Zeek PCAP conversion
2. RITA import
3. RITA view and filtering

Use search filters and threat modifiers to answer the challenge questions.

<br></br>

## **Final Answers**

| Question                                      | Answer                                                        |
| --------------------------------------------- | ------------------------------------------------------------- |
| Hosts communicating with malhare.net          | **6**                                                         |
| Threat Modifier showing host count            | **prevalence**                                                |
| Highest connections to rabbithole.malhare.net | **40**                                                        |
| RITA search filter                            | **dst:rabbithole.malhare.net beacon:>=70 sort:duration-desc** |
| Port used by host 10.0.0.13                   | **80**                                                        |

<br></br>

## **Conclusion**

* RITA identifies C2 traffic through behavioural analytics
* Zeek logs provide the structured data RITA requires
* Threat modifiers help prioritize suspicious connections
* Long connections, rare signatures, and low prevalence are strong IoCs
* RITA remains effective even when payloads are encrypted
* Proper analysis requires combining RITA results with analyst judgement

---
