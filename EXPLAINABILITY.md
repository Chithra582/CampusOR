# EXPLAINABILITY.md

This document explains the internal mechanisms, data lineage, operational boundaries, and governance framework of **CampusOR Agent** (`campusor-agent`) in accordance with the **OpenGAP v0.1.0** specification for the **HiDevs GitAgent Passport** clearance pipeline.

> **Agent Name:** CampusOR Agent (`campusor-agent`)  
> **Specification:** OpenGAP v0.1.0  
> **Category / Domain:** Productivity / Campus Queue Management & Operations  
> **Compliance Standard:** OpenGAP Checkpoint 2 (Explainability & Decision Governance), FERPA, GDPR  

---

## How the Agent Decides

CampusOR Agent is an autonomous queue orchestration and predictive dispatch intelligence designed for **CampusOR**, an online queue and reservation platform for universities, campus health centers, administrative registrar desks, and dining facilities. The agent eliminates physical waiting lines by managing dynamic token sequencing, real-time counter load balancing, and machine-learning wait-time predictions.

### 1. Decision Architecture

The decision, queueing, and dispatch process operates through a deterministic, five-stage pipeline:

```
User Request (Join Queue / Book Slot / Operator Next / Status Check)
    │
    ▼
[Stage 1: Intake Validation & Rate Limiting]
    │  - Authenticates student/visitor identity via JWT or session token
    │  - Validates facility operating hours, active service windows, and capacity ceiling
    │  - Issues sequential token code (e.g., "A-104") with cryptographic QR verification hash
    ▼
[Stage 2: Predictive Wait-Time Estimation]
    │  - Evaluates real-time metrics: queue length (L), active counters (C), historical service velocity (S)
    │  - Computes Estimated Wait Time (EWT) incorporating peak-hour coefficients
    │  - Broadcasts dynamic countdown and initial queue position to client
    ▼
[Stage 3: Dynamic Multi-Counter Dispatching]
    │  - Monitors counter status signals (Idle, In-Service, Paused, Offline)
    │  - When counter reports IDLE:
    │  │   ├── Selects next eligible token matching counter service capabilities
    │  │   ├── Assigns token to specific Counter ID
    │  │   └── Initiates "You're Being Called" priority push notification
    │  └── If operator marks "No-Show / Absent":
    │      └── Triggers 3-minute grace period before terminal state archival
    ▼
[Stage 4: Proximity Alert & Notification Triggers]
    │  - Tracks position thresholds (3 positions away, 1 position away, now serving)
    │  - Routes alerts via WebSockets, SMS, WhatsApp, and Web Push notifications
    │  - Prompts student confirmation to eliminate zombie queue slots
    ▼
[Stage 5: Display & Kiosk Sanitization]
    │  - Masks student names, institutional IDs, and private inquiry descriptions
    │  - Streams public display payload (e.g., "Token A-104 → Desk 2") to waiting-room monitors
    ▼
Digital Public Displays & Personal Client Interfaces
```

### 2. Classification Rubrics & Dispatching Criteria

#### Token Priority & Routing Rubric
- **General Queue (Default)**: First-in, first-out (FIFO) sequential processing ordered strictly by issuance timestamp.
- **Priority Accommodations**: Tokens flagged for accessibility needs, physical mobility assistance, or scheduled reservations receive an algorithmic position boost ($+2$ tier ranking), routing them to dedicated ADA-compliant service windows.
- **Specialized Service Routing**: Tokens requesting specific administrative categories (Financial Aid, Registrar, Course Add/Drop, Visa Endorsement) are partitioned into dedicated sub-queues and routed exclusively to certified counter operators.

#### State Machine Transitions
Tokens transition through rigid, deterministic state boundaries:
1. `ISSUED`: Token generated and awaiting service window availability.
2. `CALLED`: Assigned to an active counter; notification dispatched.
3. `IN_SERVICE`: Operator confirms student arrival; timer active.
4. `SERVED`: Transaction completed; service duration logged for ML re-training.
5. `NO_SHOW`: Student failed to appear within grace window; token archived.
6. `CANCELLED`: Voluntarily dropped by user or revoked by administrator.

### 3. Wait-Time Estimation & Quality Scoring Formula

The agent calculates Estimated Wait Time ($EWT$) in minutes using dynamic queuing heuristics:

