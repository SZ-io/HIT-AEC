Claude



















Lab1 a report · MD
# Lab 1a Report - SOC Detection & MITRE ATT&CK Mapping
 
**Students**:
 
**Matan Elgrissi**
 
**Or Simovich**
 
## 1. Docker Verification & Dashboard Screenshot
 
Initial finding count from clean baseline: 2
 
<img width="1600" height="839" alt="PHOTO-2026-10-29-11-14-53" src="https://github.com/user-attachments/assets/71d75189-5b39-44bb-880d-f4763e6a3aa1" />

## 2. Evidence → Rule → Technique → Tactic
 
| | T1110 | T1046 |
|---|---|---|
| Evidence | Log lines 2-4: three "Failed password for root from 172.18.0.5" records | Log line 5: "Network scan detected from 172.18.0.7 against ports 22,80,443,3306" |
| Rule | 3 or more "Failed password" lines from the same source IP (threshold = 3, no time window) | Any line containing "network scan detected" |
| Technique | T1110 - Brute Force | T1046 - Network Service Discovery |
| Tactic | Credential Access | Discovery |
| Uncertainty | The log does not show whether a login later succeeded, and three failures over any period could be a user mistyping a password. | The rule only matches text in the log. It does not analyze traffic, so a false or injected log line would trigger it too. |
 
## 3. Controlled Experiments
 
### C1
- Change:
- Predicted:
- Actual:
- Explanation:
### C2
- Change:
- Predicted:
- Actual:
- Explanation:
### C3
- Change:
- Predicted:
- Actual:
- Explanation:
### C4
- Change:
- Predicted:
- Actual:
- Explanation:
## 4. Simulator Observation
 
## 5. SOC Narrative
 
Claude finished the response
