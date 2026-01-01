# Advent of Cyber 2025 - Day 19 - ICS/Modbus: Claus for Concern

## 1. Introduction

The snow falls heavily over Wareville as chaos erupts at TBFC headquarters. What should be the busiest shipping day of the season has turned into a disaster.

"Another chocolate egg?!" shouts a frustrated warehouse worker, holding up yet another Easter-themed package."We're supposed to be shipping Christmas presents!"

The delivery drones buzz overhead, their mechanical hums sounding almost... mocking. Each one returns from its route empty, having successfully delivered its cargo. But the cargo is all wrong.

You're called into the command centre, where screens flicker with delivery statistics. Everything looks normal on the surface—1,000 presents in stock, 98% success rate, and all systems are operational. But the phones won't stop ringing with confused citizens asking why they're receiving chocolate eggs instead of the toys and gifts they ordered.

The logistics manager pulls up a delivery manifest. "Look at this", she says, pointing at the screen."The system indicates that we delivered a teddy bear to the Miller family, but they received a chocolate bunny instead. It's the same weight, exact dimensions, but completely different items."

Then, on one of the monitoring screens, a message flashes for just a second before disappearing:

```
🐰 EGGSPLOIT v6.66 - Property of HopSec Island 🐰
"Why should Christmas have all the fun?" - King Malhare
```

Someone has compromised the drone fleet's control systems. The attack is sophisticated, falsifying sensor data, manipulating inventory selection, and erasing all traces. This isn't just a prank—it's a calculated assault on Christmas itself.

Your mission is clear: investigate the TBFC Drone Delivery System, uncover how King Malhare's Eggsploit team has compromised it, and restore Christmas deliveries before SOC-mas is ruined.

But be warned: King Malhare doesn't leave systems undefended. Traps are waiting for the careless investigator. One wrong move and you might make things much worse.

### A Mysterious Discovery

As you walk through the warehouse control room, something catches your eye—a crumpled piece of paper on the floor near the PLC terminal. It looks like someone dropped it in a hurry.

You pick it up and unfold it. The handwriting is hurried, almost frantic:

```
TBFC DRONE CONTROL - REGISTER MAP
(For maintenance use only)

HOLDING REGISTERS:
HR0: Package Type Selection
     0 = Christmas Gifts
     1 = Chocolate Eggs
     2 = Easter Baskets

HR1: Delivery Zone (1-9 normal, 10 = ocean dump!)

HR4: System Signature/Version
     Default: 100
     Current: ??? (check this!)

COILS (Boolean Flags):
C10: Inventory Verification
     True = System checks actual stock
     False = Blind operation

C11: Protection/Override
     True = Changes locked/monitored
     False = Normal operation

C12: Emergency Dump Protocol
     True = DUMP ALL INVENTORY
     False = Normal

C13: Audit Logging
     True = All changes logged
     False = No logging

C14: Christmas Restored Flag
     (Auto-set when system correct)

C15: Self-Destruct Status
     (Auto-armed on breach)

CRITICAL: Never change HR0 while C11=True!
Will trigger countdown!

- Maintenance Tech, Dec 19
```
 
You stare at the note, confusion washing over you. "Register map? Coils? What is all this?"

The terminology is foreign—HR0, C11, "Modbus" scribbled in the margin. But something about it feels important, like a key you don't yet know how to use.

You pocket the note carefully. "I'll figure out what this means later", you think. For now, you need to understand the systems you're dealing with.

Little do you know, this crumpled note will be exactly what saves Christmas...

<br></br>

### Learning Objectives
---

- How SCADA (Supervisory Control and Data Acquisition) systems monitor industrial processes
- What PLCs (Programmable Logic Controllers) do in automation
- How the Modbus protocol enables communication between industrial devices
- How to identify compromised system configurations in industrial systems
- Techniques for safely remediating compromised control systems
- Understanding protection mechanisms and trap logic in ICS environments

<br></br>

### Connecting to the Machine
---

Start your target machine by clicking the Start Machine button below. The machine will need about 2 minutes to fully boot. Additionally, start your AttackBox by clicking the Start AttackBox button below. The AttackBox will start in split view. In case you can not see it, click the Show Split View button at the top of the page.

<br></br>

## 2. SCADA, PLCs & Modbus — Industrial Control System Compromise & Recovery

### SCADA (Supervisory Control and Data Acquisition)
--- 

#### **What is SCADA?**

SCADA systems act as the **central command layer** for industrial operations. They collect data from physical processes, present it to operators, and allow control actions to be issued back to machinery.

In TBFC’s environment, SCADA orchestrates:

* Drone loading
* Inventory routing
* Conveyor control
* Delivery validation

Without SCADA, large-scale automation like Christmas logistics would collapse.

#### **Core Components of a SCADA System**

| Component               | Role                                                                |
| ----------------------- | ------------------------------------------------------------------- |
| **Sensors & Actuators** | Sensors observe the environment; actuators perform physical actions |
| **PLCs**                | Execute automation logic and make decisions                         |
| **Monitoring Systems**  | Dashboards, CCTV feeds, alarms                                      |
| **Historians**          | Databases storing operational and audit data                        |

