
# SMB Anonymous Access

## TAGS

smb-anon, smb, anonymous-access, enumeration, active-directory, windows

---

## 📌 Overview

SMB anonymous access occurs when a target allows unauthenticated users to enumerate shares or access share contents without valid credentials.

This can expose sensitive files, configuration data, internal documentation, scripts, backups, or credential material that leads to further compromise.

---

## 🔍 When to Use

IF:

* Port 445 is open
* The host appears to be Windows or AD-related
* You do not yet have credentials

THEN:
→ Test for anonymous SMB access immediately

---

## ✅ Requirements

* Network access to TCP/445
* SMB enumeration tools

---

## ⚙️ Enumeration

### 1. List Shares Anonymously

```bash
smbclient -L //<target> -N
```

Example:

```bash
smbclient -L //10.129.18.70 -N
```

---

### 2. Attempt to Access Interesting Shares

```bash
smbclient //<target>/<share> -N
```

Examples:

```bash
smbclient //10.129.18.70/Replication -N
smbclient //10.129.18.70/SYSVOL -N
smbclient //10.129.18.70/Users -N
```

---

### 3. Recursively Pull Files if Needed

From inside `smbclient`:

```text
recurse ON
prompt OFF
mget *
```

---

## 🎯 What to Look For

High-value findings include:

* `Groups.xml`
* `Services.xml`
* `ScheduledTasks.xml`
* Password files
* Scripts with hardcoded creds
* Backup files
* User home directories
* Internal documentation
* Configuration exports
* Old logs containing usernames or passwords

---

## 🚀 Next Steps

If anonymous SMB works:

* Check for GPP credentials
* Search for plaintext passwords
* Search for domain names and usernames
* Re-authenticate with any recovered creds
* Enumerate additional shares and permissions

---

## ⚠️ Detection Opportunities

* Anonymous share listing
* Unauthenticated access to sensitive shares
* Large file downloads from SMB
* Access to SYSVOL / Replication from unexpected hosts

---

## 🧠 Notes

* This is a high-value early enumeration step
* Particularly important on:

  * Domain controllers
  * File servers
  * Legacy Windows hosts
* Even read-only anonymous access can be enough for compromise
* Often chained with:

  * GPP credential extraction
  * Password spraying
  * LDAP enumeration once creds are obtained

---

## 🔗 Common Follow-On Patterns

* GPP credential extraction
* Authenticated SMB enumeration
* Kerberoasting
* LDAP user enumeration

---
