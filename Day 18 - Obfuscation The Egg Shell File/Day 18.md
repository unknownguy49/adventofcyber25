# Advent of Cyber 2025 - Day 18 - Obfuscation: The Egg Shell File

## 1. Introduction

WareVille has not felt right since the wormhole appeared. Everyone in TBFC is on high alert: Systems are going haywire, dashboards are spiking, and SOC alerts have been firing nonstop. Amid the chaos, McSkidy keeps her focus on a particular alert that caught her interest: an email posing as northpole-hr. It’s littered with carrot emojis, but the weird thing is this: there is no North Pole human resources department. TBFC’s HR is at theSouth Pole.

Digging further she found a tiny PowerShell file from that email was downloaded. Among the code are random strings of characters, all gibberish, nothing readable at a glance.

McSkidy knows malicious actors often hide code and data using a technique called obfuscation. But what is it, really? And how can we decipher it?

<br></br>

### Learning Objectives
---

- Learn about obfuscation, why and where it is used.
- Learn the difference between encoding, encryption, and obfuscation.
- Learn about obfuscation and the common techniques.
- Use CyberChef to recover plaintext safely.

<br></br>

### Connecting to the Machine
---

Start your VM by clicking the Start Machine button below. The machine will start in split view and will take about two minutes to load. In case the machine is not visible, you can click the Show Split View button at the top of the task.

<br></br>

## 2. Obfuscation & Deobfuscation

### Understanding the Gibberish – Obfuscation & Deobfuscation
---

**Obfuscation** is the practice of deliberately making data hard to read or analyse. Attackers use it to:

* Evade signature-based detection
* Delay investigations
* Hide intent from analysts and automated tools

Unlike encryption, obfuscation is not meant to be secure — just *annoying enough*.

#### **ROT Ciphers**

A simple example is **ROT (Rotate) ciphers**.

* **ROT1**: Shifts each letter forward by 1

  * `a → b`, `b → c`
  * Example:
    `carrot coins go brr` → `dbsspu dpjot hp css`

* **ROT13**: Shifts letters by 13 positions

  * Common in CTFs and malware droppers
  * Easy to reverse once identified

ROT ciphers preserve:

* String length
* Spaces
* Character structure

This makes them weak, but fast and effective against naïve keyword detection.

<br></br>

### Obfuscation in the Real World
---

Real malware rarely stops at ROT ciphers. One commonly used technique is **XOR obfuscation**.

#### **XOR (Exclusive OR)**

* Operates at the byte level
* Combines each byte with a key
* Output often contains:

  * Non-printable characters
  * Symbols
  * Gibberish text

Key property:

> XOR is **reversible** — applying the same key again restores the original data.

#### **XOR with CyberChef**

Steps used:

1. Open CyberChef
2. Input: `carrot supremacy`
3. Add **XOR** operation
4. Key: `a`
5. Key format: **HEX**
6. Bake

**Result:**

```
ikxxe~*yzxogkis!
```

This output:

* Same length as input
* Looks random
* Not human-readable

Exactly what attackers want.

<br></br>

### Detecting Obfuscation Patterns
---

Even obfuscated data leaks clues. Common visual indicators:

| Technique | Visual Clues                                                          |
| --------- | --------------------------------------------------------------------- |
| ROT1      | Text looks “one letter off”, spaces intact                            |
| ROT13     | Short words mutate (`the → gur`, `and → naq`)                         |
| Base64    | Long strings, A–Z a–z 0–9 `+ /`, often ending with `=`                |
| XOR       | Random symbols, same length as original, sometimes repeating patterns |

Once you suspect the method, reverse it using the **opposite CyberChef operation**:

* `To Base64` → `From Base64`
* `XOR` → `XOR` with same key
* `ROT13` → `ROT13` again

<br></br>

### Unfamiliar Patterns & CyberChef Magic
---

Not sure what you’re dealing with? CyberChef provides **Magic**.

#### **Magic Operation**

* Automatically tries common decoders
* Produces multiple possible outputs
* Analyst decides which result makes sense
* Optional **Intensive Mode** increases coverage

⚠️ Magic is a **hint engine**, not a silver bullet:

* Won’t reliably solve custom XOR keys
* Won’t fully reverse layered obfuscation alone

Use it to narrow possibilities, not replace reasoning.

<br></br>

#### Layered Obfuscation
---

Attackers often **stack multiple techniques** to increase complexity.

#### **Example Layered Payload**

```
gzip → XOR(key) → Base64
```

Produces a long encoded blob that looks opaque at first glance.

#### **Deobfuscation Rule**

> Always reverse in the **opposite order**

So the correct recovery chain is:

1. **From Base64**
2. **XOR** with key
3. **Gunzip**

CyberChef makes this trivial by chaining operations in sequence.

<br></br>

### Unwrapping the Easter Egg – Practical Analysis
---

McSkidy extracted an obfuscated PowerShell script from a suspicious PDF:

* File: `SantaStealer.ps1`
* Location: **Desktop**
* Environment: Isolated VM

#### **Part 1 – Deobfuscation**

1. Open the script in Visual Studio
2. Navigate to **“Start here”**
3. Follow the comments
4. Save the file
5. Run from PowerShell:

```powershell
cd .\Desktop\
.\SantaStealer.ps1
```

This decodes the obfuscated **C2 URL** and reveals the first flag.

#### **Part 2 – Obfuscation**

1. Follow **Part 2** instructions in comments
2. Obfuscate the attacker’s API key using **XOR**
3. Run the script again
4. Retrieve the second flag

This reinforces:

* XOR reversibility
* Obfuscation logic used by real malware
* How attackers hide configuration data

<br></br>

## **Final Answers**

| Question    | Answer                          |
| ----------- | ------------------------------- |
| First flag  | `THM{C2_De0bfuscation_29838}`   |
| Second flag | `THM{API_Obfusc4tion_ftw_0283}` |

<br></br>

## **Conclusion**

* Obfuscation delays detection, it does not provide security
* Visual patterns often reveal the technique
* XOR is reversible and widely abused
* Layered obfuscation is common in real malware
* CyberChef is indispensable for rapid analysis
* Understanding *why* something works matters more than clicking Magic

---