# The Hood - DFIR Writeup

## Scope
Offline analysis of the provided Windows image at `D:\ctf2\The-Hood\The Hood\C` to answer 17 challenge questions about the intrusion and data theft.

## Data Sources
- Event logs: `D:\ctf2\The-Hood\The Hood\C\Windows\System32\winevt\Logs\Microsoft-Windows-Storsvc%4Diagnostic.evtx`
- Notifications DB: `D:\ctf2\The-Hood\The Hood\C\Users\a1l4m\AppData\Local\Microsoft\Windows\Notifications\wpndatabase.db`
- Prefetch: `D:\ctf2\The-Hood\The Hood\C\Windows\Prefetch\`
- USN journal: `D:\ctf2\The-Hood\The Hood\C\$Extend\$J`
- MFT: `D:\ctf2\The-Hood\The Hood\C\$MFT`
- Download cache: `D:\ctf2\The-Hood\The Hood\C\Windows\System32\config\systemprofile\AppData\LocalLow\Microsoft\CryptnetUrlCache\Content\965B295F92685B983726E076B583D923`
- Extracted exfil tool: `D:\ctf2\extracted_tools\Deep Inside.exe`
- Decoded stage artifacts: `D:\ctf2\decoded_stage.ps1`, `D:\ctf2\shellcode_disasm.txt`

## Findings and Evidence
### USB device identification (Q1-3)
- The Storsvc Diagnostic event XML contains the device serial, vendor, and product:
  - `SerialNumber=UM2I126E`
  - `VendorId=JetFlash`, `ProductId=Transcend 8GB`
  - Source: `D:\ctf2\The-Hood\The Hood\C\Windows\System32\winevt\Logs\Microsoft-Windows-Storsvc%4Diagnostic.evtx`
- The friendly label for the removable drive appears in the notification database:
  - `OMKALALA (F:)`
  - Source: `D:\ctf2\The-Hood\The Hood\C\Users\a1l4m\AppData\Local\Microsoft\Windows\Notifications\wpndatabase.db`

### Intrusion window (Q4)
- Correlated storage activity around the USB connection with user activity to bound the session:
  - Start: `2024-12-10 21:59:52`
  - End: `2024-12-10 22:05:45`

### Recon and exploitation (Q5-8)
- Recon app: Task Manager confirmed by prefetch:
  - `TASKMGR.EXE-4C8500BA.pf`
  - Source: `D:\ctf2\The-Hood\The Hood\C\Windows\Prefetch\TASKMGR.EXE-4C8500BA.pf`
- Vulnerability: `CVE-2024-34329`
- Exploit file SHA1: `7ba477a58eb546b6d3cac3a86633b531ba82fa50`
- Technique mapped to DLL search order hijacking: `T1574.002`

### Defense evasion (Q9)
- Logging suppression executed from a script named:
  - `svc1D3C.ps1`

### Payload download and persistence (Q10, Q14-15)
- Staging payloads retrieved from the storage C2 at:
  - `3.75.217.26:8080`
- `tools.7z` cached in CryptnetUrlCache; SHA256:
  - `0905089bb59887880312af06c769cebd967ffa7d2f652fe397ee972ddbed3d25`
  - Source: `D:\ctf2\The-Hood\The Hood\C\Windows\System32\config\systemprofile\AppData\LocalLow\Microsoft\CryptnetUrlCache\Content\965B295F92685B983726E076B583D923`
- Exfil tool last execution time from prefetch:
  - `2024-12-11 04:42:35`
  - Source: `D:\ctf2\The-Hood\The Hood\C\Windows\Prefetch\DEEP INSIDE.EXE-1B0D20D6.pf`

### C2 shell activity (Q11-13)
- Shell start time: `2024-12-11 04:01:41`
- C2 shell IP:port: `3.121.196.122:55099`
- Command used to confirm access: `whoami`
- Supporting decode path: `D:\ctf2\decoded_stage.ps1` and `D:\ctf2\shellcode_disasm.txt`

### Exfiltration (Q16-17)
- Static analysis of `Deep Inside.exe` plus USN evidence shows:
  - The tool stages a zip in `%TEMP%` and then builds a PNG payload.
  - The final exfiltrated file name is `Would you lose.png`.
  - USN evidence shows creation of `Exfiltrated_data.zip` followed by `Would you lose.png`, then zip deletion.
  - Source: `D:\ctf2\The-Hood\The Hood\C\$Extend\$J`
- MFT confirms the PNG path under user temp:
  - Source: `D:\ctf2\The-Hood\The Hood\C\$MFT`
- Exfiltrated file list (alphabetical) from USN journal at the exfil timestamp:
  - `important.txt-Meetings.txt-reminders.txt-research.txt-Stand_Proud_You_Are_Strong.png-tasks.txt-todolist.txt`

## Answer Key
1. UM2I126E
2. Transcend
3. OMKALALA
4. 2024-12-10 21:59:52_2024-12-10 22:05:45
5. TASKMGR.EXE
6. CVE-2024-34329
7. 7ba477a58eb546b6d3cac3a86633b531ba82fa50
8. T1574.002
9. svc1D3C.ps1
10. 3.75.217.26:8080
11. 2024-12-11 04:01:41
12. 3.121.196.122:55099
13. whoami
14. 0905089bb59887880312af06c769cebd967ffa7d2f652fe397ee972ddbed3d25
15. 2024-12-11 04:42:35
16. Would you lose.png
17. important.txt-Meetings.txt-reminders.txt-research.txt-Stand_Proud_You_Are_Strong.png-tasks.txt-todolist.txt

## Flag (challenges3.ctf.sd:33456)
0xL4ugh{c84afabbd76133a117cea1356f1ab6db}
