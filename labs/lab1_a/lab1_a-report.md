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
- Change: Deleted the 09:02:11 "Failed password" line from 172.18.0.5, leaving two failed-password lines from that IP.
- Predicted: T1110 disappears, T1046 remains, total findings becomes 1.
- Actual: T1110 disappeared, T1046 remained, total findings was 1 (High 0, Medium 1).
- Explanation: The rule needs 3 failures from the same source IP. With only 2, the threshold isn't reached. The T1046 rule doesn't depend on the failed-password lines, so it was unaffected.

<img width="733" height="736" alt="image" src="https://github.com/user-attachments/assets/d01c8ae1-e2c2-499e-8c31-3b41a1599648" />

### C2
- Change: Changed the three "Failed password" lines to use different source IPs (172.18.0.5, 172.18.0.6, 172.18.0.8).
- Predicted: No T1110 finding.
- Actual: T1110 did not appear; only the T1046 finding remained (total 1).
- Explanation: The detector counts failures per source IP. Each IP had only 1 failure, so none reached the threshold of 3.

<img width="1495" height="662" alt="image" src="https://github.com/user-attachments/assets/066750bf-fddc-412b-a538-b6c134b562bb" />

### C3
- Change: Appended three "Failed password" lines from a new source IP, 192.0.2.55, to the end of the baseline log.
- Predicted: A new T1110 finding for 192.0.2.55.
- Actual: A new High-severity T1110 finding appeared for 192.0.2.55. Total findings went to 3: T1046 from 172.18.0.7, T1110 from 172.18.0.5 and T1110 from 192.0.2.55.
- Explanation: The detector counts failures per source IP. 192.0.2.55 had 3 failures, which reaches the threshold, so it gets its own finding. The original 172.18.0.5 finding is unaffected.

<img width="1498" height="817" alt="image" src="https://github.com/user-attachments/assets/948bad8c-bdca-4e81-9d5d-66344eeb9070" />

### C4
- Change: Appended one line to the log: "Network scan detected from 192.0.2.77 against ports 22,80,443".
- Predicted: A new T1046 finding for 192.0.2.77.
- Actual: A new Medium-severity T1046 finding appeared for 192.0.2.77. Total findings went to 4 (2 High T1110, 2 Medium T1046).
- Explanation: The T1046 rule creates one finding for every line containing "network scan detected", whatever the IP. The detector doesn't check for a real scan, only the text in the log line.

<img width="1502" height="851" alt="image" src="https://github.com/user-attachments/assets/5ed164f8-e3a6-42c3-89fa-bd769c1bbf34" />

## 4. Simulator Observation
 
## 5. SOC Narrative
 
Claude finished the response
