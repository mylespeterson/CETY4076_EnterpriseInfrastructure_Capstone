# 📘 PROJECT DOCUMENT 1
## Enterprise Network Infrastructure & Security — 14-Week Capstone Project

**Course Level:** Advanced Networking / Cybersecurity
**Duration:** 14 Weeks | **Class Time:** 4 Hours/Week
**Format:** Group Project (Groups of 2–4 Students)
**Instructor Role:** CEO / IT Manager (students report to professor as their employer)

---

## 🏢 Business Scenario / Problem Statement

You have been hired as a junior IT team at **Meridian Financial Solutions**, a mid-sized financial services company that is expanding its internal infrastructure. The company currently runs an aging, flat network with no segmentation, no centralized identity management, and no formal security monitoring. The CIO has mandated a full rebuild of the internal server environment aligned to **Zero Trust Architecture** principles, modern endpoint management, and regulatory compliance requirements.

Your team has been assigned a **dedicated blade server** in the company's on-premises data center, connected to SAN storage. Your mandate is to design, build, document, and present a fully operational secure infrastructure environment by end of term. The CIO has also directed that new builds be **automated and repeatable** (Infrastructure as Code) and that the security team begin evaluating **AI-assisted tooling** for both offense (red team) and defense (blue team) operations.

The CIO (your professor) will meet with your team at the start of the project to hand off requirements and will conduct scheduled check-ins throughout the project lifecycle.

---

## 🖥️ Hardware Resources (Per Group)

| Resource | Specification |
|---|---|
| Blade Server RAM | 64 GB |
| CPU Cores (Physical) | As assigned per blade |
| Storage | SAN-connected shared storage |
| Hypervisor | Proxmox VE (free/open-source) |
| BMC Module | iDRAC / iLO (vendor-dependent) |

---

## 💰 Budget System & Cost Table

Each group is assigned a **virtual budget of $6,750 CAD** for the duration of the project (increased from $6,500 to accommodate the Ansible control node and optional AI tooling). Students must track and plan their resource consumption against this budget. The budget applies to vCPU allocation, RAM allocation, storage, and Microsoft licensing.

> **Important:** All costs are simulated/representative for educational purposes and are based on approximate real-world values.

### Resource Cost Table

| Resource | Unit | Cost (CAD) | Notes |
|---|---|---|---|
| **vCPU** | Per vCPU allocated | $40/vCPU | Applies to all VMs combined |
| **RAM** | Per GB allocated | $6/GB | Applies to all VMs combined |
| **SAN Storage** | Per 50 GB block | $15/block | Thin provisioning allowed |
| **Windows Server 2022 Standard** | Per 16-core license | $1,400 | Covers 2 VMs per license |
| **Windows Server 2022 Datacenter** | Per 16-core license | $6,200 | Unlimited VMs |
| **Windows Server User CAL** | Per user | $38 | Required per AD user account |
| **Windows 11 Pro (Education)** | Per device | $65 | Per client workstation VM |
| **Palo Alto VM-50 (1-year license)** | Per instance | $1,800 | Includes base threat prevention |
| **Palo Alto Threat Prevention Add-on** | Annual | $600 | Optional but recommended |
| **Wazuh** | Free | $0 | Open-source SIEM |
| **Ubuntu Server 22.04 LTS** | Free | $0 | Open-source |
| **Kali Linux** | Free | $0 | Open-source |
| **Proxmox VE** | Free (community) | $0 | No subscription required for lab |
| **Ansible (Community)** | Free | $0 | Open-source, run from control node |
| **Ansible AWX/Tower (optional)** | Free (AWX) | $0 | Optional UI/scheduling layer, extra credit |
| **AI Attack/Scan Tooling** (local LLM via Ollama, PentestGPT-style agents) | Free | $0 | Local model preferred to avoid external API cost/data exposure |
| **Cloud LLM API credits (optional)** | Per group allotment | $50 | Only if using hosted models (e.g., GPT-4/Claude API) instead of a local model |

### 📊 Sample Budget Calculation (Example — Not Student Submission)

