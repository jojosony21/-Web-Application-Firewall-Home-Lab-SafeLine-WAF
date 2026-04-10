# 🔥 Web Application Firewall Home Lab — SafeLine WAF

> A hands-on cybersecurity home lab demonstrating **Web Application Firewall (WAF) deployment, attack simulation, and defense** using **SafeLine WAF** on **Zorin OS**, with attack traffic generated from **Kali Linux**.

---

## 📌 Overview

This project builds a complete, realistic SOC/Blue Team home lab environment that demonstrates:

- Hosting a deliberately vulnerable web application (**DVWA**) behind a WAF
- Simulating real-world web attacks from Kali Linux (SQL Injection, HTTP Flood)
- Observing **SafeLine WAF** detect, block, and log each attack in real time
- Configuring advanced WAF defenses: rate limiting, access control, custom deny rules, and authentication gateway

---

## 🧱 Lab Architecture

```
┌─────────────────────┐          HTTP/HTTPS Attacks           ┌──────────────────────────────┐
│   Kali Linux        │  ══════════════════════════════════>  │        Zorin OS VM           │
│   (Attacker)        │   SQLi / HTTP Flood / Custom Rules    │  ┌────────────────────────┐  │
│                     │                                        │  │   SafeLine WAF         │  │
│  Tools used:        │                                        │  │   (Nginx Reverse Proxy)│  │
│  • Browser          │                                        │  │   Port 443 (HTTPS)     │  │
│  • sqlmap / manual  │                                        │  └──────────┬─────────────┘  │
│  • ab / siege       │                                        │             │ Inspects &      │
│  • curl             │                                        │             │ Filters Traffic │
└─────────────────────┘                                        │             ▼                 │
                                                               │  ┌────────────────────────┐  │
                                                               │  │   DVWA (Apache:8080)   │  │
                                                               │  │   MySQL Backend        │  │
                                                               │  └────────────────────────┘  │
                                                               └──────────────────────────────┘
```

**DNS Resolution:** `/etc/hosts` entry `dvwa.local` → Zorin OS IP  
**SSL:** Self-signed certificate (OpenSSL, RSA 2048)  
**WAF Admin Panel:** `https://<ZorinOS-IP>:9443`

---

## 🧰 Tech Stack

| Component | Tool / Version |
|---|---|
| Host OS (Defender) | Zorin OS |
| Attacker Machine | Kali Linux |
| Vulnerable App | DVWA (Damn Vulnerable Web App) |
| Web Server | Apache2 (port 8080) |
| Database | MySQL (LAMP Stack) |
| WAF | SafeLine WAF (Nginx-based reverse proxy) |
| SSL | Self-Signed Certificate (OpenSSL) |
| DNS | `/etc/hosts` + optional BIND9 |
| Virtualization | VirtualBox (Bridged Networking) |

---

## 📁 Repository Structure

```
safeline-waf-homelab/
│
├── README.md                        ← This file
│
├── configs/
│   ├── apache_ports.conf            ← Apache port 8080 config snippet
│   ├── dvwa_virtualhost.conf        ← Apache VirtualHost config
│   └── bind_zone_dvwa.local         ← Optional BIND9 zone file
│
├── docs/
│   ├── setup_guide.md               ← Full step-by-step setup guide
│   └── attack_and_defense.md        ← Attack scenarios & WAF responses
│
└── screenshots/                     ← POC evidence (see screenshot guide below)
    ├── 01_lab_architecture.png
    ├── 02_dvwa_running.png
    ├── 03_safeline_dashboard.png
    ├── 04_sqli_attempt_kali.png
    ├── 05_sqli_blocked_waf.png
    ├── 06_waf_sqli_log.png
    ├── 07_http_flood_test.png
    ├── 08_http_flood_blocked.png
    ├── 09_custom_deny_rule.png
    ├── 10_kali_ip_blocked.png
    ├── 11_access_limit_config.png
    └── 12_auth_gateway.png
```

---

## ⚙️ Lab Setup Summary

### Environment
- Both VMs run on VirtualBox with **Bridged Networking** — both appear on the same LAN
- Zorin OS hosts DVWA (Apache on port 8080) + SafeLine WAF (port 9443 admin, port 443 public)
- Kali Linux is the attacker — all attacks originate from here

