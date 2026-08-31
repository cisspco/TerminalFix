# TerminalFix — Host IOCs

Cumulative. Provenance noted per item.

## File paths
- `C:\ProgramData\<16-hex>\` — payload extraction/staging directory (e.g. observed `C:\ProgramData\f47f2a8c21c9df4e\`); set hidden+system attributes (`attrib +h +s`) — verified
- `1.bat` — batch launcher dropped in the ProgramData staging directory — verified
- `LockScreenContentServer.exe` — legitimately-signed Microsoft binary, sideloading vector. Legitimate location is `C:\Windows\SystemApps`; execution from any other path is the signal — verified
- `dui70.dll` — malicious DLL sideloaded from the same directory as `LockScreenContentServer.exe` — verified

## Registry
- `HKCU\Software\Microsoft\Windows\CurrentVersion\Run\LockScreenContentServer_MuODG5yBM` — Run key persistence, re-launches `LockScreenContentServer.exe` — verified

## Scheduled tasks
- Task name `LockScreenContentServer_MuODG5yBM` — re-executes `LockScreenContentServer.exe` roughly every 60 minutes — verified

## Steganography
- Payload reassembled from RGBA pixel data of 3 PNG images fetched from attacker infrastructure — seed

## Recon commands
- `nltest` — domain trust enumeration — seed
- `net group` — group/membership enumeration — seed
- ADSI queries — domain trusts, Domain Admins, AD users/computers — seed
- Ping sweeps of suspected internal servers — seed

## Async tasking
- PowerShell file-watcher loop: reads commands from a text file, executes via `Invoke-Expression`, writes results to a second text file — seed

## Reverse tunnel
- `pythonw.exe` running `client.py` with `--server`, `--uuid`, `cert.pem` arguments — verified
- TLS/WebSocket connection to `gitnow.dev:443`, relaying arbitrary TCP to internal IPv4/IPv6/hostnames reachable from the host — verified
- Delivered via official embeddable Python 3.14.5 runtime — seed

## Not observed
Microsoft's analysis did not observe privilege escalation, security-control tampering, data exfiltration, or ransomware deployment in this chain. Do not assert these unless a fetched source in a future run reports them.
