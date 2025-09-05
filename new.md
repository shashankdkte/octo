# CASHNET High-Level Flow

```mermaid
flowchart LR
  %% High level CASHNET flow with numbered steps

  M["1️⃣ Merchant / ATM / E-com"]

  subgraph ACQ["2️⃣ Acquirers"]
    A1["HSBC ACQ/ISS"]
    A2["CITI ACQ/ISS"]
  end

  subgraph REN["3️⃣ Renaissance Online (REN) — Switch/Router"]
    R["REN Core"]
    subgraph ADP["6️⃣ Issuer Adapters"]
      ADP1["Barclays Adapter"]
      ADP2["Deutsche Adapter"]
      ADP3["HSBC Adapter"]
      ADP4["CITI Adapter"]
      ADP5["Other Issuers"]
    end
  end

  subgraph HSMBox["4️⃣ Security"]
    HSM["HSM — Crypto / PIN / EMV"]
  end

  subgraph OBS["9️⃣ Observability & Ops"]
    GLOG["Graylog Logs"]
    GRAF["Grafana Metrics"]
    UI["REN UI Console"]
  end

  subgraph BATCH["8️⃣ Downstream & Batch"]
    WS["WS Feed (Real-time Facts)"]
    SFTP["SFTP Folder (Machine Files)"]
    DMS["DMS Reports (Human-readable)"]
    OFF["Offline RT / Clearing & Reco"]
  end

  subgraph ISS["7️⃣ Issuer Banks"]
    I1["Barclays"]
    I2["Deutsche"]
    I3["HSBC"]
    I4["CITI"]
    I5["Others"]
  end

  %% Flows
  M -->|"Step 1: Txn"| A1
  M -->|"Step 1: Txn"| A2

  A1 -->|"Step 2: Auth Request"| R
  A2 -->|"Step 2: Auth Request"| R

  R -->|"Step 3: Crypto Check"| HSM
  HSM -->|"Step 3: OK/Fail"| R

  R -->|"Step 4: Route BIN"| ADP1 -->|"Step 5"| I1 -->|"Step 6 Resp"| R
  R --> ADP2 -->|"5"| I2 -->|"6"| R
  R --> ADP3 -->|"5"| I3 -->|"6"| R
  R --> ADP4 -->|"5"| I4 -->|"6"| R
  R --> ADP5 -->|"5"| I5 -->|"6"| R

  R -->|"Step 7: Push Facts"| WS -->|"Step 8 Reco"| OFF
  R -->|"Step 7a: Files"| SFTP -->|"8a Reco"| OFF
  R -->|"7b Reports"| DMS --> OFF

  R -->|"Step 9: Logs"| GLOG
  R -->|"Step 9a: Metrics"| GRAF
  UI <-->|Step 10: Control| R
```
### How to Read

1.  **1 → 2**: Merchant sends txn → Acquirer forwards to REN
    
2.  **3**: REN Core calls HSM for crypto check
    
3.  **4–6**: REN routes to correct Issuer Adapter → Issuer Bank → decision back
    
4.  **7–8**: REN pushes facts & files downstream (WS, SFTP, DMS → Offline/Clearing)
    
5.  **9**: Logs to Graylog, metrics to Grafana
    
6.  **10**: Ops use REN UI to monitor/control

