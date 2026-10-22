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
 
| # | Behavior | Status |
|---|----------|--------|
| 1 | Malicious Axios versions (1.14.1, 0.30.4) are published with a hidden extra dependency, `plain-crypto-js` 4.2.1 | Observed |
| 2 | The dependency's install script (`node setup.js`) runs automatically on install and identifies the operating system | Observed |
| 3 | The script contacts the attacker's server (`sfrclak[.]com:8000`) and downloads a payload for that OS | Observed |
| 4 | Windows: a hidden PowerShell payload runs and a Run registry key (`MicrosoftUpdate`) is created for persistence | Observed |
| 5 | The Windows tool collects host information, beacons to the server and can run commands | Observed |
| 6 | The loader deletes its own files and swaps the manifest so the install hook disappears | Observed |

## Attack flow
 
```mermaid
flowchart LR
    A["Initial access<br/>Developer runs npm install/update<br/>and pulls malicious Axios<br/>(Observed)"]
    B["Execution / discovery<br/>Install script runs automatically<br/>and checks the OS<br/>(Observed)"]
    C["C2 contact<br/>sfrclak[.]com:8000<br/>(Observed)"]
    D["Payload by OS<br/>macOS / Windows / Linux<br/>(Observed)"]
    E["Windows only<br/>PowerShell RAT + Run key<br/>(Observed)"]
    F["Later activity<br/>Beacons, runs commands<br/>(Observed)"]
    G["Cleanup<br/>Deletes setup.js and swaps manifest<br/>(Observed)"]
    A --> B --> C --> D --> E --> F --> G
```
 
