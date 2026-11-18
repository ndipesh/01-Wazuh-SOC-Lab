# 01-Wazuh-SOC-Lab

Wazuh SIEM SOC home lab on Apple M3 — threat detection, compliance monitoring (NIST 800-53, PCI-DSS, HIPAA), and incident response documentation.

## Overview

This home lab demonstrates end-to-end SIEM deployment for a SOC Analyst portfolio. It covers installation, agent deployment, real-time log monitoring, and incident documentation — the core daily tasks of a Tier 1 SOC Analyst.

> **ARM64 Note:** Wazuh's quickstart installer targets x86_64 only. This lab documents the full workaround to deploy on Apple Silicon, including an architecture patch and manual package installation.

## Architecture

```
Mac Host (Apple M3)              Ubuntu VM (192.168.64.3)
Dipeshs-Laptop.local      ─────►  wazuh-server
Wazuh Agent 001                   Wazuh Manager 4.14 ARM64
         TCP 1514/1515 (OSSEC encrypted)
```

| Component | Details |
|-----------|---------|
| SIEM | Wazuh Manager 4.14.10-1 (ARM64) |
| Agent | Wazuh Agent for macOS (ID: 001) |
| Server OS | Ubuntu Server 22.04.5 LTS ARM64 |
| Hypervisor | UTM 4.x / QEMU (Apple M3) |
| Network | UTM Shared — 192.168.64.0/24 |

## Skills Demonstrated

`SIEM Deployment` `Log Analysis` `Incident Response` `Linux Administration`
`Network Security` `Compliance Mapping` `Virtualization (UTM)` `SSH`
`systemd Services` `PAM Security`

## Setup

### 1 — Provision Ubuntu VM (UTM)
- Download Ubuntu Server 22.04 ARM64 ISO
- Create UTM VM: QEMU mode, 2 CPU, 4 GB RAM, 20 GB disk
- Install Ubuntu with OpenSSH server enabled
- Note VM IP: `ip a` → typically `192.168.64.x`

### 2 — Install Wazuh Manager (ARM64 workaround)

```bash
# Add Wazuh repository
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | \
  sudo gpg --no-default-keyring \
  --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg \
  --import && sudo chmod 644 /usr/share/keyrings/wazuh.gpg

echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] \
  https://packages.wazuh.com/4.x/apt/ stable main" | \
  sudo tee /etc/apt/sources.list.d/wazuh.list

# Install manager only (ARM64 package exists)
sudo apt-get update && sudo apt-get install -y wazuh-manager
sudo systemctl enable --now wazuh-manager
```

### 3 — Deploy Mac Agent

```bash
curl -so wazuh-agent.pkg \
  https://packages.wazuh.com/4.x/macos/wazuh-agent-4.14.0-1.arm64.pkg
sudo installer -pkg wazuh-agent.pkg -target /
sudo /Library/Ossec/bin/agent-auth -m 192.168.64.3
sudo sed -i '' 's/MANAGER_IP/192.168.64.3/g' /Library/Ossec/etc/ossec.conf
sudo /Library/Ossec/bin/wazuh-control start
```

### 4 — Monitor Live Alerts

```bash
ssh dipesh@192.168.64.3
sudo tail -f /var/ossec/logs/alerts/alerts.log
```

## Results

- ✅ Wazuh Manager running on ARM64 Ubuntu VM
- ✅ Mac endpoint registered as Agent 001 (Active)
- ✅ Live alerts: Rule 5402 (sudo escalation), Rule 5501 (PAM login)
- ✅ Compliance tags active: NIST 800-53, PCI-DSS, HIPAA
- ✅ Incident report documented: `reports/IR-2024-001-sudo-escalation.md`

## Repository Structure

```
01-Wazuh-SOC-Lab/
├── README.md
├── docs/
│   ├── architecture.md
│   └── arm64-install-notes.md
├── configs/
│   └── ossec.conf.example
├── reports/
│   └── IR-2024-001-sudo-escalation.md
└── screenshots/
```

## Resources

- [Wazuh Documentation](https://documentation.wazuh.com/)
- [MITRE ATT&CK](https://attack.mitre.org/)
- [NIST SP 800-53 Rev. 5](https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final)
