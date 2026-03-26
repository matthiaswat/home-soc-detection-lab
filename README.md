# Home SOC lab for detection engineering
A fully functional Security Operations Center (SOC) lab built with open‑source tools to simulate real‑world attacks, implement detection rules, and automate incident response. The lab demonstrates **end‑to‑end detection engineering** – from telemetry collection to MITRE ATT&CK‑aligned alerting and SOAR‑like automation.

## Overview
**Key capabilities:**
- Network Security Monitoring (Suricata IDS/IPS)
- Endpoint Detection & Response (Wazuh EDR)
- SIEM correlation (Splunk Enterprise)
- Data normalization pipeline (Logstash)
- Automated alerting via Discord webhooks
- 100% detection coverage of a 7‑phase kill chain

## Architecture
<img width="970" height="622" alt="image" src="https://github.com/user-attachments/assets/dc42c433-d41e-4879-a95a-2aa1fa61f402" />
 
The lab consists of 5 isolated VMs:
- Suricata (IDS/IPS) – monitors network traffic, generates EVE JSON logs.
- Wazuh Manager – collects endpoint telemetry from Windows target.
- Kali Linux – attacker machine (Metasploit, Nmap, Hydra).
- Windows 10 – victim endpoint (Wazuh agent installed).
- Splunk Enterprise – central SIEM, dashboards, alerting.

Data flow:
1. Collection – Suricata (network) & Wazuh (endpoint) generate logs.
2. Transport – Logstash ingests, normalizes, and forwards logs to Splunk HEC.
3. Storage – Logs indexed in Splunk (separate indexes for network/endpoint).
4. Detection – Custom SPL queries correlate events, mapped to MITRE ATT&CK.
5. Response – Splunk alerts trigger Discord webhook (SOAR simulation).

## Detection Rules & MITRE Coverage

All custom detection rules are available in [`rules/`](rules/). Each rule is mapped to a specific MITRE technique.

| Attack Phase | MITRE Technique | Detection Method | Rule File |
|--------------|-----------------|------------------|-----------|
| Reconnaissance | T1595.001 – Active Scanning | Suricata alert on TCP SYN flood | `local.rules` |
| Discovery | T1046 – Network Service Discovery | Suricata alert on service scan | `local.rules` |
| Initial Access | T1190 – Exploit Public-Facing App | Suricata SMB access alert | `local.rules` |
| Credential Access | T1110.001 – Password Guessing | Suricata threshold on failed SMB attempts | `local.rules` |
| Persistence | T1053.005 – Scheduled Task | Wazuh audit policy change (Event ID 4702) | Wazuh SCA rule |
| Privilege Escalation | T1078.002 – Domain Accounts | Wazuh detection of new local admin | Wazuh rule |
| Exfiltration | T1041 – Exfiltration Over C2 | Suricata alert on outbound HTTP to attacker | `local.rules` |

**Validation:** All 7 phases were simulated and successfully detected – see dashboards below.


## Dashboards & Visualizations
Splunk dashboards provide real‑time visibility:
**Cyber Kill Chain Overview** – bar chart of events per phase  
  <img width="1106" height="183" alt="image" src="https://github.com/user-attachments/assets/c3139ea1-3ff6-4a09-8116-0faa19b33c19" />
**MITRE ATT&CK Techniques** – distribution of detected techniques  
  <img width="1166" height="154" alt="image" src="https://github.com/user-attachments/assets/4dc28b0d-47a8-46ca-a5a1-a8691ed7658c" />
**Attack Timeline** – chronological progression of activities  
  <img width="1166" height="176" alt="image" src="https://github.com/user-attachments/assets/6b2e566c-eaea-4ef2-b654-263c81e81368" />


## SOAR Automation
When a high‑severity alert (e.g., data exfiltration) is triggered, Splunk sends a webhook to Discord, notifying the analyst in real-time.

<img width="779" height="64" alt="image" src="https://github.com/user-attachments/assets/229d27a7-ed24-4f53-873e-81f251b1e77d" />

This simulates a basic SOAR workflow, reducing MTTD and enabling immediate investigation.

## Lessons Learned & Future Improvements
**Detection Engineering:** Writing custom rules is powerful but requires continuous tuning to avoid false positives.
**Normalization:** Logstash is essential to unify different log formats – without it, correlation in Splunk would be painful.
**SOAR:** Even a simple webhook alert drastically speeds up response. Next step: integrate with VirusTotal API to enrich alerts.
