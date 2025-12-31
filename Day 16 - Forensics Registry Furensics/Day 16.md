# Advent of Cyber 2025 - Day 16 - Forensics: Registry Furensics

## 1. Introduction

TBFC is under attack. Systems are exhibiting weird behavior, and the company is now feeling the absence of its lead defender, McSkidy. However, McSkidy made sure the legacy continues.

McSkidy’s team, determined and well-trained, is fully confident in securing all the systems and regaining control before the big event, SOCMAS.

They have now decided to conduct a detailed forensic analysis on one of the most critical systems of TBFC, dispatch-srv01. The dispatch-srv01 coordinates the drone-based gifts delivery during SOCMAS. However, recently it was compromised by King Malhare’s bandits of bunnies.

TBFC’s defenders have decided to split into specialized teams to uncover the attack on this system through detailed forensics. While some of the other team members investigate logs, memory dumps, file systems, and other artefacts, you will work to investigate the registry of this compromised system.

<br></br>

### Learning Objectives
---

- Understand what the Windows Registry is and what it contains.
- Dive deep into Registry Hives and Root Keys.
- Analyze Registry Hives through the built-in Registry Editor tool.
- Learn Registry Forensics and investigate through the Registry Explorer tool.

<br></br>

### Connecting to the Machine
---

Start your target machine by clicking the Start Machine button below. The machine will open in split view and will need about 2 minutes to fully boot. In case you can not see it, click the Show Split View button at the top of the page.

<br></br>

## 2. Investigate the Gifts Delivery Malfunctioning

### Windows Registry Overview
---

Just like the human brain stores behaviour, habits, and memories, Windows relies on the **Windows Registry** as its central configuration database.
The registry stores everything the OS needs to function properly, including:

* System configuration
* Installed software
* User preferences
* Security policies
* Startup programs
* Hardware information

Unlike a human brain, the registry is not stored in one place. Instead, it is split into multiple binary files called **Registry Hives**, each responsible for a specific category of configuration data.

<br></br>

### Registry Hives
---

Registry hives are physical files stored on disk. Each hive contains a specific class of configuration information.

| Hive Name    | Contains                                   | Location                                                         |
| ------------ | ------------------------------------------ | ---------------------------------------------------------------- |
| SYSTEM       | Services, boot config, drivers, hardware   | `C:\Windows\System32\config\SYSTEM`                              |
| SECURITY     | Local security policies, audit settings    | `C:\Windows\System32\config\SECURITY`                            |
| SOFTWARE     | Installed programs, autostarts, OS info    | `C:\Windows\System32\config\SOFTWARE`                            |
| SAM          | Usernames, password hashes, group info     | `C:\Windows\System32\config\SAM`                                 |
| NTUSER.DAT   | User preferences, recent files, autostarts | `C:\Users\username\NTUSER.DAT`                                   |
| USRCLASS.DAT | Shellbags, Jump Lists                      | `C:\Users\username\AppData\Local\Microsoft\Windows\USRCLASS.DAT` |

Each hive stores far more data than the examples listed above.

<br></br>

### Registry Editor & Root Keys
---

Registry hive files are **binary** and cannot be opened directly.
Windows provides the **Registry Editor** to view registry data.

When opening the Registry Editor, you don’t see hive names directly. Instead, Windows presents **Root Keys**, which internally map to hive files.

#### **Hive to Root Key Mapping**

| Hive on Disk | Registry Editor Location                   |
| ------------ | ------------------------------------------ |
| SYSTEM       | `HKEY_LOCAL_MACHINE\SYSTEM`                |
| SECURITY     | `HKEY_LOCAL_MACHINE\SECURITY`              |
| SOFTWARE     | `HKEY_LOCAL_MACHINE\SOFTWARE`              |
| SAM          | `HKEY_LOCAL_MACHINE\SAM`                   |
| NTUSER.DAT   | `HKEY_USERS\<SID>` and `HKEY_CURRENT_USER` |
| USRCLASS.DAT | `HKEY_USERS\<SID>\Software\Classes`        |

