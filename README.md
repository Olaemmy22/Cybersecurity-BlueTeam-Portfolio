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
