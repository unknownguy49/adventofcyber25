# Advent of Cyber 2025 - Day 2 - Phishing: Merry Clickmas

## 1. Introduction

In light of several recent cyber security threats against The Best Festival Company (TBFC), the local red team has scheduled several penetration tests. The red teamers proceeded to carry out a regular penetration test against their TBFC. Part of this exercise is to ensure that the employees are diligent when clicking links and that the company is well protected against the latest phishing attacks. This type of authorised phishing is a proven way to learn whether the cyber security awareness training has fruited.

In this task, you will be part of the TBFC local red team with the elves Recon McRed, Exploit McRed, and Pivot McRed. You will help them plan and execute their phishing campaign. It is time to see if more cyber security awareness training is required.
<br></br>

### Learning Objectives

---

- Understand what social engineering is
- Learn the types of phishing
- Explore how red teams create fake login pages
- Use the Social-Engineer Toolkit to send a phishing email
  <br></br>

### Connecting to the Machine

---

Start your target VM by clicking the Start Machine button below. The machine will need about 2 minutes to fully boot. Additionally, start your AttackBox by clicking the Start AttackBox button below. The AttackBox will start in split view. In case you can not see it, click the Show Split View button at the top of the page.

<br></br>

## 2. Phishing Exercise for TBFC

### Starting the Phishing Server
---

We navigate to the provided directory and launch the fake TBFC login portal:

```bash
cd ~/Rooms/AoC2025/Day02
./server.py
```

**Output:**

```
Starting server on http://0.0.0.0:8000
```

**Meaning:**
The phishing page is now hosted on **port 8000**, exposed on all interfaces (`0.0.0.0`).
The victim will see the fake login form when visiting:

* `http://CONNECTION_IP:8000`
  or
* `http://127.0.0.1:8000`

<br></br>

### Previewing the Fake Login Page
---

Open Firefox inside the AttackBox and browse to:

```text
http://CONNECTION_IP:8000
```

**Result:** You confirm that the phishing portal is functional and ready for delivery.

<br></br>

### Launching Social-Engineer Toolkit (SET)
---

Start the toolkit:

```bash
setoolkit
```

**Menu Path Chosen:**

1. **Social-Engineering Attacks**
2. **Mass Mailer Attack**
3. **E-Mail Attack Single Email Address**

This prepares the environment to send a crafted phishing email.

<br></br>

### Configuring Email Delivery
---

Fill in SET's prompts:

```text
Send email to: factory@wareville.thm
Use your own server or open relay
From address: updates@flyingdeer.thm
From name: Flying Deer
SMTP server: MACHINE_IP
SMTP port: 25
High priority: no
Attach file: n
Inline file: n
Subject: Shipping Schedule Changes
```

Now enter the email body:

```
Dear elves,
Kindly note that there have been significant changes to the shipping schedules due to increased shipping orders.
Please confirm the new schedule by visiting http://CONNECTION_IP:8000
Best regards,
Flying Deer
END
```

**SET Output:**

```
[*] SET has finished sending the emails
```

<br></br>

### Waiting for Credential Harvesting
---

Switch back to the terminal running `server.py`.

SET sends the phishing email → Target clicks → Target enters credentials → Credentials appear instantly in the server terminal.

**Result:**
A captured login set appears on the screen.

**Recovered TBFC Portal Password:**

```
unranked-wisdom-anthem
```

<br></br>

### Testing Credential Reuse on TBFC Mail Portal
---

Navigate inside AttackBox to:

```text
http://MACHINE_IP
```

Attempt login using:

* **Username:** factory
* **Password:** unranked-wisdom-anthem

**Result:** Login successful.

Inside the inbox, a relevant shipping report mail reveals:

**Total number of toys expected for delivery:**

```
1984000
```

<br></br>

## Final Answers

| Question                            | Answer                     |
| ----------------------------------- | -------------------------- |
| Password used to access TBFC portal | **unranked-wisdom-anthem** |
| Total toys expected for delivery    | **1984000**                |

<br></br>

## Conclusion

This exercise demonstrated:

- Hosting a phishing login clone
- Sending realistic phishing emails via SET
- Harvesting credentials from victims in real time
- Testing credential reuse across internal systems

The red team successfully proved that TBFC staff remain vulnerable to phishing attacks, highlighting the importance of continuous awareness training.

---