## Mitigating the Axios npm supply chain compromise
# Source
 
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
 
