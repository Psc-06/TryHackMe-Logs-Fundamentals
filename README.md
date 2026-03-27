# 🔍 Log Analysis Investigation – TryHackMe Logs Fundamentals

## 📌 Project Description

This project demonstrates a SOC-style log analysis investigation conducted using the TryHackMe *Logs Fundamentals* lab. The focus is on analyzing system and web server logs to detect suspicious activities, identify potential threats, and understand attacker behavior.

---

## 📖 Overview

Logs are a critical component in cybersecurity, acting as a primary source of evidence during incident investigations. In this project, multiple log sources were analyzed to uncover anomalies and reconstruct possible attack scenarios.

---

## 🎯 Objective

* Analyze logs to detect suspicious behavior
* Identify Indicators of Compromise (IOCs)
* Understand attacker patterns through log data

---

## 🛠️ Tools & Technologies Used

* Linux CLI (`grep`, `cat`, `less`)
* Windows Event Viewer
* Web Server Logs (Apache/Nginx)
* TryHackMe Lab Environment

---

## 🧠 Skills Demonstrated

* Log Analysis & Interpretation
* Threat Detection
* Incident Investigation
* Pattern Recognition
* Basic Digital Forensics

---

## 🔍 Methodology

### 1. Log Exploration

Reviewed different log sources:

* System logs
* Authentication logs
* Web server logs

---

### 2. Filtering Suspicious Activity

Example:

```bash
grep "POST" access.log
```

* Used filtering to identify suspicious HTTP requests
* Focused on keywords like `login`, `error`, `failed`, `admin`

---

### 3. Identifying Anomalies

* Multiple failed login attempts
* Unusual IP activity
* Access to restricted resources
* Abnormal traffic patterns

---

### 4. Event Correlation

* Correlated logs across sources
* Reconstructed possible attacker actions

---

## 🚨 Key Findings

* Suspicious IPs interacting repeatedly with the system
* Evidence of brute-force login attempts
* Unusual web requests indicating probing or enumeration
* Potential unauthorized access attempts

---

## 🧪 Indicators of Compromise (IOCs)

* Repeated failed authentication attempts
* Suspicious IP addresses
* Abnormal request patterns
* Access to sensitive endpoints

---

## 🧠 Personal Insights

During this investigation, I realized how critical log monitoring is for early threat detection. Even small anomalies, such as repeated failed logins, can indicate larger attack attempts like brute-force attacks. This highlights the importance of continuous monitoring in real-world SOC environments.

---

## 🔐 Real-World Relevance

This project reflects tasks performed by:

* SOC Analysts
* Incident Responders
* Blue Team Engineers

Log analysis is essential for:

* Threat detection
* Security monitoring
* Incident response

---


## 🚀 Future Improvements

* Integrate SIEM tools like Splunk or ELK Stack
* Automate log parsing using scripts
* Perform deeper threat hunting

---

## ✅ Conclusion

This project demonstrates foundational SOC-level skills in log analysis, threat detection, and incident investigation. It highlights the importance of logs in understanding and responding to cybersecurity incidents.

---
