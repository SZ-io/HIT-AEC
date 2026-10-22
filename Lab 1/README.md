# Mitigating the Axios npm supply chain compromise

## Source
 
- **Title:** Mitigating the Axios npm supply chain compromise
- **Publisher:** Microsoft Security Blog (Microsoft Threat Intelligence and Microsoft Defender Security Research Team)
- **Date:** April 1, 2026 (last modified April 16, 2026)
- **URL:** https://www.microsoft.com/en-us/security/blog/2026/04/01/mitigating-the-axios-npm-supply-chain-compromise/

## <u>students</u>
- Matan Elgrissi
- Or Simovich
# Behaviors supported by the report:

## Behaviors
 
| # | Behavior |
|---|----------|
| 1 | Malicious Axios versions (1.14.1, 0.30.4) are published with a hidden extra dependency, `plain-crypto-js` 4.2.1 |
| 2 | The dependency's install script (`node setup.js`) runs automatically on install and identifies the operating system |
| 3 | The script contacts the attacker's server (`sfrclak[.]com:8000`) and downloads a payload for that OS |
| 4 | Windows: a hidden PowerShell payload runs and a Run registry key (`MicrosoftUpdate`) is created for persistence |
| 5 | The Windows tool collects host information, beacons to the server and can run commands |
| 6 | The loader deletes its own files and swaps the manifest so the install hook disappears |
 
 
## Attack flow
 
 
```mermaid
flowchart TD
    B1["1. Malicious Axios published<br/>with hidden dependency<br/>(Initial access)"]
    B2["2. Install script runs automatically<br/>and identifies the OS<br/>(Execution / discovery)"]
    B3["3. Contacts sfrclak[.]com:8000<br/>and downloads payload for the OS<br/>(Execution / C2)"]
    B4["4. Windows: hidden PowerShell payload<br/>and Run key persistence<br/>(Execution / persistence)"]
    B5["5. Collects host info, beacons,<br/>can run commands<br/>(Later activity)"]
    B6["6. Deletes its own files and<br/>swaps the manifest<br/>(Cleanup)"]
    B1 --> B2 --> B3 --> B4 --> B5 --> B6
```
 
# ATT&CK Tactics and Techniques (Axios npm compromise)
 
**1. Malicious Axios published with hidden dependency:**
- Initial Access: [T1195.001](https://attack.mitre.org/techniques/T1195/001/) - Supply Chain Compromise: Compromise Software Dependencies and Development Tools

**2. Install script runs automatically and identifies the OS:**
- Execution: [T1059.007](https://attack.mitre.org/techniques/T1059/007/) - Command and Scripting Interpreter: JavaScript
- Discovery: [T1082](https://attack.mitre.org/techniques/T1082/) - System Information Discovery

**3. Contacts sfrclak[.]com:8000 and downloads payload for the OS:**
- Command and Control: [T1071.001](https://attack.mitre.org/techniques/T1071/001/) - Application Layer Protocol: Web Protocols
- Command and Control: [T1105](https://attack.mitre.org/techniques/T1105/) - Ingress Tool Transfer
- Command and Control: [T1571](https://attack.mitre.org/techniques/T1571/) - Non-Standard Port (lower confidence)

**4. Windows: hidden PowerShell payload and Run key persistence:**
- Execution: [T1059.005](https://attack.mitre.org/techniques/T1059/005/) - Command and Scripting Interpreter: Visual Basic
- Execution: [T1059.001](https://attack.mitre.org/techniques/T1059/001/) - Command and Scripting Interpreter: PowerShell
- Stealth: [T1564.003](https://attack.mitre.org/techniques/T1564/003/) - Hide Artifacts: Hidden Window
- Stealth: [T1036.003](https://attack.mitre.org/techniques/T1036/003/) - Masquerading: Rename Legitimate Utilities
- Persistence: [T1547.001](https://attack.mitre.org/techniques/T1547/001/) - Boot or Logon Autostart Execution: Registry Run Keys / Startup Folder

**5. Collects host info, beacons, can run commands:**
- Command and Control: [T1071.001](https://attack.mitre.org/techniques/T1071/001/) - Application Layer Protocol: Web Protocols
- Discovery: [T1082](https://attack.mitre.org/techniques/T1082/) - System Information Discovery
- Execution: [T1059.001](https://attack.mitre.org/techniques/T1059/001/) - Command and Scripting Interpreter: PowerShell
- Stealth: [T1620](https://attack.mitre.org/techniques/T1620/) - Reflective Code Loading (lower confidence; could be T1055 Process Injection)

**6. Deletes its own files and swaps the manifest:**
- Stealth: [T1070.004](https://attack.mitre.org/techniques/T1070/004/) - Indicator Removal: File Deletion
