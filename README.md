# 🔍 Log Analysis Investigation — TryHackMe: Logs Fundamentals

> **Type:** SOC Blue Team Project | **Platform:** TryHackMe | **Focus:** Log Analysis & Threat Detection

---

## 📌 Project Overview

This project documents a SOC-style log analysis investigation conducted within the TryHackMe *Logs Fundamentals* lab environment. The investigation involved parsing and correlating system, authentication, and web server logs to detect anomalous activity, identify Indicators of Compromise (IOCs), and reconstruct attacker behavior using a structured forensic methodology.

---

## 🎯 Objective

- Parse multi-source logs to surface suspicious patterns and authentication anomalies
- Identify and document Indicators of Compromise (IOCs) with supporting log evidence
- Reconstruct attacker activity using event correlation and timeline analysis
- Apply SOC-standard investigative techniques to a simulated incident

---

## 🛠️ Tools & Technologies

| Tool / Technology | Purpose |
|---|---|
| Linux CLI (`grep`, `awk`, `cut`, `less`) | Log filtering, field extraction, pattern matching |
| `auth.log` / `/var/log/secure` | Authentication failure analysis |
| Apache `access.log` | HTTP request analysis and anomaly detection |
| Windows Event Viewer | Windows authentication and system event review |
| TryHackMe Lab Environment | Controlled investigation environment |

---

## 🧠 Skills Demonstrated

- Log parsing and multi-source correlation
- Brute-force attack identification via authentication log analysis
- HTTP traffic analysis (status codes, request methods, endpoint targeting)
- IOC documentation and attacker TTP (Tactics, Techniques, Procedures) mapping
- Incident timeline reconstruction
- Basic digital forensics methodology

---

## 🔍 Methodology

### 1. Log Source Enumeration
Identified and catalogued all available log sources:
- **`/var/log/auth.log`** — SSH and PAM authentication events
- **`/var/log/apache2/access.log`** — Inbound HTTP/HTTPS requests
- **`/var/log/syslog`** — General system events

### 2. Targeted Filtering for Suspicious Activity

```bash
# Isolate POST requests to login endpoints
grep "POST /login" access.log

# Extract authentication failures with source IP
grep "Failed password" auth.log | awk '{print $11}' | sort | uniq -c | sort -rn

# Identify 401/403 HTTP responses indicating access control triggers
grep -E " 401 | 403 " access.log
```

### 3. Anomaly Identification
- Clustered failed login attempts from a single source IP within a compressed timeframe
- Identified repeated `POST /login` requests returning `HTTP 401` — consistent with credential stuffing or brute-force behavior
- Detected probing of sensitive endpoints (`/admin`, `/wp-login.php`) returning `HTTP 403`

### 4. Event Correlation & Timeline Construction
Cross-referenced timestamps across `auth.log` and `access.log` to establish a sequential attacker timeline, linking HTTP-layer activity to system-level authentication events.

---

## 📄 Sample Log Evidence

### Raw Log Entry

```
192.168.1.105 - - [14/Oct/2023:03:22:11 +0000] "POST /login HTTP/1.1" 401 512 "-" "python-requests/2.28.1"
192.168.1.105 - - [14/Oct/2023:03:22:14 +0000] "POST /login HTTP/1.1" 401 512 "-" "python-requests/2.28.1"
192.168.1.105 - - [14/Oct/2023:03:22:17 +0000] "POST /login HTTP/1.1" 401 512 "-" "python-requests/2.28.1"
```

### Analysis

| Field | Value | Significance |
|---|---|---|
| Source IP | `192.168.1.105` | Single origin — no IP rotation observed |
| HTTP Method | `POST` | Credential submission to login form |
| Endpoint | `/login` | Authentication endpoint targeted |
| Status Code | `401` | Server-side authentication failure |
| User-Agent | `python-requests/2.28.1` | Automated/scripted request — not a browser |
| Request Interval | ~3 seconds | High-frequency, consistent timing — tool-driven |

### Behavioral Interpretation

The combination of a non-browser User-Agent (`python-requests`), a consistent 3-second request interval, and unbroken `HTTP 401` responses across multiple entries is strongly indicative of **automated brute-force or credential stuffing activity**. A legitimate user would not generate this pattern. The absence of IP rotation suggests either a low-sophistication actor or a controlled internal test environment.

---

## 🚨 Key Findings

1. **Sustained Brute-Force Against Authentication Endpoint**
   Source IP `192.168.1.105` submitted **repeated POST requests to `/login`** over a 4-minute window, generating 47 consecutive `HTTP 401` responses. Request cadence (~3s intervals) and consistent User-Agent confirm automated tooling.

