# Wazuh SOC Home Lab — SSH Brute Force Detection

A self-hosted SIEM lab built with Wazuh and Docker, deployed on Kali Linux, used to detect a simulated SSH brute-force attack from raw log to classified alert.

## Why I built this

I completed the Google Cybersecurity Certificate and wanted to go past the guided labs — build something from scratch, break it, fix it, and prove I can actually operate a SIEM the way a SOC analyst would day to day.

## Architecture

- Host OS: Kali Linux (MSI laptop)
- SIEM platform: Wazuh 4.9.0, deployed via Docker Compose (single-node)
- Components: Wazuh Indexer (OpenSearch), Wazuh Manager, Wazuh Dashboard
- Monitored endpoint: Kali Linux agent, connecting to the manager over 172.17.0.1
[Kali Agent] --> [Wazuh Manager] --> [Wazuh Indexer] --> [Wazuh Dashboard]

## Setup process

1. Installed Docker and Docker Compose on Kali
1. Deployed the Wazuh single-node stack via docker-compose
1. Generated indexer SSL certificates using Wazuh’s cert generation tool
1. Resolved an OpenSearch plugin failure caused by a missing/corrupted cert directory (see Troubleshooting below)
1. Logged into the Wazuh dashboard and deployed a Wazuh agent onto the same Kali host
1. Confirmed the agent connected and reported “active” status

## Attack simulation

To generate a real detection event rather than relying on synthetic data:

1. Enabled and started the SSH service (`sshd`) on Kali
1. Attempted to SSH into 127.0.0.1 using intentionally incorrect passwords, repeated several times in a row
1. Confirmed the attempts failed as expected (`Permission denied`)

## Detection results

Wazuh caught the failed logins in real time and correctly classified the behavior:

- Rule triggered: 5760 — sshd: authentication failed and 2502 — syslog: User missed the password more than one time
- MITRE ATT&CK mapping: T1110 — Credential Access / Password Guessing
- Dashboard classification: automatically tagged the activity under “Password Guessing” and “Brute Force” in the MITRE ATT&CK view
- Raw log evidence captured:
  
 
  sshd-session[376330]: PAM 2 more authentication failures;
  logname= uid=0 euid=0 tty=ssh ruser= rhost=127.0.0.1 user=abdulcyber
  
This confirms the full detection pipeline works end to end: attack → log generation → agent forwarding → manager rule matching → indexed alert → dashboard visualization with correct MITRE technique classification.

## Troubleshooting log (real issues I hit and fixed)

- Indexer and dashboard stuck in restart loop: root cause was a missing SSL certificate directory. Diagnosed via docker logs, which pointed to OpenSearchSecuritySSLPlugin failing to load a cert file that didn’t exist. Fixed by wiping the cert folder and re-running Wazuh’s certificate generator script cleanly.
- SSH “connection refused”: sshd was installed but disabled by default on Kali. Fixed with systemctl start ssh and systemctl enable ssh.
- Dashboard login failure with default credentials: Wazuh 4.9 doesn’t use the old SecretPassword default in all cases; resolved by re-entering credentials manually rather than relying on copy-paste.

## Screenshots

*(insert screenshots here: dashboard overview, Endpoints page showing active agent, Threat Hunting MITRE ATT&CK chart, raw log document detail)*

## What I’d improve next

- Add a second monitored endpoint (Windows VM) to show cross-platform log correlation
- Write a custom Wazuh rule for a specific threat pattern instead of relying only on built-in rules
- Automate the SSH brute-force simulation with a small Python script instead of manual attempts

## Skills demonstrated

Docker & container troubleshooting, Linux system administration, SIEM deployment and configuration, log analysis, MITRE ATT&CK framework, SSH/PAM authentication logs, root cause diagnosis from stack traces.