```mermaid
flowchart TB
  %% Module 2 — Components of CASHNET with numbered roles

  M["1️⃣ Merchant / ATM / E-com\n(Start: where card txn begins)"]

  subgraph ACQ["2️⃣ Acquirers\n(First stop — receive txn from merchant)"]
    A1["HSBC ACQ/ISS"]
    A2["CITI ACQ/ISS"]
  end

  subgraph REN["3️⃣ Renaissance Online (REN)\n(Central switch/router — brain)"]
    R["REN Core"]
  end

  subgraph HSMBox["4️⃣ HSM\n(Security fortress — PIN/EMV/MAC)"]
    HSM["Hardware Security Module"]
  end

  subgraph ADP["6️⃣ Issuer Adapters\n(Language translators per bank)"]
    ADP1["Barclays Adapter"]
    ADP2["Deutsche Adapter"]
    ADP3["HSBC Adapter"]
    ADP4["CITI Adapter"]
    ADP5["Other Issuers"]
  end

  subgraph ISS["7️⃣ Issuer Banks\n(Decision makers)"]
    I1["Barclays Bank"]
    I2["Deutsche Bank"]
    I3["HSBC Bank"]
    I4["CITI Bank"]
    I5["Other Banks"]
  end

  subgraph BATCH["8️⃣ Downstream Systems"]
    WS["WS Feed\n(Real-time stream)"]
    SFTP["SFTP Folder\n(Machine files)"]
    DMS["DMS\n(Human reports)"]
    OFF["Offline RT / Clearing\n(Reconciliation, GL)"]
  end

  subgraph OBS["9️⃣ Observability & Ops"]
    GLOG["Graylog\n(Logs)"]
    GRAF["Grafana\n(Metrics/Alerts)"]
    UI["REN UI\n(Ops Console)"]
  end

  %% Connections
  M -->|"Step 1: Send Txn"| ACQ
  ACQ -->|"Step 2: Forward to REN"| R
  R -->|"Step 3: Crypto Ops"| HSM
  R -->|"Step 4: Route to Adapters"| ADP
  ADP -->|"Step 5: Talk to Issuers"| ISS
  ISS -->|"Step 6: Approve/Decline"| R
  R -->|"Step 7: Push Facts"| WS --> OFF
  R -->|"Step 7a: Files"| SFTP --> OFF
  R -->|"7b Reports"| DMS --> OFF
  R -->|"Step 8: Logs"| GLOG
  R -->|"Step 9: Metrics"| GRAF
  UI <-->|Step 10: Control| R


```

-   **1️⃣ Merchant/ATM/E-com** → Origin of transactions.
    
-   **2️⃣ Acquirers (HSBC, CITI)** → First entry into banking world.
    
-   **3️⃣ REN Core** → The “traffic cop” / brain.
    
-   **4️⃣ HSM** → Safe for PIN/EMV/crypto.
    
-   **6️⃣ Issuer Adapters** → Translate REN language → Issuer protocols.
    
-   **7️⃣ Issuer Banks** → Approve/Decline.
    
-   **8️⃣ Downstream** → Files, feeds, and reconciliation.
    
-   **9️⃣ Observability & Ops** → Eyes & controls: Graylog, Grafana, REN UI.
    
-   **10** → Ops interact safely with REN.

```mermaid
sequenceDiagram
    %% Module 3: Real-time Authorization Flow
    autonumber

    participant C as 1️⃣ Cardholder<br/>(Customer paying)
    participant M as 2️⃣ Merchant / ATM / E-com<br/>(Point of Sale)
    participant ACQ as 3️⃣ Acquirer Bank<br/>(HSBC / CITI)
    participant REN as 4️⃣ REN Core<br/>(Switch/Router)
    participant HSM as 5️⃣ HSM<br/>(Crypto Safe)
    participant ADP as 6️⃣ Issuer Adapter<br/>(Translator)
    participant ISS as 7️⃣ Issuer Bank<br/>(Decision Maker)
    participant WS as 8️⃣ WS Feed<br/>(Real-time data stream)
    participant LOG as 9️⃣ Logs & Metrics<br/>(Graylog/Grafana)

    C->>M: Step 1: Tap / Swipe card<br/>(Start transaction)
    M->>ACQ: Step 2: Send transaction request<br/>(PAN, Amount, PIN/EMV)
    ACQ->>REN: Step 3: Forward Auth Request<br/>(ISO-8583/JSON over TLS)

    REN->>HSM: Step 4: Crypto checks<br/>(PIN translate, ARQC, MAC)
    HSM-->>REN: Step 4a: Return OK / Fail

    REN->>ADP: Step 5: Route to Issuer Adapter<br/>(based on BIN rules)
    ADP->>ISS: Step 6: Send Issuer-specific message
    ISS-->>ADP: Step 7: Approve / Decline + Auth Code
    ADP-->>REN: Step 8: Return response
    REN-->>ACQ: Step 9: Send decision
    ACQ-->>M: Step 10: Show Approved / Declined

    REN->>WS: Step 11: Push txn facts downstream
    REN->>LOG: Step 12: Log + Metrics<br/>(Graylog / Grafana dashboards)

```


-   **Step 1–3**: Cardholder → Merchant/ATM → Acquirer → REN.
    
-   **Step 4**: REN asks **HSM** to validate PIN/EMV cryptograms & MAC.
    
