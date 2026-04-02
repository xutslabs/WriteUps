# WriteUps

Structured penetration testing write-ups and attack chains from Hack The Box, Proving Grounds, and lab environments.
Focused on clear methodology, enumeration, and reproducible exploitation paths.

---

## 🔴 Writeups – HTB & Proving Grounds

A curated collection of completed machines from Hack The Box and Proving Grounds.

This repository documents methodology, tooling, and thought process across a wide range of targets — from initial enumeration to full compromise.

The goal is not just to solve boxes, but to build **repeatable, real-world tradecraft**.

---

## 📌 Purpose

* Build a structured knowledge base of real-world attack paths
* Reinforce methodology over memorization
* Track growth across difficulty levels
* Serve as a reference for future engagements (CTF, OSCP, red teaming, etc.)

---

## 🧠 Approach

Each writeup focuses on:

* Enumeration → Initial Access → Privilege Escalation
* Why something worked (not just what worked)
* Alternative paths when applicable
* Key takeaways / lessons learned

---

## 🤖 EXO Integration (Private Use)

Each machine includes an additional file:

```text
exo.md
```

This file is a **structured, machine-readable version** of the attack chain.

Unlike the main writeup, which is designed for humans, `exo.md` is designed for:

* Automation
* Pattern recognition
* Decision support during engagements

### Key Characteristics

* Step-based logic instead of narrative
* Explicit commands and outcomes
* Clear decision points (IF / THEN logic)
* Credential tracking and attack chaining

### Purpose

These files are used to power a private red team assistant ("EXO") that can:

* Recognize attack patterns across targets
* Suggest next steps based on observed conditions
* Reuse proven techniques from previous machines

> Think of `exo.md` as a **playbook**, not a writeup.

---

## 🧩 Pattern Library

This repository also includes a `/patterns/` directory.

These are **reusable attack techniques extracted from real machines**.

### Example Patterns

* GPP Credential Extraction
* Kerberoasting
* SMB Anonymous Enumeration

---

### Why Patterns Matter

Each individual writeup shows:

> “What worked on this box”

Pattern files generalize that into:

> “When you see this → do this”

---

### Example Logic

```text
IF:
- SMB allows anonymous access
- SYSVOL or Replication is accessible

THEN:
→ Check for GPP credentials
```

---

### Outcome

This allows:

* Faster enumeration decisions
* Reduced guesswork
* Reusable attack chains across environments
* Foundation for automation and AI-assisted operations

---

## 📂 Repository Structure

```text
writeups/
│
├── htb/
│   └── <machine>/
│       ├── README.md        # Human writeup
│       ├── exo.md           # Structured attack chain
│       ├── scans/
│       ├── loot/
│       └── screenshots/
│
├── pg/
│   └── ...
│
patterns/
├── gpp.md
├── kerberoast.md
└── smb-anon.md
```

---

## 📁 Machine Layout

```text
Machine-Name/
├── README.md        # Full writeup (human-readable)
├── exo.md           # Machine-readable attack chain
├── scans/           # nmap, gobuster, etc.
├── loot/            # creds, hashes, artifacts
└── screenshots/     # proof / key steps
```

---

## 🏷️ Tagging System

Each machine and pattern includes tags to build a **searchable attack index**.

### Categories

* **Techniques:**
  `gpp`, `kerberoast`, `smb-anon`, `lfi`, `rce`

* **Services:**
  `smb`, `ldap`, `kerberos`, `http`, `ftp`

* **Platform:**
  `windows`, `linux`, `active-directory`

* **Tools:**
  `impacket`, `hashcat`, `bloodhound`, `nmap`

* **Difficulty:**
  `easy`, `medium`, `hard`

---

## 🛠️ Tooling

Common tools used across engagements:

* nmap
* gobuster / ffuf
* enum4linux / smbclient
* linpeas / winpeas
* impacket
* burpsuite
* hashcat
* Custom scripts & one-offs

---

## ⚠️ Disclaimer

* These writeups are for educational purposes only
* All targets were legally accessed lab environments (HTB / PG)
* Do not attempt these techniques on systems without proper authorization

---

## 🚀 Progress Tracking

| Platform        | Status      | Notes              |
| --------------- | ----------- | ------------------ |
| Hack The Box    | In Progress | Active grinding    |
| Proving Grounds | In Progress | Focus on OSCP prep |

---

## 🧠 Notes

* Some writeups may be redacted or delayed depending on platform rules
* Methodology will evolve as skillset improves
* Expect inconsistencies early on — this repository reflects real growth

---