| Item | Qty | Unit Cost | Total |
|---|---|---|---|
| vCPU (across all VMs) | 21 vCPUs | $40 | $840 |
| RAM (across all VMs) | 54 GB | $6 | $324 |
| SAN Storage (5 × 50 GB blocks) | 5 | $15 | $75 |
| Windows Server 2022 Standard (×2 licenses) | 2 | $1,400 | $2,800 |
| Windows Server User CAL (×20 users) | 20 | $38 | $760 |
| Windows 11 Pro Education (×2 VMs) | 2 | $65 | $130 |
| Palo Alto VM-50 | 1 | $1,800 | $1,800 |
| Cloud LLM API credits (optional) | 1 | $50 | $50 |
| **Total** | | | **$6,779** |

> 🔴 **Note:** This sample is intentionally slightly over budget. Students will recognize this and must adjust their design or submit a budget increase request.

---

## 📋 VM Environment Overview

Students must plan and deploy the following virtual machines within Proxmox:

| VM # | Role | OS | Min vCPU | Min RAM | Notes |
|---|---|---|---|---|---|
| **VM-00** | **Ansible Control Node** | Ubuntu Server 22.04 | 1 | 2 GB | Runs all playbooks; SSH keys + WinRM creds vaulted with `ansible-vault` |
| VM-01 | Palo Alto Virtual Firewall | PAN-OS (VM-50) | 2 | 5.5 GB | Base config manual; security policy pushes may use `pan-os-ansible` collection |
| VM-02 | Primary Domain Controller (DC1) | Windows Server 2022 | 2 | 4 GB | Promotion manual; OU/user provisioning via `microsoft.ad` Ansible collection required |
| VM-03 | Secondary DC + DHCP/DNS/VPN/SMB/DFSR | Windows Server 2022 | 2 | 6 GB | |
| VM-04 | Certificate Authority (CA) Server | Windows Server 2022 | 2 | 4 GB | |
| VM-05 | Ubuntu Server (Samba/SSH/Nginx/SFTP) | Ubuntu Server 22.04 | 2 | 4 GB | Built primarily via Ansible role, not manual `apt install` |
| VM-06 | Wazuh SIEM + Manager | Ubuntu Server 22.04 | 4 | 8 GB | Agent enrollment automated via Ansible (`wazuh-ansible` roles) |
| VM-07 | Windows 11 Client #1 | Windows 11 Pro | 2 | 4 GB | Domain join, GPO refresh, Wazuh agent installed via Ansible |
| VM-08 | Windows 11 Client #2 | Windows 11 Pro | 2 | 4 GB | Same as VM-07 |
| **VM-09** | **AI-Augmented Attack Node** | Kali Linux + AI tooling | 4 | 8 GB | Kali base plus AI recon/attack-planning/triage agents (see AI section below) |
| **Total** | | | **~23 vCPU** | **~47.5 GB** | |

> Students may also add optional monitoring VMs (e.g., Zabbix/Grafana) within budget.

---

## 🌐 Network Design Requirements

Students must design and document a segmented network topology. The following network zones are required:

| Zone | Purpose | Notes |
|---|---|---|
| **BMC/Management Network** | Secure OOB access to iDRAC/iLO | Isolated, no internet access |
| **External/WAN Zone** | Internet-facing, Kali/AI attacker access | Managed by Palo Alto |
| **DMZ** | Web-facing services (Nginx) | Controlled ingress/egress |
| **Internal LAN** | AD, clients, servers | Segmented from DMZ |
| **Server/Infrastructure Zone** | DC, CA, Ubuntu, Wazuh | Isolated, admin-only access |
| **Automation Zone (NEW)** | Ansible control node | Restricted management-only access to all other zones via SSH/WinRM; no direct internet exposure |

### Zero Trust Requirements
- All traffic must be explicitly permitted via Palo Alto security policies (deny-by-default)
- Micro-segmentation between all zones
- All admin access via VPN or jump host (no direct exposure)
- MFA documented/recommended for remote admin access
- BMC access restricted to dedicated management VLAN only
- All communications (where possible) must use certificates issued by the CA server
- Ansible control node credentials (SSH keys, WinRM secrets) must be stored using `ansible-vault` — never in plaintext in the repository

---

## 🏗️ Technical Requirements by Component

### 🤖 Ansible Control Node (VM-00) — NEW
- Deploy an Ubuntu Server VM as the dedicated Ansible control node
- Build and maintain an Ansible repository containing:
  - A static or dynamic inventory (`hosts.ini` or Proxmox dynamic inventory plugin)
  - Roles organized per service (e.g., `ad_dc`, `ubuntu_base`, `wazuh_agent`, `windows_client`)
  - Playbooks that are idempotent (safe to re-run without unintended changes)