-   **Step 5–7**: REN routes to correct **Issuer Adapter** → Issuer Bank decides approve/decline.
    
-   **Step 8–10**: Response flows back → REN → Acquirer → Merchant → Cardholder.
    
-   **Step 11**: REN pushes transaction facts to **WS Feed** for back-office.
    
-   **Step 12**: Logs & metrics sent to Graylog/Grafana; Ops monitor via REN
```mermaid
flowchart TB
  %% Module 4: ISO-8583 core fields, validation, mapping, masking (parser-safe)

  subgraph ACQ["1️⃣ Incoming Auth Message (from Acquirer)"]
    MSG["ISO-8583 / JSON over TLS"]
  end

  subgraph FIELDS["2️⃣ Important Fields (subset)"]
    F2["F2 PAN\n(card number)\nLog: mask 6+4"]
    F3["F3 Processing Code\n(purchase/refund)"]
    F4["F4 Amount (minor units)"]
    F7["F7 Trans DateTime (MMDDhhmmss)"]
    F11["F11 STAN (local trace)"]
    F37["F37 RRN (global trace)"]
    F41["F41 Terminal ID"]
    F42["F42 Merchant ID"]
    F49["F49 Currency"]
    F52["F52 PIN Data (encrypted)"]
    F55["F55 EMV Data (ARQC TLVs)"]
    F64["F64/128 MAC (message auth)"]
  end

  subgraph NORM["3️⃣ Normalizer (REN)"]
    VAL["Validation\n(required fields / formats)"]
    MAP["Field Mapping\n(variant → REN model)"]
    MASK["Masking/Redaction\n(PAN 6+4; no CVV/PIN)"]
  end

  subgraph MODEL["4️⃣ REN Internal Model"]
    RQ["ren_txn_request\n{ pan_masked, amount_minor,\nstan, rrn, merchant, terminal,\nemv, currency, ... }"]
  end

  subgraph OBS["5️⃣ Observability"]
    LOG["Graylog JSON Log\n{txn_id, stage, pan_masked,\nstan, rrn, amount_minor, ...}"]
    MET["Grafana Metrics\n(tps, p95, p99, errors)"]
  end

  %% Flow
  MSG --> FIELDS
  FIELDS --> VAL --> MAP --> MASK --> RQ
  RQ --> LOG
  RQ --> MET

  %% Notes (as separate nodes; dotted links)
  N1["📝 EMV F55 TLVs:\n• ARQC (card cryptogram)\n• TVR/TSI (risk indicators)"]
  F55 -.-> N1

  N2["🛡️ Never log CVV,\nfull PAN, clear PIN, or keys.\nMask PAN as 6+4 only."]
  MASK -.-> N2

```

-   **1 → 2**: Auth message arrives with key ISO fields.
    
-   **3**: REN validates, maps issuer variant → internal model, and **masks** sensitive data.
    
-   **4**: Clean internal `ren_txn_request` is produced.
    
-   **5**: Safe, structured log + metrics emitted (no secrets).
    



```mermaid
flowchart TB
  %% Module 5: HSM and Keys (parser-safe with numbers)

  subgraph HSM["5️⃣ HSM — The Crypto Safe"]
    LMK["Key 5.1: LMK\n(Local Master Key)\nProtects all other keys"]
    ZMK["Key 5.2: ZMK\n(Zone Master Key)\nExchange keys with partners"]
    ZPK["Key 5.3: ZPK\n(Zone PIN Key)\nEncrypts PIN blocks"]
    BDK["Key 5.4: BDK\n(Base Derivation Key)\nGenerates unique keys"]
    DUKPT["5.5: DUKPT Keys\nDerived per txn/device"]
    PIN["5.6: PIN Translation\n(ISO-0/1/3 formats)"]
    EMV["5.7: EMV ARQC/ARPC\nVerify & Generate"]
    MAC["5.8: MAC / HMAC\nIntegrity check"]
  end

  %% Relationships
  LMK --> ZMK
  ZMK --> ZPK
  BDK --> DUKPT
  ZPK --> PIN
  DUKPT --> EMV
  LMK --> MAC

  subgraph Controls["5.9 Controls"]
    DC["Dual Control\n(2 people required)"]
    AUD["Audit Ceremonies\nDocumented key handling"]
    ROT["Planned Key/Cert Rotation"]
  end

  DC --> LMK
  AUD --> LMK
  ROT --> LMK
  ROT --> ZMK
  ROT --> ZPK
  ROT --> BDK

```

