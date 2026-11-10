# TerminalFix Snapshot — 2026-11-10 UTC

## 수집 상태
- 페치 성공: [TerminalFix campaign deploys a reverse tunnel through multistage intrusion](https://www.microsoft.com/en-us/security/blog/2026/08/28/terminalfix-campaign-deploys-reverse-tunnel-through-multistage-intrusion/)
- 페치 차단: gbhackers[.]com, hendryadrian[.]com, mishcon[.]com, vpncentral[.]com, itsmalware.substack[.]com, detection[.]fyi
- seed: 4 (인프라 3 + 손상 사이트 1) / 신규 검증: 0 (신규 세부사항은 verified로 직접 추가) / 미검증: 0

## 캠페인 개요
변동 없음. 가짜 Cloudflare Turnstile CAPTCHA가 클립보드에 PowerShell 명령을 심고, 이를 Windows Terminal/PowerShell에 붙여넣도록 유도하는 ClickFix 계열 캠페인. ZIP 다운로드 → DLL 사이드로딩(`LockScreenContentServer.exe` + `dui70.dll`) → PNG 스테가노그래피 2차 페이로드 → 지속성(Run 키 + 예약 작업) → 내부 정찰 → 비동기 명령 루프 → `gitnow[.]dev`로의 TLS/WebSocket 역방향 터널로 이어짐. Microsoft 원본 블로그를 이번 실행에서 재확인했고, 어제 기록한 Defender AV 탐지명과는 별개로 Defender for Endpoint **경고(alert) 제목** 4개를 새로 추출했다(신규 인프라/해시는 없음). 권한 상승·보안 통제 변조·탈취·랜섬웨어 배포는 계속 미관찰로 명시됨.

## 공격 체인
1. 가짜 CAPTCHA 유도 — 손상된 정상 사이트(`linked-log[.]com`)에 가짜 Cloudflare Turnstile 화면이 뜨고, 체크박스 클릭 시 악성 PowerShell 명령이 클립보드에 복사됨 (T1204.004)
2. 초기 페이로드 실행 — 클립보드 명령이 ZIP(`verify_pkg.zip`)을 다운로드해 `C:\ProgramData\<16-hex>\`에 압축 해제 후 `1.bat` 실행
3. DLL 사이드로딩 — 정상 서명된 `LockScreenContentServer.exe`가 같은 디렉터리의 악성 `dui70.dll`을 로드 (정상 위치는 `C:\Windows\SystemApps`)
4. 스테가노그래피 페이로드 — 3개 PNG 이미지의 RGBA 픽셀 데이터에서 후속 실행 파일/DLL을 재조립
5. 지속성 — HKCU Run 키 및 약 60분 주기 예약 작업(`LockScreenContentServer_MuODG5yBM`)이 `LockScreenContentServer.exe`를 재실행; 페이로드 폴더는 숨김+시스템 속성
6. 내부 정찰 — `nltest /domain_trusts`, `nltest /dclist:`, `net group "domain admins" /domain`, ADSI 질의(도메인 트러스트, Domain Admins, AD 사용자/컴퓨터), 내부 서버 핑 스윕 (영/스페인/독 다국어 변형)
7. 비동기 명령 실행 — PowerShell 파일감시 루프가 텍스트 파일의 명령을 `Invoke-Expression`으로 실행하고 결과를 다른 파일에 기록
8. 역방향 터널 — 내장형 Python 3.14.5 런타임의 `pythonw.exe`가 `client.py`(`--server`, `--uuid`, `cert.pem`)를 실행, `gitnow[.]dev:443`로 TLS/WebSocket 연결하여 내부 IPv4/IPv6/호스트명으로 임의 TCP 중계

## 공격자 인프라 (차단 대상) (3)
- gitnow[.]dev — 역방향 터널 C2, TCP/443 — seed | verified
- bestsocialmedianewspapper[.]com — 스테가노그래피 페이로드 전달(PNG RGBA 임베딩) — seed | verified
- offlineupdater[.]com — 페이로드 전달 failover — seed | verified

## 손상된 정상 사이트 (차단 금지, 모니터링만) (1)
- linked-log[.]com — 가짜 Cloudflare Turnstile CAPTCHA 호스팅, 피해자 사이트 — seed | verified

## 파일 해시 (11)
### SHA-256
- 18c2090e8a0ae0568af9b87e59eaf8270f23d2909600ed9db91a9444fd8b278f — 초기 ZIP (verify_pkg.zip) — verified
- b8d107800403b9197e5b7609ceacd8e4cac1b0f9a1d156e6dacd6c3f7794b36a — 역방향 터널 client.py — verified
- ba77feed86bcda49308746421bdc684a432dd5d68c363975b2a3c6831bda3f07 — 악성 dui70.dll — verified
- 026478003fe354134c03acf6890e7d3b153ba08a836eca42350db48f213872ab — 악성 dui70.dll — verified
- 032b529fac61e550f5dc9489686f519b82d64625fa05a8d9ecf8ba8be9b2ad22 — 악성 dui70.dll — verified
- df8221a933b38284ebdcb8bffc2df62123c9f5b5f421dd0b070e13e668b3eabf — 악성 dui70.dll — verified
- eb1b4be34d05b394fb74efdeb95faecd1d1963be6ecc1b9db2b4757b491f01f0 — 악성 dui70.dll — verified
- 5d43abf5c36ea203176d3300ff14af27b4be81810ad2679b3a62b255e3d6e1c8 — 악성 dui70.dll — verified
- 9a7b4dcd51d9251c177d323d6aaecdfc86674f69bc1af048dc872926d22aaa24 — 악성 dui70.dll — verified
- 342df92235c9dec81203b837addaa38bb85b64b4a48fe71b5303ca86d991991e — 악성 dui70.dll — verified
- ededeacf30e493dd632d477fe770ba419aa2848f685ea049381a0a8d2cc3e84d — 악성 dui70.dll — verified

## 호스트 IOC · 행위
전날 대비 신규 항목 있음 (verified). 상세는 `host-iocs.md` 참조:
- **신규** — Microsoft Defender for Endpoint 경고(alert) 제목 4종: "Possible ClickFix activity", "An executable file loaded an unexpected DLL file"(`dui70.dll` 사이드로딩), "Suspicious scheduled task"(`LockScreenContentServer_MuODG5yBM`), "Possibly malicious use of proxy or tunneling tool"(`gitnow.dev` 역방향 터널) — 전날 기록한 Defender AV 탐지명과는 별개 항목으로, Microsoft 원본 IOC 표에서 이번 실행에 처음 추출
- 기존 항목 재확인: ProgramData 16진수 스테이징 폴더, `1.bat`, `LockScreenContentServer.exe`/`dui70.dll` 사이드로딩, HKCU Run 키 및 예약 작업, 다국어(영/스페인/독) 정찰 스크립트, `client.py` 역방향 터널, Defender AV 탐지명(`Trojan:Win32/ClickFix.*` 등), 정찰 명령 정확한 문법

## 이번 실행 변경사항
- 신규 인프라: 없음
- 신규 해시: 없음
- 승격(seed→검증): 없음
- 폐기(sinkhole/takedown): 없음
- 신규 보고서: 없음(Microsoft 원본 블로그 재확인 성공; 신규 세부사항은 verified로 host-iocs.md·hunting/defender-xdr.kql에 직접 추가 — Defender for Endpoint 경고 제목 4종). SOC Prime의 다국어 TerminalFix 미러("TerminalFix Campaign Builds Reverse Tunnels for Persistent Access")와 Malwarebytes 프랑스어 번역본이 검색 스니펫에 재등장했으나 신규 정보 없음(두 사이트 모두 직접 페치 시도하지 않음, 기존 차단 목록에 포함).

## 인접 캠페인 (참고)
- Microsoft가 2026년 2월경부터 추적한 별도의 Windows Terminal 악용 ClickFix 캠페인(TechRadar/VPNCentral/GadgetReview) — 가짜 보안 경고가 피해자에게 `Win+X, I`로 Windows Terminal을 직접 열어 명령을 붙여넣도록 유도하며, 최종 페이로드는 Lumma Stealer. 이번 실행 검색에서도 재등장했으나 신규 세부사항 없음. TerminalFix와는 유인 메커니즘·시작 시점·최종 페이로드가 모두 달라 별개 캠페인으로 판단 — 계속 미검증. TerminalFix IOC에 포함하지 않음.
- "Velvet Tempest" 귀속 ClickFix 캠페인(VPNCentral, 2026-02 테스트 환경 보도) — CastleRAT·DonutLoader, 유인/시기/페이로드 모두 다른 별개 캠페인. TerminalFix IOC에 포함하지 않음.
- "ARTEX" / "Storm-3358" — Microsoft 블로그의 관련 게시물 티저에서 발견된 별개 캠페인(2026-11-09 기록). TerminalFix 본문과 무관함 확인됨. 로그만 남김.
- Berlin/BSI/Rhysida 랜섬웨어 내러티브(미검증, 2026-09-16부터 연속 미확인) — 이번 실행에서 별도 검색 없음, 상태 변동 없음
- Sophos STAC4924/Lorem Ipsum Loader/GOLD VICTOR 귀속(미검증, 2026-10-14부터 표류 중) — 이번 실행에서도 재확인 불가 상태 유지
- 별도의 가짜 Windows Update 화면 기반 ClickFix 캠페인(이미지 스테가노그래피로 LummaC2/Rhadamanthys 전달) — 검색 결과에 재등장, 유인/페이로드가 달라 TerminalFix와 무관한 별개 캠페인으로 계속 분류

## 차단 운영 포맷 (복붙용)

### Domain blocklist (un-defanged, one per line — attacker infrastructure only)
```
bestsocialmedianewspapper.com
gitnow.dev
offlineupdater.com
```

## 출처
- [TerminalFix campaign deploys a reverse tunnel through multistage intrusion](https://www.microsoft.com/en-us/security/blog/2026/08/28/terminalfix-campaign-deploys-reverse-tunnel-through-multistage-intrusion/) — 2026-08-28 — fetched (재확인; Defender for Endpoint 경고 제목 4종을 이번 실행에서 처음 추출)
- gbhackers.com "ClickFix Attack Uses Fake Cloudflare Verification" — 날짜 미확인 — blocked
- hendryadrian.com "TerminalFix Uses Fake Cloudflare CAPTCHAs to Deploy Reverse-Tunnel Backdoor" — 날짜 미확인 — blocked
- mishcon.com "From clipboard to compromise: steganographic techniques observed in ClickFix campaign" — 날짜 미확인 — blocked
- vpncentral.com "New ClickFix campaign uses Windows Terminal to launch Lumma Stealer" — 날짜 미확인 — blocked
- itsmalware.substack.com "ClickFix attack uses fake Windows..." — 날짜 미확인 — blocked
- detection.fyi "Potential System DLL Sideloading From Non System Locations" (Sigma rule) — 날짜 미확인 — blocked
