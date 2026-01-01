# Advent of Cyber 2025 - Day 20 - Race Conditions: Toy to The World

## 1. Introduction

The Best Festival Company (TBFC) has launched its limited-edition SleighToy, with only ten pieces available at midnight. Within seconds, thousands rushed to buy one, but something strange happened. More than ten lucky customers received confirmation emails stating that their orders were successful. Confusion spread fast. How could everyone have bought the "last" toy? McSkidy was called in to investigate.  

She quickly noticed that multiple buyers purchased at the exact moment, slipping through the system’s timing flaw. Sir Carrotbane’s mischievous Bandit Bunnies had found a way to exploit this chaos, flooding the checkout with rapid clicks. By morning, TBFC faced angry shoppers, missing stock, and a mystery that revealed just how dangerous a few milliseconds could be during the holiday rush.

<br></br>

### Learning Objectives
---

- Understand what race conditions are and how they can affect web applications.
- Learn how to identify and exploit race conditions in web requests.
- How concurrent requests can manipulate stock or transaction values.
- Explore simple mitigation techniques to prevent race condition vulnerabilities.

<br></br>

### Connecting to the Machine
---

Start your target VM by clicking the Start Machine button below. The machine will need about 2 minutes to fully boot. Additionally, start your AttackBox by clicking the Start AttackBox button below. The AttackBox will start in split view. In case you can not see it, click the Show Split View button at the top of the page. Once the machine is up and running, you can access the application by visiting http://MACHINE_IP in the browser of your AttackBox.

<br></br>

## 2. Race Condition

A **race condition** occurs when two or more actions execute simultaneously, and the final outcome depends on the order in which they complete.
In web applications, this often appears when multiple requests interact with the same shared resource (such as inventory, balances, or records) **without proper synchronization**.

This can lead to:

* Duplicate transactions
* Oversold or negative inventory
* Unauthorized state changes

<br></br>

### Types of Race Conditions
---

Race condition attacks generally fall into three categories:

#### **1. Time-of-Check to Time-of-Use (TOCTOU)**

The system checks a condition first and uses it later, but the data changes in between.

**Example:**
Two users purchase the “last item” at the same time because the stock check happens before the stock update.

#### **2. Shared Resource Race**

Multiple requests attempt to modify the same resource simultaneously.

**Example:**
Two updates to inventory occur at the same time, and the last write overwrites the first.

#### **3. Atomicity Violation**

An operation that should execute as a single unit is split into multiple steps, allowing interference.

**Example:**
Payment is recorded, but order confirmation fails because another request interrupts the process.

<br></br>

### Configuring the Environment
---

#### **Step 1: Configure Firefox Proxy**

1. Open Firefox on the AttackBox
2. Click the **FoxyProxy** icon
3. Select the **Burp** profile to route traffic through Burp Suite

#### **Step 2: Launch Burp Suite**

1. Click the **Burp Suite** icon on the Desktop
2. Choose **Temporary project in memory**
3. Click **Next** → **Start Burp**

#### **Step 3: Disable Intercept**

* Navigate to **Proxy → Intercept**
* Ensure the button shows **Intercept off**

This allows requests to pass through without being blocked.

<br></br>

### Making a Legitimate Request
---

1. Open the web app at:

```
http://MACHINE_IP
```

2. Log in using:

```
Username: attacker
Password: attacker@123
```

3. On the dashboard, observe:

* **SleighToy Limited Edition**
* Stock available: **10 units**

4. Click:

* **Add to Cart**
* **Checkout**
* **Confirm & Pay**

A success message confirms a legitimate purchase.

<br></br>

### Capturing the Checkout Request
---

1. Open **Burp Suite**
2. Navigate to **Proxy → HTTP history**
3. Locate the **POST** request to:

```
/process_checkout
```

4. Right-click the request → **Send to Repeater**

This copies the exact request (headers, cookies, body) into Repeater.

<br></br>

### Preparing the Race Condition Attack
---

#### **Step 1: Create a Tab Group**

1. Go to **Repeater**
2. Right-click the request tab
3. Select **Add tab to group**
4. Click **Create tab group**
5. Name it: `cart`

#### **Step 2: Duplicate Requests**

1. Right-click the request tab
2. Select **Duplicate tab**
3. Enter **15** copies

You now have multiple identical checkout requests.

#### **Step 3: Send Requests in Parallel**

1. In Repeater toolbar, click **Send ▼**
2. Select:

```
Send group in parallel (last-byte sync)
```

This ensures all requests hit the server **at the same time**, maximizing overlap.

#### **Step 4: Launch the Attack**

* Click **Send group (parallel)**

All checkout requests are processed simultaneously.

<br></br>

### Observing the Impact
---

1. Return to the web application
2. Observe:

   * Multiple confirmed orders
   * Inventory reduced incorrectly
   * Stock count becomes **negative**

This confirms the race condition vulnerability.

<br></br>

### Mitigation
---

The vulnerability exists because stock deduction and order creation are not synchronized.

#### **Recommended Mitigations**

* Use **atomic database transactions**
* Perform **final stock validation before commit**
* Implement **idempotency keys** for checkout requests
* Apply **rate limiting and concurrency controls**

These prevent duplicate processing and overselling.

<br></br>

## **Final Answers**

| Question                                            | Answer                        |
| --------------------------------------------------- | ----------------------------- |
| Flag when SleighToy stock becomes negative          | **THM{WINNER_OF_R@CE007}**    |
| Flag when Bunny Plush (Blue) stock becomes negative | **THM{WINNER_OF_Bunny_R@ce}** |

<br></br>

## **Conclusion**

* Race conditions occur due to improper synchronization of shared resources.
* Parallel request execution can bypass logical checks.
* Burp Repeater’s **Send group in parallel** is ideal for exploiting timing issues.
* Inventory systems must enforce atomicity and concurrency controls.
* Even legitimate functionality can be abused without proper safeguards.

---