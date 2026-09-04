# TerminalFix

Defensive IOC tracker for the **TerminalFix** campaign — a ClickFix-derivative social-engineering intrusion chain (fake Cloudflare Turnstile CAPTCHA → clipboard-pasted PowerShell → DLL sideloading → PNG steganography → reverse WebSocket tunnel to `gitnow[.]dev`). Source: Microsoft Threat Intelligence, 2026-08-28, with daily automated re-checks for new reporting.

**Last updated:** 2026-09-04 00:00 UTC

## Counts
- Attacker infrastructure (blockable): 3
- Compromised third-party sites (do not block): 1
- File hashes: 11
- Unverified infrastructure candidates: 0

## Blocklist
Raw domain list, safe to feed into a firewall/proxy:
`https://raw.githubusercontent.com/cisspco/TerminalFix/main/blocklists/domains.txt`

## ⚠️ Blocking guidance
- **`blocklists/domains.txt` is the ONLY file safe to feed into a firewall.** It contains attacker-owned/attacker-controlled infrastructure only.
- **`blocklists/compromised-sites.txt` must NEVER be blocked.** It lists legitimate third-party sites that were compromised to host the campaign's fake CAPTCHA — these are victims, not attacker infrastructure, and blocking them harms the victim, not the attacker.
- `blocklists/unverified-domains.txt` holds candidate attacker infrastructure seen only in unfetched/unconfirmed reporting — review before acting on it.

## Layout
- `latest.md` — most recent full snapshot report (Korean)
- `snapshots/<date>.md` — dated snapshot history
- `blocklists/domains.txt` — verified attacker infrastructure, firewall-ready
- `blocklists/unverified-domains.txt` — unverified attacker infrastructure candidates
- `blocklists/compromised-sites.txt` — compromised victim sites, monitor only, do not block
- `hashes/sha256.txt` — cumulative SHA-256 IOCs
- `host-iocs.md` — paths, registry keys, scheduled tasks, command-line patterns
- `hunting/defender-xdr.kql` — Microsoft Defender XDR Advanced Hunting detection queries