- Use `ansible-vault` to encrypt all credentials (SSH keys, WinRM passwords, API tokens)
- Demonstrate a clean re-deployment: wipe a VM, re-run the applicable playbook, and confirm the service returns to a fully configured state
- Run `--check` (dry-run) mode against a live environment and capture the diff as evidence of idempotency

### 🔒 Palo Alto Virtual Firewall (VM-01)
- Configure all inter-zone security policies (deny by default)
- Set up NAT rules for internal-to-external traffic
- Configure URL filtering and threat prevention profiles
- Enable Security Event logging — forward logs to Wazuh via Syslog
- Set up GlobalProtect VPN for IT admin remote access
- Demonstrate blocking of at least two threat categories
- Create at least one custom security profile
- *(Optional/Extra Credit)* Push a subset of security policy changes using the `paloaltonetworks.panos` Ansible collection and document the workflow

### 🏢 Active Directory & Group Policy (VM-02 & VM-03)

**DC1 — Primary Domain Controller:**
- Install and configure AD DS
- Create Organizational Units (OUs): IT, Finance, HR, Servers, Workstations
- Create and manage user accounts (minimum 10 test users across departments) — OU/user provisioning must use the `microsoft.ad` Ansible collection (or documented PowerShell DSC) rather than manual ADUC entry only
- Create and link Group Policy Objects (GPOs):
  - Password policy (complexity, length, expiry)
  - Account lockout policy (threshold, duration, observation window)
  - BitLocker Drive Encryption — enforced via GPO, keys stored in AD
  - Windows Defender Firewall policy
  - Security Event Log audit policy (logon events, privilege use, object access, account management, policy changes)
  - Software restriction/AppLocker policies
  - PowerShell execution policy enforcement
  - Certificate auto-enrollment via CA
- Build an Ansible playbook that validates GPO application across clients (queries `gpresult` or relevant registry keys) as an automated compliance check

**DC2 — Secondary DC + Services:**
- Promote as additional domain controller (AD DS replication)
- Install and configure:
  - DHCP Server (scopes for all internal subnets)
  - DNS (forwarders, conditional forwarders, split-brain DNS)
  - VPN (RRAS or SSTP VPN endpoint for internal users)
  - SMB File Share with Storage Pools (create tiered storage pools using SAN-backed volumes)
  - DFSR (DFS Replication) between DC1 and DC2 for SYSVOL and share redundancy

### 🔐 Certificate Authority (VM-04)
- Deploy a two-tier CA hierarchy (Root CA + Issuing CA) — or single-tier for simplified design (must justify)
- Issue certificates for:
  - All Windows Server internal hostnames (LDAPS, WinRM, etc.)
  - Ubuntu server (HTTPS for Nginx, SFTP certificate)
  - Palo Alto firewall admin interface
  - VPN endpoints
- Configure CA certificate auto-distribution via GPO to all domain-joined devices
- Document certificate lifecycle management and revocation process (CRL/OCSP)

### 🐧 Ubuntu Server (VM-05)
- Join to the Active Directory domain (using SSSD or `realmd`)
- Configure AD authentication — domain users must be able to log in
- Services to configure (deployed via a dedicated Ansible role, not manual package installation):
  - **Samba** — file share accessible to AD users (with share permissions tied to AD groups)
  - **SSH** — key-based and password authentication, AD users permitted
  - **Nginx** — web server serving a basic internal company intranet page (HTTPS using CA cert)
  - **SFTP** — secure file transfer using SSH subsystem, directory isolation per user
- Create a **PowerShell/Bash script** (or Ansible task) to import users from a CSV file into the Linux environment (create local home directories, set shell, assign groups)
- Wazuh agent installed and reporting to Wazuh Manager (installation automated via Ansible role)

### 📊 Wazuh SIEM (VM-06)
- Deploy Wazuh Manager + Dashboard
- Install Wazuh agents on all VMs using an Ansible role (`wazuh-ansible` or equivalent) rather than manual per-host installation:
  - All Windows Server VMs
  - Ubuntu Server
  - Windows 11 Clients