-   **LMK (Local Master Key)** → the “master lock” inside HSM, protects everything.
    
-   **ZMK (Zone Master Key)** → used to exchange keys with partner banks securely.
    
-   **ZPK (Zone PIN Key)** → used for encrypting customer PIN blocks.
    
-   **BDK (Base Derivation Key)** → generates one-time keys (DUKPT).
    
-   **DUKPT Keys** → unique per transaction/device.
    
-   **PIN Translation, EMV cryptograms, MAC/HMAC** → done only inside the HSM.
    
-   **Controls** → dual control, ceremonies, rotations to ensure compliance & safety.

```mermaid
flowchart LR
  %% Module 6: Observability and Ops (parser-safe, corrected)

  subgraph REN["REN Core (Switch/Router)"]
    R["6.1 REN Engine"]
  end

  subgraph Graylog["6.2 Graylog"]
    LOG["Structured JSON Logs"]
  end

  subgraph Grafana["6.3 Grafana / Prometheus"]
    MET["Metrics (TPS, p95/p99, errors, queues, HSM uptime, SFTP lag)"]
    ALERT["Alert Rules"]
  end

  subgraph Ops["6.4 REN UI (Ops Console)"]
    UI["Monitor & Control\n(RBAC + Audit)"]
  end

  %% Flows
  R --> LOG
  R --> MET
  MET --> ALERT
  ALERT --> UI
  UI <--> R
```


-   **Step 1** → Logs go into **Graylog** for deep transaction tracing.
    
-   **Step 2** → Metrics (TPS, latency, errors, HSM uptime) go to **Grafana/Prometheus**.
    
-   **Step 3** → **Alert rules** trigger warnings to Ops before customers feel impact.
    
-   **Step 4** → Ops use **REN UI** (RBAC-controlled) for safe operational actions.

```mermaid

flowchart  TB

%% Module 7: EOD Clearing & Settlement (parser-safe)

  

subgraph  ITM["7.1 ITM / Scheduler"]

CUT["Cutover  Window\n(e.g. 23:30–23:59)"]

end

  

subgraph  REN["7.2 REN  Core"]

AGG["Aggregate  Txns\n(by acquirer / issuer / merchant)"]

BUILD1["Build  ESHTRNOP  File\n(HSBC/CITI)"]

BUILD2["Build  Settlement  Reports\n(Other Banks)"]

BUILD3["Build  REN  FD  File\n(Finance / GL)"]

end

  

subgraph  SFTPBox["7.3 SFTP  Folder"]

SFTP["Machine-readable  files\n.csv + .sha256 + .asc"]

end

  

subgraph  DMSBox["7.4 DMS"]

DMS["Human-readable  Reports\n(PDF/CSV/HTML)"]

end

  

subgraph  WSBox["7.5 WS  Feed"]

WS["Real-time  Txn  Facts"]

end

  

subgraph  OFF["7.6 Offline  RT / Clearing"]

RECO["Reconciliation & GL  Posting\n(Exception handling, disputes)"]

end

  

%% Flows

CUT  --> AGG

AGG  --> BUILD1  --> SFTP

AGG  --> BUILD2  --> SFTP

AGG  --> BUILD3  --> SFTP

BUILD3  --> DMS

  

WS  --> RECO

SFTP  --> RECO

DMS  --> RECO
```

-   **7.1 ITM (Scheduler)** → opens the **cutover window** at end of day.
    
-   **7.2 REN Core** → aggregates all daily transactions and builds 3 types of files:
    
    -   **ESHTRNOP** → for HSBC & CITI
        
    -   **Settlement Reports** → for other issuers
        
    -   **REN FD file** → finance/general ledger
        
-   **7.3 SFTP Folder** → stores machine-readable files with **.part → rename → checksum → ACK** process.
    
-   **7.4 DMS** → publishes human-readable reports for finance/ops.
    
-   **7.5 WS Feed** → streams near-real-time data.
    
-   **7.6 Offline RT/Clearing** → consumes WS + SFTP + DMS to perform reconciliation, exception handling, and GL posting.