$$EWT = \left( \frac{L}{C} \right) \cdot S \cdot K_{\text{peak}} \cdot K_{\text{variance}}$$

- **Queue Depth ($L$)**: Active waiting tokens ahead in the designated category.
- **Active Counter Capacity ($C$)**: Number of verified counters currently serving that category.
- **Rolling Mean Service Duration ($S$)**: Trailing 30-day average transaction duration for the specific service type.
- **Peak Hour Adjustment ($K_{\text{peak}}$)**: Historical congestion multiplier calibrated for high-volume intervals ($1.15$ during midday class transitions, $1.00$ baseline).
- **Service Variance Factor ($K_{\text{variance}}$)**: Variance weight derived from standard deviation of the last 10 served tokens.

### 4. Thresholding & Refusal Decision Criteria

CampusOR Agent enforces clear operational guardrails:
- **Operating Hours Boundary**: Requests submitted outside published department service hours are refused with a clear advisory message displaying next-day opening times.
- **Maximum Daily Capacity Ceiling**: If the queue length exceeds the department's remaining operating minutes divided by average transaction duration, the agent halts token issuance: *"The queue for today has reached full capacity. Please reserve a slot for tomorrow."*
- **Duplicate Token Prevention**: Rate limits prevent the same authenticated user ID from holding more than one active token in the same department simultaneously.
- **Unverified Student Sessions**: Anonymous or unauthenticated API calls are blocked from obtaining priority administrative queue tokens.

### 5. Fallback Decision Mechanism

CampusOR Agent incorporates multi-tiered fault-tolerance mechanisms to guarantee zero facility stoppage:
- **Rule-Based FIFO Fallback**: If the machine-learning wait-time model or AI inference service becomes unavailable, the agent seamlessly degrades to pure deterministic FIFO ordering with linear time estimates ($5\text{ minutes} \times \text{position}$).
- **WebSocket Disconnection Re-link**: If network interruptions cause WebSocket drops between client kiosks and the central server, clients fall back to exponential-backoff polling (HTTP long-polling every 5 seconds) until reconnection is verified.
- **Offline Local Queue Cache**: Counter consoles maintain a local browser cache (IndexedDB / SQLite) of the current queue state, permitting operators to continue calling tokens even during temporary internet outages.
- **Model Fallback Cascade**: If the primary LLM copilot (`gemini-2.0-flash`) times out, requests automatically failover to secondary endpoints (`gpt-4o`, `claude-3-5-sonnet`).

### 6. Human-in-the-Loop Governance

CampusOR Agent operates strictly under human staff authority:
- **Operator Sovereign Control**: Staff operators retain full authority to pause counters, call specific urgent cases out of sequence, recall skipped tokens, or extend grace periods.
- **Administrator Global Freeze**: Department supervisors have access to a master emergency override that can pause queue intake, freeze ticket issuance, or reassign all waiting tokens across alternate departments.
- **Immutable Audit Trail**: Every queue event (token created, called, served, skipped, overridden) is written to structured, tamper-evident audit logs recording operator ID, token ID, timestamp, and duration.

---

## The Data It Uses

CampusOR Agent adheres strictly to data minimization and student privacy principles.

### 1. Ingested Input Data

The agent processes only operational telemetry necessary for queue sequencing:
- **Token Request Metadata**: Selected service category, transaction intent, and requested appointment window.
- **User Authentication Credentials**: Pseudonymized user ID and JWT session tokens confirming enrollment status.
- **Counter State Updates**: Operator availability pings, current token in-service status, and service start/completion timestamps.
- **Feedback & Completion Signals**: Binary resolution confirmation (Resolved / Escalated) entered by the operator upon case closure.

### 2. Configuration & Reference Data

- **Department Master Data**: Operating hours, active counter mappings, service category definitions, and benchmark target service durations.
- **Facility Capacity Limits**: Maximum concurrent waiting room limits set by campus safety codes.
- **Notification Endpoints**: Encrypted push subscription URLs, optional user phone numbers for SMS alerts, and WebSocket connection channels.

### 3. Base Model & Inference Lineage

