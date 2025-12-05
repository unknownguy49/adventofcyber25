# Advent of Cyber 2025 - Day 4 - AI in Security: old sAInt nick

## 1. Introduction

The lights glimmer and servers hum blissfully at The Best Festival Company (TBFC), melting the snow surrounding the data centre. TBFC has continued its pursuit of AI excellence. After the past two years, they realise that Van Chatty, their in-house chatbot, wasn’t quite meeting their standards. 

Unfortunately for the elves at TBFC, they are also not immune to performance metrics. The elves aim to find ways of increasing their velocity; something to manage the tedious, distracting tasks, which allows the elves to do the real magic. 

TBFC, adventurous as ever, is trialling their brand new cyber security AI assistant, Van SolveIT, which is capable of helping the elves with all their defensive, offensive, and software needs. They decide to put this flashy technology to use as Christmas approaches, to identify, confirm, and resolve any potential vulnerabilities, before any nay-sayers can.
<br></br>

### Learning Objectives

---

- How AI can be used as an assistant in cyber security for a variety of roles, domains and tasks
- Using an AI assistant to solve various tasks within cyber security
- Some of the considerations, particularly in cyber security, surrounding the use of AI
  <br></br>

### Connecting to the Machine

---

Start your target VM by clicking the Start Machine button below. The machine will need about 2 minutes to fully boot. Additionally, start your AttackBox by clicking the Start AttackBox button below. The AttackBox will start in split view. In case you can not see it, click the Show Split View button at the top of the page.

<br></br>

## 2. AI for Cyber Security Showcase

### The Boom of AI Assistants
---

Artificial intelligence has become a persistent part of modern technology — especially within cyber security. As organisations increasingly adopt AI to speed up repetitive, time-consuming tasks, its role continues to expand. AI is no longer viewed as simply a tool for “lazy searching” but as a deeply integrated component of workflows, enabling faster processing, higher efficiency, and enhanced decision-making.

Today’s exploration focuses on how AI is utilised across cyber security, along with the important considerations that come with deploying AI within these environments.

<br></br>

### AI in Cyber Security

---
AI adoption across vendors and organisations has surged — not only as a marketing buzzword but as a meaningful enhancement to cyber operations. Below is a simplified breakdown of AI’s capabilities and how they map directly to cyber security:

| **Features of AI**               | **Cyber Security Relevance**                                                             |
| -------------------------------- | ---------------------------------------------------------------------------------------- |
| Processing large amounts of data | Analysing massive volumes of logs, telemetry, and other data types from multiple sources |
| Behaviour analysis               | Detecting anomalies by comparing activity to established behavioural baselines           |
| Generative AI                    | Summarising complex events or providing contextual reasoning around alerts               |

AI’s usefulness is prominent across multiple sectors: defence, offence, and software development.

<br></br>

### Defensive Security
---

AI agents assist blue teams by enhancing:

* **Detection** — automatically flagging abnormal network or endpoint activity
* **Investigation** — correlating logs and events faster than manual triage
* **Response** — autonomously isolating compromised systems, blocking malicious emails, or alerting on unusual authentication attempts

Vendors are beginning to embed AI directly into their appliances, such as:

* AI-assisted firewalls
* Intrusion detection/prevention systems
* Automated SOC tooling

AI boosts speed and consistency, allowing defenders to handle large-scale telemetry with significantly less manual overhead.

<br></br>


### Offensive Security

---

AI is revolutionising red-team workflows by automating traditionally slow and labour-intensive tasks like:

* Reconnaissance
* OSINT
* Processing large scanner outputs
* Mapping attack surfaces

This frees penetration testers to focus on the human-critical aspects of exploitation and creative problem-solving.

However, AI is **not** a complete replacement for human reasoning — it is a force multiplier.

<br></br>

### Software Security
---

While AI-generated code can be risky (“vibe coding”), AI contributes positively to software security by:

* Acting as a pseudo-colleague for ideation
* Assisting in code review
* Supporting automated **SAST/DAST** scanning
* Identifying vulnerabilities in code or applications

Ironically, AI is often *better at finding* vulnerabilities than writing secure code.

<br></br>


### Considerations of AI in Cyber Security
---

AI is powerful — but far from perfect. Security professionals must consider:

* Accuracy of AI-generated output (NEVER assume it’s 100% correct)
* Risks in offensive operations:
  e.g., AI triggering race conditions or overloading client systems
* Data privacy and training-data governance
* Transparency and fairness in AI-driven decision making
* Model security and protection from adversarial manipulation

AI should be treated as a tool that still requires human validation and oversight.

<br></br>

### Practical Exercise Overview
---

In the hands-on portion, users interact with **Van SolveIT**, an AI-powered assistant guiding them through three cyber security showcases:

1. **Red Team:** Generate and run an exploit script
2. **Blue Team:** Analyse web server attack logs
3. **Software Security:** Identify vulnerabilities in source code

The exercise is accessed at:

**http://MACHINE_IP**

Tips:

* Responses may take time to generate
* Restart the chat if the AI becomes confused
* Stages unlock progressively
* Use full-screen mode on the AttackBox for better visibility

<br></br>

## Final Answers

| **Question**                                                                              | **Answer**            |
| ----------------------------------------------------------------------------------------- | --------------------- |
| Complete the AI showcase. What is the final flag?                                         | **THM{AI_MANIA}**     |
| Execute the exploit generated by the red team AI agent. What flag does the script output? | **THM{SQLI_EXPLOIT}** |

<br></br>

## Conclusion

* AI is transforming cyber security across defensive, offensive, and software domains
* Despite its strengths, AI requires strict oversight, validation, and ethical deployment
* It can automate repetitive tasks but cannot replace human reasoning
* AI-powered agents like **Van SolveIT** demonstrate how AI can augment cyber operations in real-world simulations

---