```mermaid
flowchart TB
  %% Module 8: High Availability & DR

  subgraph DC1["8.1 Primary Data Center"]
    R1["REN-A (Active)"]
    R2["REN-B (Active)"]
    H1["HSM-1"]
    H2["HSM-2"]
    A1["Issuer Adapters (Scaled)"]
    S1["SFTP Cluster"]
    O1["Obs: Graylog/Grafana"]
  end

  subgraph DC2["8.2 DR Data Center"]
    RDR["REN-DR (Standby / Active)"]
    HDR["HSM-DR"]
    ADR["Issuer Adapters DR"]
    S2["SFTP DR"]
    O2["Obs DR"]
  end

  %% Connections in primary
  R1 <--> R2
  R1 --> H1
  R2 --> H2
  R1 --> A1
  R2 --> A1
  R1 --> S1
  R2 --> S1
  R1 --> O1
  R2 --> O1

  %% Replication / DR
  R1 --> RDR
  R2 --> RDR
  H1 --> HDR
  H2 --> HDR
  S1 --> S2
  O1 --> O2

  %% Notes
  N1["📝 High Availability:\n- REN nodes in Active/Active\n- HSM in pairs\n- Adapters scaled"]
  N2["📝 Disaster Recovery:\n- Target RTO ~1h\n- RPO ~0 for Auth\n- Quarterly DR drills"]
  N3["📝 Capacity Planning:\n- Design for 2× peak TPS\n- Keep HSM util <60%\n- Monitor CPU/queues"]

  DC1 --- N1
  DC2 --- N2
  S1 --- N3
```

-   **8.1 Primary DC** → multiple REN nodes run **Active/Active**; HSM in **pairs**; Adapters scale horizontally.
    
-   **8.2 DR DC** → standby systems replicate configs, routing tables, keys, and logs.
    
-   **Replication** → REN, HSM backups, SFTP, and observability dashboards sync to DR.
    
-   **Capacity** → system designed to handle **2× peak load** with **HSM <60% utilization** at peak.
```mermaid
flowchart TD
  %% Module 9: Runbooks & Checklists

  subgraph Daily["9.1 Start-of-Shift Checks"]
    D1["Step 1: Check TPS/Latency in Grafana"]
    D2["Step 2: Verify Queue Depth < threshold"]
    D3["Step 3: Issuer Adapters status = GREEN"]
    D4["Step 4: HSM HA status & key versions"]
    D5["Step 5: ITM calendar (holidays, cutover)"]
    D6["Step 6: SFTP space >20% & last ACK received"]
    D7["Step 7: DMS reports published"]
    D8["Step 8: WS Feed consumer lag <60s"]
  end

  subgraph EOD["9.2 End-of-Day Monitoring"]
    E1["Pre-cutover: ensure inflight ≈ 0, queues drained"]
    E2["Check ITM job chain start/finish times"]
    E3["Verify ESHTRNOP / Settlement / REN FD files built"]
    E4["Check .part → final rename, .sha256, PGP"]
    E5["Confirm ACK files from downstream"]
    E6["Validate DMS reports accessible"]
    E7["Reco totals match acquirer/issuer summaries"]
  end

  subgraph Incident["9.3 Quick-Play (Issuer Down)"]
    I1["Step A: Confirm issue (Grafana p95/p99, Graylog ISS-001)"]
    I2["Step B: Contain — Pause/Throttle route in REN UI"]
    I3["Step C: Coordinate with Issuer NOC (open ticket)"]
    I4["Step D: Recover — Warm-up traffic after green"]
    I5["Step E: Post-Incident — Document & RCA"]
  end

  %% Flow arrows (linear in each subgraph)
  D1 --> D2 --> D3 --> D4 --> D5 --> D6 --> D7 --> D8
  E1 --> E2 --> E3 --> E4 --> E5 --> E6 --> E7
  I1 --> I2 --> I3 --> I4 --> I5

```
-   **Daily checks (9.1)** → Operators start shift by checking Grafana (TPS/latency), queues, adapter health, HSM, ITM jobs, SFTP space/ACKs, DMS reports, and WS lag.
    
-   **EOD monitoring (9.2)** → During cutover, ensure no inflight txns, verify all files built, renamed, checksummed, ACK’d, and reconciliation balances.
    
-   **Incident quick-play (9.3)** → 5 steps: Confirm → Contain → Coordinate → Recover → Post-incident learning.


