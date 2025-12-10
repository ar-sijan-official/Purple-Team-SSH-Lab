# **Purple Team SSH Lab — Brute Force Detection with Splunk**

![Splunk](https://img.shields.io/badge/SIEM-Splunk-black?style=for-the-badge\&logo=splunk)
![Kali Linux](https://img.shields.io/badge/Attacker-Kali_Linux-blue?style=for-the-badge\&logo=kalilinux)
![Windows](https://img.shields.io/badge/Victim-Windows_10-blue?style=for-the-badge\&logo=windows)

---

<h3 align="center">📺 Watch the Full Project Walkthrough</h3>

<p align="center">
  <a href="https://www.youtube.com/watch?v=Y8wrNAIpDtA" target="_blank">
    <img src="https://img.youtube.com/vi/Y8wrNAIpDtA/maxresdefault.jpg" 
         alt="Purple Team SSH Lab Walkthrough" 
         width="700">
  </a>
</p>

---

## 📌 **Overview**

This repository contains a hands-on **Purple Team** home-lab project that demonstrates how to detect and visualize **SSH Brute Force attacks** using Splunk.

The simulation includes:

* Launching a brute-force attack from **Kali Linux** using **Hydra**
* Running an **OpenSSH Server** on a Windows 10 VM
* Forwarding logs to **Splunk Enterprise** via Universal Forwarder
* Creating a custom Splunk dashboard for:

  * Failed login analysis
  * Username enumeration
  * Attacker IP extraction
  * Geo-visualization

This lab showcases threat emulation, log engineering, SPL mastery, and SIEM analytics.


<img width="1900" height="986" alt="Screenshot from 2025-12-10 01-18-15" src="https://github.com/user-attachments/assets/4f2abb6e-7ae7-4aec-8f77-1c943c49f606" />
---

## 🏗️ **Network Architecture**

| Role                        | System / OS     | IP               | Tools                                      |
| --------------------------- | --------------- | ---------------- | ------------------------------------------ |
| **Attacker (Red Team)**     | Kali Linux (VM) | `192.168.x.x`    | Hydra, Nmap                                |
| **Victim (Target Machine)** | Windows 10 (VM) | `192.168.x.x`    | OpenSSH Server, Splunk Universal Forwarder |
| **SIEM (Blue Team)**        | Ubuntu Host     | `localhost:8000` | Splunk Enterprise                          |

---

## 🛠️ **Implementation Steps**

### **Phase 1 — Red Team Attack**

Performed a dictionary-based SSH brute force attack using Hydra:

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://<Windows_IP>
```

---

### **Phase 2 — Log Forwarding Setup**

Windows OpenSSH logs are written to **XmlWinEventLog:OpenSSH**, so the Universal Forwarder was configured accordingly.

**File:** `inputs.conf`
**Source:** `XmlWinEventLog:OpenSSH`
**Destination:** Splunk indexer (`port 9997`)

---

### **Phase 3 — Detection & Visualization (Blue Team)**

A custom Splunk dashboard was built using SPL queries and regular expressions.

**Challenges solved:**

* **Custom log parsing:** Windows OpenSSH logs differ from Linux `auth.log`.
* **Regex-based field extraction:** Normalizing user fields for both valid/invalid patterns.
* **Geolocation fallback:** Lab IPs do not resolve, so custom logic assigns a default region (Bangladesh).

---

## 📊 **Dashboard Panels & SPL Queries**

### **1. Total Failed SSH Login Attempts**

```spl
index=main sourcetype="XmlWinEventLog:OpenSSH" "Failed password"
| stats count AS "Failed Login Attempts"
```

---

### **2. Top Targeted Usernames**

Extracts usernames—handling both valid and invalid login attempts.

```spl
index=main sourcetype="XmlWinEventLog:OpenSSH" "Failed password"
| rex field=_raw "for (?:invalid user )?(?<user>\S+) from"
| top limit=10 user
```

---

### **3. Attacker IP Extraction + Geolocation Map**

```spl
index=main sourcetype="XmlWinEventLog:OpenSSH" "Failed password"
| rex field=_raw "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| iplocation src_ip
| eval Country=if(isnull(Country) OR Country="", "Bangladesh", Country)
| stats count by Country
| geom geo_countries featureIdField="Country"
```

---

## 📸 **Screenshots (Suggested Sections)**

1. Hydra attack execution in Kali Linux End-to-end

   <img width="1900" height="986" alt="Screenshot from 2025-12-10 01-18-50" src="https://github.com/user-attachments/assets/10621d0f-733a-45e8-a53c-17954075d4fc" />

   <img width="1900" height="986" alt="Screenshot from 2025-12-10 01-19-09" src="https://github.com/user-attachments/assets/831603b2-786d-4ed3-ac86-184f8137a920" />
3. Final Splunk dashboard displaying:

   * Real-time failed login counter
   * Username targeting analysis
   * Geolocation heat map

     <img width="1920" height="1080" alt="Screenshot from 2025-12-10 01-13-28" src="https://github.com/user-attachments/assets/69cffce5-11a2-4852-b8db-3d7944d7619b" />




---

## 🧠 **Key Takeaways**

* **SIEM Architecture:** 
 setup of indexer, forwarder, and data flow across heterogeneous systems.
* **Log Analysis:** Deep understanding of Windows OpenSSH event structure.
* **Regex Skill:** Extracting fields from unstructured logs using advanced patterns.
* **Threat Intelligence:** Recognizing characteristics of brute force attacks (high-volume 4625 events, repeated user probes).

---

## 🚀 **How to Use This Dashboard**

1. Download `dashboard_source.xml` from this repository.
2. Open your Splunk instance → **Dashboards** → *Create New Dashboard*.
3. Click **Edit Source**.
4. Paste the XML code.
5. Save & enjoy real-time monitoring.

---