- Configure rules and alerts for:
  - **Brute Force Detection** — AD logon failures (EventID 4625), SSH failures
  - **SSH Attack Detection** — repeated failed SSH attempts, port scanning signatures
  - **Network Mapping/Reconnaissance** — alerts on Nmap signatures and port sweeps
  - **Privilege Escalation Attempts**
  - **CIS Policy Benchmarks** — run CIS benchmark scans against all Windows and Linux systems, report compliance scores in Wazuh dashboard
- Configure **Active Response**:
  - Auto-block IP addresses on repeated brute force (Windows & Linux)
  - Auto-block SSH brute force (firewall rule injection on Ubuntu via `iptables`)
  - Alert + block on detected Nmap scans from the attack node
- Create a custom Wazuh dashboard showing:
  - Top alerting hosts
  - CIS compliance scores per host
  - Authentication failure trends
  - Active response events
- Use an **AI-assisted triage agent** to summarize Wazuh alert JSON output and recommend response actions; document how the AI's recommendation compared to the student's own analyst judgment

### 📡 Network Monitoring — Zabbix or Grafana + Prometheus (Optional but Recommended — Extra Credit)
- Deploy an open-source network monitoring solution (e.g., **Zabbix** or **Prometheus + Grafana**)
- Monitor:
  - Interface utilization on all VMs
  - CPU/RAM usage per VM
  - Disk I/O on SAN-backed volumes
  - Network latency between zones
  - DHCP lease counts
- Create dashboards showing real-time and historical trends

### 💻 Windows 11 Clients (VM-07 & VM-08)
- Join to the Active Directory domain
- Apply all GPOs (BitLocker, firewall, audit, certificate trust)
- BitLocker:
  - Enabled via GPO
  - Recovery keys stored in Active Directory (verify in ADUC)
  - TPM + PIN or TPM-only mode (document choice)
- Wazuh agent installed (via Ansible role)
- Test domain user login from both clients
- Test Samba and SFTP file access from clients to servers

### 🖥️ AI-Augmented Attack Node (VM-09)
- Connected to **both external and internal network** (dual-homed)
- Built on a Kali Linux base with AI tooling layered on top: a local LLM runtime (e.g., Ollama/LM Studio) or a sandboxed API connection to a hosted model, plus an AI-assisted recon/attack-planning wrapper (e.g., a PentestGPT-style tool)
- Used near end of project (Week 11–12) for:
  - **AI-Assisted Reconnaissance** — run Nmap/Nikto and feed the output to an AI agent that interprets results and recommends next steps, rather than the student manually chaining commands from memory
  - **AI-Assisted Attack Planning** — use an AI agent to generate a brute-force/exploitation plan and target prioritization against AD and SSH; the actual attack is still executed with standard tools (Hydra, CrackMapExec) — the AI directs strategy and reporting, not blind auto-exploitation
  - **Brute Force Attack** against AD (using Hydra or CrackMapExec), informed by the AI's plan
  - **SSH Brute Force** against Ubuntu (using Hydra)
  - **Network Mapping** using Nmap to scan all internal subnets
  - **Web Vulnerability Scan** against Nginx using Nikto or dirb
  - Student must document findings as an **AI-Assisted Attack Report** and then relate each finding back to their Wazuh alerts and Palo Alto logs
  - Demonstrate that active response blocked or mitigated each attack
  - Document every AI-suggested action **before** executing it — no blind auto-execution of AI-generated exploit commands. A human (the student) remains in the loop for every action taken against the environment.

#### AI Tooling Guardrails
- Prefer **local LLMs** (Ollama, LM Studio) over public cloud APIs to avoid sending sensitive AD data, credentials, hashes, or internal hostnames off the lab network
- If a hosted API is used, it must be documented, budgeted (see Cost Table), and scrubbed of any real credentials/secrets before submission to the model
- Maintain an **AI Tooling Disclosure** log: which tools/models were used, what prompts were given, and what guardrails were applied

---

## 📅 Weekly Breakdown & Checkpoints

