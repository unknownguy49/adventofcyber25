# Advent of Cyber 2025 - Day 5 - IDOR: Santa's Little IDOR

## 1. Introduction

The elves of Wareville are on high alert since McSkidy went missing. Recently, the support team has been receiving many calls from parents who can't activate vouchers on the TryPresentMe website. They also mentioned they are receiving many targeted phishing emails containing information that is not public. The support team is wary and has enlisted the help of the TBFC staff. When looking into this peculiar case, they discovered a suspiciously named account named Sir Carrotbane, which has many vouchers assigned to it. For now, they have deleted the account and retrieved the vouchers. But something is going on. Can you help the TBFC staff investigate the TryPresentMe website and fix the vulnerabilities?
<br></br>

### Learning Objectives

---

- Understand the concept of authentication and authorization
- Learn how to spot potential opportunities for Insecure Direct Object References (IDORs)
- Exploit IDOR to perform horizontal privilege escalation
- Learn how to turn IDOR into SDOR (Secure Direct Object Reference)
  <br></br>

### Connecting to the Machine

---

Start your target VM by clicking the Start Machine button below. The machine will need about 2 minutes to fully boot. Additionally, start your AttackBox by clicking the Start AttackBox button below. The AttackBox will start in split view. In case you can not see it, click the Show Split view button at the top of the page. Inside your AttackBox, open a web browser and navigate to the TryPresentMe application at `http://10.49.175.21`.

<br></br>

## 2. IDOR on the Shelf

### It’s Dangerously Obvious, Really
---

Have you ever seen a link that looks like this: `https://awesome.website.thm/TrackPackage?packageID=1001?`

When you saw a link like this, have you ever wondered what would happen if you simply changed the `packageID` to `11` or `12`? In its simplest form, this can be a potential case for **IDOR**.

**IDOR** stands for **Insecure Direct Object Reference** and is a type of access control vulnerability. Web applications often use references to determine what data to return when you make a request. However, if the web server doesn't perform checks to ensure you are allowed to view that data before sending it, it can lead to serious sensitive information disclosure.

Why does this happen so often? We need to understand references and web development a bit more to answer this.

Consider a table storing package numbers:

| packageID | person       | address                      | status           |
| --------- | ------------ | ---------------------------- | ---------------- |
| 1001      | Alice Smith  | 123 Main St, Springfield     | Delivered        |
| 1002      | Bob Johnson  | 42 Elm Ave, Shelbyville      | In Transit       |
| 1003      | Carol White  | 9 Oak Rd, Capital City       | Out for Delivery |
| 1004      | Daniel Brown | 77 Pine St, Ogdenville       | Pending          |
| 1005      | Eve Martinez | 5 Maple Ln, North Haverbrook | Returned         |

If the web request lets a user supply `packageID` and the backend runs:

```
SELECT person, address, status FROM Packages WHERE packageID = value;
```

then sequential IDs make guessing trivial. If the application doesn't verify ownership, an attacker can retrieve other users’ package details. If no authentication is required at all, there is no way to link requests to a user.

<br></br>

### Identity Defines Our Reach
---
**Authentication** vs **Authorization**

* **Authentication:** verifying who you are (e.g., username & password).
* **Authorization:** verifying what you are allowed to do (e.g., view admin pages).

Authentication produces session information (cookie/token) sent with every request; the server authenticates each request using that session. If session info expires, sites redirect you to login again.

**Important:** Authorization cannot happen before authentication. If IDOR doesn’t require authentication (like the package example), fix authentication first, then authorization.

**Privilege escalation types:**

* **Vertical privilege escalation:** gain access to higher-privilege features (e.g., admin-only actions).
* **Horizontal privilege escalation:** access data belonging to other users while using allowed features.

  * **IDOR is usually horizontal privilege escalation.** You may be allowed to use the tracking feature but should only see packages you own.

<br></br>

### Iterate Digits, Observe Responses (Practical Exploitation)
---

Authenticate using the provided credentials:

* **Username:** `niels`
* **Password:** `TryHackMe#2025`
* **IP / Website:** `http://10.49.175.21`

Once logged in, use browser Developer Tools → **Network** tab. Refresh the dashboard and inspect requests. Look at the `view_accountinfo` request — the request shows `user_id` = `10` in the request (or storage) and the response confirms it is your user.

Now experiment:

1. In Developer Tools → **Storage** → **Local Storage**, edit the `auth_user` data entry value: change `user_id` from `10` to `11` and press Enter.
2. Refresh the page — you now appear as a different user.

This is a direct IDOR: changing `user_id` in client-side storage changes which account is shown. To continue with challenges, revert `user_id` to `10` (or logout and login again).

<br></br>

### In Disguise: Obvious References (Encoded IDs)
---
Not all references are plain integers. Some are encoded.

Example: clicking an eye icon for the first child triggers a request with a parameter like `Mg==`.

* `Mg==` is the Base64 encoding of `2`.
* You can perform IDOR by Base64-encoding the ID you want and using it in the request.

<br></br>

### In Digests, Objects Remain (Hashed/Obfuscated IDs)
---

IDs may look like hashes (MD5/SHA1). When you click the edit icon next to a child, the endpoint uses a hash-like value. If you can determine what was hashed, you can reproduce the hash and perform IDOR by generating the same value.

