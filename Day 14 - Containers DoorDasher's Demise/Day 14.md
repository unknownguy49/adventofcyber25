# Advent of Cyber 2025 - Day 14 - Containers: DoorDasher's Demise

## 1. Introduction

It seemed as the sun rose this morning, it had already been decided that today would be another day of chaos in Wareville. At least that’s the feeling all the folks at “DoorDasher” got. DoorDasher is Warevilles local food delivery site, a favourite of the workers in The Best Festival Company, especially on long days when they get home from work and just can’t bring themself to make dinner. We’ve all been there, I’m sure.

Well, one Wareville resident was feeling particularly tired this morning and so decided to order breakfast. Only to find King Malhare and his bandit bunny battalions had seized control of yet another festive favourite. DoorDasher had been completely rebranded as Hopperoo. All of the ware’s favourite dishes had been changed as well. Reports started flooding into the DoorDasher call centre. And not just from customers. The health and safety food org was on the line too, utterly panicked. Apparently, multiple Wareville residents were choking on what turned out to be fragments of Santa’s beard. Wareville authorities were left tangled in confusion today as Hopperoo faced mounting backlash over reports of “culinary impersonation.” Customers across the region claim to have been served what appears to be authentic strands of Santa’s beard in place of traditional noodles.

A spokesperson for the Health & Safety Food Bureau confirmed that several diners required “gentle untangling” and one incident involved a customer “achieving accidental facial hair synchronisation.”

Immediately, one of the security engineers managed to log on and make a script that would restore DoorDasher to its original state, but just before he was able to run it, Sir CarrotBaine caught wind of his attempt and locked him out of the system. All was lost, until the SOC team realised they still had access to the system via their monitoring pod, an uptime checker for the site. Your job? As a SOC team member of DoorDasher, can you escape the container and escalate your privileges so you can finish what your team started and save the site!

<br></br>

### Learning Objectives
---

- Learn how containers and Docker work, including images, layers, and the container engine
- Explore Docker runtime concepts (sockets, daemon API) and common container escape/privilege-escalation vectors
- Apply these skills to investigate image layers, escape a container, escalate privileges, and restore the DoorDasher service
- DO NOT order “Santa's Beard Pasta”

<br></br>

### Connecting to the Machine
---

Start the lab by clicking the Start Machine button below. The machine will start in split view and will take about two minutes to load. In case the machine is not visible, click the Show Split View button at the top of the page. Once the machine has loaded, you should be given access to the mrbombastic user. You will be given commands to run on this virtual machine in the next task. Additionally, start the AttackBox by pressing Start AttackBox down below. 

Note: It’s recommended to open both machines in full-screen view using the small icon on the far left in the screenshot below; otherwise, you might get kicked out of the Docker container when switching tabs in split view. If you still prefer to use split view, you can switch between the target machine and the AttackBox using the bottom tabs.

<br></br>

## 2. Container Security

### What Are Containers?
---

Modern applications are complex and often suffer from common deployment problems:

* **Installation issues** – environment-specific quirks slow setup
* **Troubleshooting pain** – hard to tell if failures come from the app or the system
* **Dependency conflicts** – multiple apps or versions competing for resources

The solution to all of this is **isolation**.

#### **Containerisation**

Containerisation packages an application **together with its dependencies** into a single isolated unit called a **container**. This guarantees consistent behaviour across environments and eliminates configuration conflicts.

Key benefit:

* **Lightweight execution**, since containers share the host OS kernel instead of shipping an entire OS.

<br></br>

### Containers vs Virtual Machines
---

#### **Virtual Machines (VMs)**

* Run on a **hypervisor**
* Include a full guest OS
* Heavy but fully isolated
* Best for running different operating systems or legacy software

#### **Containers**

* Share the **host OS kernel**
* Isolate only applications and dependencies
* Start quickly and use fewer resources
* Ideal for scalable microservices

