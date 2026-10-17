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
| [ADhammer](https://github.com/icedracon/adhammer) | git + cargo | AD audit + attack validation (41 checks, DCSync, tickets, relay, ADCS) |
| [Impacket](https://github.com/fortra/impacket) | git | Windows protocol suite |
| [adnullenum](https://github.com/crypt0p3g/adnullenum) | git | Anonymous (null session) AD enumeration over SAMR/LSARPC |
| [CVE-2026-54121 (Certighost)](https://github.com/aniqfakhrul/CVE-2026-54121) | git | AD CS cdc-redirect PoC — coerces CA into issuing DC certificate |
| [BloodHound.py](https://github.com/dirkjanm/BloodHound.py) | git | AD enumeration |
| [ADscan](https://github.com/ADScanPro/adscan) | pipx | AD enumeration / attack CLI |
| [Poetry](https://github.com/python-poetry/poetry) | Official installer | Python package manager |
| [NetExec](https://github.com/Pennyw0rth/NetExec) | git + pipx | Network protocol execution |
| [Coercer](https://github.com/p0dalirius/Coercer) | git | Authentication coercion |
| [RelayKing-Depth](https://github.com/depthsecurity/RelayKing-Depth) | git + pipx | NTLM & Kerberos relay detection |
| [SCCMHunter](https://github.com/garrettfoster13/sccmhunter) | git + pipx | SCCM/MECM asset discovery and attack |
| [pxethiefy](https://github.com/csandker/pxethiefy) | git + venv | PXE boot media discovery (SCCM) |
| [snafflepy](https://github.com/cisagov/snafflepy) | git + venv | Snaffler port for Linux: LDAP computer discovery + SMB share/file hunting |
| [Snaffler](https://github.com/SnaffCon/Snaffler) | GitHub release (`Snaffler.exe`) | Windows share/file hunting binary, downloaded for transfer (not run by the script) |
| [PowerShell](https://github.com/PowerShell/PowerShell) | GitHub `.deb` | Shell runtime (Locksmith2 dependency) |
| [Locksmith2](https://github.com/jakehildreth/Locksmith2) | PSGallery | AD CS misconfiguration auditor (ESC1-ESC16) |
| [Claude Code](https://code.claude.com/docs) | Signed apt repo | Operator AI assistant (CLI) |
| [Claude-Red](https://github.com/SnailSploit/Claude-Red) | git → skills | Offensive Claude Code skill pack |
| [Anthropic-Cybersecurity-Skills](https://github.com/mukul975/Anthropic-Cybersecurity-Skills) | git → skills | 754-skill cybersecurity skill pack |
| [CyberStrike](https://github.com/CyberStrikeus/CyberStrike) | npm | Autonomous AI-driven pentest agent platform |

## Notes

- **RustHound-CE** requires `clang`, `libclang-dev`, `libgssapi-krb5-2`, and `libsasl2-modules-gssapi-mit` for its GSSAPI/Kerberos bindgen step — these are now included in base deps. The cargo build also pulls crates on first run and can take several minutes
- **Rust** is installed via `rustup` rather than apt, as the apt package is too old for several required crates
- **Certipy** and **NetExec** are installed via `pipx` to avoid conflicts with Kali's system-managed Python packages. The script sets `PIPX_HOME=/opt/pipx` and `PIPX_BIN_DIR=/usr/local/bin`, so the `pipx` apps land on PATH for root and under `sudo` (previously they went to `/root/.local/bin`, which new shells did not have on PATH). NetExec re-runs use `pipx install --force` against the local clone rather than `pipx upgrade`
- **NetExec latest build**: the step purges Kali's `netexec` apt package and deletes any leftover `/usr/bin/nxc`, pulls the latest commit into `/opt/NetExec` (fast-forward only; a failed pull is warned and the existing checkout is kept), then runs `pipx install --force`. It fails unless `nxc` and `netexec` both resolve to `/usr/local/bin`, so the Kali binary cannot be the one you run
- **NetExec version check**: after install the script reads the installed version and compares its commit suffix (e.g. `1.5.1+713.d640fb78`) to the HEAD of `/opt/NetExec`. It warns if the build is stale or has no commit suffix, which means it is the Kali package rather than the repo build
- **NetExec results**: `NXC_PATH` is set to `/opt/results/nxc` in root's `.bashrc`/`.zshrc` (NetExec keeps workspaces, logs and the credential DB there, default `~/.nxc`). The directory is `chmod 700`. Existing `/root/.nxc` is left in place with a warning so you can copy anything you want to keep. Other users are unchanged
- **Responder** reads `Responder.conf` from its own clone directory. If an existing `/opt/Responder` checkout is missing the `WPADScript` option (startup fails with `NoOptionError`), the script backs up the file to `Responder.conf.bak` and restores the upstream copy from `origin/HEAD`
- **Impacket** is installed with `pip3` from the GitHub checkout. After install the script checks that `MSSQLRelayServer` imports, which confirms the GitHub copy is the one being used. If the check fails, an older Kali `python3-impacket` is shadowing it; the script prints the `apt purge` command to fix that, then the step can be re-run
- **ADscan** is installed via `pipx install adscan`; the script then runs `adscan install` to download BloodHound CE and configure the tool workspace — this step requires internet access and may take several minutes. If `adscan install` fails during automated setup, run it manually after the script completes
- **pxethiefy** is installed in an isolated Python venv at `/opt/pxethiefy/venv`; the `/usr/local/bin/pxethiefy` wrapper auto-escalates via `sudo` when not running as root, as the tool requires raw packet access (`CAP_NET_RAW`)
- **snafflepy** (CISA's Python port of Snaffler) clones to `/opt/snafflepy` and runs from an isolated venv at `/opt/snafflepy/venv`; the `/usr/local/bin/snafflepy` wrapper forwards arguments to `snaffler.py`. Runs from Linux over SMB/LDAP, so no Windows host is needed. Example: `snafflepy <ip> -u <user> -p <pass> -d <domain> -v`
- **Snaffler** (SnaffCon, .NET) is a Windows tool, so the script only downloads the latest `Snaffler.exe` release asset to `/opt/Snaffler/Snaffler.exe` for transfer to a foothold. Nothing is executed on the Kali box. The release tag is recorded in `/opt/Snaffler/VERSION`; re-runs skip the download when it matches the latest release
- **Locksmith2** is a PowerShell module, so the script first installs the latest **PowerShell** `.deb` (arch-detected: `amd64`/`arm64`) from the official PowerShell GitHub releases, then installs Locksmith2 from the PowerShell Gallery (`AllUsers` scope). The `/usr/local/bin/locksmith2` wrapper imports the module and runs `Invoke-Locksmith2`, forwarding any arguments
- **ADhammer** is a pure-Rust static binary — built with `cargo build --release` (same rustup toolchain as RustHound-CE, no OpenSSL dependency). Clones to `/opt/adhammer`; binary symlinked to `/usr/local/bin/adhammer`. Covers 41 AD audit checks plus live-validation modules (DCSync, golden/silver tickets, ADCS ESC1-ESC15, NTLM relay, RCE). First build pulls crates and can take a few minutes
- **RelayKing-Depth** is installed via `pipx` with a `pip3` fallback; clones to `/opt/RelayKing-Depth`
- **SCCMHunter** is installed via `pipx` with a `pip3` fallback; clones to `/opt/sccmhunter`. See the [wiki](https://github.com/garrettfoster13/sccmhunter/wiki) for module usage (`find`, `show`, `smb`, `http`, `dpapi`, etc.)
- **adnullenum** clones to `/opt/adnullenum`; single script, no separate dependencies beyond impacket (already installed by the Impacket step). Symlinked to `/usr/local/bin/adnullenum`. Usage: `adnullenum <dc-ip> --mode full`
- **CVE-2026-54121 (Certighost)** clones to `/opt/CVE-2026-54121`; the `certighost.py` script requires root (binds ports 389/445 for rogue LDAP/SMB listeners). The script installs `cryptography pyasn1 asn1crypto pycryptodome dnspython` via pip; impacket is already present from the Impacket step. Usage: `sudo certighost -d <domain> -u <user> -p <pass> --dc-ip <dc-ip>`
- **Claude Code** is installed from Anthropic's signed apt repository (`downloads.claude.ai`), not hardcoded into this script. No API key is stored: set `ANTHROPIC_API_KEY` in your environment, or run `claude` to log in interactively, before first use
- **Skill packs** ([Claude-Red](https://github.com/SnailSploit/Claude-Red), [Anthropic-Cybersecurity-Skills](https://github.com/mukul975/Anthropic-Cybersecurity-Skills)) are cloned to `/opt` and every `SKILL.md` directory is flattened into the invoking operator's `~/.claude/skills/` (resolved via `$SUDO_USER`). Each is prefixed by source (`cr-`, `acs-`) so the two collections never clobber each other on disk, and the `~/.claude` tree is chowned back to the operator. Re-running refreshes the deployed copies
- **CyberStrike** is installed globally via `npm install -g @cyberstrike-io/cyberstrike` (Node.js/`npm` are now included in base deps). It needs an API key/subscription from at least one supported AI provider (Anthropic, OpenAI, Google, AWS Bedrock, Azure, etc.) or a local Ollama/LM Studio endpoint — none is stored by this script. Run `cyberstrike` to configure a provider interactively
- If any tool fails, the script continues and reports failures in the final summary
- Re-running the script is safe — existing git repos are pulled rather than re-cloned

## Log

Full output is written to `/var/log/pentest_setup.log` for troubleshooting failed installs.

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| `netexec: command not found` after install | pipx apps went to `/root/.local/bin`, which is not on PATH | Fixed in the script (`PIPX_BIN_DIR=/usr/local/bin`). On an existing box, either re-run the script's NetExec step, or link it: `sudo ln -sf /root/.local/bin/netexec /usr/local/bin/netexec` |
| `nxc` runs the Kali build (`/usr/bin/nxc`) instead of the pipx build | Kali `netexec` package or its binary was left behind | Fixed in the script (purges the package and removes `/usr/bin/nxc`). On an existing box: `sudo apt purge -y netexec && sudo rm -f /usr/bin/nxc`, then re-run the NetExec step |
| `pipx upgrade netexec` → `Unable to parse package spec: /opt/NetExec` | pipx stored the local path as the package spec | `sudo pipx install --force /opt/NetExec` (the script now does this) |
| `pipx install git+https://github.com/Pennyw0rth/NetExec` → "already seems to be installed" | An existing venv is kept unless forced | `pipx install --force ...` |
| `KeyError: 'wpadscript'` / `NoOptionError: No option 'wpadscript' in section: 'HTTP Server'` from Responder | `/opt/Responder/Responder.conf` lacks the `WPADScript` option | Re-run the script (restores the file), or `cd /opt/Responder && git checkout origin/HEAD -- Responder.conf` |
| `ImportError: cannot import name 'MSSQLRelayServer'` from `ntlmrelayx.py` | Kali's older `python3-impacket` in `/usr/lib/python3/dist-packages` is shadowing the GitHub install | `sudo apt purge -y python3-impacket impacket-scripts`, then re-run the Impacket step |

## Legal

This tool is intended for use on authorized engagements only. Ensure you have written permission before deploying on any network or system.
