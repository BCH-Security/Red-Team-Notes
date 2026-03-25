#  Red Team Notes

## 🔹 Overview

This repository contains **practical Red Team notes and cheat sheets** focused on **Active Directory attacks, credential access, evasion techniques, and Command & Control (C2)**.

The goal is to provide **concise, hands-on references** for real-world engagements, labs, and certifications (e.g., OSCP, CRTO, CPTS).

---

#  Repository Structure

##  Credential Access

###  Dump SAM and LSA

* Extract local account hashes and secrets from:

  * SAM hive (local accounts)
  * LSA secrets (cached credentials, service accounts)
* Tools & techniques:

  * Impacket (`secretsdump`)
  * netexec
  * Meterpreter

---

###  Dump NTDS

* Dump Active Directory database (`ntds.dit`)
* Extract domain user hashes

🔹 Techniques:

* Offline dumping (VSS + `esentutl`)
* Remote DCSync attacks

🔹 Tools:

* Impacket
* Mimikatz
* netexec

---

##  Active Directory Attacks

###  AD-ACL Cheat Sheet

* Abuse of misconfigured ACLs:

  * `GenericAll`, `GenericWrite`
  * `WriteOwner`, `AddMember`, `ForceChangePassword`
* Delegation attacks:

  * Unconstrained Delegation
  * Constrained Delegation
  * Resource-Based Constrained Delegation (RBCD)

🔹 Tools:

* bloodyAD
* certipy

---

###  Kerberoast Attacks

* Kerberos-based attacks:

  * AS-REP Roasting
  * Kerberoasting
  * Silver Ticket
  * Golden Ticket

🔹 Tools:

* Impacket
* hashcat

---

##  Evasion Techniques

###  ETW-AMSI Evasion

* Bypass Windows defensive mechanisms:

  * ETW (Event Tracing for Windows)
  * AMSI (Antimalware Scan Interface)

🔹 Techniques:

* In-memory patching
* PowerShell reflection abuse

---

##  Command & Control (C2)

###  Sliver C2 Cheat Sheet

* Usage of Sliver:

  * Server & client setup
  * Implant generation
  * Session handling
  * Post-exploitation modules

---

# 🔹 Key Focus Areas

* Active Directory exploitation
* Credential dumping & abuse
* Kerberos attacks
* Privilege escalation
* Defense evasion
* C2 operations

---

#  Disclaimer

This repository is intended for:

* Educational purposes
* Authorized penetration testing
* Red Team engagements

❗ Do not use these techniques on systems without proper authorization.

---



# 🔹 Final Note

These notes are designed to be:

* **Practical** → copy/paste ready
* **Concise** → minimal theory, maximum utility
* **Real-world focused** → based on common attack paths

---
