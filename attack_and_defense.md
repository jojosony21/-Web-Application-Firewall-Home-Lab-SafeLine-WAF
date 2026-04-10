# ⚔️ Attack Scenarios & WAF Defense — Detail Log

## Lab Environment
- **Attacker:** Kali Linux
- **Defender / WAF Host:** Zorin OS (SafeLine WAF + DVWA)
- **Target App:** DVWA at `https://dvwa.local`

---

## Scenario 1 — SQL Injection

### Attack Method
- Navigated to `https://dvwa.local` from Kali browser
- Logged into DVWA with default credentials (`admin` / `password`)
- Set security level to **Low**
- Went to **SQL Injection** module
- Entered payload: `1' OR '1'='1` and `admin'--`

### WAF Response
- SafeLine WAF intercepted the HTTP POST request
- Matched SQL Injection pattern via semantic analysis engine
- Returned **403 Block Page** to the browser
- Logged the event: source IP, payload, rule matched, timestamp

### MITRE ATT&CK Mapping
- **Tactic:** Initial Access
- **Technique:** T1190 — Exploit Public-Facing Application

---

## Scenario 2 — HTTP Flood

### Attack Method
- Used Apache Bench from Kali terminal:
```bash
ab -n 2000 -c 200 https://dvwa.local/
```
- Also tested with `siege`:
```bash
siege -c 100 -t 30S https://dvwa.local/
```

### WAF Response
- SafeLine HTTP Flood Defense triggered at configured threshold
- Source IP received rate-limit penalty
- After repeated violations, IP was **temporarily banned**
- WAF logs showed flood event with request rate and ban duration

### MITRE ATT&CK Mapping
- **Tactic:** Impact
- **Technique:** T1499 — Endpoint Denial of Service

---

## Scenario 3 — Custom IP Deny Rule

### Configuration
- Identified Kali Linux IP (e.g., `192.168.x.x`)
- Added custom deny rule in SafeLine WAF:
  - Match: Source IP = Kali IP
  - Action: Block

### Result
- All HTTP/HTTPS requests from Kali IP were immediately blocked
- Block page returned with no traffic reaching DVWA backend

### MITRE ATT&CK Mapping
- **Tactic:** Defense Evasion / Command and Control
- **Technique:** T1071 — Application Layer Protocol

---

## Scenario 4 — Access Rate Limiting

### Configuration
- Configured per-IP request rate limits in SafeLine WAF
- Threshold: e.g., 30 requests / 10 seconds
- Penalty: temporary IP ban

### Result
- Rapid scanning or scraping attempts from Kali were throttled
- Protected DVWA from brute-force enumeration even at low speeds

---

## Scenario 5 — Authentication Gateway

### Configuration
- Enabled SafeLine Auth Sign-In for the DVWA application route
- Set a WAF-level username/password

### Result
- Any browser accessing `https://dvwa.local` from Kali was intercepted
- SafeLine presented an **auth prompt before traffic reached DVWA**
- Only authenticated users could proceed to the application
- Defense-in-depth: WAF auth layer + DVWA login layer

---

## Summary Table

| Scenario | Attack Tool | WAF Feature Used | Result |
|---|---|---|---|
| SQL Injection | Browser (manual payload) | Semantic SQLi Detection | Blocked + Logged |
| HTTP Flood | `ab` / `siege` | HTTP Flood Defense | Rate-limited + Banned |
| Custom IP Block | Any (Kali IP) | Custom Deny Rules | Hard Blocked |
| Rate Limiting | Rapid requests | Access Limiting | Throttled |
| Auth Gateway | Browser | Auth Sign-In | Intercepted pre-app |
