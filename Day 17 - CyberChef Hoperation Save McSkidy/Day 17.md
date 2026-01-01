# Advent of Cyber 2025 - Day 17 - CyberChef: Hoperation Save McSkidy

## 1. Introduction

McSkidy is imprisoned in King Malhare's Quantum Warren. Sir BreachBlocker III was put in charge of securing the fortress and implemented several access controls to prevent any escape. His defenses are worthy of his name.

However, McSkidy managed to send vital clues to his team using harmless bunny pictures. One message revealed that five locks needed to be disabled to secure an escape route. The locks can be broken by examining their logic and leveraging the system's built-in chat for the guards. They can be eluded in revealing vital details or even passwords. However, you will need to speak their language.

<br></br>

### Learning Objectives
---

- Introduction to encoding/decoding
- Learn how to use CyberChef
- Identify useful information in web applications through HTTP headers

<br></br>

### Connecting to the Machine
---

Start your target machine by clicking the Start Machine button below. The machine will need about 2 minutes to fully boot. Additionally, start your AttackBox by clicking the Start AttackBox button below. The AttackBox will start in split view. In case you can not see it, click the Show Split View button at the top of the page.

<br></br>

## 2. Encoding, Decoding & CyberChef – Fortress Lock Breakout

### Encoding and Decoding
---

**Encoding** is the process of transforming data to ensure compatibility and usability across systems. It is **not designed for security**.

| Encoding               | Encryption        |
| ---------------------- | ----------------- |
| Purpose: Compatibility | Purpose: Security |
| Standardised           | Algorithm + Key   |
| No confidentiality     | Confidential      |
| Fast                   | Slower            |
| Example: Base64        | Example: TLS      |

**Decoding** simply reverses encoding to restore the original readable data.

Understanding this difference is critical, as attackers often rely on encoding (not encryption) to hide data in plain sight.

<br></br>

### CyberChef Overview
---

CyberChef is often called the **Cyber Swiss Army Knife**. It allows analysts to build **recipes** by chaining operations together.

#### **Key Areas**

* **Operations** – Available transformations (Base64, XOR, ROT, etc.)
* **Recipe** – The ordered chain of operations
* **Input** – Data you want to process
* **Output** – Result of the recipe

#### **Simple Example**

1. Add **To Base64**
2. Input: `IamRoot`
3. Add **From Base64**
4. Observe the original input restored

Operations can be enabled or disabled individually, allowing rapid experimentation.

<br></br>


### Inspecting Web Pages
---

Beyond rendered content, browsers expose valuable information through **Developer Tools**.

#### **Access Developer Tools**

* **Chrome**: More tools → Developer tools
* **Firefox**: ☰ → More tools → Web Developer Tools
* **Edge**: Settings → More tools → Developer tools
* **Safari**: Develop → Show Web Inspector

Useful tabs:

* **Elements** – Page structure
* **Network** – Headers, responses
* **Debugger** – Client-side login logic

These tools are essential for understanding how each lock validates credentials.

<br></br>

### Task 3 – First Lock: Outer Gate
---

#### **Approach**

1. Identify the **guard name**
2. Encode the guard name in **Base64** → username
3. Extract the **magic question** from headers
4. Encode the magic question in **Base64**
5. Send it via chat
6. Decode the guard’s response
7. Login logic shows **Base64 encoding**
8. Use encoded username + plaintext password

#### **Result**

The decoded password reveals the first lock’s secret.

#### **Answer**

* **First Lock Password:** `Iamsofluffy`

<br></br>

### Task 4 – Second Lock: Outer Wall
---

#### **Approach**

1. Identify and Base64-encode guard name
2. Extract and encode the magic question
3. Guard returns encoded password
4. Login logic shows **double Base64 encoding**
5. Decode the password **twice**
6. Login using stored username

Chaining operations becomes essential from this stage onward.

#### **Answer**

* **Second Lock Password:** `Itoldyoutochangeit!`

<br></br>

### Task 5 – Third Lock: Guard House
---

This level introduces **XOR encoding** combined with Base64.

#### **Key Information**

* XOR key: `cyberchef`
* No magic question
* Short polite messages speed up guard replies

#### **Login Logic**

1. Password XOR’d with key
2. Result encoded in Base64

#### **Reversal Logic**

1. From Base64
2. XOR with the same key

> XOR is reversible: applying the same key twice restores the original data.

#### **Answer**

* **Third Lock Password:** `BugsBunny`

<br></br>

### Task 6 – Fourth Lock: Inner Castle
---

This lock introduces **hashing**.

#### **Observations**

* Guard response decodes to a **hash**
* Login logic uses **MD5**

#### **Key Insight**

MD5 is one-way, but common hashes can be cracked using precomputed databases.

#### **Steps**

1. Copy the MD5 hash
2. Use an online hash lookup (e.g., CrackStation)
3. Retrieve plaintext password
4. Login normally

#### **Answer**

* **Fourth Lock Password:** `passw0rd1`

<br></br>

### Task 7 – Fifth Lock: Prison Tower
---

The final lock dynamically changes logic using a **Recipe ID**.

#### **Recipe ID Cheat Sheet**

| Recipe ID | Reverse Logic                    |
| --------- | -------------------------------- |
| 1         | From Base64 → Reverse → ROT13    |
| 2         | From Base64 → From Hex → Reverse |
| 3         | ROT13 → From Base64 → XOR(key)   |
| 4         | ROT13 → From Base64 → ROT47      |

#### **Steps**

1. Extract encoded guard name
2. Read Recipe ID from headers
3. Match the correct decoding chain
4. Build recipe in CyberChef
5. Decode final password
6. Unlock the last gate

#### **Answers**

* **Fifth Lock Password:** `51rBr34chBl0ck3r`
* **Flag:** `THM{M3D13V4L_D3C0D3R_4D3P7}`

<br></br>

### **Epilogue**
---

McSkidy escapes the fortress as its defences crumble.
Another disaster narrowly avoided in Wareville — but the Eggsploit Bunnies are never far behind.

<br></br>

## **Conclusion**

* Encoding is about compatibility, not security
* CyberChef enables rapid chaining of decoding logic
* Developer Tools expose hidden logic and headers
* XOR, Base64, ROT, and hashes are commonly combined
* Attackers rely on obscurity; defenders rely on understanding
* Correctly matching encoding logic is key to reversing protections

---