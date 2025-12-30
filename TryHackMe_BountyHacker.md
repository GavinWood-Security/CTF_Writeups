TryHackMe: Bounty Hacker
========================

This project is a walkthrough of the **Bounty Hacker** room on TryHackMe.\
It covers basic Linux enumeration, credential harvesting, and privilege escalation via sudo misconfiguration.

* * * * *

Basic Information
-----------------

-   **Room/Box name:** Bounty Hacker

-   **Platform:** TryHackMe

-   **Link:** <https://tryhackme.com/room/cowboyhacker>

-   **Difficulty:** Easy

* * * * *

Overview
--------

The **Bounty Hacker** room simulates a real-world scenario where poor service configuration and weak privilege controls allow an attacker to gain full root access. The attack chain involves anonymous FTP access, SSH credential brute forcing, and privilege escalation using an unsafe sudo rule.

* * * * *

Features
--------

-   Linux service enumeration

-   Anonymous FTP access

-   Credential discovery and brute force

-   SSH access

-   Privilege escalation via `sudo`

-   User and root flag retrieval

* * * * *

Task 1: Reconnaissance
----------------------

### Port Scanning

I ran the following Nmap command to enumerate open ports and services:

`nmap -sC -sV <target-ip>`

### Results

The scan revealed the following open ports:

-   **21** -- FTP (anonymous login enabled)

-   **22** -- SSH

-   **80** -- HTTP

* * * * *

Task 2: Gain Access
-------------------

### FTP Enumeration

I connected to the FTP service using anonymous credentials:

`ftp <target-ip>`

After listing the directory contents, I found two files:

-   `task.txt`

-   `locks.txt`

### Credential Discovery

-   `task.txt` revealed a valid system user: **lin**

-   `locks.txt` contained a list of potential passwords

### SSH Brute Force

I used **Hydra** to brute-force SSH using the discovered username and password list:

`hydra -l lin -P locks.txt ssh://<target-ip>`

Hydra returned valid credentials:

-   **Username:** `lin`

-   **Password:** `RedDr4gonSynd1cat3`

### SSH Login

I logged into the machine using SSH:

`ssh lin@<target-ip>`

Once logged in, I retrieved the user flag:

`cat user.txt`

* * * * *

Task 3: Privilege Escalation
----------------------------

### Enumeration with linPEAS

I transferred **linPEAS** to the target using SCP:

`scp linpeas.sh lin@<target-ip>:/tmp`

Then executed it on the target:

`chmod +x /tmp/linpeas.sh
/tmp/linpeas.sh`

### Findings

linPEAS revealed the following escalation paths:

-   The system appeared vulnerable to **CVE-2021-3560 (polkit)**

-   The user `lin` could execute `/bin/tar` as **root** via `sudo`

The sudo misconfiguration provided the quickest method of obtaining root access.

### Exploiting sudo `tar`

GNU `tar` supports checkpoint actions that allow arbitrary command execution. Since `tar` was allowed via `sudo`, I used the following command to spawn a root shell:

`sudo tar -cf /dev/null /dev/null\
--checkpoint=1\
--checkpoint-action=exec=/bin/sh`

### Verification

I confirmed root access by running:

`id`

* * * * *

Task 4: Find Flags
------------------

### Root Flag

With root access, I retrieved the final flag:

`cat /root/root.txt`

* * * * *

Conclusion
----------

This room demonstrates how small misconfigurations can lead to complete system compromise. The attack chain included:

1.  Service enumeration with Nmap

2.  Anonymous FTP access

3.  Credential discovery and brute forcing

4.  SSH access

5.  Privilege escalation via unsafe sudo configuration

### Key Takeaways

-   Anonymous services should be disabled whenever possible

-   Sudo permissions must follow the principle of least privilege

-   Misconfigurations are often more dangerous than unpatched vulnerabilities
