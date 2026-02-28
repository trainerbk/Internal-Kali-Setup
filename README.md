# Internal-Kali-Setup

Automated installer for an authorized internal penetration testing toolkit on Kali Linux. Designed to be deployed on a NUC shipped to a client site.

## Usage

```bash
sudo ./pentest_setup.sh
```

> Must be run as root. Tested on Kali Linux (2023+).

## What It Does

- Runs full system `apt update`, `upgrade`, and `dist-upgrade`
- Installs all base dependencies (Python, Go, Rust via rustup, libpcap, Kerberos libs, etc.)
- Clones all git-based tools to `/opt/<toolname>`
- Installs Python tools via `pip3` or `pipx` where isolation is needed
- Builds Go and Rust tools from source
- Creates symlinks in `/usr/local/bin` for easy CLI access
- Logs everything to `/var/log/pentest_setup.log`
- Prints a pass/fail summary on completion

## Tools Installed

| Tool | Source | Category |
|------|--------|----------|
| [Croc](https://github.com/schollz/croc) | Official installer | File transfer |
| [Responder](https://github.com/lgandx/Responder) | git | LLMNR/NBT-NS poisoning |
| [Certipy](https://github.com/ly4k/Certipy) | git + pipx | Active Directory / ADCS |
| [KrbRelayX](https://github.com/dirkjanm/krbrelayx) | git | Kerberos relay |
| [mitm6](https://github.com/dirkjanm/mitm6) | git | IPv6 MITM |
| [LdapRelayScan](https://github.com/zyn3rgy/LdapRelayScan) | git | LDAP relay detection |
| [cloud_enum](https://github.com/initstring/cloud_enum) | git | Cloud asset enumeration |
| [CloudHunter](https://github.com/belane/CloudHunter) | git | Cloud bucket hunting |
| [dnshunter](https://github.com/5amu/dnshunter) | git | DNS recon |
| [msftrecon](https://github.com/Arcanum-Sec/msftrecon) | git | Microsoft recon |
| [PCredz](https://github.com/lgandx/PCredz) | git | Credential extraction (pcap) |
| [lnk-it-up](https://github.com/wietze/lnk-it-up) | git | LNK file generation |
| [RustHound-CE](https://github.com/g0h4n/RustHound-CE) | git + cargo | BloodHound data collection |
| [Impacket](https://github.com/fortra/impacket) | git | Windows protocol suite |
| [BloodHound.py](https://github.com/dirkjanm/BloodHound.py) | git | AD enumeration |
| [Poetry](https://github.com/python-poetry/poetry) | Official installer | Python package manager |
| [NetExec](https://github.com/Pennyw0rth/NetExec) | git + pipx | Network protocol execution |
| [Coercer](https://github.com/p0dalirius/Coercer) | git | Authentication coercion |

## Notes

- **RustHound-CE** and the cargo build require an internet connection to pull crates on first run — this can take several minutes
- **Rust** is installed via `rustup` rather than apt, as the apt package is too old for several required crates
- **Certipy** and **NetExec** are installed via `pipx` to avoid conflicts with Kali's system-managed Python packages
- If any tool fails, the script continues and reports failures in the final summary
- Re-running the script is safe — existing git repos are pulled rather than re-cloned

## Log

Full output is written to `/var/log/pentest_setup.log` for troubleshooting failed installs.

## Legal

This tool is intended for use on authorized engagements only. Ensure you have written permission before deploying on any network or system.
