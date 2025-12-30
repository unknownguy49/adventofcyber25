# Advent of Cyber 2025 - Day 12 - Phishing: Phishmas Greetings

## 1. Introduction

Since McSkidy’s disappearance, TBFC’s defences have weakened, and now the Email Protection Platform is down.

With filters offline, the staff must triage every suspicious message manually.
The SOC Team suspects Malhare’s Eggsploit Bunnies have sent phishing messages to TBFC’s users to steal credentials and disrupt SOC-mas.

You’ve joined the Incident Response Task Force to help identify which emails are legit or phishing attempts.

But beware, some phishing attempts are clever, disguised as routine TBFC operations, volunteer sign-ups, or SOC-mas event logistics. Every wrong call could bring Wareville one step closer to EAST-mas becoming a reality.

<br></br>

### Learning Objectives
---

- Spotting phishing emails
- Learn trending phishing techniques
- Understand the differences between spam and phishing

<br></br>

### Connecting to the Machine
---

Start your target machine by clicking the Start Machine button below. The machine will need about 2 - 3 minutes to fully boot. You can then access the Wareville's Email Threat Inspector at https://10-48-182-41.reverse-proxy.cell-prod-ap-south-1a.vm.tryhackme.com.

Note: If you get a 502 error when accessing the link, please give the machine more time to fully boot up.

<br></br>

## 2. Spotting Phishing Emails

### Phishing Overview
---

Phishing is one of the oldest cyber attack techniques, yet it remains one of the most effective. As organisations like TBFC deploy advanced email filtering and security tools, attackers adapt by crafting more **targeted, convincing, and context-aware** phishing messages.

Modern phishing relies heavily on **precision and persuasion**, often impersonating real employees, departments, or trusted services to deceive even cautious users.

#### **Common Goals of Phishing**

* **Credential theft** – stealing usernames, passwords, VPN access
* **Malware delivery** – via malicious links or attachments
* **Data exfiltration** – harvesting sensitive internal information
* **Financial fraud** – payroll changes, fake invoices, payment approvals

With hardened corporate perimeters, phishing remains the **easiest initial access vector**, targeting the weakest link: people.

<br></br>

### Why Spam Is Not Phishing
---

Spam and phishing are often confused, but they serve different purposes.

#### **Spam**

* High volume, low precision
* Usually harmless marketing or clickbait
* Sent in bulk
* Goals:

  * Promotion
  * Traffic generation
  * Data harvesting (email lists)
  * Low-effort scams

#### **Phishing**

* Targeted and intentional
* Designed to deceive specific users
* Aims to compromise accounts or systems

Not every unwanted email is dangerous — understanding **intent** is key when triaging messages.

<br></br>

### The Phishmas Takeover – Common Phishing Techniques
---

Most phishing emails contain at least one of the following techniques.

---

#### **Impersonation**

Attackers pretend to be a trusted person, department, or service.

Example:

* Email subject: **“URGENT: McSkidy VPN access for incident response”**
* Sender uses a **free email domain (gmail.com)** instead of TBFC’s official domain

Checking the sender’s domain against known internal patterns quickly exposes impersonation attempts.

---

#### **Social Engineering**

Social engineering manipulates **human emotions** instead of exploiting software.

Observed techniques:

* **Impersonation** – posing as McSkidy
* **Urgency** – words like *urgent*, *immediately*
* **Side-channel discouragement** – telling users not to verify via normal channels
* **Malicious intent** – requesting VPN credentials, approvals, or sensitive data

---

#### **Typosquatting & Punycode**

Attackers register domains that look visually similar to legitimate ones.

* **Typosquatting:**

  * Example: `glthub.com` instead of `github.com`
* **Punycode:**

  * Uses Unicode characters resembling Latin letters
  * Example: replacing `f` with `ƒ` or Latin letters with Cyrillic equivalents

Punycode can be detected by checking **email headers**, especially the **Return-Path**, where encoded domains appear.

---

#### **Spoofing**

Email spoofing forges the sender field to appear legitimate.

Indicators:

* SPF, DKIM, DMARC failures in **Authentication-Results**
* Mismatched **Return-Path**

If SPF and DMARC fail together, the email is almost certainly spoofed.

---

#### **Malicious Attachments**

Classic phishing technique using attachments disguised as legitimate files.

Observed case:

* Attachment claimed to be a voice message
* File type: **.html**

HTML/HTA files are dangerous because they execute scripts **outside browser sandboxing**, granting full system access.

<br></br>

### Trending Phishing Techniques
---

As email security improves, attackers shift tactics:

* Avoid direct malware delivery
* Lure users to **leave the secure corporate environment**
* Abuse **legitimate platforms** (OneDrive, Google Docs, Dropbox)

#### **Legitimate Tool Abuse**

* Shared documents with enticing proposals (salary raises, upgrades)
* Fake domains with punycode
* Redirection to credential-harvesting pages

#### **Fake Login Pages**

* Mimic Microsoft 365, Google, or email portals
* Hosted on look-alike domains
* Primary goal: **credential theft**

---

#### **Side-Channel Communication**

Attackers move conversations off email to:

* WhatsApp
* SMS
* Telegram
* Phone calls
* Shared platforms

This bypasses corporate monitoring and security controls.

<br></br>

## **SOC-mas Mission**

Objective:

* Triage incoming emails
* Separate **spam** from **phishing**
* Identify **three clear phishing indicators** per malicious email

Indicators include:

* Impersonation
* Spoofing
* Social engineering
* Malicious attachments
* Typosquatting / punycode
* Side-channel abuse

Correct classification of each phishing email yields a flag.

<br></br>

## **Final Answers**

| Email                  | Flag                                         |
| ---------------------- | -------------------------------------------- |
| Classify the 1st email | **THM{yougotnumber1-keep-it-going}**         |
| Classify the 2nd email | **THM{nmumber2-was-not-tha-thard!}**         |
| Classify the 3rd email | **THM{Impersonation-is-areal-thing-keepIt}** |
| Classify the 4th email | **THM{Get-back-SOC-mas!!}**                  |
| Classify the 5th email | **THM{It-was-just-a-sp4m!!}**                |
| Classify the 6th email | **THM{number6-is-the-last-one!-DX!}**        |

<br></br>

## **Conclusion**

* Phishing remains the most effective initial access vector
* Precision, impersonation, and social engineering are the core tactics
* Spam ≠ phishing — intent determines risk
* Domain analysis, header inspection, and behavioural cues are critical
* Legitimate platforms are increasingly abused to bypass filters
* User awareness and verification processes are essential to defending SOC-mas

---