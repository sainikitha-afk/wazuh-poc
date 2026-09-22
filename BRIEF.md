# Wazuh SIEM/SOAR Proof of Concept

## Objective
Stand up a minimal but complete Wazuh SIEM deployment in an isolated AWS sandbox,
demonstrating log ingestion, detection, alerting, and automated response against a
deliberately vulnerable target app — following Wazuh's own documented PoC methodology
("Detecting a brute-force attack" and "Detecting an SQL injection attack").

## Acceptance Criteria
- Wazuh manager+indexer+dashboard running (Docker), all containers healthy
- DVWA deployed on Linux endpoint; Wazuh agent installed and shows Active
- Custom rule detects brute-force login attempts (tagged MITRE T1110)
- Custom rule detects SQL injection attempts on DVWA (tagged MITRE T1190)
- Correlation rule fires when the same source IP triggers both within a time window
  (multi-stage attack detection)
- All rules verified by actually running the attacks, not just config review
- Slack or email alert fires on high-severity rule match
- Active Response auto-blocks attacker IP after repeated failed logins
- Dashboard built: agent status, alerts by severity, brute-force/SQLi/multi-stage hits
- Issues + resolutions logged live during the build (see LOG.md)

## Expected Outcome
- I understand how a raw log line (like a failed login) turns into something Wazuh
  can actually search and alert on
- I can write/edit Wazuh's detection rules so it recognizes specific attacks
  (brute-force, SQL injection) and label them using MITRE ATT&CK
- I know how to connect Wazuh to Slack/email so I actually get notified when
  something serious happens
- I understand the difference between just detecting an attack vs. automatically
  doing something about it (SOAR) — like auto-blocking an attacker's IP
- I can build a dashboard that shows what matters at a glance — agent status,
  alert volume/severity, and my specific test attacks

## Subtasks
1. Provision AWS infrastructure (VPC/SG verification, EC2 instances)
2. Deploy Wazuh central components via Docker (manager, indexer, dashboard)
3. Deploy DVWA on Linux endpoint + install Wazuh agent
4. Verify agent enrollment (Active status in dashboard)
5. Write brute-force detection rule (MITRE T1110)
6. Write SQL injection detection rule (MITRE T1190)
7. Write correlation rule: same source IP triggers both = multi-stage attack alert
8. Configure Slack/email alerting for high-severity matches
9. Configure Active Response (auto-block attacker IP)
10. Build Wazuh dashboard (agent status, severity breakdown, custom detections)
11. Windows endpoint and remaining Wazuh PoC