| Week | Topics / Deliverables | Checkpoint |
|---|---|---|
| **Week 1** | **Introduction & Project Kickoff** — Professor briefs as CEO. Groups formed, blade servers assigned. Students review business requirements document. Initial meeting scheduled. | ✅ **Checkpoint 1:** Team contract signed, initial meeting with professor completed, blade server access confirmed |
| **Week 2** | **Environment Planning** — Network diagram drafted (zones, VLANs, IP scheme). VM list created (including Ansible control node and AI attack node). Budget plan submitted (draft). Proxmox installed on blade server. Draft Ansible inventory and decide on role structure. | Budget draft due |
| **Week 3** | **Proxmox Build & Network Setup** — VLANs and virtual networks configured in Proxmox. Palo Alto VM deployed, base firewall rules configured. BMC management network isolated. Deploy the Ansible control node VM. Write and test a first playbook (base OS hardening role — users, SSH keys, timezone, updates) applied to all Linux VMs. | |
| **Week 4** | **AD Core Build** — DC1 promoted, AD DS installed, OUs and users created (via `microsoft.ad` Ansible collection or documented PowerShell DSC). DNS configured. DC2 promoted, DHCP and DNS secondary configured. | ✅ **Checkpoint 2 (Week 4 Check-in Meeting):** Prof reviews AD structure, network diagram, Proxmox VM list, Ansible repo progress, and budget plan. Discuss blockers. Budget plan v1 due (finalized). |
| **Week 5** | **Certificate Authority** — CA server deployed. Root/Issuing CA configured. GPO auto-enrollment configured. All domain devices receiving certificates. | |
| **Week 6** | **Group Policy Deep Dive** — BitLocker GPO, audit policy, password policy, AppLocker, PowerShell execution policy, Certificate auto-enrollment. Test on Windows 11 clients. Build an Ansible playbook that validates GPO application across clients. | |
| **Week 7** | **Ubuntu Server Setup (Automation Showcase)** — Ubuntu server built primarily via Ansible role (Samba, SSH, Nginx HTTPS, SFTP). Wazuh agent role scaffolded. AD user login on Ubuntu tested. | ✅ **Checkpoint 3 (Week 7 Check-in Meeting):** Prof reviews GPO implementation, Ansible role for Ubuntu build, BitLocker keys in AD. Over-budget scenario introduced (see below). |
| **Week 8** | **Over-Budget Scenario** — Professor announces a new requirement (e.g., adding a second Ubuntu server for a web app, adding a Grafana/Zabbix monitoring VM, or requiring hosted AI API credits). Students must recalculate budget, identify overage, and **book a formal meeting with the professor** to request a budget increase with written rationale. Scenario now also includes: "The CIO wants the environment redeployable in under 2 hours if the blade fails — demonstrate an Ansible re-run from a wiped VM." | 📋 **Budget Increase Request Document Due** |
| **Week 9** | **Budget Meeting with Professor** — Formal budget review meeting. Students present rationale, impact analysis, and revised resource plan. | ✅ **Checkpoint 4:** Budget meeting completed. Approved or revised scope confirmed. |
| **Week 10** | **Wazuh Deployment (Automated)** — Wazuh manager + dashboard deployed. Agents installed on all VMs via Ansible role. CIS benchmark scans run. Brute force and SSH rules configured. Active response configured. | |
| **Week 11** | **AI-Assisted Attack Simulation** — AI-augmented attack node deployed. AI-assisted recon/scan agent run against the environment; AI-assisted attack plan generated for brute force/exploitation against AD and SSH; Nmap scans performed. Results documented. Wazuh and Palo Alto logs reviewed against attacks, including AI-assisted alert triage. PowerShell/Bash user import script completed and tested. | |
| **Week 12** | **Hardening, Review & Remediation** — Address findings from the AI-assisted attack simulation using a **playbook-driven hardening pass** (e.g., an Ansible role that disables weak ciphers, enforces fail2ban/auditd config). Final GPO review. Full environment validation. Documentation finalized. Ansible idempotency test performed (`--check` diff captured). | ✅ **Checkpoint 5 (Week 12 Final Check-in):** All systems operational, documentation reviewed, presentation assigned |
| **Week 13** | **Presentation Preparation** — Groups prepare 15-minute presentation and live demo. | Presentation slides/demo environment ready |
| **Week 14** | **Presentations & Demo Day** — Each group presents (~15 minutes) to class + professor. Live demo required. Q&A from class. Final documentation submitted. | 🎓 **Final Submission Due** |

---

## 📁 Deliverables (Full Project)

