# Advent of Cyber 2025 - Day 11 - XSS: Merry XSSMas

## 1. Introduction

After last year's automation and tech modernisation, Santa's workshop got a new makeover. McSkidy has a secure message portal where you can contact her directly with any questions or concerns. However, lately, the logs have been lighting up with unusual activity, ranging from odd messages to suspicious search terms. Even Santa's letters appear to be scripts or random code. Your mission, should you choose to accept it: dig through the logs, uncover the mischief, and figure out who's trying to mess with McSkidy. 
<br></br>

### Learning Objectives
---

- Understand how XSS works
- Learn to prevent XSS attacks
  <br></br>

### Connecting to the Machine
---

Start your target machine by clicking the Start Machine button below. The machine will need about 2 minutes to fully boot. Additionally, start your AttackBox by clicking the Start AttackBox button below. The AttackBox will start in split view. In case you can not see it, click the Show Split View button at the top of the page.
<br></br>

## 2. Leave the Cookies, Take the Payload

### Equipment Check

For this task, we use the provided web app:

`http://MACHINE_IP`

You will see a simple portal interface containing:

* A **search bar**
* A **message submission form**
* A **system logs section**

We use this environment to test two major types of Cross-Site Scripting:

* **Reflected XSS**
* **Stored XSS**

Before jumping into exploitation, let’s recap what XSS actually is.

<br></br>

### Understanding XSS (Cross-Site Scripting)
---

XSS allows attackers to inject **malicious JavaScript** into web pages viewed by other users.
When websites fail to validate or sanitize input, browsers may treat attacker-supplied values as executable code.

#### **Impacts:**

* Cookie/session theft
* Account impersonation
* Defacement
* Fake prompts
* Unauthorized actions (performed as the victim)

#### **Reflected XSS**

Occurs when malicious input is **immediately** reflected in the HTTP response.

Example:

```
https://site/search?term=<script>alert('XSS')</script>
```

Triggered typically via phishing or malicious links.

<br></br>

#### **Stored XSS**

Occurs when malicious input is **saved by the server**, then displayed to *everyone* visiting that page.

Works like a "set and forget" attack.
More dangerous because:

* It persists
* Every user becomes a victim
* Often executes automatically

<br></br>

### Protection Against XSS
---

Key mitigations:

* Use **textContent** instead of innerHTML
* Set cookies with **HttpOnly**, **Secure**, **SameSite**
* Sanitize & encode all user-supplied inputs
* Strip dangerous tags/scripts, event handlers, javascript: URLs
* Validate + encode outputs at every render point

<br></br>

### Exploiting Reflected XSS
---

The first exploit vector is the **Search Messages** field.

#### **Payload:**

```html
<script>alert('Reflected Meow Meow')</script>
```

#### **Steps:**

1. Enter the payload into the search bar
2. Click **Search Messages**
3. If the page pops an alert box → **Reflected XSS confirmed**

#### **Why it worked?**

* User input is echoed in the search results
* The server does not encode characters like `<` or `>`
* Browser interprets it as legitimate JavaScript

You can confirm execution by checking the **System Logs** panel — it logs your input and code execution.

<br></br>

### **Exploiting Stored XSS**
---

The second vector is the **Send Message** form.
Messages here are stored permanently on the backend for McSkidy to review later.

#### **Payload:**

```html
<script>alert('Stored Meow Meow')</script>
```

#### **Steps:**

1. Navigate to the message form
2. Submit the above payload
3. Reload the page

If the alert pops **every time**, you have confirmed **Stored XSS**.

#### **Why it worked?**

* Data is saved in the backend
* Displayed to every visitor
* Rendered unsafely → executes every time the page loads

This confirms that the website has **no input sanitization**, and malicious scripts persist across sessions.

<br></br>

## **Final Answers**

| Question                                                            | Answer                   |
| ------------------------------------------------------------------- | ------------------------ |
| Which type of XSS requires payloads to be persisted on the backend? | **stored**               |
| What’s the reflected XSS flag?                                      | **THM{Evil_Bunny}**      |
| What’s the stored XSS flag?                                         | **THM{Evil_Stored_Egg}** |

<br></br>

## **Conclusion**

* The site is vulnerable to both **Reflected** and **Stored** XSS.
* Reflected XSS executes immediately from user input.
* Stored XSS persists on the server, affecting *every* user.
* Input output sanitization, encoding, secure cookie attributes, and safe rendering practices are essential defenses.
* System logs confirm how attacker actions are interpreted by the site.
* These weaknesses explain the malicious activity previously detected in the logs.

---