Use hash identifier tools to identify the hashing algorithm and replicate the function to produce valid references.

<br></br>

### It’s Deterministic, Obviously Reproducible (UUIDs & Predictability)
---

Some values look random but are deterministic. Example: vouchers that appear as UUIDs. Use a UUID decoder to examine them; you may find **UUID version 1** was used.

UUID v1 includes a timestamp component — if you know the generation time window (e.g., vouchers created between 20:00–21:00), you can generate candidate UUIDs for that period (3600 seconds → 3600 UUIDs) and brute force valid vouchers.

<br></br>

### Improve Design, Obliterate Risk (Mitigations)
---

To fix IDOR:

* **Server-side authorization checks:** Always verify “Does this user own or have permission to view this item?” on every request.
* **Don’t rely on obfuscation:** Base64 or hashed IDs are not security. They can be decoded or reproduced.
* **Use random/hard-to-guess IDs** for public links (but remember: random IDs alone are not sufficient).
* **Test for access control:** Actively try to access another user’s data and ensure it is blocked.
* **Log & monitor failed access attempts** as early indicators of exploitation attempts.

<br></br>

### **Bonus Task 1 – Finding the Child ID (Born on 2019-04-17)**
---

1. Open the web application and authenticate normally.

2. Open Developer Tools (F12) → Network tab.

3. Navigate to the "Children" section in the web app.
   - Trigger any request related to viewing a child profile.

4. Look for the request:
   /child/view/<encoded_or_hashed_value>

5. Identify whether the value is:
   - Base64 (looks like Mg==)
   - MD5 or similar (32-char hex)

6. To test Base64 version:
   - Copy the Base64 value (e.g., “Mg==”).
   - Open DevTools → Console.
   - Run atob("Mg==") to decode it → get numeric ID.
   - Manually change the Base64 portion in the request to iterate IDs:
       atob("MQ==") → 1
       atob("Mg==") → 2
       atob("Mw==") → 3
   - Repeat until the response shows a child with DOB = 2019-04-17.

7. To test MD5 version:
   - Check the request payload/params for the hash field.
   - Use DevTools → Console:
       For i from 1 to 200:
         md5(i.toString())  // If md5 lib is available on page
   - Replace the hash value in the request manually and replay it.
     (Right-click request → “Copy as Fetch” → paste into Console → edit → run.)

8. Observe the JSON response in the "Preview" or "Response" tab.
   - Stop once you find the child object where:
       "date_of_birth": "2019-04-17"

9. Note down the corresponding id_number shown in the response.
<br></br>

### **Bonus Task 2 – Finding the Valid Voucher (UUID v1 Between 20:00–24:00 on 2025-11-20)**
---

1. Open the web application and authenticate.

2. Open Developer Tools → Network tab.

3. Navigate to the Vouchers page to observe existing voucher request formats.
   - Identify the endpoint:
       /parents/vouchers/claim/<voucher_code>

4. Inspect any successful voucher request to understand:
   - Structure of the voucher (UUID v1)
   - Format of the API payload or URL parameter.

5. Understanding UUID v1:
   - It is timestamp-based.
   - Insider info: the voucher was generated between 20:00–24:00 UTC on 2025-11-20.

6. Open DevTools → Console and generate candidate UUID v1 values:
   - Load a UUID library if available on the page (sometimes apps include one):
       uuidv1 = window.uuidv1
   - If not available, paste a standard JS UUID v1 generator script into Console.

7. Generate UUIDs for every minute from 20:00 → 23:59 UTC:
   Example (pseudo-JS):
       for (let h = 20; h < 24; h++) {
         for (let m = 0; m < 60; m++) {
            let ts = new Date("2025-11-20T" + h.toString().padStart(2,'0') + ":" + m.toString().padStart(2,'0') + ":00Z");
            let voucher = uuidv1({ msecs: ts.getTime() });
            console.log(voucher);
         }
       }

8. For each generated UUID:
   - Right-click “voucher claim” request in Network → “Copy as Fetch”
   - Paste into Console, replace the voucher code with the generated one, run the request.

9. Check the Response tab:
   - You’re looking for a JSON success message (e.g., reward unlocked).

10. Once the correct UUID is found, record it as the valid voucher code.

<br></br>

## Final Answers

| Question                                                                                        | Answer                           |
| ----------------------------------------------------------------------------------------------- | -------------------------------- |
| What does IDOR stand for?                                                                       | Insecure Direct Object Reference |
| What type of privilege escalation are most IDOR cases?                                          | Horizontal                       |
| Exploiting the IDOR in `view_accounts`, what is the user_id of the parent that has 10 children? | 15                               |

<br></br>

## Conclusion

* IDOR arises when object references are accepted without server-side authorization checks.
* Authentication (session handling) is required before authorization can be enforced.
* IDOR typically results in horizontal privilege escalation (accessing other users’ data).
* References can be plain integers, Base64, hashes, or deterministic UUIDs — any of which can be abused if authorization is missing.
* Fixes: enforce server-side permission checks, avoid relying on obfuscation, use unpredictable IDs where appropriate, and monitor access patterns.

---