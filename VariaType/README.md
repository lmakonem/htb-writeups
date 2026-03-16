# VariaType - HackTheBox

![VariaType Pwned](./pwned.png)

## Machine Info

| Property | Value |
|----------|-------|
| Name | VariaType |
| OS | Linux |
| Difficulty | Medium |
| Release Date | Season 10 (2026) |
| Status | **Active** |
| IP | 10.129.6.242 |

## Skills Required

- Web Application Enumeration
- Git Repository Analysis
- CVE Research and Exploitation
- Linux Privilege Escalation
- Cronjob Analysis

## Skills Learned

- Exposed .git repository credential extraction
- fontTools CVE-2025-66034 (XML injection / arbitrary file write)
- FontForge CVE-2024-25081 (command injection via crafted filenames)
- Python setuptools CVE-2025-47273 (path traversal in PackageIndex)
- Base64 payload encoding for filename injection
- Cronjob exploitation techniques

## Writeup Status

**This writeup is currently locked as the machine is still active on HackTheBox.**

The full writeup will be available after the machine retires.

| File | Description |
|------|-------------|
| `VariaType_writeup_LOCKED.pdf` | Password-protected PDF |

## Quick Stats

- User Flag: Obtained
- Root Flag: Obtained
- Attack Vector: Web Application (.git exposure + fontTools RCE)
- Pivots Required: 2 (www-data -> steve -> root)

## Attack Path Summary

1. **Enumeration** - Nmap reveals SSH (22) and HTTP (80), discover variatype.htb and portal.variatype.htb
2. **Git Exposure** - Extract credentials (gitbot:G1tB0t_Acc3ss_2025!) from exposed .git repository
3. **Initial Access (www-data)** - Exploit fontTools CVE-2025-66034 via malicious .designspace file with path traversal to write PHP webshell
4. **Reverse Shell** - Trigger webshell for callback as www-data
5. **Lateral Movement (steve)** - Discover cronjob in /opt/process_client_submissions.bak, exploit FontForge CVE-2024-25081 via ZIP with malicious filename containing base64-encoded reverse shell
6. **Privilege Escalation (root)** - Abuse sudo permission on install_validator.py, exploit setuptools CVE-2025-47273 path traversal to write SSH public key to /root/.ssh/authorized_keys

## Services Discovered

| Port | Service | Details |
|------|---------|---------|
| 22 | SSH | OpenSSH 9.2p1 Debian |
| 80 | HTTP | nginx 1.22.1 - VariaType Labs Variable Font Generator |

## Key Vulnerabilities

| CVE | Component | Description |
|-----|-----------|-------------|
| CVE-2025-66034 | fontTools varLib | XML injection / arbitrary file write via .designspace filename attribute |
| CVE-2024-25081 | FontForge Splinefont | Command injection via crafted filenames with $() shell syntax |
| CVE-2025-47273 | Python setuptools | Path traversal via URL-encoded slashes in PackageIndex.download() |

## Domains

- `variatype.htb` - Main application (Variable Font Generator)
- `portal.variatype.htb` - Validation Portal (requires authentication)

## Credentials Obtained

| Service | Username | Source |
|---------|----------|--------|
| Portal | gitbot | .git repository commit history |
| SSH | root | CVE-2025-47273 SSH key injection |

## Files of Interest

| Path | Description |
|------|-------------|
| `/opt/process_client_submissions.bak` | Steve's cronjob script - processes fonts with FontForge |
| `/opt/font-tools/install_validator.py` | Sudo-allowed script using vulnerable setuptools |
| `/var/www/portal.variatype.htb/public/files/` | Upload directory monitored by cronjob |

## Tags

`linux` `medium` `web` `git` `fonttools` `fontforge` `setuptools` `cve` `rce` `cronjob` `path-traversal` `xml-injection`