Containers trade full OS isolation for speed and efficiency.

<br></br>

### Applications at Scale – Microservices
---

Traditional applications were **monolithic**: a single codebase deployed as one unit.

Modern applications increasingly use **microservices**, where:

* Each component serves a specific business function
* Individual components can scale independently
* High-traffic services scale without affecting the entire app

Containers are perfect for this model because they are:

* Lightweight
* Fast to deploy
* Easy to scale horizontally

<br></br>

### Container Engines & Docker

---

A **container engine** builds, runs, and manages containers using kernel features such as:

* Namespaces
* cgroups

#### **Docker**

Docker is the most popular container engine.

Key characteristics:

* Uses **Dockerfiles** to define environments
* Packages applications consistently
* Shares the host OS kernel
* Widely adopted in production systems

Docker is the engine used by **DoorDasher**, which we interact with in this lab.

<br></br>

### Container Escape & Docker Sockets
---

A **container escape** occurs when code running inside a container gains access to:

* The host system
* Other containers
* The Docker Engine itself

#### **Docker Architecture**

Docker uses a **client–server model**:

* Docker CLI → client
* Docker daemon → server
* Communication via **Unix socket** (`/var/run/docker.sock`)

If a container can access this socket, it can issue Docker API commands — effectively escaping isolation.

This misconfiguration enables attackers to:

* Start privileged containers
* Access restricted networks
* Modify host services

<br></br>

### Challenge Overview
---

#### **Objective**

Restore the defaced **Hopperoo** website back to its original **DoorDasher** service by exploiting Docker misconfigurations.

The main web service runs at:

```
http://MACHINE_IP:5001
```

The site has been defaced, indicating compromise.

<br></br>

### Discovering Running Containers
---

Check active containers:

```bash
docker ps
```

Several containers are running, including:

* Main web service
* `uptime-checker`
* `deployer`

The presence of auxiliary containers hints at a multi-container architecture.

<br></br>

### Docker Socket Escape via uptime-checker
---

Access the uptime-checker container:

```bash
docker exec -it uptime-checker sh
```

Check Docker socket access:

```bash
ls -la /var/run/docker.sock
```

The socket is exposed to the container.

Confirm Docker access:

```bash
docker ps
```

This confirms **Docker Escape** capability — the container can control the Docker daemon.

<br></br>

### Privileged Container Access
---

Access the privileged deployer container:

```bash
docker exec -it deployer bash
```

Check current user:

```bash
whoami
```

The container runs with elevated privileges.

Explore the filesystem and locate the recovery script.

<br></br>

### Restoring the Website
---

Run the recovery script:

```bash
sudo /recovery_script.sh
```

Refresh:

```
http://MACHINE_IP:5001
```

The Hopperoo site is restored to **DoorDasher**.

A flag is found in the root directory and read using:

```bash
cat /flag
```

<br></br>

### Bonus – Secret Code Discovery
---

Another service runs on port **5002**.

Investigating the news site reveals a secret code, which also serves as the password for the deployer user.

This highlights poor password hygiene and credential reuse.

<br></br>

## **Final Answers**

| Question                                                    | Answer                         |
| ----------------------------------------------------------- | ------------------------------ |
| What command lists running Docker containers?               | **docker ps**                  |
| What file defines instructions for building a Docker image? | **Dockerfile**                 |
| What’s the flag?                                            | **THM{DOCKER_ESCAPE_SUCCESS}** |
| Bonus: deployer password                                    | **DeployMaster2025!**          |

<br></br>

## **Conclusion**

* Containers solve deployment and dependency issues through isolation.
* Docker enables lightweight, scalable application delivery.
* Misconfigured Docker sockets allow **container escape attacks**.
* Access to `/var/run/docker.sock` effectively equals root-level control.
* Proper container hardening and isolation are critical in production systems.
* This challenge demonstrates how operational convenience can introduce severe security risks.

---