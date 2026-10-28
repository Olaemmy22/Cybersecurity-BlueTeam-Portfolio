# 📑 Blue Team Lab Log & Practical Portfolio

**Author:** Olaemmy22  
**Location:** Göteborg, Sweden  
**Current Academic Standing:** Computer Science (300L) | Current CGPA: 4.53  
**Focus Area:** Defensive Security / Blue Team Operations  

---

## 🛠️ Lab Environment Architecture
To perform safe analysis without endangering the host operating system or local network, an isolated malware analysis platform was constructed:
*   **Hypervisor:** Oracle VM VirtualBox running on a Windows 11 host.
*   **Guest Operating System:** REMnux (Noble Linux distribution built specifically for malware reverse engineering and security analysis).
*   **Network Security Shield Configuration:** Network Interface Card (NIC) set exclusively to **Host-only Adapter** mode. This ensures complete network isolation, preventing any executing artifacts from accessing the physical local area network or the public internet.

---

## 💻 Directory & System Commands Discovered

### 1. File & Directory Navigation
*   **`pwd` (Print Working Directory)**
    *   *Purpose:* Displays the absolute path of the directory the analyst is currently working within.
    *   *Lab Output:* `/home/remnux`
*   **`ls -la` (List All - Long Format)**
    *   *Purpose:* Lists all files and subdirectories. The `-l` flag displays file details (permissions, owner, size, date). The `-a` flag uncovers hidden files starting with a dot (`.`), a critical technique for discovering hidden malware scripts.

### 2. Linux Permissions Matrix Structure
Observed string architecture during lab logging: `drwxr-xr-x` vs `-rw-r--r--`
*   **First Character Indicator:** 
    *   `d` = Directory (Folder)
    *   `-` = Regular Data File (e.g., `.bashrc`)
*   **Security Access Blocks:**
    *   `r` = Read permissions
    *   `w` = Write (edit) permissions
    *   `x` = Execute (run a program) permissions

### 3. Search & Text Automation
*   **`grep -i "keyword" [filename]` (Global Regular Expression Print)**
    *   *Purpose:* Acts as a high-speed keyword scanner inside text files. The `-i` flag ignores case-sensitivity. 
    *   *Defensive Use Case:* Used to quickly isolate malware indicators (IP addresses, rogue domains) from massive logs without reading line-by-line.

### 4. Process & Traffic Monitoring
*   **`top` (Table of Processes)**
    *   *Purpose:* Real-time system diagnostics interface (Linux Task Manager). Monitors CPU/Memory allocation. High CPU usage combined with low Idle space (`id`) triggers an investigation for unauthorized processes (like cryptojacking or active ransomware).
*   **`ss -tulpn` (Socket Statistics)**
    *   *Purpose:* Displays an instant inventory of active network connections and listening ports. Filters specifically for **T**CP, **U**DP, **L**istening ports, displaying the **P**rogram name and raw **N**umeric port numbers.
    *   *Defensive Use Case:* Identifies unauthorized doors left open or malware establishing active connections to a Command and Control (C2) server.

---

## 🦈 Network Traffic Sniffing (Wireshark Laboratory)

### Execution Command: `wireshark &`
*   *Purpose:* Launches the Wireshark GUI packet analyzer tool. The `&` symbol forces the program to execute in the background, keeping the parent terminal open.

### Intercepted Protocols & Layer Analysis

| Protocol | Description | Layer Connection (OSI Model) |
| :--- | :--- | :--- |
| **DHCP** | Dynamic Host Configuration Protocol. Automatically handles IP address assignments to devices on the network. | Layer 7 (Application) |
| **ARP** | Address Resolution Protocol. Maps IP addresses to physical MAC hardware addresses. | Layer 2 (Data Link) |

### Active Lab Analysis Exercise
During packet sniffing on interface **`enp0s3`**, a traffic capture was frozen using the **Stop (Red Square)** control:
1.  **Hardware Identification:** Successfully isolated the physical interface identity of the testing machine through an ARP broadcast: `PCSSystemtec_c6:ee:...`
2.  **Traffic Filtering Operations:** Applied the display filter `arp` into the Wireshark filter console. This instantly purged background DHCP broadcast noise, isolating specific Layer 2 validation logs.
3.  ---

