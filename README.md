# BTCMP-16 — Disk & Memory Forensics CTF

Combined disk- and memory-forensics lab environment. Two analyst
workstations share one CIFS/SMB evidence store.

## Topology

| Host   | Role                                          | Network       | CIDR            | IP          |
|--------|-----------------------------------------------|---------------|-----------------|-------------|
| xavm   | Xubuntu Analyst VM (Xubuntu Noble)            | access-switch | 10.10.10.0/24   | 10.10.10.5  |
| wavm   | Windows Analyst VM (Windows 10)               | access-switch | 10.10.10.0/24   | 10.10.10.6  |
| router | Network Router (Debian 12)                    | access-switch | 10.10.10.0/24   | 10.10.10.1  |

## Machines

### XAVM — Xubuntu Analyst VM
Users: `LocalAdmin` / `Password123!` (admin, passwordless sudo) and
`user` / `btcmp16@admin` (autologin XFCE).

Provisions:
- Base utilities (`unzip`, `curl`, `ca-certificates`, `terminator`)
- LightDM autologin, `openssh-server`, password login enabled for `user` / `LocalAdmin`
- Disk-forensics toolkit: `sleuthkit`, `autopsy`, `ewf-tools`, `libewf-dev`,
  `libhivex-bin`, `guestmount`, `foremost`, `bulk-extractor`, `regripper`
- Memory-forensics toolkit: Python 2.7.18 (built from source) + Volatility 2
  in `/home/LocalAdmin/vol2-venv`; Volatility 3 in `/home/LocalAdmin/vol3-venv`
- `cifs-utils` and the shared evidence store mounted read-only at `/mnt/btcmp16`

### WAVM — Windows Analyst VM
User: `user` / `Password123` (RDP + local admin).

Provisions:
- Chocolatey + `7zip`, `git`, `strawberryperl`, `dotnet-9.0-desktopruntime`
- Autopsy 4.23.1 (direct MSI)
- DiskInternals Linux Reader (silent NSIS)
- FTK Imager 8.3.0.27 (checksum-pinned; CodeMeter prerequisite auto-installed)
- Registry Explorer (Eric Zimmerman, net9)
- RegRipper 4.0 (git clone)
- Volatility 3 (pip)
- Windows Defender exclusions for `C:\Cases\BTCMP16` and `C:\Program Files\AccessData`
- Desktop shortcuts for every tool
- Shared evidence store mapped to `Z:\` via loopback portproxy
  (127.0.0.2:445 → cybersec.upatras.gr:8008)

## Shared evidence store

Both VMs mount `//cybersec.upatras.gr/btcmp16` (SMB 3.1.1, port 8008)
using the credentials in `provisioning/group_vars/all.yml`.

- XAVM: `/mnt/btcmp16` (cifs, read-only)
- WAVM: `Z:\` (mapped via portproxy — Windows built-in SMB client only
  speaks 445, so a loopback portproxy forwards 127.0.0.2:445 → the share's
  actual port 8008)

## Layout

```
.
├── topology.yml
├── variables.yml
└── provisioning/
    ├── playbook.yml
    ├── requirements.yml
    └── group_vars/
        ├── all.yml
        └── windows_hosts.yml
```
