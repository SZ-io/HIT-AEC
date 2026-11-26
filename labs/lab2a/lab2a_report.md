# Lab 2a – Morpheus Lite Agentic SOC
 
## 1. Overview
 
In this lab I ran the Morpheus Lite pipeline end to end:
 
```text
Telemetry -> Redpanda/Kafka -> AI detection -> Threat investigation
          -> RAI policy -> XAI explanation -> Meta-AI review
          -> Human decision -> Audit and research export
```
 
The goal was not to accept the AI recommendation, but to check whether the
evidence really supports it, and then make my own decision.
 
---
 
## 2. Setup and running the system
 
### 2.1 Installation and tests
 
```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -e ".[research,dev]"
python -m pytest -q
```
 
Result: `5 passed`.
 
![Tests passed](photos/01_tests_passed.png)
 
### 2.2 Starting the services
 
| Terminal | Command |
|---|---|
| 1 | `docker compose up -d redpanda console` |
| 2 | `python telemetry_generator.py` |
| 3 | `python morpheus_lite_detector.py` |
| 4 | `python agent_orchestrator.py` |
| 5 | `python -m streamlit run dashboard.py` |
 
In every terminal I also ran `export MORPHEUS_INFERENCE_PROVIDER=deterministic`.
 
![Docker containers running](photos/02_docker_ps.png)
![Redpanda Console](photos/03_redpanda_console.png)
![All services running](photos/04_services_running.png)
 
### 2.3 Dashboard
 
After fetching cases from Kafka, the dashboard showed:
 
| Metric | Value |
|---|---|
| Cases | 1503 |
| High Risk | 181 |
| Pending Human Review | 181 |
| Meta-AI Escalations | 0 |
 
![Dashboard](photos/05_dashboard.png)
 
---
 
## 3. Case identification (Part C)
 
### Checkpoint C1
 
| Field | Value |
|---|---|
| `alert_id` | `alert-0-95` |
| `correlation_id` | `0ffb1919-4f8b-4fa8-b71d-025e8bc8405d` |
| event type | `suspicious_process` |
| host | `host-02` |
| user | `svc-backup` |
| risk score | 100 |
 
### Tracing the case through Kafka
 
I found the alert in the `morpheus.alerts` topic (offset 36) with all the raw
event values (see 4.1), and later my decision in the `human.decisions` topic
(see section 6).
 
![Alert in morpheus.alerts](photos/09_morpheus_alerts.png)
 
### Question C1: `alert_id` vs `correlation_id`
 
`alert_id` is a short, readable ID. The dashboard uses it so the analyst can
select one case and see the same case in every panel.
 
`correlation_id` is a unique ID that is created with the original event and
stays the same in every Kafka topic and every agent. It is used to follow one
event through all the backend services, even in topics where there is no
`alert_id` yet (for example `raw.logs`).
 
---
 
## 4. Case analysis (Part E)
 
![XAI, RAI and Meta-AI](photos/07_xai_rai_metaai.png)
 
### 4.1 Detection evidence (E1)
 
| Evidence | Observation |
|---|---|
| event type | `suspicious_process` |
| failed logins | 20 |
| outbound bytes | 43,773,068 (about 43.8 MB) |
| process count | 198 |
| geographic indicator | RU, source IP `185.220.101.5` |
| fingerprint deviation | z = 317.6 (profile based on 200 samples) |
| Isolation Forest result | -1 (anomalous), anomaly score -0.0598 |
 
**Strongest evidence:** several different signs appear together in one event:
failed logins, a large outbound transfer, a high process count, a login from a
foreign country, and a suspicious process.
 
**Weak or missing evidence:**
 
- There is no check of the computer itself (no endpoint data).
- The z-score of 317.6 is not realistic. The account `svc-backup` seems to
  behave in a very regular way, so its standard deviation is close to zero, and
  any small change becomes a huge z-score.
- The Isolation Forest score (-0.06) is very close to the normal/anomalous
  border, so the model itself is not very sure.
- The account name suggests a backup service, so a large data transfer could
  be normal. This is only an assumption from the name.
- The event already contains `"label": "suspicious"` from the generator. A real
  SOC would not have this label. This is a limit of synthetic data.
### 4.2 XAI explanation (E2)
 
1. **What triggered the alert?** All seven indicators together, which gave the
   maximum risk score of 100.
2. **Strongest evidence according to the XAI:** it lists all seven reasons
   without ranking them.
3. **Does it match the evidence?** Yes, but it only repeats the evidence list
   in one sentence and adds the recommended action.
4. **Does it overstate certainty?** Yes. It does not mention any uncertainty,
   for example the unrealistic z-score or the missing endpoint data.
5. **What is missing?** Real interpretation, uncertainty, and the MITRE ATT&CK
   mapping. The orchestrator prompt asks for all of these, but the
   `deterministic` provider uses a fixed template and does not really read the
   prompt.
### 4.3 RAI policy (E3)
 
| Field | Value |
|---|---|
| recommended action | `recommend_isolation_with_human_approval` |
| action allowed | `true` |
| human approval required | `true` |
| policy rationale | "High risk. Isolation may be recommended, but human approval is required." |
 
The policy did its job: it did not allow automatic isolation and required a
human decision. However, `policy_evidence_count = 7` only counts the reasons.
It does not check their quality, so the unrealistic z-score counts the same as
the Isolation Forest result.
 