## 🦠 Lab Log 02: Hidden Trojan Script & IoC Extraction

### Why this analysis was conducted:
In modern enterprise intrusions, attackers often deploy lightweight script components to bypass standard endpoint antivirus definitions. This lab simulates an incident response investigation where a rogue shell script (`backup-scheduler.sh`) disguised as an administrative utility was analyzed.

### Investigation & Extraction Steps:
1. **Code Audit:** Evaluated the file text using command line utilities to identify data transfer mechanisms (`curl -X POST`). The script was determined to be searching local authentication arrays to silently exfiltrate active user profile names.
2. **Indicator of Compromise (IoC) Tracking:** Executed a targeted string filter using the `grep` search utility to isolate embedded malicious pathways:
   * **Command:** `grep "http://" backup-scheduler.sh`
   * **Extracted Attacker IP:** `193.12.55.84`

### Strategic Defense Implementation:
To neutralize the active threat component without causing system disruption, an explicit **Outbound Block Rule** was deployed on the network firewall targeting the extracted destination IP (`193.12.55.84`). This ensures data exfiltration loops drop immediately at the boundary perimeter.
---

## 📧 Lab Log 03: Digital Forensics & Phishing Header Analysis

### Why this analysis was conducted:
Phishing via brand impersonation accounts for the vast majority of initial access attempts in corporate infrastructures. This lab simulates an incident response investigation where a corporate user flagged an email claiming to be an official shipping alert from PostNord.

### Forensic Investigation Steps:
1. **Header Identification:** Extracted the raw headers (`phishing_header.txt`) to audit the electronic routing trail instead of relying on the visible sender profile.
2. **Authentication Discrepancy:**
   * **Visible From Line:** `PostNord Sverige <kundservice@postnord.se>`
   * **Actual Origin Server (Received Header):** `mail.hackerhost.ru (195.22.148.90)`
3. **The Verdict:** The email is an active phishing spoofing attempt. The transaction records prove the transmission bypassed legitimate distribution networks entirely.

### Incident Remediation Actions:
* Deployed an immediate **Inbound Block Rule** on the corporate firewall targeting the rogue origin server IP (`195.22.148.90`).
* Added the sender domain (`postnord-update.se`) to the central mail gateway's absolute untrusted list to filter future variations automatically.
### 🛡️ Live Firewall Mitigation Addendum
To turn theoretical remediation into practical infrastructure defense, the extracted phishing origin signature was actively blocked using the Linux kernel's native packet filtering architecture:
* **Deployment Command:** `sudo iptables -A INPUT -s 195.22.148.90 -j DROP`
* **Verification Command:** `sudo iptables -L -n -v`

**Analysis & Impact:** The `iptables` ruleset database successfully logged the state restriction under the `INPUT` chain, hard-coding a strict drop policy for all packets originating from the hostile host (`195.22.148.90`). This actively shields the local system endpoint from subsequent reconnaissance or exploitation attempts by this actor.
---

## 📊 Lab Log 04: Server Authentication Audit & Brute-Force Triage

### Why this analysis was conducted:
Automated password-guessing (Brute-Force) attacks target open authentication gateways continuously. This lab simulates an incident response investigation where server authentication logs (`auth.log`) were triaged to isolate malicious access attempts and verify perimeter health.

### Forensic Investigation Steps:
1. **Log Ingestion:** Evaluated the native Linux authentication logging array (`/var/log/auth.log`) to parse active session requests.
2. **Automated Search Filtering:** Executed a case-sensitive string match using the `grep` utility to instantly filter malicious failures from standard background traffic:
   * **Command:** `grep "Failed password" auth.log`
3. **Attack Metrics Isolated:**
   * **Hostile External Source IP:** `198.51.100.42`
   * **Targeted High-Privilege Profiles:** `admin`, `root`
   * **Perimeter Verification:** The final record confirmed a legitimate internal connection (`172.16.5.2`) passing validation, proving the brute-force attempt was entirely mitigated.

### Incident Remediation Actions:
* Leveraged the host security system to isolate the hostile IP footprint (`198.51.100.42`) from initiating future session layer handshakes.