2. **Scripted Attack Tooling Identified**
   The `python-requests/2.28.1` User-Agent string is absent from legitimate user sessions in the log baseline, indicating a custom script or an automated attack framework (e.g., Hydra, Burp Intruder, or a custom credential sprayer).

3. **Sensitive Endpoint Enumeration**
   Prior to the brute-force sequence, the same source IP issued `GET` requests to `/admin`, `/phpmyadmin`, and `/wp-login.php`, each returning `HTTP 403`. This pattern is consistent with **pre-attack reconnaissance and endpoint discovery**.

4. **No Successful Authentication Observed**
   A full review of session tokens and `HTTP 200` responses following POST requests to `/login` confirmed **zero successful authentications** from the suspicious IP during the observed window.

5. **Off-Hours Activity**
   All malicious requests occurred between `03:18` and `03:26 UTC` — outside normal business hours — a common tactic used to reduce the likelihood of real-time detection.

---

## 🧪 Indicators of Compromise (IOCs)

| IOC Type | Value | Context |
|---|---|---|
| Suspicious IP | `192.168.1.105` | Source of all anomalous requests |
| Targeted Endpoint | `/login` | Subject of sustained POST flooding |
| Probed Endpoints | `/admin`, `/phpmyadmin`, `/wp-login.php` | Pre-attack enumeration |
| User-Agent | `python-requests/2.28.1` | Automated/non-browser client |
| HTTP Status Pattern | Consecutive `401` responses | Authentication failure sequence |
| Activity Window | `03:18–03:26 UTC` | Off-hours anomaly |

---

## 🧠 Attack Scenario Reconstruction

### Phase 1 — Reconnaissance (03:18–03:20 UTC)
The attacker initiated a series of `GET` requests targeting common administrative endpoints (`/admin`, `/phpmyadmin`, `/wp-login.php`). All returned `HTTP 403 Forbidden`, confirming endpoint existence while denying access. This phase aligns with **MITRE ATT&CK T1590 – Gather Victim Network Information** and indicates active surface mapping prior to exploitation.

### Phase 2 — Automated Credential Attack (03:22–03:26 UTC)
Following reconnaissance, the attacker pivoted to the `/login` endpoint and initiated a high-frequency `POST` sequence using a Python-based HTTP client. The consistent 3-second cadence and non-rotating source IP are characteristic of a **dictionary or credential-stuffing attack**, likely driven by a wordlist fed through a tool such as Hydra or a custom script.

Each request returned `HTTP 401`, confirming that no submitted credential pair was valid against the application's authentication mechanism during this window.

### Phase 3 — No Lateral Movement Detected
Log review across `auth.log` and application logs revealed no successful session establishment, no privilege escalation events, and no lateral movement indicators following the attack window. The attacker appears to have terminated activity after the brute-force sequence failed.

### Assessment
The attack followed a classic **reconnaissance → exploitation attempt** pattern. The use of scripted tooling, off-hours timing, and targeted endpoint selection indicates a deliberate, non-opportunistic actor. The absence of successful authentication suggests existing password policy or account lockout controls were effective deterrents.

---

## 💡 Personal Insights

This investigation reinforced that log analysis is not passive — it requires active pattern recognition and cross-source correlation to surface meaningful signals from noisy data. The most valuable skill developed was distinguishing **behavioral anomalies** (e.g., request cadence, User-Agent strings) from surface-level indicators (e.g., simple failed logins), which are easy to dismiss in isolation but damning in aggregate.

The exercise also highlighted a critical real-world gap: without alerting rules tuned to detect high-frequency `401` responses from a single IP, this attack could persist undetected for extended periods. This underscores the value of SIEM rules and threshold-based alerting in production environments.

---

## 🔐 Real-World Relevance

The techniques applied in this investigation directly mirror day-to-day tasks performed by:

- **SOC Analysts (L1/L2)** — Log triage, alert validation, IOC identification
- **Incident Responders** — Timeline reconstruction, attacker behavior analysis
- **Blue Team Engineers** — Detection rule development based on observed TTPs

---

## 🚀 Future Improvements

- Ingest logs into **Splunk or Elastic SIEM** and build detection rules for the observed brute-force pattern
- Develop a **Python log parser** to automate IOC extraction and frequency analysis
- Enrich suspicious IPs against threat intelligence feeds (e.g., AbuseIPDB, VirusTotal)
- Map all findings to the **MITRE ATT&CK framework** for structured reporting

---

## 🔗 Supporting Documentation

For extended log excerpts and raw analysis notes, see: [`logs-analysis.md`](logs-analysis.md)

---

## ✅ Conclusion

This investigation demonstrates applied SOC competencies in log analysis, behavioral threat detection, and incident reconstruction. By correlating HTTP-layer and system-level log data, a coherent attacker timeline was established — from initial reconnaissance through sustained credential attack — using the same analytical workflow employed in real-world blue team operations.