```mermaid
flowchart TB
  %% Module 10: Do's & Don'ts (parser-safe)

  subgraph DO["10.1 ✅ DO's"]
    D1["Do-1: Mask PAN (6+4 only),\nHash txn_id, redact secrets"]
    D2["Do-2: Use atomic file writes\n(.part → final) + unique run-ids"]
    D3["Do-3: Set proactive alerts\n(p95 latency, issuer timeout,\nSFTP lag)"]
    D4["Do-4: Maintain test BINs\n+ golden transactions for UAT"]
  end

  subgraph DONT["10.2 ❌ DON'Ts"]
    N1["Don't-1: Change routing/keys\nwithout ticket + peer review + rollback"]
    N2["Don't-2: Retry blindly on\ncrypto errors (investigate cause)"]
    N3["Don't-3: Re-emit batch file\nwith same run-id (causes duplicates)"]
    N4["Don't-4: Store CVV or clear PIN\nanywhere (logs, DB, backups)"]
  end

  %% Simple flows for clarity
  DO --> DONT
```
-   **DO’s (10.1)**
    
    1.  Always **mask sensitive data** (PAN 6+4).
        
    2.  Use **atomic file handling** and unique run IDs.
        
    3.  Configure **alerts early** (catch problems before customers do).
        
    4.  Keep **test BINs & golden transactions** ready for validation.
        
-   **DON’T’s (10.2)**
    
    1.  Never make routing/crypto key changes without **formal process**.
        
    2.  Don’t **blindly retry crypto errors** (they mean setup mismatch).
        
    3.  Don’t resend files with the **same run ID**.
        
    4.  Never **store CVV or clear PIN** anywhere. 	
```mermaid
flowchart TD
  %% Module 11: Error Codes & Actions (parser-safe)

  Start["⚠️ 11.0 Error Detected"]

  Start --> VAL["11.1 VAL-001\nValidation Error"]
  Start --> CRY1["11.2 CRY-001\nCrypto Error (ARQC/PIN/MAC)"]
  Start --> CRY2["11.3 CRY-002\nHSM Unavailable"]
  Start --> RTG["11.4 RTG-001\nRouting / BIN Error"]
  Start --> ISS["11.5 ISS-001\nIssuer Timeout"]
  Start --> FIL1["11.6 FIL-001\nFile Transfer Error"]
  Start --> FIL2["11.7 FIL-002\nDuplicate File"]
  Start --> SCH["11.8 SCH-001\nScheduler Job Error"]
  Start --> SEC["11.9 SEC-001\nCertificate Expired"]

  %% Actions
  VAL --> A1["Action: Check input fields,\nengage Acquirer"]
  CRY1 --> A2["Action: Check EMV params,\nalign with Issuer"]
  CRY2 --> A3["Action: Failover to backup HSM,\nraise incident"]
  RTG --> A4["Action: Update BIN table\nwith Change Control"]
  ISS --> A5["Action: Pause route in REN UI,\ncontact Issuer NOC"]
  FIL1 --> A6["Action: Retry transfer,\ncheck permissions"]
  FIL2 --> A7["Action: Use new run-id\n(ensure idempotency)"]
  SCH --> A8["Action: Check dependencies,\nrerun ITM job"]
  SEC --> A9["Action: Rotate certificates\n(update trust stores)"]

```

-   **11.1 VAL-001** → Missing/invalid ISO field → fix request, contact Acquirer.
    
-   **11.2 CRY-001** → Crypto mismatch (bad EMV/PIN) → check params with Issuer.
    
-   **11.3 CRY-002** → HSM down → failover, escalate incident.
    
-   **11.4 RTG-001** → BIN routing gap → fix routing in change window.
    
-   **11.5 ISS-001** → Issuer not responding → pause route, inform NOC, retry later.
    
-   **11.6 FIL-001** → File transfer issue → retry, check SFTP permissions.
    
-   **11.7 FIL-002** → Duplicate file run-id → regenerate with new run-id.
    
-   **11.8 SCH-001** → Scheduler job failed → check predecessor, rerun.
    
-   **11.9 SEC-001** → Certificate expired → emergency rotate, update trust stores.