| # | Deliverable | Due |
|---|---|---|
| 1 | Team Contract + Initial Meeting Notes | Week 1 |
| 2 | Network Diagram (logical + physical, including Automation Zone) | Week 2 |
| 3 | VM Inventory & Budget Plan v1 | Week 4 |
| 4 | AD Structure Documentation (OUs, GPOs, Users) | Week 6 |
| 5 | Certificate Authority Documentation | Week 6 |
| 6 | Ubuntu Configuration Documentation (including Ansible role) | Week 7 |
| 7 | Budget Increase Request Document | Week 8 |
| 8 | Wazuh Dashboard Screenshots + CIS Report | Week 11 |
| 9 | **AI-Assisted Attack Report** (attacker perspective + blue team response, including AI tool/prompt log and comparison to manual technique) | Week 12 |
| 10 | PowerShell/Bash User Import Script + Test Output | Week 12 |
| 11 | **Ansible Repository** (roles, playbooks, inventory) with README documenting execution order | Week 7 (initial), Week 12 (final) |
| 12 | **AI Tooling Disclosure Document** (tools/models used, prompts, guardrails, data privacy notes) | Week 11 |
| 13 | **Ansible Idempotency Test Evidence** (`--check` diff screenshot showing zero unintended changes) | Week 12 |
| 14 | Final Technical Documentation Package | Week 14 |
| 15 | 15-Minute Presentation + Live Demo | Week 14 |

---

## 📎 PowerShell Requirements

Students must use PowerShell for at least the following:

- Bulk creation of AD users from a CSV file
- GPO reporting (`Get-GPOReport`)
- BitLocker key retrieval from AD (`Get-ADObject`)
- Querying AD for locked-out accounts
- Generating a Security Event Log report (filtered by EventIDs: 4625, 4624, 4648, 4720, 4740)

## 🤖 Ansible Requirements

Students must use Ansible for at least the following:

- A base OS hardening role applied to all Linux VMs
- Full build-out of the Ubuntu server (Samba, SSH, Nginx, SFTP)
- Wazuh agent installation/enrollment across all VMs
- A GPO-application validation playbook run against Windows clients
- A hardening/remediation playbook triggered by findings from the AI-assisted attack simulation
- Secrets management via `ansible-vault` for all credentials used by playbooks
- An idempotency demonstration (`--check` mode re-run with no unintended changes)

## 🧠 AI Tooling Requirements

Students must use AI tooling for at least the following:

- AI-assisted reconnaissance/scan output interpretation (feeding Nmap/Nikto results to an AI agent for analysis and next-step recommendations)
- AI-assisted attack planning/target prioritization for the brute-force/exploitation phase
- AI-assisted triage of Wazuh alerts (summarization and recommended response actions)
- A documented comparison between AI-recommended actions and the student's own manual analysis/judgment
- A completed AI Tooling Disclosure document (see Deliverables)

---

## 🎤 Presentation Requirements (Week 14)

**Duration:** ~15 minutes per group + 5 minutes Q&A
**Format:** Live demo of running environment + slide deck

**Required demo elements:**
1. Show Palo Alto security policies and a blocked threat
2. Show Wazuh dashboard with active alerts and CIS compliance report
3. Demonstrate BitLocker key visible in Active Directory
4. Log in as an AD user on Ubuntu Server
5. Show certificate trust chain on a client
6. Trigger a brute force attempt from the AI-augmented attack node and show it blocked by Active Response
7. Demonstrate an Ansible playbook re-run (idempotency) against a live or freshly wiped VM
8. Show an example AI-assisted triage summary of a Wazuh alert and explain how the team validated or overrode the AI's recommendation

---

## 📝 General Notes

- **Wazuh** is free and open-source — no licensing cost
- **Palo Alto VM-50 / VM-Series** — institutions may be eligible for the **Palo Alto Networks Academic Program (PANA)** which provides free or heavily discounted VM-Series licenses for educational use. Check with your institution.
- **Ubuntu, Kali, Proxmox, Ansible** — all free/open-source, $0 cost in budget
- **Microsoft Academic/EES licensing** — many institutions have campus agreements that dramatically reduce costs; the table above uses approximate retail-adjacent education pricing for the purpose of the simulation
- Open-source network monitoring suggestions: **Zabbix** (full-featured), **Prometheus + Grafana** (metrics/dashboards), **ntopng** (network flow analysis), **LibreNMS** (SNMP-based monitoring)
- **AI tooling** should default to local/self-hosted models (e.g., Ollama) to avoid sending sensitive lab data to third-party services; any use of hosted AI APIs must be documented and budgeted