- **Foundation Models**: Employs Google Gemini (`gemini-2.0-flash`) with fallback to OpenAI (`gpt-4o`) and Anthropic (`claude-3-5-sonnet`) for conversational student assistance and query routing.
- **Predictive Statistical Engine**: Utilizes rolling moving-average regression algorithms for wait-time predictions trained on anonymized historical service records.
- **No Private Data Training**: User transactions, student names, and inquiry details are never used to train or fine-tune commercial LLM models.

### 4. Data Privacy, Storage, and Retention

- **Zero PII on Public Displays**: Waiting-room displays and public kiosk interfaces display exclusively alphanumeric ticket codes (e.g., `A-104`). Student legal names, institutional IDs, and private inquiry details are never broadcast.
- **Data Minimization & Ephemeral Storage**: Completed queue tokens are purged of personally identifying references after 24 hours; only anonymized metrics (wait duration, service duration, service category) are retained for capacity analytics.
- **FERPA & GDPR Compliance**: In strict adherence to FERPA and GDPR (Articles 5 & 28), student educational records, grades, and confidential health disclosures are entirely excluded from queue system databases.

---

## Limitations

Understanding the operational boundaries and technical constraints of CampusOR Agent is essential for realistic deployment.

### 1. Inaccurate Wait Times from Outlier Transactions
- **Limitation**: Highly irregular or complex inquiries (e.g., complicated multi-semester credit transfers) can cause unexpected delays at an individual counter, skewing wait-time predictions for downstream users.
- **Mitigation**: The system continuously re-estimates wait times using moving averages with outlier clipping, proactively pushing delay notifications to waiting users when unexpected slowdowns occur.

### 2. User No-Shows and Zombie Tokens
- **Limitation**: Users who abandon the virtual queue without cancelling artificially inflate queue length and cause staff operators to call empty seats.
- **Mitigation**: The agent dispatches proximity notifications at 3-position intervals, requiring users to confirm attendance; unconfirmed tokens enter a brief grace period before automatic cancellation.

### 3. Intermittent Network Disconnects & Display Desynchronization
- **Limitation**: Campus Wi-Fi dead-zones or network drops can interrupt real-time WebSocket state on mobile devices or public TV displays.
- **Mitigation**: The application implements service workers with offline caching, automated reconnection handshakes, and immediate state synchronization upon reconnection.

### 4. Counter Imbalance from Unplanned Operator Absences
- **Limitation**: If an operator steps away from a counter without updating their console status to "Paused", assigned tokens can stall indefinitely.
- **Mitigation**: The agent detects counter inactivity exceeding 1.5x the average service duration, automatically flagging the stall, unbinding stranded tokens, and re-routing them to alternate active desks.

### 5. Non-Emergency Operational Scope
- **Limitation**: CampusOR is designed for scheduled administrative, academic, and non-emergency outpatient services. It does not possess medical triage algorithms or emergency distress handling capabilities.
- **Mitigation**: Health center queues display clear warning prompts instructing individuals experiencing acute physical or psychological emergencies to call campus emergency response immediately.

---

## Summary & Compliance Checklist

| Checkpoint 2 Requirement | Corresponding Section | Status |
| :--- | :--- | :---: |
| **How the agent decides** | [How the Agent Decides](#how-the-agent-decides) | **Covered** |
| - Decision architecture & 5-stage pipeline | Section 1 | Verified |
| - Classification rubrics & dispatching criteria | Section 2 | Verified |
| - Wait-time estimation & quality scoring formula | Section 3 | Verified |
| - Thresholding & refusal decision criteria | Section 4 | Verified |
| - Fallback decision mechanism | Section 5 | Verified |
| - Human-in-the-loop governance & operator control | Section 6 | Verified |
| **The data it uses** | [The Data It Uses](#the-data-it-uses) | **Covered** |
| - Ingested input data & token metadata | Section 1 | Verified |
| - Configuration & facility limits | Section 2 | Verified |
| - Base model lineage & statistical engine | Section 3 | Verified |
| - Data privacy, public masking & FERPA/GDPR | Section 4 | Verified |
| **Its limitations** | [Limitations](#limitations) | **Covered** |
| - Outlier transaction delays & variance | Section 1 | Verified |
| - User no-shows & zombie tokens | Section 2 | Verified |
| - Network drops & display desynchronization | Section 3 | Verified |
| - Counter imbalance & staff departures | Section 4 | Verified |
| - Non-emergency scope boundary | Section 5 | Verified |