> **Note:**
> `HKEY_CLASSES_ROOT (HKCR)` and `HKEY_CURRENT_CONFIG (HKCC)` are dynamically generated and do not correspond to separate hive files.

<br></br>

### Extracting Information from the Registry
---

The registry works much like a file system. If you know the correct path, you can directly retrieve the required information.

#### **Example 1: USB Devices Connected to the System**

* Hive: `SYSTEM`
* Path:

```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\USBSTOR
```

This key stores:

* USB make and model
* Device IDs
* Unique identifiers per connected device

---

#### **Example 2: Programs Run via Win + R**

* Hive: `NTUSER.DAT`
* Path:

```
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\RunMRU
```

This key stores commands executed via the **Run dialog**.

These artefacts are extremely valuable during forensic investigations.

<br></br>

### Registry Forensics
---

**Registry forensics** involves extracting and analysing registry data to reconstruct user activity and system changes during an incident.

Commonly analysed forensic registry keys include:

| Registry Key              | Forensic Importance            |
| ------------------------- | ------------------------------ |
| `HKCU\...\UserAssist`     | GUI-launched applications      |
| `HKCU\...\TypedPaths`     | Paths typed in Explorer        |
| `HKLM\...\App Paths`      | Application installation paths |
| `HKCU\...\WordWheelQuery` | Explorer search history        |
| `HKLM\...\Run`            | Startup persistence            |
| `HKCU\...\RecentDocs`     | Recently accessed files        |
| `HKLM\...\ComputerName`   | Hostname                       |
| `HKLM\...\Uninstall`      | Installed applications         |

#### **Why Not Registry Editor?**

* Cannot open **offline hives**
* Displays binary data unreadably
* Risks modifying evidence on live systems

Instead, forensic analysts use dedicated tools.

<br></br>

### Practical – Registry Explorer Analysis
---

#### **Objective**

Analyse registry hives from the compromised system **dispatch-srv01** to identify malicious activity.

Hive location:

```
C:\Users\Administrator\Desktop\Registry Hives
```

#### **Step 1: Launch Registry Explorer**

* Click the Registry Explorer icon from the taskbar

#### **Step 2: Load Offline Registry Hives**

1. Click **File**
2. Select **Load hive**
3. Navigate to the Registry Hives folder

#### **Step 3: Handling Dirty Hives**

Registry hives collected from live systems may be **dirty** (incomplete transactions).

To load cleanly:

1. Select a hive (e.g., SYSTEM)
2. Hold **SHIFT**
3. Click **Open**
4. Allow transaction logs to replay
5. Repeat for remaining hives

This ensures a consistent forensic state.

#### **Step 4: Investigate Registry Keys**

To identify the hostname:

Path:

```
ROOT\ControlSet001\Control\ComputerName\ComputerName
```

Value discovered:

```
DISPATCH-SRV01
```

You may also use the **search bar** or **Available Bookmarks** to locate keys faster.

> **Note:**
> Abnormal activity began on **21st October, 2025**.

<br></br>

## **Final Answers**

| Question                                                 | Answer                                                           |
| -------------------------------------------------------- | ---------------------------------------------------------------- |
| What application was installed before abnormal activity? | **DroneManager Updater**                                         |
| Full path where the application was launched from        | **C:\Users\dispatch.admin\Downloads\DroneManager_Setup.exe**     |
| Persistence value added by the application               | **"C:\Program Files\DroneManager\dronehelper.exe" --background** |

<br></br>

## **Conclusion**

* The Windows Registry acts as the OS configuration backbone.
* Registry hives store system, user, and security data in binary form.
* Offline registry analysis is essential for forensic integrity.
* Registry Explorer enables safe parsing of offline hives.
* Startup persistence via Run keys is a common attacker technique.
* Registry artefacts helped identify malicious software installation, execution path, and persistence on **dispatch-srv01**.

---