### 4.4 Meta-AI review (E4)
 
| Field | Value |
|---|---|
| disposition | `approve` |
| verification result | `approved: true` |
| challenge or concern | none (`issues: []`) |
| uncertainty | 0 |
| additional evidence requested | none (`questions_or_requests: []`) |
 
The Meta-AI did not really check the recommendation. It reported zero
uncertainty on synthetic data with no endpoint confirmation, and its reflection
is a general sentence ("Decision is sufficiently supported for governed
continuation."). In the dashboard it approved all 1503 cases and made 0
escalations, so it works more like a rubber stamp than a real reviewer.
 
---
 
## 5. Human decision (Part F)
 
**Decision:** `request_more_evidence`
 
**Justification:**
 
> I request more evidence before isolating the server.
>
> The event looks dangerous: 20 failed logins, 43 MB of data sent out,
> 198 processes, and a login from Russia.
>
> But I cannot confirm it is a real attack. Nobody checked the computer
> itself. The account name (svc-backup) suggests a backup service, so
> sending a lot of data could be normal.
>
> The AI was 100% sure and the Meta-AI raised no questions, which is too
> confident for this evidence.
>
> Isolating a server is a big step. Before approving it, I would check the
> processes on host-02, check the IP address 185.220.101.5, and check if a
> backup was scheduled at that time.
 
Note: the first sentence was not included in the text I submitted in the
dashboard. The decision itself is stored in the `decision` field.
 
![Decision form](photos/08_decision.png)
 
---
 
## 6. Verification (Part G)
 
### G1. Dashboard
 
`alert-0-95` now shows `request_more_evidence` instead of `pending`, and
Pending Human Review went down from 181 to 180.
 
![Decision recorded](photos/10_decision_recorded.png)
 
### G2. Kafka
 
The decision appears in the `human.decisions` topic with all expected fields:
`alert_id`, `correlation_id`, `decision`, `justification`, `risk_score`,
`rai_recommendation`, `meta_ai_disposition`.
 
![human.decisions topic](photos/11_human_decisions_topic.png)
 
### G3. Audit file
 
The decision was appended to `data/audit.jsonl`. A copy of the record is saved
in [`data/audit.json`](data/audit.json).
 
---
 
## 7. Research export (Part H)
 
```bash
python export_research_data.py
```
 
Result: `Exported 1504 records to exports/morpheus_lite_20261126_091744.parquet`
(1503 cases + 1 human decision).
 
![Export](photos/12_export.png)
 
**Problem found:** the first run failed with:
 
```text
pyarrow.lib.ArrowNotImplementedError: Cannot write struct type
'provider_metadata' with no child field to Parquet.
```
 
**Cause:** with the `deterministic` provider, the field `provider_metadata` is
always an empty object (`{}`). Parquet cannot store an empty struct.
 
**Fix:** in `morpheus_lite/storage.py` I added a small function that replaces
empty objects with `None` before writing the Parquet file. No other data is
changed.
 
---
 
## 8. Additional findings
 
1. **The dashboard fetch never finishes under continuous load.** The fetch
   stops only after 2.5 seconds with no new messages (`consumer_timeout_ms=2500`).
   While the generator was running, new cases arrived all the time, so the
   dashboard stayed stuck. I had to stop the generator before the cases appeared.
2. **Many alerts look the same.** Most alerts had risk score 80 and were based
   only on the fingerprint deviation (z around 13–14). With score 80 (below the
   85 threshold), these cases do not require human approval.
3. **Possible lateral movement.** Right after my case, `alert-0-96` shows the
   same account (`svc-backup`) with the same event type and risk 100, but on
   `host-03`. This should be investigated together with my case.
---
 
## 9. Reflection (Part I)
 
**1. Did your final decision agree with the AI recommendation? Why?**
No. The AI recommended isolation. I agreed that the case is suspicious, but I
asked for more evidence first, because the attack is not confirmed and
isolating a server has a real operational cost.
 
**2. Which evidence had the greatest influence on your decision?**
The combination of failed logins, the login from Russia, and the high process
count. On the other side, the missing endpoint data and the backup account name
made me careful.
 
**3. What did the XAI explanation help you understand?**
It showed quickly which indicators caused the alert and what the recommended
action was.
 
**4. What part of the explanation did you challenge?**
Its full confidence. It did not mention any uncertainty, even though the
z-score of 317.6 is not realistic and there was no endpoint check.
 
**5. What additional evidence would improve confidence?**
The list of processes on host-02, a threat-intelligence check of
185.220.101.5, the backup schedule, and the related case on host-03.
 
**6. How did the RAI policy affect your decision?**
It stopped the system from isolating the server automatically and gave the
final decision to me. This gave me the space to ask for more evidence.
 
**7. Did the Meta-AI review make you more or less willing to rely on the recommendation?**
Less. It approved every case with zero uncertainty and asked no questions, so
it did not add a real independent check.
 
**8. Who retained final decision authority in this workflow?**
The human analyst (me). The AI agents only gave recommendations, and the RAI
policy required human approval before any action.
 
---
 
## Submission Checklist
 
- [x] Completed case-identification table
- [x] Evidence analysis
- [x] XAI evaluation
- [x] RAI and Meta-AI review
- [x] Human decision
- [x] Written justification
- [x] Kafka verification
- [x] Audit verification
- [x] Research export verification
- [x] Reflection answers
 