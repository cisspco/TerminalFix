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
- `client.py` disables TLS certificate verification (`CERT_NONE`) — verified
- WebSocket upgrade path `/tunnel` on the C2 endpoint — verified
- Custom 7-byte binary framing header (message type + stream ID + length) multiplexes multiple relayed TCP connections over one WebSocket; 8 message types incl. keepalive and `MSG_SHUTDOWN` (calls `os._exit()` to bypass Python cleanup on remote shutdown) — verified
- Rotates 4 realistic browser User-Agent strings (Chrome/Firefox/Safari) per connection; `DEBUG = False`, all logging to stderr for silent operation — verified
- Malicious `dui70.dll` PE timestamp forged to year 2104 — verified
- Initial PowerShell stage prints a fake green-text confirmation, e.g. "I am not a robot – Cloudflare ID: f47f2a8c21c9df4e" (ID matches the ProgramData staging folder name), to sell the fake CAPTCHA — verified

## Reconnaissance — locale support
- Recon scripts include English, Spanish, and German locale variants, suggesting an attempt to operate across systems configured in multiple languages — verified (2026-09-08 re-fetch of Microsoft original post)

## Targeting / attribution
- Microsoft's original post describes targeting broadly as "organizations across multiple industries" and does not attribute the campaign to any specific threat actor or nation-state — verified (re-fetched 2026-09-08, 2026-09-09, 2026-09-11; no new IOCs or attribution added)
- A search-snippet claim that Rapid7 Labs attributed TerminalFix to a North Korea-linked actor targeting South Korea's automotive/media sectors could NOT be verified (Rapid7's site was blocked by egress). Cross-referencing suggests this likely conflates TerminalFix with a separate, unrelated Rapid7-reported DPRK campaign (Ted backdoor / curlRAT via trojanized HAProxy) that happens to target the same sectors/country. Not asserted as fact for TerminalFix — flagged unverified, see 2026-09-08 snapshot for detail.

## Real-world downstream impact (unverified)
- ⚠️ (unverified) Search-snippet reporting (bankinfosecurity.com, heise.de, diesec.com, cyberdise-awareness.com, tech-insider.org, sectank.net — all direct fetches blocked as of 2026-09-18) states Germany's BSI issued a formal security advisory on 2026-09-04 naming TerminalFix as the confirmed initial-access vector into a Berlin state-government network breach, and that the Rhysida ransomware group subsequently leaked roughly 1.44 million files (personnel records, scanned ID cards, sick notes, plaintext password lists) after Berlin's mayor refused to pay a ransom. This does not contradict Microsoft's "not observed" finding below — that statement is scoped to the specific chain Microsoft analyzed; this would be a separate real-world incident. Not promoted past unverified pending a successful direct fetch.
- ⚠️ (unverified, 2026-09-19 re-check) Search snippets now surface an apparent official BSI advisory PDF (`bsi.bund.de/SharedDocs/Cybersicherheitswarnungen/DE/2026/2026-287419-1032.pdf`, titled "Deutsche Institutionen über TerminalFix-Kampagne kompromittiert") and give more specific figures: Rhysida allegedly exfiltrated ~5.79TB from two Berlin Senate departments (Building and Transport) between 2026-08-07 and 2026-08-12, demanded 30 BTC (~€2M), and threatened to leak plaintext passwords, court documents, and 46,500+ state contracts; Rhysida is attributed to the financially-motivated "Vice Spider" group (aka Vice Society, White Nefas, White Hekate, DEV-0832, Vanilla Tempest). These figures conflict with the 2026-09-18 snippet's "1.44 million files" claim, which is itself a reliability red flag for the secondary reporting. Direct fetches of the BSI PDF and every outlet carrying this story (bsi.bund.de, heise.de, rhisac.org, scworld.com, sectank.net, bankinfosecurity.com, tech-insider.org, adminbyrequest.com) were all proxy-blocked again on 2026-09-19. Microsoft's own blog post, re-fetched successfully this run, contains no mention of Berlin, BSI, or Rhysida. Remains unverified.
- ⚠️ (unverified, 2026-09-20 re-check) Same Berlin Senate / BSI / Rhysida narrative continues to surface in new secondary-reporting titles this run (cyberdise-awareness.com "TerminalFix Attack: The Berlin Hack Began With a CAPTCHA", diesec.com "TerminalFix ClickFix Attack Campaign Hits Germany"), but both direct fetches were proxy-blocked, as were malwarebytes.com, rhisac.org, theregister.com, cybersecuritynews.com, and redmondmag.com. No outlet carrying this story has been successfully fetched across five consecutive daily runs (09-16 through 09-20). Microsoft's own blog, re-fetched successfully again this run, still contains no mention of Berlin, BSI, or Rhysida. Remains unverified; treat as a standing item, not a new development, until a direct fetch succeeds.

## Not observed
Microsoft's analysis did not observe privilege escalation, security-control tampering, data exfiltration, or ransomware deployment in the chain it analyzed. Do not assert these unless a fetched source in a future run reports them. (See "Real-world downstream impact" above for an unverified claim of a separate incident involving ransomware and exfiltration.)
