# Advent of Cyber 2025 - Day 13 - YARA Rules: YARA mean one!

## 1. Introduction

When McSkidy went missing, there was chaos and uncertainty at The Best Festival Company (TBFC). However, even in her disappearance, McSkidy was trying to help the TBFC blue team. Taking a page out of the crisis communication process, McSkidy sent what looks like a bunch of images to the blue team from an anonymous location. These images looked like they were related to Easter preparations, but they contained a message sent by McSkidy. The crisis communication process outlines that a message might be sent through a folder of images containing hidden messages that can be decoded if you know the keyword. The blue team has to create a YARA rule that runs on the directory containing the images. The YARA rule must trigger on a keyword followed by a code word. After extracting all the code words in ascending order, the blue team will be able to decode the message.

<br></br>

### Learning Objectives
---

- Understand the basic concept of YARA.
- Learn when and why we need to use YARA rules.
- Explore different types of YARA rules.
- Learn how to write YARA rules.
- Practically detect malicious indicators using YARA.

<br></br>

### Connecting to the Machine
---

Start your target VM by clicking the Start Machine button. The machine will need about 2 minutes to fully boot.

In the attached VM, the folder sent by McSkidy has been downloaded to `/home/ubuntu/Downloads/easter`. We will be using this folder to run our YARA rules against.

<br></br>

## 2. Yara Rules

### YARA Overview
---

At this stage, McSkidy entrusts us with a mission involving **YARA**, a powerful pattern-matching tool used to identify and classify malware.

YARA works by scanning files, memory, or processes for **unique patterns** — digital fingerprints left behind by attackers. Instead of relying on filenames or hashes alone, YARA focuses on *behavioural and structural indicators*.

Within TBFC, YARA acts as a silent guardian, uncovering traces of malicious activity that traditional tools might miss. As SOC-mas approaches and threats rise, defenders must use YARA to restore balance across Wareville.

<br></br>

---

### Why YARA Matters and When to Use It
---

In modern cyber defence, analysts face:

* Large volumes of alerts
* Suspicious files disguised as benign
* Malware variants that evade signature-based AV

YARA enables defenders to **define what “malicious” means**, rather than relying solely on third-party detection.

#### **Common Use Cases**

* **Post-incident analysis**
  → Check whether malware found on one system exists elsewhere

* **Threat hunting**
  → Search endpoints for known malware families or behaviours

* **Intelligence-based scans**
  → Apply shared rules from trusted defenders

* **Memory analysis**
  → Scan memory dumps for injected or fileless malware

YARA transforms defenders from reactive responders into proactive hunters.

<br></br>

### YARA Values
---

YARA offers defenders critical advantages:

* **Speed** – scan thousands of files quickly
* **Flexibility** – detect text, bytes, regex, and logic
* **Control** – analysts define detection logic
* **Shareability** – rules can be reused and improved
* **Visibility** – correlate scattered clues into a clear narrative

In short, YARA allows defenders to act **before** attackers spread further.

<br></br>

### Understanding YARA Rules
---

Every YARA rule has three core components:

#### **1. Metadata**

Describes the rule:

* Author
* Purpose
* Creation date

#### **2. Strings**

The indicators YARA searches for:

* Text
* Hexadecimal bytes
* Regular expressions

#### **3. Conditions**

Logic that determines when the rule triggers.

#### **Example Rule**

```yara
rule TBFC_KingMalhare_Trace
{
    meta:
        author = "Defender of SOC-mas"
        description = "Detects traces of King Malhare’s malware"
        date = "2025-10-10"

    strings:
        $s1 = "rundll32.exe" fullword ascii
        $s2 = "msvcrt.dll" fullword wide
        $url1 = /http:\/\/.*malhare.*/ nocase

    condition:
        any of them
}
```

Metadata is optional but strongly recommended for maintainability.

<br></br>

### YARA Strings
---

#### **Text Strings**

Simple readable strings found in files or memory.

```yara
$xmas = "Christmas"
```

##### **Modifiers**

* `nocase` → ignore case
* `wide` / `ascii` → Unicode vs ASCII
* `xor` → detect XOR-encoded strings
* `base64` / `base64wide` → decode Base64 content

Example:

```yara
$hidden = "Malhare" xor
```

#### **Hexadecimal Strings**

Used for binary-level detection.

```yara
$mz = { 4D 5A }   // PE header
```

Supports wildcards (`??`) for flexibility.

#### **Regular Expression Strings**

Useful for detecting variable patterns like URLs or encoded commands.

```yara
$url = /http:\/\/.*malhare.*/ nocase
```

Regex is powerful but should be used carefully to avoid performance issues.

<br></br>

### YARA Conditions
---

Conditions decide **when** a rule triggers.

#### **Basic Conditions**

* Match one string:

```yara
condition:
    $xmas
```

* Match any string:

```yara
condition:
    any of them
```

* Match all strings:

```yara
condition:
    all of them
```

#### **Logical Operators**

```yara
condition:
    ($s1 or $s2) and not $benign
```

#### **File Property Checks**

```yara
condition:
    any of them and filesize < 700KB
```

Conditions reduce false positives and add context-aware detection.

<br></br>

### YARA Study Use Case – IcedID Detection
---

The Malhare kingdom deployed **IcedID**, a lightweight malware loader.

#### **YARA Rule**

```yara
rule TBFC_Simple_MZ_Detect
{
    meta:
        author = "TBFC SOC L2"
        description = "IcedID Rule"
        date = "2025-10-10"
        confidence = "low"

    strings:
        $mz   = { 4D 5A }
        $hex1 = { 48 8B ?? ?? 48 89 }
        $s1   = "malhare" nocase

    condition:
        all of them and filesize < 10485760
}
```

#### **Execution**

```bash
yara -r icedid_starter.yar C:\
```

#### **Result**

```
C:\Users\WarevilleElf\AppData\Roaming\TBFC_Presents\malhare_gift_loader.exe
```

#### **Useful Flags**

* `-r` → recursive scan
* `-s` → print matching strings

<br></br>

### YARA Practical Task
---

#### **Objective**

Scan `/home/ubuntu/Downloads/easter` to find strings beginning with:

```
TBFC:<alphanumeric>
```

#### **Regex Used**

```regex
/TBFC:[A-Za-z0-9]+/
```

YARA rules and regex searches were applied across images to recover McSkidy’s hidden message.

<br></br>

## **Final Answers**

| Question                                                | Answer                       |
| ------------------------------------------------------- | ---------------------------- |
| How many images contain the string TBFC?                | **5**                        |
| Regex to match TBFC followed by alphanumeric characters | **/TBFC:[A-Za-z0-9]+/**      |
| What is the message sent by McSkidy?                    | **Find me in HopSec Island** |

<br></br>

## **Conclusion**

* YARA enables pattern-based malware detection beyond hashes and filenames.
* Text, hex, and regex strings allow flexible detection strategies.
* Conditions provide logical control to reduce false positives.
* YARA excels in threat hunting, post-incident analysis, and intelligence-driven scans.
* In the practical task, regex-based detection successfully recovered McSkidy’s hidden message.

---