#### **SCADA in TBFC’s Drone System**

The compromised SCADA system controlled:

* **Package Type Selection** – which item is loaded onto drones
* **Delivery Zone Routing** – zones 1–9 normal, zone 10 emergency disposal
* **CCTV Monitoring** – real-time warehouse visibility
* **Inventory Verification** – stock validation before loading
* **Protection Mechanisms** – detect and react to changes
* **Audit Logging** – record all system modifications

Attackers abused these controls rather than breaking the system outright.

#### **Why SCADA Systems Are High-Value Targets**
--- 

SCADA systems are frequently attacked because:

* Legacy software with unpatched vulnerabilities
* Default credentials left unchanged
* Designed for availability, not security
* Control **physical processes**
* Often connected to corporate networks
* Protocols like **Modbus lack authentication**

Modbus TCP (port **502**) allows unauthenticated reads and writes to control registers.

<br></br>

### PLC & Modbus Protocol
---

#### **What is a PLC?**

A **Programmable Logic Controller (PLC)** is a hardened industrial computer designed to:

* Run continuously (24/7)
* Operate in harsh conditions
* Respond in real time
* Interface directly with physical hardware

PLCs do not prioritize security — reliability always comes first.

#### **What is Modbus?**

Modbus is a **simple request-response protocol** created in 1979. Its simplicity is both its strength and its weakness.

Key characteristics:

* No authentication
* No encryption
* No authorization
* Plaintext communication

Anyone who can reach port 502 can fully control the system.

#### **Modbus Data Types**

| Type              | Purpose               | Writable |
| ----------------- | --------------------- | -------- |
| Coils             | Digital outputs       | Yes      |
| Discrete Inputs   | Digital inputs        | No       |
| Holding Registers | Numeric configuration | Yes      |
| Input Registers   | Sensor values         | No       |

#### **TBFC Modbus Mapping**

**Holding Registers**

* HR0 – Package type
* HR1 – Delivery zone
* HR4 – System signature

**Coils**

* C10 – Inventory verification
* C11 – Protection mechanism
* C12 – Emergency dump
* C13 – Audit logging
* C14 – Christmas restored
* C15 – Self-destruct armed

<br></br>

### Initial Reconnaissance
---

#### **Service Discovery**

```bash
nmap -sV -p 22,80,502 MACHINE_IP
```

**Key Services Identified**

* Port 80 – HTTP (CCTV feed)
* Port 502 – Modbus TCP
* Port 22 – SSH

#### **Visual Confirmation**

Accessing the CCTV feed showed:

* Chocolate eggs instead of Christmas gifts
* Easter packaging
* Status marked **Compromised**

This confirmed **logic manipulation**, not system failure.

<br></br>

### Modbus Enumeration via Python
---

#### **Connecting to the PLC**

```python
from pymodbus.client import ModbusTcpClient
client = ModbusTcpClient('MACHINE_IP', port=502)
client.connect()
```

No authentication required — a critical vulnerability.

#### **Reading Holding Registers**

**HR0 – Package Type**

```python
client.read_holding_registers(0,1,slave=1)
```

Result: `1` → Chocolate Eggs

**HR1 – Delivery Zone**

```python
client.read_holding_registers(1,1,slave=1)
```

Result: `5` → Normal zone

**HR4 – System Signature**

```python
client.read_holding_registers(4,1,slave=1)
```

Result: `666` → Eggsploit detected

#### **Reading Coils**

* **C10** Inventory verification → `False`
* **C11** Protection active → `True`
* **C15** Self-destruct armed → `False`

The note warning now made sense:

> *Never change HR0 while C11=True*

<br></br>

### Threat Analysis
---

The attacker:

* Used unauthenticated Modbus access
* Forced egg delivery
* Disabled verification and logging
* Enabled a trap mechanism
* Left a signature value (666)

<br></br> 

### Safe Remediation Strategy
---

Order **matters**.

1. Disable protection (C11)
2. Change package type (HR0 → 0)
3. Enable inventory verification (C10)
4. Enable audit logging (C13)
5. Verify no self-destruct triggered

<br></br>

### Restoration Execution
---

A Python remediation script safely restored:

* Christmas package delivery
* Verification and logging
* System stability

The CCTV feed confirmed success in real time.

<br></br>

### Post-Incident Lessons
---
* ICS systems must be handled **methodically**
* Protocol-level access bypasses GUIs entirely
* Changing values blindly can trigger physical damage
* Understanding system logic is essential before remediation

<br></br>

## **Answers**

| Question        | Answer            |
| --------------- | ----------------- |
| Modbus TCP port | `502`             |
| Final flag      | `THM{eGgMas0V3r}` |

<br></br>

## **Conclusion**

You successfully:

* Investigated a SCADA compromise
* Interacted directly with a PLC
* Navigated Modbus safely
* Avoided a destructive trap
* Restored mission-critical infrastructure

Christmas is saved — and King Malhare is outplayed. 🎄
