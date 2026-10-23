# TerminalFix Snapshot — 2026-10-23 UTC

## 수집 상태
- 페치 성공: [TerminalFix campaign deploys a reverse tunnel through multistage intrusion](https://www.microsoft.com/en-us/security/blog/2026/08/28/terminalfix-campaign-deploys-reverse-tunnel-through-multistage-intrusion/)
- 페치 차단: mishcon[.]com
- seed: 11 / 신규 검증: 0 / 미검증: 0 (신규 미검증 후보 없음; 기존 표류 항목 2건 유지)

## 캠페인 개요
변동 없음. 가짜 Cloudflare Turnstile CAPTCHA가 클립보드에 PowerShell 명령을 심고, 이를 Windows Terminal/PowerShell에 붙여넣도록 유도하는 ClickFix 계열 캠페인. ZIP 다운로드 → DLL 사이드로딩(`LockScreenContentServer.exe` + `dui70.dll`) → PNG 스테가노그래피 2차 페이로드 → 지속성(Run 키 + 예약 작업) → 내부 정찰 → 비동기 명령 루프 → `gitnow[.]dev`로의 TLS/WebSocket 역방향 터널로 이어짐. Microsoft는 분석한 체인에서 권한 상승·보안 통제 변조·탈취·랜섬웨어 배포를 관찰하지 못했다고 명시.

## 공격 체인
1. 가짜 CAPTCHA 유도 — 손상된 정상 사이트(`linked-log[.]com`)에 가짜 Cloudflare Turnstile 화면이 뜨고, 체크박스 클릭 시 악성 PowerShell 명령이 클립보드에 복사됨 (T1204.004)
2. 초기 페이로드 실행 — 클립보드 명령이 ZIP(`verify_pkg.zip`)을 다운로드해 `C:\ProgramData\<16-hex>\`에 압축 해제 후 `1.bat` 실행
3. DLL 사이드로딩 — 정상 서명된 `LockScreenContentServer.exe`가 같은 디렉터리의 악성 `dui70.dll`을 로드 (정상 위치는 `C:\Windows\SystemApps`)
4. 스테가노그래피 페이로드 — 3개 PNG 이미지의 RGBA 픽셀 데이터에서 후속 실행 파일/DLL을 재조립
5. 지속성 — HKCU Run 키 및 약 60분 주기 예약 작업이 `LockScreenContentServer.exe`를 재실행; 페이로드 폴더는 숨김+시스템 속성
6. 내부 정찰 — `nltest`, `net group`, ADSI 질의(도메인 트러스트, Domain Admins, AD 사용자/컴퓨터), 내부 서버 핑 스윕
7. 비동기 명령 실행 — PowerShell 파일감시 루프가 텍스트 파일의 명령을 `Invoke-Expression`으로 실행하고 결과를 다른 파일에 기록
8. 역방향 터널 — 내장형 Python 3.14.5 런타임의 `pythonw.exe`가 `client.py`(`--server`, `--uuid`, `cert.pem`)를 실행, `gitnow[.]dev:443`로 TLS/WebSocket 연결하여 내부 IPv4/IPv6/호스트명으로 임의 TCP 중계

## 공격자 인프라 (차단 대상) (3)
- gitnow[.]dev — 역방향 터널 C2, TCP/443 — seed
- bestsocialmedianewspapper[.]com — 스테가노그래피 페이로드 전달(PNG RGBA 임베딩) — seed
- offlineupdater[.]com — 페이로드 전달 failover — seed

## 손상된 정상 사이트 (차단 금지, 모니터링만) (1)
- linked-log[.]com — 가짜 Cloudflare Turnstile CAPTCHA 호스팅, 피해자 사이트 — seed

## 파일 해시 (11)
### SHA-256
- 18c2090e8a0ae0568af9b87e59eaf8270f23d2909600ed9db91a9444fd8b278f — 초기 ZIP (verify_pkg.zip) — seed
- b8d107800403b9197e5b7609ceacd8e4cac1b0f9a1d156e6dacd6c3f7794b36a — 역방향 터널 client.py — seed
- ba77feed86bcda49308746421bdc684a432dd5d68c363975b2a3c6831bda3f07 — 악성 dui70.dll — seed
- 026478003fe354134c03acf6890e7d3b153ba08a836eca42350db48f213872ab — 악성 dui70.dll — seed
- 032b529fac61e550f5dc9489686f519b82d64625fa05a8d9ecf8ba8be9b2ad22 — 악성 dui70.dll — seed
- df8221a933b38284ebdcb8bffc2df62123c9f5b5f421dd0b070e13e668b3eabf — 악성 dui70.dll — seed
- eb1b4be34d05b394fb74efdeb95faecd1d1963be6ecc1b9db2b4757b491f01f0 — 악성 dui70.dll — seed
- 5d43abf5c36ea203176d3300ff14af27b4be81810ad2679b3a62b255e3d6e1c8 — 악성 dui70.dll — seed
- 9a7b4dcd51d9251c177d323d6aaecdfc86674f69bc1af048dc872926d22aaa24 — 악성 dui70.dll — seed
- 342df92235c9dec81203b837addaa38bb85b64b4a48fe71b5303ca86d991991e — 악성 dui70.dll — seed
- ededeacf30e493dd632d477fe770ba419aa2848f685ea049381a0a8d2cc3e84d — 악성 dui70.dll — seed

## 호스트 IOC · 행위
전날과 동일 (변동 없음). 상세는 `host-iocs.md` 참조: ProgramData 16진수 스테이징 폴더, `1.bat`, `LockScreenContentServer.exe`/`dui70.dll` 사이드로딩, HKCU Run 키 및 예약 작업(`LockScreenContentServer_MuODG5yBM`), 다국어 정찰 스크립트, `client.py` 역방향 터널(7바이트 프레이밍, User-Agent 로테이션, `MSG_SHUTDOWN`).

## 이번 실행 변경사항
- 신규 인프라: 없음
- 신규 해시: 없음
- 승격(seed→검증): 없음
- 폐기(sinkhole/takedown): 없음
- 신규 보고서: 없음 (Microsoft 원본 블로그 재확인 성공, 모든 IOC 변동 없이 재확인됨; mishcon[.]com 신규 기사 1건 페치 차단으로 내용 미확인)

## 인접 캠페인 (참고)
- mishcon[.]com "Picture this: ClickFix hides malware in an image" (2026-10-15 추정) — 페치 차단으로 TerminalFix와의 연관성 미확인, 별도 PNG 스테가노그래피 ClickFix 변종으로 추정
- 기존 표류 항목 유지(신규 검색 없음): Berlin/BSI/Rhysida 랜섬웨어 내러티브(미검증, 2026-09-16부터 연속 미확인), Sophos의 STAC4924/Lorem Ipsum Loader/GOLD VICTOR 귀속(미검증, sophos[.]com 4회 연속 DNS 차단)
- Windows Update 가짜 화면 + .NET 스테가노그래피 로더(LummaC2/Rhadamanthys) — 별도의 무관한 ClickFix 변종으로 계속 분류

## 차단 운영 포맷 (복붙용)

### Domain blocklist (un-defanged, one per line — attacker infrastructure only)
```
bestsocialmedianewspapper.com
gitnow.dev
offlineupdater.com
```

## 출처
- [TerminalFix campaign deploys a reverse tunnel through multistage intrusion](https://www.microsoft.com/en-us/security/blog/2026/08/28/terminalfix-campaign-deploys-reverse-tunnel-through-multistage-intrusion/) — 2026-08-28 — fetched (재확인, 변동 없음)
- Picture this: ClickFix hides malware in an image (mishcon[.]com) — 2026-10-15(추정) — blocked
