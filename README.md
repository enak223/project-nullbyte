# ⚡ Project NullByte
### Automated CVE Detection, Prioritization & Remediation Reporting

> *"Every unpatched CVE is a null byte waiting to be weaponized. NullByte finds them first."*

[![Status](https://img.shields.io/badge/Status-Active%20Development-brightgreen?style=flat-square)](https://github.com/enak223)
[![Stack](https://img.shields.io/badge/Stack-Nmap%20%7C%20Wazuh%20%7C%20n8n%20%7C%20AI-blue?style=flat-square)](https://github.com/enak223)
[![Schedule](https://img.shields.io/badge/Schedule-Weekly%20Automated-orange?style=flat-square)](https://github.com/enak223)
[![License](https://img.shields.io/badge/License-MIT-lightgrey?style=flat-square)](LICENSE)

---

## 📌 Overview

**NullByte** is a fully automated vulnerability management pipeline built for homelabs and small environments. It combines active network scanning, SIEM telemetry, CVE database enrichment, AI-powered prioritization, and auto-generated remediation reports — all orchestrated on a weekly schedule with zero manual intervention.

The pipeline answers three questions automatically:
- **What is exposed?** — Nmap scans discover open ports, services, and software versions across the network.
- **What does it mean?** — CVE databases (NVD, OSV, VulnDB) are queried for matching vulnerabilities enriched with CVSS scores.
- **What do we do?** — An AI agent prioritizes findings by severity and generates a structured remediation report delivered to your inbox or dashboard.

> **How NullByte fits the bigger picture:** NullByte finds the open door. [Project GhostNet](https://github.com/enak223/project-ghostnet) catches someone walking through it. Together they form a complete detection loop — *find weaknesses → watch for exploitation.*

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        NULLBYTE PIPELINE                            │
│                                                                     │
│  ┌──────────────┐      ┌──────────────┐     ┌──────────────────┐    │
│  │  TRIGGER     │────▶│   DISCOVER   │────▶│    ENRICH        │    │
│  │              │      │              │     │                  │    │
│  │ n8n Cron     │      │ Nmap Scan    │     │ NVD API Query    │    │
│  │ (Weekly)     │      │ OS/Service   │     │ OSV.dev Query    │    │
│  │              │      │ Detection    │     │ CVSS Scoring     │    │
│  └──────────────┘      └──────────────┘     └──────────────────┘    │
│                                                     │               │
│  ┌──────────────┐      ┌──────────────┐     ┌───────▼──────────┐    │
│  │  DELIVER     │◀────│  REPORT      │◀────│    PRIORITIZE    │    │
│  │              │      │              │     │                  │    │
│  │ Email/Slack  │      │ Auto-Gen PDF │     │ AI CVE Hunter    │    │
│  │ Dashboard    │      │ Remediation  │     │ Claude/GPT Agent │    │
│  │ Wazuh Alert  │      │ Action Plan  │     │ Risk Scoring     │    │
│  └──────────────┘      └──────────────┘     └──────────────────┘    │
│                                                                     │
│                    ┌──────────────────────┐                         │
│                    │   WAZUH SIEM         │                         │
│                    │ Continuous Monitoring│                         │
│                    │  Auditd / FIM / IDS  │                         │
│                    └──────────────────────┘                         │
└─────────────────────────────────────────────────────────────────────┘
```

**Data Flow:**
```
n8n Cron Job
    └──▶ Execute Nmap Scan (via bash node or API)
             └──▶ Parse XML output → extract CPE/service/version strings
                      └──▶ Query NVD API v2.0 for matching CVEs
                               └──▶ Query OSV.dev for additional coverage
                                        └──▶ AI Agent: prioritize + contextualize
                                                 └──▶ Generate remediation report
                                                          └──▶ Push alert to Wazuh
                                                                   └──▶ Deliver report
```

---

## 🧰 Tech Stack

| Component | Tool | Role |
|-----------|------|------|
| **Orchestration** | n8n (self-hosted) | Workflow automation, scheduling, API chaining |
| **Network Scanner** | Nmap 7.x | Port/service/OS/version discovery |
| **SIEM** | Wazuh 4.x | Continuous monitoring, alert correlation, FIM |
| **CVE Database** | NVD API v2.0 | Primary CVE/CVSS data source |
| **CVE Database** | OSV.dev API | Additional open-source vuln coverage |
| **CVE Database** | CIRCL CVE Search | Fallback/cross-reference |
| **AI Agent** | Claude API / GPT-4o | CVE triage, risk narrative, remediation guidance |
| **API Testing** | Postman | Manual workflow testing and API validation |
| **Reporting** | n8n + Markdown/HTML | Auto-generated weekly remediation reports |
| **Delivery** | Email (SMTP) / Slack | Report distribution |
| **Host OS** | Ubuntu 22.04 | Primary pipeline host |
| **Virtualization** | VMware Workstation | Homelab multi-VM environment |

---

## ✨ Features

### 🔍 Automated Discovery
- Weekly Nmap scans across all defined CIDR ranges
- Service version detection (`-sV`) and OS fingerprinting (`-O`)
- CPE (Common Platform Enumeration) string extraction for precise CVE matching
- Scan results exported as structured XML, parsed into JSON for pipeline processing

### 🗄️ CVE Database Enrichment
- Queries **NVD API v2.0** using CPE strings and keyword matching
- Cross-references **OSV.dev** for open-source package vulnerabilities
- Retrieves full CVE metadata: CVSS v3.1 base score, vector string, CWE, affected versions, references
- Deduplicates CVEs across multiple data sources

### 🤖 AI CVE Hunter Agent
- Claude/GPT agent analyzes enriched CVE data in context
- Scores each finding using a composite risk model: CVSS score + asset criticality + exploitability
- Generates natural-language summaries for each vulnerability
- Produces specific, actionable remediation steps (not generic patch advice)
- Flags **CISA KEV** (Known Exploited Vulnerabilities) catalog matches for immediate escalation

### 📊 Automated Remediation Reports
- Weekly HTML/Markdown report generated automatically
- Sections: Executive Summary, Critical Findings, Full Vulnerability Table, Remediation Action Plan
- Color-coded severity tiers: Critical 🔴 High 🟠 Medium 🟡 Low 🔵
- Remediation priority queue with estimated effort ratings
- Trend comparison: new CVEs vs. previous week

### 📅 Scheduled Pipeline
- n8n Cron node triggers every Sunday at 02:00 local time (configurable)
- Full scan-to-report cycle completes in ~15–30 minutes depending on scope
- Wazuh alert fired on completion with report summary

### 🔔 Alerting & Delivery
- Report delivered via SMTP email and/or Slack webhook
- Critical findings trigger immediate Wazuh alerts (severity 12+)
- Optional: push findings to a local dashboard (Grafana / n8n built-in)

---

## 📁 Project Structure

```
project-nullbyte/
├── README.md
├── .env.example                    # Environment variable template
├── docker-compose.yml              # Optional: containerized deployment
│
├── nmap/
│   ├── scan_profiles/
│   │   ├── full_network.xml        # Nmap scan profile — full network
│   │   ├── quick_top100.xml        # Nmap scan profile — top 100 ports
│   │   └── vuln_scripts.xml        # Nmap NSE vuln script profile
│   ├── run_scan.sh                 # Scan execution wrapper script
│   └── parse_nmap_xml.py           # XML → JSON parser for n8n input
│
├── n8n/
│   ├── workflows/
│   │   ├── nullbyte_main.json      # Main weekly pipeline workflow
│   │   ├── cve_hunter_agent.json   # AI CVE Hunter sub-workflow
│   │   └── report_delivery.json    # Report generation & delivery workflow
│   └── credentials/
│       └── credentials.example.json
│
├── cve_enrichment/
│   ├── nvd_query.py                # NVD API v2.0 query module
│   ├── osv_query.py                # OSV.dev API query module
│   ├── cve_dedup.py                # Cross-source deduplication logic
│   └── cvss_parser.py              # CVSS vector string parser/scorer
│
├── ai_agent/
│   ├── prompts/
│   │   ├── cve_triage_system.txt   # System prompt: CVE Hunter agent
│   │   ├── remediation_prompt.txt  # Prompt: remediation plan generation
│   │   └── executive_summary.txt   # Prompt: executive summary generation
│   └── agent_runner.py             # Agent invocation wrapper
│
├── reporting/
│   ├── templates/
│   │   ├── report_template.html    # Weekly report HTML template
│   │   └── report_template.md      # Markdown version
│   ├── generate_report.py          # Report assembly script
│   └── sample_reports/
│       └── sample_2025-01-12.html  # Example output report
│
├── wazuh/
│   ├── custom_rules/
│   │   ├── nullbyte_rules.xml      # Custom Wazuh rules for NullByte events
│   │   └── vuln_alert_rules.xml    # Rules for critical CVE alerts
│   ├── decoders/
│   │   └── nullbyte_decoder.xml    # Custom log decoder for scan output
│   └── active_response/
│       └── nullbyte_notify.sh      # Active response: notify on critical CVE
│
├── auditd/
│   ├── audit.rules                 # Custom auditd rules for monitored hosts
│   └── audit_rules_notes.md        # Rule documentation and rationale
│
├── postman/
│   ├── NullByte_API_Tests.postman_collection.json
│   └── NullByte_Environments.postman_environment.json
│
└── docs/
    ├── architecture.md
    ├── setup_guide.md
    └── api_reference.md
```

---

## ⚙️ Setup & Installation

### Prerequisites

```bash
# Core requirements
- Ubuntu 22.04 LTS (pipeline host)
- n8n self-hosted (v1.x+)
- Wazuh Server 4.x (manager)
- Nmap 7.x
- Python 3.10+
- API keys: NVD (free), Anthropic or OpenAI
```

### 1. Clone the Repository

```bash
git clone https://github.com/enak223/project-nullbyte.git
cd project-nullbyte
```

### 2. Configure Environment Variables

```bash
cp .env.example .env
nano .env
```

```dotenv
# .env
NVD_API_KEY=your_nvd_api_key_here
ANTHROPIC_API_KEY=your_anthropic_key_here
OPENAI_API_KEY=your_openai_key_here          # optional fallback

# Scan targets (CIDR notation, comma-separated)
SCAN_TARGETS=192.168.x.0/24

# Wazuh
WAZUH_HOST=192.168.x.20
WAZUH_API_USER=wazuh-wui
WAZUH_API_PASS=your_wazuh_api_password

# Delivery
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your@email.com
SMTP_PASS=your_app_password
REPORT_RECIPIENT=your@email.com

# Optional Slack
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/...
```

### 3. Install Python Dependencies

```bash
pip install -r requirements.txt
# Key packages: requests, python-nmap, jinja2, anthropic, openai
```

### 4. Set Up Nmap

```bash
sudo apt update && sudo apt install -y nmap

# Verify version
nmap --version

# Test scan (confirm scan access to your own subnet)
sudo nmap -sV -O --script=vuln 192.168.x.0/24 -oX /tmp/test_scan.xml
```

### 5. Import n8n Workflows

```bash
# Start n8n if not already running
n8n start

# Import via UI: Settings → Import Workflow
# Import: n8n/workflows/nullbyte_main.json
# Import: n8n/workflows/cve_hunter_agent.json
# Import: n8n/workflows/report_delivery.json
```

**Or via n8n CLI:**
```bash
n8n import:workflow --input=n8n/workflows/nullbyte_main.json
```

### 6. Configure Wazuh Custom Rules

```bash
# Copy rules to Wazuh manager
sudo cp wazuh/custom_rules/nullbyte_rules.xml \
    /var/ossec/etc/rules/nullbyte_rules.xml

sudo cp wazuh/custom_rules/vuln_alert_rules.xml \
    /var/ossec/etc/rules/vuln_alert_rules.xml

# Copy decoder
sudo cp wazuh/decoders/nullbyte_decoder.xml \
    /var/ossec/etc/decoders/nullbyte_decoder.xml

# Restart Wazuh manager
sudo systemctl restart wazuh-manager

# Verify rules loaded
sudo /var/ossec/bin/ossec-logtest
```

### 7. Deploy Auditd Rules (Monitored Hosts)

```bash
# Copy to target hosts (run on each monitored Linux host)
sudo cp auditd/audit.rules /etc/audit/rules.d/nullbyte.rules
sudo augenrules --load
sudo systemctl restart auditd

# Verify rules active
sudo auditctl -l | grep nullbyte
```

### 8. Schedule the Pipeline

In n8n, open `nullbyte_main.json` and configure the **Cron** node:

```
Schedule: 0 2 * * 0     (Every Sunday at 02:00 AM)
Timezone: America/New_York
```

Activate the workflow toggle to enable.

---

## 🔧 Custom Wazuh Rules

```xml
<!-- wazuh/custom_rules/nullbyte_rules.xml -->
<group name="nullbyte,vulnerability_scan">

  <!-- NullByte pipeline started -->
  <rule id="100500" level="3">
    <decoded_as>nullbyte</decoded_as>
    <field name="event_type">scan_started</field>
    <description>NullByte: Weekly vulnerability scan initiated</description>
    <group>nullbyte_info</group>
  </rule>

  <!-- Scan completed -->
  <rule id="100501" level="5">
    <decoded_as>nullbyte</decoded_as>
    <field name="event_type">scan_complete</field>
    <description>NullByte: Vulnerability scan completed - $(host_count) hosts scanned</description>
    <group>nullbyte_info</group>
  </rule>

  <!-- CVE found - Medium severity -->
  <rule id="100510" level="7">
    <decoded_as>nullbyte</decoded_as>
    <field name="severity">MEDIUM</field>
    <description>NullByte: Medium severity CVE detected - $(cve_id) on $(target_host)</description>
    <group>nullbyte_vuln,vulnerability_medium</group>
  </rule>

  <!-- CVE found - High severity -->
  <rule id="100511" level="10">
    <decoded_as>nullbyte</decoded_as>
    <field name="severity">HIGH</field>
    <description>NullByte: HIGH severity CVE detected - $(cve_id) on $(target_host) (CVSS: $(cvss_score))</description>
    <group>nullbyte_vuln,vulnerability_high</group>
  </rule>

  <!-- CVE found - Critical severity -->
  <rule id="100512" level="13">
    <decoded_as>nullbyte</decoded_as>
    <field name="severity">CRITICAL</field>
    <description>NullByte: CRITICAL CVE detected - $(cve_id) on $(target_host) (CVSS: $(cvss_score))</description>
    <group>nullbyte_vuln,vulnerability_critical</group>
    <options>alert_by_email</options>
  </rule>

  <!-- CISA KEV match — escalate immediately -->
  <rule id="100515" level="15">
    <decoded_as>nullbyte</decoded_as>
    <field name="kev_match">true</field>
    <description>NullByte: CISA KEV MATCH - $(cve_id) is actively exploited in the wild on $(target_host)</description>
    <group>nullbyte_vuln,kev_match,vulnerability_critical</group>
    <options>alert_by_email</options>
  </rule>

  <!-- Report generated successfully -->
  <rule id="100520" level="3">
    <decoded_as>nullbyte</decoded_as>
    <field name="event_type">report_generated</field>
    <description>NullByte: Weekly remediation report generated and delivered</description>
    <group>nullbyte_info</group>
  </rule>

  <!-- Pipeline error -->
  <rule id="100599" level="8">
    <decoded_as>nullbyte</decoded_as>
    <field name="event_type">pipeline_error</field>
    <description>NullByte: Pipeline error encountered - $(error_message)</description>
    <group>nullbyte_error</group>
  </rule>

</group>
```

---

## 📐 Custom Auditd Rules

These rules are deployed on all Linux hosts monitored by NullByte/Wazuh to capture suspicious activity that could indicate active exploitation of a detected CVE.

```bash
# auditd/audit.rules
# ─────────────────────────────────────────────────
# NullByte — Custom Auditd Rules
# Deployed on: Ubuntu Web Server, Ubuntu AI/Wazuh
# Purpose: Detect post-exploitation patterns matching
#          actively hunted CVE categories
# ─────────────────────────────────────────────────

# Delete all existing rules first (clean slate)
-D

# Buffer size
-b 8192

# Failure mode: log failures silently
-f 1

# ── PRIVILEGE ESCALATION ────────────────────────────
-w /usr/bin/sudo -p x -k priv_escalation
-w /bin/su -p x -k priv_escalation
-w /etc/sudoers -p wa -k sudoers_change
-w /etc/sudoers.d/ -p wa -k sudoers_change

# SUID/SGID execution (common local priv esc vector)
-a always,exit -F arch=b64 -S execve -F euid=0 -F auid>=1000 -k suid_exec
-a always,exit -F arch=b32 -S execve -F euid=0 -F auid>=1000 -k suid_exec

# ── CREDENTIAL ACCESS ───────────────────────────────
-w /etc/passwd -p wa -k credential_access
-w /etc/shadow -p wa -k credential_access
-w /etc/group -p wa -k credential_access
-w /etc/gshadow -p wa -k credential_access
-w /etc/security/opasswd -p wa -k credential_access

# SSH key access
-w /root/.ssh -p rwa -k ssh_key_access
-a always,exit -F dir=/home -F name=".ssh" -F perms=rwa -k ssh_key_access

# ── PERSISTENCE MECHANISMS ──────────────────────────
-w /etc/cron.d/ -p wa -k persistence_cron
-w /etc/crontab -p wa -k persistence_cron
-w /var/spool/cron/ -p wa -k persistence_cron
-w /etc/systemd/system/ -p wa -k persistence_systemd
-w /usr/lib/systemd/system/ -p wa -k persistence_systemd
-w /etc/init.d/ -p wa -k persistence_init
-w /etc/rc.local -p wa -k persistence_init

# ── NETWORK RECONNAISSANCE ──────────────────────────
-w /usr/bin/nmap -p x -k network_recon
-w /usr/bin/netcat -p x -k network_recon
-w /usr/bin/nc -p x -k network_recon
-w /usr/bin/curl -p x -k network_recon
-w /usr/bin/wget -p x -k network_recon

# ── FILE INTEGRITY — CRITICAL CONFIGS ───────────────
-w /etc/hosts -p wa -k fim_critical
-w /etc/hostname -p wa -k fim_critical
-w /etc/resolv.conf -p wa -k fim_critical
-w /etc/pam.d/ -p wa -k pam_config
-w /lib/security/ -p wa -k pam_module

# ── PROCESS / EXECUTION ─────────────────────────────
# Execution from temp/world-writable directories
-a always,exit -F arch=b64 -S execve -F dir=/tmp -k exec_from_tmp
-a always,exit -F arch=b64 -S execve -F dir=/dev/shm -k exec_from_tmp
-a always,exit -F arch=b64 -S execve -F dir=/var/tmp -k exec_from_tmp

# Kernel module loading (rootkit installation)
-w /sbin/insmod -p x -k kernel_module
-w /sbin/rmmod -p x -k kernel_module
-w /sbin/modprobe -p x -k kernel_module
-a always,exit -F arch=b64 -S init_module -k kernel_module_syscall

# ── WAZUH AGENT PROTECTION ──────────────────────────
-w /var/ossec/ -p wa -k wazuh_tamper
-w /etc/ossec-init.conf -p wa -k wazuh_tamper
```

---

## 🏠 Homelab Environment

```
┌─────────────────────────────────────────────────────────┐
│           NULLBYTE HOMELAB — VMware Workstation         │
│                                                         │
│  VMnet: NAT / Host-Only (192.168.x.0/24)                │
│                                                         │
│  ┌─────────────────────────────────────────────────┐    │
│  │  VM 1: Ubuntu AI + Wazuh Manager                │    │
│  │  IP: 192.168.x.20                               │    │
│  │  Role: Wazuh SIEM, n8n, NullByte pipeline       │    │
│  │  Services: wazuh-manager, n8n, opensearch       │    │
│  └─────────────────────────────────────────────────┘    │
│                                                         │
│  ┌─────────────────────────────────────────────────┐    │
│  │  VM 2: Ubuntu Web Server (Target)               │    │
│  │  IP: 192.168.x.139                              │    │
│  │  Role: Scan target, Wazuh agent deployed        │    │
│  │  Services: apache2/nginx, auditd, wazuh-agent   │    │
│  └─────────────────────────────────────────────────┘    │
│                                                         │
│  ┌─────────────────────────────────────────────────┐    │
│  │  VM 3: Kali Linux (Attacker / Validator)        │    │
│  │  IP: 192.168.x.130                              │    │
│  │  Role: Manual CVE validation, exploit testing   │    │
│  │  Tools: metasploit, nmap, searchsploit          │    │
│  └─────────────────────────────────────────────────┘    │
│                                                         │
│  ┌─────────────────────────────────────────────────┐    │
│  │  VM 4: Windows 11 (Target)                      │    │
│  │  IP: 192.168.x.128                              │    │
│  │  Role: Scan target, Wazuh agent deployed        │    │
│  │  Services: Sysmon, Wazuh agent, Windows Update  │    │
│  └─────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────┘
```

**Scan Scope:** `192.168.x.0/24`
All VMs are enrolled as Wazuh agents reporting to the manager at `192.168.x.20`.

---

## 🤖 AI CVE Hunter Agent

The AI CVE Hunter is an n8n AI Agent node powered by Claude (Anthropic) or GPT-4o. It receives enriched CVE data from the NVD/OSV query stage and performs intelligent triage.

### System Prompt (CVE Triage)

```
You are a senior vulnerability analyst and penetration tester with expertise
in CVE triage, CVSS scoring, and remediation planning.

You will receive a JSON array of CVEs detected on a network. Each CVE includes:
CVE ID, CVSS v3.1 base score, vector string, affected software/version,
target host, and description.

Your tasks:
1. Re-prioritize using a composite risk score: CVSS base score (40%),
   exploitability (30%), asset criticality (20%), CISA KEV match (10% bonus).
2. For each CVE, write a 2-sentence plain-language risk summary.
3. For each CVE, write specific remediation steps (patch version, config
   change, or workaround).
4. Flag any CVE that appears in the CISA KEV catalog as URGENT.
5. Generate an Executive Summary (max 150 words) for leadership.

Return your response as structured JSON matching the provided schema.
Do not include any text outside the JSON object.
```

### Agent Input Schema

```json
{
  "scan_date": "2025-01-12",
  "environment": "homelab-192.168.x.0/24",
  "cve_findings": [
    {
      "cve_id": "CVE-2024-XXXXX",
      "cvss_score": 9.8,
      "cvss_vector": "CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H",
      "affected_software": "OpenSSH 8.2",
      "target_host": "192.168.x.139",
      "description": "..."
    }
  ]
}
```

---

## 🔐 Security Notes

- **API keys** are stored in `.env` files and never committed to version control. `.env` is in `.gitignore`.
- Nmap scans are authorized only within your own controlled lab environment (`192.168.x.0/24`). **Never scan networks you do not own or have explicit written authorization to test.**
- Wazuh API credentials use a dedicated read-only service account (`wazuh-wui`).
- The n8n instance is not exposed to the internet — bind to `127.0.0.1` or internal network only.
- Report files containing CVE details are stored locally and delivered over encrypted channels (TLS SMTP / HTTPS).
- Postman collections contain no hardcoded credentials — all sensitive values use Postman environment variables.
- **This project is for educational and authorized homelab use only.**

---

## 🗺️ Roadmap

| Phase | Feature | Status |
|-------|---------|--------|
| v0.1 | Nmap scan → XML parse → NVD query | ✅ Complete |
| v0.1 | n8n basic pipeline skeleton | ✅ Complete |
| v0.1 | Claude AI CVE triage (HTTP Request node) | ✅ Complete |
| v0.1 | Auto-generated Markdown remediation report | ✅ Complete |
| v0.1 | Write report to disk + GitHub push | ✅ Complete |
| v0.2 | CVE deduplication (seen Set) | ✅ Complete |
| v0.2 | Loop Over Items — multi-CVE batch triage | ✅ Complete |
| v0.2 | 65s Wait throttle — rate limit handling | ✅ Complete |
| v0.2 | Email delivery via Gmail SMTP | ✅ Complete |
| v0.3 | Weekly Schedule Trigger (Sunday 2AM) | ✅ Complete |
| v0.4 | OSV.dev cross-reference + deduplication | 🔲 Planned |
| v0.4 | HTML report template | 🔲 Planned |
| v0.5 | Wazuh custom rules + alert integration | 🔲 Planned |
| v0.5 | Weekly cron schedule activation | 🔲 Planned |
| v0.6 | CISA KEV catalog matching | 🔲 Planned |
| v1.0 | Dashboard (Grafana / n8n UI) | 🔲 Future |
| v1.0 | Historical trending — week-over-week delta | 🔲 Future |
| v1.2 | Nuclei scanner integration | 🔲 Future |
| v1.3 | NullByte + GhostNet unified risk dashboard | 🔲 Future |

---

## 👤 Author

**Eliezer Fuentes** — Cybersecurity Professional

Threat Hunting | Vulnerability Management | SOC Automation | Offensive Security

[![GitHub](https://img.shields.io/badge/GitHub-enak223-181717?style=flat&logo=github)](https://github.com/enak223)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-eliezerfuentes-0A66C2?style=flat&logo=linkedin)](https://www.linkedin.com/in/eliezerfuentes/)

---

> *Every unpatched CVE is a null byte waiting to be weaponized. NullByte finds them first.*
