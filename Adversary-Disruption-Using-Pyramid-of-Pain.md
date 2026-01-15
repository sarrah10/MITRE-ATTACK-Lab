# TryHackMe “[Summit](https://tryhackme.com/room/summit)” Walkthrough  
### Chasing an Adversary up the Pyramid of Pain

This write-up documents my completion of the **TryHackMe Summit lab**, where the objective is to progressively disrupt a simulated adversary by forcing them up the **Pyramid of Pain** until they abandon their campaign.

The lab demonstrates how defenders evolve detections from **simple indicators** (hashes, IPs) to **high-impact behavioral detections** (TTPs).

---

## Lab Objective

- Identify attacker indicators
- Implement detections at increasing levels of difficulty
- Force the adversary to adapt repeatedly
- Ultimately disrupt attacker techniques and procedures

---

## 1️⃣ Blocking `sample1.exe` – Hash-Based Detection

### 🔍 Analysis
The malware sample `sample1.exe` was submitted to the **Malware Sandbox**, where file hashes were identified.

### 🛡️ Detection
- Indicator Type: **File Hash (MD5)**
- MD5 Hash:
```
cbda8ae000aa9cbe7c8b982bae006c2a
```
- Action: Added to **Manage Hashes** to block execution

### 🎯 Result
Successfully blocked the malware based on its hash.


🧠 *Pyramid of Pain Level:* Hash Values (Low)

---

## 2️⃣ Blocking `sample2.exe` – IP-Based Detection

### 🔍 Analysis
The attacker modified the malware to evade hash detection. Sandbox analysis revealed outbound HTTP traffic to a command-and-control server.

- Destination IP: `154.35.10.113`
- Port: `4444`

### 🛡️ Detection
A firewall rule was created:
- Type: **Egress**
- Source IP: Any
- Destination IP: `154.35.10.113`
- Action: **Deny**

### 🎯 Result
Outbound communication to the C2 server was blocked.


🧠 *Pyramid of Pain Level:* IP Addresses

---

## 3️⃣ Blocking `sample3.exe` – Domain-Based Detection

### 🔍 Analysis
The attacker pivoted to cloud infrastructure, frequently changing IPs. Sandbox analysis revealed a hardcoded domain:
```
emudyn.bresonicz.info
```

### 🛡️ Detection
A DNS filtering rule was created:
- Domain: `emudyn.bresonicz.info`
- Action: **Deny**

### 🎯 Result
Blocking the domain disrupted the attacker’s rotating infrastructure.


🧠 *Pyramid of Pain Level:* Domain Names

---

## 4️⃣ Blocking `sample4.exe` – Host Artifact Detection

### 🔍 Analysis
Sandbox results showed **registry modifications** designed to disable Windows Defender real-time monitoring.

- Registry Key:
```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Defender\Real-Time Protection
```
- Registry Value:
```
DisableRealtimeMonitoring = 1
```

### 🛡️ Detection
A **Sigma rule** was created using:
- Log Source: Sysmon Event Logs
- Event Type: Registry Modification
- MITRE ATT&CK Tactic: **Defense Evasion (TA0005)**

### 🎯 Result
Detection of malicious host-level artifacts.


🧠 *Pyramid of Pain Level:* Network / Host Artifacts

---

## 5️⃣ Detecting `sample5.exe` – Beaconing Behavior

### 🔍 Analysis
The provided `outgoing_connections.log` revealed suspicious network behavior:
- Repeated connections every **30 minutes**
- Packet size consistently **97 bytes**
- Destination IP frequently changing

This pattern strongly indicated **C2 beaconing**.

### 🛡️ Detection
A Sigma rule was created:
- Log Source: Sysmon Event Logs
- Event Type: Network Connections
- Frequency: 1800 seconds
- Packet Size: 97 bytes
- MITRE ATT&CK Tactic: **Command and Control (TA0011)**

### 🎯 Result
Behavior-based detection independent of IPs or domains.


🧠 *Pyramid of Pain Level:* Tools / Behavioral Patterns

---

## 6️⃣ Detecting Discovery Activity – `exfiltr8.log`

### 🔍 Analysis
Command logs showed extensive system and network discovery commands writing output to:
```
%temp%\exfiltr8.log
```

This activity aligns with **Discovery (TA0007)**.

### 🛡️ Detection
A Sigma rule was created:
- Log Source: Sysmon Event Logs
- Event Type: File Creation / Modification
- File Name: `exfiltr8.log`
- Directory: `%temp%`
- MITRE ATT&CK Tactic: **Discovery (TA0007)**

### 🎯 Result
Detection of attacker discovery techniques at the highest level of the Pyramid of Pain.


---

## ✅ Conclusion

This lab demonstrated how progressively stronger detections:
- Increase attacker effort
- Reduce attacker effectiveness
- Ultimately force adversaries to abandon their campaign

The **Summit lab** reinforces the importance of **behavioral and TTP-based detections** over simple indicators.

---

## 📌 Key Frameworks Used
- Pyramid of Pain
- MITRE ATT&CK
- Sigma Detection Rules
- Sysmon Logs