```mermaid
flowchart TD
  %% Module 12: Issuer Down Playbook (parser-safe)

  C1["12.1 Confirm Issue\n- Check Grafana (p95/p99 latency)\n- Check Graylog (ISS-001 errors)"]
  C2["12.2 Contain Impact\n- Pause/Throttle route in REN UI"]
  C3["12.3 Coordinate\n- Call Issuer NOC\n- Open incident ticket\n- Get ETA"]
  C4["12.4 Recover Service\n- Warm-up traffic gradually\n- Monitor p95/p99 latency"]
  C5["12.5 Post-Incident\n- Document timeline\n- Draft RCA\n- Capture lessons"]

  C1 --> C2 --> C3 --> C4 --> C5

```

-   **12.1 Confirm** → Use Grafana & Graylog to verify the spike in timeouts/errors.
    
-   **12.2 Contain** → Pause or throttle traffic in **REN UI** to limit customer impact.
    
-   **12.3 Coordinate** → Call the **Issuer’s NOC**, open ticket, get estimated fix time.
    
-   **12.4 Recover** → Slowly ramp up traffic once Issuer is stable, keep monitoring latency.
    
-   **12.5 Post-Incident** → Document everything, write RCA, capture learnings


```mermaid
flowchart TD
  %% Module 13: Security Layers (parser-safe)

  L1["13.1 Layer 1: Data in Motion\n- mTLS\n- FIPS-approved ciphers"]
  L2["13.2 Layer 2: Data at Rest\n- PGP encryption\n- Encrypted disks"]
  L3["13.3 Layer 3: HSM & Keys\n- LMK / ZMK / ZPK / BDK\n- DUKPT"]
  L4["13.4 Layer 4: Access Control\n- RBAC\n- Least privilege\n- Audit logs"]
  L5["13.5 Layer 5: Compliance\n- PCI DSS scope\n- Network segmentation"]
  L6["13.6 Layer 6: Monitoring\n- Mask PAN 6+4\n- Cert expiry alerts"]

  L1 --> L2 --> L3 --> L4 --> L5 --> L6

```


-   **13.1 Data in Motion** → All comms encrypted with **mTLS** + strong ciphers.
    
-   **13.2 Data at Rest** → Files stored with **PGP encryption** and encrypted disks.
    
-   **13.3 HSM & Keys** → Keys (LMK, ZMK, ZPK, BDK, DUKPT) live only inside HSM.
    
-   **13.4 Access Control** → Strict **RBAC**, dual control, audit logs.
    
-   **13.5 Compliance** → PCI DSS rules, segmentation, retention.
    
-   **13.6 Monitoring** → Mask sensitive data, alert on certificate expiry & key drift.

```mermaid
erDiagram

%% Module 14: Offline RT / Clearing Data Relationships

  

MERCHANT {

string  merchant_id  PK

string  name

string  mcc

}

  

ACQUIRER {

string  acquirer_id  PK

string  name

}

  

ISSUER {

string  issuer_id  PK

string  name

}

  

TRANSACTION {

string  txn_id  PK

string  stan

string  rrn

string  pan_masked

int  amount_minor

string  currency

string  decision

datetime  capture_ts

}

  

SETTLEMENTBATCH {

string  batch_id  PK

date  batch_date

string  type

}

  

GLPOSTING {

string  posting_id  PK

string  account

int  amount_minor

}

  

EXCEPTION {

string  ex_id  PK

string  reason

string  status

}

  

RECONCILIATION {

string  reco_id  PK

string  note

}

  

%% Relationships (numbered flows)

MERCHANT ||--o{ TRANSACTION : "14.1 initiates"

ACQUIRER ||--o{ TRANSACTION : "14.2 receives"

ISSUER ||--o{ TRANSACTION : "14.3 decides"

TRANSACTION ||--o{ SETTLEMENTBATCH : "14.4 grouped  into"

SETTLEMENTBATCH ||--o{ GLPOSTING : "14.5 produces"

TRANSACTION ||--o{ EXCEPTION : "14.6 may  flag"

EXCEPTION ||--o{ RECONCILIATION : "14.7 handled  in"
```

-   **14.1 Merchant** → initiates transactions.
    
-   **14.2 Acquirer** → receives them.
    
-   **14.3 Issuer** → decides (approve/decline).
    
-   **14.4 Transactions** are grouped into **Settlement Batches**.
    
-   **14.5 Batches** generate **GL Postings** for Finance.
    
-   **14.6 Exceptions** are linked to failed/mismatched transactions.
    
-   **14.7 Reconciliation** handles exceptions and ensures totals match.