### Key Configuration Steps
1. LAMP stack installed on Zorin OS (Apache2, PHP, MySQL)
2. DVWA cloned and configured with a dedicated MySQL database
3. Apache reconfigured to listen on **port 8080** (port 80 freed for SafeLine)
4. Self-signed SSL certificate generated with OpenSSL
5. SafeLine WAF installed via automatic script, configured as reverse proxy in front of DVWA
6. DNS resolved via `/etc/hosts`: `dvwa.local` → Zorin OS IP

---

## 🔴 Attack Scenarios & WAF Responses

### 1. SQL Injection (SQLi)

**Attack:** From Kali Linux browser, navigated to `http://dvwa.local/` (redirected to HTTPS by WAF), set DVWA security to **Low**, and submitted classic SQL injection payloads in the SQL Injection module.

**WAF Response:** SafeLine WAF detected and **blocked** the malicious request, returning a block page. The attack was logged in the WAF dashboard with source IP, payload, and timestamp.

**MITRE ATT&CK:** T1190 — Exploit Public-Facing Application

---

### 2. HTTP Flood Defense

**Attack:** Sent a high volume of HTTP requests from Kali Linux using load-testing tools (`ab`, `siege`, `curl` loops) to simulate a Layer 7 DoS/flood attack against `dvwa.local`.

**WAF Response:** SafeLine WAF's HTTP Flood Defense triggered after the configured requests-per-second threshold was crossed. The source IP was **rate-limited and temporarily banned**.

**MITRE ATT&CK:** T1499 — Endpoint Denial of Service

---

### 3. Custom Deny Rules (IP Blocking)

**Attack:** Attempted to access DVWA from the Kali Linux IP.

**WAF Response:** A custom deny rule was configured in SafeLine WAF to **block all traffic from the Kali IP address**. All requests returned an immediate blocked response.

**MITRE ATT&CK:** T1071 — Application Layer Protocol

---

### 4. Access Limiting / Rate Control

**Configuration:** Set strict per-IP request rate limits in SafeLine WAF to restrict excessive access patterns even below flood thresholds.

**Result:** Requests exceeding the threshold were queued or dropped, protecting backend DVWA from scraping or brute-force enumeration.

---

### 5. Authentication Gateway

**Configuration:** Enabled SafeLine's Auth Sign-In gateway on the DVWA application route.

**Result:** Any unauthenticated request from Kali to `dvwa.local` was intercepted by the WAF's auth prompt before traffic reached the backend — an additional layer of defense-in-depth.

---

## 📸 Screenshot Proof Guide

SafeLine dashboard
![SafeLine dashboard](screenshots/SafeLine_dash.png)

SQLi block page on Kali
![SQLi block page on Kali](screenshots/sqli_block_kali.png)
WAF SQLi log entry
![WAF SQLi log entry](screenshots/Sqli_log.png)
Custom deny rule config
![Custom deny rule config](screenshots/custom_rule.png)
HTTP flood blocked in logs
![HTTP flood blocked in logs](/screenshots/Http_flood.png)
Auth gateway intercepting Kali before DVWA loads
![Auth gateway intercepting Kali before DVWA loads](screenshots/auth_gateway.png)

---

## 🛡️ Key Takeaways

- A WAF operates as a **reverse proxy** — all traffic passes through it before reaching the application
- SafeLine uses **semantic analysis** (not just regex) to detect injection attacks, reducing false positives
- Defense-in-depth is demonstrated: SSL termination + SQLi blocking + rate limiting + IP blocking + auth gateway, all in one lab
- Real WAF logs provide actionable forensic data: attacker IP, timestamp, attack type, payload, and action taken

---

## ⚠️ Disclaimer

This lab was built entirely on personal virtual machines for educational and portfolio purposes. All attacks were performed against intentionally vulnerable applications owned by the author. No external systems were targeted.

---

## 📚 Reference

- Original lab guide by **Royden Rebello (The Social Dork)**
- SafeLine WAF: [waf.chaitin.com](https://waf.chaitin.com)
- DVWA: [github.com/digininja/DVWA](https://github.com/digininja/DVWA)

---

## 👤 Author

Built as a portfolio project for **SOC Analyst / Blue Team / Application Security** roles.

**Skills demonstrated:** WAF deployment · Reverse proxy configuration · SQL injection defense · HTTP flood mitigation · Custom firewall rules · SSL/TLS · DVWA · Linux server administration · VirtualBox networking · Incident logging & analysis
